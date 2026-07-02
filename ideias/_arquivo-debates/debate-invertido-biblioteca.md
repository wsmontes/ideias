# 🔄 Linus Cria, Steve Critica: O Produto é a Biblioteca ou o App?

> Debate Invertido — Iteração 3
> 2026-07-01

---

## Linus Torvalds — A Proposta

What matters is not the shiny UI that everyone gets excited about for six months and then abandons when the next framework comes along. What matters is the foundation.

The product is not the app. The product is the library.

That `libs/protocol-adapters` crate — the ProtocolAdapter trait, EventBus, canonical data model, sync engine, identity and keychain layer — THAT is the thing that matters. That is the thing that could last fifteen years. That is the thing that, if you do it right, becomes infrastructure.

If you get that abstraction right — and I mean RIGHT, not 80% right — then you are not just building an app. You are building the foundation that every decentralized social client will be built on for the next decade.

The Liberation Client app? It's a demo. The reference implementation. It proves the library works. If 100,000 people use it, fantastic. If 100 people use it, the library still has value.

Open source the library under MIT or Apache 2.0. Let anyone build on it. Let someone build a Nostr client that's actually good. Let someone build a Mastodon client that doesn't suck. Let someone build something we haven't even thought of yet.

Does the library make money? No. Git doesn't make money. Linux doesn't make money. But they're infrastructure. Everything else is built on top of them. This library could be the infrastructure for decentralized social media.

Stop thinking like an app developer. Start thinking like someone who builds the foundations.

---

## Steve Jobs — A Crítica

Linus, I read your proposal. And I have to say — it's exactly the kind of thing I'd expect from someone who's spent twenty years thinking about kernel modules.

You're wrong.

"Reference implementation." That's not a product. That's an EXCUSE. That's what you say when you're too afraid to commit to making something people actually want to use.

Nobody downloads a library. Nobody wakes up in the morning and says, "I hope someone releases a nice protocol adapter today." People download APPS. Beautiful, polished, magical apps that sit on their home screen. If your best work is hidden inside a crate that only twelve developers on Hacker News will ever see, you have failed at the only thing that matters: reaching people.

Git and Linux succeeded because they solved problems for DEVELOPERS. The sync engine solves a problem for USERS — fragmented social media, locked-in conversations, algorithmic hell. Users don't compile crates. They tap icons. They swipe. They share. They don't care about your architecture. They care about what it DOES.

The library is important. It's a beautiful foundation. But you don't invite people to see the foundation. You invite them to see the cathedral. The app IS the cathedral. And you're telling me the real product is the concrete and rebar.

If you position this as a library project, you will get 47 stars on GitHub, a thoughtful thread on Lobste.rs, and exactly zero impact on the actual problem. If you position this as an APP — a gorgeous, frictionless, shockingly good app — you could change how millions of people communicate.

Which one actually liberates anyone?

Don't hide your best work where nobody sees it. That's not humility. That's cowardice dressed up as engineering purity. Build the damn app.

---

## Síntese

**Linus está certo sobre:** A biblioteca é o legado duradouro. Se a abstração de protocolo for bem feita, dura 15 anos. O app pode morrer, a lib vive. Open source (MIT/Apache 2.0) é a licença certa.

**Steve está certo sobre:** O mundo não baixa bibliotecas. Baixa apps. A história que você conta é sobre o APP. A biblioteca é COMO você constrói, não O QUÊ você constrói. 47 estrelas no GitHub não liberta ninguém.

**Os dois estão certos.** A biblioteca é o legado. O app é o produto. Um não exclui o outro. Construa a biblioteca. Lance o app. Conte a história do app. Deixe a biblioteca ser descoberta por quem constrói.

---

*Debate invertido gerado na Iteração 3.*
