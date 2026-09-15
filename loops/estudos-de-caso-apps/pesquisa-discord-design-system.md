# Relatorio de Pesquisa: Sistema de Design, Modo Escuro e Acessibilidade do Discord

> **Data:** 3 de julho de 2026
> **Foco:** Engenharia de design tokens, modo escuro, acessibilidade, tipografia e ciencia da cor
> **Idioma:** Portugues brasileiro

---

## Sumario

1. [Arquitetura de Design Tokens (Mana)](#1-arquitetura-de-design-tokens-mana)
2. [Engenharia do Modo Escuro](#2-engenharia-do-modo-escuro)
3. [Acessibilidade: Auditorias e Conformidade](#3-acessibilidade-auditorias-e-conformidade)
4. [Ciencia Tipografica: gg sans](#4-ciencia-tipografica-gg-sans)
5. [Ciencia da Cor: Blurple e Contraste](#5-ciencia-da-cor-blurple-e-contraste)
6. [Hallacao e Percepcao de Contraste em Modo Escuro](#6-hallacao-e-percepcao-de-contraste-em-modo-escuro)
7. [Conclusoes e Recomendacoes](#7-conclusoes-e-recomendacoes)
8. [Bibliografia](#8-bibliografia)

---

## 1. Arquitetura de Design Tokens (Mana)

### 1.1 Visao Geral do Mana

O sistema de design do Discord chama-se **Mana**. E mantido por uma equipe multifuncional de Design Systems que engloba engenheiros Senior e Staff, com salarios divulgados entre $196.000 e $279.000 mais equity (fonte: vagas publicas oficiais do Discord via Greenhouse).

O Mana e composto por tres pilares:
- **Design tokens** (primitivas de design: cores, espacamento, tipografia)
- **Componentes de UI** (reutilizaveis e acessiveis)
- **Padroes de codigo** (patterns compartilhados entre plataformas)

### 1.2 Arquitetura Cross-Platform

O Mana e explicitamente _cross-platform_, projetado para funcionar em todas as plataformas do Discord:

| Plataforma | Tecnologias |
|---|---|
| Web (Desktop) | React, TypeScript |
| iOS | Swift, Objective-C, React Native |
| Android | Kotlin, React Native |

As descricoes de vagas afirmam: _"Somos uma equipe cross-platform e acreditamos que engenheiros devem ter autonomia para trabalhar em diversas pilhas tecnologicas."_ ([fonte](https://jobs.accel.com/companies/discord-2/jobs/48874253-senior-software-engineer-design-systems))

### 1.3 Engenharia de Temas: O Artigo "Light Theme, Redeemed"

O post mais relevante sobre a arquitetura de design tokens do Discord e o artigo **"Light Theme, Redeemed"** do blog de engenharia do Discord ([fonte](https://discord.com/blog/light-theme-redeemed)). Ele descreve:

**Sistema de Mapeamento de Variaveis CSS:**
O Discord criou um sistema onde engenheiros atribuem uma unica variavel como `--foreground-1` que **auto-seleciona a cor correta** para cada tema (claro/escuro). A paleta de cores primarias foi "forkada" em duas paletas separadas (uma para dark, uma para light).

**Convencao de Nomenclatura:**
As variaveis seguem o padrao `--{categoria}-{numero}` (ex: `--background-1`, `--foreground-2`). Isso permite que "se escolhermos ajustar o mapeamento, isso se propagaria pelo aplicativo com uma unica mudanca."

**Auto-Themer no Figma:**
Construiram um plugin Figma chamado **Auto Theme** ([codigo aberto no GitHub](https://github.com/destefanis/auto-theme)) que aplica os mesmos mapeamentos de cores nos arquivos de design, garantindo consistencia entre design e implementacao.

### 1.4 Ferramentas Internas e Open Source

A equipe do Mana desenvolve ferramentas que foram disponibilizadas como open source no GitHub sob o perfil do designer Daniel Destefanis ([fonte](https://discord.com/blog/building-open-source-design-tools-to-improve-discords-design-workflow)):

- **Auto Theme**: Plugin Figma que alterna automaticamente cores entre temas claro/escuro baseado em arquivos de mapeamento
- **Design Lint** (~468 stars): Linter Figma que sinaliza cores que nao pertencem ao design system, estilos de texto ausentes, raios de borda inconsistentes -- essencialmente "forca" o uso de design tokens na fase de design ([fonte](https://github.com/destefanis/design-lint))
- **Table of Contents**: Gera paginas de capa para navegacao ([fonte](https://github.com/destefanis/figma-table-of-contents))
- **Inspector**: Meta-informacao sobre camadas selecionadas (como DevTools no Figma) ([fonte](https://github.com/destefanis/figma-style-inspector))
- **Pipeline de assets**: Sistema automatizado de exportacao de ativos
- **Codemods**: Scripts de migracao automatica de codebases legados para novos padroes
- **Regras de lint**: Para garantir conformidade com o design system

### 1.5 Plugin PostCSS de Temas (Open Source)

Em 2023, o Discord open-sourcou o **`postcss-theme-shorthand`** ([fonte](https://github.com/discord/postcss-theme-shorthand)), um plugin PostCSS que permite a engenheiros escrever propriedades CSS com prefixos `light-` e `dark-` em uma unica regra. O plugin as expande em regras com escopo `.theme-light` / `.theme-dark`, permitindo **colocar ambos os temas em um unico bloco de regras** em vez de espalhar valores entre secoes.

```css
/* Entrada (codigo do engenheiro) */
.button {
  light-background: white;
  dark-background: black;
}

/* Saida (compilado) */
html.theme-light .button {
  background: white;
}
html.theme-dark .button {
  background: black;
}
```

Esta e a abordagem de engenharia do Discord para manter suporte eficiente a temas duais com manutencao centralizada.

### 1.6 Banco de Dados Completo de Tokens (443 Raw + 302 Semantic)

O repositorio **VendettaThemeUtil** ([fonte](https://github.com/nexpid/VendettaThemeUtil)) rastreia o sistema de tokens de cores do Discord com valores hex para modo claro/escuro em versoes versionadas (200.0 a 234.7). E a referencia open-source mais autoritativa para o sistema completo de tokens de cor do Discord:

**RawColors** (443 tokens): Paleta base de cores -- `BRAND_500` (#5865f2), `PRIMARY_100` a `PRIMARY_900` (rampa de cinzas), `WHITE` a `BLACK`, rampas de cor `BLUE`/`GREEN`/`RED`/`ORANGE`/`TEAL` (100-900), `PLUM_0` a `PLUM_26` (cinzas refinados do Discord moderno), cores de cargo, cores de gradiente (`BG_GRADIENT_AURORA`, `BLURPLE_TWILIGHT`), cores de integracoes (`SPOTIFY`, `TWITCH`, `STEAM`), e cores de niveis de premium.

**SemanticColors** (302 tokens com valores dark+light): Mapeamento completo de tokens de UI.

| Categoria | Exemplos de Tokens |
|---|---|
| **Backgrounds** | `BACKGROUND_PRIMARY` (dark `#313338` / light `#ffffff`), `BACKGROUND_SECONDARY` (`#2b2d31`/`#f2f3f5`), `BACKGROUND_TERTIARY` (`#1e1f22`/`#e3e5e8`), `BACKGROUND_ACCENT`, `BACKGROUND_FLOATING`, `BACKGROUND_MODIFIER_HOVER/ACTIVE/SELECTED` |
| **Surfaces** | `BG_BASE_PRIMARY/SECONDARY/TERTIARY`, `BG_SURFACE_OVERLAY/RAISED`, `BG_MOD_FAINT/SUBTLE/STRONG` |
| **Text** | `TEXT_NORMAL` (`#dbdee1`/`#313338`), `TEXT_MUTED` (`#949ba4`/`#5c5e66`), `TEXT_LINK` (`#00aafc`/`#006be7`), `TEXT_BRAND`, `TEXT_DANGER`, `TEXT_POSITIVE`, `TEXT_WARNING`, `TEXT_LOW_CONTRAST` |
| **Interactive** | `INTERACTIVE_NORMAL` (`#b5bac1`/`#4e5058`), `INTERACTIVE_HOVER` (`#dbdee1`/`#313338`), `INTERACTIVE_ACTIVE` (`#ffffff`/`#060607`), `INTERACTIVE_MUTED` |
| **Channels** | `CHANNELTEXTAREA_BACKGROUND` (`#383a40`/`#ebedef`), `CHANNEL_ICON`, `CHANNEL_TEXT_AREA_PLACEHOLDER` |
| **Scrollbar** | `SCROLLBAR_THIN_THUMB/TRACK`, `SCROLLBAR_AUTO_THUMB/TRACK` |
| **Buttons** | `BUTTON_SECONDARY_BACKGROUND`, `BUTTON_POSITIVE_BACKGROUND`, `BUTTON_DANGER_BACKGROUND`, `REDESIGN_BUTTON_PRIMARY/SECONDARY/DANGER` |
| **Redesign** | `REDESIGN_ONLY_BACKGROUND_ACTIVE/DEFAULT/OVERLAY/RAISED/SUNKEN`, `REDESIGN_CHAT_INPUT_BACKGROUND` |
| **Status** | `STATUS_ONLINE/DND/IDLE/OFFLINE/SPEAKING`, `STATUS_POSITIVE/WARNING/DANGER` |
| **Borders** | `BORDER_FAINT/SUBTLE/STRONG`, `DIVIDER_FAINT/SUBTLE/STRONG` |
| **Profile** | `PROFILE_GRADIENT_CARD_BACKGROUND/OVERLAY/NOTE_BACKGROUND/ROLE_PILL_BACKGROUND/BORDER` |
| **Other** | `EMBED_BACKGROUND/TITLE`, `SPOILER_HIDDEN/REVEALED_BACKGROUND`, `MENTION_BACKGROUND/FOREGROUND`, `MODAL_BACKGROUND/FOOTER`, `INPUT_BACKGROUND` |

Fonte: [VendettaThemeUtil (GitHub)](https://github.com/nexpid/VendettaThemeUtil), [serenity-rs commit](https://github.com/serenity-rs/serenity/commit/c0463c0d7f04250aab178e823c19f4866ffa9e09)

### 1.5 Paleta de Cores Oficial

As diretrizes de marca do Discord (discord.com/branding) definem estas cores como tokens semanticos:

| Token | Nome | Hex | RGB |
|---|---|---|---|
| **Blurple** (primaria) | --brand-experiment | `#5865F2` | rgb(88, 101, 242) |
| **Dark Blurple** | -- | `#4E5D94` | rgb(78, 93, 148) |
| **Green** (sucesso/online) | --status-positive | `#57F287` | rgb(87, 242, 135) |
| **Yellow** (aviso) | --status-warning | `#FEE75C` | rgb(254, 231, 92) |
| **Fuchsia** (acento) | -- | `#EB459E` | rgb(235, 69, 158) |
| **Red** (perigo/offline) | --status-danger | `#ED4245` | rgb(237, 66, 69) |
| **Greyple** | -- | `#99AAB5` | rgb(153, 170, 181) |
| Dark, but not black | -- | `#2C2F33` | rgb(44, 47, 51) |
| Not quite black | -- | `#23272A` | rgb(35, 39, 42) |

Fonte: [Colorxs - Discord Blurple Brand Color](https://www.colorxs.com/color/discord-blurple), [brandcolor.dev](https://brandcolor.dev/brands/discord)

### 1.6 Evolucao do Blurple

O Blurple original era `#7289DA` e foi escurecido para `#5865F2` em 2021, uma mudanca de aproximadamente 23% na luminancia (de 54% para 42% em L\* no CIELAB). A nova versao e mais saturada (croma ~89 vs ~73) e mais escura, resultando em melhor contraste com fundos claros mas **pior contraste com fundos escuros** (discutido na Secao 5).

---

## 2. Engenharia do Modo Escuro

### 2.1 Arquitetura de Temas via CSS Custom Properties

O Discord utiliza um sistema extenso de **CSS custom properties** (variaveis CSS) definidas no seletor `:root`. A principal fonte de documentacao sobre essas variaveis vem da comunidade BetterDiscord, que as mapeia para permitir criacao de temas personalizados.

As variaveis **nao seguem uma convencao de nomenclatura consistente** e nao possuem prefixo unificado ([BetterDiscord Docs](https://docs.betterdiscord.app/themes/introduction/environment.html)).

### 2.2 Deteccao de Tema

O Discord adiciona duas classes ao elemento `<html>` para indicar o tema ativo:

```css
html.theme-dark { /* Tema escuro */ }
html.theme-light { /* Tema claro */ }
```

### 2.3 Categorias de Variaveis CSS

Com base na documentacao da comunidade e observacao do DOM, as variaveis se organizam nas seguintes categorias:

**Cores de Fundo (Surface Hierarchy):**
```
--background-primary         #313338 (tema escuro) / #FFFFFF (tema claro)
--background-secondary       #2b2d31
--background-tertiary        #1e1f22
--background-accent          #4e5058
--background-floating        #111214
--background-mentioned       #e8b81e (com opacidade)
--background-message-hover   rgba(4,4,5,0.07)
--background-modifier-hover  rgba(79,84,92,0.16)
--background-modifier-active rgba(79,84,92,0.24)
--background-modifier-selected rgba(79,84,92,0.32)
```

O gradiente de fundo do tema escuro e: `#1e1f22` (mais escuro, terciario) → `#2b2d31` (secundario) → `#313338` (primario, mais claro). Este gradiente cria uma **hierarquia de superficies** onde elementos mais elevados (modais, popouts) sao mais escuros, enquanto superficies base (chat) sao mais claras.

**Cores de Texto:**
```
--text-normal                #dbdee1 (tema escuro)
--text-muted                 #949ba4
--text-link                  #00a8fc
--text-positive              #57F287
--text-warning               #FEE75C
--text-danger                #ED4245
--header-primary             #f2f3f5
--header-secondary           #b5bac1
--channel-textarea-placeholder #949ba4
```

**Cores Interativas:**
```
--interactive-normal         #b5bac1
--interactive-hover          #dbdee1
--interactive-active         #f2f3f5
--interactive-muted          #4e5058
```

**Cores de Marca/Componentes:**
```
--brand-experiment           #5865F2
--brand-experiment-hover     #4752c4
--brand-experiment-560       #5865F2
```

**Cores de Status:**
```
--status-positive            #57F287
--status-warning             #FEE75C
--status-danger              #ED4245
--status-online              #57F287
--status-idle                #FEE75C
--status-dnd                 #ED4245
```

**Inputs e Scrollbar:**
```
--input-background           #1e1f22
--input-placeholder-text     #949ba4
--input-focused-border       #5865F2
--scrollbar-thin-thumb       #1a1b1e
--scrollbar-thin-track       transparent
--scrollbar-auto-thumb       #1a1b1e
--scrollbar-auto-track       #2e3035
```

**Variaveis de Acessibilidade (filter):**
```
--saturation-factor          (default: 1, intervalo: 0-1)
--contrast                   (default: 1)
--brightness                 (default: 1)
```

Fontes: [BetterDiscord Docs](https://docs.betterdiscord.app/themes/introduction/environment.html), [xcruxiex/themes DeepWiki](https://deepwiki.com/xcruxiex/themes/1-overview), [DevEvil Blog](https://blog.devevil.com/how-to-customize-discord-themes), [Translucence Theme GitHub](https://github.com/CapnKitten/Translucence)

### 2.4 Sistema de Filtros de Acessibilidade

O Discord aplica as variaveis de acessibilidade via CSS `filter` property:

```css
filter: saturate(var(--saturation-factor,1))
        contrast(var(--contrast,1))
        brightness(var(--brightness,1));
```

Isso significa que temas personalizados que nao incluirem essas variaveis em seus elementos **sobrescrevem as configuracoes de acessibilidade do usuario**. A BetterDiscord recomenda que temas personalizados SEMPRE incluam `filter: saturate(var(--saturation-factor,1))` para respeitar as preferencias do usuario.

### 2.5 Implementacao do Saturation Slider

Conforme apresentado por Brandon Dail (engenheiro do Discord) no GitNation, o controle de saturacao funciona assim:

1. As cores armazenadas como hex sao convertidas para HSL
2. A saturacao e multiplicada por uma variavel CSS representando a posicao do slider (0 a 1)
3. Em 0%, a saturacao chega a zero (escala de cinza); em 100%, a cor original e preservada
4. Para **cores de cargo definidas pelo usuario**, ajustes de brilho sao aplicados diferencialmente:
   - **Modo escuro**: `brightness` se aproxima de 1.5 (aumento de 50%) quando a saturacao tende a zero
   - **Modo claro**: `brightness` se aproxima de 0.5 (reducao de 50%)
   - Isso normaliza as cores para "cinza mais escuro" (modo claro) ou "branco" (modo escuro)

### 2.6 Variaveis de Elevacao (Surface Elevation)

O Discord nao documenta publicamente seu sistema de elevacao como o Material Design (que usa `elevation` com valores 1-24). Em vez disso, a elevacao e implementada via cores de fundo progressivamente mais escuras. A hierarquia observada:

| Nivel | Variavel | Valor (Dark) | Uso |
|---|---|---|---|
| 0 (base) | --background-primary | `#313338` | Fundo do chat |
| 1 | --background-secondary | `#2b2d31` | Sidebar, inputs |
| 2 | --background-tertiary | `#1e1f22` | Arvore de canais |
| 3 | --background-floating | `#111214` | Modais, popouts, tooltips |

Esta abordagem e consistente com as recomendacoes do Material Design 3 para modo escuro: _"usar cinza escuro ao inves de preto puro para permitir efeitos de elevacao"_. (Fonte: Politecnico di Milano, [Tese de G. M. Diaz Alonso, 2021](https://www.politesi.polimi.it/bitstream/10589/179726/1/2021_10_Diaz.pdf))

### 2.7 Temas Disponiveis (Desde Marco 2025)

Desde marco de 2025, o Discord oferece 4 temas gratuitos:
- **Light** -- fundo claro tradicional
- **Ash** -- cinza claro (novo)
- **Dark** -- fundo escuro padrao
- **Onyx** -- preto profundo para telas AMOLED

### 2.8 Desafios de Acessibilidade do Modo Escuro

O modo escuro do Discord enfrenta varios desafios de engenharia:

1. **Contraste de cores de cargo**: Como as cores de cargo sao definidas pelo administrador do servidor, elas precisam funcionar tanto no tema claro quanto no escuro -- algo que as cores padrao do Discord **nao conseguem fazer** com conformidade WCAG AA (4.5:1).

2. **Hallacao**: Texto branco (`#dbdee1`) sobre fundo escuro (`#313338`) tem contraste de aproximadamente 8.5:1, que e aceitavel, mas texto claro sobre fundo muito escuro ou preto (`#111214`) se aproxima de 13:1 -- o que o APCA (Accessible Perceptual Contrast Algorithm) considera potencialmente problematica para conforto visual, pois pode causar o efeito de hallacao (borrao/glow em torno do texto).

3. **Carga Cognitiva**: Pesquisas (Palmén et al., CHI 2023) mostram que o modo claro e lido de forma "confiavel mais rapido" que o modo escuro, com 459 participantes, apesar de os usuarios paradoxalmente preferirem o modo escuro.

4. **Medicao de Contraste**: O WCAG 2.x "superestima o contraste para cores escuras a ponto de 4.5:1 ser funcionalmente ilegivel quando uma das cores esta proxima do preto" (fonte: [APCA Documentation](https://git.apcacontrast.com/documentation/APCA_in_a_Nutshell.html)).

---

## 3. Acessibilidade: Auditorias e Conformidade

### 3.1 Ausencia de VPAT Publico

O Discord **nao publica um VPAT (Voluntary Product Accessibility Template)**, que e tipicamente exigido para contratos governamentais e empresariais nos EUA (Secao 508). Nenhum relatorio de conformidade WCAG foi encontrado em nenhuma busca.

### 3.2 Review da American Foundation for the Blind (AFB)

A revisao mais completa de acessibilidade do Discord foi publicada pela AFB em 2022 ([fonte](https://afb.org/aw/23/12/18148)):

**Desktop -- Pontos Fortes:**
- "Poucos ou nenhum elemento nao-rotulado" -- links, botoes, abas e sliders estao propriamente identificados
- hCAPTCHA agora oferece opcao de pergunta textual "sim/nao"
- Elementos de navegacao variados facilitam a orientacao por leitores de tela
- Modo Browse e Focus funcionam adequadamente
- "Modelo para aplicacoes web complexas" segundo a AFB

**Desktop -- Violacoes Encontradas:**
1. **Nenhum alerta de novas mensagens** no canal em foco -- usuarios sao alertados "que alguem esta digitando mas nao quando novas mensagens sao postadas" (provável violacao de **WCAG 4.1.3 Status Messages**)
2. **Bug de foco no botao Fechar** -- apos usar o botao "Close", o usuario "ainda consegue interagir com a janela como se ela ainda estivesse presente" (**WCAG 2.4.3 Focus Order**)
3. **Imagem de login nao-rotulada** -- o logo do Discord na tela de login nao possui texto alternativo (**WCAG 1.1.1 Non-text Content**)

**Mobile (iOS/Android) -- Violacoes Mais Graves:**
4. **Ordem de foco imprevisivel** ao rolar historico de mensagens -- o foco "saltava para mensagens antigas, pulando o que deveria estar em seguida" (**WCAG 2.4.3 Focus Order**)
5. **Elementos nao-rotulados no mobile** que sao rotulados no desktop -- botoes de sticker sem identificacao (**WCAG 4.1.2 Name, Role, Value**)
6. **Audio do canal de voz padrao no alto-falante do aparelho** em vez do fone de ouvido, tornando "muito dificil ouvir o leitor de tela" (**barreira de acessibilidade grave**)
7. **Ausencia de botao "Voltar"** em certas telas (ex: tela de "Spectators") (**WCAG 2.4.3, 2.1.1**)

### 3.3 Palestra de Brandon Dail (GitNation)

A palestra "Accessibility at Discord" ([fonte](https://gitnation.com/contents/accessibility-at-discord)) revelou detalhes tecnicos importantes:

**Sistema Unificado de Focus Ring:**
O Discord construiu um sistema de focus ring customizado porque o CSS `outline` tinha limitacoes:
- `overflow: hidden` em container pai cortava o focus ring
- `outline` so pode ser aplicado ao elemento alvo, nao ao container
- `outline-offset` aceita apenas valor uniforme (nao assimetrico)
- Nao havia adaptacao automatica a cor de fundo

A API tem dois componentes principais:
- `FocusRing` -- API principal que envolve um elemento interativo
- `FocusRingScope` -- fornece uma ref de container como ponto de ancoragem

Props: `offset` (aceita numero ou objeto `{ top, left }`), `within` (como `:focus-within`), `ringTarget`, `focusTarget`.

**React D&D Accessible Backend:**
- Codigo aberto (open source)
- `Command + D` (Mac) ativa modo de arrastar, setas movem o item, `Enter` confirma
- Funciona em servidores, canais e qualquer superficie de drag-and-drop

**Runtime Accessibility Checking:**
- Sistema experimental interno
- Usa `navigator.scheduling.isInputPending` (Chromium) para verificar se o usuario esta interagindo
- MutationObserver na raiz do documento
- Debounce de ~250ms + `requestIdleCallback` para executar checagens sem impactar performance
- Desligado se a API nao estiver disponivel

### 3.4 Cores de Cargo e Falhas de Contraste

A pesquisa mais relevante sobre contraste no Discord vem do **Society for Blaseball Research (SIBR)** com o paper "Improving Accessibility: Contrast in Discord Team Roles" ([fonte](https://sibr.dev/papers/files/Improving_Accessibility__Contrast_in_Discord_Team_Roles_1.2.pdf)) e do guia da **CSUSM** ([fonte](https://www.csusm.edu/iits/services/accessibility/guides/socialmedia/discord.html)).

**Constatacao Principal:** As cores de cargo padrao do Discord **nao atendem WCAG 2.1 AA (4.5:1)** em nenhum dos temas simultaneamente.

**Recomendacao:** WCAG 2.1 Nivel A (3.1:1) como minimo pratico, que corresponde ao criterio para texto grande.

**Cores Sugeridas que Funcionam em Ambos os Temas:**
| Cor | Hex | Modo Claro | Modo Escuro | AMOLED |
|---|---|---|---|---|
| Verde ciano escuro | `#00a455` | 3.26:1 | 3.55:1 | 6.44:1 |
| Vermelho ligeiramente dessaturado | `#cd7672` | 3.27:1 | 3.54:1 | 6.43:1 |
| Azul-roxo suave | `#8877ee` | 3.55:1 | 3.26:1 | 5.92:1 |
| Vermelho vivido | `#ff3714` | 3.62:1 | 3.20:1 | 5.81:1 |
| Orquidea | `#cc66dd` | 3.22:1 | 3.59:1 | 6.51:1 |
| Verde escuro | `#50a210` | 3.22:1 | 3.59:1 | 6.52:1 |
| Azul brilhante | `#3f88fd` | 3.41:1 | 3.39:1 | 6.15:1 |
| Magenta brilhante | `#f032c9` | 3.50:1 | 3.31:1 | 6.00:1 |

### 3.5 Funcionalidades de Acessibilidade Existentes

**Configuracoes de Cargo (Settings > Accessibility > Role Colors):**
- Mostrar cor do cargo no nome
- Mostrar cor do cargo como ponto ao lado do nome (recurso adicionado em julho de 2022)
- Nao mostrar cores de cargo

**Outras Funcionalidades:**
- Slider de saturacao (0-100%)
- Controle de animacao de stickers ("Sempre", "Ao interagir", "Nunca")
- Suporte a Windows High Contrast Mode
- Texto alternativo para imagens no mobile (adicionado em julho de 2022)
- Reducao de movimento (reduced motion)
- Escalonamento de fonte no chat

### 3.6 Iniciativa Interna de Conformidade WCAG AA

O relato mais importante sobre acessibilidade no Discord vem de **San Chung**, Senior Product Designer que trabalhou com a equipe de Acessibilidade do Discord de 2021 a 2022 ([fonte](https://sanyeechung.com/work/a11y/)). Seu portifolio revela:

**Escopo do Trabalho:**
- Auditoria completa de "variaveis de cores desonestas e contraste geralmente pobre" em fundos, texto e temas claro/escuro
- Colaboracao com o **American Council for the Blind** para entender necessidades reais de usuarios com deficiencia visual
- Criacao da pagina de **configuracoes de acessibilidade** com controles de saturacao de cor e configuracao de "Reduced Motion"
- Desafios de interpretar as diretrizes WCAG para a complexidade de um aplicativo como o Discord (multi-temas, customizacao por usuario, cores de cargo definidas por admins)

**Principais Dificuldades Relatadas:**
- Identificar "variaveis de cores desonestas" -- variaveis CSS que nao seguiam o sistema de tokens
- "Contraste geralmente pobre" entre elementos de UI
- Equilibrar as necessidades conflitantes de diferentes grupos de usuarios

### 3.7 Funcionalidades Ausentes (Roadmap)

Com base na revisao da AFB e nas discussoes da comunidade, estas funcionalidades estao **ausentes**:
- Legendas ao vivo para chats de voz
- Transcricao automatica de voz para texto
- Anuncio de novas mensagens para leitores de tela (WCAG 4.1.3)
- Modo de alto contraste dedicado (apenas Windows High Contrast Mode)
- Modo daltônico dedicado (apenas slider de saturacao)
- VPAT publico
- Roadmap de acessibilidade publicado

### 3.7 Processo Judicial

Um processo foi aberto no tribunal federal do Distrito Leste de Nova York (Caso 1:24-cv-04380, 2024) relacionado a acessibilidade do Discord ([CourtListener](https://storage.courtlistener.com/recap/gov.uscourts.nyed.438000/gov.uscourts.nyed.438000.1.0_1.pdf)). Os detalhes especificos da acao não estao publicamente disponiveis neste momento.

### 3.8 Posicionamento da Empresa

O Discord afirma em sua pagina de suporte estar comprometido com acessibilidade, mas criticos apontam que a empresa frequentemente direciona solicitacoes de acessibilidade para "enviar uma sugestao de recurso" em vez de priorizar correcoes. ([fonte: Misophonia International](https://misophoniainternational.com/the-neglect-of-accessibility-in-tech-when-developers-are-dismissive-and-rude/))

---

## 4. Ciencia Tipografica: gg sans

### 4.1 Historia e Origem

O **gg sans** e uma tipografia proprietaria personalizada do Discord, introduzida em **1 de dezembro de 2022**, substituindo a fonte Whitney (de Tobias Frere-Jones, usada desde o lancamento do Discord em 2015).

Diferente do que muitos acreditam, o gg sans **nao foi projetado pela Grilli Type** (um erro frequente em artigos de terceiros). A metadados da fonte aponta consistentemente para a **Colophon Foundry** (Londres/Nova York) como fabricante e designer.

O nome "gg sans" e um trocadilho com "good game" (expressão do universo gamer) e o dominio `discord.gg` da plataforma.

### 4.2 Pesos Disponiveis

O gg sans possui 10 variantes de peso:

| Peso | Variantes |
|---|---|
| Normal (400) | Normal, Italico |
| Medium (500) | Medium, Medium Italico |
| Semi Bold (600) | Semi Bold, Semi Bold Italico |
| Bold (700) | Bold, Bold Italico |
| Extra Bold (800) | Extra Bold, Extra Bold Italico |

### 4.3 Pilha de Fontes (Font Stack)

A pilha de fallback do Discord e:
```
gg sans, Noto Sans, Helvetica Neue, Helvetica, Arial, sans-serif, Apple Symbols
```

Fonte: [discord.css no GitHub](https://github.com/edwin-shdw/discord.css/blob/main/site/docs/getting-started/font.mdx)

### 4.4 Sistema Tipografico de Tres Niveis

O sistema tipografico do Discord e hierarquico:

| Nivel | Fonte | Uso |
|---|---|---|
| Produto/UI | **gg sans** (10 variantes) | Chat, menus, navegacao, todo texto do app |
| Marca/Headlines | **Ginto Discord Nord** | Manchetes de marketing, hero text |
| Marca/Corpo | **Ginto Discord** | Subtitulos, legendas, corpo de texto em branding |

O Ginto Nord foi projetado por **Seb McLauchlan** e publicado pela fundicao **Dinamo**, introduzido em maio de 2021 como parte de um rebranding feito com Studio Koto e AKQA.

### 4.5 Filosofia de Design

O gg sans foi projetado com foco em:
- **Legibilidade durante sessoes prolongadas** (jogos, chats comunitarios ativos)
- Proporcoes amigaveis a tela (geometria suave, contadores arredondados)
- Curvas suaves versus headline bold
- Maior cobertura internacional (Whitney tinha rendering "pobre para scripts latinos extendidos" como vietnamita e esperanto)

### 4.6 Reacao da Comunidade

A transicao foi **altamente controversa**:

- **Reclamacoes de cansaco visual** devido a caracteres "ligeiramente mais estreitos/altos"
- Problemas de **legibilidade em tamanhos pequenos**
- Ausencia de opcao nativa de reversao
- Criacao de multiplos repositorios para reverter a fonte:
  - [RevertDiscordFont (BowDown097)](https://github.com/BowDown097/RevertDiscordFont)
  - [old-discord-font (Overimagine1)](https://github.com/Overimagine1/old-discord-font)
  - [nomoreggsans (lokka30)](https://awesome.ecosyste.ms/projects/github.com%2Flokka30%2Fnomoreggsans)
- Usuarios com dislexia relataram dificuldades especificas com o gg sans

### 4.7 Ciencia da Legibilidade em Modo Escuro

**Estudo Palmén et al. (Google, CHI 2023):**
O principal estudo empirico sobre legibilidade claro vs. escuro ([ACM](https://dl.acm.org/doi/fullHtml/10.1145/3544548.3581552)) com 459 participantes constatou:

1. **Modo claro e lido de forma confiavel mais rapido que o modo escuro** (diferenca estatisticamente significativa)
2. O **grade** da fonte (negrito sem alteracao de largura) melhora significativamente a legibilidade no modo claro
3. O grade **nao tem efeito significativo** na legibilidade em modo escuro para tamanhos de texto corpo
4. **Paradoxalmente**, os participantes nao preferiram o modo claro ao escuro, apesar de lerem mais devagar no escuro
5. O estudo "nem corrobora nem refuta" o uso de grade/peso para gerenciar hallacao em tamanhos de texto corpo

**Carga Cognitiva em Modo Escuro:**
- O modo escuro causa **dilatacao da pupila**, levando a mais aberracoes esfericas e "capacidade reduzida de focar detalhes finos" ([GitHub Desktop Issue #9389](https://github.com/desktop/desktop/issues/9389))
- Fontes mais grossas/negrito sao recomendadas para modo escuro, mas a evidencia empirica (Palmén) contesta que o grade da fonte tenha efeito significativo
- Estudo sobre caracteres chineses em smartphones (Chen et al., Springer/HCII 2023) ([link](https://dlnext.acm.org/doi/10.1007/978-3-031-34866-2_2)): fundo cinza escuro (HSB: 0, 50, 12) com texto cinza claro (HSB: 0, 0, 58) e fonte sans-serif (Hei) foi a combinacao otima para modo escuro

### 4.8 Diretrizes de Tipografia para Modo Escuro

Com base nas pesquisas compiladas ([Design Shack](https://designshack.net/articles/typography/dark-mode-typography/), [Google Fonts Knowledge](https://fonts.google.com/knowledge/choosing_type/exploring_typefaces_with_multiple_weights_or_grades)):

1. **Evitar branco puro (`#FFFFFF`) sobre preto puro (`#000000`)** -- usar `#E0E0E0` ou `#CCCCCC` sobre cinza escuro (`#121212`)
2. **Aumentar o peso da fonte** (regular/medium/semi-bold) para modo escuro
3. **Aumentar letter-spacing e line-height** em modo escuro compensa a perda de nitidez
4. **Fontes sans-serif** sao preferiveis em modo escuro (serifas se perdem na hallacao)
5. O eixo de **grade** em fontes variaveis pode ser usado para compensar hallacao sem alterar largura do glifo

---

## 5. Ciencia da Cor: Blurple e Contraste

### 5.1 Propriedades Colorimetricas do Blurple (#5865F2)

| Propriedade | Valor |
|---|---|
| Hex | `#5865F2` |
| RGB | `rgb(88, 101, 242)` |
| HSL | `hsl(235, 86%, 65%)` |
| CMYK | `cmyk(64, 58, 0, 5)` |
| Luminancia Relativa (sRGB) | ~0.175 |
| L\* (CIELAB) | ~47.3 |
| Croma (CIELAB) | ~89 |

### 5.2 Taxas de Contraste por Fundo

| Fundo | Texto | Taxa | WCAG AA (4.5:1) | WCAG AAA (7:1) |
|---|---|---|---|---|
| Branco (`#FFF`) | Blurple | **4.5:1** ✅ | Passa AA | Falha AAA |
| Preto (`#000`) | Blurple | **3.8:1** ❌ | **Falha AA** | Falha AAA |
| Blurple | Branco | **4.1:1** ❌ | **Falha AA** | Falha AAA |
| Blurple | Preto | **4.5:1** ✅ | Passa AA | Falha AAA |
| Branco | Blurple (texto grande) | 4.5:1 ✅ | Passa (3:1 min) | Falha |
| #2b2d31 (Discord dark) | Blurple | **~2.1:1** ❌ | **Falha grave** | Falha |

Fonte: calculos baseados em [colorify.rocks](https://colorify.rocks/color/5865f2), [hexcolor.co](https://hexcolor.co/hex/5865f2)

**Conclusao critica**: O Blurple `#5865F2` e **insuficiente para texto de tamanho normal** (AA 4.5:1) tanto sobre fundo preto quanto sobre o fundo escuro padrao do Discord (`#2b2d31`). Ele funciona marginalmente sobre fundo branco (exatos 4.5:1). Para texto grande (18px+ ou 14px+ bold), atende o minimo de 3:1.

### 5.3 Simulacao de Daltonismo

Valores simulados usando o algoritmo Brettel-Vienot-Mollon (1997):

| Condicao | Cor Simulada | Efeito |
|---|---|---|
| Visao Normal | `#5865F2` | Azul-arroxeado vivido |
| **Protanopia** (cego vermelho) | `#5E5ED0` | Desloca para azul puro |
| Protanomalia (fraca vermelho) | `#5A61E0` | Levemente dessaturado |
| **Deuteranopia** (cego verde) | `#5D5CC8` | Mais escuro, azul mais opaco |
| Deuteranomalia (fraca verde) | `#5B62DE` | Diferenca minima |
| **Tritanopia** (cego azul) | `#59B5AF` | **Muda drasticamente para azul-esverdeado** |
| Tritanomalia (fraca azul) | `#588BD8` | Azul dessaturado |
| **Acromatopsia** (monocromatico) | `#717171` | Cinza medio uniforme |

**Riscos Identificados:**
1. **Tritanopia**: O Blurple se transforma em `#59B5AF` (verde-azulado), perdendo completamente a identidade visual
2. **Protanopia/Deuteranopia**: O Blurple se torna um azul mais opaco, mas ainda distinguivel
3. **Acromatopsia**: Perde toda a cor, virando cinza -- aceitavel para elementos grandes, problematico para informacao codificada por cor

### 5.4 Linhas de Confusao para Azul-Purpureo

As **linhas de confusao** sao as direcoes no espaco de cor CIE xyY ao longo das quais as cores se tornam indistinguiveis para cada tipo de deficiencia:

- **Protanopia**: 104 linhas (P1-P104). No espectro azul-purpureo, as linhas se tornam "mais horizontais", o que diferencia protanopia de deuteranopia em regioes azuis
- **Deuteranopia**: 95 linhas (D1-D95). No azul-purpureo, orientacao diferente da protanopia
- **Tritanopia**: Linhas correm ao longo do eixo **azul-amarelo**. Purpuras e vermelhos, azuis e verdes, amarelos e rosas se tornam indistinguiveis

O purpura e uma mistura de azul e vermelho. Para protanopia (sensibilidade reduzida ao vermelho), o componente vermelho do purpura e diminuido, fazendo o purpura parecer mais azul. Para tritanopia (sensibilidade reduzida ao azul), o purpura perde seu componente azul e parece mais vermelho.

### 5.5 APCA vs. WCAG 2.x para o Blurple

O **Accessible Perceptual Contrast Algorithm (APCA)** e o metodo de contraste candidato para WCAG 3, projetado especificamente para corrigir as falhas do WCAG 2.x em modo escuro.

**Diferencas Fundamentais:**
- WCAG 2.x produz um unico ratio (1:1 a 21:1) independente de polaridade
- APCA produz **valores com sinal**: Lc positivo para texto escuro em fundo claro, Lc negativo para texto claro em fundo escuro
- APCA usa um **soft black clamp** para compensar flare em cores escuras
- APCA e **perceptualmente uniforme**: Lc 60 representa a mesma percecao de contraste independentemente da luminosidade

**Thresholds APCA (Bronze):**
- Lc 90: Preferido para texto corpo (18px/300 ou 14px/400)
- Lc 75: Minimo para texto corpo
- Lc 60: Minimo para texto conteudo nao-corpo
- Lc 45: Headlines, pictogramas com detalhes finos
- Lc 30: Placeholder, elementos desabilitados, icones solidos grandes
- Lc 15: Minimo absoluto para elementos nao-texto

**APCA no Blurple:** Com Lc ~72 para branco em Blurple (fundo), o Blurple atenderia APCA Bronze para texto corpo minimo (Lc 75 esta no limiar), mas estaria abaixo do nivel preferido (Lc 90). Para Blurple sobre fundo escuro, o valor Lc seria ainda menor, provavelmente abaixo de Lc 45.

Fonte: [APCA Documentation](https://git.apcacontrast.com/documentation/APCA_in_a_Nutshell.html), [APCA Introduction (GitHub)](https://raw.githubusercontent.com/Myndex/apca-introduction/59a21acd68ce0e55dca1b4b702d4add4ac2e4099/README.md)

### 5.6 Percepcao do Roxo: Instabilidade Perceptual

Um estudo cientifico de 2025 (Schulz-Hildebrandt, arXiv) revela que o roxo e uma cor perceptual instavel ([fonte](https://arxiv.org/html/2509.11582v4)):

1. **Zona livre de S-cones na fovea**: Apenas 8-12% dos cones sao S-cones (sensiveis ao azul), e eles estao ausentes no centro absoluto da fovea
2. **Pigmento macular**: Absorve luz azul, reduzindo ainda mais o sinal azul no centro do campo visual
3. **Natureza construtiva do roxo**: O cerebro "reconstroi" o roxo a partir de sinais fracos dos S-cones -- e uma "cor nao espectral" que nao tem comprimento de onda unico

Isso significa que o Blurple, sendo um azul-arroxeado, ja e perceptual e neurologicamente **instavel ate mesmo para usuarios com visao normal** em certas condicoes (visao periferica, baixa luminosidade).

### 5.7 Confusao Azul-Roxo no Daltonismo

A combinacao "azul e roxo" e reconhecida como problematicas em guias de acessibilidade:

- **Protanopia**: "Os purpuras sao indistinguiveis dos azuis" devido a falta de cones L que detectam o componente vermelho do roxo ([fonte](https://unique.aq.upm.es/en/accesibilidad-cromatica/))
- **Deuteranopia**: Dificuldade em distinguir "azul de roxo" ([fonte](https://www.uxmatters.com/mt/archives/2024/09/how-to-design-for-color-independence-creating-inclusive-digital-experiences.php))
- **Tritanopia**: Confusao entre "roxo e vermelho" e "azul e verde" ([fonte](https://digitalaccessibility.virginia.edu/accessibility-color))
- **Proposta do Blurple como cor CSS**: A W3C chegou a receber uma proposta (Issue #5169) para adicionar "blurple" como cor nomeada oficial do CSS em 2020, com valores `#7289DA` e `#4E5D94` ([fonte](https://github.com/w3c/csswg-drafts/issues/5169)), mas foi fechada sem resolucao

Pesquisas em HCI (ResearchGate, 2024) mostram que pessoas com deficiencia de visao de cores utilizam **estrategicamente diferencas de luminosidade** para distinguir cores de confusao como azul e roxo, mas a estrategia falha quando as cores tem luminosidades similares.

### 5.8 Reacao ao Rebrand de 2021 e Saturacao

O escurecimento do Blurple em 2021 (`#7289DA` -> `#5865F2`) gerou reacoes negativas significativas:

- Usuarios reclamaram que a nova cor "potencialmente causa sofrimento para pessoas com problemas de processamento visual" e cansaco visual ([fonte](https://knowyourmeme.com/news/discord-rebrand-criticized-by-users-over-new-logo-and-possible-accessibility-issues))
- Um tweet sobre o novo azul ser "mais cansativo para os olhos" recebeu quase **5.000 curtidas** ([fonte](https://www.makeuseof.com/why-does-everyone-hate-discord-rebrand/))
- O Discord respondeu no mesmo dia prometendo "repassar o problema internamente"
- Artigos de imprensa confirmam que o "blurple foi escurecido" e a paleta foi "renovada, com alguns matizes mais ousados" ([fonte](https://www.androidpolice.com/2021/05/14/discord-celebrates-turning-6-with-a-new-logo-new-colors-and-a-whole-refreshed-look/))

### 5.9 Desafios de Cor em Modo Escuro

Pesquisas compiladas de multiplas fontes indicam:

1. **Cores saturadas em modo escuro produzem cansaco visual** -- recomendacao e usar cores de acento dessaturadas e mais claras ([Tese de G. M. Diaz Alonso, Politecnico di Milano, 2021](https://www.politesi.polimi.it/bitstream/10589/179726/1/2021_10_Diaz.pdf))
2. O contraste entre texto e o fundo mais escuro em modo escuro precisa ser de no minimo **15.8:1** para preservar legibilidade em superficies elevadas mais claras
3. Branco puro deve ser ligeiramente escurecido para emitir menos luz e reduzir ofuscamento
4. **Discord usa cinza `#dbdee1` em vez de branco puro `#FFFFFF`** para texto normal, consistente com esta recomendacao
5. **Astigmatismo**: 28-47% da populacao tem algum grau de astigmatismo, que causa hallacao severa com texto branco sobre fundo preto ([fonte](https://www.levelaccess.com/blog/accessibility-for-people-with-astigmatism/))
6. **WCAG 3.0/APCA**: O novo padrao tratara melhor a matematica de contraste light-on-dark, substituindo o WCAG 2.x que "superestima o contraste para cores escuras" ([fonte](https://git.apcacontrast.com/documentation/APCA_in_a_Nutshell.html))

---

## 6. Hallacao e Percepcao de Contraste em Modo Escuro

### 6.1 O que e Hallacao?

Hallacao (halation) e o efeito visual de "brilho" ou "halo" ao redor de texto claro sobre fundo escuro. Ocorre porque a luz de pixeis claros se espalha dentro do olho (dispersao intraocular e aberracoes opticas), criando um "borrao" luminoso ao redor dos caracteres. Em casos extremos (branco puro sobre preto puro), a hallacao pode causar:
- Fadiga retinal
- "Vibracao visual" (oscilacao percebida das bordas)
- Dificuldade de foco
- Lacrimejamento e dor de cabeca em usuarios sensiveis

### 6.2 Evidencia Cientifica

O estudo CHI 2023 de Palmén, Gilbert e Crossland (Google) e o mais citado, mas reconhece que os resultados "nem corroboram nem refutam" o uso de peso/grade para gerenciar hallacao em tamanhos de texto corpo.

O **APCA** aborda a hallacao de duas maneiras:
1. **Soft black clamp**: Um pequeno boost (ajuste) para cores muito escuras que compensa o flare (dispersao de luz) que ocorre naturalmente em displays e no olho humano
2. **Limite de contraste maximo**: APCA define um maximo de Lc -108 (aproximadamente) para texto claro em fundo escuro, prevenindo extremos que prejudicam a legibilidade

### 6.3 Estudos Adicionais sobre Contraste

**Andrew & Tigwell (RIT, ACM 2025):**
Estudo qualitativo com 29 participantes (13 com deficiencia visual, 16 sem) identificou que o modo escuro cria "problemas significativos de acessibilidade e usabilidade" quando mal implementado. Uma configuracao que ajuda um grupo (ex: alto contraste para baixa visao) pode prejudicar outro (ex: sensibilidade luminosa).

**Laine (Turku UAS, 2025):**
Revisao de literatura sobre modo escuro concluiu que:
- Modo escuro reduz cansaco visual em **ambientes com pouca luz**
- Modo escuro **diminui a legibilidade** e aumenta o "blur visual" em **ambientes claros** ou para **leitura prolongada**
- Fundo preto puro (`#000`) com texto branco puro (`#FFF`) cria **contraste excessivo** que causa hallacao e desconforto
- Fundo cinza escuro (`#121212`) com texto off-white (`#E0E0E0`) e a **recomendacao padrao** da industria

Fonte: [Tese de Jere Laine](https://www.theseus.fi/bitstream/handle/10024/896088/Laine_Jere.pdf), [Andrew & Tigwell ACM Paper](https://par.nsf.gov/biblio/10640770)

---

## 7. Conclusoes e Recomendacoes

### 7.1 Principais Constatacoes

**Design Tokens (Mana):**
- O Discord possui um sistema de design maduro (Mana) que cobre 4 plataformas, mas sua documentacao publica e praticamente inexistente
- O sistema de tokens cross-platform nao tem artigos de engenharia publicados -- todo o conhecimento vem de vagas de emprego e engenharia reversa da comunidade
- As variaveis CSS do cliente Discord sao extensas mas "nao seguem convencao de nomenclatura" e mudam frequentemente

**Modo Escuro:**
- O sistema de elevacao do Discord (hierarquia de superficies) e implementado via cores progressivamente mais escuras, uma abordagem consistente com Material Design 3
- Os filtros CSS de acessibilidade (`--saturation-factor`, `--contrast`, `--brightness`) sao uma abordagem engenhosa para customizacao pelo usuario, mas temas personalizados frequentemente os sobrescrevem
- A transicao de 4 temas gratuitos desde marco 2025 (Light, Ash, Dark, Onyx) expande as opcoes

**Acessibilidade:**
- Nao ha VPAT publico nem relatorio de conformidade WCAG
- Desktop tem suporte robusto para leitores de tela (AFB chama de "modelo")
- Mobile tem barreiras significativas (foco imprevisivel, elementos nao-rotulados)
- Cores de cargo falham WCAG AA (4.5:1) em todos os temas simultaneamente
- Nao ha modo daltônico dedicado (apenas slider de saturacao)
- Processo judicial ativo (EDNY 1:24-cv-04380)
- Sistema de runtime accessibility checking interno e inovador (MutationObserver + requestIdleCallback)

**Tipografia:**
- gg sans (Colophon Foundry) foi uma migracao controversa do Whitney em dezembro de 2022
- A ciencia mostra que modo escuro e lido mais devagar, mas usuarios preferem
- Grade da fonte nao melhora legibilidade em modo escuro para texto corpo (Palmén, CHI 2023)
- Hallacao continua sendo um desafio nao-resolvido para o qual faltam evidencias empiricas diretas de correcao via ajuste de peso

**Ciencia da Cor:**
- Blurple `#5865F2` falha WCAG AA em fundos escuros (incluindo o proprio tema escuro do Discord)
- Blurple falha para tritanopia (transforma-se em verde-azulado `#59B5AF`)
- WCAG 2.x e inadequado para guiar design de modo escuro -- APCA/WCAG 3 oferece modelo melhor
- A abordagem do Discord de usar `#dbdee1` (em vez de branco puro) e `#1e1f22` a `#313338` (em vez de preto puro) e consistente com as melhores praticas cientificas

### 7.2 Recomendações de Design

1. **Para contraste de marca**: Usar Blurple apenas para elementos grandes (botoes, banners) com texto grande, nunca para texto de corpo em fundos escuros. Alternativa: `#4752C4` (mais escuro) atenderia 4.5:1 em branco.

2. **Para modo escuro**: Seguir a abordagem do Unity (cinza escuro em vez de preto, off-white em vez de branco puro) -- que o Discord ja implementa parcialmente.

3. **Para daltonismo**: Implementar modo daltônico dedicado (nao apenas slider de saturacao). O Blurple precisa de tratamento especial para tritanopia.

4. **Para tipografia**: Testar o gg sans em todas as combinacoes tema/idioma. Considerar que usuarios com dislexia podem precisar de alternativa.

5. **Para acessibilidade**: Publicar VPAT, implementar anuncio de novas mensagens (WCAG 4.1.3), corrigir ordem de foco no mobile, e adicionar legendas em tempo real para chats de voz.

6. **Para medicao de contraste**: Adotar APCA (WCAG 3) como complemento ao WCAG 2.x, especialmente para avaliar combinacoes de cor em modo escuro.

---

## 8. Bibliografia

### Fontes Oficiais Discord
- Discord Branding: https://discord.com/branding
- Discord Accessibility Support: https://support.discord.com/hc/en-us/articles/360057987894-Accessibility
- [Vaga Senior Software Engineer, Design Systems](https://jobs.accel.com/companies/discord-2/jobs/48874253-senior-software-engineer-design-systems)
- [Vaga Staff Software Engineer, Design Systems](https://jobs.rre.com/companies/discord-2/jobs/65387521-staff-software-engineer-design-systems)

### Artigos Tecnicos e Palestras
- Brandon Dail, "Accessibility at Discord" (GitNation): https://gitnation.com/contents/accessibility-at-discord
- Discord Engineering Blog, "Light Theme, Redeemed": https://discord.com/blog/light-theme-redeemed
- Discord Engineering Blog, "Building Open-Source Design Tools": https://discord.com/blog/building-open-source-design-tools-to-improve-discords-design-workflow
- Discord Engineering Blog, "Redesigning the Discord Overlay": https://discord.com/blog/redesigning-the-discord-overlay
- Discord Engineering Blog, "Why Discord is Sticking with React Native": https://discord.com/blog/why-discord-is-sticking-with-react-native
- Discord Engineering Blog, "Native iOS Performance with React Native": https://discord.com/blog/how-discord-achieves-native-ios-performance-with-react-native
- Discord Engineering, "postcss-theme-shorthand" (plugin PostCSS open-source): https://github.com/discord/postcss-theme-shorthand
- Envato Tuts+, "What Fonts Does Discord Use?": https://design.tutsplus.com/articles/what-fonts-does-discord-use--cms-108779

### Acessibilidade e Auditorias
- American Foundation for the Blind, "Call and Chat with Discord: an Accessibility Review": https://afb.org/aw/23/12/18148
- CSUSM, "Creating an Accessible Discord Server": https://www.csusm.edu/iits/services/accessibility/guides/socialmedia/discord.html
- Society for Blaseball Research, "Improving Accessibility: Contrast in Discord Team Roles": https://sibr.dev/papers/files/Improving_Accessibility__Contrast_in_Discord_Team_Roles_1.2.pdf
- San Chung, "Designing Accessible Experiences at Discord": https://sanyeechung.com/work/a11y/
- Forbes (Steven Aquino), "Discord Pushes Forward On Building A Place 'Where No One Is An Outsider'": https://www.forbes.com/sites/stevenaquino/2022/07/07/discord-pushes-forward-on-building-a-place-where-no-one-is-an-outsider-with-new-accessibility-improvements-for-disabled-people/
- Cool Blind Tech, "Discord Launches New Accessibility Features": https://coolblindtech.com/discord-launches-new-accessibility-features
- Misophonia International, "The Neglect of Accessibility in Tech": https://misophoniainternational.com/the-neglect-of-accessibility-in-tech-when-developers-are-dismissive-and-rude/
- CourtListener, Caso 1:24-cv-04380 (EDNY): https://storage.courtlistener.com/recap/gov.uscourts.nyed.438000/gov.uscourts.nyed.438000.1.0_1.pdf
- PWABuilder GitHub Issue #4744 (contraste 2.7:1): https://github.com/pwa-builder/PWABuilder/issues/4744
- Django OCG Issue #25: https://github.com/django/online-community-working-group/issues/25
- Level Access, "Accessibility for People with Astigmatism": https://www.levelaccess.com/blog/accessibility-for-people-with-astigmatism/
- Smashing Magazine, "Inclusive Dark Mode: Designing Accessible Dark Themes" (2025): https://www.smashingmagazine.com/2025/04/inclusive-dark-mode-designing-accessible-dark-themes/
- BOIA, "Dark Mode Doesn't Satisfy WCAG Color Contrast Requirements": https://www.boia.org/blog/offering-a-dark-mode-doesnt-satisfy-wcag-color-contrast-requirements

### Temas e Variaveis CSS (Comunidade)
- BetterDiscord Docs - Theme Environment: https://docs.betterdiscord.app/themes/introduction/environment.html
- BetterDiscord Docs - Accessibility: https://docs.betterdiscord.app/themes/concepts/accessibility
- xcruxiex/themes DeepWiki: https://deepwiki.com/xcruxiex/themes/1-overview
- xcruxiex/themes - Core Theme System: https://deepwiki.com/xcruxiex/themes/3-core-theme-system
- DevEvil Blog - Theme Customization: https://blog.devevil.com/how-to-customize-discord-themes
- CapnKitten/Translucence (GitHub): https://github.com/CapnKitten/Translucence
- MiniDiscordThemes/Demonstration (GitHub): https://github.com/MiniDiscordThemes/Demonstration
- Snakycat/DiscordNight (GitHub): https://github.com/Snakycat/DiscordNight
- BowDown097/RevertDiscordFont: https://github.com/BowDown097/RevertDiscordFont
- Overimagine1/old-discord-font: https://github.com/Overimagine1/old-discord-font
- Auto Theme Figma Plugin (Destefanis/Discord): https://github.com/destefanis/auto-theme
- Design Lint Figma Plugin (Destefanis/Discord): https://github.com/destefanis/design-lint
- Figma Table of Contents (Destefanis/Discord): https://github.com/destefanis/figma-table-of-contents
- Figma Inspector (Destefanis/Discord): https://github.com/destefanis/figma-style-inspector
- Figma Project Scaffold (Destefanis/Discord): https://github.com/destefanis/Discord-Figma-Project-Scaffold
- Figma Skeleton Generator (Bryan Berger/Discord): https://github.com/bryanberger/figma-skeleton-generator
- Tessl Registry - discord-ui-skills: https://tessl.io/registry/skills/github/ihlamury/design-skills/discord-ui-skills

### Pesquisa Cientifica - Legibilidade e Modo Escuro
- Palmén, Gilbert & Crossland (Google), "How Bold Can We Be? The Impact of Adjusting Font Grade on Readability in Light and Dark Polarities" (CHI 2023): https://dl.acm.org/doi/fullHtml/10.1145/3544548.3581552
- Chen et al., "Text Readability of Smartphone in Dark Mode: Effects of Font Type, Font Weight and Color" (Springer/HCII 2023): https://dlnext.acm.org/doi/10.1007/978-3-031-34866-2_2
- Andrew & Tigwell (RIT), "Understanding the Experiences of People With and Without Vision Impairments When Using Mobile User Interface Alternative Color Modes" (ACM 2025): https://par.nsf.gov/biblio/10640770
- Laine (Turku UAS), "Impact of Dark Mode on User Experience and Eye Comfort" (2025): https://www.theseus.fi/bitstream/handle/10024/896088/Laine_Jere.pdf
- Diaz Alonso (Politecnico di Milano), "Proposal of new WCAG Success Criteria for Mobile Accessibility" (2021): https://www.politesi.polimi.it/bitstream/10589/179726/1/2021_10_Diaz.pdf
- Design Shack, "Typography in Dark Mode: How to Optimize Fonts for Low-Light UI": https://designshack.net/articles/typography/dark-mode-typography/
- Google Fonts Knowledge, "Exploring typefaces with multiple weights or grades": https://fonts.google.com/knowledge/choosing_type/exploring_typefaces_with_multiple_weights_or_grades
- Readability Consortium, PDF do estudo Palmén: https://thereadabilityconsortium.org/wp-content/uploads/2023/07/How-bold-can-we-be-The-impact-of-adjusting-font-grade-on-readability-in-light-and-dark-polarities-1.pdf

### Ciencia da Cor (Blurple e Daltonismo)
- Colorxs - Discord Blurple Brand Color: https://www.colorxs.com/color/discord-blurple
- Colorxs - Discord Blurple Palettes: https://www.colorxs.com/palette/color/discord-blurple
- hexcolor.co - #5865F2: https://hexcolor.co/hex/5865f2
- colorify.rocks - #5865F2: https://colorify.rocks/color/5865f2
- brandcolor.dev - Discord HEX Colors: https://brandcolor.dev/brands/discord
- W3C CSSWG Issue #5169 - "Add blurple as an official CSS color": https://github.com/w3c/csswg-drafts/issues/5169
- Schulz-Hildebrandt (arXiv 2025), "Why purple appears blue in peripheral vision": https://arxiv.org/html/2509.11582v4
- Know Your Meme, "Discord Rebrand Accessibility Criticism": https://knowyourmeme.com/news/discord-rebrand-criticized-by-users-over-new-logo-and-possible-accessibility-issues
- MakeUseOf, "Why Does Everyone Hate Discord's Rebrand?": https://www.makeuseof.com/why-does-everyone-hate-discord-rebrand/
- Android Police, Discord 6th Anniversary Rebrand: https://www.androidpolice.com/2021/05/14/discord-celebrates-turning-6-with-a-new-logo-new-colors-and-a-whole-refreshed-look/
- Creative Bloq, "I can't decide what colour the Discord logo is": https://www.creativebloq.com/news/blurple-logo
- ResearchGate, "Color representations of normals and congenital red-green color deficiencies": https://www.researchgate.net/publication/381812941
- arXiv 2024, "Computational Trichromacy Reconstruction" (UIST): https://arxiv.org/abs/2408.01895
- MDPI Journal of Imaging 2025, "Effective color schemes for all types of color blindness": https://mdpi-res.com/d_attachment/jimaging/jimaging-11-00268/article_deploy/jimaging-11-00268.pdf
- serenity-rs commit adicionando cores Discord: https://github.com/serenity-rs/serenity/commit/c0463c0d7f04250aab178e823c19f4866ffa9e09

### APCA e WCAG 3
- APCA Documentation: https://git.apcacontrast.com/documentation/APCA_in_a_Nutshell.html
- APCA Introduction (Myndex Research): https://raw.githubusercontent.com/Myndex/apca-introduction/59a21acd68ce0e55dca1b4b702d4add4ac2e4099/README.md

### Teoria de Daltonismo
- Brettel-Vienot-Mollon Algorithm (1997): https://github.com/dmccreary/automating-instructional-design/blob/main/docs/sims/color-blindness-simulator/index.md
- @cantoo/color-blindness (TypeScript): https://www.jsdelivr.com/package/npm/@cantoo/color-blindness
- ArcGIS Pro CVD Simulator: https://pro.arcgis.com/en/pro-app/3.4/get-started/color-vision-deficiency-simulator.htm
- Colorlite - Confusion Lines: https://www.colorlitelens.com/red-green-color-blind-test-more.html
- UX Matters, "How to Design for Color Independence" (2024): https://www.uxmatters.com/mt/archives/2024/09/how-to-design-for-color-independence-creating-inclusive-digital-experiences.php
- University of Virginia, Color Accessibility: https://digitalaccessibility.virginia.edu/accessibility-color
- UNIQUE (UPM), Chromatic Accessibility: https://unique.aq.upm.es/en/accesibilidad-cromatica/
- Level Access, Color Blindness Accessibility: https://www.levelaccess.com/blog/color-blindness-accessibility-what-designers-need-to-know/
- Design Shack, "Color Accessibility Beyond Contrast": https://designshack.net/articles/accessibility/color-accessibility-design/

### Utilidades
- Appuals, "What is Discord's GG Sans Font?": https://appuals.com/discords-gg-sans-font/
- PiunikaWeb, "Discord 'gg sans' font announcement": https://piunikaweb.com/2022/12/02/discord-gg-sans-font-announcement-fans-asking-for-proper-reveal/
- PiunikaWeb, "Discord 'gg sans' font criticized by users": https://piunikaweb.com/2022/12/20/discord-gg-sans-font-criticized-by-users-call-for-option-to-revert-it/
- PrimaGames, "Did Discord Change its Font?": https://primagames.com/tips/did-discord-change-its-font-answered
- Making use of, "How to Use Different Fonts on Discord": https://www.makeuseof.com/how-to-use-discord-fonts
- Logos Fandom - Discord: https://logos.fandom.com/wiki/Discord
- Discord Fandom Wiki - Whitney: https://discord.fandom.com/wiki/Whitney
- GitHub Desktop Issue #9389: https://github.com/desktop/desktop/issues/9389
- GitHub PWABuilder Issue #4744 (contraste 2.7:1): https://github.com/pwa-builder/PWABuilder/issues/4744
- GitHub Django OCG Issue #25 - acessibilidade: https://github.com/django/online-community-working-group/issues/25

---

*Fim do relatorio. Pesquisa realizada em 3 de julho de 2026.*
