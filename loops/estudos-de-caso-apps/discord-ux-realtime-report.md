
# Relatorio de Pesquisa: UX, Performance Percebida e Interacao em Tempo Real no Discord

**Data:** 3 de julho de 2026
**Contexto:** Estudo de caso de UX/UI/Ergonomia — Discord

---

## 1. Performance Percebida: Os Limiares Psicologicos da Resposta Humana

### 1.1 A Estrutura Classica de Nielsen (1968/1993)

A base de toda a ciencia de performance percebida vem dos tres limiares definidos por **Robert B. Miller (1968)** e popularizados por **Jakob Nielsen (1993)** , que permanecem validos pois estao enraizados na psicologia perceptiva humana e nao mudam com a tecnologia.

| Limiar | Tempo | Efeito no Usuario | Exemplo no Discord |
|--------|-------|-------------------|-------------------|
| **Instantaneo** | < 100 ms | O usuario sente que esta manipulando _diretamente_ a interface. Nenhum feedback especial e necessario. O cerebro trata como extensao da acao do usuario. | Clique em reacao emoji (like), toggle de mute, hover em nome de usuario |
| **Fluxo** | 100 ms – 1 s | O usuario nota o atraso mas mantem o fluxo de pensamento ininterrupto. Sente que o computador esta "trabalhando". | Envio de mensagem, abertura de canal, carregamento de lista de usuarios |
| **Atencao** | 1 s – 10 s | O usuario consegue manter a atencao na tarefa por ate ~10s. Apos isso, a atencao se desvia — ele troca de aba, pega o celular, ou abandona. | Carregamento de historico de mensagens, upload de arquivo grande |
| **Abandono** | > 10 s | O usuario perde o contexto da tarefa na memoria de curto prazo. Necessario indicador de progresso com tempo estimado. | Sincronizacao inicial, carregamento de servidor grande |

**Fonte:** [Nielsen Norman Group — Response Times: The 3 Important Limits](https://www.nngroup.com/articles/response-times-3-important-limits/)
**Fonte:** [UX Tigers — Time Scales of UX](https://www.uxtigers.com/post/time-scales-ux)
**Fonte:** [Dabrowski & Munson (CHI 2001) — Is 100 Milliseconds Too Fast?](https://dl.acm.org/doi/epdf/10.1145/634067.634255)

### 1.2 Onde o Discord Se Encaixa

O Discord opera majoritariamente na faixa **< 100 ms a ~500 ms** para a maioria das interacoes cotidianas:

- **Reacoes emoji:** Instantaneas (< 50 ms localmente) devido ao padrao **Optimistic UI** — a interface atualiza antes da confirmacao do servidor.
- **Envio de mensagens:** A mensagem aparece instantaneamente no chat (otimista), com fallback silencioso se o servidor falhar.
- **Autocomplete de comandos:** Usa debounce de ~300 ms para balancear responsividade com eficiencia de API.
- **Indicadores de digitacao:** Aparecem em ~500 ms apos o inicio da digitacao, expiram apos 5 segundos de inatividade.
- **Mudanca de canal:** Pode levar 200-800 ms devido ao carregamento do historico — dentro do limiar de "fluxo" mas perceptivel.

### 1.3 Otimismo como Estrategia de Performance Percebida

O **Optimistic UI** e a principal arma do Discord para parecer mais rapido do que realmente e:

```
Fluxo Tradicional (Pessimista):
  Usuario clica → Loading spinner → Servidor processa → UI atualiza

Fluxo Otimista (Discord):
  Usuario clica → UI atualiza instantaneamente → Servidor processa em background → Confirma ou reverte silenciosamente
```

**Impacto:** A pesquisa mostra que o feedback visual imediato reduz o tempo de espera percebido em **ate 40%**, mesmo quando o tempo real de processamento e o mesmo.

**Fontes:**
- [dev.to — Optimizing for Perceived Performance](https://dev.to/maurya-sachin/optimizing-for-perceived-performance-the-psychology-behind-fast-uis-4c11)
- [EpicReact.dev — useOptimistic to Make Your App Feel Instant](https://www.epicreact.dev/use-optimistic-to-make-your-app-feel-instant-zvyuv)
- [freeCodeCamp — Improve UX with Optimistic UI and SWR](https://www.freecodecamp.org/news/improve-user-experience-with-optimistic-ui-swr)
- [A List Apart — Performance: Showing Versus Telling](https://alistapart.com/article/performance-showing-versus-telling/)

---

## 2. Padroes de UX em Tempo Real

### 2.1 Arquitetura de Eventos: Gateway WebSocket

O Discord opera um **Gateway WebSocket** proprietario que funciona como espinha dorsal de toda comunicacao em tempo real. A arquitetura e baseada em **pub/sub** com canais (rooms) como unidade de escopo.

**Componentes principais:**
- **Gateway:** Servidores Elixir/BEAM que mantem conexoes WebSocket persistentes
- **Compressao zstd-stream:** Reduz o trafego WebSocket em ~40% (compressao de ~6x para ~10x)
- **PASSIVE_UPDATE_V2:** Updates delta-only para servidores inativos, economizando 20% adicional de banda
- **Sharding:** Ate 2500 guildas por shard para balanceamento de carga

**Fontes:**
- [Discord Blog — How Discord Reduced Websocket Traffic by 40%](https://web.archive.org/web/20250207021824/https://discord.com/blog/how-discord-reduced-websocket-traffic-by-40-percent)
- [DeepWiki — Twilight Gateway Architecture](https://deepwiki.com/twilight-rs/twilight/3-gateway-system)
- [DSharpPlus Docs — Gateway Compression](https://dsharpplus.github.io/DSharpPlus/articles/advanced_topics/compression.html)

### 2.2 Indicadores de Presenca

O Discord implementa presenca em tempo real com as seguintes caracteristicas:

| Caracteristica | Implementacao |
|---------------|--------------|
| **Status online/offline/idle** | Propagado via WebSocket em < 2s |
| **Multiplos dispositivos** | O dispositivo mais recentemente ativo determina o status |
| **Timeout de idle** | 5-10 min para ferramentas em tempo real |
| **Modo invisivel/DND** | Opcoes de privacidade que nao expoem tempo de pagina |
| **Escala (10k+ usuarios)** | Pub/sub com Redis; agregacao de contagens |

**Fonte:** [IdeaPlan — Presence and Activity Indicator Spec Template](https://www.ideaplan.io/templates/presence-template)

### 2.3 Indicadores de Digitacao

O fluxo de eventos de digitacao do Discord segue este padrao:

```
Cliente → Servidor: Gateway envia evento typing_start
Servidor → Sala: Transmite para todos os outros clientes na sala
```

**Regras de implementacao:**
- Maximo de **1 emissao de typing a cada 2 segundos** (rate limiting)
- **Auto-expiracao apos 5 segundos** sem novo evento
- **Debounce de saida:** 3-5 segundos apos ultima tecla
- **Logica de UI:**
  - 1 usuario: "Fulano esta digitando..."
  - 2 usuarios: "Fulano e Ciclano estao digitando..."
  - 3+ usuarios: "Fulano e mais 2 estao digitando..."

**Fontes:**
- [dev.to — Real-Time Typing Indicators with Socket.IO](https://dev.to/forinda/real-time-typing-indicators-and-presence-tracking-with-kickjs-and-socketio-3jh1)
- [Conf42 — The Code Behind Every Real-Time Collaboration Tool](https://www.conf42.com/Platform_Engineering_2024_Vitor_Norton_code_collaboration_realtime)
- [GitHub — Roxabi Issue #229: Typing Indicator Implementation](https://github.com/Roxabi/roxabi-factory/issues/229)

### 2.4 Autocomplete de Comandos com Debounce

O autocomplete de comandos do Discord (/) usa debounce de ~300ms com as seguintes caracteristicas:

| Aspecto | Configuracao |
|---------|-------------|
| **Debounce** | 250-400ms (300ms e o ponto ideal) |
| **Min. caracteres** | 2-3 antes de buscar |
| **Max. resultados** | 25 (limite do Discord) |
| **Cancelamento** | AbortController para cancelar fetch anterior |
| **Timeout servidor** | 3 segundos (limite da API Discord) |

**Fonte:** [FreeCodeCamp — Debouncing in React Autocomplete](https://www.freecodecamp.org/news/deboucing-in-react-autocomplete-example/)
**Fonte:** [GitHub — Discord Bot Autocomplete Task #541](https://github.com/cpike5/discordbot/issues/541)

---

## 3. Micro-interacoes: Design e Timing

### 3.1 Tabela de Duracao e Base Cognitiva

| Duracao | Papel Cognitivo | Aplicacao UI no Discord | Sensacao |
|---------|----------------|------------------------|----------|
| **70 ms** | Limiar de "passagem"; subconsciente | Hover em avatar, clique de botao, toggle de mute | Instantaneo, direto |
| **100-150 ms** | Fase de conscientizacao; manipulacao direta | Arrastar membro para call, clique de reacao | Rapido, responsivo |
| **150-240 ms** | Limiar de compreensao minima; impressao causal ainda forte | Transicao de canal, expansao de sidebar | Natural, esperado |
| **240-400 ms** | Comunicacao do sistema; limite causal fraco | Toast de notificacao, modal de convite, expansao grande | Deliberado, claro |
| **400-500 ms** | Movimento grande; feedback importante | Transicao de viewport, sobreposicao de tela cheia | Intencional, pesado |
| **500 ms+** | Apenas continuo/preditivo | Placeholder de carregamento, loop de ambiente | Potencialmente frustrante |

### 3.2 Base Cognitiva Detalhada

**70ms — Pass-through threshold:**
- Tempo minimo para um objeto parecer "atravessar" outro sem causalidade percebida (Michotte, 1963)
- A 70ms, o movimento e percebido como continuo, nao causal

**100-150ms — Fase de conscientizacao:**
- 60-80ms: Gatilhos de atencao inconsciente (contraste, escala, posicao, repeticao)
- 100-150ms: Fase de conscientizacao — reconhecimento de padroes ocorre _antes_ da consciencia
- 150-200ms: O evento emerge na _consciencia_

**160ms — Limite causal:**
- Impressoes causais caem substancialmente em atrasos de 160ms ou mais (Ware et al., 1999)

**200-300ms — Pico de responsividade percebida:**
- A fixacao do olho humano leva ~350ms — o maximo antes que instinctos naturais incentivem o foco a mudar
- Faixa ideal: 150-350ms para balancear velocidade, compreensibilidade e confianca

**Fontes:**
- [Carbon Design System — Motion Guidelines](https://carbondesignsystem-git-main-carbon-design-system.vercel.app/patterns/loading-pattern/)
- [SAP Fiori Design — Motion Design](https://www.sap.com/design-system/fiori-design-web/v1-71/foundations/visual/motion-design)
- [Cybozu — Guidelines for Animation Timing](https://cybozu.chromatic.com/blog/guidelines-for-animation-timing/)
- [Break Free Design — The Ultimate Guide to Animation in UX](https://www.breakfreegraphics.com/design-blog/the-ultimate-guide-to-proper-use-of-animation-in-ux/)
- [Design Systems Collective — Micro-interactions: Creating a Motion Language](https://www.designsystemscollective.com/micro-interactions-in-design-systems-creating-a-motion-language-1a13c7e9506a)

### 3.3 Acessibilidade de Movimento (prefers-reduced-motion)

O Discord implementa suporte a `prefers-reduced-motion` para usuarios com disturbios vestibulares.

**Criterio WCAG 2.3.3 — Animacao de Interacoes (Nivel AAA):**
- Animacao de movimento acionada por interacao pode ser desabilitada, a menos que essencial
- ~35% dos adultos acima de 40 anos tem disfuncao vestibular
- Movimento pode desencadear: vertigem, nausea, dor de cabeca, desorientacao

**Implementacao:**
```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

**Fontes:**
- [W3C WCAG 2.2 — Technique C39](https://www.w3.org/WAI/WCAG22/Techniques/css/C39.html)
- [MDN — Using Media Queries for Accessibility](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Media_queries/Using_for_accessibility)
- [TestParty — WCAG 2.3.3 Animation from Interactions Guide](https://testparty.ai/blog/wcag-animation-interactions-guide)
- [Yale Usability — Animated Content and Timing](https://usability.yale.edu/digital-accessibility/accessibility-resources/accessibility-articles/animated-content-and-timing)

---

## 4. Virtualizacao e Scroll Infinito

### 4.1 O Problema do Scroll Bidirecional em Chat

Diferente de listas virtuais normais (ancoradas no topo, anexam no fim), o chat do Discord requer:

1. **Novas mensagens no fim** (mais recentes embaixo)
2. **Historico antigo prepended no topo** (scroll para cima carrega mais)
3. **Mensagens crescendo token-a-token** durante streaming
4. **"Follow only when pinned"** — se o usuario scrollou para cima, novas mensagens NAO devem puxa-lo de volta

**Fonte:** [TanStack Blog — Chat UIs Are Lists Until They Aren't](https://tanstack.com/blog/tanstack-virtual-chat)

### 4.2 Desafios Tecnicos Comuns

| Problema | Descricao | Mitigacao |
|----------|-----------|-----------|
| **Perda de posicao no prepend** | Scroll offset nao ajusta quando itens adicionados acima do viewport | Tracking por chave + delta de scroll |
| **Pulo na transicao de range** | Recalculo do intervalo visivel causa pulo abrupto | Calculo cuidadoso de offset |
| **Desorientacao** | Usuario nao sabe onde esta na lista infinita | Indicadores de scroll, marcadores de posicao |
| **Streaming** | Item crescendo empurra conteudo para baixo | Ancoragem no fim com compensacao de delta |
| **Flicker na restauracao** | Flash visivel ao restaurar scroll | requestAnimationFrame, hooks antes do paint |
| **Altura dinamica imprecisa** | Altura estimada vs real causa desvio | measureElement / callbacks de medicao |

### 4.3 Estrategias de Implementacao

**TanStack Virtual (usado por clones do Discord):**
- `anchorTo: 'end'` — preserva o fim da lista como borda estavel
- `followOnAppend: true` — auto-scroll apenas se usuario estava proximo ao fim
- **Tracking por chave estavel** (`getItemKey`) — nao por indice

**Ajuste manual de scroll:**
```js
requestAnimationFrame(() => {
  const delta = container.scrollHeight - prevScrollHeightRef.current;
  container.scrollTop = delta; // ou += delta
});
```

**Arquitetura de eventos (Discord production):**
- **Armazenamento hibrido:** Hot data em Cassandra, cold data em S3 + RocksDB
- **Snapshots incrementais** para carregamento rapido de canal
- **Chunking de eventos** (grupos de 5 minutos) para replay eficiente

**Fontes:**
- [TanStack Blog — Chat UIs Are Lists Until They Aren't](https://tanstack.com/blog/tanstack-virtual-chat)
- [TanStack Virtual Discussion #733 — Keep list from scrolling up when prepending](https://github.com/TanStack/virtual/discussions/733)
- [TanStack Virtual Discussion #1018 — Preserve scroll position when new items added](https://github.com/TanStack/virtual/discussions/1018)
- [Dev.to — How Discord Uses Event Sourcing for Message History](https://dev.to/alex_aslam/how-discord-uses-event-sourcing-for-message-history-3n9h)
- [DeepWiki — Discord Clone Chat Components](https://deepwiki.com/3xp1o1t/discord-clone/4.3-chat-components)

### 4.4 Problemas de Usabilidade do Scroll Infinito

Pesquisa academica (Universidade de Konstanz) identifica:
- Usuarios **perdem a nocao de posicao** em listas infinitas
- **Carga cognitiva aumenta** ao tentar comparar itens que ja sairam da tela
- **Encontrar itens vistos anteriormente** e dificil
- **Transferir itens para memoria ou abas separadas** introduz custo de troca de contexto

**Fonte:** [Universidade de Konstanz — Infinite Scroll Usability Research](https://kops.uni-konstanz.de/server/api/core/bitstreams/d88d224a-70a8-4af6-a825-5d34c09cd3d4/content)

---

## 5. Estados de Carregamento e Vazio

### 5.1 Skeleton Screens vs Spinners

| Padrao | Melhor Para | Tolerancia do Usuario |
|--------|------------|----------------------|
| **Spinner** | Operacoes curtas < 2s (login, salvar, confirmar) | Ate ~5s antes de frustracao |
| **Skeleton** | Telas com conteudo pesado (feeds, perfis, listas) | Ate ~11s antes de frustracao |

**Pesquisa chave:** Usuarios percebem uma espera de **3 segundos com skeleton como equivalente a 1,5 segundos com spinner** — um ganho de 30-50% na performance percebida sem alteracoes no backend.

**Fontes:**
- [Appy Pie — Loading States in Mobile Apps](https://www.appypie.com/blog/loading-states-mobile-apps)
- [Microsoft Fluent 2 — Wait UX](https://fluent2.microsoft.design/wait-ux)
- [Carbon Design System — Loading Pattern](https://carbondesignsystem-git-main-carbon-design-system.vercel.app/patterns/loading-pattern/)

### 5.2 Regras de Timing para Carregamento

| Duracao | Acao Recomendada |
|---------|------------------|
| **< 300 ms** | Nao mostrar indicador nenhum (flash parece glitch) |
| **300 ms – 1 s** | Mostrar skeleton ou feedback sutil |
| **1 – 3 s** | Spinner ou skeleton com shimmer |
| **> 3 s** | Barra de progresso ou indicador com estimativa |
| **5 s+** | Maioria dos usuarios abandona se ve apenas spinner |

**Fonte:** [Microsoft Fluent 2 — Wait UX](https://fluent2.microsoft.design/wait-ux)

### 5.3 Os Cinco Estados de Toda Interface

O framework "UI Stack" (Scott Hurff) identifica que toda tela precisa contemplar:

1. **Estado ideal** — conteudo completo, maximo valor
2. **Estado vazio** — primeiro uso, dados limpos, sem resultados
3. **Estado de erro** — algo deu errado
4. **Estado parcial** — esparsamente povoado
5. **Estado de carregamento** — dados estao sendo buscados

**Tres tipos de estado vazio:**

| Tipo | Exemplo | Melhor Pratica |
|------|---------|---------------|
| **Primeiro uso** | Usuario novo, sem dados | Onboarding, coach marks, conteudo de exemplo |
| **Dados limpos pelo usuario** | Caixa de entrada vazia | Recompensar o usuario, estimular nova acao |
| **Sem resultados** | Busca vazia | Sugerir alternativas, corrigir ortografia |

**Fontes:**
- [Smart Interface Design Patterns — How to Fix a Bad UI](https://smart-interface-design-patterns.com/articles/how-to-fix-a-bad-user-interface/)
- [zeroheight — Designing Beyond the Happy Path](https://zeroheight.com/blog/designing-beyond-the-happy-path-in-design-systems/)

### 5.4 Como o Discord Aplica Isso

- **Canais:** Skeleton de mensagens enquanto historico carrega
- **Lista de servidores:** Spinner ate sincronizacao inicial
- **Busca de mensagens:** Skeleton + shimmer para resultados
- **Upload de arquivos:** Barra de progresso com percentual
- **Primeiro uso (servidor novo):** Estado vazio com ilustracao + CTA "Enviar primeira mensagem"

---

## 6. Ciclos de Feedback em Apps Sociais

### 6.1 Reforco de Recompensa Variavel (Efeito "Caça-niqueis")

Multiplas fontes confirmam que plataformas sociais emprestam deliberadamente **esquemas de recompensa variavel** do design de maquinas caça-niqueis (condicionamento operante de Skinner):

- **Recompensas imprevisiveis** — likes, notificacoes e conteudo aparecem em intervalos irregulares, mantendo o usuario em estado de antecipacao elevada
- **Picos de dopamina na antecipacao, nao so na recompensa** — O cerebro libera dopamina _antes_ da recompensa chegar, tornando o ato de "verificar" em si mesmo reforcador
- **O gesto "puxar-para-atualizar" (pull-to-refresh)** funciona como a alavanca de um caça-niqueis

**Fontes:**
- [The Guardian — Social Media Copies Gambling Methods](https://amp.theguardian.com/technology/2018/may/08/social-media-copies-gambling-methods-to-create-psychological-cravings)
- [Responsible AI Foundation — Why Social Media Feels Addictive: Dopamine by Design](https://www.responsibleaifoundation.com/post/why-social-media-feels-addictive-dopamine-by-design)
- [Design Indaba — Engineered Seduction: How Slot Machines Keep Users Fixated](https://www.designindaba.com/articles/point-view/engineered-seduction-how-slot-machines-keep-users-fixated)
- [Make Tech Easier — Tristan Harris Testimony to US Senate](https://maketecheasier.com/mte-google-s-former-design-ethicist-tristan-harris-testified-to-the-us-senate-in-201/)
- [Deceptive Patterns — Addictive Design](https://deceptive.design/types/browse/?sel=%2Ftypes%2Faddictive-design)
- [The New Indian Express — The Feed That Feeds on You](https://www.newindianexpress.com/lifestyle/science/2026/Mar/05/the-feed-that-feeds-on-you)

### 6.2 Feedback Instantâneo e Dopamina

O Discord utiliza varios mecanismos de feedback instantaneo que engatilham circuitos de recompensa:

| Mecanismo | Gatilho Psicologico | Implementacao Discord |
|-----------|---------------------|---------------------|
| **Reacoes emoji** | Recompensa social instantanea, validacao | Update otimista, animacao suave, confirmacao visual |
| **Notificacoes de mensagem** | Antecipacao de recompensa social | Badge com contagem, som de notificacao |
| **Mencao (@)** | Sinal de relevancia social imediata | Destacamento visual, notificacao prioritario |
| **Indicador "digitando"** | Antecipacao de resposta social | Timeout de 5s, animacao de 3 pontos |
| **Status online** | Disponibilidade percebida | Propagacao em tempo real via WebSocket |
| **Streak de mensagens** | Continuidade, medo de perder | (Snapchat usa isso extensivamente) |

**Fontes:**
- [TU Eindhoven — Philosophy & Technology Journal](https://pure.tue.nl/ws/portalfiles/portal/218611428/s13347_022_00530_6.pdf)
- [University of Naples — Skinner Boxes: The Design of Social Media](https://www.scienzesociali.unina.it/en_GB/-/57684670-skinner-boxes-the-design-of-social-media-ramon-rispoli)
- [SOM Salud Mental 360 — Keys to Understanding Manipulative Design of Social Networks](https://prevencionsuicidio.som360.org/en/articulo/keys-understanding-manipulative-design-social-networks)
- [Times Live — The Addictive Architecture of Social Media](https://arenaholdings-timeslive-prod.web.arc-cdn.net/lifestyle/2026-03-07-the-addictive-architecture-of-social-media/)

### 6.3 O Custo Psicologico

- **Comparacao social ascendente** — Feeds curados corroem a autoestima, especialmente em adolescentes
- **70% dos adolescentes** expostos a conteudo de influenciadores de beleza/fitness relataram sentimentos de inferioridade
- **FOMO** (Fear of Missing Out) explorado por conteudo temporario, streaks e badges de notificacao
- **Pesquisas internas do Meta:** 33% dos usuarios (48% das meninas adolescentes) dizem que Instagram as faz sentir pior sobre seus corpos

---

## 7. O Custo UX do Electron

### 7.1 O Problema Estrutural

O Discord e construido sobre o **Electron** (Chromium + Node.js empacotados). Cada servidor, canal e funcionalidade se comporta como uma **aba separada de navegador** — a memoria acumula exponencialmente.

**Comparacao de consumo de memoria (uso real):**

| App | Framework | Memoria (Ocioso) | Memoria (Ativo) |
|-----|-----------|-----------------|-----------------|
| WhatsApp antigo | UWP/WinUI (nativo) | < 100 MB | ~100 MB |
| WhatsApp novo | WebView2 | ~300 MB | 1.2 GB+ |
| **Discord** | **Electron** | **~1 GB** | **2-4 GB+** |
| Teams | WebView2 (ex-Electron) | ~1 GB | 2 GB+ |
| Telegram | Nativo (C++) | ~150 MB | ~350 MB |

**Fontes:**
- [TechSpot — Discord is force-restarting itself on Windows 11 to stop eating your RAM](https://www.techspot.com/news/110542-discord-force-restarting-itself-windows-11-stop-eating.html)
- [Bangkok Post — Discord plans auto-restart feature to tackle high RAM usage](https://www.bangkokpost.com/life/tech/3161393/discord-plans-autorestart-feature-to-tackle-high-ram-usage)
- [ITHome — Discord tests 4GB memory breaker](https://m.ithome.com/html/902895.htm)
- [Hacker News — Discord: A case study in performance optimization](https://hn.nuxt.dev/item/47010147)
- [Hacker News — Electron resource-intensive discussion](https://news.ycombinator.com/item?id=18733989)

### 7.2 O Mecanismo de Auto-Restart a 4GB

Em 2025-2026, o Discord reconheceu publicamente o problema e introduziu um mecanismo de **auto-restart forcado** quando a memoria excede **4GB** — mas apenas quando TODAS as condicoes sao atendidas:

- App rodando por **mais de 1 hora**
- Usuario **inativo (idle)** por **pelo menos 30 minutos**
- Usuario **NAO esta em chamada de voz ou video**
- Limitado a **1 vez a cada 24 horas**

**O que o Discord ja corrigiu:**
- Desde outubro de 2025: 9+ vazamentos de memoria, deadlocks e buffer overflows corrigidos
- Nova telemetria e ferramentas de profiling de memoria
- Reducao de ~5% no P95 de memoria (5% dos piores casos)
- Trabalhando com parceiros de hardware em correcoes de driver/OS (alguns issues ligados a codificacao AV1 e drivers NVIDIA)

### 7.3 Electron vs Nativo: Percepcao do Usuario

| Aspecto | App Nativo | App Electron (Discord) |
|---------|-----------|----------------------|
| **Tempo de inicializacao** | Quase instantaneo | Lento (bundlar Chromium, parsear JS) |
| **Memoria** | Normalmente < 100 MB | 300 MB - 4 GB+ |
| **Responsividade** | Rapido, sem lag de input | Atraso via requestAnimationFrame, overhead do event loop JS |
| **Integracao com SO** | Cidadao de primeira classe do sistema | Frequentemente quebrado ou reimplementado; parece "alienigena" |
| **Eficiencia de CPU** | Baixo overhead | Alto CPU em background mesmo quando ocioso |

### 7.4 O Debate Etico da Troca

Defensores do Electron argumentam:
> "Usuarios nao se importam com RAM a menos que o app se torne inusavel."
> "VS Code e um app Electron amplamente elogiado — o problema sao apps Electron ruins, nao o Electron em si."

**Mas a tendencia e clara:** O custo do Electron para o usuario final e significativo — consumo de recursos, lentidao, drenagem de bateria. A troca e **conveniencia do desenvolvedor vs. experiencia do usuario**.

**Fonte:** [Hey.com — My 'Electron is not the future' rant](https://world.hey.com/slade/my-electron-is-not-the-future-rant-5af14ed2)

---

## 8. Arquitetura Frontend do Discord

Para contextualizar os achados acima, a arquitetura frontend do Discord merece destaque:

### 8.1 Componentes Principais

| Componente | Descricao |
|-----------|-----------|
| **Flux (fork proprietario)** | Arquitetura de estado com dispatcher central (`discord/Dispatcher`); 95%+ compartilhado entre web e desktop |
| **React (patched fork)** | Base da UI, mesma codebase para web (browser) e desktop (Electron) |
| **react-focus-rings** | Sistema de indicadores de foco para navegacao por teclado, open-source (MIT) |
| **Virtualizacao** | Listas de mensagens virtualizadas com alturas variaveis |
| **Stores Flux** | Estado da aplicacao mantido em stores que despacham e recebem eventos |

**Fontes:**
- [moonlight-mod — Cookbook: Discord Flux Architecture](https://moonlight-mod.github.io/ext-dev/cookbook/)
- [GitHub — discord/focus-rings](https://github.com/discord/focus-rings)
- [npm — react-focus-rings](https://www.npmjs.com/package/react-focus-rings)
- [GitNation — Accessibility at Discord by Brandon Dail](https://gitnation.com/contents/accessibility-at-discord)
- [Discord Blog — How Discord Implemented App-Wide Keyboard Navigation](https://blog.discord.com/how-discord-implemented-app-wide-keyboard-navigation-abf073fd71de)

### 8.2 Rewrite em Rust do Read States

Em 2019, o Discord reescreveu o servico **Read States** de Go para Rust, eliminando picos de latencia causados pelo garbage collector do Go:

| Métrica | Antes (Go) | Depois (Rust) |
|---------|-----------|--------------|
| **Picos de latencia** | A cada ~2 min (GC cycle) | Eliminados |
| **CPU** | Linha de base | Reduzido ~30% |
| **Memoria** | Linha de base | Reduzido ~20%+ |
| **Latencia media** | Milissegundos | **Microssegundos** |
| **Cache LRU** | Limitado pelo GC | 8 milhoes de entradas |

**Fonte:** [Discord Blog — Why Discord Is Switching From Go to Rust](https://discord.com/blog/why-discord-is-switching-from-go-to-rust)
**Fonte:** [Dev.to — Why Discord Keeps Rewriting Its Stack](https://dev.to/thepavansai/why-discord-keeps-rewriting-its-stack-59l3)

### 8.3 Components V2 (2025)

Lancado em marco de 2025, o Components V2 e uma reformulacao do sistema de componentes de mensagem do Discord, permitindo layouts ricos e estruturados com ate 40 componentes por mensagem.

**Componentes de display (V2):**
- **Container (type 17):** Wrapper com barra lateral de cor de acento
- **Section (type 9):** Grupo de ate 3 textos + acessorio opcional
- **Text Display (type 10):** Texto Markdown (ate 4000 chars)
- **Separator (type 14):** Espacamento vertical com ou sem linha visivel
- **Media Gallery (type 12):** Ate 10 imagens com descricoes
- **Thumbnail (type 11):** Imagem pequena como acessorio
- **File (type 13):** Arquivo anexado unico

**Limites:** Max 10 componentes de topo, max 30 aninhados, max 4000 chars.

**Fontes:**
- [Discord4J — Components V2 Docs](https://docs.discord4j.com/interactions/components-v2)
- [D++ — Components V2](https://dpp.dev/10.1.3/components_v2.html)
- [Discord.Net — Components V2 Guide](https://docs.discordnet.dev/guides/components_v2/advanced.html)
- [npm — discord-container-builder](https://www.npmjs.com/package/discord-container-builder)
- [GitHub — WoozyDev/discord-components-v2](https://github.com/WoozyDev/discord-components-v2)

---

## 9. Sintese e Recomendações de Design

### 9.1 Mapa de Decisoes de UX do Discord

| Interacao | Gatilho | Feedback Imediato | Latencia Maxima Aceitavel | Mecanismo |
|-----------|---------|-------------------|--------------------------|-----------|
| Reagir com emoji | Clique | Emoji preenchido + contagem incrementa | < 50 ms | Optimistic UI |
| Enviar mensagem | Enter | Mensagem aparece no chat | < 50 ms | Optimistic UI + WS |
| Digitar mensagem | Tecla | Indicador de digitacao aparece | ~500 ms | WebSocket + debounce |
| Toggle mute/ mute | Clique | Icone muda instantaneamente | < 30 ms | Estado local (Flux) |
| Mudar de canal | Clique nav | Loading do historico + skeleton | 200-800 ms | Virtual scroll + cache |
| Scroll para historico | Scroll up | Mais mensagens carregam | < 1 s (carga) | Infinite scroll bidirecional |
| Autocomplete / | Digitacao | Sugestoes aparecem | ~300 ms | Debounce + API Gateway |
| Upload de arquivo | Drag/click | Barra de progresso | > 1 s | Progress tracking |

### 9.2 Principais Insights de Design

1. **A ilusao de instantaneidade e o padrao mais importante** — O Discord sacrifica consistencia eventual (no lugar de esperar confirmacao do servidor) para fazer o usuario sentir que o app e rapido. Isso e correto: a pesquisa mostra que < 100 ms de feedback mantem a sensacao de manipulacao direta.

2. **O WebSocket e a espinha dorsal invisivel** — A compressao zstd, o pub/sub, e o sistema de presenca formam a infraestrutura que torna possivel a experiencia "magica" de ver outros usuarios digitando, entrando e saindo em tempo real.

3. **Virtualizacao e o ponto mais fragil da experiencia** — Scroll bidirecional com preservacao de posicao e o problema mais dificil de resolver em UIs de chat. O Discord resolve com tracking por chave, ancoragem no fim e calculo de delta. E uma area onde ate hoje nao existe solucao perfeita.

4. **O Electron e o calcanhar de Aquiles** — O maior problema de UX do Discord nao e de interacao, mas de recursos: 1-4 GB de RAM, lentidao na inicializacao, consumo de bateria. O mecanismo de auto-restart a 4GB e uma admissao implicita de que o problema e estrutural e nao foi resolvido.

5. **As micro-interacoes sao o tempero** — As animacoes de reacao, o shimmer dos skeletons, a transicao entre canais — todas operam na faixa de 70-400ms, que e onde a cognicao humana e mais receptiva. Fora dessa faixa, a experiencia sofre.

6. **Ciclos de feedback e responsabilidade etica** — O Discord (como toda plataforma social) usa recompensa variavel para engajar usuarios. O desafio de design e balancear retencao com saude digital.

---

## Fontes Completas

### Performance Percebida
- [Nielsen Norman Group — Response Times: The 3 Important Limits](https://www.nngroup.com/articles/response-times-3-important-limits/)
- [UX Tigers — Time Scales of UX](https://www.uxtigers.com/post/time-scales-ux)
- [Dabrowski & Munson (CHI 2001) — Is 100 Milliseconds Too Fast?](https://dl.acm.org/doi/epdf/10.1145/634067.634255)
- [A List Apart — Performance: Showing Versus Telling](https://alistapart.com/article/performance-showing-versus-telling/)
- [DebugBear — How Web Performance Impacts User Experience](https://www.debugbear.com/blog/web-performance-user-experience)

### Optimistic UI
- [dev.to — Optimizing for Perceived Performance](https://dev.to/maurya-sachin/optimizing-for-perceived-performance-the-psychology-behind-fast-uis-4c11)
- [EpicReact.dev — useOptimistic to Make Your App Feel Instant](https://www.epicreact.dev/use-optimistic-to-make-your-app-feel-instant-zvyuv)
- [freeCodeCamp — Improve UX with Optimistic UI and SWR](https://www.freecodecamp.org/news/improve-user-experience-with-optimistic-ui-swr)
- [Rubrik Blog — Async React Streamlining UX](https://www.rubrik.com/blog/technology/26/4/async-react-streamlining-ux-with-useroptimistic-use-api-and-useformstatus)
- [OpenReplay — How Optimistic Updates Make Apps Feel Faster](https://blog.openreplay.com/optimistic-updates-make-apps-faster/)

### Padroes Tempo Real
- [Discord Blog — How Discord Reduced Websocket Traffic by 40%](https://web.archive.org/web/20250207021824/https://discord.com/blog/how-discord-reduced-websocket-traffic-by-40-percent)
- [DeepWiki — Twilight Gateway Architecture](https://deepwiki.com/twilight-rs/twilight/3-gateway-system)
- [DSharpPlus Docs — Gateway Compression](https://dsharpplus.github.io/DSharpPlus/articles/advanced_topics/compression.html)
- [dev.to — Typing Indicators and Presence Tracking with Socket.IO](https://dev.to/forinda/real-time-typing-indicators-and-presence-tracking-with-kickjs-and-socketio-3jh1)
- [Conf42 — The Code Behind Every Real-Time Collaboration Tool](https://www.conf42.com/Platform_Engineering_2024_Vitor_Norton_code_collaboration_realtime)
- [IdeaPlan — Presence and Activity Indicator Spec Template](https://www.ideaplan.io/templates/presence-template)

### Micro-interacoes
- [Carbon Design System — Loading Pattern](https://carbondesignsystem-git-main-carbon-design-system.vercel.app/patterns/loading-pattern/)
- [SAP Fiori — Motion Design](https://www.sap.com/design-system/fiori-design-web/v1-71/foundations/visual/motion-design)
- [Cybozu — Guidelines for Animation Timing](https://cybozu.chromatic.com/blog/guidelines-for-animation-timing/)
- [Break Free Design — The Ultimate Guide to Animation in UX](https://www.breakfreegraphics.com/design-blog/the-ultimate-guide-to-proper-use-of-animation-in-ux/)
- [Design Systems Collective — Micro-interactions: Creating a Motion Language](https://www.designsystemscollective.com/micro-interactions-in-design-systems-creating-a-motion-language-1a13c7e9506a)

### Acessibilidade de Movimento
- [W3C WCAG 2.2 — Technique C39](https://www.w3.org/WAI/WCAG22/Techniques/css/C39.html)
- [MDN — Using Media Queries for Accessibility](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Media_queries/Using_for_accessibility)
- [TestParty — WCAG 2.3.3 Animation from Interactions Guide](https://testparty.ai/blog/wcag-animation-interactions-guide)
- [Yale Usability — Animated Content and Timing](https://usability.yale.edu/digital-accessibility/accessibility-resources/accessibility-articles/animated-content-and-timing)

### Virtualizacao e Scroll
- [TanStack Blog — Chat UIs Are Lists Until They Aren't](https://tanstack.com/blog/tanstack-virtual-chat)
- [TanStack Virtual Discussion #733 — Keep list from scrolling up when prepending](https://github.com/TanStack/virtual/discussions/733)
- [TanStack Virtual Discussion #1018 — Preserve scroll position when new items added](https://github.com/TanStack/virtual/discussions/1018)
- [Dev.to — How Discord Uses Event Sourcing for Message History](https://dev.to/alex_aslam/how-discord-uses-event-sourcing-for-message-history-3n9h)
- [Universidade de Konstanz — Infinite Scroll Usability Research](https://kops.uni-konstanz.de/server/api/core/bitstreams/d88d224a-70a8-4af6-a825-5d34c09cd3d4/content)
- [DeepWiki — Discord Clone Chat Components](https://deepwiki.com/3xp1o1t/discord-clone/4.3-chat-components)

### Loading e Estados Vazios
- [Appy Pie — Loading States in Mobile Apps](https://www.appypie.com/blog/loading-states-mobile-apps)
- [Microsoft Fluent 2 — Wait UX](https://fluent2.microsoft.design/wait-ux)
- [Smart Interface Design Patterns — How to Fix a Bad UI](https://smart-interface-design-patterns.com/articles/how-to-fix-a-bad-user-interface/)
- [zeroheight — Designing Beyond the Happy Path](https://zeroheight.com/blog/designing-beyond-the-happy-path-in-design-systems/)

### Ciclos de Feedback e Dopamina
- [The Guardian — Social Media Copies Gambling Methods](https://amp.theguardian.com/technology/2018/may/08/social-media-copies-gambling-methods-to-create-psychological-cravings)
- [Responsible AI Foundation — Why Social Media Feels Addictive: Dopamine by Design](https://www.responsibleaifoundation.com/post/why-social-media-feels-addictive-dopamine-by-design)
- [Design Indaba — Engineered Seduction: How Slot Machines Keep Users Fixated](https://www.designindaba.com/articles/point-view/engineered-seduction-how-slot-machines-keep-users-fixated)
- [Deceptive Patterns — Addictive Design](https://deceptive.design/types/browse/?sel=%2Ftypes%2Faddictive-design)
- [TU Eindhoven — Philosophy & Technology Journal](https://pure.tue.nl/ws/portalfiles/portal/218611428/s13347_022_00530_6.pdf)
- [University of Naples — Skinner Boxes: The Design of Social Media](https://www.scienzesociali.unina.it/en_GB/-/57684670-skinner-boxes-the-design-of-social-media-ramon-rispoli)
- [SOM Salud Mental 360 — Keys to Understanding Manipulative Design](https://prevencionsuicidio.som360.org/en/articulo/keys-understanding-manipulative-design-social-networks)
- [The New Indian Express — The Feed That Feeds on You](https://www.newindianexpress.com/lifestyle/science/2026/Mar/05/the-feed-that-feeds-on-you)
- [Times Live — The Addictive Architecture of Social Media](https://arenaholdings-timeslive-prod.web.arc-cdn.net/lifestyle/2026-03-07-the-addictive-architecture-of-social-media/)

### Electron Performance
- [TechSpot — Discord force-restarting itself on Windows 11](https://www.techspot.com/news/110542-discord-force-restarting-itself-windows-11-stop-eating.html)
- [Bangkok Post — Discord plans auto-restart feature to tackle high RAM usage](https://www.bangkokpost.com/life/tech/3161393/discord-plans-autorestart-feature-to-tackle-high-ram-usage)
- [ITHome — Discord tests 4GB memory breaker](https://m.ithome.com/html/902895.htm)
- [Hacker News — Discord: A case study in performance optimization](https://hn.nuxt.dev/item/47010147)
- [Hacker News — Electron resource-intensive discussion](https://news.ycombinator.com/item?id=18733989)
- [Hey.com — My 'Electron is not the future' rant](https://world.hey.com/slade/my-electron-is-not-the-future-rant-5af14ed2)

### Arquitetura Discord
- [Discord Blog — Why Discord Is Switching From Go to Rust](https://discord.com/blog/why-discord-is-switching-from-go-to-rust)
- [Dev.to — Why Discord Keeps Rewriting Its Stack](https://dev.to/thepavansai/why-discord-keeps-rewriting-its-stack-59l3)
- [moonlight-mod — Cookbook: Discord Flux Architecture](https://moonlight-mod.github.io/ext-dev/cookbook/)
- [GitHub — discord/focus-rings](https://github.com/discord/focus-rings)
- [GitNation — Accessibility at Discord by Brandon Dail](https://gitnation.com/contents/accessibility-at-discord)
- [Discord Blog — How Discord Implemented App-Wide Keyboard Navigation](https://blog.discord.com/how-discord-implemented-app-wide-keyboard-navigation-abf073fd71de)

### Components V2
- [Discord4J — Components V2 Docs](https://docs.discord4j.com/interactions/components-v2)
- [D++ — Components V2](https://dpp.dev/10.1.3/components_v2.html)
- [Discord.Net — Components V2 Guide](https://docs.discordnet.dev/guides/components_v2/advanced.html)
- [npm — discord-container-builder](https://www.npmjs.com/package/discord-container-builder)
- [GitHub — WoozyDev/discord-components-v2](https://github.com/WoozyDev/discord-components-v2)
