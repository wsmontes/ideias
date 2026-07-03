# 🎨 Estudo de Cores Loop 01 — Como Seu Computador Mente Sobre Cor (E Por Que Isso Importa)

> **Data:** 2026-07-02
> **Loop:** 1 de ∞
> **Tema:** Espaços de cor — RGB, HSL, LAB, OKLCH. Tudo que você sabe sobre cor digital está errado (ou pelo menos incompleto). A cor que você "vê" no Figma não é a cor que seu usuário vê no dispositivo. O HSL que você usa há 10 anos é uma abstração QUEBRADA. E existe um espaço de cor de 2020 que resolve tudo — e você provavelmente não está usando.

---

## 0. O Problema: Cor Não Existe

Cor não é uma propriedade física da luz. Cor é uma **PERCEPÇÃO** — uma construção do seu cérebro a partir de 3 tipos de cones na retina (L, M, S — long, medium, short wavelength).

Isso significa que **não existe "cor real."** Existe radiação eletromagnética (~380-750nm) que seu cérebro INTERPRETA como cor. Dois espectros diferentes podem parecer a MESMA cor (metamerismo). E a mesma cor parece diferente em fundos diferentes (contraste simultâneo — Albers, 1963).

**Implicação para apps:** a cor que você escolhe no Figma NÃO EXISTE "lá fora." Ela é um sinal digital (R, G, B) que cada dispositivo interpreta de um jeito. O mesmo hex code `#8B5CF6` parece diferente em:
- Um iPhone 16 Pro (OLED, Display P3, True Tone ON)
- Um monitor Dell barato (LCD, sRGB 65%, fábrica)
- Um Galaxy S24 (AMOLED, Vivid mode)
- Uma TV LG (OLED, Dolby Vision)

**A única verdade:** cor é PERCEPÇÃO. E a boa notícia é que existe uma ciência inteira (colorimetria) que MEDE a percepção humana.

---

## 1. RGB: O Básico Quebrado Com Qual Todo Mundo Começa

### 1.1 O Modelo

RGB representa cor como 3 números: Red (0-255), Green (0-255), Blue (0-255). É o que todo programador aprende. É como monitores funcionam (subpixels vermelhos, verdes e azuis).

```
#8B5CF6 = rgb(139, 92, 246) → um roxo/violeta
```

### 1.2 O Problema: RGB Não É Um Espaço De Cor

"RGB" sozinho não significa NADA. Existem DEZENAS de espaços RGB diferentes:

| Espaço RGB | Gamut | Uso |
|---|---|---|
| **sRGB** | ~35% das cores visíveis | Web, apps, CSS, padrão desde 1996 |
| **Display P3** | ~50% das cores visíveis | Apple (iPhone, Mac, iPad desde ~2015) |
| **Adobe RGB** | ~50% (diferente do P3) | Fotografia, impressão |
| **Rec.2020** | ~75% das cores visíveis | HDR, futuro |
| **ProPhoto RGB** | ~90% (inclui cores IMAGINÁRIAS) | Edição profissional |

**O `#8B5CF6` que você escreve no CSS é `#8B5CF6` EM QUAL DESTES ESPAÇOS?** A resposta: sRGB. Sempre sRGB na web (por enquanto). Mas seu monitor Apple está renderizando em Display P3. O navegador está fazendo uma conversão (nem sempre correta).

### 1.3 A Matemática Do Desastre

RGB é baseado na FÍSICA do fósforo do CRT, não na PERCEPÇÃO humana. As curvas de resposta dos cones L, M, S da sua retina NÃO são combinações lineares de R, G, B. Isso significa que:

- **Distância igual em RGB ≠ percepção igual de diferença.** `rgb(255,0,0)` → `rgb(255,10,0)` é quase imperceptível (variação de 10 no verde no vermelho puro). `rgb(0,255,0)` → `rgb(0,245,0)` é MUITO perceptível (mesma variação de 10 no verde).

- **Interpolação em RGB é FEIA.** Fazer um gradiente de azul para amarelo em RGB passa por... cinza. Porque a "linha reta" entre `#0000FF` e `#FFFF00` no cubo RGB passa pelo centro (cinza), não pelo arco-íris. Visualmente: uma mancha cinza no meio de um degradê horrível.

```
RGB gradient:  azul → cinza → amarelo  🤮 (porque a linha passa pelo centro do cubo)
OKLCH gradient: azul → verde → amarelo 😍 (porque segue o círculo cromático perceptualmente)
```

---

## 2. HSL/HSV: A "Solução" Que Criou Mais Problemas

### 2.1 O Que É

HSL (Hue, Saturation, Lightness) e HSV (Hue, Saturation, Value) foram criados nos anos 1970 para dar uma interface "humana" ao RGB. Em vez de 3 números arbitrários, você tem:

- **Hue (matiz):** ângulo 0-360° (0=vermelho, 120=verde, 240=azul)
- **Saturation:** 0-100% (0=cinza, 100=cor pura)
- **Lightness/Value:** 0-100% (0=preto, 100=branco)

É o color picker de TODO software: Figma, Photoshop, Chrome DevTools, macOS Color Picker.

### 2.2 Por Que HSL É UMA MENTIRA

**Problema #1: "Lightness" não é luminosidade PERCEBIDA.**

Abra qualquer color picker HSL. Coloque L=50% (supostamente "meio brilho"). Agora gire o Hue:

- **Amarelo (H=60°, S=100%, L=50%):** BRILHANTE. Quase branco amarelado. MUITO claro.
- **Azul (H=240°, S=100%, L=50%):** ESCURO. Azul marinho. MUITO escuro.

**"50% de Lightness" deveria significar "50% do brilho percebido." Mas amarelo a 50% é MUITO mais claro que azul a 50%.** O HSL MENTE.

O motivo? HSL calcula Lightness como `(max(R,G,B) + min(R,G,B)) / 2`. Isso é puramente geométrico. Mas seus olhos NÃO dão o mesmo peso para R, G e B:

- **Verde** contribui ~71.5% para a luminosidade percebida
- **Vermelho** contribui ~21.3%
- **Azul** contribui apenas ~7.2%

O azul é quase INVISÍVEL para o cálculo de brilho do HSL. Por isso azul em L=50% parece tão escuro.

```
Luminância REAL (Y) = 0.2126 × R + 0.7152 × G + 0.0722 × B
Lightness do HSL   = (max(R,G,B) + min(R,G,B)) / 2
```

**Estas duas fórmulas dão resultados RADICALMENTE diferentes.**

**Problema #2: Saturation também é uma mentira.**

HSL S=100%, L=50%:
- Amarelo (H=60°): parece uma cor "pura", vibrante ✓
- Azul (H=240°): parece ESCURO e dessaturado ✗

Mas a saturation está em 100% nos DOIS casos. Para o azul parecer tão vibrante quanto o amarelo, você precisaria aumentar o Lightness para ~65-70%.

**Problema #3: Você não pode fazer matemática de cor em HSL.**

Quer uma paleta onde cada cor tem a MESMA luminosidade? Em HSL, se você fixar L=60% e variar H, as cores vão ter luminosidades percebidas COMPLETAMENTE DIFERENTES. Amarelo vai pular da tela. Azul vai sumir no fundo.

Quer "escurecer 10%"? `L = L - 10` em HSL não é uma redução uniforme. Algumas cores mudam muito, outras quase nada.

---

## 3. CIE 1931: O Nascimento da Colorimetria Científica

### 3.1 O Experimento

Em 1931, a CIE (Commission Internationale de l'Éclairage) fez um experimento monumental:

- ~20 observadores humanos
- Tela dividida: de um lado uma cor de teste (luz monocromática em um comprimento de onda específico), do outro 3 luzes primárias (R=700nm, G=546.1nm, B=435.8nm)
- O observador ajustava a INTENSIDADE das 3 primárias até a mistura parecer IDÊNTICA à cor de teste
- Repetido para TODOS os comprimentos de onda visíveis (380-780nm)

O resultado: as **CIE 1931 Color Matching Functions** — 3 curvas que descrevem quanto de cada primária é necessário para igualar cada comprimento de onda.

### 3.2 O Diagrama de Cromaticidade (A Ferradura)

As curvas foram transformadas matematicamente no diagrama xy (a "ferradura" que todo mundo já viu). Cada ponto (x, y) representa uma CROMATICIDADE (matiz + saturação), ignorando luminosidade.

```
       y
       ↑
  0.8  │    ┌─────────────┐
       │   ╱  verde        ╲
  0.6  │  ╱                 ╲
       │ ╱    ▫ branco       ╲
  0.4  │╱   (0.33, 0.33)      ╲
       │                        ╲
  0.2  │ azul           vermelho╲
       │                         ╲
  0.0  └────────────────────────────→ x
       0.0   0.2   0.4   0.6   0.8
```

**A ferradura representa TODAS as cores que um ser humano médio pode ver.** É o "alfabeto" completo da visão humana.

**Comparação de gamuts dentro da ferradura:**

| Espaço | Cobertura da ferradura |
|---|---|
| sRGB | ~35% |
| Display P3 | ~50% |
| Adobe RGB | ~52% (diferente do P3) |
| Rec.2020 | ~75% |
| ACEScg | ~85% |

**O que isso significa para apps:** o sRGB — o PADRÃO da web — cobre apenas 1/3 das cores que você CONSEGUE VER. Todas as outras cores que existem na natureza (e que seu iPhone pode mostrar) são INVISÍVEIS no CSS tradicional.

### 3.3 CIE XYZ: O Espaço Base

CIE XYZ é o "esperanto" da cor — um espaço matemático que pode representar TODAS as cores visíveis. Toda conversão de cor passa por XYZ:

```
sRGB → linear RGB → XYZ → LAB/OKLAB → ...
```

XYZ é o "latim da colorimetria" — ninguém trabalha diretamente nele, mas tudo passa por ele.

---

## 4. CIELAB (1976): A Primeira Tentativa de Uniformidade Perceptual

### 4.1 A Estrutura

CIELAB (também conhecido como CIE L\*a\*b\*) foi projetado para ser PERCEPTUALMENTE UNIFORME:

- **L\*** (Lightness): 0=preto a 100=branco. PERCEPTUALMENTE LINEAR. L\*=50 É o "cinza médio" que seus olhos veem.
- **a\***: eixo verde (-) → vermelho (+). Valores típicos: -128 a +128.
- **b\***: eixo azul (-) → amarelo (+). Valores típicos: -128 a +128.

É baseado na **teoria do processo oponente** da visão humana (Hering, 1892): percebemos cor em pares opostos: claro/escuro, vermelho/verde, azul/amarelo.

### 4.2 Delta E: Medindo "Quão Diferente"

Delta E (ΔE) é a distância euclidiana entre duas cores no espaço CIELAB. É a MÉTRICA de diferença perceptual:

| ΔE | Percepção |
|---|---|
| < 1 | Imperceptível (até para especialistas) |
| 1-2 | Perceptível com atenção |
| 2-3 | Perceptível |
| 3-6 | Claramente diferente |
| 6-10 | Muito diferente |
| > 10 | "Outra cor" |

**Isto é REVOLUCIONÁRIO para design systems:** você pode MEDIR se duas cores são "iguais o suficiente." Pode testar se sua paleta tem "saltos" desiguais entre tons.

**Versões do Delta E:**
- **ΔE\*ab (1976):** o básico. Simples mas imperfeito (cores muito saturadas têm ΔE inflado).
- **ΔE\*94:** correção para saturação. melhor.
- **ΔE\*00 (2000):** o padrão atual. Corrige matiz, chroma, lightness com pesos diferentes. O MELHOR.

### 4.3 CIELCH: A Versão "Humana" do LAB

CIELCH é CIELAB em coordenadas polares:
- **L\***: o mesmo Lightness
- **C\*** (Chroma): distância do centro (0=cinza, 100+=cores intensas)
- **h°** (Hue angle): ângulo no círculo cromático (0°=vermelho, 90°=amarelo, 180°=verde, 270°=azul)

CIELCH é para LAB o que HSL é para RGB — a versão "humana." Mas BASEADA EM PERCEPÇÃO REAL, não em geometria de cubo arbitrária.

### 4.4 O Problema do CIELAB

CIELAB foi revolucionário em 1976, mas tem defeitos:

1. **Mudança de matiz (hue shift):** quando você altera o Lightness de uma cor azul em CIELAB, o matiz MUDA. Azul escuro → azul claro em LAB vira... azul arroxeado. Isso NÃO acontece na percepção humana.

2. **A "curvatura" do azul:** cores azuis saturadas em LAB têm um problema conhecido — o espaço "curva" o azul em direção ao roxo quando você muda o lightness.

Por décadas, isso foi aceito como "bom o suficiente." Até 2020.

---

## 5. OKLAB / OKLCH (2020): O Futuro Chegou

### 5.1 O Que É

Em 2020, **Björn Ottosson** (engenheiro da Ubisoft Massive) publicou OKLAB — um novo espaço de cor perceptual criado para resolver os problemas do CIELAB usando otimização numérica moderna.

OKLAB/OKLCH têm as mesmas coordenadas que LAB/LCH:
- **OKLAB:** L, a, b (cartesiano)
- **OKLCH:** L, C, h (polar) — esta é a versão que você vai usar

### 5.2 Por Que OKLCH É Superior

**1. Sem mudança de matiz.** Mude o Lightness de um azul em OKLCH: o matiz PERMANECE azul. Sem desvio para roxo. A cor só fica mais clara ou mais escura, como seus olhos esperam.

**2. Lightness verdadeiramente uniforme.** L=0.6 em OKLCH significa "60% do caminho perceptual entre preto e branco" para QUALQUER matiz. Amarelo, azul, vermelho — todos na mesma luminosidade percebida.

**3. Chroma com significado real.** C=0.1 em OKLCH é o MESMO "nível de saturação percebida" para todas as cores. Diferente do HSL onde S=100% significa coisas completamente diferentes.

**4. Interpolação perceptual.** Faça um gradiente entre duas cores em OKLCH e o resultado é SUAVE e PERCEPTUALMENTE UNIFORME. Sem "mancha cinza" no meio.

### 5.3 OKLCH no CSS (Já Disponível!)

CSS Color Level 4 trouxe `oklch()` para TODOS os navegadores modernos:

```css
/* OKLCH: lightness 0-1 (ou 0%-100%), chroma 0-~0.4, hue 0-360 */
--primary: oklch(0.6 0.2 270);     /* azul com 60% lightness */
--danger:  oklch(0.55 0.25 25);     /* vermelho com 55% lightness */
--success: oklch(0.6 0.2 145);      /* verde */

/* Note: TODAS têm lightness similar e vão parecer IGUALMENTE claras */
```

**Suporte dos navegadores (2026):**
- Safari 15.4+ (março 2022) — foi o PRIMEIRO
- Chrome/Edge 111+ (março 2023)
- Firefox 113+ (maio 2023)
- **Cobertura global: ~94%**

Para os 6% restantes, você usa fallback:

```css
--primary-rgb: 59 130 246;  /* fallback sRGB */
--primary: rgb(59 130 246);
--primary: oklch(0.62 0.22 260);  /* sobrescreve em navegadores modernos */
```

### 5.4 A Anatomia de uma Cor OKLCH

```
oklch(L C h)

L (Lightness): 0% a 100% (ou 0 a 1)
  - 0%: preto absoluto
  - 50%: cinza médio PERCEPTUAL (diferente do HSL!)
  - 100%: branco absoluto

C (Chroma): 0 a ~0.4 (teórico: 0 a ∞)
  - 0: cinza (sem cor)
  - 0.05: quase cinza (sutil)
  - 0.15: cor "normal" (vibrante mas não extrema)
  - 0.25: MUITO vibrante
  - 0.37: máximo do sRGB para a maioria dos matizes
  - 0.4+: Display P3 / além do sRGB

h (Hue): 0 a 360 (ângulo)
  - 0°: vermelho
  - 90°: amarelo
  - 180°: verde
  - 270°: azul
  - 360°: vermelho de novo
```

### 5.5 Exemplos Práticos de Paleta em OKLCH

**Paleta de marca uniforme (todas as cores com a MESMA luminosidade):**

```css
--brand-blue:    oklch(0.55 0.22 260);
--brand-purple:  oklch(0.55 0.22 290);
--brand-red:     oklch(0.55 0.22 20);
--brand-orange:  oklch(0.55 0.22 55);
--brand-green:   oklch(0.55 0.22 150);
```

**Olhe para essas cores. Elas têm o MESMO "peso visual."** Nenhuma salta mais que a outra. Isso é IMPOSSÍVEL de fazer em HSL.

**Escala de cinza perceptual (também OKLCH):**

```css
--gray-50:  oklch(0.98 0 0);  /* quase branco */
--gray-100: oklch(0.90 0 0);
--gray-200: oklch(0.80 0 0);
--gray-300: oklch(0.70 0 0);
--gray-400: oklch(0.60 0 0);
--gray-500: oklch(0.50 0 0);  /* cinza médio PERCEPTUAL */
--gray-600: oklch(0.40 0 0);
--gray-700: oklch(0.30 0 0);
--gray-800: oklch(0.20 0 0);
--gray-900: oklch(0.12 0 0);  /* quase preto */
```

Cada passo é PERCEPTUALMENTE IGUAL. Um salto de gray-400 para gray-500 parece o mesmo que de gray-700 para gray-800. Em RGB/HSL isso NÃO acontece.

---

## 6. CSS Color Level 4: O Arsenal Completo

OKLCH é só o começo. CSS Color Level 4 (e o emergente Level 5) trouxe funções que TRANSFORMAM como trabalhamos com cor em apps:

### 6.1 `color-mix()` — Mistura Perceptual

```css
/* Mistura em OKLCH (perceptual): 50% azul + 50% branco */
--blue-400: color-mix(in oklch, var(--blue-600) 50%, white);

/* Mistura em sRGB (geométrica): EVITE! Vai passar pelo cinza */
--bad-mix: color-mix(in srgb, blue 50%, yellow);

/* Gerar escala de cores automaticamente */
--blue-100: color-mix(in oklch, var(--blue-600) 20%, white);
--blue-200: color-mix(in oklch, var(--blue-600) 40%, white);
--blue-300: color-mix(in oklch, var(--blue-600) 60%, white);
--blue-400: color-mix(in oklch, var(--blue-600) 80%, white);
```

### 6.2 `light-dark()` — Dark Mode sem Media Query

```css
:root {
  color-scheme: light dark;
  --bg: light-dark(oklch(0.98 0 0), oklch(0.15 0 0));
  --text: light-dark(oklch(0.15 0 0), oklch(0.90 0 0));
  --surface: light-dark(oklch(1 0 0), oklch(0.20 0 0));
}
```

### 6.3 Relative Color Syntax — Manipulação Direta

```css
/* "A mesma cor, mas 20% mais escura" */
--primary-dark: oklch(from var(--primary) calc(l - 0.2) c h);

/* "Mesma cor, menos saturada" */
--primary-muted: oklch(from var(--primary) l calc(c * 0.6) h);

/* "Cor complementar" (hue oposto) */
--primary-complement: oklch(from var(--primary) l c calc(h + 180));
```

**Isso é REVOLUCIONÁRIO.** Antes você precisava de pré-processadores (Sass `darken()`) ou JavaScript. Agora é CSS NATIVO, com matemática PERCEPTUALMENTE CORRETA.

### 6.4 `color()` — Wide Gamut (Display P3+)

```css
/* Display P3 — cores que sRGB NÃO alcança */
--vivid-pink: color(display-p3 1 0 0.5);
--deep-green: color(display-p3 0 0.8 0.3);

/* Fallback automático */
--accent: oklch(0.6 0.25 350);  /* navegador converte para o melhor disponível */
```

---

## 7. O Que Isso Significa Para Seu App

### 7.1 Design Tokens Baseados em Percepção

**ANTES (HSL — quebrado):**
```json
{
  "blue-500": { "value": "hsl(240 100% 50%)" },
  "yellow-500": { "value": "hsl(60 100% 50%)" }
}
```
❌ Mesmo L=50%, amarelo parece MAIS CLARO que azul.

**DEPOIS (OKLCH — perceptual):**
```json
{
  "blue-500": { "value": "oklch(0.55 0.22 260)" },
  "yellow-500": { "value": "oklch(0.55 0.22 90)" }
}
```
✅ Mesmo L=0.55, MESMA luminosidade percebida.

### 7.2 Geração de Paletas Automática

Com OKLCH, você pode GERAR uma paleta inteira a partir de UMA cor:

```javascript
function generatePalette(hue, chroma) {
  return {
    50:  `oklch(0.95 ${chroma * 0.3} ${hue})`,
    100: `oklch(0.88 ${chroma * 0.5} ${hue})`,
    200: `oklch(0.78 ${chroma * 0.7} ${hue})`,
    300: `oklch(0.68 ${chroma * 0.9} ${hue})`,
    400: `oklch(0.60 ${chroma * 1.0} ${hue})`,
    500: `oklch(0.52 ${chroma * 1.0} ${hue})`,  // cor base
    600: `oklch(0.44 ${chroma * 0.9} ${hue})`,
    700: `oklch(0.36 ${chroma * 0.8} ${hue})`,
    800: `oklch(0.28 ${chroma * 0.6} ${hue})`,
    900: `oklch(0.20 ${chroma * 0.4} ${hue})`,
  };
}
```

Isso é basicamente o que Tailwind, Radix Colors, e Material You fazem — mas com matemática perceptual.

### 7.3 Dark Mode Previsível

A diferença entre light e dark mode em OKLCH é principalmente Lightness:

```
Light mode:  bg=oklch(0.98 0 0),  text=oklch(0.15 0 0)
Dark mode:   bg=oklch(0.12 0 0),  text=oklch(0.90 0 0)
```

Mas você também precisa reduzir o Chroma (saturação) no dark mode — cores saturadas "vibram" em fundos escuros:

```
Light mode:  accent=oklch(0.55 0.22 260)
Dark mode:   accent=oklch(0.65 0.18 260)  // mais claro, menos saturado
```

### 7.4 Acessibilidade com Precisão Matemática

Com OKLCH você pode CALCULAR contraste, não adivinhar:

```javascript
// APCA (Advanced Perceptual Contrast Algorithm) é superior ao WCAG
// e funciona com OKLCH nativamente
function apcaContrast(textL, bgL) {
  // Usa lightness perceptual (OKLCH L) em vez de luminância relativa
  // Mais preciso que WCAG 2.x para cores escuras e saturadas
}
```

---

## 8. A Linhagem Completa: De Newton ao Seu CSS

```
Newton (1666) — decomposição da luz branca em espectro com prisma
  └─ "Cor é propriedade da luz"
    └─ Young (1802) — teoria tricromática (3 tipos de receptores)
      └─ Helmholtz (1850) — formalização matemática
        └─ Hering (1892) — teoria do processo oponente (claro/escuro, red/green, blue/yellow)
          └─ Schrödinger (1920) — geometria Riemanniana do espaço de cor
            └─ CIE 1931 — primeiro espaço de cor baseado em experimentos humanos
              └─ MacAdam (1942) — elipses de discriminabilidade
                └─ CIELAB (1976) — primeiro espaço perceptual prático
                  └─ CIECAM02 (2002) — modelo de aparência de cor (adaptação cromática)
                    └─ CAM16 (2016) — simplificação do CIECAM02
                      └─ OKLAB (2020) — espaço perceptual por otimização numérica
                        └─ CSS Color Level 4 (2023) — okclh(), color-mix(), light-dark()
                          └─ CSS Color Level 5 (2026+) — color contrast(), color adjustment
```

---

## 9. O Que Ainda Falta Neste Estudo (Próximos Loops)

- **Loop 02:** Delta E, APCA, e a matemática da acessibilidade
- **Loop 03:** Dark Mode — a ciência de escurecer sem perder a alma da cor
- **Loop 04:** Geração algorítmica de paletas (Material You, Leonardo, Huemint)
- **Loop 05:** Design tokens e sistemas de cor em escala
- **Loop 06:** Gradientes, interpolação, e a arte do degradê perceptual
- **Loop 07:** Wide gamut — Display P3, HDR, e o futuro além do sRGB
- **Loop 08:** Brand color systems — estudos de caso (Stripe, Linear, Vercel, Apple)
- **Loop 09:** Cor emocional — arousal/valence, psicofísica da resposta à cor
- **Loop 10:** Cor global — como língua e cultura moldam a percepção de cor

---

## Referências

- Björn Ottosson — *A perceptual color space for image processing* (2020). O artigo original do OKLAB.
- CIE 015:2018 — *Colorimetry, 4th Edition*. A bíblia da colorimetria.
- CSS Color Module Level 4 — W3C Recommendation (2023). `oklch()`, `color-mix()`, `light-dark()`.
- CSS Color Module Level 5 — W3C Working Draft (2025). `contrast-color()`, relative color syntax.
- Lea Verou & Chris Lilley — *Color in CSS: OKLCH and the Future of Web Color* (2023).
- Bruce Lindbloom — *RGB/XYZ Matrices*. A referência definitiva para conversões.
- Charles Poynton — *A Technical Introduction to Digital Video* (1996). sRGB, gamma, espaços RGB.
- Mark Fairchild — *Color Appearance Models* (2013). CIECAM02, CAM16, modelos de aparência.
- David Briggs — *The Dimensions of Colour*. O melhor recurso online sobre teoria da cor.

---

*Fim do Estudo de Cores Loop 01. Espaços de cor: RGB é morto. HSL é mentira. OKLCH é o presente. ΔE é a verdade. Cron: segue para loop 02.*
