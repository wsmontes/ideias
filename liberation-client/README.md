# Liberation Client — Cliente Universal de Redes Abertas

**Status:** 🌱 germinando
**Início:** 2026-07-01
**Projeto:** Um aplicativo de redes sociais completamente sem servidor próprio, cliente de redes sociais federadas, cliente de IRC e outras redes abertas de comunicação.

## O que é?

Um único aplicativo que roda inteiramente no dispositivo do usuário — sem servidor próprio — e se conecta a múltiplos protocolos abertos de comunicação:
- **ActivityPub** (Mastodon, Pleroma, GoToSocial, Akkoma...)
- **AT Protocol** (Bluesky)
- **Nostr**
- **Matrix**
- **IRC**
- **XMPP**

Não é só um agregador de chats. É um cliente universal para redes sociais federadas + mensageiros abertos.

## Por quê?

É um grito de libertação das grandes big techs e da curadoria algorítmica focada em vício e rentabilidade comercial.

- **Sem algoritmo viciante** — feed cronológico, sem "Para Você", sem infinite scroll
- **Sem servidor próprio** — o app é o servidor. Seus dados ficam com você.
- **Soeverania digital** — você escolhe quais protocolos usar, quais redes acessar, qual instância te representa
- **Design anti-engajamento** — o app foi projetado para você passar menos tempo nele, não mais

## Documentos

### Iteração 1 — Paisagem
- [UX Philosophy & Design Principles](liberation-social-client-ux-philosophy.md) ✅
- [Documento de Paisagem](documentos/paisagem.md) ✅
- [Análise de Arquitetura](arquitetura/analise.md) ✅ (1557 linhas)
- [Comparação de Protocolos](pesquisas/protocolos.md) ✅
- [Análise Crítica de Soluções Existentes](pesquisas/concorrentes.md) ✅
- [Visão, Naming & Branding](design/visao.md) ✅ (21 nomes, manifesto)

### Iteração 2 — Fundamentos
- [Pesquisa Cold Start & Onboarding](pesquisas/cold-start.md) ✅
- [Fluxo de Onboarding](onboarding/fluxo-onboarding.md) ✅ (430 linhas — 6 partes)
- [Modelo de Dados Canônico](especificacao/modelo-de-dados.md) ✅ (2398 linhas — 32 tabelas, 6 protocolos)

### Iteração 3 — Segurança & Identidade
- [Pesquisa de Segurança](pesquisas/seguranca.md) ✅ (Black Hat 2025, CVEs, Mobile Keychain)
- [Identity & Keychain Architecture](especificacao/identidade-keychain.md) ✅ (2391 linhas — BIP-39, Secure Enclave, Social Recovery)
- [Threat Model](especificacao/threat-model.md) ✅ (1441 linhas — 123 superfícies de ataque, 30 riscos, 8 partes)

### Iteração 4 — Design & Comunidade
- [Wireframes UI](design/wireframes.md) ✅ (2020 linhas — 11 telas, ASCII wireframes, specs de interação)
- [Curadoria & Governança Comunitária](comunidade/governanca.md) ✅ (849 linhas — Nostr backbone, 5 papéis, 3 fases)

### Iteração 5 — Core Systems
- [Event Bus & Nostr Adapter Spec](especificacao/core-systems.md) ✅ (1611 linhas — EventBus, RelayConnectionManager, SyncEngine, OfflineQueue, crate structure, FFI)

### Iteração 6 — AT Protocol & Matrix
- [AT Protocol Adapter & Matrix Chat Module](especificacao/atproto-matrix.md) ✅ (873 linhas — AT Protocol: shrike, Jetstream, DID resolver, facets; Matrix: matrix-rust-sdk, E2EE, room classification)

## Princípios

1. **Intenção sobre Engajamento** — Toda interação começa com intenção do usuário
2. **Cronológico é Neutro** — A timeline padrão é cronológica pura
3. **Fricção é Feature** — Adicionar fricção a ações compulsivas; removê-la das intencionais
4. **Transparência é Inegociável** — Toda decisão de design é explicável
5. **Números Abstraem Pessoas** — Mostre pessoas, não contagens
6. **Propriedade por Padrão** — Dados são locais primeiro; portabilidade é um toque
7. **Multi-protocolo, Interface Unificada** — Respeitar diferenças entre protocolos com uma experiência coerente
8. **Recuse a Economia da Atenção** — Se um padrão existe primariamente para aumentar engajamento, não o construímos
