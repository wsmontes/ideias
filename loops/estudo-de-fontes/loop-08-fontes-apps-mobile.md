# 🔤 Estudo de Fontes Loop 08 — Fontes para Apps Mobile: Como as Big Techs Escolhem e Customizam Suas Fontes

> **Data:** 2026-07-02
> **Loop:** 8 de ∞
> **Tema:** As fontes de sistema das big techs — San Francisco (Apple), Roboto + Google Sans (Google), Segoe UI (Microsoft), One UI Sans (Samsung). Por que cada empresa criou a SUA fonte em vez de usar uma pronta. As decisões de design, as métricas, as variantes, e como essas fontes chegaram a 5 BILHÕES de dispositivos. E o caso Inter — a fonte que virou o "sistema operacional" da web.

---

## 0. A Tese: Fontes de Sistema São o Design Mais Invisível (E Mais Importante) do Mundo

O usuário médio olha para a tela do celular por 4-6 HORAS por dia. Cada palavra que ele lê — notificações, mensagens, menus, labels, botões — está em UMA fonte de sistema.

**5 bilhões de pessoas** usam fontes de sistema todo dia sem saber o nome delas. **San Francisco** (iOS), **Roboto** (Android), **Segoe UI** (Windows), **One UI Sans** (Samsung) — cada uma foi desenhada por uma equipe de elite para resolver um problema ESPECÍFICO:

- **SF Pro:** "Como fazer uma fonte funcionar de um relógio de 38mm a um monitor de 32 polegadas?"
- **Roboto:** "Como substituir uma fonte criticada como 'Frankenfont' por algo que o mundo INTEIRO vai ler?"
- **Segoe UI:** "Como criar uma fonte que renderize bem em ClearType, subpixel rendering, em 9pt num monitor de 96 DPI?"
- **One UI Sans:** "Como unificar a identidade Samsung em 400+ línguas e 26 sistemas de escrita?"
- **Inter:** "Como ser a fonte padrão da web quando você NÃO é uma big tech?"

---

## 1. San Francisco (SF Pro) — Apple, 2014

### 1.1 Por Que a Apple Criou Sua Própria Fonte?

Até 2015, a Apple usava **Helvetica Neue** como fonte de sistema no iOS. O problema: a Helvetica foi desenhada em 1957 para SINALIZAÇÃO impressa — não para telas de retina de 326 PPI. Em tamanhos pequenos (11px), as apertures fechadas da Helvetica faziam letras como 'a', 'e', 's' virarem BORRÕES.

Em 2014, a Apple lançou o **Apple Watch** — uma tela MINÚSCULA de 38mm. A Helvetica era ILEGÍVEL ali. A Apple precisava de uma fonte NOVA — desenhada do ZERO para telas.

### 1.2 O Design: "Helvetica Numa Dieta Low-Carb"

**SF Pro** (San Francisco) é uma neo-grotesca — como Helvetica — mas com diferenças CRUCIAIS:

| Característica | Helvetica Neue | SF Pro |
|---|---|---|
| **Aberturas** | Fechadas (letras "seladas") | GRANDES (letras "respiram") |
| **Espaçamento** | Apertado | Generoso |
| **x-height** | Alta | Mais alta ainda |
| **Curvas** | Uniformes | Levemente achatadas nas laterais (para caber mais ar entre letras) |
| **Tamanhos ópticos** | NÃO tinha | SIM — Text (≤19pt) e Display (≥20pt) |

**A diferença VISUAL entre SF Pro Text e SF Pro Display:**

- **Text (≤19pt):** Apertures MAIORES, spacing MAIS LARGO, strokes mais GROSSOS, x-height MAIS ALTA. Desenhado para LEGIBILIDADE em tamanho pequeno.
- **Display (≥20pt):** Apertures menores, spacing mais APERTADO, strokes mais FINOS, mais contraste. Desenhado para ELEGÂNCIA em tamanho grande.

O sistema operacional TROCA automaticamente entre Text e Display conforme o `font-size`. Isso é sizing óptico — a MESMA tecnologia que os punchcutters do séc. XVI usavam, agora no seu iPhone.

### 1.3 A Família SF Completa

| Variante | Ano | Uso |
|---|---|---|
| **SF Pro** | 2017 | iOS, macOS — a fonte principal |
| **SF Compact** | 2014 | watchOS — curvas mais achatadas para telas minúsculas |
| **SF Mono** | 2016 | Terminal, Xcode — monoespaçada |
| **SF Pro Rounded** | 2018 | App Saúde, Fitness — cantos arredondados |
| **SF Condensed** | 2016 | Apple News, Stocks, Mapas |
| **SF Expanded** | 2021 | Mapas, Fotos |
| **SF Camera** | 2019 | App Câmera do iPhone 11 Pro — estética "industrial" |
| **SF Arabic** | 2021 | Sistema de escrita árabe |
| **New York** (SF Serif) | 2019 | Apple Books — a parceira serif da SF |

### 1.4 A Linha do Tempo da SF

| Ano | Marco |
|---|---|
| **2014** | SF estreia no Apple Watch (só watchOS) |
| **2015** | SF substitui Helvetica Neue no iOS 9 e Lucida Grande no OS X El Capitan |
| **2017** | SF Pro substitui SF UI no iOS 11 |
| **2019** | New York (serif) lançada para Apple Books |
| **2020** | Fontes VARIÁVEIS — SF Pro ganha eixos wght e opsz |
| **2022** | Larguras VARIÁVEIS — Condensed, Compressed, Expanded integrados |
| **2025** | 20.0d10e1 — 9 pesos, múltiplos eixos, cobertura global |

### 1.5 A Restrição

SF Pro NÃO é open source. Só pode ser usada em apps para plataformas Apple (iOS, macOS, watchOS, tvOS). Você não pode usar SF Pro num app Android ou num site que não seja da Apple. Isso é PROPOSITAL — a SF é parte do "jardim murado" da Apple. A fonte É a identidade.

---

## 2. Roboto + Google Sans — Google, 2011-2025

### 2.1 Roboto (2011): A "Frankenfont" Que Deu Certo

**Christian Robertson** (designer interno do Google) criou a **Roboto** em 2011 para o **Android 4.0 Ice Cream Sandwich**. Ela substituiu a **Droid Sans** (a fonte anterior do Android, desenhada por Steve Matteson da Ascender Corporation).

**A recepção foi BRUTAL:**
- **Stephen Coles** (Typographica): *"A four-headed Frankenfont"* — uma mistura inconsistente de Helvetica, DIN e Univers.
- **Joshua Topolsky** (The Verge): "Clean and modern, but not overly futuristic."
- A comunidade tipográfica DETESTOU. As letras eram inconsistentes — o 'R' parecia de uma fonte, o 'g' de outra, o 'k' de uma terceira.

### 2.2 Roboto 2.0 (2014): A Redenção

No **Google I/O 2014**, **Matias Duarte** (VP de Design) anunciou o **Material Design** — e com ele, uma Roboto COMPLETAMENTE redesenhada:

| Mudança | Antes (2011) | Depois (2014) |
|---|---|---|
| **Largura** | Condensada | Mais larga e redonda |
| **Pontos do i/j** | Quadrados | Redondos |
| **Perna do R** | Curvada | Reta |
| **Número 7** | Curvado | Reto |
| **Pesos** | 4 (Regular, Bold, Italic, Bold Italic) | 6+ (Thin a Black) |

**A Roboto 2.0 foi UM SUCESSO.** É a fonte padrão do Android até hoje (Android 16). Está em **3+ BILHÕES de dispositivos.**

### 2.3 Product Sans (2015): A Fonte Que Não Era Para Texto

Com o **redesign do logo do Google** em 2015, a empresa precisava de uma fonte para os "lockups" de produtos ("Google Maps", "Google Photos"). Nasceu a **Product Sans** — geométrica, geométrica, geométrica. Letras baseadas em círculos. Perfeita para LOGOS. HORRÍVEL para texto corrido.

**Product Sans só funciona em tamanhos GRANDES.** Em 11px, vira um borrão geométrico ilegível.

### 2.4 Google Sans (2018): A Ponte Entre o Geométrico e o Legível

Para resolver o problema, o Google contratou a **Colophon Foundry** (UK) para criar a **Google Sans** — uma versão "utilizável" da Product Sans:

- Mais otimizada para headlines e UI
- Menos circular, mais legível
- Usada em marketing (billboards) e UI (headlines)

**Mas ainda não funcionava bem em texto PEQUENO (body text).** O Google vivia um "dual-font system": Google Sans para headlines, Roboto para body text.

### 2.5 Google Sans Text (2020): Finalmente, Body Text

A Colophon Foundry criou uma versão para corpo de texto:

- Caracteres mais ALTOS e CONDENSADOS que Google Sans
- Espaçamento MAIOR entre letras
- Numerais MENOS geométricos
- Proporções IGUAIS às da Roboto (para facilitar a troca)

**Primeiro uso:** Pixel 3. Depois, Material You (Android 12).

### 2.6 Google Sans Flex (2025): O Futuro É Open Source

Em 2025, o Google deu o passo MAIS SURPREENDENTE: **open-sourceou o Google Sans e Google Sans Flex.**

**Google Sans Flex** (Font Bureau & Pathfinders, vencedora Red Dot 2024) é uma fonte variável com **6 eixos**:
- `wght` — Peso (do sussurro ao grito)
- `wdth` — Largura
- `opsz` — Tamanho óptico (do smartwatch ao billboard)
- `slnt` — Inclinação
- `GRAD` — Grade (ajusta espessura sem mudar métricas)
- `ROND` — Arredondamento (de "sério" a "brincalhão")

**E lançaram também Google Sans Code** — uma monoespaçada para código (usada no Gemini).

### 2.7 A Família Roboto/Google Completa

| Fonte | Ano | Uso |
|---|---|---|
| **Roboto** | 2011 | Android — a fonte de sistema |
| **Roboto Slab** | 2013 | Slab serif — headlines |
| **Roboto Mono** | 2015 | Monoespaçada — código |
| **Roboto Flex** | 2021 | Roboto VARIÁVEL — 13 eixos |
| **Roboto Serif** | 2022 | Serif — 4 eixos variáveis |
| **Product Sans** | 2015 | Brand — logotipos de produtos |
| **Google Sans** | 2018 | Headlines UI + marketing |
| **Google Sans Text** | 2020 | Body text |
| **Google Sans Flex** | 2025 | Variável 6 eixos — OPEN SOURCE |
| **Google Sans Code** | 2025 | Monoespaçada para código |

---

## 3. Segoe UI — Microsoft, 2004

### 3.1 A Fonte Que Nasceu de Um Processo

**Steve Matteson** (o mesmo que fez a Droid Sans para Android) desenhou a **Segoe** na **Agfa Monotype**. A Microsoft licenciou a fonte para substituir a **Franklin Gothic** (branding) e **Tahoma** (UI).

Em 2004, a Microsoft registrou a Segoe como design original na União Europeia. A **Linotype** (criadora da Frutiger) protestou — a Segoe era MUITO similar à Frutiger. A **UE revogou o registro** em 2006. A Microsoft não recorreu.

**Mas a Segoe NÃO é uma cópia da Frutiger.** Matteson fez escolhas deliberadas para otimização de TELA:

> "I decided early on to keep the terminals squared-off and vertical... A straight cut gave a crisper result and more predictable rendering. I also emphasized a generous x-height and open counters."
> — **Steve Matteson**

Frutiger foi desenhada para o aeroporto Charles de Gaulle (sinalização IMPRESSA). Segoe foi desenhada para **ClearType** (subpixel rendering em LCDs de 96 DPI). São filosofias DIFERENTES.

### 3.2 A Evolução da Segoe

| Ano | Versão | Marco |
|---|---|---|
| **2004** | Segoe UI | Licenciada pela Microsoft |
| **2007** | Segoe UI | Default no Windows Vista — substitui Tahoma |
| **2009** | Segoe UI Light, Semibold | Windows 7 |
| **2012** | Segoe UI Semilight | Windows 8, Metro design, otimizada para 11px |
| **2015** | Segoe UI Historic | Windows 10, scripts antigos (hieróglifos, cuneiforme) |
| **2021** | **Segoe UI Variable** | Windows 11 — Display/Text/Small + geométrica redesenhada |
| **2024** | Segoe UI Emoji (COLRv1) | 3D Fluent Emoji — 3.000+ emojis em fonte híbrida |

**A Segoe UI Variable (Windows 11)** foi um redesign RADICAL:
- Glyphs regravados — de humanista para mais GEOMÉTRICO
- Display (grande), Text (médio), Small (pequeno)
- Melhor scaling em alta DPI (telas 4K+)

### 3.3 A Família Segoe

| Variante | Uso |
|---|---|
| **Segoe UI Variable** | Windows 11 — Display/Text/Small |
| **Segoe UI Emoji** | Emoji (3D Fluent, COLRv1) |
| **Segoe MDL2 Assets** | Iconografia UWP (legado) |
| **Segoe Fluent Icons** | Iconografia moderna |
| **Segoe UI Symbol** | Símbolos Unicode |
| **Segoe UI Historic** | Scripts antigos |
| **Segoe Print / Script** | Manuscritas |
| **Segoe Boot** | Tela de boot (esticada verticalmente) |
| **Segoe Pro** | Branding interno |

---

## 4. One UI Sans — Samsung, 2024

### 4.1 A Troca Rápida

A Samsung teve **3 fontes de sistema** em 9 anos:

| Período | Fonte |
|---|---|
| **Até 2016** | Fontes genéricas (Arial, Helvetica, Roboto) |
| **2016-2023** | **SamsungOne** — 26 sistemas de escrita, 400+ línguas, 25.000+ glyphs. Humanista. |
| **2024-presente** | **One UI Sans** — Grotesca, variável, minimalista. |

**Por que a troca?** A SamsungOne era humanista — quente, amigável, mas com personalidade "genérica." A **One UI Sans** é uma grotesca CONTEMPORÂNEA — mais limpa, mais geométrica, mais "premium." Alinha-se ao design system **One UI** (substituindo o Samsung Experience/TouchWiz).

### 4.2 Características da One UI Sans

- **Classificação:** Grotesca sans-serif
- **Variável:** SIM — `OneUISans-VF.ttf`
- **Personalidade:** Clean, minimalista, altamente legível
- **Cobertura:** Global (OneUISansKR-VF.ttf para coreano)

---

## 5. Inter — A Fonte Que Virou o "Sistema Operacional" da Web

### 5.1 A Origem

**Rasmus Andersson** (designer sueco, ex-Spotify, ex-Figma) criou a **Inter** em 2017 porque estava CANSADO de fontes de UI que não funcionavam em 11px no Figma.

**Inter foi desenhada especificamente para:**
- Tamanho 11-12px (o padrão de UI)
- Telas de computador (baixa/média DPI)
- Legibilidade MÁXIMA

### 5.2 Por Que a Inter VENCEU

| Fator | Por Que Importa |
|---|---|
| **x-height ALTÍSSIMA** | Em 11px, minúsculas são grandes e legíveis |
| **Apertures GRANDES** | 'c', 'e', 's' nunca "entopem" |
| **Open source (SIL OFL)** | Qualquer um pode usar em qualquer projeto |
| **Variável** | 1 arquivo, wght 100-900, opsz |
| **9 pesos** | Thin a Black com itálicos verdadeiros |
| **Google Fonts** | CDN grátis, carregamento instantâneo |

**Inter é a fonte padrão de:**
- **Figma** (a ferramenta de design onde a web é desenhada)
- **GitHub** (editor de código e界面)
- **Vercel** (plataforma de deploy)
- **Linear** (app de gestão de projetos)
- **Notion** (parcialmente)
- Milhares de startups e landing pages

**Inter é a "fonte do usuário avançado."** Se você vê Inter num site, sabe que o designer pensou em tipografia. Ela virou um sinal de "bom gosto" na web.

---

## 6. Comparação Direta: As 5 Fontes de Sistema

| Característica | SF Pro (Apple) | Roboto (Google) | Segoe UI (Microsoft) | One UI Sans (Samsung) | Inter |
|---|---|---|---|---|---|
| **Ano** | 2014 | 2011 | 2004 | 2024 | 2017 |
| **Classificação** | Neo-Grotesca | Neo-Grotesca/Humanista | Humanista → Geométrica | Grotesca | Neo-Grotesca |
| **x-height** | Muito alta | Alta | Alta | Alta | MUITO ALTA |
| **Aberturas** | Grandes | Grandes | Médias | Grandes | GRANDES |
| **Variável?** | ✅ wght, wdth, opsz | ✅ wght, opsz (Roboto Flex: 13 eixos) | ✅ wght, opsz | ✅ wght | ✅ wght, opsz |
| **Pesos** | 9 (Ultralight a Black) | 9 (Thin a Black) | 6 (Light a Black) | Variável | 9 (Thin a Black) |
| **Licença** | Proprietária (Apple only) | Apache 2.0 (open source) | Proprietária (Windows) | Proprietária (Samsung) | SIL Open Font License |
| **Tamanhos ópticos** | Text (≤19pt) + Display (≥20pt) | Não (original) / opsz (Flex) | Display + Text + Small | Via opsz | Via opsz |
| **Dispositivos** | ~2 bilhões | ~3 bilhões | ~1.5 bilhão | ~1 bilhão | Web (incontável) |

---

## 7. O CSS System Font Stack: Como Usar Tudo Isso

```css
/* A font stack canônica para 2025 */
body {
  font-family:
    -apple-system,            /* SF Pro (macOS/iOS) */
    BlinkMacSystemFont,       /* SF Pro (macOS Chrome) */
    'Segoe UI Variable',      /* Windows 11 */
    'Segoe UI',               /* Windows 10 */
    Roboto,                   /* Android */
    'One UI Sans',            /* Samsung Galaxy */
    Inter,                    /* A "fallback premium" */
    system-ui,                /* O browser decide */
    sans-serif;               /* Último recurso */
}
```

**Ou a versão minimalista moderna:**
```css
body {
  font-family: system-ui, sans-serif;
}
```

`system-ui` (suportado em TODOS os browsers modernos desde 2021) automaticamente usa:
- **SF Pro** no macOS/iOS
- **Segoe UI Variable** no Windows 11
- **Roboto** no Android
- **One UI Sans** no Samsung

---

## 8. Qual Fonte Usar no Seu App?

| Cenário | Recomendação | Por Quê |
|---|---|---|
| **App iOS nativo** | SF Pro | Já está no sistema. Zero download. Otimizada para Apple. |
| **App Android nativo** | Roboto | Já está no sistema. Material Design nativo. |
| **App React Native / Flutter** | system-ui stack | Cada plataforma renderiza sua fonte nativa. |
| **App com branding próprio** | Inter ou Google Sans Flex | Consistência visual cross-platform. Open source. |
| **Landing page / Site** | Inter | Open source, Google Fonts CDN, 9 pesos, variável. |
| **Dashboard / SaaS** | Inter | Legibilidade a 11px, x-height máxima, aberturas enormes. |
| **Editorial / Blog** | Georgia, Merriweather, ou Source Serif + Inter | Serif para leitura + sans para UI. |
| **App de código / terminal** | SF Mono, Roboto Mono, ou **IBM Plex Mono** | Monoespaçada com distinção clara entre 0/O, 1/l/I. |

---

## 9. A Tendência: Fontes Próprias Como Vantagem Competitiva

Em 2025, ter uma fonte PRÓPRIA é sinal de maturidade de design:

| Empresa | Fonte Customizada | Ano |
|---|---|---|
| **Apple** | SF Pro | 2014 |
| **Google** | Google Sans Flex | 2025 |
| **Microsoft** | Segoe UI | 2004 |
| **Samsung** | One UI Sans | 2024 |
| **Airbnb** | Airbnb Cereal | 2018 |
| **Netflix** | Netflix Sans | 2018 |
| **Uber** | Uber Move | 2018 |
| **Shopify** | Shopify Sans | 2021 |
| **Stripe** | Custom (baseada em Inter) | — |
| **Linear** | Inter | 2019 |

**Por que as empresas pagam $100K-$500K por uma fonte customizada?**

1. **Licenciamento.** Fontes comerciais cobram por pageview/instalação. Uma fonte própria elimina esse custo PARA SEMPRE.
2. **Identidade.** A fonte é a "voz" da marca. Nenhuma fonte pronta fala EXATAMENTE como você.
3. **Controle.** Você controla pesos, caracteres, e otimizações para SEUS casos de uso.
4. **Diferenciação.** Num mar de apps usando Roboto e Inter, uma fonte própria é um sinal VISÍVEL de investimento em design.

---

## Fontes Citadas Neste Loop

| Fonte | Empresa | Ano | Status |
|---|---|---|---|
| **SF Pro** | Apple | 2014-2025 | Proprietária (Apple platforms only) |
| **SF Compact** | Apple | 2014 | Proprietária (watchOS) |
| **SF Mono** | Apple | 2016 | Proprietária |
| **New York (SF Serif)** | Apple | 2019 | Proprietária |
| **Roboto** | Google/Christian Robertson | 2011 | Apache 2.0 (Open Source) |
| **Roboto Flex** | Google | 2021 | Apache 2.0 |
| **Roboto Serif** | Google | 2022 | Apache 2.0 |
| **Roboto Mono** | Google | 2015 | Apache 2.0 |
| **Product Sans** | Google | 2015 | Proprietária |
| **Google Sans** | Google/Colophon | 2018 | Open Source (2025) |
| **Google Sans Flex** | Google/Font Bureau | 2025 | SIL Open Font License |
| **Google Sans Code** | Google | 2025 | Open Source |
| **Segoe UI** | Microsoft/Steve Matteson | 2004 | Proprietária (Windows) |
| **Segoe UI Variable** | Microsoft | 2021 | Proprietária |
| **One UI Sans** | Samsung | 2024 | Proprietária (Samsung) |
| **SamsungOne** | Samsung | 2016 | Proprietária |
| **Inter** | Rasmus Andersson | 2017 | SIL Open Font License |
| **Airbnb Cereal** | Airbnb/Dalton Maag | 2018 | Proprietária |
| **Netflix Sans** | Netflix/Dalton Maag | 2018 | Proprietária |
| **Uber Move** | Uber | 2018 | Proprietária |
| **Shopify Sans** | Shopify/Monotype | 2021 | Open Source |
| **Droid Sans** | Ascender/Steve Matteson | 2007 | Apache 2.0 (substituída por Roboto) |
| **Helvetica Neue** | Linotype | 1983 | Comercial (substituída por SF Pro no iOS) |

---

> **Próximo loop sugerido:** Fontes Display e Decorativas — os tipos que NÃO são para texto. Script, Blackletter, Fat Faces, e as fontes experimentais. Ou: **Tipografia para Web Performance** — como carregar fontes sem destruir seu Core Web Vitals.
