# Estudo de Caso 36 — iFood: A Plataforma de Delivery Que Roda 150 Modelos de AI Por Pedido e Processa 10 Bilhões de Eventos Por Dia

> **Data:** 2026-07-03
> **Loop:** 36 de ∞ (Reescrita — Fase 2)
> **Categoria:** Delivery / Machine Learning / Infraestrutura de Dados
> **Tema:** O iFood processa 80 milhões de pedidos por mês no Brasil, conectando 55 milhões de usuários a 350.000 restaurantes através de 300.000 entregadores. Mas o número que define sua arquitetura é outro: 10 bilhões de eventos por dia. Cada toque no aplicativo, cada atualização de GPS de entregador, cada mudança de status de pedido, cada impressão de recomendação gera eventos que fluem através de uma malha de 2.000 microserviços orquestrados por Apache Kafka. Quando um usuário faz um pedido, mais de 100 modelos de machine learning são acionados — previsão de tempo de preparo, estimativa de entrega, detecção de fraude, personalização de cardápio, otimização de rota, alocação de entregador. A plataforma de ML — chamada ML Go! — roda sobre Amazon SageMaker e gerencia 150 modelos proprietários que fazem 14 bilhões de predições em tempo real por mês. O feature store construído sobre Redis Cloud serve features em latência sub-milissegundo. O data lake sobre Databricks armazena 210 terabytes com 800 bilhões de registros. E um sistema de dispatch baseado em Apache Ignite processa 1 milhão de requisições por minuto com latência p99 abaixo de 10 milissegundos.

---

## 1. A Malha de Eventos: 2.000 Microserviços Sobre Kafka

O iFood migrou de uma arquitetura monolítica com pipelines batch para uma arquitetura orientada a eventos usando Confluent Cloud — a versão gerenciada do Apache Kafka. A malha conecta 2.000 microserviços que se comunicam exclusivamente via eventos.

O fluxo de um pedido ilustra a topologia: o aplicativo do consumidor emite um evento `order_placed` para o tópico `orders`. O serviço de dispatch consome esse evento, consulta o feature store (Redis) para obter a posição atual dos entregadores próximos, roda um modelo de otimização de alocação, e emite um evento `driver_assigned`. O serviço de notificação consome `driver_assigned` e envia push para o entregador. O serviço de tracking consome eventos de GPS do aplicativo do entregador (via Kinesis, em um pipeline separado para dados de alta frequência) e emite atualizações de localização. O serviço de ETA consome essas atualizações e recalcula o tempo estimado de entrega.

A separação entre Kinesis (para streams de alta frequência como GPS, que geram 10 bilhões de eventos por dia) e Kafka (para eventos de negócio como mudanças de status de pedido) é uma decisão arquitetural que reflete a diferença de requisitos: GPS precisa de ingestão de altíssima velocidade com processamento mínimo; eventos de negócio precisam de garantias de ordenação e entrega.

---

## 2. Apache Ignite: Dispatch em 10 Milissegundos

O sistema de dispatch original do iFood usava polling em PostgreSQL — os restaurantes consultavam o banco periodicamente para verificar novos pedidos. Quando o volume ultrapassou 10 milhões de pedidos mensais, o polling se tornou o gargalo: consultas concorrentes ao PostgreSQL geravam contenção e latência crescente.

A substituição por Apache Ignite — um grid de dados em memória — resolveu o problema arquiteturalmente. Em vez de polling, o sistema `connection-order-events` mantém os pedidos em memória no Ignite e os entrega aos restaurantes via push. A latência p99 caiu para menos de 10 milissegundos. O throughput subiu para 1 milhão de requisições por minuto. O sistema processa 500 milhões de eventos por mês entregues a agentes externos (restaurantes, sistemas de PDV, parceiros de logística).

A escolha de Ignite em vez de Redis para esta carga de trabalho específica reflete a necessidade de processamento computacional no lado do servidor — Ignite permite executar lógica de negócio (filtros, agregações, joins) diretamente no grid de dados, enquanto Redis exigiria trazer os dados para a camada de aplicação, processar, e escrever de volta.

---

## 3. ML Go!: 150 Modelos, 14 Bilhões de Predições Por Mês

A plataforma ML Go! — construída sobre Amazon SageMaker — é a camada de inteligência que opera sobre a malha de eventos. Quando um usuário abre o aplicativo e faz um pedido, uma cascata de modelos é executada:

- **Modelos de recomendação**: personalização de cardápio baseada em histórico de pedidos, preferências implícitas (horário, localização, clima) e tendências de popularidade. Usam features do Redis Cloud com latência sub-milissegundo.
- **Modelos de previsão de preparo**: estimam quanto tempo o restaurante levará para preparar o pedido com base em carga atual, complexidade do pedido e dados históricos.
- **Modelos de ETA**: estimam o tempo de deslocamento do entregador até o restaurante e do restaurante até o cliente, usando tráfego em tempo real e padrões históricos.
- **Modelos de fraude**: detectam transações suspeitas em tempo real.
- **Modelos de otimização de rota**: determinam a alocação ótima de entregadores para pedidos, considerando proximidade, carga atual e eficiência de agrupamento.

A plataforma ML Go! implementa CI/CD para machine learning: SageMaker Pipelines orquestram retreinamento automatizado de modelos; o Model Registry gerencia versionamento; endpoints de inferência suportam scale-to-zero para ambientes de desenvolvimento e multi-model endpoints em GPU para produção, reduzindo custos em até 75% ao consolidar múltiplos modelos em uma única instância de GPU.

---

## 4. Feature Store em Tempo Real: Spark Streaming + Redis

O feature store do iFood unifica features batch (calculadas offline sobre dados históricos) e features em tempo real (calculadas sobre streams de eventos) em uma única interface de consumo. A arquitetura usa Spark Streaming com Delta Tables para processamento e Redis Cloud para serving.

O padrão de agregação em duas etapas — `flatMapGroupsWithState` — permite calcular múltiplas janelas de agregação (últimos 5 minutos, última hora, últimas 24 horas) em uma única passada sobre o stream, reduzindo o número de jobs de streaming de dezenas para aproximadamente dez. A unificação de código para features batch e real-time reduziu o volume de código em 50%.

---

## 5. O Data Lake: 210TB, 800 Bilhões de Registros

O data lake do iFood — construído sobre Databricks com arquitetura medallion (bronze, prata, ouro) — ingere 10 bilhões de eventos por dia via Amazon Kinesis. A camada bronze armazena dados brutos em Delta Lake, uma tabela por produto, particionada por data. A camada prata aplica validação e qualidade de dados usando Liquid Clustering do Databricks e pipelines declarativos Spark. A camada ouro produz tabelas analíticas para times de negócio e datasets de treinamento para ML.

A migração para pipelines declarativos reduziu o número de tabelas de aproximadamente 4.000 para 100, o custo de infraestrutura de dados em 67%, e o esforço de manutenção de pipelines em 70%. A maior tabela contém 210 terabytes e 800 bilhões de registros.

---

## 6. Ailo: Agente Conversacional com "Language Tax"

O Ailo (anteriormente iLo/ISO) é um agente de AI conversacional que permite pedir comida via linguagem natural — "estou com fome", "me surpreenda", "quero pizza com amigos". Opera tanto no aplicativo nativo quanto no WhatsApp, que tem 150 milhões de usuários no Brasil.

A arquitetura do agente é single-agent com prompts dependentes de estado: um único modelo de linguagem recebe prompts de sistema diferentes dependendo do contexto (navegação de cardápio vs. checkout vs. suporte), criando comportamento multi-agente sem complexidade de orquestração.

Um problema de engenharia específico do mercado brasileiro: prompts em português consomem aproximadamente 50% mais tokens que prompts equivalentes em inglês — o que a equipe chama de "language tax". A solução foi padronizar todos os prompts de sistema em inglês, independentemente do idioma do usuário, reduzindo o latency p95 de 30 segundos para 10 segundos. O agente traduz a entrada do usuário, processa em inglês, e traduz a resposta de volta — e a latência total ainda é menor que processar tudo em português.

---

## 7. Lições de Engenharia

### 7.1 Polling em banco de dados não escala — substitua por push em grid de dados

O sistema original de dispatch do iFood usava polling em PostgreSQL. Funcionou até 10 milhões de pedidos por mês, depois quebrou. A migração para Apache Ignite com push de eventos eliminou o gargalo arquitetural. A lição é que polling é uma solução de curto prazo que escala linearmente com o número de consumidores; push sobre grid de dados em memória escala logaritmicamente.

### 7.2 A arquitetura medallion em data lake não é dogma — é ferramenta de governança

O iFood reduziu 4.000 tabelas para 100 não por精简 de dados, mas por reorganização em camadas bronze/prata/ouro que separam ingestão, validação e consumo analítico. Cada camada tem um propósito claro e um contrato de qualidade. Isso permitiu reduzir custo e esforço de manutenção sem perder cobertura de dados.

### 7.3 Language tax é real e precisa ser medido

Prompts em português custam 50% mais tokens que em inglês para o mesmo significado semântico. A solução do iFood — processar em inglês com tradução nas bordas — é contraintuitiva (adiciona latência de tradução) mas produz latência total menor. É o tipo de otimização que só emerge quando se mede o custo real de inferência em produção, não em benchmarks de laboratório.

---

## 8. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | iFood |
| **Fundação** | 2011 |
| **Microserviços** | 2.000+ |
| **Eventos/dia** | 10 bilhões (Kinesis) |
| **ML** | 150 modelos proprietários, 14B predições/mês (SageMaker) |
| **Feature Store** | Redis Cloud, sub-ms latency |
| **Dispatch** | Apache Ignite, 1M RPM, p99 <10ms |
| **Data Lake** | Databricks Delta Lake, 210TB, 800B registros |
| **Agente AI** | Ailo (single-agent state-dependent prompts, WhatsApp + app) |

---

## Fontes

- [AWS ML Blog — How iFood built a platform to run hundreds of ML models with SageMaker](https://aws.amazon.com/cn/blogs/machine-learning/how-ifood-built-a-platform-to-run-hundreds-of-machine-learning-models-with-amazon-sagemaker-inference/)
- [Confluent — iFood Scales Cloud-Based Data Flow Architecture](https://www.confluent.io/fr-fr/customers/ifood/)
- [Databricks — iFood: Transforming data to elevate food deliveries](https://www.databricks.com/customers/ifood/lakeflow-declarative-pipelines)
- [Apache Ignite Summit 2022 — 1M RPM with 10ms Latency at iFood](https://ignite-summit.org/2022-november/sessions/393297)
- [Redis — iFood Relies on Redis Cloud for ML Operations](https://redis-io.analytics-portals.com/customers/ifood/)
- [ZenML — iFood: Building Ailo, Hyperpersonalized AI Food Ordering Agent](https://www.zenml.io/llmops-database/building-ailo-a-hyperpersonalized-ai-food-ordering-agent-for-millions-of-users)
- [SlideShare — Building a Real-Time Feature Store at iFood](https://www.slideshare.net/slideshow/building-a-realtime-feature-store-at-ifood-236481788/236481788)
