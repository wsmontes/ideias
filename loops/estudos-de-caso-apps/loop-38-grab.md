# Estudo de Caso 38 — Grab: A Plataforma de 3 Camadas Com Catwalk (1.400 Modelos ML), Coban (300 Bilhões de Eventos/Semana) e Reverse Conway Maneuver

> **Data:** 2026-07-03
> **Loop:** 38 de ∞ (Reescrita)
> **Categoria:** Super-App / Sudeste Asiático / Infraestrutura de ML

---

## 0. Linhagem

```
Táxis de rua — sem app. Sem GPS. Pagamento em dinheiro.
Grab (2012) — MyTeksi na Malásia. Ride-hailing. Depois: GrabFood, GrabPay, GrabExpress.
Grab hoje (2026) — 41M usuários transacionais/mês. 700+ cidades. 8 países.
```

O Grab opera no mercado mais fragmentado do mundo: 8 países, dezenas de idiomas, centenas de meios de pagamento, ruas que o Google Maps não cobre. A arquitetura reflete essa complexidade.

---

## 1. Arquitetura Técnica

### 1.1 A Plataforma de 3 Camadas e o Reverse Conway Maneuver

O CTO Suthen Thomas Paradatheth implementou uma **arquitetura de 3 camadas**:

1. **Business Verticals** — ride-hailing, food delivery, financial services
2. **Product Platforms** — serviços compartilhados: fulfillment, dispatch, pricing
3. **Technology Infrastructure** — deployment, infra, compliance

Aplicando o **"reverse Conway maneuver"**: a estrutura organizacional foi alinhada com a arquitetura alvo. O **Fulfillment team** consolidou engines de batching de todos os verticais em um sistema unificado, permitindo otimização cross-vertical — intercalando rides e food deliveries para o mesmo motorista.

### 1.2 Catwalk: De 8 Para 1.400 Modelos ML

A plataforma de serving de modelos **Catwalk** evoluiu em 4 fases: de soluções ad-hoc para TensorFlow Serving gerenciado (8 modelos), low-code self-serving via CI/CD (300 modelos), Kubernetes CRDs (substituindo Helm charts, com blue-green deploys), e **Catwalk Orchestrator** (200 orquestradores gerenciando **~1.400 modelos ML**). Stack: Amazon EKS, Kubernetes CRDs, ONNX, PyTorch, TensorFlow, gRPC.

**User Foundation Model (2025).** Transformer que aprende de dados tabulares + time-series clickstream. Usa modality-specific adapters (texto, numérico, ID, localização). Gera embeddings de curto e longo prazo. Aplicações: ads, fraude, churn, recomendações. Treinamento distribuído via Ray framework. Lida com centenas de milhões de entity vocabularies via hierarchical classification.

### 1.3 Coban: 300 Bilhões de Eventos Por Semana

A plataforma de streaming **Coban** (infra team) processa centenas de terabytes de tráfego diário, suportando todo o ecossistema super-app. **Midas** (métricas) + **MarketWatch** (observabilidade) sobre Apache Pinot + Kafka + Flink: **95% das requests em ~1 segundo**, 10M+ metric requests/dia.

### 1.4 Modular App: Build 60% Mais Rápido, Cold Start 4s Melhor

O app mobile foi refatorado de monólito para **1.000+ módulos independentes**: core libraries (networking, UI) + feature libraries (rides, food, wallet). Resultado: build time reduzido em ~60%, cold-start latency em ~4s.

---

## 2. Lições de Engenharia

### 2.1 Reverse Conway maneuver: arquitetura e organização são a mesma coisa

Alinhar times com a arquitetura alvo — em vez de deixar a arquitetura emergir da estrutura organizacional — permitiu consolidar engines de batching que otimizam globalmente, não por vertical.

### 2.2 Catwalk: a evolução de 8 para 1.400 modelos seguiu um padrão repetível

Managed → self-service → CRDs → orchestrators. Cada fase removeu fricção para o próximo order de magnitude de escala.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Grab |
| **Fundação** | 2012 (MyTeksi). IPO: dez 2021 (NASDAQ: GRAB) |
| **Categoria** | Super-App: ride-hailing, food, pagamentos, logística |
| **Usuários** | 41M transacionais/mês. 700+ cidades, 8 países |
| **ML** | Catwalk: 1.400 modelos. User Foundation Model (transformer + Ray) |
| **Dados** | Coban: centenas TB/dia. Midas: Apache Pinot+Kafka+Flink, 95% <1s |
| **Mobile** | 1.000+ módulos. Build -60%. Cold start -4s |
| **Concorrentes** | Gojek, Uber, ShopeeFood |

---

## Fontes

- [Grab Engineering Blog — Evolution of Catwalk: Model serving platform (1.400 modelos)](https://engineering.grab.com/catwalk-evolution)
- [Computer Weekly — Inside Grab's platform strategy (3-layer, reverse Conway, Nov 2024)](https://www.computerweekly.com/news/366615230/Inside-Grabs-platform-strategy)
- [StarTree — How Grab Built a Next-Gen Observability Stack with Apache Pinot](https://startree.ai/user-stories/grab-nextgen-observability-with-pinot/)
- [ZenML — Grab: User Foundation Models for Personalization at Scale (2025)](https://www.zenml.io/llmops-database/user-foundation-models-for-personalization-at-scale)
- [Netizen Experience — Super Apps: Lessons in Modular App Architecture (1.000+ módulos, Ago 2025)](https://www.netizenexperience.com/blog/modular-app-architecture-super-app)
