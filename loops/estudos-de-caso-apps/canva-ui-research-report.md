# Relatorio de Pesquisa: Interface de Usuario (UI) do Canva

## Estudo de Caso UX/UI -- Plataforma de Design com 260M+ MAUs

> **Data:** Julho de 2026
> **Fontes:** Pesquisa web extensiva em ingles, engenharia blog do Canva (canva.dev), documentacao do SDK, analises de terceiros

---

## Sumario

1. [Arquitetura de Componentes](#1-arquitetura-de-componentes)
2. [Padroes de Interacao](#2-padroes-de-interacao)
3. [Canvas e Renderizacao](#3-canvas-e-renderizacao)
4. [Mobile e Cross-Platform UI](#4-mobile-e-cross-platform-ui)
5. [Performance de UI e Polish](#5-performance-de-ui-e-polish)
6. [Padroes de UI de IA (Magic Studio)](#6-padroes-de-ui-de-ia-magic-studio)
7. [Referencias Consolidadas](#7-referencias-consolidadas)

---

## 1. Arquitetura de Componentes

### 1.1 Layout Geral do Editor

O editor do Canva e dividido em quatro areas principais:

| Area | Posicao | Funcao |
|------|---------|--------|
| **Canvas** | Centro | Espaco de trabalho interativo ao vivo, suporta designs multi-pagina |
| **Side Panel** | Esquerda | Abriga materiais-primas -- Templates, Elementos, Texto, Uploads, Projetos, Brand Kit, Apps, Draw |
| **Floating Toolbar** | Topo (contextual) | Aparece acima do canvas quando um elemento e selecionado; adapta opcoes por tipo de elemento |
| **Header/Menu Bar** | Topo | Controles de documento -- compartilhar, baixar, redimensionar, undo/redo, seletor de modo |

**Padroes de design de UX chave:**
- **Progressive disclosure** -- ferramentas aparecem apenas quando relevantes
- **Floating toolbar** (nao fixa) -- mantem o canvas limpo
- **Manipulacao direta** -- arrastar, redimensionar, girar com handles no canvas
- **Seletor de modo**: Alterna entre Editing / Commenting / Viewing

**Fontes:**
- https://www.canva.com/help/glow-up/
- https://www.makeuseof.com/canva-editor-page-overview/
- https://www.enterprisenation.com/learn-something/how-to-use-the-canva-editor/

### 1.2 Side Panel (Painel Lateral Esquerdo)

O side panel contem abas acessiveis por hover (visualizacao temporaria) ou clique (fixar):

| Aba | Conteudo |
|-----|----------|
| **Templates** | Layouts pre-prontos, buscaveis por categoria e estilo; subsecao "Styles" para temas de cor/fonte |
| **Elements** | Formas, stickers, linhas, ilustracoes, icones, bordas, frames, grids, graficos, gradientes, fotos, videos, audio, graficos 3D, imagens geradas por IA; itens premium marcados com icone de coroa |
| **Text** | Headings, subheadings, body text; "Font Combinations" para pares de fontes; Magic Write (geracao de texto por IA) |
| **Uploads** | Midia importada pelo usuario; upload drag-and-drop; contas vinculadas (Google Drive, Instagram) |
| **Projects / Folders** | Designs salvos e pastas organizacionais |
| **Apps** | Integracoes de terceiros e apps internos (Text to Image AI, Bitmoji, etc.) |
| **Draw** | Desenho a mao livre com opcoes de caneta, largura e cor |
| **Brand Kit** (Pro) | Armazenar cores da marca, logos e fontes |

**Comportamento UI:** Hover para "dar uma espiada", clique para fixar; novo clique ou botao de fechar para colapsar.

**Fontes:**
- https://www.canva.com/help/glow-up/
- https://www.makeuseof.com/canva-editor-page-overview/

### 1.3 Floating Toolbar e Controles Contextuais

A floating toolbar e um dos padroes de UX mais distintivos do Canva. Ela aparece no topo do canvas e muda dinamicamente baseada no elemento selecionado:

- **Texto selecionado**: Fonte, tamanho, negrito/itálico/sublinhado/tachado, alinhamento, espacamento entre letras/linhas, cor do texto, efeitos (sombra, contorno, texto curvo), cores de fundo, subscrito/sobrescrito, kerning, ligaturas.
- **Imagem selecionada**: Edicao de imagem, corte, posicionamento, borda, arredondamento, flip; botao "Edit" abre o Magic Studio (background remover, magic eraser, text grab, filtros).
- **Graphic/shape selecionado**: Cor de preenchimento, contorno, transparencia, flip, style matching.
- **Page background selecionado**: Controles de pagina, layout e ferramentas de estilo.

Inclui tambem: **Position** (arrange e camadas), **Transparency** slider, **Group/Ungroup**, **Lock** (posicao ou completo), **Duplicate/Delete**, e menu de tres pontos (vinculacao, traducao, info).

**Fontes:**
- https://education.nsw.gov.au/.../transcript-canva-editing-tools
- https://laurieexcell.com/where-is-the-floating-toolbar-in-canva/

### 1.4 O "Glow Up" -- Redesign de 2024

Anunciado no Canva Create (Maio 2024), implementado em Agosto de 2024 -- primeira grande reformulacao da UI em mais de uma decada:

- Interface redesenhada com tema claro e **painel de objetos flutuante** mostrando as abas mais relevantes
- **Floating toolbar contextual** sugerindo recursos relevantes baseados na tarefa de design atual; secao "more" para ferramentas adicionais incluindo Mockups
- **Homepage renovada**: Secao "Starred" para designs favoritos; tagging de administrador de equipe para "starred for team"
- **Side panel redesenhado**: Navegacao mais facil, fluxo de trabalho simplificado
- **Integracao mais profunda de IA**: Magic Write e Magic Switch mais incorporados
- **Suggestion Mode** em Docs; **comentarios reformulados** com facil expansao/ocultacao/filtro
- **Mode switcher**: Alternar entre Editing, Commenting e Viewing

**Fontes:**
- https://www.canva.com/help/glow-up/
- https://smallbiztrends.com/canva-glow-up-launched/
- https://itbrief.in/story/canva-unveils-ai-driven-magic-studio-revamped-ui-at-event

### 1.5 Arquitetura Tecnica de Componentes

**Stack de tecnologia do editor:**

| Componente | Tecnologia |
|------------|------------|
| Framework | React (SPA) |
| Gerenciamento de Estado | MobX (observable stores) |
| Padrao Arquitetural | Store-Presenter-Component (SPC), similar a MVVM, com Injecao de Dependencia (DI) |
| Renderizacao | WebGL + HTML Canvas API (custom-built, NAO Fabric.js ou Konva.js) |
| Adicionais | WebGPU, WebAssembly, Web Workers para caminhos criticos de performance |

**SPC Pattern breakdown:**
- **Stores** -- contem todo o estado (imutavel, observavel)
- **Presenters** -- logica de negocio, modificam apenas stores (nunca tocam views)
- **Components** -- UI pura, ignorantes de stores/presenters
- **Factories** -- conectam dependencias

O estado central do editor e uma **arvore de objetos `EditorElement`** (cada um com id, tipo, x, y, largura, altura, rotacao, zIndex, opacidade, etc.), renderizada no canvas via React + MobX.

**Fontes:**
- https://www.canva.dev/blog/engineering/behind-the-draw/
- https://www.canva.dev/blog/engineering/adding-responsiveness-to-canvas-design-system/

### 1.6 Sistema de Design: Easel + App UI Kit

**Easel (sistema de design interno do Canva):**

- Serve tres propositos: consistencia, componibilidade, eficiencia
- Sistema de layout baseado em **grid de 8px** (1u = 8px)
- Breakpoints: Small (600px), Medium (900px), Large (1200px)
- Usa CSS custom properties + cascade para props responsivas (ex.: `spacing={{ default: '2u', smallUp: '4u' }}`)
- SSR-friendly, bundle size minimo
- Inspirado pelo Braid Design System e pelo trabalho de Lea Verou sobre custom properties

**`@canva/app-ui-kit` (biblioteca de componentes SDK publica):**

- Baseada em React; versao 5.11.0 (Junho 2025)
- Usada pelos proprios engenheiros do Canva
- **Componentes incluem**: Button, TextInput, NumberInput, Select, ColorSelector, FormField, Text, Title, Rows, Columns, Scrollable, DateInput, InputPill, PillsInput, SearchInputMenu, RadioGroup, Alert, ImageCard, AudioCard, VideoCard, HorizontalCard, Tabs, Accordion, Flyout, FlyoutMenu, Menu, ProgressBar, Pill, icones, e mais.
- Inclui **design tokens** (CSS variables + JS variables), **Figma resource** com componentes correspondentes, **localizacao** (LTR/RTL).
- Envolvido em `<AppUiProvider>` para theming.

**Design Tokens:**

- **CSS Variables**: Prefixo `--ui-kit-` em kebab case
- **JavaScript Variables**: Camel case importados de `@canva/app-ui-kit` como `tokens`
- **Cores nomeadas funcionalmente** (por funcao, nao por matiz): `colorPrimary`, `colorPrimaryFore`, `colorContentFg`
- Auto-adaptacao para modo claro/escuro via mapeamento funcional

**Escala de espacamento (1u = 8px):**

| Unidades | Pixels | Token |
|----------|--------|-------|
| 0 | 0px | `space0` |
| 0.5u | 4px | `space050` |
| 1u | 8px | `space100` |
| 2u | 16px | `space200` |
| 3u | 24px | `space300` |
| 4u | 32px | `space400` |
| 6u | 48px | `space600` |
| 8u | 64px | `space800` |
| 12u | 96px | `space1200` |

**Fontes:**
- https://www.canva.dev/blog/engineering/adding-responsiveness-to-canvas-design-system/
- https://www.canva.dev/docs/apps/app-ui-kit/
- https://www.canva.dev/docs/apps/app-ui-kit/design-tokens/
- https://www.npmjs.com/package/@canva/app-ui-kit

### 1.7 Seletor de Cores, Editor de Gradiente e Controles Tipograficos

**Color Picker UI:**
- Clique no elemento -> color tile na toolbar -> abre painel de cores
- Mostra: Cores do documento, cores da marca (Pro), cores da foto (extraidas de imagens), cores sugeridas
- Botao rainbow (+) abre seletor completo com **entrada de codigo hex**, **eyedropper** (Chrome, Edge, desktop app), **busca por keyword de cor** (ex.: "ocean blue")
- Suporte a CMYK disponivel em Brand Kits para designs prontos para impressao
- Temas de cor -- usuarios Pro/Teams podem criar temas coordenados via Brand Kits

**Gradient Editor:**
- Selecione elemento -> color tile -> aba **Gradient**
- Gradientes pre-prontos para escolher (secao rolavel)
- **Gradientes personalizados**: Ate 10 pontos de cor (color stops), arraste para reorganizar, Styles predefinidos (linear, radial, etc.)
- Fade to transparency suportado
- Texto gradiente disponivel na maioria dos designs (nao em Canva Docs inline text, nem em celulas de grafico/planilha)

**Controles de Tipografia (conjunto completo na floating toolbar):**

- Selecao de fonte: Dropdown com previews, busca, filtros (idioma, Free/Pro)
- Pesos de fonte: Multiplas variantes (seta ao lado do nome da fonte)
- Tamanho do texto: Input numerico com botoes +/- (slider no mobile)
- Cor do texto: Color tile -> painel de cores (gradientes suportados)
- Enfase: Negrito, Italico, Sublinhado, Tachado
- Alinhamento: Esquerda, Centro, Direita, Justificar (nao em Docs)
- Listas: Com marcadores e numeradas
- Espacamento: Sliders de espacamento entre letras e linhas
- Case: Alternar maiusculas/minusculas
- Avancado: Subscrito/Sobrescrito, Kerning, Ligaturas
- Efeitos de texto: Cores de fundo, sombras, texto curvo via botao "Effects"
- Copy/Paste estilo de texto: Alt/Option + Ctrl/Cmd + C/V

**Fontes:**
- https://www.canva.com/help/element-color/
- https://www.canva.com/help/creating-using-gradients/
- https://www.canva.com/help/create-your-first-design-edit-text/
- https://www.canva.com/help/color-and-font-combinations/

### 1.8 Painel de Camadas e Gerenciamento de Elementos

**Acesso:** Selecione elemento -> clique "Position" na toolbar -> aba **Layers**.
No mobile: Selecione elemento -> toque "Layers" na toolbar inferior.

**Funcionalidades do painel de camadas:**
- Visualizacao em estilo de lista com miniaturas de preview e icones de elementos agrupados
- **Drag-to-reorder**: Click e segure e arraste camadas -- indicador de linha mostra ponto de insercao
- **Multi-select**: Shift + clique em multiplas camadas
- **Acoes rapidas** (desktop): Hover na camada -> icone "More" (...) para lock, hide, duplicar, deletar
- **Lock & Hide**: Lock previne movimento acidental; hide permite focar secoes complexas
- **Grouping**: Agrupar/desagrupar do painel; camadas agrupadas mostram icone especial

**Aba Arrange:** Bring to Front, Send to Back, Forward, Backward
**Aba Align:** Alinhar a pagina (elemento unico) ou relativo entre si (multiplos elementos)

**Fontes:**
- https://www.canva.com/help/finding-and-arranging-layers/
- https://www.canva.com/help/layer-group-align/
- https://www.canva.com/help/add-manage-elements/

### 1.9 Menus de Contexto e Interacoes de Clique Direito

O Canva suporta menus de contexto com botao direito para operacoes de copia/colagem em elementos e paginas inteiras:

- **Elementos**: Botao direito no(s) elemento(s) selecionado(s) -> Copy (ou Cut) -> botao direito novamente -> Paste
- **Paginas inteiras**: Botao direito na miniatura da pagina -> Copy -> navegue para outro design -> botao direito na area de miniaturas -> Paste (preserva backgrounds e layout)
- No mobile: gesto de tap-and-hold traz opcoes similares

Itens de menu de contexto tipicamente incluem: Undo, Redo, Copy, Cut, Paste e opcoes especificas de contexto (ex.: Add Row/Column em contextos de tabela).

**Fontes:**
- https://mobbin.com/explore/screens/8e4635c0-2cd9-4cbc-9d5d-e27be7c34f53
- https://designbundles.net/design-school/how-to-copy-from-one-design-to-another-in-canva

### 1.10 Dialogos Modais e Sistema de Paineis

O editor usa um **sistema de side panels** em vez de modais tradicionais para a maioria do conteudo secundario:

- **Edit Page panel**: Abre quando background/miniatura da pagina e selecionado -> clique "Edit" na floating toolbar. Contem Layouts, Styles, ferramentas AI (Resize, Translate, Magic Background, Bulk Create).
- **Comment panel**: Painel lateral direito agregando todos os comentarios; filtra por All, For You, Unread, Suggestions, Resolved; ordena por Recent ou Page.
- **Version History panel**: Abre de File -> Version History ou menu "..."; sidebar na direita mostrando versoes salvas como cards de timeline com timestamps, resumos de alteracoes e avatares de colaboradores.
- **Magic Studio panels**: Integrados ao fluxo de edicao de imagem atraves do botao Edit na floating toolbar, abrindo side panels para recursos de IA.

**Fontes:**
- https://www.canva.cn/en/help/use-page-tools/
- https://www.canva.com/help/comments/
- https://www.canva.com/help/version-history/

### 1.11 Interface de Busca para Templates e Assets

**Busca na Homepage & AI Bar:** Barra de busca central no topo da homepage; alterna entre abas "Templates" e "Your Designs"; filtra por tipo de conteudo, categoria de design, proprietario, data de modificacao, categoria, estilo.

**Busca no editor:**
- **Aba Elements**: Busca por fotos, graficos, icones, formas, ilustracoes, etc. Filtra por cor usando codigos hex ou nomes de cor keyword. Hover sobre elementos mostra keywords associadas.
- **Templates**: Filtra por cor, tema, recurso, estilo. Brand Templates mostrados primeiro para equipes.
- **Canva Assistant** (slash `/` ou `Cmd+E` / `Ctrl+E`): Menu de comando rapido para elementos, templates, fontes e recursos como Background Remover, animacoes.
- **Recursos de busca com IA**: Magic Generate (imagens, rascunhos de documentos, designs a partir de prompts), Magic Design (criacao de design assistida por IA).

**Fontes:**
- https://www.canva.com/help/search-ai-bar/
- https://www.canva.com/en/design-school/resources/using-search-and-magic-design
- https://www.canva.cn/en/help/slash-magic-shortcut-commands-cn/

---

## 2. Padroes de Interacao

### 2.1 Handles de Selecao, Redimensionamento e Rotacao

**Bounding Box e Handles:**
- Quando um elemento e selecionado, uma caixa delimitadora com handles de canto visiveis aparece -- signifiers claros que reforcam as affordances de redimensionar e reposicionar
- **Handles de canto** (circulos brancos nos 4 cantos) permitem **escala proporcional** -- mantem aspect ratio
- **Handles de borda** (pontos medios das arestas) permitem **escala assimetrica** -- estica em uma direcao apenas
- O **rotation handle** fica fora da caixa delimitadora, visualmente distinto dos handles de resize. Click e arraste para esquerda ou direita para girar. Entrada de angulo preciso disponivel via menu Position na toolbar.
- Opcoes de flip (horizontal/vertical) disponiveis via toolbar

**Selecao Aninhada (dentro de grupos):**
- Clique em um grupo, depois clique em um elemento individual dentro dele. O elemento focado ganha borda roxa, enquanto elementos nao focados mostram bordas ciano
- Duplo-clique em um grupo, depois clique em um elemento especifico para edita-lo individualmente sem desagrupar

**Principios de Design de Interacao:**
- Manipulacao direta -- usuarios interagem diretamente com elementos do canvas, reduzindo o "gulf of execution"
- Toolbar sensivel ao contexto muda baseada no tipo de elemento selecionado, usando restricoes logicas para prevenir erros
- Problema conhecido no mobile: modos distintos de selecao e edicao de texto existem, mas a transicao nem sempre e claramente sinalizada, causando "mode errors"

**Fontes:**
- https://www.ixd.prattsi.org/2026/02/design-critique-canva-ios-app/
- https://www.canva.com/help/add-edit-shapes/

### 2.2 Multi-Select e Operacoes de Grupo

**Metodos de selecao:**
- **Shift + Click** -- segure Shift e clique em elementos individuais para adicionar/remover da selecao
- **Click-and-Drag (area selection)** -- clique no canvas vazio, arraste para criar uma caixa de selecao roxa; qualquer elemento parcialmente dentro e selecionado
- **Select All** (`Ctrl+A`/`Cmd+A`) -- seleciona todos os elementos exceto o background

**Operacoes de agrupamento:**

| Acao | Metodo |
|------|--------|
| Group | Botao Group, botao direito > Group, ou `Ctrl/Cmd + G` |
| Ungroup all | Clique no grupo > Ungroup na toolbar, ou `Ctrl/Cmd + Shift + G` |
| Ungroup elemento unico | Selecione um elemento do grupo > Ungroup |
| Editar dentro do grupo | Duplo-clique no grupo, depois clique no elemento individual -- redimensione, gire, mova sem quebrar o grupo |

**Padrao Mobile/Tablet:**
1. Toque no primeiro elemento
2. Toque em "..." More menu > Select Multiple
3. Toque em elementos adicionais (cada um ganha borda roxa, caixa pontilhada branca ao redor do grupo)
4. Toque em Group no More menu

**Alinhamento e camadas em multi-selecao:**
- Position panel oferece: Align elements (relativo entre si), Arrange/Layers (mover para frente/tras na ordem de empilhamento)
- Alinhamento de elemento unico referencia limites da pagina
- Caixas/contornos roxos indicam estado de selecao

**Fontes:**
- https://icedit.org/how-to-select-multiple-elements-in-canva
- https://www.makeuseof.com/how-to-group-ungroup-canva-elements/

### 2.3 Snap-to-Grid e Guias de Alinhamento

**Grids:**
- Ativar via File > View Settings > Show Grid (ou menu View)
- Customizar espacamento do grid em View > Grid Settings (1px a 100px, em pixels ou porcentagens)
- Unidade base do Canva: 8px -- espacamento de grid recomendado

**Snap to Grid:**
- Desktop: Menu View > toggle Snap to Grid
- Mobile: menu de tres pontos > Settings > Add Guides
- Elementos auto-travam nas interseccoes de grid mais proximas quando arrastados

**Smart Guides (linhas de alinhamento dinamicas):**
- Aparecem automaticamente ao arrastar um elemento perto da borda ou centro de outro elemento
- Linhas tracejadas roxas para alinhamento de borda
- Linhas tracejadas azuis para alinhamento de centro
- Textos verdes (ex.: "Left aligned") aparecem quando o snapping e bem-sucedido
- Smart guides estao sempre ativos por padrao -- sem toggle dedicado na interface atual

**Rulers:**
- Shift+R ativa rulers no desktop. Rulers mostram realces cinza ao selecionar/mover elementos
- Arraste dos rulers para criar linhas de guia personalizadas horizontais/verticais

**Position Tool:**
- Calcula e equaliza espacamento entre multiplos elementos selecionados (distribuicao horizontal/vertical)
- Botoes de alinhamento: left/center/right align, vertical center, distribute evenly
- Recurso Tidy Up auto-equaliza espacamento

**Fontes:**
- https://www.canva.com/en_gb/help/adding-and-removing-guides/
- https://www.canva.com/en/design-school/resources/aligning-elements-for-balance
- https://graphicdesignresource.com/canva-rulers-guides-gridlines/

### 2.4 Padroes de Copy/Paste e Duplicacao

**Copy/paste padrao:**
- `Ctrl/Cmd + C` / `Ctrl/Cmd + V` entre designs
- Botao direito > Copy/Paste
- Selecione multiplos elementos via Shift+click ou drag-selection

**Comportamento de paste entre designs:**
- Elementos colados mantem tamanho original mesmo se o destino tiver dimensoes diferentes -- sem auto-escala
- Truque "Paste-in-place": Na Pagina B, nao clique no canvas antes de colar -- o elemento chega nas mesmas coordenadas de pixel da Pagina A

**Duplicate:**
- `Ctrl/Cmd + D` para duplicar elemento(s) selecionado(s) no lugar
- Icone de duplicado acima da pagina para paginas

**Por tras dos panos (Prototype Pattern):**
- Pesquisa academica mostra que o Canva usa o padrao de design Prototype para duplicacao, clonando objetos existentes em vez de reinicializar do zero
- Isso reduziu tempo de execucao em ate 60% e reduziu uso de memoria vs. instanciacao convencional

**Clipboard API (lado desenvolvedor):**
- Canva suporta escrita na area de transferencia via `navigator.clipboard.writeText`
- Leitura da area de transferencia e desabilitada -- apps nao podem ler o clipboard do usuario

**Fontes:**
- https://www.ottiliestudio.com/blog/canva-tricks-to-speed-up-your-workflow
- https://designbundles.net/design-school/how-to-copy-from-one-design-to-another-in-canva
- https://journal.eng.unila.ac.id/index.php/jitet/article/download/6975/3370/16425

### 2.5 UI de Atalhos de Teclado e Descoberta

**Abordagens de descoberta de atalhos do Canva:**

- **Quick Actions Command Palette** -- Pressione `/` ou `Cmd+E` (Mac) / `Ctrl+E` (Windows). Menu de comando buscavel com sugestoes enquanto digita. Busca em elementos, templates, recursos (Background Remover, Magic Write, etc.)
- **Referencia de atalhos de teclado** -- Pressione `?` a qualquer momento para trazer a lista completa de atalhos na tela
- Atalhos **nao estao visualmente expostos na UI** (sem tooltips overlay no hover mostrando shortcuts)

**Categorias completas de atalhos:**

- **Gerais**: Undo (Z), Redo (Shift+Z), Save (S), Select All (A), Copy (C), Paste (V), Duplicate (D), Delete (Backspace/Delete)
- **Criacao de elementos**: T=texto, R=retangulo, C=circulo, L=linha, S=sticky note
- **Formatacao de texto**: Negrito (B), Italico (I), Sublinhado (U), Tachado (Shift+S), Case (Shift+K), alinhamento, tamanho da fonte, espacamento de linhas/letras, listas
- **Operacoes com elementos**: group/ungroup (G/Shift+G), lock (Shift+L), bring forward/backward (]/[), front/back (Shift+]/[), tidy up, navegacao com Tab, nudge com setas, resize com Ctrl+setas, rotacao com Alt+. e Alt+,
- **Zoom**: in/out (+/-), 100% (0), fit (Alt+0), fill (Shift+0)
- **Video**: play/pause (Space), mute (M), loop
- **Modos de visualizacao**: scroll view (Alt+1), thumbnail view (Alt+2), grid view (Alt+3), presentation mode, show layers
- **Comentarios**: add, jump to, submit (Cmd+Enter)

**Fontes:**
- https://www.canva.com/en_au/help/canva-keyboard-shortcuts/
- https://www.canva.com/help/slash-magic-shortcut-commands/

### 2.6 Design de Interacao de Undo/Redo

**Niveis de undo/redo:**

**A. Sessao-based (imediato):**
- `Ctrl/Cmd+Z` (undo), `Ctrl/Cmd+Shift+Z` (redo)
- **Visual timeline slider**: Hover sobre o icone de undo no canto superior esquerdo revela um slider de timeline para percorrer visualmente estados recentes do canvas
- A pilha e **limpa ao recarregar a pagina** -- limitacao conhecida
- Melhoria mais solicitada pela comunidade: persistir stack de undo/redo entre recarregamentos

**B. Version History (macro-nivel, persistente):**
- File > Version History abre painel lateral
- 50 a 1.000 versoes salvas dependendo do plano (Pro, Teams, Education, Nonprofits)
- Cada versao tem timestamp, mostra quem fez a edicao (avatar)
- Preview, restore ou copy de qualquer versao passada
- Auto-salvo e persistente entre sessoes e dispositivos

**Padroes de implementacao (contexto da industria):**
- Historico baseado em pilha (Command pattern) com snapshot ou command objects
- Reconstrucao a partir do estado inicial reproduzindo operacoes ate o ponto selecionado (lida com redimensionamento de canvas entre dispositivos)
- Agrupamento de mudancas logicamente relacionadas (ex.: digitacao caractere-por-caractere agrupada em um unico passo de undo)

**Fontes:**
- https://www.canva.com/help/version-history/
- https://www.canva.com/help/undo-and-redo-changes/

### 2.7 Controles de Zoom e Pan

**Metodos de interacao de zoom:**

- **Atalhos de teclado**: `Ctrl/Cmd + +/-` (zoom in/out), `Ctrl/Cmd+0` (100%), `Ctrl/Cmd+Alt+0` (fit), `Ctrl/Cmd+Shift+0` (fill)
- **Zoom slider**: Na parte inferior do editor -- arraste esquerda/direita ou clique na porcentagem para inserir valor especifico
- **Zoom maximo**: 400% (quando usando o design mode padrao; Whiteboard pode ter zoom diferente)
- **Pinch to zoom**: No trackpad, pinch para zoom in/out. No mobile, gesto padrao de dois dedos

**Zoom na posicao do cursor:**
- Canva faz zoom centrado na posicao do cursor ao usar scroll wheel (comportamento padrao de ferramentas de design profissional)
- A implementacao tecnica usa transformacao de coordenadas: `worldPos = (screenPos - offset) / scale`, depois da mudanca de escala, ajusta offset para que o ponto do mundo permaneca sob o cursor
- Formula: `offset = screenPos - worldPos * newScale`

**Modos de visualizacao:**
- Scroll view (Alt+Ctrl+1)
- Thumbnail/timeline view (Alt+Ctrl+2) -- mostra timeline de paginas
- Grid view (Alt+Ctrl+3) -- para gerenciar multiplas paginas
- Modo tela cheia

**Pan:**
- Arraste area vazia do canvas ou use barras de rolagem
- No trackpad, arraste com dois dedos para pan
- Space bar + drag (padrao comum em muitas ferramentas de design) esta disponivel

**Fontes:**
- https://www.canva.com/help/page-view-settings/
- https://icedit.org/zoom-in-canva
- https://stackoverflow.com/questions/79426053/canva-scroll-and-descroll-on-cursor-position-as-origin

---

## 3. Canvas e Renderizacao

### 3.1 Stack de Tecnologia de Renderizacao Geral

O Canva usa uma **arquitetura de renderizacao hibrida de multicamadas** dependendo da plataforma e tarefa:

**Web (Navegador):**
- **WebGL 2.0** -- backend de renderizacao primario acelerado por GPU para graficos vetoriais, composicao de camadas, keyframes de animacao e frames de preview de IA
- **Canvas 2D** -- renderizador fallback, tambem usado para medicao de texto e graficos simples
- **WebGPU** -- backend emergente de proxima geracao (dual-stack com WebGL 2.0), fornece backends nativos Metal (macOS) e Vulkan (Windows/Linux) com menor overhead de driver
- **Rust + WebAssembly via Vello** -- renderizador 2D focado em computacao GPU (veja Lightspeed Renderer abaixo)
- **SVG DOM** -- usado para renderizacao de caminhos vetoriais da ferramenta de desenho

**iOS (Nativo):**
- **Metal** -- API grafica da Apple para o compositor de video e renderizacao Lottie (via ThorVG renderizando em buffer compartilhado Metal)
- **Core Graphics / UIKit Canvas** -- para composicao de pixels em nivel de SO

**Backend (Servidor):**
- **Instancias GPU AWS** (G5 / NVIDIA A10G) para processamento de imagem por IA (upscaling, inpainting, background removal)
- **FFmpeg** para codificacao/decodificacao de video
- **OpenCV / JavaCV** para tarefas de visao computacional

**Linguagens do Sistema:**
- **Rust** -- cada vez mais a linguagem principal para rendenizacao critica de performance (Lightspeed Renderer, Vello, modulos Wasm centrais)
- **C++** -- compositor de video, bibliotecas legadas (rLottie), motor de video nativo
- **TypeScript/JavaScript** -- camada de UI, componentes React, logica de aplicacao

**Fontes:**
- https://www.canva.dev/blog/engineering/behind-the-draw/
- https://www.lifeatcanva.com/zh/%E5%9C%A8%E6%8B%9B%E8%81%98%E4%BD%8D/6000000000963160/lightspeed-renderer-senior-software-engineer/
- https://aws.amazon.com/cn/solutions/case-studies/innovators/canva/

### 3.2 Lightspeed Renderer (Proxima Geracao)

O Canva esta construindo ativamente um **renderizador de proposito geral de proxima geracao codinome "Lightspeed Renderer"**:

- **Objetivo:** First paint dramaticamente mais rapido, interacoes mais suaves, fidelidade consistente entre navegadores e SOs, renderizacao mais barata para Export e workloads de IA
- **Stack:** Rust + wGPU + Data-Oriented Design + WebAssembly
- **Integracao Vello:** Usa o renderizador 2D [Vello](https://github.com/linebender/vello) focado em computacao GPU (do projeto Linebender / Raph Levien). Engenheiros do Canva sao contribuidores centrais do Vello (commits de `ajakubowicz@canva.com`)
- **Arquitetura ECS:** Entity Component System pattern (emprestado de game engines) direcionado especificamente para WebAssembly
- **Pipeline de texto:** Usa o ecossistema Rust: **Parley** (layout de texto rico) + **Swash** (text shaping) + **Fontique** (enumeracao/fallback de fontes) + **Skrifa** (parsing de fontes TrueType/OpenType) + **Peniko** (vocabulario de renderizacao 2D)
- **Performance:** Vello alcanca ate **177 fps** na cena de teste `paris-30k` em M1 Max a 1600px de resolucao
- **Contribuicoes:** Engenheiro do Canva Andrew Jakubowicz apresentou "Debunking Rust Wasm Performance Myths: Why We Moved Core Business Logic to Rust at Canva" no RustWeek 2026

**Fontes:**
- https://2026.rustweek.org/talks/wasm/
- https://www.lifeatcanva.com/zh/%E5%9C%A8%E6%8B%9B%E8%81%98%E4%BD%8D/6000000000963160/lightspeed-renderer-senior-software-engineer/
- https://skia.googlesource.com/external/github.com/linebender/vello/+/refs/heads/ajakubowicz-blend-earnest-attempt

### 3.3 WebGL vs Canvas 2D

O Canva usa uma **arquitetura WebGL-first, Canvas 2D-fallback**:

| Aspecto | Canvas 2D (CPU) | WebGL (GPU) |
|---------|----------------|-------------|
| Cenas grandes | Performance cai ~10.000 elementos | Suave com 10x-100x mais elementos |
| Framerate de animacao | 15-30 FPS cenas complexas | 55-60 FPS sustentado |
| Latencia de primeiro frame (filtro grande) | 300ms+ | ~40ms |
| Uso de CPU | Alto (loops de pixel single-thread) | Baixo (processamento GPU paralelo) |
| Renderizacao de texto | Nativa, nitida | Requer texture atlas, pode ser borrada |
| Cache de triangulacao | Nao pode cachear; re-triangula a cada frame | Cacheia vertex buffers para aceleracoes massivas |
| Eficiencia de memoria | Cria muitos objetos ImageData temporarios | Reutiliza texturas GPU eficientemente |

**Insight chave:** A capacidade do WebGL de **cachear geometria triangulada** em buffers GPU lhe da uma vantagem massiva durante animacoes. Canvas 2D so pode cachear instrucoes de desenho, nao a geometria resultante.

**Canvas 2D ainda e preferido para:**
- Designs com muito texto (abordagem texture-atlas do WebGL pode borrar texto pequeno)
- Graficos simples abaixo de 10.000 elementos
- Renderizacao de linhas (ambos sustentam 60 FPS neste nivel)
- Compatibilidade (~70% dos dispositivos suportam WebGL 2.0 completamente; Canvas 2D funciona em todos)

**Comportamento de fallback:** Quando o contexto WebGL e perdido (crash de driver, pressao de VRAM, drivers desatualizados, politica do navegador), o Canva degrada silenciosamente para renderizacao CPU software, causando quedas perceptiveis de framerate.

**Fontes:**
- https://pea.lib.pte.hu/bitstream/handle/pea/23611/farkas-gabor-phd-2020.pdf
- https://www.canva.com/en_in/help/webgl-not-enabled/

### 3.4 Arquitetura da Ferramenta de Desenho (Renderizacao SVG Path)

Do artigo do Canva Engineering Blog "Behind the draw", a arquitetura da ferramenta de desenho tem quatro camadas:

1. **DOM Events Layer** -- Componente React Drawing overlay com listeners de pointer events
2. **Dispatch Layer** -- Processa eventos com deteccao de touch slop, despacha para state machine
3. **Application Layer** -- Implementa paint tools: Preview (exibe stroke em tempo real) e Commit (adiciona stroke ao design)
4. **Infrastructure Layer** -- Injecao de Dependencia + Padrao Store-Presenter-Component (SPC) + Gerenciamento de estado MobX

**Tecnicas de Otimizacao SVG:**
- **Arredondamento de coordenadas:** Floats de dupla precisao arredondados para 2 casas decimais (impacto visual negligible, grande reducao de tamanho SVG)
- **Comandos de caminho relativos:** Troca de `C` absoluto para `c` relativo -- reduziu comprimento medio de stroke em ~32%
- **Curvas quadraticas:** Uso de `q` em vez de `c` para mais economia
- **Algoritmo Ramer-Douglas-Peucker:** Simplificacao de linhas, descartando pontos dentro de tolerancia epsilon configuravel
- **Algoritmo Martinez-Rueda-Feito:** Uniao booleana de regioes poligonais auto-cruzantes -- critico porque ~77% dos strokes sao preenchimentos "color-in"
- **Segmentacao de strokes:** Divisao de strokes longos para limitar pontos processados por `perfect-freehand`
- **Native overlay no mobile:** Cordova overlay customizado que bufferiza eventos de entrada mesmo quando a VM JS esta ocupada

**Fontes:**
- https://www.canva.dev/blog/engineering/behind-the-draw/
- https://www.canva.dev/blog/engineering/tag/svg-optimizations/

### 3.5 Pipeline de Renderizacao de Texto

A renderizacao de texto usa uma combinacao de abordagens:

- **Renderizacao de texto nativa do navegador** para texto Canvas 2D padrao
- **Pipeline do ecossistema Rust** para o Lightspeed Renderer:
  - **Parley** -- layout de texto rico (posicionamento x/y de glyphs, quebra de linha, resolucao bidi, selecao de texto)
  - **Swash** -- text shaping (mapeamento Unicode-para-glyph, ligaturas, modificadores emoji, scripts complexos)
  - **Fontique** -- enumeracao de fontes e selecao automatica de fallback
  - **Skrifa** -- parsing de fontes TrueType/OpenType (caminhos vetoriais escalados e hintificados)
  - **Peniko** -- tipos de vocabulario de renderizacao 2D (Color, Gradient, Brush, etc.)
- **APIs de Canvas nivel SO** no mobile para rasterizacao de texto no compositor de video

**Fontes:**
- https://www.canva.dev/docs/apps/design-guidelines/typography/
- https://docs.rs/parley/0.4.0/

### 3.6 Renderizacao de Animacao Lottie (ThorVG)

O Canva migrou a renderizacao Lottie iOS de rLottie (biblioteca C++) para **ThorVG (Thor Vector Graphics)**:

| Metrica | rLottie | ThorVG | Melhoria |
|---------|---------|--------|----------|
| Tempo para gerar frames Lottie (video 1-min) | 10.2s | 2.34s | ~80% mais rapido |
| Pico de uso de memoria | 236 MB | 72 MB | ~70% de reducao |

**Arquitetura ThorVG:**
- Biblioteca C open-source licenciada MIT
- **Cinco camadas:** public headers, shared utilities (math/memory), core renderer (Paint, Canvas, RenderMethod), rendering backends (sw_engine para CPU/SIMD, gl_engine para OpenGL, wg_engine para WebGPU), loaders (SVG, Lottie, raster), savers (GIF, TVG)
- **Selecao automatica de backend** baseada na plataforma
- **Atualizacoes incrementais:** dirty bits (`RenderUpdateFlag`) rastreiam propriedades alteradas, backends so recebem mudancas
- **LottieRenderPooler:** gerencia pools de recursos de renderizacao para atualizacoes frequentes de frame

**Implementacao iOS do Canva:**
- Criou Swift Package envolvendo API C do ThorVG
- Renderiza diretamente em buffer compartilhado com o **compositor de video baseado em Metal** do Canva
- Anteriormente na web: usava renderizador Lottie Web do Airbnb dentro de uma **instancia headless Chrome** (muito intensivo para mobile)

**Fontes:**
- https://lottiefiles.com/de/case-studies/canva
- https://docs.lottiefiles.com/en/runtimes/overview/thorvg

### 3.7 Engine de Composicao de Video

Construido principalmente em **C++ e Rust**, usando:

- **APIs Graficas:** Vulkan, OpenGL, DirectX, Metal (iOS)
- **Multimidia:** FFmpeg (decode/encode), OpenCV (estabilizacao, face tracking)
- **APIs de Canvas do SO** para composicao de pixels, Porter-Duff blending, transformacoes 2D, rasterizacao de texto

**Capacidades de composicao (da patente US 10,008,238 B2 do Canva):**
- Camadas ilimitadas, nested mask layers, pre-composites
- Mixagem de camadas independente de resolucao e framerate
- Porter-Duff blending modes por camada
- Transformacoes 2D (posicao, rotacao, pivot, escala) com keyframing
- Filtros de imagem por camada incluindo chroma keying
- Texto vetorial rasterizado via APIs do SO, totalmente keyframeavel
- Definicoes de template XML para composicao em runtime

**Fontes:**
- https://www.lifeatcanva.com/en/jobs/6000000001102507/senior-software-engineer-native-video-engine-china/
- https://patentimages.storage.googleapis.com/b6/ff/37/d3769132b590c1/US10008238.pdf

### 3.8 Otimizacao de Performance por Nivel de Dispositivo

O Canva emprega uma estrategia de performance multicamadas:

| Nivel de Dispositivo | Configuracao |
|----------------------|--------------|
| Alto (GPU discreta) | WebGL 2.0 + GPU rasterization + OffscreenCanvas |
| Medio (GPU integrada) | WebGL 2.0 + Lite mode + desabilitar animacoes |
| Baixo / hardware antigo | Forcar fallback Canvas + templates estaticos |

**Otimizacoes no lado do navegador:**
- **Lite Mode** ("for low-performance devices") -- desativa cursores ao vivo, transicoes de animacao, smart enhancements
- **OffscreenCanvas** -- composicao de animacao descarregada para thread separada via parametro URL `?force_offscreen_canvas=1`
- **WebGL + WebGPU dual-stack** -- tenta WebGPU primeiro, cai para WebGL 2.0

**Gargalos conhecidos:**
- **Perda de contexto WebGL** degrada para renderizacao por software sem notificacao clara
- **Mismatches de High-DPI** -- `devicePixelRatio` vs escala interna do canvas causa redraws de canvas inteiro
- **Limites de tamanho de textura** -- GPUs integradas antigas com tamanho max de textura <2048px disparam downscaling
- **Dependencia de WebAssembly SIMD** -- Magic Animate requer suporte a CPU AVX2
- **Interferencia de extensoes de navegador** -- bloqueadores de anuncios podem quebrar reatividade da UI

**Fontes:**
- https://www.canva.dev/docs/apps/design-guidelines/performance/
- https://www.canva.dev/docs/apps/design-guidelines/loading/

### 3.9 Renderizacao Colaborativa em Tempo Real

- **RSocket** sobre WebSockets (porta 443) -- protocolo de aplicacao para multiplexacao de canais em tempo real com backpressure por canal
- **Redis PubSub + Streams** para intermediacao de mensagens
- **Protocol Buffers** codificacao binaria (carga de CPU reduzida 30% vs JSON)
- **WebRTC** para atualizacoes de ponteiro do mouse peer-to-peer (~50% usam servidor TURN); lida com 100.000+ atualizacoes de posicao/segundo
- **Exponential backoff com jitter** para reconexao

**Fontes:**
- https://www.canva.dev/blog/engineering/enabling-real-time-collaboration-with-rsocket/
- https://www.canva.dev/blog/engineering/realtime-mouse-pointers/

---

## 4. Mobile e Cross-Platform UI

### 4.1 Arquitetura de Tecnologia Mobile

A arquitetura mobile do Canva e um **stack hibrido personalizado com React Native no centro**.

**O que NAO e:**
- Nao e Cordova/PhoneGap -- nenhuma evidencia de wrapper WebView
- Nao e totalmente nativo (Swift/ObjC no iOS, Kotlin/Java no Android) -- embora modulos nativos existam
- Nao e um wrapper PWA

**O que E:**
- React + React Native + Bazel como stack hibrido "build once, ship native"
- Bibliotecas portateis em **Rust ou C** para codigo critico de performance cross-platform
- Uma **arquitetura hibrida personalizada** que renderiza React DOM na web e React Native no mobile a partir de um unico codebase

O time de **Mobile Platform** do Canva (parte do grupo Client Platform) e dono de como o Canva roda em iOS e Android. Eles estao ativamente migrando um produto maduro e ao vivo para esta nova arquitetura -- nao e um projeto greenfield. As descricoes de vagas afirmam que este e "um dos palpites tecnicos mais importantes que a empresa esta fazendo agora" e um dos "programas de engenharia cross-platform mais ambiciosos" ja tentados, dada a escala.

**Fontes:**
- https://www.lifeatcanva.com/en/jobs/6000000001198542/principal-software-engineer-mobile-platform/
- https://www.lifeatcanva.com/en/jobs/6000000001178797/principal-front-end-engineer-mobile-platform/

### 4.2 Aplicativo Desktop (Electron)

Os aplicativos desktop do Canva para Windows e macOS sao **construidos em Electron**:

- Um membro da equipe Canva confirmou no Product Hunt: "Yup -- It's built on Electron."
- O centro de ajuda chines do Canva declara explicitamente: "O cliente desktop Canva (Windows/macOS) e construido no framework Electron."
- O app e comercializado como fornecendo uma "experiencia nativa de desktop" com capacidades indisponiveis em navegadores:
  - Integracao com central de notificacoes do sistema
  - Abrir abas de design em janelas separadas (pop-out)
  - Abrir designs de links/URLs diretamente no app
  - Distracao reduzida vs. abas do navegador
- Downloads separados para Intel vs. Apple Silicon Macs
- Requisitos minimos: Windows 10 (v1909) ou macOS 12 (Monterey), 2GB RAM (4GB recomendado)

**Fontes:**
- https://www.producthunt.com/p/canva/canva-for-desktop
- https://www.canva.com/help/technical-requirements/

### 4.3 Estrategia Cross-Platform: "UI Definida Uma Vez, Renderizada Em Todo Lugar"

O Canva esta estendendo sua **biblioteca de componentes React existente (Easel)** em uma **biblioteca de componentes React portatil** que renderiza via React DOM na web e React Native no mobile. A camada de design system absorve a complexidade cross-platform para que os times de produto construam uma vez.

Principios chave:
- **"Build once, ship native"** -- unico codebase implanta em iOS, Android, web
- **Design system como camada de abstracao** -- complexidade cross-platform tratada no Easel, nao pelos times de produto
- **"UI defined once, rendered everywhere"** -- multiplas superficies proeminentes do Canva devem entregar experiencias mobile-first antes do final de 2026
- Mais de **800+ engenheiros** construirao com este sistema
- A maioria dos MAU do Canva e agora mobile, impulsionando a urgencia desta evolucao

**Fontes:**
- https://www.lifeatcanva.com/zh/%E5%9C%A8%E6%8B%9B%E8%81%98%E4%BD%8D/6000000001178760/staff-front-end-engineer-design-systems/

### 4.4 Diferencas de UI entre Mobile e Desktop

**Layout de Navegacao:**

| Aspecto | Desktop | Mobile |
|---------|---------|--------|
| Navegacao primaria | Painel lateral esquerdo (vertical) | Barra de abas inferior (horizontal) |
| Create a design | Area superior | Pequeno circulo roxo, canto inferior direito |
| Ferramentas de design | Toolbar no topo | Toolbar inferior (contextual) |
| Menu de recursos | Painel esquerdo | Circulo roxo, canto inferior esquerdo |
| Page viewer | Painel lateral direito | Icone quadrado numerado branco na direita |
| Perfil/Configuracoes | No painel esquerdo | Topo da tela |

**Abas inferiores do Mobile (layout atualizado):**
1. **Create** -- Atalhos Quick Create (substitui antiga aba Home)
2. **Templates** -- Navegar/buscar por categoria
3. **Your Designs** -- Todos os projetos, trabalho recente no topo
4. **More** -- Brand Kit, AI/Magic Studio, Apps, Design School, Lixeira

**Navegacao personalizada** (fixar/desafixar abas) esta disponivel no desktop e Android tablet, mas **ainda nao no mobile**.

**Fontes:**
- https://www.canva.com/en_gb/help/navigate-canva-mobile-app/

### 4.5 Lagoas de Funcionalidades: Desktop vs. Mobile

| Funcionalidade | Web/Desktop | App Mobile (iOS/Android) |
|----------------|-------------|--------------------------|
| Edicao SVG | Suporte completo | Nao disponivel |
| Fontes personalizadas | Upload OTF/TTF | Sem gerenciamento de fontes |
| Ferramentas AI | 25+ ferramentas AI, 17 Magic Studio | So AI text basico, 3 estilos de arte |
| Export: 4K PNG | Sim | Max 2048px PNG |
| Export: SVG, PPTX | Sim | Nao |
| Export: PDF protegido por senha | Sim | Nao |
| Ferramentas de desenho | Controle completo de parametros (0.5-100px, cores hex, pressure sensitivity) | 6 tamanhos predefinidos, 8 cores, sem pressure |
| Colaboracao | Cursor tracking, comentarios encadeados, @mentions, version history | Comentarios simples, sem cursor tracking, historico limitado |

**Fontes:**
- https://www.makeuseof.com/canva-vs-adobe-express-on-mobile/
- https://www.php.cn/faq/2411249.html

### 4.6 Controles de Design Otimizados para Toque

**Pontos fortes:**
- **Manipulacao direta** com bounding boxes e handles de canto para redimensionar/reposicionar -- considerada uma das escolhas de design mais fortes do Canva mobile
- **Guias de alinhamento e snapping** fornecem feedback em tempo real ao arrastar
- **Toolbar inferior contextual** muda baseada no elemento selecionado, mostrando apenas acoes relevantes (reducao de carga cognitiva)
- **Pinch-to-zoom** para ajuste fino e navegacao no canvas

**Desafios de usabilidade conhecidos:**
- **Confusao de modo de edicao de texto**: Tocar no texto o seleciona sem ativar imediatamente o teclado, causando "mode errors" onde usuarios acidentalmente movem/redimensionam texto em vez de edita-lo
- **Ferramentas avancadas ocultas**: Ordem de camadas, z-ordering estao enterrados em menus secundarios -- aumenta carga cognitiva na tela limitada do mobile
- **Trabalho de design multi-camada complexo** e "quase impossivel de fazer mudancas significativas no mobile" segundo reviews de usuarios
- Problemas de performance: auto-saving excessivo, deslocamento de tela durante edicao de texto no iPad
- Selecao de texto pequeno tediosa sem uma stylus

**Otimizacoes especificas do iPad:**
- Suporte a Apple Pencil (1a e 2a geracao; USB-C relatadamente nao suportado) para precisao, pressure sensitivity
- Gestos: tap para selecionar, drag para mover, pinch para zoom, double-tap para opcoes de edicao, split-screen drag-and-drop de Photos/Files
- Sincronizacao em nuvem entre iPad, iPhone, Mac
- Quase paridade total de funcionalidades com desktop no iPad

**Fontes:**
- https://www.ixd.prattsi.org/2026/02/design-critique-canva-ios-app/
- https://en.soydemac.com/How-to-use-Canva-on-iPad-and-get-the-most-out-of-it/

### 4.7 Layout do Painel do App Mobile (para Desenvolvedores)

Conforme documentado no Canva Apps SDK:

- **Desktop**: Painel do app tem ~350px de largura, posicionado a esquerda ou direita baseado na direcao do idioma
- **Mobile**: Painel ajusta para **largura total** do dispositivo
- Recomendacao: empilhar componentes verticalmente usando `<Rows>`, usar botoes de largura total, evitar scroll horizontal, elementos sticky e nested scrollbars
- Para galerias de imagem/video, usar layouts grid (`<Columns>`)
- Testar em diferentes larguras; viewports estreitas sao a restricao mais dificil

**Fontes:**
- https://www.canva.dev/docs/apps/design-guidelines/layout/
- https://www.canva.dev/docs/apps/design-guidelines/mobile/

---

## 5. Performance de UI e Polish

### 5.1 Truques de Performance Percebida

**Diretrizes Oficiais de Loading (App UI Kit):**

1. **Placeholders (skeleton screens)** -- Recomendacao primaria. Formas basicas (circulos, retangulos) arranjadas para imitar estrutura de conteudo. Animacao de flashing por padrao. Podem ser sequenciados com prop `index`. Melhores que spinners "onde possivel."
2. **Progress bars** -- Para tempos de espera de 5 segundos ou mais. Barras determinadas dao mais incentivo ao usuario para esperar.
3. **Animated circle loaders (spinners)** -- Para esperas curtas ou espacos confinados como botoes.

**Diretrizes de Performance:**
- Loading spinners podem melhorar a *percepcao* de performance mesmo sem mudar o tempo real de carregamento
- Use `thumbnailUrl` para previews menores antes da resolucao completa
- Reduza o tamanho do bundle; descarregue tarefas para servidores

**Componentes de loading do App UI Kit:**
- `Placeholder`, `TextPlaceholder`, `TitlePlaceholder`, `ProgressBar`, `LoadingIndicator`
- Disponiveis via `@canva/app-ui-kit` no npm

**Pesquisa academica sobre skeleton screens vs alternativas:**
- Skeleton screens tiveram melhor performance percebida -- 48% dos usuarios classificaram como "rapido" (1-3 segundos)
- Paginas em branco foram as piores -- 52.4% classificaram como "lento/muito lento"
- Tecnicas ciclicas (spinner, skeleton) superaram as determinadas (progress bar)
- Combinar tecnicas (skeleton + spinner) nao melhorou a percepcao

**Fontes:**
- https://www.canva.dev/docs/apps/design-guidelines/loading/
- https://www.canva.dev/docs/apps/design-guidelines/performance/
- https://www.npmjs.com/package/@canva/app-ui-kit
- https://sol.sbc.org.br/index.php/ihc_estendido/article/download/37767/37549/

### 5.2 Animacoes e Transicoes

**Animacoes de elementos:**
- Basicas: Fade, Pop, Rise, Drift, Breathe
- Exageradas: Tumble, Stomp
- Photo Movement: Pan, Zoom
- Efeitos add-on: Flicker, Tectonic, Chromatic
- Timing: 0.1 a 30 segundos por elemento
- Gatilho: On click, On timer, With previous (sequenciamento agrupado)
- **Max 50 animacoes por design, 10 segundos max por animacao**

**Transicoes de pagina:**
- Estilos: Fade, Pan, Rise, Breathe, Glitch, Neon, Tectonic, e mais
- Duracao: 0.1 a 2.5 segundos
- Opcoes direcionais por estilo

**Magic Animate (animacao por IA):**
- IA analisa conteudo (texto, fontes, cores, imagens) e aplica efeitos de entrada, pans, zooms e transicoes de slides
- Aplicacao com um clique de animacao de deck completo

**Design Tokens de Transicao (App UI Kit):**
- `--ui-kit-hover-transition` e `--ui-kit-fade-transition` como CSS + JS design tokens para micro-interacoes consistentes

**Reduce motion settings:**
- Remove transicoes, animacoes de fundo, mascaras, overlays
- Conformidade com acessibilidade WCAG para disturbios vestibulares

**Fontes:**
- https://www.canva.com/help/animate-designs/
- https://www.canva.com/help/page-transitions/
- https://www.canva.com/help/using-magic-animate/
- https://www.canva.dev/docs/apps/app-ui-kit/transitions/
- https://www.canva.com/help/reduce-motion-settings/

### 5.3 Micro-interacoes

O curriculo oficial de certificacao de design do Canva define micro-interacoes como:

- **Hover effects**: Mudancas visuais no hover do cursor
- **Click effects**: Animacoes de feedback no clique
- **Loading animations**: Indicadores de progresso
- Descritas como "dicas sutis em uma conversa" que mantem as interacoes suaves e envolventes

**Transition tokens no App UI Kit:**
- `--ui-kit-hover-transition` para estados de hover
- `--ui-kit-fade-transition` para animacoes de fade

**Atalhos de apresentacao (micro-interacoes durante apresentacoes ao vivo):**
- `C` para confete, `D` para drumroll, `M` para mic drop, `B` para blur, `O` para bolhas

**Fontes:**
- https://baheegonline.com/guides/canva/.../5__Advanced_Design_Techniques.html
- https://www.canva.dev/docs/apps/app-ui-kit/transitions/

---

## 6. Padroes de UI de IA (Magic Studio)

### 6.1 Magic Studio: Apresentacao Geral na UI

As ferramentas de IA do Canva sao agrupadas sob **Magic Studio**, lancado em Outubro de 2023 e expandido significativamente em 2024-2026. E um toolkit de design AI integrado embutido diretamente no editor do Canva -- as ferramentas aparecem contextualmente baseadas no que o usuario selecionou.

**Organizacao do painel lateral esquerdo:**
- Home, Magic Studio, Projects, Templates, Brand Kit, Apps
- No editor: sidebar contextual a direita, painel de navegacao a esquerda
- Gemini-style toolbar no topo aparece contextualmente baseada no elemento selecionado

**Redesign de 2024:**
- Primeiro grande redesign em uma decada
- Floating, contextual editing toolbar que adapta ao tipo de projeto
- Sidebar colapsa para dar mais espaco de trabalho
- Homepage personalizavel (star/favoritar designs, pastas, templates)
- Nova sidebar esquerda para acesso rapido a recursos, projetos, ferramentas AI

**Atualizacao AI 2.0 (Canva Create 2026):**
- Mudou de geracao de prompt unico para **design partner conversacional e agente**
- Painel lateral persistente para conversa de ida-e-volta
- Entrada de voz via icone de microfone
- Categorias iniciais: Design for me, Create an image, Draft a doc, Code for me, Create a video clip

**Fontes:**
- https://www.canva.com/magic/
- https://itbrief.in/story/canva-unveils-ai-driven-magic-studio-revamped-ui-at-event
- https://www.fastcompany.com/90869562/canvas-new-generative-ai-tools-are-now-the-ones-to-beat
- https://www.theverge.com/tech/913068/canva-ai-2-update-prompt-based-editing-availability
- https://indianexpress.com/article/technology/artificial-intelligence/canva-new-visual-suite-2-0-ai-coding-voice-enabled-tools-9936871/lite/

### 6.2 Design de Input de Prompt

**Pontos de acesso para IA:**
- Da homepage: **Search and AI bar** no topo, ou **Canva AI** na sidebar esquerda
- Dentro do editor: Botao **"Ask Canva"** na toolbar abre painel lateral AI
- Teclado: `/` para acessar **Magic Commands** (slash commands)

**Prompt input e pontos de partida ao abrir AI:**
- Caixa de texto para descrever o design desejado
- Botoes de categoria: "Design for me", "Create an image", "Draft a doc", "Code for me", "Create a video clip" (AI 2.0)
- "Inspire me" para prompts de amostra

**Fontes:**
- https://www.canva.com/help/using-canva-ai/
- https://www.canva.com/help/edit-designs-with-ask-canva/
- https://www.canva.com/help/search-ai-bar/
- https://www.canva.cn/en/help/slash-magic-shortcut-command-cn/

### 6.3 Navegacao e Selecao de Resultados de IA

- **Magic Design**: Gera 3-8 variacoes de template mostradas como thumbnails de preview; clique para preview; "Apply all pages" para carregar no editor
- **Magic Media**: Resultados aparecem como grid em um painel lateral; clique para adicionar ao canvas, menu de tres pontos para "Generate more like this", botao de regenerar/refresh
- **Magic Design for Presentations**: Selecione de 3-8 opcoes de template; preview antes de aplicar; ate 10 slides por geracao

**Diretrizes de selecao (App UI Kit):**
- Mostre um **preview** antes de substituir conteudo -- nao surpreenda usuarios com auto-updates
- Use loading spinners ou progress bars durante a substituicao de conteudo

**Fontes:**
- https://www.canva.com/help/use-magic-design/
- https://www.canva.com/help/using-magic-media/
- https://www.canva.com/help/using-magic-presentations/
- https://www.canva.dev/docs/apps/design-guidelines/selection/

### 6.4 Magic Media (Geracao de Imagens AI) -- Interface Detalhada

**Fluxo completo da UI:**

1. **Acesso**: Do editor -> Apps -> Magic Media, ou da homepage -> Canva AI
2. **Seletor de abas**: [Images] [Graphics] [Videos]
3. **Input de prompt**: Caixa de texto de descricao + botao "Inspire me" para prompts de amostra
4. **Style picker**: 26+ estilos artisticos predefinidos (Filmic, Anime, 3D, Watercolour, Pixel art, etc.) + upload de imagem de referencia
5. **Aspect ratio**: Square (1:1), Landscape, Portrait
6. **CTA principal**: Botao "Generate image"
7. **Grid de resultados**: Imagens exibidas em painel; clique para adicionar ao canvas; menu de tres pontos "Generate more like this"
8. **Contador de creditos**: Mostra usos restantes

**Opcoes de estilo incluem:** Filmic, Photo, High flash, Long exposure, Minimalist, Moody, Neon, Soft focus, Vibrant, Dreamy, 3D, 3D model, Anime, Concept art, Gradient, Midcentury, Playful, Psychedelic, Retrowave, Watercolour, Colour pencil, Ink print, Oil painting, Papercut, Stained glass, Pixel art

**Fontes:**
- https://www.canva.com/help/using-magic-media/
- https://www.cnet.com/tech/services-and-software/heres-why-canvas-ai-image-generator-is-my-no-1-recommendation-for-beginners/

### 6.5 Funcionalidades Individuais do Magic Studio

| Funcionalidade | Descricao da UI |
|----------------|-----------------|
| **Magic Write** | Vive dentro de caixas de texto ou Canva Docs -- digite `/` ou clique no icone de varinha roxa Magic Write. Geracao inline sem sair do contexto de design. Ajuste de tom: professional, casual, friendly, persuasive. 100+ idiomas. |
| **Magic Design** | Gatilhado ao iniciar novo design -- barra de busca "Describe your design". Gera 3-12 opcoes de layout completas com paletas de cores, tipografia, posicoes de imagem. Usa biblioteca de templates humanos do Canva como base. |
| **Magic Eraser** | Selecione foto -> Edit image -> Magic Eraser. Brush tool pinta sobre objetos; IA detecta bordas automaticamente e preenche fundo. Nenhuma precisao necessaria. |
| **Magic Morph** | Selecione texto/forma -> Apps -> Magic Morph -> Insira descricao -> Click Magic Morph. Exemplos: "gold inflatable foil balloon", "neon lettering", "zebra stripes". Funciona melhor com fontes grossas e ousadas. |
| **Magic Edit** | Pinte sobre area -> Digite descricao -> IA gera resultado. Exemplos: "turn red shirt to blue", "add a hat", "change background to beach". |
| **Magic Switch** | Abra design -> Magic Switch -> Selecione novo formato, tamanho ou idioma. IA gera versoes adaptadas prontas para ajustes. |
| **Magic Expand** | Selecione foto -> Edit -> Magic Expand -> Arraste handles de corte alem da borda -> Click Magic Expand novamente. IA preenche conteudo alem do quadro original. |
| **Magic Animate** | IA analisa conteudo (texto, fontes, cores, imagens) e aplica efeitos de entrada, pans, zooms, transicoes de slide. Aplicacao com um clique. |

**Fontes:**
- https://dev.to/techsifted/how-to-use-canva-ai-magic-write-text-to-image-and-every-ai-tool-explained-2026-31fh
- https://www.canva.com/help/using-magic-morph/
- https://www.canva.com/help/using-magic-animate/
- https://www.shopify.com/blog/how-to-use-canva-ai

### 6.6 Indicadores de Confianca e Padroes de Disclosure de IA

**Termos de Produto AI (vigentes desde 16 Marco 2026):**
- Usuarios proibidos de: remover/alterar metadados de proveniencia C2PA, enganar outros dizendo que conteudo gerado por IA e feito por humanos, tomada de decisoes juridicamente vinculantes totalmente automatizada
- Usuarios sao donos de seu Output, mas a unicidade nao e garantida
- Deve divulgar conteudo gerado por IA em midias sociais

**Usando Magic Studio com seguranca e legalidade:**
- Canva recomenda explicitamente que usuarios **divulguem conteudo gerado por IA** ao compartilhar
- "Isso ajuda a manter a transparencia e promove confianca entre sua audiencia"
- Usuarios sao, em ultima instancia, responsaveis por revisar todo o conteudo gerado por IA

**C2PA Content Credentials (Canva Create 2025):**
- CPO Cameron Adams confirmou que outputs gerados por IA tem metadados embutidos conforme padrao C2PA
- C2PA embedding permite verificacao da origem do conteudo

**Trust Centre -- Moderacao:**
- Combina modelos ML automatizados, classificadores de conteudo, hash-matching e equipes de revisao humanas
- Moderacao multi-camada: remocao de conteudo, suspensao de conta, sistema baseado em strikes
- **Canva Shield** para seguranca comercial

**Canva AI Trust Index (VerifyWise):** Pontuacao 70/100, Nota A, #9 de 205 empresas avaliadas

**Relatorio de Marketing AI do Canva -- Lacuna de Confianca do Consumidor (2026):**
- 97% dos profissionais de marketing usam IA diariamente; 99% planejam aumentar investimento em IA
- 78% dos consumidores preferem anuncios feitos por humanos
- 52% dos consumidores querem divulgacao do uso de IA
- Mencpces de "AI slop" subiram 9x
- 70% acreditam que sera impossivel dizer se o conteudo e gerado por IA sem divulgacao

**Fontes:**
- https://www.canva.com/policies/ai-product-terms/
- https://www.canva.com/help/using-magic-studio-safely-and-legally/
- https://www.canva.com/en_in/trust/safety/moderation/
- https://verifywise.ai/ai-trust-index/canva
- https://thenextweb.com/news/canva-ai-marketing-consumer-trust-gap-report
- https://www.terms.law/ai-output-rights/canva/

---

## 7. Referencias Consolidadas

### Blogs de Engenharia do Canva
- https://www.canva.dev/blog/engineering/ -- Hub principal
- https://www.canva.dev/blog/engineering/behind-the-draw/ -- Ferramenta de desenho
- https://www.canva.dev/blog/engineering/adding-responsiveness-to-canvas-design-system/ -- Design system responsivo
- https://www.canva.dev/blog/engineering/enabling-real-time-collaboration-with-rsocket/ -- Colaboracao em tempo real
- https://www.canva.dev/blog/engineering/realtime-mouse-pointers/ -- Ponteiros do mouse em tempo real
- https://www.canva.dev/blog/engineering/tag/svg-optimizations/ -- Otimizacoes SVG

### Documentacao do SDK
- https://www.canva.dev/docs/apps/app-ui-kit/ -- App UI Kit
- https://www.canva.dev/docs/apps/app-ui-kit/design-tokens/ -- Design tokens
- https://www.canva.dev/docs/apps/design-guidelines/ -- Diretrizes de design
- https://www.canva.dev/docs/apps/design-guidelines/layout/ -- Layout
- https://www.canva.dev/docs/apps/design-guidelines/mobile/ -- Mobile
- https://www.canva.dev/docs/apps/design-guidelines/loading/ -- Loading states
- https://www.canva.dev/docs/apps/design-guidelines/performance/ -- Performance
- https://www.canva.dev/docs/apps/supporting-drag-drop/ -- Drag-and-drop

### Central de Ajuda do Canva
- https://www.canva.com/help/glow-up/ -- Redesign 2024
- https://www.canva.com/help/canva-keyboard-shortcuts/ -- Atalhos de teclado
- https://www.canva.com/help/finding-and-arranging-layers/ -- Painel de camadas
- https://www.canva.com/help/element-color/ -- Seletor de cores
- https://www.canva.com/help/creating-using-gradients/ -- Editor de gradiente
- https://www.canva.com/help/adding-and-removing-guides/ -- Guias e rulers
- https://www.canva.com/help/version-history/ -- Historico de versoes
- https://www.canva.com/help/navigate-canva-mobile-app/ -- Navegacao mobile
- https://www.canva.com/help/using-canva-ai/ -- Canva AI
- https://www.canva.com/help/use-magic-design/ -- Magic Design
- https://www.canva.com/help/using-magic-media/ -- Magic Media
- https://www.canva.com/help/using-magic-morph/ -- Magic Morph
- https://www.canva.com/help/using-magic-animate/ -- Magic Animate
- https://www.canva.com/help/animate-designs/ -- Animacoes
- https://www.canva.com/help/page-transitions/ -- Transicoes de pagina
- https://www.canva.com/help/reduce-motion-settings/ -- Reduce motion

### Analises e Artigos de Terceiros
- https://www.ixd.prattsi.org/2026/02/design-critique-canva-ios-app/ -- Design critique iOS
- https://www.fastcompany.com/90869562/canvas-new-generative-ai-tools-are-now-the-ones-to-beat -- IA do Canva (Fast Company)
- https://www.fastcompany.com/91169221/canva-new-brand-hides-bigger-ambitions -- Rebrand (Fast Company)
- https://www.theverge.com/tech/913068/canva-ai-2-update-prompt-based-editing-availability -- AI 2.0 (The Verge)
- https://www.cnet.com/tech/services-and-software/heres-why-canvas-ai-image-generator-is-my-no-1-recommendation-for-beginners/ -- Magic Media (CNET)
- https://itbrief.in/story/canva-unveils-ai-driven-magic-studio-revamped-ui-at-event -- Magic Studio (IT Brief)
- https://smallbiztrends.com/canva-glow-up-launched/ -- Glow Up (Small Biz Trends)
- https://thenextweb.com/news/canva-ai-marketing-consumer-trust-gap-report -- Confianca do consumidor (TNW)
- https://www.makeuseof.com/canva-vs-adobe-express-on-mobile/ -- Mobile vs Adobe Express
- https://lottiefiles.com/de/case-studies/canva -- ThorVG case study

### Carreiras/Vagas (Arquitetura)
- https://www.lifeatcanva.com/en/jobs/6000000001198542/principal-software-engineer-mobile-platform/ -- Mobile Platform
- https://www.lifeatcanva.com/en/jobs/6000000001178797/principal-front-end-engineer-mobile-platform/ -- Front-end Mobile Platform
- https://www.lifeatcanva.com/zh/%E5%9C%A8%E6%8B%9B%E8%81%98%E4%BD%8D/6000000000963160/lightspeed-renderer-senior-software-engineer/ -- Lightspeed Renderer
- https://www.lifeatcanva.com/zh/%E5%9C%A8%E6%8B%9B%E8%81%98%E4%BD%8D/6000000001178760/staff-front-end-engineer-design-systems/ -- Design Systems

---

*Fim do relatorio. Pesquisa realizada em Julho de 2026. Todas as informacoes foram extraidas de fontes publicas e podem nao refletir o estado atual exato do produto, que esta em constante evoluicao.*
