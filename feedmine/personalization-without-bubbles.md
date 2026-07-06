# 🫧 Personalization Without Bubbles — The Hardest Problem in Feeds

> Seed concept for loop exploration
> 2026-07-06

---

## The insight

There are two easy answers to the personalization question. Both are wrong.

**Answer 1: Full personalization.** "You clicked architecture 10 times, so here's only architecture." This works short-term — engagement goes up, satisfaction feels high. But you never discover the food writing that changes how you cook, or the biology paper that reframes how you think about structure. The feed becomes a mirror that shows you only what you already know you like. The bubble forms silently.

**Answer 2: No personalization.** "Here's everything, in chronological order, with no weighting." This avoids bubbles — but it also makes the feed useless at scale. 200 articles arrive overnight. If they're all equally weighted, you're doing the product team's job for free — manually sifting for what matters.

Feedmine rejects both. It wants to personalize — to surface what matters to you, to make your limited time count, to understand your interests. But it refuses to create a bubble as a side effect. More of what you love, but never *only* what you love — unless you explicitly say so.

**The question: how do you give someone more of what they care about without silently removing everything else?**

---

## The tension

This isn't a solved problem. It's an active tension the app lives inside:

| Force | What it wants | Risk if unchecked |
|-------|--------------|-------------------|
| **Personalization** | Surface what matters to you | Creates filter bubbles. Narrows without asking. |
| **Diversity** | Expose you to breadth | Makes the feed noisy. Dilutes signal. Wastes time. |
| **User intent** | Respect explicit preferences | "Only architecture" is a valid choice — but is it always informed? |
| **Discovery** | Let you find what you didn't know you'd love | Paternalistic if forced. Annoying if unwanted. |

These forces pull in different directions. The challenge is not picking one — it's finding the dynamic equilibrium between all four.

### The food analogy

You can't tell someone "you don't like this food" if they've never tasted it. Exposure matters. Serendipity matters. The article you'd never have clicked might be the one that opens a new interest.

But you also don't need to eat meat every day to confirm you're vegetarian. Explicit choices deserve respect. If someone says "I don't want sports content," don't keep testing that boundary.

The line between "expanding horizons" and "ignoring preferences" is blurry, contextual, and personal. There is no formula.

---

## The mechanism (current and intended)

### Today (v1): structural diversity, no learning

The current interleave guarantees source diversity by construction:
- Every source gets at most 5 slots
- Round-robin ensures no source dominates
- Non-Today sections are shuffled (fighting recency as the only axis)

This is the simplest answer: *don't personalize at all, guarantee variety instead.* It avoids bubbles entirely — but it also means the feed can't learn what matters to you.

### Intended direction: weighted emphasis, not exclusion

The goal isn't to stay at "no personalization." It's to move toward:

- **More of what you engage with** — not only, but more. If you read architecture posts consistently, architecture might appear earlier or more often — but science, culture, tech don't disappear.
- **Explicit controls respected absolutely** — if you filter to "only architecture," that's your choice. No second-guessing. You said it, you get it.
- **Implicit signals treated carefully** — clicking ≠ loving. Reading ≠ endorsing. Not clicking ≠ rejection. Behavioral data informs *emphasis*, not exclusion.
- **Peripheral content always present** — like peripheral vision. You're focused on architecture, but design and urbanism and material science are still visible at the edges. Available. Discoverable. Not hidden.
- **The algorithm is transparent about what it's doing** — "These are surfaced because you read architecture frequently" is visible. No silent shaping.

### The hard cases

| Scenario | Easy (bubble) answer | Feedmine's harder answer |
|----------|---------------------|-------------------------|
| User reads only tech for 2 weeks | Remove non-tech gradually | Increase tech emphasis. Keep others at lower frequency. Never zero. |
| User explicitly filters "only tech" | Lock to tech | Respect it completely. It's explicit. |
| User never clicks culture articles | Assume disinterest, suppress | Reduce prominence. Occasionally surface one. Don't harass. |
| User clicks one random cooking article | RECOMMEND ALL COOKING FOREVER | Note it. Maybe surface one more next week. Don't overreact. |
| User is new, no data yet | Show "popular" (algorithmic default) | Show everything equally — source diversity as starting point |

---

## Why this matters

### 1. Self-knowledge is incomplete

You don't fully know what you want. Nobody does. Interests evolve. Moods shift. The article you'd skip today might be the one you need next month. A feed that only shows "what you already like" assumes your preferences are fixed. They're not.

### 2. Clicking is not the same as caring

Behavioral data is a proxy, not the truth. You click clickbait without caring. You skip masterpieces because the title was boring. You read trash when tired and brilliance when alert. Any system that equates clicks with preference will misread you constantly.

### 3. Bubbles form slowly and invisibly

The danger of personalization isn't sudden — it's gradual. Each small optimization (remove one article you wouldn't click, promote one you might) compounds over weeks into a feed that only reflects your past self. You don't notice the walls forming because each brick is individually reasonable.

### 4. Diversity has real value beyond ideology

This isn't about "exposing people to different viewpoints" in a political sense. It's about *cognitive richness*. The architecture reader who also sees biology, cooking, and music is a better architect — because creativity comes from cross-pollination. The bubble doesn't just narrow your feed. It narrows your thinking.

### 5. The user's explicit choice is sovereign

If someone explicitly says "only architecture" — respect it. The app's role isn't to override conscious decisions. The tension only applies to *implicit* personalization: when the app is inferring preferences from behavior rather than responding to stated choices.

---

## Principles for the solution (not yet implemented)

1. **Emphasize, don't exclude.** More of X never means zero of Y. Behavior shifts weights, not gates.
2. **Explicit > implicit.** A toggle beats an inference. A filter beats a prediction.
3. **Transparency always.** If the feed is shaped by your behavior, you can see it and undo it.
4. **Slow learning, fast forgetting.** Preferences build gradually. A week of not clicking architecture doesn't erase months of reading it.
5. **The periphery is sacred.** Even in a personalized feed, 20-30% should be outside your core interests. Always.
6. **No overreaction to signal.** One click ≠ a preference. Ten clicks over two weeks ≈ a signal. The system is patient.
7. **Resettable.** One tap clears all learned preferences. Start fresh whenever you want.

---

## Questions for the loop

- What can we learn from how librarians do **readers' advisory** — recommending books based on what someone liked, but always including "you might not expect this, but..."?
- How does the concept of **serendipity** work in physical spaces? (Bookstore browsing, museum wandering, city flânerie) Can digital feeds create genuine serendipity without being random?
- What does research on **filter bubbles** (Pariser) actually show? Is the effect as strong as claimed? What are the nuances?
- How do recommendation systems in music (Spotify Discover Weekly) attempt to balance familiarity and novelty? What works? What doesn't?
- What can we learn from **nutrition** as a metaphor? A balanced diet includes things you love and things you need. Is there an information equivalent?
- How does the concept of **adjacent possible** (Stuart Kauffman, Steven Johnson) apply to content discovery? Your next interest is probably *adjacent* to your current ones, not random.
- What does **epistemology** say about the relationship between exposure and knowledge? Can you know what you don't know?
- How do **newspaper editors** balance reader interests with editorial duty? The front page isn't personalized — it's curated for everyone. Is there a digital equivalent?

---

*The easy answer is a bubble. The easy answer is no personalization. Feedmine lives in the hard space between — where the feed knows what you love, shows you more of it, and never stops showing you what you haven't tried.*
