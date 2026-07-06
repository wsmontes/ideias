# Apple News Intelligence: Complete Reference for Building a Privacy-First Feed Reader

> **Document purpose**: Comprehensive knowledge base on Apple News's architecture, algorithms, editorial process, and privacy approach — written for someone building Feedmine, a local-only feed reader that wants to replicate Apple News's best ideas.
>
> **Methodology**: Information sourced from Apple's official documentation, Apple ML Research papers, WWDC sessions, academic audits (Northwestern/CJR 2019), publisher-facing platforms (FlatPlan), industry reports (Enders Analysis, Nieman Lab, Digiday), and Apple Support pages. Each claim is marked as **[CONFIRMED]** (directly from Apple sources) or **[INFERRED]** (deduced from observable behavior, publisher reports, or academic research).
>
> **Last updated**: July 2026

---

## 1. Executive Summary

Apple News is a news aggregation platform available on iOS, iPadOS, and macOS in four countries (US, UK, Canada, Australia) with 125M+ monthly active users. It is unique among major news platforms for combining three elements that no competitor replicates together:

1. **Human editorial curation** — A team of ~30 former journalists hand-selects Top Stories, acting as a quality gate that overrides algorithmic ranking
2. **On-device personalization** — Reading preferences are processed locally using a separate identifier isolated from the user's Apple Account; Apple claims it cannot see individual reading histories
3. **Algorithmic recommendation** — For You feeds, Topic feeds, and Trending Stories use machine learning to match content to user interests, but always below the editorial layer in feed hierarchy

The result is an architecture where **editorial judgment sets the ceiling** (what everyone sees at the top) and **algorithmic personalization fills the floor** (individual interest matching below). Privacy is enforced structurally — not as a policy choice but as an architectural constraint that makes certain recommendation approaches (collaborative filtering, cross-service data fusion) impossible by design.

**Apple News+** adds a subscription layer ($9.99/mo US) with 400+ magazines/newspapers, professionally narrated audio stories, and deeper content access. Revenue is split 50/50 with publishers based on engagement share.

**Key insight for Feedmine**: Apple News proves that a hybrid editorial + algorithmic approach with strict privacy constraints can work at massive scale. The trade-off (shallower personalization vs. complete privacy) is compensated by editorial quality and explicit user signals (follows, likes, blocks).

---

## 2. The Hybrid Architecture

### Feed Structure (Today Tab)

**[CONFIRMED]** from academic audit + Apple documentation:

The Today feed presents content in a strict hierarchical order:

| Position | Section | Curation Method | Personalized? | Universal? |
|----------|---------|----------------|---------------|------------|
| 1 (top) | Top Stories | Human editors | No | Yes — same for all users |
| 2 | Trending Stories | Algorithm candidates → editor final selection | No | Yes — same for all users |
| 3 | Topic Feeds | Algorithm based on inferred interests | Yes | No |
| 4 | For You | Fully algorithmic, personal | Yes | No |
| 5 | More For You | Same algorithm, lower position | Yes | No |
| 6 | Followed Channels | Explicit subscriptions | Yes (by definition) | No |

**Critical design principle**: The algorithm CANNOT promote content above the editorial layer. Everyone sees the same Top Stories regardless of their interests. Personalization only operates in lower-priority positions.

### Three Tabs

- **Today**: The main feed with the interleaved structure above
- **News+** (subscribers): Magazine/newspaper content, Apple News+ Food, deep features
- **My Sports**: Dedicated sports ecosystem with team/league/tournament-level following
- **Audio**: Daily briefing + narrated stories (added 2020)

### The Interaction Model

**[CONFIRMED]** from FlatPlan/publisher documentation:

- Top Stories: Updated 5+ times/day, average story duration 7.2 hours, ~20.4 new stories/day
- Trending Stories: Updated every ~20 minutes (min 6, max 61 min intervals), average story duration 2.9 hours, ~50.7 new stories/day
- Topic Feeds: Small groups of stories matching user's inferred interests
- For You: Unique per reader, based on search history, Siri history, followed publications, reading history
- Followed Channels: Dedicated sections for publications the user explicitly follows

---

## 3. Signal Taxonomy

### Signals Apple News Uses

#### Explicit Signals (User-Initiated)

| Signal | What It Does | Feedmine Equivalent |
|--------|-------------|-------------------|
| Follow Channel | Subscribe to a publication → dedicated feed section + algorithm boost | Subscribe to feed |
| Follow Topic | Declare interest in a category | Follow tag/category |
| Like (thumbs up) | "Suggest More" — positive reinforcement | Upvote/star article |
| Dislike (thumbs down) | "Suggest Less" — negative signal | Downvote/hide |
| Block Channel | Never show content from this source | Block feed source |
| Block Topic | Suppress a category | Block tag |
| Save | Bookmark for later — also positive engagement signal | Save/bookmark |
| Share | Distribute via Messages/Mail/social — strong engagement signal | Share action |
| Search | In-app search reveals interests | Search history |

#### Implicit Signals (Behavioral)

| Signal | What It Does | Feedmine Equivalent |
|--------|-------------|-------------------|
| Reading time / Active time | Key algorithm signal — longer = better quality indicator | Track scroll depth + time-on-article |
| Articles read | Builds topic interest profile | Reading history |
| Siri search history | Reveals interests across the system | N/A (local search history) |
| Safari browsing | Contributes to Siri Suggestions | Browser history integration (opt-in) |
| App usage patterns | "Information contributed by other installed apps" | N/A |
| Scroll past without clicking | Negative implicit signal (headline seen but not engaged) | Impression without click |
| Notification tap-through | Validates notification relevance | N/A |

#### Population-Level Signals (Server-Side)

| Signal | What It Does | Feedmine Equivalent |
|--------|-------------|-------------------|
| Social sharing velocity | Powers Trending Stories detection | External share counts (RSS) |
| Aggregate engagement | Helps rank within Trending | Hacker News-style decay scoring |
| Source authority per topic | Trusted publishers surface first for their specialty | Source credibility scores |
| Reader retention (aggregate) | Quality proxy across all readers | Community average read time |

### Privacy Architecture of Signals

**[CONFIRMED]**:
- A **separate identifier** "specific to News and Stocks" is used — completely isolated from Apple Account
- Reading history stays on device: "Your News reading history for personalized newsletters is kept on your device"
- Cross-device sync via iCloud syncs explicit signals (follows, saved stories, reading history) but reconstructs personalization on each device
- Location data controllable separately for News (Settings > Privacy > Location Services > News)
- Personalized ads can be disabled independently

**[INFERRED]**:
- The on-device component maintains a lightweight interest profile (topic weights) updated by reading behavior
- Server-side pre-classifies articles with topics/metadata; device-side matches against interest profile
- No collaborative filtering is possible (no server-side user profiles to compare)

---

## 4. Content Understanding

### Apple News Format (ANF)

**[CONFIRMED]** from Apple Developer documentation:

ANF is a custom JSON-based format describing article structure, layout, and metadata. It's the key to Apple News's content understanding.

**Structure:**
- Article metadata: title, subtitle, author, publish date, keywords, topics, sections
- Component tree: text (body, heading, quote), images (photo, gallery, mosaic), audio, video, embeds
- Layout specifications: columns, margins, gutters — auto-adapted for iPhone/iPad/Mac
- Styling: fonts, colors, text effects — all declarative
- Interactive components: animations, parallax, 3D-like photo effects

**What ANF reveals about ranking signals:**
- **Topics/categories in metadata** → direct input to topic classification
- **Section assignment** → where it appears within the publisher's channel hierarchy
- **Keywords** → additional semantic signals for matching
- **Article length** → "Longer articles give the Apple News algorithm more context to work with, so where possible create stories above 750 words" [CONFIRMED from FlatPlan]
- **Image quality** → thumbnails affect click-through; Apple penalizes clickbait imagery
- **AI-generated content flag** → must be declared via metadata property [CONFIRMED]

### Topic Classification System

**[CONFIRMED]** topics include: Technology, Science, Travel, Sports, Entertainment, Movies, Music, Politics, Business, Health, Food, Lifestyle, Fashion, Arts, Education, Environment, Opinion

**[CONFIRMED]** from publisher documentation:
- Publishers organize channels into **sections** by topic/tag
- RSS feeds require: title, language (with locale), link per channel; title, link, description per item
- ANF articles include structured metadata with explicit topic/category declarations

**[INFERRED]** — Topic assignment likely uses:
1. Publisher-declared metadata in ANF (primary)
2. Section assignments within publisher's channel
3. NLP analysis of article text (server-side, using Apple's Natural Language framework)
4. Reading behavior patterns (cross-user topic co-occurrence for new articles)

### Content Authority Model

**[CONFIRMED]** from FlatPlan:
- "Publications whose authority on the topic is strong within Apple News" get priority in Trending
- "It's easier for the algorithm to understand the content of a channel when lots of stories are available" — volume contributes to authority
- Reader retention and active time are "strong indicators to the Apple algorithm"

**[INFERRED]**: Apple maintains a **source × topic authority matrix** where each publisher has a score per topic area, built from:
- Volume of content in that topic
- Reader retention metrics for topic-specific articles
- Engagement rates (likes, shares, saves)
- Editorial selection history
- Duration of publishing on the platform

### Entity Extraction and NLP

**[CONFIRMED]** from Apple ML Research:
- Natural Language Framework (iOS 12+): On-device APIs for tokenization, language identification, named entity recognition, sentiment analysis, text classification
- Core ML: Can run custom NLP models optimized for Apple Silicon Neural Engine
- Foundation Models Framework (iOS 26+): ~3B parameter on-device LLM with text understanding, summarization, structured output

**[INFERRED]**: For News specifically, NLP likely operates server-side at article ingestion to:
- Extract named entities (people, organizations, locations, events)
- Assign topic classifications
- Generate embeddings for similarity matching (recirculation)
- Detect duplicate/related stories across publishers

---

## 5. On-Device Personalization

### What Actually Runs on the Phone

**[CONFIRMED]** from Apple Support + ML Research:

Apple uses "local, on-device processing" for Siri Suggestions in News, Spotlight, Look Up, and Photos Memories. The system ingests signals from:
- Safari browsing history
- Emails and messages
- Contacts
- Information contributed by other installed apps
- In-app reading behavior (News-specific)

**The on-device ML infrastructure available:**

| Framework | Since | Capabilities | Likely News Role |
|-----------|-------|-------------|-----------------|
| Natural Language | iOS 12 (2018) | Tokenization, NER, sentiment, text classification, language ID | Could classify articles locally |
| Core ML | iOS 11 (2017) | Run trained models on Neural Engine | Could run interest matching model |
| Create ML | 2018 | Train custom classifiers on-device | N/A for News |
| Foundation Models | iOS 26 (2025) | ~3B param LLM, summarization, structured output | Could power article summaries |

### The Personalization Model Architecture

**[INFERRED]** — based on all available evidence:

Apple's on-device News personalization is NOT a complex neural recommendation model. It's a **lightweight interest profile** constructed from:

1. **Topic weight vector**: Each topic (Technology, Sports, Politics, etc.) has a weight based on reading frequency + recency + explicit follows
2. **Source affinity scores**: Followed channels get maximum affinity; frequently-read sources get elevated scores
3. **Negative signals**: Blocked channels/topics get zero weight; disliked content reduces related topic weights
4. **Recency decay**: Recent reading behavior weighted more heavily than older patterns

**How personalization works:**
1. Server classifies all articles at ingestion (topic, quality, freshness, source authority)
2. Server sends article metadata to device (NOT full articles — just enough for matching)
3. Device-side model scores articles against local interest profile
4. For You feed displays highest-scoring articles
5. Topic feeds display clusters of articles matching strong interest signals

**Evidence this is lightweight (not a full neural model):**
- Cross-device sync works by syncing raw signals, then each device reconstructs personalization independently
- The system works on older devices (no Neural Engine requirement for basic News)
- Apple states personalization improves "the more you read" — suggesting simple accumulation, not complex model training
- No evidence of on-device model update or retraining mechanisms

### Privacy Architecture

**[CONFIRMED]**:
- Separate identifier for News/Stocks — NOT linked to Apple Account
- No server-side reading profile maintained by Apple
- Reading history for newsletters kept on device only
- iCloud syncs: followed channels/topics, saved stories, reading history, blocked channels
- What does NOT sync: model weights, inferred interest scores, behavioral patterns

### Apple's Federated Learning System (General)

**[CONFIRMED]** from MIT Technology Review + Apple ML Research (2019-2024):

Apple has a production federated learning system used for:
- Siri speaker recognition (Hey Siri personalization)
- QuickType keyboard suggestions
- Found In Apps feature
- Combined with differential privacy since iOS 13

**How it works:**
1. Each device trains a local model copy on local data
2. Only model updates (gradients) are sent to server — never raw data
3. Differential privacy noise is injected before transmission
4. Server aggregates updates across many devices into improved master model
5. Updated master model is pushed back to devices

**[INFERRED]** for News: There is NO direct evidence that Apple uses federated learning for News recommendation. The system architecture (separate identifier, no collaborative filtering) suggests the News personalization model does NOT participate in federated learning. Instead, it appears to be purely local with no model improvement feedback loop to Apple's servers.

### Cold Start Problem

**[CONFIRMED]** from user-facing behavior:
- New users are prompted to select favorite topics and news sources during onboarding
- "When users open the app, they are prompted to select their favorite topics and news sources"
- The app presents a grid of topics (Technology, Sports, Entertainment, etc.) and suggested publications
- Before any personalization kicks in, all users see the universal Top Stories + Trending (editorial/algorithmic)

**[INFERRED]** cold start mitigation:
1. **Onboarding topic selection** provides initial interest vector
2. **Universal editorial content** (Top Stories) ensures value from day one regardless of personalization
3. **Trending Stories** (same for all users) provides engaging content without personal data
4. **Cross-device iCloud sync** means existing Apple News users on new devices get immediate personalization from synced follows/history
5. **Siri/Safari signals** may bootstrap interest profile from existing device behavior (if the user has history in other apps)

---

## 6. Editorial Curation

### Team Structure

**[CONFIRMED]** from NYT, Apple Newsroom, CJR:

- ~30 former journalists based in Sydney, London, New York, and Silicon Valley
- Led by Lauren Kern (former New York Magazine executive editor, as of 2018-2019 reporting)
- Team fields 100-200 pitches from publishers daily
- Described as having "extensive journalistic experience"

### What Editors Do

**[CONFIRMED]**:

1. **Select Top Stories**: ~5 leading stories at any time, updated 5+ times/day
2. **Curate Spotlight features**: Magazine-style feature stories
3. **Select Trending Stories**: Algorithm identifies candidates from trusted sources; editors make final selection of 6 stories
4. **Curate local news**: Editors in specific cities curate local collections
5. **Send notifications**: Editors can push notifications about Top Stories, Sports, Puzzles
6. **Weekly "Read Local" collection**: Every Thursday, curated best stories from local publications

### Editorial Values

**[CONFIRMED]** from academic audit + Apple statements:

- **Accuracy over speed**: "Prioritizes accuracy over speed" — editors deliberately avoid unverified claims
- **Anti-sensationalism**: Tim Cook: combating "the craziness of digital news" and avoiding "content that strictly has the goal of enraging people"
- **Source diversity**: Editorial selections show Shannon equitability of 0.780 vs 0.689 for algorithmic (more evenly distributed)
- **Substance over celebrity**: Top Stories headlines reflect policy, international affairs, investigations; Trending favors celebrities, entertainment
- **International breadth**: Editors surface significantly more international news than algorithms

### How Editorial Blends with Algorithm

**The hierarchy is absolute**: Editorial selections (Top Stories) ALWAYS appear above algorithmic content. The algorithm cannot override editorial judgment.

**Trending Stories is hybrid**:
1. Algorithm detects stories being shared/engaged with on social networks and internet broadly
2. Filters to "trusted publications whose authority on the topic is strong within Apple News"
3. Presents candidates to editorial team
4. Editors select final 6 stories to display

This means even "algorithmic" content passes through a human filter via source authority and editorial final selection.



---

## 7. Quality and Credibility

### Platform-Level Quality Gate

**[CONFIRMED]** from Apple's publishing guidelines:

Apple News is explicitly a "distribution platform for professional publications of a journalistic nature." The following are EXCLUDED:

- Publications not editorial/journalistic in nature
- Personal blogs or portfolios
- Publications based outside US/UK/CA/AU
- Publications primarily promoting a business
- Publications primarily reposting/redistributing others' content
- **Publications publishing factual inaccuracies or failing to adhere to widely accepted journalistic standards**

This gatekeeping requirement means Apple News operates with a **curated publisher universe** — unlike Google News or RSS readers where anyone can appear.

### Quality Signals in Ranking

**[CONFIRMED]**:
1. **ANF requirement for editorial features**: Only ANF articles can be featured in Top Stories/Spotlight — creating a quality tier (publishers investing in format = higher quality signal)
2. **Reader retention / active time**: "Reader retention and active reader time are strong indicators to the Apple algorithm" [FlatPlan]
3. **Source authority per topic**: Trusted publications with demonstrated authority surface first
4. **Notification quality enforcement**: Publishers violating guidelines (clickbait, questionable sourcing) face "temporary or permanent revocation" of notification privileges
5. **AI content labeling**: AI-generated articles MUST be labeled; using AI to mislead = channel suspension

**[INFERRED]**:
6. **Block rate as negative signal**: High block rates from users likely suppress a source's algorithmic presence
7. **Engagement quality vs quantity**: Apple's emphasis on "active time" over raw clicks suggests depth > breadth
8. **Editorial selection history**: Sources that editors frequently choose likely get authority boosts

### AI-Generated Content Policy

**[CONFIRMED]**:
- AI-generated/assisted articles MUST be labeled (byline/co-byline + metadata property)
- "Using AI to mislead readers is not allowed and could lead to your channel's suspension"
- Publishers "responsible for the accuracy of all content published to Apple News"
- Must "correct any errors in a timely and transparent manner"
- AI can be used as a tool without labeling ONLY if "the article was composed — and all information vetted — by a journalist"

### Source Authority Model

**[INFERRED]** from observable behavior:

Apple appears to maintain a **trust score per publisher** that factors in:
- Length of time on platform
- Volume of quality content
- Reader engagement metrics (retention, shares, saves)
- Editorial selection frequency
- Adherence to content guidelines
- Notification behavior (responsible use = maintained trust)
- Whether content is in ANF (vs. just RSS)

This creates a **two-tier system**:
- **Tier 1 (ANF publishers)**: Eligible for editorial features, Trending selection, full analytics, notification privileges
- **Tier 2 (RSS-only publishers)**: Appear in search and For You but never in editorial sections; rendered in web view rather than native

---

## 8. User Modeling

### On-Device Interest Profile

**[INFERRED]** — Apple does not publish model architecture, but observable behavior reveals:

The user model is a **multi-dimensional interest profile** maintained locally:

```
UserProfile {
  topic_weights: Map<Topic, Float>     // Technology: 0.8, Sports: 0.3, etc.
  source_affinities: Map<Source, Float> // NYT: 0.9, ESPN: 0.6, etc.
  blocked_sources: Set<Source>          // Hard filter, never shown
  blocked_topics: Set<Topic>           // Hard filter for topic
  followed_sources: Set<Source>        // Explicit subscription
  followed_topics: Set<Topic>          // Explicit interest declaration
  reading_history: List<ArticleRef>    // Recent articles read
  saved_articles: List<ArticleRef>     // Bookmarked for later
  search_history: List<String>         // Recent in-app searches
}
```

### How Weights Update

**[INFERRED]** from behavior patterns:

- **Reading an article**: Increases weights for that article's topics + source affinity
- **Reading time**: Longer reading = stronger positive signal (vs. quick bounce)
- **Following a channel**: Sets source affinity to maximum; ensures dedicated feed section
- **Following a topic**: Sets topic weight to maximum
- **Like/Suggest More**: Strong positive boost to topic + source
- **Dislike/Suggest Less**: Reduces topic weight; may reduce source affinity
- **Block**: Permanent exclusion (output filter, doesn't affect model otherwise)
- **Sharing**: Strong positive signal (user found it worthy of distribution)
- **Saving**: Moderate positive signal (intent to read later = interest)
- **Recency decay**: Older signals weighted less — interest profile evolves over time

### Blocking Mechanics

**[INFERRED]** from academic audit observations:
- Blocking appears to be **output filtering** — blocked sources simply don't appear in feed
- Blocking does NOT appear to affect the underlying interest model (blocking Fox News doesn't reduce interest in Politics topic — it just hides Fox News)
- Topic blocking is broader — it suppresses all content tagged with that topic regardless of source

### What Gets Synced via iCloud

**[CONFIRMED]**:
- Followed channels and topics ✓
- Saved stories ✓
- Reading history ✓
- Blocked channels ✓

**[INFERRED]** — What does NOT sync:
- Computed topic weights ✗
- Source affinity scores ✗
- Model state ✗
- These are reconstructed on each device from synced raw signals

---

## 9. Freshness and Breaking News

### Time-Sensitivity Architecture

**[CONFIRMED]** from publisher guidelines + audit data:

| Content Type | Update Frequency | Freshness Priority |
|-------------|-----------------|-------------------|
| Top Stories | 5+ times/day, punctuated schedule (morning, midday, afternoon, evening) | High — reflects current news cycle |
| Trending Stories | Every ~20 minutes (6-61 min range) | Very high — reflects real-time engagement |
| For You | Continuous | Medium — balances recency with interest match |
| Topic Feeds | Periodic | Medium — topical relevance over strict recency |
| Widget | Auto-updates | Very high — "what's happening now" |

### Breaking News Handling

**[CONFIRMED]** from Apple's notification guidelines:

**Philosophy**: "It's better to be right than to be first"

**Notification limits**:
- Mixed breaking/nonbreaking: Up to 3/day, hard cap 5
- Nonbreaking only: Up to 2/day
- Time zone targeting available (e.g., East Coast storm → Eastern time only)

**Content requirements**:
- "Don't send notifications with questionable sourcing or incomplete information"
- "During a breaking or developing news story, wait until there's substantial information"
- Can publish brief article first and update as story develops
- Headline copy: 10-95 characters

**Editorial response**:
- Apple News editors can send their own notifications about Top Stories
- Top Stories updates more frequently during major breaking events
- Example: Jacksonville shooting — team avoided publishing false claims circulating elsewhere

### Trending Detection

**[INFERRED]**:
- Trending likely uses **engagement velocity** (rate of change in sharing/reading) rather than absolute popularity
- The ~20-minute update cycle enables near-real-time trend detection
- Input signals: social network sharing, internet-wide engagement, Siri search queries
- Stories must come from "trusted publications" — prevents viral misinformation from surfacing
- Algorithm identifies candidates → editors make final selection (hybrid approach)

### Freshness Signals for Feedmine

Lessons from Apple's approach:
1. **Decay function**: Stories in feed should lose position over time (7.2 hour average in Top Stories suggests significant decay)
2. **Velocity matters more than volume**: A story getting 100 shares in 10 minutes > 1000 shares over a day
3. **Breaking news override**: When breaking news is detected, normal ranking can be temporarily overridden
4. **Accuracy > speed**: Better to surface a story 30 minutes late with confirmation than immediately with uncertainty

---

## 10. The Widget and Proactive Surfacing

### Widget Architecture

**[CONFIRMED]** from Apple Support + WWDC sessions:

**Available surfaces**:
- iPhone lock screen
- iPhone/iPad home screen
- Mac desktop
- Notification Center / Today View
- CarPlay (audio content)

**Widget sizes and content**:
- Small: Single headline
- Medium: Two headlines
- Large: Four headlines

**Content hierarchy in widget**:
1. Top Stories (editor-curated) — always highest priority
2. "Missed This?" (editor-curated catch-up stories)
3. For You (personalized recommendations)
4. Topic-specific (if user configured widget for specific topic)

**[CONFIRMED]**: "The widget updates automatically, pulling in the latest articles from the sources you follow in the News app, as well as top stories selected by Apple News editors"

### Proactive Intelligence Integration

**[CONFIRMED]** from WWDC20 "Design for Intelligence" session:

Apple's Proactive Intelligence system treats News as part of a broader "intelligence" layer:
- Siri Suggestions can surface News articles on lock screen
- Spotlight search returns News articles
- Smart Stack widget automatically positions News widget "at just the right time"
- The system uses on-device signals (time of day, usage patterns, location) to decide WHEN to surface News content

**Key quote from Apple**: "The goal of intelligence is to make your Apple products feel like they know you — your goals, intentions, habits, preferences, interests, and relationships"

### Widget Recommendation Logic

**[INFERRED]**:
- Priority: Breaking/Top Stories (editorial) → Topic-configured stories → Personalized For You
- Recency heavily weighted (widget = "what's happening now")
- Followed sources get priority in rotation
- Widget uses the SAME on-device interest model as the app — no separate engine
- Smart Stack rotation considers: time of day, reading habits (morning commuter vs. evening reader)

### Integration with Broader Apple Intelligence

**[CONFIRMED]** from Apple documentation/WWDC:
- Siri Suggestions in News powered by same on-device ML that drives Spotlight, Photos, etc.
- Apple Intelligence (2024+) brings ~3B param on-device LLM — potentially usable for article summarization in widget
- App Intents framework lets News expose capabilities to Siri AI (WWDC26)
- "Personalized Collections" concept from App Store (WWDC26) shows Apple's expanding on-device recommendation to more surfaces

**[INFERRED]**: Future integration likely includes:
- Siri AI summarizing news articles on demand
- Apple Intelligence generating article digests for widgets
- Cross-app intelligence (Calendar event → relevant News articles about that topic)



---

## 11. Publisher Ecosystem

### Content Ingestion Paths

**[CONFIRMED]**:

| Path | Method | Quality Tier | Editorial Eligible? | Native Rendering? |
|------|--------|-------------|--------------------|--------------------|
| RSS/Atom | AppleBot crawls feeds | Basic | No | No (web view) |
| Apple News Format | JSON via API or News Publisher | Rich | Yes | Yes |
| Apple News API | Programmatic REST API from CMS | Rich | Yes | Yes |

**RSS requirements**: Channel must include title, language (with locale), link. Items must include title, link, description.

**ANF capabilities**: Rich layouts, animations, inline components, VoiceOver accessibility, advertising support, structured metadata (topics, keywords, sections).

### Publisher Analytics (News Publisher Dashboard)

**[CONFIRMED]** — Full metrics available:

**Channel-level**:
- Unique viewers, Total views, Average active time
- Shares (by platform: Mail, Messages, Facebook, Twitter — hidden if <6)
- Reach (users who saw headlines in feeds OR viewed articles)
- Likes, Saves, Follows (new + net)
- Demographics (gender, age distribution)
- Video metrics (views, median view time, minutes, completion rate)
- **Discovery Source** (Today feed, topic feeds, Stocks app, widgets)
- Views by device (iPhone, iPad, Mac)
- Subscriptions (unique viewers with subscription + top 10 articles driving subs)
- Loyalty (MAU by articles viewed + days active)
- Notifications (audience, CTR per notification)
- WAU and MAU (CSV exports)

**Article-level**: Same as channel, scoped per article, plus video retention rate and subscription conversion rate.

**Key insight**: Discovery Source data reveals algorithm contribution — publishers can see what % of views came from each surface.

### Revenue Model

**[CONFIRMED]**:

**Apple News+ subscription revenue**:
- Apple takes 50%
- Remaining 50% distributed to publishers proportionate to engagement share (time spent / clicks)
- UK Apple News+ generates >£100M/year with ~1.7M subscribers

**Advertising revenue**:
- Direct-sold ads: Publisher keeps 100%
- Apple-sold backfill ads: 50/50 split with publisher (ANF only)
- Apple started selling own ad inventory directly (November 2024)
- Some reports: publishers get 70% of ad revenue in their articles

**News Partner Program**:
- 15% commission (vs standard 30%) on qualifying in-app purchase subscriptions from day one
- Requirements: robust Apple News channel in AU/CA/UK/US, all content in ANF

### Publisher Optimization Strategies

**[CONFIRMED]** from FlatPlan (Apple's preferred integration partner):

**Headlines**: Keep to 80-110 characters (aim for ~90) to avoid truncation across all Apple News surfaces.

**Standfirsts**: 130-305 characters. Space to tease readers.

**Article length**: "Longer articles give the Apple News algorithm more context to work with, so where possible create stories above 750 words"

**Publishing cadence**: "We always recommend that publishers deliver all content instead of cherry-picking stories. It's easier for the algorithm to understand the content of a channel when lots of stories are available"

**Publishing timing**: "Posting during the afternoon for UK readers allows US readers to pick up the stories in the morning during their commute"

**Thumbnail optimization**: Critical for click-through — bold, eye-catching images

**Driving Follows**: Inline Follow CTAs within articles boost algorithmic weight. "Followed channels will be seen much more regularly in personalised areas of the app and will even get whole sections dedicated to them in the Today Feed." Publishers using FlatPlan's Follow CTAs see 30-40% increase in followers within 3 months.

**Promote feature**: Push up to 6 articles per section to top position for 48 hours. Best used on articles with high Average Active Time.

**Category accuracy**: Ensure all content is accurately categorized — directly impacts topic feed surfacing.

### Audio Content

**[CONFIRMED]** from Apple Newsroom (July 2020):

**Apple News Today** (daily briefing):
- Free to all listeners
- Available mornings Monday through Friday
- Hosted by Apple News editors (Shumita Basu, Duarte Geraldino)
- Available in News app and Apple Podcasts
- Guides listeners through fascinating stories + how journalists cover them

**Apple News+ Audio Stories** (narrated features):
- ~20 audio stories produced per week
- Narrated by professional voice actors (NOT text-to-speech)
- Audio versions of best feature reporting/long-form from: Esquire, Essence, Fast Company, GQ, New York magazine, Sports Illustrated, TIME, Vanity Fair, Vogue, Wired, LA Times, WSJ
- Apple News+ subscribers only (US)

**Audio Tab**: Dedicated tab at bottom of News app with queue management and personalized recommendations.

**CarPlay integration**: Listen to audio stories while driving; sync listening progress across devices.

**Apple News+ Narrated** (podcast feed): Available on Apple Podcasts for subscribers — "Hear articles from your favorite magazines and newspapers narrated by professional voice actors and produced by Apple News"

---

## 12. Privacy-Preserving Recommendation Techniques

### Apple's Privacy Stack (General)

**[CONFIRMED]** from Apple ML Research + MIT Technology Review:

| Technique | How It Works | Apple Uses For |
|-----------|-------------|---------------|
| Differential Privacy (Local) | Adds calibrated noise to data BEFORE it leaves device | Safari domains, keyboard usage, emoji, health data |
| Federated Learning | Trains model copies on-device; sends only gradients to server | Siri speaker recognition, QuickType keyboard, Found In Apps |
| Federated Evaluation | Tests model variants on-device; reports only aggregate performance | On-device ML tuning and personalization |
| Separate Identifiers | Isolated IDs per service — can't be linked | News/Stocks use ID separate from Apple Account |
| On-Device Processing | ML inference runs locally, never sends data | News personalization, Siri Suggestions, Photos |
| Private Cloud Compute | Server-side inference with no data retention + verifiable privacy | Apple Intelligence server model (2024+) |

### How Apple's Approach Compares to Federated Learning for Recommendations

**[CONFIRMED]** from Apple's federated learning paper (2021/2024):

Apple's federated system "was originally created to support two specific federated tasks: evaluation and tuning of on-device ML systems, primarily for the purpose of personalizing these systems. In recent years, support for an additional federated task has been added: federated learning (FL) of deep neural networks."

**Applied to**: Siri, keyboard, speaker recognition
**NOT confirmed for**: News recommendation

**Key architectural difference**:
- **Federated learning** (what Apple uses for Siri): Devices collaboratively improve a shared model. Requires sending model updates to server.
- **Apple News approach** (purely local): Each device maintains its own interest profile. NO model updates sent to server. NO collaborative improvement across users.

**Why Apple News likely doesn't use federated learning**:
1. The separate News identifier suggests intentional isolation
2. Content-based filtering (topic matching) doesn't benefit much from collaborative learning
3. The explicit signals (Follow, Like) provide enough personalization without collaborative data
4. Simpler architecture = easier to audit for privacy compliance
5. No evidence of model weight transmission in News-related network traffic

### Differential Privacy in News (Specifically)

**[INFERRED]** — NOT directly confirmed for News:

Apple likely uses differential privacy for population-level signals:
- **Trending detection**: Aggregate reading/sharing patterns across devices → trend signals with DP noise
- **Popular topics**: What topics are trending today → population-level signal with privacy guarantees
- **Engagement velocity**: Rate of reads/shares for new articles → can be computed via DP aggregates

**The evidence**: Apple's 2025 DP paper describes "randomized response" where "hundreds of people using the same term are needed before the word can be discoverable." This technique could apply to trending topic detection in News.

**What's confirmed**: Apple uses DP for "Safari popular domains, keyboard usage patterns, photo attributes, health data, emoji usage" — but News is not explicitly listed.

### Privacy-Preserving Recommendation: Academic Context

**From research literature** (not Apple-specific):

The cold-start problem in privacy-preserving systems is addressed through:
1. **Content-based filtering** (what Apple uses): Relies on item features, not user-user similarity. Works without any cross-user data sharing. Trade-off: no serendipity from "users like you" signals.
2. **Hybrid approaches**: Use DP-protected population signals for new users, transition to personalized model as local data accumulates.
3. **Federated recommendation** (what Apple does NOT use for News): Multiple research papers (arxiv 2021-2026) propose federated recommender systems, but these still require gradient sharing with server.

**Apple's unique position**: By choosing PURELY local personalization (no federated learning, no collaborative filtering), Apple achieves the strongest possible privacy guarantee — at the cost of the shallowest possible personalization. They compensate with editorial curation and explicit signals.



---

## 13. Limitations and Criticism

### Structural Limitations of Apple's Approach

| Privacy Gain | Capability Cost |
|---|---|
| No server-side profiles | No collaborative filtering ("users like you also read...") |
| Separate identifier | No cross-service enrichment (Music taste ≠ News taste even though they're related) |
| On-device processing | Limited model complexity (must fit on older devices) |
| No user tracking | Can't measure true engagement quality at population scale |
| No advertiser access to data | Limited ad targeting = lower publisher revenue |

### Publisher Criticism

**[CONFIRMED]** from Nieman Lab, CJR, Digiday, industry reports:

1. **Revenue share opacity**: "Apple News is fairly opaque" — publishers struggle to understand how the 50% subscription revenue pool is divided. The engagement-proportionate model means small publishers get tiny slices even with loyal audiences.

2. **50/50 split viewed as excessive**: Apple takes 50% of subscription revenue while publishers provide ALL the content. Compare to Substack (10%) or Patreon (5-12%). Multiple publishers have publicly criticized this split.

3. **Algorithmic opacity**: Unlike Google (which provides Search Console), Apple provides limited insight into WHY specific articles surface or don't. Publishers cannot see their "authority score" or understand ranking decisions.

4. **ANF as barrier to entry**: The proprietary format requirement for editorial consideration creates a two-tier system. Local/small publishers using only RSS are permanently excluded from Top Stories, Trending, and the highest-value feed positions. CJR found The Oregonian's story was passed over in favor of The Hill's coverage — because The Hill published in ANF.

5. **Local news exclusion**: CJR audit (2019) found "not a single locally or regionally specific source was cited" in Trending Stories. Top Stories included local sources for only 8.3% of articles, half from LA Times. Apple has since expanded local coverage but it remains editor-dependent.

6. **Editorial bias accusations**: US Senator Marsha Blackburn (2026) accused Apple of "systematically suppressing news articles from right-leaning publications while amplifying articles from left-leaning publications." The academic audit found Fox News dominating Trending but appearing far less in editorially-curated Top Stories.

7. **AI content licensing concerns**: Apple in talks with news organizations for AI development (2026). Publishers express concerns about "content control" and "impact on media independence and market monopolies."

8. **No customer data sharing**: Apple does not share subscriber data with publishers. Publishers cannot build direct relationships with readers found through Apple News.

9. **Revenue model favors engagement over quality**: Time-based revenue allocation incentivizes long-form content and engagement tricks rather than concise, high-quality journalism.

10. **Geographic limitations**: Available only in 4 countries. Publishers in other markets are excluded entirely.

### Comparison to Artifact (RIP January 2024)

**[CONFIRMED]** from TechCrunch, Ideaproof, Wired:

Artifact (by Instagram co-founders Kevin Systrom & Mike Krieger) took the opposite approach from Apple News:

| Dimension | Apple News | Artifact |
|-----------|-----------|---------|
| Personalization | On-device, shallow, privacy-first | Server-side, deep, engagement-maximizing |
| Editorial | 30-person team, human curation | None — pure algorithm |
| Model | Content-based filtering | Collaborative + behavioral (TikTok-for-text) |
| Privacy | Extreme — no server profiles | Standard — full behavioral tracking |
| AI usage | Minimal (classification) | Heavy (summaries, rewrite clickbait, image gen) |
| Content source | Publisher-submitted (ANF/RSS) | Crawled from web |
| Revenue model | Subscription + ads | None (pre-revenue) |
| Social features | None | Comments, follows, curation |
| Users | 125M+ MAU | ~444K total downloads |
| Outcome | Dominant in 4 markets | Shut down after 1 year |

**Why Artifact failed where Apple News succeeded**:
- Artifact couldn't compete with built-in newsreaders (Apple News, Google News)
- 44% of downloads were US-only — no international traction
- Feature scope crept (links, text posts, places, AI images) diluting core value
- "Like Apple News but worse" (Daring Fireball) — no differentiation for users who already had a news app pre-installed
- Market opportunity "wasn't large enough to warrant continued investment" (Systrom)

**Lesson for Feedmine**: The standalone news reader app market is brutal. Apple News wins by being pre-installed + having editorial quality. Artifact proved that even deep AI personalization isn't enough without distribution advantage. Feedmine's open-source/local-only positioning must offer something NEITHER provides.

### What Doesn't Work Well in Apple News

**[INFERRED]** from user reports + academic research:

1. **Limited personalization depth**: Users report seeing repetitive content from same sources; no "users like you" serendipity
2. **Slow interest evolution**: Model appears to lag behind changing interests; users report seeing old-interest content weeks after they've moved on
3. **Filter bubble via explicit follows**: If users only follow specific channels, they may miss important stories from other sources
4. **Widget staleness**: Widget sometimes shows hours-old stories; refresh timing isn't always optimal
5. **Sports bias in My Sports**: Extensive categorization for sports but nothing equivalent for other passionate verticals (music genres, programming languages, etc.)
6. **No customizable feed layout**: Users can't reorder sections, hide Trending, or customize the feed hierarchy
7. **Notification overload from publishers**: Some publishers push notifications aggressively within the limits

---

## 14. What Feedmine Can Steal from Apple News

### 1. The Hierarchy Principle: Editorial > Algorithm

**The technique**: Never let algorithm-ranked content appear above curated/pinned content. Give users the ability to pin their own "editorial" selections.

**Implementation**: Feedmine should support a "pinned sources" tier that always appears at top, regardless of algorithmic scoring. This mimics Apple's Top Stories position for user-defined critical sources.

### 2. On-Device Interest Profile

**The technique**: Build and maintain the entire user model locally. Never send reading behavior to a server.

**Implementation**:
```
TopicWeightVector {
  weights: Map<String, Float>  // topic → weight (0.0-1.0)
  update(article_topics: List<String>, read_time_seconds: Int) {
    for topic in article_topics:
      weights[topic] += signal_strength(read_time_seconds) * LEARNING_RATE
      weights[topic] = decay(weights[topic], days_since_last_update)
  }
}
```

Key parameters to tune:
- **Learning rate**: How quickly new reading affects weights
- **Decay factor**: How quickly old interests fade
- **Minimum threshold**: Below which a topic weight is effectively zero
- **Read time → signal strength mapping**: Convert seconds to strength (diminishing returns after ~5 min)

### 3. Explicit Signals as First-Class Citizens

**The technique**: Follow, Like, Block, Save are MORE important than implicit signals. Apple compensates for limited personalization depth by leaning heavily on explicit user choices.

**Implementation**: Explicit signals should immediately and dramatically affect feed composition:
- Follow = topic/source weight set to maximum
- Block = hard filter, never show
- Like = 5x weight of a normal read
- Save = 3x weight of a normal read

### 4. Source Authority Scoring

**The technique**: Not all sources are equal. Build per-source, per-topic authority scores.

**Implementation**:
```
SourceAuthority {
  source_id: String
  topic_scores: Map<Topic, Float>  // how authoritative for each topic
  
  // Factors:
  // - Consistent publishing in topic (volume over time)
  // - User engagement with that source's topic content (read time)
  // - Recency of publishing (active sources > dormant)
  // - User-configured trust levels
}
```

### 5. The Cold Start Solution

**The technique**: Onboarding topic selection + universal quality content + immediate value without personalization.

**Implementation**:
1. First launch: present topic grid for user to select interests
2. Before personalization kicks in: show chronological feed from subscribed sources (guaranteed relevant)
3. Universal "highlights" section: high-quality content curated by community or algorithms
4. Optionally: import OPML with existing subscriptions to bootstrap immediately

### 6. Time-Decay Freshness

**The technique**: Stories lose position over time. Apple's 7.2-hour average duration for Top Stories and ~3-hour for Trending suggest aggressive decay.

**Implementation**: Score = relevance_score * freshness_multiplier(age_hours)
- freshness_multiplier(0) = 1.0
- freshness_multiplier(4) = 0.7
- freshness_multiplier(12) = 0.3
- freshness_multiplier(24) = 0.1

### 7. Reading Time as Quality Proxy

**The technique**: Articles with longer average read time = higher quality signal.

**Implementation**: Track scroll position + time on each article. Compute "engagement score":
- < 10 seconds: bounce (negative signal)
- 10-30 seconds: skim (weak positive)
- 30-120 seconds: read (positive)
- 120+ seconds: deep read (strong positive)

Use this to weight future recommendations from same source/topic.

### 8. Topic Feeds as Discovery Mechanism

**The technique**: Cluster articles by topic and surface topic-grouped recommendations between main feed items.

**Implementation**: After showing primary feed (chronological from subscriptions), insert "Topic Digest" blocks:
- "3 more articles about [your high-weight topic]"
- Pull from sources user hasn't subscribed to but that have high authority in that topic

### 9. Breaking News Detection via Velocity

**The technique**: Detect stories gaining rapid engagement across multiple sources.

**Implementation**: For Feedmine (without Apple's population signals):
- Detect when multiple subscribed sources publish about the same entity/event within a short window
- Detect sudden increase in publishing frequency from a source
- Surface these as "Breaking" or "Developing" stories above normal feed

### 10. Widget/Glance Optimization

**The technique**: Proactive surfacing of 1-4 stories based on time-of-day + user patterns.

**Implementation**: If Feedmine has a widget/notification digest:
- Morning: Top story from highest-priority sources
- Midday: Something from user's strongest interest topic
- Evening: Long-form content for deeper reading
- Breaking: Override schedule for high-velocity stories



---

## 15. What Feedmine Can Do Better Than Apple News

### 1. Full Transparency

Apple News is a black box. Feedmine can be completely transparent:
- Show WHY each article appears in the feed (topic match score, source authority, freshness)
- Let users inspect and edit their interest profile directly
- Open-source the ranking algorithm — users can audit, modify, fork
- No "algorithmic opacity" complaints possible when users can read the code

### 2. User-Controlled Algorithm

Apple gives users Follow/Block/Like — that's it. Feedmine can offer:
- **Weight sliders**: Manually adjust topic weights (more politics, less entertainment)
- **Source priority levels**: "Always show first" / "Show when relevant" / "Only if nothing else" / "Never"
- **Feed modes**: Chronological, Relevance-ranked, Topic-clustered, Source-grouped — user's choice
- **Decay tuning**: Users choose how quickly old interests fade
- **Diversity dials**: "Show me things outside my bubble" slider (opposite of filter bubble)

### 3. No Walled Garden

Apple News requires publishers to submit content in ANF. Feedmine:
- Works with ANY RSS/Atom feed — no publisher opt-in required
- Can pull from personal blogs, academic journals, niche publications
- No geographic restrictions (Apple: US/UK/CA/AU only)
- No "approved publisher" gatekeeping
- Users curate their own source universe

### 4. No Revenue Misalignment

Apple's 50/50 revenue split creates incentives to surface engaging (not necessarily best) content. Feedmine:
- No advertising incentives
- No engagement metrics sold to third parties
- Ranking optimized purely for user satisfaction
- No pressure to keep users in-app longer than they want

### 5. Configurable "Editorial" Layer

Apple has 30 editors. Feedmine can offer:
- **Community curation**: Users can publish their "editorial picks" — others can subscribe to curators
- **Algorithmic "editors"**: Rules-based curation (e.g., "show me the most-shared article from each of my 20 sources daily")
- **Self-curation**: Users can pin specific articles to their own "Top Stories" section
- **Multiple editorial sources**: Subscribe to different curators for different topics

### 6. Better Local News

Apple's local news is limited to ~11 US cities, editor-dependent. Feedmine:
- Any local RSS feed works (local newspapers, community blogs, city subreddits)
- Location-based feed filtering without Apple's geographic restrictions
- Users define their own "local" — could be a neighborhood, a city, a region
- No dependency on publishers adopting a proprietary format

### 7. Cross-Platform / No Lock-In

Apple News only works on Apple devices. Feedmine:
- Works on any platform (web, Android, Linux, Windows)
- Data export at any time (OPML, JSON)
- No vendor lock-in — switch apps, keep your data
- Self-hostable for maximum control

### 8. Granular Content Understanding

Apple classifies into broad topics (Technology, Sports, etc.). Feedmine can offer:
- User-defined taxonomy (custom tags, hierarchical categories)
- On-device NLP for article classification (using open models like sentence-transformers)
- Entity-level tracking (follow "Rust programming language" not just "Technology")
- Custom keyword alerts and filters

### 9. Reading History as Knowledge Base

Apple treats reading history as a signal for future recommendations. Feedmine can make it a product feature:
- Searchable archive of everything you've read
- "What did I read about X last month?" queries
- Spaced repetition for important articles (resurface key stories after N days)
- Reading statistics and patterns (self-knowledge)

### 10. Composable / Extensible

Apple News is take-it-or-leave-it. Feedmine:
- Plugin system for custom scoring functions
- API for integration with other tools (Obsidian, Notion, Readwise)
- Scriptable feed rules (IFTTT-style automation)
- Federated architecture potential (share curations without sharing reading data)

---

## 16. Open Questions

### About Apple News (Unresolved)

1. **Exact on-device model architecture**: What model type powers the interest profile? Simple topic weight vector vs. embedding space vs. neural model? What is its size in memory/storage?

2. **Complete topic taxonomy**: No full published list of all Apple News topics exists. How many? What's the hierarchy? Are there hidden sub-topics?

3. **Differential privacy specifically for News**: No direct confirmation. Is DP used for Trending detection? For any News-specific aggregate signals?

4. **Recirculation algorithm**: How are "related stories from other publications" selected below articles? Entity matching? Topic similarity? Collaborative signals?

5. **Apple Intelligence integration**: Has the Foundation Models framework been applied to News? Article summarization in widgets? Better topic classification?

6. **On-device model update mechanism**: How does the personalization model refresh? Online learning per reading event? Periodic batch recalculation? On what schedule?

7. **Engagement velocity calculation**: Does Trending use rate-of-change or absolute numbers? What's the detection threshold?

8. **Deduplication approach**: How does Apple handle multiple publishers covering the same story? Is there active deduplication or just source-authority-based selection?

9. **Private Cloud Compute for News**: Will future News recommendation use Apple's privacy-preserving server inference (stateless, verifiable, no data retention)?

10. **Audio story selection**: How does Apple choose which articles become narrated audio stories? Editorial selection? Engagement metrics? Publisher nomination?

### For Feedmine Design (To Be Answered)

1. **How much personalization is enough?** Apple's shallow personalization + editorial works at scale. Does Feedmine need deeper ML or would explicit signals + simple scoring suffice?

2. **Community vs. individual curation**: Should Feedmine support shared curation lists? This creates a "distributed editorial" layer without Apple's overhead.

3. **Source credibility without Apple's gatekeeping**: How to assess source quality in an open ecosystem where anyone's RSS feed is valid?

4. **On-device NLP models**: Which open-source models can run on-device for topic classification? MiniLM? DistilBERT? What's the size/performance trade-off?

5. **Cold start without Apple's distribution**: Apple pre-installs on 1B+ devices. How does Feedmine onboard users without that advantage?

6. **Breaking news without population signals**: How to detect breaking stories from individual subscription feeds alone (no aggregate engagement data)?

7. **The "editorial" problem at zero scale**: How to provide editorial-quality curation without a 30-person team? Community curation? Algorithmic quality scoring? Both?

8. **Engagement measurement without surveillance**: How to measure article quality (read time, engagement) while respecting user choice about what's tracked?

---

## Appendix A: Key Sources

| Source | Type | What It Reveals |
|--------|------|----------------|
| Apple Support pages | Official | Privacy architecture, sync behavior, settings |
| Apple Developer docs (ANF) | Official | Content format, metadata, API capabilities |
| Apple ML Research papers | Official | DP framework, federated learning, foundation models |
| Apple Newsroom announcements | Official | Feature launches, editorial philosophy, team structure |
| WWDC sessions (2019-2026) | Official | Natural Language framework, Core ML, Foundation Models, proactive intelligence |
| FlatPlan articles/guides | Publisher partner | Algorithm behavior, optimization tips, publisher metrics |
| CJR/Northwestern audit (2019) | Academic | Source concentration, personalization testing, editorial vs algorithmic comparison |
| Enders Analysis UK report (2026) | Industry | Revenue data, subscriber counts, market position |
| Nieman Lab reporting | Industry | Publisher criticism, revenue analysis |
| Digiday reporting | Industry | Revenue model details, publisher strategies |
| MIT Technology Review (2019) | Journalism | Federated learning for Siri, DP implementation |
| TechCrunch/Wired (Artifact) | Journalism | Competitor comparison, market dynamics |

## Appendix B: Apple News Timeline

| Year | Event |
|------|-------|
| 2015 | Apple News launches (replacing Newsstand) |
| 2018 | Editorial team structure revealed (NYT profile); Natural Language framework launches (iOS 12) |
| 2019 | Apple News+ subscription launches ($9.99/mo); CJR audit published; 85M MAU reported |
| 2020 | Audio features launch (Apple News Today daily briefing + narrated stories); Local news in 5 cities; CarPlay support; 125M+ MAU reported |
| 2021 | Local news expands to 11 US cities; "Read Local" weekly collection begins |
| 2022 | My Sports tab launches (50%+ unique views increase for sports publishers) |
| 2024 | Apple Intelligence announced (on-device ~3B param LLM); Apple starts selling own ad inventory; Artifact shuts down (Jan) |
| 2025 | Apple News+ Food section (iOS 18.4); Foundation Models framework (iOS 26); Apple Intelligence Foundation Language Models tech report; DP paper for Apple Intelligence |
| 2026 | WWDC26: Siri AI, Foundation Models framework expanded, Personalized Collections in App Store; UK Enders Analysis report on Apple News; Political bias accusations; AI licensing negotiations with publishers |

## Appendix C: Feedmine Design Decisions Informed by Apple News

| Apple News Pattern | Feedmine Implication | Priority |
|---|---|---|
| Editorial hierarchy overrides algorithm | Implement pinned/priority sources tier | High |
| On-device interest profile | Build local topic weight vector with decay | High |
| Explicit signals > implicit | Make Follow/Block/Like the primary ranking input | High |
| Source authority scoring | Per-source quality/trust scores (user-configurable) | Medium |
| Onboarding topic selection | First-run topic picker to bootstrap cold start | High |
| Time-decay freshness | Implement exponential decay on story age | High |
| Read time as quality proxy | Track engagement depth per article | Medium |
| Breaking via velocity | Multi-source same-entity detection | Medium |
| ANF metadata for classification | Parse RSS categories/tags + on-device NLP | Medium |
| Audio stories (narrated) | TTS integration for long-form articles | Low |
| Widget/digest proactive surfacing | Time-of-day based notification digest | Low |
| Community curation (Apple doesn't do this) | Shared curator subscriptions — Feedmine advantage | Medium |
| Configurable algorithm (Apple doesn't do this) | User-facing weight sliders and mode switches | High |

---

*End of document. This is a living reference — update as Apple reveals more about their systems or as Feedmine's design decisions evolve.*
