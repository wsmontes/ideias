# Estudo de Caso 37 — Google Photos: O Banco de 4 Trilhões de Fotos Sobre Spanner (99.999%), a Migração do Magic Editor de ML On-Device Para Cloud Generative AI e o Retorno Com Gemini Nano

> **Data:** 2026-07-03
> **Loop:** 37 de ∞ (Reescrita)
> **Categoria:** Armazenamento / AI / Edge Computing

---

## 0. Linhagem

```
Álbuns de fotos físicos — revelar, imprimir, guardar.
Picasa (2002, Google 2004) — organização desktop. Fim em 2016.
Google Photos (2015) — backup ilimitado, search por conteúdo, ML on-device.
Google Photos hoje (2026) — **9+ trilhões** de fotos/vídeos (4T era 2023). **1,5B MAU** (10º aniversário, 2025). Spanner 99.999%. Magic Editor + Nano Banana.
```

---

## 1. Spanner: O Banco de Dados de 4 Trilhões de Itens

O Google Photos migrou seus metadados para o **Cloud Spanner** — o banco NewSQL globalmente distribuído do Google. **9+ trilhões de fotos e vídeos** (4T era 2023; +125% em 2 anos), **1,5 bilhão de MAU** (10º aniversário, maio 2025). **Strong external consistency** entre regiões. **99,999% de disponibilidade**.

**Arquitetura de sharding**: read-only shards separados de read/write shards — isolando tráfego interativo de workloads batch/ML. Dozens de microserviços + dezenas de pipelines batch Flume para AI/ML, integridade de dados e processamento full-database. **Crescimento 10×** desde a migração; time confiante em suportar mais 10×.

---

## 2. O Arco do Magic Editor: On-Device → Cloud → On-Device

### 2.1 Fase 1: On-Device ML (2018-2021)

- **2018 — Segmentação e background blur**: U-Net de 10MB via TensorFlow Lite (LiteRT), aceleração EdgeTPU no Pixel
- **2021 — Magic Eraser**: orquestração de múltiplos modelos (detection → segmentation → inpainting → GL rendering), centenas de MB
- **Vantagem**: "user brings the compute" — escalonamento gratuito, zero latência de rede, privacidade completa

### 2.2 Fase 2: Cloud Generative AI (2022-2024)

- **Magic Editor**: modelos generativos multi-GB em TPUs/GPUs. Reimaginação de cena, realocação de objetos, inpainting avançado
- **Infrastructure shock**: transição de "compute grátis do usuário" para planejamento de capacidade de servidor
- **Avaliação**: benchmarks tradicionais quebraram. Time criou avaliações por caso de uso
- **UX**: guided object selection — AI como "co-editor", não "prompt engineer"
- **Hallucination como feature**: múltiplas opções geradas, usuário escolhe

### 2.3 Fase 3: Retorno ao On-Device com Gemini Nano (2025+)

Estratégia: **validar com modelos grandes na nuvem → destilar/otimizar → migrar para on-device**. Gemini Nano como plataforma de inferência local. Latência zero, privacidade completa, sem custo de servidor.

---

## 3. Lições de Engenharia

### 3.1 "User brings the compute" é o melhor modelo de escalonamento

Cada novo usuário traz consigo a capacidade computacional. Mesmo princípio do SFU do Zoom.

### 3.2 O arco on-device → cloud → on-device é o padrão da AI generativa

Modelos generativos excederam capacidade mobile em 2022. Destilação, quantização e hardware mais rápido estão fechando o gap. Padrão se repetirá para chatbots, editores de imagem e assistentes de código.

---

## 4. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Google Photos |
| **Lançamento** | 2015 |
| **Categoria** | Armazenamento de Fotos / AI / Edge Computing |
| **Usuários** | 1B+ |
| **Fotos/vídeos** | **9+ trilhões** (2025). 4T era 2023. 370M buscas/mês. 210M edições/mês. |
| **Banco** | Cloud Spanner: strong consistency, 99,999%, sharding R/W |
| **ML Fase 1** | On-device: U-Net 10MB TFLite, EdgeTPU, Magic Eraser |
| **ML Fase 2** | Cloud: modelos multi-GB TPUs/GPUs, Magic Editor |
| **ML Fase 3** | On-device: Gemini Nano, destilação de modelos cloud |

---

## Fontes

- [Google Cloud Blog — Google Photos builds user experience on Spanner (2023)](https://cloud.google.com/blog/products/databases/google-photos-builds-user-experience-on-spanner)
- [ZenML LLMOps — Google Photos Magic Editor: On-Device ML to Cloud Generative AI to Gemini Nano (2025)](https://www.zenml.io/llmops-database/google-photos-magic-editor-transitioning-from-on-device-ml-to-cloud-based-generative-ai-for-image-editing)
