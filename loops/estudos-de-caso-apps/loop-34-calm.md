# Estudo de Caso 34 — Calm: A Evolução do Motor de Recomendação de Heurísticas de Popularidade → HRNN (Amazon Personalize) → LLMs Com Busca Vetorial (Bedrock + OpenSearch, +25% Acurácia, 100% Explicabilidade)

> **Data:** 2026-07-03
> **Loop:** 34 de ∞ (Reescrita)
> **Categoria:** Bem-Estar / Sistemas de Recomendação / ML

---

## 0. Linhagem

```
Meditação em estúdio — instrutor presencial. Sem tecnologia.
Headspace (2010) — primeiro app de meditação mainstream.
Calm (2012) — Sleep Stories. Celebridades. Recomendação por ML.
Calm hoje (2026) — 100M+ downloads. Bedrock + OpenSearch. HRNN → LLM.
```

---

## 1. Arquitetura Técnica

### 1.1 O Problema: Recomendação Não É Sobre Cliques — É Sobre Completar Sessões

O Calm não é um app de música ou podcast onde engajamento se mede em minutos ouvidos. É um app de meditação e sono onde a métrica de sucesso não é "o usuário clicou" — é "o usuário completou a sessão e voltou amanhã." O sinal de treinamento não é click-through rate — é **completion rate**. Interações são ponderadas: completar > favoritar > ouvir parcialmente. Visualizações sem play são explicitamente excluídas — são ruído, não sinal.

### 1.2 Fase 1: Heurísticas de Popularidade (2017-2020)

Popularidade global + remoção de itens já ouvidos + regras manuais (não recomendar meditações para dormir às 10h). O ponto de ruptura: **Sleep Stories** narradas por celebridades (Harry Styles, Matthew McConaughey) geravam picos de popularidade mas baixa taxa de conclusão — usuários clicavam por curiosidade, ouviam 2 minutos e abandonavam. O sistema de popularidade continuava recomendando essas mesmas histórias, criando ciclo de baixa satisfação.

### 1.3 Fase 2: Amazon Personalize + HRNN-Metadata (2020-2024)

**Pipeline.** Apache Airflow sobre Kubernetes orquestra jobs diários extraindo dados do Redshift → S3. Três datasets: interações (escutas, conclusões, favoritos — ponderados), metadados de itens (narrador, duração, profundidade da voz, tópico), metadados de usuário (tempo de conta, horário favorito, país).

**Modelo: HRNN-Metadata (Hierarchical Recurrent Neural Network).** Modela sequências temporais com estrutura hierárquica — sessões dentro de dias, dias dentro de semanas. Captura padrões como "este usuário ouve meditações de foco às 9h em dias úteis e Sleep Stories às 22h." Mecanismo de gating controla pesos de recência.

**Feature acústica surpreendente.** A profundidade da voz do narrador — extraída via análise de frequência fundamental do áudio — revelou-se um dos preditores mais fortes de preferência. Usuários tendem a preferir consistência de profundidade vocal entre sessões.

**Inferência batch a cada 48h**, 500 recomendações por usuário, armazenadas em ElastiCache (Redis). **Resultado:** +3,4% em prática diária de mindfulness.

**Limitação:** HRNN-Metadata operava sobre metadados estruturados. Não capturava similaridade semântica — duas meditações sobre "ansiedade" usando vocabulário diferente não eram reconhecidas como similares.

### 1.4 Fase 3: Bedrock + OpenSearch Vector Search (2024-presente)

Em parceria com a **Firemind** (consultoria AWS), o Calm reconstruiu o motor de recomendação sobre AI generativa:

**Geração de embeddings de conteúdo.** LLM no **Amazon Bedrock** processa cada meditação e história, gerando descrição contextual rica: tom emocional, ritmo narrativo, adequação situacional, nível de energia. Essas descrições são convertidas em embeddings vetoriais e indexadas no **Amazon OpenSearch** (vector search).

**Busca híbrida.** Similaridade semântica (embedding do perfil do usuário vs. embedding do conteúdo) + filtros keyword (tipo de conteúdo, duração máxima, narrador). OpenSearch combina scores vetoriais e textuais nativamente.

**Explicabilidade.** Cada recomendação inclui justificativa gerada por LLM: "Recomendamos esta meditação porque você costuma ouvir sessões de foco matinais com narradores de voz grave." Explicabilidade não é cosmética — em produtos de bem-estar, onde confiança é o ativo central, o usuário precisa entender o porquê.

**Stack AWS:** Bedrock (LLM), Lambda, DynamoDB, OpenSearch (vector search).

**Resultados:** +25% de acurácia. 100% das recomendações com justificativa. 5 novas propriedades de metadados para personalização.

---

## 2. Lições de Engenharia

### 2.1 Um modelo não precisa ser state-of-the-art para gerar valor

HRNN-Metadata entregou +3,4% com custo de engenharia mínimo. Serviços gerenciados vencem quando ML não é core business.

### 2.2 A métrica de sucesso define a arquitetura de recomendação

Completion rate como target → ponderação de interações por qualidade de sinal. Click-through rate produziria recomendações de clickbait.

### 2.3 Explicabilidade é funcionalidade de produto em domínios de confiança

"Faz sentido para você" > "o algoritmo decidiu". Em bem-estar, confiança é o ativo central.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Calm |
| **Lançamento** | 2012 |
| **Categoria** | Bem-Estar / Sistemas de Recomendação / ML |
| **Fase 1 (2017-20)** | Heurísticas de popularidade + regras manuais |
| **Fase 2 (2020-24)** | Amazon Personalize (HRNN-Metadata), Airflow/K8s, inferência batch 48h, +3,4% prática diária |
| **Fase 3 (2024-)** | Amazon Bedrock (LLM embeddings) + OpenSearch (vector + keyword hybrid search), +25% acurácia, 100% explicabilidade |
| **Concorrentes** | Headspace, Insight Timer, Apple Fitness+ Meditation |

---

## Fontes

- [Firemind — Calm enhances personalised recommendations with generative AI (Bedrock, OpenSearch, +25%, 2025)](https://firemind.com/calm-enhances-personalised-recommendations-with-generative-ai/)
- [AWS ML Blog — Personalizing wellness recommendations at Calm with Amazon Personalize (HRNN-Metadata, Airflow/K8s)](https://aws.amazon.com/cn/blogs/machine-learning/personalizing-wellness-recommendations-at-calm-with-amazon-personalize/)
