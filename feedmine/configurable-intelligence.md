# 🎛️ Configurable Intelligence — Every Smart Behavior Is a Setting, Not a Secret

> Seed concept for loop exploration
> 2026-07-06

---

## The insight

Every smart app makes decisions for you. Netflix decides what to recommend. Spotify decides what plays next. Twitter decides what's in your timeline. They all use intelligence — behavioral analysis, pattern recognition, prediction models.

The problem isn't that they're smart. The problem is that their intelligence is **non-negotiable**. You can't adjust the weights. You can't see the logic. You can't turn off specific behaviors. The smartness is imposed. Take it or leave it.

Feedmine has smart behaviors — a circadian design engine, contextual greetings, source-diversity interleaving, mood filtering. But every intelligent behavior is **exposed as a setting**. You can see what it's doing. You can turn any piece off. You can choose your palette, your font, your density.

The intelligence is not hidden inside a product team's decisions. It's a set of toggles you control.

**The question: what happens when every smart behavior is a setting the user can reach — not a secret buried in code?**

---

## The mechanism

### What's actually configurable in Feedmine today

| Smart behavior | Setting | Options |
|----------------|---------|---------|
| **Circadian color shifting** | Adaptive Palette | On (default) / Off |
| **Circadian typography** | Adaptive Typography | On (default) / Off |
| **Color personality** | Palette Family | Warm Earth, Cool Sky, Botanical, Lavender Hour, Monochrome |
| **Font rendering** | Font Style | System (SF), New York (serif), SF Mono (mono), Georgia |
| **Text size** | Font Size | Small / Medium / Large |
| **Content filtering** | Category filter | Per OPML categories (persisted) |
| **Content type** | Type filter | All / Articles / Videos / Podcasts |
| **Mood filtering** | Mood filter | All / Serious / Fun / Technical / Inspiring |
| **Source control** | Per-source toggle | Enable/disable individual feeds |
| **Night reading** | Night Mode | Off / On (35% black overlay) |
| **Image loading** | Preload Images | On / Off |
| **Data ownership** | Export My Data | Full JSON export via share sheet |

### What this means in practice

**Turn off adaptive palette** → colors stay fixed regardless of time of day. The app still functions identically — you just lose the dawn-amber-to-night-coral color shift.

**Turn off adaptive typography** → font weight stays constant. No lighter weight at night, no medium weight in the afternoon. You pick a font style, it stays.

**Disable sources** → they're skipped entirely during fetch. Not hidden after the fact — never downloaded. Your bandwidth, your choice.

**Switch mood to "Fun"** → only items whose titles match fun-associated keywords appear. It's keyword matching (not ML) — transparent, predictable, immediate.

**Export your data** → a single JSON file with everything: read history, bookmarks, disabled sources, preferences. Your data, in a format you own.

### The spectrum of control

Feedmine's configurability sits at a specific point — one that will evolve:

| Approach | What it means |
|----------|---------------|
| No intelligence | Chronological list, no adaptation, no context |
| **Feedmine today** | **Smart defaults with visible, toggleable behaviors. Diversity guaranteed. No personalized ranking yet.** |
| Feedmine intended | Smart defaults + personalized emphasis (more of what matters, nothing excluded) — all still configurable |
| Full automation | ML-driven, opaque, personalized, non-negotiable |

The direction is toward more intelligence — but intelligence that remains configurable and never creates invisible walls. Every step toward personalization will be a setting, not a secret.

### Nothing defaults to hidden

When the app does something smart, the mechanism is simple enough to explain:

- "The colors are warmer because it's evening and your adaptive palette is on"
- "These articles are interleaved because the algorithm caps each source at 5 slots"
- "The greeting says '3-day streak' because you've opened the app 3 days running"
- "This section shows 'Earlier' above 'Yesterday' because non-Today sections are shuffled"

No black box. No "the algorithm decided." Every behavior traces to a rule you can find in the code — or a setting you can flip.

---

## Why this matters

### 1. Control and intelligence are not opposites

The industry presents a false binary: either you get a "dumb" chronological feed with full control, or a "smart" algorithmic feed with no control. Feedmine rejects this. You get circadian design AND the ability to turn it off. You get source diversity AND the ability to filter by category. Smart and controllable coexist.

### 2. Configurability is respect

When an app hides its logic and says "trust us," it's treating the user as a subject. When it exposes settings and says "here — adjust it," it's treating the user as an operator. The settings aren't there because everyone uses them. They're there because the *option* to adjust is a statement of respect.

### 3. Defaults matter, but they're not destiny

Most users will never touch the circadian settings — and that's fine. The adaptive palette works well out of the box. But the user who wants Monochrome at all hours, or SF Mono regardless of time, or no color shifting at all — they get what they want in one tap. The default is smart. The override is immediate.

### 4. Portable configurations

Because settings are explicit and stored in UserDefaults, configurations are exportable alongside data. "Here's my setup" becomes a shareable concept — not just which sources you follow, but how the app behaves for you.

### 5. Settings as documentation

Every toggle in Settings teaches you something the app does. "Adaptive Typography: On" tells you the app shifts font weight by time. "Preload Images: On" tells you images are prefetched. The settings screen is a map of the app's intelligence — one that non-technical users can read without opening source code.

---

## What this is NOT

| Common misunderstanding | The actual position |
|------------------------|-------------------|
| "Every user must become a power user" | No. Defaults work great. Configuration is optional power, not required labor. |
| "Anti-AI, anti-algorithm" | The opposite. Pro-algorithm — but transparent algorithms with off switches. |
| "The app learns your preferences" | Not yet in ranking. The interleave is behavior-agnostic today. But the intention is to move toward personalization that emphasizes without excluding — see [personalization-without-bubbles.md](personalization-without-bubbles.md). |
| "Settings pages are bad UX" | Bad settings pages are bad UX. A clear, honest settings screen is a trust document. |

---

## Questions for the loop

- What can we learn from the synthesizer/music production world — where every parameter is exposed, but presets make it accessible? (The Moog Model D has 40 knobs. Presets make it usable. Knobs make it yours.)
- How does the concept of *end-user programming* (Nardi, Lieberman) apply? Configuring intelligence is a form of programming. How do we make it humane?
- Is there a meaningful difference between "configurable" and "personalizable"? Feedmine wants both — personalization you can see, adjust, and override. What does that look like in practice?
- How does the open source ethos of "view source" translate to "view settings"? Is exposing toggles a form of transparency even for non-developers?
- What is the relationship between *agency* and *configuration*? Does having controls (even unused ones) change how users relate to a tool?
- How do professional tools (Photoshop, DAWs, IDEs) balance power-user configuration with accessible defaults? What patterns transfer?
- At what point does configurability become burden? Is there a ceiling on how many toggles serve the user vs. overwhelm them?

---

*Smart is not the opposite of free. Smart and free is the opposite of smart and captured.*
