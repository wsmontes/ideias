# ⚔️ Steve Jobs vs Linus Torvalds: Privacidade

> Debate — Iteração 5
> 2026-07-01

---

## Steve Jobs — The Keynote

There is no cloud. There never was.

Every social app you've ever used has a server. A computer in a building somewhere that you don't control. When you type a message, it goes to their computer first. Every draft, every like, every search — their computer. Their property. Their rules.

When a company tells you they take privacy seriously, that means they HAVE your data. And they're choosing not to abuse it. Today. Under this management.

What happens when management changes? What happens when the government shows up with a warrant?

That's not privacy. That's a promise. And promises break.

Privacy isn't a policy you can change. It's physics. If your data exists on their server, it can leak. It can be subpoenaed. It can be sold.

So we asked a different question. Not "how do we protect your data on our servers?" — but "why do we need servers at all?"

The Liberation Client has no servers. None. Zero. Your data lives on your device. SQLite. Local storage. When you choose to share — when you hit publish — it goes directly from your device to the protocol. The app is not a repository. It is a conduit.

Think about what this means.

We can't give your data to the government — because we don't HAVE your data. We are subpoena-proof by architecture, not by lawyers.

We can't sell your data to advertisers — because we don't HAVE your data.

We can't train AI on your conversations — because we don't HAVE your conversations.

This is not a feature you enable in settings. This is the foundation. Privacy as architecture. Not a promise — a consequence of how the system was built.

That's Liberation.

---

## Linus Torvalds — The Critique

"No servers. Right."

So you built an app that connects to a dozen Mastodon instances, half a dozen Nostr relays, some Bluesky PDS, maybe a Matrix homeserver. And your claim is "no servers."

Those relays? Those are servers. The Bluesky PDS someone spun up on a $5 VPS? Server. The Matrix homeserver that stores every room you join? Server. Mastodon instance admin who can read your DMs with a simple database query? Server. Your data IS on their hard drive. The moment you publish a post, it's replicated across federated nodes. "Your data never leaves your device" is a lie, unless the user literally never sends a single message.

"Privacy is physics, not policy." Great line. Then Alice posts a toot about her medical condition to a public Mastodon timeline. That toot is now on fifteen different servers in seven jurisdictions. It has been indexed by search engines. Someone's bot scraped it into a dataset. That's physics too — thermodynamics of information. Once you emit bits into a federated network, entropy only goes up. You cannot un-publish.

Let's talk DMs, since you brought up the FBI. Your user's DMs on ActivityPub are base64-encoded JSON sitting in a Postgres table on mastodon.social, readable by any admin with `psql`. DMs on Bluesky are in the relay's firehose — not encrypted. It's not physics stopping them, it's policy. Policy is a .txt file. The FBI loves .txt files.

Nostr relays implementing NIP-44 E2EE — genuinely better. Matrix has real E2EE — Olm/Megolm, ratchets, key verification. XMPP with OMEMO. THAT's the bar. Everything else is fancy plaintext.

When the FBI shows up, they won't come to you. They'll go to mastodon.social. Or the Bluesky PDS operator. Or the Matrix homeserver. And those admins WILL comply — either because they want to, or because a judge says so. You haven't eliminated surveillance. You've outsourced it to a long tail of underfunded volunteers who have never seen a warrant.

I'll give you this: local-first is genuinely better than server-first. But the REAL privacy problem isn't where the client runs. It's where the data lives and who controls the keys. Encrypt DMs end-to-end. Sign posts with user keys so instances can't forge them. Let users verify each other's keys at the protocol level. THAT would be worth a keynote.

---

## O Que Cada Um Acertou

**Steve está certo sobre:**
- Local-first é GENUINAMENTE melhor pra privacidade que server-first. Dados que nunca foram coletados não podem vazar.
- A empresa não pode entregar o que não tem. Isso é real.
- O app como "conduíte, não repositório" é uma arquitetura de privacidade legítima.

**Linus está certo sobre:**
- "No servers" é marketing. O app conecta em DEZENAS de servidores de terceiros. Seus dados PUBLICADOS estão neles.
- "Privacy is physics" — e a física da informação pública é: uma vez publicado, replicado, indexado, scrapado. Sem volta.
- DMs no ActivityPub NÃO são criptografados. DMs no Bluesky NÃO são criptografados. O admin LÊ.
- Quando o FBI chegar, não vai bater na porta do Liberation Client. Vai bater na porta do admin da instância Mastodon.
- E2EE de verdade (Matrix Olm/Megolm, XMPP OMEMO, Nostr NIP-44) é a BARRA. O resto é "prometo que não olho."

**A síntese possível:**
- "Seus dados locais nunca saem do dispositivo" = VERDADE (drafts, histórico, buscas)
- "Seus posts publicados são privados" = MENTIRA (estão em servidores de terceiros)
- A comunicação honesta: "NÓS não temos seus dados. Mas os protocolos que você ESCOLHE usar têm. Escolha com consciência."
- E2EE onde possível. Avisos claros onde não. Nunca chamar DM de ActivityPub de "privado."

---

*Debate gerado na Iteração 5.*
