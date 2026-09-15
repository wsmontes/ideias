# Estudo de Caso 36 — iFood: A Plataforma Que Roda 150 Modelos de AI Com SageMaker Multi-GPU, Processa 10 Bilhões de Eventos/Dia no Databricks e Reduziu P95 de 30s Para 10s Com o "Language Tax" do Português

> **Data:** 2026-07-03
> **Loop:** 36 de ∞ (Reescrita — Fase 2)
> **Categoria:** Food Delivery / ML Platform / Infraestrutura de Dados
> **Tema:** O iFood processa **120-160 milhões** de pedidos mensais (120M confirmado Reuters ago/2025; 160M em case study ZenML 2026. Meta CEO: 200M até 2028) no Brasil com uma plataforma de ML que roda 150 modelos proprietários fazendo 14 bilhões de predições em tempo real por mês sobre um data lake de 16 petabytes. A infraestrutura que sustenta isso tem três pilares: **ML Go!** — uma plataforma interna sobre Amazon SageMaker com multi-model GPU endpoints que reduziram custos de infraestrutura em 75%, scale-to-zero automático e BruceML (scaffolding que encapsula código ML como containers Docker); **Databricks com arquitetura medallion** — ingestão de 10 bilhões de eventos diários via Kinesis, bronze/silver/gold layers com Liquid Clustering (maior tabela: 210TB, 800 bilhões de registros), reduzindo custo de pipeline em 67% e tabelas de ~4.000 para 100; e **ISO/Ailo** — um agente de AI hiperpersonalizado que processa pedidos via app e WhatsApp, onde a equipe descobriu o "language tax": prompts em português consomem ~50% mais tokens que em inglês, forçando otimizações que reduziram P95 de 30 segundos para 10 segundos. Este documento disseca cada camada a partir do AWS ML Blog, Databricks case study, arXiv paper e ZenML LLMOps database.

---

## 1. ML Go!: A Plataforma de ML Sobre SageMaker

### 1.1 Arquitetura Unificada

O iFood construiu o **ML Go!** para resolver o gap entre data science e engenharia. A plataforma padroniza treinamento, deployment e monitoramento de modelos via CI/CD sobre Amazon SageMaker:

- **BruceML**: ferramenta interna de scaffolding que encapsula código de ML como containers Docker desde o início, integrando-se com SageMaker sem fricção
- **ML Go! Gateway**: camada de abstração que conecta serviços ao SageMaker Endpoints, com Inference Components para serving mais rápido e eficiente
- **Dual inference modes**: real-time (SageMaker hosted endpoints para recomendações, tracking de pedidos, fraude) e batch (SageMaker batch transform para insights de parceiros, pré-processamento de dados)
- **Multi-model GPU endpoints**: deploy de múltiplos modelos em um único endpoint GPU — reduzindo custos de infraestrutura em até **75%**
- **Scale-to-zero**: shutdown automático de endpoints ociosos, reduzindo custos de dev/test

### 1.2 Escala de Predições

150 modelos proprietários fazendo **14 bilhões de predições em tempo real por mês** sobre um data lake de 16 petabytes. Feature store no **Redis Cloud** com leituras sub-milissegundo (vs. ~10ms do DynamoDB).

---

## 2. Arquitetura de Dados: Databricks Medallion + 10B Eventos/Dia

### 2.1 O Pipeline Medallion

**Bronze layer**: ingestão raw de ~10 bilhões de eventos diários via Amazon Kinesis, consolidados em tabelas únicas por produto.

**Silver layer**: data quality rules via Liquid Clustering. Maior tabela: **210TB, 800 bilhões de registros**.

**Gold layer**: tabelas analíticas para times de negócio.

### 2.2 Resultados da Migração

| Métrica | Antes | Depois |
|---|---|---|
| Tempo de codificação | Baseline | 30% redução |
| Manutenção de pipeline | Baseline | 70% redução |
| Custo mensal | Dezenas de milhares | Milhares (67% redução) |
| Número de tabelas | ~4.000 | 100 |

---

## 3. ISO/Ailo: O Agente de AI e o "Language Tax" do Português

### 3.1 Arquitetura do Agente

O **ISO** (depois **Ailo**) é um agente de AI para pedidos de comida via app iFood e WhatsApp (crítico no Brasil: 150M+ usuários ativos de WhatsApp). Arquitetura: single-agent com system prompts dependentes de estado — comportamento multi-agente em framework unificado. Capacidades: busca de comida, gestão de carrinho, aplicação de cupons, processamento de pagamento, memória de pedidos passados.

### 3.2 O Language Tax

A equipe descobriu que prompts em **português consomem ~50% mais tokens** que prompts equivalentes em inglês — o "language tax". Isso forçou otimizações agressivas para reduzir latência sem sacrificar qualidade de resposta em português.

### 3.3 Otimizações de Latência

| Otimização | Impacto |
|---|---|
| Flow shortcuts para requests simples | Bypassa pipeline completo |
| Processamento assíncrono de contexto | Paraleliza coleta de features |
| English-only prompts internos | 50% redução de tokens vs português |
| Tool naming otimizado | Reduz bloat de system prompts |

**Resultado**: P95 reduzido de **30 segundos para 10 segundos**.

---

## 4. RED: Recommendation System Com LightGBM e Embeddings

O paper "Personalized Recommendation of Dish and Restaurant Collections on iFood" (arXiv 2025) detalha o **RED**:

- **Modelo**: LightGBM classifier scoring tuplas (usuário, coleção, contexto)
- **Features**: características da coleção, similaridade usuário-coleção (embeddings), sinais contextuais (turno de refeição, hora do dia)
- **User embeddings**: two-tower search model; usuários representados por embeddings de 3 itens previamente comprados
- **Cold-start**: representações content-based permitem recomendações para coleções recém-criadas
- **Bias mitigation**: amostragem não-viesada (0,5% de impressões) para superar visibility bias
- **Resultados**: até **97% de melhoria em Card Conversion Rate**, **1,4% de aumento em App Conversion Rate**

---

## 5. Lições de Engenharia

### 5.1 Multi-model GPU endpoints reduzem custo de inferência em 75%

Em vez de dedicar uma GPU por modelo, o iFood coloca múltiplos modelos no mesmo endpoint GPU — compartilhando recursos de computação entre modelos com padrões de tráfego complementares.

### 5.2 O "language tax" é real e mensurável

Português consome ~50% mais tokens que inglês para o mesmo significado semântico. Para agentes de AI em produção, isso se traduz diretamente em latência e custo. A solução do iFood — prompts internos em inglês, output em português — é engenhosa.

### 5.3 Arquitetura medallion reduz 4.000 tabelas para 100

A abordagem bronze/silver/gold com Liquid Clustering consolidou um ecossistema fragmentado de dados em uma plataforma unificada, reduzindo custo em 67% e manutenção em 70%.

---

## 6. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | iFood |
| **Fundação** | 2011 |
| **Categoria** | Food Delivery / ML Platform / Dados |
| **Pedidos mensais** | 160 milhões+ |
| **MAUs** | 55 milhões+ |
| **Modelos AI** | 150 modelos proprietários, 14B predições/mês |
| **ML Platform** | ML Go! sobre SageMaker: multi-model GPU (75% cost reduction), scale-to-zero, BruceML |
| **Dados** | Databricks medallion: 10B eventos/dia, 16PB data lake, 210TB maior tabela |
| **Agente AI** | ISO/Ailo: P95 30s→10s, language tax resolvido, app + WhatsApp |
| **Recomendação** | RED: LightGBM + embeddings, +97% Card Conversion |

---

## Fontes

- [AWS ML Blog — How iFood built a platform to run hundreds of ML models with Amazon SageMaker Inference (2025)](https://aws.amazon.com/cn/blogs/machine-learning/how-ifood-built-a-platform-to-run-hundreds-of-machine-learning-models-with-amazon-sagemaker-inference/)
- [Databricks — Transforming data to elevate food deliveries and experiences (iFood case study)](https://www.databricks.com/customers/ifood/lakeflow-declarative-pipelines)
- [ZenML LLMOps — iFood: Building ISO: A Hyperpersonalized AI Food Ordering Agent](https://www.zenml.io/llmops-database/building-ailo-a-hyperpersonalized-ai-food-ordering-agent-for-millions-of-users)
- [arXiv 2508.03670 — Personalized Recommendation of Dish and Restaurant Collections on iFood (RED system)](https://ar5iv.labs.arxiv.org/html/2508.03670)
- [Redis / Asia Growth Partners — iFood's Utilization of Redis Cloud for Enhanced MLOps](https://asiagrowthpartners.com/case-study/ifood-s-utilization-of-redis-cloud-for-enhanced-machine-learning-operations/c6218)
