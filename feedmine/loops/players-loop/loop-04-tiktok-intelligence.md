# TikTok Intelligence: Complete Technical Reference for Feedmine

> A comprehensive knowledge base on TikTok/ByteDance's recommendation system architecture, designed as actionable reference material for building a local-only feed reader that replicates TikTok's intelligence.

---

## 1. Executive Summary

TikTok's recommendation system is fundamentally different from every other major platform because it **optimizes for content-person fit rather than social graph proximity**. While Facebook shows you what your friends like, and YouTube recommends based on what similar viewers watched, TikTok treats every piece of content as an independent candidate evaluated against a real-time model of your current interests.

### Why TikTok wins:

1. **No social graph dependency.** A new user with zero followers sees a personalized feed within minutes. Content from a creator with 12 followers competes on equal footing with content from one with 12 million.

2. **Real-time learning at minute-level granularity.** The Monolith system (ByteDance's production architecture, published RecSys 2022) synchronizes model parameters every 60 seconds. Traditional systems batch-update daily or hourly. This means TikTok's model reflects what you did 1 minute ago, not 1 hour ago.

3. **Completion rate as primary signal.** TikTok's atomic unit is a short video watched in full-screen. This creates an unambiguous engagement signal: you either watched it or you didn't. There's no partial-attention browsing like scrolling a text feed.

4. **Multi-modal content understanding.** TikTok doesn't rely on creator-supplied metadata. It processes the actual video frames (3D CNNs), audio (fingerprinting + rhythm extraction), on-screen text (OCR via Transformers), and spoken words (ASR) to understand what content IS, independent of what the creator says it is.

5. **Tiered distribution = built-in A/B testing.** Every piece of content is tested against small audiences first, then graduated to larger pools only if it performs. This is continuous content-level experimentation at a scale no other platform matches.

The result: TikTok achieves 94% accuracy in predicting user interests (per their internal metrics), with a 14-18% AUC improvement over batch-training systems. The algorithm learns a new user's preferences within approximately 65-200 videos (roughly 30 minutes to 2 hours of use).

### The core philosophy difference:

- **Facebook/Instagram:** "Show people content from entities they've already chosen to follow"
- **YouTube:** "Show people content similar to what they've already watched"
- **TikTok:** "Predict what this person wants to see RIGHT NOW, from the entire content universe, regardless of who made it or what they watched yesterday"

---

## 2. The Recommendation Pipeline (Complete Multi-Stage Architecture)

TikTok uses a cascaded multi-stage architecture. Each stage progressively narrows the candidate set while increasing scoring complexity.

### Stage 1: Candidate Generation (Retrieval)

**Purpose:** Reduce billions of eligible videos to ~10,000 candidates in milliseconds.

**Methods used in parallel:**
- **Deep Retrieval (DR):** ByteDance's non-Euclidean retrieval structure (published 2021). Learns discrete latent codes to build retrieval paths directly from user-item interactions. Uses hierarchical clustering (K^D paths) with beam search. Achieves sub-linear computational complexity while maintaining brute-force accuracy.
- **Collaborative filtering paths:** User-to-user similarity via embedding space proximity.
- **Content-based paths:** Video embeddings matched against user interest embeddings.
- **Trending/recency paths:** Fresh content with early positive signals.
- **Creator-affinity paths:** Content from creators the user has engaged with.

### Stage 2: Pre-Ranking (Coarse Scoring)

**Purpose:** Reduce ~10,000 candidates to ~500 using lightweight models.

- Uses a simpler neural network (fewer layers, smaller embeddings)
- Filters obviously poor matches quickly
- Applies hard constraints (already-seen content, blocked creators, policy violations)

### Stage 3: Ranking (Fine Scoring)

**Purpose:** Score ~500 candidates with the full model and select the final ~20-50 for the session.

**Model architecture: Multi-tower DeepFM variant with online learning**

The production model follows a **multi-tower architecture**, with each tower predicting a different user behavior:
- Tower 1: Will the user complete the video? (completion probability)
- Tower 2: Will the user like? (like probability)  
- Tower 3: Will the user comment? (comment probability)
- Tower 4: Will the user share? (share probability)
- Tower 5: Will the user follow the creator? (follow probability)
- Tower 6: Long-term retention impact (will they come back tomorrow?)

Each tower shares a common embedding layer but has specialized upper layers. This is **Progressive Layered Extraction (PLE)**, published by Tencent at RecSys 2020 but adopted/extended by ByteDance. PLE separates shared and task-specific components with progressive routing, avoiding the "seesaw effect" where optimizing one task hurts another.

**The final ranking score** is a weighted combination of all tower predictions:

```
final_score = w1*P(complete) + w2*P(like) + w3*P(share) + w4*P(comment) + w5*P(follow) + w6*P(retention)
```

Leaked internal documents (NYT reporting) confirm the weights prioritize:
1. User value (immediate engagement) — heaviest weight
2. Long-term user value (retention)
3. Creator value (content quality incentives)
4. Platform value (ecosystem health)

### Stage 4: Re-Ranking (Diversity & Policy)

**Purpose:** Ensure the final sequence is diverse, policy-compliant, and session-optimized.

Rules applied:
- No two consecutive videos from the same creator
- No more than N videos on the same topic in a window
- No more than N videos using the same sound/music
- Inject exploration candidates (typically 10-20% of feed)
- Apply content-fatigue suppression
- Apply borderline-content distribution limits
- Optimize the sequence for session satisfaction (not just individual item scores)

### The Monolith System (Production Infrastructure)

The entire pipeline runs on **Monolith**, ByteDance's custom recommendation framework built on modified TensorFlow:

- **Collisionless embedding table:** Uses Cuckoo hashing (worst-case O(1) lookup). Eliminates hash collisions that degrade personalization in traditional systems. Each user/item gets its own embedding without sharing.
- **Streaming architecture:** Kafka queues for user actions + Flink streaming jobs for real-time feature joining + online training workers consuming data in real-time.
- **Online training:** After batch training on historical data, the model enters continuous online learning. Training workers consume real-time interaction data, update parameters, and sync to serving PS every minute.
- **Incremental parameter sync:** Only embeddings that were actually updated get synced (typically ~100K IDs per minute = ~400MB). Dense network weights sync daily.
- **Memory management:** IDs that appear fewer than N times are filtered out (never get an embedding). IDs inactive for a configurable period expire automatically. This handles the long-tail efficiently.

---

## 3. Signal Taxonomy

### 3.1 Engagement Signals (Explicit + Implicit)

| Signal | Weight (Relative) | Feedmine Equivalent |
|--------|-------------------|---------------------|
| **Video completion rate** | 10 (highest) | Read-through percentage, scroll-to-bottom |
| **Rewatch (looped)** | 10 | Re-open article, re-read |
| **Share** | 6 | Share action (if implemented) |
| **Comment** | 4 | N/A locally, could track "thought about" |
| **Like** | 2 | Explicit thumbs-up/star |
| **Follow creator** | 3 | Subscribe to feed source |
| **Add to favorites** | 4 | Bookmark/save |
| **Download** | 5 | Save offline |
| **Skip (scroll past quickly)** | -3 (negative) | Scroll past without opening |
| **Not interested (explicit)** | -8 (strong negative) | Explicit "less like this" |
| **Report** | -10 (strongest negative) | Block/hide |

### 3.2 Watch Behavior Signals (Micro-Interactions)

| Signal | What It Tells | Feedmine Equivalent |
|--------|---------------|---------------------|
| Watch duration (absolute) | Interest level | Time spent reading |
| Watch duration (% of video) | Completion/engagement | % of article scrolled |
| Pause behavior | Thinking/absorbing | Pause scrolling to read |
| Rewind/seek back | Missed something, high interest | Scroll back up |
| Speed of scroll-past | Disinterest intensity | Quick swipe = strong skip |
| Time before first interaction | Hesitation = mild interest | Hover time before click |
| Watch with sound on vs. off | Content type preference | N/A (text doesn't have audio) |

### 3.3 Content Signals

| Signal | Purpose | Feedmine Equivalent |
|--------|---------|---------------------|
| Hashtags/captions | Topic classification | Tags, categories |
| Audio/sound used | Trend detection + mood | N/A for text |
| Visual content (CV) | Object/scene recognition | Image analysis in articles |
| OCR text in video | Textual topic extraction | Body text NLP |
| Spoken words (ASR) | Topic from speech | Body text NLP |
| Video duration | Format preference signal | Article length |
| Creator identity | Creator affinity | Source/author affinity |
| Creation time | Freshness | Publication date |
| Music/audio category | Mood/genre matching | N/A |

### 3.4 Contextual Signals (Device + Environment)

| Signal | Purpose | Feedmine Equivalent |
|--------|---------|---------------------|
| Time of day | Temporal interest patterns | Current time |
| Day of week | Weekend vs. weekday behavior | Day of week |
| Device model | Economic/demographic proxy | N/A |
| OS version | Technical sophistication | N/A |
| Network type (WiFi/cellular) | Can handle heavy content? | Connection type |
| Screen size | Content format optimization | Screen/window size |
| Battery level | Session length prediction | N/A |
| Language setting | Content language matching | System locale |
| Geographic location | Regional content relevance | Location (if available) |
| App version | Feature availability | N/A |

### 3.5 Social Signals

| Signal | Purpose | Feedmine Equivalent |
|--------|---------|---------------------|
| Friends who liked | Social proof | N/A (local-only) |
| Creator's follower count | Authority signal (weak) | Source subscriber count |
| Video's total engagement | Quality/virality signal | Article popularity metrics |
| Comment sentiment | Content quality signal | N/A |
| Duet/stitch count | Conversation-worthy | N/A |

### 3.6 Negative Signals (Equally Important)

| Signal | Interpretation |
|--------|---------------|
| Scrolled past in <1 second | Strong disinterest in topic/creator/format |
| Watched 50% then scrolled | Content didn't deliver on promise |
| Opened comments but didn't watch | Curiosity about controversy, not content |
| Long-pressed then chose "Not Interested" | Explicit negative on topic |
| Hid creator | Never show this person again |
| Reported content | Policy violation signal + strong negative |



---

## 4. Content Understanding (How TikTok "Sees" Videos)

TikTok's content understanding pipeline processes every uploaded video through multiple AI systems simultaneously, creating a rich multi-modal embedding that captures what the content IS — independent of creator-supplied metadata.

### 4.1 Visual Understanding

**Architecture:** 3D Convolutional Networks for temporal pattern capture + CLIP-style image encoders for keyframe embeddings.

**What it extracts:**
- Scene classification (indoor/outdoor, kitchen, gym, office, nature)
- Object detection (food, animals, products, instruments)
- Action recognition (dancing, cooking, sports, talking)
- Face detection and expression (emotions, number of people)
- Visual aesthetics scoring (lighting, composition, resolution)
- Text overlays (via OCR with Transformer-based models, multi-language)
- Brand/logo detection
- Color palette and visual style

**Multi-modal foundation models:** ByteDance's Seed1.5-VL uses Mixture of Experts (MoE) architecture with 20 billion active parameters. Achieves SOTA on 38/60 public benchmarks for video understanding, OCR, 3D spatial comprehension.

### 4.2 Audio Understanding

**Pipeline components:**
- **Audio fingerprinting:** Identifies the exact sound/music used. Links videos using the same audio for trend detection.
- **Rhythm and tempo extraction:** Categorizes energy level (calm, upbeat, intense). Informs mood-matching for user sessions.
- **Music genre classification:** Matches musical preferences to user profiles.
- **Speech-to-text (ASR):** Transcribes spoken words for topic extraction. Multi-language support.
- **Audio event detection:** Identifies laughter, applause, nature sounds, ASMR-type audio.
- **Original vs. library sound:** Distinguishes creator-original audio from platform sound library usage.

**For Feedmine:** Audio analysis is irrelevant for text-based content. The equivalent is NLP on the body text — topic extraction, sentiment analysis, reading level, entity recognition.

### 4.3 Text Understanding (Captions, Hashtags, Comments)

**Architecture:** AMBERT (ByteDance's custom BERT variant) with multi-granularity tokenization — simultaneously processes input at character/subword AND phrase levels for enhanced semantic understanding.

**What it processes:**
- Caption semantic meaning (not just keyword matching)
- Hashtag categorization and trending detection
- Comment content analysis (sentiment, topic, toxicity)
- Search query understanding (for search-recommendation integration)

### 4.4 Multi-Modal Fusion

All modality-specific embeddings are fused into a single content representation:

```
content_embedding = fusion_network(visual_emb, audio_emb, text_emb, metadata_emb)
```

The fusion uses **cross-modal attention** — allowing, for example, audio features to inform the interpretation of visual features (a cooking video with upbeat music vs. with ASMR narration = different content types despite similar visuals).

### 4.5 "De-Tagging" — The End of Traditional Classification

ByteDance's fundamental breakthrough: **abandoning tag-based content classification entirely** in favor of end-to-end learning. Traditional systems (YouTube, Netflix) rely on human-curated taxonomies. TikTok learns content patterns directly from raw data through dense neural networks that map high-dimensional sparse features to compact representations.

This means:
- No predefined category system limits what the model can understand
- The system discovers content relationships that humans wouldn't categorize
- Novel content types are understood immediately without taxonomy updates
- Subtle content variations (same topic but different energy/style) are distinguishable

---

## 5. User Modeling (How Interests Form, Strengthen, and Decay)

### 5.1 The Sequential User Matrix (SUM)

TikTok represents each user as a **multi-channel network** where different channels represent different aspects of interest:

- Channel 1: Topic interests (what subjects they engage with)
- Channel 2: Format preferences (duration, style, energy level)
- Channel 3: Creator affinities (which voices resonate)
- Channel 4: Temporal patterns (what they want at different times)
- Channel 5: Social behavior patterns (lurker vs. commenter vs. sharer)

### 5.2 Interest Formation Speed

**Why TikTok learns faster than everyone else:**

1. **High signal density per unit time.** A user watches 10-20 videos per minute of use. Each is a full-attention, binary-outcome signal. Compare: on Facebook, a user might produce 2-3 meaningful engagement signals per minute of scrolling.

2. **No cold-start dependency on social graph.** Facebook needs your friend list to start. TikTok starts immediately with content-level signals.

3. **Minute-level model updates.** The Monolith online learning system propagates new user signals to the model within 60 seconds. Other platforms batch-update hourly or daily.

4. **Empirically measured:** Academic research (Baumann et al., 2025, Max Planck Institute) found that TikTok's algorithm identifies user interests within approximately **65-200 videos** (varies by topic). Gaming interests are detected fastest (~65 videos), Food takes longer (~140 videos). This translates to roughly 30 minutes to 2 hours of initial use.

### 5.3 Interest Strengthening

The system uses an **erase-add paradigm** with attention mechanisms at two levels:
- **Interest-level attention:** Tracks which broad topic categories are strengthening
- **Instance-level attention:** Tracks which specific content instances drove engagement

**Reinforcement dynamics (from academic audit):**
- Once interests are detected, the algorithm amplifies content aligned with them
- Gaming interest → 67.4% of feed becomes gaming content
- Food interest → 52.3% of feed becomes food content
- The system maintains ~33% non-interest content for exploration even at maximum amplification

### 5.4 Interest Decay

**Mechanisms for forgetting:**
- **Exponential time decay:** Recent interactions weighted exponentially more than older ones
- **Sliding window:** Only the last N interactions (or last T time period) are considered for the real-time model
- **Embedding expiration:** User embeddings for topics not engaged with for a configurable period are automatically expired from the system
- **Local proximity denoising:** Prevents over-optimization on very recent interactions that may be noise rather than signal

### 5.5 The "Interest Clock" (SIGIR 2024)

ByteDance published **Interest Clock** — a 24-hour personalized feature clock that uses Gaussian distribution smoothing to process discrete time encodings. It models hour-level user preferences:

- 7am: News/information content
- 12pm: Short entertainment during lunch
- 6pm: Longer, more immersive content
- 11pm: Relaxation/ASMR/low-energy content

After deployment in TikTok's music app: user active days +0.509%, app usage time +0.758%.

**Feedmine implication:** Time-of-day is a powerful contextual signal for content ranking, even locally.

### 5.6 Multi-Interest Representation

Users are NOT modeled as a single interest vector. TikTok maintains multiple simultaneous interest representations because people have multiple interests that activate in different contexts:

- User might be interested in both cooking AND gaming
- These interests don't blend into one vector — they're separate
- Context (time, session history, mood signals) determines which interest vectors are activated for the current recommendation request

**Architecture:** SASRec (Self-Attentive Sequential Recommendation) with bidirectional attention. Different attention heads focus on different aspects (semantic, temporal, format preference). Lower layers capture local patterns, higher layers capture global context.



---

## 6. The Tiered Distribution System

TikTok's content distribution is a graduated testing system — every piece of content goes through progressive audience expansion based on measured performance.

### 6.1 Tier Structure

**Tier 0: Upload Processing (0-30 minutes)**
- Content understanding pipeline processes the video
- Multi-modal embeddings generated
- Policy/safety classifiers run (automated)
- Content assigned initial topic clusters

**Tier 1: Initial Test Pool (~200-500 users)**
- Video shown to a small random sample within relevant interest clusters
- Mix of users who might like it (based on content understanding) and random exploration users
- KEY: Follower count is IRRELEVANT at this stage
- Duration: First few hours after upload

**Tier 2: Expanded Test (~5,000-50,000 users)**
- Only reached if Tier 1 metrics exceed thresholds
- Shown to broader interest-matched audiences
- Metrics measured: completion rate, engagement velocity, share rate
- Duration: 6-24 hours

**Tier 3: Broad Distribution (~100,000-1,000,000+ users)**
- Only reached if Tier 2 metrics remain strong
- Geographic and demographic expansion
- Content appears on many users' For You Pages
- Duration: 1-7 days

**Tier 4: Viral/Sustained (~millions)**
- Exceptional content that maintains engagement at scale
- May receive algorithmic boosts across regions/languages
- Can sustain distribution for weeks

### 6.2 Threshold Metrics (Approximate)

Based on creator community research and reverse engineering:

| Metric | Tier 1→2 Threshold | Tier 2→3 Threshold |
|--------|--------------------|--------------------|
| Completion rate | >50% | >60-70% |
| Like rate | >4% | >7% |
| Comment rate | >0.5% | >1% |
| Share rate | >0.5% | >2% |
| Engagement velocity | Positive acceleration | Sustained high |

**~70% completion rate is now needed for content to achieve viral distribution** (2026 data).

### 6.3 Why This Matters

The tiered system creates a **built-in content quality filter AND A/B testing mechanism:**
- Bad content dies at Tier 1 (minimal damage to user experience)
- Mediocre content reaches Tier 2 then plateaus (serves its niche)
- Great content graduates to broad distribution (scales naturally)
- Every video is an experiment: the system constantly learns what content qualities predict success

**Feedmine equivalent:** When new content arrives from subscribed sources, don't immediately rank it against proven content. Give it a small "test budget" — show it in lower-stakes positions first, measure engagement, then promote or demote based on actual performance.

---

## 7. Cold Start and Rapid Learning

### 7.1 Why TikTok Learns You in Minutes

**The Cold Start Problem:** How do you recommend content to a brand-new user with no history?

**TikTok's solution is architecturally unique:**

1. **No social graph required.** Facebook/Instagram literally cannot personalize without knowing who you follow. TikTok starts day one with zero social data.

2. **The first feed IS the experiment.** New users see a curated mix designed for maximum information extraction:
   - Popular content across diverse categories (entertainment, sports, comedy, education, music, cooking)
   - Each video the user watches/skips provides immediate signal
   - The full-screen interface makes every interaction unambiguous

3. **Signal density is extremely high.** In the first 10 minutes:
   - User has seen ~30-50 videos
   - Each video: watched duration, completion yes/no, any explicit engagement
   - That's 30-50 binary + continuous signals in 10 minutes
   - Compare: A new Netflix user might rate 3 shows in their first week

4. **Real-time model update.** Those 50 signals update the user's embedding within 60 seconds via Monolith's online learning. The 51st video recommendation already reflects all prior behavior.

5. **Content-first matching.** Because TikTok has rich content embeddings (multi-modal understanding), it can match users to content vectors immediately. No need to wait for collaborative filtering signals to accumulate.

### 7.2 The Information Extraction Strategy

TikTok's cold-start feed is designed to maximize **mutual information** — each video shown gives maximum information about the user's preferences:

- Show diverse content from different clusters
- If user completes a comedy video → narrow comedy sub-genres in next videos
- If user skips sports → eliminate sports cluster
- Binary tree search through interest space

**Measured result:** Academic research confirms the algorithm achieves significant personalization onset within 65-200 videos across different interest categories.

### 7.3 Registration Signals (Weak But Useful)

Even before first interaction, TikTok uses:
- Language/locale setting → language preference
- Device model → economic bracket proxy → content style preferences
- Country/region → cultural content pool
- Age (if provided) → age-appropriate content filtering
- Time of registration → time-zone aligned content

### 7.4 Feedmine Cold Start Strategy

For a local feed reader:
1. **First session:** Show diverse content from all subscribed sources, spread across topics
2. **Track engagement signals aggressively:** Time spent, scroll depth, re-opens, saves
3. **Rapidly build initial interest vectors** from the first 20-50 interactions
4. **Use content similarity** to immediately generalize: if user liked article A, find articles with similar embeddings
5. **Don't wait for collaborative filtering** — work content-based from day one



---

## 8. Exploration and Diversity (Preventing Rabbit Holes)

### 8.1 The Exploration-Exploitation Trade-Off

TikTok uses **reinforcement learning techniques** to balance showing content it knows you'll like (exploitation) vs. testing new content you might discover (exploration).

**Techniques confirmed:**
- **Upper Confidence Bounds (UCB):** Each candidate has a confidence interval. Items with wide uncertainty intervals get exploration bonus.
- **Thompson Sampling:** Bayesian approach where each candidate's expected reward is sampled from its posterior distribution. High-uncertainty items occasionally get sampled high.
- **BanditRank:** Treats ranking as a contextual multi-armed bandit problem. Directly optimizes ranking metrics (MAP, NDCG) while maintaining exploration.
- **Epsilon-greedy (with decay):** A fixed percentage of slots (~10-20%) are reserved for exploration candidates.

### 8.2 Content Fatigue Mechanisms

TikTok actively suppresses over-exposure to prevent user burnout:

**Same-creator fatigue:**
- No two consecutive videos from the same creator in the feed
- Frequency cap: Maximum N videos from one creator per session
- Even highly-liked creators get distributed across sessions

**Same-topic fatigue:**
- After consuming N videos on one topic, the system reduces that topic's ranking score
- Uses a per-topic saturation counter with time-based decay
- Adjacent topics may also get minor suppression (to avoid "cooking → baking → food → restaurants" tunnel)

**Same-sound fatigue:**
- Trending sounds are capped per session
- After hearing a sound N times, subsequent videos with that sound get ranked lower
- Prevents a single trending audio from dominating the feed

**Same-format fatigue:**
- If user has watched 10 talking-head videos in a row, boost visually different content
- Style diversity is actively managed

### 8.3 Measured Exploration Behavior

From the Max Planck Institute audit (2025):
- Even at maximum interest amplification (67% aligned content for gaming bots), the system maintained **~33% non-aligned content**
- The ratio of unique hashtags to total hashtags declined over time BUT never collapsed — the system preserves exploration
- Strong negative correlation: r = -0.92 between interest amplification and hashtag diversity for gaming
- The system appears to have a hard floor preventing complete filter bubble formation

### 8.4 Diversity Injection Strategies

Based on analysis of TikTok's behavior:

1. **Random exploration slots:** Fixed percentage of feed positions show content from outside known interests
2. **Adjacent-interest exploration:** Show content from topics neighboring known interests (e.g., user likes basketball → show sports training content)
3. **Trending injection:** Popular-at-platform-level content gets boosted regardless of user interest match
4. **New creator boost:** Fresh creators get exploration distribution to prevent platform stagnation
5. **Cross-interest seeding:** Deliberately test whether a cooking enthusiast might also like gardening

### 8.5 Feedmine Diversity Design

For a local feed reader:
- Reserve 15-20% of feed positions for exploration content
- Implement per-source frequency caps (no more than 3 items from same source consecutively)
- Track topic saturation counters with time decay
- Use content embedding distance to ensure visual/topical diversity in any window of N items
- Periodically inject random content from lower-ranked sources to test for interest drift

---

## 9. Session-Level Intelligence

### 9.1 Optimizing the Session, Not Just the Item

TikTok doesn't just maximize the score of each individual item — it optimizes for overall session satisfaction. This is a crucial distinction.

**Session-level objectives:**
- Maximize total session engagement (not just per-item engagement)
- Maximize probability of return (next-day retention)
- Maintain energy curve (don't exhaust the user)
- End sessions on a positive note (affects return probability)

### 9.2 Session Pacing (The "Energy Curve")

TikTok sequences content within a session to manage attention and emotion:

**Observed patterns:**
- Sessions often begin with high-engagement "hook" content (known interests, high energy)
- Gradually introduces exploration content after engagement is established
- Alternates between high-intensity and lower-intensity content
- Avoids consecutive heavy/emotional content (prevents burnout)
- Tends to present satisfying content before natural stopping points

### 9.3 Session Context Signals

The recommendation changes based on where you are in a session:

| Session Phase | Algorithm Behavior |
|---------------|-------------------|
| First 3-5 videos | Re-establish user model, show high-confidence content |
| Early session (5-20) | Mix of known interests + exploration |
| Mid session (20-50) | Deeper niche content, longer videos |
| Late session (50+) | May shift to lighter/shorter content |
| Near session end (predicted) | Show high-satisfaction content to end on positive note |

### 9.4 Inter-Session Learning

What happens between sessions matters:
- **Gap duration signal:** Long gap since last session → user might have drifted interests → increase exploration
- **Time-of-day shift:** Morning session after evening session → different content mode
- **Session-over-session trends:** If completion rates are declining across sessions → user might be getting bored → increase novelty

### 9.5 Sequence Optimization Techniques

The re-ranking stage uses sequence-aware optimization:
- **Determinantal Point Processes (DPP):** Mathematical framework for selecting diverse subsets. Maximizes a combination of individual quality and pairwise diversity.
- **Listwise learning to rank:** Rather than scoring items independently, the model learns to score item sequences.
- **Session-level reinforcement learning:** The "reward" is the full-session outcome (total engagement, return probability), not just per-item click.

### 9.6 Feedmine Session Design

For a local feed reader:
- Track session boundaries (app open/close, significant time gaps)
- Model where in the session the user is
- Front-load high-confidence content early in session to establish engagement
- Distribute exploration to mid-session (when attention is established)
- Track cross-session trends: if engagement is declining, inject more novelty
- Consider implementing a simple energy-curve: alternate between "heavy" and "light" content



---

## 10. Real-Time Signal Propagation

### 10.1 Speed of Adaptation

TikTok's system adapts to new signals faster than any competitor. Here's the timeline:

| Event | Time to Impact |
|-------|---------------|
| User likes a video | ~60 seconds (next parameter sync) |
| User skips 5 videos of a topic | ~60 seconds (immediate suppression) |
| A video gets unusual engagement spike | ~minutes (boosted in distribution) |
| User returns after weeks of absence | ~first 3-5 videos (re-calibration) |
| New trending sound emerges | ~hours (trend detection propagates) |
| Creator posts for first time | ~30 minutes (content understanding + Tier 1 distribution) |

### 10.2 The Online Learning Loop

```
User Action → Kafka Queue → Flink Online Joiner → Training Example
  → Online Training Worker → Parameter Update → Training PS
  → Incremental Sync (every ~60 seconds) → Serving PS
  → Next recommendation reflects the update
```

**Total loop latency: ~60-120 seconds from action to model update.**

Compare:
- YouTube: Hours to days (batch training)
- Netflix: Hours (near-real-time for some signals, batch for model updates)
- Facebook: Minutes for some signals (rule-based), hours for model updates
- Spotify Discover Weekly: **Weekly** (batch computation)

### 10.3 What Makes Real-Time Possible

1. **Streaming architecture:** No batch job to wait for. Data flows continuously through Kafka → Flink → Training.
2. **Incremental sync:** Only changed parameters propagate. Not the whole model.
3. **Sparse update dominance:** In any given minute, only ~100K embeddings change out of billions. This is a tiny data transfer.
4. **Tolerance for inconsistency:** ByteDance proved that having slightly stale dense parameters (updated daily) while sparse parameters are fresh (updated minutely) has negligible quality impact. This dramatically reduces sync overhead.
5. **Fault tolerance trade-off:** Monolith snapshots only once per day. If a parameter server crashes, they lose one day of updates for a tiny fraction of users. Empirically: with 0.01% PS failure rate per day, this affects ~15,000 users (out of millions) every 10 days. Acceptable.

### 10.4 Concept Drift Handling

The underlying distribution of user behavior is non-stationary — interests shift, trends come and go, cultural moments happen. This is called **Concept Drift**.

TikTok handles it through:
- **Continuous online learning:** The model is never "finished training." It always trains on the latest data.
- **Exponential recency weighting:** Recent interactions count exponentially more than old ones.
- **Embedding expiration:** Old, inactive embeddings automatically expire and free memory.
- **Single-pass training:** Historical data is trained only once. The model doesn't memorize the past — it learns from it and moves on.

### 10.5 Feedmine Real-Time Design

For a local feed reader:
- Update user model IMMEDIATELY on every interaction (no batching needed locally)
- Apply exponential time decay to all historical signals
- Implement a "session memory" that reflects the current session's signals instantly
- Track signal velocity: the RATE of engagement change matters (3 likes in a row on one topic = rapid interest formation)
- Don't wait for enough data — even a single strong signal (long read time, save, share) should immediately influence the next recommendations

---

## 11. What Can Be Done Locally (No Server Required)

### 11.1 Fully Local — High Feasibility

These TikTok techniques work perfectly on a single device:

| Technique | Local Implementation | Complexity |
|-----------|---------------------|------------|
| **Engagement signal tracking** | SQLite/local DB for all interactions | Low |
| **Time-decay interest modeling** | Exponentially-weighted moving averages | Low |
| **Content-based filtering** | Pre-computed embeddings (sentence-transformers) | Medium |
| **Completion rate tracking** | Read-time / article-length ratio | Low |
| **Session-level sequencing** | Rule-based diversity + energy curve | Medium |
| **Topic fatigue counters** | Per-topic counters with time decay | Low |
| **Source frequency capping** | Simple counters per source per session | Low |
| **Cold start exploration** | Diverse initial sampling from all sources | Low |
| **Time-of-day preferences** | Hour-bucketed engagement statistics | Low |
| **Interest Clock** | Gaussian-smoothed hourly preference vectors | Medium |
| **Multi-interest vectors** | K-means clustering on engagement history | Medium |
| **Re-ranking for diversity** | MMR (Maximal Marginal Relevance) algorithm | Medium |
| **Negative signal processing** | Explicit "less like this" + skip tracking | Low |

### 11.2 Needs Adaptation — Medium Feasibility

| Technique | Challenge for Local | Adaptation Strategy |
|-----------|--------------------|--------------------|
| **Collaborative filtering** | No other users | Use content similarity as proxy |
| **Multi-modal understanding** | CPU-intensive | Use pre-computed embeddings from lightweight models (MiniLM, TinyBERT) |
| **Online model training** | Resource usage | Use simpler models (logistic regression, small MLPs) that train instantly |
| **Embedding tables** | Memory for large vocabularies | Use dimensionality reduction, smaller embedding dims |
| **Trending detection** | No platform-level signals | Track personal "trending" (rapid engagement with new topics) |
| **Creator network effects** | No social graph | Use content similarity between sources |

### 11.3 Cannot Replicate Locally

| Technique | Why Not | Alternative |
|-----------|---------|-------------|
| **Tiered distribution** | Requires multiple users to test against | Use content quality signals (source reputation, early-engagement prediction) |
| **Social proof signals** | No other users | Ignore — use content and personal history only |
| **Platform-level trends** | Single user = no crowd | Could ingest external trend signals (RSS trending, Hacker News, etc.) |
| **Deep Retrieval at billion scale** | No billion-item catalog | Direct embedding search over thousands of items (fast enough locally) |
| **Multi-tower PLE architecture** | Overkill for single-user | Simpler multi-objective scoring function |

### 11.4 The Local Advantage

Things a local system does BETTER than TikTok:

1. **Privacy:** All data stays on device. No compromise between personalization and privacy.
2. **Transparency:** User can inspect exactly why something was recommended.
3. **User control:** User can directly modify their interest vectors, boost/suppress topics.
4. **No platform incentives:** TikTok optimizes for TIME SPENT. A local system can optimize for USER SATISFACTION or LEARNING or whatever the user actually wants.
5. **Instant updates:** No network latency. Model updates are truly instant.
6. **No filter bubble concern:** User controls exploration rate directly.
7. **Cross-source intelligence:** Can learn across all content sources (RSS, newsletters, bookmarks) — something siloed platforms can't do.



---

## 12. What Feedmine Can Steal from TikTok (Practical Techniques)

### 12.1 Steal: Completion Rate as Primary Signal

**TikTok's insight:** A 15-second video watched completely beats a 60-second video watched for 40 seconds. The RATIO matters more than absolute time.

**Feedmine implementation:**
```
completion_score = time_spent / estimated_read_time(article_length)
```

Normalize by content length. A 500-word article read for 2 minutes is a stronger signal than a 3000-word article read for 3 minutes.

**Length-aware normalization:**
- Short articles (<500 words): Completion threshold = 80%+ read time
- Medium articles (500-2000 words): Completion threshold = 60%+
- Long articles (2000+ words): Completion threshold = 40%+ (reading 40% of a long piece is still significant engagement)

### 12.2 Steal: Multi-Signal Scoring with Weighted Combination

**Don't use a single signal. Combine them:**

```python
item_score = (
    w_complete * completion_signal +      # Highest weight (0.35)
    w_reread * reread_signal +            # Very high (0.25)
    w_save * save_signal +                # High (0.15)
    w_share * share_signal +              # High (0.10)
    w_time * normalized_time_signal +     # Medium (0.10)
    w_scroll * scroll_depth_signal +      # Low (0.05)
    w_negative * skip_penalty             # Negative (-0.20)
)
```

### 12.3 Steal: The Interest Clock (Time-of-Day Modeling)

**Maintain per-hour engagement statistics:**

```python
interest_clock = {
    "morning": {"topics": ["news", "productivity"], "length_pref": "short"},
    "lunch": {"topics": ["entertainment", "tech"], "length_pref": "medium"},
    "evening": {"topics": ["deep_reads", "hobby"], "length_pref": "long"},
    "night": {"topics": ["fiction", "relaxation"], "length_pref": "short"}
}
```

Use Gaussian smoothing across adjacent hours to avoid hard boundaries.

### 12.4 Steal: Exponential Decay on All Signals

**Everything decays. Recent behavior matters exponentially more.**

```python
signal_weight = base_weight * exp(-decay_rate * hours_since_interaction)
```

- Interactions from today: weight ≈ 1.0
- Interactions from yesterday: weight ≈ 0.7
- Interactions from last week: weight ≈ 0.3
- Interactions from last month: weight ≈ 0.05

This naturally handles interest drift without explicit "forgetting" logic.

### 12.5 Steal: Session Boundary Awareness

**Reset session context on each new session. Track:**
- Topics shown this session (for fatigue)
- Sources shown this session (for frequency capping)
- Energy level of content shown (for pacing)
- Number of items consumed (for session phase detection)

### 12.6 Steal: Negative Signals Are as Valuable as Positive

**TikTok tracks what you DON'T do with equal precision:**
- Scrolled past in <1 second = strong negative on that content type
- Opened but immediately closed = content didn't deliver on headline promise
- Never engaging with a source despite repeated exposure = suppress that source

**Feedmine must track:**
- Items shown but never opened
- Items opened but immediately closed (<5 seconds)
- Items consistently scrolled past from specific sources
- These are STRONG training signals for the model

### 12.7 Steal: Tiered Confidence for New Content

**New content from unknown sources has high uncertainty.**

Borrow TikTok's tiered distribution concept:
1. New source/topic: Show in lower-stakes position (below the fold)
2. If engaged with: Promote to higher-stakes positions
3. If consistently ignored: Gradually suppress

This is exploration-exploitation with positional risk management.

### 12.8 Steal: Multi-Interest Vectors (Not One Monolithic Profile)

**Don't model the user as one preference vector. Model them as multiple interest clusters:**

```python
user_interests = [
    {"topic": "machine_learning", "strength": 0.9, "last_active": "2h_ago"},
    {"topic": "cooking", "strength": 0.6, "last_active": "1d_ago"},
    {"topic": "politics", "strength": 0.4, "last_active": "3d_ago"},
]
```

Activate different interest vectors based on time-of-day, session context, and recent behavior.

### 12.9 Steal: Content Embedding Similarity (De-Tagging)

**Don't rely on tags/categories. Use actual content embeddings:**

```python
from sentence_transformers import SentenceTransformer
model = SentenceTransformer('all-MiniLM-L6-v2')

article_embedding = model.encode(article_title + " " + article_body[:512])
user_interest_embedding = weighted_average(recent_liked_article_embeddings)

similarity = cosine_similarity(article_embedding, user_interest_embedding)
```

This captures semantic similarity that tags miss. Two articles both about "AI in healthcare" will have similar embeddings even if tagged differently.

### 12.10 Steal: Diversity Re-Ranking (MMR)

After scoring all candidates, apply **Maximal Marginal Relevance** to ensure the final list is diverse:

```python
def mmr_rerank(candidates, lambda_param=0.7):
    selected = []
    while candidates and len(selected) < target_count:
        best = max(candidates, key=lambda c:
            lambda_param * relevance_score(c) -
            (1 - lambda_param) * max_similarity_to_selected(c, selected)
        )
        selected.append(best)
        candidates.remove(best)
    return selected
```

This ensures each new item adds maximum INFORMATION to the feed, not just maximum individual relevance.

---

## 13. Key Differences from Facebook, Instagram, and Google

### 13.1 TikTok vs. Facebook/Instagram

| Dimension | TikTok | Facebook/Instagram |
|-----------|--------|-------------------|
| **Primary signal source** | Content interactions (what you watch) | Social graph (who you follow) |
| **Content discovery** | Algorithm-driven from entire universe | Primarily from followed accounts + their network |
| **Cold start** | Works immediately from behavior | Requires friend/follow connections first |
| **Creator equality** | 0-follower creator can go viral | Follower count heavily influences reach |
| **Feed composition** | ~95% from non-followed accounts | ~60-80% from followed accounts |
| **Engagement model** | Full-screen, one-at-a-time (forced choice) | Scroll-based, multiple items visible (partial attention) |
| **Signal clarity** | Binary: watched or skipped | Ambiguous: scrolled past ≠ dislike |
| **Update speed** | Minute-level model updates | Hour-level for most signals |
| **Content understanding** | Deep multi-modal (vision, audio, text) | Primarily metadata and text-based |
| **Optimization target** | Watch time + retention | Engagement (likes, comments, shares) |

### 13.2 TikTok vs. YouTube

| Dimension | TikTok | YouTube |
|-----------|--------|--------|
| **Content length** | 15-180 seconds (atomic units) | Minutes to hours (variable) |
| **Recommendation speed** | Real-time online learning | Batch + near-real-time hybrid |
| **Signal volume** | 10-20 signals per minute of use | 1-3 signals per minute of use |
| **Discovery mechanism** | Every video is a discovery opportunity | Search + sidebar recommendations |
| **Creator weight** | Low (content > creator) | High (subscriber base influences reach significantly) |
| **Architecture** | Monolith (single online system) | Candidate generation + separate ranking (two-tower) |
| **Cold start speed** | ~65 videos to personalize | Days to weeks for good recommendations |
| **Content understanding** | Real-time multi-modal at upload | Primarily metadata + thumbnails + transcripts |

### 13.3 TikTok vs. Spotify (Discover Weekly)

| Dimension | TikTok For You | Spotify Discover Weekly |
|-----------|---------------|----------------------|
| **Update frequency** | Every 60 seconds | Weekly (batch computation) |
| **Primary technique** | Deep learning + online training + content understanding | Collaborative filtering + NLP on playlists + audio features |
| **Recommendation style** | Real-time stream (instant feedback) | Curated playlist (delayed feedback) |
| **Cold start** | Minutes (behavior-based) | Weeks (needs listening history + collaborative signals) |
| **Signal type** | Completion, skip, engagement (rich implicit) | Listen duration, save, skip (limited implicit) |
| **Content understanding** | Multi-modal (video, audio, text, visual) | Audio features (tempo, key, energy) + metadata |
| **Diversity approach** | Exploration slots + fatigue mechanisms | Genre/artist diversity in playlist construction |
| **Scale of personalization** | Per-item, per-second | Per-playlist, per-week |
| **What it optimizes** | Immediate engagement + session retention | Discovery satisfaction + long-term listening breadth |

**Key insight for Feedmine:** Spotify's weekly batch approach creates anticipation and ritual, but TikTok's real-time approach creates compulsiveness. Feedmine should lean toward TikTok's responsiveness while potentially offering Spotify-style "digest" modes for intentional reading sessions.

### 13.4 TikTok vs. Google (Search/Discover)

| Dimension | TikTok | Google Discover |
|-----------|--------|----------------|
| **Trigger** | Always on (open app = feed) | Search intent OR passive discover feed |
| **User intent** | Browse/entertainment (no explicit query) | Often explicit information need |
| **Ranking inputs** | Behavioral signals dominate | Query relevance + authority + freshness |
| **Content freshness** | Critical (hours/days old content) | Important but less so (evergreen content ranks) |
| **Source authority** | Irrelevant (content quality > source reputation) | Critical (domain authority, E-E-A-T) |
| **Personalization depth** | Extreme (per-user, per-moment) | Moderate (topic interests, search history) |

---

## 14. Open Questions

### 14.1 Unknown Technical Details

1. **Exact multi-task weights.** We know the system combines P(complete), P(like), P(share), etc., but the exact weights are unknown. They likely vary by market and are continuously A/B tested.

2. **Exploration budget allocation.** What percentage of the feed is truly exploration vs. exploitation? Estimated at 10-33% but the exact mechanism for adjusting this per-user is unclear.

3. **Long-term interest vs. short-term mood.** How does TikTok distinguish between a genuine new interest forming vs. a temporary mood? The boundary between "interest drift" and "noise" is architecturally unclear.

4. **Creator scoring internals.** How does creator reliability/quality factor into content scoring? We know it's not just about content quality — creator consistency and posting patterns likely matter.

5. **Audio processing depth.** How deeply does TikTok understand audio content? Music genre and rhythm are confirmed, but emotional valence detection from audio, spoken sentiment analysis, and audio-visual correlation depth are unclear.

6. **Exact Monolith model size.** The paper mentions "multi-terabyte" models but doesn't specify architecture depth, embedding dimensions for production, or number of towers exactly.

### 14.2 Things That Remain Difficult for Local Systems

1. **Collaborative filtering without other users.** TikTok's power partly comes from "people like you also liked X." A local system has only one user. Potential workaround: Use content similarity as proxy, or ingest public "popular" signals from APIs.

2. **Content quality estimation without crowd signals.** TikTok uses engagement velocity across many users to estimate content quality. A local system must use source reputation, content features, or external popularity signals instead.

3. **Trend detection.** Detecting what's trending requires a user population. A local system could subscribe to external trend feeds or detect personal micro-trends.

4. **Scale of content understanding.** Running full multi-modal AI on every piece of content is resource-intensive. Local systems need to be strategic about which content gets deep analysis.

### 14.3 Research Questions for Feedmine

1. **Optimal decay rates for different signal types.** Should a "save" action decay at the same rate as a "high read time"? Probably not — saves indicate enduring interest.

2. **Interest vector dimensionality.** How many dimensions does a user need for good recommendations locally? TikTok uses high-dimensional embeddings; local systems might do well with 50-128 dimensions.

3. **Session pacing effectiveness.** Does actively managing content energy curve improve satisfaction in a reading context as much as video? Worth A/B testing.

4. **Negative signal calibration.** How much should a skip penalize content? Should 3 skips of a topic have the same weight as 1 explicit "not interested"?

5. **Completion rate normalization.** What's the right formula to normalize read-time across article lengths? TikTok's approach (ratio matters more than absolute) suggests a sigmoid or logarithmic normalization might work.

### 14.4 ByteDance A/B Testing Culture

From Cambridge research (2026, Strategic Management Journal):
- ByteDance's culture is built on continuous experimentation
- The company uses AI learning from one domain (news in Toutiao) to improve others (video in TikTok, e-commerce in TikTok Shop)
- Flat organizational structure enables rapid experimentation: employees are "classmates" not hierarchies
- Algorithm teams work directly with business units for rapid iteration
- The AI "self-improves through use" — more diverse application strengthens the underlying models
- Cross-domain learning: users who read urban crime stories on Fanqie Novel showed higher click-through on breaking news in Toutiao with similar emotional tones — this kind of cross-domain signal transfer is unique to ByteDance's unified data architecture

**For Feedmine:** Build a unified signal model across all content sources from day one. Don't silo newsletter reading behavior from RSS reading behavior from bookmark behavior. All signals feed one model.

---

## Sources and References

1. Liu, Z. et al. (2022). "Monolith: Real Time Recommendation System With Collisionless Embedding Table." RecSys '22. ByteDance Inc. [arxiv:2209.07663]
2. Baumann, F., Arora, N., Rahwan, I., Czaplicka, A. (2025). "Dynamics of Algorithmic Content Amplification on TikTok." Max Planck Institute for Human Development. [arxiv:2503.20231]
3. Tang, H. et al. (2020). "Progressive Layered Extraction (PLE): A Novel Multi-Task Learning Model for Personalized Recommendations." RecSys '20. Tencent. [ACM DL]
4. Zhang, Y. et al. (2024). "Interest Clock: Time Perception in Real-Time Streaming Recommendation System." SIGIR 2024. ByteDance.
5. Gao, J. et al. (2021). "Deep Retrieval: Learning A Retrievable Structure for Large-Scale Recommendations." [arxiv:2007.07203]
6. Wan, F., Yang, T., Shi, X., Rong, K., Ansari, S. (2026). "Scaling High and Wide: How Firms Leverage AI and Organizational Design to Overcome the Scale-Scope Trade-Off." Strategic Management Journal.
7. Zhou, X., Xu, H., Chen, M. (2026). "A video statistics-aware sequential recommendation model with multi-behavior feedback for short-video recommendation." Scientific Reports.
8. ByteDance Seed Team (2024). "Seed1.5-VL: A Vision-Language Foundation Model." 
9. TikTok Official (2024). "How TikTok recommends content." support.tiktok.com.
10. NYT reporting on leaked TikTok internal documents (2024) — four optimization objectives.

---

*Document created: July 2026*  
*Purpose: Technical reference for Feedmine local feed intelligence*  
*Status: Complete — ready for implementation reference*
