# ⚔️ Debate: Construir um Cliente Novo ou Contribuir com os Existentes?

> Iteração 8 do loop de ideação
> 2026-07-01

---

Esta é **a decisão mais importante** que o projeto precisa tomar. E as duas respostas têm mérito.

Em vez de fingir que já temos a resposta, vamos travar o debate honestamente.

---

## Lado A: CONSTRUIR — Por que o Liberation Client precisa existir

### A1. Ninguém está fazendo isso

**Fato**: NENHUM cliente social multi-protocolo existe. Zero.

- Ivory, Mammoth, IceCubes, Elk, Phanpy, Tusky → só ActivityPub
- Damus, Primal, Amethyst, Snort, Coracle → só Nostr
- App oficial do Bluesky → só AT Protocol
- Element, FluffyChat, Cinny → só Matrix

O AndStatus é o mais próximo (ActivityPub + Pump.io + GNU Social) e é Android-only, mantido por uma pessoa, com UI de 2013.

**Adicionar suporte multi-protocolo a um cliente existente é uma REESCRITA da arquitetura**, não uma feature. Nenhum cliente existente foi projetado com abstração de protocolo. Todos assumem UM protocolo no core.

### A2. A arquitetura multi-protocolo É a inovação

O que o Liberation Client propõe não é "mais um cliente Mastodon". É um **padrão arquitetural novo**: ProtocolAdapter trait, EventBus, canonical data model, sync engine unificado.

Isso não se "contribui" para um projeto existente. É uma fundação diferente. Seria como "contribuir" com um motor elétrico para um carro a gasolina — você reescreve o carro inteiro ou constrói um novo.

### A3. As decisões de design são radicalmente diferentes

- **Anti-engajamento**: nenhum cliente existente abraça isso como princípio fundador
- **Sem métricas de vaidade**: todo cliente mostra likes, reposts, seguidores
- **Intent-based entry**: todo cliente abre direto no feed
- **Fricção intencional**: todo cliente otimiza para velocidade
- **Tasting Menu / Starter Packs / Cold Start**: nenhum cliente tem onboarding multi-protocolo

"Contribuir" com essas mudanças para um cliente existente significaria **convencer os maintainers a mudar a filosofia do produto**. Isso é mais difícil do que construir do zero.

### A4. O valor está na DOCUMENTAÇÃO, não só no código

Mesmo que o app nunca atinja 10.000 usuários, o que já foi produzido (19 documentos de especificação, ~18.900 linhas) tem valor para TODO o ecossistema:
- O modelo de dados canônico pode ser adotado por outros clientes
- O threat model serve para qualquer app descentralizado
- A comparação de protocolos é referência
- A arquitetura de identity/keychain é reutilizável

**Construir gera conhecimento público. Contribuir gera features privadas.**

---

## Lado B: CONTRIBUIR — Por que construir do zero é um erro

### B1. O problema XKCD 927

[Existem 15 padrões concorrentes](https://xkcd.com/927/). Agora existem 16.

Cada novo cliente **fragmenta** a comunidade:
- Desenvolvedores que poderiam estar melhorando o Damus estão construindo o Liberation
- Usuários que poderiam estar no Damus estão esperando o Liberation
- Doadores que poderiam estar financiando o Damus estão sendo pedidos para financiar o Liberation

O resultado: **dois clientes meia-boca em vez de um cliente excelente.**

### B2. "Contribuir" não significa só enviar PRs

Contribuir pode significar:
- **Fork**: pegar o Damus (GPLv3), o IceCubes (AGPLv3), ou o Phanpy (MIT) e transformar no Liberation Client
- **Plugin**: construir o suporte multi-protocolo como uma camada que pluga em clientes existentes
- **Biblioteca**: construir o ProtocolAdapter + EventBus como uma lib Rust que qualquer cliente pode usar
- **Especificação**: o modelo de dados canônico como um RFC que outros clientes implementam

O valor do projeto está na **camada de abstração de protocolo**, não na UI. A UI é commodity. A abstração é o ouro.

### B3. A comunidade JÁ EXISTE nos clientes atuais

- Damus: 25+ contribuidores, iOS maduro, 3 anos de desenvolvimento, comunidade ativa
- Elk: 200+ contribuidores, web + PWA, Vue/Nuxt, stack moderno
- Phanpy: minimalist, web-first, excelente UX, MIT license
- IceCubes: SwiftUI, Apple platforms, AGPL

Construir do zero significa **competir com essas comunidades por atenção, contribuidores e financiamento.** Contribuir significa **somar forças.**

### B4. O custo de construir é subestimado

O plano fala em 15-17 meses para 2-3 engenheiros. Isso é para o CORE (Rust). A UI (Flutter/SwiftUI) é OUTROS 12-18 meses. Total: **2-3 anos até um MVP funcional.**

Em 2-3 anos contribuindo para clientes existentes, você teria:
- Multi-protocolo no Damus (ou IceCubes, ou Phanpy)
- Onboarding melhorado, anti-engajamento, identidade unificada
- Uma comunidade de contribuidores que já existe
- Usuários REAIS usando as features

### B5. O que o ecossistema PRECISA não é mais um cliente

O ecossistema precisa de:
1. **Uma biblioteca Rust de abstração de protocolo** (ProtocolAdapter + EventBus + Canonical Model)
2. **Um RFC de modelo de dados canônico**
3. **Bridges e gateways** (ActivityPub ↔ Nostr ↔ AT Protocol)
4. **Melhores ferramentas de onboarding e cold start**

Construir UM CLIENTE resolve 1 desses 4 problemas (o #1, parcialmente). Contribuir resolve potencialmente TODOS.

---

## O meio-termo: A Biblioteca como Produto, o Cliente como Demo

Uma terceira via:

1. **Produto principal**: `libs/protocol-adapters` — uma biblioteca Rust que implementa o ProtocolAdapter trait para Nostr, ActivityPub, AT Protocol. Licença AGPL ou MIT. Qualquer cliente pode usar.

2. **Cliente de referência**: Liberation Client como uma DEMO do que a biblioteca pode fazer. Não tenta ser o "melhor cliente". É a prova de conceito.

3. **Contribuições upstream**: features de anti-engajamento, onboarding, e identidade unificada como PRs para clientes existentes, usando a biblioteca como dependência.

**Vantagens:**
- A biblioteca é o legado (não o app)
- O app valida a biblioteca (e pode morrer sem matar o projeto)
- A comunidade de cada cliente decide se quer adotar a lib
- O modelo de negócios é: lib gratuita, consultoria paga para integração

**Desvantagens:**
- Ninguém financia "bibliotecas". Financiam "apps".
- O impacto é menos visível
- Leva mais tempo para chegar em usuários finais

---

## As 3 perguntas que decidem

### 1. O que é MAIS IMPORTANTE: que o app exista, ou que a ABORDAGEM (multi-protocolo, anti-engajamento) seja adotada?

Se a resposta for "o app exista" → construir.
Se a resposta for "a abordagem seja adotada" → biblioteca + contribuir.

### 2. Quanto tempo você está disposto a esperar para ter USUÁRIOS REAIS?

Construir: 2-3 anos até o MVP. Depois, conquistar usuários.
Contribuir: features aparecem em clientes existentes em meses. Usuários já estão lá.

### 3. O que te MOTIVA: a visão do produto completo ou a solução do problema técnico?

Se a visão do produto (jardim, cidade, ritual, dádiva) é o que te move → construir.
Se a solução técnica (abstração de protocolo, modelo canônico) é o que te move → biblioteca + contribuir.

---

## Veredito (provisório, contestável)

**A biblioteca deve existir, com ou sem o app.**

O `libs/protocol-adapters` (ProtocolAdapter trait, EventBus, Canonical Model, SyncEngine) é o legado duradouro. Mesmo que o Liberation Client morra, a biblioteca vive. Ela é o "XMPP" do projeto — um padrão que outros implementam.

**O app é a validação, não o produto.**

O Liberation Client existe para PROVAR que a biblioteca funciona. Se 10.000 pessoas usarem, ótimo. Se 100 usarem, a biblioteca ainda tem valor.

**O que NÃO faz sentido: competir com Damus/Ivory/Elk em features de UI.**

A UI do Liberation Client deve ser MÍNIMA. O foco é a abstração de protocolo. Deixa a UI bonita para os clientes nativos. O Liberation Client é o "cliente de referência" — funcional, não bonito.

---

## Para a próxima iteração

Debater outra decisão difícil:
- **Quais 2 protocolos primeiro?** Nostr + ActivityPub? Nostr + AT Protocol?
- **Quanto cobrar?** R$ 5/mês? R$ 10/mês? Grátis com doações?
- **Mobile-first ou desktop-first?**

---

## Dados da pesquisa (chegaram depois)

### Quem já está fazendo multi-protocolo
- **OpenVibe** (2024): agrega Mastodon, Bluesky, Nostr e Threads. $800K da Automattic. iOS + Android.
- **Tapestry** (2025): leitor multi-feed (Bluesky, Mastodon, RSS, YouTube, Reddit). $177K Kickstarter.
- **Bridgy Fed**: ponte ActivityPub ↔ AT Protocol. CC0 (domínio público).
- **Flare**: Kotlin Multiplatform, AGPLv3. Já suporta Mastodon, Bluesky, Misskey. Nostr planejado. Arquitetura de plugins.

### Clientes existentes mais receptivos a contribuições multi-protocolo
1. **Flare** — MELHOR CANDIDATO. Plugin-based, AGPLv3, já multi-protocolo, roadmap público incluindo Nostr.
2. **Fedilab** — Android, já suporta 7+ plataformas. Mantenedor solo, doações comunitárias.
3. **Bridgy Fed** — CC0. Já faz ponte ActivityPub ↔ AT Protocol. Só adicionar mais protocolos.
4. **Phanpy** — Web, Preact, MIT. Código limpo. Mantenedor já indicou interesse em Bluesky.

### Quando construir novo foi CERTO
- **Ivory (Tapbots)**: 12 anos de expertise em cliente Twitter. Nenhum cliente Mastodon existente tinha aquele nível de polish. Assinatura viável.
- **Tapestry (Iconfactory)**: necessidade genuinamente não atendida (leitor multi-feed, multi-protocolo). $177K Kickstarter validou.
- **Bluesky (AT Protocol)**: acreditavam que ActivityPub tem limitações arquiteturais fundamentais. 30M+ usuários.

### Quando construir novo foi PERDA DE TEMPO
- **Maioria dos forks do Mastodon**: disputas de governança, não necessidades técnicas. Nenhum fork superou o Mastodon.
- **Maioria dos clientes N+1**: dezenas de clientes Mastodon. Nenhum atinge massa crítica. Ivory "não sobrevive só de Mastodon" e anunciou Phoenix (Bluesky).
- **Soapbox (Pleroma fork)**: removido por ataques pessoais. Instâncias desfederaram pelo maintainer.

### O problema XKCD 927 é REAL
- ActivityPub tem 20+ clientes. 3 serviços de vídeo curto em 3 protocolos diferentes (Loops/AP, diVine/Nostr, Spark/ATProto).
- "Antes que alguém mencione XKCD #927, sim, eu sei." — usuário do fediverse
- Bridgy Fed gerou ameaças legais no GitHub por TENTAR conectar protocolos

### Nova terceira via: juntar-se a um cliente multi-protocolo EXISTENTE

Em vez de construir DO ZERO ou contribuir para um cliente single-protocolo:
- **Contribuir com Flare**: já tem arquitetura de plugins. Só implementar os protocolos que faltam.
- **Construir a biblioteca Rust e plugá-la em múltiplos clientes**: a `libs/protocol-adapters` como dependência do Flare, Fedilab, Phanpy.
- **Fork com propósito**: fork de um cliente existente (ex: Phanpy, MIT) especificamente para adicionar multi-protocolo. Não é "build from scratch" nem "send PRs". É "fork and transform".

---

*Debate. Iteração 8 do loop de ideação. Pesquisa concluída.*
