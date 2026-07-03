# Estudo de Caso 32 — KakaoTalk: O Mensageiro Que Roda em 5.000 VMs Gerenciadas Por Dois Engenheiros

> **Data:** 2026-07-03
> **Loop:** 32 de ∞ (Reescrita — Fase 2)
> **Categoria:** Mensageria / Infraestrutura / Protocolo
> **Tema:** O KakaoTalk é o mensageiro dominante da Coreia do Sul — 93% de penetração em uma população de 52 milhões. Mas o que torna sua engenharia notável não é a escala de usuários, e sim a densidade de automação: 5.000 máquinas virtuais em OpenStack gerenciadas por uma equipe de dois engenheiros. O protocolo de mensageria — LOCO — é uma implementação binária proprietária sobre BSON com criptografia AES-CFB que foi desenhada para velocidade, não para auditabilidade, e carrega vulnerabilidades documentadas que pesquisadores de segurança vêm apontando há mais de uma década. A migração recente de treinamento de modelos de AI de GPUs NVIDIA para TPUs do Google Cloud usando JAX — com modelos Mixture-of-Experts de 64 experts e throughput 2,7× maior — revela uma empresa que opera na fronteira da infraestrutura de computação.

---

## 1. O Protocolo LOCO: BSON Binário Sobre TCP Com Criptografia No Transporte

O KakaoTalk não usa XMPP, MQTT, WebSocket ou qualquer protocolo de mensageria padrão. Em vez disso, opera sobre um protocolo binário proprietário chamado LOCO, documentado pela primeira vez em 2012 por engenharia reversa. O LOCO serializa mensagens em **BSON (Binary JSON)** — uma representação binária de documentos JSON que oferece parsing mais rápido e tamanho menor que JSON textual.

O fluxo de uma mensagem no LOCO segue o modelo store-and-forward: o cliente envia a mensagem ao servidor via TCP com autenticação de sessão; o servidor persiste a mensagem e notifica o destinatário; se o destinatário estiver online, a mensagem é entregue via push sobre a conexão persistente. Se estiver offline, a mensagem é armazenada e entregue na próxima conexão.

A camada de criptografia no transporte usa **AES em modo CFB (Cipher Feedback)** . O modo CFB transforma uma cifra de bloco em uma cifra de fluxo, o que é útil para tráfego de mensageria — os dados chegam em rajadas de tamanho variável, e o CFB permite criptografar bytes individuais sem padding. Mas o CFB não oferece integridade de ciphertext: um atacante que pode modificar bits no trânsito pode produzir alterações previsíveis no texto plano descriptografado. O LOCO não implementa autenticação de mensagem (MAC) na camada de transporte.

Pesquisadores de segurança documentaram três vulnerabilidades estruturais no protocolo LOCO:

1. **Ausência de autenticação do servidor**: o cliente não verifica a identidade do servidor LOCO, tornando possível um ataque man-in-the-middle.
2. **Ausência de integridade de ciphertext**: o AES-CFB é maleável; bits podem ser alterados no trânsito sem detecção.
3. **Ausência de proteção contra replay**: o protocolo não implementa nonces ou timestamps, permitindo que mensagens capturadas sejam reenviadas.

Essas vulnerabilidades não são bugs de implementação — são consequências do design do protocolo. Elas existem porque o LOCO foi projetado para velocidade e eficiência de banda em dispositivos móveis de 2012, não para segurança contra adversários sofisticados.

---

## 2. O Modelo de Chat Duplo: Regular (Server-Side) vs. Secret (E2EE)

O KakaoTalk opera dois modelos de criptografia radicalmente diferentes.

**Regular Chat**: a mensagem é criptografada em trânsito via LOCO (AES-CFB), mas a chave de criptografia é compartilhada com a Kakao Corp. Isso significa que o servidor pode descriptografar, indexar e processar o conteúdo de todas as mensagens regulares. É uma escolha deliberada de arquitetura, não uma limitação técnica: o acesso ao conteúdo permite busca de mensagens no servidor, sugestões contextuais e moderação automatizada — funcionalidades que sistemas E2EE puros não conseguem oferecer sem processamento local pesado.

**Secret Chat**: introduzido em 2014, oferece criptografia ponta-a-ponta usando RSA. O remetente obtém a chave pública RSA do destinatário através de um banco de dados mantido pela Kakao que mapeia UUIDs de dispositivos para chaves públicas. O segredo compartilhado é derivado e usado para criptografar a mensagem no dispositivo do remetente; apenas o destinatário pode descriptografar.

O Secret Chat tem limitações significativas comparado a implementações modernas de E2EE. Não oferece **forward secrecy**: usa RSA estático em vez de Diffie-Hellman efêmero — se a chave privada de um dispositivo for comprometida, todas as mensagens passadas e futuras podem ser descriptografadas. Não implementa o algoritmo **Double Ratchet** (usado por Signal, WhatsApp, LINE Letter Sealing), que gera novas chaves para cada mensagem e garante que o comprometimento de uma chave não exponha todo o histórico. A verificação de fingerprint é opcional e raramente usada por usuários. O código não é aberto nem auditado independentemente.

---

## 3. Infraestrutura: 5.000 VMs OpenStack, Dois Engenheiros, Apache S2Graph

Em 2013, a Kakao migrou sua infraestrutura para OpenStack. Hoje, opera aproximadamente 5.000 máquinas virtuais sobre OpenStack — gerenciadas por uma equipe de dois engenheiros dedicados. Essa densidade de automação é possível porque a equipe investiu em três ferramentas internas:

- **Kengine**: gerencia automaticamente o ciclo de vida de VMs — provisionamento, scaling, health checks, descomissionamento. Operadores definem políticas; o Kengine executa.
- **CUOTA**: identifica e recupera recursos subutilizados. VMs que operam abaixo de thresholds de CPU ou memória são sinalizadas e consolidadas.
- **CROW**: plataforma unificada de métricas que coleta dados de todos os recursos físicos e virtuais, alimentando dashboards e sistemas de alerta.

O **Apache S2Graph** — um banco de dados de grafos incubado pela Kakao e construído sobre HBase — é um componente central da infraestrutura. O S2Graph gerencia as relações sociais da plataforma: quem segue quem, quem está em qual grupo, qual o grafo de amigos de cada usuário. Essas consultas de grafo são críticas para o roteamento de mensagens (para quem entregar?), sugestões de amigos e detecção de spam. O S2Graph processa consultas de grafo em milissegundos usando índices distribuídos sobre HBase.

O stack de processamento de dados usa CDC (Change Data Capture) para capturar mudanças em bancos de dados e propagá-las via Apache Kafka para múltiplos datacenters, mantendo consistência eventual entre regiões. Essa arquitetura permite que o KakaoTalk opere com latência baixa para usuários na Coreia enquanto replica dados para datacenters de disaster recovery.

---

## 4. Stack de Tempo Real: Go, gRPC, Redis Streams

Os serviços de tempo real do KakaoTalk são construídos em Go. O padrão de comunicação entre microserviços usa gRPC — chamadas de procedimento remoto com serialização Protobuf, que oferecem menor latência e maior throughput que REST/JSON. Para serviços que exigem ordenação estrita de eventos — como o KakaoLive, o serviço de streaming ao vivo — Redis Streams funciona como barramento de eventos, garantindo que eventos de sala sejam processados na ordem correta.

Conexões WebSocket com clientes móveis são gerenciadas com heartbeats e estratégias de reconexão customizadas. A camada de sinalização WebSocket usa Go pela capacidade da linguagem de gerenciar centenas de milhares de goroutines — uma por conexão — com overhead de memória mínimo.

---

## 5. Migração de AI: De GPUs NVIDIA Para TPUs Google Cloud

Em 2024-2025, a Kakao migrou o treinamento de seus modelos de linguagem — a família Kanana, com até 9,8 bilhões de parâmetros — de clusters de GPUs rodando Megatron-LM para TPUs do Google Cloud usando JAX. A migração envolveu:

- Adoção do framework **MaxText** com personalizações para blending de dados multi-fonte e processamento de tokens.
- Implementação de modelos **Mixture-of-Experts (MoE)** com 64 experts, onde apenas uma fração dos parâmetros é ativada por token, reduzindo o custo computacional de inferência.
- Uso de **XPK** para gerenciamento de clusters Kubernetes e **Grain** para pipelines determinísticos de dados.
- Ganho de throughput de 2,7× com TPUs Trillium comparado à infraestrutura anterior baseada em GPU.

A decisão de migrar foi motivada pela disponibilidade de TPUs em escala na infraestrutura do Google Cloud — a Kakao já operava significativamente nessa nuvem — e pela eficiência das TPUs em operações de multiplicação de matrizes, que dominam o treinamento de transformers.

---

## 6. Lições de Engenharia

### 6.1 A automação de infraestrutura é um multiplicador de força

Dois engenheiros gerenciando 5.000 VMs não é resultado de heroísmo — é resultado de investimento em automação (Kengine, CUOTA, CROW) que remove o trabalho manual do ciclo de operações. A lição é que o tamanho da equipe de infraestrutura não precisa ser proporcional ao tamanho da infraestrutura — precisa ser proporcional à qualidade da automação.

### 6.2 Protocolos proprietários acumulam dívida de segurança

O LOCO foi projetado em 2012 para eficiência, não para segurança. Treze anos depois, suas vulnerabilidades estruturais — ausência de autenticação do servidor, ciphertext maleável, sem proteção contra replay — continuam sem correção porque corrigi-las exigiria reescrever o protocolo, o que quebraria compatibilidade com todos os clientes existentes. A lição é que decisões de protocolo tomadas nos primeiros anos de uma empresa têm consequências de segurança que duram décadas.

### 6.3 E2EE como opt-in produz uma divisão de classes de segurança

A vasta maioria das conversas no KakaoTalk usa Regular Chat — criptografado em trânsito, mas acessível ao servidor. Uma minoria usa Secret Chat — E2EE, mas sem forward secrecy e sem auditoria independente. Essa arquitetura de dois níveis reflete uma escolha de produto: funcionalidades ricas (busca, sugestões, moderação) exigem acesso ao conteúdo. É uma decisão legítima de engenharia, mas significa que o usuário médio do KakaoTalk tem menos proteção de privacidade do que o usuário médio do WhatsApp ou Signal.

---

## 7. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | KakaoTalk |
| **Lançamento** | Março de 2010 |
| **Desenvolvedor** | Kakao Corp. (Coreia do Sul) |
| **Protocolo** | LOCO (BSON binário proprietário), AES-CFB |
| **E2EE** | Secret Chat: RSA estático, sem forward secrecy, sem Double Ratchet |
| **Backend** | Go + gRPC + Redis Streams, OpenStack (5.000 VMs), Apache Kafka + CDC |
| **Graph DB** | Apache S2Graph (HBase) — processa consultas de relacionamentos em ms |
| **AI/ML** | JAX + TPUs Google Cloud, MaxText, Mixture-of-Experts (64 experts) |

---

## Fontes

- [HITB SecConf 2024 — Leaking Kakao: LOCO protocol vulnerabilities analysis](https://conference.hitb.org/hitbsecconf2024bkk/materials/D1%20COMMSEC%20-%20Leaking%20Kakao.pdf)
- [Kakao OpenInfra — KakaoTalk speaks volumes about the future of cloud services](https://superuser.openinfra.dev/articles/kakaotalk-speaks-volumes-about-the-future-of-cloud-services/)
- [Google Cloud Blog — Kakao's journey with JAX and Cloud TPUs](https://cloud.google.com/blog/products/infrastructure-modernization/kakaos-journey-with-jax-and-cloud-tpus/)
- [Apache S2Graph — Data Hub for Apache Big Data Europe 2016](http://events.linuxfoundation.org/sites/events/files/slides/s2graph_data_hub_apache_big_data_europe_2016.pdf)
- [KoreaPlus — Inside Kakao: The Unified Messaging Pioneer the West Hasn't Noticed](https://dev.to/koreaplus-lifes/inside-kakao-the-unified-messaging-pioneer-the-west-hasnt-noticed-adn)
