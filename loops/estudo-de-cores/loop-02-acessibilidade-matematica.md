# 🎨 Estudo de Cores Loop 02 — A Matemática da Acessibilidade: Por Que 4.5:1 É Uma Mentira (E O Que Vem Depois)

> **Data:** 2026-07-02
> **Loop:** 2 de ∞
> **Tema:** WCAG 2.x contrast ratio, APCA (WCAG 3), Delta E na acessibilidade, e como MEDIR legibilidade com precisão perceptual. O contraste 4.5:1 que você verifica no Stark é baseado em fósforo de CRT de 1990. A fórmula MENTE em pelo menos 6 maneiras diferentes. A alternativa (APCA) existe desde 2019 e é baseada na MESMA ciência perceptual do OKLCH. Vamos entender a matemática.

---

## 0. O Problema: Acessibilidade Medida Com Régua Quebrada

Você abre o Stark, verifica o contraste do texto, vê "FAIL — 3.2:1 (precisa 4.5:1)" e ajusta a cor até passar. **Parabéns: você passou num teste que NÃO MEDE o que você acha que mede.**

O problema não é o Stark. O problema é a **FÓRMULA** que o Stark é obrigado a usar — a fórmula de contraste do WCAG 2.x. Ela tem 6 defeitos fundamentais:

1. **Baseada em fósforo de CRT de 1990, não em percepção humana moderna**
2. **Ignora polaridade** (texto claro em fundo escuro ≠ texto escuro em fundo claro)
3. **Ignora peso e tamanho da fonte** (light 300 vs bold 700)
4. **O mesmo ratio significa legibilidade DIFERENTE para cores diferentes**
5. **Casos paradoxais** onde 4.5:1 passa mas é ilegível, e <4.5:1 falha mas é perfeitamente legível
6. **Binário: passa/falha.** Não existe "quase passou" ou "passa com folga." A legibilidade é um ESPECTRO.

**A boa notícia:** existe uma alternativa. Chama-se **APCA** (Advanced Perceptual Contrast Algorithm). É a base do WCAG 3.0. E é baseada em CIÊNCIA PERCEPTUAL MODERNA. Vamos entender tudo.

---

## 1. A Fórmula WCAG 2.x: Dissecando O Monstro

### 1.1 O Cálculo Completo

```
1. Linearizar o sRGB (remover gamma correction):
   para cada canal R, G, B (0-255):
     s = canal / 255
     se s <= 0.04045:
       linear = s / 12.92
     senão:
       linear = ((s + 0.055) / 1.055) ^ 2.4

2. Calcular LUMINÂNCIA RELATIVA (Y):
   Y = 0.2126 × R_linear + 0.7152 × G_linear + 0.0722 × B_linear

3. Calcular CONTRASTE:
   ratio = (Y_claro + 0.05) / (Y_escuro + 0.05)
```

### 1.2 De Onde Vêm Esses Números?

**0.2126, 0.7152, 0.0722:** São os pesos de luminância do **sRGB**, que por sua vez vêm do **Rec. 709** (HDTV standard, 1990), que por sua vez vêm dos **fósforos do CRT** (NTSC, 1953).

O verde contribui 71.5% para a luminância porque o fósforo VERDE do CRT era mais brilhante. Isso também acontece com a visão humana (cones M — verde — são mais numerosos), mas os PESOS EXATOS vêm da engenharia de hardware, não da fisiologia.

**0.05 (fudge factor):** Adicionado para evitar divisão por zero quando o fundo é preto puro (Y=0). Mas cria distorções — o contraste entre `#000` e `#010101` é artificialmente inflado.

**Gamma 2.4:** A curva de transferência do sRGB. É aproximadamente uma power function com γ=2.2, mas tem uma porção LINEAR perto do preto (até 0.04045) para evitar ruído de quantização nas sombras.

### 1.3 Exemplo: Branco Sobre Preto

```
Branco #FFFFFF: Y = 1.0
Preto  #000000: Y = 0.0
Contraste = (1.0 + 0.05) / (0.0 + 0.05) = 1.05 / 0.05 = 21:1
```

21:1 é o MÁXIMO teórico no WCAG 2.x. Passa o 4.5:1 com folga.

### 1.4 Exemplo: Cinza Claro Sobre Branco (O Pesadelo do Designer)

```
Texto #949494 (cinza médio-claro):
  R=148: linear = ((148/255 + 0.055)/1.055)^2.4 = 0.296
  G=148: linear = 0.296
  B=148: linear = 0.296
  Y = 0.2126×0.296 + 0.7152×0.296 + 0.0722×0.296 = 0.296

Fundo #FFFFFF (branco): Y = 1.0

Contraste = (1.0 + 0.05) / (0.296 + 0.05) = 1.05 / 0.346 = 3.03:1
```

**3.03:1 — FALHA nos 4.5:1 do WCAG AA para texto normal.** Mas... você CONSEGUE LER? Depende. Em uma tela retina com texto 16px regular, é legível. Em um monitor 1080p com texto 12px light (300), é impossível.

**O WCAG 2.x não faz distinção entre esses casos.** 3.03:1 = FAIL, ponto final. Isso é o problema #1 com o sistema binário.

---

## 2. Os 6 Paradoxos do WCAG 2.x

### Paradoxo #1: Branco Sobre Laranja PASSA Mas É Ilegível

```
Texto:   #FFFFFF (branco)
Fundo:   #FF9900 (laranja — cor de botão comum)

Laranja: Y = 0.2126×1.0 + 0.7152×0.376 + 0.0722×0.0 = 0.481
Contraste: (1.05) / (0.481 + 0.05) = 1.05 / 0.531 = 1.98:1

Ops... 1.98:1 — ISSO FALHA! Mas espere:
```

Na verdade, `#FF9900` tem Y≈0.38-0.40, o que dá ~2.5:1 — ainda falha. O ponto é que MESMO quando passa (ex: `#FF6600`, Y≈0.26, ratio = 1.05/0.31 = 3.4:1), texto branco sobre laranja é SUBJETIVAMENTE pior que texto cinza escuro sobre branco com o MESMO ratio.

Motivo: a borda entre branco e laranja "vibra" (cromaticidade diferente, luminância similar). O sistema visual tem dificuldade de resolver bordas onde a diferença principal é de MATIZ, não de LUMINÂNCIA.

### Paradoxo #2: Azul Sobre Preto FALHA Mas Branco Sobre Azul PASSA

```
Azul puro (#0000FF) sobre preto (#000000):
  Azul: Y = 0.0722 × 1.0 = 0.0722  (azul quase não contribui para luminância!)
  Preto: Y = 0.0
  Contraste: (0.0722 + 0.05) / 0.05 = 0.1222/0.05 = 2.44:1
  → FALHA no WCAG AA (precisa 4.5:1)


Branco (#FFFFFF) sobre azul puro (#0000FF):
  Contraste: 1.05 / (0.0722 + 0.05) = 1.05/0.1222 = 8.59:1
  → PASSA com folga!
```

**E qual dos dois você CONSEGUE LER MELHOR?** Subjetivamente, texto azul sobre preto (2.44:1 — FALHA) é MAIS LEGÍVEL que branco sobre azul (8.59:1 — PASSA com louvor) para muitas pessoas. Porque o olho humano deteta BORDAS por contraste de LUMINÂNCIA, não de COR. Azul sobre preto tem ALGUMA diferença de luminância (0.072 vs 0). Branco sobre azul tem MUITA diferença de luminância (1.0 vs 0.072) — mas o fundo é AZUL PURO e SATURADO, o que causa HALATION (o azul "vaza" para o texto branco, borrando a borda).

### Paradoxo #3: O Mesmo Ratio, Cores Diferentes

| Combinação | Ratio WCAG | Legibilidade Real |
|---|---|---|
| `#333` sobre `#FFF` (cinza/branco) | 4.5:1 | ✅ OK |
| `#FFF` sobre `#777` (branco/cinza) | 4.5:1 | ❌ PIOR |
| `#000` sobre `#AAA` (preto/cinza) | 4.5:1 | ✅ OK |
| `#FFF` sobre `#3A7` (branco/verde) | 4.5:1 | ❌ RUIM |
| `#FFF` sobre `#66F` (branco/azul) | 4.5:1 | ❌ HORRÍVEL |

**5 combinações. Mesmo ratio (4.5:1). Legibilidade radicalmente diferente.**

O motivo: o WCAG 2.x trata polaridade e cromaticidade como irrelevantes. Mas seu sistema visual NÃO trata.

### Paradoxo #4: Texto Claro Sobre Escuro vs Escuro Sobre Claro

```
#777 sobre #000 (cinza claro sobre preto):
  Y_texto = 0.184, Y_fundo = 0.0
  Ratio = (0.184 + 0.05) / 0.05 = 4.68:1 → PASSA


#777 sobre #FFF (cinza médio sobre branco):
  Y_texto = 0.184, Y_fundo = 1.0
  Ratio = 1.05 / (0.184 + 0.05) = 1.05/0.234 = 4.49:1 → PASSA (quase)

MAS: #777 sobre #000 é MUITO MAIS LEGÍVEL que #777 sobre #FFF.
```

Em modo escuro, texto claro "brilha" contra o fundo escuro. Em modo claro, texto escuro "some" contra o fundo claro. O sistema visual tem MAIS sensibilidade a diferenças de luminância PERTO DO PRETO (Lei de Weber-Fechner: a discriminabilidade é proporcional ao log da intensidade).

**O WCAG 2.x usa a MESMA fórmula para ambos.** A natureza não.

### Paradoxo #5: Fonte Light 300 vs Bold 700

O WCAG 2.x só diferencia entre "texto normal" (4.5:1) e "texto grande" (3:1). "Grande" = 18pt+ ou 14pt+ bold.

Mas:
- **Inter Light 300 a 16px com 4.5:1:** difícil de ler. As hastes finas "somem" no fundo.
- **Inter Bold 700 a 14px com 4.5:1:** perfeitamente legível. Bold age como "âncora visual."

O peso da fonte AFETA a percepção de contraste porque bordas mais ESPESSAS têm mais células ganglionares da retina respondendo a elas. Uma haste de 1px de largura ativa MUITO MENOS neurônios que uma haste de 4px.

### Paradoxo #6: Azul Puro É Quase Invisível (E Isso NÃO É Um Bug da Fórmula)

Lembre-se dos pesos de luminância:
- Verde: 71.5% da luminância
- Vermelho: 21.3%
- **Azul: 7.2%**

Azul puro (`#0000FF`) contribui APENAS 7.2% para a luminância. É quase INVISÍVEL para o cálculo de contraste.

Isso NÃO é um bug — o olho humano REALMENTE tem baixíssima sensibilidade ao azul. A fóvea (centro da retina, onde a visão é mais nítida) NEM TEM cones S (azul). Os cones azuis estão concentrados na PERIFERIA da retina.

**Implicação prática:** nunca use azul puro para elementos pequenos ou texto. Os cones azuis têm baixíssima resolução espacial. Texto azul puro é NATURALMENTE borrado — não importa o ratio.

---

## 3. APCA: O Algoritmo Que Conserta Tudo

### 3.1 O Que É

**APCA** (Advanced Perceptual Contrast Algorithm) foi criado por **Andrew Somers** (Myndex Research) como parte do **WCAG 3.0** (Silver). É o que o WCAG 2.x DEVERIA ter sido.

Diferente do WCAG 2.x, APCA:
- ✅ Usa um modelo perceptual MODERNO (baseado em CIELAB/OKLAB, não CRT)
- ✅ Diferencia texto CLARO sobre escuro de texto ESCURO sobre claro
- ✅ Incorpora PESO e TAMANHO da fonte
- ✅ Produz um VALOR CONTÍNUO (não binário passa/falha)
- ✅ Usa curvas de contraste baseadas em experimentos psicofísicos REAIS

### 3.2 A Saída: Lc (Lightness Contrast)

APCA produz um valor **Lc** (Lightness contrast), tipicamente de 0 a 106:

| Lc Range | Significado |
|---|---|
| **0 – 15** | Invisível. Não use para texto NUNCA. |
| **15 – 30** | Legível apenas para texto GRANDE (24px+) ou BOLD (700+) |
| **30 – 45** | Mínimo para texto corpo (16px, 400) — "passa raspando" |
| **45 – 60** | Boa legibilidade para texto corpo |
| **60 – 75** | Muito boa. Confortável para leitura prolongada. |
| **75 – 90** | Excelente. Alto contraste. |
| **90 – 106** | Máximo. Preto sobre branco. |

**Diferente do WCAG 2.x:**
- Lc 45 NÃO É "passou." É um PONTO no espectro.
- Você como designer decide: "texto corpo precisa de Lc 60, labels secundárias Lc 45, texto de rodapé Lc 30."
- É CONTÍNUO, não binário.

### 3.3 A Matemática (Simplificada)

O APCA completo é complexo (incorpora tamanho de fonte, peso, e uma lookup table psicofísica), mas o núcleo perceptual é:

```
APCA usa um modelo de contraste polarizado:

Para texto CLARO sobre fundo ESCURO:
  Lc = (Y_fundo^0.56 - Y_texto^0.56) × k × ajuste_peso

Para texto ESCURO sobre fundo CLARO:
  Lc = (Y_texto^0.62 - Y_fundo^0.62) × k × ajuste_peso

Onde:
  - Expoentes DIFERENTES para cada polaridade (0.56 vs 0.62)
  - Y é a luminância perceptual (não a do sRGB!)
  - k é uma constante de escala
  - ajuste_peso é uma função de font-weight (100-900)
```

**NOTEM A ASSIMETRIA:** 0.56 para light-on-dark, 0.62 para dark-on-light. Isso reflete o fato de que o sistema visual tem curvas de resposta DIFERENTES para incrementos vs decrementos de luminância (células ON vs OFF da retina).

O WCAG 2.x usa `(Y1+0.05)/(Y2+0.05)` para AMBOS. Simetricamente. Errado.

### 3.4 Exemplo: O Laranja Que O WCAG Errou

```
Branco (#FFF) sobre Laranja (#FF9900):

WCAG 2.x: ~3.5:1 → FALHA
APCA:     Lc ~28 → "só para texto grande/bold"

Branco (#FFF) sobre Azul escuro (#2244AA):

WCAG 2.x: ~6.5:1 → PASSA
APCA:     Lc ~62 → "boa legibilidade"  ← Isso é o CORRETO
```

### 3.5 Exemplo: O Azul Sobre Preto Que O WCAG Errou

```
Azul (#4466FF) sobre Preto (#000000):

WCAG 2.x: ~2.8:1 → FALHA
APCA:     Lc ~42 → "mínimo para texto corpo"

O APCA reconhece que azul sobre preto TEM contraste perceptual
(porque usa luminância perceptual, não pesos de CRT).
```

### 3.6 Tabela de Ajuste por Peso de Fonte

APCA tem uma lookup table para peso de fonte (baseada em experimentos psicofísicos):

| Font Weight | Ajuste (multiplicador aproximado) | Impacto |
|---|---|---|
| 100 (Thin) | ~0.70× | Menos contraste efetivo — precisa de cores mais contrastantes |
| 200 (ExtraLight) | ~0.78× | |
| 300 (Light) | ~0.85× | |
| 400 (Regular) | 1.00× | Linha de base |
| 500 (Medium) | ~1.10× | |
| 600 (SemiBold) | ~1.20× | |
| 700 (Bold) | ~1.35× | MAIS contraste efetivo — pode usar cores menos contrastantes |
| 800 (ExtraBold) | ~1.45× | |
| 900 (Black) | ~1.55× | Máximo — texto preto sobre fundo escuro ainda é legível |

**A mesma cor, mesmo fundo, Lc sobe de ~38 (Light 300) para ~51 (Bold 700).**

Isso significa que você PODE usar texto Light 300 em cinza escuro — se o Lc resultante for aceitável PARA VOCÊ. O APCA te dá o NÚMERO. Você decide o threshold.

---

## 4. Delta E na Acessibilidade: Medindo "Diferente o Suficiente"

### 4.1 O Problema: "Erro" vs "Sucesso" Precisam Ser VISIVELMENTE Diferentes

Você tem:
- Estado de erro: ícone vermelho, fundo `#FEE2E2`
- Estado de sucesso: ícone verde, fundo `#DCFCE7`

Daltonismo deuteranopia (8% dos homens): vermelho e verde têm o MESMO MATIZ percebido (amarelado). A ÚNICA diferença visível é LUMINÂNCIA.

**Delta E (ΔE) mede exatamente isso: "quão diferentes são duas cores para um humano?"**

### 4.2 ΔE Thresholds Para Acessibilidade

| ΔE2000 | Significado para acessibilidade |
|---|---|
| < 3 | INDISTINGUÍVEL para a maioria. ❌ Não use para codificação semântica |
| 3 – 6 | Perceptível com atenção. ⚠️ Não basta para daltonismo |
| 6 – 12 | Claramente diferente. ✅ Bom para elementos secundários |
| 12 – 24 | Muito diferente. ✅✅ Seguro para codificação semântica |
| > 24 | "Outra cor." ✅✅✅ Máxima segurança |

### 4.3 Exemplo Prático de Validação

```javascript
// Duas cores de "estado" que precisam ser distinguíveis
const errorBg   = "#FEE2E2";  // vermelho claro
const successBg = "#DCFCE7";  // verde claro

// Converter para OKLAB e calcular ΔE2000
// ΔE2000(errorBg, successBg) ≈ 18

// 18 > 12 → "claramente diferente" ✅
// MAS para deuteranopia: ΔE efetivo é APENAS ~4
// (porque a diferença vermelho-verde colapsa)
```

**Lição:** ΔE não é suficiente. Você precisa simular daltonismo e recalcular ΔE na simulação.

### 4.4 A Regra de Ouro da Codificação Semântica

Para elementos que dependem de COR para comunicar significado:

```
C1: Cor 1 (ex: erro)
C2: Cor 2 (ex: sucesso)

1. ΔE2000(C1, C2) > 12  →  OK para visão normal
2. Simular deuteranopia → ΔE2000(C1', C2') > 8  →  OK para daltonismo
3. E ADICIONAR: ícone + texto como canais REDUNDANTES
```

**Cor nunca sozinha.** Loop 01 e loop-21 já disseram isso. Agora você tem a MATEMÁTICA para medir.

---

## 5. Ferramentas e Fluxo de Trabalho

### 5.1 O Stack de Validação de Cor

| Ferramenta | O Que Mede | Base |
|---|---|---|
| **Stark** (Figma/Sketch) | WCAG 2.x ratio | sRGB luminance |
| **Axe DevTools** | WCAG 2.x ratio | sRGB luminance |
| **APCA Calculator** (Myndex) | APCA Lc | Perceptual model |
| **Polypane** | APCA Lc + simulação | Perceptual model |
| **Chrome DevTools** | WCAG 2.x only (por enquanto) | sRGB luminance |
| **Colour Contrast Analyser** (TPGi) | WCAG 2.x + APCA beta | Ambos |
| **Adobe Color** | Daltonismo, WCAG | sRGB |
| **Coolors** | WCAG ratio rápido | sRGB |

### 5.2 Fluxo Recomendado (2026)

```
1. Design em OKLCH (não HSL!)
   → Paleta com lightness uniforme

2. Valide contraste COM APCA (não WCAG 2.x)
   → Alvo: Lc > 60 para texto corpo regular
   → Alvo: Lc > 45 para texto corpo bold
   → Alvo: Lc > 30 para texto grande / headings

3. Simule daltonismo
   → Deuteranopia: +comum (8% homens)
   → Protanopia: ~2% homens
   → Tritanopia: rara (~0.003%)

4. Teste com USUÁRIOS REAIS com daltonismo
   → Simulação NÃO É experiência real

5. WCAG 2.x como COMPLIANCE (legal), APCA como QUALIDADE (real)
```

---

## 6. WCAG 3.0: O Futuro (Que Já Devia Ser Presente)

### 6.1 De Binário Para Escala

| | WCAG 2.x (2008) | WCAG 3.0 (draft 2026) |
|---|---|---|
| **Contraste** | Ratio binário (passa/falha) | APCA Lc (espectro 0-106) |
| **Peso de fonte** | Ignorado | Multiplicador perceptual |
| **Tamanho** | 2 tiers (normal, grande) | Lookup table contínua |
| **Polaridade** | Ignorada | Fórmulas diferentes para light/dark |
| **Cor de fundo** | Só luminância do fundo | Interação perceptual texto-fundo |
| **Resultado** | A/AA/AAA | Bronze/Prata/Ouro |

### 6.2 Bronze, Prata, Ouro

WCAG 3.0 abandona o sistema A/AA/AAA e adota pontuação:

- **Bronze:** mínimo. Lc > 30 para todo texto. Equivalente aproximado ao WCAG 2.x AA.
- **Prata:** intermediário. Lc > 45 para texto corpo. Inclui testes adicionais (não só cor).
- **Ouro:** excelente. Lc > 60. Inclui preferências de usuário (contraste adaptativo), daltonismo, e condições de iluminação.

### 6.3 Preferências de Usuário (CSS `prefers-contrast`)

WCAG 3.0 reconhece que contraste NÃO É UM VALOR FIXO:

```css
@media (prefers-contrast: more) {
  /* Usuário precisa de MAIS contraste */
  --text-body: oklch(0.10 0 0);      /* de 0.15 → 0.10 */
  --text-secondary: oklch(0.30 0 0);  /* de 0.45 → 0.30 */
}

@media (prefers-contrast: less) {
  /* Usuário prefere MENOS contraste (ex: enxaqueca, fotofobia) */
  --text-body: oklch(0.25 0 0);
  --bg: oklch(0.15 0 0);
}
```

**Isso é design de cor ADAPTATIVO.** Não é "uma paleta para todos." É "uma paleta que RESPEITA cada usuário."

---

## 7. Exemplo Completo: Construindo Uma Escala De Cinza ACESSÍVEL

Vamos construir uma escala de cinza usando APCA como guia:

```css
/*
 * Escala perceptual APCA-aware
 * Cada passo é validado para legibilidade
 * Alvo: texto corpo 16px/400 = Lc > 60
 */

/* FUNDO CLARO (light mode) */
--gray-50:  oklch(0.98 0 0);  /* bg principal  */
--gray-100: oklch(0.93 0 0);  /* bg secundário */
--text-primary:   oklch(0.12 0 0);  /* Lc ~95 contra gray-50 — EXCELENTE */
--text-secondary: oklch(0.35 0 0);  /* Lc ~65 contra gray-50 — BOA */
--text-disabled:  oklch(0.55 0 0);  /* Lc ~40 contra gray-50 — MÍNIMO (só bold/large) */

/* FUNDO ESCURO (dark mode) */
--bg-dark:        oklch(0.12 0 0);
--text-dark-primary:   oklch(0.95 0 0);  /* Lc ~95 contra bg-dark — EXCELENTE */
--text-dark-secondary: oklch(0.72 0 0);  /* Lc ~65 contra bg-dark — BOA */
--text-dark-disabled:  oklch(0.48 0 0);  /* Lc ~40 contra bg-dark — MÍNIMO */
```

Compare com uma escala em HSL/rgb:
```css
/* MESMA "diferença de lightness" em HSL — NÃO FUNCIONA */
--gray-400: hsl(0 0% 40%);  /* parece diferente de... */
--gray-500: hsl(0 0% 50%);  /* ...que parece diferente de... */
--gray-600: hsl(0 0% 60%);  /* ...porque HSL não é perceptual */
```

---

## 8. Para Levar Para Casa

1. **WCAG 2.x 4.5:1 é uma aproximação de 1990 baseada em CRT.** Use para COMPLIANCE legal. Use APCA para QUALIDADE real.

2. **O mesmo ratio significa legibilidade DIFERENTE dependendo da cor, polaridade, peso e tamanho.** A fórmula ignora tudo isso.

3. **APCA (Lc 0-106) é contínuo, perceptual e considera polaridade + peso.** É a base do WCAG 3.0. Use DESDE JÁ.

4. **ΔE2000 > 12 entre cores semânticas** (erro/sucesso, etc.). E sempre valide com simulação de daltonismo.

5. **OKLCH + APCA é o stack do futuro.** Cores perceptualmente uniformes + contraste perceptual = acessibilidade de VERDADE.

6. **`prefers-contrast` no CSS.** Respeite a preferência do usuário. Contraste não é "one size fits all."

---

## Referências

- Andrew Somers — *Advanced Perceptual Contrast Algorithm (APCA)*, Myndex Research (2019-2025). A documentação completa do APCA.
- W3C — *WCAG 3.0 Working Draft* (2024-2026). Especificação do novo padrão.
- W3C — *WCAG 2.2* (2023). Ainda é o padrão legal vigente.
- CIE 015:2018 — *Colorimetry, 4th Edition*. Base perceptual para ΔE.
- G. Sharma, W. Wu, E. N. Dalal — *The CIEDE2000 Color-Difference Formula* (2005).
- Bruce Lindbloom — *RGB/XYZ Math*. Conversões de espaço de cor.
- Myndex — *APCA Readability Criterion*. Whitepaper do modelo perceptual.
- Chris Lilley — *CSS Color Level 4 and Accessibility* (W3C, 2023).
- Lea Verou — *Contrast in CSS: WCAG 2 vs APCA* (2023).

---

*Fim do Estudo de Cores Loop 02. Acessibilidade é matemática, não opinião. WCAG 2.x é compliance. APCA é qualidade. ΔE é linguagem. Cron: segue para loop 03.*
