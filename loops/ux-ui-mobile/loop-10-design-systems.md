# UX/UI Loop 10 — Design Systems & Tokens: Como Bauhaus Virou Código

> **Data:** 2026-07-02
> **Loop:** 10 de ∞
> **Tema:** O design system é o estágio final da evolução que começou na Bauhaus. Gropius queria "unidade de todas as artes sob a arquitetura." O design system é isso: a unidade de todos os COMPONENTES sob um TOKEN. O que era filosofia virou JSON. O que era "total design" virou `--color-primary`. E a Jina Anne (Salesforce, 2014) está para o código como Gropius está para a forma.

---

## 0. A Linhagem Completa: Da Filosofia ao Código

```
Bauhaus "Total Design" (Gropius, 1919)        — Unidade de todas as artes
  → NASA Graphics Standards Manual (1976)      — Primeiro style guide moderno
    → Apple HIG (1978, Lisa/Macintosh)         — Primeiro guia de interface digital
      → Atomic Design (Brad Frost, 2013)       — Metodologia: átomos → moléculas → organismos
        → Salesforce Lightning (Jina Anne, 2014)— INVENTOU os "design tokens"
          → W3C DTCG Spec (2025)              — Padronização mundial dos tokens
            → TODO DESIGN SYSTEM MODERNO
```

10 anos de design tokens. 50 anos de sistemas de design. 106 anos de Bauhaus. Tudo converge aqui.

---

## 1. A Pré-História: Antes de Existir "Design System"

### 1.1 NASA Graphics Standards Manual (1976)
Richard Danne e Bruce Blackburn criaram o "NASA Worm" e o manual de 90 páginas que padronizava TUDO: tipografia (Helvetica), cores (vermelho NASA + azul), grid, letterhead, veículos, até a pintura dos foguetes.

**É o primeiro "design system" moderno.** Não era digital. Mas era um SISTEMA: regras, componentes, variações, aplicações. Tudo que um design system tem hoje.

### 1.2 Apple Human Interface Guidelines (1978-1984)
Bruce Tognazzini ("Tog") e a equipe da Apple criaram os primeiros HIG para o Lisa (1978, lançado 1983) e Macintosh (1984). Eles definiram:
- **Diálogos modais** (até então, não existia padrão)
- **Menus pull-down** (invenção da Apple)
- **Ícones como metáfora** (a lixeira, a pasta, o desktop)
- **"The user is never wrong"** — filosofia que virou dogma

**O Apple HIG é o ANCESTRAL de todo design system digital.** Material Design? Filho do HIG. Fluent? Filho do HIG. Todo design system mobile deve algo ao HIG.

---

## 2. Atomic Design (Brad Frost, 2013): A Metodologia

Brad Frost publicou o post "Atomic Design" em 2013. A ideia: interfaces são construídas como a MATÉRIA — de partículas pequenas a organismos complexos.

| Nível | Nome | Exemplo |
|---|---|---|
| **Átomos** | Elementos indivisíveis | Botão, input, label, cor, fonte |
| **Moléculas** | Grupos de átomos | Campo de busca (input + botão + label) |
| **Organismos** | Grupos de moléculas | Header (logo + navegação + busca) |
| **Templates** | Layouts com organismos | Grid de página com header, sidebar, conteúdo |
| **Páginas** | Instâncias reais | Página inicial com conteúdo REAL |

**No mobile:** o mesmo Atomic Design se aplica. Um botão (átomo) + um input (átomo) = um search bar (molécula) = um header de app (organismo) = uma tela de busca (template) = a busca do app com dados reais (página).

> Brad Frost (2024): "We've gone from atomic to subatomic. Design tokens are the SUBATOMIC layer — smaller than atoms."

---

## 3. Design Tokens (Salesforce, Jina Anne, 2014): A Invenção

**Jina Anne** e **Jon Levine**, na Salesforce, cunharam o termo "design tokens" em 2014 para o Lightning Design System. A definição original:

> "Design tokens are the visual design atoms of the design system — named entities that store visual design attributes. We use them in place of hard-coded values."

**Antes dos tokens:**
```css
button { background: #0066CC; } /* O que é #0066CC? Ninguém sabe. */
text    { color: #0066CC; }      /* É o mesmo azul? Precisa adivinhar. */
```

**Depois dos tokens:**
```css
button { background: var(--color-action-primary); }
text    { color: var(--color-text-link); }
```

**Por que tokens importam:**
1. **Um valor, um nome.** `#0066CC` vira `--color-blue-500`. Se o azul muda, muda em UM lugar.
2. **Semântica.** `--color-danger` diz O QUE a cor significa (perigo), não QUAL é o hex (#FF0000). Posso trocar o hex sem mudar o SIGNIFICADO.
3. **Multi-plataforma.** O MESMO token gera CSS, Swift, Kotlin, Dart.
4. **Theming.** `--color-primary` = azul (light mode) = azul-claro (dark mode) = laranja (high contrast).

---

## 4. A Arquitetura de 3 Camadas dos Tokens

| Camada | Também chamada | Pergunta | Exemplo |
|---|---|---|---|
| **Primitive** | Reference, Global | "O que existe?" | `blue-500: #2563EB` |
| **Semantic** | Alias, Purpose | "O que significa?" | `color-action-primary: {blue-500}` |
| **Component** | Specific | "Onde se usa?" | `button-bg-primary: {color-action-primary}` |

**Exemplo real (Material 3):**
```json
{
  "ref.palette.blue40": "#2563EB",           // Primitivo
  "md.sys.color.primary": "{ref.palette.blue40}", // Semântico
  "md.comp.fab.primary.container": "{md.sys.color.primary}" // Componente
}
```

**Por que 3 camadas?** Se você usa PRIMITIVOS direto no componente, não consegue fazer theming. Se você não tem PRIMITIVOS, não consegue mudar a paleta sem quebrar tudo. As 3 camadas são a diferença entre um design system que ESCALA e um que QUEBRA.

---

## 5. O Pipeline: Do Figma ao Código

```
Figma (designer)
  → Tokens Studio plugin (tokens em JSON, padrão DTCG)
    → Git (GitHub/GitLab)
      → Style Dictionary v4 (transformação)
        → CSS Custom Properties
        → Swift UIColor
        → Kotlin properties
        → Flutter Dart
      → CI/CD (valida, testa, publica)
        → App (consumo)
```

**Ferramentas principais:**

| Ferramenta | Função |
|---|---|
| **Figma Variables** (2023) | Tokens nativos dentro do Figma |
| **Tokens Studio** | Plugin: tokens → JSON → Git |
| **Style Dictionary v4** (Amazon, Danny Banks) | Transformador: JSON → CSS/Swift/Kotlin/etc |
| **Terrazzo** (2023) | Alternativa nativa DTCG ao Style Dictionary |
| **Storybook** | Documentação de componentes + tokens |

---

## 6. W3C Design Tokens Spec (2025): O Padrão Mundial

Em **28 de outubro de 2025**, o W3C Design Tokens Community Group (DTCG) publicou a primeira versão estável do **Design Tokens Format Module**. Pela primeira vez, existe um padrão MUNDIAL para tokens:

```json
{
  "color-blue-500": {
    "$value": "#2563EB",
    "$type": "color",
    "$description": "Primary blue, step 500"
  },
  "color-action-primary": {
    "$value": "{color-blue-500}",
    "$type": "color"
  }
}
```

- **`$value`**: o valor do token
- **`$type`**: o tipo (color, dimension, fontFamily, etc.)
- **`$description`**: documentação embutida
- **`$deprecated`**: deprecação com mensagem
- **`{alias}`**: referência a outro token via chaves

**Antes do W3C:** cada ferramenta tinha seu formato. Tokens do Figma ≠ tokens do Style Dictionary ≠ tokens do Material. Era uma Torre de Babel.

**Depois do W3C:** UM formato. Qualquer ferramenta que siga o padrão DTCG se comunica com qualquer outra.

---

## 7. Design Systems Famosos (Linha do Tempo)

| Ano | Sistema | Quem | Inovação |
|---|---|---|---|
| **1976** | NASA Graphics Manual | Danne & Blackburn | Primeiro style guide moderno |
| **1984** | Apple HIG | Tognazzini, Jobs | Primeiro guia de interface digital |
| **2006** | Yahoo! Design Pattern Library | Yahoo | Primeira pattern library web |
| **2011** | Bootstrap | Twitter (Mark Otto, Jacob Thornton) | Design system como código open source |
| **2014** | Material Design | Google | Design system multiplataforma |
| **2014** | Salesforce Lightning | Jina Anne | INVENTOU design tokens |
| **2015** | Shopify Polaris | Shopify | Design system para e-commerce |
| **2016** | Carbon (IBM) | IBM | Enterprise design system open source |
| **2017** | Fluent Design | Microsoft | Luz, profundidade, escala |
| **2019** | Base (Uber) | Uber | Design system para mobilidade |
| **2021** | Material You | Google | Dynamic color, tokens pessoais |
| **2023** | Figma Variables | Figma | Tokens nativos em ferramenta de design |
| **2025** | W3C DTCG Spec | W3C | Padrão mundial de tokens |

---

## 8. O Que o Design System Ensina ao Designer Mobile

1. **Não desenhe TELAS — desenhe SISTEMAS.** Cada componente deve ser reutilizável. Cada cor deve ter um token. Cada valor deve ter um NOME.

2. **Tokens são a DOCUMENTAÇÃO que funciona.** Um JSON de tokens É documentação. O desenvolvedor não precisa LER sua doc — ele CONSOME seus tokens.

3. **O design system É o produto.** O app muda. O design system fica. Invista no SISTEMA, não nas TELAS.

4. **Automatize ou morra.** Tokens manuais viram débito técnico. O pipeline (Figma → Git → Style Dictionary → código) precisa ser AUTOMÁTICO.

5. **Um token mal nomeado é pior que hex value.** `--color-blue` é um hex value com nome. `--color-action-primary` é um TOKEN de verdade.

---

## 9. Crítica

1. **Design systems podem ser PRISÕES.** "Não está no design system = não pode ser feito." A padronização vira burocracia.

2. **Tokens são a BUROCRATIZAÇÃO da cor.** Em vez de `#FF0000`, eu escrevo `var(--md-sys-color-error)`. Isso é melhor. Mas também é MAIS COMPLEXO. Para times pequenos, hex values direto no CSS podem ser mais RÁPIDOS.

3. **Atomic Design é uma metáfora — com limites.** Átomos não são moléculas. Mas um "botão" pode ser átomo OU molécula dependendo do contexto. A metáfora é ÚTIL, não CIENTÍFICA.

4. **O design system NÃO substitui o bom gosto.** O melhor design system do mundo não salva um produto mal pensado. Tokens não substituem JONY IVE.

---

## Referências

- Richard Danne & Bruce Blackburn — *NASA Graphics Standards Manual* (1976)
- Bruce Tognazzini — Apple Human Interface Guidelines (Lisa/Macintosh, 1978-1984)
- Brad Frost — *Atomic Design* (2013, livro 2016)
- Jina Anne & Jon Levine — Salesforce Lightning Design System, "design tokens" (2014)
- Danny Banks — Style Dictionary v4 (Amazon, 2024)
- W3C Design Tokens Community Group — *Design Tokens Format Module* (2025-10-28)
- Material Design 3, Carbon (IBM), Polaris (Shopify), Fluent (Microsoft)

---

*Fim do Loop UX/UI 10. Design tokens: Bauhaus virou JSON. A filosofia virou código. Cron edf71e22 segue.*
