# ⚔️ Steve Jobs vs Linus Torvalds: Offline-First / Local-First

> Debate — Iteração 2
> 2026-07-01

---

## Steve Jobs — The Keynote

**[Stage dark. Steve walks out in black turtleneck and jeans. He holds up a phone.]**

Good morning.

There are about two billion people on social media today. Two billion. And every single one of them has had this experience.

**[He looks at the phone. He puts it down on the table.]**

You open the app. You want to see what your friends are doing. You want to write something. You want to connect. And instead you get this.

**[He turns the phone around. It shows a spinning wheel. "No Internet Connection."]**

"No Internet Connection."

You're on the subway. You're on a plane. You're in a basement. You're at a conference with twenty thousand other people and the cell towers can't keep up. And every single social app on the planet gives you the same thing.

**[Long pause.]**

A blank screen.

It says you're disconnected. But really — they're disconnected. They built a house with no foundation. They built a car that only works downhill. They built a social app that only works when everything is perfect.

And that is not good enough.

**[He paces. Stops. Looks at the audience.]**

So we asked ourselves: what if we built it differently? What if we started from the premise that the internet is *not* a given? What if we built a social app that works when you're connected — and works even *better* when you're not?

**[He holds up the phone again.]**

This is Liberation.

**[The screen lights up. Posts. Replies. A search bar.]**

Everything you see here? It's all local. On your device. SQLite. The entire experience lives in your pocket. Not in the cloud. Not on a server. Right here.

**[He scrolls. Fast. It's instant.]**

Look at that. No loading. No spinners. No "fetching..." No waiting. It just *is*. Because it's already there. It never left.

**[He taps a reply box. Types something. Hits send. The message sits there with a little clock icon.]**

I just wrote this. No internet. It doesn't matter. The app holds it. Waits. And the second I get a signal — boom. It goes. I don't think about it. I don't retry. I don't copy my text to Notes and paste it later. It *just works*.

It's like having a letter in your pocket. You write it when you want. You seal the envelope. The post office finds you when you're ready.

Now — every other company will tell you this is impossible. They'll say you need servers. You need the cloud. They'll tell you it's the only way.

They're wrong. And more than that — they're missing the point.

Because here's the thing. This isn't a "feature." It's not "offline mode." It's a fundamental rethinking of what a social app is. It's not an app that works *without* the internet. It's an *app*. The internet just makes it faster.

**[He stops. Puts his hands together. Nods slowly.]**

And there's something else.

**[He pulls up a settings screen. "Your data lives on this device."]**

Your data lives on your device. Not on our servers. Because there are no servers. Your conversations. Your drafts. Your search history. It's yours. It never leaves your phone unless you tell it to.

Privacy isn't a feature we added. It's the *foundation* we built on. When you build offline-first, privacy isn't a policy — it's physics. The data can't leak because it was never in the cloud to begin with.

**[He smiles. The audience is quiet.]**

So: an app that works anywhere. On a plane. In a tunnel. In the middle of nowhere. An app that respects you enough to keep your data where it belongs. An app that doesn't punish you for having a bad signal.

That's Liberation.

**[He points to the screen.]**

Something on your mind? Write it down. Now. Not later. Hit send. The internet will catch up.

We think that's how it should have always worked.

**[Pause. The room erupts.]**

Thank you.

---

## Linus Torvalds — The Critique

Subject: Re: "Offline-first social app" — you mean an RSS reader from 2005?

So Steve just got a standing ovation for inventing... an RSS reader. "Works offline!" "Like a letter in your pocket!" "The post office finds you!" This is what happens when you put a keynote before engineering. You get poetry about something that already existed in 2005, and you pretend it's a revolution.

Let me explain why this is not a revolution. This is a hard problem that you're hand-waving through with beautiful slides.

**1. You're serving stale data and calling it a feature.**

A Mastodon post from 2 days ago, cached on your device, is not "the conversation." It's a fossil. A photograph. The conversation moved on without you. When you reply to that stale post offline and sync 3 hours later, you're not "connecting" — you're necroposting into a thread that already died. Presenting a snapshot as the real thing is dishonest.

**2. You have no answer for sync conflicts.**

I write a reply offline. Someone else replies to the same thread before I sync. What happens? Last-writer-wins? That's not "eventually consistent" — that's "silently dropping someone's words." Merge conflicts on SOCIAL MEDIA? Do you diff the text? Do you show both replies in some kind of CRDT miracle? Have you thought about this for more than five seconds?

The spec says SQLite with FTS5 and an outbox table. That's not a sync strategy. That's a TODO list.

**3. Storage. On. A. Phone.**

A heavy Mastodon user following 500 accounts generates GIGABYTES per year. Media attachments. HTML content. FTS5 indexes. Now multiply by 6 protocols. Now add archived imports from Twitter, Instagram, Facebook (because this app ALSO wants to be your data vault — that was in the spec). Where does this live? On a phone with 128GB that also has photos, videos, and other apps?

The spec mentions an "LRU eviction policy" — that's not a storage strategy. That's "we'll delete things when it gets full and hope the user doesn't notice." What gets evicted? Their 3-year-old posts or yesterday's? Who decides?

**4. "The internet just makes it faster." No. The internet is the whole point.**

A social app without live data is a diary. A diary is a fine product. Call it a diary. Don't call it a social app and then hand-wave the "social" part as "the internet just makes it faster."

**The ONE thing that isn't bullshit:**

Reading from local SQLite IS faster than any network call. 100x faster. Sub-millisecond queries. That's genuinely good engineering. If you frame this as a PERFORMANCE feature — "we pre-sync everything so your timeline renders at 60fps even on bad connections" — I'm actually interested. That's a real problem with a real solution.

But when you frame it as "liberation from the internet" and "privacy as physics"? You're selling a database as a philosophy. Just say "we use SQLite for performance and it happens to work offline too." That's honest. That I respect.

Show me the sync protocol. Show me the CRDT types for conflict resolution. Show me the storage budget on a 128GB phone after 1 year of normal use. Show me what happens when the outbox tries to publish 200 queued messages and gets rate-limited by 4 different protocols simultaneously.

Then we can talk.

-- L

---

## O Que Cada Um Acertou

**Steve está certo sobre:**
- A experiência de abrir um app sem internet é UNIVERSALMENTE horrível
- Privacidade como propriedade emergente da arquitetura (não como "feature") é poderoso
- Performance local (SQLite) é transformadora na sensação de uso
- "Não é offline mode. É um app. A internet deixa mais rápido." — reframing poderoso

**Linus está certo sobre:**
- Dados stale não são "a conversa". São um snapshot. Seja honesto sobre isso.
- Conflitos de sync são um problema REAL e complexo. CRDTs não são mágica.
- Storage em dispositivo é limitado. 128GB com fotos, vídeos, outros apps + cache social = lota rápido.
- Framing como PERFORMANCE ("pré-sync para 60fps") é mais honesto que "libertação da internet"
- O diabo está no sync protocol. Sem ele, é um diário offline, não um app social.

**A síntese possível:**
- Vender como PERFORMANCE + PRIVACIDADE, não como "offline liberation"
- Ser HONESTO sobre staleness: "Este post é de 2 dias atrás. A conversa pode ter continuado."
- Definir storage budget transparente: "O app está usando 3.2GB. Limite: 5GB."
- Mostrar o sync protocol. Publicar a especificação. Nada de caixa preta.

---

*Debate gerado na Iteração 2.*
