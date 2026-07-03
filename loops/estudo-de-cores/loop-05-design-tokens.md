# 🎨 Estudo de Cores Loop 05 — Design Tokens: Como Estruturar Cor Em Escala (3 Camadas, Nomenclatura, Pipeline)

> **Data:** 2026-07-02
> **Loop:** 5 de ∞
> **Tema:** A arquitetura de cores em design systems. As 3 camadas de tokens (Primitive → Semantic → Component). Como NOMEAR cores para durar 10 anos. Style Dictionary como "compilador" de cor. O pipeline Figma → código que sincroniza design com engenharia. E como Stripe, Linear, Vercel e GitHub estruturam seus tokens de cor.

---

## 0. O Problema: `--color-blue-500` É Uma Decisão Que Você Vai Se Arrepender Por 10 Anos

Você está começando um design system. A primeira coisa que você faz:

```css
--color-blue-500: #3B82F6;
--color-red-500: #EF4444;
--color-green-500: #22C55E;
```

**Isso é uma bomba-relógio atômica de débito técnico.** Porque daqui a 2 anos:

1. Você faz um rebrand. Azul vira roxo. `--color-blue-500` agora é ROXO.
2. Você adiciona dark mode. `--color-blue-500` precisa ser uma cor DIFERENTE no escuro.
3. Você expande para 3 plataformas. Agora precisa do mesmo token em Swift `Color.blue500`, Kotlin `Blue500`, e CSS `--color-blue-500`.
4. Um PM diz: "o botão de sucesso precisa ser mais vibrante que o badge de sucesso." Mas ambos usam `--color-green-500`.

**Em 2 anos, seus tokens viraram uma TEIA de inconsistências que ninguém entende.**

Design tokens BEM FEITOS resolvem isso. Mas tokens MAL FEITOS são piores que tokens nenhum.

---

## 1. As 3 Camadas: A Arquitetura Que Salva Design Systems

### 1.1 O Modelo (Salesforce → Material → W3C)

```
CAMADA 3: COMPONENT TOKENS
  "Como a cor é USADA em componentes específicos"
  Ex: --button-primary-bg, --card-header-text

  CAMADA 2: SEMANTIC TOKENS
    "O que a cor SIGNIFICA no sistema"
    Ex: --color-primary, --color-danger, --text-on-primary

    CAMADA 1: PRIMITIVE TOKENS
      "O que a cor É no mundo real (perceptual)"
      Ex: --palette-blue-50, --palette-amber-400
```

**Regra de OURO:** componentes NUNCA referenciam primitives. Só referenciam semantics. Semantics referenciam primitives. Isso é uma DEPENDÊNCIA UNIDIRECIONAL.

### 1.2 Exemplo Concreto

```json
{
  "palette": {
    "blue": {
      "50":  { "$value": "oklch(0.95 0.03 260)" },
      "400": { "$value": "oklch(0.60 0.18 260)" },
      "700": { "$value": "oklch(0.35 0.15 260)" }
    }
  },
  "color": {
    "primary":         { "$value": "{palette.blue.400}" },
    "primary-hover":   { "$value": "{palette.blue.700}" },
    "primary-disabled": { "$value": "{palette.blue.50}" },
    "on-primary":      { "$value": "{palette.white}" }
  },
  "button": {
    "primary": {
      "background":   { "$value": "{color.primary}" },
      "text":         { "$value": "{color.on-primary}" },
      "hover-bg":     { "$value": "{color.primary-hover}" },
      "disabled-bg":  { "$value": "{color.primary-disabled}" }
    }
  }
}
```

**Agora quando você faz rebrand de azul para roxo:**

```json
"palette": {
  "blue": {  // NÃO MUDA O NOME! (ver seção 2)
    "400": { "$value": "oklch(0.60 0.18 290)" }  // muda SÓ o valor
  }
}
```

Tudo que referencia `{palette.blue.400}` automaticamente herda a nova cor. Tokens de componente (`button.primary.background`) não mudam porque referenciam camada SEMÂNTICA, não o valor cru.

### 1.3 O Que Acontece Se Você Não Tiver As 3 Camadas

**Só 1 camada (primitives direto nos componentes):**

```css
--btn-primary-bg: #3B82F6;  /* azul */
```

Para fazer rebrand, você precisa buscar e substituir `#3B82F6` no código INTEIRO. Incluindo lugares onde ele NÃO é um botão mas por acaso usa o mesmo azul. Pesadelo.

**2 camadas sem components:**

```css
--color-primary: #3B82F6;

/* Componentes referenciam direto: */
.button { background: var(--color-primary); }
.badge { background: var(--color-primary); }
.link { color: var(--color-primary); }
```

Quando o PM diz "badge precisa ser 10% mais claro que o botão," você não tem ONDE colocar essa diferença sem criar um token novo (e reescrever tudo).

**3 camadas completo:**

```css
--palette-blue-400: #3B82F6;
--color-primary: var(--palette-blue-400);
--button-primary-bg: var(--color-primary);
--badge-primary-bg: oklch(from var(--color-primary) calc(l + 0.1) c h);
```

PM: "Badge mais claro." Você: `calc(l + 0.1)` no token do badge. Resolvido.

---

## 2. Nomenclatura: O Nome Que Sobrevive A Rebrands, CEOs E Aquisições

### 2.1 A Regra Fundamental

**Nunca nomeie tokens pelo VALOR. Nomeie pelo SIGNIFICADO.**

| ❌ Nome pelo valor (morre no rebrand) | ✅ Nome pelo significado (sobrevive) |
|---|---|
| `--color-blue-500` | `--color-primary` |
| `--color-red` | `--color-danger` |
| `--color-green` | `--color-success` |
| `--color-yellow` | `--color-warning` |
| `--color-white` | `--color-surface` ou `--color-bg` |
| `--color-gray-200` | `--color-border` |
| `--color-black` | `--color-text-primary` |

### 2.2 A Exceção: Paleta Base (Camada 1)

Na camada de PRIMITIVES, nomear pela cor É aceitável — porque é a camada mais baixa, que muda MENOS, e descreve o que a cor É (não o que ela SIGNIFICA):

```json
{
  "palette": {
    "blue": { "400": { "$value": "oklch(0.60 0.18 260)" } },
    "amber": { "400": { "$value": "oklch(0.60 0.18 85)" } },
    "neutral": { "200": { "$value": "oklch(0.85 0 0)" } }
  }
}
```

**Mas CUIDADO:** se você nomear `palette.blue.400` e depois fizer rebrand para uma paleta roxa, você tem `palette.blue.400` contendo roxo. Isso é FEIO mas TOLERÁVEL porque ninguém referencia essa camada diretamente. Os SEMANTIC tokens (`color.primary`) escondem a feiura.

**Alternativa:** usar índices abstratos na camada palette:

```json
{
  "palette": {
    "brand": {
      "50": { "$value": "oklch(0.95 0.03 260)" },
      "400": { "$value": "oklch(0.60 0.18 260)" },
      "700": { "$value": "oklch(0.35 0.15 260)" }
    },
    "accent": {
      "400": { "$value": "oklch(0.60 0.18 85)" }
    }
  }
}
```

Assim, `palette.brand.400` sobrevive a qualquer rebrand porque "brand" não contém o nome da cor.

### 2.3 A Convenção CTI (Category-Type-Item)

O W3C Design Tokens Community Group recomenda:

```
namespace.category.type.modifier.state
```

Exemplos:
```
color.bg.primary
color.text.danger.hover
color.border.focus
color.icon.brand
spacing.gap.large
typography.heading.h1.size
```

**Para cor, uma taxonomia robusta:**

```
[CATEGORY].[TARGET].[ROLE].[STATE?]
```

| Category | Target | Role | State |
|---|---|---|---|
| `color` | `bg` (background) | `primary` | `hover` |
| `color` | `text` | `secondary` | `active` |
| `color` | `border` | `danger` | `focus` |
| `color` | `icon` | `brand` | `disabled` |

### 2.4 Anti-Padrões de Nomenclatura

| Anti-Padrão | Exemplo | Problema |
|---|---|---|
| **Cor no nome** | `--color-blue` | Morre no rebrand |
| **Número sem contexto** | `--color-1`, `--color-2` | Ininteligível em 6 meses |
| **Nível de abstração errado** | `--color-header-button-primary` | Muito específico, não reutilizável |
| **Valor no nome** | `--color-3B82F6` | Morre em QUALQUER ajuste |
| **Nome fofo** | `--color-sunshine`, `--color-ocean` | Ambigo. O que é "ocean"? Azul? Verde? |
| **Sigla misteriosa** | `--clr-prm`, `--clr-scd` | Economiza 3 caracteres, custa 3 horas de onboarding |

---

## 3. Style Dictionary: O "Compilador" De Cor

### 3.1 O Que É

**Style Dictionary** (Amazon, 2017) é um sistema de build para design tokens. Você define tokens UMA VEZ (em JSON/YAML) e ele gera saída para múltiplas plataformas:

```
tokens/color.json
tokens/spacing.json
tokens/typography.json
        ↓
  Style Dictionary
        ↓
    ┌───┴────┬──────────┬──────────┐
  CSS     Swift     Kotlin     SCSS
```

### 3.2 Configuração Mínima

```javascript
// config.js
module.exports = {
  source: ['tokens/**/*.json'],
  platforms: {
    css: {
      transformGroup: 'css',
      buildPath: 'build/css/',
      files: [{
        destination: 'variables.css',
        format: 'css/variables',
      }],
    },
    swift: {
      transformGroup: 'swift',
      buildPath: 'build/ios/',
      files: [{
        destination: 'DesignTokens.swift',
        format: 'ios-swift/class.swift',
      }],
    },
    kotlin: {
      transformGroup: 'compose',
      buildPath: 'build/android/',
      files: [{
        destination: 'Colors.kt',
        format: 'compose/object',
      }],
    },
  },
};
```

### 3.3 Entrada (JSON com OKLCH)

```json
{
  "color": {
    "bg": {
      "primary": { "$value": "oklch(0.98 0.005 260)" },
      "secondary": { "$value": "oklch(0.95 0.01 260)" }
    },
    "text": {
      "primary": { "$value": "oklch(0.12 0.005 260)" },
      "secondary": { "$value": "oklch(0.42 0.002 260)" }
    },
    "brand": {
      "primary": {
        "default": { "$value": "oklch(0.55 0.20 260)" },
        "hover": { "$value": "oklch(0.48 0.22 260)" },
        "disabled": { "$value": "oklch(0.85 0.08 260)" }
      }
    }
  }
}
```

### 3.4 Saída Multi-Plataforma

**CSS (Custom Properties):**
```css
:root {
  --color-bg-primary: oklch(0.98 0.005 260);
  --color-bg-secondary: oklch(0.95 0.01 260);
  --color-text-primary: oklch(0.12 0.005 260);
  --color-text-secondary: oklch(0.42 0.002 260);
  --color-brand-primary-default: oklch(0.55 0.20 260);
  --color-brand-primary-hover: oklch(0.48 0.22 260);
  --color-brand-primary-disabled: oklch(0.85 0.08 260);
}
```

**Swift (iOS):**
```swift
import SwiftUI

extension Color {
    static let bgPrimary = Color(oklchL: 0.98, c: 0.005, h: 260)
    static let bgSecondary = Color(oklchL: 0.95, c: 0.01, h: 260)
    static let textPrimary = Color(oklchL: 0.12, c: 0.005, h: 260)
    static let textSecondary = Color(oklchL: 0.42, c: 0.002, h: 260)
    static let brandPrimary = Color(oklchL: 0.55, c: 0.20, h: 260)
    static let brandPrimaryHover = Color(oklchL: 0.48, c: 0.22, h: 260)
    static let brandPrimaryDisabled = Color(oklchL: 0.85, c: 0.08, h: 260)
}
```

**Kotlin (Jetpack Compose / Android):**
```kotlin
object AppColors {
    val BgPrimary = Color.Oklch(0.98f, 0.005f, 260f)
    val BgSecondary = Color.Oklch(0.95f, 0.01f, 260f)
    val TextPrimary = Color.Oklch(0.12f, 0.005f, 260f)
    val TextSecondary = Color.Oklch(0.42f, 0.002f, 260f)
    val BrandPrimary = Color.Oklch(0.55f, 0.20f, 260f)
    val BrandPrimaryHover = Color.Oklch(0.48f, 0.22f, 260f)
    val BrandPrimaryDisabled = Color.Oklch(0.85f, 0.08f, 260f)
}
```

**UM arquivo JSON → 3 plataformas. Uma verdade.**

### 3.5 Transforms: Como Tokens São Convertidos

Style Dictionary tem uma pipeline de transformação:

```
Raw JSON → Transform 1 (resolve aliases) → Transform 2 (math) → Transform 3 (platform format) → Output
```

**Transforms úteis para cor:**

```javascript
// Transform custom: converter OKLCH → hex para navegadores antigos
StyleDictionary.registerTransform({
  name: 'oklch/css-fallback',
  type: 'value',
  transitive: true,
  matcher: (token) => token.value.startsWith('oklch('),
  transformer: (token) => {
    const oklch = parseOklch(token.value);
    const srgb = oklchToSrgb(oklch);
    return `rgb(${srgb.r} ${srgb.g} ${srgb.b})`;
  },
});

// Gera CSS com fallback automático:
// --color-brand: rgb(59 130 246);  ← fallback
// --color-brand: oklch(0.55 0.20 260);  ← moderno
```

---

## 4. O Pipeline Figma → Código (E De Volta)

### 4.1 O Sonho

```
Figma (designer arrasta cor)
    ↕ sync bidirecional
Tokens JSON (verdade canônica)
    ↓ Style Dictionary
Código iOS / Android / Web / React Native
```

### 4.2 A Realidade (2026)

**Figma → Código** funciona BEM com plugins como **Tokens Studio**:

1. Designers definem tokens NO FIGMA (usando Tokens Studio)
2. Exportam como JSON para o repositório
3. Style Dictionary compila para código
4. CI/CD valida que os tokens não quebraram nada

**Código → Figma** ainda é DIFÍCIL:

1. Engenheiros mudam um token no código
2. Precisam manualmente atualizar o Figma
3. Ou: CI gera um JSON "de volta" que o Tokens Studio importa

**O gargalo é SEMÂNTICA:** o Figma não entende que `--color-primary` e `md.sys.color.primary` são o MESMO token. Cada ferramenta tem seu próprio namespace.

### 4.3 O Fluxo Recomendado (2026)

```
1. REPOSITÓRIO GIT é a FONTE DA VERDADE
   tokens/
   ├── primitives/color.json
   ├── semantics/color.json
   └── components/button.json

2. FIGMA lê do repositório (Tokens Studio conectado ao GitHub)
   Designer NÃO digita hex codes. Só referencia tokens.

3. STYLE DICTIONARY compila tokens → código
   Gera CSS, Swift, Kotlin, SCSS, Tailwind config

4. CI/CD VALIDA:
   a) Tokens não têm referências cíclicas
   b) APCA contrastes mínimos
   c) Nenhum componente referencia primitives diretamente
   d) Tokens novos não quebram delta E > 3 de cores existentes

5. STORYBOOK consome tokens → renderiza componentes COM CORES REAIS
   Design review: componente usa token SEMÂNTICO, não hardcoded
```

---

## 5. Estados de Cor: Hover, Active, Disabled, Focus

### 5.1 As 3 Abordagens

**Abordagem A: Tokens EXPLÍCITOS para cada estado (mais comum)**

```json
{
  "button": {
    "primary": {
      "bg": { "$value": "{color.brand.primary}" },
      "bg-hover": { "$value": "{color.brand.primary-hover}" },
      "bg-active": { "$value": "{color.brand.primary-active}" },
      "bg-disabled": { "$value": "{color.brand.primary-disabled}" },
      "bg-focus": { "$value": "{color.brand.primary}" }
    }
  }
}
```

✅ Controle total. ❌ Verboso. 5 tokens por estado × N componentes = explosão combinatória.

**Abordagem B: Tokens de estado GENÉRICOS + composição**

```json
{
  "state": {
    "hover": { "$value": "lightness-offset: +0.05" },
    "active": { "$value": "lightness-offset: -0.03" },
    "disabled": { "$value": "opacity: 0.38" }
  }
}
```

✅ Compacto. ❌ Perde controle fino (hover de botão danger ≠ hover de link).

**Abordagem C: CSS `color-mix()` + `oklch(from ...)` (2026+)**

```css
--btn-primary-bg: var(--color-brand-primary);
--btn-primary-hover: oklch(from var(--btn-primary-bg) calc(l + 0.05) c h);
--btn-primary-active: oklch(from var(--btn-primary-bg) calc(l - 0.03) calc(c * 1.1) h);
--btn-primary-disabled: oklch(from var(--btn-primary-bg) l calc(c * 0.3) h);
```

✅ Matemático, sem tokens extras. ✅ Mudanças na cor base propagam automaticamente. ❌ Não funciona em iOS/Android nativo (ainda). **Híbrido:** use relative color syntax na web, tokens explícitos no mobile.

### 5.2 A Escala de Estados em OKLCH

```javascript
function stateVariants(baseColor) {
  return {
    default:  baseColor,
    hover:    { ...baseColor, L: baseColor.L + 0.05, C: baseColor.C * 1.00 },
    active:   { ...baseColor, L: baseColor.L - 0.03, C: baseColor.C * 1.05 },
    disabled: { ...baseColor, L: baseColor.L + 0.20, C: baseColor.C * 0.30 },
    focus:    baseColor,  // cor igual, outline adicionado
  };
}
```

**Regra prática:** hover = +5 lightness, active = −3 lightness +5% chroma, disabled = +20 lightness −70% chroma.

---

## 6. Estudos de Caso: Como Os Grandes Estruturam Cor

### 6.1 Stripe

Stripe usa uma abordagem MINIMALISTA:

- **~15 tokens de cor** no total (absurdamente poucos para uma empresa do tamanho deles)
- Paleta monocromática (azul Stripe + cinzas)
- Estados gerados por opacidade (`rgba(primary, 0.1)`) em vez de tokens explícitos
- Design system pequeno e focado: "menos tokens = menos decisões = mais consistência"

**Lição:** você NÃO PRECISA de 500 tokens de cor. Stripe opera bilhões de dólares com ~15.

### 6.2 Linear

Linear usa uma abordagem GERADA:

- Paleta baseada em OKLCH (eles foram early adopters)
- Cores geradas algoritmicamente a partir de um matiz base
- Estados: escala de lightness (não chroma) — hover = +10 lightness
- Dark mode por padrão (light mode é o alternativo)
- Tokens expostos como CSS variables + React Context + Tailwind

**Lição:** se sua paleta é GERADA (loop 04), seus tokens podem ser GERADOS também. Menos trabalho manual.

### 6.3 Vercel

Vercel usa uma abordagem geométrica:

- Sistema de cores baseado em "shades" (050-950 como Tailwind)
- Cada shade é gerado por uma curva perceptual
- Tokens SEMÂNTICOS para foreground/background (não cores nomeadas)
- `--ds-gray-100`, `--ds-blue-600` como primitives
- `--ds-background-100`, `--ds-text-primary` como semantics
- Suporte a P3 (wide gamut) com fallback sRGB

**Lição:** a separação foreground/background é mais flexível que "cor primária/secundária" porque FUNCIONA em dark mode automaticamente.

### 6.4 GitHub Primer

GitHub Primer é um dos design systems mais MADUROS em tokens:

- **Open source.** Você pode LER todos os tokens.
- 3 camadas claras: `primitives/` → `semantic/` → `components/`
- Suporte a 3 modos: light, dark, high-contrast
- Cada cor existe nos 3 modos, definida explicitamente
- Migração de LESS → CSS variables → Style Dictionary (já passaram por 3 sistemas)

**Lição:** tokens EVOLUEM. Seu sistema de tokens de 2026 vai ser reescrito em 2030. Planeje a migração desde o dia 1.

---

## 7. Token Versioning: Como Renomear Sem Quebrar O Mundo

### 7.1 O Problema

Você tem 200 componentes em produção referenciando `--color-primary`. Você quer renomear para `--color-brand-primary`. Se você só mudar o nome, 200 componentes quebram.

### 7.2 A Estratégia de Migração

**Fase 1: Deprecation (mês 1-2)**
```css
--color-primary: var(--color-brand-primary);  /* alias de compatibilidade */
--color-brand-primary: oklch(0.55 0.20 260);  /* novo token canônico */
```

**Fase 2: Warning (mês 2-4)**
CI/CD emite WARNING quando detecta `--color-primary` em código novo. Bloqueia em PRs.

**Fase 3: Migration (mês 4-6)**
Script automático (codemod) substitui todas as ocorrências. PR gigante reviewado.

**Fase 4: Removal (mês 6+)**
Remove o alias. `--color-primary` causa ERRO de build.

### 7.3 Aliasing Como Ferramenta de Transição

```json
{
  "color": {
    "primary": {
      "$value": "{color.brand.primary}",
      "$deprecated": true,
      "$deprecatedMessage": "Use color.brand.primary",
      "$deprecatedSince": "2026-03-15"
    },
    "brand": {
      "primary": { "$value": "oklch(0.55 0.20 260)" }
    }
  }
}
```

Style Dictionary pode ser configurado para:
- Emitir comentários de depreciação no CSS gerado
- Log warnings no build
- Quebrar o build após uma data cutoff

---

## 8. O Futuro: W3C Design Tokens Format

### 8.1 O Padrão Emergente

O W3C Design Tokens Community Group está padronizando um formato UNIVERSAL de tokens. Quando adotado:

```json
{
  "color-bg-primary": {
    "$type": "color",
    "$value": "oklch(0.98 0.005 260)",
    "$extensions": {
      "com.example.figma": { "styleId": "S:abc123" },
      "com.example.ios": { "colorAssetName": "BgPrimary" }
    }
  }
}
```

- `$type`: validação de tipo (color, dimension, fontFamily, etc.)
- `$value`: o valor canônico em uma unidade padrão
- `$extensions`: metadados específicos de ferramenta (Figma ID, iOS asset name, etc.)

**Isso resolve o problema de interoperabilidade.** Uma ferramenta não precisa entender os metadados de outra — só ignora as extensions que não conhece.

### 8.2 O Que Isso Significa Para Cor

Quando o padrão for adotado universalmente:

1. **Figma exporta tokens no formato W3C** (não JSON proprietário)
2. **Style Dictionary lê o formato W3C** (sem adapters custom)
3. **Xcode, Android Studio, VS Code** leem o formato W3C nativamente
4. **Design e código compartilham o MESMO ARQUIVO** — literalmente

**Até lá:** Style Dictionary com plugins de conversão. É o que funciona HOJE.

---

## 9. Checklist: Seu Sistema De Tokens De Cor Está Pronto?

- [ ] **3 camadas?** Primitives → Semantics → Components. Dependência unidirecional.
- [ ] **Nomes semânticos?** `--color-danger`, não `--color-red`.
- [ ] **Aliases?** Tokens referenciam outros tokens (não valores crus na camada 3).
- [ ] **OKLCH nos primitives?** Perceptual, multi-plataforma, pronto para dark mode.
- [ ] **Estados definidos?** Hover, active, disabled, focus para cada cor interativa.
- [ ] **Dark mode?** Tokens com variantes ou `light-dark()`.
- [ ] **Build pipeline?** Style Dictionary (ou similar) gerando CSS/Swift/Kotlin.
- [ ] **CI/CD valida?** APCA, referências cíclicas, delta E.
- [ ] **Deprecação documentada?** Plano para renomear tokens sem quebrar consumidores.
- [ ] **Figma ↔ código sync?** Tokens Studio ou bridge similar.

---

## Referências

- Amazon — *Style Dictionary* (2017-2025). A ferramenta de build de tokens.
- W3C — *Design Tokens Community Group* (2021-2026). Especificação do formato padrão.
- Jina Anne — *Design Tokens* (Salesforce, 2014). A primeira palestra sobre tokens.
- Material Design 3 — *Design Tokens* (2021-2025). Documentação pública dos tokens do Google.
- GitHub — *Primer Design System* (open source). Tokens completos em 3 camadas.
- Stripe — *Design System* (documentação interna, princípios públicos).
- Linear — *Linear Design* (documentação pública). Early adopters de OKLCH.
- Vercel — *Geist Design System* (2024-2025). Tokens com suporte P3.
- Tokens Studio — *Figma Plugin*. Sincronização Figma ↔ GitHub.
- Danny Banks — *Style Dictionary* (2017). O paper original do conceito.

---

*Fim do Estudo de Cores Loop 05. Tokens são a INFRAESTRUTURA da cor em escala. 3 camadas. Nomes semânticos. Style Dictionary. Pipeline Figma↔código. A cor não é um valor hex — é um SISTEMA de referências que precisa sobreviver a rebrands, plataformas e décadas. Cron: segue para loop 06.*
