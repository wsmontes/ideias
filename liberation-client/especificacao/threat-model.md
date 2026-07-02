# Security Threat Model: Liberation Client

> **Document Status:** v1.0  
> **Classification:** INTERNAL - Security Architecture  
> **Last Updated:** 2026-07-01  
> **Target App:** Serverless, multi-protocol social media client (iOS/Android)

---

## Table of Contents

1. [System Model & Trust Boundaries](#1-system-model--trust-boundaries)
2. [Threat Actor Enumeration](#2-threat-actor-enumeration)
3. [Attack Surface Enumeration (Per Protocol)](#3-attack-surface-enumeration-per-protocol)
4. [Cross-Cutting Attack Surfaces](#4-cross-cutting-attack-surfaces)
5. [Nostr Deep-Dive (Critical Section)](#5-nostr-deep-dive-critical-section)
6. [Risk Matrix & Mitigations](#6-risk-matrix--mitigations)
7. [Security Testing & Audit Plan](#7-security-testing--audit-plan)
8. [Incident Response Plan](#8-incident-response-plan)

---

## 1. System Model & Trust Boundaries

### 1.1 Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        USER INTERFACE                                │
│              (Flutter / SwiftUI / Jetpack Compose)                   │
├─────────────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                     RUST CORE (FFI Boundary)                    │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐  │  │
│  │  │ Nostr    │ │ AT Proto │ │ Activity │ │   Matrix (Olm/   │  │  │
│  │  │ (secp256 │ │ (signing │ │ Pub      │ │   Megolm, E2EE)  │  │  │
│  │  │ k1 ops)  │ │ keys)    │ │ (OAuth2) │ │                  │  │  │
│  │  ├──────────┤ ├──────────┤ ├──────────┤ ├──────────────────┤  │  │
│  │  │ IRC      │ │ XMPP     │ │ Cross-   │ │   Key           │  │  │
│  │  │ (SASL,   │ │ (SCRAM,  │ │ Protocol │ │   Management    │  │  │
│  │  │ raw TCP) │ │ OMEMO)   │ │ Bridge   │ │   & Storage     │  │  │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────┐  ┌──────────────────────────────┐ │
│  │       SQLite (Local)         │  │    Platform Keychain         │ │
│  │  ┌────────────────────────┐  │  │  ┌────────────────────────┐ │ │
│  │  │ Events / Posts / DMs   │  │  │  │ iOS Secure Enclave     │ │ │
│  │  │ Media Cache             │  │  │  │ Android StrongBox / TEE│ │ │
│  │  │ FTS5 Search Index       │  │  │  │ macOS Secure Enclave   │ │ │
│  │  │ Read Markers / Drafts   │  │  │  │ Biometric Auth         │ │ │
│  │  └────────────────────────┘  │  │  └────────────────────────┘ │ │
│  └──────────────────────────────┘  └──────────────────────────────┘ │
├─────────────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    OPERATING SYSTEM                             │  │
│  │        Sandbox / App Container / Permissions Model              │  │
│  ├─────────────────────────────────────────────────────────────┤  │
│  │                   HARDWARE (SE / TEE)                          │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
          │                     │                     │
          ▼                     ▼                     ▼
    ┌──────────┐        ┌──────────┐          ┌──────────┐
    │ Nostr    │        │ATP PDS   │          │Matrix    │
    │ Relays   │        │PLC Dir   │          │Homeserver│
    └──────────┘        └──────────┘          └──────────┘
          │                     │                     │
    ┌──────────┐        ┌──────────┐          ┌──────────┐
    │Mastodon  │        │IRC       │          │XMPP      │
    │Instances │        │Servers   │          │Servers   │
    └──────────┘        └──────────┘          └──────────┘
```

### 1.2 Trust Boundaries

#### TB1: UI ↔ Rust Core (FFI Boundary)

**Nature:** The UI layer (Dart/Flutter, SwiftUI, or Jetpack Compose) invokes Rust functions via FFI (Foreign Function Interface). This is the primary interface for user interaction with cryptographic operations.

**Trust Assumption:** The UI layer is NOT trusted with raw key material. All key operations (signing, encryption, decryption) MUST occur inside the Rust core. The UI receives only prepared display data and user intent (e.g., "sign this event").

**Threats:**
- Malicious UI component sending crafted input to Rust core (buffer overflow, injection)
- UI requesting signature on attacker-controlled content without user consent
- UI leaking decrypted message content via memory introspection
- Rust core returning key material to UI layer (violation of principle)

**Mitigations:**
- FFI boundary enforces strict type checking (serde, protobuf, or flatbuffers)
- All signing operations require explicit user confirmation via OS-level biometric/authentication dialog
- Rust core NEVER returns private key material across FFI boundary
- Input validation on all FFI parameters (length checks, range validation)
- Memory is zeroized in Rust core before returning to UI

#### TB2: Rust Core ↔ Keychain (Native API Boundary)

**Nature:** The Rust core stores and retrieves key material from platform keychain services through native FFI bridges. On iOS this is the Security Framework (SecAccessControl, SecItemAdd/CopyMatching with kSecAttrAccessibleWhenUnlockedThisDeviceOnly); on Android it is AndroidKeyStore (KeyGenParameterSpec, authPerOp, StrongBox).

**Trust Assumption:** The keychain is trusted for secure storage. The OS is trusted not to leak keys to other apps. The user's device passcode/biometric is trusted.

**Threats:**
- Key extraction via memory dump during keychain read/write
- Biometric bypass (e.g., Lookalike / 3D-printed face, latex fingers)
- Jailbroken/rooted device disabling keychain access controls
- Keychain data persisted in unencrypted backups (iTunes/iCloud)
- App deleted but keychain residue remains accessible to re-installed app

**Mitigations:**
- Keys stored with `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` (iOS) / `KeyGenParameterSpec` with `devicePropertiesRequired` + `invalidatedByBiometricEnrollment` (Android)
- Key material held in Rust memory for minimum duration, zeroized after use
- Key tag-based access (touch ID / face ID required for each signing operation)
- Keys tagged per-protocol for isolation
- iOS: `kSecUseDataProtectionKeychain` set
- Android: `isStrongBoxBacked` = true on compatible devices

#### TB3: Protocol Adapters ↔ Network (TLS Boundary)

**Nature:** Each protocol adapter opens TLS connections to remote servers/relays. These connections carry authentication tokens, message content, and protocol-specific commands.

**Trust Assumption:** TLS is trusted for transport security. Certificate Authorities are trusted to validate server identity. The network path (WiFi, cellular, VPN) is UNTRUSTED.

**Threats:**
- TLS interception via rogue CA (corporate proxy, nation-state CA compromise)
- Certificate pinning bypass via Frida/Objection runtime manipulation
- SSLStrip downgrade attacks on protocols that allow fallback to plaintext
- WebSocket Secure (WSS) downgrade to WS via proxy manipulation
- TLS 1.2 downgrade forcing weak cipher suites
- OCSP response blocking (no-revocation-check attack)
- Traffic analysis: metadata correlation across protocol connections

**Mitigations:**
- Certificate pinning via `rustls` with pinned SPKI hashes for known infrastructure
- Reject TLS < 1.3 where possible; mandate TLS 1.2 minimum with strong cipher suites
- No plaintext fallback for any protocol (IRC requires strict override)
- Certificate transparency monitoring for pinned domains
- Connection metadata isolation: separate TLS sessions per protocol, no correlation
- DNS-over-HTTPS (DoH) for relay/server discovery to prevent DNS poisoning
- Tor/VPN support for traffic analysis mitigation (user opt-in)

#### TB4: Cross-Protocol Data Flow (Protocol Isolation Boundary)

**Nature:** The app allows users to view and interact with content from multiple protocols in a unified feed. Data flows between protocol adapters through the cross-protocol bridge.

**Trust Assumption:** Content from one protocol should NOT be able to exploit vulnerabilities in another protocol's handler. A compromise in one protocol's adapter should NOT propagate to other adapters.

**Threats:**
- Malicious Nostr event containing payload that exploits ActivityPub HTML renderer
- Cross-protocol identity confusion: attacker posts as user across protocols via bridging
- DM from one protocol leaking metadata about another protocol's contacts
- Protocol A's vulnerability (e.g., Matrix's CVE-2022-39248) exploited via cross-protocol message relay
- Malicious bridge spoofing identity across protocols

**Mitigations:**
- Protocol adapters run in separate sandboxed threads/processes
- Cross-protocol messages transit through sanitized intermediate representation (no raw protocol data crossing boundaries)
- HTML/Markdown sanitization applied per-protocol before cross-protocol rendering
- All cross-protocol messages tagged with source protocol (user always sees which protocol a message came from)
- No automatic trust: links/mentions from Protocol A cannot authenticate user to Protocol B
- Cross-protocol identity binding requires explicit user verification

#### TB5: App ↔ OS (Sandbox Boundary)

**Nature:** The app operates within the OS sandbox. iOS uses App Sandbox containers; Android uses application sandbox with SELinux policies.

**Trust Assumption:** The OS sandbox is trusted to isolate the app from other apps. The OS is trusted to enforce permission model (camera, microphone, photos, contacts, notifications).

**Threats:**
- Sandbox escape via kernel exploit (e.g., CVE-2025-XXXX)
- App data accessible via iOS backup (iTunes encrypted/unencrypted)
- iOS: Keychain accessible via backup if access control flags not properly set
- Android: ADB backup extracting SQLite database (if `allowBackup=true`)
- Android: Shared storage (MediaStore) leakage
- iOS: UIPasteboard containing sensitive content (cross-app leakage via clipboard)
- Notification content visible on lock screen

**Mitigations:**
- `allowBackup=false` and `fullBackupContent=false` in Android manifest
- iOS: Disable NSFileProtectionNone, use `NSFileProtectionCompleteUntilFirstUserAuthentication`
- Android: Use `android.content.Context.MODE_PRIVATE` for all files
- SQLite database encrypted at rest via SQLCipher (not plain SQLite)
- Clipboard cleared after paste; disable universal clipboard
- Sensitive notifications disabled by default; user opt-in with content hidden on lock screen
- Backup exclusion: keychain material not exported in backups (use `ThisDeviceOnly` flags)

#### TB6: Device ↔ Physical World (Physical Boundary)

**Nature:** The device can be lost, stolen, or accessed by an unauthorized person with physical possession. This includes forensic extraction via Cellebrite/GrayKey, JTAG attacks, and chip-off analysis.

**Trust Assumption:** The device is trusted only when in the user's physical possession and unlocked. Physical possession = full compromise of software-based protections.

**Threats:**
- Stolen device: forensic extraction of SQLite database
- Stolen device: passcode bruteforce via GrayKey (iOS) or custom recovery (Android)
- Evil maid: device tampered with hardware keylogger/keypad overlay
- USB debugging enabled: ADB access extracting app data
- iOS: Checkm8/checkra1n bootrom exploit (iPhone 4s-X) allows filesystem read
- Android: unlocked bootloader allows custom recovery and data extraction
- SIM swap: SMS-based 2FA bypass, social engineering of support

**Mitigations:**
- SQLCipher database encryption with key derived from passcode + device UID (not stored separately)
- Remote wipe capability if remote data is accessible (note: serverless = minimal remote data)
- iOS: iCloud Find My activation lock (user responsibility)
- Android: Factory Reset Protection (user responsibility)
- USB debugging disabled in release builds; ADB backup blocked
- All sensitive material protected by biometric + device passcode
- Auto-lock after short timeout (recommend < 1 minute)
- Data self-destruct after N failed passcode attempts (iOS optional, Android varies)
- Warning UX on first launch: explain physical device compromise risk

### 1.3 Data Flow Diagram (High-Level)

```
User Action
    │
    ▼
┌──────────────┐     User Intent (e.g., "publish text note")
│     UI       │──────────────────────────────────────────────┐
│  (Flutter/   │                                              │
│   SwiftUI)   │◄─────────────────────────────────────────────│
└──────┬───────┘     Rendered content + metadata              │
       │                                                      │
       │ FFI call (serde-encoded)                             │
       ▼                                                      │
┌─────────────────────────────────────────────────────────────┤
│                    RUST CORE                                 │
│                                                              │
│  1. Receive user intent                                      │
│  2. Authenticate via biometric (if required)                  │
│  3. Retrieve signing key from Keychain (transient)           │
│  4. Build protocol-specific payload                          │
│  5. Sign payload (key never crosses FFI boundary)            │
│  6. Zeroize key material                                     │
│  7. Encrypt payload (if DM/E2EE)                             │
│  8. Dispatch to protocol adapter                             │
│  9. Adapter opens TLS to remote server/relay                 │
│ 10. Send payload                                              │
│ 11. Receive response, decrypt if needed                      │
│ 12. Validate signature (ALL messages verified)               │
│ 13. Store in SQLite (local first)                            │
│ 14. Return display payload to UI                              │
└──────┬──────────────────────────────────────────────────────┘
       │
       │ TLS connection
       ▼
┌──────────────────────────────────────────────────────────────┐
│                 Remote Server / Relay / PDS                   │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Threat Actor Enumeration

| ID | Actor | Motivation | Capabilities | Primary Target | Likelihood (1-5) |
|----|-------|-----------|-------------|---------------|------------------|
| TA1 | Remote Network Attacker | Data theft, surveillance, credential harvesting | Passive/active network interception; can observe unencrypted traffic, inject packets, perform MitM with sufficient resources | Traffic content, authentication tokens, DM plaintext | 5 (ubiquitous) |
| TA2 | Malicious Relay/Server Operator | Content manipulation, user surveillance, metadata collection | Full control of relay/server infrastructure; can omit, delay, reorder, or inject messages; logs all metadata | Timeline content, user metadata, IP addresses, connection patterns | 4 (common in federated systems) |
| TA3 | Compromised Device (Malware) | Key theft, data exfiltration, credential harvesting | App sandbox escape via OS exploit; memory dump; filesystem access; keylogging; screen capture | Private keys (all protocols), local SQLite DB, auth tokens, passphrase | 3 (targeted, not mass-market) |
| TA4 | Physical Attacker (Stolen Device) | Identity theft, data extraction, account takeover | USB debugging, jailbreak/root, forensic tools (Cellebrite, GrayKey), chip-off analysis | Keychain, SQLite DB, media cache, auth tokens | 3 (opportunistic theft) |
| TA5 | Nation-State Adversary | Mass surveillance, censorship, legal coercion | Network infrastructure control, legal demands to relay operators, zero-day exploits, CA compromise, identity documents | All user data, identity mapping, social graph, communication patterns | 2 (high-resource, targeted) |
| TA6 | Rogue App Developer (Supply Chain) | Backdoor insertion, data harvesting, cryptocurrency theft | Code signing, app store distribution, dependency manipulation, CI/CD access | All user data, private keys, crypto funds via Nostr zap manipulation | 1 (high barrier, career-ending risk) |
| TA7 | Protocol-Level Adversary | Exploit protocol weaknesses, fund theft, reputation damage | Deep protocol knowledge, custom relay/server infrastructure, academic research expertise | Users of specific protocol (Nostr keys, Matrix sessions, XMPP OMEMO) | 3 (active research community) |
| TA8 | Malicious Bridge/Gateway Operator | MitM cross-protocol communication, identity spoofing | Control of bridge infrastructure between protocols (e.g., Nostr↔Matrix bridge) | Cross-protocol messages, identity mappings, content injection | 2 (niche infrastructure) |
| TA9 | Malicious Third-Party Library | Dependency hijacking, data exfiltration | Compromised npm/cargo/pub package; typo-squatting; maintainer account takeover | Build-time injection affecting all users | 2 (supply chain attacks increasing) |
| TA10 | OS-Level Attacker (Platform Vendor) | Mass surveillance, data collection | Full control of OS, keychain, sandbox; can read all app data at OS level | All app data, user behavior, communication graph | 1 (trust model assumption, but Snowden-class revelations exist) |

---

## 3. Attack Surface Enumeration (Per Protocol)

### 3.1 Nostr

#### 3.1.1 Key Management

| AS ID | Surface | Description | Severity |
|-------|---------|-------------|----------|
| N-001 | Key Generation - Weak Randomness | secp256k1 keypair generation using insufficient entropy (especially on embedded/budget Android devices) | Critical |
| N-002 | Key Generation - Poor RNG State | Forked process reusing RNG state producing identical keys; container restore snapshot reuse | Critical |
| N-003 | nsec Export via Clipboard | User copies nsec to clipboard; clipboard monitoring malware exfiltrates | High |
| N-004 | nsec Export via Screenshot | User screenshots nsec backup; screenshot harvested by malicious app | High |
| N-005 | nsec Export via Share Sheet | User shares nsec via OS share sheet; shared with malicious app | High |
| N-006 | Key Migration Between Devices | nsec transmitted over insecure channel during setup (QR code scanned in public) | High |

**Mitigations:**
- N-001: Use OS-provided `SecRandomCopyBytes` (iOS) / `SecureRandom` (Android) for seeding; validate against weak RNG states
- N-002: Fork-safety: re-seed RNG after process fork; container-aware RNG
- N-003: Warn user on nsec export; never store nsec in clipboard automatically; nsec QR for import only in secure environment
- N-004: Detect screenshot of nsec backup screen; warn user; use secure overlay
- N-005: Block share sheet on screens showing key material
- N-006: AirDrop-style direct connection for key migration; never transmit nsec over Nostr relays

#### 3.1.2 Event Signing & Verification

| AS ID | Surface | Description | Severity |
|-------|---------|-------------|----------|
| N-007 | Signature Verification Bypass | Client accepts events without verifying signature (per Black Hat 2025: multiple Nostr clients ship with verification disabled) | Critical |
| N-008 | Delegated Event Verification | Client verifies delegator's signature but skips delegatee's signature on delegated events | High |
| N-009 | Replaceable Event Validation | Client accepts replaceable events with higher `created_at` without verifying author matches original | High |
| N-010 | Event ID Collision | Crafted event with same ID as legitimate event but different content | Medium |
| N-011 | Signature Malleability | secp256k1 signature malleability (low-S vs high-S forms); replay of modified event | Medium |

**Mitigations:**
- N-007: **MANDATORY**: Every incoming event MUST have signature verified via `secp256k1.verify()` before storage or display. Events with invalid signatures are dropped with logged warning. No "optimization" path skips verification.
- N-008: For NIP-26: verify BOTH delegator's signature ON delegation token AND delegatee's signature ON event body. Reject if either is invalid.
- N-009: Track `pubkey + kind` tuple for replaceable events; verify pubkey matches existing entry before replacement; reject pubkey changes
- N-010: Event ID is `sha256(serialized_event)`. Store by ID + pubkey to prevent substitution.
- N-011: Normalize signatures to low-S form on verification; reject high-S signatures.

#### 3.1.3 Direct Message Encryption

| AS ID | Surface | Description | Severity |
|-------|---------|-------------|----------|
| N-012 | NIP-04 CBC Malleability | CBC-mode AES allows bit-flipping; attacker can modify encrypted DM to produce predictable plaintext changes | Critical |
| N-013 | NIP-04 IV Reuse | Deterministic IV derivation allows ciphertext comparison; same plaintext = same ciphertext | High |
| N-014 | NIP-04 Plaintext Recovery | With CBC malleability + known plaintext structure, attacker can recover full plaintext (Black Hat 2025) | Critical |
| N-015 | NIP-44 Implementation Errors | New spec has complex padding, HMAC, and key derivation; implementation bugs likely | Critical |
| N-016 | NIP-44 Nonce Reuse | Nonce reuse in ChaCha20 (if NIP-44 uses it) completely breaks encryption | Critical |
| N-017 | Key Reuse Between NIPs | Same key material used for NIP-04 and NIP-44 allows cross-protocol attacks | High |
| N-018 | DM Metadata Leak | Who talks to whom, frequency, message sizes leaked via relay metadata | Medium |

**Mitigations:**
- N-012/N-013/N-014: **NIP-04 DEPRECATED.** Migration to NIP-44 enforced. If NIP-04 messages received, flag them in UI with deprecation warning but attempt decryption with authenticated encryption verification (reject unauthentic messages).
- N-015/N-016: NIP-44 implementation MUST be formally verified against test vectors. Use audited library (e.g., `rust-nip44` or reference implementation). Key derivation uses HKDF with proper domain separation.
- N-017: **Cryptographic domain separation**: generate separate sub-keys for NIP-04, NIP-44, event signing, and NIP-49 (encrypted private key). Use `convergent_derivation(key=master_seed, domain="nostr://nip44/dm")`, etc.
- N-018: Implement NIP-32 (labeling) for DM metadata obfuscation; support padded messages to hide true length.

#### 3.1.4 Relay Interactions

| AS ID | Surface | Description | Severity |
|-------|---------|-------------|----------|
| N-019 | Malicious Relay Omitting Events | Relay selectively filters events (censorship, user-targeted suppression) | High |
| N-020 | Malicious Relay Injecting Fake Events | Relay creates events with arbitrary pubkeys and timestamps | Critical |
| N-021 | Malicious Relay Returning False Subscription End | Relay sends `EOSE` before all events delivered; client stops waiting | Medium |
| N-022 | Relay Connection Metadata | Relay logs IP, connection time, subscription filters, event IDs requested | High |
| N-023 | Relay Poisoning via NIP-65 | Malicious relay added to user's relay list; user's events routed through attacker relay | High |
| N-024 | WebSocket Memory Exhaustion | Relay sends unbounded data causing OOM crash | Medium |
| N-025 | WebSocket Message Injection | Relay injects JavaScript via event content field | Medium |

**Mitigations:**
- N-019: Connect to multiple independent relays (minimum 3); cross-verify event sets; flag relays that consistently omit events
- N-020: Verify event signatures independently of relay; reject events with invalid signatures regardless of source
- N-021: Set timeout on EOSE; implement adaptive waiting based on relay performance
- N-022: Support Tor/proxy for relay connections; minimize subscription info sent; use NIP-42 (AUTH) sparingly
- N-023: User must explicitly approve new relay additions to NIP-65 list; alert on mass relay changes; rate-limit relay list modifications
- N-024: Implement message size limits per protocol; streaming deserialization; memory budget per relay connection
- N-025: HTML/Markdown content sanitized before rendering; no raw HTML rendering without sanitization

#### 3.1.5 Identity & Verification

| AS ID | Surface | Description | Severity |
|-------|---------|-------------|----------|
| N-026 | Public Key Substitution | No binding between human-readable identity and pubkey; attacker claims any pubkey is "the real Alice" | Critical |
| N-027 | NIP-05 DNS Poisoning | Attacker poisons DNS for NIP-05 identifier domain; user gets wrong pubkey mapping | High |
| N-028 | NIP-05 Domain Expiration | Original owner lets domain expire; attacker registers and maps their pubkey | High |
| N-029 | NIP-05 JSON Injection | Malformed or oversized response from NIP-05 endpoint causes parsing issues | Medium |
| N-030 | NIP-02 Contact List Poisoning | Attacker injects fake contact lists with malicious pubkey recommendations | High |
| N-031 | NIP-07/NIP-46 Signer Compromise | External signer app (browser extension, nostr-connect) has broader permissions or is malicious | Critical |

**Mitigations:**
- N-026: Implement "verified identity" badges using NIP-05 + cross-referencing with known reliable relays + out-of-band verification (user compares keys via signal/website). Show unverified status for unverified pubkeys.
- N-027: Use DoH for NIP-05 resolution; cache NIP-05 responses with TTL; warn on NIP-05 change
- N-028: Monitor domain expiration dates; alert user before verification expires; re-verify periodically
- N-029: Strict JSON parsing with size limits; validate response schema
- N-030: Verify contact list events (kind 3) signatures; user approval for new contacts; rate-limit contact list changes
- N-031: When using NIP-07/46, implement strict permission model: request-specific signing only, no bulk signing; user confirms each signature request; audit log of all signing operations

### 3.2 AT Protocol

| AS ID | Surface | Description | Severity |
|-------|---------|-------------|----------|
| A-001 | DID Resolution - PLC Directory Compromise | PLC directory tampered; attacker remaps DID to malicious PDS | Critical |
| A-002 | DID Resolution - DID Document Tampering | Malicious PDS returns tampered DID document with attacker-controlled signing key | Critical |
| A-003 | Handle Resolution - DNS Hijacking | Attacker hijacks handle's DNS to point to malicious PDS | High |
| A-004 | Handle Resolution - Domain Takeover | Domain owner loses domain; attacker takes over and controls handle | High |
| A-005 | PDS - Malicious Repository | PDS serves tampered repository records (posts, follows, etc.) | Critical |
| A-006 | PDS - Record Suppression | PDS selectively omits records from repository | High |
| A-007 | PDS - Data Exfiltration | Malicious PDS operator reads all repository content (not encrypted by default) | High |
| A-008 | Firehose / Relay Injection | Malicious relay injects fake events into firehose (relays aggregate PDS data) | High |
| A-009 | Record Signing - Key Extraction | Signing key extracted from device; attacker creates records as user | Critical |
| A-010 | Record Signing - Signature Stripping | Client accepts unsigned records; attacker modifies record content | High |
| A-011 | Blob Upload - Malicious Content | Attacker uploads malicious blob (XSS, malware) via CDN | Medium |
| A-012 | App Password - Brute Force | Weak app password guessed by attacker | Medium |
| A-013 | App Password - Phishing | User enters app password on fake website | High |
| A-014 | OAuth2 Token Persistence | Token stored insecurely; extracted via malware | High |

**Mitigations:**
- A-001/A-002: Use AT Protocol SDK for DID resolution with validation; support DID:PLC and DID:Web; verify signature on DID document; verify DID document is signed by the DID's registered recovery key
- A-003/A-004: Use DoH for handle resolution; verify handle-to-DID mapping via DID document
- A-005: Verify Merkle integrity of repository records (MST verification); compare repository root across multiple PDS endpoints if available
- A-006: Subscribe to multiple relay endpoints; cross-verify record sets
- A-007: Consider client-side encryption for sensitive content (not in spec, but implementable via custom record types)
- A-008: Verify record signatures from firehose; cross-reference with independent relay sources
- A-009: Key in keychain; never export signing key; biometric confirmation for signing operations
- A-010: Reject unsigned records; verify signature on EVERY record before storage
- A-011: Validate blob MIME type; size limits; scan for malware if feasible; render blobs in sandboxed view
- A-012: Encourage strong app passwords; implement rate limiting in client (pause after N failed attempts at protocol level)
- A-013: Educate user: app password used ONLY in app settings; never enter on external sites; in-app OAuth approval flow
- A-014: Token stored in keychain, not in shared prefs; token scope minimized; token refresh rotation

### 3.3 ActivityPub (Mastodon API)

| AS ID | Surface | Description | Severity |
|-------|---------|-------------|----------|
| AP-001 | OAuth2 Token Theft via Malware | Access token extracted from device storage | Critical |
| AP-002 | OAuth2 Token Scope Bypass | CVE-2025-62176: Streaming API bypasses token scope restrictions | High |
| AP-003 | OAuth2 Token Phishing | User tricked into authorizing malicious OAuth2 app | High |
| AP-004 | Instance Compromise - DM Reading | Malicious instance admin reads user's DMs (ActivityPub DMs transit through server) | Critical |
| AP-005 | Instance Compromise - Timeline Manipulation | Instance admin modifies timeline (boosts, filters, injects content) | High |
| AP-006 | HTTP Signatures - Signature Stripping | Attacker removes HTTP Signature from request; server accepts unsigned request | High |
| AP-007 | HTTP Signatures - Replay Attack | Attacker replays captured signed request with different content | Medium |
| AP-008 | HTML Sanitization - XSS | Malicious post content contains JavaScript; client renders without sanitization | Critical |
| AP-009 | HTML Sanitization - CSS Injection | CVE-2025-46340 variant: CSS injection via UrlPreviewService or post content | High |
| AP-010 | HTML Sanitization - Bypass | New HTML tag/attribute combination bypasses sanitizer | High |
| AP-011 | WebFinger - Server Spoofing | Attacker returns fake WebFinger response mapping to malicious instance | High |
| AP-012 | WebFinger - Identity Confusion | Multiple accounts with same username on different instances confused | Medium |
| AP-013 | Streaming API - WebSocket Hijacking | Attacker intercepts WebSocket connection to streaming API | High |
| AP-014 | Streaming API - Event Injection | Attacker injects fake streaming events | High |
| AP-015 | Media - Remote Content Loading | Post contains remote image URL; loading it leaks IP + exposes to content changes | Medium |

**Mitigations:**
- AP-001: Token stored in keychain; token scope minimized (read-only preferred); rotate tokens periodically
- AP-002: CVE-2025-62176 fix: validate token scope on streaming API connections; implement app-level filtering even if server doesn't
- AP-003: OAuth2 flow with PKCE; verify redirect URI; show user the requested scope before authorization
- AP-004: **DISPLAY CLEAR WARNING**: DMs sent via ActivityPub are visible to instance admins. Recommend encrypted DMs via cross-protocol (Nostr DM) for truly private communication. Never promise E2EE for ActivityPub DMs.
- AP-005: Verify content signatures where possible; cross-reference timeline with multiple instances; flag suspicious boosts
- AP-006: Reject unsigned responses; require HTTP Signatures on all API calls where possible; server config dependent
- AP-007: Use `(request-target)` and `Date` header in HTTP Signatures; validate timestamp within 5-minute window; reject replay
- AP-008: HTML MUST be sanitized through an allowlist-based sanitizer (e.g., `ammonia` in Rust or `sanitize-html` in JS). Strip ALL script tags, event handlers (`on*`), `javascript:` URLs, `<object>`, `<embed>`, `<iframe>`. Only allow safe tags (p, br, a, img src from allowlist, ul, ol, li, blockquote, code, pre, strong, em, h1-h6). Post-render through additional CSP.
- AP-009: Strip ALL `style` attributes and `<style>` tags; CSS allowed only via app's own stylesheet
- AP-010: Use well-audited HTML sanitizer with up-to-date allowlist; fuzz sanitizer against known XSS vectors; subscribe to sanitizer security advisories
- AP-011: Use DoH for WebFinger; verify TLS certificate of returned instance; warn on instance mismatch
- AP-012: Display full `@user@instance.com` handle; never show just `@user`; distinguish instances visually
- AP-013: WSS only (WebSocket Secure); certificate pinning; re-auth on reconnection
- AP-014: Verify event signatures; validate event types; cross-reference with REST API polling
- AP-015: Media proxied through instance proxy (if supported); or user opt-in for image loading; warn before loading external content

### 3.4 Matrix

| AS ID | Surface | Description | Severity |
|-------|---------|-------------|----------|
| M-001 | Olm Key Reuse | Using same Olm session key for multiple messages | Critical |
| M-002 | Olm/Megolm Protocol Confusion | CVE-2022-39248: Attacker tricks recipient into treating Olm message as Megolm, or vice versa | Critical |
| M-003 | Megolm Ratchet Desynchronization | Encrypted message lost causing ratchet desync; user misses all subsequent messages | Medium |
| M-004 | Cross-Signing - Key Substitution | Malicious homeserver presents substitute device keys before user verifies | Critical |
| M-005 | Cross-Signing - Verification Bypass | User accepts verification request from attacker-controlled device | High |
| M-006 | SSSS - Brute Force Recovery Passphrase | Weak recovery passphrase allows SSSS decryption | High |
| M-007 | Key Backup - Fake Backup | CVE-2022-39248 variant: malicious homeserver serves fake key backup | Critical |
| M-008 | Key Backup - Backup Version Rollback | Attacker restores old version of key backup, removing recent keys | Medium |
| M-009 | Room State - State Resolution Attack | Attacker exploits state resolution to gain moderator privileges | Critical |
| M-010 | Room State - Split-Brain Exploitation | Network partition causes room split-brain; attacker controls one side | High |
| M-011 | Federation - Malicious Server Joins Room | Malicious homeserver invited to room; eavesdrops on encrypted content (if it has keys) | High |
| M-012 | Federation - Server ACL Bypass | Attacker's server connects despite not being in server ACL | Medium |
| M-013 | Access Token Theft | Token extracted from device; attacker hijacks session | Critical |
| M-014 | Access Token - Session Hijacking | Stolen token used to read all room history (if server stores it) | Critical |
| M-015 | Push Gateway - Notification Content | Encrypted messages notifiable but content leaks via push notification metadata | Medium |
| M-016 | Matrix URI Scheme - Injection | `matrix://` URI handler opens malicious room with autojoin | Medium |

**Mitigations:**
- M-001: Never reuse Olm session keys; enforce one-time ephemeral key per message; use audited `vodozemac` library (Rust)
- M-002: Implement strict message type discrimination: Olm messages have distinct wire format from Megolm messages. Validate message type BEFORE decryption. Do not attempt Olm decryption on Megolm messages or vice versa. Reference Matrix spec section 11.2.3 error handling.
- M-003: Implement out-of-band key request for missed Megolm keys; forward- and backward-secrecy ensured via ratchet
- M-004: Cross-signing MUST complete before trusting device keys. Display visual verification (emoji/emoji + number comparison). User MUST perform verification out-of-band (video/voice call, in-person).
- M-005: Require active user confirmation for verification requests; auto-reject verification requests from unknown devices; notify on new device verification
- M-006: Enforce strong recovery passphrase requirements (min 12 characters, entropy > 80 bits); consider multi-factor SSSS (passphrase + device key)
- M-007: Validate key backup authenticity: verify backup version signature against user's cross-signing master key. Download backup version info BEFORE accepting key backup. Verify `auth_data` signature.
- M-008: Notify user on backup version rollback; require explicit confirmation; never auto-accept older backup version
- M-009: Use Matrix spec-compliant state resolution (v2); pin trusted state; monitor federation API for anomalous state events; reject state events with future timestamps
- M-010: Implement split-brain detection: monitor device connectivity; on reconnection, verify room state consistency across multiple servers; alert user on conflicts
- M-011: Client-side ACL: maintain list of allowed servers per room; warn on new server joining room; require user approval
- M-012: Verify server ACL events; query server ACL from multiple servers; ignore joins from non-ACL servers
- M-013: Token stored in keychain, NOT in shared preferences or local storage; implement token refresh; detect token usage from unknown IP/device
- M-014: Token bound to device ID; server-side session invalidation on token change; implement token rotation every 24 hours
- M-015: Notification content can include room name + sender; strip message preview from notifications for encrypted rooms; user configurable
- M-016: URI handler must show room details before joining; require user confirmation for all room joins; block auto-join

### 3.5 IRC

| AS ID | Surface | Description | Severity |
|-------|---------|-------------|----------|
| I-001 | Plaintext Traffic | Connecting to servers without TLS; passive eavesdropping on all traffic | Critical |
| I-002 | TLS Fallback via 6697 | Server advertises TLS on 6697 but also accepts plaintext on 6667; user's client may fall back | High |
| I-003 | SASL Credential Theft | SASL (PLAIN) credentials transmitted in base64 over plaintext connection | Critical |
| I-004 | SASL Mechanism Downgrade | Attacker forces downgrade from SCRAM-SHA-256 to PLAIN | High |
| I-005 | Channel Injection | Malicious server injects fake messages into channel | High |
| I-006 | Private Message Interception | Malicious server reads all PMs (no E2EE in IRC) | Critical |
| I-007 | Nick Impersonation | No cryptographic identity binding; any user can claim any nick | High |
| I-008 | Bouncer Compromise (BNC) | Attacker accesses bouncer; gains full message history and active sessions | Critical |
| I-009 | CTCP Flood | Malicious CTCP messages cause client crash or resource exhaustion | Medium |
| I-010 | DCC File Transfer - Malware | User receives malware via DCC from seemingly trusted nick | High |
| I-011 | IRC URI Scheme - Injection | `irc://` URI connects to malicious server/channel | Medium |
| I-012 | WHOIS Leak | Attacker uses WHOIS to discover user's IP address and real host | Medium |

**Mitigations:**
- I-001: **MANDATE TLS**: Reject connections to non-TLS ports. Refuse plaintext 6667 connections entirely. Only allow 6697 (TLS) or custom TLS ports.
- I-002: Never fall back to plaintext. If TLS connection fails, show error; do not connect. User must explicitly enable plaintext (with severe warning).
- I-003: SASL credentials stored in keychain; transmitted only over TLS; prefer SASL EXTERNAL (client cert) where supported
- I-004: Prefer SCRAM-SHA-256; reject PLAIN if SCRAM is available; warn on SASL mechanism downgrade; pin SASL mechanism per server
- I-005: No cryptographic mitigation for channel injection (IRC has no signing). Flag messages from "ghost" connections; alert on unusual message patterns; display connection status per user
- I-006: **DISPLAY CLEAR WARNING**: IRC PMs are not encrypted. Never transmit sensitive information via IRC PM. Recommend cross-protocol encrypted DMs (Nostr NIP-44) for private communication.
- I-007: Display "unverified" indicator for all IRC users; support SASL account tracking; highlight nick changes; no trust assumptions based on nick
- I-008: Prefer direct server connections over bouncers; if bouncer required, connect to bouncer over TLS with strong auth; warn user about bouncer security implications
- I-009: Rate-limit CTCP messages; drop malformed CTCP; disable CTCP by default (user opt-in)
- I-010: Warn on incoming DCC requests; scan file for known malware; open file in sandboxed viewer; never auto-accept DCC
- I-011: URI handler must show server/channel details before connecting; require user confirmation; block auto-connect
- I-012: Do not expose real host via WHOIS responses from client; use virtual hosting if server supports it

### 3.6 XMPP

| AS ID | Surface | Description | Severity |
|-------|---------|-------------|----------|
| X-001 | OMEMO Pre-Key Bundle Manipulation | Malicious server substitutes attacker's pre-key bundle for recipient | Critical |
| X-002 | OMEMO Session Key Compromise | Attacker obtains OMEMO session key; can decrypt all messages in session | Critical |
| X-003 | OMEMO Key Reuse | Same OMEMO key used across multiple sessions; cross-session decryption | High |
| X-004 | SASL - Mechanism Downgrade | Attacker forces downgrade from SCRAM-SHA-256 to DIGEST-MD5 or PLAIN | High |
| X-005 | SASL - Credential Sniffing | Credentials intercepted if TLS not enforced or weakened | Critical |
| X-006 | Stanza Injection | Malicious server injects stanzas (messages, presence, IQ) | High |
| X-007 | XML Attacks - Billion Laughs | XML entity expansion causes denial of service | High |
| X-008 | XML Attacks - XXE | XML external entity injection reads local files or SSRF | High |
| X-009 | XML Attacks - XSLT Injection | Malicious XSLT transformation on stanza content | Medium |
| X-010 | Presence Leaks | User's online/offline status observable by anyone subscribed | Medium |
| X-011 | Presence - Location Tracking | Presence information correlated with IP geolocation | Medium |
| X-012 | MAM - Server Archive Compromise | Server-side message archive accessed by attacker | Critical |
| X-013 | MAM - Metadata Leak | Who talks to whom, when, frequency leaked from server archive | High |
| X-014 | Carbons - Message Duplication | Carbon copies of messages sent to attacker-controlled device | High |
| X-015 | WebSocket - Downgrade to WS | WSS (Secure WebSocket) downgraded to unencrypted WS | High |

**Mitigations:**
- X-001: Verify OMEMO pre-key bundle signature; fetch from multiple sources if available; implement blind trust-after-verification: trust bundle only after user verifies fingerprint
- X-002: Implement OMEMO ratchet properly (per XEP-0384); ratchet forward after each message; zeroize session keys after use; implement key freshness checks
- X-003: Generate fresh OMEMO keys per device session; never reuse identity key across sessions
- X-004: **MANDATE TLS**: Never connect without TLS. Prefer SCRAM-SHA-256(-PLUS); reject PLAIN; XEP-0175 (SASL) negotiation with strongest mechanism selection
- X-005: SASL credentials stored in keychain; SCRAM-SHA-256 prevents plaintext password exposure (even to server); SASL EXTERNAL (client cert) when possible
- X-006: Validate stanza origin: verify 'from' attribute matches known JID for the connection; reject stanzas from unknown JIDs; implement XEP-0185 (dialback)
- X-007: Disable XML entity expansion in XML parser; set max entity depth and expansion count; use safe XML parser configuration
- X-008: Disable DOCTYPE declaration and external entity resolution in XML parser; reject XML containing `<!ENTITY` if not expected
- X-009: Disable XSLT processing; reject stanzas containing XSLT instructions
- X-010: Default presence to "only contacts" visibility; user must explicitly opt-in to public presence; implement XEP-0016 (privacy lists) for presence filtering
- X-011: Strip IP from presence information; use Tor/proxy for XMPP connections; minimize XEP-0030 (disco info) responses
- X-012: **DISPLAY CLEAR WARNING**: MAM archives on server are readable by server admin. Recommend OMEMO-encrypted messages only for sensitive content. OMEMO prevents server-side reading if properly implemented. Warn when MAM is enabled.
- X-013: Minimize metadata sent to server; use OMEMO-encrypted messages; implement XEP-0373 (OpenPGP for XMPP) for additional protection
- X-014: Bind only one device per account for sensitive communications; implement XEP-0280 (Carbons) opt-out; verify carbon target JID matches; reject unauthorized carbon copies
- X-015: WSS only; certificate pinning; never fall back to WS; reject 3xx redirect to WS

---

## 4. Cross-Cutting Attack Surfaces

### 4.1 Local Storage Attacks

| CS ID | Attack | Vector | Severity |
|-------|--------|--------|----------|
| CS-001 | SQLite File Extraction | Malware reads app's SQLite file from sandbox; file system backup extraction | Critical |
| CS-002 | FTS5 Search Index Leak | Full-text search index contains content of all messages; extracted via file access | High |
| CS-003 | Media Cache - Sensitive Images | Cached images from DMs or private groups stored in media cache folder | High |
| CS-004 | Draft Storage - Unpublished Content | Drafts of sensitive messages stored in plaintext; exposed via backup | High |
| CS-005 | Read Markers - Browsing Behavior | Read markers (which posts/messages user has seen) reveal attention patterns | Medium |
| CS-006 | Identity Links - Cross-Protocol Associations | Local DB links user's Nostr pubkey with Matrix user ID, Mastodon handle, etc. | Medium |
| CS-007 | Clipboard - Sensitive Content | User copies message content to clipboard; cross-app clipboard access | High |
| CS-008 | Screenshot - Content Leak | OS screenshot or screen recording captures sensitive content | Medium |
| CS-009 | App State - Background Snapshot | iOS app switcher shows screenshot of last screen; leaks content | Medium |

**Mitigations:**
- CS-001: SQLCipher encrypts the entire SQLite database. Encryption key derived from device passcode + keychain-stored salt. NOT simply wrapped in platform keychain.
- CS-002: FTS5 index is within encrypted SQLCipher database (not separately accessible). If performance requires separate FTS index, encrypt with derived key.
- CS-003: Media cache files encrypted; stored in app's sandboxed cache directory with `NSFileProtectionCompleteUntilFirstUserAuthentication` (iOS) or `MODE_PRIVATE` (Android). Media from DMs additionally encrypted with per-message key.
- CS-004: Drafts stored in encrypted SQLCipher database; draft preview cleared from notification center; drafts not included in app state save.
- CS-005: Read markers stored in encrypted database; anonymized on export; user can clear read marker history.
- CS-006: Cross-protocol identity links in encrypted database; user configures whether these are stored; exported profiles strip internal linking.
- CS-007: Clear clipboard after paste operation; warn user on copy of sensitive messages; iOS 14+ clipboard access notifications.
- CS-008: iOS: `UIView.userInteractionEnabled = false` during sensitive screen display; screenshot detection with warning; Android: `FLAG_SECURE` on sensitive screens.
- CS-009: iOS: `UIApplication.shouldLaunchAppSwitcher` hook; blur sensitive content in app switcher snapshot; Android: `FLAG_SECURE` or custom live thumbnail.

### 4.2 Memory Attacks

| CS ID | Attack | Vector | Severity |
|-------|--------|--------|----------|
| CS-010 | Private Keys in RAM | Keys loaded from keychain remain in Rust memory after signing | Critical |
| CS-011 | Passphrase in Memory | Backup/restore passphrase held in memory during operation | High |
| CS-012 | Decrypted Message Content | Decrypted DMs held in UI buffers for rendering | High |
| CS-013 | Key Derivation Material | Key derivation intermediates stored in plain memory | High |
| CS-014 | Core Dumps / Crash Reports | Crash dumps may contain sensitive in-memory data | Medium |
| CS-015 | Swap / Compressed Memory | OS swaps app memory to disk; swapped pages contain sensitive data | Medium |

**Mitigations:**
- CS-010: Rust core implements `zeroize` trait on all key types; `Drop` implementations call `zeroize()`; `mlock()`/`VirtualLock` key material pages to prevent swapping; explicit memory fence after zeroize
- CS-011: Passphrases zeroized as soon as key is derived; never store passphrase; use `zeroize` on passphrase buffers
- CS-012: Decrypted content in Rust core (not UI) as display-only buffer; displayed content zeroized after user leaves view; UI receives only rendered string without retained decryption capability
- CS-013: Temporary key material isolated to stack (less traceable than heap); zeroized before function return; use `[u8; N]` with explicit zeroize vs `Vec<u8>` (heap)
- CS-014: Crash reporter configured to strip sensitive fields; memory dumps excluded from crash reports; use `std::panic::set_hook` to clean up before abort
- CS-015: `mlock()`/`VirtualLock` key material; minimize sensitive data lifetime; use locked memory pools for crypto operations

### 4.3 Supply Chain Attacks

| CS ID | Attack | Vector | Severity |
|-------|--------|--------|----------|
| CS-016 | Malicious Rust Crate | Dependency hijacking in `Cargo.toml`; typo-squatting; compromised maintainer | Critical |
| CS-017 | Compromised Flutter/Dart Package | Malicious pub.dev package; dependency confusion attack | Critical |
| CS-018 | CI/CD Pipeline Compromise | Attacker gains access to build pipeline; inserts backdoor into binary | Critical |
| CS-019 | App Store Binary Substitution | Attacker intercepts app submission; substitutes tampered binary | High |
| CS-020 | Developer Machine Compromise | Attacker compromises developer workstation; injects malicious code | Critical |
| CS-021 | Git Repository Compromise | Attacker pushes malicious commit; history rewrite | Critical |
| CS-022 | Code Signing Certificate Theft | Attacker uses stolen cert to sign malicious app variant | High |

**Mitigations:**
- CS-016: `Cargo.lock` pinned; `cargo-audit` in CI with fail-on-vulnerability; `cargo-deny` for license and dependency review; vendored critical dependencies; audit trail of dependency changes; use only well-known crypto libraries (no custom crypto)
- CS-017: `pubspec.lock` pinned; `dart pub deps` audit in CI; review transitive dependencies; prefer well-maintained packages with security policy
- CS-018: CI/CD signing keys in hardware token or cloud KMS (not in CI env vars); build reproducibility verification; signed commits required; CI triggers only from PRs with required reviews; bill of materials generation per build
- CS-019: App Store transparency: verify binary hash against source build; notarization checks; user-verifiable build hash on website
- CS-020: Developer machines: full disk encryption, strong passcode, firewall, EDR; code review for ALL commits; 2FA for all developer accounts; hardware security keys
- CS-021: Signed commits (GPG/SSH); branch protection rules; required PR reviews; `git verify-commit` in CI
- CS-022: Code signing cert in hardware token (YubiKey, SmartCard); not stored on CI/CD servers; cert revocation plan; short-lived certs with auto-renewal

### 4.4 Side-Channel Attacks

| CS ID | Attack | Vector | Severity |
|-------|--------|--------|----------|
| CS-023 | Timing Attack - Key Derivation | Attacker measures key derivation time over network to infer key material | Medium |
| CS-024 | Power Analysis - Signing Operation | Physical attacker measures power consumption during secp256k1 signing | Medium |
| CS-025 | Cache Timing - Message Processing | CPU cache timing correlates with message content | Low |
| CS-026 | Acoustic/EM Emanation | Keyboard sounds or EM emissions reveal typed content (mobile keyboards) | Low |
| CS-027 | Timing Attack - Signature Verification | Variable-time signature verification leaks information | Medium |

**Mitigations:**
- CS-023: Use constant-time comparison for all cryptographic operations; key derivation (Argon2id) is intentionally slow; constant-time is less critical here due to inherent slowness
- CS-024: TEE/SE-based signing where possible (iOS Secure Enclave supports secp256k1, Android StrongBox may); power analysis requires physical access = device likely already compromised
- CS-025: Use data-independent code paths for crypto operations; avoid branch-on-secret patterns; Rust cryptographic libraries (e.g., `dalek`, `rust-secp256k1`) are constant-time
- CS-026: Mobile keyboards are touch-screen (no acoustic emanations); on-screen keyboard acoustic analysis is impractical outside of targeted lab attack
- CS-027: Use constant-time signature verification (standard in `secp256k1` and `ed25519` libraries)

### 4.5 Network Attacks

| CS ID | Attack | Vector | Severity |
|-------|--------|--------|----------|
| CS-028 | TLS Certificate Validation Bypass | Disabled or weakened TLS validation (debug builds, malicious configuration) | Critical |
| CS-029 | Certificate Pinning Bypass | Runtime patching via Frida/Objection to bypass certificate pinning | Critical |
| CS-030 | DNS Poisoning - Relay Discovery | Attacker poisons DNS for well-known relay hosts; user connects to attacker relay | High |
| CS-031 | Traffic Correlation - Protocol Linking | Attacker correlates timing/volume patterns across protocol connections | Medium |
| CS-032 | WebSocket Downgrade Attack | WSS connection downgraded to WS via transparent proxy | High |
| CS-033 | TCP RST Injection | Attacker forcibly closes TCP connection; Denial of Service | Medium |
| CS-034 | BGP Hijacking | Attacker hijacks BGP routes to relay/server IPs; traffic intercepted | High |
| CS-035 | QUIC Connection Migration Hijack | For QUIC connections (if used), attacker hijacks connection migration | Medium |

**Mitigations:**
- CS-028: `rustls` (no OpenSSL, no platform TLS delegation for crypto validation); strict TLS 1.2+ only; no debug builds with relaxed validation; configuration locked against modification
- CS-029: Certificate pinning implemented at Rust level (not in platform networking stack); Frida bypass requires hooking Rust TLS; implement certificate transparency checking as secondary validation. Note: client-side pinning bypass is ultimately always possible on rooted/jailbroken devices - use as defense-in-depth.
- CS-030: DoH for all DNS resolution; pin DoH resolver; cache DNS results with TTL; validate TLS certificate of resolved IP matches pinned certificate
- CS-031: Connection timing jitter (add random delays); separate TLS sessions per protocol (no session reuse); randomize connection ordering
- CS-032: Reject non-WSS connections; explicit `wss://` only in configuration; never auto-upgrade `ws://` to `wss://` without user confirmation
- CS-033: Implement reconnection with exponential backoff; multi-relay fallback
- CS-034: RPKI/ROA validation not feasible client-side; use multi-homed connections via Tor/VPN; monitor certificate changes for pinned domains
- CS-035: Disable QUIC connection migration; or implement path validation before accepting migrated connection

---

## 5. Nostr Deep-Dive (Critical Section)

### 5.1 Overview

The 2025 Black Hat / EuroS&P research paper ("Nostr: A Security Analysis of the Decentralized Social Network Protocol") identified **five fundamental security vulnerabilities** affecting the Nostr protocol and its implementations. Liberation Client must address ALL of these.

### 5.2 Finding 1: Public Key Substitution Attack

**Vulnerability:** Nostr has no built-in mechanism to bind a human-readable identity to a public key. The same pubkey may be claimed by multiple relay operators, and users cannot cryptographically verify that `pubkey A` belongs to "Alice" without an out-of-band trust mechanism.

**Severity:** Critical  
**Black Hat 2025 Context:** Researchers demonstrated that a malicious relay could serve a fake profile for any Nostr user, substituting their pubkey with the attacker's. Clients that automatically trust relay-served profiles would display the attacker's content as if from the legitimate user.

**Attack Scenario:**
1. Attacker operates a malicious relay (or compromises an existing relay)
2. User subscribes to the relay for user "alice@example.com"
3. Relay returns Alice's metadata event (kind 0) but with attacker's pubkey
4. Client displays attacker's profile as if it's Alice
5. Attacker posts harmful content appearing to come from Alice

**Mitigation Strategy:**

```
┌─────────────────────────────────────────────────────────────────┐
│                     Identity Verification Flow                    │
│                                                                   │
│  User sees profile "Alice" with pubkey 0xabc...                   │
│     ├── Is there a NIP-05 for this pubkey?                        │
│     │      YES → Verify DNS: does _nostr.alice.com resolve        │
│     │             to 0xabc...?                                    │
│     │         ├── YES + domain HTTPS-secured + TLSA valid        │
│     │         │    → Mark "Verified" (green badge)                │
│     │         ├── YES but domain has issues (near-expiry, HTTP)   │
│     │         │    → Mark "Partially Verified" (yellow badge)     │
│     │         └── NO → Mark "NIP-05 Claimed, Unverified"          │
│     │                                                             │
│     └── NO NIP-05 → Check cross-references:                       │
│             ├── Does this pubkey appear in MY contact list?       │
│             │    → Show "In Your Contacts" indicator              │
│             ├── Does this pubkey have a consistent identity       │
│             │    across multiple trusted relays?                   │
│             │    → Show "Multi-Relay Consistent"                  │
│             ├── Do I share a mutual contact (followed by both)?   │
│             │    → Show "Mutual Contact"                          │
│             └── None of the above                                 │
│                  → Show "Unverified" (red badge)                   │
│                                                                   │
│  ALL pubkey displays show at least first 8 hex chars of pubkey    │
│  User can tap badge to see verification details                    │
│                                                                   │
│  Out-of-band verification: compare pubkey via Signal / website     │
│  / in-person QR code                                               │
└─────────────────────────────────────────────────────────────────┘
```

**Implementation Requirements:**
1. Every displayed profile MUST show a verification badge
2. NIP-05 verification MUST use DNS-over-HTTPS (prevent DNS poisoning)
3. NIP-05 responses MUST be cached with TTL; MUST be re-verified periodically
4. Domain expiration MUST be monitored; user warned when verification expires
5. Relay-level profile data is NEVER trusted without cross-verification
6. User's manually verified pubkeys are stored locally (pinned identities)

### 5.3 Finding 2: Missing Signature Verification

**Vulnerability:** Multiple Nostr clients (including popular ones) either skip signature verification entirely or have code paths where events are processed before verification. The Black Hat research found that 3 of 6 tested clients accepted unverified events.

**Severity:** Critical  
**CVE Status:** No single CVE; pattern identified across multiple implementations

**Attack Scenario:**
1. Attacker crafts an event with arbitrary pubkey `0xdeadbeef...`
2. Event contains no valid signature (or signature is empty/malformed)
3. Client stores event without verification
4. UI displays event as if from `0xdeadbeef...`
5. Event remains in user's timeline; if the real `0xdeadbeef...` is a trusted user, attacker's content is trusted

**Mitigation Policy (Enforced at Architecture Level):**

```
┌─────────────────────────────────────────────────────────────┐
│              SIGNATURE VERIFICATION POLICY                    │
│                                                               │
│   1. DROP: Events with missing or malformed signature fields  │
│   2. DROP: Events where signature.verify(pubkey, event_id)    │
│      returns false                                            │
│   3. DROP: Events where pubkey doesn't match event's          │
│      claimed author (pubkey in event JSON)                    │
│   4. DROP: Events with created_at > 5 minutes in the future   │
│      (clock drift allowance)                                  │
│   5. DROP: Events with created_at < NIP-xx minimum           │
│                                                               │
│   VERIFY ORDER (performance optimized):                       │
│   1. JSON parse → 2. Validate structure → 3. Check pubkey     │
│      existence → 4. Verify created_at → 5. Compute event_id   │
│      (SHA256) → 6. Verify signature (secp256k1)               │
│                                                               │
│   STORE ORDER (after verification):                           │
│   Write to SQLite only after ALL verifications pass            │
│                                                               │
│   NO OPTIMIZATION PATH SKIPS VERIFICATION                     │
│   NO "BATCH VERIFY" THAT DROPS INDIVIDUAL VERIFICATION        │
│   NO DEFERRED VERIFICATION (verify at ingest, not at render)  │
└─────────────────────────────────────────────────────────────┘
```

**Performance Consideration:** secp256k1 verification is ~100x slower than SHA256. Optimization strategies (WITHOUT skipping verification):
- Batch verification using `secp256k1-batch` (schnorr batch verification for NIP-40)
- Parallel verification across relay connections
- Verification queue with priority for user's direct feed over discovery content
- Deduplicate event IDs before verification (same event from multiple relays)

**Proof of Verification (Testing):**
Insert a test that creates an event with invalid signature, submits it via mock relay, and asserts the event is REJECTED by the client. This test must be in CI.

### 5.4 Finding 3: DM Plaintext Recovery via CBC Malleability (NIP-04)

**Vulnerability:** NIP-04 specifies AES-256-CBC for DM encryption. CBC mode is malleable: an attacker can modify the ciphertext and predict how the plaintext will change after decryption. Combined with known plaintext structure (JSON), an attacker can recover the full plaintext via chosen-ciphertext techniques.

**Severity:** Critical  
**Black Hat 2025 Demonstration:** Researchers demonstrated end-to-end plaintext recovery of NIP-04 encrypted DMs with moderate computational effort (~2^40 operations).

**Technical Detail:**
```
NIP-04 Encryption:
  iv = random 16 bytes
  key = shared_secret(SHA256(secp256k1_ECDH(privkey_a, pubkey_b)))[0:32]
  ciphertext = AES-256-CBC(key, iv, plaintext_padded)
  hmac = HMAC-SHA256(key[0:16], ciphertext)

Attack:
  1. Flip specific bytes in CBC ciphertext block N
  2. This predictably corrupts block N-1 plaintext
  3. By observing whether decryption succeeds (client crashes/errors) or fails
     (padding oracle), attacker recovers plaintext block by block
  4. Even without padding oracle: CBC malleability allows attacker to modify
     DM content (e.g., change "send to 0xabc..." to "send to 0xdef...")
```

**Mitigation Strategy:**

1. **DEPRECATE NIP-04 IMMEDIATELY:**
   - Liberation Client does NOT send NIP-04 DMs
   - All outgoing DMs use NIP-44
   - Old NIP-04 DMs in database are flagged with deprecation warning
   - When decrypting NIP-04 DMs (for legacy compatibility): verify HMAC, reject if invalid, but note HMAC verification is not a cure for CBC malleability (it only detects tampering, not prevents plaintext recovery)

2. **MIGRATION PATH TO NIP-44:**
   - When sending DM to a recipient, check if recipient supports NIP-44 (via kind 10050 relay list or NIP-44 capability)
   - If recipient supports NIP-44: use NIP-44
   - If recipient does not support NIP-44: display warning that DM will be sent with deprecated encryption (NIP-04); require explicit user confirmation
   - Display banner: "This recipient uses an old encryption method. Upgrade recommended."

3. **NIP-44 Implementation Verification:**
   - Test against NIP-44 reference implementation test vectors
   - Verify: padding scheme, nonce generation (random, not derived), key derivation per-message
   - Formal verification of constant-time HMAC comparison
   - Fuzz test: encrypt random messages, decrypt, verify roundtrip

### 5.5 Finding 4: Encrypted DM Forgery via Key Reuse

**Vulnerability:** Reusing the same key material across NIP-04 and NIP-44 allows attacker to forge DMs from one protocol using information from the other. The Black Hat research found that clients using the same nsec for all operations (signing, NIP-04, NIP-44) are vulnerable to cross-protocol attacks.

**Severity:** High

**Attack Scenario:**
1. Alice's client derives both NIP-04 and NIP-44 encryption keys from the same nsec
2. Attacker observes a signed event from Alice (public)
3. Using Alice's pubkey and the signed event, attacker derives session keys
4. Attacker can now forge NIP-04 DMs that appear to be from Alice (because the key is the same)
5. If Alice's client accepts the forged DM (due to shared key material), attacker achieves DM forgery

**Mitigation Strategy (Cryptographic Domain Separation):**

```
All keys derived from master seed:
                                                
    master_seed (nsec)                           
         │                                       
         ├── Domain: "nostr://signing"           
         │    → Event signing key (same as nsec) 
         │                                       
         ├── Domain: "nostr://nip04/encrypt"    
         │    → NIP-04 shared secret derivation   
         │                                       
         ├── Domain: "nostr://nip44/encrypt"    
         │    → NIP-44 encryption key             
         │                                       
         ├── Domain: "nostr://nip44/sign"       
         │    → NIP-44 signing key                
         │                                       
         ├── Domain: "nostr://nip49/encrypt"    
         │    → NIP-49 encrypted key backup       
         │                                       
         └── Domain: "nostr://relay/auth"       
              → NIP-42 relay authentication key   
```

Key separation uses HKDF-expand:
```
subkey = HKDF-expand(
    prk = HMAC-extract(salt=domain, ikm=master_seed),
    info = domain + ":v1",
    length = 32
)
```

Each subkey is generated fresh per session and zeroized after use. Subkeys are never stored; they are derived on-demand from the master seed (which is in keychain).

### 5.6 Finding 5: Key Replace Attack

**Vulnerability:** Nostr's replaceable events (kinds 0, 3, etc.) allow any event with the same pubkey and kind to replace a previous event. If an attacker obtains a user's private key (or convinces a relay to accept event from different pubkey), they can replace the user's profile, contact list, or relay list.

**Severity:** High

**Attack Scenario:**
1. Attacker obtains Alice's nsec (via malware, phishing, or backup compromise)
2. Attacker publishes a kind 0 (metadata) event from Alice's pubkey
3. Relays replace Alice's profile with attacker's content
4. Client fetches the latest kind 0 → displays attacker's content as Alice's profile
5. Attacker can also replace relay lists (NIP-65), routing Alice through attacker's relays

**Mitigation Strategy:**

```
┌─────────────────────────────────────────────────────────────┐
│                KEY REPLACE ATTACK MITIGATION                  │
│                                                               │
│  1. KEY COMPROMISE DETECTION:                                 │
│     - Monitor for unexpected replaceable events               │
│     - Alert user: "A new profile event was published from     │
│       your key. This may indicate key compromise."            │
│     - Show diff: what changed in old vs new metadata          │
│                                                               │
│  2. NIP-05 + OUT-OF-BAND VERIFICATION:                        │
│     - After profile change, re-verify NIP-05                  │
│     - If NIP-05 still resolves to user's pubkey: signal OK    │
│     - If NIP-05 fails: signal potential compromise            │
│     - User can "acknowledge" profile change (locally)         │
│                                                               │
│  3. ROTATION DETECTION:                                       │
│     - Monitor for "key rotation" events (NIP-06)              │
│     - Alert user on key rotation                              │
│     - Require out-of-band confirmation before trusting new key│
│                                                               │
│  4. LOCAL KEY PINNING:                                        │
│     - Store user's own pubkey locally (deterministic)         │
│     - Never accept events that claim different pubkey         │
│       for the user's own identity                             │
│     - For contacts: pin pubkey after user verification        │
│     - Alert on pubkey change for pinned contacts              │
│                                                               │
│  5. RELAY DIVERSITY:                                          │
│     - Publish to multiple independent relays                  │
│     - Cross-verify replaceable events across relays           │
│     - If one relay shows different profile: alert user        │
│                                                               │
│  6. EXPIRING EVENTS (NIP-40):                                 │
│     - Prefer relays that support NIP-40 (event expiration)    │
│     - Set expiration on own events to limit window of attack  │
│                                                               │
│  7. NIP-26 DELEGATION:                                        │
│     - If delegation is used, monitor delegation token         │
│     - Alert on unexpected delegation creation                 │
│     - Scoped delegations only (limited kinds, time-limited)   │
└─────────────────────────────────────────────────────────────┘
```

### 5.7 Nostr Security Checklist

| # | Requirement | Verification Method | Priority |
|---|------------|-------------------|----------|
| 1 | All inbound events MUST have signature verified | Code audit + CI test | P0 |
| 2 | Events with invalid signatures MUST be dropped | Integration test | P0 |
| 3 | NIP-04 DM sending is DISABLED by default | Code audit | P0 |
| 4 | NIP-44 implementation passes reference test vectors | Conformance test | P0 |
| 5 | Cryptographic domain separation is implemented | Code audit | P0 |
| 6 | Identity verification badges are displayed | UI test | P0 |
| 7 | Replaceable events are monitored for unexpected changes | Integration test | P1 |
| 8 | NIP-05 verification via DoH | Integration test | P1 |
| 9 | Contact list changes require user approval | UI test | P1 |
| 10 | Relay list changes require user approval | UI test | P1 |
| 11 | secp256k1 key generation uses OS RNG | Static analysis | P0 |
| 12 | Key material zeroized after operation | Static analysis | P0 |
| 13 | NIP-07/46 strict signing permissions | Integration test | P1 |
| 14 | Multi-relay cross-verification | Integration test | P2 |

---

## 6. Risk Matrix & Mitigations

### 6.1 Risk Assessment Methodology

- **Likelihood (L):** 1 (Rare) to 5 (Almost Certain)
- **Impact (I):** 1 (Negligible) to 5 (Catastrophic)
- **Risk Score:** L × I
- **Risk Level:** Low (1-6), Medium (8-12), High (15-20), Critical (25)

### 6.2 Risk Matrix

| # | Threat | L | I | Risk | Mitigation | Effort |
|---|-------|---|---|------|-----------|--------|
| 1 | Nostr nsec extraction via malware | 3 | 5 | **15 (High)** | Keychain with biometric auth; key zeroized after use; never display nsec in UI | 2 weeks |
| 2 | Matrix access token theft | 4 | 5 | **20 (High)** | Keychain storage; token rotation every 24h; device-bound tokens | 1 week |
| 3 | HTML XSS in ActivityPub posts | 4 | 5 | **20 (High)** | Allowlist-based HTML sanitizer (ammonia); CSP; no raw HTML rendering | 3 weeks |
| 4 | Physical device theft + forensic extraction | 3 | 5 | **15 (High)** | SQLCipher DB encryption; keychain with ThisDeviceOnly; secure passcode enforcement | 4 weeks |
| 5 | Malicious relay omitting events (Nostr) | 4 | 3 | **12 (Medium)** | Multi-relay subscription; cross-verification; relay health monitoring | 3 weeks |
| 6 | Nostr signature verification bypass | 3 | 5 | **15 (High)** | Mandatory verification at ingest; CI test; no deferred verification | 2 weeks |
| 7 | NIP-04 DM plaintext recovery (CBC malleability) | 3 | 5 | **15 (High)** | Deprecate NIP-04; migrate to NIP-44; warn on legacy receive | 3 weeks |
| 8 | DID document tampering (ATP) | 3 | 5 | **15 (High)** | Verify DID signature; cross-reference with PLC directory; cache with verification | 2 weeks |
| 9 | ATP PDS serving tampered repository | 3 | 5 | **15 (High)** | Merkle MST verification; multi-PDS cross-check | 3 weeks |
| 10 | Matrix Olm/Megolm protocol confusion (CVE-2022-39248) | 3 | 5 | **15 (High)** | Strict message type discrimination; validate before decrypt | 1 week |
| 11 | Matrix cross-signing key substitution | 3 | 4 | **12 (Medium)** | Out-of-band verification required; emoji comparison; device trust UX | 2 weeks |
| 12 | Matrix key backup served fake backup | 3 | 5 | **15 (High)** | Verify backup signature against cross-signing master key | 1 week |
| 13 | IRC plaintext traffic eavesdropping | 4 | 4 | **16 (High)** | Mandate TLS; reject plaintext connections; never fallback | 1 week |
| 14 | IRC SASL credential sniffing | 4 | 5 | **20 (High)** | SCRAM-SHA-256 preferred; SASL over TLS only | 1 week |
| 15 | XMPP OMEMO pre-key bundle substitution | 3 | 5 | **15 (High)** | Fingerprint verification; blind-trust-after-verification model | 2 weeks |
| 16 | XMPP XML Billion Laughs DoS | 3 | 3 | **9 (Medium)** | Disable XML entities; set parser limits | 0.5 week |
| 17 | SQLite FTS5 index search history leak | 3 | 3 | **9 (Medium)** | FTS in encrypted SQLCipher DB; clear history option | 1 week |
| 18 | Malicious Rust crate supply chain | 2 | 5 | **10 (Medium)** | cargo-audit; vendored deps; dependency review; MSRV pinning | Ongoing |
| 19 | CI/CD pipeline compromise | 2 | 5 | **10 (Medium)** | Build reproducibility; signed commits; KMS for signing; limited CI access | 3 weeks |
| 20 | TLS certificate pinning bypass via Frida | 4 | 4 | **16 (High)** | Rust-level pinning (rustls); CT monitoring; note: bypassable on rooted device | 2 weeks |
| 21 | DNS poisoning for relay/server discovery | 3 | 4 | **12 (Medium)** | DoH for all DNS; pin resolver; validate TLS cert matches pinned cert | 1 week |
| 22 | ActivityPub OAuth2 token scope bypass (CVE-2025-62176) | 4 | 3 | **12 (Medium)** | App-level scope filtering; token scope minimization; PKCE | 1 week |
| 23 | Misskey CSS injection (CVE-2025-46340) variant | 3 | 3 | **9 (Medium)** | Strip all style attributes/tags; app-owned stylesheet only | 1 week |
| 24 | Misskey token persistence after logout (CVE-2025-24896) | 3 | 4 | **12 (Medium)** | Clear tokens from keychain on logout; verify token cleared | 0.5 week |
| 25 | Cross-protocol identity confusion | 2 | 4 | **8 (Medium)** | Source protocol always displayed; no automatic cross-protocol trust | 2 weeks |
| 26 | Clipboard sensitive content leak | 3 | 2 | **6 (Low)** | Clear clipboard after paste; warning on copy; iOS clipboard notifications | 0.5 week |
| 27 | Bitchat-style "vibe coding" crypto flaws | 2 | 5 | **10 (Medium)** | Only use audited crypto libraries; no custom crypto; formal verification | Ongoing |
| 28 | Nostr public key substitution (no identity binding) | 4 | 4 | **16 (High)** | Verification badges; NIP-05 with DoH; multi-relay cross-verify; local pinning | 3 weeks |
| 29 | Nostr cryptographic domain separation violation | 2 | 5 | **10 (Medium)** | HKDF domain separation; key isolation per subprotocol | 2 weeks |
| 30 | Matrix state resolution attack | 2 | 4 | **8 (Medium)** | Spec-compliant state res v2; monitor anomalous state; multi-server validation | 3 weeks |

### 6.3 Priority-Ranked Mitigations

| Priority | Mitigation | Risk Score | Effort | Timeline |
|----------|-----------|-----------|--------|----------|
| **P0** | Mandatory signature verification on ALL inbound events (all protocols) | 15-20 | 2 weeks | Pre-release |
| **P0** | Keychain storage with biometric auth for all secrets | 15-20 | 2 weeks | Pre-release |
| **P0** | SQLCipher database encryption | 15 | 3 weeks | Pre-release |
| **P0** | NIP-04 deprecation + NIP-44 migration (Nostr DMs) | 15 | 3 weeks | Pre-release |
| **P0** | TLS mandate for all protocols (no plaintext fallback) | 16-20 | 1 week | Pre-release |
| **P0** | HTML sanitization with allowlist-based sanitizer | 20 | 3 weeks | Pre-release |
| **P1** | Certificate pinning via rustls | 16 | 2 weeks | Pre-release |
| **P1** | Cryptographic domain separation for all key derivation | 10-15 | 2 weeks | Pre-release |
| **P1** | Identity verification badges (Nostr) | 16 | 3 weeks | Pre-release |
| **P1** | Multi-relay cross-verification (Nostr) | 12 | 3 weeks | v1.1 |
| **P1** | Cross-signing validation with out-of-band checks (Matrix) | 12-15 | 2 weeks | Pre-release |
| **P1** | Token rotation for all auth tokens | 12-20 | 1 week | v1.1 |
| **P1** | DoH for DNS resolution | 12 | 1 week | Pre-release |
| **P1** | Dependency audit automation (cargo-audit, dart pub deps) | 10 | 1 week | Pre-release |
| **P2** | Key replacement attack monitoring | 10-12 | 2 weeks | v1.1 |
| **P2** | FTS5 in encrypted database | 9 | 1 week | v1.1 |
| **P2** | Connection timing jitter for traffic analysis mitigation | 8 | 1 week | v1.2 |
| **P2** | CI/CD build reproducibility | 10 | 3 weeks | v1.1 |
| **P2** | Matrix state resolution monitoring | 8 | 3 weeks | v1.2 |
| **P2** | Media cache encryption | 8 | 1 week | v1.1 |

---

## 7. Security Testing & Audit Plan

### 7.1 Static Analysis (SAST)

| Tool | Target | Cadence | Configuration |
|------|--------|---------|---------------|
| `cargo-audit` | Rust crates (known vulnerabilities) | Every CI run | Fail on any advisory; `--deny warnings` |
| `cargo-deny` | Rust dependency license + advisory | Every CI run | Include advisory DB; fail on unmaintained crates |
| `cargo-clippy` | Rust code quality + security lints | Every CI run | `-W clippy::pedantic -W clippy::cargo` + security-specific lints |
| `dartanalyzer` | Dart/Flutter code quality | Every CI run | Enable all lint rules; fail on error |
| `semgrep` | Custom security patterns | Every CI run | Rules for: missing verification, hardcoded secrets, unsafe deserialization |
| `sonarcloud` | Full code quality + security | Per PR | Security-focused quality gate |

**Static Analysis Test Cases (Custom Semgrep Rules):**
1. **Search for:** any code path where event/fetched content bypasses signature verification
2. **Search for:** cryptographic operations using non-constant-time comparison
3. **Search for:** HTML rendering without sanitization
4. **Search for:** raw TCP connections without TLS
5. **Search for:** logging of sensitive data (keys, tokens, passwords)
6. **Search for:** `unsafe` blocks in Rust (with justification required)
7. **Search for:** branch-on-secret patterns in crypto code

### 7.2 Dynamic Analysis (DAST)

| Tool | Target | Cadence |
|------|--------|---------|
| Frida | Runtime TLS pinning bypass, memory dump, function hooking | Per release |
| Objection | Mobile app security assessment (iOS/Android) | Per release |
| MobSF | Static + dynamic mobile app analysis (OWASP MASVS) | Per release |
| Burp Suite Pro | Network traffic interception, OAuth2 testing, injection testing | Per release |
| Drozer | Android attack surface (if Android version) | Per release |
| Needle | iOS penetration testing framework (if iOS version) | Per release |

**Dynamic Analysis Test Cases:**
1. **Certificate pinning bypass:** Attempt to intercept all TLS connections using Frida script that hooks certificate validation. Verify that rustls-level pinning catches the bypass.
2. **Memory dump:** Use Frida to dump process memory while app is active. Search for keys, tokens, passwords in memory dump. Verify zeroize is working.
3. **SQLite extraction:** Attempt to read SQLite files from sandbox (requires jailbreak/root). Verify SQLCipher encryption prevents reading.
4. **Keychain dump:** On jailbroken device, attempt to dump keychain contents. Verify biometric+passcode protections.
5. **OAuth2 manipulation:** Using Burp, attempt to replay OAuth2 tokens, modify token scope, extend token lifetime.
6. **WebSocket hijacking:** Attempt to intercept and inject WebSocket messages (all protocol adapters).
7. **HTML injection:** Send crafted ActivityPub posts with XSS payloads; verify sanitizer blocks all.
8. **XML injection (XMPP):** Send Billion Laughs, XXE, XSLT payloads; verify parser blocks.

### 7.3 Fuzzing

| Target | Tool | Strategy | Coverage Goal |
|--------|------|----------|---------------|
| Nostr event parser | `cargo-fuzz` | AFL++ + libfuzzer on event deserialization | Parsing errors, crashes, panics on malformed events |
| NIP-44 encryption/decryption | `cargo-fuzz` | Differential fuzzing against reference implementation | Output mismatch, panic, wrong plaintext |
| XML parser (XMPP) | `cargo-fuzz` | AFL++ on stanza parsing | Entity expansion, buffer overflow, panic |
| HTML sanitizer | `cargo-fuzz` | Unicode fuzzing + known XSS vectors | Sanitizer bypass, XSS success |
| JSON parsers (all protocols) | `cargo-fuzz` | Structural fuzzing (valid JSON + edge cases) | Parsing discrepancies, panic |
| Matrix Olm/Megolm | `cargo-fuzz` | Protocol-level message fuzzing | Type confusion, decryption failure, ratchet desync |
| TLS handshake | `rustls` fuzz targets | Certificate fuzzing, handshake fuzzing | TLS negotiation bypass, panic |

**Fuzzing Cadence:**
- Continuous: offline fuzzing on developer machines
- Weekly: 24-hour fuzzing campaigns on CI fuzzing cluster
- Pre-release: 72-hour fuzzing on all targets
- Protocol change: New fuzzing campaign for affected component

### 7.4 Formal Verification

| Component | Method | Goal |
|-----------|--------|------|
| NIP-44 implementation | Constant-time verification | No timing side channels in HMAC comparison, AEAD |
| Key derivation (HKDF) | Property-based testing | Domain separation enforced; subkeys are independent |
| Memory zeroize | `zeroize` crate audit | All paths zeroize; compiler not optimizing away |
| Signature verification | Property-based testing | Verification fails for: wrong key, wrong message, malleable sig |

### 7.5 Protocol-Specific Testing

| Protocol | Test | Tool / Method |
|----------|------|---------------|
| Nostr | Relay event omission detection | Run malicious relay test harness; verify client detects omission |
| Nostr | Signature verification bypass | Send events with invalid/missing signatures; verify rejection |
| Nostr | NIP-04 plaintext recovery | Verify client rejects CBC-malleable NIP-04 DMs |
| Nostr | NIP-44 conformance | Test against NIP-44 reference test vectors |
| Nostr | Identity verification | Test NIP-05 verification with various edge cases (DNS failure, timeout, malformed response) |
| ATP | DID resolution with tampered document | Serve tampered DID document; verify client detects inconsistency |
| ATP | MST integrity verification | Tamper with repository records; verify client detects |
| ActivityPub | OAuth2 token scope bypass | Attempt scope escalation; verify client rejects |
| ActivityPub | HTML sanitizer bypass | OWASP XSS cheatsheet against sanitizer; verify all vectors blocked |
| Matrix | Olm/Megolm confusion | Send Olm message labeled as Megolm; verify client detects |
| Matrix | Cross-signing bypass | Attempt to substitute device keys before verification; verify detection |
| Matrix | Fake key backup | Serve tampered key backup; verify client rejects |
| IRC | Plaintext fallback | Configure IRC server on 6667; verify client refuses connection |
| IRC | SASL downgrade | Force SASL mechanism downgrade; verify client rejects |
| XMPP | OMEMO pre-key substitution | Substitute pre-key bundle; verify client detects or warns |
| XMPP | XML entity expansion | Send Billion Laughs payload; verify parser rejects |

### 7.6 Audit Timeline

| Milestone | Audit Type | Scope |
|-----------|-----------|-------|
| **Pre-alpha** (architecture complete) | Design review | Threat model verification, architecture review, trust boundaries |
| **Alpha** (core features implemented) | Internal security audit | All P0 mitigations, SAST findings, dependency audit |
| **Beta** (all protocols functional) | External penetration test | Full protocol interaction, cross-cutting attacks, fuzzing results |
| **Release candidate** | Full external audit | Third-party security audit firm engagement |
| **v1.0 release** | Bug bounty launch | Public bug bounty program (HackerOne / Immunefi) |
| **Major update** (new protocol added) | Regression + new protocol audit | Full protocol-specific testing for new protocol + regression |
| **Quarterly** | Dependency audit | `cargo-audit`, `dart pub deps` review, vulnerability scan |
| **Protocol spec change** | Targeted security review | Affected components only |
| **Critical CVE** (any protocol) | Emergency review | Immediate impact analysis + patch |

### 7.7 Recommended External Auditors

| Firm | Specialty | Estimated Cost |
|------|-----------|---------------|
| Trail of Bits | Rust, cryptography, decentralized protocols | $50-100k |
| Radically Open Security | Open source, decentralized systems | $30-80k |
| Include Security | Mobile applications, protocol security | $40-90k |
| X41 D-Sec | Matrix, XMPP, cryptography | $40-80k |

---

## 8. Incident Response Plan

### 8.1 Key Compromise Response

#### Scenario: Known or suspected key compromise per protocol

| Protocol | Compromised Element | Immediate Actions | Recovery Steps |
|----------|-------------------|-------------------|----------------|
| **Nostr** | nsec (private key) | 1. DISABLE app networking immediately<br>2. Generate new nsec offline<br>3. Publish key rotation event (NIP-06)<br>4. Update NIP-05 with new key<br>5. Inform contacts via out-of-band | 1. Revoke old nsec on all relays (NIP-49)<br>2. Migrate contacts to new key<br>3. Sign new profile/contact events<br>4. Consider: did attacker see DMs?<br>5. If yes: sensitive DMs may be exposed permanently |
| **ATP** | Signing key | 1. Rotate signing key via PLC<br>2. Update DID document<br>3. Revoke old app passwords | 1. Create new PDS session<br>2. Re-sign repository records<br>3. Audit repository for tampered records |
| **ActivityPub** | OAuth2 token | 1. Revoke token (POST /oauth/revoke)<br>2. Generate new token with PKCE<br>3. Check for unauthorized actions | 1. Rotate app password<br>2. Enable 2FA on instance<br>3. Review recent activity |
| **Matrix** | Access token | 1. Logout all sessions (POST /logout/all)<br>2. Generate new login token<br>3. Rotate device keys | 1. Re-verify cross-signing<br>2. Reset key backup<br>3. Re-verify devices |
| **Matrix** | Olm/Megolm session keys | 1. Force new session with all contacts<br>2. Rotate identity keys<br>3. Reset key backup | 1. Key compromise may expose previous messages<br>2. Inform room admins |
| **IRC** | SASL password | 1. Change password on IRC server<br>2. Update stored credentials | 1. Server password change<br>2. Verify no unauthorized sessions |
| **XMPP** | OMEMO keys | 1. Generate new OMEMO identity key<br>2. Upload new pre-key bundles<br>3. Re-establish sessions with contacts | 1. Verify fingerprints with contacts<br>2. Rotate SASL password<br>3. Check for unauthorized devices |
| **XMPP** | SASL credentials | 1. Change XMPP server password<br>2. Update stored credentials in keychain | 1. Revoke other resources (XEP-0279)<br>2. Check MAM archive for unauthorized access |
| **All protocols** | Device passcode | 1. CHANGE DEVICE PASSCODE IMMEDIATELY<br>2. Biometric re-enrollment required<br>3. Generate new app encryption keys | 1. SQLCipher re-encryption (new key)<br>2. Re-verify all protocol identities |

#### General Key Compromise Response Workflow

```
KEY COMPROMISE DETECTED (user report or automated detection)
    │
    ▼
┌──────────────────────────────────────────────────────────┐
│ 1. ASSESS                                                        │
│ - Which keys/tokens were compromised?                            │
│ - How were they compromised? (phishing, malware, physical theft?) │
│ - What is the exposure window?                                   │
│ - What data was accessible with compromised keys?                │
└──────────────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────────────┐
│ 2. CONTAIN                                                       │
│ - Disable app networking (airplane mode)                        │
│ - Revoke all tokens and sessions                                 │
│ - Generate new keys (per protocol)                               │
│ - Inform contacts via out-of-band (Signal, phone call, in-person)│
│ - Publish protocol-specific recovery events                      │
└──────────────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────────────┐
│ 3. ERADICATE                                                      │
│ - Determine attack vector and close                              │
│ - Remove malware (if compromised via malware)                    │
│ - Reset infected device (factory reset if severe)                │
│ - Change all passwords for related services                      │
└──────────────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────────────┐
│ 4. RECOVER                                                       │
│ - Re-onboard keys across all protocols                           │
│ - Re-establish sessions with contacts                            │
│ - Re-verify identity (NIP-05, cross-signing, fingerprints)       │
│ - Restore data from backup (if backup was uncompromised)         │
└──────────────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────────────┐
│ 5. POST-MORTEM                                                    │
│ - Document incident timeline                                     │
│ - Identify security gaps                                         │
│ - Update threat model                                            │
│ - Implement new mitigations                                      │
└──────────────────────────────────────────────────────────┘
```

### 8.2 Data Breach Response

**Context:** Liberation Client is serverless. All data is local-first. This fundamentally limits breach impact.

#### What Data Could Be Exposed:

| Data Type | Storage | Breach Scenario | Exposure Impact |
|-----------|---------|----------------|-----------------|
| Private keys | Keychain (HW-backed) | Physical theft + forensic extraction | Full account takeover across all protocols |
| Auth tokens | Keychain | Malware with keychain access | Session hijacking for live protocols |
| Message database | SQLCipher-encrypted SQLite | Physical theft + passcode bruteforce | All messages, DMs, timeline content |
| Media cache | App sandbox | Physical theft + filesystem access | Cached images, videos (including DMs) |
| Search index | Encrypted SQLite FTS5 | Physical theft + decryption | Search history, revealed interests |
| Drafts | Encrypted SQLite | Physical theft + decryption | Unpublished content |
| Contacts/identity links | Encrypted SQLite | Physical theft + decryption | Social graph, protocol identity mapping |
| IP addresses | Not stored (ephemeral) | N/A | N/A |
| User activity patterns | Not stored (ephemeral) | N/A | N/A |

#### Breach Response Workflow:

```
BREACH CONFIRMED (forensic extraction of device)
    │
    ▼
1. Determine extraction method:
   - Cellebrite/GrayKey (iOS) → passcode likely compromised
   - Jailbreak/root → OS protections bypassed
   - Chip-off → hardware key extraction
    │
    ▼
2. Assess data exposure:
   - Was keychain extracted? 
     YES → ALL protocol keys compromised (full account takeover)
   - Was SQLCipher DB decrypted?
     YES → ALL local messages, DMs, contacts exposed
   - Was device passcode recovered?
     YES → All above + iCloud keychain accessible
    │
    ▼
3. User notification:
   - In-app alert on next launch (if device returned)
   - Email if user registered email (optional feature)
   - Note: we cannot notify users without their contact info
   - Suggestion: implement "breach alert" feature where user provides
     emergency contact (email, push notification token) during setup
    │
    ▼
4. Key revocation & recovery:
   - Follow Key Compromise Response (section 8.1) for all protocols
   - In-app guided recovery flow
   - Remote token revocation where possible
    │
    ▼
5. Post-breach improvements:
   - Analyze attack vector
   - Can we prevent similar attack?
   - Update threat model
   - Consider: rate limit on key derivation (Argon2id) for passcode bruteforce resistance
```

#### Breach Notification: Challenges & Solutions

**Challenge:** Liberation Client is serverless with no user database. We cannot email users to notify them of a breach.

**Solutions:**
1. **Emergency Contact (Opt-In):** During onboarding, allow user to optionally register an emergency contact method (email, push notification via Firebase/APNs). This is the ONLY server-side data we store. User must be informed of this tradeoff.
2. **Recovery Code:** Generate printable recovery codes (similar to Signal) that user stores in their password manager. If device is recovered, user enters code to re-establish trust.
3. **Out-of-Band Notification Protocol:** Nostr NIP-01 kind 5 (deletion) event can signal emergency. Publish from user's new key to indicate compromise. Problem: attacker could also publish. Mitigation: pre-shared secret between devices.

**Recommendation:** Implement #1 as opt-in with clear privacy disclosure.

### 8.3 Protocol Vulnerability Disclosure

#### When a CVE / Protocol Vulnerability is Disclosed

| Phase | Actions | Timeline |
|-------|---------|----------|
| **T-0: Disclosure received** | 1. Acknowledge receipt (if participating in disclosure)<br>2. Determine if our implementation is affected<br>3. Assess severity (CVSS) | 24 hours |
| **T+1: Impact analysis** | 1. Identify affected code paths<br>2. Determine if user data is at risk<br>3. Check if users need to take action | 48 hours |
| **T+2: Patch development** | 1. Implement fix<br>2. Write regression tests<br>3. Fuzz patch (if applicable)<br>4. Code review | 1-7 days (depending on severity) |
| **T+3: Internal testing** | 1. Integration tests<br>2. Manual penetration test if applicable<br>3. Verify no regression on other protocols | 2-3 days |
| **T+4: Release** | 1. Emergency app store release (if critical)<br>2. Patch notes with severity<br>3. If coordinated disclosure: coordinate with discloser | 1-2 days after testing |
| **T+5: User communication** | 1. In-app notification to update<br>2. Blog post / social media disclosure<br>3. GitHub Security Advisory | Same day as release |

#### Vulnerability Disclosure Policy

```
Liberation Client Vulnerability Disclosure Policy
==================================================

We follow Coordinated Vulnerability Disclosure (CVD).

Reporting: security@liberation-client.example (PGP key: FINGERPRINT)
Preferred: Detailed report with reproduction steps, impact analysis,
            and suggested fix (if available).
Triage: Within 48 hours of report.
Resolution: We commit to fixing Critical/High issues within 7 days
            of confirmation.
Disclosure: Coordinated with reporter. Default: 90-day disclosure
            window from fix release.
Recognition: Named in Security Advisories and Hall of Fame (with
             permission).
Scope: Liberation Client codebase, infrastructure, dependencies.
Out of scope: Protocol-level vulnerabilities in Nostr, Matrix, ATP,
              ActivityPub, IRC, XMPP specs (report to respective
              protocol maintainers).
```

### 8.4 Communication Plan

#### Internal Notifications

| Role | Notified When | Method |
|------|--------------|--------|
| Security lead | Any confirmed security incident | PagerDuty + Signal |
| Engineering team | Critical vulnerability requiring patch | Slack + email |
| Executive | High-severity breach or legal implications | Email + calendar meeting |
| Legal | Data breach, regulatory obligations | Email + calendar meeting |

#### External Notifications

| Audience | Notification Method | Content | Timing |
|----------|---------------------|---------|--------|
| Users (all) | In-app notification | Brief description + required action (if any) + patch availability | Within 24h of fix release |
| Users (affected) | In-app notification (contextual) | Specific actions needed (key rotation, contact verification) | Within 24h of fix release |
| Users (emergency contact opt-in) | Push notification / email | Critical security alert + required action | Immediate after assessment |
| General public | Blog post + social media | Technical summary + timeline + thanks to reporter | Within 7 days of fix release |
| GitHub | Security Advisory | CVE details + affected versions + patch | Same day as fix release |
| Press (if applicable) | Press release | For significant breaches affecting user base | Coordinated with legal |

#### What to Communicate

For each incident, prepare:
1. **What happened** (technical summary, no fear-mongering)
2. **What data was impacted** (be specific, avoid speculation)
3. **What users need to do** (actionable steps)
4. **What we've done** (fix, improvement, timeline)
5. **How we'll prevent recurrence** (systemic improvements)
6. **Credit** (to finder, if applicable)

### 8.5 Scenario Walkthrough: Nostr nsec Compromise

**Scenario:** User reports that someone posted from their Nostr account while they were offline.

```
TRIAGE
├── User confirmed: they did not make the post
├── User confirms: nsec was stored in keychain (not shared)
├── User confirms: device not lost, no known malware
├── User confirms: no phishing, didn't enter nsec anywhere
└── **Likely vector: Cloud backup compromise** (iCloud/Google Drive backup of keychain)

RESPONSE
├── [T+0h] Confirm nsec must be revoked
├── [T+0h] Guide user: generate new nsec offline
├── [T+1h] Guide user: install Malwarebytes / AV scan (confirm no device-level compromise)
├── [T+2h] Guide user: 
│   ├── Generate new nsec in app
│   ├── App publishes NIP-06 key rotation event
│   ├── App updates NIP-05
│   ├── App signs new profile (kind 0) with new key
│   ├── App signs new contact list (kind 3) with new key
│   └── App publishes to 5+ independent relays
├── [T+3h] Guide user: 
│   ├── Notify key contacts via out-of-band (Signal, call, in-person)
│   ├── "I've rotated my Nostr key. Please check my new pubkey via NIP-05."
│   └── Provide new pubkey for verification
├── [T+4h] Investigate root cause:
│   ├── Check iCloud keychain compromise
│   ├── Check for passcode bruteforce
│   ├── Check for phishing targeting user
│   └── Document findings

POST-MORTEM
├── Determine: was this a 1Password/Apple keychain cloud backup issue?
├── Determine: could client-side detection have identified this earlier?
├── Improvement: Add "unexpected event from your key" detection
├── Improvement: Rate-limit total events per day from own key
├── Improvement: NIP-05 verification on profile change
└── Improvement: Recommend users disable cloud backup of keychain (documentation)
```

---

## Appendix A: Glossary

| Term | Definition |
|------|------------|
| AEAD | Authenticated Encryption with Associated Data |
| CBC | Cipher Block Chaining (mode of operation for block ciphers) |
| DID | Decentralized Identifier (W3C standard) |
| DoH | DNS-over-HTTPS |
| E2EE | End-to-End Encryption |
| EOSE | End of Stored Events (Nostr relay message) |
| FFI | Foreign Function Interface |
| FTS5 | Full-Text Search version 5 (SQLite extension) |
| HKDF | HMAC-based Key Derivation Function |
| HMAC | Hash-based Message Authentication Code |
| KDF | Key Derivation Function |
| MAM | Message Archive Management (XEP-0313) |
| MST | Merkle Search Tree (AT Protocol) |
| NIP | Nostr Implementation Possibility |
| nsEC | Nostr SECret key (private key in bech32 format) |
| PDS | Personal Data Server (AT Protocol) |
| PKCE | Proof Key for Code Exchange (OAuth2 extension) |
| PLC | DID:PLC (AT Protocol DID method) |
| SASL | Simple Authentication and Security Layer |
| SCRAM | Salted Challenge Response Authentication Mechanism |
| SE | Secure Enclave (iOS hardware security module) |
| secp256k1 | Elliptic curve used by Bitcoin and Nostr |
| SSSS | Secure Secret Storage and Sharing (Matrix spec) |
| SQLCipher | Encrypted SQLite implementation |
| TEE | Trusted Execution Environment |
| TLS | Transport Layer Security |
| XXE | XML External Entity attack |
| XSS | Cross-Site Scripting |

## Appendix B: References

1. Black Hat 2025 / EuroS&P - "Nostr: A Security Analysis of the Decentralized Social Network Protocol"
2. CVE-2025-62176 - Mastodon Streaming API Token Scope Bypass
3. CVE-2022-39248 - Matrix Olm/Megolm Protocol Confusion
4. CVE-2025-46340 - Misskey CSS Injection via UrlPreviewService
5. CVE-2025-24896 - Misskey Token Persistence After Logout
6. OWASP Mobile Security Testing Guide (MSTG)
7. OWASP Mobile Application Security Verification Standard (MASVS)
8. NIST SP 800-175B - Guideline for Cryptographic Key Management
9. Matrix Spec - Section 11.2.3 (Encryption Error Handling)
10. NIP-04 - Nostr Encrypted Direct Messages
11. NIP-44 - Nostr Encrypted Payloads (Versioned)
12. NIP-26 - Nostr Delegated Event Signing
13. NIP-05 - Nostr Mapping to DNS-based Identifiers
14. AT Protocol - Repository Data Encoding (Merkle Search Tree)
15. XEP-0384 - OMEMO Encryption (XMPP)
16. XEP-0313 - Message Archive Management (XMPP)
17. RFC 5802 - SCRAM-SHA-256 (SASL Mechanism)
18. RFC 8446 - TLS 1.3
19. RFC 5869 - HKDF (HMAC-based Key Derivation Function)
20. Bitchat Incident Analysis - "Vibe Coding" Cryptographic Failures
