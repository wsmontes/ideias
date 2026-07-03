# 🔤 Estudo de Fontes Loop 26 — Tipografia Não-Latina: Árabe, Chinês, Devanagari, Cirílico e os Desafios de Projetar Para Scripts Globais

> **Data:** 2026-07-03
> **Loop:** 26 de ∞
> **Tema:** A tipografia ALÉM do alfabeto latino — os desafios TÉCNICOS e CULTURAIS de projetar fontes para árabe, chinês, japonês, coreano, devanagari, e cirílico. Por que 26 letras é FÁCIL — e o que acontece quando você precisa de 65.000 glifos.

---

## 0. A Tese: O Alfabeto Latino É Um Bairro Pequeno Numa Cidade IMENSA

95% da literatura sobre tipografia é sobre o alfabeto LATINO. 26 letras. Mas o mundo tem MAIS de 150 sistemas de escrita. Alguns têm CENTENAS de caracteres. Outros têm MILHARES. O chinês tem DEZENAS DE MILHARES.

> *"Latin script is one small neighbourhood in a vast, complex city of writing systems."* — FontLab, "Beyond the Latin Sandbox" (2025)

Se você é um designer que só trabalha com LATIM, você está atuando em ~5% do território tipográfico. Este loop é um MAPA dos outros 95%.

---

## 1. Árabe — A Escrita Que FLUI

### 1.1 O Básico

| Característica | Descrição |
|---|---|
| **Direção** | Direita para esquerda (RTL) |
| **Letras** | 28 consoantes (vogais são diacríticos) |
| **Formas** | CADA letra tem até 4 formas (isolada, inicial, medial, final) dependendo da posição |
| **Cursividade** | SEMPRE cursiva — letras se CONECTAM |
| **Caligrafia** | Naskh (impressão, livros), Nastaliq (poesia, Persa/Urdu), Thuluth (decorativa), Diwani (oficial) |

### 1.2 O Desafio Técnico

**Uma letra = 4 glifos.** A letra ﻉ (Ayn) tem forma DIFERENTE quando está isolada, no início, no meio, ou no fim da palavra. Isso significa que 28 letras viram ~100 glifos SÓ para as formas básicas — antes de considerar diacríticos, ligaduras, ou variações estilísticas.

**Nastaliq é o PESADELO da tipografia digital.** Chamada de "a noiva da caligrafia," o Nastaliq:
- É DIAGONAL e CASCATEANTE (não horizontal)
- Cada letra muda de forma baseada nas VIZINHAS
- Os diacríticos (pontos, vogais) COLIDEM se não forem ajustados manualmente
- Precisa do motor de renderização **Graphite** (OpenType sozinho NÃO DÁ CONTA)

**Awami Nastaliq** (SIL International, 2.000) é a fonte Nastaliq gratuita mais COMPLETA — desenhada para línguas minoritárias que precisam de MAIS diacríticos do que o Urdu padrão.

**Arapix** (Sylvain Mazas, Novembro 2024) é uma fonte Naskh em PIXEL ART de 12px — e foi "tecnicamente impossível" até o designer usar ligaduras contextuais e kerning customizado para cada colisão de diacríticos.

### 1.3 Fontes Árabes Essenciais

| Fonte | Estilo | Nota |
|---|---|---|
| **Noto Naskh Arabic** | Naskh | Google. Cobertura completa. |
| **Noto Nastaliq Urdu** | Nastaliq | Google. Para Urdu. |
| **Awami Nastaliq** | Nastaliq | SIL. Melhor para línguas minoritárias. |
| **IBM Plex Arabic** | Naskh moderno | Corporativo, limpo. |
| **Arapix** | Naskh pixel | Pixel art. Estreia Nov 2024. |
| **Mirza** | Nastaliq | Amir Mahdi Moslehi. 5-6 anos de trabalho. Usada em Tehran. |

---

## 2. CJK (Chinese, Japanese, Korean) — A Escrita Dos 65.000 Glifos

### 2.1 O Básico

| Característica | Chinês (Hanzi) | Japonês (Kanji + Kana) | Coreano (Hangul + Hanja) |
|---|---|---|---|
| **Sistema** | Logográfico (cada caractere = uma palavra/ideia) | Mix: Kanji (logográfico) + Hiragana + Katakana (silábicos) | Hangul (alfabético!) + Hanja (logográfico, raro) |
| **Caracteres** | 3.000-7.000 (literacy) / 20.000+ (erudito) | ~2.000 (Jōyō Kanji) + kana | 24 letras Hangul (combinam em blocos) |
| **Espaços** | NÃO TEM espaços entre palavras | NÃO TEM espaços entre palavras | TEM espaços entre palavras |
| **Direção** | Horizontal (esquerda→direita) ou VERTICAL | Horizontal ou VERTICAL | Horizontal (vertical raro) |
| **Itálico?** | NÃO existe. Ênfase = dots ou sans alternates | NÃO existe | NÃO existe |

### 2.2 O Desafio Técnico

**65.535 glifos é o LIMITE do formato OpenType.** A **Source Han Sans** (Adobe/Google, v2.005, Junho 2025) ATINGIU esse limite — 65.535 glifos POR peso, 7 pesos, 4 variantes regionais (Simplificado Chinês, Tradicional Chinês Taiwan, Japonês, Coreano). ~450.000 glifos no TOTAL.

**Unicode unifica caracteres parecidos.** O caractere Unicode U+514C é o MESMO codepoint para chinês, japonês e coreano — mas DEVERIA renderizar de forma DIFERENTE em cada país. A solução é o OpenType `locl` (locale feature), mas é IMPERFEITA — um caractere "chinês" vai parecer "errado" num contexto japonês.

**Line-breaking sem espaços.** Chinês e japonês não têm espaços entre palavras. O browser pode quebrar a linha APÓS QUALQUER caractere. Regras complexas (kinsoku) proíbem certos caracteres no início ou fim da linha.

**Texto VERTICAL.** Japonês tradicional é escrito em colunas VERTICAIS (de cima para baixo, da direita para a esquerda). Figma AINDA não suporta texto vertical em 2025. InDesign é a única ferramenta que faz DIREITO.

### 2.3 Fontes CJK Essenciais

| Fonte | Cobertura | Nota |
|---|---|---|
| **Source Han Sans / Noto Sans CJK** | SC/TC/JP/KR | Adobe + Google. 7 pesos, 65K glifos. A referência. |
| **Source Han Serif / Noto Serif CJK** | SC/TC/JP/KR | A versão SERIF. |
| **IBM Plex Sans CJK** | SC/TC/JP/KR | Sandoll, 2025. 7 anos de desenvolvimento. |
| **Xiaolai** | CN | Hand-drawn. AI-generated. Font splitting (209 chunks). |

### 2.4 Variantes Regionais — O Mesmo Caractere, 4 Países Diferentes

O caractere para "osso" (骨) é DIFERENTE em cada país:

| Chinês Simplificado | Chinês Tradicional (Taiwan) | Japonês | Coreano |
|---|---|---|---|
| 骨 (parte de baixo: ⺼) | 骨 (parte de baixo: ⺼) | 骨 (parte de baixo DIFERENTE) | 骨 (forma PRÓPRIA) |

**Uma fonte CJK PROFISSIONAL tem TODAS as 4 variantes.** Isso significa que o mesmo texto chinês renderiza DIFERENTE se o leitor estiver em Beijing, Taipei, Tokyo, ou Seoul.

---

## 3. Devanagari & Scripts Índicos — A Escrita Que PENDUDA

### 3.1 O Básico

| Característica | Descrição |
|---|---|
| **Sistema** | Abugida (cada consoante carrega uma vogal implícita) |
| **Direção** | Esquerda para direita |
| **O que torna único** | A "barra superior" (shirorekha) conecta as letras. Diacríticos PENDURADOS acima e abaixo. |
| **Principais línguas** | Hindi, Marathi, Nepalês, Sânscrito (Devanagari). Tamil, Telugu, Bengali, Gujarati, Gurmukhi... (outros scripts) |

### 3.2 O Desafio Técnico

**Conjuncts (consoantes conjuntas).** Quando duas consoantes se juntam sem vogal no meio, elas formam uma NOVA forma — um "conjunct." Existem CENTENAS de conjuncts. Muitos são IRREGULARES (a combinação não é a "soma" das duas letras).

**Mark positioning é PESADELO.** As vogais são diacríticos que podem aparecer ACIMA, ABAIXO, À ESQUERDA, À DIREITA, ou COMBINADOS. O posicionamento é CRÍTICO e muda com cada letra.

**A shirorekha (barra superior).** A linha horizontal que conecta as letras PRECISA se alinhar perfeitamente. Kerning manual para CADA combinação.

**Fontes variáveis são ESSENCIAIS.** Noto Sans Devanagari tem ~0.5 MB por peso × 7 pesos = 3.5 MB. Multiplique por 10+ scripts índicos. Num smartwatch, isso ESTOURA o orçamento de armazenamento. Fontes variáveis reduzem DRAMATICAMENTE.

### 3.3 Fontes Devanagari Essenciais

| Fonte | Nota |
|---|---|
| **Noto Sans Devanagari** | Google. A workhorse. |
| **Noto Serif Devanagari** | Google. Para editorial. |
| **IBM Plex Devanagari** | Corporativa. |
| **Adobe Devanagari** | Adobe. Premium. |

---

## 4. Cirílico — O "Primo" do Latino (Mas Com Suas Próprias Regras)

### 4.1 O Básico

| Característica | Descrição |
|---|---|
| **Sistema** | Alfabético (como o latino) |
| **Letras** | ~33 (Russo). Outros idiomas têm letras adicionais. |
| **Direção** | Esquerda para direita |
| **Línguas** | Russo, Ucraniano, Búlgaro, Sérvio, Bielorrusso, Macedônio... |

### 4.2 O Desafio Técnico (E POLÍTICO)

**Itálico cirílico é RADICALMENTE diferente do romano.** Enquanto no latim o itálico é uma "inclinação" do romano, no cirílico várias letras MUDAM COMPLETAMENTE de forma no itálico. A letra "т" (t) vira "m" no itálico. Isso NÃO É "inclinação" — é uma letra DIFERENTE. Fontes que só "inclinam" o romano são FALSAS.

**Variações locais são POLÍTICAS.** O Ucraniano tem letras que o Russo NÃO TEM (ex: ґ, є, ї). Usar uma fonte "russa" para texto ucraniano NÃO É só "erro técnico" — é erro POLÍTICO. A fonte precisa suportar o alfabeto ESPECÍFICO do idioma.

**Bulgarian vs Russian.** O búlgaro tem formas DIFERENTES para algumas letras (o "л" búlgaro parece um "Ʌ" latino, o russo parece um "Λ"). Fontes "genéricas cirílicas" frequentemente IGNORAM essas diferenças.

### 4.3 Fontes Cirílicas Essenciais

| Fonte | Cobertura | Nota |
|---|---|---|
| **Noto Sans / Serif** | Russo + línguas minoritárias | Google. Cobertura máxima. |
| **IBM Plex** | Russo, Ucraniano, Búlgaro | Corporativa. |
| **Source Sans / Serif** | Russo | Adobe. |
| **Inter** | Russo (básico) | A workhorse UI. Limitada a Russo. |
| **TT Norms Pro** | Cirílico estendido | TypeType. Premium. Award-winning Cyrillic. |

---

## 5. Os 5 Desafios UNIVERSAIS da Tipografia Multilíngue

| # | Desafio | Descrição |
|---|---|---|
| 1 | **Número de glifos** | Latino: ~400. Árabe: ~1.000. Devanagari: ~1.500. CJK: 20.000-65.000. |
| 2 | **Direção do texto** | RTL (árabe, hebraico) + LTR (latino) no MESMO documento. Vertical (japonês) + Horizontal. |
| 3 | **Métricas diferentes** | Altura, baseline, e proporções são INCOMPATÍVEIS entre scripts. Uma UI que funciona em latim QUEBRA em árabe. |
| 4 | **Fontes "falsas"** | Estender uma fonte latina para árabe ou devanagari SEM entender o script = resultado HORRÍVEL. É preciso um designer NATIVO. |
| 5 | **Custo** | Uma família CJK custa 10-100× MAIS para desenvolver que uma família latina equivalente. |

---

## 6. O Futuro: AI, Variáveis, e Inclusão

- **Fontes variáveis reduzem tamanho de arquivo** — crucial para scripts com MUITOS glifos (CJK, famílias índicas)
- **Incremental Font Transfer (IFT)** — W3C, Q2 2026. Vai tornar fontes CJK VIÁVEIS na web pela primeira vez
- **AI gerando glifos CJK** — modelos ViT + RAG (Dez 2024) podem gerar caracteres chineses INTEIROS a partir de UM glifo de referência
- **Graphite + Harfbuzz** — motores de renderização além do OpenType, essenciais para Nastaliq e scripts complexos
- **Inclusão como padrão** — ferramentas como Figma PRECISAM de suporte RTL e texto vertical. Em 2025, AINDA NÃO TÊM.

---

## Fontes Citadas Neste Loop

| Fonte/Ferramenta | Script | Nota |
|---|---|---|
| **Noto (todas as variantes)** | TODOS | Google. A família mais multilingual do mundo. |
| **Source Han Sans / Noto Sans CJK** | CJK | Adobe + Google. 65K glifos. |
| **IBM Plex (Arabic, Devanagari, CJK, Cyrillic)** | Múltiplos | Corporativa multilíngue. |
| **Awami Nastaliq** | Árabe (Nastaliq) | SIL. Línguas minoritárias. |
| **Arapix** | Árabe (Naskh pixel) | Pixel art. 2024. |
| **Mirza** | Árabe (Nastaliq) | Amir Mahdi Moslehi. Tehran. |
| **Xiaolai** | Chinês | Hand-drawn + AI. |
| **Adobe Devanagari** | Devanagari | Adobe. Premium. |

---

> **Próximo loop sugerido:** Tipografia & Impressão — como a tecnologia de impressão moldou a forma das letras. Ou: **Tipografia para Dispositivos Móveis & Wearables** — como projetar texto para telas minúsculas.
