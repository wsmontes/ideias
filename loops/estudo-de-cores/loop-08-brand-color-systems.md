# 🎨 Estudo de Cores Loop 08 — Brand Color Systems: Como Os Melhores Do Mundo Estruturam Cor

> **Data:** 2026-07-02
> **Loop:** 8 de ∞
> **Tema:** Estudos de caso. Como Stripe, Linear, Vercel, Apple e GitHub constroem seus sistemas de cor. Não é teoria — é engenharia de cor aplicada. O que cada um acertou, o que cada um errou, e o que você pode copiar.

---

## 0. Por Que Estudar Sistemas Reais

Até agora, os loops 01-07 cobriram a TEORIA: espaços de cor, acessibilidade, dark mode, paletas algorítmicas, tokens, gradientes, wide gamut. Este loop fecha o ciclo com PRÁTICA: como 5 das melhores empresas de tecnologia do mundo APLICAM essa teoria.

Cada uma resolveu o problema de cor de um jeito diferente. Cada uma tem uma FILOSOFIA diferente. E cada uma tem lições que você pode aplicar HOJE.

---

## 1. Stripe: Minimalismo Radical (~15 Tokens de Cor)

### 1.1 A Filosofia

**"Menos cores = mais impacto por cor."**

Stripe opera um dos negócios mais valiosos do mundo com ~15 tokens de cor. Isso não é preguiça — é DISCIPLINA.

### 1.2 A Paleta

```
Stripe Blue: #635BFF (ou oklch(0.52 0.22 270))
  → A ÚNICA cor de marca. Todo o resto são cinzas.

Neutros:
  White, Gray-50, Gray-100, Gray-200, ..., Gray-900

Estados (gerados por OPACIDADE, não tokens):
  Hover:   stripeBlue + 10% white overlay
  Active:  stripeBlue + 10% black overlay
  Disabled: stripeBlue a 38% opacity
  Focus:   stripeBlue com outline
```

**Stripe NÃO TEM:**
- `--color-secondary`
- `--color-accent`
- `--color-success` / `--color-danger` explícitos (usam cinzas + ícones)
- Escala de 050-950 para azul
- Cores "de apoio"

**Stripe TEM:**
- UM azul
- Uma escala de cinza
- Gradiente gerativo como elemento de marca
- Opacidade como gerador de estados

### 1.3 O Gradiente Como Marca

A Stripe não usa o azul em tudo. O elemento mais reconhecível da marca Stripe é o **gradiente cósmico generativo** do site:

- Múltiplos centros radiais animados via WebGL
- Paleta: azul escuro → violeta → ciano → azul claro
- OKLCH internamente para transições suaves
- Animação lenta (~0.1px/s) — quase imperceptível, mas VIVA

O gradiente NÃO usa o `#635BFF` (Stripe Blue). Usa tons PRÓXIMOS — azuis mais escuros, mais violetas, mais cianos. O `#635BFF` é reservado para CTAs e elementos interativos. O gradiente cria ATMOSFERA. O azul da marca cria AÇÃO.

### 1.4 Lições da Stripe

1. **Você não precisa de 500 tokens.** 15 tokens bem usados valem mais que 500 tokens mal usados.
2. **Opacidade gera estados melhor que tokens separados.** `rgba(brand, 0.1)` é mais maintainable que `--color-brand-hover`.
3. **Separe ATMOSFERA de AÇÃO.** O gradiente cria o clima. A cor sólida chama o clique.
4. **Uma cor de marca é suficiente.** Se você precisa de 5 cores "primárias," você não tem UMA marca — você tem 5.

---

## 2. Linear: Dark-First, OKLCH-Native, Gerado Por Código

### 2.1 A Filosofia

**"Cor é código. Designers não escolhem cores — algoritmos geram."**

Linear é um app de project management para desenvolvedores. Seu sistema de cor reflete seu público: engenheiros que valorizam SISTEMAS sobre artefatos.

### 2.2 A Paleta

```
Dark mode por PADRÃO (light mode é o "alternativo")

Backgrounds (escala de elevação tonal — loop 03):
  bg-0: oklch(0.08 0 0)     # base
  bg-1: oklch(0.11 0 0)     # card
  bg-2: oklch(0.14 0 0)     # elevated
  bg-3: oklch(0.17 0 0)     # modal

Accent colors (GERADAS, não escolhidas):
  Blue:   oklch(0.62 0.18 260)
  Green:  oklch(0.62 0.18 140)
  Yellow: oklch(0.62 0.18 80)
  Red:    oklch(0.62 0.18 20)
  Purple: oklch(0.62 0.18 290)

  Notem: MESMO lightness (0.62), MESMO chroma (0.18).
  Mudou SÓ o matiz. Isso é IMPOSSÍVEL de fazer "no olho" em HSL.
```

### 2.3 O Sistema de Geração

Linear não tem um designer escolhendo `blue-500`, `blue-600`. Em vez disso:

```typescript
// Simplified Linear color system
function linearColor(hue: number, shade: number) {
  // shade: 0 (claro) a 1000 (escuro)
  const L = 0.95 - shade / 1000 * 0.80;  // lightness perceptual linear
  const C = chromaCurve(L);               // curva de sino (loop 04)
  return oklch(L, C, hue);
}

function linearPalette(hue: number) {
  return {
    0:    linearColor(hue, 0),     // quase branco
    100:  linearColor(hue, 100),
    200:  linearColor(hue, 200),
    300:  linearColor(hue, 300),
    400:  linearColor(hue, 400),
    500:  linearColor(hue, 500),   // cor "base"
    600:  linearColor(hue, 600),
    700:  linearColor(hue, 700),
    800:  linearColor(hue, 800),
    900:  linearColor(hue, 900),
    1000: linearColor(hue, 1000),  // quase preto
  };
}
```

**Todo o sistema de cor do Linear são DUAS funções.** `chromaCurve()` e `linearColor()`. O resto é composição.

### 2.4 Lições do Linear

1. **Dark-first muda tudo.** Você pensa em ELEVAÇÃO TONAL (mais claro = mais alto) desde o dia 1.
2. **OKLCH permite gerar paletas matematicamente.** Sem subjetividade, sem "parece bom." Matemática.
3. **Mesmo L e C para todas as cores de acento.** Peso visual idêntico. Nenhuma cor "grita" mais que outra.
4. **Código gera cor, não o contrário.** Seu design system pode ser uma função, não um arquivo JSON com 500 valores hardcoded.

---

## 3. Vercel: Foreground/Background, Geist, P3

### 3.1 A Filosofia

**"A separação foreground/background sobrevive a qualquer modo de cor."**

O Geist (design system da Vercel) não pensa em "cor primária," "cor secundária." Pensa em:
- **Foreground:** texto, ícones, bordas — coisas que vêm para a FRENTE
- **Background:** superfícies, cards, fundos — coisas que vão para TRÁS

### 3.2 A Paleta

```
Foreground:
  fg-100: oklch(0.98 0 0)    # quase branco (texto em dark mode)
  fg-200: oklch(0.85 0 0)
  ...
  fg-900: oklch(0.08 0 0)    # quase preto (texto em light mode)

Background:
  bg-100: oklch(0.08 0 0)    # quase preto (fundo dark mode)
  bg-200: oklch(0.12 0 0)
  ...
  bg-900: oklch(0.98 0 0)    # quase branco (fundo light mode)

Accent (minimalista):
  accent-100: oklch(0.90 0.05 270) a accent-900: oklch(0.15 0.15 270)
```

### 3.3 Por Que Foreground/Background é GENIAL

Pense em um token tradicional: `--color-text-primary`. Em light mode é preto. Em dark mode é branco. São cores OPOSTAS. O token "mente" — dependendo do modo, ele significa coisas diferentes.

No modelo foreground/background:

```css
/* Light mode */
--fg-default: var(--gray-900);  /* foreground escuro sobre... */
--bg-default: var(--gray-100);  /* ...background claro */

/* Dark mode */
--fg-default: var(--gray-100);  /* foreground claro sobre... */
--bg-default: var(--gray-900);  /* ...background escuro */
```

**Foreground e background são INVERSOS um do outro, e isso é EXPLÍCITO.** Não é `--text-primary` fingindo ser uma cor quando na verdade são duas.

### 3.4 P3 Desde o Dia 1

O Geist foi lançado COM suporte a Display P3:

```css
--geist-accent: oklch(0.55 0.28 270);
/* chroma 0.28 > 0.22 (máximo sRGB) → automaticamente P3 quando disponível */
```

### 3.5 Lições da Vercel

1. **Foreground/background > primary/secondary.** O modelo fg/bg funciona EM QUALQUER modo de cor sem "mentir."
2. **Menos matizes = mais elegância.** O Geist tem basicamente UM matiz de acento. Todo o resto é cinza.
3. **P3 desde o início.** Se você vai fazer wide gamut, faça no design system, não como afterthought.
4. **Tailwind como veículo.** O Geist é distribuído como plugin Tailwind. Milhares de devs usam sem pensar em OKLCH — mas estão usando OKLCH.

---

## 4. Apple: Cor Como Adaptação Ao Contexto

### 4.1 A Filosofia

**"Cor não é um valor fixo. É um valor que RESPONDE ao ambiente do usuário."**

O Human Interface Guidelines (HIG) da Apple é o sistema de cor mais sofisticado do mundo. Não porque tem muitas cores — porque cada cor é **dinâmica.**

### 4.2 O Sistema de Cores do iOS

```
System Colors (15 cores base):
  systemBlue, systemRed, systemGreen, systemYellow,
  systemOrange, systemPink, systemPurple, systemTeal,
  systemIndigo, systemMint, systemCyan, systemGray,
  systemBrown, systemMaroon, systemLime

CADA UMA se adapta a:
  - Light mode
  - Dark mode
  - High Contrast mode (acessibilidade)
  - Aumento de contraste (preferência do usuário)

Além disso, CADA UMA tem variantes:
  - Primary (mais vibrante)
  - Secondary (mais suave)
  - Tertiary (quase neutra)
  - Quaternary (fundo sutil)
```

**15 cores × 4 modos × 4 variantes = 240 valores de cor.** E isso são SÓ as system colors. Sem contar os milhares de semantic colors derivados.

### 4.3 Dynamic Color no SwiftUI

```swift
// SwiftUI — a cor "sabe" em que modo está
RoundedRectangle(cornerRadius: 12)
    .fill(Color.blue)  // automaticamente light/dark/high-contrast

// Semantic: a cor sabe seu SIGNIFICADO
Text("Delete")
    .foregroundColor(.red)  // automaticamente adaptado ao contexto

// Personalizada com variantes dinâmicas
Color(
    light: Color(oklchL: 0.55, c: 0.22, h: 260),
    dark: Color(oklchL: 0.65, c: 0.15, h: 260)
)
```

### 4.4 Mesh Gradients Como Linguagem Visual

A Apple popularizou mesh gradients (loop 06) como textura de fundo em:
- Wallpapers do iOS/macOS
- Apple Music (background animado que responde à capa do álbum)
- Telas de setup e onboarding
- Apple TV (screen savers aéreos com gradientes sutis)

**O mesh gradient da Apple não é "decoração" — é material design.** Ele comunica profundidade, modernidade, e "qualidade Apple."

### 4.5 Lições da Apple

1. **Cor deve ser DINÂMICA.** Uma cor fixa é uma cor quebrada em pelo menos 3 modos (light, dark, high-contrast).
2. **Sistema > Paleta.** A Apple fornece 15 system colors que se adaptam automaticamente. O desenvolvedor NÃO escolhe hex codes.
3. **Variantes de saturação.** Primary (vibrante), Secondary (suave), Tertiary (neutra), Quaternary (fundo). A mesma cor, diferentes intensidades, diferentes usos.
4. **Mesh gradients são a textura da marca.** Criam atmosfera sem competir com o conteúdo.

---

## 5. GitHub Primer: 10 Anos de Lições (Open Source)

### 5.1 A Filosofia

**"Design systems EVOLUEM. Planeje a migração desde o dia 1."**

O Primer é o design system do GitHub. É open source. Você pode LER cada token, cada decisão, cada migração. É o caso de estudo mais HONESTO porque está documentado publicamente.

### 5.2 As 3 Camadas (Exemplo Real)

```json
// CAMADA 1: Primitives (base perceptual)
"base": {
  "color": {
    "blue": {
      "0": { "$value": "oklch(0.98 0.01 260)" },
      "5": { "$value": "oklch(0.55 0.20 260)" }
    }
  }
}

// CAMADA 2: Semantics (significado funcional)
"fg": {
  "default": { "$value": "{base.color.neutral.9}" },
  "muted":   { "$value": "{base.color.neutral.6}" },
  "accent":  { "$value": "{base.color.blue.5}" },
  "danger":  { "$value": "{base.color.red.5}" }
}

// CAMADA 3: Components (uso específico)
"button": {
  "primary": {
    "bg": { "$value": "{fg.accent}" },
    "fg": { "$value": "{base.color.white}" }
  }
}
```

### 5.3 Três Modos de Cor

O Primer suporta 3 modos (não 2):

| Modo | Para quem | Diferença |
|---|---|---|
| **Light** | Padrão, usuários sem preferência | Fundo branco, texto escuro |
| **Dark** | `prefers-color-scheme: dark` | Fundo escuro, texto claro |
| **High Contrast** | `prefers-contrast: more` | Máximo contraste, mínimo chroma |

O modo HIGH CONTRAST é o diferencial — não é só "dark mode com mais contraste." É um modo totalmente separado, com escolhas de cor específicas para usuários com baixa visão.

### 5.4 A História das Migrações

O GitHub passou por **3 sistemas de cor em 10 anos:**

1. **LESS variables** (2012-2017): `@blue-500: #4183c4;`
2. **CSS custom properties** (2017-2022): `--color-blue-500: #4183c4;`
3. **Design tokens + Style Dictionary** (2022-presente): tokens JSON → CSS/Swift/Kotlin/React

**Lições das migrações:**
- Renomear `blue` para `accent` custou 6 meses e 300+ PRs
- Tokens depreciados viveram como aliases por 12 meses antes de serem removidos
- Cada migração quebrou algo em produção. A terceira quebrou MENOS porque tinha aliasing.

### 5.5 Lições do GitHub

1. **3 modos, não 2.** High contrast é um modo SEPARADO. Não é "dark mode com mais contraste."
2. **Aliasing salva vidas em migrações.** `--color-danger: var(--fg-danger)` permite migrar sem quebrar.
3. **Open source é documentação viva.** As issues e PRs do Primer contam a história REAL das decisões.
4. **Seu sistema de tokens VAI ser reescrito em 5 anos.** Estrutura de camadas e aliasing são o que tornam a reescrita POSSÍVEL.

---

## 6. Comparação: 5 Filosofias de Cor

| Dimensão | Stripe | Linear | Vercel | Apple | GitHub |
|---|---|---|---|---|---|
| **Tokens de cor** | ~15 | Gerados | ~30 | 240+ | ~200 |
| **Espaço de cor** | sRGB → OKLCH | OKLCH nativo | OKLCH + P3 | Dynamic Color | sRGB → OKLCH |
| **Abordagem** | Minimalismo | Algorítmica | Foreground/Bg | Adaptativa | 3 camadas |
| **Dark mode** | Sim | DEFAULT | Sim | Automático | Sim + High Contrast |
| **Geração** | Manual | CÓDIGO | Manual + curva | Manual + SO | Manual |
| **Estados** | Opacidade | OKLCH relativo | Opacidade | Variantes do SO | Tokens explícitos |
| **Figma sync** | Manual | Tokens → código | Tokens Studio | Apple Design Kit | Tokens JSON |
| **Filosofia chave** | "Menos é mais" | "Cor é código" | "fg/bg sobrevive" | "Cor responde" | "Migração é certa" |

---

## 7. A Síntese: O Que Copiar De Cada Um

### 7.1 De Cada Sistema, Uma Lição

| Empresa | Leve Isso Para Seu Sistema |
|---|---|
| **Stripe** | Minimize o número de cores. 15 tokens bem usados > 500 tokens. Opacidade para estados. |
| **Linear** | Gere paletas aloritmicamente com OKLCH. Mesmo L e C para todas as cores de acento. |
| **Vercel** | Use foreground/background em vez de primary/secondary. Funciona em qualquer modo. |
| **Apple** | Cor dinâmica que responde ao ambiente. Variantes de saturação para hierarquia visual. |
| **GitHub** | 3 camadas + aliasing desde o dia 1. Modo high contrast separado. Planeje a migração. |

### 7.2 O Sistema Híbrido Ideal (2026)

Juntando o melhor de todos:

```css
/* CAMADA 1: Primitives OKLCH (Linear + Vercel) */
--base-blue:    oklch(0.55 0.22 260);
--base-green:   oklch(0.55 0.22 140);
--base-red:     oklch(0.55 0.22 20);
--base-neutral: oklch(0.98 0 0);

/* CAMADA 2: Semântica Foreground/Background (Vercel) */
--fg-default:   light-dark(var(--base-neutral-900), var(--base-neutral-100));
--fg-accent:    light-dark(var(--base-blue), var(--base-blue-light));
--bg-default:   light-dark(var(--base-neutral-100), var(--base-neutral-900));

/* CAMADA 3: Componentes (GitHub) */
--btn-primary-bg:  var(--fg-accent);
--btn-primary-fg:  var(--base-white);

/* Estados por opacidade (Stripe) */
--btn-primary-hover: oklch(from var(--btn-primary-bg) calc(l + 0.05) c h);

/* Alta saturação para P3 (Vercel) */
--btn-primary-bg: oklch(0.55 0.28 260);  /* chroma > 0.22 = P3 automático */

/* High contrast (GitHub) */
@media (prefers-contrast: more) {
  --fg-muted: var(--fg-default);  /* elimina texto secundário */
  --bg-subtle: var(--bg-default);  /* elimina superfícies sutis */
}

/* Dark mode (Apple + Linear) */
/* light-dark() já resolve, mas para cores complexas: */
@media (prefers-color-scheme: dark) {
  --base-blue: oklch(0.65 0.15 260);  /* mais claro, menos chroma */
}
```

---

## 8. Para Levar Para Casa

1. **Stripe prova que minimalismo escala.** 15 tokens. Opacidade para estados. O gradiente cria atmosfera, a cor sólida cria ação.

2. **Linear prova que cor é código.** Duas funções geram o sistema inteiro. OKLCH nativo. Dark-first.

3. **Vercel prova que foreground/background é o modelo certo.** Sobrevive a qualquer modo de cor. P3 desde o início.

4. **Apple prova que cor deve ser dinâmica.** 15 system colors → 240 valores com adaptação automática ao contexto.

5. **GitHub prova que tokens evoluem.** 3 camadas + aliasing = migração possível. High contrast como modo separado.

6. **O híbrido ideal existe.** OKLCH + fg/bg + 3 camadas + opacidade para estados + P3 automático + high contrast. Dá pra fazer HOJE.

---

## Referências

- Stripe — *Brand Guidelines & Design System* (2020-2025). Documentação pública parcial.
- Linear — *Linear Design* (2021-2025). Documentação pública do design system.
- Vercel — *Geist Design System* (2024-2025). Código aberto no GitHub.
- Apple — *Human Interface Guidelines — Color* (2013-2026). Documentação completa.
- GitHub — *Primer Design System* (2012-2026). Open source. Disponível em primer.style.
- Shopify — *Polaris* (referência complementar de design system de cor).
- Radix UI — *Colors* (referência de escalas com contraste garantido).
- Tailwind CSS — *Color System* (referência de curvas de shade).

---

*Fim do Estudo de Cores Loop 08. Cor não é teoria — é engenharia aplicada. Stripe ensina minimalismo. Linear ensina código. Vercel ensina arquitetura. Apple ensina adaptação. GitHub ensina evolução. O melhor sistema de cor é o que pega o melhor de cada um. Cron: segue para loop 09.*
