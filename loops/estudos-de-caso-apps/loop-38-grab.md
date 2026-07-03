# Estudo de Caso 38 — Grab: A Plataforma Que Processa 300 Bilhões de Eventos Por Semana e Mapeia Ruas Que o Google Maps Não Vê

> **Data:** 2026-07-03
> **Loop:** 38 de ∞ (Reescrita — Fase 2)
> **Categoria:** Super-App / Infraestrutura / Mapas
> **Tema:** Grab opera em oito países do Sudeste Asiático, roda 1.000 microserviços e serve 1.400 modelos de machine learning via Catwalk — sua plataforma proprietária de serving de ML. A arquitetura de três camadas — verticais de negócio, plataformas de produto compartilhadas e infraestrutura tecnológica — foi projetada para resolver o problema fundamental de um super-app: como compartilhar infraestrutura de dispatch, pricing e mapeamento entre ride-hailing, food delivery e serviços financeiros sem que cada vertical compita pelos mesmos recursos. A plataforma de streaming Coban processa 300 bilhões de eventos por semana — tanto que a empresa precisou implementar rack-aware closest-replica fetching no Kafka 3.1 para reduzir pela metade o custo de tráfego cross-AZ. O GrabMaps, construído do zero porque nenhum mapa comercial cobria becos de moto no Vietnã ou estradas não pavimentadas na Indonésia, processa 800 bilhões de chamadas de API por mês usando imagens de 20.000 câmeras KartaCam 2 que capturam vídeo 360 graus com LiDAR.

---

## 1. A Arquitetura de Três Camadas: Por Que um Super-App Não Pode Ser um Monólito de Microserviços

Grab organiza seus 1.000 microserviços em três camadas que refletem uma estratégia organizacional — o "reverse Conway maneuver" — onde a estrutura dos times espelha a arquitetura desejada, em vez do contrário:

**Camada 1 — Verticais de Negócio**: times de produto que constroem experiências de consumidor (ride-hailing, food delivery, pagamentos). Cada vertical tem suas próprias metas de negócio e autonomia sobre features.

**Camada 2 — Plataformas de Produto**: serviços compartilhados que servem múltiplas verticais. O fulfillment platform, por exemplo, consolida os motores de batching que antes eram específicos de cada vertical — transporte e comida agora são intercalados na agenda de um mesmo motorista, maximizando utilização. O dispatch, pricing e fleet management também são plataformas compartilhadas.

**Camada 3 — Infraestrutura Tecnológica**: deployment, gerenciamento de infraestrutura, compliance, hosting de modelos ML/LLM. O Conveyor — ferramenta interna de CI/CD — gerencia aproximadamente 17.000 deployments por mês com canary deploys, monitoramento automatizado e rollback automático.

A migração para essa arquitetura não foi puramente técnica — "houve lágrimas", segundo o CTO Suthen Thomas Paradatheth. Times de verticais resistiram a abrir mão de controle sobre infraestrutura que antes era exclusiva. A resolução veio via contratos de API: cada plataforma de produto expõe funcionalidade como serviço com SLA documentado, e as verticais consomem via API, mantendo autonomia sobre a experiência do usuário.

---

## 2. Coban: A Plataforma de Streaming Que Processa 300 Bilhões de Eventos Por Semana

A plataforma de streaming do Grab — Coban, nomeada em homenagem a uma cachoeira indonésia — processa mais de 300 bilhões de eventos por semana, com ingestão de terabytes por hora. A arquitetura roda sobre AWS EKS com o operador Strimzi gerenciando clusters Kafka.

Decisões arquiteturais críticas:

**Rack-aware closest-replica fetching**: clusters Kafka implantados em três zonas de disponibilidade AWS geravam tráfego cross-AZ massivo — aproximadamente 50% do custo total de Kafka. A ativação de rack-aware fetching no Kafka 3.1, onde consumidores leem preferencialmente de réplicas na mesma zona de disponibilidade, reduziu esse custo pela metade.

**Debezium CDC**: serviços escrevem apenas no MySQL; mudanças de binlog são automaticamente publicadas no Kafka via Debezium. Isso eliminou o problema de dual-write — onde um serviço precisava escrever atomicamente no banco de dados e no Kafka, criando inconsistências quando uma das escritas falhava.

**AutoMQ**: migração para AutoMQ com armazenamento compartilhado em S3 reduziu o tempo de rebalanceamento de partições de horas para segundos. Em clusters Kafka tradicionais, um broker caindo exigia mover todos os dados da partição para outro broker; com AutoMQ, os dados estão no S3 e apenas os metadados de offset precisam ser transferidos.

**Data quality monitoring com LLM**: mais de 100 tópicos críticos são monitorados com regras de qualidade geradas por LLM, que detectam semanticamente dados inválidos — valores fora de intervalo, formatos incorretos, inconsistências entre campos — sem exigir que engenheiros escrevam regras manualmente.

---

## 3. GrabMaps: O Google Maps Que o Sudeste Asiático Não Tinha

O GrabMaps nasceu da necessidade: nenhum mapa comercial capturava becos de moto em Hanói, estradas não pavimentadas em Java, ou os nomes de ruas em escrita tailandesa que os modelos de OCR ocidentais não conseguiam ler. Em 2022, o Grab atingiu autossuficiência em mapeamento. Hoje, o GrabMaps processa 800 bilhões de chamadas de API por mês.

A infraestrutura de captura de dados usa a KartaCam 2 — uma câmera 360 graus com LiDAR integrado, vencedora do Red Dot Design Award, instalada em carros, motos e mochilas. Aproximadamente 20.000 câmeras operam até o fim de 2025 no Sudeste Asiático. A KartaDashCam processa imagens no dispositivo — detecção de placas de trânsito, marcações de pista, buracos, lombadas — e envia atualizações quase em tempo real sem estágio intermediário de servidor.

O pipeline de visão computacional extrai features que não existem em mapas ocidentais: profundidade de alagamento (detectada via ativação de limpadores de para-brisa e imagens de câmera), qualidade de iluminação pública, superfície de estrada. Modelos de linguagem fine-tuned com OpenAI processam texto em escrita local — tailandesa, vietnamita, khmer — que sistemas de OCR convencionais não reconhecem.

O ciclo de feedback é alimentado por 41 milhões de usuários ativos mensais e milhões de motoristas que percorrem as ruas diariamente, reportando erros de mapa e mudanças de estrada.

---

## 4. Catwalk: 1.400 Modelos de ML em Produção

A plataforma de serving de modelos do Grab — Catwalk — gerencia 1.400 modelos em produção através de 200 Catwalk Orchestrators. A evolução da plataforma seguiu quatro fases: TensorFlow manual (fase 1) → low-code self-service (fase 2) → Kubernetes CRDs (fase 3) → high-code Orchestrator com controle fino (fase 4). Suporta TensorFlow, ONNX, PyTorch, LightGBM e XGBoost. Features incluem blue-green deployments, bundled deployments para modelos que precisam ser servidos juntos, e automação de load testing.

O pipeline de ML features é alimentado pelo Coban (Kafka) e processado via Apache Flink e um Stream Processing Framework (SPF) interno em Go. Modelos de pricing, ETA, alocação de motoristas e detecção de fraude são retreinados continuamente com dados frescos de streaming.

---

## 5. Observabilidade: Midas e MarketWatch Sobre Apache Pinot

O Grab construiu Midas — uma plataforma centralizada de métricas — e MarketWatch — um portal de operações para monitoramento de saúde de mercado — sobre Apache Pinot. O fluxo de dados: microserviços → Kafka → Apache Flink (limpeza e enriquecimento) → Apache Pinot (OLAP em tempo real). A plataforma processa 10 milhões de requisições de métricas por mês, com 95% completadas em aproximadamente 1 segundo. Integração com LLM permite consultas conversacionais às métricas e geração de explicações.

---

## 6. Lições de Engenharia

### 6.1 Rack-aware fetching no Kafka não é otimização — é controle de custo

Metade do orçamento de streaming do Grab era tráfego entre zonas de disponibilidade. Ativar rack-aware closest-replica fetching reduziu esse custo em 50% com zero mudanças no código de aplicação. É o tipo de otimização de infraestrutura que não aparece em benchmarks mas define a viabilidade econômica de arquiteturas orientadas a eventos em escala.

### 6.2 Seu mapa não cobre o mercado — construa o seu

O Google Maps não foi feito para motos, becos e estradas não pavimentadas do Sudeste Asiático. O Grab construiu seu próprio mapa porque a alternativa era um produto que não funcionava. O investimento em hardware customizado (KartaCam 2) e pipelines de visão computacional para extrair features que não existem em mapas ocidentais foi um custo fixo alto que produziu um ativo impossível de replicar.

### 6.3 CDC via Debezium resolve dual-write, mas introduz acoplamento de schema

O Grab eliminou inconsistências de dual-write (MySQL + Kafka) usando Debezium CDC. O trade-off é que o schema do banco de dados se torna o schema dos eventos — mudanças no banco propagam para o Kafka automaticamente, o que significa que migrations de banco precisam considerar consumidores downstream.

---

## 7. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Grab |
| **Fundação** | 2012 |
| **Microserviços** | 1.000+ |
| **Streaming** | Coban (Kafka): 300B+ eventos/semana |
| **Mapas** | GrabMaps: 800B+ API calls/mês, 20.000 KartaCam 2 |
| **ML** | Catwalk: 1.400 modelos, TensorFlow/ONNX/PyTorch/LightGBM |
| **Service Mesh** | Consul → Istio (migração 2024) |
| **CI/CD** | Conveyor: 17.000 deploys/mês |
| **Observabilidade** | Midas + MarketWatch (Apache Pinot), 10M metric reqs/mês |

---

## Fontes

- [Computer Weekly — Inside Grab's platform strategy (2024)](https://www.computerweekly.com/news/366615230/Inside-Grabs-platform-strategy)
- [Factor House — How Grab uses Apache Kafka in production](https://factorhouse.io/articles/grab-kafka-architecture)
- [Grab Engineering — Service mesh evolution: From Consul to Istio](https://engineering.grab.com/service-mesh-evolution)
- [Grab Engineering — Catwalk: Model serving platform at Grab](https://engineering.grab.com/catwalk-evolution)
- [Grab Engineering — Seamless migration of high volume real-time streaming traffic](https://engineering.grab.com/seamless-migration)
- [Startree — Grab Next-Gen Observability with Pinot](https://startree.ai/user-stories/grab-nextgen-observability-with-pinot/)
- [GrabMaps — Mapping the unmappable in SEA](https://grabmaps.grab.com/resources/mapping-the-unmappable-grabs-unique-approach-in-southeast-asia)
- [Tech in Asia — Mapmaking meets AI: How GrabMaps is mapping SEA](https://www.techinasia.com/mapmaking-meets-ai-grabmaps-mapping-sea)
