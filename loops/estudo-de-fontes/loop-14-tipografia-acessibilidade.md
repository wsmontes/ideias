# 🔤 Estudo de Fontes Loop 14 — Tipografia & Acessibilidade: Como Projetar Texto Que Todo Mundo Pode Ler

> **Data:** 2026-07-02
> **Loop:** 14 de ∞
> **Tema:** WCAG, contraste, dislexia, fontes acessíveis, e como projetar tipografia que funciona para pessoas com baixa visão, daltonismo, dislexia, e envelhecimento ocular. Acessibilidade NÃO é "compliance" — é DESIGN.

---

## 0. A Tese: 15% da População Mundial Tem Alguma Deficiência. Seu Texto Precisa Funcionar Para ELES.

**1 bilhão de pessoas** têm alguma forma de deficiência. Isso inclui:
- **Baixa visão** (miopia, catarata, degeneração macular) — 2.2 bilhões
- **Daltonismo** (afeta ~8% dos homens, ~0.5% das mulheres) — 350 milhões
- **Dislexia** (dificuldade de leitura) — 10-20% da população
- **Envelhecimento ocular** (presbiopia depois dos 40) — essencialmente TODO MUNDO

Se seu texto não funciona para essas pessoas, você está EXCLUINDO 1 bilhão de usuários. E possivelmente VIOLANDO a lei.

**Accessibilidade tipográfica não é um "extra." É o BASELINE do design profissional em 2025.**

---

## 1. WCAG: A Lei da Tipografia Acessível

### 1.1 Os Critérios Que Importam

| Critério WCAG | Exigência | Nível |
|---|---|---|
| **1.4.3 Contraste (Mínimo)** | Normal (<18pt): ≥4.5:1. Grande (≥18pt ou ≥14pt bold): ≥3:1 | AA |
| **1.4.6 Contraste (Aprimorado)** | Normal: ≥7:1. Grande: ≥4.5:1 | AAA |
| **1.4.12 Text Spacing** | Line height ≥1.5×, letter spacing ≥0.12em, word spacing ≥0.16em, paragraph ≥2× | AA |
| **1.4.4 Resize Text** | Conteúdo funcional a 200% zoom | AA |
| **1.4.8 Visual Presentation** | Linha ≤80 caracteres, sem justified, cores selecionáveis | AAA |
| **1.4.1 Use of Color** | Cor NUNCA como único meio de transmitir informação | A |

### 1.2 O Cenário Legal em 2025

| Jurisdição | Regulamento | Padrão | Prazo |
|---|---|---|---|
| **EUA** | ADA Title II | WCAG 2.1 AA | Abril 2027 (50K+ pop.) |
| **UE** | European Accessibility Act | EN 301 549 (WCAG 2.1 AA) | **Em vigor desde 28 Jun 2025** |
| **Canadá** | Accessible Canada Act | WCAG 2.1 AA | Em andamento |

**Precedente jurídico:** *Domino's Pizza v. Robles (2019)* — Domino's perdeu. Fonte de 12px e baixo contraste foram citados como barreiras de acesso. A Suprema Corte dos EUA RECUSOU ouvir a apelação.

**Acessibilidade não é opcional. É obrigação legal.**

---

## 2. Contraste: O Requisito Mais Básico (E Mais Violado)

### 2.1 A Matemática

```
Contraste = (L1 + 0.05) / (L2 + 0.05)

Onde L1 = luminância relativa da cor mais clara
      L2 = luminância relativa da cor mais escura
```

| Combinação | Ratio | Passa AA? |
|---|---|---|
| **#000 sobre #FFF** | 21:1 | ✅ AAA |
| **#333 sobre #FFF** | 12.6:1 | ✅ AAA |
| **#666 sobre #FFF** | 5.7:1 | ✅ AA |
| **#767676 sobre #FFF** | 4.5:1 | ✅ AA (raspando) |
| **#888 sobre #FFF** | 3.5:1 | ❌ FALHA |
| **#CCC sobre #FFF** | 1.6:1 | ❌ FALHA |

### 2.2 O Truque do "Cinza Design"

O cinza #888 parece "elegante" em texto secundário. Mas ele FALHA no WCAG AA. A maioria dos "cinzas de design" são INACESSÍVEIS.

**Solução:** para texto secundário acessível, o cinza mais claro em fundo branco é #767676. Abaixo disso, só para texto GRANDE (≥18pt bold).

### 2.3 O Efeito "Fonte Fina"

Mesmo atingindo 4.5:1, **fontes finas (weight 300 ou menos)** podem ser ilegíveis na prática. O olho humano precisa de MASSA para perceber contraste. Uma linha de 1 pixel de espessura com contraste 4.5:1 AINDA É uma linha de 1 pixel — invisível para muita gente.

**Regra:** se o texto é importante e pequeno, use **weight ≥400 (Regular)**. Se possível, **500-600 (Medium/SemiBold)**.

### 2.4 Ferramentas de Contraste

| Ferramenta | Plataforma |
|---|---|
| **[WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)** | Web |
| **Stark** | Figma, Sketch, Adobe XD, Browser |
| **Able** | Figma |
| **Chrome DevTools** | CSS panel → contrast ratio |
| **Sim Daltonism** | macOS (simula daltonismo) |

---

## 3. Dislexia: O Que REALMENTE Funciona

### 3.1 O Mito das "Fontes de Dislexia"

**OpenDyslexic**, **Dyslexie**, **Read Regular** — fontes desenhadas "especificamente para dislexia" — têm POUCA evidência científica de eficácia. Vários estudos (2024-2025) mostram que **não há benefício mensurável** de leitura com essas fontes comparado a fontes padrão bem desenhadas.

O consenso científico atual: **dislexia é primariamente um problema de PROCESSAMENTO FONOLÓGICO** (como o cérebro mapeia sons em letras), não um problema VISUAL. Nenhuma mudança de tipo resolve isso.

### 3.2 O Que REALMENTE Ajuda Leitores Disléxicos

| Prática | Evidência |
|---|---|
| **Sans-serif** (Arial, Verdana, Inter) | FORTE — reduz clutter visual |
| **Aberturas grandes** (c, e, a "respiram") | FORTE — letras mais distintas |
| **Distinção clara** entre b/d, p/q, l/1/I | FORTE — evita confusão de caracteres |
| **Peso ≥400** (nada de Thin/Light) | MODERADA — strokes finos "somem" |
| **Line height ≥1.5** | FORTE — reduz crowding visual |
| **Linhas de 45-90 caracteres** | FORTE — facilita tracking horizontal |
| **SEM justified** (alinhado à esquerda) | FORTE — espaçamento uniforme entre palavras |
| **Bold em vez de itálico** para ênfase | FORTE — itálico distorce a forma das letras |
| **SEM All Caps em texto longo** | FORTE — all caps remove ascending/descending cues |
| **Fundo sem textura/imagem** | MODERADA — reduz ruído visual |

### 3.3 Fontes Recomendadas Para Acessibilidade

| Fonte | Pontos Fortes |
|---|---|
| **Atkinson Hyperlegible** | Desenhada pelo Braille Institute para baixa visão. Caracteres NUNCA espelhados (b ≠ d). **A melhor fonte para acessibilidade geral em 2025.** |
| **Lexend** | Desenhada para reduzir estresse visual. 8 larguras. Boa legibilidade. |
| **Verdana** | Desenhada por Matthew Carter para tela. Aberturas generosas. Distinção excelente. |
| **Tahoma** | Similar à Verdana, mais compacta. |
| **Arial** | Onipresente. Funciona. Não é bonita, mas é LEGÍVEL. |
| **Inter** | x-height alta, aberturas grandes, 9 pesos. Excelente para UI. |
| **Open Sans** | Humanista, quente, boa distinção de caracteres. |
| **Comic Sans** | Surpreendentemente eficaz para alguns disléxicos (formas irregulares são mais "memoráveis"). |

### 3.4 A Regra Mais Importante: Deixe o Usuário Escolher

A MELHOR prática de acessibilidade: **permita que o usuário troque a fonte.** Um toggle "Fonte Acessível" que muda para Atkinson Hyperlegible ou Verdana é mais útil do que qualquer "fonte mágica" imposta.

```css
body { font-family: 'Inter', system-ui, sans-serif; }
body.accessible { font-family: 'Atkinson Hyperlegible', 'Verdana', sans-serif; }
```

---

## 4. Text Spacing: O WCAG 1.4.12

### 4.1 O Que Exige

Usuários devem poder aplicar ESTES overrides SEM quebrar o layout:

```css
* {
  line-height: 1.5 !important;
  letter-spacing: 0.12em !important;
  word-spacing: 0.16em !important;
}
p {
  margin-bottom: 2em !important;
}
```

**Isso é o que um usuário com baixa visão aplica via extensão de browser. Seu layout NÃO PODE quebrar.**

### 4.2 Como Construir Para Isso

- Use `rem`/`em` para line-height e spacing (NÃO `px`)
- NUNCA use `height` fixa em containers de texto — use `min-height`
- NUNCA use `overflow: hidden` em containers de texto
- Use Flexbox/Grid — eles crescem com o conteúdo

```css
/* ❌ Vai quebrar com text spacing override */
.text-box {
  height: 60px;
  overflow: hidden;
}

/* ✅ Sobrevive a text spacing override */
.text-box {
  min-height: 60px;
}
```

---

## 5. Tamanho de Fonte: O Mínimo É 16px

### 5.1 A Regra

WCAG NÃO define tamanho mínimo. Mas o consenso da indústria e precedentes legais dizem:

| Contexto | Mínimo |
|---|---|
| **Body text (web)** | 16px |
| **Body text (mobile)** | 14-16px |
| **Caption / meta** | 12-13px (com contraste ≥4.5:1) |
| **Documentos Word/PDF** | 12pt (≈16px) |
| **Slides** | 18-24pt |

### 5.2 O Precedente Domino's

Em 2019, um homem cego processou a Domino's Pizza porque não conseguia pedir pizza no site/app. A fonte era 12px com baixo contraste. A Domino's PERDEU. A Suprema Corte dos EUA recusou a apelação.

**12px body text + baixo contraste = PROCESSO PERDIDO.**

---

## 6. Daltonismo: Cor Não É Informação

### 6.1 Os Tipos

| Tipo | Afeta | Confunde |
|---|---|---|
| **Deuteranopia** (verde fraco) | ~5% homens | Verde ↔ Vermelho |
| **Protanopia** (vermelho fraco) | ~2% homens | Vermelho ↔ Verde |
| **Tritanopia** (azul fraco) | Raro | Azul ↔ Amarelo |

### 6.2 A Regra

**Cor NUNCA deve ser o ÚNICO meio de transmitir informação.**

```html
<!-- ❌ Só cor -->
<span style="color: red">Erro: email inválido</span>

<!-- ✅ Cor + ícone + texto -->
<span style="color: red">⚠️ Erro: email inválido</span>
```

**Links** devem ser sublinhados (não só coloridos). **Gráficos** devem usar padrões + cores (não só cores). **Estados de erro/sucesso** devem usar ícone + cor + texto.

---

## 7. O Checklist de Acessibilidade Tipográfica

| # | Critério | ✅ |
|---|---|---|
| 1 | Body text ≥ **16px** | |
| 2 | Contraste ≥ **4.5:1** para texto normal | |
| 3 | Contraste ≥ **3:1** para texto grande (≥18pt) | |
| 4 | Line height ≥ **1.5×** no body | |
| 5 | Letter spacing ≥ **0.12em** possível sem quebrar layout | |
| 6 | Linhas de **45-90 caracteres** (max-width: 65ch) | |
| 7 | Alinhado à **esquerda** (não justified) | |
| 8 | **Bold** para ênfase (não itálico) | |
| 9 | **Sans-serif** para body text | |
| 10 | Distinção clara entre **l/1/I, 0/O, b/d, p/q** | |
| 11 | Conteúdo legível a **200% zoom** | |
| 12 | Cor NUNCA como único indicador | |
| 13 | Links **sublinhados** | |
| 14 | Focus visível (≥3:1 contraste) | |
| 15 | Usuário pode **trocar a fonte** | |
| 16 | Funciona em **dark mode, high-contrast mode, grayscale** | |

---

## 8. O Argumento de Negócio

Design acessível NÃO É só "compliance." É BOM NEGÓCIO:

- **1 bilhão de pessoas** têm deficiência — é o maior mercado inexplorado do mundo
- **Todo mundo envelhece** — presbiopia começa aos 40. Aos 60, TODO MUNDO precisa de fonte maior e mais contraste
- **Usuários sem deficiência também se beneficiam** — texto de alto contraste e bem espaçado é melhor para TODOS (especialmente em mobile, no sol)
- **SEO melhora** — Google favorece sites acessíveis (texto legível = texto indexável)

**Design acessível é design MELHOR. Para TODO MUNDO.**

---

## Fontes Citadas Neste Loop

| Fonte | Designer/Ano | Destaque |
|---|---|---|
| **Atkinson Hyperlegible** | Braille Institute, 2021/2025 | Melhor fonte de acessibilidade geral |
| **Lexend** | Bonnie Shaver-Troup, 2019 | Redução de estresse visual, 8 larguras |
| **Verdana** | Matthew Carter, 1996 | Desenhada para tela, aberturas generosas |
| **Tahoma** | Matthew Carter, 1994 | Compacta, boa em UI |
| **Inter** | Rasmus Andersson, 2017 | x-height alta, aberturas grandes |
| **Open Sans** | Steve Matteson, 2011 | Humanista, quente, legível |
| **Atkinson Hyperlegible Mono** | Braille Institute, 2025 | Monoespaçada acessível |
| **OpenDyslexic** | Abelardo Gonzalez, 2011 | Fonte para dislexia (pouca evidência) |

---

> **Próximo loop sugerido:** Tipografia & Emoção — como a fonte afeta o que as pessoas SENTEM ao ler. Psicologia tipográfica, o efeito de fontes "sérias" vs "brincalhonas," e estudos de caso. Ou: **Tendências Tipográficas 2025-2026** — o que está acontecendo AGORA no mundo das fontes.
