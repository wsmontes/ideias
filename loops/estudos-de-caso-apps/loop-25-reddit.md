# Estudo de Caso 25 — Reddit: A Migração Python→Go Com Tap-Compare Testing, Sister Data Stores, 18 Caminhos de Validação (p99 de 15s → 7,5s) e o Graph Neural Network (Kumo, Bilhões de Nós/Arestas) Que Gerou o Maior Feature Lift da História do pCTR

> **Data:** 2026-07-03
> **Loop:** 25 de ∞ (Reescrita)
> **Categoria:** Rede Social / Engenharia de Migração / Graph ML

---

## 0. Linhagem

```
Usenet (1980s) — newsgroups, threaded discussion. A estrutura original.
Slashdot (1997), Digg (2004) — social bookmarking + votação.
Reddit (2005) — subreddits como unidade atômica. Lisp → Python.
Reddit hoje (2026) — 126M DAU uniques. 100K+ comunidades. Python→Go migration. Graph ML.
```

---

## 1. Arquitetura Técnica

### 1.1 A Migração Python→Go: Tap-Compare Testing e 18 Caminhos de Validação

O Reddit operou por quase 20 anos sobre um monólito Python. Em 2024, iniciou decomposição em **microserviços Go domain-specific**, começando por quatro modelos core: Comments, Accounts, Posts e Subreddits. Comments e Accounts: migrados. Posts e Subreddits: em andamento (2025).

**Tap-Compare Testing Para Read Endpoints.** Tráfego real é enviado simultaneamente para serviço Python (antigo) e Go (novo). Respostas são comparadas, mas **apenas a resposta Python é retornada ao usuário**. Discrepâncias são registradas e investigadas. Valida contra tráfego de produção real — incluindo edge cases e race conditions — com zero risco.

**Sister Data Stores Para Write Endpoints.** Writes são mais complexos porque comentários tocam três data stores: **PostgreSQL** (persistência), **Memcached** (cache) e **Redis** (eventos CDC). Para validar writes sem corromper produção, a equipe criou **sister data stores**: réplicas isoladas espelhando schemas de produção, usadas exclusivamente para testes. Serviço Go escreve nas sisters; Python escreve em produção. Resultados são comparados.

**18 Caminhos de Validação.** 3 endpoints de escrita × 3 data stores × 2 serviços = 18 caminhos independentes. Tooling específico gerenciou a complexidade.

**Três Desafios Não-Óbvios:**

1. **ORM vs. SQL raw.** Python usava SQLAlchemy com otimizações implícitas (lazy loading, caching, join optimization). Go escrevia SQL raw — sem essas otimizações. Queries que funcionavam no Python geravam carga maior no PostgreSQL pelo Go. Correção: query-level optimization e connection pooling manuais.

2. **Incompatibilidade de serialização.** Dados escritos pelo Go não podiam ser desserializados pelo Python em alguns casos — timestamps, encoding Unicode, representação de nulls. Consumidores CDC falhavam. Correção: coordenação de schemas de serialização entre linguagens.

3. **Race conditions no tap-compare.** Python processava writes de produção enquanto Go processava o mesmo tráfego em paralelo nas sister stores. Concorrência gerava "falsas discrepâncias" — artefatos do paralelismo, não bugs. Correção: datasets de teste derivados de produção.

**Resultados.** p99 latency em writes críticos: de picos de **15 segundos → ~7,5 segundos** (redução de 50%). Menos pods Go para mesmo throughput (goroutines vs. threads Python com GIL). Zero regressão — migração sem perda de dados ou corrupção. Katie Shannon, Senior Software Engineer, liderou a migração.

### 1.2 Graph ML: GNN Com Link Prediction e o Maior Feature Lift do pCTR

Em parceria com a **Kumo** (setembro 2025), o time de Ads ML do Reddit construiu embeddings de grafo usando **Graph Neural Networks (GNNs)** treinadas como tarefa de **link prediction**: prever se um usuário interagirá positivamente com um anúncio.

**Escala do grafo.** Bilhões de nós e dezenas de bilhões de arestas cobrindo toda a base ativa de usuários — entidades e eventos: usuários, posts, advertiseres, subreddits, clicks, upvotes, video views, memberships.

**Message passing.** Cada nó agrega informações de vizinhos através de múltiplas rodadas de message passing. **Negative sampling** para eficiência em escala de bilhões. **Inductive embeddings**: funcionam para novos usuários e anúncios sem retreinamento — critical para plataforma com milhões de novos dados diários.

**Resultados.** Embeddings de grafo entregaram **o maior feature lift já registrado no modelo de pCTR do Reddit**. **Long clicks** (atenção sustentada): **2× melhor lift** que short clicks. Construído e deployado em **dias** (vs. meses com ferramentas open-source). Adotado como **foundational feature** — Shopping pCTR e Light Ranker. "Built and deployed in just a few days... Graph embeddings delivered a larger lift than any other feature ever added to Reddit's pCTR model."

---

## 2. Lições de Engenharia

### 2.1 Tap-compare testing com sister data stores é o padrão ouro para migração de sistemas vivos

18 caminhos de write validados sem corromper um registro de produção. Tráfego real espelhado, sister stores isoladas, zero impacto no usuário. Padrão subutilizado na indústria.

### 2.2 ORMs são otimizadores implícitos — removê-los remove as otimizações

O código Python não era eficiente — o ORM o tornava eficiente em runtime. Migrar para SQL raw é reimplementação de otimizações, não tradução de sintaxe.

### 2.3 Embeddings de grafo capturam sinal que features tradicionais não veem

Features demográficas capturam atributos. Embeddings de grafo capturam estrutura: usuários similares porque compartilham padrões de interação com os mesmos subreddits, mesmo com demografia completamente diferente. O maior lift da história do pCTR sugere que estrutura de grafo é mais preditiva que atributos.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Reddit |
| **Fundação** | Junho 2005. IPO: março 2024 (NYSE: RDDT) |
| **Categoria** | Rede Social / Comunidades / Dados |
| **DAU** | 126M uniques. 100K+ comunidades |
| **Migração** | Python→Go: Comments, Accounts (done). Posts, Subreddits (em andamento). Tap-compare + sister stores + 18 paths |
| **Data stores** | PostgreSQL, Memcached, Redis (CDC) |
| **Graph ML** | GNNs com Kumo: bilhões de nós/arestas, link prediction, inductive embeddings, maior feature lift do pCTR |
| **Concorrentes** | X (Twitter), Discord, Quora |

---

## Fontes

- [InfoQ — Reddit Migrates Comment Backend from Python to Go Microservice to Halve Latency (Nov 2025)](https://www.infoq.com/news/2025/11/reddit-comments-go-migration/)
- [Kumo.ai — Reddit Boosts Ad Performance with Graph Neural Networks and Kumo (Set 2025)](https://kumo.ai/case-studies/reddit-boosts-ad-performance-with-graph-neural-networks-and-kumo/)
- [Reddit Engineering Blog — Modernizing Reddit's Comment Backend Infrastructure (Katie Shannon, 2024)](https://www.reddit.com/r/RedditEng/)
