# Estudo de Caso 33 — Revolut: O Neobanco Que Substituiu Kafka Por PostgreSQL E Roda 1.200 Microserviços

> **Data:** 2026-07-03
> **Loop:** 33 de ∞ (Reescrita — Fase 2)
> **Categoria:** Fintech / Infraestrutura / Sistemas Distribuídos
> **Tema:** O Revolut processa pagamentos para 45 milhões de clientes com uma arquitetura de 1.200 microserviços que deliberadamente evita Apache Kafka — a escolha padrão da indústria para streaming de eventos. Em vez disso, a empresa construiu um event store proprietário sobre PostgreSQL com cursores JDBC scrollables e entregou consistência transacional onde Kafka ofereceria consistência eventual. Essa decisão é o fio condutor da arquitetura do Revolut: sempre que possível, prefira consistência forte e simplicidade operacional sobre escalabilidade horizontal e complexidade de infraestrutura. O resultado é uma plataforma onde uma equipe de DevOps de quinze pessoas suporta 1.300 engenheiros, onde o motor de detecção de fraude Sherlock processa decisões em 50 milissegundos usando uma arquitetura Lambda com Couchbase na camada de velocidade e Spark na camada de batch, e onde um modelo de AI proprietário chamado PRAGMA — treinado em 40 bilhões de eventos de transação — alimenta agentes que processam 2 milhões de tarefas de fincrime por mês.

---

## 1. Por Que PostgreSQL em Vez de Kafka: O Event Store Proprietário

A maioria dos neobanks adota Kafka como backbone de streaming de eventos. O Revolut conscientemente não o fez. A justificativa interna, documentada em apresentações de engenharia, envolve três argumentos:

**Complexidade operacional**: Kafka exige manutenção de clusters ZooKeeper (ou KRaft em versões recentes), gerenciamento de partições, retenção de tópicos e monitoramento de lag de consumidores. Para uma empresa que estava escalando de startup para banco regulado, adicionar essa superfície operacional ao stack era um risco desproporcional.

**Curva de aprendizado**: cada novo engenheiro precisaria aprender o modelo de consumo do Kafka — partições, offsets, grupos de consumidores, semânticas de entrega — antes de ser produtivo. Com PostgreSQL, virtualmente todo engenheiro backend já conhece a ferramenta.

**Consultabilidade**: eventos em Kafka são efêmeros (retidos por N dias) e difíceis de consultar ad-hoc. No PostgreSQL, cada evento é uma linha em uma tabela — pode ser consultado, indexado, agregado e analisado com SQL padrão, sem mover dados para um data warehouse separado.

O event store do Revolut funciona assim: antes de enviar um evento para outros serviços, o serviço emissor persiste o evento em uma tabela PostgreSQL — a `event_store`. Um `SingleEventConsumer` ou `MultiEventConsumer` se inscreve para receber eventos de tipos específicos. Um `event-processor` despacha eventos usando cursores JDBC scrollables sobre uma réplica de leitura da `event_store`, evitando sobrecarregar a instância primária. Um processo reconciliador verifica periodicamente se há eventos persistidos mas não entregues e os reenvia — garantindo entrega at-least-once sem depender de um broker de mensagens externo.

---

## 2. Consistência de Dados Em Três Camadas

O Revolut adota uma estratégia de consistência em três níveis, dependendo do escopo da operação:

**Nível 1 — Transações ACID**: quando todas as operações de uma transação financeira estão dentro dos limites do sistema Revolut — débito de uma conta, crédito em outra, atualização de saldo — tudo ocorre dentro de uma transação PostgreSQL padrão. Se qualquer parte falhar, a transação inteira é revertida. Isso é possível porque o Revolut controla ambas as pontas da transação.

**Nível 2 — Job-based reconciliation**: quando a operação envolve sistemas externos (APIs de bancos, redes de cartão, SWIFT), uma transação ACID não é possível. O Revolut usa um Transaction Manager que inicia a transação, persiste o estado internamente, faz a chamada externa de forma assíncrona, e usa um job de reconciliação para verificar o resultado e corrigir inconsistências.

**Nível 3 — Eventual consistency para dados não-financeiros**: para dados que não afetam saldos — preferências de usuário, configurações de notificação, analytics — o Revolut aceita consistência eventual via propagação assíncrona de eventos.

A migração planejada do core ledger de PostgreSQL para TigerBeetle — um banco de dados de transações financeiras projetado para processar mais de um milhão de transações por segundo com consistência forte e latência determinística — indica que a empresa está atingindo os limites do PostgreSQL para o workload de ledger.

---

## 3. Sherlock: Arquitetura Lambda Para Detecção de Fraude em 50ms

O motor de detecção de fraude do Revolut — Sherlock — opera com uma restrição de latência brutal: cada transação precisa ser avaliada em menos de 50 milissegundos. Para resolver isso, o Sherlock implementa uma arquitetura Lambda com duas camadas:

**Camada de velocidade (Speed Layer)** : modelos de machine learning leves rodando contra um cache em memória (Couchbase). Quando uma transação chega, o Sherlock consulta features pré-computadas no cache — padrões de gasto recentes, localização, valor, categoria do comerciante — e gera uma decisão (aprovar, bloquear, desafiar) em menos de 50ms. A latência é o requisito dominante; a precisão do modelo é secundária.

**Camada de batch (Batch Layer)** : pipelines noturnos em Apache Airflow, Dataflow e Spark processam o histórico completo de transações, treinam modelos de deep learning mais sofisticados, e atualizam as features no cache da camada de velocidade. Esses modelos consideram padrões de longo prazo — sazonalidade, mudanças graduais de comportamento, correlações entre contas — que não caberiam em uma decisão de 50ms.

Os **FinCrime AI Agents** — nove agentes especializados — processam 2 milhões de tarefas por mês em 700.000 clientes, rodando em GPUs H100 dedicadas na Nebius AI Cloud. Esses agentes não são modelos de classificação binária; são sistemas que investigam transações suspeitas, coletam evidências, geram relatórios de atividade suspeita e escalam para analistas humanos quando necessário.

---

## 4. PRAGMA: Um Modelo Fundacional Para Transações Financeiras

O PRAGMA é um modelo de AI proprietário do Revolut treinado em 40 bilhões de eventos de transação de 25 milhões de usuários. Três variantes — 10 milhões, 100 milhões e 1 bilhão de parâmetros — são otimizadas para diferentes tarefas: detecção de fraude, credit scoring e cross-sell. O modelo é baseado em transformers e opera sobre sequências de transações, similar a como modelos de linguagem operam sobre sequências de tokens.

A escolha do Nebius AI Cloud como infraestrutura de treinamento reflete uma decisão de soberania de dados: processar dados financeiros de cidadãos europeus em GPUs operadas por uma empresa europeia, mantendo conformidade com GDPR.

---

## 5. Plataforma de Desenvolvimento: 15 DevOps Para 1.300 Engenheiros

Uma equipe de quinze engenheiros de DevOps suporta 1.300 engenheiros de produto. Essa proporção — ~87:1 — é possível porque a plataforma interna de desenvolvimento é construída no modelo self-service: engenheiros criam seus próprios ambientes, implantam seus próprios serviços e monitoram sua própria produção. Não há equipe de QA dedicada, não há equipe de on-call dedicada. O modelo é "you build, you run".

A infraestrutura roda no Google Cloud Platform: Compute Engine para VMs, GKE para Kubernetes, Cloud APIs para serviços gerenciados. O provisionamento é totalmente automatizado via Infrastructure as Code. Snapshots incrementais de bancos de dados multi-terabyte são concluídos em aproximadamente cinco minutos — comparado a vinte horas com snapshots completos.

---

## 6. Lições de Engenharia

### 6.1 Kafka não é obrigatório para event-driven architecture

O Revolut provou que um event store sobre PostgreSQL com cursores JDBC scrollables e um processo reconciliador pode substituir Kafka para cargas de trabalho de fintech, com a vantagem adicional de consistência transacional (eventos e estado de negócio na mesma transação ACID). A troca é que isso não escala horizontalmente como Kafka — mas para a maioria das empresas, o limite de escala do PostgreSQL é mais alto do que sua carga de trabalho jamais atingirá.

### 6.2 A latência da detecção de fraude define sua arquitetura

Sherlock precisa decidir em 50ms. Isso força uma separação arquitetural entre a camada de velocidade (cache em memória, modelos leves) e a camada de batch (modelos pesados, atualização noturna). É um padrão que se repete em todo sistema de decisão em tempo real: a latência não é um requisito de performance — é o requisito que define a arquitetura.

### 6.3 Consistência forte em transações financeiras não é negociável

O Revolut usa transações ACID para operações internas e job-based reconciliation para operações externas. Em nenhum ponto uma transação financeira é tratada com consistência eventual. A complexidade adicional do modelo de reconciliação é o preço que se paga por não poder usar ACID em chamadas externas.

---

## 7. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Revolut |
| **Fundação** | 2015 |
| **Microserviços** | 1.200+ |
| **Backend** | Java, Kotlin, Kotlin Coroutines |
| **Event Store** | PostgreSQL (custom), JDBC scrollable cursors, sem Kafka |
| **Fraude** | Sherlock: Lambda Architecture, Couchbase + Spark |
| **AI** | PRAGMA (40B eventos, 3 variantes), FinCrime Agents (9 agentes, 2M tarefas/mês) |
| **Infra** | GCP (Compute Engine, GKE), Nebius AI Cloud (200+ H100 GPUs) |
| **DevOps** | 15 engenheiros para 1.300 devs |

---

## Fontes

- [AbnAsia — Architecture of a Neobank: Revolut](https://news.abnasia.org/blog/posts/en-architecture-of-a-neobank-revolut-3689)
- [Nebius — Revolut on the Inference Frontier](https://nebius.com/customer-stories/revolut)
- [QCon London 2024 — Unveiling the Tech Underpinning FinTech's Revolution](https://d3s75c3xtnyqxt.cloudfront.net/presentation/apr2024/unveiling-tech-underpinning-fintechs-revolution)
- [ECER — Revolut, Netflix Highlight Platform Engineering's Role in Developer Speed](https://ecweb.ecer.com/topic/en/detail-225632-revolut_netflix_highlight_platform_engineerings_role_in_developer_speed.html)
- [Google Cloud — Revolut Case Study](https://cloud.google.com/customers/revolut)
