# ⚙️ Open Algorithms — The Same Tools, Different Owner

> Seed concept for loop exploration
> 2026-07-06

---

## The insight

Algorithms are not the problem. They never were.

An algorithm is just a set of rules that decides what to show, in what order, with what priority. Every feed has one — even a chronological feed is an algorithm (sort by date, descending). The question was never "algorithm or no algorithm?" The question is:

**Who writes the rules? Who can see them? Who can change them?**

At every major platform, the answer is: the product team. A small group of engineers decides how content is ranked, what signals matter, what gets amplified, what gets buried. The user sees only the output. Feedmine uses algorithms too — ranking, interleaving, filtering, prioritization. The difference: they're open source, inspectable, and their behavior is visible in the feed.

**The question: what changes when the user can see exactly how content is ranked — and verify it?**

---

## The mechanism

### What Feedmine's algorithm actually does

There is **no ML, no engagement scoring, no personalization engine**. The ranking is a **source-diversity interleave:**

1. **Group items by source URL** — every RSS source becomes its own pool
2. **Split into recency tiers** — recent / stale / recently-surfaced — then spread by content type and category within each source
3. **Assign weighted round-robin slots** — proportional to item count per source, capped at 5 slots maximum per source
4. **Spread slots** so no two adjacent cards are from the same source
5. **Round-robin through slots** — result: every card alternates source, guaranteed variety

**Source equality is enforced:** the reservoir cap (500 items) guarantees every source gets at least 1 slot. No source can dominate by publishing volume alone.

**Anti-recency bias:** date sections after "Today" (Yesterday, This Week, Earlier) are `.shuffled()` each time they're computed. "Earlier" content can appear above "Yesterday" — deliberately fighting the assumption that newest = most important.

**Cooldown mechanics:** items surfaced on-screen are deprioritized for 30 minutes. Every 5 "load more" triggers, the reservoir re-interleaves to rotate buried content upward.

### What "open" means concretely

| Dimension | Closed (typical platform) | Open (Feedmine) |
|-----------|--------------------------|-----------------|
| **Visibility** | You can't see the algorithm | The source code is on GitHub. Read the interleave logic. |
| **Goals** | Optimized for engagement/ad revenue | Optimized for source diversity — no source dominates |
| **Behavior** | "We use machine learning" (black box) | Deterministic interleave with explicit caps and cooldowns |
| **Modification** | Impossible | Fork it. Change the caps. Adjust the cooldowns. |
| **Simplicity** | Thousands of signals, opaque weighting | Source grouping, round-robin, recency tiers. That's it. |

### What the user controls today

- **Category filter** — show only Tech, News, Science, etc. (from OPML categories)
- **Content type filter** — All / Articles / Videos / Podcasts
- **Mood filter** — Serious / Fun / Technical / Inspiring (keyword-matching on titles)
- **Source enable/disable** — toggle individual sources on or off
- **Search** — title + excerpt, with relevance scoring (exact match 100, prefix 80, contains 60, excerpt 30)
- **Shake to refresh** — marks all visible as read, re-interleaves, fetches fresh content

Filters persist across sessions. The filter state is yours, restored on every launch.

### What the algorithm does NOT do (today)

- It does not learn your preferences over time (yet — this is an intentional design space, not a limitation)
- It does not score engagement or track clicks for ranking in the main feed
- It does not weight sources by how much you read them
- It does not predict what you'll like
- It does not personalize the feed order per user behavior

The "What's New" carousel does prioritize sources the user has previously clicked — this is the single place where past behavior influences what's surfaced. The main feed interleave is behavior-agnostic today. But the *intention* is to move toward personalization that emphasizes without excluding — more of what matters to you, without creating a bubble. See [personalization-without-bubbles.md](personalization-without-bubbles.md) for the tension this creates.

---

## Why this matters

### 1. The tools exist. The access doesn't.

Content interleaving, source diversity guarantees, recency-bias correction — these are well-understood techniques used inside every major platform. The only thing that's missing is your ability to see them. Feedmine doesn't invent new technology. It *redistributes* existing technology.

### 2. Simplicity replaces trust

Feedmine's algorithm is simple enough to explain in one paragraph: "Group by source, cap at 5 slots each, round-robin so nothing repeats, shuffle non-today sections, cooldown what you've seen." You don't need to trust it because you can understand it completely. Trust becomes irrelevant when the logic fits in your head.

### 3. Open source as accountability

The code is open. If the algorithm does something unexpected, you can read why. If you don't like the 5-slot cap, change it. If you want chronological instead of interleaved, the logic is there to modify. This isn't transparency theater — it's functional accountability through readable code.

### 4. Diversity as starting point, not final answer

The interleave's optimization target today is: **no source dominates.** A niche blog with 3 posts/week gets its slot just like a news wire publishing 50/day. The algorithm is egalitarian by construction.

But this is v1's answer — a structural guarantee of variety while the harder question remains open: how do you *also* surface what matters to this specific person without collapsing into a bubble? Source diversity is the floor. Intelligent personalization that doesn't exclude is the ceiling. The space between is where Feedmine is headed.

### 5. Shuffle as philosophy

Shuffling date sections, shuffling the "What's New" carousel, re-interleaving the reservoir — the codebase uses randomness deliberately. Not chaos, but anti-pattern: fighting the tendency for feeds to calcify into predictable sequences that stop surprising you.

---

## Questions for the loop

- What can we learn from the history of open source software about what happens when tools are democratized? (Linux, Wikipedia, Arduino)
- How does the concept of *appropriate technology* (Schumacher, "Small is Beautiful") apply to algorithmic tools? Technology at the right scale, in the right hands?
- What is the relationship between *legibility* and power? (James C. Scott — when only the state can read the map, only the state can govern the territory. When the user can read the algorithm, the user governs their feed.)
- What's the value of an algorithm simple enough to hold in your head entirely — versus one that's open-source but incomprehensibly complex?
- What can we learn from the maker movement and the right-to-repair movement about user sovereignty over tools?
- Is "no personalization" itself a design choice worth examining? Source diversity guarantees variety — but variety without relevance is noise. Where's the balance?
- How does the concept of *subsidiarity* (decisions made at the lowest possible level) apply to feed algorithms?
- What's the difference between an algorithm that *excludes* and one that *emphasizes*? Can you have weighted ranking without creating walls?

---

*The algorithm is not your enemy. It's your tool — simple enough to understand, open enough to verify, yours to change.*
