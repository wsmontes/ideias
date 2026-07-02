# Multi-Protocol, Serverless Social Media Client -- Architecture Analysis

> **Document status:** Draft v1  
> **Scope:** Architecture design for a client-side-only social media application connecting to ActivityPub, AT Protocol, Nostr, Matrix, IRC, and XMPP.  
> **Design principle:** Zero proprietary backend. All protocol bridges, data storage, and processing run on-device.

---

## Table of Contents

1. [Core Architecture Pattern](#1-core-architecture-pattern)
2. [Protocol Integration Strategy](#2-protocol-integration-strategy)
3. [Local-First Data Layer](#3-local-first-data-layer)
4. [Technical Stack Recommendations](#4-technical-stack-recommendations)
5. [Challenges & Risks](#5-challenges--risks)

---

## 1. Core Architecture Pattern

### 1.1 The Unified Abstraction Layer

Six radically different protocols must be surfaced through a single, coherent interface. The key insight is that **every protocol maps to the same conceptual primitives**, even though their wire formats, auth models, and delivery guarantees differ wildly.

```
+---------------------------------------------------------------+
|                        UI Layer                               |
|  (Timeline, DMs, Profile, Composer, Notifications, Search)   |
+---------------------------------------------------------------+
|                      Unified Data Model                        |
|  Post | Message | Profile | Reaction | Attachment | Channel  |
+---------------------------------------------------------------+
|                Transport Abstraction Layer (TAL)               |
|   ┌────────┐ ┌────────┐ ┌──────┐ ┌────────┐ ┌──────┐ ┌──────┐|
|   │ APub   │ │ AT Pro │ │Nostr │ │ Matrix │ │ IRC  │ │ XMPP ||
|   │Adapter │ │Adapter │ │Adapt │ │Adapter │ │Adptr │ │Adapt │|
|   └────────┘ └────────┘ └──────┘ └────────┘ └──────┘ └──────┘|
+---------------------------------------------------------------+
|                    Offline-First Local Store                   |
|           (SQLite + Per-protocol sync engines)                |
+---------------------------------------------------------------+
```

#### Abstraction primitives (the `ProtocolAdapter` interface)

Every adapter implements this trait/interface:

```typescript
interface ProtocolAdapter {
  // Identity
  connect(credentials: CredentialMap): Promise<SessionState>;
  disconnect(): Promise<void>;
  identity: IdentityHandle;

  // Timeline
  getTimeline(opts: TimelineQuery): AsyncIterable<Post>;
  getThread(postId: string): AsyncIterable<Post>;
  
  // Posts / Messages
  publish(content: PostContent): Promise<PostReceipt>;
  delete(postId: string): Promise<void>;
  react(postId: string, emoji: string): Promise<void>;
  
  // Social graph
  getProfile(handle: string): Promise<Profile>;
  follow(target: IdentityRef): Promise<void>;
  getFollowers(handle: string): AsyncIterable<Profile>;
  
  // DMs / Direct Messages
  getConversations(): AsyncIterable<Conversation>;
  sendMessage(to: IdentityRef, content: MessageContent): Promise<MessageReceipt>;
  
  // Real-time
  subscribe(channel: SubscriptionChannel): AsyncIterable<Event>;
  
  // Protocol-specific metadata
  capabilities: ProtocolCapabilities;
}
```

#### Why an interface (not a shared base class)

Each protocol's semantics differ too much for a shared class to be cleaner than an interface. ActivityPub models posts as "Create" activities in an object graph; AT Protocol uses a firehose of repository commits; Nostr is a set of signed JSON events; Matrix is a state-resolution room model; IRC is raw line-oriented channels. Forcing shared inheritance creates leaky abstractions. An interface with protocol-specific escape hatches (`protocolAdapter.raw()` returning the native SDK handle) keeps the abstraction honest.

#### Protocol capabilities matrix

| Capability | APub | AT Pro | Nostr | Matrix | IRC | XMPP |
|---|---|---|---|---|---|---|
| Public timeline | Yes | Yes | Yes | No | Yes | No |
| Direct messages | Yes (via mentions) | Yes | Yes (NIP-04/17) | Yes | Yes (whisper/query) | Yes |
| Threaded replies | Yes | Yes | Yes | Yes | Limited | Limited |
| Rich text | HTML/Markdown | Facets | NIP-30 | HTML/rich | Plain | XHTML-IM |
| Reactions | Yes | Yes | Yes (NIP-25) | Yes | No | Yes |
| Media attachments | Yes | Yes | Yes (NIP-95) | Yes | DCC | Yes |
| End-to-end encryption | No | No | Optional (NIP-44) | Yes (Olm/Megolm) | No | Yes (OMEMO) |
| Offline delivery | Server queues | Relay/PDS | Relay stores | Server stores | Bouncer | Server stores |
| Search | Limited | Yes (atmosphere) | NIP-50 | Full-text | No | No |
| Edits | Limited | Yes | NIP-33 | Yes | No | Yes |

### 1.2 Event-Driven vs Polling

A hybrid approach is required -- not all protocols support push delivery.

| Protocol | Primary mode | Fallback | Poll interval |
|---|---|---|---|
| ActivityPub | WebSocket streaming (Mastodon API) | HTTP polling for C2S | 30-60s |
| AT Protocol | Firehose WebSocket | XRPC cursor polling | 60s |
| Nostr | WebSocket (subscription) | N/A | N/A (always-on WS) |
| Matrix | Sync API (long-poll) | N/A | N/A (sync loop) |
| IRC | Persistent TCP connection | N/A | N/A |
| XMPP | Persistent TCP (BOSH/WebSocket) | HTTP BIND fallback | 60s |

**Key design decisions:**

1. All protocol connections are managed by a **Connection Manager** singleton that tracks connection state, health, and reconnection backoff. Each adapter reports its connection status through a shared observable.

2. The **Event Bus** is an in-memory RxJS/BroadcastChannel pattern. Every adapter emits normalized events into the bus. The local data layer subscribes to the bus. The UI subscribes to the data layer. This means the UI never directly touches a protocol adapter.

3. **Idle connection management**: On mobile, if the app is backgrounded, TCP connections should be suspended. Use platform push notifications (FCM/APNs) for protocols that support them (Matrix, XMPP via the server) and a background polling service for the rest. On desktop, maintain persistent connections.

### 1.3 Local-First Architecture

This is the most critical architectural decision. The app is useless without network -- every post, message, and profile must be available locally.

#### Why not CRDTs for social data?

CRDTs (Automerge, Yjs) are excellent for collaborative editing of a single document but are a poor fit for a social media aggregator:

- **Append-only feeds**: Timelines are append-only sequence data. A simple SQLite table with `(protocol, author_id, created_at, content)` works better than a CRDT and queries faster.
- **No conflict resolution needed**: Each protocol's server is the source of truth for its own data. You never need to merge concurrent edits to the same post across protocols -- a Mastodon post and a Nostr note are different objects even if the same human authored both.
- **Query complexity**: Full-text search across six protocols, user-specific timelines, conversations -- these are relational queries. CRDTs offer no query layer.

**Verdict**: Use CRDTs only for app-level state that might be synced between the user's own devices (read status, mute lists, draft composition). Use SQLite as the primary store for social content.

#### Storage architecture

```
┌────────────────────────────────────────────────────────────┐
│                      SQLite Database                        │
│                                                            │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐  │
│  │  posts    │ │ profiles │ │conversa- │ │  attachments │  │
│  │           │ │          │ │ tions    │ │              │  │
│  ├──────────┤ ├──────────┤ ├──────────┤ ├──────────────┤  │
│  │id (hash) │ │id(handle)│ │  id      │ │  id          │  │
│  │protocol  │ │protocol  │ │protocol  │ │  protocol    │  │
│  │author_id │ │name      │ │particip. │ │  local_path  │  │
│  │content   │ │avatar_url│ │last_msg  │ │  remote_url  │  │
│  │created_at│ │bio       │ │unread    │ │  mime_type   │  │
│  │protocol_ │ │metadata  │ │metadata  │ │  status      │  │
│  │  metadata│ │(JSON)    │ │(JSON)    │ │              │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────┘  │
│                                                            │
│  ┌──────────────────────┐  ┌────────────────────────┐     │
│  │  protocol_state      │  │  app_state             │     │
│  │  (per-protocol       │  │  (read markers,        │     │
│  │   cursor/pagination) │  │   mute lists, drafts)  │     │
│  └──────────────────────┘  └────────────────────────┘     │
│                                                            │
│  ┌────────────────────────────────────────────────┐       │
│  │  FTS5 virtual tables (full-text search index)  │       │
│  │  posts_fts ON (content, author_name)           │       │
│  └────────────────────────────────────────────────┘       │
└────────────────────────────────────────────────────────────┘
```

#### Sync strategies per protocol

| Protocol | Sync mechanism | Stores cursor in | Initial sync cost |
|---|---|---|---|
| ActivityPub | Mastano API pagination (`max_id`, `since_id`) | `protocol_state` table | High (fetch all pages) |
| AT Protocol | Firehose replay + XRPC cursor | Commit CID in sync state | Very high (repo scan) |
| Nostr | Subscription filters + `since` tag | Unix timestamp or event ID | Moderate (relay-dependent) |
| Matrix | `/sync` with `since` token | `next_batch` token | Very high (room history) |
| IRC | Bouncer replay (soju `znc.in/playback`) | Server-side timestamp | Low (bouncer stores) |
| XMPP | MAM (Message Archive Management) | `last_id` or timestamp | High (server archive) |

### 1.4 Identity Model

This is the hardest design problem. A user has a Mastodon handle, a Bluesky DID, a Nostr npub, a Matrix MXID, an IRC nick, and an XMPP JID -- and none of these are linked.

#### Option A: Unified identity (rejected)

Generate a master keypair and use it to sign cross-protocol identity claims. Publish a DID document on each protocol linking back to the master DID. This is theoretically clean but practically impossible -- Bluesky uses its own DID system with PLC or web-based DIDs, ActivityPub uses `acct:user@domain`, Nostr uses `nip-05` for DNS-based verification. Cross-signing requires write access on every protocol, which most users won't have or won't bother with.

#### Option B: Per-protocol profiles with manual linking (recommended)

Each protocol adapter manages its own identity independently. The app maintains a local mapping of "these profiles belong to the same human" that is:

1. **Manually created** by the user ("link @wagnermontes@mastodon.social to @wagnermontes.bsky.social")
2. **Discoverable** via heuristic matching (same display name + same avatar hash = likely same person)
3. **Shareable** via a profile card that lists all known protocol identities

```
app_identity_links:
  local_group_id: uuid
  protocol: string        # "activitypub", "atproto", "nostr", etc.
  protocol_user_id: string # the handle/DID/npub/MXID/nick/JID
  verified_at: timestamp
  verification_method: "manual" | "heuristic" | "cryptographic" | "shared_key"
```

This avoids the complexity of cross-protocol DID resolution while still allowing the app to show "Bob on all six protocols" as a unified contact.

#### Cryptographic identity (optional but recommended)

Each protocol adapter should generate a local keypair that is stored in the OS keychain (iOS Keychain, Android Keystore, Keychain on macOS/Linux). For Nostr and Matrix, these directly map to the protocol's native key material. For ActivityPub and AT Protocol, these are OAuth client credentials. For IRC and XMPP, these are SASL passwords (stored securely, never in plaintext).

---

## 2. Protocol Integration Strategy

### 2.1 ActivityPub (Mastodon API vs native C2S)

#### Recommendation: Mastodon API (REST + WebSocket streaming)

Building a full ActivityPub C2S (Client-to-Server) implementation is a significant undertaking -- you need to handle:
- HTTP Signatures (draft-cavage-http-signatures-12)
- ActivityStreams 2.0 JSON-LD parsing
- WebFinger protocol for actor discovery
- Inbox forwarding rules
- Object ID resolution with content negotiation
- SharedInbox optimization

For reference, the Mastodon API is already the de facto standard for Mastodon-compatible services (Pleroma, Akkoma, Firefish, GoToSocial, Pixelfed all support it). It's simpler, has better library support, and covers 95% of what a client needs.

**Integration approach:**

```typescript
class MastodonAdapter implements ProtocolAdapter {
  private client: MastodonRestClient; // via @mastodon/megalodon or custom
  private stream: EventSource | WebSocket;
  
  async connect(credentials: CredentialMap) {
    // OAuth2 app registration + user token
    // Store token in keychain, never in SQLite
    this.client = new MastodonRestClient({
      instance: credentials.instanceUrl,
      accessToken: await keychain.get('mastodon_token')
    });
    
    // Start streaming
    this.stream = this.client.stream('/api/v1/streaming');
    this.stream.on('update', (post) => this.bus.emit('post', normalize(post)));
    this.stream.on('notification', (n) => this.bus.emit('notification', n));
  }
  
  // normalize() maps Mastodon's verbose API response to the unified Post model
  normalize(raw: MastodonStatus): Post {
    return {
      id: `apub:${raw.id}`,
      protocol: 'activitypub',
      author: this.normalizeAuthor(raw.account),
      content: raw.content, // HTML -- strip or render as-is
      createdAt: new Date(raw.created_at),
      attachments: raw.media_attachments.map(...),
      metrics: { likes: raw.favourites_count, boosts: raw.reblogs_count, replies: raw.replies_count },
      metadata: { visibility: raw.visibility, spoilerText: raw.spoiler_text, language: raw.language }
    };
  }
}
```

**Key libraries:**

- **TypeScript**: `megalodon` (Mastodon API client with streaming, works for Pleroma/Akkoma too) or `@acks/mastodon-api`
- **Dart/Flutter**: `mastodon_api` package
- **Rust**: `megalodon-rs` or `elefren`
- **Python**: `Mastodon.py`

**Gotchas:**
- Instance API version differences (v1 vs v2 API endpoints, especially for media uploads and filters)
- Character limits vary by instance (500 default, but Pleroma allows configurable limits up to ~infinity)
- `since_id` pagination has edge cases when posts arrive during pagination (use `offset` param as fallback)
- Mastodon API is Mastodon-centric, not ActivityPub-generic. You lose access to non-Mastodon ActivityPub servers' features (e.g., Lemmy's different post model, PeerTube's video-specific metadata)

#### Native C2S consideration (when to bother)

If cross-instance federation features are needed (send Follow to any AP actor, even those not on Mastodon-compatible servers), a minimal C2S implementation focused on the `outbox` and `inbox` endpoints is warranted. But keep it as a separate adapter module -- don't merge it with the Mastodon API adapter.

### 2.2 AT Protocol (Bluesky)

#### Integration approach: XRPC client + Firehose subscription

The AT Protocol has the most modern API design of the six protocols. It's built around:

1. **XRPC** -- HTTP-based Lexicon-driven RPC (similar to gRPC but over HTTP with Lexicon schemas)
2. **Firehose** -- A WebSocket-based relay subscription for real-time events
3. **PDS (Personal Data Server)** -- The user's data store; can be self-hosted or use Bluesky's PDS

**Architecture:**

```typescript
class AtProtocolAdapter implements ProtocolAdapter {
  private agent: AtpAgent; // @atproto/api
  private firehose: FirehoseSubscription;
  private syncCursor: string | null; // commit CID for incremental sync
  
  async connect(credentials: CredentialMap) {
    // Bluesky uses OAuth with DPoP or simple app passwords for now
    this.agent = new AtpAgent({ service: credentials.pdsUrl || 'https://bsky.social' });
    await this.agent.login({
      identifier: credentials.handle,
      password: credentials.appPassword // always use app passwords, never user password
    });
    
    // Subscribe to the relay firehose for real-time updates
    this.firehose = new FirehoseSubscription({
      service: 'wss://bsky.network/xrpc/com.atproto.sync.subscribeRepos',
      // Use a cursor to avoid re-processing old events
      cursor: this.syncCursor
    });
    
    this.firehose.on('commit', this.handleRepoCommit.bind(this));
    this.firehose.on('identity', this.handleIdentityEvent.bind(this));
    this.firehose.on('account', this.handleAccountEvent.bind(this));
  }
  
  handleRepoCommit(commit: RepoCommit) {
    // Parse the repository commit to extract created posts, likes, follows, etc.
    const operations = this.parseCommitOperations(commit);
    for (const op of operations) {
      if (op.action === 'create' && op.collection === 'app.bsky.feed.post') {
        const post = this.resolvePost(op.uri);
        this.bus.emit('post', this.normalizePost(post, commit.author));
      }
      // Handle likes, reposts, follows similarly
    }
  }
  
  // Firehose gives you raw CBOR repository commits.
  // You need to parse the MST (Merkle Search Tree) to find created records.
  parseCommitOperations(commit: RepoCommit): Operation[] {
    // Use @atproto/sync's parseRepo or parseCommit for MST traversal
    return commit.ops.map(op => ({
      action: op.action, // "create" | "update" | "delete"
      collection: op.path.split('/')[0],
      rkey: op.path.split('/')[1],
      cid: op.cid,
      // op.record is CBOR-decoded record data
      record: op.record ? cbor.decode(op.record) : null
    }));
  }
}
```

**Key libraries:**

- **`@atproto/api`** -- The official Bluesky client SDK, handles XRPC, lexicon schemas, auth, everything
- **`@atproto/sync`** -- Firehose subscription and repository sync utilities
- **`@atproto/ozone`** -- (optional) Moderation tooling
- **`@atproto/xrpc`** -- Low-level XRPC client if you want to roll your own

**Sync strategy:**

Initial sync: Use `com.atproto.sync.getRepo` to download the user's entire repository (all posts, likes, follows as a CAR file -- Content Addressable aRchive). This is the most expensive part: a user with 10,000 posts might have a repo of several megabytes.

Incremental: Store the last commit CID and use `com.atproto.sync.subscribeRepos` with a cursor. Process commits in order.

**Firehose is not optional** -- it's the primary notification mechanism. Without it, you'd need to poll the user's PDS every few seconds, which defeats the purpose.

**Gotchas:**
- The firehose emits every public event on the network. Filter aggressively by author DID + the user's follows list, or you'll drown in irrelevant data.
- Repository commits are CBOR-encoded (not JSON). You need a CBOR decoder (JavaScript has `@atproto/sync` which handles this; on other platforms, `cbor` or `cbor-x`).
- BlueSky's PDS can be rate-limited aggressively. Cache XRPC responses with appropriate TTLs.
- App passwords are the standard auth mechanism but Bluesky is migrating to full OAuth with DPoP. Plan for this migration.
- The Lexicon schema evolves. `@atproto/api` bundles the latest Lexicons, but new record types (starter packs, feeds, lists) require SDK updates.

### 2.3 Nostr

#### Integration approach: Relay management + NIP compliance

Nostr is the simplest protocol to integrate at a basic level and the most complex to integrate at a production level. The protocol is just signed JSON events over WebSocket, but the NIP (Nostr Implementation Possibilities) ecosystem is sprawling.

**Core architecture:**

```typescript
class NostrAdapter implements ProtocolAdapter {
  private relays: RelayPool;    // manages multiple relay connections
  private signer: NostrSigner;  // wraps the private key
  private pendingEvents: Map<string, Set<string>>; // event-id -> relay set
  
  async connect(credentials: CredentialMap) {
    // Nostr uses a hex private key (nsec) or a mnemonic
    this.signer = new NostrSigner(
      await keychain.get('nostr_privkey') || NostrSigner.generate()
    );
    
    // Connect to configured relays
    this.relays = new RelayPool(credentials.relayUrls || DEFAULT_RELAYS);
    
    // Subscribe to relevant events
    this.relays.subscribe([
      { kinds: [1], authors: this.followedPubkeys, limit: 100 }, // text notes
      { kinds: [7], '#e': this.myPostIds }, // reactions to my posts
      { kinds: [5], '#e': this.myPostIds }, // deletions of my posts
      { kinds: [3], authors: [this.pubkey] }, // contact list (follows)
      { kinds: [0], authors: this.followedPubkeys }, // metadata updates
    ]);
    
    this.relays.on('event', (event, relayUrl) => {
      this.bus.emit('post', this.normalizeEvent(event, relayUrl));
    });
  }
  
  normalizeEvent(event: NostrEvent, sourceRelay: string): Post | null {
    switch (event.kind) {
      case 1: // Short Text Note
        return {
          id: `nostr:${event.id}`,
          protocol: 'nostr',
          author: this.resolveProfile(event.pubkey),
          content: event.content, // plain text or JSON for NIP-30
          createdAt: new Date(event.created_at * 1000),
          tags: event.tags, // preserve protocol-specific metadata
          metadata: {
            kind: 1,
            replyTo: event.tags.find(t => t[0] === 'e')?.[1],
            rootEvent: event.tags.filter(t => t[0] === 'e').slice(-1)[0]?.[1],
            relays: event.tags.filter(t => t[0] === 'relay').map(t => t[1]),
            raw: event // store raw for NIP compliance
          }
        };
      case 7: // Reaction
        return null; // handled separately via reactions bus
      // Handle other kinds...
    }
  }
  
  async publish(content: PostContent): Promise<PostReceipt> {
    const event = this.createSignedEvent({
      kind: 1,
      content: content.text,
      tags: content.replyTo ? [['e', content.replyTo.id, content.replyTo.relay]] : [],
      created_at: Math.floor(Date.now() / 1000)
    });
    
    // Sign with the private key
    await this.signer.sign(event);
    
    // Publish to all connected relays
    const results = await this.relays.publish(event);
    // Track which relays accepted/rejected the event
    this.pendingEvents.set(event.id, new Set(results.map(r => r.url)));
    
    return { id: event.id, publishedTo: results.filter(r => r.success).map(r => r.url) };
  }
}
```

**NIP compliance strategy:**

Not all NIPs are worth implementing. Prioritize by usage and impact:

| NIP | Purpose | Priority | Notes |
|---|---|---|---|
| NIP-01 | Basic protocol (events, relay comms) | **Critical** | Foundation of everything |
| NIP-02 | Contact List (kind 3) | **Critical** | Required for following |
| NIP-04 | Encrypted DMs | **Important** | Being superseded by NIP-17 |
| NIP-05 | DNS-based identifiers | **Important** | UX improvement |
| NIP-07 | Extension signing | Medium | Desktop only |
| NIP-09 | Event deletion | **Important** | User control |
| NIP-10 | Reply conventions (e-tags) | **Important** | Thread rendering |
| NIP-11 | Relay info metadata | Medium | Relay discovery |
| NIP-13 | Proof-of-work | Low | Niche use case |
| NIP-14 | Subject tag in text events | Low | |
| NIP-15 | End-of-stored-events notice | Medium | Sync correctness |
| NIP-17 | Private DMs (kind 14) | **Important** | Replaces NIP-04 |
| NIP-18 | Reposts (kind 6) | **Important** | |
| NIP-19 | Bech32-encoded identifiers | **Important** | UX for npub/nsec |
| NIP-20 | Command results | Low | |
| NIP-21 | nostr: URI scheme | Medium | Link handling |
| NIP-22 | Event created_at limits | Low | |
| NIP-23 | Long-form content (kind 30023) | Medium | Blog-like posts |
| NIP-25 | Reactions (kind 7) | **Important** | |
| NIP-26 | Delegated event signing | Low | Niche |
| NIP-28 | Public chat (kind 40-44) | Medium | Group chats |
| NIP-30 | Custom emoji | Low | |
| NIP-31 | alt tag for accessibility | Medium | |
| NIP-32 | Labels (kind 1985) | Low | |
| NIP-33 | Parameterized replaceable events | **Important** | Profiles, lists |
| NIP-36 | Sensitive content (kind 1986) | Medium | Content warnings |
| NIP-40 | Expiration timestamps | Low | |
| NIP-42 | AUTH for relays | **Important** | Paid relays |
| NIP-44 | Versioned encryption | **Important** | Used by NIP-17 |
| NIP-46 | Nostr Connect (remote signer) | Medium | |
| NIP-47 | Wallet info (NWC) | Low | Niche |
| NIP-50 | Search (kind: 1005 filter) | **Important** | Cross-relay search |
| NIP-51 | Lists (mute, bookmark, etc.) | Medium | |
| NIP-56 | Reporting | Low | |
| NIP-57 | Lightning Zaps | Medium | Tipping |
| NIP-58 | Badges | Low | |
| NIP-65 | Relay list metadata | **Important** | Relay discovery |
| NIP-89 | Recommended app handlers | Low | |
| NIP-94 | File metadata | Medium | |
| NIP-95/96 | HTTP file storage | Medium | |
| NIP-98 | HTTP auth | Low | |

**Relay management strategy:**

```typescript
class RelayPool {
  private connections: Map<string, RelayConnection>;
  private subscriptionIdCounter = 0;
  
  // Connection quality tracking
  private reliability: Map<string, {
    latencyMs: number[],
    uptime: number,          // fraction of successful connections
    lastDisconnect: Date | null,
    acceptedEvents: number,
    rejectedEvents: number
  }>;
  
  // Read relays vs Write relays
  private readRelays: string[];
  private writeRelays: string[];
  
  subscribe(filters: NostrFilter[], options?: { relays?: string[] }) {
    const subId = `sub_${++this.subscriptionIdCounter}`;
    const targetRelays = options?.relays || this.readRelays;
    
    for (const relayUrl of targetRelays) {
      this.getOrConnect(relayUrl).send(['REQ', subId, ...filters]);
    }
    
    return () => this.closeSubscription(subId); // unsubscribe function
  }
  
  // Adaptive relay selection: prefer relays with low latency and high uptime
  private selectRelay(capability: 'read' | 'write'): string {
    const candidates = capability === 'read' ? this.readRelays : this.writeRelays;
    return candidates.sort((a, b) => {
      const ra = this.reliability.get(a);
      const rb = this.reliability.get(b);
      if (!ra) return -1;
      if (!rb) return 1;
      // Weighted score: lower latency + higher uptime
      const scoreA = (ra.uptime || 0) - (avg(ra.latencyMs) / 1000);
      const scoreB = (rb.uptime || 0) - (avg(rb.latencyMs) / 1000);
      return scoreB - scoreA;
    })[0];
  }
}
```

**Nostr key libraries:**

- **TypeScript**: `nostr-tools` (canonical, maintained, good NIP coverage), `@nostr-dev-kit/ndk` (higher-level, relay management included)
- **Rust**: `nostr-sdk` (best-in-class, async, supports most NIPs)
- **Kotlin**: `nostr-postr` (Java/Kotlin Nostr library)
- **Python**: `nostr` or `nostr-protocol`

**Gotchas:**
- Relay centralization risk: most Nostr clients use a handful of relays (nos.lol, relay.damus.io). Implement relay discovery via NIP-11 and NIP-65 to spread load.
- Event deduplication is essential. The same event may arrive from multiple relays. Use a set of `(event_id, relay_url)` to track which relays have confirmed delivery.
- Relay data is lost if the relay goes down. Important events should be persisted to multiple relays. This is a user-responsibility problem -- the client can help but can't fix it.
- `created_at` timestamps are user-controlled and frequently wrong (clock drift, intentional misrepresentation). Use the event ID's timestamp as the canonical ordering field, not user-reported `created_at`.

### 2.4 Matrix

#### Integration approach: matrix-rust-sdk via FFI

Matrix is unique among these protocols in having a single dominant, production-quality SDK: the **matrix-rust-sdk**. It handles:

- Room state resolution
- End-to-end encryption (Olm/Megolm via vodozemac)
- Sync API v2
- Sliding Sync (MSC3575, the modern Matrix sync mechanism)
- Room key management and backup

**Recommendation**: Use matrix-rust-sdk through its UniFFI bindings, which generate bindings for Kotlin, Swift, Python, and other languages. This avoids reimplementing the complex state resolution and E2EE logic.

```typescript
// If using the Rust SDK through FFI:
// In practice this is bound via Swift/Kotlin packages, but the logical API is:
class MatrixAdapter implements ProtocolAdapter {
  private client: MatrixClient;  // from matrix-rust-sdk FFI
  private syncStream: Stream<SyncResponse>;
  
  async connect(credentials: CredentialMap) {
    // Initialize the Rust SDK
    this.client = await MatrixClientBuilder::new()
      .homeserverUrl(credentials.homeserverUrl)  // e.g., https://matrix.org
      .passphrase(credentials.password)
      .sqliteStore(DB_PATH + '/matrix')
      .slidingSyncProxy(credentials.slidingSyncProxy) // MSC3575 proxy
      .build();
    
    // Start sync loop
    this.syncStream = this.client.startSync();
    
    // Process sync responses in a stream
    for await (const sync of this.syncStream) {
      for (const room of sync.rooms.join) {
        for (const event of room.timeline.events) {
          this.handleMatrixEvent(event, room);
        }
      }
    }
  }
  
  handleMatrixEvent(event: MatrixEvent, room: RoomInfo) {
    if (room.isDirect) {
      // DM conversation
      this.bus.emit('message', this.normalizeDM(event, room));
    } else if (room.isSpace || room.isPublic) {
      // Channel/Room posts
      this.bus.emit('post', this.normalizeRoomEvent(event, room));
    }
  }
}
```

#### On the lighter alternative: matrix-js-sdk

The JavaScript SDK (`matrix-js-sdk`) is a full reimplementation and works for simple use cases, but has significant drawbacks:

- E2EE support requires `@matrix-org/olm` (WASM build of Olm), which is large (~2MB WASM blob)
- Sync performance is substantially worse than the Rust SDK
- State resolution is reimplemented in JS, which is slower and diverges occasionally

Use the JS SDK only if you're constrained to a pure JS runtime (e.g., a browser extension). For a native app with the freedom to bundle native code, matrix-rust-sdk is the clear choice.

#### E2EE handling

Matrix E2EE is the most complex encryption scheme among these protocols. Key points:

1. **Keys are stored locally** in the SQLite store managed by matrix-rust-sdk. They must be backed up.
2. **Cross-signing** allows the user to verify other devices. This requires the user's signing key to be accessible.
3. **Secure backup** (key backup) stores room keys on the server using a user-chosen passphrase. The app should prompt for this during onboarding.
4. **Device verification** is manual. The app should surface verification requests in the notification feed.

**Critical**: If the local store with room keys is deleted, all past E2EE messages become unreadable. Implement key export and import.

**Key libraries:**

- **Rust**: `matrix-rust-sdk` (UniFFI, supports E2EE, Sliding Sync, all core features)
- **JS/TS**: `matrix-js-sdk` + `@matrix-org/olm`
- **Dart**: `matrix_sdk` package (a Dart port, lags behind in E2EE support)

**Gotchas:**
- Matrix homeservers vary in their Sliding Sync support. Without Sliding Sync (MSC3575), the `/sync` endpoint returns full room state on every connection, which is extremely expensive for users in many rooms. Implement both sync paths.
- Room state resolution is computationally expensive. The Rust SDK handles this, but if implementing from scratch, it's one of the hardest parts of the protocol.
- Matrix's push notification system (via pushers) requires a push gateway. For a full local-first app, consider implementing local notifications directly rather than relying on matrix.org's push service.
- E2EE verification UI/UX is notoriously user-unfriendly. Emoji comparison (MSC3907) or QR code scanning are the least bad options.

### 2.5 IRC

#### Integration approach: Bouncer model (soju) + persistent connection

IRC is ancient, simple on the surface, and surprisingly complex to get right in a modern app.

**The bouncer is not optional.** Without a bouncer:
1. You miss all messages while offline
2. You must rejoin all channels and replay history on each connection
3. SASL authentication must be re-done each time

**Recommended setup:** soju (modern Go bouncer) or znc (battle-tested C++ bouncer). soju is preferred for:
- Built-in IRCv3 support (including `soju.im/bouncer-networks`, `soju.im/read`)
- Single-binary deployment, simple configuration
- Can be self-hosted or run as a service
- Implements `draft/chathistory` for history retrieval

```typescript
class IrcAdapter implements ProtocolAdapter {
  private connection: IrcClient;
  private bouncerUrl: string;
  private channels: Map<string, ChannelState>;
  private messageQueue: Message[];
  
  async connect(credentials: CredentialMap) {
    // Connect through the bouncer, not directly to IRC networks
    // soju exposes a single TCP connection for all networks
    this.bouncerUrl = credentials.bouncerUrl || 'ircs://soju.example.com:6697';
    
    this.connection = new IrcClient({
      url: this.bouncerUrl,
      nick: credentials.nick,
      sasl: { user: credentials.nick, pass: credentials.bouncerPassword },
      // Request IRCv3 capabilities
      capabilities: [
        'sasl',
        'message-tags',
        'echo-message',
        'znc.in/server-time',
        'soju.im/bouncer-networks',
        'soju.im/read',
        'draft/chathistory'
      ]
    });
    
    this.connection.on('message', (msg) => {
      this.bus.emit('message', this.normalizeMessage(msg));
    });
    
    this.connection.on('join', (nick, channel) => {
      if (nick === this.connection.nick) {
        this.requestHistory(channel, 100); // replay last 100 messages
      }
    });
  }
  
  // IRC has no built-in reaction system, no rich text, no threads.
  // These must be emulated:
  // - Reactions: Map to a protocol extension or ignore
  // - Threads: Use `znc.in/server-time` for sorting
  // - Rich text: IRC is plain text. Apply markdown-like client-side formatting
  
  async getTimeline(opts: TimelineQuery): AsyncIterable<Post> {
    // For IRC, "timeline" = messages in joined channels
    // Filter by channel and time range
    if (opts.channel) {
      const history = await this.fetchHistory(opts.channel, opts.limit);
      return history.map(m => this.toPost(m));
    }
    // Aggregate across all joined channels
    const allMessages = [];
    for (const channel of this.channels.keys()) {
      allMessages.push(...await this.fetchHistory(channel, Math.ceil(opts.limit / this.channels.size)));
    }
    return allMessages.sort((a, b) => b.timestamp - a.timestamp).slice(0, opts.limit);
  }
  
  private async fetchHistory(channel: string, limit: number): Promise<IrcMessage[]> {
    // Use draft/chathistory to get history from the bouncer
    // soju implements IRCv3 chathistory
    const batch = await this.connection.send('CHATHISTORY', 'LATEST', channel, '*', limit.toString());
    return batch.messages;
  }
}
```

**IRCv3 capabilities to request:**

| Capability | Purpose | Required? |
|---|---|---|
| `sasl` | Authenticated login | **Yes** |
| `message-tags` | Client-only tags (reactions, labels) | **Yes** |
| `echo-message` | See your own messages in the stream | **Yes** |
| `znc.in/server-time` | Timestamps per message | **Yes** |
| `znc.in/playback` | ZNC history replay | Recommended |
| `soju.im/bouncer-networks` | Multi-network bouncer support | If using soju |
| `soju.im/read` | Read markers | If using soju |
| `draft/chathistory` | History retrieval | **Yes** |
| `draft/replies` | Message threading | Recommended |

**Key libraries:**

- **TypeScript**: `irc-framework` (best IRC client library for Node/Deno, full IRCv3 support)
- **Dart**: `irc` package (basic, limited IRCv3 support)
- **Rust**: `tokio-irc` (async, IRCv3 support)
- **Go**: `girc` or `irc` (the standard Go IRC libs)

**Gotchas:**
- IRC has no reliable delivery guarantees. Messages can be lost if the connection drops between the bouncer and the server. The client should not assume delivery.
- Nickname collision: If someone takes your nick while you're offline, you might regain connection with a different nick. Monitor nick changes and alert the user.
- Channel mode +m (moderated) prevents non-voice users from speaking. The client should detect this and disable the compose input.
- CTCP messages: handle PING, VERSION, TIME, ACTION (/me) correctly. This is more protocol complexity than it looks.
- IRC connection identifiers are case-insensitive. `#Channel` and `#channel` are the same. Normalize to lowercase for storage keys.
- DCC (Direct Client-to-Client) for file transfers is non-trivial. Consider whether to support it at all, or treat IRC as text-only.

### 2.6 XMPP

#### Integration approach: XMPP client library with OMEMO

XMPP is the second most complex protocol (after Matrix) due to its extensibility model (XEPs) and encryption requirements.

```typescript
class XmppAdapter implements ProtocolAdapter {
  private client: XmppClient;
  private messageArchive: Xep0313; // MAM (Message Archive Management)
  private omemo: Xep0384;
  private avatarManager: Xep0084;
  private bookmarkManager: Xep0048; // bookmarks for MUC rooms
  
  async connect(credentials: CredentialMap) {
    this.client = new XmppClient({
      jid: credentials.jid,      // user@domain
      password: credentials.password,
      // Use SCRAM-SHA-256 for SASL
      sasl: ['SCRAM-SHA-256', 'SCRAM-SHA-1', 'PLAIN'],
      // Use WebSocket if available, fallback to TCP (BOSH)
      transport: 'websocket',
      url: credentials.websocketUrl || `wss://${credentials.domain}/xmpp-websocket`,
      // or use BOSH: credentials.boshUrl
    });
    
    // Enable required XEPs
    await this.client.use(Xep0030); // Service Discovery
    await this.client.use(Xep0115); // Entity Capabilities
    await this.client.use(Xep0199); // XMPP Ping
    await this.client.use(Xep0313); // Message Archive Management (MAM)
    await this.client.use(Xep0384); // OMEMO Encryption
    await this.client.use(Xep0084); // User Avatar
    await this.client.use(Xep0048); // Bookmarks
    await this.client.use(Xep0060); // Publish-Subscribe
    await this.client.use(Xep0363); // HTTP File Upload
    await this.client.use(Xep0184); // Message Delivery Receipts
    await this.client.use(Xep0333); // Chat Markers
    await this.client.use(Xep0085); // Chat State Notifications
    
    await this.client.connect();
    
    // MAM: catch up on history since last sync
    const lastSync = this.getLastSyncTimestamp();
    const mamResult = await this.messageArchive.query({
      start: lastSync,
      limit: 1000
    });
    
    for (const message of mamResult.messages) {
      this.handleMessage(message);
    }
    
    // Handle incoming messages
    this.client.on('message', (msg) => {
      if (msg.body && !this.isDuplicate(msg)) {
        this.handleMessage(msg);
      }
    });
  }
  
  handleMessage(msg: XmppMessage) {
    const from = msg.from; // JID
    const body = msg.body;
    const isGroupChat = msg.type === 'groupchat';
    
    if (isGroupChat) {
      // MUC (Multi-User Chat) message -> treat like a channel post
      this.bus.emit('post', {
        id: `xmpp:${msg.id}`,
        protocol: 'xmpp',
        author: this.normalizeMucAuthor(from, msg.resource),
        content: body,
        conversation: from.bare, // room@conference.domain
        timestamp: msg.delay?.stamp || new Date(),
        encrypted: msg.encrypted ?? false
      });
    } else {
      // Direct message
      this.bus.emit('message', {
        id: `xmpp:${msg.id}`,
        conversation: this.getOrCreateConversation(from.bare, msg.to.bare),
        sender: from.bare,
        content: msg.body,
        timestamp: msg.delay?.stamp || new Date(),
        encrypted: msg.encrypted ?? false
      });
    }
  }
}
```

**OMEMO handling (critical):**

OMEMO (XEP-0384) is XMPP's E2EE protocol, based on the Signal Protocol (Double Ratchet + X3DH). Implementation requirements:

1. **Bundles**: The client must publish its OMEMO device bundles via PEP (PubSub). This includes the device's identity key, signed pre-key, and a set of one-time pre-keys.
2. **Session management**: Each conversation partner gets a Double Ratchet session. Sessions are established lazily (on first message).
3. **Key storage**: The identity key and all session state must be persisted. Loss of session state means you can't decrypt old messages and must re-establish sessions.
4. **Fingerprint verification**: Users can verify each other's identity key fingerprints (out-of-band or via QR code scanning).

**OMEMO library support:**

| Language | Library | OMEMO Support | Notes |
|---|---|---|---|
| Python | `slixmpp` + `python-omemo` | Good but maintenance concerns | |
| C | `libomemo` | Good (C library) | Used by Profanity |
| Rust | `omemo-rs` | In development | Promising but immature |
| JS/TS | `@otr/libsignal` + custom OMEMO | Partial | Signal protocol implementation |
| Java | `smack` + `omemo` module | **Best** | Full OMEMO support |

**Recommendation**: For a cross-platform app, use **Smack** (Java/Kotlin) for Android/desktop or **slixmpp** (Python) for server-side components. For full-stack synchronization, **Tigase's XMPP SDK for Kotlin** is another option worth evaluating. If targeting a Rust-based architecture (via Tauri or similar), implement a thin FFI layer over `libomemo` or `xmpp-rs`.

**Key XMPP XEPs (prioritized):**

| XEP | Purpose | Priority |
|---|---|---|
| XEP-0030 | Service Discovery | **Critical** |
| XEP-0115 | Entity Capabilities | **Critical** |
| XEP-0313 | MAM (history) | **Critical** |
| XEP-0384 | OMEMO | **Critical** |
| XEP-0363 | HTTP File Upload | **Important** |
| XEP-0084 | Avatars | **Important** |
| XEP-0045 | MUC (Multi-User Chat) | **Important** |
| XEP-0060 | Publish-Subscribe | **Important** |
| XEP-0163 | PEP (Personal Eventing) | **Important** |
| XEP-0198 | Stream Management | **Important** |
| XEP-0184 | Delivery Receipts | Medium |
| XEP-0333 | Chat Markers | Medium |
| XEP-0085 | Chat State Notification | Medium |
| XEP-0048 | Bookmarks | Medium |
| XEP-0352 | Client State Indication | Medium |
| XEP-0368 | SRV records for WebSocket | Low |

**Gotchas:**
- OMEMO Key management is fragile. If the user loses their local store, they lose the ability to decrypt past messages. Cloud backup of OMEMO keys is technically possible but breaks the E2EE model.
- MAM pagination: servers have different max page sizes. Some cap at 50 messages, some at 1000. Implement adaptive chunking.
- Not all servers support WebSocket connections. Have BOSH as a fallback.
- XMPP servers have latency -- a message can take seconds to be delivered even with `stream:management`. This is normal, don't show it as an error.
- MUC (group chat) discovery is terrible. Most clients hard-code room JIDs. The app should treat MUC rooms similarly to IRC channels (user joins by address).
- File upload (XEP-0363) requires the server to support HTTP upload slots. If not supported, fall back to a warning and disable attachments.

---

## 3. Local-First Data Layer

### 3.1 Storage Engine

**Primary: SQLite via a robust client**

SQLite is the obvious choice -- it's available on every platform, supports full-text search (FTS5), has excellent concurrency semantics, and runs at embedded-database speed.

| Platform | SQLite library | Notes |
|---|---|---|
| iOS/macOS | GRDB (Swift) | Best-in-class, type-safe, observation (ValueObservation, DatabaseRegionObservation), migrations |
| Android | Room (Kotlin) + SQLCipher for encryption | Room is the canonical Android DB, SQLCipher adds encryption |
| Desktop (Tauri) | `rusqlite` (Rust) + `tauri-plugin-sql` | Direct Rust-to-SQLite, no JS bridge overhead |
| Flutter | `drift` (formerly moor) | Type-safe, reactive queries, migrations |
| React Native | `react-native-quick-sqlite` or `expo-sqlite` | JSI-based, bypasses the React Native bridge for performance |
| Cross-platform | `sql.js` (WASM SQLite) | Works everywhere but slower, no concurrent access |

**Secondary stores (for specific needs):**

- **Keychain / Keystore** -- Protocol authentication tokens, private keys, OMEMO identity keys, Nostr nsec. Use platform-native secure storage (iOS Keychain via `KeychainSwift`, Android EncryptedSharedPreferences, `secret-service` on Linux).
- **File system** -- Media attachments (images, videos, audio). Store as files with the database tracking file paths. Use a content-addressable store (hash-based file naming) for deduplication.
- **Memory cache** -- Active timelines, notification counts, connection state. Use an in-memory LRU cache (e.g., `lru-cache` in JS, `NSCache` on iOS).

### 3.2 Schema Design (Core Tables)

```sql
-- Posts / Statuses / Notes (unified across all protocols)
CREATE TABLE posts (
    id TEXT PRIMARY KEY,                          -- "protocol:local_id"
    protocol TEXT NOT NULL,                       -- "activitypub" | "atproto" | "nostr" | "matrix" | "irc" | "xmpp"
    author_id TEXT NOT NULL,                      -- FK to profiles
    conversation_id TEXT,                         -- FK to conversations (for DMs, group chats)
    thread_root_id TEXT,                          -- For threaded replies
    thread_parent_id TEXT,                        -- Immediate parent in thread
    content TEXT NOT NULL,                        -- Rendered text (HTML stripped, or rich text)
    content_raw TEXT,                             -- Original protocol-specific content
    content_type TEXT DEFAULT 'text/plain',       -- "text/html" | "text/markdown" | "text/x.matrix-html"
    created_at INTEGER NOT NULL,                  -- Unix timestamp (milliseconds)
    edited_at INTEGER,                            -- Last edit timestamp
    visibility TEXT DEFAULT 'public',             -- "public" | "unlisted" | "followers_only" | "direct" | "private"
    language TEXT,                                -- Language code (BCP-47)
    is_deleted INTEGER DEFAULT 0,
    metadata TEXT DEFAULT '{}',                   -- JSON blob for protocol-specific data
    synced_at INTEGER NOT NULL DEFAULT (unixepoch() * 1000),
    local_created_at INTEGER NOT NULL DEFAULT (unixepoch() * 1000)
);

-- Full-text search index
CREATE VIRTUAL TABLE posts_fts USING fts5(
    content, author_display_name,
    content='posts',
    content_rowid='rowid',
    tokenize='unicode61 remove_diacritics 2'
);

-- Triggers to keep FTS in sync
CREATE TRIGGER posts_ai AFTER INSERT ON posts BEGIN
    INSERT INTO posts_fts(rowid, content, author_display_name)
    VALUES (new.rowid, new.content, (SELECT display_name FROM profiles WHERE id = new.author_id));
END;

-- Reactions / Likes / Boosts
CREATE TABLE reactions (
    id TEXT PRIMARY KEY,
    protocol TEXT NOT NULL,
    post_id TEXT NOT NULL REFERENCES posts(id),
    author_id TEXT NOT NULL REFERENCES profiles(id),
    type TEXT NOT NULL,                           -- "like" | "boost" | "repost" | "reaction" | "emoji"
    emoji TEXT,                                   -- Custom emoji value (for emoji reactions)
    created_at INTEGER NOT NULL,
    metadata TEXT DEFAULT '{}'
);

-- Profiles / Users
CREATE TABLE profiles (
    id TEXT PRIMARY KEY,                          -- "protocol:user_id"
    protocol TEXT NOT NULL,
    username TEXT,                                -- Handle / nick / MXID localpart
    display_name TEXT,
    bio TEXT,
    avatar_url TEXT,
    avatar_path TEXT,                             -- Local file path (cached)
    banner_url TEXT,
    banner_path TEXT,
    follower_count INTEGER DEFAULT 0,
    following_count INTEGER DEFAULT 0,
    post_count INTEGER DEFAULT 0,
    metadata TEXT DEFAULT '{}',                   -- JSON blob: NIP-05 domains, DID docs, etc.
    last_fetched INTEGER,
    is_bot INTEGER DEFAULT 0
);

-- Conversations (DMs, group chats, IRC channels)
CREATE TABLE conversations (
    id TEXT PRIMARY KEY,
    protocol TEXT NOT NULL,
    type TEXT NOT NULL,                           -- "direct" | "group" | "channel" | "room"
    name TEXT,                                    -- Human-readable name
    topic TEXT,                                   -- Channel topic / room description
    avatar_url TEXT,
    avatar_path TEXT,
    last_message_id TEXT,                         -- FK to posts
    last_message_at INTEGER,                      -- Timestamp of last activity
    unread_count INTEGER DEFAULT 0,
    membership_state TEXT DEFAULT 'joined',       -- "joined" | "invited" | "left" | "knocking"
    metadata TEXT DEFAULT '{}'
);

-- Conversation participants (many-to-many)
CREATE TABLE conversation_participants (
    conversation_id TEXT NOT NULL REFERENCES conversations(id),
    profile_id TEXT NOT NULL REFERENCES profiles(id),
    join_time INTEGER,
    role TEXT DEFAULT 'member',                   -- "admin" | "moderator" | "member" | "voice"
    PRIMARY KEY (conversation_id, profile_id)
);

-- Protocol sync state
CREATE TABLE protocol_sync_state (
    protocol TEXT PRIMARY KEY,
    last_sync_at INTEGER,
    cursor TEXT,                                  -- Protocol-specific cursor (token, commit CID, timestamp)
    state TEXT DEFAULT 'active',                  -- "active" | "error" | "paused"
    error_message TEXT,
    metadata TEXT DEFAULT '{}'
);

-- Cross-protocol identity linking
CREATE TABLE identity_links (
    local_group_id TEXT NOT NULL,                 -- UUID shared across rows for one person
    protocol TEXT NOT NULL,
    protocol_user_id TEXT NOT NULL,
    protocol_instance TEXT,                       -- Mastodon instance, Matrix HS, IRC network
    verified INTEGER DEFAULT 0,
    verification_method TEXT,                     -- "manual" | "heuristic" | "cryptographic"
    verified_at INTEGER,
    PRIMARY KEY (protocol, protocol_user_id, protocol_instance)
);

-- Drafts (unpublished posts)
CREATE TABLE drafts (
    id TEXT PRIMARY KEY,
    protocol TEXT NOT NULL,
    content TEXT NOT NULL,
    created_at INTEGER NOT NULL,
    updated_at INTEGER NOT NULL,
    reply_to_id TEXT,
    visibility TEXT DEFAULT 'public',
    scheduled_for INTEGER,                        -- NULL = send immediately
    attachments TEXT DEFAULT '[]',                -- JSON array of attachment references
    metadata TEXT DEFAULT '{}'
);

-- Muted/blocked items
CREATE TABLE muted_items (
    protocol TEXT NOT NULL,
    target_type TEXT NOT NULL,                    -- "user" | "thread" | "keyword" | "channel"
    target_id TEXT NOT NULL,
    muted_at INTEGER NOT NULL,
    expires_at INTEGER,                           -- NULL = permanent
    PRIMARY KEY (protocol, target_type, target_id)
);
```

### 3.3 Media Management

Media is the biggest storage challenge. An aggressive caching policy is required.

```typescript
class MediaManager {
  private cacheDir: string;
  private db: Database;
  
  // Content-addressable storage: files named by SHA-256 of URL
  async cacheMedia(url: string, opts?: { maxSize?: number }): Promise<LocalMediaRef> {
    const hash = await sha256(url);
    const localPath = path.join(this.cacheDir, hash.slice(0, 2), hash);
    
    if (await fileExists(localPath)) {
      return { path: localPath, cached: true };
    }
    
    // Download with size limit
    const response = await fetch(url, { size: opts?.maxSize || 10 * 1024 * 1024 });
    const contentType = response.headers.get('content-type');
    
    await writeFile(localPath, response.body);
    
    // Update database
    this.db.run(`INSERT INTO attachments (hash, url, local_path, mime_type, size, cached_at)
      VALUES (?, ?, ?, ?, ?, ?)`, [hash, url, localPath, contentType, response.body.length, Date.now()]);
    
    return { path: localPath, cached: true };
  }
  
  // LRU eviction when cache exceeds limit
  async evictIfNeeded(maxSizeBytes: number = 500 * 1024 * 1024) {
    const currentSize = await this.getCacheSize();
    if (currentSize <= maxSizeBytes) return;
    
    // Evict oldest 20% of files
    const toEvict = await this.db.all(`
      SELECT hash, local_path FROM attachments 
      ORDER BY last_accessed_at ASC 
      LIMIT ?
    `, Math.ceil(currentSize / maxSizeBytes * 0.2));
    
    for (const file of toEvict) {
      await deleteFile(file.local_path);
      await this.db.run(`DELETE FROM attachments WHERE hash = ?`, file.hash);
    }
  }
}
```

**Cache limits (recommended):**
- Images: 500MB cache, maximum image dimension 2048px on resize (store thumbnails, not originals)
- Videos: 2GB cache, stream instead of full download for files > 10MB
- Avatars: Keep all (they're small), refresh every 24 hours
- Total: 3GB hard cap with configurable limit

### 3.4 Offline-First Architecture

```typescript
class OfflineManager {
  private db: Database;
  private syncEngines: Map<string, SyncEngine>;
  private pendingQueue: PendingAction[];
  
  // Queue actions taken while offline
  async enqueue(action: PendingAction) {
    await this.db.run(`INSERT INTO pending_actions 
      (id, protocol, action_type, payload, created_at, status) 
      VALUES (?, ?, ?, ?, ?, 'pending')`,
      [action.id, action.protocol, action.type, JSON.stringify(action.payload), Date.now()]);
    
    // Attempt to process immediately if we're online
    if (this.isOnline) {
      await this.processQueue();
    }
  }
  
  // Process queue when connectivity returns
  async processQueue() {
    const pending = await this.db.all(
      `SELECT * FROM pending_actions WHERE status = 'pending' ORDER BY created_at ASC LIMIT 100`
    );
    
    for (const action of pending) {
      try {
        const engine = this.syncEngines.get(action.protocol);
        await engine.execute(action);
        await this.db.run(`UPDATE pending_actions SET status = 'completed' WHERE id = ?`, action.id);
      } catch (error) {
        // Retry with exponential backoff, or mark as failed after N attempts
        if (action.retry_count >= 3) {
          await this.db.run(`UPDATE pending_actions SET status = 'failed', error = ? WHERE id = ?`, 
            error.message, action.id);
        } else {
          await this.db.run(`UPDATE pending_actions SET retry_count = retry_count + 1 WHERE id = ?`, action.id);
        }
      }
    }
  }
  
  // Connectivity monitor
  private setupConnectivityListener() {
    // NetInfo for React Native
    // NWPathMonitor for iOS/macOS
    // ConnectivityManager network callback for Android
    addEventListener('connectivityChange', (online) => {
      this.isOnline = online;
      if (online) this.processQueue();
    });
  }
}
```

### 3.5 Full-Text Search Across Protocols

SQLite FTS5 is the recommended engine. It supports:
- Language-aware tokenization (built-in `unicode61`, `porter` stemmer for English)
- Prefix queries, phrase queries, column-specific queries
- Ranking (BM25 variant)
- Incremental indexing via triggers (as shown in schema)

```typescript
class UnifiedSearch {
  async search(query: string, filters?: SearchFilters): Promise<SearchResult[]> {
    // FTS5 query syntax: "hello world" for phrase, hello* for prefix
    const ftsQuery = this.buildFtsQuery(query);
    
    const results = await this.db.all(`
      SELECT p.id, p.protocol, p.content, p.created_at,
             pr.display_name, pr.avatar_url,
             rank
      FROM posts_fts
      JOIN posts p ON p.rowid = posts_fts.rowid
      JOIN profiles pr ON pr.id = p.author_id
      WHERE posts_fts MATCH ?
        AND (? IS NULL OR p.protocol = ?)
        AND (? IS NULL OR p.author_id = ?)
        AND (? IS NULL OR p.created_at >= ?)
      ORDER BY rank DESC
      LIMIT 50
    `, [ftsQuery, filters?.protocol, filters?.protocol, 
         filters?.authorId, filters?.authorId,
         filters?.since, filters?.since]);
    
    return results.map(r => ({
      ...r,
      // Highlight matching terms
      highlights: this.extractHighlights(r.content, query)
    }));
  }
  
  private buildFtsQuery(userQuery: string): string {
    // Sanitize and transform user query for FTS5
    return userQuery
      .replace(/[^\w\s*"@#-]/g, '')     // Remove special chars except wildcards and quotes
      .split(/\s+/)
      .filter(Boolean)
      .map(term => term + '*')            // Prefix matching for better UX
      .join(' AND ');
  }
}
```

### 3.6 Sync Orchestration

```typescript
class SyncOrchestrator {
  private adapters: Map<string, ProtocolAdapter>;
  private db: Database;
  private bus: EventBus;
  
  async syncAll(): Promise<SyncReport> {
    const report: SyncReport = { started: Date.now(), results: {} };
    
    // Run sync for each protocol in parallel
    const syncs = Array.from(this.adapters.entries()).map(async ([protocol, adapter]) => {
      try {
        const state = await this.db.get(
          'SELECT * FROM protocol_sync_state WHERE protocol = ?', protocol
        );
        
        const since = state?.cursor || null;
        const result = await adapter.sync(since);
        
        // Update cursor
        await this.db.run(
          `INSERT OR REPLACE INTO protocol_sync_state (protocol, last_sync_at, cursor, state)
           VALUES (?, ?, ?, 'active')`,
          [protocol, Date.now(), result.newCursor]
        );
        
        report.results[protocol] = { success: true, items: result.items, error: null };
      } catch (error) {
        report.results[protocol] = { success: false, items: 0, error: error.message };
        await this.db.run(
          `UPDATE protocol_sync_state SET state = 'error', error_message = ? WHERE protocol = ?`,
          [error.message, protocol]
        );
      }
    });
    
    await Promise.all(syncs);
    report.ended = Date.now();
    return report;
  }
  
  // Quick sync: only pull new data (no historical catch-up)
  async quickSync(): Promise<SyncReport> {
    // For each adapter, request only data newer than last sync
    return this.syncAll(); // same method, relies on cursor being set
  }
  
  // Initial sync: might take a long time (minutes)
  async initialSync(progressCallback: (progress: SyncProgress) => void): Promise<void> {
    const protocols = Array.from(this.adapters.keys());
    let completed = 0;
    
    for (const protocol of protocols) {
      const adapter = this.adapters.get(protocol)!;
      
      await adapter.connect(); // Ensure connected
      await adapter.sync(null); // Full sync (no cursor)
      
      completed++;
      progressCallback({ protocol, progress: completed / protocols.length });
    }
  }
}
```

---

## 4. Technical Stack Recommendations

### 4.1 Cross-Platform Framework

| Framework | Pros | Cons | Verdict |
|---|---|---|---|
| **Tauri (Rust + webview)** | Small binary, native performance, direct Rust FFI, good SQLite support via rusqlite | Limited mobile support (Tauri Mobile is early), WebView JS bridge overhead for heavy UIs | **Best for desktop-first** |
| **Flutter (Dart)** | Excellent cross-platform (mobile + desktop), good Rust FFI via flutter_rust_bridge, rich UI | Dart ecosystem for protocol libraries is weakest, larger binary | **Best for mobile-first** |
| **React Native** | Largest ecosystem, TypeScript-native, many protocol libs in JS/TS, Expo ecosystem | JS bridge overhead for native modules, complex native config | **Best for rapid prototyping** |
| **Kotlin Multiplatform** | First-class XMPP support (Smack), Jetpack Compose UI, growing ecosystem | iOS support still maturing, smaller community, complex build setup | **Niche: protocol-heavy apps** |

**Recommendation: Hybrid approach**

Use **Tauri** for desktop targets (macOS, Windows, Linux) with the UI built in a webview framework of the user's choice (Svelte, React, Solid -- all work). Use **Flutter** for mobile targets (iOS, Android) with a shared Rust core via `flutter_rust_bridge` for the heavy protocol lifting.

Rationale:
- Rust is the only language with first-class support for all 6 protocols (via FFI or native libs)
- Sharing a Rust core between Tauri and Flutter is realistic; sharing a Dart core with a Rust desktop app is not
- The webview-based UI is lighter for desktop; Flutter's hardware-accelerated canvas is better for mobile animations

### 4.2 Language Recommendations for Protocol Implementations

| Component | Recommended language | Why |
|---|---|---|
| **Protocol adapters (shared core)** | Rust | matrix-rust-sdk, nostr-sdk, megalodon-rs, libomemo, tokio-irc, xmpp-rs. Rust has the best protocol library coverage. |
| **SQLite layer** | Rust (rusqlite) | Direct access, no FFI overhead, compile-time checked queries |
| **UI** | Svelte (Tauri) / Dart (Flutter) | Svelte is lightweight for desktop; Flutter is mature for mobile |
| **FFI bridge** | flutter_rust_bridge (Flutter) / Tauri commands (Rust-JS bridge) | Both mature and well-documented |
| **Media processing** | Rust (image, ffmpeg-next) | Performance-critical resizing and transcoding |
| **Encryption** | Rust (vodozemac for Matrix, omemo-rs for XMPP, nostr-sdk for Nostr) | Cryptographic code should be in a memory-safe language with timing-attack resistance |

### 4.3 Key Libraries Summary

| Protocol | JavaScript/TypeScript | Rust | Dart | Kotlin/Java |
|---|---|---|---|---|
| **ActivityPub** | megalodon | elefren / megalodon-rs | mastodon_api | Mastodon4J |
| **AT Protocol** | @atproto/api | atproto-rs (community) | bluesky (Dart) | atprotocol (Kotlin) |
| **Nostr** | nostr-tools | nostr-sdk | nostr (Dart) | nostr-postr |
| **Matrix** | matrix-js-sdk | **matrix-rust-sdk** | matrix | matrix-android-sdk2 |
| **IRC** | irc-framework | tokio-irc | irc (Dart) | pircbotx / kirc |
| **XMPP** | xmpp.js / stanza.js | xmpp-rs | xmpp (Dart) | **Smack** |
| **Database** | sql.js / react-native-quick-sqlite | rusqlite | drift (moor) | Room + SQLCipher |
| **Auth** | expo-secure-store / keytar | keyring-rs | flutter_secure_storage | Android Keystore |
| **UI** | Svelte / React | (webview) | Flutter widgets | Jetpack Compose |

### 4.4 Build & Distribution Strategy

**Desktop (Tauri):**
- Single binary per platform (macOS .app, Windows .msi, Linux .AppImage/.deb)
- User data stored in standard locations (~/Library/Application Support/, ~/.local/share/)
- Auto-update via Tauri updater (or native mechanisms like Sparkle on macOS)
- System tray for background operation (critical for IRC, XMPP presence)

**Mobile (Flutter):**
- iOS: App Store distribution, capable of background fetch (30s window)
- Android: Play Store + F-Droid, background service for persistent connections
- Push notifications via FCM/APNs for protocols that support them
- On Android, use a Foreground Service for persistent IRC/XMPP connections (with ongoing notification, as required by Android 12+)

---

## 5. Challenges & Risks

### 5.1 Rate Limiting

Every protocol server has rate limits, and aggregating six protocols means hitting six different limit regimes simultaneously.

| Protocol | Typical limits | Mitigation |
|---|---|---|
| ActivityPub (Mastodon API) | 300 req / 5 min per IP (default), 300 req / 5 min per user (app-level) | Cache aggressively, batch reads, use streaming API for real-time |
| AT Protocol | 3000 req / 5 min per PDS, varies by PDS operator | Local firehose subscription reduces XRPC calls by ~90% |
| Nostr | Relay-level limits (varies widely, 50-500 req/min common) | Distribute reads across relays; maintain connection quality table |
| Matrix | Homeserver-dependent, typically 50 req/10s | Use `/sync` long-poll, batch operations |
| IRC | Server-dependent, "flood" protection (2-8 lines/sec) | Rate-limit output (one message per 200ms minimum) |
| XMPP | No fixed limit, server-dependent ("karma" systems exist) | Use stream management, don't flood stanza bursts |

**System-wide rate limiter:**

```typescript
class RateLimiter {
  private limits: Map<string, { maxRequests: number, windowMs: number, counts: number[] }>;
  
  async throttle(protocol: string): Promise<void> {
    const limit = this.limits.get(protocol)!;
    const now = Date.now();
    
    // Remove expired entries from the sliding window
    limit.counts = limit.counts.filter(t => now - t < limit.windowMs);
    
    if (limit.counts.length >= limit.maxRequests) {
      const waitMs = limit.counts[0] + limit.windowMs - now;
      if (waitMs > 0) {
        await sleep(waitMs);
      }
    }
    
    limit.counts.push(now);
  }
}
```

### 5.2 Storage Growth Management

A heavy user across 6 protocols can accumulate data at alarming rates.

**Estimated growth rates:**

| Data type | Rate per user | Annual estimate |
|---|---|---|
| Posts (text) | ~100/day across all protocols | 36,500 posts (~50MB) |
| Messages (DMs) | ~50/day across protocols | 18,250 messages (~30MB) |
| Media (images) | ~20/day downloaded | ~3.6GB (at 500KB avg) |
| Profile data | ~200 profiles active | ~5MB |
| Sync state | Minimal | ~1MB |
| Database indices | Growing | ~200MB for FTS index on 100K posts |

**Mitigation:**
- Configurable retention policies: "Keep last 30/90/365 days of posts"
- Auto-archive posts older than N days to a compressed SQLite store
- Transcode images to WebP/AVIF with quality 80% and max dimension 1920px
- Cache videos as HLS streams (low, medium, high quality) rather than full originals
- Offer a "compact mode" that stores text-only for protocols where the user primarily follows text posters
- Allow per-protocol data deletion ("delete all cached IRC messages older than 7 days")

### 5.3 Battery & Performance on Mobile

Persistent connections to 6+ protocols are a battery killer.

**Connection strategy:**

| Protocol | Connection type | Background behavior |
|---|---|---|
| ActivityPub | WebSocket | Disconnect when backgrounded. Use polling every 5min + push notification for mentions |
| AT Protocol | Firehose WebSocket | Disconnect when backgrounded. Poll PDS cursor every 10min |
| Nostr | WebSocket x N relays | Disconnect when backgrounded. Use a single relay (the most reliable one) for light polling |
| Matrix | /sync long-poll | Keep alive with 30s timeout. Use Sliding Sync for reduced bandwidth. Signal app keeps this alive, so it's proven workable. |
| IRC | Persistent TCP | Disconnect when backgrounded. Bouncer does the heavy lifting. |
| XMPP | WebSocket/TCP | Keep alive with `stream:management` ack every 60s. |

**Additional optimizations:**
- **Coalesced timers**: Use iOS `beginBackgroundTask` and Android `WorkManager` to sync all protocols in a single background window
- **Batch notifications**: Present "3 new posts from 2 networks" instead of individual notifications
- **Throttle renders**: The UI should not re-render on every event. Batch events into 500ms windows before updating view state.
- **Lazy media**: Only download media when the post is visible on screen (use `IntersectionObserver` on desktop, `VisibilityDetector` on Flutter)

### 5.4 Protocol Evolution & Breaking Changes

Every protocol evolves, and a client supporting 6 of them is exposed to 6x the breakage risk.

| Protocol | Stability | Breaking change risk | Monitoring strategy |
|---|---|---|---|
| ActivityPub (Mastodon API) | Stable (v1/v2 APIs stable for years) | Low | Pin a specific API version |
| AT Protocol | Rapid evolution (Lexicon changes, OAuth migration) | **High** | Pin `@atproto/api` SDK version, subscribe to Bluesky developer changelog |
| Nostr | Rapid evolution (NIPs added weekly) | **High** | Accept NIP drift; implement a NIP version matrix and feature detection |
| Matrix | Slow evolution (MSC process takes 2+ years) | Low | Pin matrix-rust-sdk version |
| IRC | Extremely stable (core protocol unchanged for decades) | Very low | Support both RFC 1459 and RFC 7194 (IRCv3) |
| XMPP | Slow evolution (XEP process) | Low | Pin XEP support at specific versions |

**Mitigation strategies:**
1. **Version-lock** each adapter's SDK dependencies. Don't auto-upgrade.
2. **Feature flags**: Each adapter reports its capabilities as part of the connection handshake. The UI can degrade gracefully ("reactions not available on this protocol").
3. **Protocol health checks**: Periodic connectivity tests ping each protocol's test endpoints.
4. **Build-time validation**: CI pipeline that connects to test instances of each protocol and validates data model serialization.
5. **Graceful degradation**: If Bluesky changes the Lexicon for posts, the AT Protocol adapter falls back to showing `raw.record` until the SDK is updated, rather than crashing.

### 5.5 Legal & Compliance Issues

#### GDPR and Data Ownership

Since this is a client-side app with no backend, most GDPR compliance falls on the user. However:

- **Data export**: The app must provide a way to export all stored data in a portable format (JSON + media files). This is both a GDPR requirement (for the user to exercise data portability) and a practical necessity (the user owns their data).
- **Data deletion**: Allow the user to delete all data associated with a protocol account without affecting others.
- **Personal data minimization**: Don't store data you don't need. Profile data older than 30 days without interaction can be purged.

#### End-to-End Encryption Regulations

This is the most legally complex area:

- **Matrix (Olm/Megolm)**: E2EE is enabled by default in Matrix. In some jurisdictions (UK with the Online Safety Bill, proposed EU Chat Control), E2EE is under regulatory pressure. The app should not weaken encryption but should be transparent about its use.
- **XMPP (OMEMO)**: Same considerations as Matrix, but OMEMO is not enabled by default in most clients. The app should offer OMEMO as opt-in (or warn the user before enabling).
- **Nostr (NIP-04/NIP-17)**: Nostr's E2EE is optional and basic. NIP-17 is more robust but still not equivalent to the Signal protocol. The app should clearly communicate the security level of each protocol.
- **Local encryption**: The database should be encrypted at rest using SQLCipher (on mobile) or OS-level file encryption (FileVault on macOS, BitLocker on Windows, LUKS on Linux). The encryption key should be derived from the user's device passcode/biometrics via platform keychain.

#### Content Moderation Liability

Different jurisdictions impose different content liability rules:

- **Germany's NetzDG**: Requires swift removal of flagged illegal content. The app could be considered a "platform" even though it's a client. Mitigation: The app does not host content -- it merely displays content from external servers. This should exempt it from platform liability.
- **Section 230 (US)**: Protects platforms from liability for user content. The client app is likely protected as an "interactive computer service" provider.
- **Terrestrial bans**: Some protocols are banned in certain regions (e.g., Matrix was blocked in Russia, XMPP in China). The app should respect geoblocking for protocol-specific features.

**Recommended approach:**
- Clearly state in the app's TOS/README that the user is responsible for the content they view and publish.
- Implement client-side content filtering (keyword mutes, user blocks, instance blocks) that works across all protocols.
- Do NOT implement server-side content reporting from the client -- this is the protocol server's responsibility.
- Provide a "report" function that passes the content to the appropriate protocol's moderation system (Mastodon report API, Matrix room reporting, etc.).

#### Export Control

Cryptography export regulations (US EAR, EU Dual-Use Regulation) may apply:
- Matrix's Olm/Megolm and XMPP's OMEMO use AES-256 and Curve25519, which are classified under 5A002 (mass market encryption, exempt from notification in most jurisdictions).
- The app should be distributed through app stores that handle export classification (Apple, Google).
- For side-loading (F-Droid, direct download), include an export compliance notice.

---

## Appendix A: Protocol Comparison Quick Reference

| Aspect | ActivityPub | AT Protocol | Nostr | Matrix | IRC | XMPP |
|---|---|---|---|---|---|---|
| Year introduced | 2018 | 2022 | 2020 | 2014 | 1988 | 1999 |
| Transport | HTTP + WebSocket | HTTP + WebSocket | WebSocket | HTTPS + WebSocket | TCP (raw) | TCP + WebSocket + BOSH |
| Data format | JSON-LD | CBOR + JSON | JSON | JSON | Plain text + IRCv3 tags | XML (stanzas) |
| Identity | acct:user@domain | DID:plc/did:web + handle | npub/nsec hex key | @user:server | nick@network | user@domain |
| Authentication | HTTP Signatures / OAuth2 | OAuth + DPoP | Ed25519 signing | Access token | SASL (NickServ) | SASL (SCRAM) |
| Encryption | No | No | Optional (NIP-44) | Olm/Megolm (E2EE) | No (plaintext) | OMEMO (E2EE) |
| Decentralization | Federated | Federated | Fully distributed | Federated | Federated (IRC network) | Federated |
| Offline storage | Server | PDS | Relay | Server | Bouncer | Server |
| Query language | REST API | XRPC (Lexicon) | Subscription filters | /sync + filters | IRC commands | XMPP IQ stanzas |
| Search availability | API-dependent | Good (atmosphere) | NIP-50 (varies) | Full-text search | No | No |

## Appendix B: Glossary

| Term | Definition |
|---|---|
| **ActivityPub** | W3C standard for decentralized social networking. Used by Mastodon, Pleroma, Akkoma, Lemmy, PeerTube. |
| **AT Protocol** | Bluesky's protocol stack. Includes XRPC, Lexicon schemas, DID resolution, and a relay-based firehose. |
| **Bouncer** | A server-side proxy for IRC that maintains persistent connection to IRC servers while the client is offline. Example: ZNC, soju. |
| **BOSH** | Bidirectional-streams Over Synchronous HTTP. An XMPP transport that emulates TCP over HTTP long-polling. |
| **CAR** | Content Addressable aRchive. The file format used by AT Protocol to bundle repository commits. |
| **C2S** | Client-to-Server protocol (ActivityPub). Distinct from the Server-to-Server (S2S/federation) protocol. |
| **CRDT** | Conflict-free Replicated Data Type. A data structure that allows concurrent edits to be merged without conflicts. |
| **DID** | Decentralized Identifier. Used by AT Protocol (DID:PLC, DID:WEB). Not to be confused with Matrix's user IDs. |
| **FTS5** | SQLite's Full-Text Search extension (version 5). Supports BM25 ranking, prefix queries, column-specific search. |
| **JID** | Jabber ID. The XMPP user identifier format: user@domain/resource. |
| **Lexicon** | AT Protocol's schema system. Defines the structure of records (posts, likes, follows) as machine-readable JSON schemas. |
| **MAM** | Message Archive Management (XEP-0313). XMPP protocol for querying message history from the server. |
| **MSC** | Matrix Spec Change. The process by which the Matrix protocol evolves. MSC->Proposed, MSC->Accepted, MSC->Merged. |
| **MST** | Merkle Search Tree. The data structure used by AT Protocol to organize records in a user's repository. |
| **MUC** | Multi-User Chat (XEP-0045). XMPP's group chat protocol. |
| **MXID** | Matrix user ID. Format: @user:server. |
| **NIP** | Nostr Implementation Possibility. Nostr's equivalent of XEPs or MSCs -- a numbered specification document. |
| **OMEMO** | XEP-0384. XMPP's E2EE protocol based on the Signal Protocol (Double Ratchet + X3DH). |
| **PDS** | Personal Data Server. AT Protocol's user data store. Can be self-hosted or provided by Bluesky. |
| **PLC** | DID:PLC. AT Protocol's DID method, managed by the PLC directory. |
| **SASL** | Simple Authentication and Security Layer. Used by IRC (SASL PLAIN) and XMPP (SCRAM-SHA-256). |
| **Sliding Sync** | MSC3575. Matrix's modern sync mechanism for efficient incremental sync with room filtering. |
| **XEP** | XMPP Extension Protocol. XMPP's specification extension mechanism. |
| **XRPC** | AT Protocol's RPC framework. Uses Lexicon schemas to define typed API endpoints. |

---

*This architecture analysis is a living document. Protocol integrations will evolve as the underlying protocols evolve. The abstraction layer is designed to isolate these changes to individual adapter modules.*
