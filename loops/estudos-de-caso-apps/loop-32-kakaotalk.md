# Estudo de Caso 32 — KakaoTalk: O Protocolo LOCO (BSON, AES-CFB Sem MAC, Sem Autenticação de Servidor, Sem Replay Protection), a Infraestrutura OpenStack Com 7.000 Clusters K8s + Cilium/eBPF e o Kanana MoE (64 Experts, 13,4B Parâmetros, TPU Trillium 2,7×)

> **Data:** 2026-07-03
> **Loop:** 32 de ∞ (Reescrita)
> **Categoria:** Mensageria / Infraestrutura / Protocolo / AI

---

## 0. Linhagem

```
SMS (2000s) — texto pago por unidade. Coreia: operadoras cobravam caro.
KakaoTalk (2010) — mensagens gratuitas via dados móveis. Adoção instantânea.
Kakao (2014) — fusão com Daum. KakaoPay, KakaoBank, KakaoMobility.
KakaoTalk hoje (2026) — 93% da Coreia do Sul. 7.000 clusters K8s. Kanana MoE. LOCO.
```

O KakaoTalk não venceu por superioridade técnica — venceu porque entrou no mercado coreano no momento exato em que smartphones se massificavam e operadoras ainda cobravam por SMS. Uma vez que 93% da população estava na plataforma, o custo de troca tornou-se proibitivo. Desse monopólio de mensageria, a Kakao expandiu-se para pagamentos (KakaoPay), banco digital (KakaoBank), transporte (KakaoMobility) e AI (Kanana).

---

## 1. Arquitetura Técnica

### 1.1 O Protocolo LOCO: Velocidade Sobre Segurança

O protocolo LOCO — a espinha dorsal da comunicação no KakaoTalk — é um protocolo binário proprietário documentado apenas por engenharia reversa. Opera sobre **TCP raw** (não HTTP), com payloads em **BSON (Binary JSON)** e criptografia **AES-CFB**. Foi projetado em 2010 para velocidade em redes móveis coreanas (3G, alta latência, baixa largura de banda).

**Três vulnerabilidades estruturais** documentadas por Dawin Schmidt (HITB Bangkok, BSides Munich 2024):

1. **Sem autenticação de servidor.** O cliente LOCO não valida a identidade do servidor de mensageria. Um atacante capaz de executar MITM (Wi-Fi comprometida, DNS spoofing, BGP hijack) pode interceptar e modificar mensagens sem detecção — catastrófico para um app usado para banking e pagamentos.

2. **AES-CFB sem MAC.** O modo CFB (Cipher Feedback) é malleable — bits do ciphertext podem ser flipados para produzir alterações previsíveis no plaintext, sem que o receptor detecte a modificação. A ausência de Message Authentication Code significa zero garantia de integridade. O ataque EFAIL de 2018 demonstrou que malleability de ciphertext era explorável em escala.

3. **Sem prevenção de replay.** O protocolo não inclui nonces, timestamps ou contadores de mensagem. Um atacante que capture uma mensagem criptografada pode reenviá-la posteriormente.

**Secret Chat (E2EE).** Não ativado por default. Sem forward secrecy (troca de chaves não é efêmera). Sem Double Ratchet Algorithm. Distribuição de chaves públicas confia no servidor Kakao — que pode substituir chaves sem detecção.

**Cronologia.** Vulnerabilidades reportadas pela primeira vez em 2016. Kakao começou a trabalhar em correções apenas em **julho de 2024** — oito anos depois. Um 1-click exploit foi reportado em dezembro 2023 via Bug Bounty (apenas cidadãos coreanos recebem recompensas).

### 1.2 Infraestrutura: 7.000 Clusters K8s, 120.000 Nós, OpenStack, Cilium/eBPF

O KakaoTalk opera sobre **OpenStack** on-premise com **7.000+ clusters Kubernetes** e **120.000+ nós** em múltiplas zonas de disponibilidade. **Cilium/eBPF** substituiu kube-proxy como CNI — iptables escala mal em clusters grandes; eBPF programa o kernel dinamicamente com programas verificados. **Cilium Cluster Mesh + Hubble** para gerenciamento multi-zona.

A plataforma de engenharia oferece K8s como serviço sobre OpenStack: times de produto solicitam clusters via API; a plataforma provisiona networking (Cilium), storage, monitoring e logging automaticamente. A filosofia de automação — "se um humano precisa tocar numa VM, a automação falhou" — permitiu que 2 engenheiros gerenciassem 5.000 VMs em 2013, escalando para a infraestrutura atual sem crescimento proporcional da equipe.

### 1.3 AI Infrastructure: Kanana MoE Com 64 Experts em TPU Trillium

Em 2024, a Kakao migrou treinamento de AI de GPUs NVIDIA para **Cloud TPUs do Google** usando framework **JAX**:

**Kanana 2.1B**: modelo denso treinado do zero com MaxText + FSDP. Performance comparável ao pipeline GPU Megatron-LM. Depth upscaling: 8B → 9.8B com melhorias consistentes.

**Kanana-MoE**: upcycling do modelo denso de 2.1B para **Mixture-of-Experts de 13,4B parâmetros** (2,3B ativos por token), **64 experts, 8 ativos por token**. Treinado em TPU v5e com kernels Megablocks MoE (Group GEMM) integrados ao JAX. Ganhos particularmente fortes em code e math.

**TPU Trillium (v6e)**: **throughput 2,7× maior** que v5e, alterando apenas parâmetros de configuração do cluster XPK. Modelos Kanana-MoE open-source no Hugging Face.

**Avaliação de segurança (dezembro 2025):** MSIT + AI Safety Institute da Coreia conduziram primeira avaliação oficial de segurança de AI usando benchmark AssurAI (11.480 exemplos multimodais em coreano). Kanana Essence 1.5 mostrou "boa estabilidade técnica" em todas as categorias de risco.

---

## 2. Críticas

**2.1 LOCO como dívida técnica de segurança de oito anos.** Reportado em 2016, corrigido em 2024. A demora ilustra o custo de protocolos proprietários sem escrutínio público — vulnerabilidades foram descobertas por engenharia reversa, não por revisão de especificação.

**2.2 Monopólio de mensageria como risco sistêmico.** 93% de penetração torna KakaoTalk infraestrutura crítica nacional. O incêndio no datacenter SK C&C em 2022 derrubou a plataforma por dias, paralisando comunicação, pagamentos e banking. O governo coreano iniciou investigações antitruste.

---

## 3. Lições de Engenharia

### 3.1 Protocolos proprietários acumulam dívida técnica de segurança mais rápido que protocolos abertos

LOCO nunca foi auditado publicamente. Signal Protocol, MLS e TLS 1.3 recebem escrutínio contínuo; protocolos proprietários recebem escrutínio apenas de atacantes.

### 3.2 Cilium/eBPF é a substituição correta para kube-proxy em clusters >1.000 nós

Iptables escala linearmente com regras; eBPF escala com programas verificados no kernel. A diferença é existencial em 7.000 clusters.

### 3.3 Upcycling de modelos densos para MoE é a estratégia de escala mais eficiente

Treinar MoE do zero é caro e instável. Treinar denso → validar → upcycle para MoE preserva investimento. Mesma estratégia da Google (Gemini) e Anthropic.

---

## 4. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | KakaoTalk (Kakao Corporation) |
| **Fundação** | Março 2010. Fusão com Daum: 2014 |
| **Categoria** | Mensageria / Super-App / Infraestrutura |
| **Penetração** | 93% da Coreia do Sul (~49M de 52M) |
| **Protocolo** | LOCO (proprietário): TCP raw, BSON, AES-CFB (sem MAC, sem server auth, sem replay). Secret Chat: ECDH sem forward secrecy |
| **Infraestrutura** | OpenStack on-premise. 7.000+ clusters K8s. 120.000+ nós. Cilium/eBPF CNI |
| **AI** | JAX + TPU v5e/Trillium v6e. Kanana-MoE: 13,4B params (2,3B ativos), 64 experts, 8 ativos. Trillium 2,7× throughput |
| **Concorrentes** | WhatsApp, LINE, Telegram (mínima penetração na Coreia) |

---

## Fontes

- [HITB Bangkok 2024 / BSides Munich 2024 — Leaking Kakao: How a Combination of Bugs in KakaoTalk Compromises User Privacy (Dawin Schmidt, LOCO protocol analysis)](https://conference.hitb.org/hitbsecconf2024bkk/)
- [Google Cloud Blog — Kakao's journey with JAX and Cloud TPUs (Ago 2025, Kanana MoE, Trillium 2,7×)](https://cloud.google.com/blog/products/infrastructure-modernization/kakaos-journey-with-jax-and-cloud-tpus)
- [CNCF Case Study — Kakao: 7.000+ K8s clusters, 120.000+ nodes, Cilium/eBPF](https://www.cncf.io/case-studies/kakao/)
- [Superuser OpenInfra — KakaoTalk speaks volumes about the future of cloud services (5.000 VMs, 2 engenheiros)](https://superuser.openinfra.dev/articles/kakaotalk-speaks-volumes-about-the-future-of-cloud-services/)
