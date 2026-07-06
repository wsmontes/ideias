# Loop 09: Apollo — How Christian Selig Set the Standard for iOS Feed Clients

> A deep research document on how Apollo for Reddit (2017-2023) became the gold standard for native iOS content browsing — through customizable gestures, haptic vocabulary, Jump Bar navigation, and obsessive attention to iOS HIG patterns. What Feedmine can learn from the best feed-browsing iOS client ever built, even though it was shut down by API pricing.

---

## 1. Executive Summary

Apollo by Christian Selig (a former Apple intern) was the definitive third-party Reddit client for iOS from 2017 until its shutdown on June 30, 2023 due to Reddit's API pricing changes. In its 6 years, it became the reference implementation for "how to build a native iOS content browsing client" — and remains the benchmark that every social/feed app is measured against.

**What makes it the reference standard:**

- **Built by a former Apple intern.** Selig's Apple experience meant Apollo felt like it could have been built by Apple themselves. Every iOS design pattern was implemented correctly and beautifully.
- **Customizable swipe gestures.** Users configured what left/right swipes did on every post and comment (upvote, downvote, save, reply, collapse). This gesture vocabulary became second-nature for millions.
- **Jump Bar for lightning-fast navigation.** Tap the navigation title → type a subreddit name → instantly navigate there. No menus, no hierarchy drilling. Direct teleportation.
- **Haptic feedback on every interaction.** Upvote, downvote, save, scroll-to-top — each had a distinct haptic signature. The phone communicated state through touch.
- **Full Markdown editor.** Rich text composition with live preview, formatting shortcuts, link insertion — the best mobile Markdown writing experience at the time.
- **Pixel Pal.** A virtual pet that lived on the Dynamic Island/Lock Screen, feeding on your Reddit activity. Pure delight, showing creative use of Live Activities beyond utility.

**The lesson for Feedmine:** Apollo is the CLOSEST analog to what Feedmine should feel like — a native iOS client for browsing a content feed. Apollo proved that the same content (Reddit posts) feels completely different when presented through a native iOS interface with gestures, haptics, and system integration vs. a web-wrapped experience. Feedmine should study Apollo like a bible.

---

## 2. Platform Integration Inventory

### 2.1 Customizable Swipe Gestures

Apollo's signature interaction: configurable swipe actions on posts and comments.

| Gesture | Configurable Actions |
|---------|---------------------|
| Short swipe right | Upvote (default) |
| Long swipe right | Downvote (default) |
| Short swipe left | Save (default) |
| Long swipe left | Reply / Collapse (default) |

**What made it special:**
- Users chose what each gesture did (from 10+ available actions)
- Gestures were consistent across posts AND comments
- Color feedback during swipe: orange = upvote, blue = downvote, green = save
- Haptic confirmation at trigger threshold
- Undo available if you overshot

**Feedmine parallel:** This is EXACTLY what Feedmine needs for article actions. Swipe right = save/bookmark. Swipe left = mark as read/dismiss. Long swipe = share. Configurable by the user.

### 2.2 Jump Bar (Navigation)

Tap the title in the navigation bar → search field appears → type any subreddit name → instant navigation.

- No drilling through menus
- Favorites shown immediately
- Recent history available
- Fuzzy matching (type "ios" → finds r/ios, r/iOSProgramming, etc.)
- Keyboard appears automatically, ready for input

**Feedmine parallel:** Tap the header → type a source/topic name → instantly filter or navigate to that feed. Direct access to any content without hierarchy navigation.

### 2.3 Haptic Feedback Vocabulary

Apollo had one of the richest haptic vocabularies of any iOS app:

| Action | Haptic Type |
|--------|-------------|
| Upvote (swipe trigger) | Medium impact |
| Downvote (swipe trigger) | Light impact |
| Save post | Success notification |
| Pull-to-refresh complete | Soft impact |
| Scroll to top (tap status bar) | Light tick |
| Long-press context menu appear | Heavy impact |
| Comment collapse | Soft tick |
| Post loaded | Subtle confirmation |

Every haptic was intentional and consistent. Users developed PROPRIOCEPTIVE knowledge of app state — their hand knew what happened before their eyes confirmed it.

### 2.4 Context Menus (3D Touch / Long Press)

- Long-press any post → rich preview with actions
- Long-press any link → preview destination
- Long-press any image → save/share/copy options
- Context menus used system `UIContextMenuConfiguration` (native iOS)
- Peek/Pop on devices with 3D Touch

### 2.5 Media Viewer

"Supercharged Media Viewer" was a core selling point:
- Full-screen image viewing with pinch-to-zoom
- Swipe between images in galleries
- Video playback with audio
- GIF playback
- Swipe down to dismiss (fluid, physics-based animation)
- Background blurred/darkened for focus

### 2.6 Dynamic Type & Accessibility

- Full Dynamic Type support (all text scaled with system preference)
- VoiceOver labels on every interactive element
- High contrast mode support
- Reduced Motion respected (no parallax effects)
- Per-subreddit appearance customization (different themes per community)

### 2.7 Widgets

- Home Screen widgets showing:
  - Trending posts from favorite subreddits
  - Specific subreddit feed
  - Karma count
  - Wallpaper widget (random image from a photo subreddit)
- Lock Screen widgets
- **Pixel Pal:** A virtual pet on the Dynamic Island/Lock Screen that "lived" on your iPhone

### 2.8 Siri Shortcuts

- "Open r/[subreddit]" → launches directly into a subreddit
- "Show my saved posts" → opens saved section
- Custom shortcuts for common navigation patterns

### 2.9 iPad Support

- Full Split View and Slide Over support
- Multi-column layout on larger screens
- Keyboard shortcuts for navigation
- Drag and drop (images, links out of Apollo)
- Pointer/trackpad support with hover effects

### 2.10 Apple Watch

- View notifications (replies, mentions)
- Quick actions on notifications (upvote, reply)
- Complication for karma count or unread notifications

### 2.11 Notifications & Filtering

- Push notifications for replies, mentions, messages
- **Notification filtering:** Choose which types of notifications to receive
- Rich notifications with inline content preview
- Actionable notifications (reply from notification without opening app)

### 2.12 Markdown Editor

- Full Markdown composition with formatting bar
- Live preview toggle
- Bold, italic, strikethrough, links, quotes, code, lists
- Image upload integration
- Formatting shortcuts (select text → tap bold = wraps in **)

---

## 3. Interaction Design Patterns

### 3.1 The "Feels Like Apple Built It" Standard

MacStories review: "Apollo was created by a former Apple employee, which explains why the app feels like it could have been an Apple invention."

What this means technically:
- Tab-based navigation (standard UITabBarController)
- Swipe-back gesture (standard UINavigationController)
- Pull-to-refresh (standard UIRefreshControl)
- Context menus (standard UIContextMenuConfiguration)
- SF Symbols throughout (consistent iconography)
- System colors and materials (blur effects, vibrancy)
- Accent colors that follow system appearance (dark/light mode)

Apollo didn't invent new paradigms — it implemented EXISTING iOS patterns so well that it felt native.

### 3.2 Per-Subreddit Theming

Each subreddit could have its own accent color and appearance. r/leagueoflegends looked different from r/music. This created visual wayfinding — you KNEW where you were by color alone.

**Feedmine parallel:** Per-source or per-topic theming. Tech articles could have a blue accent, news could have red, lifestyle could have green. Visual wayfinding in the feed.

### 3.3 Compact/Regular/Large Post Sizes

Users chose their preferred content density:
- **Compact:** Title + small thumbnail (high density, many posts visible)
- **Regular:** Title + medium thumbnail + preview text
- **Large:** Title + full-width image + excerpt (one post fills the screen)

**Feedmine parallel:** Let users choose article card density. Title-only for scanning, title+excerpt for browsing, full-preview for leisurely reading.

### 3.4 Mark-as-Read Behavior

Apollo tracked which posts you'd seen (grayed out) vs. new. This simple visual treatment made re-visits productive — you instantly saw what was new since last visit.

### 3.5 Scroll Position Memory

Apollo remembered your scroll position in every subreddit. Leave r/ios, browse r/swift, come back to r/ios — you're exactly where you left off. This eliminated the "where was I?" problem.

---

## 4. Development Philosophy

### 4.1 One Developer, Apple-Quality Product

Christian Selig maintained Apollo alone for most of its life. His Apple internship gave him:
- Deep knowledge of iOS HIG and system patterns
- Understanding of when to follow conventions vs. innovate
- Aesthetic sensibility aligned with Apple's design language
- Technical knowledge of private-ish APIs and system behaviors

### 4.2 Native iOS, No Compromise

Apollo was pure UIKit (later some SwiftUI). No React Native, no Flutter, no web views for content. Every scroll, every animation, every haptic was native iOS.

### 4.3 Premium via Quality, Not Scarcity

Apollo's business model:
- Free: Full Reddit browsing
- Pro ($4.99 one-time): Posting, custom gestures, notifications
- Ultra ($12.99/year or $49.99 lifetime): Extra icons, Pixel Pal, Ultra features

Users paid because the quality was worth it — not because free was deliberately crippled.

### 4.4 Community-Driven Development

Selig was active on r/apolloapp, taking feature requests, responding to bugs, and shipping user-requested features rapidly. The community felt ownership of the app.

### 4.5 Death by API Pricing

Apollo was killed not by competition or quality issues, but by Reddit's decision to charge ~$20M/year for API access. This demonstrates the risk of building on someone else's platform without owning the data layer — a lesson directly relevant to Feedmine (which uses open RSS/Atom standards, not proprietary APIs).

---

## 5. What Feedmine Can Steal

### 5.1 Customizable Swipe Gestures (THE Critical Feature)

Apollo proved that configurable swipe gestures on feed items is the highest-value interaction pattern for a content browsing app.

**For Feedmine:** This is non-negotiable:
- Short swipe right → Save/bookmark (default, configurable)
- Long swipe right → Share (default, configurable)
- Short swipe left → Mark as read (default, configurable)
- Long swipe left → Hide/not interested (default, configurable)
- Color feedback: green = save, gray = read, red = hide
- Haptic confirmation at trigger threshold

### 5.2 Jump Bar for Direct Navigation

**For Feedmine:** Tap the navigation title → type source or topic name → instantly filter/navigate. No menu hierarchies. Direct access to any feed.

### 5.3 Haptic Vocabulary

**For Feedmine:** Build a consistent haptic language:
- Save = success notification haptic
- Mark read = soft impact
- Refresh complete = light tick
- New articles arrived = subtle notification
- "You're caught up" marker = medium impact

### 5.4 Content Density Options

**For Feedmine:** Three display modes:
- **Compact:** Title + source + time (maximum scanning speed)
- **Regular:** Title + excerpt + thumbnail (balanced)
- **Large:** Title + full image + long excerpt (magazine feel)

User-configurable, with per-source overrides possible.

### 5.5 Mark-as-Read Visual State

**For Feedmine:** Clearly distinguish seen/unseen articles:
- Unread: Full opacity, bold title
- Read: Reduced opacity, normal weight
- This makes returning to the feed productive — you instantly see what's new

### 5.6 Scroll Position Memory

**For Feedmine:** Remember scroll position per-view. Leave "Tech" feed, browse "News" feed, return to "Tech" — you're where you left off.

### 5.7 Per-Source/Topic Visual Theming

**For Feedmine:** Subtle color coding per topic or source:
- Technology articles: blue accent
- News: red accent  
- Science: green accent
- Instant wayfinding without reading labels

### 5.8 Native iOS Patterns Executed Perfectly

**For Feedmine:** Don't reinvent. USE the standard iOS patterns:
- Tab bar navigation
- Swipe-back gesture
- Pull-to-refresh
- Context menus on long-press
- SF Symbols for icons
- System blur/vibrancy materials
- Standard navigation controllers

Then ADD value on top (gesture customization, haptics, intelligent ranking).

### 5.9 Open Standards Protect Against API Death

Apollo died because Reddit owned the API and changed the terms. Feedmine uses RSS/Atom — open standards no single company controls. This is a structural advantage that should be explicitly communicated to users.

---

## 6. Anti-Patterns Apollo Avoided

1. **No ads in the browsing experience.** Clean feed, always.
2. **No algorithmic manipulation hidden from users.** Posts sorted by user-chosen method (hot, new, top, rising).
3. **No dark patterns.** Free tier was genuinely complete for browsing.
4. **No web views for content.** Everything rendered natively.
5. **No social pressure.** No "your friend upvoted this" notifications.
6. **No notification spam.** Only notifications the user explicitly requested.
7. **No tracking/analytics beyond basic crash reporting.**

---

## 7. Key Takeaways for Feedmine

1. **Apollo IS the blueprint for Feedmine's interaction model.** Customizable swipe gestures + Jump Bar + haptic feedback + content density options + mark-as-read state = the core interaction vocabulary for a native iOS feed client.

2. **"Feels like Apple built it" is the highest compliment.** Follow iOS HIG perfectly, then add value on top. Don't fight the system — enhance it.

3. **Haptics create proprioceptive memory.** When your thumb KNOWS what happened by feel alone, you've achieved interaction mastery.

4. **One developer CAN build a best-in-class feed client** if they deeply understand the platform.

5. **Open standards > proprietary APIs.** Apollo died because it depended on Reddit's API. Feedmine lives because RSS is an open standard no company can revoke.

6. **Content density is a user preference, not a design decision.** Some users want to scan 20 titles. Others want to see one beautiful article card. Support both.

7. **Scroll position memory eliminates re-finding friction.** Never make users re-scroll to where they were.

8. **Community involvement creates evangelists.** When users feel heard, they become missionaries for your app.

---

*Document generated: 2026-07-06. Research rounds: Broad (feature overview, design philosophy, Apple intern background, shutdown), Focused (swipe gestures, Jump Bar, media viewer, iPad support, widgets), Gaps (Pixel Pal, haptic vocabulary, per-subreddit theming, mark-as-read UX, notification filtering). Sources: Wikipedia, MacStories review, Apple App Store story, MacRumors, AlternativeTo, iospack.com, Macworld.*
