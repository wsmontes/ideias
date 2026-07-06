# Feedly Intelligence — Complete Technical Reference for Feedmine

> **Purpose:** Actionable knowledge base for building a local-only feed reader that replicates Feedly's intelligence layer. Written as reference material for the Feedmine project.
>
> **Research basis:** Feedly developer API docs (ArticleJSON spec), engineering blog posts, case studies, user forums, third-party reviews, and competitive analysis. Research conducted July 2026.

---

## 1. Executive Summary

### What Feedly Is

Feedly started as a Google Reader replacement in 2013 — a simple RSS aggregator. It has since evolved into an **AI-powered research and intelligence platform** serving two distinct markets:

1. **Individual users** ($0–$13/mo) — content curation, noise reduction, personal research
2. **Enterprise teams** ($1,600–$3,200+/mo) — cyber threat intelligence (CTI), market intelligence, competitive analysis

The pivot happened around 2018–2019 when Feedly introduced "Leo," an AI research assistant. Rather than building a social feed algorithm, Feedly built a **rule-based priority system powered by ML annotations**. This is a critical distinction:

- **Social media approach:** Opaque algorithm learns from engagement, optimizes for time-on-platform
- **Feedly approach:** Transparent user-configured rules evaluated against ML-extracted metadata. No implicit behavioral learning. No engagement optimization.

### Key Architecture Facts

- **100% server-dependent.** All ML processing, deduplication, clustering, engagement scoring, and source crawling happens on Feedly's servers.
- **REST API** at `api.feedly.com/v3` with OAuth 2.0. Client apps are thin presentation layers.
- **Processes millions of articles daily** from 140M+ indexed sources.
- **30,000+ pre-trained AI models** (topic classifiers, entity recognizers, event detectors) running in parallel.

### Why This Matters for Feedmine

Feedly proves that:
- Rule-based priority + ML annotation is more transparent and user-trustable than learned ranking
- Deduplication and clustering are the highest-value features for high-volume readers
- Entity extraction + salience scoring is the key to "understanding" articles beyond keyword matching
- You don't need behavioral learning to build useful intelligence — explicit user configuration works

---

## 2. Leo AI — Complete Feature Breakdown

### What Leo Is

Leo is Feedly's AI assistant — not a single model but a **collection of ML skills** that annotate articles, which users then configure rules against. Leo does NOT learn from reading behavior. It is a configurable filter, not a recommendation engine.

### Leo Skills (ML Capabilities)

| Skill | What It Does | Technical Mechanism |
|-------|-------------|-------------------|
| **Named Entity Recognition** | Detects people, companies, products, threat actors, malware families | NER model + Wikipedia/knowledge base disambiguation + salience scoring |
| **Topic Classification** | Classifies articles into topics with confidence scores (0–1) | Multi-label classifier, 30,000+ pre-trained topic models |
| **Business Event Detection** | Detects strategic moves (funding rounds, acquisitions, partnerships, product launches) | Event extraction model with structured output |
| **Vulnerability Detection** | Extracts CVEs, CVSS scores, affected vendors | Domain-specific NER + structured extraction |
| **Sentiment/Tone** | Assesses article sentiment | Not heavily exposed in UI but present in API |
| **Summarization** | Extractive summaries (2 most salient sentences with scores) | Sentence salience scoring, NOT generative |
| **Engagement Scoring** | Absolute engagement count + relative engagement rate (>1 = viral) | Social signal aggregation across platforms |
| **Deduplication** | Identifies articles with 85% content overlap | Locality-Sensitive Hashing (LSH) |
| **Clustering** | Groups related articles about the same story | Density-based clustering on article embeddings |

### Priority Types (User-Configured Rules)

Users don't "train" Leo in the traditional ML sense. They configure **priority rules** that Leo evaluates against its ML annotations:

1. **Topic Priority** — "Prioritize articles about artificial intelligence" → Leo checks its topic classification scores
2. **Entity/Mention Priority** — "Prioritize articles mentioning Microsoft" → Leo checks NER results
3. **Business Event Priority** — "Prioritize funding rounds" → Leo checks event detection
4. **Like Board Priority** — "Prioritize articles similar to my Research board" → Leo computes similarity to saved articles (mechanism: likely embedding-based cosine similarity, though Feedly hasn't disclosed specifics)
5. **Vulnerability Priority** — "Prioritize CVSS > 8 vulnerabilities affecting my vendors" → Leo checks structured CVE data

### How "Training Leo" Actually Works (User Experience)

Despite the marketing language of "train Leo," the actual UX is:

1. User navigates to a feed or folder
2. Clicks "Train Leo" / "Add Priority"
3. Selects priority type (topic, entity, event, like-board)
4. Configures parameters (e.g., selects "artificial intelligence" from topic taxonomy)
5. Optionally adds boolean logic: `+AND`, `+OR` to combine conditions
6. Optionally scopes to specific feeds/folders or applies globally
7. Leo immediately starts evaluating new articles against the rule

**There is no thumbs-up/thumbs-down training loop.** There is no implicit feedback. The "training" is pure configuration. This is both a strength (transparent, predictable) and weakness (requires upfront effort, no adaptation).

### Leo Summaries

Two types:
- **`leoSummary`** — Extractive. The 2 most salient sentences from the article, each with a salience score. Generated automatically at ingestion time. Fast, deterministic.
- **`aiActions[]`** — On-demand LLM-generated summaries, translations, or custom prompts. Uses large-context LLMs (NOT RAG). User can ask questions about the article. Introduced ~2024.

### Ask AI (LLM Feature)

Architecture decisions (from Feedly engineering blog):
- **Rejected RAG** in favor of stuffing full articles into large-context LLMs
- RAG produced inconsistent results for their use case
- Uses **two models**: one optimized for speed, one for quality
- Proprietary prompt engineering for inline citations
- Models are swapped as better ones become available
- Available only on Enterprise Advanced plans

### Deduplication

- **85% LSH (Locality-Sensitive Hashing) threshold** for duplicate detection
- Server-side stateful dedup via `contextId` tracking last 10,000–20,000 articles
- Stream processing (instant) — when a duplicate is found, it inherits cluster assignments immediately
- Duplicates stored in `duplicates[]` array on the article object
- Users see "X similar articles" collapsed under the primary

### Priority Inbox

The output of all priorities is a filtered view:
- Articles matching ANY priority rule get a green "priority" badge
- Priority articles appear in a dedicated "Priority" tab
- Non-priority articles remain in the main feed (not hidden, just not elevated)
- No ranking within priorities — it's binary: priority or not


---

## 3. Content Understanding Pipeline

### Article Ingestion Flow

```
Source Crawl → Raw HTML → Content Extraction → NLP Pipeline → Article JSON → User Rules → Priority Badge
```

### What Feedly Extracts (from ArticleJSON spec)

Every article processed by Feedly gets enriched with:

```json
{
  "id": "entry/...",
  "title": "...",
  "content": { "content": "...", "direction": "ltr" },
  "fullContent": "...",           // Feedly-fetched beyond RSS excerpt
  "author": "...",
  "origin": { "title": "...", "streamId": "..." },
  "published": 1234567890,
  
  // === ML ANNOTATIONS ===
  "entities": [
    {
      "id": "nlp/f/entity/...",
      "label": "Microsoft",
      "type": "org",              // org, person, product, threatActor, malwareFamily, mitreAttack, vulnerability
      "mentions": [{ "text": "Microsoft Corp" }],
      "salienceLevel": "about",   // about | mention
      "disambiguated": true
    }
  ],
  "commonTopics": [
    {
      "id": "nlp/f/topic/...",
      "label": "Artificial Intelligence",
      "score": 0.92,
      "salienceLevel": "about"
    }
  ],
  "businessEvents": [
    {
      "type": "fundingRound",
      "score": 0.87
    }
  ],
  "leoSummary": {
    "sentences": [
      { "text": "...", "score": 0.95 },
      { "text": "...", "score": 0.82 }
    ]
  },
  "duplicates": ["entry/abc123", "entry/def456"],
  "clusters": [{ "id": "cluster/..." }],
  "featuredMeme": { "id": "meme/...", "label": "..." },
  "engagement": 4523,
  "engagementRate": 2.3,          // >1 means viral relative to source's baseline
  
  // === CYBERSECURITY SPECIFIC ===
  "indicatorsOfCompromise": [...],  // STIX 2.1 format
  "vulnerabilities": [{ "cvssScore": 9.1, "cve": "CVE-2024-..." }],
  
  // === ON-DEMAND ===
  "aiActions": [{ "type": "summary", "result": "..." }]
}
```

### NLP Model Architecture (What We Know and Don't Know)

**Confirmed:**
- Feedly runs **30,000+ pre-trained models** (mostly topic classifiers for specific domains)
- NER includes disambiguation against Wikipedia and news-derived knowledge bases
- Salience scoring distinguishes "article IS ABOUT entity" vs "article MENTIONS entity"
- Topic classification produces scores 0–1 with salience levels
- Models operate on article text at ingestion time (not on-demand)
- Community feedback loop: users can flag incorrect entity/topic classifications, which feeds back into model retraining

**Inferred (high confidence):**
- NER likely uses a BERT-variant or similar transformer encoder fine-tuned on news/blog text
- Topic classification likely uses multi-label text classification (possibly distilled BERT or similar)
- The 30,000+ models are likely fine-tuned classifiers sharing a common backbone, not 30,000 separate architectures
- Salience scoring likely combines position features (title mention, first paragraph) with attention/frequency

**Unknown:**
- Exact model architectures and sizes
- Whether they use a single multi-task model or separate models per skill
- Training data sources beyond community feedback
- Model retraining cadence
- Infrastructure stack (cloud provider, GPU fleet)
- Embedding dimensions used for clustering/similarity

### Entity Disambiguation

Feedly's NER doesn't just find "Apple" — it determines WHETHER it's Apple Inc., apple (fruit), or Apple Records. The disambiguation system:
- Links entities to Wikipedia/Wikidata IDs
- Uses context from surrounding text
- Handles aliases (e.g., "MSFT" → Microsoft, "Fancy Bear" → APT28)
- For cybersecurity: maps threat actors to known aliases across naming conventions (MITRE, CrowdStrike, Microsoft naming)

### Engagement Scoring

Two metrics:
- **`engagement`** (absolute) — total social interactions (shares, likes, comments) across platforms
- **`engagementRate`** (relative) — normalized against the source's typical engagement. A value >1 means the article is performing above average for that source. This is the more useful signal.

### Full Content Fetching

Feedly goes beyond RSS excerpts:
- The `fullContent` field contains the full article text when Feedly's crawler fetches the original page
- This enables better NLP processing (more text = better entity/topic extraction)
- Not all articles get full content (depends on source accessibility and plan)

---

## 4. Signal Taxonomy

### Signals Feedly Uses

| Signal | Type | Source | Feedmine Equivalent |
|--------|------|--------|-------------------|
| Topic classification scores | Explicit (ML-generated) | NLP pipeline | Local topic classifier (BERT/DistilBERT) |
| Entity mentions + salience | Explicit (ML-generated) | NER pipeline | Local NER (spaCy/transformers) |
| Business events | Explicit (ML-generated) | Event extraction | Custom classifier or LLM extraction |
| Engagement count | Implicit (social) | Social platform APIs | Not available locally (or limited via APIs) |
| Engagement rate | Implicit (computed) | engagement / source baseline | Could compute if tracking source history |
| Duplicate detection | Explicit (computed) | LSH on content | MinHash/SimHash locally |
| Cluster membership | Explicit (computed) | Density-based clustering | HDBSCAN on local embeddings |
| Source reputation | Implicit (curated) | Feedly's source index | User-defined source tiers |
| Publication recency | Explicit (metadata) | Article timestamp | Direct from feed |
| Author | Explicit (metadata) | Article metadata | Direct from feed |
| Article length | Explicit (computed) | Word count | Trivial to compute |
| Content type | Inferred | Analysis | Classify: news/opinion/tutorial/press-release |
| User priority rules | Explicit (user-configured) | User settings | User-configured rules |
| Board similarity | Explicit (user + ML) | Embedding similarity to saved articles | Embedding similarity to bookmarked items |
| Mute filters | Explicit (user-configured) | User settings | User-configured filters |

### What Feedly Does NOT Use (Confirmed)

- ❌ Reading time / dwell time
- ❌ Click-through behavior
- ❌ Scroll depth
- ❌ Read/unread patterns over time
- ❌ Time-of-day preferences
- ❌ Cross-user collaborative filtering
- ❌ A/B testing of content ordering
- ❌ Recency decay functions
- ❌ Personalized ranking models

This is remarkable. Feedly's priority system is **entirely rule-based with ML annotations**. There is no learned personalization beyond what the user explicitly configures.


---

## 5. Priority and Ranking Logic

### How Content Gets Elevated

Feedly's priority system is **NOT a ranking algorithm**. It is a **binary classification**: priority or not-priority. Here's the evaluation flow:

```
New Article Arrives
    ↓
NLP Pipeline annotates (entities, topics, events, engagement, etc.)
    ↓
For each user who subscribes to this source:
    ↓
Evaluate ALL priority rules against annotations
    ↓
If ANY rule matches → Mark as PRIORITY (green badge)
    ↓
Article appears in Priority tab AND in main feed
```

### Priority Rule Evaluation

Rules are evaluated as boolean conditions against ML annotations:

- **Topic priority:** `article.commonTopics[].score >= threshold AND salienceLevel == "about"`
- **Entity priority:** `article.entities[].label MATCHES configured_entity AND salienceLevel == "about"`
- **Event priority:** `article.businessEvents[].type == configured_type`
- **Like Board:** `similarity(article_embedding, board_centroid) >= threshold` (inferred)
- **Vulnerability:** `article.vulnerabilities[].cvssScore >= configured_minimum AND vendor IN configured_vendors`

### What "Salience" Means

Feedly distinguishes two salience levels:
- **"about"** — the article is primarily ABOUT this entity/topic (it's the focus)
- **"mention"** — the article merely MENTIONS this entity/topic in passing

Users can configure priorities to match only "about" level, which dramatically reduces noise. For example: an article about cloud computing that briefly mentions Microsoft won't trigger a "Microsoft" priority set to "about" salience.

### No Within-Priority Ranking

Within the priority inbox, articles are displayed **chronologically**. There is no scoring like "this priority article is more important than that one." It's binary: you asked for it, here it is.

This is a deliberate design choice — it keeps the system transparent and predictable. Users know exactly why each article was prioritized.

### How Content Gets Muted

Mute filters are the inverse of priorities. Six types:

1. **Keyword mute** — simple string matching in title/body
2. **Company mute** — AI-powered entity matching (handles aliases)
3. **Person mute** — AI-powered entity matching
4. **Topic mute** — AI-powered topic classification matching
5. **Author mute** — metadata matching
6. **Title pattern mute** — regex/pattern matching on titles only

Mute characteristics:
- Scoped per-folder or global
- Duration: 1 day, 1 week, 1 month, or forever
- Boolean OR support (mute if ANY term matches)
- 25–200 terms depending on plan tier
- AI-powered types use the same NER/topic models as priorities

### Engagement as a Non-Priority Signal

Feedly exposes engagement data but does NOT use it for automatic prioritization. The `engagementRate` field is informational — users can see that an article went viral, but it won't automatically rise to priority unless they configure a rule for it. This is philosophically opposed to social media's engagement-driven ranking.

---

## 6. Content Organization

### Hierarchy

```
Account
├── Feeds (individual RSS/Atom sources)
├── Folders (groups of feeds by topic/theme)
│   ├── Feed A
│   ├── Feed B
│   └── Feed C
├── Boards (saved articles, curated collections)
│   ├── Board: "Research"
│   ├── Board: "Share Later"
│   └── Board: "Competitors"
├── AI Feeds (virtual feeds from Feedly's 140M+ source index)
├── Newsletters (email-ingested content)
└── Team Feeds (enterprise shared feeds)
```

### Feeds

- Standard RSS/Atom subscription
- Feedly crawls on user's behalf (server-side polling)
- Polling frequency: adaptive based on source update frequency (high-volume sources polled more often)
- Users can add any URL — Feedly attempts to find RSS; if none exists, it can monitor the page
- Source limits: 100 (free), 1,000 (Pro), unlimited (Pro+/Enterprise)

### Folders

- Logical grouping of feeds
- Priority rules can be scoped to folders
- Mute filters can be scoped to folders
- "All" view aggregates across all folders

### Boards

- Manually curated collections of saved articles
- Used as input for "Like Board" priority (similarity matching)
- Can be shared (collaborative curation)
- Articles on boards are preserved indefinitely (unlike feed articles which expire)
- Boards can have tags for sub-organization

### AI Feeds

- Virtual feeds that scan Feedly's entire 140M+ source index
- User defines a topic/query; Feedly continuously finds matching articles from ANY source
- Not limited to user's subscriptions — this is the "scan the entire web" feature
- Available on Pro+ and Enterprise plans
- Limits: 100 (Market Intelligence) to 200 (Advanced)
- Use the same NLP pipeline for matching (topic models, entity models, boolean queries)

**How AI Feeds scan 140M sources (inferred):**
- Sources are pre-indexed and annotated at crawl time
- AI Feed queries are evaluated against the pre-computed annotations
- This is NOT a real-time crawl — it's a query against an existing annotated index
- New sources get added to the index continuously by Feedly's crawlers

### Newsletters

- Feedly generates a unique email address per user (e.g., `user123@feedly.com`)
- User subscribes to newsletters with that address
- Incoming emails are parsed and converted to article format
- Content appears in feeds like any RSS article
- Supports forwarding from existing inbox
- Limits: 75 (Pro+) to 200 (Enterprise)
- Same NLP pipeline applies to newsletter content (entities, topics, etc.)
- Auto-translation available for non-English newsletters

### Tags and Notes

- Articles can be tagged with custom labels
- Notes can be added to articles (highlights + annotations)
- Available on Pro and above
- Tags are searchable
- Notes support rich text


---

## 7. Deduplication and Clustering

### The Problem

A major news story (e.g., "Company X acquires Company Y") might appear across 50+ sources within hours. Without dedup/clustering, the user sees 50 near-identical articles. This is the #1 complaint about RSS readers.

### Feedly's Two-Stage Approach

**Stage 1: Deduplication (Stream Processing — Instant)**

- Algorithm: Locality-Sensitive Hashing (LSH)
- Threshold: 85% content overlap
- Processing: Real-time as articles arrive
- State: Server tracks last 10,000–20,000 articles per context (`contextId`)
- Output: `duplicates[]` array on the article object
- UX: Duplicates are hidden, shown as "X similar articles" expandable

**Stage 2: Clustering (Batch Processing — Every 15–20 minutes)**

- Algorithm: Density-based clustering (likely DBSCAN or HDBSCAN variant)
- Input: Batches of ~20,000 articles
- Complexity: Quadratic (O(n²) pairwise comparisons)
- Optimization: Dedup-first reduces effective workload to 1/5 (since duplicates inherit cluster assignment)
- Output: `clusters[]` array + `featuredMeme` (the story/narrative the cluster represents)
- UX: Clustered articles shown as a story with multiple source perspectives

### How Dedup Reduces Clustering Cost

```
20,000 articles arrive
    ↓
LSH dedup identifies 16,000 as duplicates of 4,000 unique articles
    ↓
Clustering runs on 4,000 unique articles (quadratic on 4K, not 20K)
    ↓
Duplicates inherit their parent's cluster assignment instantly
```

This is elegant engineering. The 85% LSH pass is O(n) and eliminates 80% of the work before the expensive O(n²) clustering step.

### Duplicate Propagation

When a duplicate is detected:
- It inherits all ML annotations from its "primary" (first-seen) article
- It inherits cluster assignments
- It's marked in the primary's `duplicates[]` array
- In the UI, it's collapsed under the primary
- User can expand to see all source perspectives

### Cluster Features

- **Featured Meme:** Each cluster has a `featuredMeme` — the narrative/story the cluster represents
- **Multi-source view:** Users can see the same story from different editorial perspectives
- **Time evolution:** Clusters can grow as new articles about the same story arrive
- **No cross-language clustering confirmed** — but multilingual AI suggests this may be coming

### Feedmine Implementation Notes

For a local reader processing hundreds (not millions) of articles:
- LSH dedup is still valuable even at small scale (SimHash or MinHash libraries)
- Full clustering might be overkill for <1000 articles/day — simpler cosine similarity with a threshold could work
- Consider incremental clustering: assign new articles to existing clusters if similarity > threshold, else create new cluster
- Store embeddings for all articles; run clustering periodically or on-demand

---

## 8. User Modeling

### What Feedly Learns About Users

**Explicitly captured:**
- Feed subscriptions (what sources you chose)
- Folder organization (how you categorize sources)
- Priority rules (what topics/entities/events you care about)
- Mute filters (what you want to suppress)
- Board contents (what articles you saved)
- Tags and notes (what you annotated)
- AI Feed queries (what you're actively tracking)
- Newsletter subscriptions (what emails you're ingesting)

**NOT captured (confirmed):**
- Reading time / dwell time
- Click patterns
- Scroll behavior
- Read/unread velocity
- Time-of-day patterns
- Cross-session behavioral patterns

### The "Like Board" Signal

The closest Feedly gets to implicit learning is the "Like Board" priority:
- User saves articles to a board
- System computes a representation of that board (likely centroid of article embeddings)
- New articles are compared against the board representation
- High-similarity articles get priority

This is still explicit (user chose to save those articles) but produces emergent behavior (the system infers what "similar" means without the user defining rules).

**Inferred mechanism:**
- Articles are embedded into a vector space at ingestion
- Board representation = average/centroid of member article embeddings
- Similarity = cosine similarity between new article embedding and board centroid
- Threshold for priority: likely configurable or fixed (not disclosed)

### No Collaborative Filtering

Feedly does NOT use cross-user signals:
- "Users who read X also read Y" — doesn't exist
- "Popular among users like you" — doesn't exist
- "Trending in your interest graph" — doesn't exist (beyond raw engagement data)

Each user is an island. The ML annotations are computed globally (same for all users), but the priority/mute rules are entirely individual.

### Implications for Feedmine

This is good news for a local reader:
- No need for a user base or collaborative filtering infrastructure
- The entire personalization layer is just: user rules + ML annotations
- "Like Board" can be replicated with local embeddings + cosine similarity
- All state is per-user and self-contained


---

## 9. Muting and Negative Signals

### Mute Filter Types (Detailed)

| Type | Matching | AI-Powered? | Example |
|------|----------|-------------|---------|
| **Keyword** | String match in title + body | No | "NFT", "metaverse" |
| **Company** | Entity recognition + aliases | Yes | "Meta" (catches "Facebook", "Meta Platforms Inc.") |
| **Person** | Entity recognition + aliases | Yes | "Elon Musk" (catches "Musk", "@elonmusk") |
| **Topic** | Topic classification model | Yes | "Cryptocurrency" (catches articles about crypto even without the keyword) |
| **Author** | Metadata field match | No | "John Smith" |
| **Title pattern** | Pattern/regex on title only | No | Articles with "sponsored" in title |

### AI-Powered vs Keyword Muting

The distinction matters enormously:
- **Keyword mute "Apple"** → hides articles about fruit, Apple Inc., apple pie, Apple Records
- **Company mute "Apple Inc."** → hides only articles about the tech company, using NER disambiguation

This is one of Feedly's strongest features. Topic muting via ML classification means you can mute "cryptocurrency" as a concept, not just the word — catching articles about "Bitcoin ETF approval" even if they don't contain the word "cryptocurrency."

### Mute Configuration

- **Scope:** Per-folder or global (across all feeds)
- **Duration:** 1 day, 1 week, 1 month, forever
- **Boolean logic:** OR support (mute if ANY term matches)
- **Limits by plan:**
  - Pro+: 25 mute terms
  - Enterprise: up to 200 mute terms
- **Evaluation order:** Mute filters are evaluated AFTER priority rules. If an article matches both a priority and a mute, the mute wins (muted articles never appear).

### The "Less Like This" Concept

Feedly does NOT have a "less like this" button that gradually learns to suppress content. Muting is always explicit and rule-based. This means:
- Pros: No mysterious disappearing content, fully auditable
- Cons: Users must proactively identify and configure what they don't want

### Feedmine Muting Design Implications

For a local reader:
- Keyword muting is trivial (string search)
- AI-powered muting requires NER and topic classification running locally
- The value proposition is: mute "cryptocurrency" as a CONCEPT, not just a word
- Duration-based muting is useful for ephemeral topics (e.g., "election coverage" for 1 week after an event)
- Consider adding what Feedly lacks: implicit "less like this" learning from dismiss/skip behavior

---

## 10. Enterprise Intelligence Features

### What Enterprise Reveals About Content Understanding

Feedly's enterprise products ($1,600–$3,200+/mo) are primarily sold to:
1. **Cyber Threat Intelligence (CTI) teams** — monitoring vulnerabilities, threat actors, TTPs
2. **Market Intelligence teams** — tracking competitors, trends, strategic moves
3. **Corporate security teams** — monitoring risks to employees and assets

The enterprise features reveal the depth of Feedly's content understanding because they expose capabilities hidden from consumer users.

### Threat Intelligence Specific Features

**Threat Graph:**
- Knowledge graph connecting threat actors, malware families, vulnerabilities, TTPs (MITRE ATT&CK)
- Links entities across articles to build a connected intelligence picture
- Enables queries like "show me all articles about groups using Cobalt Strike"

**Indicators of Compromise (IoCs):**
- Automatic extraction of IP addresses, domains, hashes, URLs from articles
- Structured output in STIX 2.1 format (standard for threat intel sharing)
- Exportable to SIEM/SOAR platforms (Splunk, Sentinel, etc.)

**Vulnerability Monitoring:**
- CVE extraction with CVSS scoring
- Vendor correlation (which CVEs affect which software)
- Exploit availability tracking (PoC detection)
- CVSS threshold-based alerting

**Dark Web Sources:**
- Enterprise plans include access to dark web forum content
- Feedly crawls these sources server-side
- Same NLP pipeline applies to dark web content

### Market Intelligence Features

**Business Event Detection:**
- Funding rounds (with amounts, investors)
- Acquisitions and mergers
- Product launches
- Leadership changes
- Partnerships and collaborations
- Layoffs and restructuring

**Emerging Trends Dashboard:**
- Aggregates topic velocity across sources
- Detects when a topic is growing in coverage volume
- Available on Enterprise Advanced plans

### Enterprise Workflow Features

**Team Collaboration:**
- Shared feeds and boards
- Team-wide priority configurations
- Collaborative annotation and tagging
- Shared AI Feed configurations

**Automated Newsletters:**
- Curated digests auto-generated from priority/AI feed content
- Scheduled delivery to team or stakeholders
- Template-based formatting
- Up to 20 newsletter templates (Enterprise Advanced)

**API Access:**
- Full REST API for programmatic access
- Webhook notifications for priority articles
- Integration with Slack, Teams, JIRA, etc.
- STIX 2.1 export for threat intel platforms

### Case Study Insights

From published case studies:
- **Airbus CyberSecurity:** Replaced individual information gathering with shared Feedly platform. Collaborative visibility across CTI team.
- **GreyNoise:** Doubled detection output using Feedly to find vulnerability PoCs and exploits.
- **Gematik:** Reduced threat report processing from 5–10 minutes to 2 seconds using AI translation and summarization. 70% less time on manual OSINT.
- **NXO:** SOC team launched new customer-facing service using Feedly for industry-specific risk monitoring.
- **Pharmaceutical company:** Tracks TTPs and threat actors across global sources to stay ahead of targeted attacks.

### What This Reveals for Feedmine

The enterprise features show that Feedly's content understanding goes far beyond "what is this article about":
- **Structured extraction** (entities → knowledge graph → relationships)
- **Cross-article linking** (same entity across multiple articles builds context over time)
- **Domain-specific vocabularies** (cybersecurity, finance, pharma all have specialized models)
- **Actionable output formats** (STIX 2.1, structured events, not just "here's an article")

For Feedmine, this suggests:
- Building a local knowledge graph of entities mentioned across articles over time
- Tracking entity frequency/velocity (is "Company X" appearing more often this week?)
- Structured extraction beyond basic NER (events, relationships, not just entity detection)

### Multilingual Intelligence

Feedly's multilingual capabilities (enterprise feature):
- **15 supported languages:** English, French, Spanish, German, Italian, Chinese, Arabic, Polish, Ukrainian, Vietnamese, and more
- **Single-query multilingual search:** Write a query in English, get results in all 15 languages
- **Auto-translation:** First 1,600 characters auto-translated to English; full article via Ask AI
- **Cross-language deduplication:** Same story reported in French and English gets deduplicated
- **All 30,000+ AI models work across languages** (topic classification, NER, etc.)
- **Available only on Advanced plans** (Market Intelligence Advanced, Threat Intelligence Advanced)

Technical mechanism (inferred):
- Multilingual transformer models (likely mBERT or XLM-RoBERTa variants) for cross-language understanding
- Translation likely via third-party API (Google Translate, DeepL) rather than in-house models
- Cross-language dedup likely uses multilingual embeddings for similarity comparison


---

## 11. Limitations and Weaknesses

### Confirmed Criticisms (from user forums, reviews, HN)

**1. Leo's recommendations are often poor for personal/generalist use**
- From Mac Power Users forum (Dec 2022): "I think Leo is utterly useless. The recommendations it makes are at best useless, often laughable. The summaries rarely get the gist of a message."
- Context: Leo works best for domain-specific professional use (cybersecurity, competitive intel) where the 30,000+ pre-trained models are well-tuned. For personal/hobby interests, the models may not have good coverage.
- The "non-black-box" design means if the underlying topic model is poor for your niche, the output is consistently poor — there's no behavioral fallback to learn what you actually like.

**2. No implicit learning — requires upfront configuration effort**
- Users must manually configure every priority and mute rule
- No "less like this" or "more like this" feedback loop
- No adaptation over time from reading patterns
- For users who don't want to invest time in configuration, Leo provides zero value

**3. Pricing gates ALL AI features behind Pro+ ($12.99/mo)**
- Free plan: 100 sources, 3 feeds, zero AI
- Pro ($6.99/mo): Unlimited feeds, notes, search — but NO Leo, NO AI Feeds, NO summaries
- Pro+ ($12.99/mo): First plan with ANY AI features
- Enterprise ($1,600/mo+): Full AI suite, team features, API
- There is NO middle ground — you either pay $13/mo for AI or get nothing

**4. 100% server-dependent — no offline intelligence**
- All ML processing happens server-side
- If Feedly's servers are slow/down, no AI features work
- No way to run models locally
- Privacy concern: all your articles pass through Feedly's NLP pipeline
- If Feedly shuts down or changes pricing, everything disappears

**5. Protest-tracking backlash (2023)**
- Feedly released "Protests" and "Violent Protests" AI models for enterprise
- Marketed poorly: "How to track protests posing a risk to your company's assets"
- Public backlash: users interpreted it as corporate surveillance/anti-labor tool
- Feedly CEO acknowledged "poor marketing," clarified it was for journalist safety and employee protection
- Revealed the dual-use nature of intelligence tools — same capability can serve legitimate and concerning purposes
- Some users quit Feedly over this incident

**6. Summaries are extractive, not generative (for leoSummary)**
- The automatic `leoSummary` just picks 2 salient sentences — it's not a true summary
- For actual summarization, users need "Ask AI" (Enterprise Advanced only)
- The extractive approach often "rarely gets the gist" per user feedback

**7. No social media integration**
- Feedly only reads RSS/web content and newsletters
- Cannot follow Twitter/X accounts, Reddit threads, YouTube channels directly
- For comprehensive monitoring, users need additional tools
- Enterprise plans access some social via third-party aggregation

**8. Information overload persists despite AI**
- Per AI Product Reviews: "occasional information overload" remains an issue
- Priority helps surface important articles but doesn't reduce total volume
- Users still face hundreds of unread articles in non-priority view
- No automatic "daily digest" or "top 10 for today" for individual plans

**9. Hacker News community criticism (2024)**
- Sentiment: "I just wish they'd stop trying to add AI and automated stuff" 
- Users who want a simple RSS reader feel the product has become too complex
- The $5/month simple reader doesn't exist — you get either crippled free or AI-bundled paid
- Some users moved to Miniflux, FreshRSS, or NetNewsWire for simplicity

**10. Lock-in via annotation data**
- You can export your feed list (OPML) but NOT your priority rules, board contents with annotations, or AI Feed configurations
- Years of curation work becomes platform-locked
- No standard format for "intelligence configuration"

### Structural Limitations

- **Binary priority (not graduated):** An article is either priority or not. No "high/medium/low" priority scoring. Cannot rank articles by importance within the priority set.
- **No temporal decay:** Old priority articles don't fade — they stay flagged until read/dismissed.
- **No source quality scoring:** Feedly treats all sources equally in priority evaluation. A priority match from a low-quality blog gets the same badge as one from the New York Times.
- **Limited boolean logic:** Priority rules support AND/OR but not complex nested conditions or exclusions within a single rule.
- **No cross-article intelligence for individuals:** Enterprise gets knowledge graph and trend detection. Individual users get per-article annotation only.


---

## 12. What Feedmine Can Steal from Feedly

### High-Value, Implementable Locally

**1. LSH Deduplication**
- Technique: MinHash or SimHash on article content
- Threshold: 85% (Feedly's proven threshold)
- Implementation: `datasketch` library (Python) or custom Rust implementation
- Cost: Cheap. O(n) per article against a rolling window
- Value: Extremely high. Eliminates the #1 RSS reader complaint

**2. Entity Extraction + Salience Scoring**
- Technique: spaCy NER or a fine-tuned BERT-NER model
- Salience: Heuristic — entity in title or first paragraph = "about", elsewhere = "mention"
- Disambiguation: Link to Wikidata IDs using entity linking (spaCy + Wikidata, or REL library)
- Cost: Moderate (requires model download, ~500MB). Inference is fast on CPU
- Value: Enables AI-powered muting and priority (mute "cryptocurrency" as concept, not keyword)

**3. Topic Classification with Confidence Scores**
- Technique: Zero-shot classification (BART-MNLI) or fine-tuned multi-label classifier
- Output: Topic label + score (0–1) + salience level
- User-defined taxonomy: Let users define their topic categories
- Cost: Moderate. Zero-shot is slower but needs no training data. Fine-tuned is faster but needs labels
- Value: Enables topic-based priority and muting without keyword fragility

**4. Extractive Summarization**
- Technique: TextRank, or sentence embedding + MMR (Maximal Marginal Relevance)
- Output: Top 2–3 sentences with salience scores
- Cost: Cheap. No GPU needed for TextRank
- Value: Quick scan of long articles without reading fully

**5. Engagement-Rate-Style Source Normalization**
- Technique: Track articles-per-day per source. Flag articles that get unusual external engagement (if available) or unusual reading behavior locally
- Feedmine twist: Track YOUR reading patterns per source → articles from sources you rarely read fully could be deprioritized
- Cost: Trivial (just counters)
- Value: Moderate. Helps with source quality differentiation

**6. Rule-Based Priority System**
- Technique: User configures rules (topic, entity, keyword, source) → rules evaluated against local ML annotations
- UI: "Add Priority" → select type → configure parameters → apply to folders or globally
- Cost: Trivial (just boolean evaluation)
- Value: High. Transparent, predictable, user-controlled

**7. Board-Based Similarity ("Like Board")**
- Technique: Embed articles with sentence-transformers → compute centroid of board → cosine similarity for new articles
- Model: `all-MiniLM-L6-v2` (fast, good quality, 80MB)
- Threshold: Tune empirically (start at 0.7)
- Cost: Moderate (embedding computation per article)
- Value: Enables emergent interest discovery without explicit rules

**8. Density-Based Clustering**
- Technique: HDBSCAN on article embeddings
- Run periodically (daily or on-demand) for "story" grouping
- Simpler alternative: just show articles with >60% cosine similarity as "related"
- Cost: Moderate for full clustering; cheap for pairwise similarity
- Value: "See this story from 5 perspectives" is compelling UX

**9. Mute Filters with Duration**
- Technique: Same as priorities but inverted — filter OUT matching articles
- Duration support: Attach expiry timestamp to mute rules
- AI-powered: Use NER/topic models for concept-level muting
- Cost: Same as priority evaluation
- Value: High. "Mute election coverage for 2 weeks" is powerful

**10. Full Content Fetching**
- Technique: `newspaper3k`, `readability-lxml`, or `trafilatura` for content extraction
- Go beyond RSS excerpts for better NLP processing
- Cost: Network requests + parsing. Adds latency per article
- Value: Critical for NLP quality — topic classification on a 2-sentence excerpt is unreliable

### Medium-Value, More Complex

**11. Business Event Detection**
- Technique: Few-shot classification or LLM extraction for events (funding, acquisitions, launches)
- Could use a local LLM (Llama 3, Mistral) for structured extraction
- Cost: High if using LLM inference for every article
- Optimization: Only run on articles that match certain topic classifiers first

**12. Cross-Article Entity Tracking**
- Technique: Build a local entity database. Track frequency, co-occurrence, temporal patterns
- "Entity X mentioned 3x more this week than average" → surface as trend
- Cost: Moderate (database + aggregation queries)
- Value: Brings enterprise-level intelligence to individual use

**13. Newsletter Ingestion**
- Technique: Run a local IMAP listener or use a service like Kill-the-Newsletter
- Parse HTML emails into article format
- Apply same NLP pipeline
- Cost: Moderate (email infrastructure)
- Value: Unified reading experience across RSS + newsletters

---

## 13. What Feedmine Can Do BETTER Than Feedly

### Advantages of Local-Only Architecture

**1. Implicit Learning (Feedly's Biggest Gap)**
- Feedly explicitly refuses to learn from behavior. Feedmine can.
- Track: time spent reading, articles opened vs skipped, scroll depth, read-to-completion rate
- Build a lightweight preference model that adapts over time
- Still keep explicit rules as primary — implicit as supplementary signal
- This is the single biggest opportunity to exceed Feedly's intelligence

**2. Graduated Priority Scoring (Not Binary)**
- Feedly: article is priority or not (binary)
- Feedmine: article gets a composite score from 0–100 based on multiple weighted signals
- Score = Σ(signal_weight × signal_value) across all matching rules + behavioral signals
- Show articles sorted by score, with score visible for transparency
- User can tune weights per signal type

**3. Temporal Intelligence**
- Feedly: no recency decay, no time-of-day awareness
- Feedmine: articles decay in priority over time (configurable half-life)
- Time-of-day matching: "I read tech news in the morning, politics in the evening"
- Velocity detection: "this topic is spiking today" (local computation across feeds)

**4. Source Quality Scoring**
- Feedly: all sources treated equally
- Feedmine: build per-source quality metrics over time
  - Your reading completion rate from that source
  - Average article length/depth from that source
  - How often that source's articles match your priorities
  - Source freshness (how quickly it covers stories vs others)
- Use source quality as a multiplier on priority scores

**5. Privacy**
- Feedly: all articles pass through their NLP pipeline. Your reading patterns on their servers.
- Feedmine: everything local. Zero data leaves your machine.
- No "protest tracking" controversy possible
- No pricing changes can take your intelligence away
- Your configuration and learned model are YOUR data

**6. Offline-First Intelligence**
- Feedly: no server = no AI
- Feedmine: ML models run locally. Works offline. No latency.
- Batch-process during sync, enjoy instant classification offline
- Can run on a Raspberry Pi or NAS with sufficient RAM

**7. Custom Model Training**
- Feedly: 30,000 pre-trained models, take-it-or-leave-it
- Feedmine: users can fine-tune classifiers on their own reading history
- "Articles I saved to my Research board" → train a classifier that learns YOUR research interests
- Uses local labeled data (saved vs skipped articles) as training signal

**8. Richer Muting with Learning**
- Feedly: explicit mute rules only
- Feedmine: "less like this" button that feeds into a negative preference model
- Gradual suppression rather than binary hide/show
- "I keep skipping articles from this source about this topic" → auto-suggest mute rule

**9. Flexible Scoring Architecture**
- Feedly: closed system, fixed evaluation pipeline
- Feedmine: plugin architecture for scoring signals
- Users can add custom scoring functions (Python/Lua scripts)
- Community can share scoring plugins
- "Score articles higher if they contain code blocks" — trivial custom signal

**10. Cross-Article Intelligence for Everyone**
- Feedly gates knowledge graph and trend detection behind Enterprise ($1,600/mo)
- Feedmine: local entity database + temporal tracking available to all users
- "Show me all articles about Company X over the past month" — basic query, not premium feature
- Entity relationship mapping (who's mentioned with whom) as standard feature

**11. True Daily Digest**
- Feedly: no automatic "here's your top 10 for today"
- Feedmine: generate a daily/weekly digest based on composite scoring
- "Show me the 15 most important articles from yesterday" with explanations of WHY each scored high
- Configurable digest frequency and size

**12. Reading Mode Intelligence**
- Feedly: same presentation regardless of context
- Feedmine: adapt to reading mode
  - "Quick scan mode" — show only titles + 1-sentence summary for low-priority articles
  - "Deep read mode" — full content with highlights for high-priority articles
  - "Catch-up mode" — only show priority articles from the last 3 days


---

## 14. Open Questions

### Technical Unknowns

1. **Exact model architectures** — Feedly has never disclosed whether they use BERT, DistilBERT, RoBERTa, or custom architectures. The 30,000+ models are likely fine-tuned classifiers sharing a common backbone, but this is inference.

2. **"Like Board" similarity computation** — Confirmed to exist as a priority type, but the embedding model, similarity metric, and threshold are undisclosed. Likely sentence-transformer style embeddings with cosine similarity.

3. **Clustering algorithm specifics** — Confirmed density-based with batch processing every 15–20 min on 20K article batches. But is it DBSCAN? HDBSCAN? Custom? What distance metric? What epsilon/min_samples?

4. **Embedding dimensions** — What size vectors does Feedly use for article representations? 384? 768? 1024?

5. **Model retraining cadence** — Community feedback improves models, but how often are models retrained? Daily? Weekly? Monthly?

6. **Infrastructure** — Cloud provider, GPU fleet size, serving architecture for 30,000+ models. Are models served individually or as a multi-task model?

7. **AI Feed source scanning** — How does querying 140M sources work? Is it a pre-computed annotation index (most likely) or on-demand crawling? What's the freshness guarantee?

8. **Feed polling optimization** — How does Feedly decide polling frequency per source? Adaptive based on update frequency? PubSubHubbub/WebSub? What's the maximum freshness delay?

9. **Dark web access** — Enterprise plans include dark web sources. How? Tor crawling? Third-party dark web aggregators? What sources specifically?

10. **Cross-language dedup** — Multilingual AI deduplicates across languages. Does this use multilingual embeddings (mBERT/XLM-R) for cross-language similarity, or translation-then-dedup?

### Strategic Questions for Feedmine

1. **How much local ML is "enough"?** — Feedly runs 30,000+ models. A local reader can't. What's the minimum viable set? (Likely: NER + 1 topic classifier + embeddings for similarity)

2. **CPU vs GPU tradeoff** — Can meaningful NLP run on CPU-only for a desktop app? (Yes, with DistilBERT or ONNX-optimized models, but with latency)

3. **When to involve an LLM vs lighter models?** — Feedly uses LLMs only for on-demand Ask AI, not for classification. This suggests lighter models are sufficient for annotation tasks.

4. **How to handle the cold-start problem?** — Feedly's models are pre-trained on millions of articles. A local reader starts with zero context. Pre-trained open-source models help, but domain-specific fine-tuning needs local data.

5. **Dedup window size locally** — Feedly tracks 10–20K articles for dedup context. For a personal reader with 100–500 articles/day, what window is optimal?

---

## Appendix A: Competitive Positioning

### Feedly vs Key Competitors

| Feature | Feedly Pro+ | Inoreader Pro | NewsBlur Premium | Reeder (Mac) | NetNewsWire |
|---------|-------------|---------------|-----------------|--------------|-------------|
| Price | $12.99/mo | ~$7/mo | $36/year | $5 one-time | Free |
| AI Classification | Yes (30K models) | Rules only | Basic intelligence | None | None |
| Deduplication | ML-powered (85% LSH) | Rule-based | Basic | None | None |
| NER | Yes + disambiguation | No | No | No | No |
| Topic Models | Yes (30K+) | No | No | No | No |
| Mute Filters | AI-powered (6 types) | Keyword only | Keyword training | None | None |
| Priority/Ranking | Rule-based + ML | Rules + filters | Trained classifier | None (chronological) | None (chronological) |
| Newsletters | Yes (email ingestion) | Yes | No | No | No |
| Full Content Fetch | Yes | Yes | Yes | Yes (via service) | No |
| API | Enterprise only | Yes (all plans) | Yes | N/A (local) | N/A (local) |
| Self-hosted | No | No | Yes (open source) | N/A (local) | N/A (local) |
| Open Source | No | No | Yes | No | Yes |

### Key Differentiators

- **Feedly's moat:** The 30,000+ pre-trained topic/entity models and the 140M source index. No competitor has this depth of content understanding.
- **Inoreader's strength:** More accessible power-user features (rules, filters, monitoring) at lower price. No AI, but excellent automation.
- **NewsBlur's strength:** Open source, self-hostable, has a basic "intelligence trainer" (thumbs up/down that trains a per-feed classifier). Closest OSS competitor on the ML front.
- **NetNewsWire's strength:** Free, native Mac/iOS, fast, no tracking. The "anti-Feedly" — pure RSS reading with zero intelligence.
- **Reeder's strength:** Best reading experience / UI design. Connects to multiple backends (Feedly, Inoreader, iCloud, etc.).

### Open Source Alternatives for Intelligence Features

| Feedly Feature | Open Source Equivalent | Notes |
|---------------|----------------------|-------|
| RSS aggregation | Miniflux, FreshRSS, Tiny Tiny RSS | Mature, well-maintained |
| AI Smart Feeds | **Glean** (new, 2026) | Self-hosted RSS with semantic search + smart feeds via embeddings |
| NER | spaCy, Stanza, Flair | Production-ready, CPU-capable |
| Topic classification | HuggingFace zero-shot, SetFit | No labeled data needed for zero-shot |
| Embeddings/similarity | sentence-transformers (`all-MiniLM-L6-v2`) | 80MB model, fast on CPU |
| Deduplication | datasketch (MinHash), simhash | Well-tested libraries |
| Clustering | HDBSCAN, scikit-learn DBSCAN | Standard implementations |
| Summarization | sumy (TextRank), BART-based | TextRank needs no GPU |
| Full content extraction | trafilatura, newspaper3k, readability-lxml | High quality extraction |
| Newsletter parsing | Kill the Newsletter (OSS) | Converts email newsletters to RSS |
| LLM summaries | Ollama + Llama 3 / Mistral | Local LLM for on-demand queries |

**Glean** (discovered during research) is particularly relevant:
- Free, open source, self-hosted RSS reader
- Supports AI embeddings for semantic search
- "Smart Feeds" that surface relevant content using embeddings
- OPML import from Feedly
- Available on Elestio for managed hosting
- Still early-stage but validates the approach Feedmine is taking

---

## Appendix B: Pricing Implications for the Market

### What Feedly's Pricing Reveals

```
Free:       $0    → RSS reader, no intelligence (100 sources, 3 feeds)
Pro:        $7/mo → RSS reader, power features, no intelligence  
Pro+:       $13/mo → Intelligence (Leo, AI Feeds, summaries)
Enterprise: $1,600/mo → Full intelligence + collaboration + API
```

**Key insight:** There is a $0 → $13/mo gap for AI features. No $5/mo "lite AI" tier exists. This suggests:
1. AI features are expensive to run server-side (GPU inference costs)
2. Feedly decided the intelligence layer is premium-only to fund model development
3. The consumer market may not value AI enough to pay $13/mo (hence Feedly's enterprise pivot)

**Market gap Feedmine fills:** Local AI = zero marginal cost per article. The intelligence layer costs nothing once models are downloaded. This fundamentally changes the economics — AI features can be free if they run locally.

### Enterprise Pricing Context

- Market Intelligence: $1,600/mo (10 seats, 100 AI feeds)
- Market Intelligence Advanced: $3,200/mo (25 seats, 200 AI feeds, Ask AI, API)
- Threat Intelligence: similar tiers
- Per-seat model for teams

The enterprise price reflects: dedicated source crawling, team collaboration infrastructure, API access, support SLAs, and the value of the content (CTI teams make security decisions worth millions based on this intel).

---

## Appendix C: Feedly's Evolution Timeline

| Year | Milestone |
|------|-----------|
| 2008 | Feedly founded (originally DevHD) |
| 2013 | Google Reader shuts down. Feedly goes from 4M to 15M users in weeks |
| 2014 | Pro plan introduced ($5/mo) |
| 2018 | Leo AI introduced (NER skill first). Pivot toward intelligence |
| 2019 | Leo 0.5, 0.6 — topic priorities, cross-feed priorities |
| 2020 | COVID-19 models. Cybersecurity focus intensifies |
| 2021 | Enterprise product launched. Business events, vulnerability tracking |
| 2022 | AI Feeds (140M source scanning). "Like Board" priority |
| 2023 | Protest-tracking backlash. 30,000+ AI models |
| 2024 | Ask AI (LLM integration). Rejected RAG for context-stuffing |
| 2025 | Multilingual AI (15 languages). Auto-translation. Newsletter templates |
| 2026 | Cyberattack Intel Agent. Threat Actor Insights. Continued enterprise focus |

---

*End of document. Last updated: July 2026.*
*Research sources: Feedly developer docs (ArticleJSON spec), Feedly engineering blog, Feedly changelog, case studies, PCMag, Hacker News, Mac Power Users forum, third-party reviews, Elestio/Glean documentation.*
