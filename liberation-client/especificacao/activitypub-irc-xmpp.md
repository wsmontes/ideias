# Liberation Client — ActivityPub, IRC & XMPP Adapters

> Completando os 6 protocolos: Tier 2 (ActivityPub) + Tier 4 (IRC/XMPP).
> **Iteração 7** — 2026-07-01

---

## Part 1: ActivityPub Adapter (Mastodon API)

### 1.1 Why Mastodon API, Not Native ActivityPub C2S

- **Mastodon API is the de facto standard.** Mastodon, Pleroma, Akkoma, GoToSocial, Friendica, Firefish, Pixelfed — all implement it.
- ActivityPub C2S (Client-to-Server) requires a server to handle outbox/inbox. A client-only implementation fights the protocol design.
- `megalodon` crate already provides a unified Rust interface across platforms.
- **One adapter covers 70%+ of the Fediverse.**

### 1.2 Architecture

```
┌─────────────────────────────────────────┐
│       ActivityPubAdapter                 │
│                                          │
│  ┌──────────────────────────────────┐   │
│  │  Megalodon Client (HTTP)         │   │
│  │  - GET /api/v1/timelines/home    │   │
│  │  - GET /api/v1/notifications     │   │
│  │  - POST /api/v1/statuses         │   │
│  └──────────────┬───────────────────┘   │
│                 │                        │
│  ┌──────────────▼───────────────────┐   │
│  │  Streaming WebSocket             │   │
│  │  - wss://instance/api/v1/stream  │   │
│  │  - Subscribe: user, public, tag  │   │
│  └──────────────┬───────────────────┘   │
│                 │                        │
│  ┌──────────────▼───────────────────┐   │
│  │  Content Sanitizer (CRITICAL)    │   │
│  │  - HTML → plain text             │   │
│  │  - Allowlist: <p><br><a><img>    │   │
│  │  - Strip: <script><iframe><style>│   │
│  └──────────────┬───────────────────┘   │
│                 │                        │
│  ┌──────────────▼───────────────────┐   │
│  │  Mastodon → Canonical Mapper     │   │
│  └──────────────┬───────────────────┘   │
│                 │                        │
│          EventBus (emit)                 │
└─────────────────────────────────────────┘
```

### 1.3 Session & Multi-Account

```rust
// libs/protocols/activitypub/src/session.rs

pub struct MastodonSession {
    /// Instance URL (e.g., https://mastodon.social).
    pub instance_url: String,
    /// OAuth2 access token.
    pub access_token: String,
    /// Authenticated account info.
    pub account: MastodonAccount,
    /// The megalodon client.
    client: megalodon::Megalodon,
}

pub struct ActivityPubAdapter {
    /// Support multiple Mastodon/Pleroma/etc accounts simultaneously.
    sessions: Vec<MastodonSession>,
    /// Default session for posting.
    default_session_index: usize,
    bus: Arc<EventBus>,
}

impl ActivityPubAdapter {
    /// Register OAuth app and obtain token.
    /// Uses PKCE flow for security.
    pub async fn login(instance_url: &str) -> Result<MastodonSession> {
        // Step 1: Register app.
        let client = megalodon::generator(
            megalodon::SNS::Mastodon,
            instance_url.to_string(),
            None,   // client_id (auto-registered)
            None,   // client_secret
        );

        // Step 2: Get OAuth URL for user.
        let app_data = client.register_app("Liberation Client", &[
            megalodon::oauth::Scope::Read,
            megalodon::oauth::Scope::Write,
            megalodon::oauth::Scope::Follow,
        ]).await?;

        // Step 3: User visits URL, gets authorization code.
        // (This is interactive — the app opens a browser/webview.)
        let auth_url = client.get_authorization_url(
            &app_data.client_id,
            &app_data.client_secret,
        )?;

        // Step 4: Exchange code for token.
        let token_data = client.fetch_access_token(
            &app_data.client_id,
            &app_data.client_secret,
            &code, // From OAuth redirect.
            &app_data.redirect_uri,
        ).await?;

        // Step 5: Verify credentials.
        let account = client.verify_account_credentials().await?;

        Ok(MastodonSession {
            instance_url: instance_url.to_string(),
            access_token: token_data.access_token,
            account: map_mastodon_account(account),
            client,
        })
    }
}
```

### 1.4 Timeline Fetching

```rust
// libs/protocols/activitypub/src/timeline.rs

impl ActivityPubAdapter {
    /// Cold start: fetch home timeline.
    pub async fn cold_start_timeline(&self, session_index: usize, limit: u32) -> Result<Vec<UnifiedPost>> {
        let session = &self.sessions[session_index];
        let statuses = session.client
            .get_home_timeline(Some(megalodon::megalodon::GetTimelineOptions {
                limit: Some(limit.min(40)), // Mastodon default is 40.
                ..Default::default()
            }))
            .await?;

        Ok(statuses.json.into_iter()
            .filter_map(|s| self.map_status_to_post(&s).ok())
            .collect())
    }

    /// Incremental: fetch since a status ID.
    pub async fn incremental_timeline(
        &self,
        session_index: usize,
        since_id: Option<&str>,
        max_id: Option<&str>,
        limit: u32,
    ) -> Result<Vec<UnifiedPost>> {
        let session = &self.sessions[session_index];
        let statuses = session.client
            .get_home_timeline(Some(megalodon::megalodon::GetTimelineOptions {
                since_id: since_id.map(|s| s.to_string()),
                max_id: max_id.map(|s| s.to_string()),
                limit: Some(limit.min(40)),
                ..Default::default()
            }))
            .await?;

        Ok(statuses.json.into_iter()
            .filter_map(|s| self.map_status_to_post(&s).ok())
            .collect())
    }

    /// Streaming: WebSocket for real-time updates.
    pub async fn subscribe_streaming(&self, session_index: usize, bus: Arc<EventBus>) -> Result<tokio::task::JoinHandle<()>> {
        let session = &self.sessions[session_index];

        // Connect to streaming API.
        let streaming_url = format!(
            "wss://{}/api/v1/streaming?access_token={}&stream=user",
            session.instance_url.trim_start_matches("https://"),
            session.access_token
        );

        let (ws_stream, _) = tokio_tungstenite::connect_async(&streaming_url).await?;
        let (_, read) = ws_stream.split();

        let handle = tokio::spawn(async move {
            let mut reader = read;
            while let Some(msg) = reader.next().await {
                match msg {
                    Ok(tungstenite::Message::Text(text)) => {
                        let event: StreamingEvent = serde_json::from_str(&text).unwrap_or_default();
                        match event.event.as_str() {
                            "update" => {
                                // New or updated status.
                                if let Some(status) = event.payload {
                                    if let Ok(post) = map_status_to_unified_post(&status) {
                                        let _ = bus.new_post.send(Arc::new(NewPostEvent {
                                            post,
                                            protocol: Protocol::ActivityPub,
                                            is_backfill: false,
                                            received_at: Utc::now(),
                                        }));
                                    }
                                }
                            }
                            "delete" => {
                                // Status deleted.
                                if let Some(deleted_id) = event.payload {
                                    let _ = bus.deleted_post.send(Arc::new(DeletedPostEvent {
                                        post_id: deleted_id,
                                        protocol: Protocol::ActivityPub,
                                        reason: None,
                                    }));
                                }
                            }
                            "notification" => {
                                // New notification (mention, follow, etc.).
                                // Handled by notification subscriber.
                            }
                            _ => {}
                        }
                    }
                    Err(e) => {
                        tracing::warn!("Streaming WebSocket error: {}", e);
                        break;
                    }
                    _ => {}
                }
            }
        });

        Ok(handle)
    }
}
```

### 1.5 Status → Canonical Mapping (CRITICAL PATH)

```rust
// libs/protocols/activitypub/src/status_mapper.rs

impl ActivityPubAdapter {
    /// Map Mastodon/Megalodon Status → UnifiedPost.
    fn map_status_to_post(&self, status: &megalodon::entities::Status) -> Result<UnifiedPost> {
        // Sanitize HTML content — CRITICAL for security.
        let plain_text = Self::sanitize_html_to_plain_text(&status.content);
        let rich_content = Self::sanitize_html_keep_formatting(&status.content);

        // Extract tags (hashtags).
        let tags: Vec<String> = status.tags.iter()
            .filter(|t| t.name.starts_with('#'))
            .map(|t| t.name.trim_start_matches('#').to_string())
            .collect();

        // Extract mentions (@handles).
        let mentions: Vec<String> = status.mentions.iter()
            .map(|m| m.acct.clone())
            .collect();

        // Extract media attachments.
        let media: Vec<UnifiedAttachment> = status.media_attachments.iter()
            .map(|m| UnifiedAttachment {
                id: m.id.clone(),
                kind: match m.kind {
                    megalodon::entities::attachment::AttachmentType::Image => AttachmentType::Image,
                    megalodon::entities::attachment::AttachmentType::Video => AttachmentType::Video,
                    megalodon::entities::attachment::AttachmentType::Gifv => AttachmentType::Video,
                    megalodon::entities::attachment::AttachmentType::Audio => AttachmentType::Audio,
                    _ => AttachmentType::Document,
                },
                url: m.url.clone(),
                mime_type: None,
                alt_text: m.description.clone(),
                width: m.meta.as_ref().and_then(|meta| meta.original.as_ref()).map(|o| o.width as u32),
                height: m.meta.as_ref().and_then(|meta| meta.original.as_ref()).map(|o| o.height as u32),
                size: None,
                blurhash: m.blurhash.clone(),
                local_cache_path: None,
            })
            .collect();

        // Map visibility.
        let visibility = match status.visibility {
            megalodon::entities::status::StatusVisibility::Public => Visibility::Public,
            megalodon::entities::status::StatusVisibility::Unlisted => Visibility::Public, // Unlisted is still public, just not in timelines.
            megalodon::entities::status::StatusVisibility::Private => Visibility::Followers,
            megalodon::entities::status::StatusVisibility::Direct => Visibility::Direct,
        };

        Ok(UnifiedPost {
            id: status.uri.clone().unwrap_or_else(|| status.url.clone().unwrap_or_default()),
            protocol: Protocol::ActivityPub,
            protocol_id: status.id.clone(),
            author: map_mastodon_account_to_author(&status.account),
            content: plain_text,
            rich_content: Some(RichText::Html(rich_content)),
            created_at: status.created_at,
            reply_to: status.in_reply_to_id.clone(),
            root_post: status.in_reply_to_account_id.clone(), // Approximate — Mastodon doesn't expose thread root.
            media,
            mentions,
            tags,
            visibility,
            language: status.language.clone(),
            source: Some(serde_json::to_value(status)?),
        })
    }

    /// Sanitize HTML to plain text. Strip ALL tags.
    fn sanitize_html_to_plain_text(html: &str) -> String {
        // Use ammonia crate with empty allowlist for plain text.
        // Alternatively: regex-based tag stripping.
        let re = regex::Regex::new(r"<[^>]*>").unwrap();
        let text = re.replace_all(html, "");
        // Decode HTML entities.
        let text = html_escape::decode_html_entities(&text);
        text.to_string()
    }

    /// Sanitize HTML keeping safe formatting tags.
    /// CRITICAL SECURITY: Strip <script>, <iframe>, <style>, <object>, <embed>, event handlers.
    fn sanitize_html_keep_formatting(html: &str) -> String {
        use ammonia::Builder;
        Builder::new()
            .add_tags(&["p", "br", "a", "strong", "em", "ul", "ol", "li", "blockquote", "code", "pre", "span"])
            .add_allowed_classes("span", &["h-card", "invisible", "ellipsis"])
            .link_rel(None) // Strip rel attributes.
            .clean(html)
            .to_string()
    }
}
```

### 1.6 Publishing

```rust
// libs/protocols/activitypub/src/publish.rs

impl ActivityPubAdapter {
    /// Publish a status to Mastodon-compatible instance.
    pub async fn publish_post(
        &self,
        session_index: usize,
        content: &PostContent,
    ) -> Result<UnifiedPost> {
        let session = &self.sessions[session_index];

        // Upload media first.
        let media_ids: Vec<String> = futures::future::join_all(
            content.media.iter().map(|m| {
                let client = &session.client;
                async move {
                    let attachment = client.upload_media(
                        m.data.clone(),
                        Some(m.mime_type.clone().unwrap_or_default()),
                        m.alt_text.clone(),
                    ).await?;
                    Ok::<_, anyhow::Error>(attachment.id)
                }
            })
        ).await.into_iter().collect::<Result<Vec<_>>>()?;

        // Build status options.
        let mut options = megalodon::megalodon::PostStatusOptions {
            status: Some(content.text.clone()),
            media_ids: Some(media_ids),
            sensitive: Some(content.content_warning.is_some()),
            spoiler_text: content.content_warning.clone(),
            visibility: Some(match content.visibility {
                Visibility::Public => megalodon::entities::status::StatusVisibility::Public,
                Visibility::Followers => megalodon::entities::status::StatusVisibility::Private,
                Visibility::Direct => megalodon::entities::status::StatusVisibility::Direct,
            }),
            in_reply_to_id: content.reply_to.clone(),
            language: content.language.clone(),
            ..Default::default()
        };

        // Post!
        let status = session.client.post_status(options).await?;
        let post = self.map_status_to_post(&status.json)?;

        // Emit locally.
        let _ = self.bus.new_post.send(Arc::new(NewPostEvent {
            post: post.clone(),
            protocol: Protocol::ActivityPub,
            is_backfill: false,
            received_at: Utc::now(),
        }));

        Ok(post)
    }

    /// Boost (repost) a status.
    pub async fn boost(&self, session_index: usize, status_id: &str) -> Result<()> {
        let session = &self.sessions[session_index];
        session.client.reblog_status(status_id.to_string()).await?;
        Ok(())
    }

    /// Favorite (like) a status.
    pub async fn favorite(&self, session_index: usize, status_id: &str) -> Result<()> {
        let session = &self.sessions[session_index];
        session.client.favourite_status(status_id.to_string()).await?;
        Ok(())
    }
}
```

### 1.7 ActivityPub Sync Strategy

| Phase | Method | Batch | Interval |
|-------|--------|-------|----------|
| Cold start | REST `get_home_timeline` | 40 posts | Once |
| Incremental | REST with `since_id`/`max_id` | 40 posts | 120s (poll) |
| Realtime | WebSocket streaming API | Per event | Continuous |
| Profile refresh | REST `get_account` | Per profile | 24h + jitter |
| Instance check | REST `get_instance` | Once | On connect |

---

## Part 2: IRC Adapter

### 2.1 Philosophy

IRC is the simplest protocol (TCP, text lines, no persistence). We implement it natively — no external bouncer required for basic use, but **soju/ZNC recommended** for always-on connectivity.

### 2.2 Implementation (Minimal)

```rust
// libs/protocols/irc/src/lib.rs

use tokio::net::TcpStream;
use tokio::io::{BufReader, AsyncBufReadExt, AsyncWriteExt};
use tokio_native_tls::TlsConnector;

pub struct IRCAdapter {
    bus: Arc<EventBus>,
    connections: HashMap<String, IRCConnection>,
}

struct IRCConnection {
    server: String,
    nick: String,
    channels: Vec<String>,
    write: tokio::io::WriteHalf<tokio_native_tls::TlsStream<TcpStream>>,
    /// Bouncer URL (optional, for history replay).
    bouncer_url: Option<String>,
}

impl IRCAdapter {
    /// Connect to an IRC server with optional TLS.
    pub async fn connect(
        server: &str,
        port: u16,
        use_tls: bool,
        nick: &str,
        password: Option<&str>,
        channels: Vec<String>,
    ) -> Result<IRCConnection> {
        let tcp = TcpStream::connect((server, port)).await?;
        let stream = if use_tls {
            let tls = TlsConnector::from(tokio_native_tls::native_tls::TlsConnector::builder()
                .danger_accept_invalid_certs(false)
                .build()?);
            tls.connect(server, tcp).await?
        } else {
            return Err(anyhow::anyhow!("Plaintext IRC not supported. Use TLS."));
        };

        let (read, mut write) = tokio::io::split(stream);

        // IRC handshake.
        if let Some(pass) = password {
            write.write_all(format!("PASS {}\r\n", pass).as_bytes()).await?;
        }
        write.write_all(format!("NICK {}\r\n", nick).as_bytes()).await?;
        write.write_all(format!("USER {} 0 * :Liberation Client\r\n", nick).as_bytes()).await?;

        // Join channels.
        for channel in &channels {
            write.write_all(format!("JOIN {}\r\n", channel).as_bytes()).await?;
        }

        // Spawn read loop.
        let connection = IRCConnection {
            server: server.to_string(),
            nick: nick.to_string(),
            channels,
            write,
            bouncer_url: None,
        };

        Ok(connection)
    }

    /// IRC read loop — parse PRIVMSG, NOTICE, JOIN, PART, etc.
    async fn read_loop(
        mut reader: BufReader<tokio::io::ReadHalf<tokio_native_tls::TlsStream<TcpStream>>>,
        bus: Arc<EventBus>,
        server: String,
    ) {
        let mut line = String::new();
        loop {
            line.clear();
            match reader.read_line(&mut line).await {
                Ok(0) => break, // Connection closed.
                Ok(_) => {
                    if let Some(event) = parse_irc_line(&line, &server) {
                        match event {
                            IRCEvent::PrivMsg { from, to, message } => {
                                let is_dm = !to.starts_with('#');
                                let msg = UnifiedMessage {
                                    id: format!("irc:{}:{}:{}", server, from, chrono::Utc::now().timestamp_millis()),
                                    conversation_id: to.clone(),
                                    protocol: Protocol::IRC,
                                    sender_id: from,
                                    body: message,
                                    encrypted_body: None,
                                    sent_at: chrono::Utc::now(),
                                    received_at: chrono::Utc::now(),
                                    edit_history: Vec::new(),
                                    encryption_status: EncryptionStatus::None,
                                };
                                let _ = bus.new_message.send(Arc::new(NewMessageEvent {
                                    message: msg,
                                    protocol: Protocol::IRC,
                                }));
                            }
                            IRCEvent::Join { nick, channel } => {
                                tracing::debug!("IRC: {} joined {}", nick, channel);
                            }
                            IRCEvent::Part { nick, channel } => {
                                tracing::debug!("IRC: {} left {}", nick, channel);
                            }
                            IRCEvent::Ping(token) => {
                                // PONG response handled by write loop.
                            }
                        }
                    }
                }
                Err(e) => {
                    tracing::error!("IRC read error: {}", e);
                    break;
                }
            }
        }
    }

    /// Send a message to a channel or user.
    pub async fn send_message(&mut self, connection_id: &str, target: &str, message: &str) -> Result<()> {
        let conn = self.connections.get_mut(connection_id)
            .ok_or_else(|| anyhow::anyhow!("No connection: {}", connection_id))?;
        conn.write.write_all(format!("PRIVMSG {} :{}\r\n", target, message).as_bytes()).await?;
        Ok(())
    }

    /// Join a channel.
    pub async fn join_channel(&mut self, connection_id: &str, channel: &str) -> Result<()> {
        let conn = self.connections.get_mut(connection_id)
            .ok_or_else(|| anyhow::anyhow!("No connection: {}", connection_id))?;
        conn.write.write_all(format!("JOIN {}\r\n", channel).as_bytes()).await?;
        conn.channels.push(channel.to_string());
        Ok(())
    }
}

fn parse_irc_line(line: &str, server: &str) -> Option<IRCEvent> {
    // Parse :prefix COMMAND params... :trailing
    // e.g., ":alice!user@host PRIVMSG #channel :Hello world"
    // ... basic parsing logic ...
    None
}

enum IRCEvent {
    PrivMsg { from: String, to: String, message: String },
    Join { nick: String, channel: String },
    Part { nick: String, channel: String },
    Ping(String),
}
```

### 2.3 IRC Design Constraints

- **Ephemeral by nature.** Messages have no persistence. History requires a bouncer.
- **No reactions, no threading, no media.** Plain text only.
- **DM vs Channel**: `PRIVMSG #channel` = channel. `PRIVMSG nick` = DM.
- **Bouncer recommended**: `soju` (single binary, IRCv3, SQLite) for always-on + history replay.
- **Anti-IRC-flood**: Max 2-5 msg/sec per connection.

---

## Part 3: XMPP Adapter

### 3.1 Architecture

```rust
// libs/protocols/xmpp/src/lib.rs

use tokio_xmpp::Client as XMPPClient;
use xmpp_parsers::{message::Message, presence::Presence, jid::Jid};

pub struct XMPPAdapter {
    bus: Arc<EventBus>,
    connections: HashMap<String, XMPPSession>,
}

struct XMPPSession {
    jid: Jid,
    client: XMPPClient,
    /// OMEMO encryption state.
    omemo_store: Option<OmemoStore>,
}

impl XMPPAdapter {
    /// Connect to an XMPP server via WebSocket.
    pub async fn connect(
        jid: &str,
        password: &str,
        server: Option<&str>,
    ) -> Result<XMPPSession> {
        let jid: Jid = jid.parse()?;
        let server = server.unwrap_or_else(|| jid.domain_str());

        // Connect via WebSocket (XEP-0459).
        let client = XMPPClient::new(jid.clone(), password.to_string())
            .set_websocket_url(format!("wss://{}:5443/ws", server));

        let mut client = client.connect().await?;

        // Enable Stream Management (XEP-0198) for reliable delivery.
        // Enable MAM (XEP-0313) for history.
        // Enable OMEMO (XEP-0384) for E2EE.

        // Spawn read loop.
        let bus = self.bus.clone();
        tokio::spawn(async move {
            loop {
                match client.next().await {
                    Some(Ok(event)) => handle_xmpp_event(event, &bus).await,
                    Some(Err(e)) => tracing::error!("XMPP error: {}", e),
                    None => break,
                }
            }
        });

        Ok(XMPPSession { jid, client, omemo_store: None })
    }

    /// Send a message.
    pub async fn send_message(
        &mut self,
        session_id: &str,
        to: &str,
        body: &str,
        encrypt: bool,
    ) -> Result<()> {
        let session = self.connections.get_mut(session_id)
            .ok_or_else(|| anyhow::anyhow!("No session: {}", session_id))?;

        let mut message = Message::new(Some(to.parse()?));
        message.body = Some(body.to_string());
        message.type_ = xmpp_parsers::message::MessageType::Chat;

        // OMEMO encrypt if enabled.
        if encrypt && session.omemo_store.is_some() {
            message = encrypt_omemo(message, &session.omemo_store.unwrap())?;
        }

        session.client.send(message).await?;
        Ok(())
    }

    /// Join a MUC (Multi-User Chat).
    pub async fn join_muc(&mut self, session_id: &str, room: &str, nick: &str) -> Result<()> {
        let session = self.connections.get_mut(session_id)?;
        let muc_jid: Jid = format!("{}/{}", room, nick).parse()?;
        let presence = Presence::new(xmpp_parsers::presence::Type::None)
            .with_to(muc_jid);
        session.client.send(presence).await?;
        Ok(())
    }
}

/// Handle incoming XMPP events.
async fn handle_xmpp_event(event: tokio_xmpp::Event, bus: &Arc<EventBus>) {
    match event {
        tokio_xmpp::Event::Stanza(stanza) => match stanza {
            xmpp_parsers::Element::Message(msg) => {
                let unified = map_xmpp_message_to_canonical(&msg);
                let _ = bus.new_message.send(Arc::new(NewMessageEvent {
                    message: unified,
                    protocol: Protocol::XMPP,
                }));
            }
            _ => {}
        },
        tokio_xmpp::Event::Online => {
            let _ = bus.protocol_connected.send(Arc::new(ProtocolConnectedEvent {
                protocol: Protocol::XMPP,
                identity: "connected".to_string(),
                connected_at: Utc::now(),
            }));
        }
        tokio_xmpp::Event::Disconnected(e) => {
            let _ = bus.protocol_disconnected.send(Arc::new(ProtocolDisconnectedEvent {
                protocol: Protocol::XMPP,
                reason: DisconnectReason::NetworkError,
                will_retry: true,
                retry_in_seconds: Some(30),
            }));
        }
    }
}
```

### 3.2 XMPP Essential XEPs

| XEP | Description | Priority |
|-----|-------------|----------|
| XEP-0030 | Service Discovery | REQUIRED |
| XEP-0045 | Multi-User Chat (MUC) | HIGH |
| XEP-0085 | Chat State Notifications | MEDIUM |
| XEP-0198 | Stream Management | REQUIRED |
| XEP-0280 | Message Carbons | REQUIRED |
| XEP-0313 | Message Archive Management | REQUIRED |
| XEP-0352 | Client State Indication | HIGH |
| XEP-0363 | HTTP File Upload | MEDIUM |
| XEP-0384 | OMEMO Encryption | REQUIRED |
| XEP-0459 | XMPP over WebSocket | REQUIRED |

### 3.3 XMPP Design Constraints

- **XML.** Expensive to parse, but `xmpp-parsers` handles it.
- **Messaging-only.** No social feed concept. Routes to Messages tab.
- **Federation.** Server-to-server via DNS SRV records.
- **MAM (XEP-0313).** Server-side history. Requires server support.
- **OMEMO (XEP-0384).** Double Ratchet for E2EE. Complex to implement from scratch. Use library.

---

## Part 4: Protocol Completion Summary

| Protocol | Adapter | Crate | Status |
|----------|---------|-------|--------|
| **Nostr** | ✅ Iter 5 | `nostr-sdk` | Fully spec'd (code-level Rust) |
| **AT Protocol** | ✅ Iter 6 | `shrike` | Fully spec'd |
| **ActivityPub** | ✅ Iter 7 | `megalodon` | Fully spec'd |
| **Matrix** | ✅ Iter 6 | `matrix-rust-sdk` | Chat module spec'd |
| **IRC** | ✅ Iter 7 | Raw `tokio::net::TcpStream` | Lightweight spec |
| **XMPP** | ✅ Iter 7 | `tokio-xmpp` + `xmpp-parsers` | Lightweight spec |

**All 6 protocols now have implementation specifications.**

---

*Documento gerado na Iteração 7 do /loop. Completando os 6 adapters de protocolo.*
