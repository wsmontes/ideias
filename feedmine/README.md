# Feedmine — The Tools They Keep From You

**Date:** 2026-07-06
**Status:** 🌿 growing
**Repository:** [github.com/wsmontes/feedmine](https://github.com/wsmontes/feedmine)

## What is it?

An open-source iOS feed reader that gives the user the same tools product teams at major platforms keep for themselves: content algorithms, source-diversity interleaving, circadian-aware design, contextual awareness — all of it open, configurable, and running entirely on your device.

Nothing in Feedmine is technically new. Algorithms exist. Personalization exists. Smart feeds exist. Behavioral modeling exists. What's new is who has access: **you do.**

## What it actually does

Feedmine is an iPhone RSS/Atom/JSON feed reader (iOS 18+, Swift 6) that:

- Loads sources from **bundled OPML files** — articles, YouTube channels, and podcasts in one unified feed
- Ranks content using a **source-diversity interleave** — guaranteed variety, no source dominates
- Applies a **circadian design system** that shifts colors, typography weight, and spacing based on time of day
- Generates a **contextual greeting** (the MomentCard) that reflects time, season, session length, and reading patterns
- Provides **in-app reading** (WKWebView) and **background podcast playback** (AVPlayer)
- Runs with a **single external dependency** (FeedKit for RSS parsing) — everything else is native SwiftUI

**No accounts. No server. No analytics. No ads. No syncing.** One device, one user, one feed.

## Why?

Every feed platform uses powerful technology to shape what you see. These tools are not evil — they're incredibly useful. The problem is that in every existing product, they run behind closed doors, serving the company's goals. The user never sees them. The user can't configure them.

Feedmine takes these tools and puts them in the user's hands. Open source. Configurable. Transparent.

It's not anti-technology. It's technology **rescued back to the user's reach.**

## Core concepts

Each concept has its own document:

| Concept | Document | Core question |
|---------|----------|---------------|
| Open Algorithms | [open-algorithms.md](open-algorithms.md) | What changes when the user can see exactly how content is ranked? |
| The Bridge | [the-bridge.md](the-bridge.md) | What does a feed look like when it connects rather than captures? |
| Configurable Intelligence | [configurable-intelligence.md](configurable-intelligence.md) | What happens when every smart behavior is a setting, not a secret? |
| Structural Privacy | [structural-privacy.md](structural-privacy.md) | What does intelligence look like when there's no server? |
| Circadian Adaptation | [circadian-adaptation.md](circadian-adaptation.md) | How does an interface become the right tool at the right time? |
| Personalization Without Bubbles | [personalization-without-bubbles.md](personalization-without-bubbles.md) | How do you give someone more of what they care about without silently removing everything else? |

## Key positions

- **Technology is not the enemy.** Algorithms, personalization, behavioral modeling — these are powerful tools. The problem was never the tools. It was who held them.
- **Personalization is desirable — but not at the cost of bubbles.** The app wants to learn what matters to you. But "more of X" should never mean "none of Y." Emphasis, not exclusion.
- **Infinite scroll is fine. Triggers are the problem.** The scroll pushes you outward toward content. No likes, no reactions pulling you back in.
- **Data collection is fine. The pipeline is the problem.** Collection → local intelligence → better experience → end. No server.
- **The app doesn't tell you what to do.** It observes and adapts. It never restricts, gates, or prescribes.
- **Open source, open algorithms, configurable behavior.** The code is the documentation. The settings are the controls.

## Technical identity

- **Platform:** iOS 18+ (Swift 6, SwiftUI, strict concurrency)
- **Data format:** OPML for sources, compressed JSON for local state
- **Architecture:** Zero-server. No account. No backend. No telemetry.
- **Dependencies:** FeedKit (RSS/Atom/JSON parsing). Everything else native.
- **Intelligence:** Source-diversity interleave, circadian design engine, routine-aware MomentCard — all on-device, all configurable.

---

*Every feature that a product team uses to control you — Feedmine gives to you.*
