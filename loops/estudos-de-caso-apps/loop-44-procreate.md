# Estudo de Caso 44 — Procreate: O Motor de Renderização Valkyrie Sobre Metal Com 100+ Parâmetros de Brush, Motion Filtering, Barrel Roll, Wet Mix e a Recusa Explícita de Usar AI Generativa

> **Data:** 2026-07-03
> **Loop:** 44 de ∞ (Reescrita)
> **Categoria:** Ilustração Digital / Renderização GPU / Ferramentas Criativas

---

## 0. Linhagem

```
Arte-final manual — mesa de luz, nanquim, letras transferíveis. Profissional.
Photoshop (1990) — padrão profissional. Curva de aprendizado de semestres.
Procreate (2011) — iPad. Metal. Toque + Pencil. "Photoshop no bolso."
Procreate hoje (2026) — 30M+ usuários. Valkyrie 64-bit. 16K×8K. Zero AI generativa.
```

---

## 1. Origem

James Cuda, web designer freelance em Hobart, Tasmânia, ganhou um iPad em 2010. Tentou desenhar. Os apps disponíveis eram lentos, com latência que quebrava o fluxo criativo. Conheceu Lloyd Bottomley — programador autodidata — e juntos passaram 18 meses construindo o Procreate, financiados por freelas. Reescreveram o app três vezes do zero. Últimos A$ 15.000 investidos no lançamento. Três pessoas: Cuda (design), Bottomley (engine), Alanna Cuda (finanças). Zero venture capital. Nunca aceitaram um centavo de VC.

---

## 2. Arquitetura Técnica

### 2.1 Valkyrie: A Engine de Renderização 64-bit Sobre Metal

O Procreate é construído sobre a engine proprietária **Valkyrie** — um motor de pintura 64-bit que usa a API **Metal** da Apple para acesso direto à GPU. A evolução: OpenGL (Silica, 2011) → Metal 2 (Silica-M, 2017) → Metal 3 (Valkyrie, 2019).

**Pipeline de renderização de stroke:**

1. **Input sampling**: Apple Pencil captura pressure (2048 níveis), tilt (0-90°), azimuth e velocity a até 240 Hz. Dados brutos → processados pelo stroke path.

2. **Stroke path processing**: StreamLine (smoothing por média móvel, speed-dependent), Motion Filtering (algoritmo avançado que deleta extremidades de wobble em vez de fazer média — "deletes wobble extremities entirely rather than averaging them"). Expression slider reintroduz sensação manual.

3. **Brush engine computation**: Shape (geometria da ponta, até 16 stamps por ponto) × Grain (textura, Moving ou Texturized, com Scale, Depth, Blend Mode) × Rendering mode (6 modos: Light Glaze a Intense Blending, wet edges, burnt edges). Wet Mix: Dilution (água na tinta), Charge (tinta carregada no pincel, depleta com stroke longo), Attack (quantidade de tinta que adere à tela), Pull (força de arrasto da tinta existente).

4. **GPU compositing**: Valkyrie renderiza strokes via Metal shaders customizados. 120 FPS em ProMotion. Latência próxima de zero. Layer compositing: 25+ blend modes, layer masks, clipping masks.

**Color**: 64-bit color pipeline interno. P3 Wide Color. sRGB. Sem CMYK nativo.

**Canvas**: máximo 16K×8K pixels em iPad Pro. Layer count dinâmico — limitado por RAM (M4 iPad Pro: centenas de layers).

### 2.2 O Brush Studio: 100+ Parâmetros Customizáveis

**Shape**: Source Library com 150+ formas. Input Style: Touch, Azimuth (tilt), Azimuth and barrel roll (Apple Pencil Pro). Scatter, Count (até 16 por ponto), Randomização, Flip X/Y, Roundness Graph com pressure/tilt squash.

**Grain**: Source Library com 100+ texturas. Comportamento Moving (streaky, paint roller travado) ou Texturized (estampa estática). Scale, Zoom, Rotation, Depth, Blend Mode, Brightness/Contrast, Grain Filtering. 3D: "Grain follows camera."

**Stroke Path**: Spacing (frequência de stamp, zero = stroke fluido), Jitter (offset aleatório), Fall Off (fade). Taper: dual slider Pressure/Touch, Size, Opacity, Tip (fine a chunky), Classic Taper toggle.

**Dynamics**: Speed (Size e Opacity por velocidade). Jitter (Size e Opacity aleatórios por stamp).

**Apple Pencil**: Pressure (Size, Opacity, Flow, Bleed — cada com curva customizável). Tilt (Opacity, Gradation, Bleed, Size, Size Compression — trigger point via Tilt Graph 0-90°). Barrel Roll (Pencil Pro: Size, Opacity, Bleed, Relative to stroke toggle, preview com Hover).

**Hover**: Opacity slider, Hover Fill (None/Shape/All), Cursor Outline (None/Contrast/Active color).

**Color Dynamics**: Stamp Jitter e Stroke Jitter (Hue, Saturation, Lightness, Darkness, Secondary Color). Color Pressure, Color Tilt, Color Barrel Roll.

### 2.3 QuickShape e StreamLine

**QuickShape**: desenhe forma aproximada → segure Pencil → snap para geometria perfeita (linha, círculo, quadrado, triângulo, curva). Control points para ajuste pós-snap.

**StreamLine**: Amount (0-100%), Pressure (aplicação de pressão ao longo do stroke). **Motion Filtering**: algoritmo avançado que deleta wobble extremities em vez de fazer média — "more advanced algorithms" que removem tremor preservando intenção. Expression slider para reintroduzir sensação manual. Estabilização global em Actions → Prefs → Pressure & Smoothing.

### 2.4 A Recusa da AI Generativa

Agosto 2024: James Cuda postou vídeo no X. "I really fucking hate generative AI." O Procreate nunca usará AI generativa. A reação: artistas aplaudiram, indústria tech criticou. Ações da Adobe caíram. A decisão é consistente com a filosofia desde 2011: pague uma vez (US$ 12,99), sem assinatura, sem DRM, sem coleta de dados, sem nuvem obrigatória. "Your work belongs to you."

---

## 3. Lições de Engenharia

### 3.1 Rewrite do zero três vezes antes de ter usuários é um luxo que VC não permite

18 meses, 3 reescritas completas, A$ 15.000. Sem VC pressionando por métricas. O resultado: uma engine de rendering que nenhum concorrente com pressão de time-to-market conseguiria construir.

### 3.2 Motion Filtering > StreamLine para estabilização de stroke

Deletar extremidades de wobble preserva mais intenção artística que fazer média móvel. É o tipo de otimização que só importa para artists profissionais — e que define se eles usam seu app ou o concorrente.

### 3.3 "No AI" é uma estratégia de diferenciação de mercado

Em um setor onde Adobe, Canva e todos os concorrentes adicionam AI generativa, dizer "não" a todos eles é mais poderoso que qualquer feature técnica.

---

## 4. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Procreate (Savage Interactive) |
| **Lançamento** | 16 março 2011. Preço: US$ 12,99 (compra única) |
| **Fundadores** | James Cuda (CEO/Design), Lloyd Bottomley (Engine), Alanna Cuda (Finanças) |
| **Categoria** | Ilustração Digital / Renderização GPU |
| **Engine** | Valkyrie: Metal 3, 64-bit color, 120 FPS, 16K×8K canvas, P3 Wide Color |
| **Brush** | 100+ parâmetros: Shape (150+), Grain (100+), 6 Rendering modes, Wet Mix, Color Dynamics |
| **Pencil** | Pressure 2048 níveis, Tilt 0-90°, Azimuth, Barrel Roll (Pro), Hover, Motion Filtering |
| **AI** | Zero AI generativa. "I really fucking hate generative AI." |
| **Concorrentes** | Adobe Fresco, Clip Studio Paint, Affinity Designer |

---

## Fontes

- [Procreate Handbook v5.3 — Brush Studio Settings (todos os 100+ parâmetros)](https://help.procreate.com/procreate/handbook/5.3/brushes/brush-studio-settings)
- [Procreate Official Site — Valkyrie Engine](https://procreate.com/procreate)
- [Ars Technica — Procreate defies AI trend, pledges "no generative AI" (Ago 2024)](https://arstechnica.com/information-technology/2024/08/procreate-defies-ai-trend-pledges-no-generative-ai-in-its-illustration-app/)
- [9to5Mac — iPad app Procreate rejects AI path (Ago 2024)](https://9to5mac.com/2024/08/19/ipad-app-procreate-rejects-ai-path-of-competitors-says-your-work-belongs-to-you/)
- [MacRumors — Procreate 4 Silica M Engine, 64-bit color, P3 Wide Color (2017)](https://www.macrumors.com/2017/09/19/procreate-4-ios-11-support/)
