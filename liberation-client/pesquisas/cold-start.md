# Pesquisa: Cold Start & Onboarding em Plataformas Descentralizadas

> Notas de pesquisa para a Iteração 2 — como resolver o feed vazio sem algoritmos.
> Data: 2026-07-01

---

## Bluesky: Starter Packs como Mecanismo de Onboarding

### O que são
- Pacotes curados de contas (até 150) + feeds customizados (até 3)
- Criados por usuários da comunidade, não por algoritmos
- **280,000+ starter packs** disponíveis na plataforma
- Seguir todas as contas com um clique, reversível individualmente

### Eficácia
- **43% dos novos follows** vêm de starter packs
- Novos usuários que chegam via pack seguem **8-12 contas imediatamente**
- Timeline se enche rapidamente pois as contas no pack já interagem entre si
- Fornece densidade social imediata

### Limitações
- Não são searchable dentro do Bluesky — espalham via links e diretórios third-party
- Packs não atualizam automaticamente, curador precisa manter
- Packs de 15-30 contas ativas têm melhor resultado (qualidade > quantidade)

### Adoção
- X (Twitter) anunciou cópia do modelo em Janeiro 2026
- Bluesky: 40M+ usuários, crescendo para 60-100M até fim de 2026

---

## Mastodon: O Problema Persiste

### Situação Atual
- Feed vazio é a **maior barreira de adoção** do Mastodon
- Sem algoritmo de recomendação, timeline fica "conspicuamente vazia"
- Soluções atuais são todas **manuais/tutoriais**:
  - Seguir hashtags (#photography, #rstats)
  - Usar timelines Local e Federada
  - Postar #Introduction
  - Diretórios curados (Fedi.Directory, @FediFollows)

### Mastodon "Packs" (em desenvolvimento)
- Anunciado Outubro 2025, target v4.6
- Similar aos Starter Packs do Bluesky
- Diferencial de privacidade: **opt-out** (usuário pode recusar ser incluído)
- Usuário é **notificado** quando adicionado a um Pack
- Sendo desenvolvido como FEP (Fediverse Enhancement Proposal)

### Problemas Técnicos
- Instâncias pequenas: trending/explore fica vazio (precisa de 5+ interações)
- Não há mecanismo padronizado de descoberta cross-instância

---

## Nostr: O Mais Difícil

### Situação
- **Sem onboarding estruturado** — gerar chave, conectar relays, feed potencialmente vazio
- Gerenciamento de relays é exposto ao usuário → fuga de não-técnicos
- NIP-05 (verificação DNS) ajuda mas é complexo para novos usuários
- NIP-65 (relay lists) é essencial mas subutilizado
- Spam é problema existencial sem moderação server-side

### Comunidade Discutindo
- "Outbox is harder than it should be" — arquitetura de relays precisa ser repensada
- Clients hardcodam listas de relays em vez de usar NIP-65
- Bibliotecas Nostr ainda usam pool fixo de relays
- Bootstrapping de novos usuários é "o happy path é evil"

---

## Padrões Local-First (SQLite)

### Colunas de Sync Padrão
```sql
revision INTEGER DEFAULT 1,
updated_at DATETIME,
created_at DATETIME,
deleted_at DATETIME,       -- soft delete / tombstone
pending_sync BOOLEAN DEFAULT 0,
last_synced_at DATETIME
```

### Estratégias de Conflito
| Estratégia | Quando Usar |
|-----------|------------|
| Last Write Wins (LWW) | Campos não-críticos |
| Field-level merge | Default para edições |
| CRDT / per-field merge | Edição colaborativa |
| Manual review | Campos de alto risco |

### Sync Delta
- **Revision-based**: Cada linha tem revision int. Buscar `WHERE revision > last_revision`
- **Timestamp-based**: `WHERE updated_at > last_sync_time ORDER BY updated_at, id`
- **Tombstones**: Soft delete com deleted_at. Manter por 30-90 dias

### Anti-Padrões
1. Confiar em relógios de dispositivo para ordenação
2. Snapshots completos para sync (enviar só changed fields)
3. Ignorar tombstones → "record resurrection"
4. Bloquear UI durante sync
5. Estratégia única de conflito para todos os dados

---

## Ideias-Chave Para Nossa Solução

1. **Tasting Menu**: 20 posts curados cross-protocolo, atualizados semanalmente, resolvem o feed vazio IMEDIATAMENTE
2. **Starter Packs cross-protocolo**: Importar packs do Bluesky, Mastodon Packs (quando disponível), listas Nostr
3. **Importação de grafo social**: Conectar conta existente → importar follows, mutes, lists (local-only)
4. **Relays invisíveis**: Nostr relays são detalhe de implementação, nunca expostos ao usuário
5. **"Also on..." cross-protocol**: Descobrir a mesma pessoa em outros protocolos
6. **Métricas éticas**: Completamento, retenção, diversidade de protocolos — tudo local, opt-in
