# Estudo de Caso 33 — Revolut: O Neobanco Com PostgreSQL Como Event Store (Sem Kafka), Sherlock Fraud Detection (Lambda Architecture, Couchbase Speed Layer <50ms, Spark Batch Layer), PRAGMA AI (40B Eventos, 3 Variantes) e 1.200 Microserviços Com 15 DevOps

> **Data:** 2026-07-03
> **Loop:** 33 de ∞ (Reescrita)
> **Categoria:** Fintech / Infraestrutura / Sistemas Distribuídos

---

## 0. Linhagem

```
Bancos tradicionais — agências físicas. Mainframes. Semana para abrir conta.
Neobanks (2015-) — app-first. Minutos para abrir conta. Sem agências.
Revolut (2015) — Londres. Multi-currency. Crypto. 45M clientes.
Revolut hoje (2026) — 1.200 microserviços. PostgreSQL event store. Sherlock. PRAGMA.
```

---

## 1. Arquitetura Técnica

### 1.1 PostgreSQL Como Event Store: A Decisão Anti-Kafka

A maioria dos neobanks adota Kafka como backbone de streaming. O Revolut conscientemente rejeitou essa escolha. Em vez disso, cada serviço persiste eventos em uma tabela `event_store` no **PostgreSQL**, na **mesma transação** que modifica o estado de negócio — garantindo atomicidade entre estado e evento. Um `event-processor` lê eventos pendentes usando **cursores JDBC scrollables** sobre réplica de leitura, evitando sobrecarregar a instância primária. Um processo **reconciliador** verifica periodicamente eventos persistidos mas não entregues, reenviando-os — garantia at-least-once sem broker de mensagens externo.

**Por que não Kafka?** Complexidade operacional (ZooKeeper/KRaft, partições, consumer groups, monitoramento de lag), curva de aprendizado (cada engenheiro precisa aprender o modelo de consumo antes de ser produtivo), consultabilidade (eventos em Kafka são efêmeros; no PostgreSQL, cada evento é uma linha consultável com SQL padrão). O trade-off: não escala horizontalmente como Kafka — mas o throughput de eventos de um banco com 45M clientes está confortavelmente dentro dos limites do PostgreSQL.

**Estratégia de consistência em 3 níveis:**
- **Nível 1 — ACID**: operações internas (débito + crédito na mesma transação PostgreSQL)
- **Nível 2 — Job-based reconciliation**: operações envolvendo sistemas externos usam Transaction Manager + job de reconciliação periódica
- **Nível 3 — Consistência eventual**: dados não-financeiros (preferências, notificações, analytics)

**Migração planejada para TigerBeetle**: banco de dados de transações financeiras open-source com Viewstamped Replication, >1M TPS, double-entry accounting nativo — indica que o PostgreSQL está atingindo limites para o workload de ledger.

### 1.2 Sherlock: Arquitetura Lambda Para Decisão de Fraude em <50ms

**Camada de velocidade (Speed Layer).** Modelos de ML leves (árvores de decisão, regressão logística) rodando contra **Couchbase** — banco NoSQL com cache em memória integrado. Features pré-computadas: padrão de gasto nas últimas 24h, localização do dispositivo, valor relativo à média do usuário, categoria do comerciante. Decisão (approve/block/challenge) em **<50ms**. Falsos positivos são preferíveis a falsos negativos — custo de um falso negativo é perda financeira total; falso positivo é fricção de verificação adicional.

**Camada de batch (Batch Layer).** Pipelines noturnos em Apache Airflow + Spark processam histórico completo, treinam modelos deep learning mais sofisticados e atualizam features agregadas no cache da camada de velocidade. Capturam padrões de longo prazo: sazonalidade, mudanças graduais, correlações entre contas.

**FinCrime AI Agents.** 9 agentes especializados processam 2 milhões de tarefas de fincrime por mês em 700.000 clientes, rodando em GPUs H100 dedicadas na **Nebius AI Cloud** (Holanda — conformidade GDPR). Cada agente é especializado em um tipo de crime financeiro; um orquestrador decide qual ativar.

### 1.3 PRAGMA: Modelo Fundacional Para Transações Financeiras

**PRAGMA (PRedictive AGent for Monetary Activities)** — modelo proprietário treinado em **40 bilhões de eventos de transação** de 25 milhões de usuários. Arquitetura transformer sobre sequências de transações: cada transação é um "token" com features categóricas, numéricas e temporais. Três variantes: 10M parâmetros (decisões em tempo real), 100M (credit scoring), 1B (cross-sell e personalização).

### 1.4 Plataforma de Desenvolvimento: 15 DevOps Para 1.300 Engenheiros

**Proporção 87:1** possível porque a plataforma é self-service. Engenheiros criam ambientes, deployam serviços e monitoram produção sem depender de DevOps. Modelo "you build, you run." Infraestrutura: **Google Cloud Platform** (Compute Engine, GKE). **1.200 microserviços** primariamente Java/Kotlin com Kotlin Coroutines.

---

## 2. Lições de Engenharia

### 2.1 PostgreSQL como event store é viável para a vasta maioria das empresas

Se seu throughput de eventos é medido em milhares ou dezenas de milhares por segundo — não milhões — PostgreSQL é suficiente. A decisão de adotar Kafka deve ser baseada em requisitos de escala, não em pressão da indústria.

### 2.2 A latência alvo define a arquitetura de decisão

Sherlock precisa decidir em <50ms. Isso força separação entre speed layer (Couchbase, modelos simples) e batch layer (Spark, modelos complexos). Todo sistema de decisão em tempo real enfrenta esse trade-off.

### 2.3 Consistência forte em transações financeiras não é negociável — mas é graduável

ACID quando pode; reconciliação quando não pode; consistência eventual apenas para dados não-financeiros.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Revolut |
| **Fundação** | 2015 (Londres). Fundadores: Nik Storonsky (CEO), Vlad Yatsenko (CTO) |
| **Categoria** | Neobank / Fintech |
| **Clientes** | 45M+ |
| **Microserviços** | 1.200+ (Java/Kotlin, Kotlin Coroutines) |
| **Event Store** | PostgreSQL: JDBC scrollable cursors + event-processor + reconciliador. Sem Kafka |
| **Fraude** | Sherlock: Lambda (Couchbase <50ms + Airflow/Spark batch). 9 FinCrime AI Agents (2M tarefas/mês, H100/Nebius) |
| **AI** | PRAGMA: 40B eventos, 3 variantes (10M/100M/1B params). Transformer |
| **Infra** | GCP (Compute Engine, GKE). 15 DevOps : 1.300 engenheiros (87:1) |
| **Concorrentes** | N26, Monzo, Wise, Chime |

---

## Fontes

- [QCon London 2024 — Unveiling the Tech Underpinning FinTech's Revolution (Revolut PostgreSQL event store, Sherlock)](https://d3s75c3xtnyqxt.cloudfront.net/presentation/apr2024/unveiling-tech-underpinning-fintechs-revolution)
- [Nebius — Revolut on the Inference Frontier (200+ H100 GPUs, PRAGMA training)](https://nebius.com/customer-stories/revolut)
- [CloudWars — How Couchbase Helps Fintech Customer Fight Fraud (Sherlock, 50ms decisions)](https://cloudwars.com/acceleration-economy-minute/how-couchbase-helps-fintech-customer-fight-fraud-while-saving-millions/)
- [AbnAsia — Architecture of a Neobank: Revolut (event store, Sherlock, PRAGMA)](https://news.abnasia.org/blog/posts/en-architecture-of-a-neobank-revolut-3689)
