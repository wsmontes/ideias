# Loop 02: Instagram Intelligence — Complete Technical Reference

> **Purpose:** Actionable knowledge base for building a local-only feed reader (Feedmine) that replicates Instagram's ranking intelligence without server infrastructure.
> **Scope:** Instagram-specific mechanisms only. Facebook differences noted where relevant.
> **Date:** July 2026 | **Sources:** Meta engineering papers, system cards, SIGKDD/ICML/ECML-PKDD publications, official transparency documents, Adam Mosseri public statements.

---

## 1. Executive Summary

### How Instagram Ranking Works (Plain Language)

Instagram does not have "an algorithm." It operates **four separate ranking systems** — one each for Feed, Stories, Explore, and Reels — each optimizing for different outcomes using different signal hierarchies.

The core loop is:
1. **Candidate generation:** Gather eligible content (from followed accounts for Feed/Stories; from the entire platform for Explore/Reels)
2. **Feature extraction:** For each candidate, compute hundreds of features about the content, the creator, the viewer, and the relationship between them
3. **Multi-task prediction:** A deep neural network predicts probabilities of multiple actions (like, save, share, comment, watch completion, "see fewer posts like this")
4. **Value scoring:** Combine predictions into a single score using learned weights
5. **Diversity reranking:** Apply penalties to prevent repetition by author, topic, or format
6. **Display:** Show ranked results with balanced content mix

The **three most important signals** across all surfaces (confirmed by Adam Mosseri, January 2025):
1. **Watch time** — how long you spend with content
2. **Likes per reach** — proportion of people shown who liked
3. **Sends per reach** — proportion who shared via DM (strongest for discovery)

### How Instagram DIFFERS from Facebook

| Dimension | Instagram | Facebook |
|-----------|-----------|----------|
| Primary graph | Interest graph (what you like) | Social graph (who you know) |
| Visual processing | Primary classification method | Secondary to text/links |
| Discovery orientation | Core product goal (Explore, Reels) | Secondary feature |
| Signal hierarchy | DM shares > saves > comments > likes | Meaningful social interactions > reactions |
| Content goal | Individual interest exploration | Community connection |
| Political content | Filtered from all recommendations (2024+) | More permissive |
| Original content bonus | 3-5× reach multiplier for originals | Less aggressive |
| Unified metric | "Views" across all formats (2026) | No equivalent |
| User control | "Your Algorithm" topic declarations | Not available |

**Key architectural difference:** Instagram builds user identity around an **interest embedding** (DV365) trained on 70,000 interactions per user across all surfaces. Facebook builds identity around **social connections and community membership**. This means Instagram's approach is more transferable to a local feed reader that has no social graph.

---

## 2. The Four Ranking Systems

### 2.1 Feed (Home)

**What it ranks:** Posts from accounts you follow + interleaved recommendations from non-followed accounts.

**Pipeline:**
1. Gather all recent posts from followed accounts
2. Lightweight model selects ~700 most relevant candidates
3. Score ~500 candidates with full-feature model
4. Apply integrity/safety filters
5. Final ranking with diversity constraints
6. Ensure balanced mix of content types (not all Reels, not all photos)
7. Interleave recommended content from non-followed accounts

**Primary signals:**
- Relationship closeness (interaction history with creator)
- Interest alignment (content topic matches user interests)
- Recency (newer posts ranked higher)
- Popularity (early engagement velocity)
- Content type preferences (user's historical format affinity)

**Optimization target:** Keep followed-account content feeling fresh and relevant while introducing discovery.

**Feed Recommendations (Suggested Posts):**
Once a user scrolls past all available content from followed accounts, the system transitions to **Feed Recommendations** — content from accounts the user does NOT follow. This uses the same pipeline as Explore but operates within the Feed surface. It's the bridge between Feed and Explore: same candidate generation (ig2vec, collaborative filtering, session-based signals) but displayed inline rather than in a grid.

The Feed Recommendations system:
- Is powered by the same AI system as Explore
- Uses multiple ML models working together
- Selects candidates based on predicted value to the specific user
- Applies the same diversity and integrity filters
- Users can control this via "Suggested Posts" preferences

### 2.2 Stories

**What it ranks:** Stories from accounts you follow only (no discovery).

**Primary signals:**
- Viewing history (do you regularly watch this person's Stories?)
- Relationship closeness (DMs, comments, profile visits)
- Timeliness (24-hour lifecycle)
- Interactive engagement (replies to polls, questions, quizzes)

**Key characteristic:** Stories is NOT a discovery surface. It deepens existing relationships. 70% of Instagram users watch Stories daily. The ranking is primarily about **ordering the circles at the top** — putting the most relevant connections first.

### 2.3 Explore

**What it ranks:** Content from accounts you do NOT follow — pure discovery.

**Pipeline (most complex of the four):**

**Stage 1: Candidate Generation**
- Identify "seed accounts" (accounts user has interacted with)
- Use **ig2vec** (word2vec-like account embeddings) to find similar accounts
- Retrieve recent media from similar accounts
- Apply collaborative filtering ("people who liked X also liked Y")
- Apply **Personalized PageRank on session graphs** (2024 addition — ECML PKDD paper)
- Result: tens of thousands of eligible candidates

**Stage 2: Three-Pass Ranking**

| Pass | Model | Candidates In | Candidates Out |
|------|-------|---------------|----------------|
| First | Distilled lightweight model | 500 | 150 |
| Second | Medium neural network | 150 | 50 |
| Final | Deep MTML model | 50 | 25 (first page) |

**Stage 3: Diversity Reranking**
- Down-rank same-author posts progressively
- Down-rank posts from same seed account (topic dedup)
- Multiplicative demotion penalties (0-1 range)
- Maximal marginal relevance approach

**Value Model (Final Pass) predicts:**
- P(Like), P(Save), P(Share), P(Comment), P(Click)
- P(See Fewer Posts Like This) — negative weight
- P(Watch completion) — for video content

**Score formula:**
```
Score = W_like × P(Like) + W_save × P(Save) + W_share × P(Share)
      + W_comment × P(Comment) - W_negative × P(SFPLT) + ...
```

Weights tuned via offline replay + Bayesian optimization. Exact weights not published.

### 2.4 Reels

**What it ranks:** Short-form video, primarily from accounts you don't follow (discovery engine).

**Primary signals (in order of importance):**
1. **Watch time** — #1 factor, confirmed publicly
2. **Completion rate** — carries "300% more weight" than previously (2026 update). 80% completion sends dramatically stronger signal than 30% completion.
3. **Replay rate** — watching more than once directly multiplies watch time signal
4. **DM shares per reach** — most important engagement signal for unconnected distribution
5. **Likes per reach** — important for connected distribution
6. **Saves** — indicates long-term value

**Key mechanism: Debiased watch-time scoring**
```
score = ln(dwell_time + 1) / ln(γ × video_duration + 1)
```
This corrects for longer videos naturally accumulating more absolute watch time. A 15-second Reel watched completely scores higher than a 60-second Reel watched 40%.

**Reels-specific rules:**
- Under 3 minutes for recommendation eligibility
- Audio required (even background music)
- No watermarks from other platforms
- Original content gets 3-5× reach multiplier vs reposts
- Surveys users on "entertaining" and "funny" — unique entertainment optimization
- "Chaining": after initial Reel from a connection, chains to unconnected discovery content

**Trial Reels:** Test content with non-followers before releasing to your audience. Bypasses connected ranking → goes straight to recommendations.

---

## 3. Signal Taxonomy

### 3.1 Engagement Signals (User → Content)

| Signal | Strength | Surface Priority | Feedmine Equivalent |
|--------|----------|------------------|---------------------|
| DM share (send) | Strongest | Reels, Explore | Star/highlight + share action |
| Save | Very high | Explore, Feed | Bookmark/save to collection |
| Comment | High | Feed, Stories | — (no social layer) |
| Like | Medium | Feed, connected reach | Explicit "like" or thumbs-up |
| Watch time (absolute) | High | Reels | Time spent viewing item |
| Watch completion % | Very high | Reels | Scroll-past speed / dwell ratio |
| Replay/re-watch | High | Reels | Re-open same item |
| Carousel swipe | Medium | Feed | Scroll within multi-image |
| Story reply | Medium | Stories | — |
| Profile visit after viewing | Medium | All | Click-through to source |
| Follow after viewing | High | Explore, Reels | Subscribe to feed |

### 3.2 Negative Signals

| Signal | Strength | Feedmine Equivalent |
|--------|----------|---------------------|
| "Not Interested" / SFPLT | Strong | Explicit "less like this" |
| Hide post | Stronger | Remove from feed |
| Mute account | Strong | Mute source |
| Report | Strongest | Block source |
| Rapid scroll-past (<3s) | Implicit negative | Low dwell time |
| Session exit after viewing | Moderate negative | App close timing |
| No carousel swipe | Weak negative | No scroll in multi-item |
| Unfollow after viewing | Strong | Unsubscribe |

### 3.3 Content Signals (Properties of the item)

| Signal | What it tells the system | Feedmine Equivalent |
|--------|--------------------------|---------------------|
| Visual classification | Objects, scenes, composition | On-device CLIP/ViT embedding |
| On-screen text (Rosetta) | Topic from text overlays | Local OCR |
| Caption NLP | Keywords, topic, sentiment | Local NLP on description |
| Hashtags (max 5) | Classification confirmation | Tags from source |
| Alt text | Explicit content label | Alt text from RSS/feed |
| Audio type | Music vs speech vs original | Audio classification |
| Video duration | Format expectations | Duration metadata |
| Media type | Photo/video/carousel/reel | Content type field |
| Posting time | Freshness signal | Publication timestamp |
| Account consistency | Topic reliability | Source topic stability |

### 3.4 Relationship Signals (User ↔ Creator)

| Signal | Description | Feedmine Equivalent |
|--------|-------------|---------------------|
| Interaction frequency | How often user engages with creator | Engagement count per source |
| Interaction recency | When user last engaged | Last interaction timestamp |
| Mutual following | Bidirectional connection | — (no social layer) |
| DM history | Private messaging indicates closeness | — |
| Tagged together | Real-world connection indicator | — |
| Profile visits | Active interest in creator | Source page visits |
| Comment replies | Direct interaction depth | — |

### 3.5 Context Signals (Session & temporal)

| Signal | Description | Feedmine Equivalent |
|--------|-------------|---------------------|
| Time of day | Morning vs evening content preferences | Local clock |
| Day of week | Weekday vs weekend patterns | Local calendar |
| Session duration | How long user has been browsing | Session timer |
| Session depth | Items already viewed this session | View counter |
| Device type | Phone vs tablet vs desktop | Device detection |
| Network speed | Affects video recommendation | Connection quality |
| User activity pattern | Historical active hours | Usage pattern tracking |



---

## 4. Content Understanding (How Instagram "Sees" Content)

### 4.1 Visual Embedding Pipeline

Instagram processes every uploaded image and video through multiple vision systems:

**Layer 1: Low-level visual features**
- Object detection and scene classification (what's in the image)
- Face detection and counting
- Color palette extraction
- Compositional analysis (rule of thirds, symmetry, visual balance)
- Resolution and quality assessment

**Layer 2: Semantic understanding**
Meta's research lab has produced multiple visual embedding architectures used internally:

- **I-JEPA (Image Joint Embedding Predictive Architecture, 2023):** Self-supervised model that learns by predicting masked regions of images in abstract representation space (not pixel space). Learns semantic structure without data augmentation tricks.
- **V-JEPA (Video JEPA, 2024):** Extension to video — predicts missing parts of video in feature space. Used for understanding video content semantics.
- **ImageBind (2023):** Joint embedding space across 6 modalities (image, text, audio, depth, thermal, IMU). Enables cross-modal retrieval — find images similar to a text description, or audio similar to an image.
- **DINOv3 (2025+):** Self-supervised vision backbone producing high-resolution features. Powers lightweight adapter training for downstream tasks (classification, segmentation, tracking).

**Production deployment:** Instagram's content understanding likely uses distilled versions of these research models, optimized for throughput. The key insight is that Meta's approach learns **abstract semantic representations** of images rather than pixel-level matching — meaning two photos of different dogs in different poses will be recognized as "similar content" at the semantic level.

**Layer 3: Multi-modal fusion**
Content is not just visual. The full representation combines:
- Visual embedding (from vision model)
- Text embedding (from caption + on-screen text)
- Audio embedding (for video — music identification, speech transcription)
- Metadata features (posting time, format, account history)

These are fused into a single content representation used for retrieval and ranking.

### 4.2 Rosetta: Text-in-Images System

Deployed since 2018, processes **>1 billion images daily**:

- **Detection stage:** Faster-RCNN locates text regions in images
- **Recognition stage:** CNN decodes detected text regions
- **Multi-language support:** Works across all major languages
- **Real-time processing:** Runs at upload time
- **Output:** Feeds into classifiers that understand text-in-context (meme understanding, hate speech detection, content categorization)

This means Instagram "reads" text overlays on images — infographics, memes, quotes, instructional content — and incorporates that text into content understanding. A carousel with text "5 Tips for Better Sleep" gets classified as health/wellness content even if the images are just color backgrounds.

### 4.3 Account-Level Embeddings (ig2vec)

Rather than classifying individual posts in isolation, Instagram builds **account-level topic profiles:**

- **Training method:** Word2vec-like framework applied to sequences of account interactions within browsing sessions
- **Intuition:** If a user interacts with Account A and Account B in the same session, those accounts are "contextually similar" (like words appearing in the same sentence)
- **Session-bounded:** Only co-interactions within a session count — this ensures topical coherence
- **Result:** Every account gets an embedding vector; cosine distance measures topical similarity
- **Coverage:** Millions of accounts in the embedding space
- **Emergent taxonomy:** Topics emerge from clustering rather than being predefined. Covers everything from "Arabic calligraphy to model trains to slime"
- **Retrieval:** FAISS-based KNN lookup for nearest neighbors

**Key insight for Feedmine:** ig2vec means Instagram doesn't need a fixed topic taxonomy. It learns "this account is similar to that account" purely from user behavior patterns. A local system can replicate this by tracking which feeds the user engages with in the same session.

### 4.4 The Shift from Hashtags to AI Understanding (2024-2026)

- Hashtag limit reduced from 30 to **5 per post** (December 2025)
- Hashtags now function as **classification confirmation**, not reach drivers
- The algorithm infers topic without any explicit tags
- **Keywords in captions** now more important than hashtags for discovery
- Bio keywords, alt text, and on-screen text all feed classification
- Account's cumulative classification profile influences new content categorization

### 4.5 Content Quality vs Engagement Bait Detection

Instagram distinguishes quality content from engagement bait through:

**Engagement bait indicators (downranked):**
- Excessive calls to action ("like if you agree", "comment YES")
- Clickbait headlines with misleading preview
- Content that generates high CTR but low completion/satisfaction
- Content where engagement rate >> satisfaction rate (detected via "SFPLT" negative signal)
- Artificial engagement spikes (anomalous patterns)

**Quality indicators (boosted):**
- High completion rate relative to content length
- Save rate (indicates lasting value)
- DM share rate (indicates worth sharing privately)
- Low "See Fewer Posts Like This" rate
- Consistent account-level engagement (not one-hit spikes)
- Original content (not reposts)

**The asymmetry:** High likes alone is NOT a quality signal. The system specifically models the gap between "clicks/likes" and "satisfaction/SFPLT" to detect content that hooks but disappoints.

### 4.6 Originality Detection System

**Mechanisms:**
- **Visual fingerprinting:** Audio and visual signal matching to detect copies
- **Watermark detection:** TikTok/CapCut/other platform watermarks → automatic deprioritization
- **Rolling 30-day evaluation:** If most content in window is reposted → classified as aggregator
- **Aggregator threshold:** 10+ reposts in 30 days → removed from ALL recommendation surfaces
- **Material transformation exemption:** Memes, parodies, green-screen reactions, new voiceovers are NOT penalized
- **Copy replacement:** When copy detected, original version directly replaces it in recommendations
- **Labels:** Copies get labels linking back to original creator

**Impact (December 2025 enforcement):**
- Aggregator accounts: 60-80% reach drops
- Original creators: 40-60% reach increases
- Original Instagram-created Reels: 3-5× reach multiplier vs reposts

---

## 5. User Modeling (How Interest Profiles Are Built)

### 5.1 DV365: The Foundational User Embedding

**Source:** SIGKDD 2025 paper — "Extremely Long User History Modeling at Instagram"

DV365 is Instagram's **core user identity representation** — deployed across 15 production models on Instagram and Threads since November 2023.

**What it encodes:**
- Up to **70,000 interactions** per user (average 40,000)
- Split into two timelines:
  - **Explicit Timeline:** User actions (likes, shares, comments, saves) — max ~35K
  - **Implicit Timeline:** Impression history with dwell time — max ~35K

**Per-interaction record:**
- Media ID, Author ID, Topic ID
- Event timestamp, Action type
- Video duration, User watch time
- Surface type (Reels, Feed, Explore, Stories)
- Media type (photo, video, carousel)

### 5.2 Multi-Slicing and Summarization (MSS)

DV365 converts raw history into features through "multi-slicing":

1. **Action-based slicing:** Separate sequences per action type (liked, commented, shared, saved)
2. **Watch-time slicing:** Filter by dwell time thresholds (watched >15s vs <3s)
3. **Watch-ratio slicing:** Filter by proportion of video watched (debiased formula)
4. **Time-based slicing:** Apply time buckets (3 days, 7 days, full history)
5. **Debiased scoring:** `s(E) = ln(dwell_time + 1) / ln(γ × video_duration + 1)`

Result: **200 derived features**, each a 256-dimensional embedding.

### 5.3 Compression Architecture (Funnel Summarization)

The 200 × 256 tensor is compressed through:
1. Token-wise transposition: [200, 256] → [256, 200]
2. **Funnel Transformer:** Self-attention layers with mean-pooling between blocks (stride > 1)
3. **Linear Compression Encoder (LCE):** Parallel path for additive gain
4. **4-bit quantization:** Final output = 58 embeddings × 256 dimensions, stored as 58×17 long integers

**Compression ratio: 50×** (from 200×256 fp32 → 58×17 int4)

### 5.4 Training Strategy: "Distant Interest Prediction"

Critical design: DV365 **removes the most recent 24 hours** from training input. This forces learning of **long-term stable interests** rather than ephemeral ones.

**Result:** Embedding remains >90% stable (cosine similarity) even when 7 days stale. This enables:
- Pre-computation every 6 hours (not real-time)
- Storage in key-value store indexed by user ID
- Downstream models consume cached embeddings

### 5.5 Downstream Integration

DV365 feeds into all major models:
- **DLRM ranking models:** Concatenated with sparse embeddings
- **HSTU sequence models:** Prepended before latest engagements
- **Two-Tower retrieval models:** Integrated via GateNet (better than linear projection)

**Production impact:** +0.7% Instagram app time spent (cumulative across 15 launches). Ranking models see 0.4%+ normalized entropy improvement.

### 5.6 "Your Algorithm" — Explicit User Intent Declaration (2025-2026)

Instagram rolled out user-facing algorithm controls:
- **Reels** (October 2025): Users declare topic interests
- **Explore** (April 2026): Topic-level preferences
- **Main Feed** (June 2026): Full feed customization

How it works:
- Users see topics the system has tagged to them based on engagement
- Users can add/remove topics
- Adjustments sync across Feed, Reels, and Explore
- LLM describes content clusters in natural language for user understanding
- A single adjustment ripples across all surfaces

**Testing (July 2026):** Instagram is testing real-time conversational prompts, in-stream feedback buttons, and drag-down access to algorithm controls.

### 5.7 Interest Decay and Stability

DV365's time-slicing creates separate signals for:
- **3-day window:** Very recent, possibly ephemeral interests
- **7-day window:** Weekly patterns
- **Full history:** Long-term stable interests

The system distinguishes "what you liked today" from "what you've consistently liked for months." Stable interests persist in recommendations even during temporary browsing of different topics.



---

## 6. Discovery and Explore (Surfacing Content from Unfollowed Accounts)

### 6.1 Candidate Generation: How Instagram Finds "Related" Content

**Source 1: ig2vec (Account Embeddings)**
- Word2vec framework on session-bounded account interaction sequences
- "Words in a sentence" = accounts user interacts with in one session
- Produces dense embeddings for every account
- KNN lookup via FAISS finds accounts similar to user's "seed accounts"
- Retrieve recent media from those similar accounts

**Source 2: Collaborative Filtering**
- "People who liked X also liked Y"
- User-user similarity based on engagement overlap
- Item-item co-engagement matrices

**Source 3: Personalized PageRank on Session Graphs (2024 — Published ECML PKDD)**
- **Graph construction:** Nodes = posts, edges = session-bounded co-interactions (two posts engaged with in same session get an edge)
- **Algorithm:** Personalized PageRank from user's recently engaged posts
- **Result:** Finds contextually related posts that share engagement patterns
- **Captures:** Real-time emerging interests within a browsing session
- **Impact:** Significant increase in user engagement, DAU, and sessions in A/B tests
- **Key insight:** Session-bounded co-interaction captures topical coherence without content analysis

**Source 4: IGQL (Instagram Graph Query Language)**
- Domain-specific language for expressing candidate generation queries
- Enables composing multiple retrieval strategies as subqueries
- Example subqueries:
  - "Accounts similar to liked accounts → their posted media"
  - "Accounts similar to followed accounts → their posted media"
  - "Media liked by users similar to this user"

**Combined output:** Tens of thousands of eligible candidates per user per session.

### 6.2 SilverTorch: The Next-Generation Retrieval System (May 2026)

Meta's latest engineering breakthrough, deployed across Instagram, Facebook, and Threads:

**Paradigm: "Index as Model"**
- All retrieval components unified into a **single PyTorch model**
- Previous microservices (ANN search, eligibility filtering, scoring) become tensor operations within one neural network
- The item index itself becomes a tensor inside the model

**Architecture:**
- ANN search: Fused Int8 kernel (embeddings in 8-bit, GPU-native)
- Eligibility filtering: GPU Bloom index (replaces CPU inverted index)
- Neural reranking: Multi-layer perceptrons or attention within same forward pass
- Multi-task scoring: Predicts multiple engagement actions simultaneously
- All executed in a single forward pass, sharing GPU memory

**Performance:**
- 23.7× higher throughput than traditional multi-service baseline
- 20.9× compute cost efficiency vs CPU-based solution
- Supports 100s of thousands top-k candidates (vs 2,048 with FAISS-GPU)
- Sub-100ms latency for the entire retrieval pipeline
- Same-day posts now represent significant portion of recommendations

**Impact on recommendations:**
- Wider candidate funnel (1-2 orders of magnitude more candidates evaluated)
- More sophisticated scoring earlier in pipeline
- Better content freshness (streaming index updates without redeployment)
- Neural reranking practical within latency budget

**Key insight for Feedmine:** SilverTorch shows that unifying retrieval into a single model pass is more efficient than chaining services. A local system can similarly benefit from a single embedding → filter → rank pipeline rather than separate stages.

### 6.3 The Three-Stage Ranking Cascade

After candidate generation, Explore uses progressive filtering:

| Stage | Purpose | Complexity | Candidates |
|-------|---------|------------|------------|
| Distillation model | Fast pre-screen | Super-lightweight | 500 → 150 |
| Lightweight NN | Full dense features | Medium | 150 → 50 |
| Deep MTML model | Full dense + sparse | Heavy | 50 → 25 |

The distillation model is trained to mimic stages 2+3 using NDCG ranking loss. This enables evaluating more candidates without proportional compute cost.

### 6.4 The Tiered Distribution Funnel

All content (from any creator size) goes through progressive distribution:
1. Small initial audience predicted to enjoy it
2. As engagement arrives → slightly wider audience
3. Best performers → even wider
4. Pattern continues — viral content breaks through multiple tiers

**Critical:** Small creators get the same initial test pool as large ones. Distribution is earned by content performance, not account size.

### 6.5 Connected vs Unconnected Distribution

Instagram explicitly models two paths:
- **Connected reach:** Shown to followers (likes per reach slightly more important)
- **Unconnected reach:** Shown to non-followers (sends per reach slightly more important)

Same three core signals for both, slightly different weighting.

---

## 7. Temporal and Session Intelligence

### 7.1 Freshness and Decay

**In Feed:**
- Newer posts ranked higher (freshness is a core signal)
- System gathers "recent posts" from followed accounts as starting inventory
- Stale content progressively deprioritized

**In Explore/Reels:**
- Less about absolute freshness, more about **engagement velocity**
- Content can be days old and still get recommended if engagement curve remains strong
- SilverTorch's streaming updates mean same-day content now appears in recommendations

**In DV365:**
- Time-sliced features: 3-day, 7-day, full history windows
- Creates separate interest signals at different time horizons
- "What you liked today" vs "what you've liked for months"

### 7.2 Engagement Velocity: The First-Hour Window

Instagram evaluates new content in a **progressive evaluation window:**

1. **Detection:** ~60 seconds post-publication, platform begins evaluation
2. **First 30-60 minutes:** Early engagement velocity is diagnostic
3. **Relative evaluation:** Performance measured against account size and historical baseline
   - 400 likes + 200 saves from 5K-follower account = very strong
   - Same numbers from 500K-follower account = weak
4. **Gradual accumulation preferred:** Organic curves > simultaneous spikes (spikes = anomaly)
5. **Account-level reliability:** Consistent early engagement across multiple posts → persistent distribution allocation increase

### 7.3 Session Context

Instagram adapts to session type:
- **Morning browse:** Potentially different content mix (quick catch-up)
- **Evening scroll:** Longer, more exploratory content
- **Active hours:** Content posted when audience is active gets faster initial engagement → better velocity
- **Session depth:** As user scrolls deeper, content type mix may shift

**Session-based Personalized PageRank (2024):**
- Captures emerging interests WITHIN a single session
- If user engages with posts A, B, C in one session, the system finds posts similar to {A, B, C} as a cluster
- Real-time adaptation without waiting for embedding recomputation

### 7.4 Posting Cadence Signals

- Accounts with regular posting patterns get more predictable distribution
- Consistency over isolation: repeated early engagement → account-level signal adjustments
- Posting when audience is active → faster velocity → better distribution outcome

### 7.5 User Activity Pattern Modeling

- Time of day / day of week factored into predictions
- User's historical active hours influence when content surfaces
- The system knows when you typically browse and prioritizes fresh content for those windows



---

## 8. Anti-Staleness and Diversity

### 8.1 The Content Fatigue Problem

Meta's diversity-aware ranking framework (September 2025) addresses a core problem: models optimized for click-through rate lead to repetitive, spammy-feeling content that degrades user satisfaction even while short-term metrics look good.

**The diagnosis:** High CTR ≠ high satisfaction. Users click on similar content repeatedly but report lower satisfaction when feeds feel repetitive.

### 8.2 Diversity-Aware Ranking Framework

**Mechanism: Multiplicative Demotion Penalties**
- Operates as a layer on TOP of existing engagement prediction models
- Does not replace ranking — modifies it
- Applies penalty multipliers (0 to 1) to candidates too similar to recently shown content

**Dimensions tracked:**
1. Same content creator (don't show 5 posts from one account in a row)
2. Same topic cluster (don't show only cooking content)
3. Same content type/format (mix Reels, photos, carousels)
4. Same notification category (for notification ranking)
5. Same product surface origin

**How it works:**
- Binary similarity indicators determine threshold violations per dimension
- Demotion multiplier ranges 0 to 1, reducing the relevance score
- Penalty increases progressively as more similar items appear in a batch
- Configurable weights per dimension (team can tune)

**Approach:** Maximal Marginal Relevance (MMR) — a standard information retrieval technique adapted for social recommendation. At each position in the ranked list, the next item is selected to maximize a combination of relevance and diversity from already-selected items.

### 8.3 Intra-Batch Diversity Rules (Explore-specific)

- Down-rank posts from same author with increasing penalty factor
- Down-rank posts from same seed account (topic deduplication)
- Penalty increases progressively down the ranked batch
- No single creator dominates a page of results

### 8.4 Feed Composition Control

Instagram's Feed system enforces:
- Balanced mix of content types (not all Reels, not all photos)
- Mix of connected (followed) and recommended (unconnected) content
- No single creator dominates
- Topic variety across visible positions

### 8.5 Content Ineligibility for Recommendations

Content that is **allowed** on the platform but **not recommended:**
- Self-harm, violence, sexually suggestive content
- Tobacco/vaping, adult products, pharmaceutical claims
- Cosmetic procedures, misleading health/finance claims
- Political content (filtered from recommendations since 2024)
- Clickbait and engagement bait
- Unoriginal/repurposed content without material value
- Misinformation (fact-checker flagged)
- Low-quality content (watermarked, bordered, low-resolution, majority text overlay)

### 8.6 Future Directions (Meta's Published Roadmap)

- **Dynamic demotion strategies:** Adapting penalties to context (timing, frequency, user state)
- **LLMs for semantic similarity:** Using language models to measure how "similar" two pieces of content really are (beyond topic labels)
- **Context-adaptive fatigue:** Different diversity requirements for different session types

---

## 9. What Can Be Done Locally (No Server Required)

### 9.1 Content Understanding — Fully Local

**Visual embeddings:**
- **CLIP (OpenAI, various sizes):** Map images → vectors that capture semantic meaning. Available in sizes from ViT-B/32 (149M params, runs on any GPU) to ViT-L/14 (428M params). Can classify images by comparing to text descriptions without training.
- **SigLIP (Google):** Improved CLIP variant, better zero-shot classification, available in compact sizes.
- **MobileNet V3:** Lightweight CNN for on-device image classification. Under 10MB model size.
- **LLaVA-OneVision-0.5B:** Full vision-language model in 829MB that runs offline via LiteRT. Can describe images, answer visual questions, understand content semantically.
- **DINOv2 (Meta):** Self-supervised visual features, excellent for similarity and clustering. ViT-S variant runs on CPU.

**Text understanding:**
- Local NLP on RSS descriptions, titles, alt text
- Keyword extraction, topic classification
- Sentence transformers (all-MiniLM-L6-v2: 80MB, runs on CPU)
- Zero-shot classification via small language models

**OCR (replicating Rosetta locally):**
- Tesseract OCR for text-in-image extraction
- PaddleOCR for multi-language support
- Run on images in feed to extract overlay text

### 9.2 User Modeling — Fully Local

**Replicating DV365's approach at small scale:**
- Track all interactions: views (with dwell time), likes, saves, shares, clicks
- Maintain per-feed-source engagement history
- Apply time-slicing: 3-day, 7-day, 30-day, all-time windows
- Compute per-source average dwell time, engagement rate
- Build user embedding from engagement-weighted content embeddings

**The DV365 insight that transfers:**
- Separate explicit actions (likes, saves) from implicit signals (dwell time)
- Time-slice both timelines separately
- Use debiased watch-time scoring for video content
- Pre-compute user embedding periodically (every few hours), not real-time
- 24-hour exclusion window for training (learn stable interests, not ephemeral)

**Session tracking:**
- Track which items are viewed together in one session
- Co-viewed items create "session context" (like ig2vec sessions)
- Build source-source similarity from session co-occurrence

### 9.3 Ranking — Fully Local

**Value model (lightweight version):**
```
Score = W_save × P(save) + W_dwell × normalized_dwell_time
      + W_click × P(click_through) + W_fresh × freshness_decay
      - W_neg × P(skip) + W_diversity × diversity_bonus
```

All probabilities can be estimated from local history:
- P(save) = historical save rate for this source × content similarity to previously saved items
- P(skip) = historical skip rate for similar content
- Freshness decay = exponential decay from publication time

**Three-stage cascade (local version):**
1. **Filter:** Remove muted sources, seen items, content older than threshold
2. **Light rank:** Score all candidates with simple features (source affinity × freshness × type preference)
3. **Full rank:** Top N candidates scored with content embeddings, diversity penalties, user model

### 9.4 Discovery — Locally Achievable

**Replicating ig2vec without a social graph:**
- Build source-source embeddings from session co-engagement
- "Sources engaged in same session are similar" (same training signal as ig2vec)
- Find sources similar to user's high-engagement sources
- Recommend content from similar-but-unfollowed sources

**Replicating Personalized PageRank:**
- Build item-item graph from session co-viewing
- When user engages with item X in current session, walk the graph to find related items
- Lightweight graph traversal, works with local data only

**Content-based cold start:**
- New items (no engagement history) scored by content embedding similarity to user's liked items
- CLIP embedding of new image vs average CLIP embedding of user's saved/liked items
- Solves cold-start without collaborative filtering

### 9.5 Diversity — Fully Local

**Maximal Marginal Relevance (MMR):**
```
score(item) = λ × relevance(item) - (1-λ) × max_similarity(item, already_shown)
```

- For each position in feed, select item that maximizes score
- `relevance` = the raw ranking score
- `max_similarity` = highest cosine similarity between item's embedding and any already-displayed item's embedding
- λ controls exploration vs exploitation (typically 0.5-0.7)

**Additional diversity rules (directly from Instagram's approach):**
- Max N items per source per page
- No more than M consecutive items of same content type
- Topic cluster tracking: penalize over-representation

### 9.6 What CANNOT Be Done Locally

| Capability | Why it needs a server | Workaround |
|------------|----------------------|------------|
| Collaborative filtering | Requires cross-user data | Content-based similarity instead |
| Engagement velocity | Requires many users viewing same content | Use source historical averages |
| Viral content detection | Requires platform-wide engagement signals | Follow curated "trending" feeds |
| Real-time content moderation | Requires global policy enforcement | Trust feed sources |
| Social proof (friend activity) | Requires social graph | Not replicable |
| A/B testing at scale | Requires user population | Local bandit optimization instead |



---

## 10. What Feedmine Can Steal from Instagram

### 10.1 The Multi-Slicing User Embedding (From DV365)

**Technique:** Don't model the user as a single preference vector. Instead, slice their history into multiple views:
- By action type: saved items ≠ liked items ≠ items you spent time on
- By recency: 3-day interests ≠ 30-day interests ≠ all-time interests
- By dwell threshold: items you spent >30s on ≠ items you scrolled past in 2s
- By content type: video preferences ≠ image preferences

Each slice produces a separate embedding. Combine them with a lightweight attention mechanism or simple weighted concatenation. This captures the nuance that "what you save" represents different intent than "what you like" — and both matter differently.

**Implementation:** For each slice, compute the mean embedding of the content items in that slice. Stack the slice embeddings into a matrix. Optionally compress with a small MLP or just concatenate.

### 10.2 Session-Based Contextual Recommendation (From Personalized PageRank)

**Technique:** Within a single browsing session, build a real-time context vector:
1. Track items viewed this session
2. Compute average embedding of engaged-with items this session
3. Use that session embedding to boost similar unread items
4. Build an item-item co-view graph over time; use graph walk from current session items to find related candidates

**Why this matters:** The user's immediate context (what they're browsing RIGHT NOW) often differs from their long-term profile. Morning sessions might be news-focused; evening might be entertainment. Session context captures this without explicit modeling.

### 10.3 Debiased Watch-Time Scoring (From DV365/Reels)

**Technique:** Raw dwell time is misleading — longer content naturally gets more time. Use:
```
score = ln(dwell_time + 1) / ln(γ × content_length + 1)
```
Where `content_length` could be:
- Article word count (for text)
- Video duration (for video)
- Image count (for galleries/carousels)
- Estimated reading time

This normalizes a 2-minute read of a 3-minute article as stronger engagement than a 2-minute read of a 20-minute article.

### 10.4 Distant Interest Prediction (From DV365 Training)

**Technique:** When building user models, exclude the most recent N hours from training data. Force the model to predict interests from stable historical patterns, not today's ephemeral browsing.

**Practical benefit:** The user embedding becomes more stable and robust. Prevents the "one weird click" problem where a single accidental engagement with a topic (e.g., clicking a celebrity gossip link) doesn't immediately flood the feed.

**Feedmine implementation:** When updating user embeddings, use a 6-24 hour delay on the most recent interactions. This creates a "cooling off" period that naturally filters noise.

### 10.5 Content Freshness as Exponential Decay

**Technique:** Instagram treats freshness as a core signal with decay:
```
freshness_score = exp(-λ × hours_since_publication)
```
Where λ controls decay rate. For Feed (following): fast decay (hours matter). For Explore/discovery: slower decay (days are okay if engagement is strong).

**Feedmine twist:** Allow different decay rates per content type. News: fast decay (hours). Tutorials: slow decay (weeks). Research papers: very slow decay (months).

### 10.6 Originality Scoring (Adapted for Feed Sources)

**Technique:** Instagram penalizes aggregators who repost others' content. Feedmine can adapt:
- Track content fingerprints (title hash, first-paragraph hash, image perceptual hash)
- Detect when multiple sources publish the same content
- Prefer the original source (earliest publication time)
- Downrank or deduplicate aggregator sources that primarily republish

### 10.7 Multi-Task Value Model (From Explore Ranking)

**Technique:** Don't optimize for a single metric. Predict multiple actions and combine:
```
Score = W₁ × P(save) + W₂ × P(long_dwell) + W₃ × P(click_through)
      + W₄ × P(return_to) - W₅ × P(mark_as_read_quickly)
```

The negative term is critical — it models the Instagram "SFPLT" signal. Items that users quickly dismiss get penalized even if they have high click-through rates.

### 10.8 Progressive Diversity Penalties (From Feed Composition Control)

**Technique:** As the user scrolls, apply increasing penalties for same-source or same-topic items:
- Position 1-5: No penalty
- Position 6-10: 20% penalty for same source appearing again
- Position 11-20: 40% penalty for same source
- Same topic (by embedding similarity threshold): Progressive penalty prevents filter bubbles

### 10.9 The "Trial Content" Concept (From Trial Reels)

**Technique:** When the user subscribes to a new source, don't immediately give it full feed real estate. Instead:
- Show 1-2 items from the new source in discovery positions
- Measure engagement (dwell, save, click)
- If engagement exceeds threshold → promote to regular feed placement
- If low engagement → keep in low-priority discovery positions

This prevents "subscription regret" where following a new source floods the feed with unwanted content.

### 10.10 Source-Level Embeddings (From ig2vec)

**Technique:** Build feed-source embeddings from session co-engagement:
- Track which sources the user engages with in the same session
- Train word2vec (or simpler PMI-based) embeddings on session sequences
- Cosine similarity between source embeddings = topical similarity
- Use for: "more like this source" recommendations, source clustering, diversity enforcement

---

## 11. Key Differences from Facebook (Why Instagram's Approach Matters Separately)

### 11.1 Architectural Philosophy

| Aspect | Instagram | Facebook | Implication for Feedmine |
|--------|-----------|----------|--------------------------|
| Primary graph | Interest graph | Social graph | Instagram's approach works without friends |
| Discovery % | ~50% of app time on unconnected content | ~20% unconnected | Instagram's discovery mechanisms are more developed |
| Visual-first | Images/video are primary signal | Text/links are primary | Instagram's vision pipeline is more relevant for visual feeds |
| Creator vs community | Individual creators | Groups and Pages | Instagram models individual-to-individual relationships |
| Signal hierarchy | Shares > saves > comments > likes | Comments > shares > reactions | Instagram values "I want to send this to someone" highest |
| Content format | Unified "Views" metric | No unified metric | Instagram has cleaner cross-format comparison |

### 11.2 Technical Differences

- **DV365** serves both Instagram AND Threads — Facebook uses different user modeling
- **ig2vec** produces account-level embeddings — Facebook uses community detection
- **IGQL** is Instagram's domain-specific candidate generation language — Facebook uses different infrastructure
- **Reels ranking** surveys users on "entertaining" and "funny" — unique to Instagram's entertainment optimization
- **Personalized PageRank on session graphs** — Instagram-specific published research
- **Political content filtering** — Instagram removes from ALL recommendations; Facebook is more permissive
- **"Your Algorithm" user controls** — Instagram-only feature for explicit interest declaration

### 11.3 Why Instagram's Approach Is More Relevant for Feedmine

1. **No social graph required:** Instagram's core discovery (Explore, Reels) works on interest similarity, not friend connections. Feedmine has no social graph, so Instagram's interest-graph approach is directly applicable.

2. **Visual-first content understanding:** If Feedmine handles visual feeds (image blogs, photo feeds, design inspiration), Instagram's vision pipeline approach is the template.

3. **Creator-to-viewer relationship:** Instagram models "how much does this viewer like this creator's content" — directly analogous to "how much does this user like this feed source."

4. **Discovery dominance:** Instagram dedicates more of its system to "showing you things you haven't asked for but will like" — exactly what Feedmine's discovery feature needs.

5. **Session-based adaptation:** Instagram's session PageRank approach captures within-session context shifts — valuable for a feed reader where users browse different topics at different times.

---

## 12. Open Questions

### 12.1 Architecture Questions (Could Inform Feedmine Design)

1. **HSTU vs simpler models:** Meta's Generative Recommender with HSTU architecture (1.5 trillion parameters, 12.4% improvement in A/B tests) suggests the scaling law applies to recommendations. But at what scale does it start to matter? Can a small HSTU-like model (few million params) beat hand-tuned heuristics for a single user?

2. **Multi-modal fusion details:** How exactly are visual, text, and audio embeddings combined into a single content representation at Instagram? Is it concatenation → MLP? Cross-attention? Gated fusion? The optimal architecture for local multi-modal content understanding is unclear.

3. **Value model weight ratios:** Instagram doesn't publish the relative W values in their scoring formula. What's the relative weight of save vs like vs share? Internal reports suggest saves and shares are 3-5× more valuable than likes, but exact ratios would be useful for calibrating a local model.

4. **Cross-surface signal transfer:** A like on Reels vs a like on Feed for the same creator — how much does surface context matter? DV365 includes surface type as a feature, but the exact transfer mechanism is unclear.

### 12.2 Content Understanding Questions

5. **Topic taxonomy granularity:** ig2vec captures similarity, but how many effective clusters exist? At what embedding distance threshold does "similar topic" become "different topic"? This affects diversity enforcement.

6. **Quality scoring internals:** Instagram clearly has a "quality" signal separate from engagement, used to filter "low-quality" content from recommendations. What features define quality? Resolution, composition, text-to-image ratio, originality — but what thresholds?

7. **Content lifecycle prediction:** Can content "freshness" value be predicted at upload time based on content type? (News expires fast, tutorials stay relevant — can this be predicted automatically?)

### 12.3 System Design Questions

8. **Optimal embedding dimensionality for local use:** Instagram uses 256-dimensional embeddings. Is that overkill for a single-user system with ~1000 feed sources? Could 64 or 128 dimensions suffice?

9. **Update frequency tradeoffs:** DV365 recomputes every 6 hours and stays >90% stable for 7 days. For a local feed reader, what's the optimal re-embedding frequency? Daily? Weekly?

10. **Cold-start strategy:** When Feedmine starts with zero history, what's the fastest path to useful recommendations? Instagram uses account-level embeddings from session behavior — but with one user and zero sessions, content-based features must dominate initially.

### 12.4 A/B Testing Insights (What Instagram Has Learned)

From public information about Instagram's experiments:

- **Showing fewer friend bubbles worked better than showing more** (quality > quantity in social proof)
- **Completion rate given 300% more weight** in 2026 ranking update → favoring genuinely watched content over clickbait
- **"Your Algorithm" user controls** improve satisfaction even though most users don't change defaults — the perception of control matters
- **Relationship signals weighted 60% vs engagement metrics 40%** (2026 shift) → for connected content, who you are matters more than what you did
- **Views unified metric** — treating all formats equally actually improved cross-format fairness
- **Same-day content freshness** improved with SilverTorch's streaming updates → users respond positively to fresher content even in discovery
- **Session-based PageRank** showed significant increases in DAU and session count when deployed — capturing real-time context matters more than better long-term modeling

### 12.5 Unresolved Technical Gaps

11. **Real-time vs pre-computed balance:** What percentage of Instagram's recommendations come from pre-generated offline sources vs real-time retrieval? This informs whether Feedmine should run ranking on every app open or pre-compute feed order.

12. **Shopping intent detection:** Instagram clearly ranks product content differently from entertainment content. The signals for "this user is in shopping mode" vs "browsing mode" are not public.

13. **Age-appropriate content filtering:** Instagram applies different recommendation rules for teen accounts. The mechanisms are relevant if Feedmine ever needs user-specific content policies.

14. **Dynamic demotion strategies:** Meta's "future work" mentions context-adaptive fatigue penalties — adjusting diversity requirements based on time, frequency, and user state. This represents an unsolved problem even at Meta's scale.

---

## Appendix A: Key Papers and Sources

| Paper/Source | Year | Key Contribution |
|-------------|------|------------------|
| "Extremely Long User History Modeling at Instagram" (DV365) | 2025 | User embedding architecture, multi-slicing, funnel compression |
| "Trillion-Parameter Sequential Transducers for Generative Recommendations" (HSTU) | 2024 | Core ranking architecture, scaling laws for recommendations |
| "Leveraging Session Signals with Personalized PageRank for Instagram Explore" | 2024 | Session-based candidate generation |
| "SilverTorch: Index as Model" | 2026 | Unified retrieval system, GPU-native pipeline |
| "Scaling Generative Recommendations with Context Parallelism" (ULTRA-HSTU) | 2025 | 5× training scaling, 21× inference scaling |
| Instagram System Cards (Meta Transparency Center) | 2024 | Official ranking documentation |
| Adam Mosseri public communications | 2025-2026 | Signal hierarchy, format guidance, originality rules |
| Meta Diversity-Aware Ranking Framework | 2025 | Anti-repetition, MMR adaptation |
| Rosetta (OCR system) | 2018 | Text-in-image understanding at scale |
| I-JEPA / V-JEPA / ImageBind | 2023-2024 | Visual embedding research |

## Appendix B: Glossary

| Term | Definition |
|------|-----------|
| **DV365** | Instagram's foundational user embedding (365 = daily, V = vector) |
| **HSTU** | Hierarchical Sequential Transduction Unit — Meta's core ranking architecture |
| **ig2vec** | Instagram's account embedding system (word2vec on session sequences) |
| **IGQL** | Instagram Graph Query Language for candidate generation |
| **MTML** | Multi-Task Multi-Label model (predicts multiple engagement actions) |
| **SFPLT** | "See Fewer Posts Like This" — explicit negative feedback signal |
| **MMR** | Maximal Marginal Relevance — diversity-preserving reranking technique |
| **FAISS** | Facebook AI Similarity Search — KNN lookup library |
| **SilverTorch** | Meta's unified retrieval system (2026), "Index as Model" paradigm |
| **GR** | Generative Recommender — Meta's new recommendation paradigm replacing DLRMs |
| **DLRM** | Deep Learning Recommendation Model — Meta's previous-generation architecture |
| **NE** | Normalized Entropy — Meta's internal model quality metric |
| **M-FALCON** | Microbatched-Fast Attention Leveraging Cacheable Operations — inference optimization |
| **Trial Reels** | Feature to test content with non-followers before full release |
| **Connected/Unconnected reach** | Distribution to followers vs non-followers |
| **Engagement velocity** | Rate of engagement in first 30-60 minutes post-publication |

---

*End of Loop 02: Instagram Intelligence*
*Next: Loop 03 — Synthesis across all player research*
