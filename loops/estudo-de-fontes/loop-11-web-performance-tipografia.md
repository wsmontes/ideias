# 🔤 Estudo de Fontes Loop 11 — Web Performance & Tipografia: Como Carregar Fontes Sem Destruir Seus Core Web Vitals

> **Data:** 2026-07-02
> **Loop:** 11 de ∞
> **Tema:** Fontes na web são o MAIOR vilão de performance depois de imagens. Uma fonte mal carregada causa texto INVISÍVEL por segundos (FOIT), layout shift (CLS), e LCP penalizado. Mas fontes BEM carregadas são invisíveis — o usuário NUNCA nota. Este loop te dá TODAS as técnicas para chegar lá.

---

## 0. O Problema: Fontes São ~3% do Tráfego Web... e 30% dos Problemas de Performance

Fontes web são tipicamente 50-300 KB. Parece pouco. Mas:

- Elas são **bloqueantes de renderização** (o browser espera a fonte para mostrar texto)
- Elas causam **layout shift** (CLS) quando a fonte carrega e as métricas mudam
- Elas demoram (DNS + TCP + TLS + download) — especialmente em 3G/4G
- Elas são frequentemente **over-fetched** (você carrega 600 caracteres para usar 80)

O resultado: **FOIT** (texto invisível), **FOUT** (texto piscando), **CLS** (página "dançando"), e **LCP** (usuário esperando pra LER).

**A boa notícia:** 90% desses problemas se resolvem com 5 técnicas. Vamos a elas.

---

## 1. `font-display`: A Primeira Linha de Defesa

### 1.1 O Que É

`font-display` é uma propriedade CSS que controla o que o browser faz ENQUANTO a fonte carrega.

```css
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter.woff2') format('woff2');
  font-display: swap;
}
```

### 1.2 Os 5 Valores Explicados

| Valor | Bloqueio Inicial | Swap Period | Comportamento | Use Para |
|---|---|---|---|---|
| **`swap`** | 0s (mostra fallback IMEDIATAMENTE) | Infinito | Texto SEMPRE visível. Troca quando a fonte carregar. | **Body text, conteúdo crítico.** |
| **`block`** | ~3s | Infinito | Texto INVISÍVEL por até 3s aguardando a fonte. Depois fallback, depois swap. | Icon fonts (ícone errado é pior que ícone invisível) |
| **`fallback`** | ~100ms | ~3s | Quase imperceptível. Se a fonte não carregar rápido, usa fallback. | Headlines de marca |
| **`optional`** | ~100ms | 0s (NUNCA faz swap depois do block) | A fonte SÓ é usada se já estiver em cache. Performance MÁXIMA. | Fontes decorativas, não-essenciais |
| **`auto`** | Browser decide (~3s FOIT) | Variável | Imprevisível — NÃO USE. | — |

### 1.3 A Regra de Ouro

```css
/* Texto que o usuário precisa LER → visível IMEDIATAMENTE */
body { font-display: swap; }

/* Ícones → invisível é melhor que ícone errado */
.icon-font { font-display: block; }

/* Fontes decorativas → só use se estiver em cache */
.decorative { font-display: optional; }
```

### 1.4 Google Fonts

Adicione `&display=swap` à URL:

```
https://fonts.googleapis.com/css2?family=Inter&display=swap
```

---

## 2. FOIT vs FOUT: A Guerra da Percepção

| Fenômeno | O Que Acontece | Impacto |
|---|---|---|
| **FOIT** (Flash of Invisible Text) | Texto INVISÍVEL até a fonte carregar | LCP PÉSSIMO. Usuário vê uma página em BRANCO. |
| **FOUT** (Flash of Unstyled Text) | Fallback visível → swap para fonte customizada | LCP OK. Usuário LÊ imediatamente. Pode causar CLS. |

**Consenso 2025: FOUT é melhor que FOIT.** O usuário pode LER o conteúdo imediatamente. O "flash" de troca de fonte é um problema ESTÉTICO. O texto invisível é um problema de USABILIDADE.

**A única exceção:** ícones. Um ícone "errado" (fallback) é pior que um ícone invisível (que vira visível com o glyph correto). Para icon fonts, use `font-display: block`.

---

## 3. Preload: Diga ao Browser "Esta Fonte É IMPORTANTE"

### 3.1 A Sintaxe

```html
<link rel="preload"
      href="/fonts/inter-regular.woff2"
      as="font"
      type="font/woff2"
      crossorigin="anonymous">
```

**`crossorigin` é OBRIGATÓRIO** — mesmo para fontes do mesmo domínio. Sem ele, o preload é ignorado.

### 3.2 Regras do Preload

- **Preload SÓ 1-2 fontes críticas.** Pré-carregar 5 fontes COMPETE com outros recursos (CSS, JS, imagens).
- **Preload a fonte que renderiza ABOVE THE FOLD.** O headline, o título, o hero text.
- **NÃO faça preload de todas as variantes.** Só o peso principal (Regular 400). Os outros pesos podem esperar.
- **Teste SEMPRE.** Preload mal usado PIORA a performance.

### 3.3 Preload + font-display

```html
<!-- Preload: alta prioridade -->
<link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin>

<!-- CSS: fallback IMEDIATO + swap quando carregar -->
<style>
  @font-face {
    font-family: 'Inter';
    src: url('/fonts/inter.woff2') format('woff2');
    font-display: swap;
  }
  body { font-family: 'Inter', system-ui, sans-serif; }
</style>
```

---

## 4. Subsetting: Você Não Precisa de 600 Glifos

### 4.1 O Problema

Uma fonte "Latin" típica tem ~250-600 glifos (letras, números, pontuação, acentos). Você USA ~80-120 deles numa página típica em inglês. Os outros 400+ glifos são BYTES MORTOS.

**Subsetting resolve isso:** corta fora os glifos que sua página NÃO USA.

| Cenário | Tamanho Original (WOFF2) | Após Subsetting | Redução |
|---|---|---|---|
| **Inter (Latin, Regular)** | ~42 KB | ~18 KB (só ASCII) | 57% |
| **Inter (Latin, 4 pesos)** | ~180 KB | ~80 KB | 56% |
| **Noto Sans CJK (chinês)** | ~18 MB | ~100 KB (1 página) | **99.4%** |
| **Material Icons** | ~150 KB | ~8 KB (10 ícones) | 95% |

### 4.2 Ferramentas de Subsetting

| Ferramenta | Uso |
|---|---|
| **[glyphhanger](https://github.com/filamentgroup/glyphhanger)** | CLI que scaneia seu site com Puppeteer, extrai caracteres usados, gera subset |
| **[fonttools (pyftsubset)](https://github.com/fonttools/fonttools)** | Python. `pyftsubset font.ttf --unicodes="U+0020-007E" --flavor=woff2` |
| **[subfont](https://github.com/Munter/subfont)** | Automático — analisa HTML e gera subsets por página |
| **[Google Web Fonts Helper](https://google-webfonts-helper.herokuapp.com/)** | Download de fontes do Google com subsetting manual |
| **Nuxt Fonts** | Zero-config para Nuxt.js — subsetting + fontaine automáticos |

### 4.3 unicode-range: Subsetting Nível CSS

Em vez de gerar UM arquivo subsettado, você pode DIVIDIR a fonte em ranges e deixar o browser baixar SÓ o que precisa:

```css
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter-latin.woff2') format('woff2');
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC;
  font-display: swap;
}

@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter-latin-ext.woff2') format('woff2');
  unicode-range: U+0100-024F, U+0259, U+1E00-1EFF;
  font-display: swap;
}

@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter-cyrillic.woff2') format('woff2');
  unicode-range: U+0400-045F, U+0490-0491;
  font-display: swap;
}
```

**Resultado:** um site em inglês baixa SÓ o `inter-latin.woff2`. Um site em russo baixa `inter-latin` + `inter-cyrillic`. Um site em português (que usa caracteres Latin Extended: ç, ã, õ, é, etc.) baixa os dois — mas SÓ quando a página TEM esses caracteres.

### 4.4 Chrome 125+ (2024): Mudança Crucial

**Chrome 125+ REMOVEU o subsetting automático de fontes do lado do cliente.** Isso significa que fontes `.woff2` COMPLETAS não são mais automaticamente reduzidas pelo browser. **Você PRECISA fazer subsetting server-side.**

---

## 5. CLS: A "Dança" do Layout Quando a Fonte Carrega

### 5.1 O Problema

`font-display: swap` resolve FOIT, mas CRIA outro problema: **Cumulative Layout Shift (CLS).**

Quando a fonte fallback (ex: Arial) é trocada pela fonte customizada (ex: Inter), as MÉTRICAS mudam — x-height, largura, line-height. O texto "salta" alguns pixels. Isso é CLS. E o Google PENALIZA sites com CLS > 0.1.

### 5.2 A Solução: Font Metric Overrides

CSS Fonts Level 4 introduziu descritores no `@font-face` que AJUSTAM a fallback para CASAR com a fonte customizada:

```css
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter.woff2') format('woff2');
  font-display: swap;
}

@font-face {
  font-family: 'Inter-Fallback';
  src: local('Arial'); /* fallback */
  size-adjust: 102%;           /* Ajusta escala geral */
  ascent-override: 95%;        /* Ajusta altura acima da baseline */
  descent-override: 25%;       /* Ajusta altura abaixo da baseline */
  line-gap-override: 10%;      /* Ajusta line-gap */
}
```

**Ferramenta Fontaine** (Daniel Roe, 2024): calcula esses valores AUTOMATICAMENTE para qualquer par fonte-fallback. Zero configuração manual.

### 5.3 A Estratégia Completa Anti-CLS

1. **Calcule** os metric overrides com Fontaine
2. **Aplique** os overrides na `@font-face` da fallback
3. **Use `font-display: swap`** na fonte customizada
4. **Preload** a fonte crítica
5. **Resultado:** texto visível IMEDIATAMENTE. Quando a fonte carregar, a troca é IMPERCEPTÍVEL. CLS = 0.

---

## 6. WOFF2: O Único Formato Que Você Precisa

| Formato | Compressão | Suporte 2025 |
|---|---|---|
| **WOFF2** | Brotli. ~30% menor que WOFF1. | TODOS os browsers modernos |
| WOFF1 | Gzip | Legacy |
| TTF/OTF | Sem compressão (ou gzip server-side) | Não recomendado |
| EOT | Morto | NÃO USE |
| SVG | Morto | NÃO USE |

**Você só precisa de WOFF2.** Web Almanac 2022: *"Forget about everything else."*

---

## 7. Self-Hosting vs CDN (Google Fonts, Adobe Fonts)

| Self-Hosting | CDN (Google Fonts) |
|---|---|
| Sem DNS/TCP/TLS extra | Conexão adicional a `fonts.googleapis.com` |
| Controle TOTAL de cache (`immutable, max-age=31536000`) | Cache controlado pelo Google |
| Privacidade: zero data sharing | Google recebe IP do visitante |
| 1-2 domínios = HTTP/2 multiplexing melhor | Mais domínios = mais conexões |
| **Mais rápido** em benchmarks | Conveniente, mas mais lento |

**Consenso 2025: self-host é MELHOR.** Use o [Google Web Fonts Helper](https://google-webfonts-helper.herokuapp.com/) para baixar fontes do Google e self-hostá-las. Ou use Nuxt Fonts (zero-config).

**Exceção:** fontes com restrição de licença (Adobe Fonts) que NÃO permitem self-hosting.

---

## 8. Fontes Variáveis: Performance + Flexibilidade

Uma fonte variável substitui MÚLTIPLOS arquivos estáticos:

| Estático | Variável |
|---|---|
| Inter-Regular.woff2 (42 KB) | Inter-Variable.woff2 (~150 KB) |
| Inter-Bold.woff2 (42 KB) | (contém TODOS os pesos em 1 arquivo) |
| Inter-Italic.woff2 (44 KB) | |
| **Total: ~180 KB (4 arquivos)** | **Total: ~150 KB (1 arquivo)** |

**Quando usar variável para performance:**

- **3+ pesos** → variável É MAIS LEVE que estático
- **1-2 pesos** → estático pode ser mais leve
- **Animações de peso** → SÓ variável consegue

**Exemplo com Inter:**

```css
@font-face {
  font-family: 'InterVariable';
  src: url('/fonts/InterVariable.woff2') format('woff2-variations');
  font-weight: 100 900;
  font-display: swap;
}
```

---

## 9. Checklist de Performance Tipográfica (2025)

| # | Técnica | Impacto | Esforço |
|---|---|---|---|
| 1 | **`font-display: swap`** em todas as fontes de texto | ALTO (LCP, FCP) | Baixo |
| 2 | **WOFF2 only** — delete TTF, OTF, EOT, WOFF1 | ALTO (bytes) | Baixo |
| 3 | **Preload** 1-2 fontes críticas | MÉDIO (LCP) | Baixo |
| 4 | **Self-host** as fontes | MÉDIO (tempo de conexão) | Médio |
| 5 | **Subsetting** — remova glifos não usados | ALTO (bytes) | Médio |
| 6 | **Font metric overrides** (Fontaine) para CLS = 0 | ALTO (CLS) | Baixo |
| 7 | **Variável** se 3+ pesos | MÉDIO (bytes + requests) | Baixo |
| 8 | **`unicode-range`** para sites multilíngues | MÉDIO (bytes condicionais) | Médio |
| 9 | **Cache headers** `immutable, max-age=31536000` | MÉDIO (repeat views) | Baixo |
| 10 | **Font stack** robusta (`system-ui, sans-serif`) | BAIXO (fallback) | Baixo |

---

## 10. A Configuração Canônica (2025)

```html
<!-- HTML: preload da fonte crítica -->
<link rel="preload" href="/fonts/inter-latin.woff2" as="font" type="font/woff2" crossorigin>
```

```css
/* CSS: @font-face com TODAS as otimizações */

/* Fonte customizada (swap = FOUT) */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter-latin.woff2') format('woff2');
  font-display: swap;
  font-weight: 100 900; /* variável */
}

/* Fallback com métricas ajustadas */
@font-face {
  font-family: 'Inter-Fallback';
  src: local('Arial');
  size-adjust: 102%;
  ascent-override: 95%;
  descent-override: 25%;
}

/* Font stack final */
body {
  font-family: 'Inter', 'Inter-Fallback', system-ui, sans-serif;
  font-optical-sizing: auto;
}
```

**Resultado:**
- Texto visível em **<100ms** (zero FOIT)
- Troca de fonte **imperceptível** (zero CLS)
- **1 HTTP request** para fontes (preload + cache)
- **~18 KB** de fonte (subset Latin ASCII)
- **LCP e CLS** passam no Core Web Vitals

---

## 11. O Futuro: Incremental Font Transfer (IFT)

O **W3C** está desenvolvendo o **Incremental Font Transfer** (Candidate Recommendation, 2025). A ideia: em vez de baixar uma fonte INTEIRA, o browser baixa APENAS os glifos necessários para a página ATUAL — e faz PATCH incremental quando o usuário navega para páginas com novos caracteres.

```css
@font-face {
  font-family: 'MyFont';
  src: url('/font.otf') tech(incremental);
}
```

**Vantagens sobre unicode-range:** preserva kerning e ligaduras entre ranges. Funciona com scripts complexos (árabe, devanagari). Ainda é experimental (CRD), mas é o FUTURO.

---

## 12. Não Otimize Demais

Uma nota de sanidade: fontes são importantes, mas NÃO são o maior problema de performance da web. JavaScript e imagens são BEM piores.

**Se seu JS bundle tem 800 KB e você está gastando 4 horas otimizando 40 KB de fonte... você está otimizando a coisa errada.**

Aplique as técnicas deste loop. Depois VÁ OTIMIZAR SEU JAVASCRIPT.

---

## Fontes Citadas Neste Loop

| Ferramenta/Tecnologia | Criador | Nota |
|---|---|---|
| **Fontaine** | Daniel Roe | Métricas automáticas fallback → custom |
| **Nuxt Fonts** | Nuxt.js team | Zero-config font optimization |
| **[glyphhanger](https://github.com/filamentgroup/glyphhanger)** | Filament Group | Subsetting com Puppeteer |
| **[fonttools (pyftsubset)](https://github.com/fonttools/fonttools)** | Just van Rossum + comunidade | Subsetting industrial |
| **[subfont](https://github.com/Munter/subfont)** | Peter Müller | Subsetting automático por página |
| **[Google Web Fonts Helper](https://google-webfonts-helper.herokuapp.com/)** | Mario Ranftl | Download + self-host Google Fonts |
| **WOFF2 (Brotli)** | W3C | Compressão de fontes |
| **IFT (Incremental Font Transfer)** | W3C Working Group | O futuro do carregamento de fontes |
| **Core Web Vitals (LCP, CLS, FCP)** | Google | Métricas de performance web |
| **Inter Variable** | Rasmus Andersson | Fonte variável referência |

---

> **Próximo loop sugerido:** Tipografia em Design Systems — type ramp, escalas tipográficas, como estruturar pesos e tamanhos, e manter consistência entre web e mobile. Ou: **Monospace & Code Fonts** — as melhores fontes para código em 2025.
