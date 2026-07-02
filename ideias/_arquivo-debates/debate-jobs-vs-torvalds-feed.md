# ⚔️ Steve Jobs vs Linus Torvalds: O Feed Unificado

> Debate — Iteração 1
> 2026-07-01

---

## Steve Jobs — The Vision

*(Steve Jobs walks to the center of the stage. Black turtleneck. Levi's 501s. New Balance sneakers. He holds a marker, but doesn't use it yet. He looks at the screen. He looks at the audience. Silence for four beats.)*

So.

We looked at the internet. And we saw something. Most people saw the fediverse. They saw ActivityPub here, Nostr there, AT Protocol over there, Matrix, IRC, XMPP. Six protocols. Six communities. Six ways of thinking about communication.

And most companies would say: "Great. We'll support all six. Give the user a dropdown. Let them switch between timelines." Most companies. That is what most companies would do.

*(He shakes his head slowly.)*

That is *bullshit*.

The user does not want six timelines. The user wants *one* timeline. The user does not want to think about protocols. The user wants to think about what their friend just said.

Let me ask you something. When you pick up a phone and call your mother — do you think about SS7 signaling? Do you think about the switching network? No. You think about your mother's voice. That is what technology is supposed to do. It is supposed to *disappear*.

*(He starts to pace.)*

I looked at the specs for this Unified Feed. And you know what I saw? I saw engineers solving the wrong problem. I saw protocol negotiators. I saw content-type mappers. I saw adapter layers. I saw *six things glued together with duct tape*.

And it made me *angry*.

Because here's the truth: if you need to tell the user what protocol a message came from, you have already failed. You have failed at design. You have failed at engineering. You have failed at *taste*.

The magic of this thing — the *only* thing that matters — is that a user reads a thread, and they cannot tell you whether the original post came from Mastodon, the reply came from Bluesky, and the reaction came from a Matrix bridge. They can't tell you. Because it doesn't matter. It's *one conversation*.

*(He picks up the marker. Draws a single horizontal line across the whiteboard. Bold. Clean.)*

This is the timeline. That's it. One line. No tabs. No "ActivityPub Feed." No "Nostr Feed." No "AT Protocol Feed." One. Beautiful. Unified. Stream.

An iPod had five thousand songs in your pocket. You didn't think about MP3 versus AAC versus WAV. You thought about the music.

This feed — this Liberation Feed — it has every conversation, on every protocol, in your pocket. You don't think about ActivityPub versus AT Protocol versus Nostr. You think about the conversation.

That's the product. Ship it.

---

## Linus Torvalds — The Critique

Subject: Re: [RFC] Unified Multi-Protocol Social Feed -- the invisible protocol layer

So someone sent me this vision document about "unifying" ActivityPub, AT Protocol, Nostr, Matrix, IRC, and XMPP into a single timeline. And I read the whole thing, waiting for the part where you explain how any of this actually works. I got to the end. That part doesn't exist.

Here's the problem: you're solving a social problem with a UI animation and calling it engineering.

Let me be very specific about why this is bullshit. You say the protocol should be "invisible" to the user. That is a lie. Protocols have failure modes. Mastodon is HTTP-based with CDN-backed media. Nostr is relay-based and there is no guarantee *any* relay has *any* particular event. Matrix uses federated rooms with state resolution that can take seconds. IRC has no history at all -- you connect, you see what people say *after* you joined. XMPP has presence stanzas and MUC semantics that predate half of these protocols by two decades.

You want to smash these into one timeline and have them "look the same." Why? A Mastodon post has HTML formatting, media attachments, content warnings, and threading. A Nostr event is a JSON blob with a plaintext field. An IRC message is 512 bytes. A Matrix event has a whole room state behind it. These are NOT interchangeable. Presenting them as interchangeable is not unification -- it is destruction of information. You are building a lossy compression layer for human communication, and calling it a feature.

And you use the word "magic." "The protocol bridge handles federation magic." Magic is what you call it when you don't understand your own system. I want to know what happens when the Nostr relay is down but Matrix is up. Does the feed have a hole? Do events arrive out of order? Does the UI show a spinner forever? You didn't specify, because you don't know. You wrote "graceful degradation" which is consultant-speak for "figure it out later."

Also: who is this for? Show me the person with meaningful social graph presence on six protocols. That person doesn't exist. The Nostr people don't want Mastodon content in their feed. The IRC people will laugh at you. The Matrix people want proper threading and state. You are building a product for zero users.

Now. Here is the one thing that might work, if you actually care about making something useful instead of writing vision documents. **A unified transport layer that normalizes identity and does nothing else.** Map each protocol's identity to a canonical DID. Handle routing between protocols. Everything above that -- content model, threading, presentation -- stays protocol-specific. The user sees *one protocol at a time* with a bridge routing messages from other protocols to it, with explicit labels. "This message arrived via Nostr relay X." "This reply came from Matrix room Y." No magic. No "invisible" protocol. Just honest engineering that tells the user what is happening.

But that is harder than writing the vision document, which is why you didn't write that version.

I'll believe in unified protocol feeds when I see the kernel patch. Not before.

-- L

---

## O Que Cada Um Acertou

**Steve está certo sobre:**
- O usuário não quer pensar em protocolos. Quer se comunicar.
- A analogia do iPod é poderosa: formatos de arquivo sumiram, a música ficou.
- Seis timelines separadas é pior que uma unificada. Mesmo com arestas.

**Linus está certo sobre:**
- "Invisível" é mentira. Protocolos têm falhas diferentes e o usuário PRECISA saber quando algo quebrou.
- Conteúdo de protocolos diferentes NÃO é a mesma coisa. Perder formatação é "lossy compression."
- A solução pragmática: camada de transporte + identidade canônica. O resto fica protocolo-específico. Labels explícitos.
- "Quem é o usuário com presença em 6 protocolos?" — pergunta válida. O nicho é pequeno.

**A síntese possível:**
- A experiência DEFAULT é unificada e invisível (Steve)
- Quando algo dá errado, a transparência é total (Linus)
- O "Modo Simples" = Steve. O "Modo Avançado" = Linus.
- Ambos cabem no mesmo app.

---

*Debate gerado na Iteração 1. Próximo tópico no loop.*
