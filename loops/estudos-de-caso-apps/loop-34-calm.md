# Estudo de Caso 34 — Calm: O App de Meditação Que Usa Redes Neurais Hierárquicas Para Recomendar Qual História Ouvir Antes de Dormir

> **Data:** 2026-07-03
> **Loop:** 34 de ∞ (Reescrita — Fase 2)
> **Categoria:** Bem-Estar / Sistemas de Recomendação / ML
> **Tema:** O problema de engenharia central do Calm não é streaming de áudio — isso é essencialmente resolvido com CDN. O problema central é recomendação: dado um catálogo de centenas de meditações, histórias para dormir e sons ambientes, qual conteúdo mostrar para este usuário, neste momento, com base no seu histórico de escuta, hora do dia, humor auto-reportado e características acústicas do conteúdo? O Calm começou com um sistema simples baseado em popularidade mais remoção de itens já ouvidos — que funcionou até não funcionar mais, quando usuários começaram a clicar em conteúdo mas não completá-lo. A migração para um pipeline de ML baseado em Amazon Personalize com redes neurais hierárquicas — e depois para busca vetorial com LLMs generativos no Amazon Bedrock — é um caso de estudo em como sistemas de recomendação evoluem de heurísticas para deep learning para AI generativa em uma única empresa.

---

## 1. Fase 1: Heurísticas Baseadas em Popularidade (2017-2020)

O primeiro sistema de recomendação do Calm era determinístico: mostrar o conteúdo mais popular, remover itens que o usuário já ouviu, aplicar algumas regras manuais de curadoria (não recomendar meditações para dormir às 10h da manhã, por exemplo). Para um catálogo pequeno e uma base de usuários em crescimento, isso funcionava.

O ponto de ruptura veio com as Sleep Stories — histórias narradas por celebridades como Harry Styles e Matthew McConaughey. A popularidade explodia para algumas histórias (o "Harry Styles effect" gerava picos de tráfego que duravam semanas), mas a taxa de conclusão era baixa: usuários clicavam por curiosidade, ouviam dois minutos e abandonavam. O sistema baseado em popularidade continuava recomendando essas mesmas histórias para novos usuários, criando um ciclo de baixa satisfação. Ficou claro que popularidade global não era um bom preditor de satisfação individual.

---

## 2. Fase 2: Amazon Personalize + HRNN-Metadata (2020-2024)

O Calm considerou construir um sistema de deep learning in-house, mas optou por Amazon Personalize — um serviço gerenciado de recomendação — porque a equipe de engenharia era pequena e o custo de oportunidade de construir infraestrutura de ML do zero era alto.

A arquitetura do pipeline:

**Ingestão de dados**: Apache Airflow rodando sobre Kubernetes orquestra jobs diários que extraem dados de interação do Redshift (data warehouse) e os depositam no S3 (data lake). Três tipos de dados alimentam o modelo:

- **Interações**: escutas, conclusões e favoritos. Conclusão de uma meditação tem peso maior que uma escuta parcial; favoritar tem peso maior que concluir. Visualizações sem escuta não são usadas como sinal.
- **Metadados de itens**: narrador, duração do áudio, profundidade da voz, tópico (natureza, fantasia, atenção plena), adequação para crianças, data de lançamento. A profundidade da voz do narrador — uma feature acústica extraída automaticamente — revelou-se surpreendentemente preditiva para preferências de usuário.
- **Metadados de usuário**: tempo de conta, horário favorito do dia, país.

**Modelo**: o algoritmo usado é HRNN-Metadata — Hierarchical Recurrent Neural Network com metadados. HRNNs são variantes de RNNs que modelam sequências temporais de interações com uma estrutura hierárquica: sessões dentro de dias, dias dentro de semanas. Isso captura padrões como "este usuário ouve meditações de foco às 9h em dias úteis e histórias para dormir às 22h".

**Inferência**: batch a cada 48 horas, gerando até 500 recomendações por usuário. O resultado é armazenado no ElastiCache (Redis) para acesso em sub-milissegundo pela API. Uma camada de lógica de negócios em Python pós-processa as recomendações: agrupa em coleções (como "7 Dias de Calma"), filtra por tipo de conteúdo, aplica shuffle para variabilidade e remove duplicatas.

**Resultado**: aumento de 3,4% na prática diária de mindfulness entre membros — um ganho modesto mas significativo para um produto onde a métrica principal é hábito, não engajamento.

---

## 3. Fase 3: Bedrock + OpenSearch Vector Search (2024-2025)

A limitação do HRNN-Metadata era que ele operava sobre metadados estruturados — narrador, tópico, duração. Não conseguia capturar similaridade semântica entre conteúdos. Duas meditações poderiam ser sobre "ansiedade" e usar palavras completamente diferentes para descrever a mesma sensação.

A migração para **Amazon Bedrock** (LLMs como fundação) e **Amazon OpenSearch** (busca vetorial) resolveu isso:

**Geração de embeddings de conteúdo**: um LLM no Bedrock processa cada meditação e história, gerando uma descrição contextual rica que captura tom emocional, ritmo narrativo e adequação situacional — informações que não existiam nos metadados originais. Essas descrições são convertidas em embeddings vetoriais e indexadas no OpenSearch.

**Busca híbrida**: uma consulta combina similaridade semântica (embedding do perfil do usuário vs. embedding do conteúdo) com filtros keyword (tipo de conteúdo, duração máxima, narrador). O OpenSearch suporta busca híbrida nativamente, combinando scores de relevância vetorial e textual.

**Explicabilidade**: cada recomendação agora inclui uma justificativa gerada por LLM — "Recomendamos esta meditação porque você costuma ouvir sessões de foco matinais com narradores de voz grave" — substituindo o modelo de caixa-preta do Personalize. A explicabilidade não é cosmética: aumenta a confiança do usuário e a taxa de aceitação de recomendações.

**Resultado**: melhoria de 25% na acurácia de recomendação. Cem por cento das recomendações agora têm justificativa.

---

## 4. A Infraestrutura de Streaming de Áudio

Embora a recomendação seja o problema de engenharia mais interessante, a entrega de áudio é o problema mais fundamental. O Calm usa CDN para streaming de arquivos de áudio pré-codificados — essencialmente o mesmo padrão de qualquer serviço de streaming de música ou podcast. O desafio específico do Calm é que as sessões de áudio são longas (meditações de 10-30 minutos, histórias de 20-45 minutos) e os usuários frequentemente baixam conteúdo para uso offline (modo avião, retiros, áreas sem conectividade). Isso exige um sistema de cache agressivo no cliente e pré-download inteligente baseado nas recomendações personalizadas.

---

## 5. Lições de Engenharia

### 5.1 Um modelo não precisa ser state-of-the-art para gerar valor de negócio

O HRNN-Metadata do Amazon Personalize não é o algoritmo de recomendação mais avançado disponível. Mas entregou um aumento de 3,4% em prática diária com custo de engenharia mínimo — a alternativa teria sido uma equipe de ML de três a cinco pessoas mantendo infraestrutura de treinamento e serving. Para uma empresa onde engenharia de ML não é o core business, serviços gerenciados frequentemente superam soluções customizadas.

### 5.2 Features acústicas são surpreendentemente preditivas para conteúdo de áudio

A profundidade da voz do narrador — uma feature que qualquer pipeline de processamento de áudio pode extrair automaticamente — foi um dos preditores mais fortes de preferência de usuário. É o tipo de feature que engenheiros de ML frequentemente ignoram porque não é "semântica", mas que usuários percebem fortemente.

### 5.3 Explicabilidade não é cosmética — é funcionalidade de produto

As justificativas geradas por LLM para recomendações aumentaram a taxa de aceitação porque transformaram recomendações de "o algoritmo decidiu" para "faz sentido para você". Em produtos de bem-estar, onde a confiança é o ativo central, explicabilidade é tão importante quanto acurácia.

---

## 6. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Calm |
| **Lançamento** | 2012 |
| **Plataforma** | iOS, Android, Web |
| **Orquestração ML** | Apache Airflow sobre Kubernetes |
| **Armazenamento** | S3 (data lake), Redshift (data warehouse), DynamoDB (metadados), ElastiCache (Redis) |
| **Motor de Recomendação** | Amazon Personalize (HRNN-Metadata) → Amazon Bedrock (LLM) + OpenSearch (vector search) |
| **Inferência** | Batch a cada 48h (Fase 2); batch + real-time (Fase 3) |
| **Streaming** | CDN padrão para arquivos de áudio pré-codificados |

---

## Fontes

- [AWS ML Blog — Personalizing wellness recommendations at Calm with Amazon Personalize](https://aws.amazon.com/cn/blogs/machine-learning/personalizing-wellness-recommendations-at-calm-with-amazon-personalize/)
- [Firemind — Calm enhances personalised recommendations with generative AI (2025)](https://firemind.com/calm-enhances-personalised-recommendations-with-generative-ai/)
- [AWS — AWS customers use the cloud to make mental health and wellness resources more accessible](https://www.aboutamazon.com/news/aws/aws-customers-use-the-cloud-to-make-mental-health-and-wellness-resources-more-accessible)
