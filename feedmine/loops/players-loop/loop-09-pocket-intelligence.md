# Pocket Intelligence: Complete Knowledge Base for Save-Based Feed Intelligence

> **Purpose**: Comprehensive reference for building a local-only feed reader (Feedmine) that replicates Pocket's save-based intelligence. Distinguishes confirmed facts (from source code, documentation, telemetry specs) from inferences.
>
> **Sources**: Pocket/recommendation-api GitHub repository (Apache 2.0), Mozilla Bugzilla tickets, Firefox Activity Stream telemetry documentation, Pocket for Publishers FAQ, Mozilla Metric Hub, mozilla/readability GitHub repository, Quality News (social-protocols) research.
>
> **Last updated**: July 2026

---

## 1. Executive Summary

### What Pocket Is

Pocket is a read-it-later service acquired by Mozilla in 2017, integrated into Firefox as the recommendation engine powering the New Tab page. It reaches 200M+ Firefox users daily. Unlike algorithmic recommendation engines (Google Discover, Apple News), Pocket operates an **"algotorial" model**: human editors select ~50 articles per day per locale, and algorithms re-rank that curated set based on engagement signals.

### How Saves Create Unique Intelligence

The "save" action — clicking a button to store an article for later reading — is Pocket's foundational signal. Unlike clicks (impulsive, low-commitment) or likes (social performance), a save represents **deliberate future intent**. The user is saying: "This is worth my future time." This signal is:

- **High-friction**: Requires explicit action beyond passive consumption
- **Future-oriented**: Signals perceived long-term value, not momentary interest
- **Non-performative**: Private by default (no social signaling motive)
- **Compositional**: A user's save library builds a rich interest profile over time

Pocket uses aggregate save velocity to identify content worth curating, save counts as quality thresholds in vector search (≥1,000 saves minimum for fallback recommendations), and individual save history to build user profiles for the Discover feed.

### The Mozilla/Firefox Relationship

- **2017**: Mozilla acquires Pocket (then Read It Later, Inc.)
- **Integration**: Pocket recommendations appear on Firefox New Tab for all users
- **Privacy commitment**: All personalization runs on-device; Mozilla cannot see browsing history
- **Revenue**: Sponsored content (SPOCs) in position 3 of New Tab generates revenue; organic recommendations are ad-free
- **Scale**: ~200M Firefox desktop users see Pocket recommendations daily; Pocket Hits newsletter reaches up to 7M users/day
- **Open source**: The recommendation-api, curated-corpus-api, content-monorepo, curation-admin-tools, and mozilla/readability are all publicly available on GitHub

---

## 2. The Recommendation Pipeline

### Complete Architecture (Confirmed from Source Code)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        CONTENT INGESTION                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  [Web Articles] → [Readability Parser] → [Text + Metadata Extraction]   │
│                                                                          │
│  [Aggregate Save Signals] → [Editorial Team Dashboard]                   │
│                                                                          │
│  [Editorial Team] → [Curated Corpus API] → [Scheduled Surfaces]         │
│       ↑                                                                  │
│  (curation-admin-tools)                                                  │
│                                                                          │
├─────────────────────────────────────────────────────────────────────────┤
│                        CANDIDATE SELECTION                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Curated Corpus API serves scheduled items per surface:                  │
│    - NEW_TAB_EN_US, NEW_TAB_DE_DE, etc.                                 │
│    - Each item: corpus_item_id, topic, publisher, url, scheduled_date    │
│    - ~50 items per locale per day                                        │
│                                                                          │
├─────────────────────────────────────────────────────────────────────────┤
│                        RANKING (Server-Side)                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. Thompson Sampling (Beta-Binomial, engagement-weighted)               │
│  2. Publisher Spread (no same-publisher within distance=6)               │
│  3. Recency Sort (most recently scheduled first)                         │
│  4. Syndicated Boost (new Pocket-hosted article → slot 1, <3M impr.)    │
│  5. Regional Weighting (country-specific engagement at 0.95 weight)      │
│                                                                          │
│  Engagement data from: AWS SageMaker Feature Store                       │
│  Metrics: opens/impressions over 1-day, 7-day, 14-day, 21-day, 28-day   │
│                                                                          │
├─────────────────────────────────────────────────────────────────────────┤
│                        PERSONALIZATION (Client-Side, On-Device)           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  V1 (2017-2019): Domain Affinity                                         │
│    - Server sends related domains with each story                        │
│    - Firefox compares against local browsing history                      │
│    - Matching stories promoted in local ranking                          │
│                                                                          │
│  V2 (2019+): Interest Vector + Recipe Executor                           │
│    1. Download personalization "recipe" from server                       │
│    2. Run taggers on local browsing history                              │
│    3. Build interest vector from tagged history                           │
│    4. Score each story against interest vector                            │
│    5. Re-rank locally                                                    │
│                                                                          │
│  ALL PROCESSING ON-DEVICE. Nothing sent back.                            │
│                                                                          │
├─────────────────────────────────────────────────────────────────────────┤
│                        DISPLAY                                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Firefox New Tab: Story cards with title, excerpt, thumbnail, publisher  │
│  Pocket App: Discover feed, For You slate, Topic slates                  │
│  Newsletter: Pocket Hits (7-10 articles daily, up to 7M recipients)      │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Key Technology Stack (Confirmed)

| Component | Technology |
|-----------|-----------|
| Recommendation API | Python, FastAPI, GraphQL |
| Vector Search | Qdrant |
| Engagement Store | AWS SageMaker Feature Store |
| Topic Storage | DynamoDB |
| Backend Monorepo | TypeScript, NestJS |
| Content Parser | mozilla/readability (JavaScript) |
| Curation Tools | Pocket/curation-admin-tools (TypeScript) |
| Corpus API | Pocket/curated-corpus-api (TypeScript) |

### The Item-to-Item Recommender

For "related articles" within the Pocket ecosystem, a vector similarity system uses Qdrant:
- Two collections: `_recs` (curated corpus) and `_all` (all items for lookup)
- Filters: `is_curated`, `is_syndicated`, `domain`, `save_count`, `freshness_days` (90 days)
- Minimum save count for fallback: **1,000 saves**
- English-only for vector recommendations
- Embedding model: **Unknown** — the recommendation-api consumes embeddings but doesn't generate them

---

## 3. Signal Taxonomy

### 3.1 Save Signals

| Signal | How Pocket Uses It | Feedmine Equivalent |
|--------|-------------------|-------------------|
| **Save action** | Primary quality indicator; aggregate save velocity drives editorial discovery | Star/bookmark action on feed items |
| **Save count (aggregate)** | Quality threshold in Qdrant (≥1,000 for fallback recs); metadata on vector embeddings | Local popularity score across all items |
| **Save velocity** | Editors monitor which articles are getting saved rapidly to identify trending content | Rate of stars over time window |
| **Save-to-view ratio** | Not explicitly in code, but implied as editorial quality signal | Stars / views ratio per feed |
| **Save co-occurrence** | Users who saved X also saved Y (likely in defunct `feed-machine` service) | Local co-occurrence matrix from user's library |

### 3.2 Engagement Signals (CTR-Based)

| Signal | How Pocket Uses It | Feedmine Equivalent |
|--------|-------------------|-------------------|
| **Opens (clicks)** | Primary metric for Thompson Sampling on New Tab; tracked per item/surface/country | Click-throughs to full article |
| **Impressions** | Denominator for CTR calculation; tracked in trailing windows | Times item appeared in feed view |
| **Opens/Impressions ratio** | The CTR that Thompson Sampling optimizes | Local CTR per feed/topic |
| **Trailing windows** | 1-day, 7-day, 14-day, 21-day, 28-day engagement windows | Configurable decay windows |

### 3.3 Dwell/Completion Signals

| Signal | Status in Pocket | Feedmine Equivalent |
|--------|-----------------|-------------------|
| **Time on page** | Collected in Context Graph research (opt-in only); NOT in recommendation-api | Reading time tracking (local) |
| **Read completion** | Not confirmed in open-source code; likely tracked in separate analytics | Scroll depth / article completion |
| **Archive action** | User marks article as "read" in Pocket; **inference**: feeds back as completion signal | Mark-as-read action |
| **Word count / reading time** | Displayed in Pocket UI; likely used for content categorization | Computed locally from extracted text |

### 3.4 Negative Signals

| Signal | How Pocket Uses It | Feedmine Equivalent |
|--------|-------------------|-------------------|
| **Dismiss/block** | User can dismiss individual stories; blocked stories never shown again | Hide/dismiss action |
| **Block publisher** | User blocks entire domain from recommendations | Mute feed source |
| **SPOC block** | User blocks specific sponsored content | N/A (no ads) |
| **Low CTR (implicit)** | Thompson Sampling naturally demotes items with low open rates | Decay score for unclicked items |
| **Blocklist** | `app/resources/blocklists.json` — permanently blocked item IDs | User-maintained blocklist |

### 3.5 Preference Signals

| Signal | How Pocket Uses It | Feedmine Equivalent |
|--------|-------------------|-------------------|
| **Explicit topic preferences** | User selects preferred topics; boosted in For You slate | Tag/topic preferences |
| **Browsing history (local)** | V2 interest vector built from tagged history | Reading history analysis |
| **Domain affinity (local)** | V1 system matched stories to frequently-visited domains | Feed source affinity scores |

### Critical Distinction: Saves vs. CTR in Pocket's System

This is a crucial architectural insight:
- **New Tab recommendations** are ranked by **CTR** (clicks/impressions) via Thompson Sampling
- **Editorial selection** of what enters the candidate pool is informed by **save signals**
- **Discover/For You** feeds incorporate **save history** for personalization
- **Vector recommendations** use **save_count** as a quality threshold

The save signal operates at a **different layer** than the ranking signal. Saves inform the *candidate set*; CTR ranks within that set.


---

## 4. Content Understanding

### 4.1 Article Extraction: Mozilla Readability

Pocket uses **mozilla/readability** (open source, GitHub, 4.9k stars) to extract article content from web pages. This is the same library that powers Firefox Reader View.

**The Algorithm (confirmed from source code analysis):**

1. **Preprocess**: Remove scripts, styles, hidden elements, "unlikely candidates" (sidebars, ads, share widgets identified by class/id regex patterns)
2. **Score DOM nodes**: Assign scores based on:
   - Tag type: `DIV` (+5), `PRE`/`TD`/`BLOCKQUOTE` (+3), lists/forms (-3), headings (-5)
   - Class/ID weight: Positive patterns (`article|body|content|entry|main|post|text|blog|story`) get bonus; negative patterns (`-ad-|hidden|banner|comment|footer|sidebar|sponsor|widget`) get penalty
   - Text content: Score = 1 + comma_count + min(3, floor(text_length/100))
   - Link density: ratio of link text to total text (high density = navigation, penalized)
3. **Propagate scores**: Parent node gets full score, grandparent gets half
4. **Select top candidate**: Best-scoring container node
5. **Clean**: Remove remaining junk from selected container
6. **Retry with relaxed flags** if result too short (< 500 characters)

**Output (the Article object):**
- `title`: Cleaned article title
- `content`: Sanitized HTML of main content
- `textContent`: Plain text (no HTML)
- `length`: Character count
- `excerpt`: Short description/summary
- `byline`: Author information
- `siteName`: Publication name
- `lang`: Content language
- `publishedTime`: Publication date
- `dir`: Text direction (LTR/RTL)

**Key Design Properties:**
- Environment-agnostic (works in browser or Node.js via JSDOMParser)
- No network requests — operates on already-fetched HTML
- Configurable thresholds (`charThreshold`, `nbTopCandidates`, `linkDensityModifier`)
- `isProbablyReaderable()` — fast heuristic to check if a page is worth parsing at all
- Handles international content (RTL, CJK character counting)

### 4.2 Topic Classification

**The 16 Official Pocket Topics:**
```
SELF_IMPROVEMENT, POLITICS, BUSINESS, HEALTH_FITNESS,
SCIENCE, PARENTING, EDUCATION, CORONAVIRUS,
TECHNOLOGY, FOOD, ENTERTAINMENT, PERSONAL_FINANCE,
CAREER, TRAVEL, SPORTS, GAMING
```

**How topics are assigned** (partially confirmed, partially inferred):
- Topics are "server-defined" and "attached to the story" (Bugzilla #1666630)
- Each corpus item has exactly ONE topic field
- The recommendation-api does NOT contain a classifier — topics arrive pre-assigned from the Curated Corpus API
- **Inference**: Topics are either manually assigned by editors during curation, or assigned by an upstream classifier not in the open-source codebase

**What we DON'T know:**
- Whether an NLP classifier assists editors (likely, given scale)
- What model generates the vector embeddings stored in Qdrant
- Whether topic assignment uses the extracted text from Readability or some other signal

### 4.3 Content Scoring (Quality Assessment)

Pocket does NOT have a single "content quality score." Instead, quality is enforced through multiple gates:

1. **Human Editorial Gate**: Content must be explicitly scheduled by editors to enter the recommendation pipeline. This IS the primary quality filter.
2. **Publisher Track Record**: "Stories are sourced from a wide and diverse assortment of publications with a track record of trustworthy and accurate coverage" (Pocket for Publishers FAQ)
3. **Engagement Validation**: Thompson Sampling naturally demotes low-CTR items — items that don't get clicked develop poor posteriors
4. **Blocklisting**: Permanently blocked item IDs in `blocklists.json`
5. **is_curated flag**: Vector recommender explicitly filters `is_curated=True`

**For Feedmine**: Quality scoring must be computed locally. Pocket delegates this to human editors — Feedmine must approximate it algorithmically.

---

## 5. User Modeling from Saves

### 5.1 How Sparse Intentional Signals Build Rich Profiles

A user's Pocket library — their collection of saved articles — represents one of the richest interest profiles available in any system. Unlike:
- **Browsing history** (noisy, includes accidental clicks, work browsing, passive consumption)
- **Likes/reactions** (social signaling, impulsive, low-cost)
- **Purchases** (limited to commercial intent)

Saves represent **"I believe this is worth my future time"** — the strongest possible quality endorsement from a user.

### 5.2 Pocket's V2 Interest Vector (On-Device)

**Architecture** (from telemetry event names):
1. **Taggers**: Classify the user's browsing history into tags/topics (runs locally)
2. **Interest Vector**: Multi-dimensional representation of user interests (computed locally)
3. **Item Relevance Score**: Dot-product or cosine similarity between interest vector and story vectors

**Key properties:**
- Entirely on-device — no profile ever sent to servers
- Built from browsing history, NOT just Pocket saves (saves are in a different system)
- Periodically recomputed (telemetry measures duration of this computation)
- Server provides the "recipe" (model weights/rules) but never sees the data

### 5.3 The "For You" Personalization

From `for_you_slate_provider.py`:
1. User-specific random seed for Thompson Sampling consistency (same user sees same ranking within a session)
2. Impression capping: Demote items already shown too many times
3. Topic spreading: Ensure diversity across topics
4. Preferred topic boosting: User's explicit topic preferences promoted to top positions

### 5.4 Collaborative Filtering (Inference)

The defunct `feed-machine` service (referenced in code comments, repository likely private/deleted) probably implemented:
- **User-user CF**: Users who saved similar articles get similar recommendations
- **Item-item CF**: Articles saved together frequently become "related"
- **Save co-occurrence matrix**: The foundation for Discover feed personalization

**Evidence**: The item2item recommender in the open-source code uses `recommend_points` (vector similarity), which is a content-based approach. True collaborative filtering from save behavior was in a separate, non-public system.

---

## 6. Content Quality Scoring

### 6.1 How Pocket Separates "Worth Reading" from "Just Popular"

Pocket's quality model is fundamentally **editorial + engagement validation**, not a quality score algorithm. The sequence:

```
EDITORIAL JUDGMENT (human picks it)
    → ENGAGEMENT VALIDATION (Thompson Sampling confirms people click it)
    → DIVERSITY ENFORCEMENT (publisher spread prevents dominance)
    → FRESHNESS CONTROL (scheduled_date prevents staleness)
```

### 6.2 Thompson Sampling: The Core Ranking Math

```python
# For each candidate item:
posterior = Beta(
    opens + alpha_prior,           # successes + prior
    impressions - opens + beta_prior  # failures + prior
)
score = posterior.sample()  # Stochastic — enables exploration

# Firefox New Tab priors (experimentally derived 2018):
alpha_prior = 132    # ~0.85% expected CTR
beta_prior = 15600

# General priors:
alpha_prior = 0.02
beta_prior = 1.0
```

**Why Thompson Sampling is brilliant for this use case:**
- **Explores new content**: Items with no data get sampled from the prior — fair chance to be shown
- **Exploits proven content**: Items with high CTR develop strong posteriors — shown more often
- **Stochastic**: Different users see different rankings (reduces filter bubble)
- **Self-correcting**: Bad items naturally get demoted as they accumulate impressions without clicks
- **Mathematically principled**: Bayesian optimal exploration strategy

### 6.3 Quality vs. Popularity: The Key Distinction

In Pocket's system:
- **Quality** = Editor chose it (human judgment, publisher track record)
- **Popularity** = High CTR on New Tab (engagement validation)
- **The combination**: Only high-quality content CAN be popular in this system, because only editorially-approved content enters the ranking

This solves the "clickbait problem" — you can't game Thompson Sampling because you can't get INTO the candidate pool without editorial approval.

### 6.4 The Save Threshold as Quality Signal

In the vector recommender, a minimum of **1,000 saves** is required for "frequently saved curated" fallback recommendations. This threshold operationalizes: "If 1,000+ people thought this was worth saving, it has enduring quality."

### 6.5 Comparison to Hacker News Quality Signals

The **Quality News** project (social-protocols/quality-news) provides an illuminating comparison:

| Mechanism | Hacker News | Pocket |
|-----------|-------------|--------|
| Signal | Upvotes (public, social) | Saves (private, intentional) + CTR |
| Quality metric | `upvoteRate` (upvotes / expected upvotes given rank/time) | CTR via Thompson Sampling (similar Bayesian approach) |
| Bias correction | Adjusts for rank exposure and traffic timing | Adjusts for impression count per surface/country |
| Explore/exploit | Random initial placement on /new | Thompson Sampling prior distribution |
| Fatigue modeling | Exponential decay of expected engagement | Impression capping (demote over-shown items) |
| Diversity | None (pure voting) | Publisher spread (distance=6), topic spreading |
| Editorial gate | None (community-only) | Hard editorial gate (human must approve) |

**Key parallel**: Both systems use Bayesian approaches to estimate "true quality" while correcting for exposure bias. HN's `upvoteRate = totalUpvotes / totalExpectedUpvotes` is structurally identical to Pocket's CTR posterior — both ask "given how many people saw this, is the engagement rate above average?"

**Key difference**: HN has no editorial gate, making it vulnerable to gaming. Pocket's human editorial layer prevents low-quality content from ever entering the ranking system.


---

## 7. The Firefox New Tab Integration

### 7.1 How Recommendations Reach 200M+ Users

Firefox New Tab is the single largest distribution surface for Pocket content. Every time a Firefox user opens a new tab, they see Pocket story cards below the search bar and top sites.

**Content delivery flow:**
1. Firefox client requests stories from Pocket API (CDN-cached, refreshed periodically)
2. Server returns ranked list of ~50 curated items for user's locale
3. Client applies local personalization (V2 interest vector re-ranking)
4. Client renders 8-20 story cards depending on viewport
5. Impression pings sent back (tile_id only, no user identity)

### 7.2 Metrics Mozilla Tracks (from Metric Hub)

Confirmed metrics in Mozilla's experimentation framework:

| Metric | Definition |
|--------|-----------|
| `organic_content_impressions` | Count of impressions on organic (non-sponsored) content |
| `organic_content_clicks` | Count of clicks on organic content |
| `sponsored_content_ctr` | Rate of clicks per impression for sponsored content |
| `sponsored_content_impressions` | Count of impressions of sponsored content |
| `sponsored_content_clicks` | Count of clicks on sponsored content |
| `any_organic_content_clicks` | Boolean: client had any content clicks during experiment period |
| `newtab_content_enabled` | Whether content is enabled on New Tab |
| `newtab_sponsored_content_stories_enabled` | Whether sponsored stories are enabled |

**Data source**: `newtab_clients_daily` — New Tab visits aggregated to client-day level.

**Statistics used**: `binomial`, `linear_model_mean`, `deciles`, `bootstrap_mean`, `population_ratio`

### 7.3 The oCTR Study (2018)

Mozilla ran a Shield study (Bugzilla #1476815): "Pref Flip Study: New Tab Pocket Recommendations oCTR" — specifically measuring organic click-through rate as the key success metric. Users saw new recommendations every ~90 minutes. This study experimentally derived the Firefox-specific Thompson Sampling priors (alpha=132, beta=15600) that remain in the codebase.

### 7.4 Sponsored Content (SPOC) Architecture

- Position: Fixed at slot 3 (3rd position in the story grid)
- Labeling: Clearly marked as `card_type: "spoc"` vs `"organic"`
- Quality gate: `below_min_score` — SPOCs must meet minimum quality threshold
- Frequency cap: Per-campaign and per-domain caps prevent over-exposure
- User controls: Can dismiss individual SPOCs or disable all sponsored stories
- Revenue: Primary monetization mechanism for the recommendation system

### 7.5 Localization

Content surfaces are per-locale:
- NEW_TAB_EN_US, NEW_TAB_DE_DE, NEW_TAB_FR_FR, NEW_TAB_ES_ES, NEW_TAB_IT_IT, NEW_TAB_EN_GB
- Each surface has independent engagement metrics (same article, different CTR per country)
- Regional metrics weighted at 0.95 (high weight for local engagement)
- Regional prior multiplied by 0.1 (acknowledging lower impression counts in smaller markets)

---

## 8. Editorial Curation Layer

### 8.1 The Human-Algorithm Interaction

```
EDITORS:                           ALGORITHMS:
├── Discover content               ├── Re-rank editorial picks
│   (via save velocity,            │   (Thompson Sampling)
│    trending signals,             ├── Enforce diversity
│    publication monitoring)       │   (publisher spread)
├── Evaluate quality               ├── Validate via engagement
│   (editorial judgment)           │   (CTR confirms choices)
├── Assign topic                   ├── Personalize locally
│   (one of 16 categories)        │   (interest vector)
├── Schedule to surface            └── Explore new content
│   (NEW_TAB_EN_US, etc.)             (prior distribution)
└── Blocklist violations
    (permanent removal)
```

### 8.2 Open-Source Curation Tools

Pocket's GitHub organization includes:
- **`Pocket/curation-admin-tools`** (TypeScript): The editorial team's interface for managing curated content
- **`Pocket/curated-corpus-api`** (TypeScript): API that serves scheduled editorial selections to the recommendation engine
- **`Pocket/collection-api`** (archived): Managed Pocket's themed curated collections

### 8.3 Editorial Principles (from Pocket for Publishers FAQ)

> "With the help of Pocket users and algorithms, our team of curators identify which articles should be recommended. The stories are sourced from a wide and diverse assortment of publications with a track record of trustworthy and accurate coverage."

**What editors look for** (confirmed):
- Track record of trustworthy and accurate coverage
- Diversity of publications (not just major outlets)
- Informed by what is being read AND saved in Pocket
- Oversight of both New Tab recommendations and Pocket Hits newsletter

**What editors DON'T do** (inference from architecture):
- They don't rank content — algorithms handle ordering
- They don't personalize — that's client-side
- They don't monitor individual users — privacy architecture prevents this

### 8.4 Content Moderation and Misinformation

**Pocket's approach to misinformation** (confirmed mechanisms):

1. **Pre-publication gate**: Only editorially-approved content enters the system. This is the strongest misinformation defense — no viral spread of unvetted content.
2. **Publisher track record**: "trustworthy and accurate coverage" is an editorial criterion
3. **Blocklisting**: Specific items can be permanently removed via `blocklists.json`
4. **No user-generated candidate selection**: Unlike social media, users cannot inject content into the recommendation pool
5. **Domain-level controls**: The system tracks publishers/domains and can filter at that level

**What Pocket does NOT do** (notable absences):
- No automated fact-checking or misinformation detection in the open-source code
- No content-level semantic analysis for truthfulness
- No community flagging mechanism for recommended content
- No third-party fact-check integration

**Mozilla Foundation's broader stance**: The Foundation has published research on algorithmic transparency ("In Transparency We Trust?", 2024) and investigated YouTube's algorithm recommending policy-violating content. They advocate for third-party auditing of recommendation systems. Pocket's own system is partially transparent through open source code but editorial decision-making remains opaque.

---

## 9. Trending and Freshness

### 9.1 How Pocket Detects Trending Content

**Critical finding from source code**: There is NO explicit "trending detection" algorithm in the recommendation-api. Instead, trending is an emergent property of several mechanisms:

1. **Editorial observation of save velocity**: Editors notice articles getting saved rapidly and schedule them for recommendation. This is the primary trending mechanism — human-mediated, not algorithmic.

2. **Thompson Sampling with 1-day windows**: Items gaining rapid engagement in the last 24 hours naturally develop strong posteriors. An article that suddenly gets many clicks will have a high sampled score, rising in rank.

3. **Scheduled dates**: Editors schedule items for specific days, creating a natural refresh cycle. New items replace old ones on a daily cadence.

4. **Impression cap at 3M**: Once an item reaches 3 million impressions, the syndicated boost is removed. This prevents stale content from permanently occupying top positions.

### 9.2 Freshness Mechanisms

| Mechanism | How It Works | Effect |
|-----------|-------------|--------|
| `scheduled_date` sort | Recency is a ranking factor (more recent = higher) | Daily refresh cycle |
| 1-day trailing window | Thompson Sampling uses most recent 24h engagement | Rapid response to changing interest |
| 90-day freshness filter | Vector recommender excludes items older than 90 days | Prevents stale related recommendations |
| Editorial scheduling | Editors actively replace old items with new ones | Human-paced content rotation |
| Impression saturation | After 3M impressions, syndicated boost removed | Natural decay of over-exposed content |

### 9.3 The Virality Lifecycle of Pocket Content

Based on the architecture, content follows this lifecycle:

```
Day 0: Article published on the web
Day 0-3: Early adopters save it to Pocket (save velocity rises)
Day 1-5: Editorial team notices save velocity, evaluates quality
Day 2-7: If approved, editor schedules to Curated Corpus (enters candidate set)
Day 3-10: Thompson Sampling begins ranking based on CTR
Day 3-14: High-CTR items rise; low-CTR items naturally decay
Day 7-30: Engagement stabilizes; item cycles off as newer items scheduled
Day 30-90: Item available for vector-based "related article" recommendations
Day 90+: Removed from vector recommender freshness filter
```

**Key insight for Feedmine**: There's no real "viral" period for Pocket content because the editorial gate introduces a multi-day delay. Content that trends on social media may take 2-7 days to appear in Pocket recommendations. This is a feature, not a bug — it filters out flash-in-the-pan viral content that lacks lasting value.


---

## 10. Privacy Architecture

### 10.1 The Core Privacy Principle

Mozilla's approach to Pocket is **privacy by architecture, not by policy**. The system is designed so that personal data physically cannot reach Mozilla's servers.

### 10.2 Data That NEVER Leaves the Device

- Full browsing history
- Domain visit frequency
- Interest vector (V2 personalization)
- Personalization scores per story
- Which stories the user actually clicked (only aggregate counts returned)
- The user's Pocket save library (for personalization purposes — sync is separate)

### 10.3 Data Collected (Aggregate, Non-Identifying)

| Data Point | Purpose | Identifying? |
|-----------|---------|-------------|
| Impression count per tile_id | Thompson Sampling denominator | No — aggregated across all users |
| Click count per tile_id | Thompson Sampling numerator | No — aggregated across all users |
| `impression_id` | Session-level correlation | No — random per session, not linked to user |
| Locale, addon version | Bucketing for experiments | Low — shared by millions |
| SPOC fill/non-fill reasons | Ad quality monitoring | No — aggregate |
| Personalization timing | Performance monitoring | No — just duration metrics |

### 10.4 The "No client_id" Design

In Pocket impression pings, `client_id` is explicitly set to `"n/a"`. This means:
- Mozilla CANNOT correlate which stories different users see
- They get only aggregate impression/click totals per tile
- No user-level engagement profile exists on the server
- A/B experiments use separate randomization, not persistent user IDs

### 10.5 OHTTP (2025+)

Mozilla's recent privacy notice mentions "privacy-preserving technology called OHTTP that hides your IP address and other identifying data from all parties (including Mozilla itself)" — used for newer suggestion features. OHTTP (Oblivious HTTP) routes requests through a relay that strips IP addresses before reaching Mozilla's servers.

### 10.6 Context Graph Research (Historical)

Mozilla's 2017-2018 "Context Graph" research project did collect browsing data, but:
- Only 10,000 opt-in participants
- Data: URL, tab ID, time/date, time spent on page
- Sensitive sites filtered via domain blocklist
- Only a handful of developers had raw access
- Data deleted after defined retention period
- Differential privacy explored as a research direction
- This was research, never deployed to general users

### 10.7 Privacy Implications for Feedmine

Pocket's privacy architecture is instructive but Feedmine's situation is fundamentally simpler:
- **Feedmine is local-only** — no server component means no privacy leakage by design
- **No aggregate signals available** — Feedmine can't use Thompson Sampling on population engagement because there's no population
- **Trade-off**: Privacy is free, but collaborative filtering is impossible without a network component
- **Opportunity**: Feedmine can use signals Pocket deliberately avoids (full browsing history, reading time, scroll depth) because they never leave the device

---

## 11. The Save Action as the Ultimate Signal

### 11.1 Why Saves > Likes > Clicks

The hierarchy of user intent signals, from strongest to weakest:

```
STRONGEST INTENT
    │
    ├── Purchase/Subscribe (money commitment)
    ├── Save/Bookmark (time commitment, future intent)
    ├── Share (social endorsement, reputation at stake)
    ├── Comment (engagement, but includes negative engagement)
    ├── Like/Heart (low-friction, social signaling)
    ├── Click/Open (curiosity, often impulsive)
    ├── Impression/View (passive, may not be conscious)
    │
WEAKEST INTENT
```

### 11.2 Properties of the Save Signal

| Property | Saves | Clicks | Likes |
|----------|-------|--------|-------|
| **Friction** | High (deliberate action) | Low (impulsive) | Low (tap) |
| **Future orientation** | Yes (planning to read later) | No (immediate) | No (immediate) |
| **Social pressure** | None (private by default) | None | High (visible) |
| **Quality indication** | Strong (worth future time) | Weak (could be clickbait) | Medium (approved) |
| **Reversibility** | Low (people rarely unsave) | N/A | Medium (unlikes common) |
| **Spam resistance** | High (no incentive to fake-save) | Low (clickbait optimizes for clicks) | Medium (engagement bait) |
| **Sparsity** | Very sparse (few saves per session) | Dense (many clicks per session) | Medium |

### 11.3 Research on Saves as Recommendation Signals

**From academic literature on implicit feedback in recommender systems:**

The seminal framework comes from Hu, Koren, and Volinsky (2008) "Collaborative Filtering for Implicit Feedback Datasets" which establishes that implicit signals have varying **confidence levels**. A save/bookmark is a high-confidence positive signal because:
- It requires explicit user action (not inferred from passive behavior)
- It's unambiguous in direction (positive — user wants to revisit)
- It's resistant to noise (accidental saves are rare)

Modern systems (per 2025 research) increasingly weight saves at **5-7x** the value of likes in algorithmic scoring. Instagram and TikTok have confirmed that saves are their highest-weighted engagement signal for content distribution.

**Key insight from platform algorithm research:**
- Saves predict **return visits** (the user will come back to the platform to access saved content)
- Saves correlate with **long-term satisfaction** (users who save report higher satisfaction in surveys)
- Saves resist **engagement optimization** (you can't trick someone into saving clickbait — they save what they genuinely want to revisit)
- Saves create **compounding value** (each save enriches the user's profile, making future recommendations better)

### 11.4 The Save-to-View Ratio as Quality Metric

While Pocket uses CTR (opens/impressions) for ranking, a hypothetical **save-to-view ratio** would be an even stronger quality signal:
- A high save-to-view ratio means: "People who see this think it's worth their future time"
- This filters out clickbait (high CTR, low save rate) from genuine quality (high CTR AND high save rate)
- Pocket likely uses this signal internally for editorial prioritization, though it's not explicit in the open-source ranking code

### 11.5 The Archive Signal: Read Completion

When a Pocket user archives an article, they're signaling: "I read this and I'm done." This is a completion signal:
- **Save + Archive** = "I intended to read this AND I followed through" (highest quality confirmation)
- **Save + Never Archive** = "I thought this was worth reading but never got to it" (intention without follow-through)
- **Save + Delete** = "I changed my mind about this" (weak negative signal)

**Status in Pocket's system**: The archive action is NOT confirmed in the open-source recommendation-api engagement metrics (which only track opens/impressions). However, it's reasonable to infer that Pocket's internal analytics track completion rates as a quality indicator. The open-source code represents only the New Tab ranking surface, not the full Pocket ecosystem.

---

## 12. Limitations and Weaknesses

### 12.1 Confirmed Limitations

1. **Small candidate pool**: ~50 items per locale per day means limited coverage. Niche interests may never be served. A user interested in advanced mathematics or obscure hobbies will rarely see relevant recommendations.

2. **English-first**: Vector recommendations are English-only. Non-English locales get editorial curation but lack the item2item similarity system.

3. **No long-tail content**: The editorial gate explicitly prevents niche content from entering recommendations. An article with 50 saves from passionate specialists is invisible to the system — it needs 1,000+ saves to even qualify for vector fallback.

4. **Domain-level dedup only**: `unique_domains_first()` moves duplicate domains to end of list but doesn't detect semantically similar content from different publishers covering the same story.

5. **No read-through optimization**: The New Tab engagement metric is "opens" (clicks), not read completion. The system optimizes for headlines that attract clicks, not content that satisfies readers — though the editorial gate mitigates this somewhat.

6. **Cold-start for new publishers**: Publisher track record is an editorial criterion, making it difficult for new/independent publications to enter the recommendation pool.

7. **Fixed topic taxonomy**: 16 topics cannot capture the full range of human interests. Cross-cutting topics (e.g., "climate science policy," "AI in healthcare") must be forced into a single category.

8. **Personalization ceiling**: The V2 interest vector system is limited by the server-provided "recipe." Users cannot customize how personalization works or override algorithmic decisions beyond topic preferences.

### 12.2 Structural Criticisms

1. **Editorial bottleneck**: The system's quality depends entirely on the editorial team's judgment, taste, and capacity. If editors are overworked, biased, or understaffed, quality degrades silently.

2. **Opacity of editorial decisions**: While the ranking algorithms are open source, the editorial decision-making process is not transparent. Why was article X chosen and article Y rejected? No public criteria or appeals process.

3. **Engagement optimization still present**: Even within an editorially-curated pool, Thompson Sampling optimizes for clicks. Headlines that are more provocative will still outperform thoughtful ones, just within a quality-gated set.

4. **No user feedback on recommendation quality**: Users can dismiss stories but cannot signal "this was great" or "this didn't match my interests" — only negative feedback (block/dismiss) is captured in the open-source system.

5. **Popularity bias within curated set**: Thompson Sampling naturally promotes already-popular items (rich-get-richer within the candidate set). The prior distribution helps with exploration, but mainstream content will still dominate.

6. **US/English cultural bias**: The editorial team, publisher selection, and topic taxonomy all reflect primarily US/English media consumption patterns. International users may find recommendations culturally misaligned.

### 12.3 Technical Gaps in the Open-Source Code

| Gap | What's Missing | Impact |
|-----|---------------|--------|
| Embedding generation | How vectors are created for Qdrant | Cannot replicate semantic similarity |
| feed-machine | Collaborative filtering from saves | Cannot replicate personalized Discover |
| Topic classifier | How topics are assigned | Cannot replicate automated classification |
| Curation criteria | Internal editorial guidelines | Cannot replicate quality judgment |
| Dwell time tracking | How reading time is measured | Cannot replicate completion signals |
| V2 recipe format | What the personalization recipe contains | Cannot replicate local personalization model |


---

## 13. What Feedmine Can Steal from Pocket

### 13.1 Thompson Sampling for Feed Ranking (Direct Port)

**Implementation**: Use Beta-Binomial Thompson Sampling to rank items within each feed.

```python
# For each feed item:
alpha = clicks + alpha_prior  # successes
beta = impressions - clicks + beta_prior  # failures
score = random.betavariate(alpha, beta)  # sample from posterior

# Priors for a personal reader (much smaller scale than Firefox):
alpha_prior = 1    # expect ~10% CTR for interesting content
beta_prior = 9
```

**Why this works for Feedmine**:
- Naturally explores new/unread content (items with no engagement get sampled from prior)
- Adapts to user behavior over time (frequently-clicked feeds rise)
- Stochastic ranking prevents filter bubble (different ordering each view)
- Simple to implement, no ML infrastructure needed
- Works with sparse data (a single user's engagement)

### 13.2 The Star/Save as Primary Quality Signal

**Implementation**: Make the star/save action the strongest signal in the system.

```
Signal weights (for Feedmine):
  Star/Save:    5.0x
  Read/Archive: 3.0x
  Click/Open:   1.0x
  Impression:   0.0 (just denominator)
  Dismiss:     -2.0x
```

Use save co-occurrence for "related articles":
- If user saves article A and article B in same session → similarity signal
- Build local item-item similarity from save patterns over time
- Require minimum 3 saves on an item before using it for similarity (local equivalent of Pocket's 1,000-save threshold, scaled to single user)

### 13.3 Publisher Spread / Source Diversity

**Implementation**: Pocket's `spread_publishers` algorithm with `distance=6`.

```python
def spread_sources(ranked_items, min_distance=4):
    """No two items from same feed source within min_distance positions."""
    result = []
    recent_sources = []
    deferred = []
    
    for item in ranked_items:
        if item.source in recent_sources[-min_distance:]:
            deferred.append(item)
        else:
            result.append(item)
            recent_sources.append(item.source)
    
    # Append deferred items at end
    result.extend(deferred)
    return result
```

### 13.4 The Readability Extraction Pipeline

**Direct use**: mozilla/readability is open source (Apache 2.0-compatible). Use it directly for:
- Extracting article text for local NLP processing
- Computing word count / estimated reading time
- Displaying clean reader view
- Feeding text to local classification models

```javascript
// Direct integration:
const { Readability } = require('@mozilla/readability');
const { JSDOM } = require('jsdom');

const doc = new JSDOM(html, { url: articleUrl });
const article = new Readability(doc.window.document).parse();
// article.textContent → feed into local classifier
// article.length → reading time estimate
// article.title, article.byline → metadata enrichment
```

### 13.5 Interest Vector from Save History

**Implementation**: Build a local interest vector from the user's save/star history.

```python
# Simple TF-IDF based approach:
1. Extract text from all saved articles (via Readability)
2. Build vocabulary from saved articles
3. Compute TF-IDF vectors for each saved article
4. User interest vector = weighted average of saved article vectors
   (weight by recency — recent saves count more)
5. Score new articles: cosine_similarity(article_vector, interest_vector)
```

For topic-level preferences:
```python
# Topic distribution from saves:
topic_weights = {}
for saved_article in user_library:
    topic = classify(saved_article)  # local classifier
    recency_weight = decay(saved_article.save_date)
    topic_weights[topic] = topic_weights.get(topic, 0) + recency_weight

# Normalize to probability distribution
# Use for topic boosting in feed ranking
```

### 13.6 Engagement Trailing Windows

**Implementation**: Track engagement in multiple time windows like Pocket.

```sql
-- Local engagement metrics per feed/item:
trailing_1_day_clicks, trailing_1_day_impressions
trailing_7_day_clicks, trailing_7_day_impressions
trailing_30_day_clicks, trailing_30_day_impressions

-- Use shortest window for "trending" detection
-- Use longest window for "reliable quality" detection
```

### 13.7 The "Recipe" Model (Downloadable Personalization)

**Adaptation for Feedmine**: Instead of downloading a recipe from a server, Feedmine can:
- Ship with a default personalization model (topic weights, diversity parameters)
- Allow users to export/import personalization configs
- Let power users tweak parameters (exploration rate, diversity distance, topic weights)
- Periodically retrain the local model based on accumulated engagement data

### 13.8 Impression Capping

**Implementation**: From Pocket's "For You" slate — demote items shown too many times.

```python
def apply_impression_cap(items, max_impressions=5):
    """Demote items user has seen too many times without clicking."""
    for item in items:
        if item.impressions > max_impressions and item.clicks == 0:
            item.score *= 0.1  # Heavy penalty
    return sorted(items, key=lambda x: x.score, reverse=True)
```

---

## 14. What Feedmine Can Do Better Than Pocket

### 14.1 Full Signal Depth (No Privacy Constraints)

Since everything is local, Feedmine can use signals Pocket deliberately avoids:

| Signal | Pocket Can't Use (privacy) | Feedmine Can Use (local) |
|--------|---------------------------|------------------------|
| Reading time per article | ❌ | ✅ Precise dwell time |
| Scroll depth / completion | ❌ | ✅ Exact scroll percentage |
| Copy/paste actions | ❌ | ✅ Content extraction signals |
| Tab switching patterns | ❌ | ✅ Attention modeling |
| Time of day preferences | ❌ (no user ID) | ✅ Personal schedule patterns |
| Article length preferences | ❌ | ✅ Optimal length modeling |
| Cross-feed correlations | ❌ | ✅ Full reading graph |

### 14.2 Configurable Quality Criteria

Pocket's quality is editorial judgment — one size fits all. Feedmine can let users define quality:

```yaml
quality_rules:
  - min_word_count: 800          # No listicles
  - max_link_density: 0.3        # No link farms
  - require_author: true         # No anonymous content
  - min_paragraph_count: 5       # Substantial articles only
  - preferred_reading_time: 5-15min
  - penalize_clickbait_patterns: true
  - boost_if_saved_similar: true
```

### 14.3 No Editorial Bottleneck

Pocket is limited to ~50 items/day per locale because humans must review everything. Feedmine can process unlimited content because:
- Quality assessment is algorithmic + user-configured
- No staffing constraints
- User's own behavior IS the editorial judgment
- Long-tail content is accessible (niche feeds that Pocket would never curate)

### 14.4 True Long-Tail Support

Pocket requires 1,000+ saves for vector recommendation fallback. A niche article about Rust compilers or Mesoamerican archaeology will never reach that threshold. Feedmine can:
- Recommend based on a SINGLE user's save history (no crowd needed)
- Build similarity from content features, not popularity metrics
- Value niche content equally with mainstream content
- Use the user's feed subscriptions as explicit interest declaration

### 14.5 Read-Completion as First-Class Signal

Pocket's open-source ranking only uses "opens" (clicks). Feedmine can track what Pocket can't:
- **Started but abandoned** (opened, read <10% → negative signal on content or timing)
- **Completed** (scrolled to end → strong positive signal)
- **Re-read** (opened again after first read → extremely strong quality signal)
- **Saved after reading** (read then starred → confirmed quality, not just headline appeal)

### 14.6 Personal Archive Intelligence

Pocket's archive is a flat list. Feedmine can build intelligence FROM the archive:
- Detect reading patterns (what time of day, what topics, what length)
- Identify "holes" in the user's knowledge graph (topics saved but never read)
- Surface old saved items that relate to newly published content
- Build a personal knowledge base from extracted article text

### 14.7 Transparent, Explainable Ranking

Pocket's editorial decisions are opaque. Feedmine can show:
- WHY each item is ranked where it is (which signals contributed)
- What the Thompson Sampling posterior looks like for each feed
- How the user's interest vector maps to content
- Which similar saved articles influenced the recommendation

### 14.8 No Sponsored Content Pollution

Pocket inserts SPOCs in position 3. Feedmine is ad-free by nature. The entire ranking serves the user's interests, never an advertiser's.

### 14.9 Flexible Topic Taxonomy

Pocket has 16 fixed topics. Feedmine can:
- Use emergent topics from the user's save library (no predefined categories)
- Support arbitrary depth (Technology > Programming > Rust > Async)
- Allow user-defined categories and tags
- Adapt taxonomy as interests evolve

---

## 15. Open Questions

### 15.1 Technical Unknowns

1. **What embedding model does Pocket use for Qdrant vectors?** The recommendation-api consumes embeddings but doesn't generate them. Could be sentence-transformers, a custom BERT variant, or something else entirely.

2. **What did the `feed-machine` service do?** Referenced in code comments but repository is private/deleted. Likely implemented collaborative filtering from save co-occurrence — the "Users who saved X also saved Y" system.

3. **What does a V2 personalization "recipe" look like?** Downloaded from server to Firefox for local execution. Format undocumented. Could be: model weights, topic mappings, scoring rules, or a combination.

4. **Does Pocket track dwell time?** The open-source code only shows "opens/impressions" but Mozilla's Context Graph research collected time-on-page. The broader Pocket ecosystem likely measures reading time internally.

5. **How does the topic classifier work?** Topics arrive pre-assigned in the Curated Corpus API. An upstream classifier (or editor) assigns them, but the mechanism is not in public code.

### 15.2 Strategic Questions for Feedmine

1. **Can Thompson Sampling work with a single user?** Yes, but convergence is slow. With one user generating maybe 20-50 interactions/day, you need weeks of data before posteriors are meaningfully different from priors. Consider: faster decay, stronger priors from initial setup, or hybrid scoring.

2. **Should Feedmine implement collaborative filtering?** Without a network component, traditional CF is impossible. Alternatives:
   - Content-based similarity from local embeddings
   - Import signals from external sources (HN upvotes, Reddit karma, citation counts)
   - Optional anonymized signal sharing (federated learning, if privacy allows)

3. **How to replicate the editorial quality gate locally?** Options:
   - Use the feed subscription itself as a trust signal (user chose to follow this source)
   - Source-level quality scores (based on historical reading completion rate)
   - Content heuristics (word count, link density, author presence, Readability score)
   - Optional integration with external quality signals (domain reputation databases)

4. **How to handle the cold-start problem without crowd signals?** Options:
   - Strong priors from feed subscription (user already trusts these sources)
   - Content features for initial scoring (length, topic match, source quality)
   - Aggressive exploration in early days (show diverse content, learn fast)
   - Allow manual seeding (import Pocket library, starred items from RSS reader)

5. **Is the save signal sufficient for a single user?** A single user might save 2-5 items per day. This is extremely sparse compared to Pocket's millions of aggregate saves. Feedmine must:
   - Weight each save heavily (it's precious data)
   - Supplement with lower-cost signals (clicks, reading time, scroll depth)
   - Use content features to generalize from few saves to many recommendations
   - Build slowly — the system improves over months, not days

### 15.3 Research Directions

1. **Local language models for content classification**: Can a small local model (e.g., a fine-tuned DistilBERT or a quantized LLM) replicate what Pocket's editors + classifiers do?

2. **Bayesian user modeling from saves**: Can we formally model a user's evolving interests as a Bayesian prior updated by each save action?

3. **Save prediction as engagement metric**: Can we train a local model to predict "would the user save this?" as a proxy quality score, then use that prediction for ranking?

4. **Cross-source trending detection**: Without aggregate saves, can Feedmine detect trending content from publication velocity across subscribed feeds?

5. **The "anti-Thompson" question**: Thompson Sampling optimizes for CTR, which can still favor sensational content within a quality-gated set. Can Feedmine optimize for a better metric — perhaps save rate, or reading completion rate — that directly measures lasting value?

---

## Appendix A: Pocket's Open-Source Repositories

| Repository | Purpose | License |
|-----------|---------|---------|
| `Pocket/recommendation-api` | Main ranking/recommendation engine | Apache 2.0 |
| `Pocket/curated-corpus-api` | Editorial curation management API | - |
| `Pocket/curation-admin-tools` | Editorial team's UI tools | - |
| `Pocket/pocket-monorepo` | All Pocket TypeScript backend services | Apache 2.0 |
| `Pocket/content-monorepo` | Content processing services | - |
| `Pocket/collection-api` (archived) | Themed curated collections | - |
| `Pocket/web-client` | User-facing web application | MPL 2.0 |
| `Pocket/firefox-android-home-recommendations` (archived) | Android home recs API | Apache 2.0 |
| `Pocket/dynamore` | DynamoDB utilities | MPL 2.0 |
| `mozilla/readability` | Article content extraction | Apache 2.0 |

## Appendix B: Key Parameters (from Source Code)

```python
# Thompson Sampling Priors
DEFAULT_ALPHA_PRIOR = 0.02
DEFAULT_BETA_PRIOR = 1.0
DEFAULT_FIREFOX_ALPHA_PRIOR = 132      # ~0.85% expected CTR
DEFAULT_FIREFOX_BETA_PRIOR = 15600

# Publisher Diversity
NEW_TAB_SPREAD_DISTANCE = 6
GENERAL_SPREAD_DISTANCE = 3

# Regional Weighting
REGIONAL_WEIGHT = 0.95
REGIONAL_PRIOR_MULTIPLIER = 0.1

# Content Thresholds
MIN_SAVE_COUNT_VECTOR_FALLBACK = 1000
VECTOR_FRESHNESS_DAYS = 90
SYNDICATED_BOOST_IMPRESSION_CAP = 3_000_000

# Trailing Windows
TRAILING_PERIODS = [1, 7, 14, 21, 28]  # days
NEW_TAB_TRAILING_PERIOD = 1  # day
FOR_YOU_TRAILING_PERIOD = 14  # days

# Engagement Metrics Per Item
METRICS_KEYED_ON = "surface_id / slate_config_id / corpus_item_id / country"
```

## Appendix C: The Signal Hierarchy (Pocket vs. Feedmine)

```
POCKET SIGNAL HIERARCHY:
  Editorial Approval → Candidate Pool (hard gate)
  Thompson Sampling CTR → Ranking (algorithmic)
  Save Count → Vector Quality Threshold (1,000+)
  Save Velocity → Editorial Discovery (human-mediated)
  User Save History → Discover Personalization (separate system)
  Local Interest Vector → Client-Side Re-ranking (on-device)

PROPOSED FEEDMINE SIGNAL HIERARCHY:
  Feed Subscription → Source Trust (user chose it)
  Star/Save Action → Quality Signal (5x weight)
  Read Completion → Confirmation Signal (3x weight)
  Click/Open → Interest Signal (1x weight)
  Content Features → Cold-Start Scoring (Readability, length, author)
  Local Interest Vector → Personalization (always on-device)
  Thompson Sampling → Ranking (on local engagement data)
  Source Diversity → Feed Diversity (spread algorithm)
  Impression Cap → Freshness (demote stale/over-shown)
```

---

*End of document. This represents the complete knowledge base on Pocket's intelligence as of July 2026, drawn from open-source code, official documentation, telemetry specifications, and public research. Where facts are inferred rather than confirmed, this is explicitly noted.*
