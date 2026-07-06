# Loop 01: Facebook Intelligence — What Meta Knows About Ranking Feeds

> A complete, actionable knowledge base for building a local-only feed reader that replicates the intelligence Facebook uses — without any server, without any user data leaving the device.

---

## 1. Executive Summary

Facebook's News Feed is the most sophisticated content ranking system ever built for general consumption. Here's what it actually does, in plain language:

**The core loop:** For every user session, Facebook takes ~1,000-2,000 candidate posts from friends, pages, and groups, scores each one with a prediction model that estimates the probability of 6+ engagement types (like, comment, share, click, dwell, hide), combines those predictions into a single "value" score weighted by what that specific user finds meaningful, then sorts and serves the top ~50-300 items depending on session length.

**The model:** A multi-stage pipeline. First, a lightweight retrieval model (candidate generation) narrows billions of potential posts to thousands. Then a heavy neural network (the "main ranker" — a DLRM-family model with 1600+ features compressed into embeddings) scores each candidate. Finally, a re-ranking layer applies diversity rules, integrity filters, and business constraints.

**The user model:** Each user is represented by ~600 sparse features (who they are, what they follow, demographic buckets) and ~1000 dense features (behavioral history compressed into embeddings). These get updated in near-real-time — Meta's SUM (Scalable User Model) paper shows that stale embeddings perform 4x worse than fresh ones.

**The shift from engagement to satisfaction:** In January 2018, Zuckerberg announced the "Meaningful Social Interactions" (MSI) pivot — deprioritizing publisher content and boosting posts that generate conversation between friends. This was an explicit acknowledgment that engagement ≠ satisfaction. Internal documents later showed MSI inadvertently boosted polarizing content because heated debates generate lots of comments. The lesson: measuring "meaningful interaction" by comment volume is a trap.

**What makes it work:** Not the model complexity — it's the signal richness. Facebook has access to signals no other system can match: who your real friends are (social graph), what you do after you leave (conversion tracking), what you almost did (viewport impressions you didn't click), and millions of users to learn from simultaneously. Feedmine can't replicate most of these. But it can replicate the *architecture* and *reasoning patterns* at a fraction of the scale.

---

## 2. The Signal Taxonomy

Every ranking system is only as good as its signals. Facebook uses hundreds; Feedmine needs ~20-50. Here's the complete taxonomy, grouped by category.

### 2.1 Explicit Engagement Signals (User Did Something)

| Signal | Strength | What It Means | Feedmine Equivalent |
|--------|----------|---------------|---------------------|
| Like/React | Medium | Quick approval | Star/favorite |
| Comment | High | Active investment | N/A for RSS (no comment system) |
| Share | Very High | "Others should see this" | Share externally |
| Save/Bookmark | Very High | "I want this later" | Save/bookmark |
| Click-through | Medium | Interest (but clickbait-prone) | Open article |
| Hide post | Strong negative | "Don't show me this" | Mark as not interested |
| Unfollow | Strong negative | Source quality decline | Unsubscribe |
| Report | Very strong negative | Content quality issue | N/A |
| New follow triggered | Very High | Content so good it triggered discovery | New subscription from article link |

### 2.2 Implicit Behavioral Signals (User Revealed Preference Through Behavior)

| Signal | Strength | What It Means | Feedmine Equivalent |
|--------|----------|---------------|---------------------|
| Dwell time (normalized) | High | Actually read/consumed | Reading time |
| Scroll-past speed | Medium-negative | Didn't find interesting | Scroll velocity |
| Viewport impression without action | Weak negative | Saw but ignored | Seen but not opened |
| Return-to-read | High | Valued enough to revisit | Reopened article |
| Session depth after engagement | Medium | Post put user in good state | Items read after this one |
| Time-to-first-action | Medium | Immediately compelling | Time from seeing to opening |
| Scroll-back-up | Medium | Reconsidered | Scrolled back to item |

### 2.3 Content Signals (Properties of the Post Itself)

| Signal | Category | Notes |
|--------|----------|-------|
| Content type | Structural | Text, image, video, link, poll |
| Content length | Structural | Affects expected dwell time |
| Topic/category | Semantic | Classified by NLP models |
| Entity mentions | Semantic | People, places, orgs mentioned |
| Sentiment | Semantic | Positive/negative/neutral tone |
| Freshness (age) | Temporal | Exponential decay |
| Source authority | Quality | Historical engagement rate of source |
| Engagement velocity | Social proof | How fast others are engaging |
| Text quality signals | Quality | Readability, grammar, clickbait detection |

### 2.4 User Context Signals (State of the User Right Now)

| Signal | What It Captures |
|--------|-----------------|
| Time of day | Morning = news, evening = entertainment |
| Day of week | Weekday vs weekend browsing patterns |
| Time since last session | Catching up vs checking in |
| Device type | Phone = shorter sessions, tablet = longer |
| Session position | First items vs deep scroll |
| Items already seen this session | Diversity and fatigue tracking |
| Current scroll velocity | Quick scan vs deliberate reading |

### 2.5 Relational Signals (Between User and Source)

| Signal | What It Captures |
|--------|-----------------|
| Historical engagement rate with source | Per-source affinity |
| Recency of last engagement with source | Relationship freshness |
| Frequency of source posts | High-volume sources get throttled |
| Source topic diversity | Does this source cover one or many topics? |
| Time since subscription | New subscriptions get initial boost |

### 2.6 The Signal Quality Hierarchy

Research (WeChat "Click Reweighting" paper, 2023) establishes a clear hierarchy:

1. **Strongest positive:** Save/bookmark, share externally, triggered new subscription, return-to-read
2. **Strong positive:** Long dwell time (normalized per content length), comment/annotation
3. **Medium positive:** Click/open, moderate dwell time
4. **Weak/noisy positive:** Raw click (clickbait-prone), short dwell time
5. **Negative:** Scroll-past quickly, hide, mark as seen without opening
6. **Strong negative:** Unsubscribe, explicit "not interested"

Key insight from WeChat research: Clicks are not binary signals. A click with 60s dwell time is fundamentally different from a click with 3s dwell time. The WeChat team defines a "valid read" as a click where dwell time exceeds exp(μ-σ) of the log-normal dwell time distribution (approximately 15 seconds in their system). Only valid reads should count as positive training signals.

---

## 3. The Ranking Pipeline

### 3.1 Facebook's Multi-Stage Architecture

```
[All possible posts] → Candidate Generation → ~1500 candidates
                                                    ↓
                                            Light Ranker (fast scoring)
                                                    ↓
                                              ~500 candidates
                                                    ↓
                                            Heavy Ranker (DLRM neural network)
                                                    ↓
                                              Scored candidates
                                                    ↓
                                            Re-ranking layer (diversity, integrity, business rules)
                                                    ↓
                                              Final feed order
```

### 3.2 The Heavy Ranker: What It Predicts

The Heavy Ranker doesn't produce a single "relevance score." It predicts multiple engagement probabilities simultaneously:

- P(click) — will the user click?
- P(like) — will the user react?
- P(comment) — will the user comment?
- P(share) — will the user share?
- P(dwell > threshold) — will the user spend meaningful time?
- P(hide) — will the user hide this? (negative)
- P(report) — will the user report? (strong negative)

These predictions are combined via a weighted sum:

```
score = w1*P(click) + w2*P(like) + w3*P(comment) + w4*P(share) 
      + w5*P(long_dwell) - w6*P(hide) - w7*P(report)
```

The weights are NOT uniform. After the 2018 MSI shift, comments and shares got significantly higher weights than likes and clicks. This is the mechanism through which "meaningful interaction" gets operationalized.

### 3.3 The DLRM Architecture (Open-Sourced by Meta)

Meta's Deep Learning Recommendation Model:

```
Input: sparse features + dense features
         ↓                    ↓
  Embedding tables         Bottom MLP
         ↓                    ↓
    Sparse embeddings    Dense embeddings
              ↓      ↓
      Pairwise dot-product interactions
                    ↓
               Top MLP
                    ↓
             Prediction (sigmoid)
```

- **Sparse features** → Categorical: user_id, source_id, topic_id, content_type, device_type
- **Dense features** → Continuous: time_since_published, user_historical_CTR, source_engagement_rate, dwell_time_avg
- **Interactions** → The dot-product layer captures pairwise feature interactions without explicit engineering

At Feedmine scale (1 user, ~50 features, ~500 items, ~1000 vocabulary items), this entire model fits in <10MB RAM and scores all items in <10ms.

### 3.4 X/Twitter's Open-Source Alternative (2025-2026)

X's open-sourced algorithm provides a real-world reference:

**Architecture:** SimClusters (community detection via follow patterns) → candidate sourcing (~1500 posts, 50% in-network / 50% out-of-network) → Heavy Ranker (Grok-1 transformer predicting 19 engagement types) → filtering.

**Signal weights (from open-source code):**
- Reply: 13.5x base (like)
- Retweet/Repost: 20x base
- Reply chain with author engagement: 75x base
- Bookmark: high weight (lasting value signal)
- Profile click: high (genuine interest)
- Dwell time: moderate (content quality)
- Link click: low (off-platform, suppressed)
- Mute/Block: strong negative

**Key insight for Feedmine:** X's system confirms that conversation-generating and save-worthy content is valued 10-75x more than simple clicks/likes. This maps directly to save/bookmark being the king signal for a local reader.

### 3.5 Feedmine's Simplified Pipeline

For a single-user local system with 50-500 candidate items:

```
[New items from subscriptions] → Feature extraction (content + source signals)
                                         ↓
                                  Single-pass scorer (lightweight model)
                                         ↓
                                  Scored candidates
                                         ↓
                                  Diversity re-ranking (MMR or slot-based)
                                         ↓
                                  Session-aware final order
```

No candidate generation stage needed — the subscription list IS the candidate set. No light ranker needed — scoring 500 items with a small model takes <50ms. The entire pipeline collapses to: score → diversify → serve.


---

## 4. Content Understanding

### 4.1 What Facebook Does

Facebook classifies every post along multiple dimensions:
- **Topic taxonomy:** Hierarchical (Sports > Football > Premier League)
- **Content type:** Text, image, video, link, poll, event, etc.
- **Entities:** Named entity recognition for people, places, organizations, events
- **Sentiment:** Positive/negative/neutral/mixed
- **Quality signals:** Clickbait detection, misinformation likelihood, engagement-bait patterns
- **Integrity classification:** Violence, hate speech, nudity, spam

They use massive transformer models (billions of parameters) running on GPU clusters. Feedmine cannot replicate this at scale — but doesn't need to.

### 4.2 What Can Run Locally

**Lightweight text classification models suitable for on-device:**

| Model | Size | Speed | Accuracy (GLUE) | Best For |
|-------|------|-------|-----------------|----------|
| DistilBERT | 66M params, ~250MB | 60% faster than BERT | 97% of BERT | General text classification |
| TinyBERT | 14.5M params, ~60MB | 7.5x faster than BERT | 96% of BERT | Mobile inference |
| MobileBERT | 25M params, ~100MB | 4x faster than BERT | Competitive | Optimized for mobile |
| MiniLM | 22M params, ~90MB | Best throughput | Excellent | Speed-critical applications |
| ALBERT | 12M params, ~50MB | Variable | High multi-domain | Parameter efficiency |

**Practical for Feedmine:** DistilBERT fine-tuned for topic classification runs inference at ~0.12 seconds per article on a mobile Intel CPU. For an RSS reader processing 50-200 articles per sync, that's 6-24 seconds of background processing — entirely acceptable.

### 4.3 The Topic Classification Approach

**Option A: Zero-shot classification (no training needed)**
Use a pre-trained NLI model (e.g., `facebook/bart-large-mnli` distilled to smaller) to classify articles into topics without fine-tuning. Slower but requires zero labeled data.

**Option B: Fine-tuned classifier (best accuracy)**
Fine-tune DistilBERT or MiniLM on a topic taxonomy. Requires ~100-500 labeled examples per topic. Can be bootstrapped by using feed categories as weak labels.

**Option C: Embedding similarity (most flexible)**
Compute article embeddings using a sentence-transformer model (e.g., `all-MiniLM-L6-v2`, 80MB). Compare against topic prototype embeddings. No fixed taxonomy — topics emerge from clusters.

**Recommended for Feedmine: Option C** — embedding-based approach.
- No fixed taxonomy to maintain
- Topics emerge organically from the user's actual subscriptions
- New topics appear automatically when new feeds are added
- Similarity scores are continuous, not categorical — better for ranking
- Model: `all-MiniLM-L6-v2` — 80MB, runs in ~30ms per article

### 4.4 Content Feature Extraction Pipeline

For each incoming RSS article, extract:

```
1. Title embedding (sentence-transformer, 384-dim vector)
2. Content embedding (first 512 tokens → sentence-transformer)
3. Content length (word count → normalized)
4. Has images (boolean)
5. Estimated reading time (word_count / 238 wpm)
6. Source ID (which feed it came from)
7. Publication time (ISO timestamp)
8. Topic cluster (nearest cluster from user's topic model)
9. Readability score (Flesch-Kincaid or similar)
10. Title sentiment (simple lexicon-based, or model-based)
```

Total processing time per article: ~50-100ms on modern hardware. Background-processable.

### 4.5 The "Enough Intelligence" Principle

Facebook uses 100+ content features. Feedmine needs ~10. Why?

Facebook must distinguish between millions of diverse content types from billions of sources for billions of users. Feedmine has:
- 1 user (no cross-user learning needed)
- Known sources (user explicitly subscribed)
- Consistent content types (mostly articles/blog posts)
- Limited vocabulary of topics (defined by subscription choices)

The content understanding system exists to answer one question: "What is this article about, and how does it relate to what this specific user cares about?" An embedding similarity score answers this directly.

---

## 5. User Modeling

### 5.1 How Facebook Builds User Profiles

Meta's Scalable User Model (SUM) represents each user as two embeddings of dimension 96, compressed from ~1600 raw features. These embeddings encode:

- **Identity features:** Demographics, account age, device, location
- **Behavioral history:** Actions taken on posts (aggregated)
- **Social graph features:** Who they interact with, group memberships
- **Temporal patterns:** When they're active, how their interests shift

The critical finding: **Real-time user embeddings give 4x more improvement than stale ones.** User models must be updated continuously.

### 5.2 Feedmine's User Model Structure

For a single-user local system, the user model is much simpler but follows the same principles:

```python
UserModel = {
    # Topic affinities (updated every interaction)
    "topic_scores": {
        "topic_cluster_id": {
            "short_term": 0.0-1.0,   # last session, EMA α=0.3
            "medium_term": 0.0-1.0,  # last 2 weeks, EMA α=0.1
            "long_term": 0.0-1.0     # all history, EMA α=0.01
        }
    },
    
    # Source affinities (per-feed scores)
    "source_scores": {
        "feed_id": {
            "engagement_rate": 0.0-1.0,    # opens / impressions
            "satisfaction_rate": 0.0-1.0,   # valid_reads / opens
            "last_engaged": timestamp,
            "total_interactions": int
        }
    },
    
    # Content length preference
    "length_preference": {
        "short_affinity": 0.0-1.0,   # <3 min articles
        "medium_affinity": 0.0-1.0,  # 3-10 min
        "long_affinity": 0.0-1.0     # >10 min
    },
    
    # Temporal patterns
    "temporal_patterns": {
        "hour_of_day_engagement": [24 floats],  # engagement rate by hour
        "day_of_week_engagement": [7 floats]
    },
    
    # Session context (ephemeral, current session only)
    "current_session": {
        "items_seen": int,
        "items_opened": int,
        "topics_consumed_this_session": set,
        "session_type": enum,  # quick_check, browse, deep_read, catch_up
        "avg_dwell_this_session": float
    }
}
```

### 5.3 Interest Decay Functions

Research confirms that user interests shift over time and must be decayed:

**The half-life model** (Ardagelou & Arampatzis, 2017):
- Tested on movie ratings spanning 7 months
- Optimal half-life: **~150 days** for broad interests
- Formula: `weight(t) = 0.5^(days_since_interaction / half_life)`
- A rating from 150 days ago counts half as much as today's rating
- This yielded "large improvements in prediction accuracy"

**Multi-timescale exponential moving average (EMA):**

For Feedmine, use three parallel decay rates:

```
Short-term (captures current mood/session):
  score_new = α * signal + (1 - α) * score_old,  α = 0.3
  Half-life equivalent: ~2 interactions

Medium-term (captures weekly/monthly interests):
  score_new = α * signal + (1 - α) * score_old,  α = 0.1
  Half-life equivalent: ~7 interactions (~1 week of daily use)

Long-term (captures stable preferences):
  score_new = α * signal + (1 - α) * score_old,  α = 0.01
  Half-life equivalent: ~70 interactions (~2-3 months)
```

**Combining timescales for scoring:**
```
combined_topic_score = 0.4 * short_term + 0.35 * medium_term + 0.25 * long_term
```

Weight the short-term more during active sessions (user is "in a mood"), but the long-term provides stability and prevents thrashing.

### 5.4 Handling Interest Drift

Detect interest drift via these signals:
- **Spike in skip-rate** for a previously high-scoring topic → interest waning
- **Sudden engagement** with a new topic cluster → emerging interest
- **Source unsubscription** → immediate zeroing of source score
- **New subscription** → bootstrap from topic similarity to existing preferences

The piecewise decay function (from time-aware CF research): different topics decay at different rates. A sports topic during the season decays slowly; the same topic off-season decays fast. Implementation: track per-topic engagement frequency and adjust half-life proportionally.

### 5.5 Cold Start Strategy

Feedmine has a unique cold-start advantage over general recommender systems: **it knows what the user subscribed to from Day 1.**

**Bootstrapping sequence:**
1. User adds feeds → classify each feed by topic (using first 20 articles from each)
2. Initial user profile = weighted average of feed topic embeddings
3. First ranking = source diversity + recency (safe default)
4. After 10-20 interactions → begin personalization
5. After 50-100 interactions → full personalization active
6. Optional: 5-8 onboarding questions (research shows attribute-based questions outperform item-based: "Do you prefer long reads?" beats "Did you like this article?")

### 5.6 The Learning Algorithm

For Feedmine's single-user, 20-50 feature model, three approaches are viable:

**Option A: Online SGD (Stochastic Gradient Descent)**
- Update model weights after every interaction
- Simple, well-understood, fast
- Risk: can overfit to recent behavior
- Mitigation: use a small learning rate (0.001-0.01) and L2 regularization

**Option B: Contextual Bandits (Thompson Sampling)**
- Treat each item as an "arm" in a contextual bandit problem
- Naturally balances exploration vs exploitation
- Provides principled uncertainty estimates (knows when it's unsure)
- Best for the cold-start phase when the system is still learning
- More complex to implement but theoretically optimal for the problem

**Option C: Bayesian Online Learning**
- Maintain probability distributions over feature weights
- Update beliefs after each interaction via Bayes' rule
- Provides natural uncertainty quantification
- Can use uncertainty for exploration (present items the model is uncertain about)
- Computational cost: moderate (viable for <50 features)

**Recommended: Hybrid approach**
- Start with contextual bandits (Thompson Sampling) for exploration during cold start
- Transition to online SGD with periodic Bayesian calibration once sufficient data exists
- Use uncertainty estimates to drive exploration rate (more uncertain → more exploration)


---

## 6. Anti-Bubble Mechanisms

### 6.1 The Problem Is Real

Meta-analysis of filter bubble research (29/34 studies confirm): algorithmic personalization narrows content diversity over time. The feedback loop: user engages with topic A → system shows more A → user engages more with A → topic B atrophies from the feed → user loses exposure to B entirely.

For Feedmine this is particularly dangerous because:
- Single user = no collaborative filtering to inject diversity from similar users
- Small candidate pool = narrowing effects are more pronounced
- No editorial team = no human curation to counterbalance

### 6.2 What Facebook Does

1. **Content type diversity:** Forces a mix of post types (video, text, link, photo) regardless of prediction scores
2. **Source diversity:** Caps how many posts from a single source appear in sequence
3. **Topic diversity:** Ensures feed contains multiple topic clusters
4. **"Explore" features:** Dedicated sections for content outside the user's bubble
5. **Community Notes (on X):** Fact-checking layer that can suppress narrow content
6. **"Why am I seeing this?"** transparency feature

### 6.3 Practical Anti-Bubble Techniques for Feedmine

**Technique 1: MMR (Maximal Marginal Relevance) Re-ranking**

After scoring, don't just sort by score. Use MMR to balance relevance with diversity:

```
MMR(item) = λ * relevance(item) - (1-λ) * max_similarity(item, already_selected_items)

λ = 0.7 (relevance-dominant) to 0.5 (balanced)
```

For each position in the feed, select the item that maximizes MMR. This naturally pushes diverse items up even if their raw relevance score is slightly lower.

**Technique 2: Exploration Slots (ε-greedy)**

Reserve 10-20% of feed positions for exploration:
- 80-90% of items: ranked by model score (exploitation)
- 10-20% of items: selected with deliberate diversity intent

Exploration items can be:
- Random items from low-engagement sources (give them a chance)
- Items from topic clusters the user hasn't engaged with recently
- Items from newly subscribed feeds (cold-start diversity)
- Items the model is most uncertain about (active learning)

**Technique 3: Multi-timescale Interest Tracking**

The three-timescale model (short/medium/long) IS an anti-bubble mechanism:
- Short-term captures current mood → allows temporary deep dives
- Long-term maintains stable, broad interests → prevents permanent narrowing
- If short-term diverges significantly from long-term → signal temporary interest, don't update long-term aggressively

**Technique 4: Source Rotation**

Prevent source starvation:
```
For each source the user is subscribed to:
  - Minimum guaranteed representation: at least 1 item per session (if available)
  - Maximum cap: no more than 30% of feed from a single source
  - Staleness penalty: if a source hasn't appeared in 3+ sessions, boost it
```

**Technique 5: Topic Diversity Constraint**

After scoring, enforce:
```
No more than 40% of the top-20 items can be from the same topic cluster
At least 3 different topic clusters must be represented in the top-10
```

**Technique 6: Serendipity Injection**

Research shows 5-10% intentionally random or surprising items increases user satisfaction. These are items that:
- Score below the threshold but are from high-quality sources
- Cover topics adjacent to (but not identical to) the user's interests
- Are unusually popular (high engagement velocity) regardless of topic match

**Technique 7: User-Controlled Diversity Slider**

Let the user explicitly set their diversity preference:
- "Focused" (λ=0.9): give me exactly what I want
- "Balanced" (λ=0.7): mostly relevant, some variety (default)
- "Discovery" (λ=0.5): surprise me frequently

### 6.4 The WeChat Finding on Algorithms vs Bubbles

A 2025 randomized experiment on WeChat with 2.1 million users found that **algorithmic curation actually promoted more engagement with novel content than peer-sharing**, even when social cues were present. Algorithms demonstrated a "stronger positive effect than peer influence" in exposing users to non-redundant, diverse content.

This challenges the naive "algorithms = bubbles" narrative. Well-designed algorithms with diversity constraints can EXPAND rather than narrow content exposure. The key is intentional design — an unconstrained optimization will bubble, but a diversity-aware system won't.

### 6.5 When to Stop Pushing Novelty

Recent research ("When Recommender Systems Should Stop Pushing Novelty," 2025) warns that global hyperparameters for exploration are suboptimal. Exploration should be personalized:
- New users need more exploration (learning phase)
- Established users with stable preferences need less
- After a user explicitly narrows (unsubscribes from sources), respect that choice
- Exploration rate should decay over time: start at 20%, stabilize around 5-10%

Formula for adaptive exploration rate:
```
exploration_rate = max(0.05, 0.20 * (1 - confidence_in_user_model))
```

Where `confidence_in_user_model` increases with total interactions and model accuracy on recent predictions.

---

## 7. Temporal Intelligence

### 7.1 Time-of-Day Patterns

Facebook learned that content preferences shift predictably by time of day:
- **Morning (6-9am):** News, updates, professional content — users catching up
- **Midday (12-2pm):** Light content, entertainment, social — lunch breaks
- **Afternoon (2-5pm):** Mixed — varies by user
- **Evening (6-10pm):** Entertainment, long reads, personal content — relaxation
- **Late night (10pm+):** Highly personal, niche interests, longer consumption

For Feedmine: track per-user engagement rates by hour-of-day, then use as a feature weight modifier:
```
temporal_boost = user_engagement_rate[current_hour] / user_avg_engagement_rate
```

### 7.2 Session Type Detection

Session intent can be inferred from 3 signals within the first 5-10 seconds:

| Signal | Quick Check | Deep Browse | Catching Up | Habitual Scan |
|--------|-------------|-------------|-------------|---------------|
| Time since last session | <2 hours | >4 hours | >24 hours | 4-12 hours |
| Time of day | Any | Evening/weekend | Morning | Fixed time daily |
| Initial scroll speed | Fast | Slow | Medium | Medium-fast |

**Detection algorithm:**
```python
def detect_session_type(time_since_last, time_of_day, initial_scroll_speed):
    if time_since_last < 2_hours:
        return "quick_check"  # Just checking for new stuff
    elif time_since_last > 24_hours:
        return "catching_up"  # Missed time, wants highlights
    elif time_of_day in evening_weekend AND initial_scroll_speed < threshold:
        return "deep_browse"  # Has time, wants quality
    else:
        return "habitual_scan"  # Default daily check
```

**How session type affects ranking:**

| Session Type | Strategy |
|-------------|----------|
| Quick check | Show only highest-confidence items, limit to 5-10, prioritize recency |
| Deep browse | Show full ranked feed, include longer articles, boost exploration |
| Catching up | Prioritize "best of" from missed period, summarize high-volume sources |
| Habitual scan | Standard ranking, moderate length, balanced diversity |

### 7.3 Recency and Freshness

Content ages differently by type:
- **Breaking news:** Half-life of 2-4 hours. Value drops fast.
- **Analysis/opinion:** Half-life of 1-3 days. Still relevant after initial window.
- **Tutorials/evergreen:** Half-life of weeks to months. Barely decays.
- **Personal blogs:** Half-life of 1-7 days depending on author cadence.

**Freshness decay function:**
```
freshness_score = exp(-age_hours / half_life_hours)

Where half_life_hours is estimated from:
  - Source type (news = 4h, blog = 48h, tutorial = 720h)
  - Historical engagement pattern of similar content from this source
  - User's own recency sensitivity (some users only want fresh, others don't mind older)
```

### 7.4 The "Catching Up" Problem

When a user returns after absence (>24h), the naive approach (sort by score) fails because:
- Too many candidates (3 days of posts from all subscriptions)
- User wants "what was important" not "everything"
- Chronological order becomes critical (understanding narrative flow)

**Solution: "Highlights" mode**
1. Score all items from the missed period normally
2. Apply aggressive filtering: only show items scoring above 2x the user's average threshold
3. Group by source, show max 2-3 per source (prevent flooding from high-volume sources)
4. Within each group, sort chronologically (preserve narrative)
5. Add explicit "See all from [Source]" expansion option

### 7.5 Engagement Velocity as a Signal

X's algorithm heavily weights early engagement velocity — how quickly a post gains interaction in its first 30-60 minutes. For Feedmine (single user), this doesn't directly apply. But the concept translates:

**Source-level velocity:** If a source that normally publishes once per day suddenly publishes 3 posts in an hour, that's a signal (live event, breaking story). Boost those items.

**Cross-source velocity:** If multiple subscribed sources all write about the same topic simultaneously (detectable via embedding similarity), that's a trending signal. Boost the best one.

### 7.6 Fatigue and Diminishing Returns

Both WeChat research and X's author diversity penalty confirm: repetition breeds fatigue.

**Apply in Feedmine:**
- **Topic fatigue:** After showing 3+ items on the same topic in a session, apply diminishing weights to subsequent same-topic items
- **Source fatigue:** Cap any single source at ~30% of a session's items
- **Length fatigue:** If user has read 2+ long articles (>10 min), boost shorter content next
- **Session fatigue:** Overall engagement rate drops with session length. After ~20 items, most value has been delivered — signal "you're caught up"


---

## 8. What Can Be Done Locally (No Server Required)

### 8.1 Feasibility Assessment

| Component | Facebook Scale | Feedmine Scale | Feasibility |
|-----------|---------------|----------------|-------------|
| Candidate generation | Billions → thousands | 50-500 items (subscription-bounded) | Trivial |
| Feature extraction | GPU clusters | CPU, <100ms per item | Easy |
| Model inference | Distributed, <50ms/request | Single model, <10ms total | Trivial |
| User model update | Real-time streaming infra | After every interaction | Easy |
| Content classification | Transformer clusters | DistilBERT/MiniLM on-device | Moderate |
| Training data | Billions of interactions | 10-100 interactions per day | Challenging (sparse) |
| Collaborative filtering | Billions of users | 1 user (impossible) | N/A — not needed |
| A/B testing | Millions of users in control/treatment | Single user, no control | Must use other methods |

### 8.2 What's Actually Hard Locally

1. **No collaborative filtering:** Can't learn "users like you also liked X." Must rely entirely on content-based and behavioral signals.
2. **Sparse training data:** With 10-50 interactions per day, the model needs to learn efficiently from very few examples.
3. **No A/B testing:** Can't run experiments with control groups. Must use interleaving, counterfactual evaluation, or periodic user satisfaction surveys.
4. **Content understanding ceiling:** Without GPU, limited to models under ~300MB. Can't run GPT-class models for deep content understanding.
5. **Cold start for new topics:** Without other users' data, a new topic has zero signal until the user explicitly engages.

### 8.3 The On-Device Stack

```
┌─────────────────────────────────────────────────┐
│                   Feedmine Architecture           │
├─────────────────────────────────────────────────┤
│  Presentation Layer                              │
│  ├── Feed UI (ranked items)                      │
│  ├── Session type indicator                      │
│  └── Diversity slider / preferences              │
├─────────────────────────────────────────────────┤
│  Ranking Engine                                  │
│  ├── Scorer (lightweight DLRM or logistic model) │
│  ├── Diversity re-ranker (MMR)                   │
│  ├── Session adapter (adjusts for session type)  │
│  └── Exploration injector (ε-greedy slots)       │
├─────────────────────────────────────────────────┤
│  User Model                                      │
│  ├── Topic affinities (3 timescales)            │
│  ├── Source affinities                           │
│  ├── Temporal patterns                           │
│  ├── Content length preferences                  │
│  └── Session state (ephemeral)                   │
├─────────────────────────────────────────────────┤
│  Content Understanding                           │
│  ├── Embedding generator (MiniLM, 80MB)         │
│  ├── Topic clusterer (KMeans on embeddings)      │
│  ├── Readability scorer                          │
│  └── Freshness calculator                        │
├─────────────────────────────────────────────────┤
│  Signal Collection                               │
│  ├── Dwell time tracker                          │
│  ├── Scroll behavior monitor                     │
│  ├── Explicit actions (save, share, hide)        │
│  └── Session boundary detector                   │
├─────────────────────────────────────────────────┤
│  Data Layer                                      │
│  ├── SQLite (interaction history, user model)    │
│  ├── Embeddings cache (per-article vectors)      │
│  └── Model weights (<10MB)                       │
└─────────────────────────────────────────────────┘
```

### 8.4 Resource Budget

| Resource | Budget | Notes |
|----------|--------|-------|
| Storage (model) | <10MB | Logistic model or tiny DLRM |
| Storage (embeddings cache) | ~50MB | 384-dim × ~10K articles |
| Storage (user history) | ~5MB | SQLite, years of interactions |
| RAM during scoring | <50MB | Model + current batch of items |
| CPU per ranking pass | <100ms | Score 500 items + diversify |
| CPU per article ingestion | ~100ms | Embedding + feature extraction |
| Background sync | ~30 sec | Fetch + process 200 new articles |

All of this runs comfortably on a 2020-era smartphone or any modern laptop.

---

## 9. What Feedmine Can Steal

### 9.1 The Top 10 Implementable Techniques

**1. Multi-prediction scoring (from Facebook's Heavy Ranker)**

Don't predict a single "relevance" score. Predict multiple engagement types and weight them:
```
score = 0.15 * P(open) + 0.30 * P(valid_read) + 0.35 * P(save) + 0.20 * P(long_dwell) - 0.40 * P(skip)
```

Implementation: train separate lightweight classifiers for each action type, or use a multi-head model.

**2. Valid Read as the core metric (from WeChat)**

Steal WeChat's "valid read" concept wholesale:
- Compute the log-normal distribution of dwell times across all articles
- Set threshold at exp(μ - σ) — approximately 15-30 seconds for articles
- Only count clicks with dwell time above threshold as positive signals
- Use normalized dwell time (sigmoid function) to weight the positive signals

This single insight — not all clicks are equal — will dramatically improve signal quality.

**3. Dwell time normalization (from WeChat Click Reweighting)**

```python
def normalized_dwell_time(dwell_seconds, offset=15, tau=20):
    """
    Returns 0-1 score where:
    - Short dwells (<offset) → low score with steep gradient near threshold
    - Long dwells → diminishing returns (flattens out)
    """
    A = 2.319
    B = 0.744
    return A / (1 + math.exp(-(dwell_seconds - offset) / tau)) - B
```

This addresses both clickbait (penalizes short dwells) and over-emphasis on long reads (caps benefit of very long sessions).

**4. Three-timescale user model (from temporal CF research)**

Implement the EMA triplet:
- α=0.3 for session-level interests (what are you into RIGHT NOW?)
- α=0.1 for weekly interests (what's your current focus?)
- α=0.01 for stable preferences (what do you always care about?)

Combine with: `0.4 * short + 0.35 * medium + 0.25 * long`

**5. Session type detection (from session-based recommendation research)**

Classify sessions in the first 5 seconds, then adapt:
- Quick check → top 5-10 items only, aggressive filtering
- Catching up → highlights mode, chronological within groups
- Deep browse → full feed, exploration enabled, longer articles boosted
- Habitual → standard personalized ranking

**6. MMR diversity re-ranking (from information retrieval)**

After scoring, apply MMR instead of pure sort:
```python
def mmr_rerank(scored_items, lambda_param=0.7):
    selected = []
    remaining = scored_items.copy()
    while remaining:
        best = max(remaining, key=lambda item:
            lambda_param * item.score - 
            (1 - lambda_param) * max_similarity(item, selected))
        selected.append(best)
        remaining.remove(best)
    return selected
```

**7. Source throttling (from X's author diversity penalty)**

No single source dominates:
```
max_items_per_source_per_session = max(2, total_items * 0.3)
```

If a source publishes 50 items/day and the user sees 30 items/session, show max 9 from that source. Forces diversity across subscriptions.

**8. Exploration via uncertainty (from bandit literature)**

Instead of random exploration, explore where the model is uncertain:
- Track prediction confidence for each item
- Items with high uncertainty get a boost: `adjusted_score = score + β * uncertainty`
- This focuses exploration where it's most informative (active learning)
- β decays over time as the model gains confidence

**9. Freshness decay calibrated per source (from temporal research)**

Learn the "natural freshness" of each source:
```python
def source_freshness_halflife(source_id):
    # Compute from historical engagement patterns
    # How quickly do items from this source lose engagement?
    avg_engagement_age = mean(time_to_engagement for items in source)
    return avg_engagement_age * 2  # Half-life ≈ 2x average engagement age
```

A news source might have a 4-hour half-life; a weekly essay blog might have a 72-hour half-life. Auto-calibrate from engagement data.

**10. "You're caught up" signal (from Instagram)**

Detect when the user has seen all high-value content:
```
if all items below score_threshold have been shown:
    show "You're all caught up!" marker
    below this: only exploration items and lower-confidence recommendations
```

Respects the user's time. Doesn't keep them scrolling through diminishing-value content.

### 9.2 The Minimum Viable Feature Set (First 20 Features)

For the scorer model, start with these 20 features:

**Item features (10):**
1. Content freshness (hours since publication, log-scaled)
2. Source historical engagement rate (opens/impressions for this source)
3. Source satisfaction rate (valid_reads/opens for this source)
4. Topic similarity to user's short-term profile (cosine similarity)
5. Topic similarity to user's long-term profile (cosine similarity)
6. Content length (estimated reading time, normalized)
7. Has been seen but not opened (boolean — already scrolled past)
8. Source posting frequency (items per day, log-scaled — high-volume penalty)
9. Content type embedding (cluster ID for article style)
10. Time since last engagement with this source (hours, log-scaled)

**User context features (5):**
11. Hour of day (cyclical encoding: sin and cos)
12. Session items already consumed (count)
13. Session average dwell time so far
14. Time since last session (hours, log-scaled)
15. Session type (one-hot: quick_check, deep_browse, catching_up, habitual)

**Interaction history features (5):**
16. User's overall engagement rate (opens/impressions, last 30 days)
17. User's valid read rate (valid_reads/opens, last 30 days)
18. Topic engagement trend (is this topic trending up or down for user?)
19. Items from this source in current session already (count — source fatigue)
20. Same-topic items in current session already (count — topic fatigue)

### 9.3 The Ranking Distillation Insight

From the KDD 2018 Ranking Distillation paper: a student model with <50% of the teacher's parameters can achieve equal or BETTER ranking performance when trained with ranking distillation.

**Application to Feedmine:** Feedmine doesn't have a "teacher" in the traditional sense. But:
- Use a large pre-trained model (e.g., GPT-class) to score a sample of 1000 articles offline
- Train the lightweight local model to replicate those rankings
- The local model learns the "reasoning" of the large model compressed into a tiny model
- Periodically refresh: re-score a sample with the large model, retrain the local model

This is a one-time (or monthly) cloud operation that could be opt-in. The trained local model runs entirely on-device afterward.

### 9.4 The Knowledge Distillation Pipeline for Feedmine

```
OFFLINE (optional, periodic, opt-in):
  1. Collect 1000 recent articles from user's subscriptions
  2. Score each with a large model (GPT-4 class) for "value to this user" (1-10 scale)
  3. Train lightweight local model to replicate these rankings
  4. Export model weights to device (~5MB)

ON-DEVICE (always):
  1. Use the distilled lightweight model for all ranking
  2. Update continuously with user behavioral signals
  3. The model starts smart (from distillation) and gets smarter (from online learning)
```

---

## 10. Open Questions and Tensions

### 10.1 Fundamental Tensions

**Engagement vs. Value**
The WeChat research and Meta's MSI pivot both confirm: what users click ≠ what makes them satisfied. Users click clickbait, doomscroll news, and engage with outrage — then report lower satisfaction. Feedmine must explicitly optimize for "value" signals (save, long dwell, return-to-read) not just "engagement" signals (clicks, opens).

But: how do you know what "value" means for a specific user without asking? The proxy signals (save, dwell) are imperfect. Periodic satisfaction surveys (1-question: "Was today's feed worth your time?" Yes/No) may be the only ground truth.

**Personalization vs. Bubble**
More personalization = better relevance = narrower exposure. The optimal point varies by user and by time. Some sessions the user wants deep focus (high personalization). Other sessions they want discovery (low personalization). The diversity slider partially addresses this, but most users won't touch settings.

**Recency vs. Quality**
A mediocre article from 2 hours ago vs. an excellent article from 3 days ago. How much should freshness dominate? The answer is session-dependent: "catching up" sessions tolerate older content, "quick check" sessions demand freshness.

**Model Complexity vs. Interpretability**
A neural network gives better predictions but the user can't understand why items are ranked. A simple weighted-feature model is worse but transparent ("This is here because you read 5 tech articles today"). Feedmine should prioritize interpretability — it builds trust and helps users tune the system.

### 10.2 Unsolved Problems

**Evaluation for N=1 systems:**
How do you know if the ranking is good when there's only one user and no control group? Approaches:
- Interleaving: show two rankings alternately, track which one gets more engagement
- Counterfactual evaluation: estimate what would have happened under a different policy
- Longitudinal metrics: track satisfaction survey scores over weeks/months
- "Regret" estimation: how often did the user scroll past highly-ranked items to engage with lower-ranked ones?

**Optimal exploration rate:**
How much to explore vs. exploit? Theory says it should decrease over time (as confidence grows), but sudden interest shifts require re-exploration. No clean answer exists — heuristic approaches (5-10% baseline, spike on detected drift) are the practical standard.

**Content understanding without a taxonomy:**
Feedmine can't maintain a fixed topic taxonomy (the user's interests are unique). Embedding-based clustering is flexible but:
- How many clusters? (Start with k=20, auto-adjust based on subscription diversity)
- When to merge/split clusters? (When cosine similarity between cluster centroids > 0.85, merge)
- How to name clusters for the user? (Show top-3 keywords from representative articles)

**When to give up on a source:**
If a user subscribed to a feed but never engages with it, at what point should the system effectively zero it out? And should it tell the user "You haven't read anything from [Source] in 3 months — still want it?"

### 10.3 What Facebook Learned the Hard Way

1. **MSI backfired on polarization:** Rewarding comments boosted outrage content. Feedmine lesson: weight saves/bookmarks higher than any social signal.

2. **Real-time > batch:** Updating user models in real-time (every interaction) massively outperforms nightly batch updates. The freshness of the user model matters more than model complexity.

3. **Light users get worse recommendations:** The system optimizes for heavy users who generate training data. Light users get stuck in cold-start forever. Feedmine must handle the case where a user checks the app twice a week with only 2-3 interactions per session.

4. **Over-optimization degrades satisfaction:** Pure score-based ranking eventually makes feeds feel "same-y." Controlled randomness (5-10% unexpected items) consistently improves long-term satisfaction metrics even though it temporarily reduces click-through rates.

5. **Content quality ≠ engagement:** The most engaging content is often the lowest quality (outrage, gossip, conflict). Explicitly weighting for "positive" engagement (saves, long thoughtful reads) over "any" engagement (clicks, rage-comments) is necessary.

6. **Users can't articulate preferences:** Explicit preferences (what users SAY they want) diverge from revealed preferences (what they actually engage with). Use both: explicit preferences for hard constraints (topics to avoid), implicit signals for soft ranking.

### 10.4 Research Gaps Relevant to Feedmine

1. **Single-user temporal evaluation:** Almost no academic work on evaluating recommendation quality for a single user over time. Feedmine will need to invent its own evaluation framework.

2. **Cross-device state for on-device models:** If the user uses Feedmine on phone and laptop, the user model needs to sync without a server. Local-first sync protocols (CRDTs?) could solve this.

3. **LLM-assisted cold start:** Can a local LLM (Llama-class, 4-bit quantized) bootstrap user preferences from the subscription list + first 20 articles? This is unexplored but promising.

4. **Natural language feed control:** "Show me more long reads this week" / "I'm tired of AI articles for now" — translating natural language into model parameter adjustments. Feasible with small local LLMs.

5. **Reading comprehension vs. skimming:** Dwell time alone can't distinguish "read carefully" from "left tab open." Scroll depth + scroll speed + return patterns could create a better "actual reading" signal.

---

## Appendix: Reference Implementations

### Open-Source Feed Ranking Systems

| System | What It Does | Relevance to Feedmine |
|--------|-------------|----------------------|
| X/Twitter Algorithm (GitHub: xai-org/x-algorithm) | Full "For You" ranking, open-sourced Jan 2026 | Reference for signal weights, engagement hierarchy |
| Meta DLRM (GitHub: facebookresearch/dlrm) | Deep Learning Recommendation Model | Reference architecture for the scorer |
| Meta Early Stage Retrieval (GitHub: facebookresearch/early_stage_retrieval) | Candidate generation + collaborative filtering + RL | Concepts for ranking with exploration |
| Winds RSS Reader (getstream.io) | Personalized RSS with Stream's ML | Closest existing product (discontinued) |
| airss (iOS App) | "Intelligent RSS reader powered by on-device ML" | Direct competitor/inspiration |
| RSS-GPT (GitHub: yinan-c/RSS-GPT) | Uses ChatGPT to summarize/filter RSS | Cloud-dependent approach |

### Key Papers (By Relevance to Feedmine)

1. **Reweighting Clicks with Dwell Time** (Xie et al., WWW 2023) — Valid read definition, normalized dwell time function
2. **Ranking Distillation** (Tang & Wang, KDD 2018) — Student model achieves teacher performance at <50% size
3. **A Half-Life Decaying Model** (Ardagelou & Arampatzis, 2017) — 150-day half-life for interest decay
4. **DistilBERT** (Sanh et al., 2019) — 40% smaller BERT retaining 97% capability, on-device proof-of-concept
5. **Meta SUM/DLRM** papers — Real-time embeddings, sparse+dense feature architecture
6. **Adaptive CF with Personalized Time Decay** (2023) — Per-user, per-topic decay rates
7. **Shaping Engagement with Novel Content** (WeChat, 2025) — Algorithms > peer influence for content diversity

---

*Document generated: 2026-07-06. Based on research rounds 1-3 of the Players Loop, Facebook intelligence deep dive. This is a living document — update as Feedmine features are implemented and validated.*
