# Estudo de Caso 43 — Gojek: O Super-App Indonésio Com 1.000+ Microserviços, Kafka Backbone, Dispatch Multi-Vertical (Ride-Hailing + Food + Logística) e a Maior Migração de Nuvem do Sudeste Asiático (GCP→Tencent Cloud, 4h54, 50+ Produtos)

> **Data:** 2026-07-03
> **Loop:** 43 de ∞ (Reescrita)
> **Categoria:** Super-App / Sudeste Asiático / Infraestrutura de Nuvem

---

## 0. Linhagem

```
Ojek (pré-2015) — moto-táxis informais. Esquinas (pangkalan). Dinheiro.
Gojek (2015) — call center. 20 motoristas. Depois: app. Depois: super-app.
GoTo (2021) — fusão Gojek + Tokopedia. IPO: US$ 32B (IDX).
GoTo hoje (2026) — 1.000+ microserviços. Tencent Cloud. Maior migração do Sudeste Asiático.
```

---

## 1. Arquitetura Técnica

### 1.1 Plataforma de Microserviços: 1.000+ Serviços, Kafka Backbone

O Gojek opera mais de **1.000 microserviços** com granularidade baseada em frequência de acesso. Serviços de alta frequência (dispatch, pagamentos) são mais granulares; baixa frequência (suporte, onboarding) são mais monolíticos. **Apache Kafka** é o backbone de mensageria, processando dezenas de bilhões de eventos diários.

**Fulfillment multi-vertical**: motor de dispatch unificado gerencia **ride-hailing (GoRide, GoCar), food delivery (GoFood) e logística (GoSend)**. O sistema decide em tempo real qual motorista pega qual pedido, otimizando utilização cross-vertical — o mesmo motorista intercala viagem de moto com entrega de comida no trajeto.

**GoPay**: carteira digital que resolveu o problema de troco (Indonésia era majoritariamente dinheiro). Infraestrutura híbrida: pagamentos em datacenter próprio (reguladores indonésios exigem soberania de dados financeiros), dispatch e marketplace em nuvem pública.

### 1.2 A Migração de Nuvem: GCP → Tencent Cloud (Junho 2025)

A **maior migração de nuvem do Sudeste Asiático**: **1.000+ microserviços** migrados para Tencent Cloud com **hot migration** (live, seamless server transfer). Downtime total: **4 horas e 54 minutos** (1 hora abaixo do planejado de 6 horas). Preparação: **8 meses** de planejamento, testes e ensaios (setembro 2024 a junho 2025).

**Stack destino**: 50+ produtos Tencent Cloud — CVM, CBS, WAF, **TKE (Tencent Kubernetes Engine)**, ES. **10+ tipos de banco de dados** com replicação, disaster recovery e rollback. Região de Jakarta expandida de 2 para **3 availability zones**. Terceiro datacenter na Indonésia ativado.

**Motivação**: dados previamente armazenados fora da Indonésia (GCP) criavam latência e questões de residência de dados. Parceria Tencent originalmente anunciada em setembro 2024. A migração para datacenters em Jakarta melhora data residency e latência.

---

## 2. Lições de Engenharia

### 2.1 Migrar 1.000+ microserviços entre nuvens em <5 horas requer hot migration e 8 meses de preparação

Não foi lift-and-shift. Exigiu re-arquitetura, sincronização prévia de dados, redirecionamento gradual de tráfego com 50+ produtos de nuvem.

### 2.2 O super-app não é estratégia — é consequência de ser a primeira experiência digital do usuário

Na Indonésia, centenas de milhões tiveram primeiro contato com internet via smartphone Android barato. Gojek era frequentemente o primeiro app — portal para todos os serviços digitais.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Gojek (GoTo Group) |
| **Fundação** | 2015, Jakarta. IPO: 2021 (IDX, US$ 32B) |
| **Categoria** | Super-App: ride-hailing, delivery, pagamentos, logística |
| **Microserviços** | 1.000+ |
| **Mensageria** | Apache Kafka (dezenas de bilhões de eventos/dia) |
| **Migração** | GCP → Tencent Cloud (jun 2025). 50+ produtos. 4h54 downtime. 8 meses preparação |
| **Híbrido** | GoPay em datacenter próprio (regulação indonésia) |
| **Concorrentes** | Grab, ShopeeFood |

---

## Fontes

- [Xinhua — 腾讯云助力GoTo集团实现迁云实践 (Jun 2025, 1.000+ serviços, 4h54)](http://www.xinhuanet.com/tech/20250605/a0b1b12ebac5497e98f038294eaf4d00/c.html)
- [GeekPark — 腾讯云助力 GoTo 集团创东南亚最大迁云实践 (Jun 2025, 50+ produtos, 8 meses)](https://www.geekpark.net/news/350085)
- [ComputerWeekly — How Indonesia's Go-Jek scales the heights with cloud (Kafka, microservices)](https://www.computerweekly.com/news/252446425/How-Indonesias-Go-Jek-scales-the-heights-with-cloud)
