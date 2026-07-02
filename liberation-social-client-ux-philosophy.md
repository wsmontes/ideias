# The Liberation Client: UX Philosophy & Design Principles

> A design manifesto for a multi-protocol, serverless social media client that rejects the attention economy.

---

## Preamble: Design as Liberation

This is not a "better social media app." This is an escape pod.

Every mainstream social platform is engineered to maximize engagement minutes — not your wellbeing, not your autonomy, not your connection. The architecture of the feed, the timing of notifications, the opacity of the algorithm, the withholding of likes and shares until you've scrolled long enough — these are not bugs. They are the product.

This document outlines the design philosophy for a client that breaks that contract. It connects to six protocols (ActivityPub, AT Protocol, Nostr, Matrix, IRC, XMPP) directly from the user's device, with no intermediary server. It takes no metrics. It optimizes for nothing but user intent.

The design principles herein are not suggestions. They are the specifications of liberation.

---

## 1. Core Design Principles

### 1.1 Intent Over Engagement

The dominant paradigm asks: *How do we keep this person scrolling?* We ask: *What does this person want to do?*

Every interaction should be preceded by explicit user intent. The app should never suggest, infer, or manufacture a need to check it.

**Concrete implementation:** The app opens to a "What are you here for?" screen — not a feed. Three options:
- **Browse** (intentional content exploration)
- **Respond** (check replies and DMs)
- **Share** (compose a new post)

Each path takes the user exactly where they need to go and no further. Browsing ends. Replying resolves. Sharing publishes. The app does not try to extend the session.

### 1.2 Chronological Is Neutral — Curated Is Political

Algorithmic feeds are political statements dressed as engineering. They decide what you see, in what order, and what they hide. The only neutral timeline is chronological — but chronological alone is insufficient because it conflates recency with relevance.

**Concrete implementation:** Three-tier timeline architecture:
1. **Raw chronological** — every post from every followed account, in order. No filtering, no prioritization. This is the default.
2. **List-based** — user-curated lists that group accounts by context (e.g., "Work," "Friends," "Photography inspiration"). Lists are manually maintained.
3. **Manual pinnable filters** — user-configured keyword mute lists, account mutes, and domain blocks. These are *negative* filters (what to exclude), never *positive* filters (what to amplify). The user controls the sieve.

There is no "For You" page. There is no "Discover" algorithm. There is no "trending" section.

### 1.3 Friction as a Feature

The attention economy optimizes for frictionless consumption — infinite scroll, autoplay, one-tap reactions. We invert this. Friction creates intention. Friction creates deliberation. Friction is a feature.

**Where to add friction:**
- **Composing a public post** vs. a private message: Public posting requires a deliberate gesture (e.g., a long-press on the send button) and a confirmation dialog: "This will be visible to everyone. Are you sure?"
- **Muting or blocking someone**: No instant tap. A two-step confirmation that asks "Why are you taking this action?" — not to harvest data but to encourage reflection.
- **Following a new account**: The app shows you their last 3 posts before you confirm, plus a 24-hour "cooling off" period before the follow activates. This reduces impulse follows from algorithmic suggestion.
- **Scrolling past a certain threshold**: After 50 posts in one session, the app gently asks: "You've been here a while. Would you like to set a timer, or are you finding what you need?"

**Where NOT to add friction:**
- Replying to a DM from someone you regularly talk to.
- Checking urgent notifications (direct mentions, DMs).
- Emergency access to trusted contacts.

Friction targets *compulsive* behavior, not *intentional* communication.

### 1.4 Radical Transparency

Every design decision must be explainable to the user. There is no black box.

**Concrete implementation:**
- **The Timeline Manifesto** — a one-tap info panel on every feed view that states: "This timeline shows [N] posts from [X] accounts, in chronological order. No posts have been hidden. No posts have been promoted. Sort order: oldest-to-newest by server timestamp."
- **Protocol transparency** — every post has a visible protocol badge. If a post was boosted/reposted, the original source protocol and author are shown.
- **Data transparency** — a dashboard showing exactly what data the app stores locally, what it sends to each protocol's network, and what it never collects (interaction data, scroll depth, time spent, session duration).

### 1.5 Ownership by Default

The user owns their data, their identity, and their attention. The app is a tool, not a platform.

**Concrete implementation:**
- Complete local data export in standard formats (JSON, CSV, Markdown) available from the settings screen with one tap.
- All data stored locally first; protocol servers are mirrors, not sources.
- The app can function fully offline — composing drafts, reading cached content, managing your address book. Network is a feature, not a requirement.
- Account deletion from any protocol does not delete local data. The user's archive is theirs permanently.

---

## 2. Feed & Content Architecture

### 2.1 The Unified Inbox — But With Structural Honesty

The promise of multi-protocol is a single place for all communication. But protocols have different cultures, norms, and technical constraints. A unified feed that mashes them together without context is noise.

**Design decision:** A **unified chronological feed as the default view**, but with persistent protocol-aware visual grouping and the ability to split at any time.

**Concrete implementation:**
- The unified timeline shows all content interleaved by timestamp. Each post carries a protocol badge (color-coded, shape-coded for colorblind accessibility) and a subtle divider between protocol "sections."
- A **"Filter by protocol"** pill bar sits above the timeline — tap any protocol to see only that network's content. Multiple can be selected (e.g., ActivityPub + AT Protocol, hide IRC and Matrix).
- A **"Context" view** is always one tap away: showing the thread as it exists on the *original* protocol, maintaining threading integrity even when posts come from different sources.

### 2.2 Content Type Mapping

Each protocol has its own content types. The UX must normalize without erasing.

| Protocol | Native Term | Mapped Term | Special Behavior |
|---|---|---|---|
| ActivityPub | Toot / Post | **Post** | Can be boosted, liked, replied |
| AT Protocol | Skeet / Post | **Post** | Can be reposted, liked, replied |
| Nostr | Note | **Post** | Can be reposted (kind-6), liked (kind-7) |
| Matrix | Message | **Message** | No public boost; E2EE when applicable |
| IRC | Message | **Message** | Transient; not stored by default; no reactions |
| XMPP | Message | **Message** | Usually 1:1 or small group |

**Design decision:** Three atomic types:
1. **Public Posts** — anything intended for a broad audience (ActivityPub, AT Protocol, Nostr). Full interaction set.
2. **Group Messages** — Matrix rooms, XMPP MUCs, IRC channels. Lower visibility, conversation-focused.
3. **Direct Messages** — 1:1 on any protocol. Highest privacy bar.

Each type has a distinct visual treatment (color, iconography, spacing) that is *consistent across protocols* — so a DM from Matrix looks like a DM from XMPP.

### 2.3 Handling Context Collapse

Following someone on Mastodon is not the same as following them on Nostr. A user's social graph is fragmented across protocols, and the app must not pretend otherwise.

**Concrete implementation:**
- **Per-protocol follow lists** — the "following" view shows which protocol each account is followed on. You can follow someone on multiple protocols (e.g., someone who posts different content on Mastodon vs. Bluesky).
- **Protocol-aware contact cards** — each person's profile shows what protocols they're reachable on, with their handle on each. This replaces the single "social graph" with a multi-dimensional identity map.
- **"Same person?" identity clustering** — optional local matching that suggests whether two accounts on different protocols might be the same person (based on handle similarity, profile links, manually confirmed pairs). This is done locally, never sent to a server.

---

## 3. Anti-Engagement Design Patterns

### 3.1 No Infinite Scroll

Infinite scroll is the crack cocaine of engagement. It removes the natural stopping cues that allow users to consciously end a session.

**What replaces it:**

- **Paginated timelines** with a hard stop at 50 posts per page. At the end: "You've reached the end. [Load more?]"
- **Natural end-of-day markers** — a subtle "--- End of content from [date] ---" divider when you've caught up.
- **"Catch-up complete" state** — when you've read all new posts since last visit, the app displays: "You're all caught up as of [time]. Check again later, or do something else."
- **Pull-to-refresh replaced with "check for new" button** — the user must explicitly choose to look for new content. No passive refresh.

### 3.2 Vanity Metrics Are Eliminated

Like counts, follower counts, repost counts, and view counts are engagement engines. They trigger social comparison loops. They are removed.

**What replaces them:**

- **Likes become "acknowledgments"** — a binary indicator that you have seen and acknowledged a post. No count is displayed. The post author sees *who* acknowledged, not *how many*.
- **Follower counts hidden** — you see *who* follows you (a list of people), not *how many*. The number is accessible but not visible on profiles.
- **Reposts/boosts show "shared by" not "shared N times"** — same logic: people, not numbers.
- **A "sentiment pulse" for authors only** — if an author wants to know how their post resonated, they can opt into a private dashboard showing *qualitative* engagement: reply sentiment analysis (positive/neutral/negative ratio), not counts.

**Design principle:** Numbers abstract people. Show people instead.

### 3.3 Notification Architecture Without Addiction

Notifications are the primary re-engagement vector for all social platforms. We redesign them from first principles.

**Notification types and their treatment:**

| Type | Default Behavior | Rationale |
|---|---|---|
| Direct mention (@username) | Push notification, immediate | Someone is speaking *to* you |
| Reply to your post | Push notification, immediate | Someone is engaging with your words |
| Direct message | Push notification, immediate | Private communication |
| New follower | Silent; daily digest | Not urgent; the person will still be there |
| Like/acknowledgment | No notification at all | Low-signal event |
| Repost/boost | No notification at all | |
| Friend's new post | Never a notification | You check the feed when you want to |

**Concrete implementation:**
- **Notification bundling by intent** — all notifications are grouped by *why* someone is contacting you, not by *who* or *when*.
- **Quiet hours are system-level** — the app respects OS-level focus modes and adds its own "no notification" schedule that defaults to your local night hours.
- **Digest mode** — optional daily or weekly digest of "non-urgent" notifications (new followers, replies to old posts). Delivered at a user-chosen time.

### 3.4 The BONSAI Interaction Model

The BONSAI paper (Bridging Online Networks with Structured Awareness and Intent) on Bluesky proposes that social media should be *intention-based* rather than *algorithmically-paced*. We adopt this as a core model.

**Principles adapted from BONSAI:**

1. **Structured interaction types** — the app distinguishes between *browsing*, *responding*, *sharing*, and *connecting*. Each has a different UI flow and different friction profile.
2. **Session types** — the user declares their intent when opening the app (see 1.1). The app then tailors the experience to that intent and no further.
3. **Asynchronous by default** — the app never expects real-time engagement. Even if the protocol supports it, the UI treats all interactions as asynchronous unless the user explicitly enters a "real-time" mode (e.g., for a live event).
4. **Attentional boundaries** — the app tracks estimated session length and gently prompts the user to reflect after thresholds are crossed.

---

## 4. Identity & Privacy UX

### 4.1 The Multi-Identity Dashboard

A user may have: a Mastodon account on `mastodon.social`, a Bluesky handle at `user.bsky.social`, a Nostr npub, a Matrix ID at `@user:matrix.org`, an IRC nickname, and an XMPP JID. These are not the same identity, but they are all *this person's* identities.

**Concrete implementation:**
- **The Home screen shows the user's identity cloud** — all their handles across protocols displayed as a visually cohesive cluster. Each handle shows its protocol badge and connection status (connected/connecting/offline).
- **"Act as" selector** — before composing, the user picks which identity to post from. The app remembers the last-used identity per context (e.g., always reply from the protocol the original post was on).
- **Cross-protocol identity linking** — optional feature where the user can link their identities locally (e.g., "my Mastodon and Bluesky accounts are both me"). The app can then show unified notifications and thread conversations across protocols. This linking is *local only* — never published.

### 4.2 Encryption Awareness, Not Encryption Theater

Most users don't understand E2EE, PGP, or key management. But they should benefit from it without needing a cryptography degree.

**Concrete implementation:**
- **Visual privacy gradient** — every conversation window has a subtle colored border indicating encryption status:
  - **Green** = E2EE confirmed (Matrix with verified devices)
  - **Yellow** = E2EE attempted but unverified (first key exchange)
  - **Gray** = TLS-only (server can read it, typical for Mastodon/Bluesky/IRC)
  - **Red** = Unencrypted (some IRC channels, legacy XMPP)
- **Key verification simplified** — emoji-based key comparison (like Signal) for Matrix and XMPP E2EE. One-tap "verify this conversation."
- **"This message is private" indicator** — a floating lock icon on DMs that when tapped explains: "This message is encrypted between you and [recipient]. [Server] cannot read it. Learn more."
- **No encryption clutter** — never show the user a raw key fingerprint unless they explicitly ask.

### 4.3 Self-Sovereign Identity UX

Self-sovereign identity (SSI) means the user controls their identifiers without relying on any central authority. But "control your own identifiers" is an abstract concept.

**Concrete implementation:**
- **Identity portability wizard** — "Move my identity" flow that migrates follows, mutes, blocks, and account metadata from one instance/server to another of the same protocol. This makes leaving a server as easy as changing your email provider.
- **Local alias system** — the user can assign local nicknames to any account across protocols. These are never broadcast. "My dad on Bluesky" rather than `@long-alphanumeric-handle.bsky.social`.
- **Handle export** — export a vCard-like file with all your protocol handles for sharing.
- **"Proof of identity"** — optional cross-posting of a signed statement linking two identities (e.g., posting on Mastodon: "My Nostr npub is [hash]"). The app can automatically generate and cross-sign these proofs.

### 4.4 Data Portability Made Visible

Portability is not just a feature — it's a visual statement that the user is not locked in.

**Concrete implementation:**
- **One-tap full export** — Settings > Data > "Export everything" generates a .zip with all posts, messages, media, contacts, and settings. Shows estimated file size and time before starting.
- **Live portability status** — a widget on the dashboard showing: "Your data is stored locally. You can leave any protocol at any time. [Learn more]"
- **Import from platform** — tools to import data dumps from Twitter, Instagram, Facebook, Reddit, converting them to local-format posts with appropriate protocol attribution.

---

## 5. Multi-Protocol UX

### 5.1 Protocol-Aware Design

Each protocol has a social culture, technical constraints, and user expectations. The UI must honor these differences without creating a fractured experience.

**Protocol personality map:**

| Protocol | Tone | Technical Model | UX Implication |
|---|---|---|---|
| **ActivityPub** | Conversational, community-oriented | Server-based, federation | Show instance information; respect local moderation |
| **AT Protocol** | Microblogging with algorithmic opt-in | PDS-based, relay-based | Show custom feeds the user has subscribed to |
| **Nostr** | Censorship-resistant, raw | Relay-based, no server | Show relay list; ephemeral content feel |
| **Matrix** | Group chat, collaboration | Server-based, E2EE | Conversation view; rooms over feeds |
| **IRC** | Transient, real-time | Server-based, no history | Live scrolling; no persistence warnings |
| **XMPP** | 1:1 messaging | Server-based, federated | Chat view; presence indicators |

**Concrete implementation:**
- **Skins per protocol** — subtle visual "skins" that adapt to each protocol's culture without being caricatures. Mastodon gets a warmer palette, Nostr gets a stark/raw minimalism, Matrix gets a collaborative feel, IRC stays text-focused.
- **Consistent interaction primitives** — underneath the skins, the core interactions (reply, acknowledge, share, bookmark) are identical. The user learns one interaction model for all protocols.
- **Protocol-aware smart defaults** — posting to Nostr defaults to plain text with no formatting (Nostr culture). Posting to ActivityPub allows rich text and alt text (Mastodon culture). The composer adapts.

### 5.2 Cross-Posting Without Spam

A user on multiple protocols may want to post to all of them. But cross-posting the same content verbatim to every protocol violates each community's norms and creates noise.

**Concrete implementation:**
- **Protocol-specific composer tabs** — when composing, the user can toggle which protocols to post to. Each tab shows a preview of how the post will render on that protocol.
- **"Adapt" mode** — the user writes a core message, and the app suggests adaptations per protocol (e.g., truncating for Nostr's 280-char limit, adding hashtags for Mastodon discoverability, removing formatting for IRC). The user approves each before sending.
- **Thread awareness** — if a discussion starts on one protocol, the app can optionally bridge replies from other protocols (with permission). The user sees a unified thread even though participants are on different networks.
- **No auto-crossposting** — cross-posting is always manual per post. The default is single-protocol. Permission must be given for each cross-post.

### 5.3 Visual Identity System

Six protocols, one UI. The visual system must distinguish without chaos.

**Design system rules:**

1. **Color as protocol accent** — each protocol has a subtle accent color used for badges, indicators, and the "protocol strip" on the left side of a post. These are:
   - ActivityPub: Purple (borrowed from Mastodon)
   - AT Protocol: Blue (Bluesky)
   - Nostr: Mint/Teal
   - Matrix: Green
   - IRC: Gray
   - XMPP: Orange

2. **Shape as secondary encoding** — for colorblind accessibility, each protocol also has a unique shape marker:
   - ActivityPub: Circle
   - AT Protocol: Diamond
   - Nostr: Hexagon
   - Matrix: Square
   - IRC: Triangle
   - XMPP: Rounded square

3. **Hierarchical protocol display** — primary content is the post; the protocol badge is secondary, always visible but never competing with the content itself. Badges are small, top-right of each post card.

4. **Cross-protocol interactions show path** — if you reply on Mastodon to a Bluesky post, the UI shows: "Reply via Mastodon to [user] on Bluesky" with both protocol badges.

---

## 6. Accessibility & Inclusivity

### 6.1 Screen Reader Compatibility

This is not an afterthought — it's a first-class concern.

**Concrete implementation:**
- All protocol badges include `aria-label` with protocol name (e.g., "Posted via Mastodon, a federated social network").
- Timeline ordering announcements include count: "Showing 50 posts from 42 accounts, ordered chronologically."
- "Acknowledged by [people]" notifications are navigable as a list, not read as a number.
- Every action (follow, mute, block, acknowledge) has a clear audio cue that differs from the standard "button pressed" sound.
- Alt text for images is mandatory when composing to protocols that support it — the composer refuses to post without alt text on ActivityPub and AT Protocol (configurable).

### 6.2 Low-Bandwidth Mode

Many users are on metered connections or slow networks. The app must be usable on 2G.

**Concrete implementation:**
- **Text-first mode** — images and videos are shown as placeholders by default. Tap to load. Settings allow "never auto-load media."
- **Protocol-aware media compression** — images fetched from protocols are downsized to a configurable maximum resolution (default 480px width).
- **Offline queue** — compose and send when you have connectivity. The app queues outbound posts per protocol and sends when online.
- **Data usage dashboard** — per-protocol breakdown of data consumed (fetched vs. sent) in the current billing period.
- **Proxy support** — for users in restrictive networks, the app supports SOCKS5 proxies and Tor (where protocol servers allow it).

### 6.3 Multi-Language Support

Decentralized protocols are global by nature. The interface must be equally global.

**Concrete implementation:**
- Full localization framework from day one. Minimum v1.0 supports: English, Spanish, French, German, Japanese, Arabic, Hindi, Portuguese, Russian, Chinese (Simplified).
- All user-facing strings in a standard i18n format (e.g., Flutter ARB or iOS strings).
- Content language detection — posts in the timeline show a language badge if they're in a language different from the user's UI language.
- Translation integration — optional integration with local ML translation (on-device, privacy-preserving) or user-chosen translation service API.

### 6.4 Cognitive Accessibility

The multi-protocol, multi-identity nature of this app is inherently complex. Complexity is a barrier.

**Concrete implementation:**
- **"Simple mode" toggle** — hides protocol complexity. Shows a unified timeline, one identity, no protocol badges. All cross-protocol details are accessible via a "Details" expandable section but hidden by default.
- **Visual noise reduction settings**:
  - "Minimal" mode: text-only posts, no avatars, no badges, monochrome.
  - "Comfortable" mode: increased line height, larger fonts, more spacing.
  - "High contrast" mode for visual accessibility.
- **Reading mode** — single-post view with no timeline surrounding it. Just the post and its thread. Reduces visual overwhelm.
- **Notification volume controls** — granular sliders per notification type (see 3.3) with clear "this will notify you X times per day" estimate.
- **Session timer** — optional automatic timer that closes the app after a user-defined duration (15 min, 30 min, 60 min). Not a nudge — it actually closes.

---

## 7. Anti-Patterns and What We Refuse To Do

Equally important to what we build is what we explicitly refuse to build.

| Anti-Pattern | Why We Refuse | Alternative |
|---|---|---|
| **Infinite scroll** | Eliminates natural stopping cues | Paginated timeline with hard limits |
| **Pull-to-refresh** | Encourages compulsive checking | Explicit "check for new" button |
| **Unread badges on app icon** | Anxiety-driven re-engagement | None; the app icon shows nothing |
| **Streaks or daily goals** | Gamification of addiction | The app has no streak concept |
| **Auto-playing video** | Passive consumption hijacks attention | Videos are still images; tap to play |
| **"You might like" suggestions** | Algorithmic amplification without consent | The user curates their own networks |
| **Social graph import** | Privacy invasion of address book | Manual search or handle input only |
| **Notification "badge" counts** | Numeric anxiety triggers | People-based indicators only |
| **In-app purchasing / ads** | Misaligned incentives | Paid app or donation model only |
| **Dark patterns (confirm-shaming, hidden unsubscribes)** | Manipulative by definition | None of these patterns exist in the codebase |

---

## 8. References & Inspiration

### 8.1 Applications

| App | What It Does Well | What We Learn |
|---|---|---|
| **Mammoth** (Mastodon) | Clean onboarding; protocol explanation for non-tech users | How to explain federation to normals |
| **Ivory** (Mastodon) | Beautiful native design; timeline customization | That a federated client can feel premium |
| **Damus** (Nostr) | Nostr key management UX; relay selection | How to not scare users with crypto keys |
| **Beeper** (multi-protocol) | Unified inbox; cross-protocol chat | The promise and pitfalls of multi-protocol UI |
| **Signal** | Encryption done seamlessly; key verification UX | Green/gray privacy indicators; emoji key comparison |
| **Arc browser** | Spaces and profiles as organizational primitives | "Spaces" concept for protocol separation |
| **Matter (read-it-later)** | Queue-based consumption; no infinite scroll | "Read later" as an anti-addiction pattern |
| **Cara** (anti-AI social) | Chronological feed as a core feature; no algorithm | That a chronological-only social app can grow |

### 8.2 Design Manifestos & Movements

- **Center for Humane Technology** — "Time Well Spent" principles; the original framing of the attention economy as a design problem.
- **IndieWeb principles** — Own your data; use your domain; POSSE (Publish on Own Site, Syndicate Elsewhere).
- **The Slow Web movement** (Jack Cheng) — "The Slow Web asks: is this real-time connection necessary? Could it be done asynchronously? Could it wait?"
- **Cal Newport's "Digital Minimalism"** — "The Law of the Vital Few": a technology must serve something you deeply value, it must be the best way to serve that value, and it must have constraints on how you use it.
- **BONSAI research** (Bridging Online Networks with Structured Awareness and Intent) — intention-based interaction model for social networks; structured rather than algorithmic content delivery.
- **Decentralized Web Principles** (DWeb) — Privacy by design, user agency, permissionless innovation.

### 8.3 Academic & Research References

1. **BONSAI: Designing Social Media with Structured Awareness and Intent** (Graeber et al., 2024) — The foundational paper for intention-based social interaction design.
2. **The Effect of Social Media on Wellbeing** (Orben & Przybylski, 2019) — Large-scale study showing passive consumption correlates with lower wellbeing; active interaction does not.
3. **The Deletion of Digital Habits** (Lukoff et al., 2021) — Research on friction-based interventions for reducing compulsive social media use.
4. **A Design Space for Digital Wellbeing** (Purohit et al., 2020) — Taxonomy of digital wellbeing interventions, including friction, goal-setting, and awareness tools.
5. **Nostr: A Protocol for Censorship-Resistant Social Networks** (Alotto et al.) — Technical foundations for relay-based social networking.
6. **ActivityPub W3C Recommendation** — The standard for federated social web.

### 8.4 Philosophical Foundations

- **Ivan Illich, "Tools for Conviviality" (1973)** — A convivial tool is one that gives the user control over its purpose and use. This app must be a convivial tool.
- **Tristan Harris, "The Attention Economy" (2016)** — The original framing of attention as a resource being extracted by technology.
- **James Williams, "Stand Out of Our Light" (2018)** — Attention is not just a resource; it's the foundation of human will and autonomy.
- **Shoshana Zuboff, "The Age of Surveillance Capitalism" (2019)** — The economic model this app explicitly rejects.

---

## Appendix A: Implementation Checklist

A living document of concrete UX features to build, organized by priority.

### P0 (Ship-blocking)
- [ ] Intent-based app entry ("What are you here for?")
- [ ] Paginated chronological timeline (default)
- [ ] Protocol badges on all content
- [ ] Notification architecture per 3.3
- [ ] No vanity metrics visible
- [ ] Minimum accessibility (screen reader labels on all interactive elements)

### P1 (Core experience)
- [ ] Per-protocol filters
- [ ] Simple Mode toggle (hides protocol complexity)
- [ ] Low-bandwidth / text-first mode
- [ ] Cross-protocol identity linking (local only)
- [ ] Encryption status indicators (green/yellow/gray/red)
- [ ] Friction patterns (long-press to publish, follow confirmation)

### P2 (Delight & depth)
- [ ] Adaptive composer (protocol-specific formatting suggestions)
- [ ] Sentiment pulse for authors (private, qualitative only)
- [ ] Protocol skins (visual personality per network)
- [ ] "Catch-up complete" state
- [ ] Daily notification digest
- [ ] Session timer (auto-close after N minutes)

### P3 (Stretch)
- [ ] Identity portability wizard
- [ ] "Same person?" cross-protocol identity clustering
- [ ] Import from Twitter/Instagram/Facebook/Reddit dumps
- [ ] Tor/proxy support
- [ ] On-device ML translation

---

## Appendix B: Design Principles Poster

A one-page summary for design reviews:

> **LIBERATION PRINCIPLES**
>
> 1. Intent over Engagement — Every interaction begins with user intent.
> 2. Chronological is Neutral — The default timeline is raw chronological.
> 3. Friction is a Feature — Add friction to compulsive actions; remove it from intentional ones.
> 4. Transparency is Non-Negotiable — Every algorithm and design decision is explainable.
> 5. Numbers Abstract People — Show people, not counts.
> 6. Ownership by Default — Data is local first; portability is one tap away.
> 7. Protocol-aware, User-unified — Respect protocol differences while presenting a coherent interface.
> 8. Simple is Accessible — A "Simple Mode" that hides all protocol complexity is required.
> 9. Refuse the Attention Economy — If a pattern exists primarily to increase engagement, we don't build it.
> 10. Design for Departure — The app should help you spend less time in it, not more.

---

*This document is a living manifesto. Every design decision in the app should trace back to one of these principles. If a feature cannot be justified by them, it does not belong in the Liberation Client.*
