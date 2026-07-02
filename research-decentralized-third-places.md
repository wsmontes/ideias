# Decentralized, Federated, and Open-Protocol Third Places -- Research Synthesis

**Date:** 2026-07-01

---

## 1. Secure Scuttlebutt (SSB): Offline-First Pubs as Local Third Places

### Project Overview
- **Creator:** Dominic Tarr (protocol design, 2014+)
- **Repo:** `ssbc/ssb-server` (Node.js reference implementation, now minimally maintained since ~2022)
- **Website:** `scuttlebutt.nz`
- **Wikipedia:** `/wiki/Secure_Scuttlebutt_(SSB)` (last revised March 2026)
- **Estimated users:** 15,000-30,000 across all apps (Forbes/Mozilla citations)
- **License:** MIT/Beerware

### Architectural Approach

SSB inverts the client-server model. Every peer runs a local database of append-only, cryptographically signed feeds. Data propagates through a **gossip protocol** along social graph edges (friend-of-friend). The internet is not strictly required -- SSB works over LAN, Bluetooth, and sneakernet.

**Pubs** are simply always-online peers that mirror feeds for availability. They emerged as a practical necessity because home routers behind NAT cannot accept incoming connections. The SSB Design Handbook explicitly warns against treating pubs as special: "Avoid Centralization and Singletons."

**Rooms** (introduced 2019, announced on Manyverse blog) are the key third place innovation. Per Andre Staltz's announcement:
- Pubs store data (like "someone's home" -- intimate, private)
- Rooms store nothing, only connect currently online peers (like "a cafe/pub")
- Rooms use open invite codes and web interfaces to connect strangers around shared interests
- The FreedomBox Forum discussion captured the analogy: *"Pub servers are extremely intimate because they actually host and save everyone's feeds... Room servers are public meeting places that only connect user feeds to each other when users are online at the same time. Quite literally, they are like real life pubs."*

### Academic Foundation
| Paper | Year | Focus |
|-------|------|-------|
| Tarr et al., *ACM ICN* | 2019 | Foundational protocol paper -- "identity-centric" approach to information-centric networking |
| Mannell & Smith, *Social Media + Society* | 2022 | 3-year ethnography of SSB governance (Scuttlecamp dispute, Cobudgeting, "community gardening") |
| Vermillion, *ASU Thesis* | 2021 | Whuffie reputation algorithm using conditional probabilities for SSB |
| Depaz, *arXiv* | 2024 | Critical comparison of IPFS and SSB as alternative Internet protocols |
| Mannell & Smith (DOI: 10.1177/20563051221122448) -- Open access, ~43 citations |

Key source: `https://handbook.scuttlebutt.nz/stories/design-challenge-avoid-centralization-and-singletons.html`
Also: `https://hacks.mozilla.org/2018/08/dweb-social-feeds-with-secure-scuttlebutt/` (Mozilla Hacks, 2018)
Also: `https://dl.acm.org/doi/10.1145/3357150.3357396` (ACM ICN 2019 paper)

### What Worked

1. **Offline-first genuinely works.** SSB functions without internet (LAN sync, Bluetooth, mesh). This is a real architectural achievement over federated systems like Mastodon. Demonstrated at festivals and in scenarios with intermittent connectivity.

2. **Pub/Room distinction created a clear social metaphor.** Users intuitively understood the difference between "someone's home" (pub -- private, trust-based) and "a public gathering place" (room -- open, transient). This metaphor maps directly onto Oldenburg's first-place/third-place distinction.

3. **Strong governance experimentation.** The community trialed bounties, rotating adjudicators, elected councils with diversity targets, and Cobudgeting (group allocation of shared funds). The term **"community gardening"** was coined to label and value non-technical contributions (welcoming, mediation, hosting discussions).

4. **Pub operators as community stewards.** Running a pub was a social role, not just a technical one. Pub operators curated their invite codes, set community tone, and often knew their users. This created genuine local third places around specific pubs.

### What Did Not Work

1. **The Pub funding problem.** Pubs cost money to run (server, bandwidth) with no revenue mechanism. They were always considered a "temporary workaround," but no permanent solution fully replaced them until DHT invites arrived in Manyverse. Many pubs came and went.

2. **New user invisibility.** Because SSB propagates content through follow graphs, new users are invisible by default unless someone follows them. Multiple reports of people posting for months with zero engagement (HN discussion 2019: "I used SSB for months... no responses").

3. **Key management.** No "forgot password." Lose your cryptographic key, lose your account. Multi-device impossible -- key is single-device only. This was a fundamental barrier for mainstream users.

4. **Immutability conflicts.** Append-only logs mean nothing can be truly deleted. Apps can honor edit/delete requests, but original content remains on disk forever on every peer that replicated it. Legal and social problems.

5. **Governance blind spots.** Because SSB is decentralized, members see only subsets of discussions. This led to the **2019 Scuttlecamp conflict** when a proposal to incorporate in New Zealand caught many off-guard. The ethnography by Mannell & Smith documents this in detail.

6. **Maintenance slowdown.** The core Node.js reference implementation (`ssbc/ssb-server`) has seen minimal activity since mid-2022. Documentation is described as "sparse and confusing." Newer implementations exist in Go, Rust, and Python but no single authoritative client emerged.

### Key Takeaway
The hardest problems in decentralized social networking are not technical (gossip protocol, append-only logs) but social: onboarding, discoverability, content moderation, and governance sustainability. SSB solved the technical problems beautifully but could not solve the social ones at scale.

---

## 2. Manyverse / Planetary: SSB on Mobile

### Manyverse
- **Creator:** Andre Medeiros (Staltz)
- **Launched:** ~2018 (mobile), desktop later
- **Blog:** `manyver.se/blog`
- **Funding:** Open Collective, NGI Pointer grants
- **Status:** Dormant as of 2025

**Approach and Design Philosophy:**
- Mobile-first SSB client with "offline-first" architecture
- Deliberately stayed at "version zero" -- Staltz framed v1.0 as "a future milestone for mainstream readiness" requiring the app to be "easy and intuitive for anyone to use, as well as (socially) safe and (technically) stable"
- Emphasis on **user agency and attention**: The Connections tab was designed as a gauge -- it "calls your attention with a red color" when something needs it, reflecting the principle that "your attention is a delicate life-resource that belongs to you" (Dec 2021 update)
- Introduced **SSB Rooms** (2019 announcement) and later **DHT invites** to reduce pub dependency

**Current Status:**
- July 2024: Jacob Karlsson (Powersource), who took over after Staltz left, stated he "will not do any more work on Manyverse"
- With 80%+ of the project's manpower gone, the app is in a dormant/sunset state
- Jacob proposed renaming the Open Collective to "P4P development funding" to cover related P2P projects
- **PZP protocol** (previously "ppppp") was created by Staltz as SSB's successor: storage capped ~100MB, multi-device support via Account tangle, DAG messages instead of linear feeds, deletion support, partial replication

Key source: `https://opencollective.com/manyverse/updates/launch-of-the-pzp-protocol-and-the-future-of-manyverse`

### Planetary
- **Creator:** Evan Henshaw-Plath (Rabble, first employee at Odeo/Twitter) and Planetary Social
- **Launched:** ~2020, iOS only
- **GitHub:** `planetary-social/planetary-ios`
- **Website:** `planetary.social` (still live as of Feb 2025 per Wayback Machine, but development stalled)
- **Status:** Dormant; founder pivoted to Nostr

**Approach:**
- **Curated "topical pubs"** for onboarding -- themed servers (e.g., music, design, tech) that new users could join based on interests
- **Planetary.name hub** -- a central alias service for easy-to-remember usernames
- Focus on aesthetic, mainstream-friendly UI to make SSB accessible to non-technical users
- Last release mention of topical pubs: v2.1.2 ("Remove topical pubs that are shutting down")

**The Pivot to Nostr:**
- By 2024-2025, Henshaw-Plath had strategically shifted from Planetary (SSB) to **Nos.social** (Nostr protocol)
- He describes this as part of an ongoing quest for decentralized social media, with Nostr offering better sustainability and development momentum
- Multiple podcast interviews document this transition (We Distribute S2E3, RevolutionZ Ep 286, Decentered Podcast)

**Key Challenges for SSB Mobile:**
- **Storage**: SSB's full-replication model is problematic on mobile devices with limited storage. PZP's 100MB cap was a direct response.
- **Onboarding**: The key management and pub-discovery flow was too complex for mainstream users
- **Single-device**: SSB's key model cannot handle multi-device use, a baseline expectation for mobile users
- **Network effects**: Without critical mass, SSB social graphs remained small and engagement low
- **Funding**: Open Collective grants and NGI funding were insufficient to sustain continuous development

**What Worked in SSB Mobile UX:**
- Offline-first SSB replication was genuinely useful for travelers, festival-goers, and low-connectivity areas
- The transparent connection model (showing actual peer relationships) was refreshing for privacy-conscious users
- Rooms + aliases partially solved the discoverability and naming problems

---

## 3. Aether: Ephemeral, Self-Moderating P2P Communities

### Project Overview
- **Founder:** Burak Nehbit (ex-Google YouTube UX, ex-Facebook design lead)
- **Co-founder:** Benedict Lau (joined 2019)
- **Company:** Aether Technologies, Inc. (San Francisco)
- **Domain:** `getaether.net` (registered Nov 2012, renewed through Nov 2027)
- **GitHub:** `github.com/aethereans/aether-app` (1,249 stars, 44 forks, last code commit July 2021)
- **License:** AGPL-3.0
- **Status:** Effectively abandoned since mid-2021; website still up, code dormant

### Architectural Design

**Protocol:** Custom **Mim Protocol** (v0.1) -- "a purely functional distributed network": stateless, no distributed consensus, no blockchain.
**Network topology:** **Flood network** -- every node stores and serves all content. Different from federated models (ActivityPub) where data is partitioned across instances.
**Data structure:** DAG (Directed Acyclic Graph) stored in SQLite.
**Tech stack:** Go (backend/graph compiler) + Vue.js (frontend) + Electron (desktop shell) + gRPC (IPC). ~100,000+ lines of code.
**Identity:** Keypair-based, "minted" by solving modified Hashcash proof-of-work at signup.
**Peer discovery:** Bootstrap nodes + probabilistic search; only the last hop is visible (source anonymity).

### Ephemeral Content Model
- **Default lifetime:** Content auto-deleted after 6 months of inactivity
- **Active propagation:** Only last 2 weeks actively propagated
- **Protocol-level enforcement:** Deletion guaranteed at protocol level -- even modified clients cannot retrieve old content from other nodes
- **Rationale:** "The freedom to be wrong, and move on"

### Self-Moderation System (Aether's Most Innovative Feature)
- **Democratic elections:** Triggered at 100+ active users, requires 51% positive vote from 5% of community
- **Local override (key innovation):** Any user can disable a moderator locally, reverting all their actions
- **Full transparency:** All moderator actions public and auditable; deletions reveal who, why, and how to restore
- **Client-side block lists** for illegal content compliance
- **Subscribe to filters:** Users can adopt moderation filters from trusted third parties

### Project Timeline
| Date | Milestone |
|------|-----------|
| Nov 14, 2012 | Domain registered |
| 2012-2013 | Initial dev in Python/Twisted with Qt5 |
| Nov 2013 | First public announcement (Twisted mailing list + HN) |
| Nov 2013 | The Verge covers as "a Reddit for the privacy-conscious" |
| Jan 2014 | Google hires Nehbit (age 23) for YouTube UX team |
| ~2017 | Complete rewrite: Python -> Go + Vue.js + Electron |
| Nov 2018 | Aether v2 first public release (soft launch) |
| Feb 2019 | 500+ concurrent nodes, HN front page |
| Jun 2021 | Launched "Aether Hosted Universes" (later Aether Pro) |
| Mid-2021 | Blog goes silent (last of 14 posts) |

### Community Size
- ~2,000 MAU on P2P app (2021)
- 500+ concurrent nodes (Feb 2019 peak)
- ~1,200 GitHub stars

### Why It Did Not Succeed
1. **Extremely low adoption:** 2,000 MAU is too small for network effects
2. **Desktop-only:** No mobile app -- fatal for a social platform in the 2020s
3. **Pre-alpha quality:** README explicitly says "unstable and untested, not production-ready"
4. **Technical complexity:** 100K+ lines, multi-language toolchain, hours to compile
5. **No VC funding:** Relied on Patreon and Aether Pro SaaS ($10/user/month) -- not enough
6. **Pivot to SaaS diluted mission:** Hosted product undercut the decentralized value proposition
7. **Founder opportunity costs:** Nehbit's Google/Facebook roles likely consumed his attention
8. **Toxic user reports:** Multiple reports of alt-right/fascist presence; self-moderation could not prevent bad actors, only filter them locally

### Legacy
Aether is notable as one of the earliest serious attempts at a decentralized Reddit alternative (predating Lemmy's 2019 launch). Its "subjective moderation" concept was genuinely innovative -- giving users granular control while maintaining transparency. The project demonstrated both the technical feasibility of P2P discussion platforms and the fundamental challenge of user adoption.

Key sources:
- `https://getaether.net`
- `https://github.com/aethereans/aether-app`
- `https://ricmac.org/2021/07/26/aether-a-decentralized-reddit-with-self-moderation-and-privacy/`
- `https://www.theverge.com/2013/11/27/5150758/aether-aims-to-be-a-reddit-for-the-privacy-conscious`
- `https://mail.python.org/archives/list/twisted@python.org/message/4HT3ZC6LAAREOZQSPK5DW55DXOOIREFR/`

---

## 4. Matrix Spaces: Federated Chat as Third Places

### Project Overview
- **Created by:** The Matrix.org Foundation (Amandine Le Pape, Matthew Hodgson)
- **Protocol:** Matrix (open standard, Apache 2.0)
- **Server:** Synapse (reference implementation), Dendrite, Conduit
- **Key client:** Element (desktop, mobile, web)
- **Spaces launched:** 2021 (GA)
- **Website:** `matrix.org`

### Architectural Approach

**Rooms:** Persistent chat spaces with state resolution v2, E2EE, event DAG per room. Rooms exist on specific homeservers but are federated -- users on any server can join and participate.

**Spaces (launched 2021):** Rooms with `type: "m.space"` that act as hierarchical containers. Key features:
- `m.space.child` state events define the hierarchy
- **Hierarchy API** (MSC2946) for navigable room listings within a space
- **Restricted rooms** (MSC3083, room version 8+) -- membership grants from being in a parent space
- Spaces can contain both rooms and sub-spaces, enabling nested community structures

### How Spaces Function as Third Places

- **Federation model** allows one-identity access to communities across independent servers
- **Third Room** project (2022-2024, `thirdroom.io`) explicitly built 3D spatial worlds on Matrix for social gatherings
- **Commune** (project by Element) was designed as a public-by-default Discord alternative with Spaces as the organizing primitive, targeting third-place use cases
- **Community governance** operates at two levels: homeserver-level (admin controls) and room-level (power levels, moderation)

### Adopted Communities
- **Mozilla** -- uses Matrix for community chat (thousands of users)
- **KDE** -- fully migrated to Matrix for community communication
- **FOSDEM** -- conference with 57k+ Matrix users during events
- **Ubuntu** -- official community chat on Matrix
- **Gitter** -- acquired by Element, migrated to Matrix protocol
- **Public sector:** Germany (Bundeswehr), France, Netherlands government deployments

### Challenges as Third Places

1. **Onboarding funneled through matrix.org.** While the protocol is federated, most new users join via the matrix.org homeserver, creating a single point of failure and control. Server choice is confusing for new users.

2. **Poor room discoverability.** Matrix has no built-in public room directory across servers. Finding communities is difficult. (Feb 2025: a curated directory was introduced to address this.)

3. **CSAM spam via federation.** Malicious actors can federate illegal content from any server. This forced Matrix to implement aggressive anti-spam measures and reputation systems.

4. **Historically weak moderation tooling.** Initially, moderation was limited to room-level bans and server-level blocks. The ecosystem has since matured:
   - **Draupnir / Mjolnir bots** for automated moderation
   - **Shared ban lists** (MSC3848) for cooperative moderation
   - **Policy servers** (MSC4284) for proactive moderation
   - **Bulk redaction** (MSC4194)
   - Room versions v8-v12 progressively hardened the ecosystem

5. **Tension between decentralization and safety.** Matrix's federation model means no single entity can enforce community standards across the network. Effective third places require trust and safety guarantees that are harder to maintain in federated environments.

### Matrix vs ActivityPub (Context)
Matrix prioritizes **consistency and E2EE for real-time chat**; ActivityPub prioritizes **availability for social media**. Most analyses conclude they complement rather than compete. Matrix rooms are better as real-time third places (conversation as main activity); ActivityPub instances are better as broadcast/publication platforms.

Key sources:
- `https://matrix.org`
- `https://spec.matrix.org`
- Oshinowo et al. (2025) -- "Seeing the Politics of Decentralized Social Media Protocols," arXiv:2505.22962
- Kim et al. (2025) -- "Discord's Design Encourages Third Place Social Media Experiences," arXiv:2501.09951

---

## 5. Cross-Cutting Analysis and Design Principles

### The Third Places Framework (Oldenburg, 1989)

Oldenburg's eight characteristics, applied to digital decentralized spaces:

| Characteristic | Digital Application |
|---|---|
| Neutral Ground | Free to come and go; protocol-level permissionless access |
| Leveler | Pseudonyms, cryptographic identities reduce real-world hierarchy |
| Conversation as main activity | Text/voice chat as primary mode (Matrix rooms, SSB threads) |
| Accessibility & Accommodation | Always-on, federation/P2P transcends time zones |
| The Regulars | Trust graph (SSB), room regulars (Matrix) who set the tone |
| Low Profile | Plain design, focus on substance over algorithmic feed |
| Playful Mood | Memes, inside jokes, shared games in community spaces |
| Home Away From Home | Sense of comfort and belonging through persistent community |

### Cross-Protocol Comparison

| Dimension | SSB | Matrix | Aether | ActivityPub (Mastodon) | AT Protocol (Bluesky) | Farcaster |
|-----------|-----|--------|--------|----------------------|----------------------|-----------|
| Network type | Pure P2P (gossip) | Federated | Pure P2P (flood) | Federated | Federation-lite | Hub + chain |
| Identity | Public key | Homeserver UID | PoW-keypair | Server domain | DID (portable) | ENS on-chain |
| Offline-first | Yes | No | Yes | No | No | No |
| Moderation | Trust graph + blocking | Room/server power levels | Subjective (local override) | Per-instance | Composable labelers | App-level |
| Third place scale | Neighborhood (trust graph) | Guild hall (room) | Town square (community) | Town (instance) | City of towns (protocol) | Gated community |
| Maturity | Mature P2P, niche | Mature chat standard | Abandoned | Most mature ecosystem | Rapidly growing | Niche |
| Key weakness | Key mgmt, discoverability | Onboarding, moderation | Adoption, desktop-only | Identity portability | Current centrality | Crypto UX barrier |

### Synthesized Design Principles for Digital Third Places

1. **Themed persistent spaces anchor community identity.** Dedicated channels/rooms/communities with clear topical focus create a sense of place. (Kim et al., 2025; SSB rooms, Matrix Spaces)

2. **User autonomy and customization build ownership.** Custom roles, server branding, and user-controlled governance create investment in the space. (Kim et al., AT Protocol ethos, Neighbourhoods Network)

3. **Composable moderation enables trust at scale.** Communities define their own norms; users choose which moderation filters to subscribe to. (Aether's subjective moderation, AT Protocol labelers, SSB's local blocking)

4. **Portable identity protects user investment.** Identity not tied to a server means community membership carries across spaces. (AT Protocol DID, SSB keypair, Farcaster ENS) -- ActivityPub's identity-per-server is a structural weakness.

5. **Low-barrier entry with tiered engagement.** Preview before joining, lurk before participating. (SSB's "invisibility of new users" was a design failure here; Matrix Spaces improved on this with restricted rooms)

6. **Conversation over broadcast.** Text/voice as primary mode, not algorithmic content feeds. Decentralized spaces succeed where they facilitate conversation, not broadcast. (Oldenburg-derived, consistently validated)

7. **Ephemerality or low stakes reduces toxicity.** Transient content reduces performance pressure and enables candor. (Aether's 6-month TTL, Discord chat flow, Signal disappearing messages)

8. **Local governance with escape hatches.** Community self-governance (elected moderators, power levels) combined with "credible exit" -- the ability to leave without losing your identity or connections. (Oshinowo et al. 2025, AT Protocol "lazy trust")

### Key Takeaways

1. **Pure P2P is technically elegant but socially fragile.** SSB and Aether both demonstrated that full decentralization (no servers at all) creates fundamental challenges in moderation, discoverability, and UX that federated models handle better.

2. **Federation (Matrix) is the most viable middle ground for third places.** It provides consistent identity, room-level governance, and real-time interaction while avoiding the cold-start problem of pure P2P. Matrix Spaces map most cleanly onto Oldenburg's third place concept.

3. **Mobile access is non-negotiable.** Every project that failed to achieve mobile adoption (Aether) or struggled with it (SSB) failed as a social platform. Planetary and Manyverse showed that even well-designed mobile SSB clients could not overcome the protocol's structural barriers (storage, key management).

4. **Funding sustainability is the unsolved problem.** No successful long-term funding model has emerged for decentralized social platforms. Open Collective grants sustain experimentation but not production. Aether's SaaS pivot diluted its mission. SSB's pub operators paid out of pocket.

5. **Moderation at protocol level remains an open challenge.** Every project struggled with this differently: SSB's trust graph was insufficient at scale, Aether's self-moderation required too much user effort, Matrix's federation made enforcement complex, and ActivityPub's instance-level model creates inconsistency.

6. **The most successful protocol-level third places are chat, not feed.** Real-time conversational spaces (Matrix rooms, SSB rooms, Discord servers) map more naturally onto Oldenburg's third place characteristics than algorithmic content feeds (Mastodon, Bluesky). Conversation is the "main activity" of a third place; content consumption is not.

---

## Sources (Comprehensive List)

### Secure Scuttlebutt
- `https://handbook.scuttlebutt.nz/stories/design-challenge-avoid-centralization-and-singletons.html` -- SSB Design Handbook
- `https://hacks.mozilla.org/2018/08/dweb-social-feeds-with-secure-scuttlebutt/` -- Mozilla Hacks introduction
- `https://dl.acm.org/doi/10.1145/3357150.3357396` -- ACM ICN 2019 foundational paper (Tarr et al.)
- `https://journals.sagepub.com/doi/full/10.1177/20563051221122448` -- Mannell & Smith 3-year ethnography
- `https://arxiv.org/abs/2404.02158` -- Depaz protocol critique
- `https://keep.lib.asu.edu/items/147661/metadata` -- Vermillion ASU thesis on Whuffie reputation
- `https://www.manyver.se/blog/announcing-ssb-rooms` -- SSB Rooms announcement
- `https://discuss.freedombox.org/t/private-social-networks-using-secure-scuttlebutt/49/5` -- FreedomBox third place analogy
- `https://www.theatlantic.com/technology/archive/2017/05/meet-the-counterantidisintermediationists/527553/` -- Atlantic profile of Dominic Tarr
- `https://news.ycombinator.com/item?id=20162613` -- HN user experience reports
- `https://blog.opencollective.com/enriching-commons-by-practicing-with-money/` -- SSB Cobudgeting governance
- `https://cryptovalley.swiss/290-dominic-tarr-secure-scuttlebutt-the-localized-but-distributed-social-network/` -- Epicenter podcast
- `https://dev.to/rook_damon/why-agora-is-not-scuttlebutt-5bgm` -- Critical analysis of SSB limitations

### Manyverse / Planetary
- `https://www.manyver.se/blog/2021-04-update` -- Staltz April 2021 update (SSB mobile progress)
- `https://www.manyver.se/blog/2021-12-update/` -- December 2021 update (design philosophy)
- `https://opencollective.com/manyverse/updates/launch-of-the-pzp-protocol-and-the-future-of-manyverse` -- Jacob Karlsson July 2024 status
- `https://github.com/planetary-social/planetary-ios/releases` -- Planetary release history
- `https://web.archive.org/web/20250219213458/https://www.planetary.social/` -- Planetary website snapshot
- `https://wedistribute.org/podcast/s2e3-rabble-from-nos-social/` -- Rabble on Nos.social pivot (2025)

### Aether
- `https://getaether.net` -- Official site
- `https://github.com/aethereans/aether-app` -- GitHub repo
- `https://getaether.net/mim-docs/` -- Mim Protocol documentation
- `https://ricmac.org/2021/07/26/aether-a-decentralized-reddit-with-self-moderation-and-privacy/` -- Richard MacManus review
- `https://www.theverge.com/2013/11/27/5150758/aether-aims-to-be-a-reddit-for-the-privacy-conscious` -- The Verge 2013 launch coverage
- `https://mail.python.org/archives/list/twisted@python.org/message/4HT3ZC6LAAREOZQSPK5DW55DXOOIREFR/` -- Original announcement
- `https://archive.org/details/12120iadweb` -- DWeb SF talk at Internet Archive
- `https://news.ycombinator.com/item?id=6787807` -- HN launch thread (2013)
- `https://news.ycombinator.com/item?id=18370208` -- HN Aether v2 discussion (2018)

### Matrix
- `https://matrix.org` -- Official site
- `https://spec.matrix.org` -- Protocol specification
- `https://arxiv.org/abs/2505.22962` -- Oshinowo et al. politics of decentralized protocols

### Cross-Cutting / Theory
- `https://doi.org/10.1177/1461444806061953` -- Soukup (2006) CMC as virtual third place
- `https://onlinelibrary.wiley.com/doi/full/10.1111/j.1083-6101.2006.00300.x` -- Steinkuehler & Williams online games as third places
- `https://journals.sagepub.com/doi/full/10.1177/2056305116665857` -- McArthur & White Twitter chats as third places
- `https://ar5iv.labs.arxiv.org/html/2501.09951` -- Kim et al. (2025) Discord's design encourages third places
- `https://arxiv.org/abs/2505.22962` -- Oshinowo et al. politics of decentralized protocols
- `https://ar5iv.labs.arxiv.org/html/2402.03239` -- Kleppmann et al. AT Protocol analysis
- `https://atproto.com/articles/atproto-ethos` -- AT Protocol ethos / design principles
- `https://arxiv.org/pdf/2306.13941` -- Shapiro Grassroots Social Networking (encoding locality)
- `https://www.chaincatcher.com/en/article/2058521` -- Jay Graber (Bluesky) SSB vs Matrix vs Aether comparison
- `https://journals.sagepub.com/doi/10.1177/20563051251399015` -- van Ommen & Yahanashi emergent third places in Discord
- `https://overreacted.io/open-social/` -- Dan Abramov "Open Social" essay
