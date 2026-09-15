# Estudo de Caso 41 — Linear: O Sync Engine Local-First Com IndexedDB+MobX (Cell-Level Observability), Object Pool, Transaction Queue, GraphQL+WebSocket Delta Packets (Sync ID Monotônico), Rebase Offline e LWW — Servidor De ~US$ 80/Mês

> **Data:** 2026-07-03
> **Loop:** 41 de ∞ (Reescrita)
> **Categoria:** Ferramentas de Desenvolvimento / Local-First / Sync Engine

---

## 0. Linhagem

```
Jira (2002) — customizável, pesado, lento. O padrão que todos usam e ninguém gosta.
Asana (2008), Monday (2012) — alternativas. Mais rápidas. Menos opinativas.
Linear (2019) — local-first. Keyboard-first. Sync engine próprio. Opinionated.
Linear hoje (2026) — 150K+ teams. US$ 400M valuation. 80 funcionários. LSE reverse-engineered.
```

"Literally the first lines of code that I wrote was the sync engine, which is very uncommon to what you usually do when you're a startup." — Tuomas Artman, cofundador e CTO.

---

## 1. Arquitetura Técnica

### 1.1 O Sync Engine: IndexedDB + MobX + Object Pool + Transaction Queue

O **Linear Sync Engine (LSE)** é uma camada local-first que trata o navegador como banco de dados primário. O servidor é um **sync target**, não a source of truth para a UI.

**Object Graph (MobX).** Camada mais alta: modelos em memória com propriedades MobX observáveis. Cada propriedade é seu próprio observable — **cell-level re-renders**: mudar `issue.title` re-renderiza apenas o componente que lê aquele campo. 50 issues atualizadas = 50 cell re-renders, não um re-render de lista inteira. O UI nunca lê da rede; sempre lê deste pool local.

**Object Pool.** Mapa normalizado em memória de todas as instâncias de modelo, keyed por UUID. Modelos são **hydrated** de JSON (IndexedDB) para objetos JavaScript vivos com métodos (`.save()`, etc.). Suporta **lazy hydration** (propriedades carregadas apenas quando acessadas) e **partial loading** (Issue, Comment carregados sob demanda; nem todos no boot).

**Transaction Queue.** Toda mutação é encapsulada como **transação** (Create, Update, Delete) que:
1. É aplicada imediatamente ao MobX object graph (optimistic update — latência zero)
2. É persistida em IndexedDB (`_transaction` table — crash recovery)
3. Entra na `TransactionQueue` em memória
4. É agrupada em **batch** e enviada via **GraphQL mutation** ao servidor
5. Se offline, acumula; quando online, flush automático

**Model System (TypeScript decorators):**

```typescript
@ClientModel("Issue")
class Issue extends Model {
  @Property() title: string;
  @Property({ serializer: PrioritySerializer }) priority: Priority;
  @Reference(() => User, "assignedIssues", { nullable: true, indexed: true }) assignee: User | null;
}
```

`@ClientModel` registra no `ModelRegistry` com `loadStrategy` (instant, lazy, partial, local). `@Property`/`@Reference` registram metadados (type, serializer, lazy load, indexed).

### 1.2 Bootstrap e Fluxo de Dados

**Bootstrap (initial load):**
1. Cria ObjectStore para cada modelo
2. Abre conexão IndexedDB (meta database + workspace database `linear_<hash>`)
3. Determina tipo: `full` (core models), `partial` (Comment/IssueHistory diferidos) ou `local`
4. Fetch `/sync/bootstrap?type=full&onlyModels=...`
5. Persiste em IndexedDB
6. **Hydrate** JSON → objetos MobX vivos no Object Pool
7. Abre WebSocket para updates incrementais

**Write Path:**
```
Frontend: issue.title = "New Title"; issue.save()
  → MobX observable → UI re-render instantâneo
  → Object Pool update
  → Transaction escrita em IndexedDB _transaction
  → TransactionQueue → batch → GraphQL mutation → Server
  → Server: processa, atribui Sync ID, broadcast delta via WebSocket
  → Client: recebe delta, atualiza IndexedDB model tables, remove de _transaction
```

**Read Path (remote updates):**
```
Outro cliente faz mudança → Server broadcast delta via WebSocket
  → Client: aplica delta ao IndexedDB + Object Pool
  → MobX reactivity → re-render dos componentes afetados
```

**Delta Packets via WebSocket.** Arrays de `SyncAction` objetos: `id` (Sync ID monotônico), `modelName`, `modelId`, `action` (I/U/D), `data`. No reconnect, cliente faz delta sync: `/sync/delta?lastSyncId=X&toSyncId=Y`.

### 1.3 Conflitos: Rebase Offline + LWW

Linear usa **Last-Writer-Wins** — não CRDTs (exceto para descrições de issues em texto rico, onde usa **Yjs + ProseMirror**). Conflitos são raros em issue tracking: propriedades discretas (status, assignee, priority) raramente são editadas concorrentemente.

**Rebase offline:** quando cliente reconecta após período offline:
1. Recebe último estado do servidor via delta sync
2. **Rebaiseia** transações locais pendentes no topo do estado mais recente
3. Transações rebaiseadas são enviadas ao servidor
4. Servidor aplica (LWW) e incrementa Sync ID

**Princípios de design (Tuomas Artman):** Centralized ordering (servidor gera Sync IDs globais), client-side prediction (memory-first writes, zero-latency UX), local-first (dataset completo em IndexedDB, offline built-in), transaction-based (toda mutação é atômica, reversível, cacheable, rebase-able), pragmatic conflict resolution (LWW sobre OT/CRDT para SaaS).

### 1.4 Stack e Custo

**Frontend:** React + MobX + TypeScript. IndexedDB via wrapper `idb`. GraphQL via `graphql-request`. Yjs + ProseMirror (CRDT para texto rico). Comlink (Worker RPC). Rolldown-Vite bundler (50% menos JS enviado, 59% mais rápido first-paint em issues view).

**Backend:** Node.js + TypeScript. PostgreSQL (Cloud SQL, particionado 300 vias). Redis (event bus + cache + sync cursors). Kubernetes (GCP). Cloudflare Workers (multi-region edge proxy).

**Custo:** datacenter europeu serve ~1.000 CCU em ~2 CPU cores, ~US$ 80/mês. "Since data is persisted on local devices, you don't pay for database reads as often."

---

## 2. Lições de Engenharia

### 2.1 O sync engine é a feature mais importante — comece por ele

Tuomas Artman escreveu o sync engine como primeiras linhas de código. A maioria das startups começa pela UI; o resultado é arquitetura que nunca funciona offline.

### 2.2 Cell-level MobX observability é o segredo da performance multi-editor

Cada propriedade é um observable separado. 50 pessoas editando issues diferentes geram apenas os re-renders necessários. Redux — com state tree único — não consegue essa granularidade.

### 2.3 LWW é suficiente para a vasta maioria de apps colaborativos

CRDTs são necessários para texto rico (Yjs + ProseMirror), não para propriedades discretas. Linear adicionou CRDTs depois, apenas onde necessário.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Linear |
| **Fundação** | 2019. Público: 2020. Fundadores: Karri Saarinen, Tuomas Artman, Jori Lallo |
| **Categoria** | Issue Tracking / Local-First |
| **Valuation** | US$ 400M. 80 funcionários. 150K+ teams |
| **Sync Engine** | LSE: IndexedDB + MobX + Object Pool + Transaction Queue + GraphQL + WebSocket deltas + LWW |
| **CRDT** | Yjs + ProseMirror (apenas descrições de issues) |
| **Stack** | React, MobX, TypeScript, Node.js, PostgreSQL, Redis, K8s/GCP, Cloudflare Workers |
| **Servidor EU** | ~2 CPU cores, ~US$ 80/mês para ~1.000 CCU |
| **Concorrentes** | Jira, Asana, Monday.com |

---

## Fontes

- [FujiMon — Linear's Sync Engine Architecture (reverse engineering aprovado pelo CTO, 2024)](https://www.fujimon.com/blog/linear-sync-engine)
- [GitHub — backupManager/reverse-linear-sync-engine-dev (endorsed by Tuomas Artman)](https://github.com/backupManager/reverse-linear-sync-engine-dev)
- [Hacker News — Reverse engineering of Linear's sync engine (2025)](https://news.ycombinator.com/item?id=44123131)
- [WeChat/Weixin — 揭秘 Linear 数据同步引擎 (Chinese deep-dive, IndexedDB, MobX, sync IDs)](http://mp.weixin.qq.com/s?__biz=MzA5ODI5NDYxNA==&mid=2649781038&idx=1&sn=b77f6eb5dcf54493d22265eb4ff26e55)
