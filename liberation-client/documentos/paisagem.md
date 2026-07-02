# 🌐 Liberation Client — Documento de Paisagem (Landscape)

> **Síntese da Iteração 1** — Pesquisa de paisagem, análise de protocolos, arquitetura técnica, UX, e análise crítica.
> **Data:** 2026-07-01
> **Status:** Primeira iteração do /loop de desenvolvimento

---

## Índice

1. [Visão Geral](#visão-geral)
2. [O Que Existe Hoje](#o-que-existe-hoje)
3. [Os 6 Protocolos](#os-6-protocolos)
4. [Arquitetura Proposta](#arquitetura-proposta)
5. [Filosofia de Design](#filosofia-de-design)
6. [Análise de Viabilidade](#análise-de-viabilidade)
7. [Recomendações](#recomendações)
8. [Próximas Iterações](#próximas-iterações)

---

## Visão Geral

**O que é:** Um único aplicativo que roda inteiramente no dispositivo do usuário — sem servidor próprio — conectando-se a múltiplos protocolos abertos de comunicação: ActivityPub, AT Protocol, Nostr, Matrix, IRC, XMPP.

**O que NÃO é:** Um agregador de chats, uma rede social própria, uma plataforma com curadoria algorítmica.

**Por que existe:** É um grito de libertação das grandes big techs e da curadoria algorítmica focada em vício e rentabilidade comercial. O usuário é dono dos próprios dados, da própria identidade, da própria atenção.

**Dificuldade estimada:** 8.5/10

---

## O Que Existe Hoje

### Agregadores de Chat
| App | Protocolos | Feed Social? | Serverless? |
|-----|-----------|-------------|-------------|
| Beeper | 15+ (Matrix bridges) | ❌ Chat only | ❌ Cloud bridges |
| Ferdium/Rambox | Web wrappers | ❌ | ❌ |
| Pidgin/Adium | XMPP, IRC (legado) | ❌ | ✅ Mas sem protocolos modernos |

### Clientes de Redes Federadas
| App | Protocolos | Multi-Protocolo? | Serverless? |
|-----|-----------|-----------------|-------------|
| Ivory/Mammoth/IceCubes | ActivityPub (Mastodon API) | ❌ Só 1 | ❌ Precisa de instância |
| Bluesky app | AT Protocol | ❌ Só 1 | ❌ Precisa de PDS |
| Damus/Primal/Amethyst | Nostr | ❌ Só 1 | ✅ Mas relays expostos ao usuário |

### Os "Quase Lá"
| App | Diferencial | Limitação |
|-----|-----------|-----------|
| **Holos** | Roda servidor ActivityPub no dispositivo | Só ActivityPub, só iOS |
| **AndStatus** | Multi-protocolo desde 2013 | Só Android, UI datada, sem Nostr/ATP |
| **Plebbit** | P2P via IPFS, serverless de verdade | Só protocolo próprio, sem pontes |

### A Lacuna
**NENHUM app combina:** Feed social + Chat + Múltiplos protocolos + Serverless + Design anti-engajamento.

---

## Os 6 Protocolos

### Comparação Rápida

| Protocolo | Tipo | Transporte | Identidade | Força | Fraqueza | Score |
|-----------|------|-----------|-----------|-------|----------|:-----:|
| **Nostr** | Social (relay) | WebSocket | npub (chave) | Simplicidade máxima | Spam existencial | **8/10** |
| **AT Protocol** | Social (repo) | XRPC/HTTP | DID | Melhor modelo de dados | Centralizado (Bluesky) | **7/10** |
| **ActivityPub** | Social (federado) | REST/WS | @user@domain | Maior base de usuários | Precisa de servidor | **5/10** |
| **Matrix** | Chat (salas) | REST/Sync | @user:server | Melhor E2EE | Não é rede social | **5/10** |
| **IRC** | Chat (canais) | TCP puro | nick@server | Trivial | Sem persistência | **2/10** |
| **XMPP** | Chat (stanzas) | TCP/WS | user@domain | Federação estável | XML, em declínio | **3/10** |

### Recomendação de Prioridade

```
Tier 1 (Nativo):     Nostr + AT Protocol
Tier 2 (Gateway):    ActivityPub (maior base, mas precisa de bridge server-side)
Tier 3 (Chat):       Matrix (módulo de chat separado)
Tier 4 (Bridge):     IRC + XMPP (via bridges existentes do Matrix)
```

---

## Arquitetura Proposta

### Stack Técnica
- **Linguagem core:** Rust (todos os adapters de protocolo + camada de dados)
- **UI:** Flutter (mobile-first) ou Tauri (desktop-first)
- **Banco local:** SQLite + FTS5 (fonte da verdade)
- **Identidade:** Chaves por protocolo, stitching local de perfis

### Padrão Arquitetural

```
┌─────────────────────────────────┐
│     UI (Flutter / Tauri)        │
├─────────────────────────────────┤
│     Event Bus (canônico)        │
├──────┬──────┬──────┬────────────┤
│Nostr │ ATP  │ APub │  Matrix    │
│Adapter│Adapter│Gateway│Chat Module│
├──────┴──────┴──────┴────────────┤
│     SQLite + FTS5 (local)       │
└─────────────────────────────────┘
```

### Protocol Adapter Pattern
Cada protocolo implementa uma interface comum (`ProtocolAdapter`), traduzindo eventos nativos para um modelo canônico interno. O Event Bus distribui para as views. A camada de dados é local-first — a rede é mirror, não fonte.

### Roadmap
| Fase | Escopo | Esforço |
|------|--------|---------|
| 1. Fundação | SQLite, Event Bus, keychain | 3 meses |
| 2. Nostr | Relay manager, NIPs | 2 meses |
| 3. AT Protocol | XRPC, Jetstream, facets | 2 meses |
| 4. ActivityPub | Mastodon API | 2 meses |
| 5. Matrix | Chat module, E2EE | 3 meses |
| 6. Polimento | Search, cache, bateria | 2 meses |

**Total:** ~14-17 meses para time de 2-3 engenheiros.

---

## Filosofia de Design

### Os 10 Princípios da Libertação

1. **Intenção sobre Engajamento** — Toda interação começa com intenção do usuário
2. **Cronológico é Neutro** — A timeline padrão é cronológica pura
3. **Fricção é Feature** — Adicionar fricção a ações compulsivas; removê-la das intencionais
4. **Transparência é Inegociável** — Toda decisão de design é explicável
5. **Números Abstraem Pessoas** — Mostre pessoas, não contagens
6. **Propriedade por Padrão** — Dados são locais primeiro; portabilidade é um toque
7. **Multi-protocolo, Interface Unificada** — Respeitar diferenças com experiência coerente
8. **Simples é Acessível** — "Modo Simples" que esconde complexidade de protocolos
9. **Recuse a Economia da Atenção** — Se existe para aumentar engajamento, não construímos
10. **Design para a Partida** — O app deve ajudar você a passar menos tempo nele, não mais

### Anti-Patterns (O Que NUNCA Faremos)
- ❌ Infinite scroll → Timeline paginada com limites
- ❌ Pull-to-refresh → Botão explícito "ver novos posts"
- ❌ Badges de não-lidos → Nada no ícone do app
- ❌ Contagem de likes/seguidores → Pessoas, não números
- ❌ "Para Você" algorítmico → Zero sugestões algorítmicas
- ❌ Streaks, gamificação, dark patterns → NENHUM

---

## Análise de Viabilidade

### O Que Pode Matar o Projeto (Top 3)

1. **Dívida de manutenção de protocolos** (Risco: 9/10) — 4-5 protocolos evoluindo independentemente. Bridges quebram. O custo de manter tudo atualizado pode exceder os recursos disponíveis.

2. **O problema do feed vazio** (Risco: 9/10) — Na primeira abertura, não há conteúdo. Sem algoritmo de descoberta, como preencher a timeline? Se o usuário não vê nada, deleta o app em 30 segundos.

3. **Performance mobile pobre** (Risco: 8/10) — Múltiplas conexões persistentes drenam bateria. Background sync é limitado em iOS. O app pode ser percebido como "lento" e "gastador de bateria".

### O Que Joga a Favor
- Nostr tem a arquitetura mais amigável para client-side (10/10)
- Rust tem bibliotecas excelentes para todos os protocolos principais
- O movimento anti-big-tech está crescendo (Bluesky, Nostr, Mastodon)
- A lacuna é real e não preenchida — oportunidade de ser o primeiro

---

## Recomendações

### MVP Recomendado (6 meses, 2 engenheiros)
- **Nostr** como primeiro protocolo (mais simples, mais alinhado filosoficamente)
- **ActivityPub** como segundo (maior base de usuários)
- Timeline cronológica unificada
- Postagem com texto e imagens
- SQLite local, leitura offline
- iOS OU Android (escolher um)

### Para a Próxima Iteração
- Aprofundar o modelo de dados canônico
- Esboçar a UI da timeline unificada
- Prototipar o Nostr adapter em Rust
- Definir o modelo de identidade local
- Explorar naming e branding

---

## Documentos Relacionados
- [UX Philosophy & Design Principles](../liberation-social-client-ux-philosophy.md)
- [Análise de Arquitetura](arquitetura/analise.md)
- [Comparação de Protocolos](pesquisas/protocolos.md)
- [Análise Crítica de Concorrentes](pesquisas/concorrentes.md)
- [Visão & Naming](design/visao.md) (em construção)

---

*Documento vivo gerado pelo /loop de desenvolvimento. Iteração 1 concluída em 2026-07-01.*
