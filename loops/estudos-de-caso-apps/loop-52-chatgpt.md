# Estudo de Caso 52 — ChatGPT: A Infraestrutura de Inferência Com H100, Continuous Batching (10-23×), Disaggregated Prefill-Decode, FlashAttention-3, PagedAttention, FP8 Quantization e Speculative Decoding

> **Data:** 2026-07-03
> **Loop:** 52 de ∞ (Reescrita)
> **Categoria:** Inteligência Artificial / Infraestrutura de Inferência / LLM

---

## 0. Linhagem

```
GPT-1 (2018) — 117M parâmetros. Pré-treinamento generativo.
GPT-2 (2019) — 1.5B. "Too dangerous to release."
GPT-3 (2020) — 175B. Few-shot learning.
ChatGPT (Nov 2022) — GPT-3.5 + RLHF. 100M usuários em 2 meses.
ChatGPT hoje (2026) — 400M+ usuários semanais. Infraestrutura de inferência em escala de datacenter.
```

---

## 1. Arquitetura de Inferência

### 1.1 Continuous Batching e PagedAttention

O gargalo fundamental em servir LLMs é que o carregamento dos pesos do modelo da VRAM domina o tempo de processamento. **Continuous batching** (vLLM, TGI v3) agrupa centenas de requisições em um único batch, amortizando o custo de weight loading. Throughput: **10-23×** superior a request-level dynamic batching.

**PagedAttention** (SOSP 2023): gerencia KV cache como páginas de memória virtual, eliminando fragmentação. Prefix caching: reusa KV cache para prefixos compartilhados entre requisições. TGI v3: 13× vs vLLM com cache, 30× sem cache em prompts longos.

### 1.2 Disaggregated Prefill-Decode (2024-2025)

A inovação arquitetural mais significativa: separar **prefill** (compute-bound, processa prompt de entrada) e **decode** (memory-bound, gera tokens sequencialmente) em pools de GPU distintos.

- **Prefill cluster**: H100/B200 — compute forte, batches menores
- **Decode cluster**: H200/B200/B300 — alta bandwidth de memória, batches grandes
- **KV cache transfer**: InfiniBand/NVLink, GPU Direct RDMA, async pipelining, KV compression (FP8/INT8)

Sistemas como **DistServe** (OSDI 2024), **Mooncake** e **DeepSeek** implementam essa arquitetura. A transferência do KV cache entre clusters é o desafio central.

### 1.3 Paralelismo de Modelo

- **Tensor Parallelism (TP)**: divide operações de matriz entre GPUs via NVLink (~centenas GB/s). Single-node.
- **Pipeline Parallelism (PP)**: divide camadas sequencialmente entre GPUs via PCIe (~64 GB/s). Cross-node.
- **Expert Parallelism (EP)**: All-to-All RDMA para arquiteturas Mixture-of-Experts.
- **Combinação TP+PP**: TP intra-node via NVLink, PP cross-node via InfiniBand/RoCE.

### 1.4 Otimizações de Kernel e Quantização

**FlashAttention-3** (julho 2024): explora Hopper async Tensor Cores + FP8. 1,5-2,0× sobre FA2 em H100. **Multi-Query Attention (MQA)** e **Grouped-Query Attention (GQA)**: reduzem KV head count, cortando tamanho e bandwidth do KV cache.

**Quantização de pesos**: AWQ (4-bit, MLSys 2024), GPTQ (3-4-bit). **Ativações**: SmoothQuant (INT8, W8A8). **KV cache**: KIVI (2-bit, ICML 2024), KVQuant (3-bit, NeurIPS 2024). FP8 inference paths em H100 via TensorRT-LLM.

### 1.5 Speculative Decoding

Modelo draft (menor, rápido) propõe tokens → modelo target (maior, preciso) verifica em paralelo. **2-3× speedups**. **Medusa** (2024): multi-token heads no mesmo modelo, 2,3-3,6× reportado. Reduz latência percebida sem degradar qualidade de output.

### 1.6 O Pipeline de Treinamento: Pre-training → SFT → RLHF

**Pre-training**: transformer treinado em trilhões de tokens para prever próximo token. **Supervised Fine-Tuning (SFT)**: fine-tuned em demonstrações humanas de conversas ideais. **Reward Model**: modelo separado treinado para pontuar respostas via comparações humanas. **RLHF**: Proximal Policy Optimization (PPO) usando reward model como proxy de preferência humana. RLHF é o que transforma um autocompletador de texto em um assistente utilizável.

---

## 2. Lições de Engenharia

### 2.1 Disaggregated prefill-decode resolve o conflito fundamental entre compute e memory

Prefill é compute-bound; decode é memory-bound. Forçá-los a compartilhar GPUs subutiliza ambos. Separar em pools dedicados otimiza cada workload independentemente.

### 2.2 PagedAttention transforma KV cache de gargalo de memória em problema de gerenciamento de páginas

Fragmentação de KV cache era o limitante de throughput. PagedAttention aplica técnicas de memória virtual — páginas, alocação dinâmica, defragmentação — a um problema que não era reconhecido como gerenciamento de memória.

### 2.3 Speculative decoding é a otimização mais elegante: use um modelo pequeno para acelerar um grande

Modelo draft rápido gera candidatos; modelo target verifica em paralelo. Ganho de 2-3× sem perda de qualidade. Não requer modificar o modelo target.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | ChatGPT (OpenAI) |
| **Lançamento** | 30 novembro 2022 |
| **Categoria** | Inteligência Artificial / LLM / API |
| **Usuários** | 400M+ semanais (2025) |
| **Inferência** | H100/Blackwell. vLLM, TensorRT-LLM. Continuous batching (10-23×). Disaggregated prefill-decode |
| **Paralelismo** | TP + PP + EP. NVLink + InfiniBand/RoCE |
| **KV Cache** | PagedAttention. Prefix caching (13-30×). FP8/INT8 quantization |
| **Atenção** | FlashAttention-3 (1,5-2×). MQA/GQA. Speculative decoding (2-3×) |
| **Treinamento** | Pre-training → SFT → Reward Model → RLHF (PPO) |
| **Concorrentes** | Anthropic Claude, Google Gemini, Meta Llama |

---

## Fontes

- [vLLM / PagedAttention — SOSP 2023 (arXiv:2309.06180)](https://arxiv.org/abs/2309.06180)
- [FlashAttention-3 — arXiv:2407.08608 (Jul 2024)](https://arxiv.org/abs/2407.08608)
- [AskHNDigests — How ChatGPT Serves 700M Users (inference architecture deep-dive)](https://askhndigests.com/blog/how-chatgpt-serves-millions-ai-inference)
- [DistServe — Disaggregated Prefill-Decode (OSDI 2024)](https://arxiv.org/abs/2401.09670)
- [KIVI — KV Cache Quantization (ICML 2024, arXiv:2402.02750)](https://arxiv.org/abs/2402.02750)
- [Speculative Decoding (arXiv:2211.17192)](https://arxiv.org/abs/2211.17192)
