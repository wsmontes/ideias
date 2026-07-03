# 🔤 Estudo de Fontes Loop 13 — Tipografia em Design Systems: Type Ramp, Escalas e Consistência

> **Data:** 2026-07-02
> **Loop:** 13 de ∞
> **Tema:** Como estruturar tipografia em design systems — type ramp, escalas modulares, tokens atômicos e compostos, fluid typography, e os sistemas das big techs (Apple HIG, Material 3, Fluent). O que separa um sistema tipográfico ROBUSTO de "tamanhos aleatórios que alguém escolheu."

---

## 0. O Problema: "H1: 32px, H2: 24px, Body: 16px... Por Quê?"

A maioria dos projetos define tipografia ASSIM: "H1: 32px, H2: 24px, Body: 16px, Caption: 12px." São números ARBITRÁRIOS. Alguém escolheu porque "parecia bom."

Isso funciona para UM site. Mas quando você tem:
- 3 plataformas (web, iOS, Android)
- Light + Dark mode
- 2 marcas com fontes diferentes
- 20 componentes que consomem tipografia

...os números arbitrários COLAPSAM. Você precisa de um SISTEMA.

**Um design system tipográfico robusto responde a 4 perguntas:**
1. **Qual a escala?** (os tamanhos e suas relações matemáticas)
2. **Qual a hierarquia?** (Display → Headline → Title → Body → Caption)
3. **Como se adapta?** (responsive, fluid, breakpoints)
4. **Como se governa?** (tokens, variáveis, theming)

Este loop responde as 4.

---

## 1. Type Ramp: O "Menu" de Tamanhos do Seu Sistema

### 1.1 O Que É

Uma **type ramp** é um conjunto PRE-DEFINIDO de tamanhos tipográficos — tipicamente 6 a 12 níveis. Componentes NUNCA escolhem tamanhos arbitrários — eles referenciam um nível da ramp.

```
Display Large   → 57px
Display Medium  → 45px
Display Small   → 36px
Headline Large  → 32px
Headline Medium → 28px
Headline Small  → 24px
Title Large     → 22px
Title Medium    → 16px
Title Small     → 14px
Body Large      → 16px
Body Medium     → 14px
Body Small      → 12px
Label Large     → 14px
Label Medium    → 12px
Label Small     → 11px
```

**Regra de ouro:** "Não quebre a ramp." Se um componente PRECISA de 23px e a ramp só tem 20px e 24px, o problema é o DESIGN — não a ramp. A ramp é uma RESTRIÇÃO CRIATIVA, não uma limitação.

### 1.2 Naming: Papéis, Não Tamanhos

❌ **Ruim:** `h1`, `h2`, `text-12`, `font-16-bold`  
✅ **Bom:** `Display/Large`, `Headline/Medium`, `Body/Regular`, `Caption/Small`

**Por quê?** Nomes baseados em PAPEL sobrevivem a redesigns. Se amanhã o Body muda de 16px para 15px, o nome `Body/Medium` continua válido. O nome `text-16`... não.

| Papel | Uso Típico |
|---|---|
| **Display** | Hero, landing pages. SÓ texto curto e importante. |
| **Headline** | Títulos de seção, H1, H2. |
| **Title** | Subtítulos, cards, diálogos. |
| **Body** | Parágrafos, texto corrido. |
| **Label** | Botões, formulários, chips, badges. |
| **Caption** | Metadados, timestamps, notas de rodapé. |

### 1.3 Como as Big Techs Nomeiam

| Apple (HIG) | Material 3 (Google) | Fluent (Microsoft) |
|---|---|---|
| Large Title | Display Large | Display |
| Title 1 | Display Medium | Title 1 |
| Title 2 | Headline Large | Title 2 |
| Title 3 | Headline Medium | Subtitle 1 |
| Headline | Headline Small | Body 1 |
| Body | Body Large | Body 2 |
| Callout | Body Medium | Caption 1 |
| Subhead | Body Small | Caption 2 |
| Footnote | Label Large | Overline |
| Caption 1 | Label Medium | — |
| Caption 2 | Label Small | — |

**Todas usam papéis, não tamanhos.**

---

## 2. Escalas Tipográficas: A Matemática Por Trás dos Tamanhos

Uma **escala tipográfica** define a RELAÇÃO matemática entre os tamanhos. Cada passo é o passo anterior × uma razão:

```
fontSize(n) = baseSize × ratioⁿ
```

### 2.1 As 7 Razões Clássicas

| Escala | Razão | Contraste | Exemplo (base 16px → 6 passos) | Melhor Para |
|---|---|---|---|---|
| **Minor Second** | 1.067 | Muito sutil | 16 → 24 | Dashboards, apps densos |
| **Major Second** | 1.125 | Sutil | 16 → 32 | Product UI, SaaS |
| **Minor Third** | 1.200 | Moderado | 16 → 48 | Texto pesado, blogs |
| **Major Third** | 1.250 | Moderado | 16 → 61 | Web geral (a mais versátil) |
| **Perfect Fourth** | 1.333 | Médio-alto | 16 → 90 | Blogs, educação, editorial |
| **Golden Ratio** | 1.618 | Alto | 16 → 288 | Portfolios, luxo, landing pages |
| **Perfect Fifth** | 1.500 | Alto | 16 → 182 | Posters, marketing |

### 2.2 Como Escolher

| Projeto | Escala Recomendada |
|---|---|
| **Dashboard, app denso** | Minor Second (1.067) ou Major Second (1.125) |
| **Site de produto, SaaS** | Major Second (1.125) ou Minor Third (1.200) |
| **Blog, conteúdo** | Major Third (1.250) ou Perfect Fourth (1.333) |
| **Landing page, marketing** | Perfect Fourth (1.333) ou Golden Ratio (1.618) |
| **Portfolio criativo** | Golden Ratio (1.618) |

### 2.3 A Regra do Múltiplo de 4

Arredonde TODOS os tamanhos para múltiplos de 4px (ou 8px). Razão: grids de layout usam base 4 ou 8. Tamanhos que não são múltiplos criam DESALINHAMENTO vertical.

```
❌ 13px, 15px, 17px, 19px, 23px, 31px
✅ 12px, 16px, 20px, 24px, 28px, 32px
```

---

## 3. Tokens: A Arquitetura Que Sustenta Tudo

### 3.1 Tokens Atômicos vs Compostos

**Tokens atômicos** são os tijolos. **Tokens compostos** são as paredes.

```json
// Atômicos (decisões individuais)
{
  "fontFamily": { "default": { "value": "Inter" } },
  "fontSize":   { "md": { "value": "16px" } },
  "fontWeight": { "bold": { "value": "700" } },
  "lineHeight": { "body": { "value": "1.5" } },
  "letterSpacing": { "tight": { "value": "-0.02em" } }
}

// Compostos (tipografia completa)
{
  "body": {
    "regular": {
      "value": {
        "fontFamily": "{fontFamily.default}",
        "fontSize": "{fontSize.md}",
        "fontWeight": "{fontWeight.regular}",
        "lineHeight": "{lineHeight.body}",
        "letterSpacing": "{letterSpacing.normal}"
      }
    }
  }
}
```

**A mágica:** mudar `fontFamily.default` de Inter para Roboto Flex ATUALIZA automaticamente TODO componente que referencia tipografia. É uma MUDANÇA, não 50.

### 3.2 Exemplo Prático: CSS Custom Properties

```css
:root {
  /* Atômicos */
  --font-sans: 'Inter', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', monospace;
  --font-base: 1rem; /* 16px */
  --font-scale: 1.25; /* Major Third */
  --lh-body: 1.5;
  --lh-heading: 1.2;

  /* Compostos (calculados via calc ou preprocessador) */
  --text-body: 1rem;
  --text-h6: calc(var(--text-body) * var(--font-scale));
  --text-h5: calc(var(--text-h6) * var(--font-scale));
  --text-h4: calc(var(--text-h5) * var(--font-scale));
  --text-h3: calc(var(--text-h4) * var(--font-scale));
  --text-h2: calc(var(--text-h3) * var(--font-scale));
  --text-h1: calc(var(--text-h2) * var(--font-scale));
}
```

---

## 4. Responsive & Fluid Typography

### 4.1 A Estratégia Antiga: Breakpoints Fixos

```css
h1 { font-size: 28px; }
@media (min-width: 768px) { h1 { font-size: 40px; } }
@media (min-width: 1024px) { h1 { font-size: 48px; } }
```

**Problema:** "saltos" visíveis no resize. Funciona, mas não é elegante.

### 4.2 A Estratégia Moderna: Fluid com clamp()

```css
h1 {
  /* clamp(min, preferido, max) */
  font-size: clamp(2rem, 1rem + 2vw, 3.5rem);
}
```

- **Mobile (375px):** `1rem + 2 × 3.75vw` = 1.75rem (28px)
- **Desktop (1440px):** `1rem + 2 × 14.4vw` = 3.88rem → clampado em 3.5rem (56px)

**Sem breakpoints. Sem saltos. Transição SUAVE entre mobile e desktop.**

### 4.3 A Ferramenta: Utopia.fyi

[Utopia](https://utopia.fyi/) é o padrão 2025 para fluid typography. Você define:
- Viewport mínimo (ex: 320px)
- Viewport máximo (ex: 1440px)
- Base size mínimo (ex: 16px)
- Base size máximo (ex: 18px)
- Escala (ex: Major Third 1.25)

**Utopia gera TODOS os tokens fluidos.** Pronto para CSS.

### 4.4 Fluid + Escala Dupla (Avançado)

Você pode usar uma escala DIFERENTE em mobile vs desktop:

```json
{
  "viewportMin": 320,
  "viewportMax": 1500,
  "baseMin": 16,
  "baseMax": 18,
  "scaleMin": "minor-third",    // 1.200 (mobile)
  "scaleMax": "perfect-fourth"  // 1.333 (desktop)
}
```

**Mobile:** hierarquia sutil (pouco espaço). **Desktop:** hierarquia dramática (muito espaço).

---

## 5. Line Height: A Régua Invisível

### 5.1 A Regra Proporcional

**Tamanhos GRANDES precisam de line-height MENOR. Tamanhos PEQUENOS precisam de line-height MAIOR.**

| Tamanho | Line Height | Razão |
|---|---|---|
| Display (48px+) | 1.05-1.1 | Apertado — letras grandes não colidem |
| Headline (24-47px) | 1.15-1.25 | Moderado |
| Title (18-23px) | 1.2-1.3 | Normal |
| Body (14-17px) | 1.4-1.6 | Generoso — ajuda a "respirar" |
| Caption (11-13px) | 1.3-1.5 | Ligeiramente generoso |

### 5.2 Line Length: 45-75 Caracteres

**O texto NÃO DEVE ocupar a largura INTEIRA da tela.** Uma linha de 200 caracteres é ILEGÍVEL — o olho se perde na volta.

```css
article {
  max-width: 65ch; /* ~65 caracteres */
}
```

**`ch`** é a unidade mágica: 1ch = largura do caractere '0' na fonte atual. `65ch` = ~65 caracteres por linha. Universal.

---

## 6. Os Sistemas das Big Techs: O Que Copiar

### 6.1 Apple Human Interface

**Precisão ABSURDA.** Cada estilo tem tracking, leading, e optical size especificados:

| Style | Size | Weight | Tracking |
|---|---|---|---|
| Large Title | 34pt | Bold | -1.05px |
| Title 1 | 28pt | Bold | -0.8px |
| Body | 17pt | Regular | -0.43px |
| Caption 1 | 12pt | Regular | +0.12px |

**Regra Apple:** tracking NEGATIVO em sizes grandes (letras "grudam"), tracking POSITIVO em sizes pequenos (letras "respiram").

### 6.2 Material 3 (Google)

**Token-first.** TUDO é token. Light/dark mode nativo. 5 roles (Display, Headline, Title, Body, Label) × 3 sizes (Large, Medium, Small) = 15 estilos base.

**Inovação M3:** `font-optical-sizing: auto` como default. O browser ajusta o desenho dos glifos AUTOMATICAMENTE baseado no tamanho.

### 6.3 Fluent (Microsoft)

**Alta legibilidade como invariante.** Segoe UI Variable com Display/Text/Small optical sizes. Foco em CONTRASTE (WCAG AAA sempre que possível) e DENSIDADE (UI do Windows precisa funcionar em telas 4K a 200%).

---

## 7. O Checklist de Um Design System Tipográfico Robusto

| # | Critério | ✅ |
|---|---|---|
| 1 | **Base size definida** (16px web, 17pt iOS, 16sp Android) | |
| 2 | **Escala com razão consistente** (1.067-1.618, documentada) | |
| 3 | **Type ramp com 8-12 níveis** nomeados por papel (Display, Headline, Title, Body, Label) | |
| 4 | **Line height proporcional** (1.05-1.1 display, 1.5 body, 1.3 caption) | |
| 5 | **Line length limitada** (max-width: 65ch para corpo de texto) | |
| 6 | **Tokens atômicos + compostos** (mudar fonte família em 1 lugar atualiza tudo) | |
| 7 | **Responsivo fluido** (clamp() ou breakpoints com escalas diferentes) | |
| 8 | **Acessibilidade** (WCAG AA contraste, mínimo 14-16px body) | |
| 9 | **Multi-plataforma** (web: rem, iOS: pt, Android: sp) | |
| 10 | **Multi-marca** (theming via tokens — troca fontFamily e escala) | |
| 11 | **Dark mode** (contraste validado em ambos os modos) | |
| 12 | **Documentado** (cada token explicado, exemplos visuais) | |

---

## Fontes Citadas Neste Loop

| Ferramenta/Sistema | Criador | Nota |
|---|---|---|
| **Utopia.fyi** | James Gilyead / Trys Mudford | Fluid typography padrão 2025 |
| **Typescale.com** | Jeremy Church | Visualizador de escalas |
| **Figma Variables** | Figma | Tokens atômicos + modos |
| **Tokens Studio** | Tokens Studio | Tokens JSON ↔ Figma ↔ código |
| **Style Dictionary** | Amazon | Gerador de tokens multi-plataforma |
| **Apple HIG** | Apple | 11 estilos com tracking/leading precisos |
| **Material 3** | Google | Token-first, light/dark nativo |
| **Fluent 2** | Microsoft | Segoe UI Variable, foco em acessibilidade |
| **IBM Carbon** | IBM | Documentação + auditoria de contraste |

---

> **Próximo loop sugerido:** Tipografia & Acessibilidade — WCAG, contraste, dislexia, fontes acessíveis, e como projetar texto que TODO MUNDO pode ler. Ou: **Tipografia & Emoção** — como a fonte afeta o que as pessoas SENTEM ao ler.
