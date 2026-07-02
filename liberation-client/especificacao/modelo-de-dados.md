# Multi-Protocol Social Media Client: Canonical Data Model

> **Author:** Database Architecture & Protocol Engineering
> **Iteration:** 2
> **Target:** Serverless on-device client for ActivityPub, AT Protocol, Nostr, Matrix, IRC, XMPP
> **Storage:** SQLite + FTS5 on device (no remote database)

---

## Table of Contents

1. [Canonical Type Definitions](#1-canonical-type-definitions)
2. [Complete SQLite Schema (DDL)](#2-complete-sqlite-schema-ddl)
3. [Protocol Mapping Rules](#3-protocol-mapping-rules)
4. [Sync Strategies Per Protocol](#4-sync-strategies-per-protocol)
5. [Migration and Schema Evolution](#5-migration-and-schema-evolution)

---

## 1. Canonical Type Definitions

### 1.1 UnifiedPost

The canonical representation of a "post" (status, note, tweet, message in a public channel) across all protocols.

```typescript
interface UnifiedPost {
  /** Canonical ID: "{protocol}:{protocol_native_id}" 
   *  Examples:
   *    "nostr:32charhex"
   *    "atproto:at://did:plc:abc/app.bsky.feed.post/rkey123"
   *    "activitypub:https://server/@user/123456"
   *    "matrix:$eventid:matrix.org"
   *    "irc:libera.chat/#channel:1719000000:seq42"
   *    "xmpp:user@domain/resource:stanza-id"
   */
  id: string;
  
  /** Source protocol identifier */
  protocol: 'nostr' | 'atproto' | 'activitypub' | 'matrix' | 'irc' | 'xmpp';
  
  /** Reference to canonical profile ID of the author */
  authorId: string;
  
  /** Plain text content (required: stripped HTML, normalized whitespace, max 64KB) */
  content: string;
  
  /** Rich content representation (sanitized HTML or structured JSON for rendering) */
  richContent?: string;
  
  /** ISO 8601 timestamp of original creation on the protocol */
  createdAt: string;
  
  /** Canonical ID of the post this is a reply to (null if top-level) */
  replyToId?: string;
  
  /** Canonical ID of the root post in the thread (null if top-level) */
  rootPostId?: string;
  
  /** Ordered array of attached media/files */
  media?: UnifiedAttachment[];
  
  /** Array of mentioned profile canonical IDs */
  mentions?: string[];
  
  /** Array of hashtag strings (without # prefix) */
  tags?: string[];
  
  /**
   * Visibility/access control.
   * Rationale: Mastodon has public/unlisted/private/direct; Nostr lacks native visibility
   * but NIP-36 provides flags; AT Protocol is public-only as of now.
   * 'unknown' when protocol doesn't expose visibility.
   */
  visibility: 'public' | 'unlisted' | 'private' | 'direct' | 'unknown';
  
  /** BCP-47 language tag (ISO 639-1 + region). Null if undetectable. */
  language?: string;
  
  /** Original protocol-native object serialized as JSON (for debugging, re-publishing, bridging) */
  source?: Record<string, unknown>;
  
  /** 
   * Cross-protocol references: when this post references content from another protocol
   * (e.g., a Nostr reply to an AT Protocol post).
   */
  crossProtocolRefs?: CrossProtocolRef[];
}

interface CrossProtocolRef {
  /** Canonical ID of the referenced content */
  targetId: string;
  /** Protocol of the referenced content */
  targetProtocol: string;
  /** Relationship type */
  relationship: 'reply' | 'repost' | 'quote' | 'reference' | 'anchor';
}
```

### 1.2 UnifiedProfile

Identity representation unified across protocols. Note that the same human may have profiles on multiple protocols; identity linking connects these.

```typescript
interface UnifiedProfile {
  /** Canonical ID: "{protocol}:{protocol_native_id}" */
  id: string;
  
  protocol: 'nostr' | 'atproto' | 'activitypub' | 'matrix' | 'irc' | 'xmpp';
  
  /** Protocol-native handle/identifier display string.
   *  Nostr: "npub1..." or NIP-05 "user@domain"
   *  AT Proto: "handle.bsky.social"
   *  ActivityPub: "@user@server"
   *  Matrix: "@user:matrix.org"
   *  IRC: "nick!user@host"
   *  XMPP: "user@domain/resource"
   */
  handle: string;
  
  /** Display/nick name */
  displayName?: string;
  
  /** URL of avatar image on remote server */
  avatarUrl?: string;
  
  /** Local file path of cached avatar (set by cache layer) */
  avatarLocalPath?: string;
  
  /** URL of banner/header image */
  bannerUrl?: string;
  
  /** Plain-text biography/description */
  bio?: string;
  
  /** Numeric counts (may be approximate or unavailable from some protocols) */
  followersCount?: number;
  followingCount?: number;
  postsCount?: number;
  
  /** 
   * Protocol-level verification signals, NOT global identity verification.
   * Nostr: NIP-05 verified
   * AT Proto: DID-plc verified handle
   * Mastodon: Account has verified link in profile
   */
  verificationMethod?: string;    // 'nip05' | 'did_plc' | 'website_link' | 'none'
  verificationValue?: string;     // verified handle, linked domain
  
  /** Protocol-specific metadata as JSON blob */
  metadata?: Record<string, unknown>;
  
  /** Timestamps */
  createdAt?: string;   // when this profile was created on the protocol
  updatedAt: string;    // when we last synced this profile
  
  /** Whether we have the full profile or just a stub (from mention/tag) */
  isStub: boolean;
}
```

### 1.3 UnifiedReaction

Covers likes, reposts/boosts, emoji reactions, bookmarks.

```typescript
interface UnifiedReaction {
  /** Canonical ID: "{protocol}:{protocol_native_id}" */
  id: string;
  
  /** Target post canonical ID */
  postId: string;
  
  /** Reactor's canonical profile ID */
  authorId: string;
  
  /** 
   * Reaction type.
   * 'like'      -> mastodon favourite, bluesky like, nostr kind:7 with content "+" or "❤️"
   * 'repost'    -> mastodon boost, bluesky repost, nostr kind:6
   * 'bookmark'  -> mastodon bookmark, nostr kind:10003 (not unified yet)
   * 'emoji'     -> generic emoji reaction (Matrix, some ActivityPub implementations)
   */
  type: 'like' | 'repost' | 'bookmark' | 'emoji';
  
  /** For type='emoji': the emoji character or custom emoji shortcode.
   *  For type='like': typically "❤️" or "+".
   *  Null for 'repost' and 'bookmark'.
   */
  emoji?: string;
  
  /** ISO 8601 timestamp */
  createdAt: string;
  
  protocol: string;
  
  /** Original protocol object */
  source?: Record<string, unknown>;
}
```

### 1.4 UnifiedMessage

Private/direct messages and encrypted communications.

```typescript
interface UnifiedMessage {
  /** Canonical ID: "{protocol}:{protocol_native_id}" */
  id: string;
  
  /** Conversation this message belongs to */
  conversationId: string;
  
  protocol: 'nostr' | 'atproto' | 'activitypub' | 'matrix' | 'irc' | 'xmpp';
  
  /** Sender's canonical profile ID */
  senderId: string;
  
  /** Plain text body */
  body: string;
  
  /** Rich text body (for protocols that support formatting) */
  richBody?: string;
  
  /** Whether the message is end-to-end encrypted */
  encrypted: boolean;
  
  /** Encryption metadata (algorithm, key hints, etc.) when encrypted */
  encryptionInfo?: {
    algorithm: string;      // 'nostr.nip04' | 'olm' | 'megolm' | 'omemo' | 'none'
    deviceId?: string;
    sessionId?: string;
  };
  
  /** ISO 8601 send timestamp (client clock or server timestamp) */
  sentAt: string;
  
  /** ISO 8601 receive timestamp (local device time) */
  receivedAt?: string;
  
  /** Parent message ID for replies within conversation */
  replyToId?: string;
  
  /** Attachments within the message */
  attachments?: UnifiedAttachment[];
  
  /** Edit history: ordered list of previous body texts */
  editHistory?: { body: string; editedAt: string }[];
  
  /** Delivered status tracking */
  status: 'sending' | 'sent' | 'delivered' | 'read' | 'failed';
  
  source?: Record<string, unknown>;
}
```

### 1.5 UnifiedConversation

Chat rooms, DM threads, channels, MUCs.

```typescript
interface UnifiedConversation {
  /** Canonical ID: "{protocol}:{protocol_native_id}"
   *  Matrix: room_id
   *  IRC: "server/#channel" or "server/nick"
   *  XMPP: room JID
   *  ActivityPub: conversation URI from Mastodon API
   *  Nostr: computed from participant public keys (sorted, hashed)
   */
  id: string;
  
  protocol: string;
  
  /** 'dm' -> two-person direct message
   *  'group' -> multi-person group chat (Nostr kind:14, XMPP MUC)
   *  'channel' -> public/private channel (IRC, Matrix public room)
   *  'room' -> generic Matrix room
   */
  type: 'dm' | 'group' | 'channel' | 'room';
  
  /** Human-readable name (room name, channel topic, participant names for DM) */
  name?: string;
  
  /** Avatar for group conversations */
  avatarUrl?: string;
  
  /** Array of participant canonical profile IDs */
  participantIds: string[];
  
  /** Last message canonical ID for quick reference */
  lastMessageId?: string;
  
  /** ISO 8601 timestamp of last activity */
  lastMessageAt?: string;
  
  /** Encryption state */
  isEncrypted: boolean;
  
  /** Unread count (local tracking) */
  unreadCount: number;
  
  /** Read marker: timestamp or message ID up to which we've read */
  readUpTo?: string;
  
  /** Protocol-specific metadata */
  metadata?: Record<string, unknown>;
}
```

### 1.6 UnifiedAttachment

Media and file attachments.

```typescript
interface UnifiedAttachment {
  id: string;                       // "{protocol}:{attachment_id}"
  type: 'image' | 'video' | 'audio' | 'document' | 'gifv';
  /** Remote URL of the attachment on the originating server */
  url: string;
  /** MIME type (e.g., "image/jpeg", "video/mp4") */
  mimeType: string;
  /** Alternative text for accessibility */
  altText?: string;
  /** Original filename extracted from protocol metadata */
  filename?: string;
  /** Dimensions (present for images and video) */
  width?: number;
  height?: number;
  /** File size in bytes */
  sizeBytes?: number;
  /** Blurhash string for placeholder rendering */
  blurhash?: string;
  /** Local filesystem path after caching */
  localCachePath?: string;
  /** Thumbnail URL (video thumb, etc.) */
  thumbnailUrl?: string;
  /** Additional protocol-specific metadata */
  metadata?: Record<string, unknown>;
}
```

### 1.7 UnifiedNotification

Aggregated notifications from all protocols.

```typescript
interface UnifiedNotification {
  id: string;
  protocol: string;
  type: 'mention' | 'reply' | 'like' | 'repost' | 'follow' | 'dm' | 'invite';
  /** The profile that triggered this notification */
  actorId: string;
  /** The target post (for mentions, replies, likes, reposts) */
  postId?: string;
  /** Conversation (for DMs, invites) */
  conversationId?: string;
  /** ISO 8601 timestamp */
  createdAt: string;
  /** Whether the user has seen this notification */
  isRead: boolean;
  source?: Record<string, unknown>;
}
```

---

## 2. Complete SQLite Schema (DDL)

### 2.1 Schema Versioning & Pragmas

```sql
-- Applied once at database open
PRAGMA journal_mode = WAL;
PRAGMA foreign_keys = ON;
PRAGMA synchronous = NORMAL;
PRAGMA cache_size = -64000;        -- 64 MB page cache
PRAGMA busy_timeout = 5000;
PRAGMA recursive_triggers = ON;

-- ============================================================
-- Schema Version Tracking
-- ============================================================
CREATE TABLE IF NOT EXISTS schema_version (
    version     INTEGER PRIMARY KEY,
    description TEXT    NOT NULL,
    applied_at  TEXT    NOT NULL DEFAULT (datetime('now'))
);

INSERT INTO schema_version (version, description) VALUES (1, 'Initial canonical data model');
```

### 2.2 Core Tables

#### Profiles

```sql
CREATE TABLE profiles (
    id                TEXT PRIMARY KEY,   -- canonical ID
    protocol          TEXT    NOT NULL,
    handle            TEXT    NOT NULL,
    display_name      TEXT,
    avatar_url        TEXT,
    avatar_local_path TEXT,
    banner_url        TEXT,
    bio               TEXT,
    followers_count   INTEGER DEFAULT 0,
    following_count   INTEGER DEFAULT 0,
    posts_count       INTEGER DEFAULT 0,
    verification_method TEXT,
    verification_value  TEXT,
    metadata          TEXT,               -- JSON blob
    is_stub           INTEGER NOT NULL DEFAULT 1,
    created_at        TEXT,
    updated_at        TEXT    NOT NULL DEFAULT (datetime('now')),
    inserted_at       TEXT    NOT NULL DEFAULT (datetime('now')),
    UNIQUE(protocol, handle)
);

CREATE INDEX idx_protocol_handle ON profiles(protocol, handle);
CREATE INDEX idx_profiles_protocol ON profiles(protocol);
CREATE INDEX idx_profiles_updated ON profiles(updated_at);
```

#### Posts

```sql
CREATE TABLE posts (
    id              TEXT    PRIMARY KEY,  -- canonical ID
    protocol        TEXT    NOT NULL,
    author_id       TEXT    NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    content         TEXT,                 -- plain text
    rich_content    TEXT,                 -- sanitized HTML or structured JSON
    created_at      TEXT    NOT NULL,
    reply_to_id     TEXT    REFERENCES posts(id) ON DELETE SET NULL,
    root_post_id    TEXT    REFERENCES posts(id) ON DELETE SET NULL,
    visibility      TEXT    NOT NULL DEFAULT 'public'
                            CHECK(visibility IN ('public','unlisted','private','direct','unknown')),
    language        TEXT,
    is_deleted      INTEGER NOT NULL DEFAULT 0,
    raw_data        TEXT,                 -- JSON blob of original protocol object
    inserted_at     TEXT    NOT NULL DEFAULT (datetime('now')),
    updated_at      TEXT    NOT NULL DEFAULT (datetime('now'))
);

CREATE INDEX idx_posts_author ON posts(author_id);
CREATE INDEX idx_posts_protocol ON posts(protocol);
CREATE INDEX idx_posts_created ON posts(created_at DESC);
CREATE INDEX idx_posts_reply_to ON posts(reply_to_id);
CREATE INDEX idx_posts_root ON posts(root_post_id);
CREATE INDEX idx_posts_visibility ON posts(visibility);
CREATE INDEX idx_posts_protocol_created ON posts(protocol, created_at DESC);

-- Partial index for non-deleted posts (common query filter)
CREATE INDEX idx_posts_active ON posts(created_at DESC) WHERE is_deleted = 0;
```

#### Mentions (many-to-many: posts → profiles)

```sql
CREATE TABLE post_mentions (
    post_id    TEXT NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    profile_id TEXT NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    index      INTEGER NOT NULL DEFAULT 0,  -- ordinal position in post
    PRIMARY KEY (post_id, profile_id)
);

CREATE INDEX idx_post_mentions_profile ON post_mentions(profile_id);
```

#### Tags/Hashtags

```sql
CREATE TABLE post_tags (
    post_id TEXT NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    tag     TEXT NOT NULL COLLATE NOCASE,  -- hashtag without '#'
    index   INTEGER NOT NULL DEFAULT 0,
    PRIMARY KEY (post_id, tag)
);

CREATE INDEX idx_post_tags_tag ON post_tags(tag);
```

#### Cross-Protocol References

```sql
CREATE TABLE cross_protocol_refs (
    id                INTEGER PRIMARY KEY AUTOINCREMENT,
    source_post_id    TEXT NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    target_canonical_id TEXT NOT NULL,   -- target canonical ID (may not exist locally yet)
    target_protocol   TEXT NOT NULL,
    relationship      TEXT NOT NULL CHECK(relationship IN ('reply','repost','quote','reference','anchor')),
    created_at        TEXT NOT NULL DEFAULT (datetime('now')),
    UNIQUE(source_post_id, target_canonical_id)
);

CREATE INDEX idx_cross_refs_target ON cross_protocol_refs(target_canonical_id);
```

#### Reactions

```sql
CREATE TABLE reactions (
    id         TEXT    PRIMARY KEY,  -- canonical ID
    post_id    TEXT    NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    author_id  TEXT    NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    type       TEXT    NOT NULL CHECK(type IN ('like','repost','bookmark','emoji')),
    emoji      TEXT,                 -- NULL for 'repost'/'bookmark'
    protocol   TEXT    NOT NULL,
    created_at TEXT    NOT NULL,
    raw_data   TEXT,                 -- JSON blob
    UNIQUE(post_id, author_id, type, COALESCE(emoji, ''))
);

CREATE INDEX idx_reactions_post ON reactions(post_id);
CREATE INDEX idx_reactions_author ON reactions(author_id);
CREATE INDEX idx_reactions_post_type ON reactions(post_id, type);
```

#### Conversations

```sql
CREATE TABLE conversations (
    id                TEXT    PRIMARY KEY,  -- canonical ID
    protocol          TEXT    NOT NULL,
    type              TEXT    NOT NULL CHECK(type IN ('dm','group','channel','room')),
    name              TEXT,
    avatar_url        TEXT,
    topic             TEXT,
    last_message_id   TEXT    REFERENCES messages(id) ON DELETE SET NULL,
    last_message_at   TEXT,
    participant_count INTEGER DEFAULT 0,
    is_encrypted      INTEGER NOT NULL DEFAULT 0,
    unread_count      INTEGER NOT NULL DEFAULT 0,
    read_up_to        TEXT,                 -- message ID or timestamp
    metadata          TEXT,                 -- JSON blob
    created_at        TEXT    NOT NULL DEFAULT (datetime('now')),
    updated_at        TEXT    NOT NULL DEFAULT (datetime('now'))
);

CREATE INDEX idx_conversations_protocol ON conversations(protocol);
CREATE INDEX idx_conversations_updated ON conversations(last_message_at DESC);
```

#### Conversation Participants

```sql
CREATE TABLE conversation_participants (
    conversation_id TEXT NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
    profile_id      TEXT NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    joined_at       TEXT NOT NULL DEFAULT (datetime('now')),
    role            TEXT DEFAULT 'member' CHECK(role IN ('admin','moderator','member','invited')),
    membership      TEXT NOT NULL DEFAULT 'join' CHECK(membership IN ('join','leave','ban','invite')),
    PRIMARY KEY (conversation_id, profile_id)
);

CREATE INDEX idx_conv_participants_profile ON conversation_participants(profile_id);
```

#### Messages

```sql
CREATE TABLE messages (
    id              TEXT    PRIMARY KEY,  -- canonical ID
    protocol        TEXT    NOT NULL,
    conversation_id TEXT    NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
    sender_id       TEXT    NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    body            TEXT    NOT NULL,
    rich_body       TEXT,
    encrypted       INTEGER NOT NULL DEFAULT 0,
    encryption_alg  TEXT,                 -- 'nostr.nip04' | 'olm' | 'megolm' | 'omemo'
    sent_at         TEXT    NOT NULL,
    received_at     TEXT,
    reply_to_id     TEXT    REFERENCES messages(id) ON DELETE SET NULL,
    status          TEXT    NOT NULL DEFAULT 'sent'
                            CHECK(status IN ('sending','sent','delivered','read','failed')),
    edit_history    TEXT,                 -- JSON array of {body, editedAt}
    is_deleted      INTEGER NOT NULL DEFAULT 0,
    raw_data        TEXT,                 -- JSON blob
    inserted_at     TEXT    NOT NULL DEFAULT (datetime('now'))
);

CREATE INDEX idx_messages_conversation ON messages(conversation_id, sent_at DESC);
CREATE INDEX idx_messages_sender ON messages(sender_id);
CREATE INDEX idx_messages_status ON messages(status);
-- For showing latest message in conversation preview
CREATE INDEX idx_messages_conv_latest ON messages(conversation_id, inserted_at DESC);
```

#### Attachments

```sql
CREATE TABLE attachments (
    id               TEXT    PRIMARY KEY,  -- canonical ID
    source_type      TEXT    NOT NULL CHECK(source_type IN ('post','message','profile')),
    source_id        TEXT    NOT NULL,     -- post.id or message.id or profile.id
    type             TEXT    NOT NULL CHECK(type IN ('image','video','audio','document','gifv')),
    url              TEXT    NOT NULL,
    mime_type        TEXT,
    alt_text         TEXT,
    filename         TEXT,
    width            INTEGER,
    height           INTEGER,
    size_bytes       INTEGER,
    blurhash         TEXT,
    local_cache_path TEXT,
    thumbnail_url    TEXT,
    metadata         TEXT,                 -- JSON blob
    index            INTEGER NOT NULL DEFAULT 0,  -- ordinal position in parent
    inserted_at      TEXT    NOT NULL DEFAULT (datetime('now')),
    FOREIGN KEY (source_id) REFERENCES posts(id) ON DELETE CASCADE
        -- Note: For messages and profiles, foreign key cannot be enforced via single column
        -- since source_type discriminates the target table.
        -- Application-level enforcement required.
);

CREATE INDEX idx_attachments_source ON attachments(source_type, source_id);
CREATE INDEX idx_attachments_type ON attachments(type);
CREATE INDEX idx_attachments_mime ON attachments(mime_type);
-- For cache maintenance
CREATE INDEX idx_attachments_cached ON attachments(local_cache_path) WHERE local_cache_path IS NOT NULL;
```

### 2.3 Protocol Extension Tables

Each table stores protocol-native data that is not (or cannot be) fully normalized into the canonical model. These are used for re-publishing, bridging, debugging, and protocol-specific features.

#### Nostr Events

```sql
CREATE TABLE nostr_events (
    event_id        TEXT    PRIMARY KEY,  -- Nostr event ID (hex, 64 chars)
    pubkey          TEXT    NOT NULL,      -- Author pubkey (hex, 64 chars)
    kind            INTEGER NOT NULL,
    content         TEXT    NOT NULL,
    tags_json       TEXT    NOT NULL,      -- JSON array of tag arrays
    sig             TEXT    NOT NULL,      -- Signature (hex)
    created_at      INTEGER NOT NULL,     -- Unix timestamp
    relay_url       TEXT,                  -- Which relay this was received from
    is_ephemeral    INTEGER NOT NULL DEFAULT 0,
    is_deleted      INTEGER NOT NULL DEFAULT 0,
    canonical_post_id    TEXT REFERENCES posts(id) ON DELETE SET NULL,
    canonical_message_id TEXT REFERENCES messages(id) ON DELETE SET NULL,
    canonical_profile_id TEXT REFERENCES profiles(id) ON DELETE SET NULL,
    inserted_at     TEXT    NOT NULL DEFAULT (datetime('now'))
);

-- Identifying event kind for quick lookup
CREATE INDEX idx_nostr_events_kind ON nostr_events(kind);
CREATE INDEX idx_nostr_events_pubkey ON nostr_events(pubkey);
CREATE INDEX idx_nostr_events_created ON nostr_events(created_at DESC);
CREATE INDEX idx_nostr_events_relay ON nostr_events(relay_url);
```

#### AT Protocol Records

```sql
CREATE TABLE atproto_records (
    uri             TEXT    PRIMARY KEY,  -- AT URI: at://did/collection/rkey
    did             TEXT    NOT NULL,      -- Author DID
    collection      TEXT    NOT NULL,      -- Record type (e.g., app.bsky.feed.post)
    rkey            TEXT    NOT NULL,
    record_json     TEXT    NOT NULL,      -- Full record as JSON
    cid             TEXT    NOT NULL,      -- Content hash
    created_at      TEXT    NOT NULL,      -- ISO 8601 from record
    indexed_at      TEXT    NOT NULL,      -- When the relay indexed it
    canonical_post_id    TEXT REFERENCES posts(id) ON DELETE SET NULL,
    canonical_profile_id TEXT REFERENCES profiles(id) ON DELETE SET NULL,
    inserted_at     TEXT    NOT NULL DEFAULT (datetime('now'))
);

CREATE INDEX idx_atproto_collection ON atproto_records(collection);
CREATE INDEX idx_atproto_did ON atproto_records(did);
CREATE INDEX idx_atproto_created ON atproto_records(created_at DESC);
CREATE UNIQUE INDEX idx_atproto_did_rkey ON atproto_records(did, collection, rkey);
```

#### ActivityPub Objects

```sql
CREATE TABLE activitypub_objects (
    uri             TEXT    PRIMARY KEY,  -- ActivityPub object ID (URL or URI)
    object_type     TEXT    NOT NULL,      -- 'Note', 'Article', 'Image', 'Video', etc.
    object_json     TEXT    NOT NULL,      -- Full ActivityPub object JSON-LD
    attributed_to   TEXT,                  -- Actor URL
    published       TEXT,                  -- ISO 8601
    canonical_post_id    TEXT REFERENCES posts(id) ON DELETE SET NULL,
    canonical_profile_id TEXT REFERENCES profiles(id) ON DELETE SET NULL,
    inserted_at     TEXT    NOT NULL DEFAULT (datetime('now'))
);

CREATE INDEX idx_ap_objects_type ON activitypub_objects(object_type);
CREATE INDEX idx_ap_objects_actor ON activitypub_objects(attributed_to);
CREATE INDEX idx_ap_objects_published ON activitypub_objects(published DESC);
```

#### Matrix Events

```sql
CREATE TABLE matrix_events (
    event_id        TEXT    NOT NULL,      -- Matrix event ID ($event_id:server)
    room_id         TEXT    NOT NULL,      -- Room ID (!room_id:server)
    event_type      TEXT    NOT NULL,      -- 'm.room.message', 'm.reaction', etc.
    state_key       TEXT,                  -- Non-null for state events
    content_json    TEXT    NOT NULL,      -- Event content as JSON
    origin_server_ts INTEGER NOT NULL,     -- Unix timestamp ms
    sender          TEXT    NOT NULL,      -- MXID (@user:server)
    unsigned_json   TEXT,                  -- Unsigned data
    is_state        INTEGER NOT NULL DEFAULT 0,
    is_redacted     INTEGER NOT NULL DEFAULT 0,
    canonical_post_id    TEXT REFERENCES posts(id) ON DELETE SET NULL,
    canonical_message_id TEXT REFERENCES messages(id) ON DELETE SET NULL,
    inserted_at     TEXT    NOT NULL DEFAULT (datetime('now')),
    PRIMARY KEY (event_id, room_id)
);

CREATE INDEX idx_matrix_events_room ON matrix_events(room_id, origin_server_ts DESC);
CREATE INDEX idx_matrix_events_type ON matrix_events(event_type);
CREATE INDEX idx_matrix_events_sender ON matrix_events(sender);
```

#### IRC Messages

```sql
CREATE TABLE irc_messages (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    server          TEXT    NOT NULL,      -- IRC server hostname
    channel         TEXT    NOT NULL,      -- #channel or nick for queries
    command         TEXT    NOT NULL,      -- PRIVMSG, NOTICE, etc.
    nick            TEXT    NOT NULL,      -- Source nick
    user            TEXT,                  -- Source username
    host            TEXT,                  -- Source hostname
    text            TEXT,                  -- Message body
    tags_json       TEXT,                  -- IRCv3 message tags
    timestamp       INTEGER NOT NULL,     -- Unix timestamp
    is_action       INTEGER NOT NULL DEFAULT 0,
    canonical_message_id TEXT REFERENCES messages(id) ON DELETE SET NULL,
    inserted_at     TEXT    NOT NULL DEFAULT (datetime('now'))
);

CREATE INDEX idx_irc_server_channel ON irc_messages(server, channel, timestamp DESC);
CREATE INDEX idx_irc_timestamp ON irc_messages(timestamp);
```

#### XMPP Stanzas

```sql
CREATE TABLE xmpp_stanzas (
    id              TEXT    PRIMARY KEY,  -- Stanza ID or generated
    jid             TEXT    NOT NULL,      -- Full JID of sender
    bare_jid        TEXT    NOT NULL,      -- user@domain
    resource        TEXT,                  -- Resource identifier
    stanza_type     TEXT    NOT NULL,      -- 'message', 'presence', 'iq'
    payload_xml     TEXT    NOT NULL,      -- Full stanza XML
    body            TEXT,                  -- Extracted <body> text
    thread          TEXT,                  -- XEP-0047 thread ID
    timestamp       TEXT    NOT NULL,      -- ISO 8601 (from delay stamp or received)
    is_delayed      INTEGER NOT NULL DEFAULT 0,
    canonical_message_id TEXT REFERENCES messages(id) ON DELETE SET NULL,
    inserted_at     TEXT    NOT NULL DEFAULT (datetime('now'))
);

CREATE INDEX idx_xmpp_jid ON xmpp_stanzas(bare_jid, timestamp DESC);
CREATE INDEX idx_xmpp_thread ON xmpp_stanzas(thread);
CREATE INDEX idx_xmpp_type ON xmpp_stanzas(stanza_type);
```

### 2.4 Social Graph Tables

#### Follows

```sql
CREATE TABLE follows (
    follower_id  TEXT NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    following_id TEXT NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    protocol     TEXT NOT NULL,          -- Which protocol this follow relationship came from
    lists        TEXT,                   -- Comma-separated list IDs (Mastodon feature)
    followed_at  TEXT NOT NULL DEFAULT (datetime('now')),
    is_muted     INTEGER NOT NULL DEFAULT 0,
    PRIMARY KEY (follower_id, following_id, protocol)
);

CREATE INDEX idx_follows_follower ON follows(follower_id);
CREATE INDEX idx_follows_following ON follows(following_id);
CREATE INDEX idx_follows_protocol ON follows(protocol);
```

#### Lists

```sql
CREATE TABLE lists (
    id          TEXT    PRIMARY KEY,     -- canonical ID
    protocol    TEXT    NOT NULL,
    name        TEXT    NOT NULL,
    description TEXT,
    is_private  INTEGER NOT NULL DEFAULT 1,
    created_at  TEXT    NOT NULL DEFAULT (datetime('now')),
    updated_at  TEXT    NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE list_members (
    list_id    TEXT NOT NULL REFERENCES lists(id) ON DELETE CASCADE,
    profile_id TEXT NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    added_at   TEXT NOT NULL DEFAULT (datetime('now')),
    PRIMARY KEY (list_id, profile_id)
);

CREATE INDEX idx_list_members_profile ON list_members(profile_id);
```

#### Muted & Blocked Accounts

```sql
CREATE TABLE muted_accounts (
    profile_id   TEXT NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    muted_by_id  TEXT NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    protocol     TEXT NOT NULL,
    reason       TEXT,
    created_at   TEXT NOT NULL DEFAULT (datetime('now')),
    expires_at   TEXT,                  -- NULL = permanent
    PRIMARY KEY (profile_id, muted_by_id, protocol)
);

CREATE TABLE blocked_accounts (
    profile_id     TEXT NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    blocked_by_id  TEXT NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    protocol       TEXT NOT NULL,
    reason         TEXT,
    created_at     TEXT NOT NULL DEFAULT (datetime('now')),
    PRIMARY KEY (profile_id, blocked_by_id, protocol)
);
```

#### Identity Links (Cross-Protocol Profile Linking)

```sql
CREATE TABLE identity_links (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    profile_a   TEXT NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    profile_b   TEXT NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    confidence  REAL    NOT NULL DEFAULT 1.0 CHECK(confidence >= 0.0 AND confidence <= 1.0),
    method      TEXT    NOT NULL CHECK(method IN (
                    'manual',           -- User explicitly linked these
                    'same_handle',      -- Same handle name across protocols (e.g., @user)
                    'nip05_domain',     -- Nostr NIP-05 + Mastodon verified domain match
                    'did_web',          -- AT Proto did:web + website ownership
                    'algorithmic'       -- Content/style similarity matching
                )),
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    UNIQUE(profile_a, profile_b),
    CHECK(profile_a < profile_b)         -- Prevent (a,b) and (b,a) duplicates
);

CREATE INDEX idx_identity_links_profile ON identity_links(profile_a, profile_b);
```

### 2.5 Sync Metadata

#### Sync State Per Protocol Stream

```sql
CREATE TABLE sync_state (
    protocol      TEXT NOT NULL,
    stream        TEXT NOT NULL,     -- 'timeline', 'notifications', 'dms', 'firehose', 'home',
                                    -- 'global', 'room:!roomid:server', 'channel:server/#chan'
    cursor        TEXT,              -- Protocol-specific cursor value
    extra_data    TEXT,              -- JSON blob for additional metadata (sub-cursors, filters)
    last_synced_at TEXT NOT NULL DEFAULT (datetime('now')),
    status        TEXT NOT NULL DEFAULT 'idle'
                  CHECK(status IN ('idle','syncing','error','backfilling')),
    error         TEXT,
    retry_count   INTEGER NOT NULL DEFAULT 0,
    PRIMARY KEY (protocol, stream)
);

-- Seed initial sync states
INSERT OR IGNORE INTO sync_state (protocol, stream, status) VALUES
    ('atproto',     'timeline',       'idle'),
    ('atproto',     'notifications',  'idle'),
    ('atproto',     'firehose',       'idle'),
    ('activitypub', 'timeline',       'idle'),
    ('activitypub', 'notifications',  'idle'),
    ('activitypub', 'dms',            'idle'),
    ('nostr',       'home',           'idle'),
    ('nostr',       'global',         'idle'),
    ('nostr',       'notifications',  'idle'),
    ('matrix',      'sync',           'idle'),
    ('irc',         'scrollback',     'idle'),
    ('xmpp',        'archive',        'idle');
```

#### Relay/Server Connection State

```sql
CREATE TABLE connection_config (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    protocol    TEXT NOT NULL,
    server_url  TEXT NOT NULL,           -- Relay URL, server hostname, etc.
    is_primary  INTEGER NOT NULL DEFAULT 0,
    enabled     INTEGER NOT NULL DEFAULT 1,
    priority    INTEGER NOT NULL DEFAULT 0,
    credential  TEXT,                    -- Encrypted token/api key
    options     TEXT,                    -- JSON: connection params, capabilities
    last_connected TEXT,
    last_error     TEXT,
    created_at     TEXT NOT NULL DEFAULT (datetime('now')),
    UNIQUE(protocol, server_url)
);

CREATE INDEX idx_connection_protocol ON connection_config(protocol, enabled);
```

### 2.6 Queue Tables

#### Outbox (Pending Posts)

```sql
CREATE TABLE outbox (
    id                  TEXT    PRIMARY KEY,
    protocol            TEXT    NOT NULL,
    action              TEXT    NOT NULL CHECK(action IN ('create_post','delete_post','update_post',
                                                         'create_reaction','create_message')),
    canonical_target_id TEXT,              -- posts.id or messages.id for delete/update
    serialized_payload  TEXT    NOT NULL,  -- Protocol-specific serialized payload as JSON
    status              TEXT    NOT NULL DEFAULT 'pending'
                        CHECK(status IN ('pending','sending','sent','failed','cancelled')),
    retry_count         INTEGER NOT NULL DEFAULT 0,
    max_retries         INTEGER NOT NULL DEFAULT 5,
    next_retry_at       TEXT,              -- For exponential backoff
    error               TEXT,
    created_at          TEXT    NOT NULL DEFAULT (datetime('now')),
    updated_at          TEXT    NOT NULL DEFAULT (datetime('now')),
    -- Index for prioritized dispatch
    priority            INTEGER NOT NULL DEFAULT 0
);

-- Queue dispatch query index
CREATE INDEX idx_outbox_dispatch ON outbox(status, priority DESC, created_at ASC)
    WHERE status = 'pending' OR status = 'failed';

-- Cleanup index
CREATE INDEX idx_outbox_old ON outbox(created_at) WHERE status IN ('sent','cancelled');
```

#### Pending Interactions (Likes, Follows, etc.)

```sql
CREATE TABLE pending_interactions (
    id                  TEXT    PRIMARY KEY,
    protocol            TEXT    NOT NULL,
    interaction_type    TEXT    NOT NULL CHECK(interaction_type IN (
                            'like','unlike','repost','unrepost','bookmark','unbookmark',
                            'follow','unfollow','block','unblock','mute','unmute'
                        )),
    target_id           TEXT    NOT NULL,  -- Profile ID or Post ID depending on type
    actor_id            TEXT    NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    serialized_payload  TEXT,              -- Protocol-specific payload as JSON
    status              TEXT    NOT NULL DEFAULT 'pending'
                        CHECK(status IN ('pending','sending','sent','failed','cancelled')),
    retry_count         INTEGER NOT NULL DEFAULT 0,
    max_retries         INTEGER NOT NULL DEFAULT 5,
    error               TEXT,
    created_at          TEXT    NOT NULL DEFAULT (datetime('now')),
    updated_at          TEXT    NOT NULL DEFAULT (datetime('now'))
);

CREATE INDEX idx_pending_interactions_dispatch ON pending_interactions(status, created_at ASC)
    WHERE status = 'pending';
```

### 2.7 App State

#### Drafts

```sql
CREATE TABLE drafts (
    id              TEXT    PRIMARY KEY,
    protocol        TEXT    NOT NULL DEFAULT 'atproto',  -- Target protocol
    content         TEXT    NOT NULL DEFAULT '',
    rich_content    TEXT,
    reply_to_id     TEXT    REFERENCES posts(id) ON DELETE SET NULL,
    mentions_json   TEXT,                 -- JSON array of profile IDs
    tags_json       TEXT,                 -- JSON array of tag strings
    attachments_json TEXT,                -- JSON array of local attachment metadata
    visibility      TEXT    DEFAULT 'public',
    created_at      TEXT    NOT NULL DEFAULT (datetime('now')),
    updated_at      TEXT    NOT NULL DEFAULT (datetime('now'))
);
```

#### Read Markers

```sql
CREATE TABLE read_markers (
    marker_id       TEXT    PRIMARY KEY,  -- E.g., 'timeline:home', 'conversation:{id}'
    last_read_id    TEXT,                 -- Canonical post/message ID
    last_read_at    TEXT    NOT NULL DEFAULT (datetime('now')),
    updated_at      TEXT    NOT NULL DEFAULT (datetime('now'))
);
```

#### Bookmarks (User's Own Bookmarks)

```sql
CREATE TABLE bookmarks (
    post_id     TEXT NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    profile_id  TEXT NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    note        TEXT,                    -- Optional user note
    created_at  TEXT NOT NULL DEFAULT (datetime('now')),
    PRIMARY KEY (post_id, profile_id)
);

CREATE INDEX idx_bookmarks_profile ON bookmarks(profile_id, created_at DESC);
```

#### Search History

```sql
CREATE TABLE search_history (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    query       TEXT    NOT NULL,
    filters     TEXT,                    -- JSON: protocol, author, date range, etc.
    result_ids  TEXT,                    -- JSON array of canonical IDs (pinned results)
    created_at  TEXT    NOT NULL DEFAULT (datetime('now'))
);

CREATE INDEX idx_search_history_time ON search_history(created_at DESC);
-- Limit search history retention (application-level cleanup)
```

#### App Settings (Key-Value Store)

```sql
CREATE TABLE app_settings (
    key         TEXT    PRIMARY KEY,
    value       TEXT    NOT NULL,
    updated_at  TEXT    NOT NULL DEFAULT (datetime('now'))
);

-- Seed defaults
INSERT OR IGNORE INTO app_settings (key, value) VALUES
    ('theme', 'system'),
    ('language', 'en'),
    ('default_visibility', 'public'),
    ('auto_cache_media', 'true'),
    ('max_cache_size_mb', '1024'),
    ('sync_on_wifi_only', 'false'),
    ('notifications_enabled', 'true'),
    ('experimental_firehose', 'false'),
    ('last_active_protocol', 'atproto');
```

### 2.8 FTS5 Virtual Tables

#### Content Full-Text Search

```sql
-- Posts + Messages unified search
-- Note: FTS5 with content_sync approach cannot use TEXT rowid, so we use
-- external content approach via the rowid of posts and messages tables.
-- Since both tables have TEXT primary keys, we create a separate FTS index
-- with content_id as a UNINDEXED column and populate via triggers.

CREATE VIRTUAL TABLE content_fts USING fts5(
    content_id UNINDEXED,       -- Canonical ID for lookups
    source_type UNINDEXED,      -- 'post' or 'message'
    text_content,                -- The searchable text
    author_display_name,         -- Denormalized for inclusion in results
    language UNINDEXED,          -- BCP-47 language tag
    tokenize='unicode61 remove_diacritics 2'
    -- Note: For CJK support, replace tokenizer with 'unicode61 tokenchars'
    -- or use ICU tokenizer if available in SQLite build.
);
```

Triggers to keep FTS in sync:

```sql
-- Posts: Insert
CREATE TRIGGER after_post_insert_fts AFTER INSERT ON posts BEGIN
    INSERT INTO content_fts (content_id, source_type, text_content, language)
    VALUES (
        NEW.id,
        'post',
        COALESCE(NEW.content, ''),
        COALESCE(NEW.language, 'und')
    );
END;

-- Posts: Delete
CREATE TRIGGER after_post_delete_fts AFTER DELETE ON posts BEGIN
    INSERT INTO content_fts (content_fts, content_id, source_type, text_content, language)
    VALUES ('delete', OLD.id, 'post', '', '');
END;

-- Posts: Update (delete old, insert new)
CREATE TRIGGER after_post_update_fts AFTER UPDATE ON posts BEGIN
    -- Remove old entry
    INSERT INTO content_fts (content_fts, content_id, source_type, text_content, language)
    VALUES ('delete', OLD.id, 'post', '', '');
    -- Insert new entry if content changed
    INSERT INTO content_fts (content_id, source_type, text_content, language)
    VALUES (
        NEW.id,
        'post',
        COALESCE(NEW.content, ''),
        COALESCE(NEW.language, 'und')
    );
END;

-- Messages: Insert
CREATE TRIGGER after_message_insert_fts AFTER INSERT ON messages BEGIN
    INSERT INTO content_fts (content_id, source_type, text_content, language)
    VALUES (
        NEW.id,
        'message',
        COALESCE(NEW.body, ''),
        'und'
    );
END;

-- Messages: Delete
CREATE TRIGGER after_message_delete_fts AFTER DELETE ON messages BEGIN
    INSERT INTO content_fts (content_fts, content_id, source_type, text_content, language)
    VALUES ('delete', OLD.id, 'message', '', '');
END;

-- Messages: Update
CREATE TRIGGER after_message_update_fts AFTER UPDATE ON messages BEGIN
    INSERT INTO content_fts (content_fts, content_id, source_type, text_content, language)
    VALUES ('delete', OLD.id, 'message', '', '');
    INSERT INTO content_fts (content_id, source_type, text_content, language)
    VALUES (
        NEW.id,
        'message',
        COALESCE(NEW.body, ''),
        'und'
    );
END;
```

#### Profiles Full-Text Search

```sql
CREATE VIRTUAL TABLE profiles_fts USING fts5(
    profile_id UNINDEXED,
    display_name,
    handle,
    bio,
    tokenize='unicode61 remove_diacritics 2'
);

CREATE TRIGGER after_profile_insert_fts AFTER INSERT ON profiles BEGIN
    INSERT INTO profiles_fts (profile_id, display_name, handle, bio)
    VALUES (
        NEW.id,
        COALESCE(NEW.display_name, ''),
        COALESCE(NEW.handle, ''),
        COALESCE(NEW.bio, '')
    );
END;

CREATE TRIGGER after_profile_delete_fts AFTER DELETE ON profiles BEGIN
    INSERT INTO profiles_fts (profiles_fts, profile_id, display_name, handle, bio)
    VALUES ('delete', OLD.id, '', '', '');
END;

CREATE TRIGGER after_profile_update_fts AFTER UPDATE ON profiles BEGIN
    INSERT INTO profiles_fts (profiles_fts, profile_id, display_name, handle, bio)
    VALUES ('delete', OLD.id, '', '', '');
    INSERT INTO profiles_fts (profile_id, display_name, handle, bio)
    VALUES (
        NEW.id,
        COALESCE(NEW.display_name, ''),
        COALESCE(NEW.handle, ''),
        COALESCE(NEW.bio, '')
    );
END;
```

#### Tags/Entities Index

```sql
CREATE VIRTUAL TABLE tags_fts USING fts5(
    tag,
    post_id UNINDEXED,
    tokenize='ascii'
);

CREATE TRIGGER after_post_tag_insert_fts AFTER INSERT ON post_tags BEGIN
    INSERT INTO tags_fts (tag, post_id) VALUES (NEW.tag, NEW.post_id);
END;

CREATE TRIGGER after_post_tag_delete_fts AFTER DELETE ON post_tags BEGIN
    INSERT INTO tags_fts (tags_fts, tag, post_id) VALUES ('delete', OLD.tag, OLD.post_id);
END;
```

### 2.9 Additional Indexes for Query Performance

```sql
-- Feed queries: home timeline by protocol
CREATE INDEX idx_feed_home_atproto ON posts(created_at DESC, author_id)
    WHERE protocol = 'atproto' AND is_deleted = 0;
CREATE INDEX idx_feed_home_activitypub ON posts(created_at DESC, author_id)
    WHERE protocol = 'activitypub' AND is_deleted = 0;
CREATE INDEX idx_feed_home_nostr ON posts(created_at DESC, author_id)
    WHERE protocol = 'nostr' AND is_deleted = 0;

-- Cross-protocol timeline queries
CREATE INDEX idx_posts_cross_protocol ON posts(protocol, created_at DESC)
    WHERE is_deleted = 0;

-- Mention lookup
CREATE INDEX idx_post_mentions_lookup ON post_mentions(post_id);

-- Conversation last message fast lookup
CREATE INDEX idx_messages_conv_latest_only ON messages(conversation_id, sent_at DESC)
    WHERE is_deleted = 0;

-- Reactions for display
CREATE INDEX idx_reactions_display ON reactions(post_id, type, created_at DESC);

-- Sync state next candidate
CREATE INDEX idx_sync_state_stale ON sync_state(status, last_synced_at)
    WHERE status = 'idle' OR status = 'error';
```

---

## 3. Protocol Mapping Rules

### 3.1 ActivityPub (Mastodon API)

The Mastodon REST API provides JSON responses that map directly to our canonical model.

```
FUNCTION mastodon_status_to_unified_post(status: MastodonAPI.Status) -> UnifiedPost:

    id = "activitypub:" + (status.uri ?? status.url)
    protocol = "activitypub"

    author = mastodon_account_to_unified_profile(status.account)

    -- Strip HTML tags for plain text
    content = strip_html(status.content)
    rich_content = sanitize_html(status.content)

    -- Timestamp: already ISO 8601, ensure UTC
    created_at = normalize_iso8601(status.created_at)

    -- Threading: resolve to canonical post IDs
    reply_to_id = status.in_reply_to_id
        ? "activitypub:" + status.in_reply_to_id
        : null
    root_post_id = status.in_reply_to_account_id
        ? resolve_root_post(status.in_reply_to_id)  -- Walk up chain, cache result
        : null

    -- Visibility: direct mapping
    visibility = status.visibility  -- 'public'|'unlisted'|'private'|'direct'

    -- Language
    language = status.language ?? status.content_language

    -- Media attachments
    media = status.media_attachments.map(mastodon_attachment_to_unified)

    -- Tags (hashtags)
    tags = status.tags
        .filter(t => t.type == "hashtag")
        .map(t => t.name.toLowerCase())

    -- Mentions
    mentions = status.mentions.map(m => "activitypub:" + m.url)

    -- Source (keep for re-publish, bridging)
    source = status  -- Full original object

    RETURN UnifiedPost { ... }

FUNCTION mastodon_account_to_unified_profile(acct: MastodonAPI.Account) -> UnifiedProfile:
    id = "activitypub:" + acct.url
    protocol = "activitypub"
    handle = "@" + acct.acct  -- "@user@server" format
    display_name = acct.display_name
    avatar_url = acct.avatar
    banner_url = acct.header
    bio = strip_html(acct.note)
    followers_count = acct.followers_count
    following_count = acct.following_count
    posts_count = acct.statuses_count
    verification_method = acct.verified ? "website_link" : "none"
    verification_value = acct.fields.find(f => f.verified_at)?.value
    created_at = acct.created_at
    updated_at = now()
    is_stub = false
    RETURN UnifiedProfile { ... }

FUNCTION mastodon_notification_to_unified_react(notif: MastodonAPI.Notification) -> UnifiedReaction:
    -- Type mapping
    TYPE_MAP = {
        "favourite":    "like",
        "reblog":       "repost",
        "follow":       null,   -- Not a reaction, handled as social graph
        "mention":      null,   -- Not a reaction
        "poll":         null,
        "status":       null,   -- Not a reaction from Notification perspective
    }

    if TYPE_MAP[notif.type] == null:
        RETURN null  -- Not a reaction

    id = "activitypub:" + notif.id
    post_id = "activitypub:" + notif.status?.uri
    author_id = "activitypub:" + notif.account.url
    type = TYPE_MAP[notif.type]
    emoji = type == "like" ? "❤️" : null  -- Mastodon doesn't expose specific emoji
    created_at = notif.created_at
    protocol = "activitypub"

    RETURN UnifiedReaction { ... }

--- Reverse (Publish) Mappings ---

FUNCTION unified_post_to_mastodon_api(post: UnifiedPost) -> MastodonAPI.StatusParams:
    params = {
        status: post.content,
        visibility: post.visibility,  -- Must match Mastodon enum
        language: post.language,
    }

    -- Media IDs: must have been uploaded first via POST /api/v1/media
    if post.media?.length > 0:
        params.media_ids = post.media.map(m => upload_and_get_mastodon_id(m))

    -- Mentions: Mastodon parses @mentions from content text,
    -- but we may need to inject them for protocols that don't embed them
    if post.mentions?.length > 0:
        inject_mentions_into_content(params, post.mentions)

    -- Tags: Mastodon parses #hashtag from content

    -- Reply context
    if post.replyToId:
        params.in_reply_to_id = extract_protocol_id(post.replyToId)
        -- Local mastodon status ID, not canonical ID

    return params

FUNCTION unified_profile_to_mastodon_api(profile: UnifiedProfile) -> MastodonAPI.CredentialsUpdateParams:
    return {
        display_name: profile.displayName,
        note: profile.bio,
        avatar: profile.avatarUrl,   -- Must be a valid URL, Mastodon fetches it
        header: profile.bannerUrl,
        -- Mastodon handles fields/verification differently
    }
```

### 3.2 AT Protocol (Bluesky)

```
FUNCTION atproto_post_to_unified(record: app.bsky.feed.post, uri: string, cid: string) -> UnifiedPost:

    id = "atproto:" + uri           -- at://did/app.bsky.feed.post/rkey
    protocol = "atproto"

    author_id = "atproto:" + record.did  -- Already in profile table

    -- Content: plain text from record.text
    content = record.text

    -- Rich text: reconstruct from facets
    rich_content = render_atproto_facets(record.text, record.facets ?? [])

    -- Created at: ISO 8601 from record
    created_at = normalize_iso8601(record.createdAt)

    -- Reply threading
    if record.reply:
        reply_to_id = "atproto:" + record.reply.parent.uri
        root_post_id = "atproto:" + record.reply.root.uri

    -- Media embeds
    media = []
    if record.embed?.$type == "app.bsky.embed.images":
        media = record.embed.images.map(img => {
            id = "atproto:" + img.image.ref.link    -- CID
            type = "image"
            url = build_bluesky_image_url(record.did, img.image.ref.link)
            mime_type = img.image.mimeType ?? "image/jpeg"
            alt_text = img.alt
            width = img.image.width
            height = img.image.height
            blurhash = detect_blurhash_from_embed(img)
            -- Bluesky uses CID-based blob store; generate URL from DID + CID
        })
    else if record.embed?.$type == "app.bsky.embed.external":
        media = [{
            id = "atproto:" + record.embed.external.uri
            type = detect_external_media_type(record.embed.external)
            url = record.embed.external.uri
            mime_type = "text/html"  -- External link
            alt_text = record.embed.external.title
            thumbnail_url = record.embed.external.thumb
        }]
    else if record.embed?.$type == "app.bsky.embed.video":
        media = [{
            id = "atproto:" + record.embed.video.ref.link
            type = "video"
            url = build_bluesky_video_url(record.did, record.embed.video.ref.link)
            mime_type = "video/mp4"
            alt_text = record.embed.video.alt ?? ""
            width = record.embed.video.width
            height = record.embed.video.height
        }]

    -- Tags from facets
    tags = []
    mentions = []
    if record.facets:
        for facet in record.facets:
            for feature in facet.features:
                if feature.$type == "app.bsky.richtext.facet#tag":
                    tags.add(feature.tag.lower())
                if feature.$type == "app.bsky.richtext.facet#mention":
                    mentions.add("atproto:" + feature.did)

    -- Visibility: AT Protocol is public-only currently
    visibility = "public"

    -- Language: record.langs is an array, take first
    language = record.langs?[0]

    source = { uri, cid, record }

    RETURN UnifiedPost { ... }

FUNCTION atproto_profile_to_unified(record: app.bsky.actor.profile, did: string, handle: string) -> UnifiedProfile:
    id = "atproto:" + did
    protocol = "atproto"
    handle = handle                  -- e.g., "user.bsky.social"
    display_name = record.displayName
    avatar_url = build_bluesky_image_url(did, record.avatar?.ref.link) if record.avatar else null
    banner_url = build_bluesky_image_url(did, record.banner?.ref.link) if record.banner else null
    bio = record.description
    followers_count = null           -- Must be fetched separately
    following_count = null
    posts_count = null
    verification_method = "did_plc"  -- Bluesky handles are verified via PLC directory
    verification_value = handle
    created_at = null                -- Not available in profile record
    updated_at = normalize_iso8601(record.createdAt) if record.createdAt else now()
    is_stub = false
    metadata = { did: did }
    RETURN UnifiedProfile { ... }

FUNCTION atproto_like_to_unified(record: app.bsky.feed.like, uri: string) -> UnifiedReaction:
    id = "atproto:" + uri
    post_id = "atproto:" + record.subject.uri   -- The liked post URI
    author_id = "atproto:" + extract_did_from_uri(uri)
    type = "like"
    emoji = null
    protocol = "atproto"
    created_at = record.createdAt
    source = { uri, record }
    RETURN UnifiedReaction { ... }

--- Reverse (Publish) Mappings ---

FUNCTION unified_post_to_atproto(post: UnifiedPost) -> app.bsky.feed.post.CreateRecord:
    record = {
        $type: "app.bsky.feed.post",
        text: post.content,
        createdAt: new Date().toISOString(),
    }

    -- Facets from mentions and tags
    facets = []
    if post.mentions?.length > 0:
        for mention_id in post.mentions:
            did = extract_did_from_canonical(mention_id)
            byte_range = find_mention_offset(post.content, did)
            if byte_range:
                facets.push({
                    index: { byteStart: byte_range.start, byteEnd: byte_range.end },
                    features: [{ $type: "app.bsky.richtext.facet#mention", did: did }]
                })

    if post.tags?.length > 0:
        for tag in post.tags:
            byte_range = find_tag_offset(post.content, tag)
            facets.push({
                index: { byteStart: byte_range.start, byteEnd: byte_range.end },
                features: [{ $type: "app.bsky.richtext.facet#tag", tag: tag }]
            })

    if facets.length > 0:
        record.facets = facets

    -- Reply threading
    if post.replyToId:
        record.reply = {
            parent: { uri: extract_at_uri(post.replyToId), cid: null },  -- CID resolved before send
            root: { uri: extract_at_uri(post.rootPostId ?? post.replyToId), cid: null }
        }

    -- Media
    if post.media?.length > 0:
        -- Upload images via com.atproto.repo.uploadBlob first
        image_embed = {
            $type: "app.bsky.embed.images",
            images: post.media.filter(m => m.type == "image").map(m => ({
                alt: m.altText ?? "",
                image: { $type: "blob", ref: { link: uploaded_cid }, mimeType: m.mimeType }
            }))
        }
        if post.media.some(m => m.type == "video"):
            video_embed = {
                $type: "app.bsky.embed.video",
                video: { $type: "blob", ref: { link: uploaded_cid }, mimeType: "video/mp4" },
                alt: post.media.find(m => m.type == "video")?.altText ?? ""
            }
        record.embed = image_embed  -- or composite embed

    return {
        repo: extract_did_from_canonical(post.authorId),
        collection: "app.bsky.feed.post",
        record: record
    }
```

### 3.3 Nostr

```
FUNCTION nostr_event_to_unified(event: NostrEvent, relay: string) -> UnifiedPost:

    id = "nostr:" + event.id
    protocol = "nostr"
    author_id = "nostr:" + event.pubkey

    content = event.content  -- Plain text (NIP-28 may contain JSON for markdown)
    rich_content = null      -- Nostr is plain text; NIP-23 (long-form) uses markdown

    created_at = unix_to_iso8601(event.created_at)

    -- Extract e tags for threading
    e_tags = event.tags.filter(t => t[0] == "e")
    reply_to_id = e_tags.find(t => t[3] == "reply")?.[1]
                   ?? e_tags.find(t => !t[3] || t[3] == "" || t[3] == "mention")?.[1]
                   ?? e_tags[0]?.[1]  -- Fallback: first e tag
    if reply_to_id:
        reply_to_id = "nostr:" + reply_to_id

    root_post_id = e_tags.find(t => t[3] == "root")?.[1]
    if root_post_id:
        root_post_id = "nostr:" + root_post_id

    -- Tags: t tags = hashtags; p tags = mentions/participants
    tags = event.tags
        .filter(t => t[0] == "t")
        .map(t => t[1].toLowerCase())

    mentions = event.tags
        .filter(t => t[0] == "p")
        .map(t => "nostr:" + t[1])

    -- Nostr has no native visibility; assume public
    visibility = "public"

    -- Nostr has no native language field (NIP-34 proposed but not widespread)
    language = detect_language(event.content)  -- Heuristic

    source = { event, relay }

    -- Media references in content: look for NIP-19 bech32 URLs (nmedia, etc.)
    -- or embedded image URLs in content
    media = extract_media_from_content(event.content, event.tags)

    RETURN UnifiedPost { ... }

FUNCTION nostr_kind0_to_unified(event: NostrEvent) -> UnifiedProfile:
    -- kind:0 (metadata) event
    meta = JSON.parse(event.content)

    id = "nostr:" + event.pubkey
    protocol = "nostr"
    handle = meta.nip05 ?? "npub1" + bech32_encode(event.pubkey)
    display_name = meta.display_name ?? meta.name
    avatar_url = meta.picture
    banner_url = meta.banner
    bio = meta.about
    followers_count = null   -- Not in metadata
    following_count = null
    posts_count = null

    -- NIP-05 verification
    if meta.nip05:
        verification_method = "nip05"
        verification_value = meta.nip05  -- "user@domain"
    else:
        verification_method = "none"

    -- Note: Nostr profiles are inherently stubs until we fetch kind:0
    is_stub = false
    metadata = { pubkey: event.pubkey, lud06: meta.lud06, lud16: meta.lud16 }

    created_at = unix_to_iso8601(event.created_at)
    updated_at = now()

    RETURN UnifiedProfile { ... }

FUNCTION nostr_kind3_to_follows(event: NostrEvent) -> Follow[]:
    -- kind:3 (contact list) event
    -- Each p tag is a followed pubkey
    my_pubkey = event.pubkey
    my_profile_id = "nostr:" + my_pubkey

    follows = event.tags
        .filter(t => t[0] == "p")
        .map(t => {
            follower_id: my_profile_id,
            following_id: "nostr:" + t[1],
            protocol: "nostr",
            followed_at: unix_to_iso8601(event.created_at),
            lists: t[2] ? [t[2]] : null,  -- Optional relay hint as list context
            is_muted: false
        })

    RETURN follows

FUNCTION nostr_kind7_to_unified(event: NostrEvent) -> UnifiedReaction:
    -- kind:7 (reaction) event
    -- event.content is typically "+" (like), "-" (dislike), or an emoji
    -- e tag references the reacted event
    e_tag = event.tags.find(t => t[0] == "e")
    if not e_tag:
        return null

    id = "nostr:" + event.id
    post_id = "nostr:" + e_tag[1]
    author_id = "nostr:" + event.pubkey

    content = event.content
    is_like = content == "+" || content == "❤️" || content == "👍"

    type = "like"
    emoji = is_like ? content : null

    protocol = "nostr"
    created_at = unix_to_iso8601(event.created_at)
    source = { event }

    RETURN UnifiedReaction { ... }

FUNCTION nostr_kind6_to_unified(event: NostrEvent) -> UnifiedReaction:
    -- kind:6 (repost) event
    -- Often uses q tag or e tag for the original event
    e_tag = event.tags.find(t => t[0] == "e" || t[0] == "q")
    if not e_tag:
        return null

    id = "nostr:" + event.id
    post_id = "nostr:" + e_tag[1]
    author_id = "nostr:" + event.pubkey
    type = "repost"
    protocol = "nostr"
    created_at = unix_to_iso8601(event.created_at)
    source = { event }

    RETURN UnifiedReaction { ... }

FUNCTION nostr_kind4_to_message(event: NostrEvent) -> UnifiedMessage:
    -- kind:4 (encrypted DM) event
    -- NIP-04: content is base64-encoded, encrypted with shared secret
    -- p tag references the recipient
    p_tag = event.tags.find(t => t[0] == "p")
    if not p_tag:
        return null

    id = "nostr:" + event.id
    protocol = "nostr"

    -- Conversation ID: deterministic from sorted pubkeys
    participants = [event.pubkey, p_tag[1]].sort()
    conversation_id = "nostr:" + sha256(participants.join(","))

    sender_id = "nostr:" + event.pubkey
    body = event.content   -- Encrypted; decryption happens at app layer
    encrypted = true
    encryption_alg = "nostr.nip04"
    sent_at = unix_to_iso8601(event.created_at)
    status = "sent"

    RETURN UnifiedMessage { ... }

--- Reverse (Publish) Mappings ---

FUNCTION unified_post_to_nostr_event(post: UnifiedPost, private_key: hex) -> NostrEvent:
    event = {
        kind: 1,
        content: post.content,
        tags: [],
        created_at: Math.floor(Date.now() / 1000),
        pubkey: derive_pubkey(private_key),
    }

    -- Tags
    if post.tags:
        for tag in post.tags:
            event.tags.push(["t", tag.toLowerCase()])

    if post.mentions:
        for mention_id in post.mentions:
            pubkey = extract_nostr_pubkey(mention_id)
            event.tags.push(["p", pubkey])

    -- Threading (e tags)
    if post.replyToId:
        parent_hex = extract_nostr_event_id(post.replyToId)
        event.tags.push(["e", parent_hex, "", "reply"])

    if post.rootPostId:
        root_hex = extract_nostr_event_id(post.rootPostId)
        event.tags.push(["e", root_hex, "", "root"])

    -- Sign and finalize
    event.id = compute_nostr_event_id(event)
    event.sig = schnorr_sign(event.id, private_key)

    return event
```

### 3.4 Matrix

Matrix distinguishes between room types: DM rooms (two-person) are treated as conversations; channel/forum rooms have timeline events treated as posts.

```
FUNCTION matrix_room_classify(room: MatrixRoom) -> 'dm' | 'channel' | 'group':
    -- Heuristic classification based on room state
    if room.is_direct || room.joined_members.length <= 2:
        -- Is DM if:
        -- 1. m.direct account data includes this room, OR
        -- 2. Exactly 2 joined members (self + one other)
        return 'dm'

    if room.room_type == 'm.space':
        return 'space'  -- Not a conversation; skip

    -- Otherwise it's a channel/room:
    return 'channel'

FUNCTION matrix_event_to_unified(event: MatrixEvent, room: MatrixRoom) -> UnifiedPost | UnifiedMessage:

    if event.type != "m.room.message":
        -- Only m.room.message maps to posts/messages currently
        -- Other event types (reactions, membership changes) handled separately
        return null

    room_class = matrix_room_classify(room)

    content = event.content

    -- Extract plain text
    text = content.body ?? ""

    -- Extract formatted body
    rich_text = content.formatted_body ?? content.body

    canonical_id = "matrix:" + event.event_id

    if room_class == 'dm':
        -- DM -> UnifiedMessage
        return UnifiedMessage {
            id: canonical_id,
            protocol: "matrix",
            conversation_id: "matrix:" + room.room_id,
            sender_id: "matrix:" + event.sender,
            body: text,
            rich_body: rich_text,
            sent_at: timestamp_to_iso(event.origin_server_ts),
            encrypted: content.algorithm != null,  -- m.encrypted
            encryption_alg: content.algorithm,      -- "m.megolm.v1.aes-sha2"
            status: "sent",
            source: event
        }
    else:
        -- Channel/Room -> UnifiedPost
        media = []
        if content.url:
            -- mxc:// URL needs to be resolved to HTTP
            media.push(matrix_attachment_to_unified(content, event))

        -- Extract mentions from matrix event content
        mentions = extract_matrix_mentions(content)
        tags = extract_matrix_tags(content)

        return UnifiedPost {
            id: canonical_id,
            protocol: "matrix",
            author_id: "matrix:" + event.sender,
            content: text,
            rich_content: rich_text,
            created_at: timestamp_to_iso(event.origin_server_ts),
            visibility: "unlisted",   -- Matrix rooms have access control at room level
            language: null,            -- Not per-message in Matrix
            media: media,
            mentions: mentions,
            tags: tags,
            source: event
        }

FUNCTION matrix_reaction_to_unified(event: MatrixEvent) -> UnifiedReaction:
    -- m.reaction event
    if event.type != "m.reaction":
        return null

    relates_to = event.content["m.relates_to"]
    if not relates_to or relates_to.rel_type != "m.annotation":
        return null

    return UnifiedReaction {
        id: "matrix:" + event.event_id,
        post_id: "matrix:" + relates_to.event_id,
        author_id: "matrix:" + event.sender,
        type: "emoji",
        emoji: relates_to.key,    -- The reaction emoji
        protocol: "matrix",
        created_at: timestamp_to_iso(event.origin_server_ts),
        source: event
    }

--- Reverse (Publish) Mappings ---

FUNCTION unified_message_to_matrix(message: UnifiedMessage, room_id: string) -> MatrixSendRequest:
    content = {
        msgtype: "m.text",
        body: message.body,
    }

    if message.encrypted:
        content.algorithm = "m.megolm.v1.aes-sha2"
        -- Actual encryption happens at the Olm/Megolm layer

    if message.replyToId:
        content["m.relates_to"] = {
            "m.in_reply_to": { event_id: extract_matrix_event_id(message.replyToId) }
        }

    return {
        room_id: room_id,
        event_type: "m.room.message",
        content: content
    }

FUNCTION unified_post_to_matrix(post: UnifiedPost, room_id: string) -> MatrixSendRequest:
    content = {
        msgtype: "m.text",
        body: post.content,
    }

    if post.richContent:
        content.format = "org.matrix.custom.html"
        content.formatted_body = post.richContent

    if post.replyToId:
        content["m.relates_to"] = {
            "m.in_reply_to": { event_id: extract_matrix_event_id(post.replyToId) }
        }

    return {
        room_id: room_id,
        event_type: "m.room.message",
        content: content
    }
```

### 3.5 IRC

```
FUNCTION irc_privmsg_to_unified(msg: IRCMessage, server: string) -> UnifiedMessage:

    -- IRC is inherently ephemeral and message-oriented
    -- PRIVMSG #channel => channel message
    -- PRIVMSG nick     => DM

    target = msg.params[0]     -- #channel or nick
    text = msg.params[1]       -- The message text

    is_channel = target.startsWith("#") || target.startsWith("&")
                || target.startsWith("+") || target.startsWith("!")

    -- Strip IRC formatting codes (bold, color, underline)
    clean_text = strip_irc_formatting(text)

    -- Detect CTCP ACTION (/me)
    is_action = clean_text.startsWith("\x01ACTION") && clean_text.endsWith("\x01")
    if is_action:
        clean_text = clean_text.slice(8, -1)  -- Remove ACTION wrapper

    -- Generate deterministic ID from server + timestamp + nick hash
    synthetic_id = "irc:" + server + "/" + (is_channel ? target : target)
                   + ":" + msg.timestamp + ":" + msg.nick

    -- Conversation ID
    if is_channel:
        conv_id = "irc:" + server + "/" + target.lower()
        conv_type = "channel"
    else:
        -- DM conversation: sorted nicks
        participants = [msg.nick, local_nick].sort()
        conv_id = "irc:" + server + "/dm:" + sha256(participants.join(","))
        conv_type = "dm"

    return UnifiedMessage {
        id: synthetic_id,
        protocol: "irc",
        conversation_id: conv_id,
        sender_id: build_irc_profile_id(server, msg.nick),
        body: is_action ? "* " + msg.nick + " " + clean_text : clean_text,
        encrypted: false,
        sent_at: unix_to_iso8601(msg.timestamp),
        received_at: now_iso8601(),
        status: "delivered",
        source: { raw: msg.raw, server: server }
    }

FUNCTION build_irc_profile_id(server: string, nick: string) -> string:
    -- IRC profiles are session-scoped; we create transient profile stubs
    profile_id = "irc:" + server + ":" + nick

    -- Ensure profile stub exists in database
    upsert_profile_stub(profile_id, "irc", nick, server)

    return profile_id

--- Reverse (Publish) Mappings ---

FUNCTION unified_message_to_irc(msg: UnifiedMessage, server: string) -> IRCCommand:
    -- IRC is fire-and-forget; no delivery confirmation from protocol
    target = extract_irc_target(msg.conversationId)

    if msg.body.startsWith("* "):
        -- CTCP ACTION
        return { command: "PRIVMSG", params: [target, "\x01ACTION " + msg.body.slice(2) + "\x01"] }

    return { command: "PRIVMSG", params: [target, msg.body] }

--- Note on IRC profiles ---
-- IRC has no persistent identity system (no NickServ integration yet).
-- Profile stubs are created with nick!user@host if available.
-- Identity linking can be done manually by the user.
```

### 3.6 XMPP

```
FUNCTION xmpp_message_to_unified(stanza: Element, jid: string) -> UnifiedMessage:

    body_text = stanza.getChildText("body") ?? ""
    thread_id = stanza.getChildText("thread")
    reply_id = stanza.getChild("reply")?.getAttribute("id")
    delay_stamp = stanza.getChild("delay", "urn:xmpp:delay")?.getAttribute("stamp")

    -- Extract bare JID for conversation identification
    bare_jid = jid.split("/")[0]
    local_bare_jid = local_jid.split("/")[0]

    -- Determine conversation type and ID
    if stanza.getAttribute("type") == "groupchat":
        -- MUC message
        conv_id = "xmpp:" + stanza.getAttribute("from").split("/")[0]
        conv_type = "channel"
        sender_nick = stanza.getAttribute("from").split("/")[1]
        sender_jid = resolve_muc_real_jid(sender_nick, conv_id) ?? stanza.getAttribute("from")

        sender_id = "xmpp:" + sender_jid
    else:
        -- Chat message (DM)
        participants = [bare_jid, local_bare_jid].sort()
        conv_id = "xmpp:" + participants.join(",")
        conv_type = "dm"
        sender_id = "xmpp:" + bare_jid

    -- Detect encryption
    encrypted = stanza.getChild("encrypted", "urn:xmpp:omemo:0") != null
                || stanza.getChild("crypto", "urn:xmpp:encryption") != null
    encryption_alg = "omemo" if stanza.getChild("encrypted", "urn:xmpp:omemo:0") else
                     "openpgp" if stanza.getChild("openpgp", "urn:xmpp:openpgp:0") else
                     null

    -- Stanza ID for deduplication
    stanza_id = stanza.getAttribute("id")
                ?? stanza.getChild("stanza-id", "urn:xmpp:sid:0")?.getAttribute("id")
                ?? bare_jid + ":" + thread_id + ":" + delay_stamp

    synthetic_id = "xmpp:" + stanza_id

    timestamp = delay_stamp ?? now_iso8601()

    return UnifiedMessage {
        id: synthetic_id,
        protocol: "xmpp",
        conversation_id: conv_id,
        sender_id: sender_id,
        body: body_text,
        encrypted: encrypted,
        encryption_alg: encryption_alg,
        sent_at: normalize_iso8601(timestamp),
        received_at: delay_stamp ? null : now_iso8601(),
        replyToId: reply_id ? "xmpp:" + reply_id : null,
        status: "delivered",
        source: { stanza_xml: stanza.toString(), jid: jid }
    }

--- Reverse (Publish) Mappings ---

FUNCTION unified_message_to_xmpp(msg: UnifiedMessage, to_jid: string) -> XMPPStanza:
    stanza_type = detect_conversation_type(msg.conversationId) == "channel" ? "groupchat" : "chat"

    root = Element("message")
    root.setAttribute("to", to_jid)
    root.setAttribute("type", stanza_type)
    root.setAttribute("id", generate_stanza_id())

    body = SubElement(root, "body")
    body.text = msg.body

    if msg.replyToId:
        reply = SubElement(root, "reply", { "xmlns": "urn:xmpp:reply:0" })
        reply.setAttribute("id", extract_xmpp_stanza_id(msg.replyToId))

    return root
```

---

## 4. Sync Strategies Per Protocol

### 4.1 ActivityPub (Mastodon API)

| Aspect | Strategy |
|--------|----------|
| **Cold Start** | Fetch 40 posts from /api/v1/timelines/home, 30 notifications from /api/v1/notifications, 20 DMs from /api/v1/conversations. Store `since_id` = min(ID) from first batch for incremental sync. |
| **Incremental Sync** | REST polling: GET /api/v1/timelines/home?since_id={last_seen_id}. Mastodon API uses `since_id` (newer), `max_id` (older), `min_id` (even newer, for streaming gaps). Store `since_id` per stream in sync_state. Poll interval: 30s for timeline, 60s for notifications. |
| **Realtime** | WebSocket via Mastodon streaming API (/api/v1/streaming). Subscribe to `user` stream (home + notifications). Parse Server-Sent Events (SSE). Reconnect with exponential backoff (1s, 5s, 30s, 60s cap). On reconnect, use `since_id` to fill missed events. |
| **Conflict Resolution** | Last-write-wins for posts (remote canonical). For local edits that haven't been pushed, check `updated_at` on the remote object. If local is newer, offer to republish. Mastodon does not support editing, so this is limited to bookmarks/favourites. |
| **Offline Queue** | Posts go into `outbox` table with status='pending'. On reconnection, send in FIFO order. Interactions (like, follow, boost) go into `pending_interactions`. Mastodon API is idempotent for most actions (duplicate like returns 200). |

### 4.2 AT Protocol (Bluesky)

| Aspect | Strategy |
|--------|----------|
| **Cold Start** | GET app.bsky.feed.getTimeline with limit=50. GET app.bsky.notification.listNotifications with limit=50. Resolve handle to DID (com.atproto.identity.resolveHandle). PDP = PLC directory for DID resolution. Subscribe to firehose relay (e.g., `jetstream.atproto.tools` or `bsky.network`). |
| **Incremental Sync** | Timeline: GET app.bsky.feed.getTimeline?cursor={cursor}. Cursor is an opaque string from the response `cursor` field. Notifications: GET app.bsky.notification.listNotifications?cursor={cursor}. Batching is HTTP-only; no long-poll. Poll interval: 30s. |
| **Realtime** | WebSocket subscription to the relay firehose (com.atproto.sync.subscribeRepos). Parse CBOR-encoded commit events. Filter for relevant collections (app.bsky.feed.post, app.bsky.feed.like, app.bsky.feed.repost, app.bsky.actor.profile). Track cursor (`rev` field on commits) in sync_state for resubscription. |
| **Conflict Resolution** | AT Protocol records are immutable once created (no editing). For outbox: if a record create fails, check if it already exists (idempotency via rkey). For likes: app.bsky.feed.like is unique per (actor, subject); duplicate returns existing. |
| **Offline Queue** | Posts: serialize as app.bsky.feed.post record JSON in outbox. On send, need to: (1) upload blobs first via com.atproto.repo.uploadBlob, (2) create record via com.atproto.repo.createRecord. Store blob CIDs in outbox for retry. Pending interactions queue for likes/reposts/follows. |

### 4.3 Nostr

| Aspect | Strategy |
|--------|----------|
| **Cold Start** | Connect to configured relays. Subscribe to kind:0 + kind:3 for followed pubkeys (fetch metadata and contact lists). Subscribe to kind:1 since=(now - 24h) for home feed. Limit=500 events. Build follows graph from kind:3 events. Fetch kind:0 (metadata) for each new pubkey encountered. |
| **Incremental Sync** | REQ filter with `since` = last known timestamp (Unix epoch). Each relay sends events as they appear. Nostr has no server-side cursor; use `since` timestamp. Filter subscriptions per-kind. Store `cursor` in sync_state as unix timestamp per relay+kind. |
| **Realtime** | Persistent WebSocket connection to relays. Single shared connection per relay. Subscription filters: `{ "kinds": [1, 6, 7, 0, 3], "limit": 0 }` = live only. Handle NOTICE messages for rate limiting. Handle EOSE (end of stored events) before switching to realtime mode. Reconnect: 1s -> 5s -> 30s, randomize jitter. |
| **Conflict Resolution** | Nostr events are immutable by design. Same event ID = same content everywhere. Different events with same pubkey+content are duplicates; deduplicate by event ID. For follows: last kind:3 from a pubkey wins (full replace list). |
| **Offline Queue** | Events are signed and serialized client-side. Outbox stores the full signed event JSON. On reconnect, publish via EVENT message. Idempotency: relays return OK even for duplicates. Store relay list per outbox item (some may have succeeded). |

### 4.4 Matrix

| Aspect | Strategy |
|--------|----------|
| **Cold Start** | Initial sync via /_matrix/client/v3/sync with filter for latest 50 events per room. Use `since` = null (initial sync). Process all room state (m.room.member, m.room.name, etc.) to build room list. Download room timeline events. Process account data (m.direct for DM list, m.push_rules, m.fully_read). |
| **Incremental Sync** | /sync?since={next_batch}. next_batch token from previous sync response. Filter: only timeline events (not state) for efficiency. Batch messages into conversation timeline. Process to-device messages for E2EE key exchange. Poll interval: 10s idle, 2s app-in-foreground. |
| **Realtime** | Matrix /sync is a long-poll endpoint (holds connection up to 30s). No separate WebSocket needed unless using Sliding Sync (MSC3575). For E2EE rooms, must also maintain Olm/Megolm session state. Use `/sync` with incremental timeout=30000ms to emulate realtime. |
| **Conflict Resolution** | Matrix uses event ID ordering within each room (origin_server_ts, but not guaranteed monotonic). For sent messages: track via event_id returned by /send. If /send returns an event_id for a pending outbox item, mark as sent. If /send fails, retry with new transaction ID (idempotent). |
| **Offline Queue** | Messages queue in outbox with the `txn_id` (transaction ID) generated client-side. On reconnect: /send/{roomType}/{eventType}/{txnId}. Matrix deduplicates by txnId. Failed messages: retry 3x, then move to 'failed' status. |

### 4.5 IRC

| Aspect | Strategy |
|--------|----------|
| **Cold Start** | IRC has no message history on connect. Connect to server, join channels, request channel topic (/TOPIC). Optionally use ZNC bouncer's playback feature if configured. For networks with NickServ: request account info. No history available unless using a bouncer. |
| **Incremental Sync** | Not applicable in standard IRC. Messages arrive only while connected. If using ZNC bouncer: PLAYBACK command to retrieve missed messages since last disconnect. Store last message timestamp per channel in sync_state for bouncer playfill. |
| **Realtime** | Persistent TCP connection to server. IRC is push-only; all messages arrive in realtime as long as connected. Handle PING/PONG keepalive (2-minute intervals). Reconnect: 1s -> 10s -> 60s, re-join channels on reconnect. |
| **Conflict Resolution** | IRC has no persistence. No conflict resolution needed. Sent messages from outbox are fire-and-forget. |
| **Offline Queue** | Messages queue in outbox. On reconnect, replay FIFO. IRC has no delivery confirmation; assume success if no TCP error. Rate limit: 1 message per 500ms (IRC flood protection). Queue drains at controlled rate. |

### 4.6 XMPP

| Aspect | Strategy |
|--------|----------|
| **Cold Start** | Stream initiation: open TCP, TLS handshake, SASL auth (PLAIN/SCRAM-SHA-1), resource binding. Request roster (contact list) via `<iq type='get'><query xmlns='jabber:iq:roster'/></iq>`. Request MUC bookmarks (XEP-0048) if available. If MAM (Message Archive Management, XEP-0313) supported: query last 50 messages per conversation. |
| **Incremental Sync** | If MAM supported: query archive since last stanza ID: `<iq type='set'><query xmlns='urn:xmpp:mam:2'><set xmlns='http://jabber.org/protocol/rsm'><after>{last_id}</after></set></query></iq>`. Store `last_mam_id` per conversation in sync_state. Poll MAM every 60s if no realtime connection. |
| **Realtime** | Persistent TCP + TLS. Bidirectional XML stanzas over the same stream. Keepalive: whitespace ping every 60s (XEP-0199). Handle `<presence>` stanzas for online/offline state. Handle `<message>` stanzas for incoming messages. Reconnect: 1s -> 5s -> 60s. On reconnect: resource binding (new resource), re-join MUCs. |
| **Conflict Resolution** | XMPP message IDs are server-assigned; use stanza ID (XEP-0359) for deduplication. MAM returns stable IDs. For sent messages: track by stanza ID. XMPP has no edit history standard. Last-wins for roster/presence state. |
| **Offline Queue** | Messages queue in outbox. On reconnect: send with unique stanza ID. XMPP server handles offline messages if recipient is offline (XEP-0160). For MUC: messages are delivered only when online. Retry on error stanza (503, 405). |

---

## 5. Migration & Schema Evolution

### 5.1 Schema Versioning Strategy

```sql
-- Every migration adds a row to schema_version
CREATE TABLE IF NOT EXISTS schema_version (
    version     INTEGER PRIMARY KEY,
    description TEXT    NOT NULL,
    applied_at  TEXT    NOT NULL DEFAULT (datetime('now')),
    checksum    TEXT,              -- SHA-256 of migration SQL for integrity
    duration_ms INTEGER            -- How long the migration took
);

-- Current version check
-- SELECT MAX(version) FROM schema_version;
```

Migration files are stored in `assets/db/migrations/` with sequential naming:

```
migrations/
  001_initial_schema.sql
  002_add_mentions_table.sql
  003_add_matrix_sync_state.sql
  004_add_cross_protocol_refs.sql
  ...
```

**Application-level migration logic:**

```
FUNCTION apply_migrations(db: SQLiteConnection):
    current_version = db.exec("SELECT COALESCE(MAX(version), 0) FROM schema_version")
    pending_migrations = list_migration_files()
        .filter(f => f.version > current_version)
        .sort_by(f => f.version)

    FOR migration in pending_migrations:
        db.exec("PRAGMA defer_foreign_keys = ON")
        db.exec("BEGIN TRANSACTION")

        start = now()
        migration_sql = read_file(migration.path)

        TRY:
            db.exec(migration_sql)
            checksum = sha256(migration_sql)
            duration = now() - start
            db.exec("INSERT INTO schema_version (version, description, checksum, duration_ms) " +
                     "VALUES (?, ?, ?, ?)",
                     [migration.version, migration.description, checksum, duration])
            db.exec("COMMIT")
        CATCH error:
            db.exec("ROLLBACK")
            REPORT migration failure
            THROW  -- App should not proceed on migration failure
```

### 5.2 Adding a New Protocol

When adding a new protocol (e.g., Telegram, Signal):

1. **Define the protocol prefix constant**: `protocol_id = "telegram"` (4-16 chars, alphanumeric)
2. **Create protocol extension table**: `telegram_messages` or similar for raw data
3. **Add canonical mapping functions**: Implement the `*_to_unified` and `*_to_native` mappers
4. **Add sync_state rows**: `INSERT OR IGNORE INTO sync_state (protocol, stream, status) VALUES ('telegram', 'messages', 'idle')`
5. **Add connection_config records**: For server/API credentials
6. **Migration example**:

```sql
-- Migration 010_add_telegram.sql
ALTER TABLE sync_state ADD COLUMN protocol_check INTEGER;

-- Add new protocol extension table
CREATE TABLE telegram_messages (
    message_id   INTEGER NOT NULL,
    chat_id      INTEGER NOT NULL,
    text         TEXT,
    from_id      INTEGER,
    date         INTEGER NOT NULL,
    raw_json     TEXT,
    canonical_message_id TEXT REFERENCES messages(id) ON DELETE SET NULL,
    PRIMARY KEY (chat_id, message_id)
);

-- Add to connection config seed (application code handles this)
```

### 5.3 Adding New Canonical Fields

**Principle: additive-only changes to core tables.**

```sql
-- Migration 011_add_post_location.sql
ALTER TABLE posts ADD COLUMN latitude REAL;
ALTER TABLE posts ADD COLUMN longitude REAL;
ALTER TABLE posts ADD COLUMN place_name TEXT;

-- New index
CREATE INDEX idx_posts_location ON posts(latitude, longitude) WHERE latitude IS NOT NULL;

-- Rebuild FTS triggers would need update IF content is added to FTS
-- Since location isn't searchable text, no trigger change needed
```

**Rules:**
- `ALTER TABLE ... ADD COLUMN` is always safe (SQLite does not support DROP COLUMN before 3.35.0; use this only for renames/deletions)
- New columns must be nullable or have DEFAULT values (existing rows get NULL/DEFAULT)
- New NOT NULL columns without DEFAULT require a multi-step migration (add as nullable, backfill, then enforce via application logic since SQLite cannot add NOT NULL to existing table)
- Index changes are safe (CREATE INDEX IF NOT EXISTS)
- FTS changes: ALTER TABLE on FTS5 tables only supports `ADD COLUMN` for unindexed columns. For indexed column changes, rebuild the FTS table

### 5.4 Protocol-Specific Schema Changes

Each protocol evolves independently. The extension tables accommodate this naturally:

```sql
-- Nostr adds NIP-xx new feature
-- Migration 012_nostr_add_nip_xx.sql
ALTER TABLE nostr_events ADD COLUMN nip_xx_data TEXT;

-- Or add protocol-specific tags storage
CREATE TABLE nostr_event_tags (
    event_id TEXT NOT NULL REFERENCES nostr_events(event_id) ON DELETE CASCADE,
    tag_name TEXT NOT NULL,      -- Single character: 'a', 'e', 'p', 't', 'd', etc.
    tag_values TEXT NOT NULL,    -- JSON array of the tag values
    PRIMARY KEY (event_id, tag_name, tag_values)
);
```

**Policy for extension tables:**
- Raw protocol data is stored as JSON blobs (`raw_data`, `object_json`, etc.)
- Structured columns are added only when the field is needed for queries (WHERE, JOIN, ORDER BY)
- Protocol-specific fields that don't need querying stay as JSON metadata
- Each protocol adapter is responsible for its own extension table migrations

### 5.5 Handling Breaking Protocol Changes

When a protocol makes a backward-incompatible change (e.g., Mastodon API v2 vs v1):

1. **Version the adapter**: Each adapter has a `protocol_version` field in connection_config
2. **Store both versions**: Raw data in extension tables preserves both old and new format
3. **Dual mapping functions**: Maintain both `ap_v1_to_unified` and `ap_v2_to_unified`
4. **Migration by re-sync**: For format changes, drop and re-sync affected data:

```sql
-- Example: Mastodon API v2 changes media attachment format
-- Migration 020_mastodon_v2_media.sql

-- New column for v2 format
ALTER TABLE activitypub_objects ADD COLUMN media_v2 TEXT;

-- Mark affected sync states for re-sync
UPDATE sync_state SET status = 'idle', last_synced_at = NULL
WHERE protocol = 'activitypub' AND stream IN ('timeline', 'notifications', 'dms');
```

### 5.6 Triggers for Data Integrity

Beyond FTS triggers, consider:

```sql
-- Auto-update parent conversation.last_message_at on new message
CREATE TRIGGER after_message_insert_conv_update AFTER INSERT ON messages BEGIN
    UPDATE conversations
    SET last_message_id = NEW.id,
        last_message_at = NEW.sent_at
    WHERE id = NEW.conversation_id;
END;

-- Cascade profile display name updates to FTS
CREATE TRIGGER after_profile_update_display AFTER UPDATE OF display_name ON profiles
WHEN OLD.display_name IS NOT NEW.display_name
BEGIN
    -- Update content_fts for posts by this author (denormalized)
    -- This is expensive for large datasets; alternative is to resolve at query time
END;

-- Update extended table canonical reference when post is deleted
CREATE TRIGGER after_post_delete_cascade_refs AFTER UPDATE OF is_deleted ON posts
WHEN NEW.is_deleted = 1 AND OLD.is_deleted = 0
BEGIN
    UPDATE nostr_events SET canonical_post_id = NULL WHERE canonical_post_id = OLD.id;
    UPDATE atproto_records SET canonical_post_id = NULL WHERE canonical_post_id = OLD.id;
    UPDATE activitypub_objects SET canonical_post_id = NULL WHERE canonical_post_id = OLD.id;
    UPDATE matrix_events SET canonical_post_id = NULL WHERE canonical_post_id = OLD.id;
END;
```

### 5.7 Migration Safety Checklist

- [ ] Always wrap migrations in a transaction
- [ ] Set `PRAGMA defer_foreign_keys = ON` during migration (allows out-of-order FK enforcement)
- [ ] Test migration on a copy of the database first
- [ ] Never drop a column that contains data (SQLite requires table rebuild)
- [ ] For column renames: add new column, copy data, update application to write both, then remove old column in a future migration
- [ ] Keep a `schema_version` checksum to detect manual edits
- [ ] Migration should be idempotent: use `IF NOT EXISTS`, `CREATE INDEX IF NOT EXISTS`, `INSERT OR IGNORE`
- [ ] Store migration rollback scripts alongside forward scripts

---

## Appendix A: Canonical ID Scheme

```
Format:  "{protocol}:{protocol_specific_id}"

Examples:

  nostr:32hexcharacters00000000000000000000000000000000000000000000aabbccdd
  atproto:at://did:plc:abc123def456/app.bsky.feed.post/3jwdw7e5y
  activitypub:https://mastodon.social/@user/1234567890
  matrix:$eventid123:matrix.org
  irc:libera.chat/#rust:1719000000:johndoe
  xmpp:user@chat.example.com/resource:stanza-id-42

Pattern breakdown:
  nostr:        {event_id hex}
  atproto:      {at-uri}
  activitypub:  {actor_url_or_uri}
  matrix:       {event_id}
  irc:          {server}/{channel_or_nick}:{timestamp}:{sender_nick}
  xmpp:         {bare_jid}:{stanza_id_or_thread}
```

## Appendix B: SQLite Configuration Notes

1. **WAL mode** is critical for performance: allows concurrent reads during writes.
2. **foreign_keys = ON** must be set at the start of every connection.
3. **synchronous = NORMAL** balances durability with performance. Use FULL for critical data.
4. **cache_size = -64000** allocates 64MB for the page cache; adjust based on device.
5. **busy_timeout = 5000** prevents "database is locked" errors during concurrent access.
6. **FTS5** requires SQLite 3.9.0+. iOS has this built-in; verify on desktop.
7. **Maximum blob size**: SQLite supports up to 2GB per row; practical limit is 1MB per JSON blob.
8. **Maximum database size**: ~140TB; not a concern for on-device storage.
9. **Backup**: Use `.backup` command or `sqlite3_backup_init()` API for safe hot backups.
10. **Memory-mapped I/O**: Consider `PRAGMA mmap_size = 268435456` (256MB) for large databases.

## Appendix C: Common Query Patterns

```sql
-- Home timeline (all protocols, unified)
SELECT p.*, pr.display_name, pr.handle, pr.avatar_url
FROM posts p
JOIN profiles pr ON p.author_id = pr.id
WHERE p.is_deleted = 0
  AND p.author_id IN (
      SELECT following_id FROM follows WHERE follower_id = :my_profile_id
  )
ORDER BY p.created_at DESC
LIMIT 50;

-- Single protocol timeline
SELECT p.* FROM posts p
WHERE p.protocol = :protocol AND p.is_deleted = 0
ORDER BY p.created_at DESC
LIMIT 50;

-- Full-text search across all content
SELECT c.content_id, c.source_type, c.text_content, snippet(content_fts, 0, '<b>', '</b>', '...', 32)
FROM content_fts c
WHERE content_fts MATCH :search_query
ORDER BY rank
LIMIT 50;

-- Thread view
SELECT * FROM posts
WHERE root_post_id = :root_id OR id = :root_id
ORDER BY created_at ASC;

-- Conversation messages
SELECT m.*, pr.display_name, pr.avatar_url
FROM messages m
JOIN profiles pr ON m.sender_id = pr.id
WHERE m.conversation_id = :conv_id AND m.is_deleted = 0
ORDER BY m.sent_at ASC;

-- Pending outbox items for dispatch
SELECT * FROM outbox
WHERE status IN ('pending', 'failed')
  AND (next_retry_at IS NULL OR next_retry_at <= datetime('now'))
ORDER BY priority DESC, created_at ASC;

-- Sync state, next to sync
SELECT * FROM sync_state
WHERE status = 'idle'
ORDER BY last_synced_at ASC
LIMIT 1;

-- Cross-protocol identity search
SELECT p1.*, p2.*, il.confidence, il.method
FROM identity_links il
JOIN profiles p1 ON il.profile_a = p1.id
JOIN profiles p2 ON il.profile_b = p2.id
WHERE p1.id = :profile_id OR p2.id = :profile_id;
```
