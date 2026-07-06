# 🔒 Structural Privacy — No Server, Not by Philosophy, but by Design

> Seed concept for loop exploration
> 2026-07-06

---

## The insight

The privacy debate is stuck in a false frame: "how much data should companies collect?" This assumes the collection→server→monetization pipeline is inevitable, and we're just negotiating the terms.

Feedmine sidesteps the debate entirely. There is no server. Not because of a privacy philosophy. Not because "data collection is bad." But because **a server has no function here.**

Think about what a server does in a typical app:
- Stores user accounts → Feedmine has no accounts
- Syncs across devices → Feedmine runs on one device
- Sends push notifications → Feedmine fetches RSS directly
- Runs ML models → There are no ML models
- Collects analytics → For whom? There's no business team.
- Serves ads → There are no ads.

When you remove every reason for a server to exist, privacy isn't a feature you build. It's a consequence of architecture. The data can't leak because there's nowhere for it to leak *to*.

**The question: what does it mean when privacy is not a choice but a structural inevitability?**

---

## The mechanism

### What Feedmine actually collects

This needs to be clear: Feedmine is not a privacy-minimalist app. It observes and stores:

| Data | Storage | Purpose | Retention |
|------|---------|---------|-----------|
| Read item IDs + timestamps | Compressed JSON file | Dimming read cards, filtering "What's New" | Auto-cleanup after 90 days |
| Bookmarked item IDs | JSON file | Bookmark sheet access | Until user clears |
| Disabled source IDs | JSON file | Skip sources during fetch | Until re-enabled |
| Cached feed items (200 max) | Compressed JSON file | Instant launch without network | Overwritten each session |
| Clicked source URLs | JSON file | "What's New" carousel prioritizes engaged sources | Until user clears |
| App open timestamps | UserDefaults | Routine detection (exact/approximate/unusual) | Rolling 7 days |
| Session minutes today | UserDefaults | MomentCard session slot | Resets daily |
| Streak count | UserDefaults | MomentCard streak slot | Resets on miss |
| Days with app | UserDefaults | MomentCard context | Persistent |
| Filter selections | UserDefaults | Restore active filters on launch | Persistent |
| Circadian/font preferences | UserDefaults | Visual settings | Persistent |

The app also **computes but does NOT persist:** time of day, season, weekday, moon phase (Julian date calculation), weather condition, battery state, connectivity. These are derived from the system clock and APIs, used for the MomentCard and circadian engine, then discarded.

### The pipeline ends at your phone

```
RSS sources (public web) → Phone fetches directly → Local parsing → Local storage → Local display → (end)
```

There is no step where data leaves the device. The only network traffic is **outbound requests to RSS URLs** — the same request any browser makes when you visit a website. No data goes the other direction.

No analytics endpoint. No telemetry. No crash reporting service. No third-party SDKs (the single dependency, FeedKit, is a pure parser with no network component). No phone-home behavior.

### What "structural" means

| Privacy approach | Mechanism | Failure mode |
|-----------------|-----------|--------------|
| Privacy policy | "We promise not to misuse your data" | Company gets acquired. Policy changes. Data gets breached. |
| Privacy settings | "Toggle what you share" | Dark patterns. Default-to-share. Confusing options. |
| Data minimization | "We collect as little as possible" | The app becomes dumb. Features suffer. |
| **Structural privacy** | "There is no server to receive data" | None. The pathway doesn't exist. |

You can't breach a server that doesn't exist. You can't subpoena data that was never transmitted. You can't change a policy for a system that doesn't need one.

### The user owns the data completely

- **Export My Data** — generates a JSON file via the system share sheet. Everything the app knows, in a format you can read.
- **Clear Read History** — deletes all read tracking. Cards return to unread state.
- **Clear All Bookmarks** — wipes the bookmark list.
- **Reset All Data** — factory reset with confirmation dialog. Nuclear option, one tap.

There's no "request my data" process. No GDPR form to fill out. No 30-day waiting period. The data is on your phone. You can export or delete it immediately.

---

## Why this matters

### 1. Intelligence and privacy are not at odds

The industry says: "If you want a smart experience, you must give up privacy." This is only true if "smart" requires a server. Feedmine runs its circadian engine, interleave algorithm, routine detection, and MomentCard generation entirely on-device. The intelligence uses local data locally. Smart AND private — not as a compromise, but as a natural consequence of architecture.

### 2. The question shifts from "should they?" to "could they?"

With a typical app, you ask: "Should this company have my reading history?" With Feedmine, the question doesn't arise. There is no "they." The code runs on your phone. The data stays on your phone. The architecture makes the ethical question moot.

### 3. Future-proof privacy

Privacy policies change. Companies get acquired. Laws change. Governments overreach. But architecture doesn't change retroactively. If there was never a server, there was never a copy of your data outside your device. No future event can expose what was never transmitted.

### 4. The user doesn't need to think about privacy

No cookie banners. No "what data do you want to share?" dialogs. No privacy settings to configure. No account creation. No email address requested. The system is private by construction. The user gets contextual intelligence and privacy without choosing between them — because there was never a choice to make.

### 5. Open source amplifies structural privacy

The code is open. Anyone can verify there's no telemetry, no hidden endpoints, no phone-home behavior. `grep` the codebase for network calls — you'll find only FeedKit fetch requests to RSS URLs. Structural privacy plus open source means: *trust nothing, verify everything.*

---

## The relationship to open algorithms

Structural privacy is what makes open algorithms *safe*. In a typical platform, exposing the algorithm might reveal how invasive the data collection is. In Feedmine:

- The algorithm uses clicked source URLs? Fine — that data stays on your device.
- The MomentCard uses your streak and session time? Fine — those numbers exist only in UserDefaults.
- Routine detection tracks your open timestamps? Fine — 7 rolling days, on your phone, nowhere else.

Openness and privacy reinforce each other when the server doesn't exist. You can inspect every piece of logic without fear of what it reveals — because what it reveals is a system that talks only to itself.

---

## Questions for the loop

- How does Feedmine's structural approach compare to end-to-end encryption? (E2EE trusts math over institutions. Feedmine trusts architecture over policies. Both eliminate the need for trust.)
- What can we learn from the history of the personal diary — a technology of self-knowledge that is structurally private because it lives in a drawer?
- How does the concept of *privacy by design* (Cavoukian) relate to *privacy by absence of infrastructure*? Is "no server" the ultimate Privacy by Design?
- What is the relationship between local-first software (Ink & Switch, CRDTs, Automerge) and structural privacy? Feedmine is local-only — not even local-first (no sync). Is that a limitation or a feature?
- How does the absence of a server change the legal status of user data? (Data on a personal device may have different legal protections than data on a company server — Fourth Amendment, GDPR)
- What can we learn from the concept of *air gaps* in security? Feedmine's privacy is effectively an air gap between user data and external networks.
- The app has no sync. Is this a privacy win or a usability loss? Can you have structural privacy AND multi-device? (Probably: encrypted local sync via iCloud keychain. But that's not built today.)

---

*The app knows your patterns. It uses everything it learns. And none of it goes anywhere. Not because we promised. Because there's nowhere for it to go.*
