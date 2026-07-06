# Loop 01: Things 3 — How Cultured Code Masters iOS

> A deep research document on how Things 3 leverages every layer of the Apple platform — from haptics to Vision Pro — and what Feedmine can learn about building an app that *belongs* on iOS.

---

## 1. Executive Summary

Things 3 by Cultured Code is arguably the single best example of iOS platform mastery in the App Store. It has won the Apple Design Award **twice** (2009 and 2017), ships native apps across every Apple platform (iPhone, iPad, Mac, Apple Watch, Apple Vision Pro), and has maintained a one-time purchase model for over 8 years — proving that deep platform investment pays for itself.

**What makes it the reference standard:**

- **Every new iOS feature, adopted on day one.** Interactive widgets (iOS 17), Lock Screen widgets (iOS 16), StandBy mode, Smart Stack on watchOS 10, Liquid Glass (iOS 26), Windowed Apps on iPadOS 26, Control Center toggles, Vision Pro spatial design — Things ships support for each within weeks of OS release.
- **Keyboard and touch as first-class equals.** The iPad app supports 70+ keyboard shortcuts, full arrow-key selection, and "Type Travel" (type-to-navigate), while the touch experience has custom gestures (Magic Plus, swipe-to-select, drag-to-reorder) that feel like they were designed by Apple themselves.
- **Full-stack Swift.** Client apps are SwiftUI. Cloud sync was rewritten entirely in server-side Swift (Vapor + SwiftNIO), running on AWS with 3x cost reduction and dramatically faster sync. Same language, client to server.
- **Zero friction philosophy.** No accounts, no onboarding walls. Sync "just works" via iCloud identity. Haptic feedback confirms every action. Animations are fluid and informational, never decorative.

**The lesson for Feedmine:** Things 3 proves that mastering the platform IS the product strategy. The app's competitive moat is not feature count — it's the quality of integration with iOS. Every new Apple API becomes a new surface where Things delivers value.

---

## 2. Platform Integration Inventory

### 2.1 Widgets Ecosystem (WidgetKit)

Things implements widgets across every available surface:

| Surface | What It Shows | Interactive? |
|---------|--------------|-------------|
| Home Screen (small/medium/large) | Today's to-dos | ✅ Yes — tap checkbox to complete |
| Lock Screen (iOS 16+) | Upcoming count or next to-do | No (glanceable) |
| StandBy Mode (iOS 17+) | Large-format today list | ✅ Yes — tap to complete |
| iPad Lock Screen | Strip along left side or above clock | No |
| Apple Watch Smart Stack (watchOS 10+) | Next 3 to-dos | Glanceable |
| Vision Pro workspace | Freely placed in immersive space | ✅ Yes |

**Key implementation details:**
- Widgets now support four appearance styles: Default, Dark, Tinted, and Clear (iOS 26)
- Interactive widgets use SwiftUI Button with AppIntent behind the scenes — completing a to-do from the widget triggers the same sync engine without opening the app
- Widget content is timeline-based, refreshing when to-dos change or at the start of each day

**What Feedmine can steal:** Interactive widgets that mark articles as "read" or "saved" without launching the app. A StandBy widget showing "morning briefing" items. Lock Screen widgets showing unread count by topic.

### 2.2 Control Center & Quick Controls

Things 3.22 added:
- **macOS Control Center toggles:** "New To-Do" and "List" (with user-configurable destination)
- **Apple Watch control:** "New To-Do" accessible via swipe, typing, or voice

These use the ControlWidget API (iOS 18+/macOS 15+), making the app accessible from system-level UI without ever opening it.

**What Feedmine can steal:** Control Center toggle for "Save current clipboard as bookmark" or "Show next unread article."

### 2.3 Apple Watch (watchOS)

Things on Apple Watch is a standalone experience:
- View and reorder Today list
- Mark to-dos complete with haptic confirmation
- Create new to-dos via voice, scribble, or typing
- Smart Stack widget (watchOS 10+) for passive glancing
- Complication showing next to-do or today count

The watch app is **built entirely with SwiftUI** (rebuilt for watchOS 10), enabling rapid iteration on the latest platform capabilities.

**What Feedmine can steal:** A watch complication showing "3 unread highlights" or a Smart Stack widget with the morning's top article headline.

### 2.4 Apple Vision Pro (visionOS)

Things for Vision Pro is a **dedicated native app** — not a compatibility-mode iOS port:
- Designed for the infinite spatial canvas
- Uses the volumetric window model (standard SwiftUI window)
- Widgets can be placed freely in the user's immersive workspace
- Supports Writing Tools framework (rewrite, summarize, proofread task text)

Apple featured it on the App Store page for Vision Pro specifically because it demonstrates what a "well-done visionOS productivity app" looks like.

**What Feedmine can steal:** If Feedmine ever targets Vision Pro, the approach is clear: a standard windowed SwiftUI app (not immersive) that treats the infinite canvas as "multiple lists visible at once."

### 2.5 Shortcuts & Automation (AppIntents)

Things has a rich automation surface:

| Integration | What It Enables |
|-------------|----------------|
| URL Scheme (`things:///`) | Create to-dos, show lists, deep-link to items, add projects from other apps |
| Siri Shortcuts (AppIntents) | "Add to my Things inbox", voice-activated task creation |
| Share Extension | Send content from any app into Things as a new to-do |
| Spotlight | Create to-do directly from macOS Spotlight ("Create To-Do" → Return → type title) |
| Handoff / Deep Links | Copy a `things:///` link to any to-do/project, paste anywhere, tap to navigate directly |

**URL Scheme depth:**
The `things:///` scheme supports:
- `add` — create to-dos with title, notes, date, tags, checklist items
- `add-project` — create entire projects with to-dos (enables "handover" from other apps like MindNode)
- `show` — navigate to any list or specific item by ID
- `search` — open search with a query
- `json` — bulk operations via JSON payload

This enables an ecosystem: Raycast extensions, Drafts actions, Apple Shortcuts automations, and third-party integrations all build on this URL scheme without requiring Cultured Code to maintain partnerships.

**What Feedmine can steal:** A `feedmine:///` URL scheme that enables: `feedmine:///add-source?url=...`, `feedmine:///show?list=today`, `feedmine:///article?id=...`. Share Extension to subscribe to feeds from Safari. Spotlight indexing of saved/bookmarked articles.

### 2.6 Spotlight & Search

- All to-dos are indexed in Core Spotlight — searchable from the iOS home screen
- On macOS, typing "Create To-Do" in Spotlight creates a task in the Inbox without launching the app
- Deep links from search results navigate directly to the relevant item

**What Feedmine can steal:** Index saved articles in Core Spotlight. A user searching "climate change" from the home screen should see their bookmarked Feedmine articles in results.

### 2.7 Keyboard Mastery (iPad/Mac)

Things 3.6 (2018) was a landmark update for iPad keyboard support:

- **70+ keyboard shortcuts** covering every action
- **Arrow-key list selection** — move up/down to select, Shift+arrow for multi-select
- **Type Travel** — start typing a list name from anywhere to instantly navigate there
- **Quick Popovers** — all date/tag/schedule popovers are keyboard-navigable with arrow keys + type-to-filter
- **Seamless touch+keyboard hybrid** — start selecting with keyboard, add more items by touch, continue with keyboard

The philosophy: "Almost anything you could do in Things on your Mac, you can now do on your iPad."

In iPadOS 26, Things added full support for **Windowed Apps mode** — multiple windows via right-click → "Open in New Window" or File menu, plus a full system menu bar.

**What Feedmine can steal:** Full keyboard navigation for feed reading. Arrow keys to move between articles. Return to open. `j`/`k` navigation (like RSS readers of old). Type-to-filter for feed sources.

### 2.8 Haptic Feedback

Things uses haptics deliberately and sparingly:
- **Task completion:** A satisfying vibration when checking off a to-do (success haptic)
- **Undo prevention:** When unchecking a completed task, a confirmation dialog appears (preventing slips)
- **Drag feedback:** Subtle haptic ticks during reorder operations
- **Magic Plus insertion:** Light haptic when the floating button "snaps" into position between items

The haptics are never gratuitous — they confirm meaningful state changes. This matches Apple's HIG principle: "Use haptics to provide feedback, not decoration."

**What Feedmine can steal:** Haptic on article save/bookmark. Subtle tick when pull-to-refresh completes. Success haptic when marking all items "caught up."

### 2.9 Sync Architecture (Things Cloud)

Things Cloud is a custom sync solution (not raw CloudKit):
- **Inspired by Git internals and operational transformations** — conflict-free sync across devices
- **12+ years in production** — battle-tested reliability
- **Recently rewritten in server-side Swift** (Vapor + SwiftNIO on AWS Kubernetes)
- **Results:** 3x compute cost reduction, dramatically faster response times, single Swift codebase for client and server
- Uses the user's iCloud identity for authentication (no separate account needed)
- Push notifications via APNSwift

**Architecture stack:**
- HTTP framework: Vapor (on SwiftNIO)
- Database: Amazon Aurora MySQL (via MySQLKit)
- Cold storage: S3 (via Soto)
- Ephemeral data: Redis (via RediStack)
- Push notifications: APNSwift
- Infrastructure: Terraform → AWS → Kubernetes → HAProxy → Swift services
- Monitoring: Swift Prometheus + CloudWatch + PagerDuty
- Resilience: Daily chaos testing (automated random disruptions to validate recovery)

**What Feedmine can steal:** The architecture validates that a Swift-everywhere approach works. Feedmine is local-only (no server), but the principle of "one language, zero cognitive switching" applies — Swift on client, Swift for any tooling.

### 2.10 Liquid Glass & Design System Adoption (iOS 26)

Things 3.22 demonstrates day-one adoption of Apple's newest design language:
- Complete visual refresh for Liquid Glass
- Updated app icon in 4 variants (Default, Dark, Tinted, Clear)
- Redesigned curves for windows, dialogs, and controls
- Wider spacing and visual transparency in the sidebar
- **Dynamic interface elements:** glassy buttons that scale and glow on touch
- **Magic Plus button:** fluid-like deformation when moved (custom physics simulation)

This is the hallmark of platform mastery: when Apple ships a new design system, Things doesn't just "support" it — it makes the new system look like it was designed FOR Things.

---

## 3. Interaction Design Patterns

### 3.1 The Magic Plus Button

Things' signature UX innovation: a floating `+` button that can be **dragged** to any position in a list to insert a new to-do exactly there. Not just "add to bottom" — add precisely where you want it.

On iOS 26, this button now shows "fluid-like deformation" when moved — a custom spring/physics animation that makes the button feel alive.

### 3.2 Swipe Vocabulary

| Gesture | Action | Feedback |
|---------|--------|----------|
| Swipe left on a to-do | Reveal selection circle | Visual only |
| Swipe right on a to-do | Show calendar icon → schedule it | Yellow calendar signifier |
| Swipe down on circles (right edge) | Multi-select | Sequential haptic ticks |
| Long press + drag | Reorder | Item lifts, shadow appears |

### 3.3 Type Travel (Keyboard Navigation)

From any view, start typing a list name ("Today", "Inbox", "Work") and the app instantly navigates there. No ⌘+something shortcut to memorize — just type where you want to go.

### 3.4 Progressive Disclosure

The new-task view starts minimal (title only). Icons along the bottom reveal: notes, date, tags, sub-checklist, deadline. Nothing is hidden, but nothing is forced. The UI adapts to the complexity the user chooses.

---

## 4. Development Philosophy

### 4.1 One-Time Purchase, Platform-Deep

Cultured Code maintains a one-time purchase model (no subscription). This is economically viable because:
- Deep platform integration drives word-of-mouth and App Store featuring
- Apple regularly features Things in "Best of" collections and OS update showcases
- Narrow feature scope = maintainable codebase
- Premium pricing ($9.99 iPhone, $19.99 iPad, $49.99 Mac) enabled by perceived quality

### 4.2 Apple-Only, Zero Cross-Platform

Things exists exclusively on Apple platforms. There is no Android version, no web app, no Windows port. This is not a limitation — it's a strategic choice that enables:
- Use of every Apple-specific API without abstraction layers
- SwiftUI across all platforms (shared UI code where appropriate)
- Native performance without JavaScript bridges or Flutter overhead
- First-in-class adoption of new Apple features (because there's no "but what about Android?" conversation)

### 4.3 Feature Restraint

Things deliberately does NOT add:
- Collaboration features
- File attachments
- Markdown/rich text
- Calendar views
- Gantt charts
- AI summarization

The restraint is the product. Every feature that exists is polished to perfection. This enables a small team (Cultured Code is ~15 people in Stuttgart) to maintain apps across 5 platforms without burning out.

### 4.4 SwiftUI Migration

Things' Apple Watch app was rebuilt entirely in SwiftUI for watchOS 10. The main iOS/Mac apps use a hybrid approach (UIKit foundation with SwiftUI for newer features and widgets). This pragmatic migration path — not "rewrite everything" but "new surfaces in SwiftUI" — is how most professional apps adopt the framework.

---

## 5. What Feedmine Can Steal

### 5.1 Day-One Platform Adoption as a Competitive Strategy

Things proves that being first to adopt new iOS features IS a competitive moat. When Apple ships interactive widgets, Things is ready. When Lock Screen widgets arrive, Things is there. When Vision Pro launches, Things has a native app.

**For Feedmine:** Treat every WWDC as a product roadmap. New APIs = new surfaces to deliver value. Live Activities for "currently reading" sessions. Dynamic Island for background podcast progress. Interactive widgets for "mark as read." StandBy for morning briefing display.

### 5.2 The Widget-First Mindset

Things' widget strategy covers EVERY surface: Home Screen, Lock Screen, StandBy, Watch Smart Stack, Vision Pro workspace. Each widget is tailored to its context — not the same widget everywhere, but the RIGHT information for each surface.

**For Feedmine:** Design widgets purpose-built for each context:
- Lock Screen: unread count or next recommended article title
- Home Screen (interactive): scroll through top 3 articles, tap to save
- StandBy: morning briefing with large text, readable from across the room
- Watch: "You have 5 highlights waiting"

### 5.3 The `things:///` URL Scheme Model

The URL scheme turns Things into a platform that OTHER apps can integrate with. Drafts, Raycast, Shortcuts, MindNode — all build on Things' URL scheme without Cultured Code doing extra work.

**For Feedmine:** Build `feedmine:///` from the start:
- `feedmine:///subscribe?url={feed_url}` — add a feed
- `feedmine:///show?list=today` — open today's recommendations
- `feedmine:///article?id={id}` — deep link to a specific article
- `feedmine:///search?q={query}` — search saved articles
- Share Extension for Safari → subscribe to discovered feeds

### 5.4 Keyboard + Touch Hybrid

Things proves that keyboard support isn't just "add ⌘K shortcuts." It requires:
1. A persistent selection model in lists
2. Arrow-key navigation that mirrors touch scrolling
3. Popovers that respond to keyboard input (type-to-filter)
4. Seamless switching between keyboard and touch mid-action

**For Feedmine:** On iPad with keyboard: `j`/`k` to navigate articles, `o` to open, `s` to save, `n` for next source. But also: tap an article mid-keyboard-session and the keyboard selection follows.

### 5.5 Haptics as Confirmation Language

Things' haptic vocabulary is minimal but consistent:
- Check = success
- Drag = positioning feedback
- Insert = placement confirmation

**For Feedmine:** Build a haptic vocabulary:
- Save article = success haptic
- Pull-to-refresh complete = subtle impact
- Scroll past "caught up" marker = soft notch
- New articles arrived (background) = none (don't disturb)

### 5.6 Swift Everywhere

Things Cloud being server-side Swift validates the "one language" approach. For Feedmine (local-only), this means: all business logic in Swift, all models in Swift, all tooling in Swift. No context-switching to Python or JavaScript.

---

## 6. Anti-Patterns Things Avoids

1. **No onboarding carousel.** The app opens and you're immediately productive.
2. **No gamification.** No streaks, no badges, no "you completed 7 tasks today!" celebrations.
3. **No social features.** The app is personal and private.
4. **No feature creep.** If it doesn't belong in a task manager, it doesn't ship.
5. **No subscription nag screens.** One purchase, done.
6. **No notification spam.** Things doesn't push "don't forget!" reminders by default.
7. **No AI branding.** Even if ML powers some features internally, it's never marketed as "AI-powered."

**For Feedmine:** Adopt these anti-patterns. No onboarding. No gamification. No social. No subscription. No notification spam. Let the quality of the feed speak for itself.

---

## 7. Timeline of Platform Adoption

| Date | iOS Feature | Things Response |
|------|-------------|-----------------|
| 2017 | iOS 11 Drag & Drop | Full multi-item drag & drop on iPad |
| 2018 Feb | URL Schemes (things:///) | Deep automation surface |
| 2018 May | iPad keyboard productivity | 70+ shortcuts, Type Travel |
| 2020 | iOS 14 Home Screen Widgets | WidgetKit widgets on day one |
| 2022 Sep | iOS 16 Lock Screen Widgets | Lock Screen widgets shipped |
| 2023 Sep | iOS 17 Interactive Widgets | Tap-to-complete from Home Screen |
| 2023 Sep | iOS 17 StandBy | Large-format StandBy widgets |
| 2023 Sep | watchOS 10 Smart Stack | Watch widget for Smart Stack |
| 2024 Feb | visionOS 1.0 | Native Vision Pro app |
| 2025 May | Server-side Swift (internal) | Things Cloud rewritten in Swift |
| 2025 Sep | iOS 26 Liquid Glass | Complete visual refresh, dynamic elements |
| 2025 Sep | iPadOS 26 Windowed Apps | Multi-window + system menu bar support |
| 2025 Sep | Control Center controls | Quick toggles for New To-Do |
| 2025 Sep | Writing Tools (Vision Pro) | Rewrite/summarize/proofread task text |

---

## 8. Key Takeaways for Feedmine

1. **Platform mastery IS the product.** Things doesn't compete on feature count. It competes on how perfectly it fits into iOS.

2. **Every Apple surface is a distribution channel.** Widgets, Spotlight, Shortcuts, Control Center, Share Extension — each is a place users encounter your app without opening it.

3. **Swift all the way.** One language from Watch to Server removes cognitive overhead and enables code sharing.

4. **Restraint enables quality.** A narrow feature scope lets a small team maintain excellence across 5 platforms.

5. **Adopt new APIs as product features.** Don't wait for APIs to "mature." Being first creates press coverage, App Store featuring, and user delight.

6. **Custom sync > CloudKit.** For a complex data model, a bespoke sync engine (inspired by Git/OT) provides better reliability than generic cloud frameworks.

7. **Haptics, animations, and gestures form a language.** Consistent use of subtle feedback makes the app feel alive without being noisy.

8. **Accessibility is not optional.** Full VoiceOver support, Dynamic Type, keyboard navigation — these aren't features, they're table stakes for a platform-native app.

---

*Document generated: 2026-07-06. Research rounds: Broad (platform integration landscape), Focused (specific API implementations and architecture), Gaps (keyboard/accessibility, animation details, URL scheme depth). Source: Cultured Code blog, swift.org, MacRumors, 9to5Mac, Apple Developer documentation.*
