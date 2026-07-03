# 🎨 Estudo de Cores Loop 04 — Geração Algorítmica de Paletas: A Matemática de Criar Cores Que Funcionam Juntas

> **Data:** 2026-07-02
> **Loop:** 4 de ∞
> **Tema:** Como algoritmos geram paletas de cores harmoniosas. A geometria das harmonias cromáticas (complementar, análoga, triádica, tetrádica). O algoritmo do Material You (CAM16, amostragem Monte Carlo no wallpaper). Leonardo AI, Huemint, ColorBrewer. E como construir seu próprio gerador de paletas em OKLCH com restrições de acessibilidade.

---

## 0. O Problema: "Escolher Cores" É Um Pesadelo Que Pode Ser Automatizado

Designers passam HORAS ajustando cores. E a maioria faz no ESCURO: HSL color picker, arrastando sliders, "parece bom" como critério.

Mas gerar paletas é um problema MATEMÁTICO com restrições bem definidas:

1. **Harmonia perceptual** — as cores precisam "conversar" entre si
2. **Acessibilidade** — contrastes mínimos contra fundo e entre si
3. **Hierarquia** — cores primárias, secundárias, de suporte
4. **Identidade** — a paleta precisa "dizer algo" sobre a marca
5. **Dark/light** — funcionar nos dois regimes (loop 03)
6. **Cobertura de estados** — hover, active, disabled, focus

**Se você consegue definir isso como restrições matemáticas, você consegue GERAR paletas.** Não "escolher" — GERAR. É isso que Material You, Leonardo, Huemint, e ColorBrewer fazem. Cada um com uma abordagem diferente.

---

## 1. A Geometria das Harmonias Cromáticas

### 1.1 Por Que Harmonias "Funcionam": Teoria do Processo Oponente

O sistema visual humano tem 3 canais oponentes (Hering, 1892):

| Canal | Par | Resposta |
|---|---|---|
| **Luminância** | Claro ↔ Escuro | Excitação/inibição por intensidade |
| **Vermelho-Verde** | R ↔ G | +R excita, +G inibe (ou vice-versa) |
| **Azul-Amarelo** | B ↔ Y | +B excita, +Y inibe (ou vice-versa) |

**Uma paleta é "harmoniosa" quando EQUILIBRA a ativação desses 3 canais.** Cores complementares (opostas no círculo) se EQUILIBRAM porque excitam canais opostos que se cancelam mutuamente. Cores análogas (vizinhas) são "calmas" porque excitam o MESMO canal, sem competição.

Esta é a BASE NEUROLÓGICA da harmonia cromática. Não é "gosto." É PROCESSAMENTO DE SINAL na retina e no núcleo geniculado lateral.

### 1.2 O Círculo Cromático em OKLCH

Em OKLCH, o círculo cromático é definido pelo ângulo de matiz (h):

```
0°    = vermelho
60°   = amarelo
120°  = verde
180°  = ciano
240°  = azul
300°  = magenta
360°  = vermelho (loop)
```

### 1.3 As 6 Harmonias Clássicas (Geometria no Círculo)

| Harmonia | Ângulos (em OKLCH) | Efeito Perceptual | Quando usar |
|---|---|---|---|
| **Complementar** | h, h+180° | Máximo contraste de matiz. "Vibrante." | CTAs, logos, destaque máximo |
| **Análoga** | h, h±15°, h±30° | Mínimo contraste. "Calma, coesa." | Backgrounds, gradientes suaves |
| **Triádica** | h, h+120°, h+240° | Equilíbrio dinâmico. "Energética." | Apps criativos, jogos, dashboards |
| **Split-Complementar** | h, h+150°, h+210° | Contraste com suavidade. | UI principal — uma cor forte + 2 de apoio |
| **Tetrádica (retângulo)** | h, h+60°, h+180°, h+240° | 2 pares complementares. Rica. | Aplicações com múltiplas seções |
| **Quadrada** | h, h+90°, h+180°, h+270° | 4 cores igualmente espaçadas. | Visualizações categóricas |

### 1.4 A Fórmula Geral (OKLCH)

Para gerar qualquer harmonia, fixe lightness (L) e chroma (C) e varie APENAS o matiz (h):

```javascript
function harmony(baseHue, type) {
  const angles = {
    complementary:       [0, 180],
    analogous:           [0, -30, 30, -15, 15],
    triadic:             [0, 120, 240],
    splitComplementary:  [0, 150, 210],
    tetradic:            [0, 60, 180, 240],
    square:              [0, 90, 180, 270],
  };

  return angles[type].map(offset => ({
    h: (baseHue + offset + 360) % 360,  // matiz variado
    L: 0.55,                              // MESMA luminosidade perceptual!
    C: 0.18,                              // MESMO chroma!
  }));
}

// Exemplo: paleta triádica baseada em azul (h=260)
// → [260° (azul), 20° (vermelho), 140° (verde)]
// Todas com L=0.55, C=0.18 → MESMO peso visual. IMPOSSÍVEL em HSL.
```

**A beleza do OKLCH:** gerar harmonias é literalmente SOMAR ÂNGULOS. E as cores resultantes têm o mesmo "peso visual" automaticamente — porque L e C são perceptualmente uniformes.

### 1.5 Por Que Algumas Harmonias "Falham" Em HSL

Tente gerar uma tríade em HSL com H=60 (amarelo), S=100%, L=50%:

```
Amarelo hsl(60, 100%, 50%)   — parece BRILHANTE (L percebido ~90%)
Vermelho hsl(180, 100%, 50%) — parece MÉDIO (L percebido ~50%)
Azul    hsl(300, 100%, 50%)  — parece ESCURO (L percebido ~30%)
```

**A tríade "perfeita" em HSL tem PESOS VISUAIS completamente diferentes.** O amarelo GRITA. O azul SOME. A harmonia falha — não por causa da geometria, mas porque HSL mente sobre lightness.

**Em OKLCH, a MESMA tríade:**
```
Amarelo oklch(0.55 0.18 90)  — MESMA luminosidade
Ciano   oklch(0.55 0.18 210) — MESMA luminosidade
Magenta oklch(0.55 0.18 330) — MESMA luminosidade
```
Tríade REALMENTE equilibrada. Todas as 3 cores "pesam" o mesmo.

---

## 2. Material You: O Algoritmo Que Extrai Cor Do Wallpaper

### 2.1 A Pipeline Completa

```
Wallpaper do usuário
    ↓
[1] Amostragem de pixels (Monte Carlo com quantização)
    ↓
[2] Conversão para CAM16 (modelo de aparência de cor)
    ↓
[3] Seleção da "seed color" (cor dominante perceptual)
    ↓
[4] Geração de 5 paletas tonais (CAM16 → tons)
    ↓
[5] Atribuição de tokens (accent1-3, neutral1-2)
    ↓
65+ cores → 200+ variantes com light/dark
    ↓
Aplicadas via design tokens a TODOS os componentes
```

### 2.2 Passo 1: Amostragem Monte Carlo com Quantização

Não adianta "fazer média" dos pixels do wallpaper. A média vira marrom. O algoritmo:

1. **Reduz o wallpaper** para ~64×64 pixels (ignora detalhes)
2. **Converte para CAM16** (espaço perceptual — como OKLAB, mas com modelo de adaptação cromática)
3. **Agrupa pixels por similaridade perceptual** (quantização — reduz milhões de cores para ~128 clusters)
4. **Pontua cada cluster** por:
   - População (quantos pixels tem essa cor)
   - Saturação (cores muito cinzas = baixa pontuação)
   - Contraste com outras cores proeminentes
5. **Seleciona a seed color:** a cor com maior pontuação

### 2.3 Passo 2: CAM16 — O Modelo de Aparência de Cor

CAM16 (Color Appearance Model, 2016) é o que o Material You usa em vez de OKLAB. Ele modela:

- **Adaptação cromática:** como o olho se ajusta ao "branco" do ambiente (display branco, papel branco, luz do dia)
- **Background relativo:** a mesma cor parece diferente em fundo claro vs escuro
- **Surround:** condições de visualização (quarto escuro vs escritório iluminado)

**CAM16 é "OKLAB com contexto."** Ele não só diz "essa cor é azul" — ele diz "essa cor vai PARECER azul DADO que o usuário está num quarto escuro com tela OLED."

### 2.4 Passo 3: Geração de Paletas Tonais

A partir da seed color, o Material You gera PALETAS TONAIS — famílias de cores com o MESMO matiz mas luminâncias diferentes:

```
Tonal Palette "Accent 1" (seed = azul, matiz ~260°):
T0    → quase branco azulado
T10   → azul muito claro
T20   → azul claro
...
T50   → azul médio (seed original)
...
T90   → azul escuro
T100  → quase preto azulado
```

**13 tons por paleta × 5 paletas = 65 cores base.** Com light/dark mode = 130 variantes. Com estados (hover, focus, pressed, disabled) = 500+ valores.

O segredo das paletas tonais: **a curva de CHROMA varia com o TONE.** Tons muito claros (T0-T20) e muito escuros (T80-T100) têm chroma REDUZIDO — porque senão vibram (loop 03). Tons médios (T40-T60) têm chroma MÁXIMO:

```
Curva de chroma típica do Material You:
T0:   C=2   (quase branco, chroma mínimo)
T20:  C=8   (subindo)
T50:  C=30  (pico de saturação)
T80:  C=10  (descendo)
T100: C=2   (quase preto, chroma mínimo)
```

### 2.5 As 5 Paletas Tonais

| Paleta | Função | Matiz |
|---|---|---|
| **Accent 1** (Primary) | Cor principal da UI | Seed hue |
| **Accent 2** (Secondary) | Cor de apoio, menos proeminente | Seed hue ± 10-20° |
| **Accent 3** (Tertiary) | Contraste criativo | Seed hue + 60° (análoga distante) |
| **Neutral 1** | Fundos, cards, texto | Seed hue com chroma ~0 (cinza quente) |
| **Neutral 2** (Neutral Variant) | Superfícies secundárias | Seed hue com chroma ~2 (tom sutil) |

**O gênio do Material You:** as cores NEUTRAS também carregam o matiz da seed. Se sua seed é azul, o "cinza" do fundo tem um LEVE tom azulado. Nada é cinza puro. Tudo respira a cor do wallpaper.

---

## 3. ColorBrewer: A Bíblia das Paletas para Dados

### 3.1 Cynthia Brewer e o Problema

Cynthia Brewer (Penn State, 2002) resolveu um problema que TODO mundo que faz gráficos enfrenta: "como escolher cores para um mapa que sejam:
1. Distinguíveis entre si
2. Funcionem para daltônicos
3. Funcionem em impressão P&B
4. Funcionem em projeção (projetor = baixo contraste)"

**ColorBrewer é o padrão OURO para visualização de dados.** É um sistema de paletas curado MANUALMENTE e validado empiricamente.

### 3.2 Os 3 Tipos de Paleta

| Tipo | Uso | Exemplo |
|---|---|---|
| **Sequential** | Dados ordenados de baixo→alto | Densidade populacional, temperatura |
| **Diverging** | Dados com ponto médio significativo | Desvio da média, crescimento/declínio |
| **Qualitative** | Categorias NÃO ordenadas | Tipos de solo, partidos políticos |

### 3.3 Paletas Sequenciais

Vão de claro (valor baixo) a escuro (valor alto). MULTI-MATIZ para melhor discriminabilidade:

```
Blues sequencial (9 classes):
Claro → Escuro
#F7FBFF → #DEEBF7 → #C6DBEF → #9ECAE1 → #6BAED6 → #4292C6 → #2171B5 → #08519C → #08306B
```

Em OKLCH, uma sequencial seria:
```
oklch(0.95 0.02 260) → oklch(0.85 0.05 260) → ... → oklch(0.20 0.12 260)
(L decresce gradualmente, C sobe e depois desce nos extremos)
```

**Por que multi-matiz?** Uma sequencial só de azul (L variando, C e h fixos) é MENOS discriminável que uma que varia sutilmente o matiz. Exemplo: "Yellow-Green-Blue" — de amarelo claro a azul escuro, passando pelo verde. Mais faixas de cor = mais fácil discriminar valores.

### 3.4 Paletas Divergentes

Têm um ponto médio NEUTRO (branco/cinza claro) e divergem para DUAS cores nos extremos:

```
RdBu (Red-Blue, 9 classes):
Vermelho forte → ... → Branco → ... → Azul forte
```

**A matemática OKLCH de uma divergente:**
```javascript
function divergingPalette(hueLow, hueHigh, steps) {
  const mid = Math.floor(steps / 2);
  const palette = [];

  for (let i = 0; i < steps; i++) {
    if (i < mid) {
      // Lado baixo: chroma sobe, lightness desce
      const t = i / (mid - 1);
      palette.push({
        L: 0.90 - t * 0.50,      // de claro a médio
        C: t * 0.25,              // chroma do mínimo ao máximo
        h: hueLow,
      });
    } else if (i === mid) {
      palette.push({ L: 0.95, C: 0, h: 0 }); // neutro
    } else {
      // Lado alto: chroma sobe, lightness desce
      const t = (i - mid - 1) / (steps - mid - 2);
      palette.push({
        L: 0.85 - t * 0.50,
        C: t * 0.25,
        h: hueHigh,
      });
    }
  }
  return palette;
}
```

### 3.5 Paletas Qualitative (Categóricas)

Cores para categorias NÃO ordenadas. Precisam ser MAXIMAMENTE distinguíveis entre si:

```javascript
function qualitativePalette(n, hueStart = 0) {
  // Espaçamento IGUAL no círculo cromático
  return Array.from({ length: n }, (_, i) => ({
    L: 0.55,  // MESMA luminosidade para todas as categorias
    C: 0.18,  // MESMO chroma (sem viés perceptual)
    h: (hueStart + (i * 360) / n) % 360,
  }));
}

// 5 categorias, começando em 15° (vermelho-alaranjado)
qualitativePalette(5, 15);
// → 15°, 87°, 159°, 231°, 303°
// Espaçamento de 72° entre cada categoria
```

**O problema de ≥7 categorias:** com mais de 7 cores, fica IMPOSSÍVEL discriminar todas sem usar chroma E lightness diferentes. ColorBrewer resolve isso manualmente, cor por cor.

---

## 4. Leonardo (Adobe): Paletas por Rede Neural

### 4.1 A Abordagem

Leonardo (Adobe, 2020-presente) usa redes neurais treinadas em milhões de exemplos de design (Behance, Dribbble, Adobe Stock) para gerar paletas.

Diferente de Material You (que extrai de UMA imagem) e ColorBrewer (curado manualmente), Leonardo:

1. **Aprende "semântica de cor"** — "azul = corporativo", "verde = natureza", "roxo = criativo"
2. **Gera paletas de TEXTO** — você digita "paleta solar alegre verão praia" e ele gera cores
3. **Aplica regras de harmonia** implicitamente (aprendidas, não programadas)
4. **Gera múltiplas variações** e você escolhe

### 4.2 O Que Leonardo Acerta (E O Que Ele Erra)

**Acertos:**
- Variedade criativa que algoritmos determinísticos não alcançam
- Captura "tendências" implícitas nos dados de treinamento
- Gera paletas emocionalmente coerentes ("triste", "energético", "nostálgico")

**Limitações:**
- Acessibilidade não é garantida (AO CONTRÁRIO de ColorBrewer e Material You)
- "Caixa preta" — você não sabe POR QUE a paleta funciona
- Viés do dataset (predominantemente ocidental, design de luxo)

---

## 5. Huemint: Perceptual + Acessibilidade Garantida

### 5.1 A Filosofia

Huemint (2021-presente) é um gerador de paletas que combina:
- **Espaço de cor perceptual** (OKLCH)
- **Restrições de acessibilidade** (APCA)
- **Harmonia geométrica** (círculo cromático)
- **Otimização estocástica** (tentativa e erro guiado)

### 5.2 O Algoritmo (Simplificado)

```javascript
function huemintGenerate(config) {
  const { n, minContrast, harmonyType } = config;

  for (let attempt = 0; attempt < 10000; attempt++) {
    // 1. Gerar paleta candidata aleatória
    const candidate = randomPalette(n, harmonyType);

    // 2. Verificar restrições
    if (!checkAccessibility(candidate, minContrast)) continue;
    if (!checkDistinguishability(candidate)) continue;

    // 3. Pontuar qualidade perceptual
    const score = scorePalette(candidate);
    // Score alto = harmonious + distinguishable + accessible

    if (score > bestScore) {
      bestScore = score;
      bestPalette = candidate;
    }
  }

  return bestPalette;
}

function scorePalette(palette) {
  // Combina:
  // 1. Variância de L (lightness) — deve ser BAIXA para harmonia
  // 2. Distância perceptual entre cores — deve ser ALTA para distinção
  // 3. Penalidade por chroma extremo (>0.3 vibra, <0.02 é sem graça)
  // 4. Bônus por APCA score alto contra fundo
  return weightedSum(lightnessVariance, distinguishability, chromaPenalty, apcaBonus);
}
```

### 5.3 Por Que Funciona

Huemint trata geração de paletas como um problema de **OTIMIZAÇÃO COM RESTRIÇÕES.** Milhares de candidatos aleatórios, filtrados por regras duras (acessibilidade, distinguibilidade) e ranqueados por qualidade perceptual.

**É "força bruta com inteligência perceptual."** Não é IA. É geometria + fisiologia + probabilidade.

---

## 6. Construindo Seu Próprio Gerador de Paletas em OKLCH

### 6.1 O Algoritmo Completo

```javascript
/**
 * Gera uma paleta de cores acessível e harmoniosa.
 *
 * @param {object} config
 * @param {number} config.baseHue - Matiz base (0-360)
 * @param {string} config.harmony - 'complementary'|'analogous'|'triadic'|'splitComplementary'
 * @param {number} config.minAPCA - APCA Lc mínimo para texto (default: 60)
 * @param {number} config.n - Número de cores na paleta
 * @returns {Array<{oklch: string, hex: string, role: string}>}
 */
function generatePalette(config = {}) {
  const {
    baseHue = 260,
    harmony = 'triadic',
    minAPCA = 60,
    n = 5,
  } = config;

  const bgLight = { L: 0.98, C: 0, h: 0 };
  const bgDark  = { L: 0.10, C: 0, h: 0 };

  // 1. Gerar matizes pela geometria da harmonia
  const hues = harmonyAngles(harmony, baseHue);

  // 2. Para cada matiz, gerar tons (light → dark)
  const palette = [];
  for (const hue of hues) {
    // Gerar 5 tons: mais claro, claro, base, escuro, mais escuro
    const tones = [
      { L: 0.85, C: 0.08, h: hue, role: 'bg' },      // fundo claro
      { L: 0.65, C: 0.14, h: hue, role: 'secondary' }, // cor secundária
      { L: 0.52, C: 0.20, h: hue, role: 'primary' },   // cor da marca
      { L: 0.40, C: 0.16, h: hue, role: 'hover' },     // hover state
      { L: 0.30, C: 0.12, h: hue, role: 'active' },    // active state
    ];

    for (const tone of tones) {
      // Validar acessibilidade
      const lightAPCA = computeAPCA(tone, bgLight, 'dark-on-light');
      const darkAPCA  = computeAPCA(tone, bgDark, 'light-on-dark');

      if (tone.role === 'primary' && lightAPCA < minAPCA) {
        // Ajustar lightness para passar APCA
        tone.L = adjustLightnessForAPCA(tone, bgLight, minAPCA);
      }

      palette.push({
        ...tone,
        lightAPCA,
        darkAPCA,
        hex: oklchToHex(tone.L, tone.C, tone.h),
        oklch: `oklch(${tone.L.toFixed(2)} ${tone.C.toFixed(2)} ${tone.h})`,
      });
    }
  }

  return palette;
}

function harmonyAngles(type, base) {
  const angles = {
    complementary:       [0, 180],
    analogous:           [0, -30, 30],
    triadic:             [0, 120, 240],
    splitComplementary:  [0, 150, 210],
  };
  return angles[type].map(a => ((base + a) % 360 + 360) % 360);
}
```

### 6.2 A Curva de Chroma (O Segredo)

A distribuição de chroma ao longo dos tons NÃO é linear. É uma curva em sino:

```javascript
function chromaCurve(lightness) {
  // Chroma máximo em torno de L=0.55 (cores médias)
  // Cai para zero nos extremos (L=0 e L=1)

  const peak = 0.55;    // pico do chroma
  const width = 0.25;   // largura da curva
  const maxChroma = 0.22;

  // Gaussian-like centrada em peak
  const curve = Math.exp(-((lightness - peak) ** 2) / (2 * width ** 2));

  // Scale para o chroma máximo desejado
  return curve * maxChroma;
}

// Exemplo:
chromaCurve(0.10) // → 0.003  (quase preto, chroma mínimo)
chromaCurve(0.30) // → 0.07   (subindo)
chromaCurve(0.55) // → 0.22   (PICO — cor da marca)
chromaCurve(0.80) // → 0.04   (caindo)
chromaCurve(0.95) // → 0.002  (quase branco, chroma mínimo)
```

### 6.3 O Gerador com Restrições de APCA

```javascript
function adjustLightnessForAPCA(color, background, targetLc) {
  // Busca binária: encontra o L que dá o APCA desejado
  let lo = 0.05, hi = 0.95;

  for (let i = 0; i < 20; i++) {
    const mid = (lo + hi) / 2;
    const candidateColor = { L: mid, C: color.C, h: color.h };
    const lc = computeAPCA(candidateColor, background, 'dark-on-light');

    if (lc < targetLc) {
      // Preto sobre claro: precisa escurecer = reduzir L
      lo = mid;
    } else {
      hi = mid;
    }
  }

  return (lo + hi) / 2;
}
```

---

## 7. Comparação dos Sistemas de Geração

| Sistema | Método | Espaço de Cor | Acessibilidade | Melhor Para |
|---|---|---|---|---|
| **Material You** | Monte Carlo + CAM16 | CAM16 | ✅ Sim (tokens validados) | Extrair cor de imagem do usuário |
| **ColorBrewer** | Curadoria manual + validação empírica | sRGB (convertível) | ✅ Sim (daltônicos, P&B, projeção) | Visualização de dados |
| **Leonardo (Adobe)** | Rede neural + dataset de design | — (caixa preta) | ❌ Não garantida | Criatividade, exploração visual |
| **Huemint** | Otimização estocástica com restrições | OKLCH | ✅ Sim (APCA) | UI design com acessibilidade |
| **Tailwind Colors** | Curvas manuais refinadas | HSL → sRGB | Parcial | Desenvolvimento web rápido |
| **Radix Colors** | Escalas com restrições de contraste | HSL → sRGB | ✅ Sim (WCAG garantido) | Design systems |
| **Gerador OKLCH** (acima) | Harmonia geométrica + APCA | OKLCH | ✅ Sim (APCA) | Controle total, sem caixa preta |

---

## 8. Para Levar Para Casa

1. **Harmonia cromática é geometria + neurofisiologia.** Complementares equilibram canais oponentes. Análogas excitam o mesmo canal. Isso não é "gosto" — é processamento de sinal na retina.

2. **OKLCH torna a geração de paletas TRIVIAL.** Fixe L e C, varie h pela geometria da harmonia. Pronto: paleta equilibrada, sem pesos visuais diferentes.

3. **A curva de chroma em sino** é o segredo das paletas profissionais. Máximo chroma nos tons médios, mínimo nos extremos (claro e escuro).

4. **Material You = CAM16 + Monte Carlo + paletas tonais.** 1 wallpaper → 65+ cores → 500+ tokens com light/dark.

5. **ColorBrewer é curadoria humana validada.** Para dados, não invente — USE ColorBrewer. É o padrão ouro por um motivo.

6. **Huemint = otimização com restrições.** Força bruta guiada por APCA e harmonia. Sem IA, sem caixa preta, 100% determinístico.

7. **Você pode construir seu próprio gerador em 100 linhas de JavaScript + OKLCH + APCA.** Não é mágica. É geometria + fisiologia.

---

## Referências

- Cynthia Brewer — *ColorBrewer 2.0* (Penn State, 2002-2025). colorbrewer2.org
- Google — *Material Design 3 — Dynamic Color* (2021-2025). Documentação oficial do algoritmo.
- Adobe — *Leonardo* (2020-2025). Gerador de paletas por IA.
- Huemint — *Algorithmic Color Palette Generator* (2021-2025). huemint.com
- Johannes Itten — *The Art of Color* (1961). A base das harmonias cromáticas (pré-perceptual, mas fundacional).
- Ewald Hering — *Zur Lehre vom Lichtsinne* (1878). Teoria do processo oponente.
- Björn Ottosson — *OKLAB* (2020). O espaço que torna tudo isso possível.
- Andrew Somers — *APCA* (2019-2025). O algoritmo de contraste perceptual.
- CIE — *CIECAM02 / CAM16* (2002/2016). Modelos de aparência de cor.
- Tailwind CSS — *Color System* (refinamentos manuais de curvas HSL).
- Radix UI — *Colors* (escalas com contraste garantido por step).

---

*Fim do Estudo de Cores Loop 04. Gerar paletas não é "ter bom gosto." É geometria no círculo cromático, otimização com restrições de acessibilidade, e uma curva de chroma em forma de sino. OKLCH torna tudo isso programável. Cron: segue para loop 05.*
