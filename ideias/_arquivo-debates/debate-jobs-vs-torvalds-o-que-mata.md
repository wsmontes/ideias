# 💀 Steve Jobs vs Linus Torvalds: O Que Mata Este App

> Debate Final — Iteração 6
> 2026-07-01

---

## Steve Jobs — As 3 Coisas Que Matam (Produto)

**1. O Suicídio dos Seis Protocolos.**

Vocês querem lançar com ActivityPub, AT Protocol, Nostr, Matrix, IRC e XMPP ao mesmo tempo. Isso não é ambição. É uma receita para que nenhum funcione bem. Cada protocolo dilui o foco da engenharia. Cada protocolo multiplica os edge cases. Cada protocolo dá ao usuário uma razão para dizer "isso não funciona". Escolham UM. ActivityPub. Façam ele perfeito. Depois adicionem o segundo. Dizer sim para tudo não é visão — é covardia.

**2. A Sala Vazia.**

O onboarding tem seis etapas: gere chaves, escolha servidor, conecte contas, importe grafos, siga packs, poste introdução. E no final — não tem ninguém lá. Zero posts. Zero conversas. Tasting Menu e Starter Packs são curativos. O problema real são os primeiros TRÊS SEGUNDOS. A arquitetura offline-first torna isso MAIS difícil, não mais fácil — porque não tem servidor para consultar conteúdo fresco.

**3. Falta de Opinião.**

Grandes produtos dizem NÃO para quase tudo. Este projeto diz SIM para seis protocolos, modo offline, importação de dados de quatro plataformas legadas, Tor, BIP-39, tradução ML on-device, governança comunitária... o documento de 460 linhas sobre filosofia de UX é um manifesto, não um produto. A frase que deveria definir o Liberation — "Você escreve um post dentro de um túnel" — está enterrada na página 7. Deveria ser o TÍTULO.

---

## Linus Torvalds — As 3 Coisas Que Matam (Engenharia)

**1. O Atoleiro de Manutenção.**

Seis protocol adapters. Cada protocolo muda independentemente. Bluesky lança uma mudança de Lexicon — seu adapter quebra. Matrix atualiza a spec — quebra. Nostr adiciona um NIP novo — quebra. Mastodon muda um endpoint da API — você está reescrevendo testes de integração às 2 da manhã. Isso não são 6 coisas para manter. São 6 × (número de mudanças upstream por ano). Pra 2 desenvolvedores? Eles vão se afogar. 80% do tempo vai ser apagando incêndio de breaking change dos outros.

**2. O Problema de Sync é uma Tese de PhD.**

Mastodon é REST polling. Nostr é assinatura WebSocket com relays que somem no meio do stream. Matrix é long-poll com resolução de estado. AT Protocol é firehose + XRPC + Merkle Search Trees. E você quer uma camada de sync UNIFICADA em cima disso? Com resolução de conflitos? Com fila offline? Isso não é feature. É uma tese de PhD. Cada protocolo tem garantias de consistência diferentes, semânticas de ordenação diferentes, modos de falha diferentes. Você vai construir uma camada que não lida BEM com NENHUM e lida MAL com TODOS.

**3. Servidores dos Outros São SUA Responsabilidade.**

"Serverless." Tradução: você é 100% dependente de instâncias Mastodon ficarem online, relays Nostr não caírem, Bluesky PDS estar reachable, Matrix homeservers disponíveis. Você não controla NENHUM. Quando mastodon.social te rate-limitar, seu app INTEIRO gagueja. Quando um relay Nostr cair durante uma conferência, seu feed tem BURACOS. Você não eliminou dependências de servidor — você MULTIPLICOU elas. Um app normal tem 1 domínio de falha. Você tem 6, todos gerenciados por estranhos.

---

## O Que Ambos Concordam

**Ambos dizem: COMECE COM UM.** Steve diz ActivityPub. Linus não discorda. Seis protocolos no lançamento = zero protocolos funcionando.

**Ambos dizem: O ONBOARDING É TUDO.** Steve fala da sala vazia. Linus fala do sync que falha. O usuário não perdoa primeira experiência ruim.

**Ambos dizem: MENOS É MAIS.** Steve quer foco no produto. Linus quer foco na engenharia. Os dois apontam para o mesmo lugar: corte escopo. Escolha batalhas.

---

## Veredito Final (Ambos)

**O app MERECE existir.** Mas:

1. **Comece com 1 protocolo** (ActivityPub ou Nostr). Faça PERFEITO.
2. **Resolva a sala vazia** ANTES de qualquer outra feature. Tasting Menu + Starter Packs são o MVP.
3. **Abra mão dos outros 5 protocolos** no lançamento. Roadmap, não launch.
4. **Uma frase**: "Você escreve um post dentro de um túnel. E ele chega."
5. **Dois desenvolvedores. Um protocolo. Um ano.** Depois escala.

---

*Debate final. Loop encerrado.*
