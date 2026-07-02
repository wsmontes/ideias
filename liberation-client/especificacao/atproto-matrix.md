# Liberation Client — AT Protocol Adapter & Matrix Chat Module

> Especificação dos adapters Tier 1 (AT Protocol) e Tier 3 (Matrix Chat).
> **Iteração 6** — 2026-07-01

---

## Part 1: AT Protocol Adapter

### 1.1 Architecture

```
┌─────────────────────────────────────────┐
│          ATProtocolAdapter               │
│                                          │
│  ┌──────────────────────────────────┐   │
│  │  XRPC Client (shrike::xrpc)      │   │
│  │  - app.bsky.feed.getTimeline     │   │
│  │  - com.atproto.repo.createRecord │   │
│  │  - com.atproto.identity.*        │   │
│  └──────────────┬───────────────────┘   │
│                 │                        │
│  ┌──────────────▼───────────────────┐   │
│  │  Jetstream Consumer              │   │
│  │  (shrike::streaming)             │   │
│  │  - Single WebSocket              │   │
│  │  - wantedDids=[user's DID]       │   │
│  │  - compress=true                 │   │
│  │  - Auto-reconnect + backoff      │   │
│  └──────────────┬───────────────────┘   │
│                 │                        │
│  ┌──────────────▼───────────────────┐   │
│  │  ATP Event Mapper                │   │
│  │  (Record → Canonical Event)      │   │
│  └──────────────┬───────────────────┘   │
│                 │                        │
│  ┌──────────────▼───────────────────┐   │
│  │  DID Resolver + PLC Cache        │   │
│  │  (did:plc → PDS endpoint)        │   │
│  └──────────────┬───────────────────┘   │
│                 │                        │
│          EventBus (emit)                 │
└─────────────────────────────────────────┘
```

### 1.2 Session & Auth

```rust
// libs/protocols/atproto/src/session.rs

pub struct ATProtocolSession {
    /// DID of the authenticated user.
    pub did: String,
    /// Handle (e.g., user.bsky.social).
    pub handle: String,
    /// Access JWT for XRPC requests.
    pub access_jwt: String,
    /// Refresh JWT for renewing access.
    pub refresh_jwt: String,
    /// PDS endpoint for writes (from DID document).
    pub pds_endpoint: String,
    /// Relay endpoint for firehose (from well-known or config).
    pub relay_endpoint: String,
    /// Session created at.
    pub created_at: DateTime<Utc>,
    /// Access token expires at.
    pub expires_at: DateTime<Utc>,
}

impl ATProtocolAdapter {
    /// Create a session from Bluesky credentials.
    /// Uses app password, NEVER the account password.
    pub async fn login(handle: &str, app_password: &str) -> Result<ATProtocolSession> {
        // Step 1: Resolve handle → DID.
        let did = resolve_handle(handle).await?;

        // Step 2: Create session.
        let session = xrpc_client
            .call(com_atproto::server::create_session::Request {
                identifier: handle.to_string(),
                password: app_password.to_string(),
            })
            .await?;

        // Step 3: Resolve DID → PDS endpoint.
        let did_doc = resolve_did(&did).await?;
        let pds_endpoint = did_doc.get_pds_endpoint()
            .ok_or_else(|| anyhow::anyhow!("No PDS endpoint in DID document"))?;

        // Step 4: Get relay endpoint (default Jetstream).
        let relay_endpoint = "wss://jetstream2.us-east.bsky.network/subscribe".to_string();

        Ok(ATProtocolSession {
            did,
            handle: session.handle,
            access_jwt: session.access_jwt,
            refresh_jwt: session.refresh_jwt,
            pds_endpoint,
            relay_endpoint,
            created_at: Utc::now(),
            expires_at: Utc::now() + chrono::Duration::hours(2),
        })
    }

    /// Auto-refresh JWT before it expires.
    pub async fn ensure_fresh_token(&mut self) -> Result<()> {
        if Utc::now() > self.expires_at - chrono::Duration::minutes(5) {
            let new_session = xrpc_client
                .call(com_atproto::server::refresh_session::Request {
                    refresh_jwt: self.refresh_jwt.clone(),
                })
                .await?;
            self.access_jwt = new_session.access_jwt;
            self.refresh_jwt = new_session.refresh_jwt;
            self.expires_at = Utc::now() + chrono::Duration::hours(2);
        }
        Ok(())
    }
}
```

### 1.3 Timeline & Jetstream

```rust
// libs/protocols/atproto/src/timeline.rs

impl ATProtocolAdapter {
    /// Cold start: fetch initial timeline via XRPC.
    pub async fn cold_start_timeline(&self, limit: u32) -> Result<Vec<UnifiedPost>> {
        let response = self.xrpc
            .call(app_bsky::feed::get_timeline::Request {
                algorithm: None, // Default reverse-chronological.
                limit: Some(limit.min(100)),
                cursor: None,
            })
            .await?;

        let posts: Vec<UnifiedPost> = response.feed
            .into_iter()
            .filter_map(|item| self.map_feed_item_to_post(item).ok())
            .collect();

        Ok(posts)
    }

    /// Incremental: fetch since cursor.
    pub async fn incremental_timeline(&self, cursor: &str, limit: u32) -> Result<(Vec<UnifiedPost>, Option<String>)> {
        let response = self.xrpc
            .call(app_bsky::feed::get_timeline::Request {
                algorithm: None,
                limit: Some(limit.min(100)),
                cursor: Some(cursor.to_string()),
            })
            .await?;

        let posts: Vec<UnifiedPost> = response.feed
            .into_iter()
            .filter_map(|item| self.map_feed_item_to_post(item).ok())
            .collect();

        Ok((posts, response.cursor))
    }

    /// Jetstream: real-time event stream.
    /// Connects to wss://jetstream2.us-east.bsky.network/subscribe
    /// with wantedDids=[user's DID], compress=true.
    pub async fn subscribe_jetstream(&self, bus: Arc<EventBus>) -> Result<tokio::task::JoinHandle<()>> {
        use shrike::streaming::{JetstreamClient, JetstreamConfig};

        let config = JetstreamConfig {
            wanted_dids: vec![self.session.did.clone()],
            compress: true,
            ..Default::default()
        };

        let mut client = JetstreamClient::connect(&self.session.relay_endpoint, config).await?;

        let deduplicator = Arc::new(EventDeduplicator::new());
        let mapper = Arc::new(ATPRecordMapper::new());

        let handle = tokio::spawn(async move {
            while let Some(batch) = client.next_batch().await {
                for commit in batch.commits {
                    // Dedup by commit CID.
                    if !deduplicator.is_new(&commit.cid).await {
                        continue;
                    }

                    match commit.operation {
                        JetstreamOp::Create { record } => {
                            if let Ok(canonical) = mapper.map_create_to_events(&commit.did, &record) {
                                for event in canonical {
                                    event.emit(&bus);
                                }
                            }
                        }
                        JetstreamOp::Delete { .. } => {
                            // Emit deletion event.
                            let _ = bus.deleted_post.send(Arc::new(DeletedPostEvent {
                                post_id: format!("at://{}/{}", commit.did, commit.rkey),
                                protocol: Protocol::ATProtocol,
                                reason: None,
                            }));
                        }
                    }
                }
            }
        });

        Ok(handle)
    }
}
```

### 1.4 Record Mapping (ATP → Canonical)

```rust
// libs/protocols/atproto/src/record_mapper.rs

impl ATPRecordMapper {
    /// Map app.bsky.feed.post → UnifiedPost.
    pub fn map_post_record(
        did: &str,
        rkey: &str,
        record: &app_bsky::feed::post::Record,
    ) -> Result<UnifiedPost> {
        let post_id = format!("at://{}/app.bsky.feed.post/{}", did, rkey);

        // Extract threading.
        let reply_to = record.reply.as_ref().map(|r| r.parent.uri.clone());
        let root_post = record.reply.as_ref().map(|r| r.root.uri.clone());

        // Extract facets → tags, mentions.
        let mut tags = Vec::new();
        let mut mentions = Vec::new();
        if let Some(facets) = &record.facets {
            for facet in facets {
                for feature in &facet.features {
                    match feature {
                        app_bsky::richtext::facet::Feature::Tag(tag) => {
                            tags.push(tag.tag.clone());
                        }
                        app_bsky::richtext::facet::Feature::Mention(mention) => {
                            mentions.push(mention.did.clone());
                        }
                        _ => {}
                    }
                }
            }
        }

        // Extract media from embed.
        let media = match &record.embed {
            Some(app_bsky::embed::Embed::Images(images)) => {
                images.images.iter().map(|img| UnifiedAttachment {
                    id: img.image.r#ref.to_string(),
                    kind: AttachmentType::Image,
                    url: format!("https://cdn.bsky.app/img/feed_fullsize/{}", img.image.r#ref),
                    mime_type: Some(img.image.mime_type.clone()),
                    alt_text: img.alt.clone(),
                    width: img.aspect_ratio.as_ref().map(|a| a.width as u32),
                    height: img.aspect_ratio.as_ref().map(|a| a.height as u32),
                    size: img.image.size,
                    blurhash: None,
                    local_cache_path: None,
                }).collect()
            }
            Some(app_bsky::embed::Embed::External(ext)) => {
                vec![UnifiedAttachment {
                    id: ext.external.uri.clone(),
                    kind: AttachmentType::Link,
                    url: ext.external.uri.clone(),
                    mime_type: None,
                    alt_text: Some(ext.external.description.clone()),
                    width: ext.external.thumb.as_ref().and_then(|t| t.aspect_ratio.as_ref()).map(|a| a.width as u32),
                    height: ext.external.thumb.as_ref().and_then(|t| t.aspect_ratio.as_ref()).map(|a| a.height as u32),
                    size: ext.external.thumb.as_ref().map(|t| t.size),
                    blurhash: ext.external.thumb.as_ref().and_then(|t| t.blurhash.clone()),
                    local_cache_path: None,
                }]
            }
            _ => vec![],
        };

        Ok(UnifiedPost {
            id: post_id.clone(),
            protocol: Protocol::ATProtocol,
            protocol_id: post_id,
            author: UnifiedAuthor {
                id: did.to_string(),
                handle: None, // Resolved later.
                display_name: None,
                avatar_url: None,
                protocol: Protocol::ATProtocol,
            },
            content: record.text.clone(),
            rich_content: record.facets.as_ref().map(|f| {
                // Convert facets to RichText with byte offsets.
                RichText { text: record.text.clone(), facets: f.clone() }
            }),
            created_at: record.created_at,
            reply_to,
            root_post,
            media,
            mentions,
            tags,
            visibility: Visibility::Public,
            language: record.langs.as_ref().and_then(|l| l.first().cloned()),
            source: Some(serde_json::to_value(record)?),
        })
    }

    /// Map app.bsky.feed.like → UnifiedReaction.
    pub fn map_like_record(did: &str, record: &app_bsky::feed::like::Record) -> Result<UnifiedReaction> {
        Ok(UnifiedReaction {
            id: format!("like/{}/{}", did, record.subject.uri),
            post_id: record.subject.uri.clone(),
            author_id: did.to_string(),
            reaction_type: ReactionType::Like,
            emoji: None,
            created_at: record.created_at,
            protocol: Protocol::ATProtocol,
        })
    }

    /// Map app.bsky.feed.repost → UnifiedReaction.
    pub fn map_repost_record(did: &str, record: &app_bsky::feed::repost::Record) -> Result<UnifiedReaction> {
        Ok(UnifiedReaction {
            id: format!("repost/{}/{}", did, record.subject.uri),
            post_id: record.subject.uri.clone(),
            author_id: did.to_string(),
            reaction_type: ReactionType::Repost,
            emoji: None,
            created_at: record.created_at,
            protocol: Protocol::ATProtocol,
        })
    }
}
```

### 1.5 Publishing

```rust
// libs/protocols/atproto/src/publish.rs

impl ATProtocolAdapter {
    /// Publish a post to the user's PDS.
    pub async fn publish_post(&self, content: &PostContent) -> Result<UnifiedPost> {
        // Upload any media blobs first.
        let mut embed: Option<app_bsky::embed::Embed> = None;
        if !content.media.is_empty() {
            embed = Some(self.upload_media_embed(&content.media).await?);
        }

        // Build facets (rich text) from content.
        let facets = Self::build_facets(&content.text);

        // Create the record.
        let record = app_bsky::feed::post::Record {
            text: content.text.clone(),
            facets,
            embed,
            reply: content.reply_to.as_ref().map(|r| app_bsky::feed::post::Reply {
                parent: r.parent.clone().into(),
                root: r.root.clone().into(),
            }),
            created_at: Utc::now(),
            langs: content.language.clone().map(|l| vec![l]),
            ..Default::default()
        };

        // Write to PDS.
        let response = self.xrpc
            .call(com_atproto::repo::create_record::Request {
                repo: self.session.did.clone(),
                collection: "app.bsky.feed.post".to_string(),
                record: serde_json::to_value(&record)?,
                ..Default::default()
            })
            .await?;

        // Map the created record to UnifiedPost.
        let post_id = format!("at://{}/app.bsky.feed.post/{}", self.session.did, response.rkey);
        let post = self.map_post_to_unified(&post_id, &record)?;

        // Emit locally.
        let _ = self.bus.new_post.send(Arc::new(NewPostEvent {
            post: post.clone(),
            protocol: Protocol::ATProtocol,
            is_backfill: false,
            received_at: Utc::now(),
        }));

        Ok(post)
    }

    /// Build AT Protocol facets from text.
    /// Detects: @mentions, #hashtags, https://links.
    fn build_facets(text: &str) -> Option<Vec<app_bsky::richtext::facet::Facet>> {
        let mut facets = Vec::new();

        // Detect mentions: @handle.bsky.social
        // Detect hashtags: #topic
        // Detect links: https://...

        // Use regex-based detection. Each facet has byteStart + byteEnd + feature.
        // This is critical — AT Protocol facets are BYTE OFFSET based.

        // ... (regex scanning logic omitted for brevity)

        if facets.is_empty() { None } else { Some(facets) }
    }

    /// Upload images and build embed.
    async fn upload_media_embed(&self, media: &[UnifiedAttachment]) -> Result<app_bsky::embed::Embed> {
        let mut images = Vec::new();
        for item in media {
            // Upload blob to PDS.
            let blob = self.xrpc
                .call(com_atproto::repo::upload_blob::Request {
                    data: item.data.clone(),
                    ..Default::default()
                })
                .await?;

            images.push(app_bsky::embed::images::Image {
                image: blob.blob,
                alt: item.alt_text.clone().unwrap_or_default(),
                aspect_ratio: item.width.and_then(|w| item.height.map(|h| app_bsky::embed::images::AspectRatio {
                    width: w as i32,
                    height: h as i32,
                })),
            });
        }
        Ok(app_bsky::embed::Embed::Images(app_bsky::embed::images::Main { images }))
    }
}
```

### 1.6 DID Resolution

```rust
// libs/protocols/atproto/src/did_resolver.rs

/// Resolve DID to DID document. Pluggable backends.
pub struct DIDResolver {
    /// Cache of resolved DIDs (1-hour TTL).
    cache: LruCache<String, (DIDDocument, DateTime<Utc>)>,
    /// PLC directory endpoint.
    plc_directory: String,
}

impl DIDResolver {
    pub fn new() -> Self {
        Self {
            cache: LruCache::new(NonZeroUsize::new(10_000).unwrap()),
            plc_directory: "https://plc.directory".to_string(),
        }
    }

    /// Resolve a DID to its DID document.
    pub async fn resolve(&mut self, did: &str) -> Result<DIDDocument> {
        // Check cache first.
        if let Some((doc, cached_at)) = self.cache.get(did) {
            if Utc::now() - *cached_at < chrono::Duration::hours(1) {
                return Ok(doc.clone());
            }
        }

        let doc = match did.split(':').next() {
            Some("plc") => self.resolve_plc(did).await?,
            Some("web") => self.resolve_web(did).await?,
            _ => return Err(anyhow::anyhow!("Unsupported DID method: {}", did)),
        };

        self.cache.put(did.to_string(), (doc.clone(), Utc::now()));
        Ok(doc)
    }

    async fn resolve_plc(&self, did: &str) -> Result<DIDDocument> {
        let url = format!("{}/{}", self.plc_directory, did);
        let resp = reqwest::get(&url).await?;
        Ok(resp.json().await?)
    }

    async fn resolve_web(&self, did: &str) -> Result<DIDDocument> {
        let domain = did.strip_prefix("did:web:").unwrap();
        let url = format!("https://{}/.well-known/did.json", domain);
        let resp = reqwest::get(&url).await?;
        Ok(resp.json().await?)
    }
}
```

### 1.7 AT Protocol Sync Strategy

| Phase | Method | Batch | Interval |
|-------|--------|-------|----------|
| Cold start | XRPC `getTimeline` | 100 posts | Once |
| Incremental | XRPC `getTimeline` with cursor | 50 posts | 5 min (fallback) |
| Realtime | Jetstream WebSocket | Per commit | Continuous |
| Profile refresh | XRPC `getProfile` | Per profile | 24h + jitter |
| Media backfill | CDN fetch on scroll | Per image | On demand |

---

## Part 2: Matrix Chat Module

### 2.1 Architecture

```
┌─────────────────────────────────────────┐
│         MatrixChatModule                 │
│                                          │
│  ┌──────────────────────────────────┐   │
│  │  matrix-rust-sdk                 │   │
│  │  ┌──────────┐  ┌──────────────┐  │   │
│  │  │ Client   │  │ RoomList     │  │   │
│  │  │ (sync)   │  │ Service      │  │   │
│  │  └────┬─────┘  └──────┬───────┘  │   │
│  │       │               │           │   │
│  │  ┌────▼───────────────▼───────┐  │   │
│  │  │ matrix-sdk-crypto          │  │   │
│  │  │ (Olm/Megolm + vodozemac)   │  │   │
│  │  └────────────────────────────┘  │   │
│  └──────────────┬───────────────────┘   │
│                 │                        │
│  ┌──────────────▼───────────────────┐   │
│  │  Matrix Event Mapper             │   │
│  │  (TimelineEvent → Canonical)     │   │
│  └──────────────┬───────────────────┘   │
│                 │                        │
│  ┌──────────────▼───────────────────┐   │
│  │  Room Classifier                  │   │
│  │  (DM vs Group vs Channel)         │   │
│  └──────────────┬───────────────────┘   │
│                 │                        │
│          EventBus (emit)                 │
└─────────────────────────────────────────┘
```

### 2.2 Session & Sync

```rust
// libs/protocols/matrix/src/chat_module.rs

use matrix_sdk::{
    Client, ClientBuilder,
    config::SyncSettings,
    room::Room,
    ruma::events::room::message::RoomMessageEventContent,
};

pub struct MatrixChatModule {
    client: Client,
    bus: Arc<EventBus>,
    /// Background sync task.
    sync_handle: Option<tokio::task::JoinHandle<()>>,
}

impl MatrixChatModule {
    /// Login to a Matrix homeserver and start syncing.
    pub async fn connect(homeserver_url: &str, username: &str, password: &str) -> Result<Self> {
        let client = ClientBuilder::new()
            .homeserver_url(homeserver_url)
            .build()
            .await?;

        // Login.
        client.matrix_auth()
            .login_username(username, password)
            .initial_device_display_name("Liberation Client")
            .await?;

        // Restore or bootstrap cross-signing.
        if let Err(_) = client.encryption().wait_for_e2ee_initialization_tasks().await {
            tracing::info!("Bootstrapping cross-signing for new session");
            client.encryption().bootstrap_cross_signing().await?;
        }

        Ok(Self { client, bus: /* ... */, sync_handle: None })
    }

    /// Start the sync loop. Runs forever in background.
    pub async fn start_sync(&mut self, bus: Arc<EventBus>) -> Result<()> {
        let client = self.client.clone();

        // Register event handler BEFORE starting sync.
        client.add_event_handler(move |event: matrix_sdk::ruma::events::room::message::OriginalSyncRoomMessageEvent,
                                         room: Room,
                                         client: Client| {
            let bus = bus.clone();
            async move {
                let classification = classify_room(&room, &client).await;
                let canonical = map_matrix_message_to_canonical(event, room, classification);
                // Route to Message (DM) or Post (channel) based on classification.
                match classification {
                    RoomKind::DirectMessage => {
                        let _ = bus.new_message.send(Arc::new(NewMessageEvent {
                            message: canonical.into_message(),
                            protocol: Protocol::Matrix,
                        }));
                    }
                    RoomKind::GroupChat | RoomKind::Channel => {
                        let _ = bus.new_post.send(Arc::new(NewPostEvent {
                            post: canonical.into_post(),
                            protocol: Protocol::Matrix,
                            is_backfill: false,
                            received_at: Utc::now(),
                        }));
                    }
                }
            }
        });

        // Start sync loop with filter.
        let sync_settings = SyncSettings::default()
            .timeout(Duration::from_secs(30));

        let sync_handle = tokio::spawn(async move {
            loop {
                match client.sync_once(sync_settings.clone()).await {
                    Ok(response) => {
                        tracing::debug!(
                            "Sync OK. Next batch: {:?}, rooms: {}",
                            response.next_batch,
                            response.rooms.join.len() + response.rooms.invite.len()
                        );
                    }
                    Err(e) => {
                        tracing::error!("Sync error: {}. Retrying in 5s...", e);
                        tokio::time::sleep(Duration::from_secs(5)).await;
                    }
                }
            }
        });

        self.sync_handle = Some(sync_handle);
        Ok(())
    }
}

/// Classify a Matrix room as DM, Group Chat, or Channel.
/// DM: 2 members + m.direct flag.
/// Group: 3+ members, no public join rules.
/// Channel: public join rules, typically larger.
async fn classify_room(room: &Room, client: &Client) -> RoomKind {
    if room.is_direct().await.unwrap_or(false) {
        return RoomKind::DirectMessage;
    }
    if let Some(joined) = room.joined_members_count() {
        if joined <= 2 {
            return RoomKind::DirectMessage;
        }
    }
    // Check join rules.
    if let Room::Joined(joined) = room {
        if let Ok(rules) = joined.join_rules().await {
            if rules == matrix_sdk::ruma::events::room::join_rules::JoinRule::Public {
                return RoomKind::Channel;
            }
        }
    }
    RoomKind::GroupChat
}
```

### 2.3 E2EE Integration

```rust
// libs/protocols/matrix/src/e2ee.rs

impl MatrixChatModule {
    /// Verify a device via emoji comparison (Signal-style SAS).
    pub async fn verify_device(&self, user_id: &str, device_id: &str) -> Result<VerificationResult> {
        let user = self.client.get_user(user_id).await?;
        let device = user.get_device(device_id).await?;

        // Start verification.
        let verification = device.request_verification().await?;

        // Generate emoji/SAS for user comparison.
        let emoji = verification.emoji().unwrap_or_default();
        // Display emoji to user. They compare with the other party.
        // If they match, confirm.

        // User confirms match → verification complete.
        verification.confirm().await?;

        Ok(VerificationResult::Verified)
    }

    /// Check E2EE status for a room.
    pub async fn room_encryption_status(&self, room_id: &str) -> EncryptionStatus {
        let room = self.client.get_room(room_id);
        if room.is_none() {
            return EncryptionStatus::None;
        }
        let room = room.unwrap();

        if !room.is_encrypted().await.unwrap_or(false) {
            return EncryptionStatus::None;
        }

        // Check if all members' devices are verified.
        let members = room.joined_members().await.unwrap_or_default();
        let mut all_verified = true;
        for member in members {
            let user = self.client.get_user(&member.user_id).await;
            if let Ok(user) = user {
                let devices = user.devices().await.unwrap_or_default();
                for device in devices {
                    if !device.is_verified() {
                        all_verified = false;
                        break;
                    }
                }
            }
        }

        if all_verified {
            EncryptionStatus::Verified
        } else {
            EncryptionStatus::EncryptedUnverified
        }
    }

    /// Matrix-specific: restore E2EE sessions from SSSS backup.
    pub async fn restore_from_backup(&self, recovery_key: &str) -> Result<()> {
        self.client.encryption()
            .enable_backup_v1(recovery_key)
            .await?;
        Ok(())
    }
}

#[derive(Debug, Clone)]
pub enum EncryptionStatus {
    Verified,          // 🟢 All devices verified
    EncryptedUnverified, // 🟡 E2EE on, unverified devices exist
    None,              // ⚪ No E2EE (public room, IRC bridge)
    Ephemeral,         // 🔴 Not applicable (IRC)
}
```

### 2.4 Room → Canonical Mapping

```rust
// libs/protocols/matrix/src/event_mapper.rs

/// Map Matrix room message → canonical event.
fn map_matrix_message_to_canonical(
    event: OriginalSyncRoomMessageEvent,
    room: Room,
    classification: RoomKind,
) -> CanonicalResult {
    let event_id = event.event_id.to_string();
    let room_id = room.room_id().to_string();
    let sender = event.sender.to_string();

    match classification {
        RoomKind::DirectMessage => {
            CanonicalResult::Message(UnifiedMessage {
                id: event_id,
                conversation_id: room_id,
                protocol: Protocol::Matrix,
                sender_id: sender,
                body: event.content.body().to_string(),
                encrypted_body: None, // SDK handles decryption.
                sent_at: event.origin_server_ts,
                received_at: Utc::now(),
                edit_history: Vec::new(),
                encryption_status: EncryptionStatus::EncryptedUnverified, // Check actual status.
            })
        }
        RoomKind::Channel | RoomKind::GroupChat => {
            CanonicalResult::Post(UnifiedPost {
                id: event_id.clone(),
                protocol: Protocol::Matrix,
                protocol_id: event_id,
                author: UnifiedAuthor {
                    id: sender,
                    handle: None,
                    display_name: None,
                    avatar_url: None,
                    protocol: Protocol::Matrix,
                },
                content: event.content.body().to_string(),
                rich_content: event.content.formatted().map(|f| f.body.clone()),
                created_at: event.origin_server_ts,
                reply_to: event.content.relates_to.as_ref()
                    .and_then(|r| r.in_reply_to().map(|e| e.event_id.to_string())),
                root_post: None,
                media: Vec::new(), // Extract from msgtype.
                mentions: Vec::new(),
                tags: Vec::new(),
                visibility: Visibility::Public,
                language: None,
                source: None,
            })
        }
    }
}
```

### 2.5 Matrix Sync Strategy

| Phase | Method | Batch | Interval |
|-------|--------|-------|----------|
| Cold start | `/sync` with filter, lazy_load_members | 20 events/room | Once |
| Incremental | `/sync` with `since` token | Per sync response | Continuous (30s timeout) |
| Realtime | `add_event_handler` callback | Per event | Instant |
| E2EE bootstrap | Cross-signing bootstrap | Per session | Once |
| Key backup | SSSS restore | Per session | Once |
| Room discovery | `get_joined_rooms` | All rooms | On demand |

### 2.6 Discussion: Why Matrix is a CHAT MODULE, Not a Social Feed

Matrix is engineered for messaging — rooms, state resolution, E2EE. It is NOT engineered for:
- Public microblogging timelines
- Following/unfollowing (roster ≠ follow graph)
- Reposts, quote posts, trending topics
- Chronological public feeds

**Our approach**: Matrix rooms classified as:
- **DM (2 members, m.direct)** → Routed to `UnifiedMessage` → appears in Messages tab
- **Group Chat (3+ members, invite-only)** → Routed to `UnifiedMessage` → appears in Messages tab
- **Channel (public join, large)** → Routed to `UnifiedPost` → appears in Timeline as "group conversation" posts

This classification is heuristic-based and can be manually overridden by the user.

**IRC and XMPP follow the same pattern**: routed to Messages tab, not Timeline. They are chat protocols, not social feed protocols.

---

## Part 3: Dependency Map

### Cargo.toml entries

```toml
# libs/protocols/atproto/Cargo.toml
[dependencies]
shrike = { version = "0.1", features = ["xrpc", "streaming", "api", "identity"] }
# OR atrium for XRPC with jetstream-oxide for Jetstream:
# atrium-api = "0.25"
# atrium-xrpc-client = "0.5"
# jetstream-oxide = "0.1"
tokio-tungstenite = "0.21"
tokio = { version = "1", features = ["full"] }
serde_json = "1"
chrono = "0.4"
lru = "0.12"
```

```toml
# libs/protocols/matrix/Cargo.toml
[dependencies]
matrix-sdk = { version = "0.9", features = ["e2e-encryption", "sqlite"] }
matrix-sdk-ui = "0.9"  # Timeline, RoomList
tokio = { version = "1", features = ["full"] }
```

### Crates chosen

| Protocol | Crate | Rationale |
|----------|-------|-----------|
| AT Protocol | `shrike` | Most complete (May 2026). Jetstream + firehose in one crate. XRPC server optional. |
| AT Protocol (alt) | `atrium-api` + `jetstream-oxide` | More mature XRPC client. Separate Jetstream. |
| Matrix | `matrix-rust-sdk` | Official. Element X uses it. UniFFI for FFI. E2EE via vodozemac. |

---

*Documento gerado na Iteração 6 do /loop.*
