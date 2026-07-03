# 🎨 Estudo de Cores Loop 07 — Wide Gamut: As Cores Que Seu Hex Não Alcança (E Como Usá-las)

> **Data:** 2026-07-02
> **Loop:** 7 de ∞
> **Tema:** sRGB cobre apenas 35% das cores que você enxerga. Display P3 cobre 50%. Rec.2020 cobre 75%. HDR adiciona 10,000 nits de brilho — cores MAIS BRILHANTES QUE BRANCO. Como sair da prisão do hex code e usar cores que literalmente NÃO EXISTEM em `#RRGGBB`. E por que a maioria dos apps ainda vive em 1996.

---

## 0. A Prisão Colorida: 30 Anos Vendo 1/3 Das Cores

1996. Bill Clinton é presidente. Spice Girls lançam "Wannabe". A HP e a Microsoft criam o **sRGB** — um espaço de cor para monitores CRT que define o padrão da web.

**2026. Você ainda está usando sRGB.**

Seu iPhone 16 Pro pode mostrar cores que o sRGB NEM CONHECE. Seu MacBook Pro tem tela XDR com 1,600 nits de pico. Sua TV LG OLED suporta Dolby Vision com cores além de Rec.2020.

**E você ainda escreve `#8B5CF6`.**

`#8B5CF6` é um código hexadecimal que SÓ pode representar cores dentro do triângulo sRGB. É uma PRISÃO de 6 caracteres. Toda cor que você já escreveu como hex code é uma cor MUTILADA — a melhor aproximação que o sRGB consegue fazer da cor que você realmente queria.

**Exemplos de cores que NÃO EXISTEM em hex:**

- O verde de uma folha iluminada pelo sol de verão
- O laranja neon de um tênis Nike Vaporfly
- O rosa de um outdoor da Glossier em Sunset Boulevard
- O azul do mar de Angra dos Reis ao meio-dia
- O roxo de uma flor de ipê-roxo em Brasília

Todas essas cores estão ALÉM do sRGB. Você NUNCA as viu numa tela — a não ser que estivesse usando um app com suporte a wide gamut (quase nenhum).

---

## 1. O Mapa do Território: Quem Cobre O Quê

### 1.1 Os Gamuts Dentro da Ferradura CIE 1931

```
Quanto da visão humana cada espaço cobre:

sRGB:       ████████░░░░░░░░░░░░  35%
Display P3: ████████████░░░░░░░░  50%  (+43% mais cores que sRGB)
Adobe RGB:  ████████████░░░░░░░░  52%  (focado em ciano/verde — impressão)
DCI-P3:     ██████████████░░░░░░  54%  (cinema digital — irmão do Display P3)
Rec.2020:   ██████████████████░░  75%  (HDR standard — ainda inalcançável em hardware)
ACEScg:     ███████████████████░  85%  (VFX profissional)
Visão humana: ████████████████████ 100%
```

### 1.2 Onde o P3 Ganha do sRGB

O P3 não adiciona cores UNIFORMEMENTE. Ele expande MUITO em:
- **Vermelhos profundos** (maçã, Ferrari, batom)
- **Laranjas vibrantes** (pôr-do-sol, tangerina, neon)
- **Verdes intensos** (folhagem tropical, grama, esmeralda)
- **Magenta/rosa saturados** (flores, moda, cosméticos)

E expande POUCO em:
- Azuis (sRGB já cobre bem)
- Cianos (sRGB já cobre bem)

**Por que isso importa:** apps de moda, beleza, comida, natureza, esportes e luxo são os que MAIS se beneficiam de P3. Apps corporativos (muito azul e cinza) se beneficiam MENOS.

### 1.3 O Hardware Que Suporta Wide Gamut (2026)

| Dispositivo | Gamut | Desde |
|---|---|---|
| iPhone (todos a partir do 7) | Display P3 | 2016 |
| MacBook Pro (todos a partir de 2016) | Display P3 | 2016 |
| iPad Pro (todos) | Display P3 | 2016 |
| iMac (todos a partir de 2015) | Display P3 | 2015 |
| Mac Studio Display / Pro Display XDR | Display P3 + HDR | 2022/2019 |
| Samsung Galaxy S (série) | DCI-P3 | ~2018 |
| Google Pixel | Display P3 | ~2019 |
| Monitores profissionais (Eizo, BenQ, ASUS ProArt) | Adobe RGB / DCI-P3 | variado |
| Monitores baratos / laptops Windows básicos | **sRGB apenas** | sempre |
| TVs OLED LG/Sony/Samsung | DCI-P3 + HDR | ~2018 |

**Cobertura estimada de dispositivos com P3: ~70% dos smartphones, ~50% dos laptops.** Mas a maioria dos apps e sites NÃO USA — entrega sRGB para todo mundo.

---

## 2. CSS: Como Sair da Prisão do Hex

### 2.1 `color(display-p3 ...)` — Cor Direta em P3

```css
/* sRGB (o que você sempre fez) */
--red-srgb: #FF3B30;           /* vermelho "vivo" do sRGB */
--red-srgb: rgb(255 59 48);    /* mesmo, em rgb() */

/* Display P3 (o que você DEVERIA fazer) */
--red-p3: color(display-p3 1 0.15 0.12);  /* vermelho QUE NÃO EXISTE EM HEX */
```

A notação `color(display-p3 R G B)` usa valores de 0 a 1 (não 0-255). As mesmas coordenadas produzem cores RADICALMENTE mais vivas:

```css
/* Mesmos números, espaços diferentes = cores diferentes */
--red-srgb: rgb(1, 0, 0);             /* vermelho sRGB — "OK" */
--red-p3:   color(display-p3 1 0 0);  /* vermelho P3 — MUITO MAIS VIVO */

/* São cores DIFERENTES! display-p3(1,0,0) está FORA do sRGB */
```

### 2.2 Fallback: A Regra de OURO

```css
/* 1. Declare sRGB primeiro (fallback universal) */
/* 2. Sobrescreva com P3 (navegadores modernos) */

--vivid-red: rgb(255 59 48);                     /* fallback sRGB */
--vivid-red: color(display-p3 1 0.15 0.12);     /* P3 para quem pode */

/* OU use @supports */
--vivid-red: rgb(255 59 48);

@supports (color: color(display-p3 1 0 0)) {
  --vivid-red: color(display-p3 1 0.15 0.12);
}
```

### 2.3 OKLCH Além do sRGB

OKLCH é o JEITO CERTO de especificar cores wide gamut porque:
- Chroma > ~0.37 automaticamente entra em P3
- Chroma > ~0.50 entra em Rec.2020
- Você NÃO PRECISA saber qual gamut está usando — o navegador resolve

```css
/* sRGB máximo para azul: */
--blue-srgb-max: oklch(0.55 0.22 260);

/* P3 (chroma mais alto que o sRGB alcança): */
--blue-p3: oklch(0.55 0.30 260);    /* chroma 0.30 > 0.22 máximo do sRGB */

/* Rec.2020 (chroma ainda mais alto): */
--blue-2020: oklch(0.55 0.40 260);   /* chroma 0.40 — só telas HDR topo de linha */

/* O NAVEGADOR FAZ O GAMUT MAPPING AUTOMATICAMENTE */
/* Se a tela for sRGB, ele reduz o chroma para o máximo que o sRGB alcança */
/* Se for P3, mostra em P3. Se for Rec.2020, mostra em Rec.2020. */
```

**Isso é REVOLUCIONÁRIO.** Você escreve `oklch(0.55 0.35 260)` UMA VEZ e o navegador adapta para o gamut da tela. Sem fallbacks. Sem `@supports`. Sem código condicional.

### 2.4 A Tabela de Chroma Máximo por Gamut

| Matiz | Chroma máx sRGB | Chroma máx P3 | Chroma máx Rec.2020 |
|---|---|---|---|
| Vermelho (30°) | 0.25 | 0.40 | 0.55 |
| Laranja (60°) | 0.22 | 0.38 | 0.52 |
| Verde (140°) | 0.24 | 0.42 | 0.58 |
| Ciano (200°) | 0.18 | 0.21 | 0.30 |
| Azul (260°) | 0.22 | 0.30 | 0.42 |
| Magenta (320°) | 0.25 | 0.40 | 0.56 |

**O P3 MAIS dobra o chroma em verdes, vermelhos e magentas.** E OKLCH te dá acesso a isso com um parâmetro.

---

## 3. HDR: Cores Mais Brilhantes Que Branco

### 3.1 O Que É HDR

HDR não é "mais cores." HDR é **MAIS LUMINÂNCIA.** Em SDR (Standard Dynamic Range), o branco é ~100 nits. Em HDR, o branco pode ser 1,000–10,000 nits. E objetos DENTRO da cena podem ter brilhos DIFERENTES — uma lâmpada pode ser 500 nits enquanto o resto da tela está em 100 nits.

**Tradução:** em HDR, existe "branco" (fundo da tela, ~100 nits) e existe BRANCO (reflexo do sol numa poça, ~1,000 nits). São cores DIFERENTES. O hex code `#FFFFFF` não sabe qual dos dois você quer.

### 3.2 PQ e HLG: As Curvas de Transferência

| Curva | Nome | Máximo | Uso |
|---|---|---|---|
| **PQ** (ST.2084) | Perceptual Quantizer | 10,000 nits | Dolby Vision, HDR10 |
| **HLG** | Hybrid Log-Gamma | 5,000 nits | Broadcast TV (compatível com SDR) |

PQ é a curva do Dolby Vision e HDR10. HLG é usada em TV aberta (BBC, NHK) porque é retrocompatível: uma TV SDR mostra HLG como SDR razoável; uma TV HDR mostra o HDR completo.

### 3.3 CSS `dynamic-range` e HDR

```css
/* Detecta se o dispositivo suporta HDR */
@media (dynamic-range: high) {
  :root {
    /* Cores HDR — brilho além de SDR */
    --sun-reflection: oklch(0.95 0.05 90);  /* L=0.95 + HDR = MAIS BRILHO */
    --neon-sign: oklch(0.90 0.35 30);        /* vermelho NEON em HDR */
  }
}

@media (dynamic-range: standard) {
  :root {
    --sun-reflection: oklch(0.95 0.02 90);   /* SDR: reduz chroma */
    --neon-sign: oklch(0.85 0.22 30);         /* SDR: sem o brilho extra */
  }
}
```

### 3.4 O Futuro: `color()` com Perfis HDR

```css
/* CSS Color Level 5 (draft) — já funciona em Safari */
--hdr-red: color(display-p3 1.5 0 0);  /* valores >1 = HDR! */

/* Rec.2100 PQ — HDR absoluto */
--absolute-hdr: color(rec2100-pq 0.4 0.3 0.2);
```

---

## 4. Como Testar Wide Gamut (Sem Chutar)

### 4.1 O Hardware Necessário

- Mac com tela P3 (MacBook Pro 2016+, iMac 2015+, Studio Display)
- iPhone 7+ (todos têm P3)
- iPad Pro (todos têm P3)

**Monitores externos baratos NÃO servem.** A maioria é sRGB. Você precisa de um monitor "DCI-P3 95%+" ou "Display P3" para ver as cores.

### 4.2 Ferramentas de Desenvolvimento

**Chrome DevTools:**
1. Rendering tab → "Emulate CSS media feature: color-gamut: p3"
2. Isso força o navegador a REPORTAR suporte a P3 (mesmo em tela sRGB)

**Safari Web Inspector:**
- Suporte nativo a P3 — sem necessidade de emulação
- Mostra cores P3 no seletor de cores

**Ferramentas de validação:**
```javascript
// Verificar se o display atual suporta P3
const supportsP3 = window.matchMedia('(color-gamut: p3)').matches;
console.log('Display P3:', supportsP3);
```

### 4.3 O Teste Visual

Crie um quadrado com `color(display-p3 1 0 0)` ao lado de um quadrado com `rgb(255 0 0)`. Se você vê diferença, sua tela é P3. Se são idênticos, sua tela é sRGB.

```html
<div style="display:flex;gap:20px">
  <div style="width:200px;height:200px;background:rgb(255,0,0)"></div>
  <div style="width:200px;height:200px;background:color(display-p3 1 0 0)"></div>
</div>
```

---

## 5. Imagens Wide Gamut

### 5.1 O Problema do PNG sRGB

PNGs tradicionais são sRGB. Se você salvar uma cor P3 como PNG sRGB, o navegador CONVERTE para sRGB e você PERDE a cor.

**Formatos que SUPORTAM wide gamut:**
- **AVIF** com perfil ICC P3
- **WebP** com perfil ICC P3
- **JPEG 2000** com perfil ICC P3 (Safari)
- **HEIF/HEIC** (Apple — já é P3 por padrão)

### 5.2 `<canvas>` e P3

```javascript
const canvas = document.createElement('canvas');
const ctx = canvas.getContext('2d', {
  colorSpace: 'display-p3',  // ← A MÁGICA
});

// Agora todas as cores são em P3
ctx.fillStyle = 'color(display-p3 1 0 0)';
ctx.fillRect(0, 0, 100, 100);
```

### 5.3 SVG e P3

SVG suporta P3 via CSS:

```svg
<svg xmlns="http://www.w3.org/2000/svg">
  <rect width="100" height="100">
    <style>
      rect { fill: color(display-p3 1 0 0); }
    </style>
  </rect>
</svg>
```

---

## 6. O Pipeline Wide Gamut Completo

### 6.1 Do Design ao Código

```
1. DESIGN (Figma)
   Figma já está em Display P3 desde 2023.
   Cores que você escolhe no Figma JÁ SÃO P3.
   Problema: exportar como hex code PERDE o P3.

2. EXPORTAÇÃO
   ❌ hex code #FF3B30 — PERDE o P3
   ✅ oklch(0.55 0.35 30) — PRESERVA o P3
   ✅ color(display-p3 1 0.15 0.12) — PRESERVA o P3

3. CÓDIGO
   CSS com oklch() para tudo.
   Navegador faz gamut mapping automático.
   Sem fallbacks manuais para 95% dos casos.

4. IMAGENS
   Exportar como AVIF com perfil Display P3.
   PNG só para fallback sRGB.
```

### 6.2 O que Fazer HOJE (2026)

```css
:root {
  /* Use OKLCH para TODAS as cores da marca */
  /* Chroma > 0.30 automaticamente usa P3 quando disponível */
  --brand-red:    oklch(0.55 0.30 30);    /* P3 em telas Apple, sRGB em telas baratas */
  --brand-orange: oklch(0.60 0.28 60);    /* automático */
  --brand-green:  oklch(0.50 0.32 150);   /* automático */
  --brand-purple: oklch(0.50 0.28 300);   /* automático */

  /* Para cores CRÍTICAS (ex: logo), use fallback explícito */
  --logo-red: rgb(255 59 48);                   /* sRGB garantido */
  --logo-red: color(display-p3 1 0.15 0.12);   /* P3 quando possível */
}
```

---

## 7. O Lado Sombrio: O Que Ainda Não Funciona

### 7.1 Captura de Tela (Screenshot)

Fazer screenshot de uma cor P3 geralmente CONVERTE para sRGB. Você perde a cor. Isso é um problema para:
- Compartilhar designs
- Reportar bugs visuais
- Postar screenshots do app nas redes sociais

### 7.2 Gradientes e P3

Gradientes que interpolam entre sRGB e P3 podem ter artefatos se o espaço de interpolação não for bem escolhido. OKLCH resolve isso automaticamente (interpola em OKLCH, projeta no gamut disponível).

### 7.3 Performance

Cores wide gamut ocupam MAIS bits. Um buffer P3 consome +50% de memória que um buffer sRGB. Para apps com MUITA renderização (games, animações complexas), isso importa.

### 7.4 A Maldição do Hex Code

Hex codes (`#RRGGBB`) são DEFINIDOS como sRGB. Não existe "hex code P3." Isso significa que TODA ferramenta que usa hex code como interface de cor está PRESA no sRGB.

**Ferramentas que já suportam OKLCH/P3:**
- Chrome DevTools (color picker mostra P3)
- Safari Web Inspector
- Figma (internamente P3, exporta OKLCH)
- Tokens Studio
- Coolors, Huemint

**Ferramentas que NÃO suportam (ainda):**
- A MAIORIA dos color pickers de SO
- Design systems legados que usam hex code
- Tailwind (por padrão — mas você pode usar `oklch()` nas configs)
- Bootstrap, Chakra, MUI, Ant Design (melhorando gradualmente)

---

## 8. Para Levar Para Casa

1. **sRGB é uma prisão de 1996.** Cobre 35% das cores que você vê. Seu iPhone cobre 50%. Use isso.

2. **Hex code NÃO pode representar P3.** `#FF3B30` é sRGB, sempre. Para P3, use `oklch()` ou `color(display-p3 ...)`.

3. **OKLCH é o caminho mais simples para wide gamut.** Chroma > 0.30-0.37 automaticamente usa P3. Sem fallbacks, sem `@supports`, sem config.

4. **HDR não são "cores mais vibrantes" — é LUMINÂNCIA.** Branco pode ser 100 nits (SDR) ou 10,000 nits (HDR). São "brancos" diferentes.

5. **`dynamic-range: high` é a media query do HDR.** Use para ajustar cores que "brilham" (destaques, reflexos, neon).

6. **Imagens precisam de AVIF/WebP com perfil P3.** PNG = sRGB para sempre.

7. **Teste com hardware REAL.** Emulação de P3 em tela sRGB não mostra as cores de verdade — só simula o comportamento do CSS.

8. **O gamut mapping do OKLCH é automático.** A mesma cor `oklch(0.55 0.35 30)` fica "o mais vermelha possível" em sRGB, P3, ou Rec.2020. Você escreve uma vez.

---

## Referências

- W3C — *CSS Color Module Level 4* (2023). `color()`, `oklch()`, `color-gamut` media query.
- W3C — *CSS Color Module Level 5* (draft 2025). `dynamic-range`, `color-mix()` com gamut mapping.
- Apple — *Display P3* (2015). O perfil de cor que equipa todos os dispositivos Apple modernos.
- ITU-R — *BT.2020 / BT.2100* (2012/2016). Parâmetros para UHDTV e HDR.
- SMPTE — *ST.2084 (PQ)* (2014). A curva de transferência perceptual do Dolby Vision.
- Björn Ottosson — *OKLAB* (2020). Gamut mapping automático via OKLCH.
- Chris Lilley (W3C) — *Wide Color Gamut on the Web* (2020-2025). Série de artigos técnicos.
- Dean Jackson (Apple) — *Wide Gamut Color in WebKit* (2016-2025). Implementação de referência.
- Lea Verou — *The Future of Color on the Web* (CSS Day, 2024). Palestra seminal sobre wide gamut.
- Greg Benz — *The sRGB Color Prison* (2020). Fotografia e wide gamut.

---

*Fim do Estudo de Cores Loop 07. sRGB é 1996. Display P3 é 2015. Rec.2020 é o futuro. HDR é luminância. OKLCH é a ponte entre todos eles. Pare de escrever hex codes. Comece a escrever cores que EXISTEM. Cron: segue para loop 08.*
