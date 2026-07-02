# Liberation Client — Core Systems: Event Bus & Nostr Adapter Spec

> Especificação de implementação: o backbone de eventos e o adapter do protocolo Tier 1.
> **Iteração 5** — 2026-07-01

---

## Part 1: Event Bus Architecture

### 1.1 Design Rationale

The Event Bus is the central nervous system of the Liberation Client. Every protocol adapter emits normalized events into it. The local data layer subscribes to persist events. The UI subscribes to render updates. No component directly touches another.

**Why `tokio::sync::broadcast`**: 
- O(1) dispatch per event type
- Strong compile-time type safety (no `dyn Any` downcasting)
- No lock contention vs shared `Vec<Handler>`
- Battle-tested in production (GitLab Knowledge Graph, WhatsApp Rust client)
- Single-threaded receivers are fine for UI updates via `tokio::sync::watch` or `ArcSwap`

**What we avoid**:
- Actor frameworks (unnecessary complexity for a local-only bus)
- CQRS/Event Sourcing (overengineered for local-first; SQLite is the event store)
- `async_broadcast` (extra dependency; `tokio::sync::broadcast` suffices)

### 1.2 Canonical Event Types

```rust
// libs/event-bus/src/lib.rs

use chrono::{DateTime, Utc};
use tokio::sync::broadcast;

/// Capacity per event channel. 1024 is generous for a single-user client.
const EVENT_CHANNEL_CAPACITY: usize = 1024;

/// The canonical event bus. One sender per event type.
/// All protocol adapters emit into these channels.
/// The local data layer and UI subscribe to these channels.
pub struct EventBus {
    // --- Timeline events ---
    pub new_post: broadcast::Sender<Arc<NewPostEvent>>,
    pub deleted_post: broadcast::Sender<Arc<DeletedPostEvent>>,
    pub edited_post: broadcast::Sender<Arc<EditedPostEvent>>,

    // --- Profile events ---
    pub profile_updated: broadcast::Sender<Arc<ProfileUpdatedEvent>>,

    // --- Interaction events ---
    pub new_reaction: broadcast::Sender<Arc<NewReactionEvent>>,
    pub removed_reaction: broadcast::Sender<Arc<RemovedReactionEvent>>,

    // --- Message events ---
    pub new_message: broadcast::Sender<Arc<NewMessageEvent>>,
    pub message_edited: broadcast::Sender<Arc<MessageEditedEvent>>,
    pub typing_indicator: broadcast::Sender<Arc<TypingIndicatorEvent>>,

    // --- Connection events ---
    pub protocol_connected: broadcast::Sender<Arc<ProtocolConnectedEvent>>,
    pub protocol_disconnected: broadcast::Sender<Arc<ProtocolDisconnectedEvent>>,
    pub connection_health: broadcast::Sender<Arc<ConnectionHealthEvent>>,

    // --- Sync events ---
    pub sync_started: broadcast::Sender<Arc<SyncStartedEvent>>,
    pub sync_progress: broadcast::Sender<Arc<SyncProgressEvent>>,
    pub sync_completed: broadcast::Sender<Arc<SyncCompletedEvent>>,
    pub sync_error: broadcast::Sender<Arc<SyncErrorEvent>>,

    // --- App events ---
    pub connectivity_changed: broadcast::Sender<Arc<ConnectivityChangedEvent>>,
    pub identity_linked: broadcast::Sender<Arc<IdentityLinkedEvent>>,
}

impl EventBus {
    pub fn new() -> Self {
        let (new_post_tx, _) = broadcast::channel(EVENT_CHANNEL_CAPACITY);
        let (deleted_post_tx, _) = broadcast::channel(EVENT_CHANNEL_CAPACITY);
        let (edited_post_tx, _) = broadcast::channel(EVENT_CHANNEL_CAPACITY);
        let (profile_updated_tx, _) = broadcast::channel(EVENT_CHANNEL_CAPACITY);
        let (new_reaction_tx, _) = broadcast::channel(EVENT_CHANNEL_CAPACITY);
        let (removed_reaction_tx, _) = broadcast::channel(EVENT_CHANNEL_CAPACITY);
        let (new_message_tx, _) = broadcast::channel(EVENT_CHANNEL_CAPACITY);
        let (message_edited_tx, _) = broadcast::channel(EVENT_CHANNEL_CAPACITY);
        let (typing_indicator_tx, _) = broadcast::channel(256);
        let (protocol_connected_tx, _) = broadcast::channel(32);
        let (protocol_disconnected_tx, _) = broadcast::channel(32);
        let (connection_health_tx, _) = broadcast::channel(64);
        let (sync_started_tx, _) = broadcast::channel(32);
        let (sync_progress_tx, _) = broadcast::channel(128);
        let (sync_completed_tx, _) = broadcast::channel(32);
        let (sync_error_tx, _) = broadcast::channel(64);
        let (connectivity_changed_tx, _) = broadcast::channel(16);
        let (identity_linked_tx, _) = broadcast::channel(32);

        Self {
            new_post: new_post_tx,
            deleted_post: deleted_post_tx,
            edited_post: edited_post_tx,
            profile_updated: profile_updated_tx,
            new_reaction: new_reaction_tx,
            removed_reaction: removed_reaction_tx,
            new_message: new_message_tx,
            message_edited: message_edited_tx,
            typing_indicator: typing_indicator_tx,
            protocol_connected: protocol_connected_tx,
            protocol_disconnected: protocol_disconnected_tx,
            connection_health: connection_health_tx,
            sync_started: sync_started_tx,
            sync_progress: sync_progress_tx,
            sync_completed: sync_completed_tx,
            sync_error: sync_error_tx,
            connectivity_changed: connectivity_changed_tx,
            identity_linked: identity_linked_tx,
        }
    }
}
```

### 1.3 Event Payload Definitions

```rust
// libs/event-bus/src/events.rs

use crate::canonical::{UnifiedPost, UnifiedProfile, UnifiedMessage, UnifiedReaction, Protocol};

use chrono::{DateTime, Utc};
use std::sync::Arc;

/// A new post has been received from any protocol.
#[derive(Debug, Clone)]
pub struct NewPostEvent {
    pub post: UnifiedPost,
    /// Which protocol this came from.
    pub protocol: Protocol,
    /// True if this is a backfill (historical sync), false if live.
    pub is_backfill: bool,
    /// When the event was emitted locally.
    pub received_at: DateTime<Utc>,
}

/// A post was deleted (or tombstoned).
#[derive(Debug, Clone)]
pub struct DeletedPostEvent {
    pub post_id: String,
    pub protocol: Protocol,
    /// The protocol-native deletion reason, if any.
    pub reason: Option<String>,
}

/// A post was edited.
#[derive(Debug, Clone)]
pub struct EditedPostEvent {
    pub post: UnifiedPost,
    pub protocol: Protocol,
    /// Fields that changed.
    pub changed_fields: Vec<String>,
}

/// A profile was updated.
#[derive(Debug, Clone)]
pub struct ProfileUpdatedEvent {
    pub profile: UnifiedProfile,
    pub protocol: Protocol,
}

/// A new reaction (like, repost, emoji react) received.
#[derive(Debug, Clone)]
pub struct NewReactionEvent {
    pub reaction: UnifiedReaction,
    pub protocol: Protocol,
}

/// A reaction was removed.
#[derive(Debug, Clone)]
pub struct RemovedReactionEvent {
    pub reaction_id: String,
    pub post_id: String,
    pub protocol: Protocol,
}

/// A new direct/group message.
#[derive(Debug, Clone)]
pub struct NewMessageEvent {
    pub message: UnifiedMessage,
    pub protocol: Protocol,
}

/// A message was edited.
#[derive(Debug, Clone)]
pub struct MessageEditedEvent {
    pub message: UnifiedMessage,
    pub protocol: Protocol,
}

/// Someone is typing (if protocol supports it).
#[derive(Debug, Clone)]
pub struct TypingIndicatorEvent {
    pub conversation_id: String,
    pub user_id: String,
    pub is_typing: bool,
    pub protocol: Protocol,
}

/// A protocol adapter connected successfully.
#[derive(Debug, Clone)]
pub struct ProtocolConnectedEvent {
    pub protocol: Protocol,
    pub identity: String,        // e.g. "@alice@mastodon.social"
    pub connected_at: DateTime<Utc>,
}

/// A protocol adapter disconnected.
#[derive(Debug, Clone)]
pub struct ProtocolDisconnectedEvent {
    pub protocol: Protocol,
    pub reason: DisconnectReason,
    pub will_retry: bool,
    pub retry_in_seconds: Option<u64>,
}

#[derive(Debug, Clone)]
pub enum DisconnectReason {
    NetworkError,
    AuthFailed,
    RateLimited,
    ServerError,
    UserInitiated,
    Timeout,
}

/// Periodic connection health report.
#[derive(Debug, Clone)]
pub struct ConnectionHealthEvent {
    pub protocol: Protocol,
    pub status: ConnectionStatus,
    pub latency_ms: Option<u64>,
    pub connected_relays: Option<usize>,  // Nostr-specific
    pub last_event_at: Option<DateTime<Utc>>,
}

#[derive(Debug, Clone)]
pub enum ConnectionStatus {
    Healthy,
    Degraded,     // slow but working
    Reconnecting,
    Down,
}

/// A sync operation started.
#[derive(Debug, Clone)]
pub struct SyncStartedEvent {
    pub protocol: Protocol,
    pub sync_type: SyncType,
}

/// Sync progress update.
#[derive(Debug, Clone)]
pub struct SyncProgressEvent {
    pub protocol: Protocol,
    pub sync_type: SyncType,
    pub items_processed: u64,
    pub items_total: Option<u64>,  // None if unknown
    pub estimated_completion: Option<DateTime<Utc>>,
}

/// Sync completed successfully.
#[derive(Debug, Clone)]
pub struct SyncCompletedEvent {
    pub protocol: Protocol,
    pub sync_type: SyncType,
    pub items_synced: u64,
    pub duration: chrono::Duration,
}

/// Sync failed.
#[derive(Debug, Clone)]
pub struct SyncErrorEvent {
    pub protocol: Protocol,
    pub sync_type: SyncType,
    pub error: String,
    pub retryable: bool,
}

#[derive(Debug, Clone)]
pub enum SyncType {
    ColdStart,
    Incremental,
    FullResync,
    MediaBackfill,
}

/// Device connectivity changed.
#[derive(Debug, Clone)]
pub struct ConnectivityChangedEvent {
    pub is_online: bool,
    pub connection_type: ConnectionType,
}

#[derive(Debug, Clone)]
pub enum ConnectionType {
    Wifi,
    Cellular,
    Ethernet,
    Unknown,
}

/// Two protocol identities were linked locally.
#[derive(Debug, Clone)]
pub struct IdentityLinkedEvent {
    pub protocol_a: Protocol,
    pub identity_a: String,
    pub protocol_b: Protocol,
    pub identity_b: String,
    pub confidence: f64,         // 0.0 to 1.0
    pub linking_method: LinkingMethod,
}

#[derive(Debug, Clone)]
pub enum LinkingMethod {
    ManualConfirmation,
    DomainMatch,
    CrossPostProof,
    HandleSimilarity,
}
```

### 1.4 Subscriber Pattern

```rust
// libs/event-bus/src/subscriber.rs

use tokio::sync::broadcast;

/// A subscriber listens to one or more event channels.
/// Each subscriber spawns a tokio task that processes events.
pub struct EventSubscriber {
    /// Unique subscriber ID for logging/debugging.
    pub id: String,
}

impl EventSubscriber {
    /// Subscribe to new posts. Returns a Receiver that yields Arc<NewPostEvent>.
    pub fn on_new_post(bus: &EventBus) -> broadcast::Receiver<Arc<NewPostEvent>> {
        bus.new_post.subscribe()
    }

    /// Subscribe to all timeline-relevant events (posts, reactions, deletions).
    pub fn on_timeline_events(
        bus: &EventBus,
    ) -> (
        broadcast::Receiver<Arc<NewPostEvent>>,
        broadcast::Receiver<Arc<DeletedPostEvent>>,
        broadcast::Receiver<Arc<EditedPostEvent>>,
        broadcast::Receiver<Arc<NewReactionEvent>>,
    ) {
        (
            bus.new_post.subscribe(),
            bus.deleted_post.subscribe(),
            bus.edited_post.subscribe(),
            bus.new_reaction.subscribe(),
        )
    }

    /// Subscribe to connection events for all protocols.
    pub fn on_connection_events(
        bus: &EventBus,
    ) -> (
        broadcast::Receiver<Arc<ProtocolConnectedEvent>>,
        broadcast::Receiver<Arc<ProtocolDisconnectedEvent>>,
        broadcast::Receiver<Arc<ConnectionHealthEvent>>,
    ) {
        (
            bus.protocol_connected.subscribe(),
            bus.protocol_disconnected.subscribe(),
            bus.connection_health.subscribe(),
        )
    }
}

/// Macro to simplify consuming from multiple channels with select!.
/// Prevents silent lag (dropped events when receiver buffer fills).
#[macro_export]
macro_rules! consume_events {
    ($rx:expr, $handler:expr) => {
        loop {
            match $rx.recv().await {
                Ok(event) => $handler(event).await,
                Err(broadcast::error::RecvError::Lagged(n)) => {
                    tracing::warn!("Event subscriber lagged by {} events", n);
                    // Continue — we missed events, but protocol sync will reconcile.
                }
                Err(broadcast::error::RecvError::Closed) => break,
            }
        }
    };
}
```

### 1.5 Data Layer Integration

The local data layer (`libs/storage/`) subscribes to the event bus and persists events to SQLite:

```rust
// libs/storage/src/event_persister.rs

pub struct EventPersister {
    db: SqlitePool,
    subscriber_id: String,
}

impl EventPersister {
    pub fn new(db: SqlitePool) -> Self {
        Self { db, subscriber_id: "storage/event-persister".into() }
    }

    /// Start persisting events. Runs forever.
    pub async fn run(self, bus: Arc<EventBus>) {
        let mut post_rx = bus.new_post.subscribe();
        let mut del_rx = bus.deleted_post.subscribe();
        let mut edit_rx = bus.edited_post.subscribe();
        let mut profile_rx = bus.profile_updated.subscribe();
        let mut reaction_rx = bus.new_reaction.subscribe();
        let mut msg_rx = bus.new_message.subscribe();
        let mut link_rx = bus.identity_linked.subscribe();

        loop {
            tokio::select! {
                Ok(event) = post_rx.recv() => {
                    if let Err(e) = self.persist_post(&event).await {
                        tracing::error!("Failed to persist post: {}", e);
                    }
                }
                Ok(event) = del_rx.recv() => {
                    if let Err(e) = self.persist_deletion(&event).await {
                        tracing::error!("Failed to persist deletion: {}", e);
                    }
                }
                Ok(event) = edit_rx.recv() => {
                    if let Err(e) = self.persist_edit(&event).await {
                        tracing::error!("Failed to persist edit: {}", e);
                    }
                }
                Ok(event) = profile_rx.recv() => {
                    if let Err(e) = self.persist_profile(&event).await {
                        tracing::error!("Failed to persist profile: {}", e);
                    }
                }
                Ok(event) = reaction_rx.recv() => {
                    if let Err(e) = self.persist_reaction(&event).await {
                        tracing::error!("Failed to persist reaction: {}", e);
                    }
                }
                Ok(event) = msg_rx.recv() => {
                    if let Err(e) = self.persist_message(&event).await {
                        tracing::error!("Failed to persist message: {}", e);
                    }
                }
                Ok(event) = link_rx.recv() => {
                    if let Err(e) = self.persist_identity_link(&event).await {
                        tracing::error!("Failed to persist identity link: {}", e);
                    }
                }
                else => break, // All senders dropped
            }
        }
    }
    // ... persist methods omitted for brevity (see modelo-de-dados.md for schema)
}
```

---

## Part 2: Nostr Adapter Specification

### 2.1 Architecture Overview

```
┌─────────────────────────────────────────┐
│            NostrAdapter                  │
│                                          │
│  ┌──────────────────────────────────┐   │
│  │     RelayConnectionManager       │   │
│  │  ┌────────┐ ┌────────┐ ┌──────┐ │   │
│  │  │Relay A │ │Relay B │ │Relay C│ │   │
│  │  │ WS conn│ │ WS conn│ │WS conn│ │   │
│  │  └───┬────┘ └───┬────┘ └───┬───┘ │   │
│  │      │           │          │      │   │
│  │  ┌───▼───────────▼──────────▼───┐ │   │
│  │  │     EventDeduplicator        │ │   │
│  │  │  (event_id → seen set)       │ │   │
│  │  └─────────────┬────────────────┘ │   │
│  └────────────────┼──────────────────┘   │
│                   │                       │
│  ┌────────────────▼──────────────────┐   │
│  │       EventMapper                 │   │
│  │  (Nostr Event → Canonical Event)  │   │
│  └────────────────┬──────────────────┘   │
│                   │                       │
│  ┌────────────────▼──────────────────┐   │
│  │       NostrKeyManager             │   │
│  │  (sign, verify, NIP-44 encrypt)   │   │
│  └──────────────────────────────────┘   │
│                   │                       │
│          EventBus (emit)                  │
└──────────────────────────────────────────┘
```

### 2.2 RelayConnectionManager

```rust
// libs/protocols/nostr/src/relay_manager.rs

use std::collections::{HashMap, HashSet};
use std::sync::Arc;
use std::time::Duration;
use tokio::sync::{RwLock, Semaphore};
use tokio::time::MissedTickBehavior;

/// Manages connections to multiple Nostr relays.
/// Handles reconnection with exponential backoff, health tracking, and NIP-65 awareness.
pub struct RelayConnectionManager {
    relays: Arc<RwLock<HashMap<RelayUrl, RelayConnection>>>,
    /// Maximum concurrent WebSocket connections (mobile: 3, desktop: 8).
    max_connections: usize,
    /// Semaphore to limit concurrent connection attempts.
    connection_semaphore: Arc<Semaphore>,
    /// Per-relay backoff state.
    backoffs: Arc<RwLock<HashMap<RelayUrl, BackoffState>>>,
    /// Event bus for emitting events.
    bus: Arc<EventBus>,
}

struct RelayConnection {
    url: RelayUrl,
    state: RelayState,
    /// Capabilities discovered via NIP-11.
    capabilities: RelayCapabilities,
    /// Last health check result.
    health: RelayHealth,
    /// Active subscriptions on this relay.
    subscriptions: Vec<ActiveSubscription>,
}

#[derive(Debug, Clone, PartialEq)]
enum RelayState {
    Disconnected,
    Connecting,
    Connected,
    /// Connected but degraded (high latency, errors).
    Degraded,
    /// Temporarily sleeping to conserve resources.
    Sleeping,
    /// Permanently banned (policy violation, repeated failures).
    Banned,
}

#[derive(Debug, Clone)]
struct RelayCapabilities {
    read: bool,
    write: bool,
    search: bool,       // NIP-50
    supports_nip_44: bool,
    supports_nip_65: bool,
    supports_nip_77: bool,  // negentropy
    max_event_size: usize,
    max_subscriptions: usize,
    auth_required: bool,    // NIP-42
    payment_required: bool,
}

#[derive(Debug, Clone)]
struct RelayHealth {
    connected_at: Option<chrono::DateTime<chrono::Utc>>,
    last_event_at: Option<chrono::DateTime<chrono::Utc>>,
    avg_latency_ms: f64,
    error_count: u32,
    reconnection_count: u32,
}

#[derive(Debug, Clone)]
struct BackoffState {
    attempt: u32,
    current_delay: Duration,
    max_delay: Duration,
}

impl BackoffState {
    fn new() -> Self {
        Self {
            attempt: 0,
            current_delay: Duration::from_secs(1),
            max_delay: Duration::from_secs(300), // 5 minutes max
        }
    }

    /// Exponential backoff with jitter.
    fn next_delay(&mut self) -> Duration {
        self.attempt += 1;
        let base = Duration::from_secs(2u64.pow(self.attempt.min(10)));
        // Add ±25% jitter
        let jitter = (base.as_millis() as f64 * 0.25 * (rand::random::<f64>() * 2.0 - 1.0)) as u64;
        let delay = base + Duration::from_millis(jitter);
        self.current_delay = delay.min(self.max_delay);
        self.current_delay
    }

    fn reset(&mut self) {
        self.attempt = 0;
        self.current_delay = Duration::from_secs(1);
    }
}
```

### 2.3 Connection Lifecycle

```rust
impl RelayConnectionManager {
    /// Bootstrap: discover relays from NIP-65, then connect.
    pub async fn bootstrap(
        bus: Arc<EventBus>,
        key_manager: Arc<NostrKeyManager>,
        bootstrap_relays: Vec<String>,
    ) -> Result<Self> {
        let manager = Self {
            relays: Arc::new(RwLock::new(HashMap::new())),
            max_connections: if cfg!(target_os = "android") || cfg!(target_os = "ios") { 3 } else { 8 },
            connection_semaphore: Arc::new(Semaphore::new(if cfg!(target_os = "android") || cfg!(target_os = "ios") { 3 } else { 8 })),
            backoffs: Arc::new(RwLock::new(HashMap::new())),
            bus: bus.clone(),
        };

        // Phase 1: Connect to bootstrap relays.
        for url in &bootstrap_relays {
            manager.connect_to_relay(url.parse()?).await;
        }

        // Phase 2: Fetch user's NIP-65 relay list (kind:10002).
        // This tells us which relays the user publishes to and reads from.
        let nip65_relays = manager.fetch_user_relay_list().await?;
        for url in &nip65_relays.read_relays {
            if !bootstrap_relays.contains(url) {
                manager.connect_to_relay(url.parse()?).await;
            }
        }

        // Phase 3: Fetch followed users' NIP-65 relay lists.
        // This optimizes content discovery.
        let followed_pubkeys = manager.get_followed_pubkeys().await?;
        manager.discover_relays_from_contacts(&followed_pubkeys).await?;

        Ok(manager)
    }

    /// Connect to a specific relay with backoff-aware retry.
    async fn connect_to_relay(&self, url: RelayUrl) {
        // Check if already connected or banned.
        {
            let relays = self.relays.read().await;
            if let Some(conn) = relays.get(&url) {
                if conn.state == RelayState::Connected || conn.state == RelayState::Banned {
                    return;
                }
            }
        }

        // Acquire connection slot.
        let _permit = self.connection_semaphore.acquire().await.unwrap();

        // Get or create backoff state.
        let delay = {
            let mut backoffs = self.backoffs.write().await;
            let backoff = backoffs.entry(url.clone()).or_insert_with(BackoffState::new);
            backoff.next_delay()
        };

        // Wait for backoff if needed.
        tokio::time::sleep(delay).await;

        // Attempt connection.
        let result = self.establish_connection(&url).await;

        match result {
            Ok(connection) => {
                // Reset backoff on success.
                self.backoffs.write().await.get_mut(&url).unwrap().reset();

                // Fetch NIP-11 relay info document.
                if let Ok(info) = self.fetch_relay_info(&url).await {
                    connection.capabilities = info;
                }

                self.relays.write().await.insert(url.clone(), connection);

                // Emit connection event.
                let _ = self.bus.protocol_connected.send(Arc::new(ProtocolConnectedEvent {
                    protocol: Protocol::Nostr,
                    identity: url.to_string(),
                    connected_at: Utc::now(),
                }));

                // Start health check loop.
                self.spawn_health_check(url.clone());
            }
            Err(e) => {
                tracing::warn!("Failed to connect to relay {}: {}. Retry in {:?}", url, e, delay);
                // Will retry on next schedule.
            }
        }
    }

    /// Establish WebSocket connection to a relay.
    async fn establish_connection(&self, url: &RelayUrl) -> Result<RelayConnection> {
        let (ws_stream, _) = tokio_tungstenite::connect_async(url.as_str()).await?;

        // Split into read/write halves for bidirectional communication.
        let (write, read) = ws_stream.split();

        let connection = RelayConnection {
            url: url.clone(),
            state: RelayState::Connected,
            capabilities: RelayCapabilities::default(),
            health: RelayHealth::default(),
            subscriptions: Vec::new(),
        };

        // Spawn read loop.
        self.spawn_read_loop(url.clone(), read);
        // Spawn write loop (queued messages).
        self.spawn_write_loop(url.clone(), write);

        Ok(connection)
    }
}
```

### 2.4 Event Deduplication

```rust
// libs/protocols/nostr/src/deduplicator.rs

use std::collections::HashSet;
use std::sync::Arc;
use tokio::sync::RwLock;
use lru::LruCache;
use std::num::NonZeroUsize;

/// Deduplicates Nostr events by event_id.
/// Critical because the same event arrives from every connected relay.
pub struct EventDeduplicator {
    /// Recently seen event IDs with timestamp for TTL-based eviction.
    /// LRU cache with 50,000 entries — covers ~1 day of heavy Nostr usage.
    seen_events: Arc<RwLock<LruCache<String, chrono::DateTime<chrono::Utc>>>>,
}

impl EventDeduplicator {
    pub fn new() -> Self {
        Self {
            seen_events: Arc::new(RwLock::new(
                LruCache::new(NonZeroUsize::new(50_000).unwrap())
            )),
        }
    }

    /// Check if an event has already been seen.
    /// Returns true if the event is new (not a duplicate).
    pub async fn is_new(&self, event_id: &str) -> bool {
        let mut cache = self.seen_events.write().await;
        if cache.contains(event_id) {
            false
        } else {
            cache.put(event_id.to_string(), chrono::Utc::now());
            true
        }
    }

    /// Prune entries older than `max_age`. Called periodically.
    pub async fn prune(&self, max_age: chrono::Duration) {
        let mut cache = self.seen_events.write().await;
        let cutoff = chrono::Utc::now() - max_age;
        // LRU naturally evicts oldest, so this is fast.
        // We just need to peek and pop if too old.
        while let Some((_, ts)) = cache.peek_lru() {
            if *ts < cutoff {
                cache.pop_lru();
            } else {
                break;
            }
        }
    }
}
```

### 2.5 EventMapper — Nostr Event → Canonical Event

```rust
// libs/protocols/nostr/src/event_mapper.rs

impl NostrEventMapper {
    /// Map a Nostr Event to zero or more canonical events.
    /// Returns None if the event kind is unsupported or the event is malformed.
    pub async fn map_event(
        event: &nostr_sdk::Event,
        bus: &Arc<EventBus>,
    ) -> Result<Vec<CanonicalEventType>> {
        match event.kind {
            // Kind 0: Profile metadata
            nostr_sdk::Kind::Metadata => {
                let profile = Self::map_metadata_to_profile(event)?;
                Ok(vec![CanonicalEventType::ProfileUpdated(ProfileUpdatedEvent {
                    profile,
                    protocol: Protocol::Nostr,
                })])
            }

            // Kind 1: Short text note (the main "post" type)
            nostr_sdk::Kind::TextNote => {
                let post = Self::map_text_note_to_post(event)?;
                Ok(vec![CanonicalEventType::NewPost(NewPostEvent {
                    post,
                    protocol: Protocol::Nostr,
                    is_backfill: false,
                    received_at: Utc::now(),
                })])
            }

            // Kind 3 / 10000: Contact list (follows)
            nostr_sdk::Kind::ContactList => {
                let follows = Self::extract_follows_from_contact_list(event)?;
                Ok(follows.into_iter().map(|follow| {
                    CanonicalEventType::ProfileUpdated(follow)
                }).collect())
            }

            // Kind 5: Event deletion
            nostr_sdk::Kind::EventDeletion => {
                let deleted_ids = event.tags.iter()
                    .filter(|t| t.kind() == TagKind::Event)
                    .map(|t| t.content().to_string())
                    .collect::<Vec<_>>();
                Ok(deleted_ids.into_iter().map(|id| {
                    CanonicalEventType::DeletedPost(DeletedPostEvent {
                        post_id: id,
                        protocol: Protocol::Nostr,
                        reason: Some(event.content.clone()),
                    })
                }).collect())
            }

            // Kind 6: Repost
            nostr_sdk::Kind::Repost => {
                let reaction = Self::map_repost_to_reaction(event)?;
                Ok(vec![CanonicalEventType::NewReaction(NewReactionEvent {
                    reaction,
                    protocol: Protocol::Nostr,
                })])
            }

            // Kind 7: Reaction (like/emoji)
            nostr_sdk::Kind::Reaction => {
                let reaction = Self::map_reaction_event(event)?;
                Ok(vec![CanonicalEventType::NewReaction(NewReactionEvent {
                    reaction,
                    protocol: Protocol::Nostr,
                })])
            }

            // Kind 4: Encrypted DM (NIP-04) — DEPRECATED, only for backward compat
            nostr_sdk::Kind::EncryptedDirectMessage => {
                tracing::warn!("Received NIP-04 DM. NIP-04 is deprecated due to CBC malleability (Black Hat 2025).");
                // Decrypt only if no NIP-44 alternative exists for this conversation.
                // Prefer NIP-44 (kind 14 / 1059).
                Ok(vec![])
            }

            // Kind 14: Sealed DM (NIP-17) — preferred
            // Kind 1059: Gift wrap (NIP-59)
            nostr_sdk::Kind::GiftWrap => {
                let message = Self::decrypt_and_map_gift_wrap(event)?;
                Ok(vec![CanonicalEventType::NewMessage(NewMessageEvent {
                    message,
                    protocol: Protocol::Nostr,
                })])
            }

            // Kind 10002: Relay list metadata — internal use only
            nostr_sdk::Kind::RelayList => {
                // Handled by RelayConnectionManager, not emitted as canonical events.
                Ok(vec![])
            }

            // Kind 30000: Parameterized replaceable event — Starter Pack
            // (used by our community curation system, see governanca.md)
            nostr_sdk::Kind::Custom(30000) => {
                let pack = Self::map_starter_pack(event)?;
                Ok(vec![CanonicalEventType::StarterPackUpdated(pack)])
            }

            // Kind 30001: Parameterized replaceable event — Tasting Menu
            nostr_sdk::Kind::Custom(30001) => {
                let menu = Self::map_tasting_menu(event)?;
                Ok(vec![CanonicalEventType::TastingMenuUpdated(menu)])
            }

            _ => {
                // Unknown kind — ignore but log for discovery.
                tracing::debug!("Ignoring Nostr event kind: {:?}", event.kind);
                Ok(vec![])
            }
        }
    }

    /// Map Nostr kind:1 (text note) to UnifiedPost. CRITICAL path.
    fn map_text_note_to_post(event: &nostr_sdk::Event) -> Result<UnifiedPost> {
        // Extract threading from e tags (NIP-10).
        let mut reply_to: Option<String> = None;
        let mut root_post: Option<String> = None;
        let mut mentioned_events: Vec<String> = Vec::new();

        for tag in &event.tags {
            if tag.kind() == TagKind::Event {
                let event_id = tag.content().to_string();
                // Check for NIP-10 markers.
                if let Some(marker) = tag.kind().to_string().as_ref() {
                    match marker {
                        "reply" => reply_to = Some(event_id.clone()),
                        "root" => root_post = Some(event_id.clone()),
                        _ => mentioned_events.push(event_id),
                    }
                } else {
                    // No marker: first e tag is root, last is reply (NIP-10 fallback).
                    mentioned_events.push(event_id);
                }
            }
        }

        // If no markers, use NIP-10 positional convention.
        if reply_to.is_none() && !mentioned_events.is_empty() {
            reply_to = mentioned_events.pop();
        }
        if root_post.is_none() && !mentioned_events.is_empty() {
            root_post = Some(mentioned_events.remove(0));
        }

        // Extract hashtags from t tags.
        let tags: Vec<String> = event.tags.iter()
            .filter(|t| t.kind() == TagKind::Hashtag)
            .map(|t| t.content().to_string())
            .collect();

        // Extract mentioned pubkeys from p tags.
        let mentions: Vec<String> = event.tags.iter()
            .filter(|t| t.kind() == TagKind::Pubkey)
            .map(|t| t.content().to_string())
            .collect();

        // Extract media from imeta tags (NIP-92).
        let media: Vec<UnifiedAttachment> = event.tags.iter()
            .filter(|t| t.kind() == TagKind::Custom("imeta"))
            .filter_map(|t| Self::parse_imeta_tag(t))
            .collect();

        Ok(UnifiedPost {
            id: event.id.to_hex(),
            protocol: Protocol::Nostr,
            protocol_id: event.id.to_hex(),
            author: UnifiedAuthor {
                id: event.pubkey.to_hex(),
                handle: None, // Resolved later via kind:0.
                display_name: None,
                avatar_url: None,
                protocol: Protocol::Nostr,
            },
            content: event.content.clone(),
            rich_content: None, // Nostr is plain text. Optional: Markdown parsing.
            created_at: chrono::DateTime::from_timestamp(event.created_at.as_i64(), 0)
                .unwrap_or_default(),
            reply_to: reply_to,
            root_post: root_post,
            media,
            mentions,
            tags,
            visibility: Visibility::Public,
            language: None,
            source: serde_json::to_value(event).ok(),
        })
    }
}
```

### 2.6 Subscription Strategy

```rust
// libs/protocols/nostr/src/subscriptions.rs

impl NostrAdapter {
    /// Subscribe to the user's home feed.
    /// Uses NIP-65 to route subscriptions to optimal relays.
    pub async fn subscribe_home_feed(&self, since: Option<chrono::DateTime<chrono::Utc>>) {
        let followed_pubkeys = self.get_followed_pubkeys().await.unwrap_or_default();

        // Build filter.
        let mut filter = nostr_sdk::Filter::new()
            .kinds(vec![nostr_sdk::Kind::TextNote])
            .authors(followed_pubkeys.clone())
            .limit(500);

        if let Some(since) = since {
            filter = filter.since(since.into());
        }

        // Route to relays.
        // For each author, find their NIP-65 read relays.
        // If gossip is available, use GossipRelayResolver.
        // Otherwise, send to all connected read-capable relays.

        let relay_urls = self.get_optimal_relays_for_authors(&followed_pubkeys).await;

        for url in &relay_urls {
            let filter = filter.clone();
            let bus = self.bus.clone();
            let deduplicator = self.deduplicator.clone();
            let mapper = self.mapper.clone();

            tokio::spawn(async move {
                let stream = relay.stream_events(vec![filter])
                    .timeout(Duration::from_secs(30))
                    .policy(ReqExitPolicy::WaitForEventsAfterEOSE(10))
                    .await;

                // Process events as they arrive.
                while let Some(event_result) = stream.next().await {
                    match event_result {
                        Ok(event) => {
                            // DEDUP FIRST — critical.
                            if !deduplicator.is_new(&event.id.to_hex()).await {
                                continue;
                            }

                            // Map to canonical.
                            if let Ok(canonical_events) = mapper.map_event(&event, &bus).await {
                                for canonical in canonical_events {
                                    canonical.emit(&bus);
                                }
                            }
                        }
                        Err(e) => {
                            tracing::warn!("Error in Nostr subscription stream: {}", e);
                        }
                    }
                }
            });
        }
    }

    /// Subscribe to notifications: replies to user's posts, mentions, DMs.
    pub async fn subscribe_notifications(&self, user_pubkey: &str) {
        let filter = nostr_sdk::Filter::new()
            .kinds(vec![
                nostr_sdk::Kind::TextNote,  // posts that mention us
                nostr_sdk::Kind::GiftWrap,  // DMs
            ])
            .pubkey_tag(user_pubkey) // p tag = mentioned
            .limit(200);

        self.subscribe_to_all_read_relays(vec![filter]).await;
    }

    /// Subscribe to a specific thread.
    pub async fn subscribe_thread(&self, root_event_id: &str) {
        let filter = nostr_sdk::Filter::new()
            .kinds(vec![nostr_sdk::Kind::TextNote])
            .event_tag(root_event_id) // e tag references root event
            .limit(100);

        self.subscribe_to_all_read_relays(vec![filter]).await;
    }

    /// Subscribe to all connected read relays.
    async fn subscribe_to_all_read_relays(&self, filters: Vec<nostr_sdk::Filter>) {
        let relays = self.relay_manager.get_connected_read_relays().await;
        for relay_url in relays {
            let filters = filters.clone();
            // Same pattern as subscribe_home_feed...
        }
    }
}
```

### 2.7 Publishing

```rust
// libs/protocols/nostr/src/publish.rs

impl NostrAdapter {
    /// Publish a text note (kind:1) to write-capable relays.
    pub async fn publish_post(&self, content: &str, tags: Vec<Tag>) -> Result<UnifiedPost> {
        let pubkey = self.key_manager.get_pubkey()?;

        // Build event.
        let event = nostr_sdk::EventBuilder::new()
            .kind(nostr_sdk::Kind::TextNote)
            .content(content)
            .tags(tags)
            .sign_with_keys(&self.key_manager.get_signer()?)?;

        // Publish to ALL write-capable relays.
        let write_relays = self.relay_manager.get_write_relays().await;
        let publish_results = futures::future::join_all(
            write_relays.iter().map(|url| {
                let event = event.clone();
                async move {
                    let relay = self.relay_manager.get_relay(url).await?;
                    relay.send_event(event).await
                }
            })
        ).await;

        // At least one relay must accept.
        let success_count = publish_results.iter().filter(|r| r.is_ok()).count();
        if success_count == 0 {
            return Err(anyhow::anyhow!("Failed to publish to any relay"));
        }

        tracing::info!("Published to {}/{} relays", success_count, write_relays.len());

        // Map the event to a UnifiedPost and return it.
        let post = self.mapper.map_text_note_to_post(&event)?;

        // Emit locally immediately (optimistic).
        let _ = self.bus.new_post.send(Arc::new(NewPostEvent {
            post: post.clone(),
            protocol: Protocol::Nostr,
            is_backfill: false,
            received_at: Utc::now(),
        }));

        Ok(post)
    }

    /// Publish a reaction (kind:7) to a post.
    pub async fn publish_reaction(&self, target_event_id: &str, emoji: &str) -> Result<()> {
        let event = nostr_sdk::EventBuilder::new()
            .kind(nostr_sdk::Kind::Reaction)
            .content(emoji)
            .tags(vec![
                Tag::event(target_event_id.parse()?),
                Tag::pubkey(self.get_author_pubkey_of_event(target_event_id).await?),
            ])
            .sign_with_keys(&self.key_manager.get_signer()?)?;

        self.publish_to_write_relays(event).await
    }

    /// Publish a NIP-44 encrypted DM.
    pub async fn publish_dm(&self, recipient_pubkey: &str, plaintext: &str) -> Result<()> {
        // NIP-44: Encrypt with ChaCha20-Poly1305 using ECDH shared secret.
        let encrypted = self.key_manager.nip44_encrypt(recipient_pubkey, plaintext)?;

        // Wrap in gift wrap (NIP-59) for metadata protection.
        let gift_wrap = nostr_sdk::EventBuilder::new()
            .kind(nostr_sdk::Kind::GiftWrap)
            .content(encrypted)
            .tags(vec![Tag::pubkey(recipient_pubkey.parse()?)])
            .sign_with_keys(&self.key_manager.get_signer()?)?;

        self.publish_to_write_relays(gift_wrap).await
    }
}
```

### 2.8 NIP Compliance Checklist

Required NIPs with implementation status:

| NIP | Description | Priority | Status |
|-----|-------------|----------|--------|
| NIP-01 | Basic protocol | REQUIRED | ✅ Spec'd |
| NIP-02 | Contact list (kind:3) | REQUIRED | ✅ Spec'd |
| NIP-05 | DNS verification | HIGH | 🔲 To implement |
| NIP-09 | Event deletion (kind:5) | REQUIRED | ✅ Spec'd |
| NIP-10 | Thread markers | REQUIRED | ✅ Spec'd |
| NIP-11 | Relay info document | REQUIRED | ✅ Spec'd |
| NIP-19 | bech32 encoding (npub/nsec) | REQUIRED | ✅ Use nostr-sdk |
| NIP-25 | Reactions (kind:7) | HIGH | ✅ Spec'd |
| NIP-42 | AUTH | HIGH | 🔲 To implement |
| NIP-44 | Encrypted DMs (ChaCha20-Poly1305) | REQUIRED | ✅ Spec'd |
| NIP-50 | Search | MEDIUM | 🔲 Relay-dependent |
| NIP-57 | Zaps | LOW | 🔲 Optional |
| NIP-59 | Gift wrap | HIGH | ✅ Spec'd |
| NIP-65 | Relay list (kind:10002) | REQUIRED | ✅ Spec'd |
| NIP-77 | Negentropy sync | MEDIUM | 🔲 Future |
| NIP-92 | Media attachments (imeta) | MEDIUM | ✅ Spec'd |

**NIP-04 (Encrypted DMs — AES-CBC) is EXPLICITLY NOT supported** due to the CBC malleability vulnerability discovered in the Black Hat 2025 security evaluation. We NIP-44 only.

---

## Part 3: Sync Engine

### 3.1 Sync Orchestrator

```rust
// libs/sync/src/orchestrator.rs

/// The SyncEngine coordinates sync across ALL protocol adapters.
/// It respects battery state, connectivity, and user preferences.
pub struct SyncEngine {
    bus: Arc<EventBus>,
    adapters: HashMap<Protocol, Box<dyn ProtocolAdapter>>,
    /// Per-protocol sync state persisted in SQLite.
    sync_state: Arc<RwLock<HashMap<Protocol, SyncState>>>,
    /// Connectivity monitor.
    connectivity: Arc<ConnectivityMonitor>,
    /// Background sync task handle.
    background_task: Option<tokio::task::JoinHandle<()>>,
}

#[derive(Debug, Clone)]
pub struct SyncState {
    pub protocol: Protocol,
    pub last_sync_at: Option<DateTime<Utc>>,
    pub last_cursor: Option<String>,
    pub items_synced: u64,
    pub status: SyncStatus,
}

#[derive(Debug, Clone, PartialEq)]
pub enum SyncStatus {
    Idle,
    Syncing(SyncType),
    Error(String),
}

impl SyncEngine {
    /// Cold start: full initial sync for a newly connected protocol.
    pub async fn cold_start_sync(&self, protocol: Protocol) -> Result<()> {
        let _ = self.bus.sync_started.send(Arc::new(SyncStartedEvent {
            protocol,
            sync_type: SyncType::ColdStart,
        }));

        let adapter = self.adapters.get(&protocol)
            .ok_or_else(|| anyhow::anyhow!("No adapter for {:?}", protocol))?;

        // Step 1: Fetch profile (our own identity).
        let _ = adapter.get_own_profile().await?;

        // Step 2: Fetch follow list (contact graph).
        let follows = adapter.get_follows(None).await?;
        let _ = self.bus.sync_progress.send(Arc::new(SyncProgressEvent {
            protocol,
            sync_type: SyncType::ColdStart,
            items_processed: follows.len() as u64,
            items_total: Some(follows.len() as u64),
            estimated_completion: None,
        }));

        // Step 3: Fetch home timeline (latest posts from followed accounts).
        // Batch size: 200 for cold start.
        let posts = adapter.get_home_feed(None, 200).await?;
        let _ = self.bus.sync_progress.send(Arc::new(SyncProgressEvent {
            protocol,
            sync_type: SyncType::ColdStart,
            items_processed: posts.len() as u64,
            items_total: None, // Unknown for feeds.
            estimated_completion: None,
        }));

        // Step 4: Fetch conversations (DMs, group chats).
        let conversations = adapter.get_conversations(50).await?;

        // Update sync state.
        let mut state = self.sync_state.write().await;
        state.insert(protocol, SyncState {
            protocol,
            last_sync_at: Some(Utc::now()),
            last_cursor: adapter.get_sync_cursor().await,
            items_synced: (posts.len() + conversations.len()) as u64,
            status: SyncStatus::Idle,
        });

        let _ = self.bus.sync_completed.send(Arc::new(SyncCompletedEvent {
            protocol,
            sync_type: SyncType::ColdStart,
            items_synced: (posts.len() + conversations.len()) as u64,
            duration: chrono::Duration::zero(), // calculated from start time
        }));

        Ok(())
    }

    /// Incremental sync: fetch only new content since last cursor.
    pub async fn incremental_sync(&self, protocol: Protocol) -> Result<()> {
        let cursor = {
            let state = self.sync_state.read().await;
            state.get(&protocol).and_then(|s| s.last_cursor.clone())
        };

        let adapter = self.adapters.get(&protocol)
            .ok_or_else(|| anyhow::anyhow!("No adapter for {:?}", protocol))?;

        let posts = adapter.get_home_feed(cursor.as_deref(), 50).await?;
        // ... update state, emit events.
        Ok(())
    }

    /// Background sync loop. Runs with adaptive intervals.
    pub async fn background_sync_loop(&self) {
        let mut interval = tokio::time::interval(Duration::from_secs(120)); // 2 min default.
        interval.set_missed_tick_behavior(MissedTickBehavior::Skip);

        loop {
            interval.tick().await;

            // Only sync if online and battery allows.
            if !self.connectivity.is_online() {
                continue;
            }
            if self.connectivity.is_low_battery() && !self.connectivity.is_charging() {
                // Throttle to 10-minute intervals on low battery.
                tokio::time::sleep(Duration::from_secs(600)).await;
                continue;
            }

            // Sync each connected protocol sequentially (to avoid connection storms).
            for protocol in self.get_connected_protocols().await {
                if let Err(e) = self.incremental_sync(protocol).await {
                    tracing::warn!("Incremental sync failed for {:?}: {}", protocol, e);
                }
                // Small gap between protocols.
                tokio::time::sleep(Duration::from_secs(5)).await;
            }
        }
    }
}
```

### 3.2 Per-Protocol Sync Cadences

| Protocol | Cold Start Batch | Incremental Interval | Realtime Mechanism | Battery Saver |
|----------|-----------------|---------------------|-------------------|---------------|
| **Nostr** | 500 events | 60s (WebSocket live) | WebSocket subscription | Poll every 5 min |
| **AT Protocol** | 200 posts | 60s (Jetstream live) | Jetstream WebSocket | Poll every 5 min |
| **ActivityPub** | 200 posts | 120s (polling) | Streaming API (SSE) | Poll every 10 min |
| **Matrix** | 20 events/room | 30s (sync loop) | Sync long-poll | Poll every 5 min |
| **IRC** | Full scrollback | N/A (persistent TCP) | Persistent TCP | Disconnect, rely on bouncer |
| **XMPP** | 50 messages | N/A (persistent) | Persistent WebSocket | Disconnect, rely on MAM |

---

## Part 4: Offline Queue

```rust
// libs/sync/src/offline_queue.rs

/// Queues posts, reactions, and follows that were created while offline.
/// Replays them in order when connectivity is restored.
pub struct OfflineQueue {
    db: SqlitePool,
    bus: Arc<EventBus>,
}

#[derive(Debug, Clone, sqlx::FromRow)]
pub struct QueuedAction {
    pub id: String,
    pub protocol: String,
    pub action_type: String,      // "publish_post", "follow", "reaction", "dm"
    pub payload_json: String,
    pub created_at: DateTime<Utc>,
    pub retry_count: i32,
    pub max_retries: i32,
    pub last_error: Option<String>,
    pub status: QueuedStatus,
}

#[derive(Debug, Clone, sqlx::Type)]
pub enum QueuedStatus {
    Pending,
    Retrying,
    Failed,
    Completed,
}

impl OfflineQueue {
    /// Enqueue an action for later delivery.
    pub async fn enqueue(
        &self,
        protocol: Protocol,
        action_type: &str,
        payload: serde_json::Value,
    ) -> Result<String> {
        let id = uuid::Uuid::new_v4().to_string();
        sqlx::query(
            "INSERT INTO outbox (id, protocol, action_type, payload_json, created_at, retry_count, max_retries, status)
             VALUES (?, ?, ?, ?, ?, 0, 5, 'Pending')"
        )
        .bind(&id)
        .bind(protocol.to_string())
        .bind(action_type)
        .bind(serde_json::to_string(&payload)?)
        .bind(Utc::now())
        .execute(&*self.db.pool)
        .await?;
        Ok(id)
    }

    /// Process all pending actions in order.
    /// Called when connectivity is restored.
    pub async fn flush(&self, adapters: &HashMap<Protocol, Box<dyn ProtocolAdapter>>) -> Result<FlushResult> {
        let pending = sqlx::query_as::<_, QueuedAction>(
            "SELECT * FROM outbox WHERE status IN ('Pending', 'Retrying')
             ORDER BY created_at ASC LIMIT 50"
        )
        .fetch_all(&*self.db.pool)
        .await?;

        let mut succeeded = 0;
        let mut failed = 0;

        for action in &pending {
            let adapter = adapters.get(&action.protocol.parse()?);
            if adapter.is_none() {
                continue;
            }
            let adapter = adapter.unwrap();

            // Mark as retrying.
            sqlx::query("UPDATE outbox SET status = 'Retrying', retry_count = retry_count + 1 WHERE id = ?")
                .bind(&action.id)
                .execute(&*self.db.pool)
                .await?;

            let result = match action.action_type.as_str() {
                "publish_post" => {
                    let payload: PublishPostPayload = serde_json::from_str(&action.payload_json)?;
                    adapter.publish_post(&payload.content, &payload.tags).await.map(|_| ())
                }
                "follow" => {
                    let payload: FollowPayload = serde_json::from_str(&action.payload_json)?;
                    adapter.follow(&payload.target).await
                }
                "reaction" => {
                    let payload: ReactionPayload = serde_json::from_str(&action.payload_json)?;
                    adapter.react(&payload.post_id, &payload.reaction).await
                }
                "dm" => {
                    let payload: DmPayload = serde_json::from_str(&action.payload_json)?;
                    adapter.send_message(&payload.to, &payload.content).await
                }
                _ => Err(anyhow::anyhow!("Unknown action type: {}", action.action_type)),
            };

            match result {
                Ok(_) => {
                    sqlx::query("UPDATE outbox SET status = 'Completed' WHERE id = ?")
                        .bind(&action.id)
                        .execute(&*self.db.pool)
                        .await?;
                    succeeded += 1;
                }
                Err(e) => {
                    let new_status = if action.retry_count >= action.max_retries {
                        "Failed"
                    } else {
                        "Pending" // Will retry on next flush.
                    };
                    sqlx::query("UPDATE outbox SET status = ?, last_error = ? WHERE id = ?")
                        .bind(new_status)
                        .bind(e.to_string())
                        .bind(&action.id)
                        .execute(&*self.db.pool)
                        .await?;
                    failed += 1;
                }
            }
        }

        Ok(FlushResult { succeeded, failed })
    }
}
```

---

## Part 5: Crate Structure

```
libs/
├── event-bus/                    # Canonical events + EventBus
│   ├── Cargo.toml
│   └── src/
│       ├── lib.rs                # EventBus struct
│       ├── events.rs             # All event types
│       ├── subscriber.rs         # Subscriber helpers
│       └── canonical.rs          # UnifiedPost, UnifiedProfile, etc.
│
├── canonical/                    # Shared data types (no deps on event-bus)
│   ├── Cargo.toml
│   └── src/
│       ├── lib.rs
│       ├── post.rs
│       ├── profile.rs
│       ├── message.rs
│       ├── reaction.rs
│       └── protocol.rs           # Protocol enum
│
├── storage/                      # SQLite persistence layer
│   ├── Cargo.toml
│   └── src/
│       ├── lib.rs
│       ├── pool.rs               # SqlitePool setup + migrations
│       ├── event_persister.rs    # Subscribes to EventBus, persists
│       ├── queries/              # Named query functions
│       │   ├── timeline.rs
│       │   ├── profiles.rs
│       │   ├── messages.rs
│       │   └── search.rs
│       └── migrations/           # SQL migration files
│           ├── 001_init.sql
│           ├── 002_fts5.sql
│           └── ...
│
├── sync/                         # Sync orchestration
│   ├── Cargo.toml
│   └── src/
│       ├── lib.rs
│       ├── orchestrator.rs
│       ├── offline_queue.rs
│       └── connectivity.rs       # Network + battery monitoring
│
├── identity/                     # Key management (see identidade-keychain.md)
│   ├── Cargo.toml
│   └── src/
│       ├── lib.rs
│       ├── derivation.rs         # BIP-39 / SLIP-0010 key derivation
│       ├── keychain.rs           # Platform keychain integration
│       ├── backup.rs             # Backup/recovery flows
│       └── linking.rs            # Cross-protocol identity linking
│
├── protocols/
│   ├── nostr/
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs            # NostrAdapter struct
│   │       ├── relay_manager.rs  # RelayConnectionManager
│   │       ├── deduplicator.rs   # EventDeduplicator
│   │       ├── event_mapper.rs   # NostrEvent → CanonicalEvent
│   │       ├── subscriptions.rs  # Feed/notification subscription logic
│   │       ├── publish.rs        # Publishing logic
│   │       └── nip/              # Per-NIP implementation modules
│   │           ├── nip01.rs
│   │           ├── nip02.rs
│   │           ├── nip10.rs
│   │           ├── nip44.rs
│   │           └── ...
│   │
│   ├── atproto/                  # (Future iteration)
│   ├── activitypub/              # (Future iteration)
│   ├── matrix/                   # (Future iteration)
│   ├── irc/                      # (Future iteration)
│   └── xmpp/                     # (Future iteration)
│
└── ui/                           # Platform-specific UI (Flutter/SwiftUI)
    └── (not part of this spec)
```

---

## Part 6: FFI Boundary

The Rust core communicates with platform UI via UniFFI:

```rust
// libs/uniffi-bindings/src/lib.rs

#[uniffi::export]
pub async fn initialize_core(
    db_path: String,
    bootstrap_relays: Vec<String>,
) -> Result<Arc<CoreHandle>> { ... }

#[uniffi::export]
pub async fn connect_protocol(
    handle: Arc<CoreHandle>,
    protocol: Protocol,
    credentials: String, // JSON-encoded credentials
) -> Result<()> { ... }

#[uniffi::export]
pub async fn get_timeline(
    handle: Arc<CoreHandle>,
    protocol_filter: Option<Vec<Protocol>>,
    cursor: Option<String>,
    limit: u32,
) -> Result<Vec<UnifiedPost>> { ... }

// Events flow FROM Rust TO platform via callback interfaces:
#[uniffi::export(callback_interface)]
pub trait EventCallback: Send + Sync {
    fn on_new_post(&self, post: UnifiedPost);
    fn on_connection_changed(&self, protocol: Protocol, connected: bool);
    fn on_sync_progress(&self, protocol: Protocol, progress: f64);
}

#[uniffi::export]
pub fn subscribe_to_events(handle: Arc<CoreHandle>, callback: Box<dyn EventCallback>) -> Result<()> { ... }
```

---

*Documento gerado na Iteração 5 do /loop. Cobre o EventBus, Nostr Adapter, Sync Engine, Offline Queue, estrutura de crates, e FFI boundary.*
