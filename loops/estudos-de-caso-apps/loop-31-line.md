# Estudo de Caso 31 — LINE: A Stack de Mensageria Com LEGY (Erlang Gateway), talk-server (Java/Spring/Thrift), Armeria (Netty+HTTP/2, 600K req/s, 99% <10ms), Kafka (150B Mensagens/Dia) e Rede Leaf-Spine Com 1.000 Switches Mellanox/Cumulus Linux

> **Data:** 2026-07-03
> **Loop:** 31 de ∞ (Reescrita)
> **Categoria:** Mensageria / Infraestrutura de Rede / Engenharia de Plataforma

---

## 0. Linhagem

```
SMS (2000s) — texto pago por unidade.
KakaoTalk (2010) — Coreia. WhatsApp (2009) — global.
LINE (2011) — Naver Japan. Resposta ao tsunami de Tōhoku.
LINE hoje (2026) — 165M MAUs. 5B mensagens/dia. LY Corporation (Naver + Yahoo Japan).
```

O LINE foi construído pela Naver (Coreia) para o mercado japonês — uma configuração organizacional que forçou decisões arquiteturais que nenhum mensageiro ocidental precisou tomar. Sem poder depender de cloud pública americana (latência, soberania de dados), o LINE construiu sua própria infraestrutura de rede física.

---

## 1. Arquitetura Técnica

### 1.1 A Stack de Mensageria

**LEGY (Erlang).** API Gateway e proxy reverso escrito em Erlang, deployado globalmente. Gerencia conexões de clientes, terminação TLS, rate limiting e roteamento de requisições para o talk-server. A escolha de Erlang reflete o mesmo raciocínio do Discord com a BEAM: atores isolados com supervisão, concorrência massiva e a filosofia "let it crash."

**talk-server (Java 8 + Spring + Apache Thrift).** Camada de lógica de negócio. Processa envio e recebimento de mensagens, gerencia grupos, aplica regras de entrega. Comunicação interna via **Apache Thrift** como protocolo RPC — escolha anterior à popularização do gRPC, oferecendo desempenho similar com suporte multi-linguagem. Thrift permite definir interfaces de serviço em IDL neutra.

**Armazenamento híbrido: Redis + HBase.** Mensagens recentes e dados de sessão em Redis (in-memory, latência <1ms). Mensagens históricas e dados de conta em HBase (persistente, escalável horizontalmente). Redis atua como cache write-through: escritas vão para Redis e são persistidas assincronamente no HBase; leituras vão primeiro ao Redis, com fallback ao HBase.

### 1.2 Armeria: 600K Requisições Por Segundo, 99% Abaixo de 10ms

**Armeria** é a biblioteca RPC assíncrona open-source do LINE, construída sobre **Netty** e **HTTP/2**. Diferentemente de frameworks RPC tradicionais que são síncronos (thread-per-request) ou assíncronos com callback hell, Armeria oferece API unificada para REST, gRPC, Thrift e WebSocket sobre o mesmo pipeline de I/O não-bloqueante.

No sistema de autenticação do LINE — que valida tokens para cada requisição de mensagem — Armeria processa **600 mil requisições por segundo** com **99% das requisições completando em menos de 10ms**. Netty gerencia I/O de forma assíncrona; HTTP/2 multiplexa múltiplas requisições sobre uma única conexão TCP; API reativa (CompletableFuture, Reactive Streams) evita bloqueio de threads.

### 1.3 Kafka: 150 Bilhões de Mensagens Por Dia

O LINE opera um dos maiores deployments de Apache Kafka do mundo: **150 bilhões de mensagens por dia** (~3 milhões de mensagens por segundo). Usado para comunicação assíncrona interna: sincronização entre datacenters, notificações push, processamento de eventos de analytics, pipelines de dados para recomendação e detecção de spam.

A engenharia de performance opera em três níveis: aplicação (batching de producers, compressão, particionamento otimizado), JVM (tuning de GC, heap sizing) e sistema operacional (page cache, I/O scheduler, configuração de disco).

### 1.4 A Rede Leaf-Spine: 1.000 Switches Mellanox, Cumulus Linux, 100 Gbps

A decisão de construir rede própria — em vez de usar cloud pública — foi tanto econômica quanto filosófica. **1.000 switches Mellanox Spectrum** com Cumulus Linux (Debian-based network OS) implementam topologia **leaf-spine com roteamento L3-CLOS**: cada servidor está a exatamente dois hops de qualquer outro (leaf → spine → leaf). **100 Gbps** entre qualquer par, sem oversubscription.

**Cumulus Linux** transforma o switch em um servidor Linux com interfaces de rede de alta velocidade: BGP via FRRouting, automação via Ansible/Chef, troubleshooting via ferramentas Linux padrão. O modelo operacional: "trate switches como servidores." **Open19 racks** padronizam dimensões, alimentação e conectividade.

### 1.5 Letter Sealing: E2EE Com ECDH em Curve25519

**Letter Sealing** (2015) é o protocolo de criptografia ponta-a-ponta do LINE. **ECDH (Elliptic Curve Diffie-Hellman) sobre Curve25519** para troca de chaves: cada dispositivo gera par de chaves; chave pública registrada no servidor; chave de sessão efêmera derivada via ECDH localmente, nunca transmitida. Mensagens criptografadas com AES-256-GCM, nonce derivado de contador monotônico (anti-replay). Servidor atua como relay passivo — armazena e encaminha ciphertext sem capacidade de decriptação.

---

## 2. Lições de Engenharia

### 2.1 Leaf-spine com BGP é a topologia correta para datacenters >100 racks

Sem Spanning Tree, sem single point of failure, ECMP para balanceamento automático. Cumulus Linux permite tratar switches como servidores.

### 2.2 Armeria prova que unificar REST, gRPC e Thrift sobre mesmo pipeline de I/O elimina duplicação de stack

Um serviço expõe REST e gRPC simultaneamente, compartilhando lógica de negócio e observabilidade. Netty + HTTP/2 como fundação comum.

### 2.3 HBase + Redis é o padrão de armazenamento híbrido para mensageria

Redis para latência; HBase para durabilidade. Mesmo padrão do Discord (ETS + ScyllaDB).

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | LINE (LY Corporation) |
| **Fundação** | Junho 2011 (Naver Japan). IPO: 2016 |
| **Categoria** | Mensageria / Plataforma |
| **MAUs** | 165M (Japão, Taiwan, Tailândia, Indonésia) |
| **Mensagens/dia** | 5 bilhões |
| **Gateway** | LEGY (Erlang) |
| **Application** | talk-server (Java 8 + Spring + Apache Thrift) |
| **Armazenamento** | Redis (cache/sessão) + HBase (persistência) |
| **RPC** | Armeria (Netty + HTTP/2): 600K req/s, 99% <10ms |
| **Async** | Apache Kafka: 150B mensagens/dia (3M msg/s) |
| **Criptografia** | Letter Sealing: ECDH Curve25519 + AES-256-GCM |
| **Rede** | 1.000 switches Mellanox Spectrum, Cumulus Linux, leaf-spine L3-CLOS, BGP/FRRouting, 100 Gbps, Open19 racks |
| **Concorrentes** | WhatsApp, KakaoTalk, WeChat, Telegram |

---

## Fontes

- [LINE Campus Talk at UC Berkeley — Messaging Server Architecture (Yuto Kawamura, SpeakerDeck)](https://speakerdeck.com/player/5f9809e15ddb4e95871e2bca3d47049d)
- [LINE Engineering — Building a company-wide data pipeline on Apache Kafka (150B messages/day, SlideShare)](https://de.slideshare.net/linecorp/building-a-companywide-data-pipeline-on-apache-kafka-engineering-for-150-billion-messages-per-day)
- [LINE Corporation — Collaboration with Mellanox and Cumulus Networks (press release)](https://www.nyoooz.com/features/technology/line-taps-mellanoxcumulus-switches-for-messaging-network.html/1125/)
- [GitHub — LINE Armeria: open-source async RPC library (Netty, HTTP/2, Thrift, gRPC)](https://github.com/line/armeria)
