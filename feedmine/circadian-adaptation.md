# 🌅 Circadian Adaptation — The Right Tool at the Right Time

> Seed concept for loop exploration
> 2026-07-06

---

## The insight

Every app you use presents the same interface 24 hours a day. Same layout, same density, same tone. The only concession to time is dark mode — a binary switch that knows night from day and nothing more.

But product teams at major platforms know better. They A/B test by time of day. They know engagement patterns shift by hour. They optimize delivery timing. They understand that 7am and 11pm are different cognitive contexts.

They use this knowledge to *serve themselves* — sending notifications when you're most likely to open the app, showing content when you're most likely to engage.

Feedmine uses time-of-day awareness to *serve you*. The app adapts its visual interface to the time of day — not to manipulate behavior, but because a tool should feel right for the moment you're using it.

**The question: what does it look like when an interface adapts to time — not to capture attention, but to fit the moment?**

---

## The mechanism

### What the CircadianEngine actually does

The `CircadianEngine` is a **purely visual theming system** today. It shifts colors, typography weight, and spacing based on the system clock. Currently it does not affect content selection or ranking — though time-of-day could eventually inform content emphasis (surfacing long reads in the evening, quick hits in the morning) as personalization evolves.

Five time periods, each with distinct design parameters:

| Period | Hours | Font weight | Letter spacing | Card padding | Card gap | Corner radius |
|--------|-------|-------------|---------------|--------------|----------|---------------|
| **Dawn** | 5–8 | Light | +0.3pt | 16px | 16px | 14px |
| **Morning** | 8–12 | Regular | 0 | 14px | 12px | 14px |
| **Afternoon** | 12–17 | Medium | -0.1pt | 14px | 10px | 10px |
| **Evening** | 17–21 | Regular | +0.1pt | 18px | 14px | 14px |
| **Night** | 21–5 | Light | +0.5pt | 22px | 18px | 16px |

The pattern: **morning is compact and efficient** (tighter gaps, medium weight). **Evening and night open up** (more padding, more breathing room, lighter weight). The interface becomes more spacious as the day winds down.

### Color palettes

Five palette families, each with 5 period-specific accent colors (25 total):

- **Warm Earth** (default): Amber at dawn → Deep Coral at night
- **Cool Sky**: Ice blue → Indigo
- **Botanical**: Moss → Pine
- **Lavender Hour**: Lavender → Amethyst
- **Monochrome**: Warm gray throughout

Page backgrounds also shift subtly:
- Dawn/Morning: `#FAF8F5` (warmest, brightest)
- Afternoon: `#F8F5F0`
- Evening: `#F5F0E8`
- Night: `#F0EBE4` (warmest, deepest)

All on warm paper tones — there is no dark mode. Night Mode is a separate toggle that applies a 35% black overlay, independent of the circadian system.

### How transitions work

- The engine checks the system clock **once per hour** (debounced — if the hour hasn't changed, no update fires)
- Period transitions animate with `easeInOut(duration: 2.0)` — a slow, imperceptible crossfade
- No announcement, no toggle, no user action required

You don't notice the shift happening. You notice the app *feeling right*.

### What it does NOT do (today)

- **Does not change content ranking** — today, ranking is handled by the interleave algorithm independently of time. But time-aware content emphasis is a natural direction (long reads surface in the evening, quick news in the morning).
- **Does not learn your personal schedule** — it reads the system clock, not your behavior. 5pm is Evening for everyone. Routine detection exists only in the MomentCard greeting.
- **Does not restrict or gate content** — every article is available at every hour. The circadian engine never removes or hides content.
- **Does not suggest break times** — the MomentCard might mention session length as context, but never prescribes behavior.

### The MomentCard: temporal context as text

Separate from the visual engine but time-aware: the MomentCard generates a contextual greeting that updates every 30 seconds. It factors in:

- Time of day ("Good morning", "Late night")
- Day of week ("Monday — fresh start")
- Season and holidays ("Happy Halloween 🎃")
- Article count and source count
- Session duration ("45 min — maybe stretch?")
- Reading streak ("3-day streak 🔥")
- Routine detection: the app tracks open timestamps over 7 days and detects whether your current session is at your usual time (exact match), close (approximate), or unusual

The MomentCard **reflects** your temporal context. It does not prescribe behavior.

---

## Why this matters

### 1. The same tools, pointed at you

Every feed platform uses time-of-day intelligence. They know when to send the push notification that maximizes open rates. They know your vulnerability windows. Feedmine uses time-of-day awareness to make the interface more comfortable — lighter typography when you're likely tired, more spacing when you're likely browsing casually.

### 2. Routine is observed, not enforced

The app notices when you typically open (routine detection in MomentGreeting). It uses this to personalize the greeting tone. But it never says "you shouldn't be reading now" or restricts access. If you open at 3am, you get the night palette — warm, spacious, light-weight text — and a greeting that says "Late night. No rush." Not a scolding. A welcome.

### 3. The newspaper ritual, rebuilt

People had a relationship with their morning paper. It was *their* time. Digital feeds destroyed this by being always-available, always-the-same. Circadian adaptation rebuilds temporal texture: the 6:30am Feedmine literally looks different from the 10pm Feedmine. Same content sources, different atmosphere. Your time feels like *your* time.

### 4. Technology serving the body

Screens treat humans as disembodied attention units — available 24/7, undifferentiated. Circadian adaptation acknowledges that the user at dawn and the user at midnight are in different states. A lighter font weight at night. More padding. Warmer accent colors. Small adjustments that add up to: the tool fits the moment.

---

## Fully configurable

| Setting | What it does |
|---------|-------------|
| **Adaptive Palette: Off** | Colors stay fixed (morning palette) regardless of time |
| **Adaptive Typography: Off** | Font weight and letter spacing stay constant |
| **Palette Family** | Choose Warm Earth, Cool Sky, Botanical, Lavender Hour, or Monochrome |
| **Font Style** | System (SF), New York, SF Mono, or Georgia — applied within circadian weight rules |
| **Font Size** | Small / Medium / Large — independent of circadian adaptation |
| **Night Mode** | Manual override: 35% black overlay, separate from circadian system |

Both palette shifting and typography shifting can be independently toggled. You can have shifting colors with fixed font weight, or fixed colors with shifting weight, or neither, or both. The system is modular.

---

## What this is NOT

| Common misunderstanding | The actual position |
|------------------------|-------------------|
| "The app limits you at certain hours" | Never. It adapts visually. It never restricts content. |
| "It learns when you read and adapts content" | No. The circadian engine reads the clock only. Routine detection exists in MomentCard only for greeting text. |
| "It's a wellness/screen-time feature" | It's a design system feature. It makes the interface fit the time. It doesn't manage your screen time. |
| "Night owls are pathologized" | Night owls get a beautiful night palette. Their 2am is designed for, not designed against. |
| "It requires setup" | Zero setup. Reads the system clock. Works immediately. Configuration is optional refinement. |
| "It controls what articles you see" | The circadian engine is purely visual. Content ranking is handled separately and is time-independent. |

---

## Questions for the loop

- What can we learn from how physical spaces adapt to time — a café that's bright in the morning and dim in the evening? Same space, different atmosphere, no user action required.
- How do professional tools adapt to context? (IDEs change color themes. Cameras adjust to light. Musical instruments have different voicings for different registers.)
- What is the relationship between *Umwelt* (von Uexküll — the perceptual world of an organism) and time-adaptive interfaces? Each time of day is a different Umwelt.
- What can we learn from responsive architecture — buildings that adapt to inhabitants, light, and time? (Smart glass, kinetic facades, daylight-responsive lighting)
- The circadian engine is time-of-clock, not time-of-body. Does this matter? Is reading the system clock sufficient, or would biometric input (sleep data, screen time patterns) make adaptation genuinely circadian rather than merely temporal?
- What can we learn from chronobiology about the *shape* of a day — not just sleep/wake, but alertness, creativity, analytical thinking, receptivity? Could typography density map to cognitive mode?
- How do other cultures mark the transitions of the day? (Calls to prayer, tea times, aperitivo hour) What can a time-aware interface learn from these markers?

---

*The app doesn't have a time for you. It has your time — whatever hour that turns out to be.*
