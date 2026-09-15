# Estudo de Caso 30 — Pokémon GO / Niantic: O Lightship VPS Com Pipeline de Reconstrução 3D em 5 Estágios (Splitting→Mapping→Connected Components→Dense 3D Many-Depth→Semantics 20 Classes), o LGM de 150 Trilhões de Parâmetros (50M Redes Neurais) e o Pivot Para Plataforma Espacial (Scopely US$ 3,5B)

> **Data:** 2026-07-03
> **Loop:** 30 de ∞ (Reescrita)
> **Categoria:** AR / Infraestrutura Geoespacial / 3D Computer Vision

---

## 0. Linhagem

```
Keyhole (2001) → Google Earth (2004). Niantic Labs (2010) dentro do Google.
Ingress (2012) — 15M downloads. 5M POIs submetidos. O banco de dados geoespacial.
Pokémon GO (2016) — o maior mecanismo de coleta de dados geoespaciais disfarçado de jogo.
Niantic Spatial (2024-2025) — pivot para plataforma. Games vendidos para Scopely (US$ 3,5B).
```

O Pokémon GO não é um jogo — é o maior mecanismo de coleta de dados geoespaciais do mundo, disfarçado de entretenimento. Cada jogador que aponta a câmera para uma igreja, estátua ou parque está gerando scans 3D que alimentam a infraestrutura de posicionamento visual que a Niantic está construindo.

---

## 1. Arquitetura Técnica

### 1.1 Lightship VPS: Posicionamento Visual Centimétrico

O **Visual Positioning System (VPS)** determina a pose 6-DoF (posição + orientação) de um dispositivo com **precisão centimétrica** usando apenas a câmera do smartphone. GPS: erro de 5-15m. VPS: erro de centímetros. A diferença é a diferença entre "o Pokémon está em algum lugar nesta rua" e "o Pokémon está exatamente neste degrau da escada."

**Escala**: 10 milhões de localizações escaneadas, **1 milhão ativadas** em produção. ~1 milhão de novos scans por semana. Cobertura densa em San Francisco, Los Angeles, Seattle, Nova York, Londres, Tóquio. ~250 milhões de pessoas a menos de 5 minutos de caminhada de uma localização VPS-ativada.

### 1.2 O Pipeline de Reconstrução 3D em 5 Estágios

**Stage 1 — Splitting Scans.** Clipes de 15-30 segundos (~300 frames) de qualquer smartphone são divididos em frames individuais. Deriva de GPS e erros de posição são corrigidos dividindo scans em múltiplos "nós."

**Stage 2 — Location Mapping.** Cada scan dividido é transformado em mapas 3D especializados para localização — otimizados para matching de imagem contra query, não para visualização. Dois tipos de mapa construídos concorrentemente: localization map (machine-readable) e mesh+texture map (human-readable).

**Stage 3 — Connected Components.** Mapas 3D são relacionados entre si usando GPS para limitar complexidade combinatória. **Global bundle adjustment** minimiza erro de reprojeção e rejeita outliers — ajustando simultaneamente posições de câmera e pontos 3D. Output: sequência coerente de 5-10 minutos.

**Stage 4 — Dense 3D Reconstruction.** Mapas de profundidade computados por imagem usando **Many-Depth** — o modelo proprietário de depth estimation da Niantic. Redundância na sequência filtra ruído para produzir mesh 3D fundido com textura de alta resolução. **Não requer LiDAR** — funciona com câmera RGB de qualquer smartphone.

**Stage 5 — Context + Semantics.** Segmentação semântica classifica cada pixel em **20 classes**: céu, solo natural/artificial, água, pessoas, edifícios, vegetação, grama, flores, troncos de árvores, animais, areia, telas, terra, veículos, comida, assentos, neve. Permite AR context-aware.

**Gaussian Splatting (2024).** Scaniverse processa scan iOS → Gaussian splat → visualizável em Meta Quest em <5 minutos. "World's largest collection of 3D Gaussian splats."

**Infraestrutura cloud:** hierarchical LRU cache (RAM+SSD), map sharding com virtualization layer em cloud VMs, indexing via Google Cloud Bigtable. Latência típica: poucas centenas de ms após inicialização do cliente.

### 1.3 Large Geospatial Model: 150 Trilhões de Parâmetros, 50 Milhões de Redes Neurais

O **LGM** (novembro 2024) não é um modelo monolítico. São **50 milhões de redes neurais** — uma por localização no mundo — cada uma com ~3 milhões de parâmetros comprimindo milhares de imagens em representação neural que codifica a cena 3D implicitamente em seus pesos (baseado nas pesquisas ACE 2023 e ACE Zero 2024). Total: **150 trilhões de parâmetros** distribuídos.

**De local para global.** Atualmente, redes locais são independentes — só reconhecem o que foram treinadas para reconhecer. A visão do LGM é **destilar informação comum** entre todas as redes locais em um modelo global capaz de **extrapolar** — inferir como é a parte de trás de uma igreja nunca vista baseado em milhares de igrejas similares. "Extrapolating locally by interpolating globally."

**ACE Zero (ECCV 2024, oral presentation).** Treina visual relocalizer com uma única imagem, estima poses para milhares de imagens em **1 hora ou menos em uma única GPU** — on par com SOTA em reconstrução 3D mas significativamente mais rápido.

**Controvérsia de dados (novembro 2024):** reportagens revelaram que jogadores estavam contribuindo scans para treinar o LGM sem saber. Niantic confirmou o uso dos dados — scans de locais públicos, anonimizados.

### 1.4 O Pivot: Games Para Scopely (US$ 3,5B), Plataforma Espacial

Março 2025: games (Pokémon GO, Pikmin Bloom, Monster Hunter Now) vendidos para Scopely por **US$ 3,5 bilhões**. **Niantic Spatial Inc.** emerge como empresa pura de plataforma. Três pilares: **Capture** (Scaniverse, crowdsourced scans, drones), **Localize** (VPS, LGM), **Augment** (Lightship ARDK, 8th Wall, Niantic Studio). Parcerias: **Snap Inc.** (VPS no Snapchat e Spectacles, junho 2025), **Vantor** (defesa: navegação de drones sem GPS).

---

## 2. Lições de Engenharia

### 2.1 Entretenimento é o mecanismo de coleta de dados mais eficiente já inventado

Nenhuma campanha paga conseguiria motivar milhões de pessoas a caminhar quilômetros apontando câmeras. Pokémon GO conseguiu porque não era coleta de dados — era um jogo.

### 2.2 Global bundle adjustment transforma scans amadores em mapas precisos

GPS deriva, câmera treme, iluminação muda. Bundle adjustment ajusta posições e pontos simultaneamente — o algoritmo que transforma caos em geometria coerente.

### 2.3 Redes neurais locais com destilação global são scaling mais eficiente que modelo monolítico

50M redes de 3M parâmetros cada: paralelizável, incremental, novas localizações não exigem retreino global. O design oposto de um LLM monolítico — e correto para o domínio.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Pokémon GO / Niantic Spatial Inc. |
| **Lançamento** | 6 julho 2016. Niantic Spatial: março 2025 |
| **Categoria** | AR / Infraestrutura Geoespacial |
| **VPS** | 10M+ localizações escaneadas, 1M+ ativadas, ~1M scans/semana, precisão centimétrica |
| **Pipeline 3D** | 5 estágios: Splitting → Location Mapping → Connected Components (bundle adjustment) → Dense 3D (Many-Depth) → Semantics (20 classes) |
| **LGM** | 150T parâmetros, 50M redes neurais (~3M params/local). ACE Zero (ECCV 2024). Gaussian Splats |
| **Pivot** | Games → Scopely (US$ 3,5B). Niantic Spatial Inc. Parcerias Snap, Vantor |
| **Concorrentes** | Snap (Spectacles), Meta (Quest), Apple (Vision Pro) |

---

## Fontes

- [Niantic Labs — Lightship VPS Part 2: Building Our 3D Map From Crowdsourced Scans (5-stage pipeline detalhado)](https://nianticlabs.com/news/vps-part-2)
- [Niantic Spatial — Large Geospatial Model: Advancing Spatial Intelligence (Nov 2024, 150T params, 50M networks)](https://www.nianticspatial.com/blog/largegeospatialmodel)
- [Niantic Labs — ECCV 2024: ACE Zero, DoubleTake, MicKey (4 papers, oral presentation)](https://www.nianticlabs.com/news/eccv2024)
- [Frontier VC — Building a Large Geospatial Model to Achieve Spatial Intelligence (Dez 2024)](https://frontiervc.com/partner-pov/2024/12/11/building-a-large-geospatial-model-to-achieve-spatial-intelligence)
- [Ars Technica — Niantic uses Pokémon Go player data to build AI navigation system (Nov 2024)](https://arstechnica.com/ai/2024/11/niantic-uses-pokemon-go-player-data-to-build-ai-navigation-system/)
- [Harmony Studios — Niantic Spatial SDK features and pricing (Abr 2025)](https://www.harmony.co.uk/insights/niantic-spatial-sdk-lightship)
