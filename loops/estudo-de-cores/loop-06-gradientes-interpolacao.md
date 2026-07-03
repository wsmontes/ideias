# 🎨 Estudo de Cores Loop 06 — Gradientes e Interpolação: A Arte do Degradê Que Respeita Seus Olhos

> **Data:** 2026-07-02
> **Loop:** 6 de ∞
> **Tema:** Por que gradientes RGB passam pelo cinza (e como OKLCH resolve). A matemática da interpolação perceptual. Mesh gradients e a estética Apple. `color-mix()` em OKLCH. Easing functions para transições de cor. E como Stripe, Vercel e Linear usam gradientes generativos.

---

## 0. O Problema: Todo Gradiente RGB É Feio No Meio

Pegue duas cores vibrantes: azul `#0000FF` e amarelo `#FFFF00`. Faça um gradiente linear em RGB:

```
Azul puro ────→ ??? ────→ Amarelo puro
#0000FF                        #FFFF00
   ↓                            ↓
R=0, G=0, B=255          R=255, G=255, B=0

No meio do caminho (50%):
R = (0+255)/2 = 127
G = (0+255)/2 = 127
B = (255+0)/2 = 127

Resultado: #7F7F7F = CINZA.
```

**Seu degradê azul→amarelo tem uma MANCHA CINZA no meio.** Não é um arco-íris. Não é um pôr-do-sol. É uma transição que passa por uma cor que NÃO EXISTE entre azul e amarelo na natureza.

**Por que isso acontece:** a interpolação RGB é LINEAR NO CUBO RGB. A linha reta entre dois pontos do cubo CORTA o centro (cinza) sempre que as duas cores estão em faces opostas do cubo (ex: azul e amarelo, vermelho e ciano, verde e magenta — todos os pares complementares!).

**A solução:** interpolar em OKLCH. A "linha reta" entre duas cores no espaço cilíndrico do OKLCH segue o CÍRCULO CROMÁTICO. Azul → amarelo passa pelo VERDE (ou pelo magenta, dependendo da direção). NUNCA pelo cinza.

---

## 1. A Matemática da Interpolação

### 1.1 Interpolação RGB (Geométrica, Errada)

```javascript
function lerpRGB(color1, color2, t) {
  return {
    r: color1.r + (color2.r - color1.r) * t,
    g: color1.g + (color2.g - color1.g) * t,
    b: color1.b + (color2.b - color1.b) * t,
  };
}
```

Faz EXATAMENTE o que você pediu. O problema é que o que você PEDIU (linha reta no cubo RGB) não é o que você QUER (transição perceptual suave).

### 1.2 Interpolação HSL (Pior Ainda)

HSL interpola o matiz pelo caminho MAIS CURTO no círculo (menor ângulo). Isso parece bom, MAS:

- Vermelho (H=0°) → Ciano (H=180°): caminho mais curto = 180° passando por amarelo/verde. Ok.
- Vermelho (H=350°) → Vermelho (H=10°): caminho mais curto = 20° (350→360→10). Isso é CORRETO.

Mas o problema NÃO É o matiz. É que HSL interpola lightness e saturation GEOMETRICAMENTE, não perceptual. O gradiente ainda tem "solavancos" de luminosidade percebida.

### 1.3 Interpolação OKLCH (Perceptual, Correta)

```javascript
function lerpOKLCH(color1, color2, t) {
  // Decide a direção do matiz (caminho mais curto ou mais longo)
  let hueDelta = color2.h - color1.h;
  if (Math.abs(hueDelta) > 180) {
    hueDelta = hueDelta > 0 ? hueDelta - 360 : hueDelta + 360;
  }

  return {
    L: color1.L + (color2.L - color1.L) * t,
    C: color1.C + (color2.C - color1.C) * t,
    h: color1.h + hueDelta * t,
  };
}
```

**O resultado:**
- Azul (h=260°) → Amarelo (h=90°): passa por CIANO (h=180°) e VERDE (h=140°). Arco-íris verdadeiro.
- Vermelho (h=20°) → Verde (h=140°): passa pelo AMARELO (h=90°). Pôr-do-sol.
- Todas as cores intermediárias têm a mesma "vibração" perceptual. Sem manchas cinzas.

### 1.4 O Truque do Chroma no Meio

Interpolar LINEARMENTE o chroma funciona, mas pode ser melhor:

```javascript
function lerpOKLCHSmooth(color1, color2, t) {
  const hueDelta = shortestHueDelta(color1.h, color2.h);

  // Chroma com CURVA (pico no meio para evitar "cinza" nos extremos)
  const chromaT = Math.sin(t * Math.PI);  // 0→1→0: máximo no meio
  const C = color1.C + (color2.C - color1.C) * t
          + 0.05 * chromaT;  // bump extra de saturação no meio

  return {
    L: color1.L + (color2.L - color1.L) * t,
    C: Math.min(C, 0.37),  // clamp no máximo sRGB
    h: color1.h + hueDelta * t,
  };
}
```

**Por que o bump de chroma?** Em interpolação linear simples, o MEIO do gradiente pode parecer menos saturado que as pontas (porque as pontas têm chroma "concentrado" em um matiz, enquanto o meio está "espalhado" entre dois matizes). Um bump sutil de chroma no meio compensa essa percepção.

---

## 2. `color-mix()`: Interpolação Nativa no CSS

### 2.1 A Sintaxe

```css
/* MISTURA em OKLCH (perceptual) — USE SEMPRE */
color-mix(in oklch, blue 50%, yellow 50%)

/* MISTURA em sRGB (geométrica) — EVITE */
color-mix(in srgb, blue 50%, yellow)
```

### 2.2 Gradientes com `color-mix()`

```css
/* Gradiente de 3 paradas usando color-mix() */
.gradient-sunset {
  --start: oklch(0.65 0.22 35);   /* laranja */
  --mid: color-mix(in oklch, var(--start) 50%, var(--end));
  --end: oklch(0.55 0.22 290);    /* roxo */

  background: linear-gradient(
    to right,
    var(--start),
    var(--mid),
    var(--end)
  );
}
```

### 2.3 Múltiplas Paradas Com Progressão Perceptual

```css
/* Aurora borealis — 5 paradas em OKLCH */
.gradient-aurora {
  background: linear-gradient(
    in oklch to right,  /* CSS Color Level 5 — futuro próximo */
    oklch(0.50 0.25 160),  /* verde-água */
    oklch(0.55 0.22 190),  /* ciano */
    oklch(0.60 0.18 240),  /* azul */
    oklch(0.55 0.20 280),  /* violeta */
    oklch(0.50 0.22 320)   /* magenta */
  );
}
```

**Nota:** `linear-gradient(in oklch to right, ...)` ainda é CSS Color Level 5 (draft). Em 2026, use a notação tradicional com cores OKLCH — o navegador interpola em OKLCH por padrão se todas as cores forem `oklch()`.

### 2.4 O Padrão de Interpolação do Navegador

```css
/* Força interpolação em OKLCH para TODO o elemento */
.gradient-smooth {
  interpolate-size: oklch;        /* futura propriedade */
  background: linear-gradient(to right, blue, yellow);
  /* O navegador vai interpolar blue→yellow em OKLCH! */
}
```

Enquanto `interpolate-size` não chega, a técnica é usar cores `oklch()` explicitamente — o navegador já interpola cores do mesmo espaço corretamente.

---

## 3. Tipos de Gradiente e Quando Usar

### 3.1 Linear — O Clássico

```css
/* OKLCH garante transição suave */
.hero-gradient {
  background: linear-gradient(
    135deg,
    oklch(0.40 0.20 260) 0%,    /* azul escuro */
    oklch(0.55 0.18 290) 50%,   /* roxo */
    oklch(0.65 0.15 340) 100%   /* rosa */
  );
}
```

**Melhor para:** fundos, headers, seções, sobreposições de imagem.

### 3.2 Radial — Profundidade e Foco

```css
/* Efeito "holofote" com centro claro */
.spotlight {
  background: radial-gradient(
    circle at 30% 20%,
    oklch(0.70 0.05 260) 0%,     /* centro claro */
    oklch(0.40 0.15 260) 60%,    /* médio */
    oklch(0.15 0.10 260) 100%    /* borda escura */
  );
}
```

**Melhor para:** cards com destaque, modais, tooltips, indicadores de foco.

### 3.3 Cônico — Ângulos e Relógios

```css
/* Arco-íris circular perceptual */
.color-wheel {
  background: conic-gradient(
    in oklch,
    oklch(0.55 0.22 0),
    oklch(0.55 0.22 60),
    oklch(0.55 0.22 120),
    oklch(0.55 0.22 180),
    oklch(0.55 0.22 240),
    oklch(0.55 0.22 300),
    oklch(0.55 0.22 360)
  );
}
```

**Melhor para:** color pickers, gráficos de pizza, loaders circulares, mostradores.

### 3.4 Mesh — A Estética Apple

Mesh gradients (gradientes de malha) são a ASSINATURA visual da Apple desde o iOS 11:

```css
/* Mesh gradient: múltiplos pontos de cor com posições 2D */
/* CSS não tem nativo ainda. Soluções: PNG/SVG ou Canvas */
.mesh-apple-style {
  /* Simulação com múltiplos radiais sobrepostos */
  background:
    radial-gradient(circle at 20% 30%, oklch(0.60 0.10 300 / 0.6), transparent 50%),
    radial-gradient(circle at 80% 20%, oklch(0.55 0.15 200 / 0.5), transparent 50%),
    radial-gradient(circle at 50% 80%, oklch(0.65 0.12 30 / 0.4), transparent 50%),
    oklch(0.15 0.02 260);  /* fundo base */
}
```

**Mesh gradients são 3-5 gradientes radiais com posições e opacidades sobrepostas.** Cada "ponto" é uma cor, e as transições entre eles criam a textura rica e "vítrea" que a Apple usa nos wallpapers, Apple Music, e telas de setup.

---

## 4. Easing Functions Para Cor (Não Só Para Movimento)

### 4.1 O Problema: Transições Lineares de Cor São "Duras"

```css
button {
  background: var(--bg);
  transition: background 0.2s linear;  /* linear = mecânico */
}
button:hover {
  background: var(--bg-hover);
}
```

Uma transição LINEAR entre duas cores parece "instantânea" no começo e "arrastada" no final. Isso acontece porque somos MAIS sensíveis a MUDANÇAS de cor (adaptação cromática rápida) do que à cor ESTÁVEL.

### 4.2 Easing Functions para `transition`

```css
/* Ease-out: começa rápido, termina suave — mais "natural" */
button {
  transition: background 0.25s ease-out;
}

/* Custom cubic-bezier para cor */
.card {
  /* overshoot sutil — a cor "passa um pouco" e volta */
  transition: background 0.35s cubic-bezier(0.34, 1.56, 0.64, 1);
}
```

### 4.3 O Easing Perceptual Ideal Para Cor

Pesquisas mostram que o easing perceptual para MUDANÇA DE COR é diferente do easing para MOVIMENTO:

- **Movimento:** ease-out (começa rápido, desacelera) — simula inércia física
- **Cor:** ease-in-out com pico em ~30% (começa devagar, acelera no meio, desacelera) — simula adaptação visual

```css
/* Easing "perceptual" para transições de cor */
:root {
  --ease-color: cubic-bezier(0.4, 0.0, 0.2, 1);   /* Material standard */
  --ease-color-smooth: cubic-bezier(0.25, 0.1, 0.25, 1);  /* linear perceptual */
}
```

### 4.4 Interpolação com Curva de Chroma

Se você está animando de cinza (C=0) para uma cor vibrante (C=0.25):

```javascript
// Easing que PRIORIZA o chroma no começo
function chromaFirstEasing(t) {
  // t=0: cinza. t=0.3: já 80% do chroma. t=0.5-1: ajuste fino.
  if (t < 0.3) return t / 0.3 * 0.8;
  return 0.8 + (t - 0.3) / 0.7 * 0.2;
}
```

A "cor" aparece quase instantaneamente (nos primeiros 30% da transição), e o resto é ajuste fino de lightness. Isso parece mais "responsivo" que uma transição linear.

---

## 5. Gradientes Generativos: A Estética Das Startups

### 5.1 Stripe: O Gradiente Que Virou Marca

O site da Stripe (2022-presente) usa gradientes CÓSMICOS generativos como plano de fundo:

- Gerados proceduralmente (WebGL/Canvas, não imagens)
- Paleta: azul escuro → violeta → ciano → azul claro
- Animados lentamente (velocidade ~0.1px/s, imperceptível)
- OKLCH internamente para transições suaves

```javascript
// Simplified Stripe-style generative gradient
function stripeGradient(ctx, width, height, time) {
  // Múltiplos centros radiais com ruído
  const points = [
    { x: 0.3, y: 0.4, h: 260, phase: 0 },      // azul
    { x: 0.7, y: 0.6, h: 200, phase: 1.5 },     // ciano
    { x: 0.5, y: 0.3, h: 290, phase: 3.0 },     // violeta
  ];

  for (const p of points) {
    const grad = ctx.createRadialGradient(
      p.x * width, p.y * height, 0,
      p.x * width + Math.sin(time + p.phase) * 100,
      p.y * height + Math.cos(time + p.phase) * 100,
      width * 0.7
    );
    grad.addColorStop(0, oklch(0.55, 0.20, p.h, 0.8));
    grad.addColorStop(1, oklch(0.10, 0.05, p.h, 0));
    ctx.fillStyle = grad;
    ctx.fillRect(0, 0, width, height);
  }
}
```

### 5.2 Vercel: Geist e o Gradiente "Sutil"

Vercel usa gradientes MINIMALISTAS — quase imperceptíveis:

```css
.geist-subtle-gradient {
  background:
    linear-gradient(180deg, oklch(0.12 0.005 260) 0%, oklch(0.10 0 0) 100%);
  /* Apenas 2% de diferença de lightness. Quase imperceptível. Luxuoso. */
}
```

### 5.3 Linear: Gradiente Como Produto

O app Linear usa gradientes FUNCIONAIS:

- Barras de progresso com gradientes triádicos
- Indicadores de prioridade com escala perceptual
- Estados de issue: azul→roxo para backlog, verde→azul para done

```css
.linear-progress {
  background: linear-gradient(
    to right,
    oklch(0.55 0.22 260),  /* azul (todo) */
    oklch(0.55 0.22 160),  /* verde (in progress) */
    oklch(0.55 0.22 290)   /* roxo (done) */
  );
}
```

---

## 6. Acessibilidade de Gradientes

### 6.1 Texto Sobre Gradiente É Pesadelo

O problema: o contraste do texto MUDA ao longo do gradiente. O que é legível sobre a parte clara é ilegível sobre a parte escura.

**Soluções:**

1. **Overlay sólido sobre gradiente:**
```css
.gradient-bg {
  background: linear-gradient(to right, oklch(0.40 0.20 260), oklch(0.80 0.10 60));
}
.text-overlay {
  background: oklch(0 0 0 / 0.35);  /* overlay escuro uniforme */
  color: oklch(1 0 0);
}
```

2. **Gradiente de texto (text clipping):**
```css
.gradient-text {
  background: linear-gradient(to right, oklch(0.50 0.25 30), oklch(0.55 0.25 290));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  /* Apenas para DISPLAY. Não para corpo de texto. */
}
```

3. **`text-shadow` para garantir legibilidade:**
```css
.text-on-gradient {
  color: oklch(1 0 0);
  text-shadow: 0 1px 4px oklch(0 0 0 / 0.5);  /* contorno escuro */
}
```

### 6.2 O Teste do Gradiente Acessível

Para validar que seu gradiente não mata a legibilidade:

1. Amostre 10 pontos ao longo do gradiente
2. Calcule APCA do texto contra cada ponto
3. Todos os 10 devem ter Lc > 60

Se algum ponto falhar, você precisa de overlay ou ajuste de cores.

---

## 7. A Tabela de Decisão: Qual Interpolação Usar

| Situação | Espaço | Motivo |
|---|---|---|
| Gradiente decorativo (fundo) | OKLCH | Transição perceptual suave |
| Gradiente de dados (heatmap) | OKLCH ou CIELAB | Passos perceptualmente iguais |
| Animação de cor (hover) | OKLCH | Mudança "natural" |
| Fallback navegador antigo | sRGB (com OKLCH como progressive enhancement) | Compatibilidade |
| Duas cores MUITO próximas | sRGB (OKLCH não adiciona valor) | Performance |
| Cor → cinza (dessaturação) | OKLCH com redução linear de C | Cinza perceptual verdadeiro |
| Múltiplas paradas (>4) | OKLCH sempre | Evita "nós" de cinza |
| Impressão (CMYK) | LAB (OKLAB não suportado em fluxos CMYK) | Padrão da indústria gráfica |

---

## 8. Para Levar Para Casa

1. **RGB gera cinza no meio do degradê.** A interpolação corta o centro do cubo. OKLCH segue o círculo cromático.

2. **`color-mix(in oklch, ...)` é a ferramenta certa.** `in srgb` é a errada. A diferença é brutal.

3. **Mesh gradients são múltiplos radiais sobrepostos.** A "assinatura Apple" é replicável com 3-5 `radial-gradient()` com opacidade.

4. **Transições de cor precisam de easing diferente de movimento.** Ease-in-out com pico em 30%. A cor aparece rápido e se ajusta devagar.

5. **Bump de chroma no meio do gradiente** compensa a "dispersão" perceptual entre dois matizes diferentes.

6. **Texto sobre gradiente é perigoso.** Overlay semitransparente ou valide APCA em 10 pontos.

7. **OKLCH é o padrão ouro para interpolação.** CSS já interpola cores `oklch()` corretamente. Use `oklch()` explícito em todos os color stops.

---

## Referências

- Björn Ottosson — *OKLAB* (2020). O espaço que torna a interpolação perceptual possível.
- W3C — *CSS Color Module Level 4* (2023). `color-mix()`, interpolação em diferentes espaços.
- W3C — *CSS Images Module Level 4* (2024). Gradientes cônicos, `in oklch` para `conic-gradient`.
- Apple — *Human Interface Guidelines — Color* (2019-2026). Filosofia de mesh gradients.
- Stripe — *Design Engineering* (2020-2025). Gradientes generativos.
- Vercel — *Geist Design System* (2024-2025). Gradientes minimalistas.
- Rune Madsen — *Programming Design Systems* (2017). Capítulo sobre interpolação de cor.
- Mike Bostock — *D3.js color interpolation* (2011-2025). Implementação de referência.
- Lea Verou — *CSS Secrets* (2015). Gradientes, text clipping, técnicas avançadas.

---

*Fim do Estudo de Cores Loop 06. Gradiente não é "duas cores com uma linha entre elas." É uma JORNADA perceptual pelo espaço cromático. Em RGB, essa jornada passa pelo deserto cinza. Em OKLCH, ela segue a costa do arco-íris. Escolha seu caminho com intenção. Cron: segue para loop 07.*
