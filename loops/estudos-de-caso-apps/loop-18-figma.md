# Estudo de Caso 18 — Figma: O App Que Colocou o Design na Nuvem (E Quase Foi Comprado Por $20 Bilhões)

> **Data:** 2026-07-02
> **Loop:** 18 de ∞ (Fase 2: Design & Criação)
> **Categoria:** Design / Colaboração / Ferramentas
> **Tema:** 2012. Dylan Field, um estudante de ciência da computação da Brown University, ganha a **Thiel Fellowship** — $100.000 para largar a faculdade e construir uma empresa. Ele e Evan Wallace (seu TA em Brown) se mudam para San Francisco com uma visão: construir uma ferramenta de design PROFISSIONAL que rode 100% no NAVEGADOR. A ideia era considerada IMPOSSÍVEL. O Photoshop era desktop. O Sketch era desktop. Design era DESKTOP. Mas Field e Wallace perceberam que o **WebGL** — uma API de renderização GPU no navegador, recém-lançada — tinha poder suficiente. Eles passaram **4 ANOS** construindo antes de lançar publicamente. Em 2016, o Figma saiu do stealth. Designers ODIARAM. "Ferramenta de design no BROWSER? Ridículo." Mas havia algo que NENHUMA ferramenta desktop tinha: **multiplayer.** Várias pessoas editando o MESMO arquivo ao MESMO TEMPO. Como Google Docs para design. Em 2022, a Adobe tentou comprar por **$20 bilhões.** Reguladores BLOQUEARAM. A Adobe pagou $1 bilhão de breakup fee e DESCONTINUOU o Adobe XD. Em julho de 2025, o Figma fez IPO. Abriu a $33. Fechou a $115. Market cap: **$68 bilhões** — mais que o TRIPLO da oferta da Adobe. Esta é a história do app que provou que o NAVEGADOR pode substituir o DESKTOP — e que multiplayer não é feature, é PARADIGMA.

---

## 0. A Linhagem: Como Designers Criavam Antes do Browser

```
Papel, caneta, nanquim (pré-digital): prancheta. Régua T. Letraset. Físico. SOLITÁRIO.
      ↓
Photoshop (1990), Illustrator (1987): bitmap + vetor. Desktop. Arquivos .PSD e .AI.
      ↓
Sketch (2010): vetor-first, UI-focused. AINDA desktop. AINDA solo. Arquivos .sketch.
      ↓
Figma (2016): browser. WebGL. Multiplayer. Zero instalação. "Google Docs para design."
      ↓
Adobe XD (2017-2023): resposta da Adobe. FRACASSOU. Descontinuado após deal bloqueado.
      ↓
Figma IPO (2025): $68B. WebGPU. AI (Figma Make). Dev Mode. O "work OS" do design.
```

O Figma não inventou o design digital (Photoshop, Illustrator, Sketch). Inventou o design COLABORATIVO — onde o arquivo não é um ARQUIVO, é um ESPAÇO. Onde designer, PM, dev e stakeholder editam JUNTOS, em TEMPO REAL, de QUALQUER dispositivo. O navegador DEIXOU de ser "limitado demais para design profissional" e VIROU a plataforma PADRÃO.

---

## 1. A Origem: Thiel Fellowship, WebGL e 4 Anos no Stealth

### Dylan Field: O Jovem Que Largou a Faculdade

- Estudava **Ciência da Computação na Brown University**.
- Estagiou no **LinkedIn**, **Flipboard**, **O'Reilly Media**.
- **2012**: ganhou a **Thiel Fellowship** — $100.000 de Peter Thiel para LARGAR a faculdade e construir uma empresa.
- Conheceu **Evan Wallace** (seu TA em Brown). Wallace era um GÊNIO em computação gráfica.

### As Ideias Que Não Vingaram

Field e Wallace testaram VÁRIAS ideias antes do Figma:
- **Drone para pegar motoristas bêbados.** Não foi.
- **Gerador de MEMES com WebGL.** Field se olhou no espelho e pensou: _"I dropped out of Brown for THIS?"_
- **Ferramentas 3D, edição de foto, conteúdo generativo.**

A virada veio quando perceberam que o **WebGL** — uma API nova de renderização GPU no navegador — era POTENTE o suficiente para rivalizar com apps nativos. A maioria das pessoas achava IMPOSSÍVEL. Eles acharam INEVITÁVEL.

### Os Investidores Que Acreditaram (Quando Ninguém Acreditava)

- **Danny Rimer** (Index Ventures) conheceu Field quando ele era estagiário de 19 anos na Flipboard. Liderou a **seed de $1.7M em 2013.**
- **John Lilly** (Greylock) inicialmente RECUSOU. Depois mudou de ideia. Liderou a Série A.
- **Peter Skomoroch**, ex-chefe de Field no LinkedIn, também investiu.

### 4 Anos no Stealth (2012-2016)

Field e Wallace passaram **QUATRO ANOS** construindo antes de mostrar ao mundo. Não era "lean startup." Não era "MVP rápido." Era CONVICÇÃO TÉCNICA PROFUNDA.

Construíram:
- Um **motor de renderização CUSTOM** em C++ compilado para **WebAssembly**.
- Renderização via **WebGL** (GPU-accelerated).
- Um sistema de **multiplayer** em tempo real (CRDT-inspired).
- **Zero instalação.** Abra o navegador. Pronto.

### 2016: O Lançamento Que Ninguém Pediu

Setembro de 2016. Figma sai do stealth.

Designers ODIARAM. "Ferramenta de design... no BROWSER?" "Vai ser LENTO." "Não tem como." "Sketch já resolve."

Mas havia ALGO que o Sketch não tinha: **multiplayer.** Abra um link. Compartilhe. Edite JUNTO. Em TEMPO REAL.

Empresas começaram a ADOTAR. Microsoft, Uber, Airbnb, Shopify. Uma designer convencia o time INTEIRO.

---

## 2. A Filosofia do Produto: "Multiplayer Não É Feature, É o Produto"

### Design Deveria Ser Aberto, na Nuvem, na Web

> *"Design should be more open, cloud-first, and on the Web."* — Dylan Field

O Figma foi CONSTRUÍDO em torno do multiplayer desde o DIA 1. Não foi "adicionado depois" como o Sketch (que TENTOU e FALHOU). O multiplayer é a ARQUITETURA do produto.

### Os Pilares

| Pilar | Significado |
|---|---|
| **Browser-native** | Zero instalação. Windows, Mac, Linux, ChromeOS. Sempre atualizado. |
| **Multiplayer por padrão** | Todo arquivo é COLABORATIVO. Não existe "modo solo." |
| **Compartilhar = link** | Envie um URL. A pessoa ABRE. Pronto. Sem baixar. Sem "versão 3_final_FINAL." |
| **Design systems como wedge** | Componentes compartilhados. Bibliotecas. Times INTEIROS no Figma. Switching cost ALTÍSSIMO. |
| **Comunidade como motor** | Plugins, templates, widgets, Friends of Figma, Config (conferência). |

### "Google Docs Para Design"

Essa frase SIMPLES virou o PITCH inteiro do Figma. Todo mundo entendia Google Docs. Todo mundo entendia o PODER de editar o mesmo documento ao mesmo tempo. Por que DESIGN era diferente?

O Figma RESPONDEU: não é. Design TAMBÉM pode ser colaborativo.

---

## 3. As Inovações Que o Figma Trousse ao Mundo

### 3.1 Design Multiplayer (2016)

Antes: designer trabalha sozinho no Sketch. Exporta PNG. Manda por email. PM responde: "muda a cor do botão." Designer MUDA. Exporta de novo. Manda de novo. LOOP INFERNAL.

Depois do Figma: PM, dev, designer — todos DENTRO do mesmo arquivo. Vendo em TEMPO REAL. Comentando. Inspecionando. O designer edita. O dev JÁ VÊ.

### 3.2 WebGL → WebAssembly → WebGPU (Stack Técnico Revolucionário)

O Figma PROVOOU que o navegador pode rodar ferramentas profissionais com performance NATIVA:
- **C++ → WebAssembly** (Emscripten). Código nativo compilado para o browser.
- **Custom renderer** em WebGL/WebGPU. NÃO usa HTML/SVG/Canvas do browser.
- **Sem GC pauses.** Controle total de memória.
- **2× performance nativa** (LLVM-optimized).

### 3.3 Community (2019): O "App Store" do Design

Plugins, templates, widgets, wireframes, design systems — tudo criado pela COMUNIDADE, disponível DENTRO do Figma.

Isso criou um ECOSSISTEMA que nenhum concorrente consegue replicar. O Figma não é mais "uma ferramenta." É uma PLATAFORMA.

### 3.4 Dev Mode (2023): Fim do "Design-to-Code Handoff"

O desenvolvedor ABRE o Figma. Vê o design. Inspeciona CSS, SwiftUI, Compose. Copia. Cola. Pronto. Sem "Zeplin." Sem "exportar assets." Sem "qual é a font size mesmo?"

### 3.5 Figma Make (2025): AI Gera Protótipos

- "Faça um app de delivery com 3 telas." → Figma gera.
- "Transforme este wireframe num design polido." → Figma faz.

### 3.6 Figma Slides, FigJam, Figma Sites

Expansão para ALÉM do design de UI:
- **FigJam**: whiteboard colaborativo.
- **Figma Slides**: apresentações.
- **Figma Sites (beta)**: publicar designs como WEBSITES.

---

## 4. A Linha do Tempo do Design Visual

```
2012-2015 — Stealth. Sem marca pública.
2016 — Logo: "F" geométrico. Roxo escuro. Clean. Tech.
2017-2019 — Roxo + cinza. UI escura. Tipografia Whyte.
2020-2023 — Brand refinado. Roxo vibrante. Comunidade. Config (evento).
2024 — REBRAND. Figma Sans (Grilli Type). 29 cores. "Unapologetically jumbo." Playground.
2025 — IPO. Config 2025. Figma Make. WebGPU. Sites.
```

### Identidade Visual (Rebrand 2024)

| Elemento | Especificação |
|---|---|
| **Cores** | Roxo (assinatura) + **29 cores** (espectro). Neons. Terrosos. "Linguagem, não sistema." |
| **Tipografia** | **Figma Sans** (Grilli Type, 2024). Grotesco opinativo. Text, Display, Condensed, Mono. "i" com pingo QUADRADO. |
| **Ilustração** | "Primitives" — círculo, quadrado, triângulo como blocos. "Unapologetically JUMBO." Playground. |
| **Motion** | **15 fps** (redução intencional de 60). "Tátil, feito à mão." |
| **Tom** | "Linguagem visual, não sistema de regras." Flexível. Vivo. |

---

## 5. Arquitetura Técnica: C++ no Navegador

### Stack

| Componente | Tecnologia |
|---|---|
| **Core** | C++ → WebAssembly (Emscripten). LLVM-optimized. |
| **Renderer** | Custom tile-based 2D engine. WebGPU (2024+). WebGL (legacy fallback). |
| **UI Shell** | React (DOM) — menus, paletas, toolbars. |
| **Multiplayer** | CRDT-inspired LWW (Last-Writer-Wins). Servidor central. WebSocket. |
| **Infra** | AWS. LiveGraph (realtime DB, schema-based, GraphQL-inspired). |

### Por Que C++ → Wasm?

- **Controle de memória TOTAL.** Sem GC pauses do JavaScript.
- **Dentro de 2× performance nativa.**
- **3× mais rápido que asm.js** no load time.
- **Renderizador customizado** (não HTML/SVG/Canvas — todos têm limitações para zoom, blur, blend modes).

### Multiplayer: Last-Writer-Wins por Propriedade

```
Cliente A: muda posição   ──┐
                              ├── Servidor: último valor por propriedade
Cliente B: muda cor       ──┘
```

- **Propriedades DIFERENTES**: sem conflito.
- **MESMA propriedade**: último a escrever GANHA.
- Mais SIMPLES que OT (Operational Transformation). Perfeito para design tools (≠ editores de texto).

---

## 6. Lições do Figma para Quem Constrói Apps Mobile

### 6.1 "Impossível Técnico" É Uma Janela de Oportunidade

Todo mundo disse: "ferramenta de design profissional no browser? IMPOSSÍVEL." Field e Wallace construíram PROVANDO o contrário. 4 anos no stealth. Stack C++/WebGL.

**Lição**: quando TODO MUNDO acha que algo é impossível, e você DESCOBRE que não é — você tem uma janela de 4-5 anos sem competição.

### 6.2 Multiplayer Não É Feature, É Paradigma

O Figma CONSTRUIU o produto em torno do multiplayer. O Sketch TENTOU ADICIONAR multiplayer depois. FALHOU. Não se adiciona multiplayer. Se ARQUITETA em torno dele.

**Lição**: se seu produto PRECISA de colaboração, a colaboração precisa estar no DNA da arquitetura. Não é "add-on."

### 6.3 Community É Fosso Competitivo

O Figma Community tem milhares de plugins, templates, design systems. Um ecossistema que a Adobe NÃO CONSEGUE replicar — porque é construído pela COMUNIDADE, não pela empresa.

**Lição**: uma plataforma com ECOSSISTEMA é 10× mais difícil de competir do que uma ferramenta isolada.

### 6.4 Links Matam Arquivos

"Figma link" matou "Sketch file." Por quê? Porque link é INSTANTÂNEO. Arquivo precisa ser BAIXADO, ABERTO, VERSIONADO, ENVIADO. Link = zero fricção.

**Lição**: se seu produto gera ARQUIVOS, pergunte: "isso pode ser um LINK?"

---

## 7. Ficha Técnica do App

| Atributo | Valor |
|---|---|
| **Nome** | Figma |
| **Fundação** | Agosto de 2012 |
| **Fundadores** | Dylan Field, Evan Wallace |
| **IPO** | Julho de 2025 (NYSE: FIG). $68B. |
| **Categoria** | Design / Colaboração |
| **Plataformas** | Web, Desktop (Electron), iOS, Android |
| **Preço** | Free. Professional: $12/editor/mês. Organization: $45. Enterprise: $75. |
| **Design System** | Figma Sans (Grilli Type). 29 cores. "Primitives." 15fps motion. |
| **Tech Stack** | C++ → Wasm, WebGPU, React, CRDT-inspired multiplayer, AWS LiveGraph |
| **Concorrentes** | Sketch, Adobe XD (descontinuado), Penpot (open source), Canva |

---

## Fontes e Referências

- [TechCrunch — Designers hated Figma at first, but grew to love it](https://techcrunch.com/2023/06/25/figma-design-tool-history/)
- [Fortune — Dylan Field: From LinkedIn intern to billionaire](https://fortune.com/2022/09/16/dylan-field-figma-adobe-acquisition/)
- [Y Combinator — Figma's $20B, 10 year overnight success](https://www.ycombinator.com/library/Jp-garry-s-channel-figma-s-20b-10-year-overnight-success)
- [Forbes — Figma Stock Price Up Over 250% After IPO (2025)](https://www.forbes.com/sites/zacharyfolk/2025/07/31/figma-stock-price-jumps-250-thursday-after-ipo/)
- [Figma Blog — Building a professional design tool on the web](https://www.figma.com/fr/blog/building-a-professional-design-tool-on-the-web/)
- [Figma Blog — WebAssembly cut Figma's load time by 3×](https://www.figma.com/fr-fr/blog/webassembly-cut-figmas-load-time-by-3x/)
- [Figma Blog — Figma Rendering: Powered by WebGPU](https://www.figma.com/blog/figma-rendering-powered-by-webgpu/)
- [Figma Blog — Under the hood of Figma's infrastructure](https://www.figma.com/fr-fr/blog/under-the-hood-of-figmas-infrastructure/)
- [Figma Blog — Figma on Figma: Evolving Our Visual Language](https://www.figma.com/blog/figma-on-figma-evolving-our-visual-language/)
- [Figma Blog — Just Our Type: The Story of Creating Figma Sans](https://www.figma.com/blog/the-story-of-creating-figma-sans/)
- [Liveblocks — Understanding sync engines: Figma vs Linear vs Google Docs](https://liveblocks.io/blog/understanding-sync-engines-how-figma-linear-and-google-docs-work)
