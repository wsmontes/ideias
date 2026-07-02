# 🔄 Linus Cria, Steve Critica: E2EE em Todo Lugar

> Debate Invertido — Iteração 2
> 2026-07-01

---

## Linus Torvalds — A Proposta

End-to-end encryption is not a checkbox. It is not a "premium feature." If you are building a "liberation" tool and shipping messages in plaintext, you are not building liberation. You are building a prettier cage.

Let me walk through the threat model. You send a DM on Mastodon. The instance admin reads it. Not because they're malicious — most aren't — but because the protocol stores the message as plaintext JSON. The FBI doesn't subpoena you. They subpoena the admin of `fursuits.social` who has never seen a warrant, and now every DM you've ever sent is in a federal courthouse.

Bluesky? Same story. The relay sees everything. IRC? From 1988. Security model of a postcard taped to a wall.

The engineering mandate:

1. **Protocols with native E2EE** (Matrix, XMPP, Nostr): DEFAULT. Not a toggle. You opt OUT with a warning.
2. **Protocols without E2EE** (ActivityPub, Bluesky): application-layer encryption. Message encrypted on-device with recipient's public key before touching the network. PGP-style without the PGP baggage.
3. **Key verification mandatory.** Signal-style safety numbers. Emoji comparison. Nag until verified.
4. **IRC gets a red warning.** Every. Single. Message. "Anyone can read this. Continue?" Tap through.

If every DM is plaintext, you have not liberated anyone. You have just given them a different set of people who can read their mail.

E2EE everywhere. Not negotiable.

---

## Steve Jobs — A Crítica

You're missing the point completely. Completely.

Let me tell you what happens when grandma opens her messaging app and sees a red warning. You think she feels protected? She feels TERRIFIED. She calls her son, who tells her to ignore it. Now she's learned that red warnings mean nothing. You've just burned the most important security signal you had.

You want her to compare emoji codes to verify a key? She can barely find the camera button.

This is the problem with engineers. You see a cryptographic problem and you want a cryptographic solution. But you're not building Signal. You're building a multi-protocol social client. Your job is federation — making it so people can talk across platforms without a PhD in applied cryptography.

Key management is the hardest problem in computer security. People lose phones. They forget passphrases. Every key recovery flow you design will be either too complex (nobody uses it) or too simple (defeats the encryption). You cannot solve this at the app layer.

The right approach: encrypt where the protocol supports it NATIVELY. Matrix, XMPP — use their E2EE. Don't touch it. Pass it through. For protocols that don't support it, don't fake it. Don't layer PGP on top like it's 1991. Be HONEST. Two states: "This conversation is private" or "This conversation is NOT private." That's it. Two. Not seven lock colors. Not red warnings. Not emoji comparison. Two.

You want Signal-level security? Use Signal. We're building something that connects to everything. Stop trying to solve the hardest problem in cryptography in a messaging client.

---

## Síntese

**Linus está certo sobre:** DMs em ActivityPub e Bluesky são plaintext. O admin LÊ. Isso é inaceitável para um app que se chama "Liberation." E2EE onde o protocolo suporta nativamente DEVE ser o default. Sem toggle.

**Steve está certo sobre:** Key verification com emoji é UX hostil para não-técnicos. Sete cores de cadeado treinam o usuário a ignorar. "Application-layer PGP" é reinventar a roda de 1991 e vai quebrar.

**A síntese:** 
- Matrix, XMPP, Nostr → E2EE nativo, transparente, sem emoji. A verificação existe mas não bloqueia.
- ActivityPub, Bluesky, IRC → banner informativo, não warning aterrorizante. "Esta conversa NÃO é privada. O administrador do servidor pode lê-la."
- Dois estados. Sem reinvenção de PGP.
- Se o usuário quer E2EE garantido, use Matrix ou XMPP. O app deixa isso CLARO.

---

*Debate invertido gerado na Iteração 2.*
