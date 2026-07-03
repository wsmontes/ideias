# 🎨 Estudo de Cores Loop 03 — Dark Mode: A Ciência De Escurecer Sem Perder A Alma Da Cor

> **Data:** 2026-07-02
> **Loop:** 3 de ∞
> **Tema:** A física e fisiologia do dark mode. Por que "só inverter as cores" é um DESASTRE perceptual. O que acontece com a saturação em fundos escuros (chroma shift). Como construir elevação SEM sombra (usando cor). Halation: por que texto branco sobre preto "vaza." E a transformação OKLCH que gera dark mode automaticamente preservando a identidade da marca.

---

## 0. O Problema: Seu Cérebro Não É Um Inversor De Cor

A intuição ingênua de dark mode é:

```
Light mode → inverte → Dark mode
#FFFFFF → #000000
#000000 → #FFFFFF
#1A1A2E → #E5E5D1 (inverte cada canal RGB)
```

**Isso FALHA de 7 maneiras diferentes.** Você não pode "só inverter" porque:

1. **Percepção de luminância não é linear.** Branco → preto não é o mesmo que cinza médio → cinza médio.
2. **Saturação "vibra" em fundo escuro.** Cores que são confortáveis em fundo branco AGRIDEM em fundo preto.
3. **Contraste não é simétrico.** Texto claro sobre escuro ≠ texto escuro sobre claro (APCA já mostrou isso).
4. **Elevação some.** Em light mode, SOMBRA indica elevação. Em dark mode, não existe sombra no preto.
5. **Halation.** Texto branco sobre preto "vaza" — as bordas borram fisiologicamente na retina.
6. **Adaptação visual.** Seu olho está em estado MESÓPICO (entre fotópico/claro e escotópico/escuro) — os cones VERMELHOS perdem sensibilidade.
7. **Brilho do dispositivo muda tudo.** O mesmo `#121212` parece diferente com brilho a 30% vs 80%.

---

## 1. Por Que "Só Inverter" É Um Desastre

### 1.1 O Experimento: Inverter Uma Paleta Inteira

Paleta light mode (Material Design 3 baseline):

```css
--bg:      #FFFBFE;  /* superfície principal */
--surface: #F3EDF7;  /* card / superfície elevada */
--primary: #6750A4;  /* cor da marca */
--text:    #1C1B1F;  /* texto principal */
```

Invertendo RGB ingênuamente:

```css
--bg:      #000401;  /* ~preto (era #FFFBFE) */
--surface: #0C1208;  /* verde escuro??? (era #F3EDF7) */
--primary: #98AF5B;  /* verde musgo??? (era #6750A4 — ROXO!) */
--text:    #E3E4E0;  /* cinza claro (era #1C1B1F) */
```

**Resultado:** sua marca ROXA virou VERDE. O fundo levemente rosado virou preto puro. O card lilás virou verde escuro.

Isso acontece porque inverter canais RGB NÃO preserva MATIZ. É uma operação puramente geométrica no cubo RGB, sem nenhum significado perceptual.

### 1.2 O Que Deveria Acontecer

```css
/* Light mode */
--bg:      oklch(0.98 0.005 300);  /* quase branco, leve tom lilás */
--surface: oklch(0.95 0.01 290);   /* card lilás claro */
--primary: oklch(0.50 0.18 290);   /* roxo da marca */
--text:    oklch(0.12 0.005 290);  /* quase preto */

/* Dark mode — MESMO MATIZ, luminância adaptada */
--bg:      oklch(0.15 0.005 300);  /* escuro, MESMO tom lilás */
--surface: oklch(0.20 0.01 290);   /* card escuro, MESMO matiz */
--primary: oklch(0.65 0.15 290);   /* roxo mais claro, MESMA identidade */
--text:    oklch(0.90 0.005 290);  /* quase branco */
```

**A diferença é que usamos OKLCH e ajustamos APENAS a luminância (e um pouco do chroma).** O matiz (290°) permanece IDÊNTICO. A marca continua ROXA.

---

## 2. Chroma Shift: Por Que Cores "Vibram" No Escuro

### 2.1 O Fenômeno

Pegue um botão azul vibrante que funciona perfeitamente em light mode:

```css
--btn-primary: oklch(0.50 0.22 260);  /* azul saturado, 50% lightness */
```

Agora coloque ele num fundo escuro `oklch(0.12 0 0)`:

**RESULTADO:** o botão PARECE mais saturado do que em fundo claro. Ele "vibra." As bordas ficam DIFUSAS. Causa fadiga visual em segundos.

**Isso NÃO é ilusão.** É o efeito **Helmholtz-Kohlrausch:** cores saturadas parecem MAIS CLARAS do que realmente são. Em fundo escuro, esse efeito é AMPLIFICADO porque o contraste de cromaticidade compete com o contraste de luminância.

### 2.2 A Regra de Redução de Chroma

Para neutralizar o Helmholtz-Kohlrausch em dark mode:

| Lightness original | Redução de Chroma para dark mode |
|---|---|
| L=0.40–0.50 (cores "médias") | Reduzir chroma em **15-25%** |
| L=0.50–0.60 (cores "claras") | Reduzir chroma em **10-20%** |
| L=0.60-0.70 (cores "muito claras") | Reduzir chroma em **5-15%** |
| Cores de alerta/erro (ex: vermelho) | Reduzir MAIS — 25-35%. Vermelho em fundo escuro é AGRESSIVO |

**Fórmula simplificada:**

```javascript
function darkModeChroma(lightness, chroma) {
  // Redução proporcional: quanto mais escuro, menos chroma
  const factor = 0.75 + (lightness - 0.4) * 0.5;
  return chroma * Math.max(0.65, Math.min(0.90, factor));
}
```

### 2.3 Exemplo Visual (Descritivo)

```
Light mode:  azul oklch(0.50 0.22 260) em fundo oklch(0.98 0 0)
  → Chroma 0.22 parece "azul normal." Agradável.

Dark mode:   azul oklch(0.50 0.22 260) em fundo oklch(0.12 0 0)
  → Chroma 0.22 parece "azul NEON." Agressivo. Vibra.

Dark mode corrigido: azul oklch(0.60 0.16 260) em fundo oklch(0.12 0 0)
  → Chroma reduzido para 0.16. "Azul escuro elegante." A marca respira.
```

**Três ajustes ocorreram:**
1. Lightness subiu (0.50 → 0.60) — cor mais clara para contrastar com fundo escuro
2. Chroma reduziu (0.22 → 0.16) — menos saturação para não "vibrar"
3. Hue permaneceu (260°) — a identidade da cor foi PRESERVADA

---

## 3. Elevação Tonal: Substituindo Sombra Por Brilho

### 3.1 O Problema Fundamental

Em light mode, ELEVAÇÃO é comunicada por SOMBRA:

```
Quanto mais alto o elemento, MAIS SOMBRA ele projeta.
Card nível 1: sombra leve
Card nível 3: sombra pronunciada
Modal: sombra pesada + overlay escuro
```

**Em dark mode, não existe sombra no preto.** Você não pode projetar escuridão sobre escuridão.

### 3.2 A Solução: Elevação Por LUMINÂNCIA (Material You)

O Material Design 3 introduziu **Elevação Tonal** — em vez de sombra, elementos mais elevados são MAIS CLAROS:

```
Fundo principal:     oklch(0.10 0 0)   ← o mais escuro
Card (nível 1):      oklch(0.13 0 0)   ← um pouco mais claro
Card (nível 2):      oklch(0.16 0 0)
Dialog/Modal:        oklch(0.19 0 0)   ← mais claro ainda
Sheet (nível max):   oklch(0.22 0 0)   ← "flutuando" sobre tudo
```

**É o INVERSO da intuição.** Em dark mode, "mais perto do usuário" = "mais claro." Isso mimetiza como a LUZ funciona no mundo real: objetos mais próximos de uma fonte de luz são mais iluminados.

### 3.3 Overlays de Elevação

Outra técnica é sobrepor branco translúcido como "elevação":

```css
/* Em vez de sombra, use overlay de branco com opacidade */
.surface-elevated {
  background: oklch(0.12 0 0);  /* base escura */
  /* Overlay de "elevação" via pseudo-elemento ou gradiente */
  box-shadow: 0 0 0 1px oklch(1 0 0 / 0.05);  /* borda sutil de luz */
}
```

Material You define overlays assim:

| Elevação (dp) | Overlay branco (opacidade) |
|---|---|
| 1 | 5% |
| 2 | 8% |
| 3 | 11% |
| 4 | 12% |
| 5 | 14% |

Implementação CSS moderna com `color-mix()`:

```css
--elevation-1: color-mix(in oklch, var(--surface) 95%, white);
--elevation-2: color-mix(in oklch, var(--surface) 92%, white);
--elevation-3: color-mix(in oklch, var(--surface) 89%, white);
```

---

## 4. Halation: Quando O Texto Branco "Vaza"

### 4.1 O Fenômeno Fisiológico

**Halation** (ou "halo") é o efeito onde texto BRANCO sobre fundo PRETO parece ter as bordas BORRADAS — como se a luz do texto "vazasse" para o fundo.

Isso NÃO acontece na tela. Acontece na sua RETINA:

1. Luz do texto branco atinge a retina com ALTA intensidade
2. Os fotorreceptores saturam localmente
3. O sinal "vaza" para células vizinhas (spread of activation nas camadas da retina)
4. Células ganglionares respondem ao contraste, mas o sinal já está BORRADO

**Resultado perceptual:** texto branco sobre preto parece MENOS NÍTIDO que texto preto sobre branco, mesmo com o mesmo "contraste" matemático.

### 4.2 Quem É Mais Afetado

- **Astigmatismo:** ~30% da população. A curvatura irregular da córnea AMPLIFICA o halation em dark mode.
- **Miopia sem correção:** luz "espalha" mais na retina míope.
- **Idosos:** o cristalino perde transparência, causando mais light scatter.
- **Todo mundo:** em ambientes MUITO escuros (tela com brilho alto no quarto escuro).

### 4.3 Como Minimizar Halation

1. **Nunca use branco PURO `#FFFFFF` sobre preto puro `#000000`.** Use `oklch(0.90 0 0)` sobre `oklch(0.08 0 0)`.

2. **Aumente o peso da fonte em dark mode.** Bold é mais resistente ao halation que light. As hastes mais ESPESSAS "seguram" a borda.

3. **Adicione espaçamento entre letras (tracking).** Texto em dark mode se beneficia de +0.01–0.02em de letter-spacing extra.

4. **Use fontes com maior x-height.** A altura-x maior aumenta a área de "sinal" sobre "ruído."

5. **Reduza o brilho da tela.** Halation é proporcional à intensidade ABSOLUTA da luz. Menos brilho = menos halation.

### 4.4 Halation e Cor

Texto COLORIDO sobre fundo escuro sofre MAIS halation que texto branco:

| Cor do texto | Halation em fundo escuro | Motivo |
|---|---|---|
| Branco | Moderado | Alta luminância, mas acromático |
| Amarelo | **ALTO** | Altíssima luminância + cromático |
| Ciano | **ALTO** | Alta luminância + borda difusa (cones S ausentes na fóvea) |
| Azul | Baixo | Baixa luminância (7.2%) |
| Vermelho | Moderado | Média luminância |
| Verde | Moderado-Alto | Alta luminância (71.5%) |

**Lição:** em dark mode, EVITE texto amarelo ou ciano puro. Se precisar de cor, use tons PASTÉIS (baixo chroma, alto lightness).

---

## 5. A Transformação OKLCH Para Dark Mode

### 5.1 O Algoritmo

Dada uma cor em light mode `oklch(L, C, h)`, a transformação para dark mode é:

```javascript
function toDarkMode(lightColor) {
  const [L, C, h] = lightColor; // OKLCH

  // 1. INVERTER luminância (perceptual!)
  const L_dark = invertLightness(L);

  // 2. REDUZIR chroma (anti-vibração)
  const C_dark = reduceChroma(C, L_dark);

  // 3. PRESERVAR matiz (identidade da cor)
  const h_dark = h; // NÃO MUDA!

  return [L_dark, C_dark, h_dark];
}

function invertLightness(L) {
  // Inversão perceptual: o "oposto" de 0.98 (quase branco) é 0.08 (não 0.02!)
  // Curva assimétrica porque o olho é mais sensível perto do preto

  if (L > 0.90) return 0.08 + (1.0 - L) * 0.7;  // superfícies claras → bem escuras
  if (L > 0.50) return 0.15 + (0.90 - L) * 0.3;  // cores médias-claras → médias-escuras
  return 0.85 - L * 0.7;                           // cores escuras → claras
}

function reduceChroma(C, L_dark) {
  // Cores mais escuras precisam de MAIS redução
  if (L_dark < 0.20) return C * 0.65;
  if (L_dark < 0.40) return C * 0.75;
  return C * 0.85;
}
```

### 5.2 Exemplos da Transformação

| Cor Original (light) | Dark Mode Transformada |
|---|---|
| `oklch(0.98 0.005 300)` — bg lilás claro | `oklch(0.10 0.003 300)` — bg lilás escuro |
| `oklch(0.55 0.22 260)` — azul médio | `oklch(0.60 0.15 260)` — azul dark mode |
| `oklch(0.50 0.25 25)` — vermelho vibrante | `oklch(0.58 0.16 25)` — vermelho dark mode |
| `oklch(0.12 0.005 290)` — texto escuro | `oklch(0.90 0.003 290)` — texto claro |

**Notem:** o matiz (último número) NUNCA muda. A identidade da cor sobrevive intacta.

### 5.3 CSS Nativo Com `light-dark()`

```css
:root {
  color-scheme: light dark;

  /* Superfícies */
  --bg:      light-dark(oklch(0.98 0.005 300), oklch(0.10 0.003 300));
  --surface: light-dark(oklch(0.95 0.01 290), oklch(0.16 0.007 290));
  --elevated: light-dark(oklch(0.98 0 0), oklch(0.20 0 0));

  /* Marca */
  --primary: light-dark(oklch(0.50 0.18 290), oklch(0.65 0.13 290));
  --on-primary: light-dark(oklch(1 0 0), oklch(0.15 0 0));

  /* Texto */
  --text: light-dark(oklch(0.12 0.005 290), oklch(0.90 0.003 290));
  --text-secondary: light-dark(oklch(0.40 0 0), oklch(0.70 0 0));

  /* Estados */
  --error: light-dark(oklch(0.50 0.22 20), oklch(0.65 0.15 20));
  --success: light-dark(oklch(0.50 0.22 150), oklch(0.65 0.15 150));
}
```

**Isso é 100% CSS nativo em 2026.** Sem JavaScript. Sem classes `.dark`. O navegador aplica automaticamente baseado em `prefers-color-scheme`.

---

## 6. A Fisiologia da Adaptação Ao Escuro

### 6.1 Os Três Regimes de Visão

| Regime | Luminância (cd/m²) | Receptores | Sensibilidade de cor |
|---|---|---|---|
| **Fotópico** (luz do dia) | > 3 cd/m² | Cones (L, M, S) | Máxima — vemos TODAS as cores |
| **Mesópico** (crepúsculo) | 0.001 – 3 cd/m² | Cones + Bastonetes | Reduzida — cores perdem saturação |
| **Escotópico** (noite) | < 0.001 cd/m² | Só Bastonetes | ZERO — só vemos tons de cinza |

**Dark mode num smartphone típico a 30% de brilho opera no regime MESÓPICO.** Isso significa:

- Seus cones (visão de cor) estão PARCIALMENTE ativos
- Seus bastonetes (visão noturna, sem cor, ALTA sensibilidade) TAMBÉM estão ativos
- Os bastonetes são MAIS sensíveis a comprimentos de onda CURTOS (azul/verde)
- Os cones VERMELHOS são os primeiros a "desligar" na baixa luminância

### 6.2 Efeito Purkinje

Johannes Purkinje (1825) descobriu que em baixa luminância, o pico de sensibilidade do olho MUDA:

```
Luz do dia (fotópico):  pico em ~555 nm (verde-amarelado)
Baixa luz (escotópico): pico em ~507 nm (verde-azulado)

DIFERENÇA: ~48 nm = desvio para o AZUL
```

**Implicação para dark mode:** em ambientes escuros, cores AZUIS e VERDES parecem MAIS CLARAS do que realmente são. Cores VERMELHAS parecem MAIS ESCURAS.

Isso significa que, no seu dark mode, o VERMELHO do botão de "deletar" pode PARECER menos urgente do que você pretendia — porque os cones vermelhos estão "dormindo."

**Correção:** vermelhos em dark mode precisam de MAIS lightness do que azuis para ter o MESMO impacto perceptual:

```
Vermelho dark mode: oklch(0.60 0.15 25)   ← L=0.60
Azul dark mode:     oklch(0.55 0.15 260)  ← L=0.55 (pode ser mais escuro!)
```

### 6.3 A Curva de Adaptação Temporal

Quando o usuário alterna de light para dark mode (ou vice-versa), a adaptação visual NÃO é instantânea:

```
0s:      Mudança.
0-5s:    Desconforto. "Muito escuro!" ou "Muito claro!"
5-30s:   Adaptação rápida (cones ajustam ganho).
30s-5m:  Adaptação lenta (bastonetes regeneram rodopsina).
5-30m:   Adaptação completa (máxima sensibilidade noturna).
```

**Implicação de design:** uma transição BRUSCA entre light/dark mode causa desconforto por 5-30 segundos. Transições SUAVES (com `transition: background-color 0.3s`) ajudam mas não eliminam o problema — o desconforto é FISIOLÓGICO, não estético.

---

## 7. OLED vs LCD: A Cor Muda Com O Hardware

### 7.1 Preto Verdadeiro vs Preto Cinza

| Tecnologia | Preto `#000000` | Preto `#121212` |
|---|---|---|
| **OLED** (iPhone Pro, Galaxy S) | Pixel DESLIGADO. Preto ABSOLUTO. Zero luz. | Pixel ligado no mínimo. Cinza MUITO escuro. |
| **LCD** (iPhone SE, monitores) | Backlight sempre ligado. Preto é CINZA ESCURO. | Igual ao `#000000` — ambos são cinza. |
| **Mini-LED** (MacBook Pro, iPad Pro) | Zona de dimming desligada. Quase OLED. | Pequeno vazamento de zonas vizinhas. |

**Em OLED, `#000000` e `#121212` são RADICALMENTE diferentes.** Um é zero fótons. O outro é alguns fótons. Em LCD, são praticamente iguais (a backlight está sempre acesa).

### 7.2 Smearing Em OLED

OLED tem um problema ÚNICO: **black smearing.** Quando um pixel transita de DESLIGADO (0) para LIGADO (>0), há um micro-atraso (~1-2ms). Isso causa um rastro preto ao scrollar texto branco sobre fundo preto puro.

**Solução:** NUNCA use `#000000` como fundo em dark mode para apps com scroll. Use `#121212` ou `oklch(0.08 0 0)`. O pixel nunca desliga completamente, e o smearing desaparece.

### 7.3 Bateria: O Mito e a Realidade

OLED: pixel preto = desligado = não consome energia. Dark mode economiza bateria.
LCD: backlight sempre acesa. Dark mode NÃO economiza bateria (pode até consumir MAIS).

**Google (2018):** YouTube dark mode em OLED a 100% de brilho economiza 15-60% de bateria. A 50% de brilho, a economia cai para ~5-15%.

**Realidade (2026):** a economia de bateria do dark mode em OLED é REAL mas MODESTA para a maioria dos apps — a não ser que o app seja majoritariamente escuro (ex: terminal, code editor, leitor noturno).

---

## 8. Construindo Uma Paleta Dark Mode Completa

### 8.1 As 5 Regras

1. **Nunca preto puro (`#000`).** Use `oklch(0.08–0.12)`. Elimina smearing OLED e halation.
2. **Nunca branco puro (`#FFF`).** Use `oklch(0.88–0.93)`. Texto branco puro "queima" na retina.
3. **Reduza chroma em 15-25%.** Cores saturadas VIBRAM em fundo escuro.
4. **AUMENTE lightness das cores da marca.** O mesmo lightness de light mode parece MAIS ESCURO em dark mode.
5. **Use elevação tonal (mais claro = mais elevado).** Não existe sombra no escuro.

### 8.2 Paleta de Referência

```css
/* ========== SUPERFÍCIES (escala de elevação tonal) ========== */
--bg-0:     oklch(0.08 0 0);      /* fundo absoluto (nunca #000) */
--bg-1:     oklch(0.11 0 0);      /* cards nível 1 */
--bg-2:     oklch(0.14 0 0);      /* cards nível 2 */
--bg-3:     oklch(0.17 0 0);      /* dialogs/modals */
--bg-4:     oklch(0.20 0 0);      /* sheets/toasts */

/* ========== TEXTO ========== */
--text-primary:     oklch(0.92 0 0);    /* headings, corpo */
--text-secondary:   oklch(0.72 0 0);    /* labels, metadados */
--text-disabled:    oklch(0.45 0 0);    /* placeholder, inativo */
--text-on-primary:  oklch(0.12 0 0);    /* texto sobre cor da marca */

/* ========== MARCA (chroma reduzido 20%) ========== */
--primary:          oklch(0.62 0.15 290);  /* roxo escuro elegante */
--primary-hover:    oklch(0.67 0.14 290);
--primary-active:   oklch(0.57 0.16 290);

/* ========== ESTADOS (chroma ainda mais reduzido) ========== */
--error:            oklch(0.58 0.16 20);   /* vermelho contido */
--success:          oklch(0.58 0.13 150);  /* verde musgo */
--warning:          oklch(0.60 0.12 80);   /* âmbar suave */
--info:             oklch(0.58 0.12 240);  /* azul sereno */

/* ========== BORDAS (sutis, quase imperceptíveis) ========== */
--border:           oklch(0.25 0 0);
--border-hover:     oklch(0.40 0 0);
```

### 8.3 O Teste Final

Pegue sua paleta dark mode. Aplique em um app. Agora:

1. Use o app por 30 minutos à noite, com luzes apagadas.
2. Se algo "incomoda" ou "cansa" → chroma está alto demais.
3. Se tudo parece "cinza sem vida" → chroma está baixo demais.
4. Se você NÃO PERCEBE as cores → está perfeito. Dark mode bom é INVISÍVEL.

---

## 9. Para Levar Para Casa

1. **Nunca "só inverta."** A transformação de light para dark mode é perceptual (OKLCH), não geométrica (RGB).
2. **Chroma sempre reduz em dark mode.** 15-25%. Cores saturadas em fundo escuro AGRIDEM.
3. **Elevação = brilho.** Em dark mode, elementos mais altos são mais CLAROS. É o oposto do light mode.
4. **Nunca `#000` nem `#FFF`.** Preto puro causa smearing. Branco puro causa halation.
5. **Vermelho precisa de MAIS lightness que azul em dark mode.** Efeito Purkinje: cones vermelhos são menos sensíveis no escuro.
6. **Halation é fisiológico, não óptico.** Aumente o peso da fonte e o letter-spacing em dark mode.
7. **OLED vs LCD importa.** `#000` é preto absoluto em OLED (bom para bateria, ruim para scroll). Em LCD, `#000` = `#121212`.
8. **`light-dark()` no CSS + OKLCH = dark mode declarativo e perceptual.** Sem JS, sem classes, sem sofrimento.

---

## Referências

- Johannes Purkinje — *Beobachtungen und Versuche zur Physiologie der Sinne* (1825). O efeito que leva seu nome.
- Google — *Dark Theme & Battery Savings* (Android Dev Summit, 2018). Economia de bateria OLED.
- Google — *Material Design 3 — Dark Theme* (2022). Especificação de elevação tonal.
- Apple — *Human Interface Guidelines — Dark Mode* (2019-2026). Filosofia da Apple para dark mode.
- Hermann von Helmholtz — *Handbuch der physiologischen Optik* (1867). Halation, contraste simultâneo, Helmholtz-Kohlrausch.
- Andrew Somers — *Lightness Contrast in Dark Mode* (Myndex, 2021). APCA em dark mode.
- W3C — *CSS Color Module Level 4* (2023). `light-dark()`, `color-scheme`.
- Björn Ottosson — *OKLAB* (2020). O espaço de cor que torna a transformação perceptual possível.
- Raymond P. Soneira — *Display Technology Shoot-Out* (DisplayMate). Comparações OLED vs LCD.

---

*Fim do Estudo de Cores Loop 03. Dark mode não é "light mode negativo." É um regime visual DIFERENTE — com fisiologia, matemática e elegância próprias. OKLCH + elevação tonal + chroma shift = dark mode que respeita o olho humano. Cron: segue para loop 04.*
