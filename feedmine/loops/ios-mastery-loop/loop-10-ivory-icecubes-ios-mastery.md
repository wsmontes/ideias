# Loop 10: Ivory & Ice Cubes — How Mastodon Clients Master Open-Protocol iOS Design

> A deep research document comparing two approaches to building a native iOS social feed client on an open protocol (ActivityPub/Mastodon): Ivory (polished commercial app by Tapbots, makers of Tweetbot) and Ice Cubes (open-source SwiftUI showcase by Thomas Ricouard). What Feedmine can learn about building a native client for an open protocol — the architecture, the design language, and the competitive advantage of open standards.

---

## 1. Executive Summary

Ivory and Ice Cubes represent two philosophies for building native iOS clients on open protocols — directly relevant to Feedmine (which builds on open RSS/Atom). Both prove that a third-party client can be BETTER than first-party apps when built with deep platform knowledge.

**Ivory (Tapbots):**
- Commercial, polished, 12+ years of Twitter client experience ported to Mastodon
- "Beautiful visual and sound design" — obsessive craft
- Timeline position sync across devices (rock-solid)
- Subscription model ($1.99/month or $14.99/year)
- Available on iOS, iPadOS, macOS (native AppKit)

**Ice Cubes (Thomas Ricouard):**
- Fully open-source (GitHub, 80+ contributors, 15+ languages)
- Built entirely in SwiftUI (iOS 16+) — a modern architecture showcase
- Swift Package-based modular architecture (MVVM, NavigationStack, EnvironmentObject)
- Ships fast: MVP to App Store in ~1 month, daily updates
- 50,000+ downloads, 4.8 rating

**The lesson for Feedmine:** These apps prove that open protocols (ActivityPub for Mastodon, RSS/Atom for feeds) enable client diversity — where native design quality becomes the competitive differentiator, not data access. Feedmine's advantage: RSS is even MORE open than Mastodon's API. No authentication, no rate limits, no API changes. The playing field is the client experience.

---

## 2. Ivory — Platform Integration

### 2.1 Timeline Position Sync

Ivory's signature feature: **never lose your place in the timeline across devices.**

- Read position syncs between iPhone, iPad, and Mac
- Open Ivory on Mac → continue exactly where you left off on iPhone
- Uses Tapbots' own sync service (not iCloud)
- "Rock-solid timeline position retention" — emphasized in marketing because it's genuinely hard to implement

**Feedmine parallel:** Scroll position memory PER feed/view, synced across devices (if Feedmine ever adds multi-device via iCloud).

### 2.2 Multiple Custom Timelines

- Create custom timelines from people or hashtags organized into lists
- Pin timelines for quick access
- View multiple timelines side by side in columns (Mac/iPadOS)
- Quickly switch between timelines via customizable bottom bar

### 2.3 Visual & Sound Design

- Beautiful custom animations
- Sound effects for actions (can be disabled)
- Multiple theme options
- Custom app icon choices
- "We obsess over every detail and put good design and experience first"

### 2.4 Powerful Mute Filters

- Regex-based mute filters
- Filter by keyword, user, client, content type
- Temporary or permanent mutes
- Content warning awareness

**Feedmine parallel:** Feed-level and topic-level muting/filtering with the same granularity.

### 2.5 Post Statistics

- Daily post engagement stats
- Weekly activity graphs
- Notification analytics

### 2.6 Multi-Platform (Native on Each)

| Platform | Implementation |
|----------|---------------|
| iOS | Native UIKit/SwiftUI |
| iPadOS | Multi-column sidebar layout |
| macOS | Native AppKit with menu bar icon |

Mac-specific: Share Extension, Handoff, extensive keyboard shortcuts, menu bar quick-access, Intel support.

### 2.7 Customizable Navigation

- Bottom tab bar with 5 configurable slots
- Choose from: Home, Mentions, Notifications, Search, Lists, Favorites, Bookmarks, Stats, Filters, Profile
- Each user builds their own navigation structure

### 2.8 Notifications & Push

- Customizable push notifications
- Choose which notification types trigger push
- Rich notification previews
- Notification tab with filtering

### 2.9 Content Handling

- Content warnings (CW) display/hide
- Polls: create and participate
- GIF search and posting
- Hashtag following
- Bookmark posts privately
- Local and federated timeline browsing
- Trending posts view

---

## 3. Ice Cubes — Architecture & Platform Integration

### 3.1 100% SwiftUI Architecture

Ice Cubes is built entirely with SwiftUI, using the latest APIs:

| Component | Technology |
|-----------|-----------|
| UI Framework | SwiftUI (iOS 16+) |
| Navigation | NavigationStack with programmatic path |
| State Management | @StateObject, @EnvironmentObject, MVVM |
| Image Loading | Nuke (high-performance image library) |
| Package Management | Swift Packages (modular by domain) |
| Networking | URLSession + Codable |
| Multi-platform | iOS tab bar, iPadOS sidebar, macOS via Catalyst |

### 3.2 Swift Package Modular Architecture

The app is split into domain-specific Swift Packages:
- `Status` — post display and interaction
- `Account` — user profiles
- `Timeline` — feed display and management
- `Notifications` — push and in-app
- `Env` — shared environment objects
- `Network` — API client
- `Models` — data models

**Why this matters:** Each package builds independently, enables isolated testing, and speeds up SwiftUI previews. This is the architecture pattern Feedmine should follow.

### 3.3 Centralized Router Pattern

Ice Cubes uses a centralized `RouterPath` ObservableObject:
- Defines all possible navigation destinations as an enum
- Defines all possible sheets as an enum
- NavigationStack's path bound to the router
- Any view can trigger navigation by appending to the path
- Sheets presented via a single published property

This pattern provides:
- Type-safe navigation (no string-based routes)
- Programmatic deep linking
- Back navigation support
- Testable navigation logic

### 3.4 SwiftUI Native Components Used

- `ToolbarTitleMenu` — tap title for navigation options
- `.swipeActions()` — swipe gestures on list items
- `NavigationStack` with `NavigationLink`
- `@EnvironmentObject` for dependency injection
- System blur/vibrancy materials
- SF Symbols throughout

### 3.5 Remote Timeline Pinning (Killer Feature)

Ice Cubes' unique differentiator: browse ANY Mastodon server's local timeline without having an account there.

- Add any server URL
- Browse its local timeline
- Pin multiple remote timelines
- Discover content from topic-specific communities

**Feedmine parallel:** This maps to "discover new feeds" — the ability to browse curated feed collections or trending sources without subscribing first.

### 3.6 Open Source Benefits

- 80+ contributors, 400+ merged PRs
- Available in 15+ languages (community translations)
- 50,000+ downloads
- Community-driven feature development
- Transparent development process
- Learning resource for SwiftUI developers

### 3.7 Performance Optimizations

- Minimal view updates during scrolling (targeted @ObservedObject usage)
- Nuke library for efficient image caching/loading
- Split EnvironmentObjects to prevent unnecessary re-renders
- View composition for encapsulated updates

---

## 4. Comparative Analysis

| Aspect | Ivory | Ice Cubes |
|--------|-------|-----------|
| **Architecture** | UIKit + SwiftUI hybrid | 100% SwiftUI |
| **Business model** | Subscription ($14.99/yr) | Free, open source |
| **Polish level** | Extremely high (12yr experience) | High, improving rapidly |
| **Timeline sync** | Cross-device via own server | Local only |
| **Multi-platform** | Native on each (AppKit Mac) | iOS + Catalyst Mac |
| **Sound design** | Custom sounds | System defaults |
| **Theme options** | Multiple polished themes | Theme support |
| **Community** | Closed development | Open GitHub |
| **Update frequency** | Monthly releases | Almost daily |
| **Unique feature** | Timeline position sync | Remote timeline pinning |

---

## 5. What Feedmine Can Steal

### 5.1 Open Protocol = Client Diversity = Design Wins

Both Ivory and Ice Cubes exist because Mastodon uses ActivityPub (open protocol). RSS/Atom is even more open — no auth, no rate limits, no API owner who can kill you.

**For Feedmine:** Feedmine's structural advantage is ABSOLUTE openness. No company can shut down RSS access. Communicate this explicitly: "Your feeds belong to you. No company controls your access."

### 5.2 Timeline Position Sync (Ivory's Killer Feature)

Never losing your place is the #1 UX requirement for a feed client used across devices.

**For Feedmine:** If multi-device is ever added, scroll position per-feed MUST sync. This is harder than it sounds (different screen sizes, different amounts loaded) but Ivory proves it's achievable and invaluable.

### 5.3 Ice Cubes' Swift Package Architecture

The modular Swift Package approach is the ideal architecture for Feedmine:

**For Feedmine:**
- `FeedEngine` — RSS/Atom parsing + refresh logic
- `RankingEngine` — scoring + diversity + personalization
- `Models` — article, source, user preference data models
- `UI` — SwiftUI views (article row, feed list, reader)
- `Storage` — SQLite/SwiftData persistence
- `Env` — shared environment (user model, session state)

Each package builds independently, enables previews, and supports testing.

### 5.4 Centralized Router for Deep Linking

Ice Cubes' RouterPath pattern enables:
- URL scheme handling (`feedmine:///article?id=...`)
- Spotlight search deep links
- Notification tap → navigate to article
- Widget tap → navigate to specific view
- Shortcuts → programmatic navigation

### 5.5 Customizable Navigation Bar (Ivory)

Ivory lets users pick which 5 views appear in the bottom tab bar. Different users need different primary views.

**For Feedmine:** Let users configure the tab bar:
- Default: Feed, Discover, Saved, Settings
- Customizable: swap in Topics, Sources, Podcasts, Search, Stats

### 5.6 Mute Filters with Regex

Ivory's powerful mute system (regex, keywords, temporary/permanent) maps perfectly to feed filtering.

**For Feedmine:**
- Mute keywords ("crypto", "NFT") from all feeds
- Mute specific sources temporarily ("hide [Source] for 2 weeks")
- Regex-based advanced filters for power users
- Content-type muting (hide all video posts, show only articles)

### 5.7 Remote Timeline Discovery (Ice Cubes)

Browse other servers' content without commitment → subscribe if you like it.

**For Feedmine:**
- "Discover" tab showing curated feed collections by topic
- Preview a feed's recent content before subscribing
- "Trending in [Topic]" showing popular articles across sources
- Browse OPML directories to find new feeds

### 5.8 Open Source as Competitive Advantage (Ice Cubes)

Ice Cubes got 80 contributors, 15+ translations, and 50K downloads because it's open source.

**For Feedmine:** Open source IS Feedmine's strategy. The benefits are identical:
- Community contributions (bug fixes, features, translations)
- Trust (users can verify no tracking/data collection)
- Learning resource (attracts developer attention)
- Marketing (open source gets press, GitHub stars, word-of-mouth)

### 5.9 Performance-Conscious SwiftUI

Ice Cubes demonstrates that SwiftUI CAN perform well for feed scrolling IF you:
- Minimize @Published property updates during scroll
- Use targeted EnvironmentObjects (split by concern)
- Compose views from small, encapsulated subviews
- Use efficient image loading (Nuke/Kingfisher)
- Avoid unnecessary re-renders through careful state management

---

## 6. Anti-Patterns Both Apps Avoid

1. **No algorithmic feed manipulation.** Timeline is chronological or user-configured. Transparent.
2. **No ads injected into the timeline.** Clean feed, always.
3. **No engagement tricks.** No "you haven't opened in 3 days" guilt notifications.
4. **No data harvesting.** Privacy-first on open protocols.
5. **No platform lock-in.** Switch clients anytime (Mastodon account portable). Switch feed readers anytime (OPML export).
6. **No web views for content.** Native rendering throughout.

---

## 7. Key Takeaways for Feedmine

1. **Open protocols enable client excellence.** When you don't own the data layer, you compete on experience quality. RSS being open means Feedmine's moat is the iOS experience itself.

2. **Swift Package modular architecture is the gold standard for new SwiftUI apps.** Domain-split packages with MVVM and centralized navigation = maintainable, testable, previewable.

3. **Timeline position memory is non-negotiable for feed clients.** Users will abandon apps that lose their scroll position.

4. **Customizable navigation respects diverse use cases.** Not everyone uses a feed reader the same way. Let users configure their primary views.

5. **Mute/filter systems need power AND simplicity.** Keyword mutes for basic users, regex for power users. Temporary mutes for trending annoyances.

6. **Open source multiplies development velocity.** 80 contributors > 1 developer. Feedmine should actively cultivate contributors.

7. **Both "12 years of craft" (Ivory) and "ship fast, iterate daily" (Ice Cubes) can produce excellent products.** Feedmine should combine both: ship an MVP fast, then polish relentlessly.

8. **Design for the open protocol's strengths.** Mastodon's strength is decentralization → remote timeline pinning. RSS's strength is universality → any website can be a feed source, no permission needed.

---

*Document generated: 2026-07-06. Research rounds: Broad (Ivory features/positioning, Ice Cubes open-source story), Focused (Ivory timeline sync, mute filters, multi-platform; Ice Cubes SwiftUI architecture, NavigationStack router, package structure), Gaps (performance optimization, remote timeline discovery, navigation customization, comparative analysis). Sources: tapbots.com, Apple App Store stories, 9to5Mac, Thomas Ricouard Medium articles, GitHub repository, Apple Design Awards 2023.*
