# Estudo de Caso 46 — Superhuman: O Sync Engine Local-First, a Migração de 3 Bilhões de Embeddings em 1 Dia Para Turbopuffer (p90 60ms) e a Arquitetura Multi-Agente de AI Search

> **Data:** 2026-07-03
> **Loop:** 46 de ∞ (Reescrita)
> **Categoria:** Email / Local-First / AI Search

---

## 0. Linhagem

```
Gmail (2004) — webmail. Labels. Search. O padrão.
Superhuman (2014) — sub-100ms. Keyboard-first. US$ 30/mês. Fila de espera de 3 anos.
Superhuman hoje (2026) — US$ 100M+ ARR. Turbopuffer. Multi-agent AI. Local-first.
```

---

## 1. Arquitetura Técnica

### 1.1 O Sync Engine Local-First

O Superhuman não é um frontend que faz chamadas de API para o Gmail. É um cliente **local-first** que mantém réplica completa da caixa de entrada no dispositivo, sincronizada via Google APIs (não IMAP). A arquitetura: estado primário no cliente, sync assíncrono com o servidor de email, operações instantâneas porque não há round-trip de rede. O sync engine gerencia replicação inicial, sync incremental, resolução de conflitos e offline mode completo — ler, responder, arquivar emails sem conexão, com sync ao reconectar. O design local-first foi o que permitiu sobreviver a outages massivas (Google, Cloudflare) sem impacto perceptível.

### 1.2 A Migração Para Turbopuffer: 3B Embeddings em 1 Dia

Até 2024, o maior gargalo do Superhuman era o vector database que alimentava o Ask AI:

| Problema | Impacto |
|---|---|
| Write throttling manual (400 req/s por nó) | Ingestão capada para não derrubar o provedor |
| Cap de 10.000 tenants ativos | Time construiu shuffle manual de inboxes entre cache e cold storage |
| Indexing lag de 24h+ em pico | Emails visíveis na inbox mas não encontráveis por search |
| Limite de 1 ano de histórico | Indexar mais degradava query performance |
| Feature rationing | Email classification e auto-archive relegados ao Postgres |

**Migração para turbopuffer**: namespace-per-inbox (routing via auth token, cross-tenant access impossível por design, AES-256). **Indexing path separation**: query nodes e indexing nodes separados — indexing load não afeta query performance. **Pre-warming**: quando usuário abre Ask AI, índice carregado em cache sem evictar outros. **Hybrid search**: vector (embedding) + BM25 full-text com filterable attributes (timestamp, hasAttachments, sender). **Multi-query API**: hybrid search com client-side LLM re-ranking. **Strong consistency**: novos emails imediatamente retornáveis.

| Métrica | Antes | Depois |
|---|---|---|
| Histórico indexado | 1 ano | 5+ anos |
| p90 latency | Degradado | **60ms** |
| Recall@10 | — | **97%+** |
| Custo | Baseline | **20%+ redução** |
| Embeddings | 3B+ migrados em 1 dia (sem re-embedding, $300K+ economizado) | 10B+ documentos |

### 1.3 A Arquitetura Multi-Agente de AI Search

O Ask AI evoluiu de single-prompt RAG para **multi-agent cognitive architecture**:

1. **Tool Classification** (paralelo): classifica intent — email search only, email+calendar, availability, scheduling, LLM response direto
2. **Metadata Extraction** (paralelo): time filters, sender names, attachment context
3. **Task-Specific Prompts**: prompts diferentes para cada intent, com context-specific instructions, semantic few-shot examples, encoded user preferences
4. **Hybrid Search + Reranking**: semantic + keyword com client-side LLM re-ranking
5. **Response < 2 segundos**: hard constraint que forçou paralelização

**Prompt engineering**: "Double dipping" — repetir instruções críticas no system prompt e na user message final — "ensures that essential guidelines are rigorously followed." Structured prompts: chatbot rules + task-specific guidelines + semantic few-shot examples.

**Resultados**: 14% redução no tempo de search (5 minutos/semana economizados). Rollout em 4 meses: internal → company-wide → beta → community champions → GA.

---

## 2. Lições de Engenharia

### 2.1 O vector database é o gargalo invisível do AI search

A maioria das discussões foca no modelo (LLM, prompt, chunking). O Superhuman descobriu que o gargalo real era o vector DB: write throughput, tenant caps, indexing lag. Trocar o vector DB resolveu problemas que nenhuma otimização de prompt resolveria.

### 2.2 Multi-agent com task-specific prompts > single-prompt RAG

Classificar intent + extrair metadados em paralelo + selecionar prompt específico produz resultados mais confiáveis que um prompt monolítico. "Performance varied significantly by search type" com single-prompt; a arquitetura multi-agent resolve isso.

### 2.3 Namespace-per-inbox é o design correto para search multi-tenant

Cada inbox com seu próprio namespace, routing via auth token, cross-tenant access impossível por construção. Indexing path separation garante que indexing load não afeta query performance.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Superhuman |
| **Fundação** | 2014. Founder: Rahul Vohra |
| **Categoria** | Email / Produtividade Premium |
| **Preço** | US$ 30-40/mês |
| **Sync** | Local-first, Google APIs, offline mode |
| **Vector DB** | turbopuffer: 10B+ docs, p90 60ms, 97%+ recall@10, 5+ anos de histórico |
| **AI** | Multi-agent: parallel classification + extraction, task-specific prompts, sub-2s |
| **Concorrentes** | Gmail, Outlook, Spark |

---

## Fontes

- [Turbopuffer — Superhuman Mail trusts 5x more emails to turbopuffer (case study, 2025)](https://turbopuffer.com/customers/superhuman)
- [ZenML LLMOps — Superhuman: AI-Powered Email Search Assistant with Advanced Cognitive Architecture](https://www.zenml.io/llmops-database/ai-powered-email-search-assistant-with-advanced-cognitive-architecture)
- [Syntax.fm #918 — Extreme Native Perf on the Web with Superhuman (local-first, sync engine, Jul 2025)](https://syntax.fm/show/918/extreme-native-perf-on-the-web-with-superhuman)
