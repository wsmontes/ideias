# Estudo de Caso 28 — Zoom: O MMR (Multimedia Router SFU) Com SVC+Simulcast (Bitmask de 3 Bytes em RTP Header Extension), o Problema N² (700+ Streams Para 25 Participantes) e o Scallop (Princeton SIGCOMM 2025): SFU em Hardware Tofino2 Com 7-422× Mais Escala

> **Data:** 2026-07-03
> **Loop:** 28 de ∞ (Reescrita)
> **Categoria:** Videoconferência / SFU / Infraestrutura de Mídia

---

## 0. Linhagem

```
Conferências telefônicas — áudio apenas. Bridge hardware.
Polycom/Tandberg (1990s-2000s) — videoconferência em salas. US$ 10K+/endpoint.
Skype (2003) — VoIP P2P. Colapsa com >4 participantes.
WebEx (1995, Cisco 2007) — webconferência corporativa. MCU hardware.
Zoom (2013) — cloud-native. SFU/MMR. SVC. Mobile-first.
Zoom hoje (2026) — MMR como SFU padrão. Scallop SIGCOMM 2025.
```

O Zoom não inventou videoconferência. Substituiu o MCU (que decodifica, mixa e recodifica streams) pelo **MMR (Multimedia Router)** — um SFU que apenas roteia pacotes — transferindo computação do servidor para o cliente e reduzindo custo por participante em ~14×.

---

## 1. Arquitetura Técnica

### 1.1 MMR: O Multimedia Router Que Roteia Sem Transcodificar

O MMR do Zoom é um **Selective Forwarding Unit (SFU)**. Diferentemente de um MCU que decodifica, mixa e recodifica streams de todos os participantes, o MMR apenas roteia pacotes: cada participante envia seu stream ao MMR; o MMR replica para outros participantes **sem decodificar**. O cliente recebe N streams separados e é responsável por decodificar cada um.

**Análise de produção de Princeton (SIGCOMM 2025).** O paper *"Scalable Video Conferencing Using SDN Principles"* (Oliver Michel, Satadal Sengupta, Hyojoon Kim, Ravi Netravali, Jennifer Rexford — Princeton University) analisou **19.704 reuniões Zoom em produção** e confirmou que o SFU encaminha cópias exatas de pacotes RTP — reescrevendo apenas IPs, portas e sequence numbers, sem inspeção de payload. O SFU usa **custom proprietary headers sobre UDP** encapsulando RTP padrão.

### 1.2 O Problema N²: 700+ Streams Para 25 Participantes

Streams crescem quadraticamente: N participantes produzem N×(N-1) streams. Reuniões de 10 pessoas geram ~200 streams; 25 pessoas geram **700+ streams**. Em picos de CPU, tail jitter excede 100ms e o vídeo torna-se inutilizável.

**Scallop (Princeton, SIGCOMM 2025).** SFU em três tiers com hardware data plane: switch **Intel Tofino2** (12.8 Tbps ASIC) ou SmartNIC **NVIDIA BlueField-3** para replicação, forwarding e selective dropping em hardware via P4. Switch agent (CPU) para bandwidth estimation. Controller centralizado para session management.

**Resultados (Tofino2) vs. servidor commodity 32-core:**

| Métrica | Melhoria |
|---|---|
| Escalabilidade (reuniões concorrentes) | **7-422×** (até 128.000 reuniões em um switch) |
| Latência de forwarding | **26,8×** redução mediana |
| Pacotes processados em hardware | **96,5%** dos pacotes, **99,7%** dos bytes |

**Limitações atuais:** sem suporte a E2EE no data plane, sem transcrição ao vivo ou efeitos visuais no SFU, forte dependência de hardware Tofino2 (Packet Replication Engine).

### 1.3 SVC + Simulcast: Adaptação de Bitrate Sem Transcodificação

O Zoom usa **SVC (Scalable Video Coding)** combinado com **Simulcast**. A análise dos traces (Princeton) revelou que pacotes RTP do Zoom carregam um **bitmask de 3 bytes em RTP header extension** indicando a camada de cada pacote. O SFU adapta banda para cada receiver **dropando seletivamente camadas de enhancement** — sem decodificar, apenas filtrando por header.

**Exemplo concreto dos traces:** SFU reduziu bitrate para participante 12 em ~110s e participante 17 em ~200s, puramente por selective forwarding — dropando subsets de pacotes identificados pelo bitmask de camada.

**Simulcast:** cliente envia múltiplas versões (720p, 360p, 180p); SFU seleciona qual encaminhar por receiver. SVC não funciona bem em topologias multi-hop (uma vez que um nó recebe subset de camadas, todos downstream são limitados). Zoom usa ambos na prática.

### 1.4 Zone Controller e Zoom Meetings Hybrid

**Zone Controller:** plano de controle que decide qual MMR hospeda cada reunião. Separação ZC (controle) e MMR (dados) permite escalar independentemente.

**Zoom Node (on-premise):** Hybrid Mode: mídia local roteada pelo Node. Private Mode (iMMR): mídia nunca sai da rede corporativa. **Survivability:** 30 dias offline.

### 1.5 O Scaling da Pandemia: 10M → 300M em 3 Meses

Três fatores: datacenters próprios com ~50% headroom, overflow para Oracle Cloud e AWS, arquitetura stateless do MMR (adicionar servidores = registrar no ZC, sem migração de estado).

---

## 2. Lições de Engenharia

### 2.1 SFU é arquiteturalmente superior a MCU porque transfere computação para o cliente

Cada novo usuário traz consigo a capacidade para processar seu próprio vídeo. A arquitetura escala linearmente.

### 2.2 SVC + Simulcast resolve heterogeneidade de rede sem transcodificação

Dropando camadas por header de 3 bytes, sem decodificar. O SFU nunca vê o payload de vídeo.

### 2.3 Hardware SFU (Tofino2) é 7-422× mais eficiente que software SFU

96,5% dos pacotes em hardware. 128.000 reuniões em um switch. É o futuro da infraestrutura de videoconferência.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Zoom Video Communications |
| **Fundação** | 2011. Lançamento: janeiro 2013. IPO: abril 2019 |
| **Fundador** | Eric Yuan (CEO) |
| **Categoria** | Videoconferência / SFU / SaaS |
| **Arquitetura** | MMR (SFU) + Zone Controller. SVC + Simulcast. RTP/UDP com custom proprietary headers. Bitmask de 3 bytes |
| **Pesquisa** | Scallop (Princeton SIGCOMM 2025): 19.704 reuniões analisadas, N² scaling, Tofino2 7-422×, 128K reuniões/switch, 26,8× latência |
| **Zoom Node** | On-prem: Hybrid + Private (iMMR). Survivability 30d |
| **Concorrentes** | Microsoft Teams, Google Meet, Cisco Webex |

---

## Fontes

- [Princeton / SIGCOMM 2025 — Scallop: Scalable Video Conferencing Using SDN Principles (19.704 Zoom meetings, N² streams, Tofino2 7-422×, 128K meetings/switch)](https://dl.acm.org/doi/10.1145/3718958.3750489)
- [arXiv 2503.11649 — Scallop: Scalable Video Conferencing Using SDN Principles (full paper)](https://ar5iv.labs.arxiv.org/html/2503.11649)
- [Zoom Technical Library — Zoom Meetings Hybrid Explainer (MMR, Zone Controller, iMMR)](https://library.zoom.com/advanced-enterprise-services/zoom-node/zoom-node-explainer/zoom-meetings-hybrid)
