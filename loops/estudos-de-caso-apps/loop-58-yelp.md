# Estudo de Caso 58 — Yelp: A Migração De Lucene Custom Para Elasticsearch Com Plugin Hot-Reloadable (URLClassLoader), Geosharding + Microsharding, Fraud Detection Com SparseFraudNet HIN Graph e Recommendation Filter

> **Data:** 2026-07-03
> **Loop:** 58 de ∞ (Reescrita)
> **Categoria:** Reviews Locais / Search / Detecção de Fraude

---

## 0. Linhagem

```
Páginas Amarelas — listas telefônicas. Sem reviews. Sem busca.
Yelp (2004) — reviews de negócios locais. Lucene custom. PayPal Mafia.
Yelp hoje (2026) — 308M+ reviews. 6M+ negócios. Elasticsearch. SparseFraudNet.
```

---

## 1. Arquitetura Técnica

### 1.1 A Migração de Lucene Custom Para Elasticsearch Com Plugin Hot-Reloadable

O sistema de busca original do Yelp era construído sobre **Lucene custom** com arquitetura master-slave: master gerenciava writes/indexing, slaves serviam tráfego real. Snapshots de índice eram uploaded para S3; slaves periodicamente baixavam — causando **minutes-long indexing delays**. Cada instância precisava baixar dezenas de GB do S3, aquecer Lucene caches e forçar GC — pushes de código levavam horas.

A migração para **Elasticsearch** implementou uma arquitetura plugin-based que desacopla lógica de ranking do backend de busca:

1. **Interfaces** — `ScorerFactory`, `Scorer`, `Document` — abstraem acesso a fields do código de ranking
2. **Module (ranking JAR)** — contém lógica core de ranking, hot-reloaded sem restart do Elasticsearch
3. **Elasticsearch Plugin** — wiring específico do ES, delega scoring ao module

**Hot-reload mechanism**: ranking JAR uploaded para S3 → endpoint REST custom dispara reload → `URLClassLoader` privado carrega novo JAR sem restart do cluster. Pushes de código: de horas para minutos, múltiplas vezes ao dia.

**Custom data handling**: query context data (ex.: "quais queries estão associadas a este negócio") armazenada como **binary data serializada em doc values**. Queries mapeadas para Long IDs monotônicos para economizar espaço e permitir registros de tamanho fixo. Patch submetido ao Elasticsearch para suportar ScriptDocValues para binary data lookups.

**Sharding**: **geosharding** (negócios particionados por localização geográfica) + **microsharding** (`business_id % n` para escala horizontal).

### 1.2 O Recommendation Filter e Fraud Detection

**Recommendation Filter**: reviews que o algoritmo considera suspeitas são movidas para seção "não recomendadas" — não deletadas, mas não afetam a nota média do negócio. Transparência como mecanismo de defesa contra acusações de censura.

**Cold-start problem**: novos usuários e novos negócios têm conexões de grafo esparsas — métodos graph-based tradicionais falham. O Yelp lida com isso via múltiplas abordagens acadêmicas:

- **SparseFraudNet (ACM 2025)**: **Heterogeneous Information Network (HIN)** com 5 tipos de relação (same user, same product, same rating, same time window, high TF-IDF text similarity). Usa **Reinforcement Learning (Bernoulli Multi-Armed Bandit)** para selecionar vizinhos ótimos dinamicamente. **Graph Coarsening + Sparse Spectral Clustering** para mitigar esparsidade de adjacency matrix em cold-start.

- **DFraud³ (IEEE TIFS 2021)**: classificação multi-componente (reviews, fraudsters, AND fraud-targeted items simultaneamente). **62,18% dos fraudsters no Yelp começam com reviews genuínas** — camouflage handling via neighbor propagation multi-hop.

- **CRSDnet**: user-user clique graph + Belief Propagation + Active Learning. AUC forte no Yelp Chicago com apenas **2,5% labeled data** (vs. 40-80% para métodos GNN).

### 1.3 Data Quality Pipeline

Três pilares: **Freshness** (centenas de milhares de updates diários, verificação de horários), **Quality** (reviews mais longas e de maior qualidade que outras plataformas US), **Trust** (AI pipeline para inappropriate language detection + moderação humana).

**Streaming infrastructure**: sincronização em tempo real de business properties data entre Redshift (offline) e streaming consumers. **CHAOS framework**: "Content Hosting Architecture with Optimization Strategies" para server-driven UI em 8 client platforms.

---

## 2. Lições de Engenharia

### 2.1 URLClassLoader hot-reload elimina o maior gargalo de iteração em search

Pushes de código que levavam horas (download S3, warm Lucene, force GC) passaram a minutos com hot-reload de ranking JAR. A capacidade de iterar no algoritmo de ranking múltiplas vezes ao dia é o que permite melhorar search quality continuamente.

### 2.2 O recommendation filter é uma solução elegante para o problema de confiança

Em vez de deletar reviews suspeitas (censura), movê-las para seção separada mantém transparência sem contaminar a nota média.

### 2.3 Fraude de review é um problema de cold-start em grafos

Novos usuários e negócios têm poucas conexões. HIN + RL-based neighbor selection + graph coarsening resolvem o problema de esparsidade sem exigir 40-80% de dados labelados.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Yelp |
| **Fundação** | Outubro 2004. IPO: março 2012 (NYSE: YELP) |
| **Fundadores** | Jeremy Stoppelman, Russel Simmons (PayPal Mafia) |
| **Categoria** | Reviews Locais / Search / Detecção de Fraude |
| **Reviews** | 308M+. Negócios: 6M+ |
| **Search** | Elasticsearch + plugin hot-reloadable (URLClassLoader). Geosharding + microsharding |
| **Fraude** | Recommendation Filter. SparseFraudNet (HIN + RL). DFraud³ (multi-component, 62% camouflage) |
| **Concorrentes** | Google Maps, TripAdvisor, Facebook Local |

---

## Fontes

- [Yelp Engineering Blog — Migrate Yelp's core merchant search function to Elasticsearch (2024)](https://engineeringblog.yelp.com/amp/2024/03/)
- [ACM TOIS 2025 — SparseFraudNet: Graph-based Cold-start Fraud Detection (HIN, RL BMAB, Graph Coarsening)](https://dl.acm.org/doi/full/10.1145/3748719)
- [IEEE TIFS 2021 — DFraud³: Multi-Component Fraud Detection Free of Cold-Start (HIN, 62% camouflage, 13% accuracy improvement)](https://ieeexplore.ieee.org/document/9435380)
- [CRSDnet — Active Learning + Belief Propagation on User-User Graphs (2.5% labeled data)](https://ar5iv.labs.arxiv.org/html/2205.13422)
