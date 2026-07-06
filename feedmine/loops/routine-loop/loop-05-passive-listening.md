# Loop 05: Passive Listening — How Devices Collect, Filter, and Understand User Data

> **Purpose:** Understanding how Apple and Android devices passively collect data about user behavior, how that raw data is filtered into meaningful signals, and how those signals become understanding. Focus on the MECHANISMS, not the morals. The goal: learn how to passively understand a user's routine and needs from behavioral data.
> **Scope:** The complete passive data pipeline — from hardware sensors and always-on processors through on-device ML classification to behavioral understanding — across Apple and Android platforms. Extracted for what Feedmine can learn about building passive routine awareness from reading behavior alone.
> **Date:** 2026-07-06 | **Sources:** Apple ML Research, Apple Developer Documentation (Core Motion, SensorKit), Apple Newsroom (Privacy), Google Developer Documentation (Activity Recognition, Transition API), Google Research Blog (Now Playing), Android Authority (Audio Memory leak), TechCrunch, medrxiv (SensorKit study 2025), PCMag, Lifehacker, security.org.

---

## 1. The Three Layers of Passive Understanding

Every modern device — Apple or Android — runs the same fundamental pipeline for understanding its user. The pipeline has three layers, each transforming chaos into clarity:

### Layer 1: RAW COLLECTION

Sensors, microphones, GPS, accelerometers, usage logs capture data continuously. This is the firehose. Thousands of data points per second. Accelerometer readings. GPS coordinates. Screen touches. App opens. Notification arrivals. Audio samples. Light levels. Barometric pressure.

Raw data is meaningless by itself. A stream of accelerometer values tells you nothing. A GPS coordinate is just a number. A timestamp of an app launch is noise without context.

**The raw layer exists to be consumed by the filter layer — never to be stored or transmitted as-is.**

### Layer 2: FILTERING

On-device ML converts raw data into classifications. This is where noise becomes signal.

- Accelerometer readings → "walking" / "driving" / "still"
- Audio samples → "wake word detected" / "silence"
- GPS coordinates → "at home" / "at work" / "commuting"
- App launch timestamps → "morning routine" / "evening wind-down"
- Touch patterns → "actively engaged" / "passively scrolling"

The filter layer is the intelligence. It doesn't store raw data — it produces LABELS. Classifications. Categories. The filter decides what matters and discards everything else.

### Layer 3: UNDERSTANDING

Filtered signals combine over time to build a model of the user's routine, preferences, and needs.

- "Walking every morning at 7am" + "opens podcast app during walks" → suggest podcast at 6:55am
- "At work" + "opens news app at lunch" → surface news digest at 11:50am
- "Driving" + "connected to car Bluetooth" → suggest navigation to calendar's next event

Understanding is temporal. It requires HISTORY of filtered signals. A single classification ("walking") is a filter output. Weeks of classifications ("walks every morning at 7am except weekends") is understanding.

### For Feedmine: The Same Three Layers

| Layer | Device Example | Feedmine Equivalent |
|-------|---------------|-------------------|
| Raw Collection | Accelerometer values, GPS coords, audio samples | Timestamps, scroll events, taps, article opens, time-on-screen |
| Filtering | "Walking" / "Driving" / "Still" | "Read-to-completion" / "Skimmed" / "Abandoned" / "Deep read" |
| Understanding | "Exercises at 7am, commutes at 8am, reads at lunch" | "Reads AI news mornings, long-form evenings, skips politics weekends" |

The critical insight: **every passive understanding system follows this pipeline.** The differences between Apple and Google aren't in the pipeline itself — they're in WHERE the understanding lives (device vs. cloud) and WHO it serves (user vs. advertiser).

---


## 2. Apple — Collection Mechanisms

Apple's collection architecture is defined by a single constraint: **everything stays on-device.** This isn't a policy — it's an engineering decision baked into silicon. The hardware is DESIGNED to process locally.

### 2.1 Hardware Sensors (Always-On)

**Motion Coprocessor (M-series chip):**
The M-series motion coprocessor is always on, always collecting. It tracks accelerometer, gyroscope, magnetometer, and barometer data continuously — without ever waking the main CPU. Power draw is negligible. The coprocessor runs independently, accumulating motion data 24/7 whether the phone is "awake" or not.

This is the foundation of all physical activity understanding on iOS. Steps counted. Flights climbed. Activity type detected. All from a chip that never sleeps and never asks permission to observe.

**Always-On Audio DSP:**
A dedicated low-power chip continuously samples background audio. Its ONLY purpose: wake word detection. The architecture is deliberately limited:

1. Audio captured by microphone
2. Converted to spectrogram (frequency representation)
3. Fed to small neural network on dedicated DSP
4. Network trained to detect ONE specific sibilant frequency pattern: "Hey Siri"
5. If match: wake main processor. If no match: discard immediately.

No other audio is analyzed. No audio is stored. No audio leaves the chip. The orange dot indicator appears when the main microphone activates — the always-on DSP operates below this threshold by design.

**Additional always-on sensors:**
- Ambient light sensor: screen brightness adaptation, True Tone
- Proximity sensor: screen on/off during calls, face detection
- Barometer: floor counting, weather-related altitude changes
- LiDAR (Pro models): spatial awareness, depth mapping

**Apple Watch sensor suite:**
- Optical heart rate sensor (continuous background sampling)
- Blood oxygen sensor (SpO2)
- ECG electrodes (on-demand but data stored persistently)
- Skin temperature sensor (baseline tracking overnight)
- Accelerometer + gyroscope (fall detection, wrist raise, activity classification)

The Watch represents Apple's most aggressive passive collection — it touches your BODY. Heart rate variability, respiratory rate (derived), sleep stages, skin temperature trends. All processed on-wrist, synced to iPhone, never transmitted to Apple.

### 2.2 Software Passive Collection

**Screen Time (iOS 12+):**
Records continuously without user interaction:
- Every app launch and session duration
- Device pickups (how often you reach for the phone)
- Notification counts per app
- Category breakdown: Social Networking, Productivity, Entertainment, Reading & Reference, Games, etc.
- First pickup time each day
- Longest session duration

All local. Apple cannot see this data. It exists solely to serve the user's own awareness — and to power Siri Suggestions about app usage patterns.

**Significant Locations:**
On-device, end-to-end encrypted history of frequently visited places. The system observes WHERE you go, HOW LONG you stay, and HOW OFTEN you return. Used for:
- Maps traffic predictions (knows your commute)
- Photos Memories (location-based grouping)
- Calendar travel time estimates
- Siri Suggestions (location-triggered)

Never accessible to Apple. Never transmitted. Encrypted with device passcode. Exists in Settings → Privacy → Location Services → System Services → Significant Locations.

**Core Motion API (CMMotionActivityManager):**
Automatically detects activity type without any app running:
- Stationary
- Walking
- Running
- Cycling
- Driving (automotive)

Maintains 7 days of history accessible to authorized apps. Each reading includes a CONFIDENCE level: low, medium, or high. The API doesn't expose raw sensor data — only the classification result.

**Apple SensorKit (Research API):**
Available only to approved research studies (requires IRB approval + Apple approval):
- Ambient light levels over time
- Keyboard metrics: typing speed, typing patterns, error rate
- Phone usage: call duration, frequency (not content)
- Device usage: screen on/off events, unlock frequency
- Messages usage: message frequency, conversation count (never content)
- Visits data: location visits with duration

The 2025 medrxiv study used SensorKit data from accelerometers alone to train a foundation model predicting diverse health targets — demonstrating that MOTION DATA is sufficient for significant health inference.

**App Privacy Report (iOS 15.2+):**
Logs every time any app accesses:
- Location
- Camera
- Microphone
- Contacts
- Photos

Shows network activity per app. Not collection FOR intelligence — but collection ABOUT collection. Meta-observation.

---


### 2.3 How Apple Filters Raw Data

The genius of Apple's architecture: raw data is consumed by filters and NEVER surfaces. You cannot access the raw accelerometer stream that Core Motion uses. You get the CLASSIFICATION. The filter is the interface.

**Hey Siri Pipeline:**
```
Always-on processor → audio sample → spectrogram conversion → neural network inference → binary decision (wake word / not wake word)
```
Everything before the binary decision is DISCARDED. No buffer. No history. No recording. The neural network is tiny — trained to recognize one specific acoustic pattern. When the main Siri activates (orange dot), that's a DIFFERENT system. The always-on DSP is below the level of "microphone active."

**Core Motion Filtering:**
```
Raw accelerometer + gyroscope + magnetometer → motion coprocessor → activity classification
```
The motion coprocessor classifies activity types in real-time. Apps receive: `{ activity: "walking", confidence: .high, startDate: ... }`. They never receive: `{ x: 0.023, y: -0.981, z: 0.012, timestamp: ... }`. The RAW data is consumed by the filter and destroyed.

**Significant Locations:**
GPS coordinates → frequency analysis → "significant" locations identified → encrypted on-device storage. The system knows "home" and "work" not because you told it — because you STAY there. Duration + frequency = significance. Raw GPS history is not retained — only the significant conclusions.

**Neural Engine Processing:**
Apple's Neural Engine handles:
- Wake word detection
- Activity recognition
- Photo classification (faces, objects, scenes)
- Text prediction
- Natural language understanding

All without cloud. The Neural Engine is a dedicated ML accelerator — 16+ cores in recent chips — designed explicitly for on-device inference. The architectural message: intelligence doesn't require transmission.

**Differential Privacy:**
When Apple DOES need aggregate data (emoji usage frequency, Safari crash rates, health trends), it applies differential privacy: mathematical noise injected into individual data points before they leave the device. The aggregate is statistically useful. The individual is mathematically unrecoverable. This is filtering at the transmission layer — even the rare data that leaves is filtered first.

### 2.4 How Apple Builds Understanding

Filtered signals accumulate into understanding over time. This is where ROUTINE emerges.

**Siri Suggestions — The Understanding Engine:**

Siri Suggestions observes patterns across multiple filtered signals simultaneously:
- Time of day (when you use which app)
- Location (where you use which app)
- Frequency (how often you use which app)
- Accessories (what's connected when you use which app)
- Calendar (what events correlate with which app usage)

From these signals, it builds predictions:
- "User opens Podcast app when AirPods connect at 7:15am on weekdays"
- "User opens News app at 12:00pm at work location"
- "User opens Fitness app at 5:30pm on M/W/F"

**Proactive Features (the output of understanding):**
- News app appears in Siri Suggestions at morning reading time
- Fitness app surfaces before the user's regular workout
- Music app suggested when headphones connect
- Navigation to work appears at commute time
- "Do Not Disturb" suggested at bedtime

**The understanding NEVER LEAVES the device.** There is no server-side model of you. No cloud profile. Apple's business model doesn't require understanding you — it requires SELLING you hardware. The intelligence is a feature of the product, not extraction from the user.

**Apple ML Research (2025) — Foundation Model from Accelerometer Data:**
A landmark paper demonstrated that a foundation model trained on accelerometer data alone — from Apple Watch and iPhone motion coprocessors — could predict a wide variety of health outcomes. The model learned representations of human movement that generalized across tasks: sleep quality, cardiovascular fitness, fall risk, gait abnormalities.

The implication: **a single raw signal (motion), properly filtered and accumulated, contains extraordinary understanding.** You don't need many signals. You need deep understanding of ONE signal over time.

---


## 3. Android/Google — Collection Mechanisms

Google's collection architecture is defined by a different constraint: **understanding requires comprehensiveness.** The more signals, the better the model. The better the model, the more relevant the ads. The more relevant the ads, the more revenue. Collection IS the business.

This doesn't make Google evil — it makes Google's incentives structurally different from Apple's. Apple sells hardware. Google sells understanding. The collection mechanisms reflect this.

### 3.1 Hardware Sensors

**Standard Android sensor suite:**
- Accelerometer, gyroscope, magnetometer, barometer, ambient light, proximity
- Same physics, same chips, often same manufacturers as Apple
- Same always-on low-power operation for activity detection

**Always-on Hotword DSP:**
"Hey Google" detection uses the same architecture as Apple's "Hey Siri":
- Dedicated low-power DSP
- Continuous audio sampling
- Small neural network for wake word only
- On-device, no cloud transmission for detection
- Audio only sent to servers AFTER wake word confirmed and user speaks command

The always-on audio architecture is architecturally identical across platforms. The divergence happens AFTER detection.

**Pixel-Specific: Now Playing:**
Google's most elegant passive collection feature. Available on Pixel phones since Pixel 2 (2017):
- Constantly samples ambient audio
- Creates digital fingerprints of detected music
- Matches against on-device database (~50,000+ songs, updated weekly)
- Displays recognized song on lock screen
- **Entirely on-device.** Audio never recorded. Never transmitted. Only the match result shown.

Now Playing is Google demonstrating it CAN do Apple-style on-device intelligence. It's the proof that the cloud dependency elsewhere is a CHOICE, not a technical necessity.

### 3.2 Software Passive Collection

**Google Web & App Activity:**
The central nervous system of Google's understanding. Saves:
- Every Google Search query
- Every app interaction with Google services
- YouTube watch history (every video, every pause, every skip)
- Chrome browsing history (if signed in)
- Google Maps navigation history
- Voice commands to Google Assistant (audio recordings optional)
- Location data from Android device
- Device information (model, OS version, screen size)

Enabled by default. Retained indefinitely unless user changes settings. The aggregate of Web & App Activity IS Google's model of you.

**Google "Search Services History" (2026):**
A significant expansion discovered in 2026: Google now saves images, audio recordings, and video captured during Search interactions. This data is explicitly used to train AI models. The retention policy: kept for 4 years even if the user "deletes" it from their visible activity history.

This represents Google's most aggressive collection expansion — extending beyond text queries into multimedia captured during search interactions, with retention that survives user deletion.

**Digital Wellbeing:**
Google's equivalent of Apple's Screen Time:
- App usage frequency and duration
- Notifications received per app
- Phone unlocks per day
- Screen time breakdown by app

Like Screen Time, this data stays local. It serves the user's awareness. The irony: Google provides a tool to monitor usage of apps that Google itself uses to monitor the user.

**Google My Activity:**
The user-facing dashboard for ALL Google collection:
- Central hub showing everything Google knows
- Filterable by product, date, activity type
- Deletion controls (though deletion may not mean erasure per Search Services History)

**Location History (Timeline):**
Records everywhere you go:
- GPS coordinates
- Duration at each location
- Travel routes between locations
- Mode of transport
- Frequency of visits

When enabled, builds a complete geographic history of the user's life. Unlike Apple's Significant Locations (which stores only SIGNIFICANT places encrypted on-device), Google stores EVERYWHERE, on Google's servers, accessible via web dashboard.

**Android Advertising ID (GAID):**
A unique identifier assigned to each Android device:
- Tracks cross-app behavior
- Allows ad networks to build profiles
- Resettable by user (but apps often fingerprint around resets)
- Google announced deprecation of GAID in favor of Privacy Sandbox (2024+), but transition is slow

**Audio Memory (leaked 2026):**
Android Authority reporting revealed an upcoming Google feature extending Now Playing into a broader ambient audio service:
- Extends beyond music recognition
- May include ambient conversation awareness
- Google's official statement: "Background conversations and audio are never sent to Google"
- Privacy researchers noted the feature's scope exceeds Now Playing significantly
- Unclear boundary between on-device processing and cloud features

The leak illustrates Google's directional trajectory: always-on audio understanding that goes beyond single-purpose (music) toward general ambient intelligence.

---


### 3.3 How Android Filters Raw Data

Android's filtering is architecturally sophisticated on-device — often matching Apple's approach at the sensor layer. The divergence happens at the understanding layer, where filtered signals feed cloud models.

**Activity Recognition Transition API:**
Google's most elegant filtering design. The API doesn't stream raw sensor data to apps. It doesn't even stream continuous activity classifications. It only notifies on STATE CHANGES:

```
Transition: STARTED walking (timestamp)
Transition: STOPPED walking (timestamp)
Transition: STARTED driving (timestamp)
```

This is filtering at its purest: from thousands of accelerometer readings per second → a single event when behavior CHANGES. The noise ratio drops from infinite to near-zero. Apps don't process data — they receive conclusions.

**Now Playing Fingerprint Pipeline:**
```
Ambient audio → FFT → spectral fingerprint → match against local database → display result or discard
```
The audio is NEVER stored. Only the fingerprint (a compact mathematical representation) exists momentarily. Only the MATCH RESULT persists. This is the same architectural pattern as Apple's Hey Siri: the raw signal is consumed by the filter and destroyed.

**Federated Analytics (Now Playing):**
Google learns which songs are popular by region — to update the on-device database — without ever seeing individual users' data:
- Devices report: "matched song X" (with differential privacy noise)
- Aggregate: "Song X is popular in region Y"
- Update: push Song X to more devices in region Y
- Individual: no one at Google knows which USER heard which song

This is Google doing privacy-preserving analytics when it CHOOSES to — proving the capability exists.

**Gemini Nano (On-Device LLM):**
Google's on-device large language model (2-8B parameters):
- Text summarization
- Smart Reply generation
- Notification summarization
- On-device, no cloud required for inference
- Available through AICore system service

AICore manages on-device GenAI execution across different Android chipsets — abstracting hardware differences. This is Google's investment in LOCAL intelligence, running parallel to their cloud intelligence.

**The filtering duality:** Google filters raw sensor data on-device with the same rigor as Apple. Activity Recognition, Now Playing, Gemini Nano — all local. But the BEHAVIORAL data (searches, browsing, watching, navigating) flows to cloud with minimal filtering. The architecture is: filter HARDWARE signals locally, transmit SOFTWARE signals to cloud.

### 3.4 How Google Builds Understanding

Google's understanding is comprehensive because it's CROSS-PRODUCT. No single signal builds the model — ALL signals do, together.

**Web & App Activity → Interest Profile:**
Every Google product contributes to a unified understanding:
- Search queries reveal INTENT ("best running shoes" → fitness interest)
- YouTube watching reveals ENGAGEMENT (30min marathon training video → serious runner)
- Chrome browsing reveals EXPLORATION (reading running blogs → deepening interest)
- Maps navigation reveals BEHAVIOR (navigates to gym 4x/week → active lifestyle)
- Android app usage reveals HABITS (opens Strava every morning → morning runner)

**The profile drives everything:**
- Search results personalized to inferred interests
- YouTube recommendations tuned to viewing patterns
- Google Discover feed curated to predicted interests
- Ad targeting matched to purchase intent
- Maps suggestions based on behavioral patterns
- Gmail smart features informed by communication patterns

**Cross-Product Signal Fusion:**
The power — and the privacy concern — is signal COMBINATION:
```
Search: "headache remedies" +
Maps: visited pharmacy +
YouTube: watched migraine video +
Android: opened health app +
Chrome: browsed WebMD
= Understanding: "User experiencing recurring headaches, actively seeking treatment"
```

No single signal reveals this. The COMBINATION creates understanding that exceeds what any individual product could infer. This is the strength of comprehensive collection — and the reason privacy advocates raise concerns.

**Unlike Apple: the understanding lives in Google's CLOUD.**
- Associated with your Google Account
- Accessible across all devices signed into that account
- Persistent across device changes (new phone = same profile)
- Used by Google's advertising system
- Partially visible to the user via My Activity and Ad Settings

**The trade-off made explicit:** Google offers superior cross-device, cross-product understanding. Your new Pixel phone immediately knows your interests because the understanding isn't on any device — it's in the cloud, waiting for any device you sign into. Apple's understanding starts from scratch on each new device (though iCloud Keychain and device backup partially address this).

---


## 4. The Fundamental Architectural Difference

Strip away the marketing. Strip away the privacy policies. Look at the ARCHITECTURE:

| Dimension | Apple | Google/Android |
|-----------|-------|----------------|
| Default posture | Minimal collection, local processing | Maximum collection, cloud storage |
| User action needed | None (privacy by architecture) | Opt OUT to reduce collection |
| Where understanding lives | On-device only | Google's cloud servers |
| Cross-product signals | Limited (within device ecosystem) | Extensive (Search+YouTube+Chrome+Maps+Android) |
| Data for advertising | Never | Primary business model |
| On-device ML | Neural Engine, Core Motion coprocessor | Gemini Nano, AICore, Activity Recognition API |
| Audio processing | Wake word only, never stored | Wake word + optional audio history (Audio Memory) |
| Activity recognition | On-device, 7 days history, local only | On-device detection, cloud profile building |
| Research framework | SensorKit (IRB + Apple approval required) | Web & App Activity (opt-out, enabled by default) |
| Data retention | On-device until deleted by user | Years on Google servers (4+ years for Search Services) |
| Cross-device continuity | Limited (each device learns independently) | Complete (cloud profile follows any signed-in device) |
| Business model alignment | Intelligence sells hardware | Intelligence sells ads |

### What They Share

Both architectures share foundational engineering:
- On-device sensor fusion ML for real-time physical context
- Low-power always-on coprocessors for activity detection
- Neural network wake word detection (same architecture)
- Activity classification from accelerometer + gyroscope data
- Local ML inference for immediate responsiveness

**The COLLECTION is similar. The STORAGE and USE diverge completely.**

Both Apple and Google have invested heavily in on-device ML. Both can classify activity, detect audio patterns, and predict behavior without cloud. The difference is what happens AFTER the on-device intelligence produces its output:

- **Apple:** Output stays on-device. Used for local predictions. Dies with the device.
- **Google:** Output feeds cloud profile. Used for cross-product personalization. Lives forever in Google's infrastructure.

### The Philosophical Fork

Apple's position: "We don't need to understand you to serve you. The device understands you. We just build the device."

Google's position: "Understanding you IS serving you. Better understanding = better service. Cloud understanding = best understanding."

Both positions are internally consistent. Both produce useful products. The user's choice between them is a VALUES decision about where they want their understanding to live.

**For Feedmine:** The choice is already made. Feedmine is an iOS app built on Apple's philosophy. All intelligence is local. All understanding dies with the device. But Feedmine can LEARN from Google's comprehensiveness about WHAT signals matter — then implement that understanding entirely on-device.

---


## 5. What Feedmine Can Learn About Passive Understanding

Ten patterns from device-level passive intelligence, translated to reading behavior:

### 5.1 The Transition API Pattern

Google's Activity Recognition Transition API doesn't track every accelerometer reading. It fires ONLY on state changes: "Started walking." "Stopped driving." Transitions are the signal; continuous streams are noise.

**Feedmine equivalent:** Don't track every scroll position. Track STATE CHANGES:
- "Started reading" (article opened)
- "Stopped reading" (scrolled away / app backgrounded)
- "Switched source" (moved from one feed to another)
- "Returned after absence" (app opened after hours/days away)
- "Reading mode changed" (switched from quick-scan to deep-read based on time-on-article)

Transitions reveal routine. Continuous tracking overwhelms.

### 5.2 The Fingerprint Approach

Now Playing doesn't record audio. It creates a compact digital fingerprint and matches it against a database. The raw signal is destroyed. Only the match result persists.

**Feedmine equivalent:** Don't store full reading history. Create reading fingerprints:
- Topic embeddings (compact vector representing article's subject)
- Source affinity scores (how much engagement with each feed)
- Time-of-day preference vectors (what topics at what hours)

Match new content against the user's reading fingerprint. The fingerprint is small, private, and useful. The raw history is large, sensitive, and unnecessary.

### 5.3 On-Device Classification

Apple's Core Motion: Raw accelerometer data → "walking" / "running" / "still". The classification replaces the data. Apps never see raw sensor values.

**Feedmine equivalent:** Raw scroll behavior → reading mode classification:
- **Skimming:** fast scroll, short time-on-article, many articles per session
- **Deep reading:** slow scroll, long time-on-article, few articles per session
- **Searching:** rapid open-close, jumping between sources, looking for something specific
- **Routine scan:** moderate pace, familiar sources, consistent time-of-day

Classify the MODE, not the data. The mode is actionable. The raw scroll events are not.

### 5.4 The 7-Day Window

Core Motion keeps exactly 7 days of activity history. Not 30. Not forever. Seven days.

**Feedmine equivalent:** Keep short-term detailed data (7 days of specific articles read, exact timestamps, individual scroll depths). Compress to patterns for long-term storage:
- Short-term: "Read 3 AI articles Tuesday morning, 1 design article Tuesday evening"
- Long-term: "Reads AI content mornings, design content evenings, 3-4 articles per session"

Recent behavior is predictive. Ancient history is noise. The 7-day window is the sweet spot between "enough data for patterns" and "not so much data it becomes surveillance."

### 5.5 Significant Locations → Significant Sources

Apple tracks frequently visited places. Not every GPS coordinate — just the SIGNIFICANT ones. Significance = frequency × duration.

**Feedmine equivalent:** Track Significant Sources:
- Which feeds does the user return to repeatedly?
- Which topics accumulate the most reading time?
- Which sources have the highest completion rate?

Same formula: frequency × duration = significance. A source visited daily with high completion rate is "home." A source visited once with quick abandonment is "passing through." Use significance for predictions — surface significant sources prominently.

### 5.6 Screen Time as Reading Telemetry

Apple's Screen Time data model maps directly to reading telemetry:

| Screen Time Metric | Feedmine Reading Metric |
|-------------------|------------------------|
| App launches | Article opens |
| Session duration | Reading session length |
| Device pickups | App opens (reading sessions initiated) |
| Notifications received | New content arrivals |
| First pickup time | First reading session of the day |
| Category breakdown | Topic distribution |
| Daily/weekly trends | Reading pattern trends |

The data model is IDENTICAL. Feedmine is essentially running Screen Time for reading behavior.

### 5.7 Confidence Levels

Core Motion's activity detection always includes confidence: low, medium, or high. The system acknowledges UNCERTAINTY in its own classifications.

**Feedmine should attach confidence to every prediction:**
- "Interested in AI" (high confidence — reads daily, high completion rate)
- "Interested in cooking" (low confidence — read 2 articles last week, abandoned 1)
- "Morning reader" (high confidence — 90% of sessions before 9am)
- "Prefers long-form" (medium confidence — completes long articles but also reads short ones)

Confidence prevents over-fitting. It prevents the system from confidently recommending based on weak signals.

### 5.8 The Wake-Word Architecture

Hey Siri is always listening — but ONLY for one thing. The always-on processor runs continuously but has a single, narrow purpose. It doesn't analyze conversations. It doesn't transcribe. It listens for ONE pattern.

**Feedmine equivalent:** Always processing, but only for ROUTINE SIGNALS:
- Time patterns (when does the user read?)
- Completion rates (what do they finish?)
- Source visits (where do they go?)
- Session frequency (how often do they return?)

Not analyzing content meaning in real-time. Not building topic models from article text. Just watching BEHAVIORAL signals — the reading equivalent of listening for a wake word. Simple. Focused. Low-power.

### 5.9 Federated Analytics

Google's federated approach to Now Playing: learn aggregate patterns (popular songs by region) without ever seeing individual data. Each device contributes noise-protected signals. The aggregate is useful. The individual is private.

**Feedmine equivalent (if multi-user ever):** Learn aggregate patterns:
- "AI content is most read at 8-9am across all users"
- "Long-form articles have highest completion rate on weekends"
- "Source X has declining engagement across the user base"

Without ever seeing any individual user's reading patterns. Differential privacy + federated learning = collective intelligence without individual surveillance. For a single-user local app today, this is future architecture — but worth designing for.

### 5.10 State Change > Continuous Monitoring

The Transition API's core insight: only fire when activity CHANGES. A user walking for 30 minutes generates ONE event (started walking) and ONE event (stopped walking). Not 1,800 seconds of "still walking" readings.

**Feedmine should fire updates when behavioral STATE changes:**
- User's reading pattern changes (morning reader → evening reader)
- New topic interest emerges (first AI article → third AI article this week)
- Source engagement drops (daily reader of X → hasn't opened X in 5 days)
- Time-of-day preference shifts (weekday mornings → weekday lunches)
- Reading depth changes (was deep-reading → now skimming)

State changes are the moments when the system should UPDATE its understanding. Between state changes, the existing understanding holds. This is efficient, respectful, and accurate.

---


## 6. Implications for Feedmine's Architecture

Feedmine follows Apple's philosophy: collect minimally, process locally, never transmit. But it can LEARN from Google's comprehensiveness about WHAT signals matter — then implement that understanding entirely on-device.

### The Passive Listening Architecture for Feedmine

**Layer 1 — Always Collect (lightweight, continuous):**
- Timestamps of every article open
- Time-to-completion (or time-to-abandonment)
- Source identifier for every interaction
- Scroll depth (percentage of article consumed)
- Session boundaries (app foreground → background)
- Time between sessions (gap duration)

These are Feedmine's "accelerometer readings." Cheap to capture. Meaningless individually. Powerful in aggregate.

**Layer 2 — Filter On-Device (classify immediately):**
- Reading mode classification: skimming / deep reading / searching / routine scan
- Completion detection: finished / abandoned / partial (with threshold)
- Source affinity computation: engagement score per feed
- Time-of-day bucketing: morning / midday / evening / night reader
- Session type: quick check / long session / catch-up after absence

These are Feedmine's "activity classifications." The raw data is consumed by the filter. Only classifications persist.

**Layer 3 — Build Understanding Locally (accumulate over time):**
- Routine detection: WHEN the user reads (daily patterns, weekly patterns)
- Interest modeling: WHAT the user likes (topic affinities with confidence levels)
- Need anticipation: what to surface NEXT (based on routine + interest + recency)
- Source significance: which feeds are "home" vs. "occasional visit"
- Reading capacity: how much content the user can consume per session/day

These are Feedmine's "Siri Suggestions." Understanding that emerges from weeks of filtered signals. Never explicitly stated by the user. Observed, classified, accumulated.

**Layer 4 — Never Transmit:**
All intelligence stays on-device. No analytics server. No cloud model. No "improving our service" data collection. The architecture makes privacy violations IMPOSSIBLE, not merely prohibited.

If the device is wiped, the understanding is gone. This is a FEATURE, not a bug. Understanding should be as ephemeral as the user's relationship with the product.

**Layer 5 — Present as Proactive Service:**
The output of passive listening is PROACTIVE content delivery:
- Content ready before the user asks (pre-ranked feed at their usual reading time)
- Ranked for their current moment (morning = AI news, evening = long-form essays)
- Amount calibrated to available time (quick session = 3 articles, long session = deeper queue)
- New sources suggested based on interest fingerprint (not collaborative filtering — individual affinity)
- Declining sources de-prioritized without explicit unsubscribe

The user never configures this. They never set preferences. They never rate articles. They just READ — and the system watches state changes, classifies modes, and builds understanding. Exactly like Apple's Siri Suggestions: behavior IS configuration.

---


## 7. Summary — 5 Principles of Passive Understanding

Everything in this document distills to five principles. These govern how Feedmine should passively listen to reading behavior:

**1. Collect raw, filter immediately.**
Raw data is noise until classified. The filter IS the intelligence. Don't store scroll positions — store "deep read" or "skimmed." Don't store timestamps — store "morning reader" or "evening reader." The classification is useful. The raw data is liability.

**2. State changes > continuous data.**
Transitions reveal routine; continuous streams overwhelm. Fire when behavior CHANGES: new interest emerges, reading time shifts, source engagement drops. Between changes, the existing model holds. This is efficient (fewer computations), respectful (less tracking), and accurate (changes are the meaningful events).

**3. Confidence matters.**
Not all signals are equal. Attach certainty levels to every inference. "Interested in AI" (high — daily reading, high completion) is actionable. "Might like cooking" (low — read one article last Thursday) is not. Confidence prevents over-fitting to noise and enables graceful degradation when signals are weak.

**4. Short-term detail, long-term patterns.**
Seven days of specifics. Months of compressed patterns. Recent behavior predicts immediate needs. Historical patterns predict routine. The combination: serve today's session with this week's data, calibrate the feed's personality with this quarter's patterns. Delete the specifics. Keep the understanding.

**5. Understanding serves the user.**
The ONLY purpose of passive collection is to serve better. If a signal doesn't improve the reading experience, don't collect it. If a classification doesn't lead to better content ranking, don't compute it. If an understanding doesn't help the user, don't build it. Collection without service is surveillance. Collection WITH service is intelligence.

---

## Appendix: Sources

| Source | Type | Key Contribution |
|--------|------|-----------------|
| Apple ML Research (2025): Foundation model from accelerometer data | Research paper | Single sensor → broad health predictions; deep signal > many signals |
| Apple Developer Documentation: Core Motion (CMMotionActivityManager) | Documentation | 7-day history, confidence levels, activity classification API |
| Apple Developer Documentation: SensorKit | Documentation | Research-only passive collection: keyboard, ambient light, visits, messages |
| Apple Newsroom: Privacy (2024-2026) | Press releases | On-device processing commitment, differential privacy, Neural Engine |
| Apple Developer Documentation: Hey Siri / always-on processing | Documentation | DSP architecture, spectrogram → neural network → wake word pipeline |
| Google Developer Documentation: Activity Recognition Transition API | Documentation | State-change-only notifications, transition architecture |
| Google Research Blog: Now Playing (2017-2024) | Blog posts | On-device fingerprinting, federated analytics, local-only music recognition |
| Google Developer Documentation: Gemini Nano / AICore | Documentation | On-device LLM, 2-8B parameters, local inference |
| Android Authority: Audio Memory leak (2026) | News reporting | Extended ambient audio capabilities, privacy implications |
| TechCrunch: Google Search Services History (2026) | News reporting | 4-year retention, multimedia collection, AI training use |
| medrxiv: SensorKit accelerometer study (2025) | Research paper | Foundation model from motion data predicting health outcomes |
| PCMag: Google Web & App Activity analysis | Analysis | Comprehensive breakdown of what Google collects and retains |
| Lifehacker: Digital Wellbeing and Screen Time comparisons | Analysis | Feature parity and divergence between platforms |
| security.org: Android advertising ID and tracking | Research | GAID mechanics, cross-app tracking, Privacy Sandbox transition |
| Wikipedia: Differential Privacy | Reference | Mathematical framework for privacy-preserving aggregate data |

---

*End of Loop 05: Passive Listening. This document focuses on the MECHANISMS of passive data collection and understanding — how devices observe, filter, and build models of user behavior. The moral questions are important but deliberately excluded here. The goal is architectural understanding: how do you build a system that passively knows what a user needs? Apple and Google both answer this question. Feedmine takes Apple's privacy architecture and Google's signal comprehensiveness — implementing the latter within the former.*
