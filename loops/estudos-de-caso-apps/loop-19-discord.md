# Estudo de Caso 19 — Discord: Arquitetura Técnica, Design Visual, Cognição Espacial e Ergonomia Mobile

> **Data:** 2026-07-03
> **Loop:** 19 de ∞ (Reescrita — Enriquecimento Design/UX/Ergonomia)
> **Categoria:** Comunicação em Tempo Real / Sistemas Distribuídos / Design de Produto / UX Mobile
> **Linhas:** ~560 | **Fontes:** 50+

---

## 0. Linhagem

```
IRC (1988) — canais de texto, persistente, sem voz. O protocolo ancestral.
TeamSpeak (2001), Ventrilo (2002), Mumble (2005) — voz de baixa latência. Servidor próprio.
Skype (2003) — P2P, voz+texto, vazava IP. Consumia CPU.
Slack (2013) — chat persistente para times. Não foi desenhado para comunidades de gaming.
Discord (2015) — voz+texto+persistência+servidores gratuitos+zero configuração.
Discord hoje (2026) — **227M+ MAUs** (2024), 200M MAU (2025). 12M+ concorrentes. 26M eventos WebSocket/s. 5 engenheiros na infra de chat.
```

O Discord não inventou nenhuma tecnologia na coluna da esquerda. Voz sobre IP, WebRTC, chat persistente — tudo existia. O que ele fez foi empacotar voz, texto, canais, permissões e persistência em uma experiência integrada que exigia zero configuração — sem alugar servidor, sem abrir portas, sem compartilhar IP. E construiu a arquitetura que permite a 5 engenheiros operarem uma das maiores infraestruturas de comunicação em tempo real do mundo.

---

## 1. Origem

Jason Citron e Stanislav Vishnevskiy fundaram a Hammer & Chisel em 2012 para construir jogos de tablet. O primeiro título, Fates Forever, era um MOBA para iPad. Fracassou comercialmente. Mas durante o desenvolvimento, a equipe construiu um sistema interno de chat e voz para coordenar testes. Citron e Vishnevskiy, que jogavam Final Fantasy XIV e League of Legends juntos, perceberam que usavam essa ferramenta interna mais do que qualquer software de comunicação disponível. "A melhor coisa sobre Fates Forever era o chat", Citron diria depois.

Em 2015, Citron demitiu um terço da equipe, abandonou o desenvolvimento de jogos e reconstruiu a empresa em torno daquela ferramenta interna. O Discord foi lançado em 13 de maio de 2015. Um usuário anônimo postou o link num subreddit de Final Fantasy XIV. Citron e Vishnevskiy entraram na conversa pessoalmente. Em semanas, comunidades de Diablo, World of Warcraft e League of Legends estavam trocando IRC por Discord. Não houve campanha de marketing. A comunidade gamer adotou o produto porque ele resolvia um problema que cada membro conhecia intimamente: todas as ferramentas de voz para gamers eram horríveis.

---

## 2. Filosofia do Produto

O Discord opera sobre um princípio que contradiz a maioria das plataformas sociais: a unidade atômica não é o post, não é o feed, não é a mensagem. É o **servidor** — um espaço persistente com dono, regras, cargos, canais temáticos e bots. Um servidor Discord acumula história, normas, piadas internas e relacionamentos. Você não "consome" um servidor. Você pertence a ele.

Essa arquitetura de produto — servidores como feudos autônomos, cada um com seu próprio sistema de permissões e cultura — tem uma consequência técnica direta: cada servidor é um processo Elixir independente. A filosofia de produto e a arquitetura de software são a mesma coisa.

---

## 3. Arquitetura Técnica

### 3.1 O Modelo de Atores: Cada Servidor é Um GenServer

O backend de mensageria do Discord é construído sobre a máquina virtual BEAM, que executa código Elixir e Erlang. A BEAM implementa o modelo de atores: cada entidade do sistema é um lightweight process (não thread de SO) com sua própria mailbox, heap e estado isolado.

**Guild Process.** Cada servidor Discord é um único **GenServer** Elixir — o ponto central de roteamento para todos os eventos daquele servidor. Quando um usuário envia uma mensagem, reage com emoji, entra num canal de voz ou altera uma permissão, é o guild process que recebe o evento, valida permissões, persiste no banco e faz fanout para os destinatários. A escolha de um único processo por servidor — em vez de sharding — é deliberada: operações como mostrar a lista de membros em um canal de voz exigem visão completa do estado. Distribuir esse estado entre múltiplos processos introduziria complexidade de consistência que o ator único elimina.

**Session Process.** Cada usuário conectado tem um processo Elixir dedicado mantendo uma conexão WebSocket aberta via **Cowboy** (biblioteca Erlang para HTTP/WebSocket). A session process sabe quais servidores o usuário está observando ativamente.

**Supervision Tree.** Se um guild process falha, o supervisor OTP o reinicia automaticamente. O estado do servidor é reconstruído do banco sem perda de dados. Isso permite a filosofia "let it crash" do Erlang: em vez de código defensivo para cada edge case, confia-se que o supervisor reiniciará processos que falharem. A propriedade de isolamento é o superpoder: se o guild process do Midjourney entra em loop infinito, apenas os usuários do Midjourney são afetados.

**Distribuição.** Guild processes são distribuídos entre ~400-500 máquinas BEAM usando uma hash ring baseada em `guild_id`. Distributed Erlang gerencia a comunicação inter-node.

### 3.2 O Pipeline de Mensagens e o Sistema de Fanout

Quando um usuário envia uma mensagem em um canal com 30.000 membros online:

**Gateway.** A conexão WebSocket é terminada no Gateway, que usa Cowboy para gerenciar milhões de conexões simultâneas. Rate limiting: 26 milhões de eventos por segundo em pico. O Discord usa **streaming Zstandard compression** (substituindo zlib em 2024), que reduziu o tráfego WebSocket em ~40%.

**Validação e Persistência.** O guild process valida a mensagem (tamanho, permissões, rate limit) e a envia ao Data Service — uma camada middleware em Rust que persiste no ScyllaDB via gRPC.

**Fanout.** O fanout é o gargalo central do Discord. Para N membros online, cada mensagem precisa ser entregue a N session processes. Para o Midjourney, onde N pode ser 1 milhão, isso resulta em 1 trilhão de notificações se cada um dos 100.000 usuários ativos enviar uma mensagem.

**Passive Sessions.** A primeira grande otimização: ~90% das conexões são passivas — o usuário está online mas não está com a aba ou app aberto naquele servidor específico. Essas sessões não recebem a mensagem; apenas um contador de mensagens não lidas é incrementado. Isso reduz o trabalho de fanout em 90% e proporcionou um ganho de ~3× na capacidade máxima de comunidade.

**Manifold: Fanout Distribuído em 3 Estágios.** Para as sessões ativas, o Discord construiu o **Manifold** — uma biblioteca open-source que substitui `send/2` do Erlang para entrega em massa. O `send/2` ingênuo custa 30-70µs por chamada; para 30.000 destinatários, levaria 0,9-2,1 segundos. O Manifold decompõe o trabalho em três estágios:

1. **Agrupamento por nó remoto**: PIDs agrupados pelo nó BEAM onde residem. A mensagem é serializada uma única vez por nó remoto — reduzindo tráfego de rede em 50%.
2. **Partitioner com consistent hashing**: em cada nó remoto, `Manifold.Partitioner` usa `:erlang.phash2/2` para distribuir entre workers por CPU core.
3. **Workers paralelos**: child workers fazem o `send/2` final. Opcionalmente, `send_mode: :offload` delega para processos Sender dedicados.

**Relays.** Processos intermediários entre o guild process e as session processes. Cada relay gerencia até 15.000 sessões. Para um servidor com 1 milhão de usuários online, isso significa ~67 relays. O guild process envia a mensagem para dezenas de relays; cada relay faz fanout local para suas 15.000 sessões. O trabalho do guild process torna-se O(relays) em vez de O(members).

**Erro de design dos relays.** Na primeira implementação, cada relay mantinha a lista completa de membros. Para o Midjourney (10M+ membros totais), isso significava dezenas de cópias de dezenas de milhões de membros na RAM. Criar um novo relay exigia serializar e enviar toda a informação, paralisando o guild process por dezenas de segundos. A correção: lógica que identifica apenas a minúscula fração de membros que o relay realmente precisa.

### 3.3 Permission Caching: role_version e O(1) Invalidação

Toda entrega de mensagem requer verificação de permissão: "este usuário pode ler este canal?" O Discord usa um sistema de **bitmask RBAC** onde permissões são armazenadas como int64 — cada bit representa uma capability específica. O algoritmo de cálculo tem 8 camadas de precedência: Guild Owner (ALL) → Administrator flag (ALL) → @everyone role → todas as roles do membro (OR) → channel overwrites.

A inovação está no cache: o Discord pré-computa um **effective permission bitmask** por tupla `(user_id, channel_id)`, armazenado com uma chave `role_version`. Quando um administrador altera permissões, um único version bump invalida todas as entradas de cache stale. Operação O(1), não O(users × channels). No caminho quente, a verificação é um hash lookup.

### 3.4 O Sistema de Presença e o Problema N²

O sistema de presença — mostrando quais usuários estão online — enfrenta scaling quadrático. Com 10.000 membros e 1.000 mudanças de status por segundo, o modelo ingênuo geraria 10 milhões de eventos por segundo. O tráfego de presença pode exceder o tráfego de mensagens em ~5×.

**Subscription scoping**: clientes só se inscrevem para presença de usuários visíveis em sua UI atual — reduzindo tráfego em 5×. **Batching**: deltas de presença são agregados em janelas de 5 segundos. **Consistência eventual**: usuários toleram 5-10 segundos de staleness. Segunda-feira 9h (500K logins em 60 segundos) é mitigado com reconnect jitter (30s), rate limiting e degradação para "eventualmente correto em 30s."

### 3.5 Rust SortedSet NIF: 160× Mais Rápido no Pior Caso

A lista de membros online exibida na sidebar requer uma estrutura ordenada com centenas de milhares de entradas e mutações concorrentes frequentes. A implementação pura em Elixir tornou-se um gargalo.

O Discord construiu uma **NIF (Native Implemented Function) em Rust** usando o crate Rustler para integração segura com a BEAM. A estrutura de dados é um Vector of Vectors (similar a skip-list) com bucket size de 500. A busca: linear scan para encontrar o bucket, binary search dentro do bucket.

**Performance**: inserção de 0,4µs (melhor caso) a 3,68µs (pior caso, 1M itens). Melhor caso 6,5× mais rápido que Elixir puro; pior caso **160×** mais rápido. Sort de 30M itens: 21,6s (Elixir) → 4,75s (Rust), speedup de 4,6×.

### 3.6 FastGlobal: 0,33µs Por Lookup Via Constant Pool da BEAM

**FastGlobal** explora uma otimização pouco conhecida da BEAM: constant pools em módulos compilados. Funções que retornam dados estáticos têm esses dados armazenados no heap somente-leitura do módulo, compartilhado entre todos os processos sem cópia. FastGlobal compila módulos em runtime para armazenar dados grandes que precisam ser lidos por múltiplos processos.

| Operação | Tempo médio |
|---|---|
| fastglobal get | 0,33 µs/op |
| ets get | 7,64 µs/op |
| agent get | 12,67 µs/op |

FastGlobal é **20× mais rápido que ETS** e **38× mais rápido que Agent**. O padrão foi tão bem-sucedido que a Ericsson o incorporou como feature nativa no Erlang/OTP 22: o módulo `persistent_term`.

### 3.7 GenStage: 1 Milhão de Push Notifications Por Minuto

O sistema de push notifications do Discord enfrenta bursts massivos. Durante o verão de 2016, servidores de /r/Overwatch e Pokémon GO excediam 25.000 usuários concorrentes, e o sistema de push existente não aguentava.

A solução usa **GenStage** com dois estágios: **Push Collector (Producer)** — coleta requisições de push, bufferiza quando consumidores estão saturados, faz load shedding (descarta notificações antigas) quando o buffer enche; **Pusher (Consumer)** — envia pushes ao Firebase via XMPP, limitado a 100 in-flight requests por conexão. Usa `GenStage.ask()` para demandar apenas o que pode processar. O Producer nunca envia mais do que o Consumer pediu — backpressure natural.

O Producer ignora `handle_demand` — eventos são despachados via `handle_cast` assim que chegam. Se consumidores estão lentos, eventos acumulam no buffer interno do GenStage. José Valim, criador do Elixir, confirmou este padrão híbrido (push-producer + pull-consumer) como abordagem válida. Throughput: 1 milhão de pushes por minuto, depois dobrado.

### 3.8 Armazenamento de Mensagens: MongoDB → Cassandra → ScyllaDB

**MongoDB (2015).** Schema flexível permitia iteração rápida, mas escalabilidade limitada.

**Cassandra (2015-2022).** O cluster `cassandra-messages` cresceu para **177 nós** armazenando trilhões de mensagens. Schema: `PRIMARY KEY ((channel_id, bucket), message_id)` com bucket de 10 dias e IDs Snowflake. Problemas: hot partitions (canais grandes degradavam latência de todo o cluster), compaction atrasada ("gossip dance" manual para compactar nós fora de rotação), GC pauses da JVM causando spikes de latência.

**ScyllaDB (2022-presente).** C++ em vez de Java, eliminando GC. Arquitetura shard-per-core (Seastar framework). O bloqueador técnico — reverse queries lentas — foi resolvido pelo time do ScyllaDB a pedido do Discord.

**Rust Data Service.** Camada middleware entre API e banco com request coalescing: se múltiplos usuários solicitam a mesma linha, apenas uma query é executada. Consistent hash routing por `channel_id` garante que todas as requisições do mesmo canal vão para a mesma instância. Redução de 10-50× em taxa de queries para canais quentes.

**Migração.** Migrador Rust lendo token ranges do Cassandra, checkpointing via SQLite, streaming para ScyllaDB a **3,2 milhões de mensagens por segundo**. Tempo total: **9 dias** (estimativa Spark: 3 meses). Travou a 99,9999% por tombstones não-compactados; compactar o token range resolveu em segundos. Validação: leituras simultâneas em ambos os bancos com comparação de resultados.

| Métrica | Cassandra (177 nós) | ScyllaDB (72 nós) |
|---|---|---|
| p99 read (histórico) | 40-125ms | 15ms |
| p99 write | 5-70ms | 5ms estável |

**Teste de stress: Copa do Mundo 2022.** Nove spikes de mensagens mapeados para eventos da partida. Sistema "não suou."

### 3.9 MaxJourney: 1 Milhão de Usuários Concorrentes em Um Único Servidor

O servidor do Midjourney atingiu 15M+ membros totais e picos de 1M+ de usuários online simultâneos — em um único guild process. O time "MaxJourney" aplicou **observability-first optimization**:

**Stack tracing de produção.** `Process.info(pid, :current_stacktrace)` capturava mil stacktraces com 100ms de intervalo — 2 minutos de amostragem revelando exatamente onde o processo passava tempo.

**Instrumentação do event loop.** Contadores por tipo de mensagem com max/min/avg/total de processamento permitiram filtrar operações <1% do tempo e focar nas caras.

**Memory estimation library.** `erts_debug.size` era lento demais para objetos grandes. O time escreveu uma biblioteca que amostra maps e lists em vez de percorrer cada elemento.

**GC patológico do Manifold offload.** Quando o Manifold offload foi ativado — delegando fanout para processos sender dedicados — a performance piorou. `erlang.trace(pid, true, [:garbage_collection])` revelou GC contínuo disparado pelo **virtual binary heap**: recuperava centenas de KB ao custo de copiar GB de heap. A correção: ajustar `min_bin_vheap_size` via `Process.flag/2` para alguns MB.

**Worker Processes + ETS.** ETS (Erlang Term Storage) compartilha a lista de membros entre processos. Modelo híbrido: conjunto completo em ETS (leitura concorrente segura), mudanças recentes no heap do guild process. Worker processes executam operações caras (todos @everyone, guild hand-off) sem bloquear o guild process.

### 3.10 Voz e Vídeo: SFU C++ e DAVE E2EE Sobre MLS

O sistema de voz do Discord usa um **Selective Forwarding Unit (SFU)** escrito em C++: o servidor retransmite pacotes RTP entre participantes sem decodificar. Modificações ao WebRTC padrão: eliminação de ICE/STUN (tráfego passa pelo servidor, IP protegido), substituição de DTLS+SRTP por Salsa20 (criptografia mais rápida), supressão de silêncio e redução de SDP ao mínimo.

Em setembro de 2024, o Discord lançou o protocolo **DAVE (Discord Audio & Video End-to-End Encryption)**. Usa **MLS (Messaging Layer Security) 1.0** para troca de chaves de grupo: ciphersuite DHKEMP256_AES128GCM_SHA256_P256, cada remetente exporta uma chave simétrica ratcheted per-sender. A criptografia opera na camada **WebRTC Encoded Transform API** — frames são criptografados antes da packetização em RTP. O SFU retransmite pacotes RTP criptografados sem capacidade de decriptação. Forward Secrecy e Post-Compromise Security via MLS: novos membros não decriptam mídia anterior; membros que saem não decriptam mídia futura.

A biblioteca cliente `libdave` é open-source em C++. Auditada pela Trail of Bits, que encontrou issues de memory safety e recomendou migração futura para Rust. Durante a transição, chamadas com participantes sem DAVE fazem downgrade para transporte criptografado; quando o último participante sem suporte sai, a chamada faz upgrade de volta para E2EE.

---

## 4. Design Visual e Identidade: Do Blurple ao Mana

### 4.1 O Sistema de Cores: Blurple e o Gradiente de Cinzas

A cor mais icônica do Discord é o **Blurple** — uma mescla de azul e roxo que passou por duas iterações:

| Versão | Hex | RGB | Contexto |
|--------|-----|-----|----------|
| Original (2015-2021) | `#7289DA` | rgb(114, 137, 218) | Tom mais suave, pastel |
| Atual (2021-presente) | `#5865F2` | rgb(88, 101, 242) | Mais escuro, mais saturado |

A mudança de cor em 2021 foi um dos aspectos mais controversos do rebrand. O novo Blurple foi descrito por usuários como "agressivo para os olhos" e "causador de tensão ocular". A raiz do problema é técnica: `#5865F2` sobre o fundo escuro do chat (`#313338`) tem razão de contraste de aproximadamente 4,6:1 — no limite mínimo do WCAG AA (4,5:1) para texto grande, mas abaixo para texto de corpo. O Blurple anterior (`#7289DA`) tinha melhor legibilidade em telas escuras.

Para daltônicos, o Blurple apresenta desafios específicos: em protanopia (ausência de cones L/vermelho, ~2% dos homens), o componente vermelho do roxo desaparece, fazendo o Blurple parecer azul; em tritanopia (ausência de cones S/azul, <0,01%), o Blurple perde o componente azul e tende ao avermelhado. O Discord não possui modos dedicados para daltonismo — apenas um slider de saturação (0-100%).

O **tema escuro** do Discord, ativo por padrão desde 2015, foi pioneiro. Enquanto a maioria dos aplicativos usava tema claro, o Discord apostou no escuro por alinhamento com seu público gamer — usuários que passam horas em ambientes de pouca luz. A paleta usa um gradiente progressivo de cinzas:

| Elemento | Cor | Função Cognitiva |
|----------|-----|-----------------|
| Barra de servidores | `#1e1f22` | O mais escuro — âncora visual periférica |
| Sidebar de canais | `#2b2d31` | Cinza médio — área de navegação |
| Área de chat | `#313338` | O mais claro — foco primário |
| Texto primário | `#dbdee1` | Branco acinzentado, ~8,5:1 de contraste (AAA) |
| Texto secundário | `#949ba4` | Cinza médio, ~4,5:1 (AA) |

Este gradiente — progressivamente mais claro da esquerda para a direita — cria profundidade sem sombras, guiando o olhar naturalmente para o conteúdo. Nenhum fundo é preto puro: os tons de cinza azulado reduzem o contraste extremo, diminuindo a fadiga ocular em sessões prolongadas. O Discord usa três variáveis CSS de acessibilidade como filtro global: `--saturation-factor`, `--contrast` e `--brightness`, aplicadas via `filter: saturate(...) contrast(...) brightness(...)` — permitindo que o usuário ajuste a intensidade visual sem quebrar o design system.

Desde março de 2025, a reforma desktop expandiu para **quatro temas gratuitos**: Light, Ash (cinza médio), Dark (clássico) e Onyx (preto verdadeiro para telas OLED). Adicionou também **três densidades de UI** (Default, Spacious, Compact) independentes do layout de mensagem.

### 4.2 O Design System Mana e a Arquitetura de Tokens

O sistema de design do Discord chama-se **Mana**, mantido por uma equipe dedicada de engenharia de design systems (salários de $196K-$279K/ano). É cross-platform: React/TypeScript no desktop/web, Swift/Objective-C/React Native no iOS, Kotlin/React Native no Android.

O artigo de engenharia **"Light Theme, Redeemed"** revela a arquitetura de tokens: o Discord criou um sistema onde engenheiros atribuem uma única variável como `--foreground-1` que **auto-seleciona a cor correta** para cada tema. A paleta foi bifurcada em duas (dark e light), seguindo a convenção `--{categoria}-{número}`. Uma única mudança de mapeamento se propaga pelo aplicativo inteiro.

A equipe do Mana construiu ferramentas open-source para o Figma: o **Auto Theme** (plugin que alterna cores entre temas baseado em arquivos de mapeamento), o **Design Lint** (~468 stars, linter que força uso de design tokens na fase de design, sinalizando cores fora do sistema), e o **Inspector** (meta-informação sobre camadas, como DevTools no Figma).

### 4.3 Tipografia: De Whitney a gg sans

A jornada tipográfica do Discord reflete sua evolução de ferramenta gamer para plataforma social:

1. **2015-2021**: **Whitney** (Tobias Frere-Jones) como fonte de interface. **Uni Sans Heavy** para o wordmark em caixa alta.
2. **Maio 2021**: O rebrand introduz **Ginto Discord Nord** para o novo wordmark "Discord" em title-case. Desenhada por Seb McLauchlan e customizada pela foundry Dinamo (Fabian Harb e Renan Rosatti). Descrita como "exuberante, geométrico-humanista" com formas que lembram controles de console.
3. **Dezembro 2022**: **gg sans** substitui Whitney em toda a interface. Fonte proprietária criada internamente. O nome "gg" significa "good game". A recepção foi mista: usuários reclamaram de legibilidade reduzida e solicitaram opção de reverter.

A pesquisa científica sobre tipografia em modo escuro (Palmén, Gilbert & Crossland, Google, CHI 2023) demonstrou que **grade da fonte não melhora legibilidade em polaridade escura** para texto corpo — o aumento de weight é mais eficaz. O Discord usa gg sans com pesos de Normal a Extra Bold, permitindo hierarquia tipográfica sem depender de grade.

### 4.4 Clyde e a Identidade Gamer

O mascote Clyde é uma figura deliberadamente dupla: parece simultaneamente um **controle de video game** e um **rosto sorridente**. Esta ambiguidade visual encapsula o DNA do produto — ferramenta e comunidade, hardware e emoção.

No rebrand de 2021, Clyde foi libertado do balão de fala que o continha desde 2015, tornou-se simétrico para versatilidade em merchandise, ganhou bordas arredondadas (evocando ombros) e múltiplas expressões. A Studio Moth, em 2024, desenvolveu a direção de ilustração "Multiverso": personagens renderizados em 2D, 3D ou Low-Poly convivem harmonicamente, cada estilo representando um tipo diferente de comunidade. A cabeça de Clyde contém um easter egg — uma porta de controle NES.

---

## 5. Arquitetura da Informação e Cognição: A Casa de Cômodos

### 5.1 O Modelo Mental Espacial

Pesquisa acadêmica (Kim et al., arXiv 2501.09951) identificou que usuários do Discord desenvolvem um modelo mental de **"casa com cômodos"**: servidores são casas, categorias são andares, canais são cômodos individuais. Como descreveu um participante: *"Se você está andando pela sua casa e vê sua irmã falando ao telefone no quarto dela, você não vai entrar naquele quarto. Você vai encontrar outro quarto."*

Esta metáfora espacial não é acidental — é a base da arquitetura de produto. A hierarquia `Servidor → Categoria → Canal → Thread` mapeia diretamente para como humanos organizam espaços físicos. Oldenburg (1989) definiu o conceito de **"terceiro lugar"** — espaço distinto da casa (primeiro lugar) e do trabalho (segundo lugar) onde comunidades se formam. O Discord é o terceiro lugar digital: cada servidor é um espaço persistente com dono, regras, cargos e canais temáticos. Você não "consome" um servidor. Você pertence a ele.

A pesquisa identificou 21 elementos de design do Discord alinhados ao conceito de terceiro lugar, incluindo persistência do espaço, apropriação pelo usuário (customização de servidor), e separação entre espaços públicos (canais) e privados (DMs).

### 5.2 Carga Cognitiva e Externalização

O Discord opera como um sistema de **externalização cognitiva** — transfere carga da memória de trabalho do usuário para a interface. A pesquisa de Sweller sobre Teoria da Carga Cognitiva explica o mecanismo:

O layout de 4 colunas fixas (trilha de servidores 72px, canais 240px, chat flexível, membros 240px) funciona como um **mapa cognitivo persistente**. O usuário não reconstrói o estado da interface a cada navegação. As cores progressivas (mais escuro à esquerda, mais claro no centro) criam hierarquia visual sem exigir processamento consciente.

Indicadores de não-lido (bolinhas e números vermelhos) funcionam como **lembretes externos** — offloading cognitivo documentado na literatura de HCI. O contador numérico ao lado do servidor informa exatamente quantas notificações aguardam, eliminando a necessidade de memorizar canais com atividade pendente.

A **segmentação por canal** gerencia ativamente a largura de banda cognitiva. Canais separados por tópico evitam que o usuário processe informações irrelevantes — um princípio conhecido como **controle de interferência proativa** na psicologia cognitiva. A pesquisa mostra que usuários gerenciam tipicamente **2-4 conversas concorrentes**, alternando entre canais.

### 5.3 Paralisia de Decisão e os Mecanismos de Redução

Comunidades bem-sucedidas no Discord seguem a proporção empírica de `canais ≈ membros / 20`. Servidores com muitos canais e poucos membros sofrem do "efeito cemitério" — cada canal parece vazio, desestimulando participação.

Os **Reaction Roles** são o principal mecanismo de redução de paralisia: novos membros auto-atribuem interesses, regiões ou preferências reagindo com emojis a mensagens. A comunidade recomenda **5-10 cargos por mensagem** como ponto ideal — acima disso, a paralisia por excesso de opções (Schwartz, Paradox of Choice) aparece. O feedback é instantâneo (250-400ms), criando um loop de engajamento sem fricção.

Os **Canais de Fórum** (2022) introduziram um paradigma híbrido: em vez de scroll infinito, discussões são organizadas em posts pesquisáveis com tags. Tags funcionam como filtros de atenção — `Bug`, `Resolvido`, `Solicitação de Recurso`. O arquivamento automático de threads inativas evita o acúmulo de "canais mortos". Contudo, comunidades que substituíram todos os canais de texto por fóruns relataram queda no engajamento casual — o formato fórum sacrifica a espontaneidade do chat em nome da organização.

### 5.4 Divulgação Progressiva: O Que o Usuário Não Vê Não Cansa

O Discord aplica divulgação progressiva em quatro camadas:

1. **Onboarding**: o sistema nativo de 2023 unificou verificação, seleção de cargos e canais em um fluxo único. Máximo de 5 perguntas — acima disso, usuários abandonam.
2. **Server Discovery**: navegação por categorias → preview (modo lurker) → aceitação de regras → acesso completo. O atrito (regras) é colocado **depois** que o interesse foi estabelecido.
3. **Canais e cargos**: comunidades usam cargos como filtros — o usuário só vê canais relevantes. Um servidor de 500 canais pode parecer ter 15 para um membro específico.
4. **Configurações avançadas**: permissões RBAC em 3 camadas (servidor → categoria → canal) só são expostas a administradores.

### 5.5 O Sistema de Notificações: Uma Árvore de Decisão Multicamadas

O motor de notificações do Discord avalia cada evento contra uma árvore de decisão que a pesquisa em HCI chama de **gerenciamento de interrupção por breakpoint** (Iqbal & Bailey, 2005-2008):

1. Status DND ativo? → Suprimir tudo
2. Horário silencioso ativo? → Suprimir tudo
3. Usuário ativo no desktop? → Suprimir push mobile (evitando alarme duplicado)
4. Servidor/canal mutado? → Suprimir (mas atenção: mutar NÃO suprime @menções — a "armadilha do mudo" é uma queixa recorrente)
5. Tipo de evento: DM direta (sempre push) > @username (prioritário) > @role (médio) > @everyone (suprimível)
6. Status de leitura: já visualizou? → Agrupar ou descartar

Pesquisa com 163 participantes (Talypova, Lingler & Wintersberger, MUM 2023) confirma que usuários aceitam adiamento de notificações apenas se sentirem **controle** sobre o sistema. O Discord implementa isto com controles granulares: Global > Servidor > Canal, mais horas silenciosas agendadas e supressão cross-device.

---

## 6. UX Mobile e Ergonomia Física

### 6.1 Zonas de Polegar e a Tensão Desktop-vs-Mobile

O estudo de Steven Hoober (2013, 1.333 usuários observados) estabeleceu que **49% dos usuários operam o celular com uma mão**. As três zonas do polegar — natural (terço inferior), estiramento (terço médio), difícil (terço superior e bordas opostas) — definem o que é ergonomicamente acessível.

O layout do Discord mobile após o redesign de 2023-2024 ilustra a tensão de adaptar um produto desktop-first para mobile:

| Elemento | Zona do Polegar | Avaliação |
|----------|----------------|-----------|
| Tab bar inferior (Servidores, Mensagens, Notificações, Você) | Verde — Ótimo | Maior acerto do redesign |
| Lista de servidores (lateral esquerda) | Amarelo-Vermelho | Exige estiramento ou mudança de pegada |
| Botão de membros (canto superior direito) | Vermelho — Péssimo | O elemento mais criticado |
| Campo de busca (topo da tela) | Vermelho — Difícil | Praticamente inalcançável com uma mão |

O redesign de 2023 gerou reação massiva. Usuários avançados reportaram **aumento de 40-60% nos gestos** para tarefas comuns. Um moderador documentou: rotina de verificação em 8 servidores foi de 12 para 31 segundos — aumento de ~158% no tempo. A causa raiz: ações que eram automáticas (tap em local familiar, processadas pelo cerebelo) tornaram-se ações que exigem decisão consciente (long-press, swipe-up drawer, processadas pelo córtex pré-frontal), aumentando significativamente a fadiga cognitiva.

### 6.2 Alvos de Toque e a Lei de Fitts

| Padrão | Tamanho Mínimo | Discord (aproximado) |
|--------|---------------|---------------------|
| Apple HIG | 44×44 pt | Ícones de servidor: ~36-40px — **abaixo** |
| Material Design | 48×48 dp | Tab bar inferior: ~48-56px — **conforme** |
| WCAG 2.5.5 (AAA) | 44×44 CSS px | Botões de ação: ~32-36px — **abaixo** |
| WCAG 2.5.8 (AA) | 24×24 CSS px | Canais: ~40-44px — **conforme** |

Os ícones de servidor, sendo o principal mecanismo de navegação, estão abaixo do recomendado por Apple e Google. A Lei de Fitts estabelece que o tempo para atingir um alvo é proporcional a `log₂(distância/tamanho)`. Ícones menores significam mais erros e mais tempo — particularmente problemático para usuários com tremor ou dificuldades motoras.

### 6.3 O Vocabulário de Gestos e Seus Limites

Pesquisa publicada na Scientific Reports (2022) demonstrou que a precisão de memorização de gestos cai de significativamente alta com 6-9 pares gesto-ação para **52% com 15 pares** e **41% com 22 pares**. O Discord mobile possui pelo menos 6 gestos distintos (tap, long-press, swipe-left para reply, swipe-right para membros, swipe-up, swipe-down), posicionando-se próximo ao limite superior da capacidade de memorização.

As heurísticas de Chuan, Sivaji & Ahmad (2015) para interação gestual estabelecem que gestos são invisíveis por natureza — exigem descoberta ou memorização. A recomendação da indústria é: **sempre fornecer alternativas visíveis**. O swipe-to-reply tem alternativa (long-press), mas o swipe entre servidores não tem equivalente visível, violando WCAG 2.5.1 (Pointer Gestures, Nível A).

### 6.4 Acessibilidade: Entre o Modelo e a Lacuna

O desktop do Discord é amplamente considerado um modelo de acessibilidade para aplicações complexas. A American Foundation for the Blind (AFB, 2023) descreveu o suporte a leitores de tela como "surpreendentemente bom": nomes de usuários são marcados como headings, permitindo navegação rápida entre mensagens; quase todos os elementos são rotulados com ARIA; o foco é gerenciado corretamente ao abrir menus.

Mas o mobile fica atrás. No VoiceOver (iOS), o Action Rotor é reportado como quebrado; o foco salta imprevisivelmente no histórico; alguns elementos não são rotulados; e a barra de rolagem é **invertida** (0% = mensagens mais recentes, o oposto da maioria dos apps). O Discord não possui VPAT público nem relatório de conformidade WCAG.

Em junho de 2024, o Discord foi alvo de uma ação judicial no Distrito Leste de Nova York (1:24-cv-04380) por violações do Americans with Disabilities Act (ADA). O Discord respondeu com contratações na equipe de acessibilidade e a implementação de um **runtime accessibility checker** interno — MutationObserver + requestIdleCallback que detecta problemas de acessibilidade na UI em tempo real.

**Resumo das lacunas de acessibilidade:**

| Área | Status |
|------|--------|
| Leitores de tela (desktop) | NVDA/JAWS — bom. Headings corretos, foco gerenciado. |
| Leitores de tela (mobile) | VoiceOver — problemático. Action Rotor quebrado. Foco imprevisível. |
| Contraste de cores | Role colors frequentemente falham WCAG AA. Sem modo daltônico dedicado. |
| Legendas em voz | Sem legendagem automática nativa para chat de voz — depende de bots. |
| Alvos de toque | Abaixo de 44pt/48dp em ícones de servidor e botões secundários. |
| Gestos | Sempre exige alternativa visível (WCAG 2.5.1). Swipe entre servidores sem fallback. |
| Redução de movimento | Toggle "Reduced Motion" disponível. Desabilita animações e GIFs automáticos. |

---

## 7. UI Components e Padrões de Interação

### 7.1 Componentes V2: A Reforma do Sistema de Mensagens (2025)

Em 2025, o Discord lançou **Components V2** (também chamados "display components"), substituindo o modelo antigo `content` + `embeds` + `attachments` por uma árvore de componentes composicional. Quando a flag `IS_COMPONENTS_V2` está ativa (bit `1 << 15`), mensagens são construídas como árvores de componentes com até 40 nós e 4.000 caracteres totais:

```
Message (flags: IS_COMPONENTS_V2)
  └── Container (cor de acento, spoiler opcional)
      ├── Text Display (Markdown)
      ├── Action Row → Button
      └── Separator
  └── Section
      ├── Text Display
      └── Accessory: Thumbnail
  └── Media Gallery (até 10 itens)
```

Os tipos de componentes incluem: Action Row (container para 1-5 botões), Button (Primary/Secondary/Success/Danger/Link/Premium), Select Menus (string, usuário, cargo, canal), Section (1-3 Text Displays + acessório), Text Display (Markdown), Thumbnail, Media Gallery, File, Separator, Container, e Radio/Checkbox Groups para modais.

### 7.2 A Pilha Frontend: React, Flux e Webpack

O frontend do Discord compartilha **~95% do código entre web e desktop** e **~98% das Flux stores/ações entre plataformas**. A arquitetura:

- **React** com um fork interno fortemente modificado do padrão **Flux** (Meta). Flux Stores mantêm estado; Flux Dispatcher (`discord/Dispatcher`) é o barramento central de eventos.
- **Webpack** para code splitting: apenas JS, CSS, fontes e traduções mínimos são enviados na inicialização. Assets restantes são lazy-loaded.
- **react-focus-rings**: biblioteca open-source do Discord para indicadores de foco centralizados. Garante navegação completa por teclado.
- **@discord/react-native-bottom-sheet**: biblioteca open-source de bottom sheet para React Native, com snapping, integração com React Navigation e suporte Expo.
- **Overlapping Panels**: biblioteca open-source Android para navegação por gestos entre painéis sobrepostos.

### 7.3 Virtualização de Chat: O Problema do Scroll Infinito Bidirecional

A lista de mensagens é virtualizada — apenas mensagens visíveis são montadas no DOM. Mas o scroll infinito **bidirecional** (carregar mensagens antigas ao scrollar para cima E novas ao scrollar para baixo) apresenta desafios específicos:

1. **Preservação de posição no prepend**: quando mensagens antigas chegam, a posição de scroll não pode saltar. O Discord tira snapshot da posição antes do fetch e restaura após renderização.
2. **Alturas dinâmicas**: imagens e embeds carregam assincronamente, alterando altura. `ResizeObserver` ajusta após cada carregamento.
3. **Virtualização com altura variável**: ao contrário de listas de altura fixa (tweets), mensagens do Discord variam dramaticamente. Implementações usam estimativas de altura e recalibração.

No Android, o Discord construiu o **FastestList** — RecyclerView nativo customizado — que eliminou telas em branco em dispositivos de baixo custo, reduziu slow frames em 60% e cortou uso de memória em 14%.

### 7.4 Micro-Interações e Performance Percebida

O Discord investe em performance percebida tanto quanto em performance real. Os três limiares de Nielsen guiam o design:

| Limiar | Percepção Humana | Aplicação no Discord |
|--------|-----------------|---------------------|
| 100ms | Instantâneo | Transições de hover, clique em botão, toggle de mute |
| 1s | Fluxo contínuo | Envio de mensagem (optimistic UI), troca de canal, abertura de modal |
| 10s | Limite de atenção | Login, carregamento inicial, busca em histórico |

O princípio do **Optimistic UI** é aplicado agressivamente: a mensagem aparece na tela do remetente antes da confirmação do servidor. Se o servidor rejeitar (rate limit, permissão), a UI reverte com indicador de erro. Esta técnica reduz a latência percebida em até 40% comparado a esperar confirmação antes de atualizar a UI.

As durações de animação seguem pesquisa cognitiva: 70ms para acionamento de botões (abaixo do limiar de percepção de demora), 150ms para hover e pequenas transições, 240ms para comunicação do sistema (notificações, transições de tela), 400ms para aquisições maiores.

O ciclo de feedback de performance (Observar → Analisar → Otimizar → Repetir) é central na cultura de engenharia. A migração de emojis animados de GIF para WebP manteve 60fps em dispositivos low-end. A compressão Zstandard reduziu tráfego WebSocket em 40%, beneficiando conexões móveis lentas. A reescrita do Read States em Rust reduziu latência de milissegundos para microssegundos com -30% de CPU.

### 7.5 O Custo UX do Electron

O Discord desktop roda em Electron (Chromium + Node.js). A decisão permitiu compartilhar 95% do código com a versão web e lançar rapidamente em Windows, macOS e Linux. O custo UX é documentado: consumo de **1-4 GB de RAM** em repouso, descrito por usuários como "um monstro lento e inflado".

O mecanismo de **auto-restart a 4 GB** de consumo de memória é uma admissão implícita do problema. Comparado a apps nativos — Telegram (~150 MB), Slack (~800 MB) — o Discord é o mais pesado. A equipe tem mitigado com otimizações progressivas (zstd compression, Rust rewrites, virtualização), mas a arquitetura Electron impõe um piso de consumo de recursos que nenhuma otimização elimina completamente.

---

## 8. Inovações

**8.1 Manifold: fanout O(relays) em vez de O(members).** A decomposição do fanout em três estágios — agrupamento por nó, consistent hashing, workers paralelos — reduz o trabalho do guild process de linear no número de membros para linear no número de relays. É uma inovação arquitetural que nenhum sistema de chat anterior implementou.

**8.2 FastGlobal: abuso produtivo de constant pools da BEAM.** Compilar módulos em runtime para armazenar dados no heap somente-leitura compartilhado não era o uso pretendido de constant pools. O Discord descobriu que funcionava — e era 20× mais rápido que ETS.

**8.3 role_version: invalidação de cache O(1) para permissões.** Em vez de invalidar entradas de cache individualmente quando permissões mudam, um único version bump invalida todas. É um padrão que deveria ser adotado em qualquer sistema com cache de permissões.

**8.4 DAVE: E2EE sobre SFU.** Adicionar criptografia ponta-a-ponta a uma arquitetura de servidor de mídia que não foi desenhada para isso, mantendo latência de voz <50ms, via MLS e WebRTC Encoded Transform — é um feito criptográfico e de engenharia de sistemas.

---

## 9. Críticas

**9.1 Centralização extrema.** O Discord é completamente centralizado — sem federação, sem self-hosting. Um servidor Discord não é propriedade do criador; é um espaço alugado na plataforma de uma empresa privada.

**9.2 Threading não-nativo.** O sistema de Threads (2021) é uma feature adicionada, não o modelo padrão. Em servidores grandes, informação se perde no fluxo contínuo. Slack, com threading como padrão, é superior para produtividade.

**9.3 Custo cognitivo do Electron.** O cliente desktop consome 1-4 GB de RAM em repouso. O mecanismo de auto-restart a 4 GB é uma confissão arquitetural. Comparado a apps nativos como Telegram (~150 MB), o Discord é ordens de magnitude mais pesado, e cada atualização do Chromium embarcado força o download de uma stack completa de navegador.

**9.4 O redesign mobile de 2023 e a quebra de muscle memory.** A reforma mobile aumentou em 40-60% os gestos para tarefas comuns e em 2,7 segundos o tempo por ação. A transição de ações automáticas (cerebelo) para ações conscientes (córtex pré-frontal) gera fadiga cognitiva real. O Discord reverteu parcialmente em maio de 2024, mas o episódio expôs a tensão entre onboard de novos usuários e eficiência de usuários avançados — e a falta de um modo opcional para experts.

**9.5 Acessibilidade reativa, não proativa.** O Discord não tem VPAT público. Foi processado em 2024 por violações do ADA. As cores de cargo, customizáveis livremente por admins, falham WCAG AA na maioria das combinações. Não há modo daltônico dedicado — apenas um slider de saturação. O chat de voz não possui legendagem automática nativa. O Action Rotor do VoiceOver é reportado como quebrado há anos.

**9.6 A reforma desktop de 2025 e o desperdício de espaço.** A atualização de fevereiro de 2025 aumentou a espessura da barra de título e da caixa de texto, isolou o uploader de imagem em linha separada, sobrepôs o painel de usuário sobre a lista de servidores (escondendo notificações), reduziu ícones e indicadores de digitação. Usuários descreveram como "um desastre de design" e "um caso clássico de consertar o que não estava quebrado".

**9.7 Blurple em fundo escuro.** O Blurple `#5865F2` sobre o fundo `#313338` do chat mal atinge 4,6:1 de contraste — abaixo do ideal para texto de corpo, e problemático para daltônicos (especialmente tritanopia, onde o Blurple se transforma em verde-azulado). A cor da marca não funciona bem no próprio produto.

**9.8 Fóruns vs. chat: o trade-off entre organização e espontaneidade.** Canais de fórum resolvem o problema de descoberta de informação, mas comunidades que os adotaram como substitutos de canais de texto relatam queda no engajamento casual. A indexação é limitada ao ecossistema Discord — Google não indexa threads de fórum, tornando soluções invisíveis para busca externa.

---

## 10. Lições

**10.1 Um processo por servidor é a abstração correta — até não ser.** O guild process único oferece isolamento de falhas e simplicidade de programação imbatíveis. Mas quando um servidor atinge 1M de usuários, o processo torna-se um ponto de contenção. A resposta não foi shardear — foi offloadar: relays, worker processes com ETS, Manifold para delegar send. Você não precisa abandonar o modelo quando ele atinge limites; precisa identificar qual trabalho pode ser movido para fora sem quebrar a semântica.

**10.2 Coalescimento de requisições resolve hot partitions melhor que trocar de banco.** Migrar de Cassandra para ScyllaDB reduziu p99 read de 125ms para 15ms. Mas o problema fundamental — 100.000 usuários consultando a mesma mensagem — foi resolvido pelo Rust Data Service com request coalescing. Hot partitions são um problema de padrão de acesso, não de tecnologia de banco.

**10.3 Rewrites parciais em Rust como NIFs são o sweet spot.** O Discord não reescreveu o backend em Rust. Reescreveu em Rust apenas os hot paths: SortedSet para listas, Data Service para coalescimento, migrador de dados. Essa granularidade maximiza ROI de engenharia.

**10.4 O GC da BEAM é rápido — exceto quando não é.** `min_bin_vheap_size` causando GC major para recuperar centenas de KB ao custo de copiar GB de heap é o tipo de interação emergente que nenhum benchmark sintético revela. A única forma de encontrar: `erlang.trace` em produção com dados reais.

**10.5 A metáfora espacial é o superpoder cognitivo do produto.** O modelo "casa com cômodos" (servidor → categoria → canal) não é um detalhe de UX — é o que permite que usuários naveguem intuitivamente em servidores com centenas de canais. A arquitetura de informação e a arquitetura de software são a mesma coisa: cada servidor é um processo isolado, e cada canal é um tópico dentro dele. O alinhamento entre modelo mental do usuário e arquitetura do sistema é o que permite a 5 engenheiros operarem infra para 150M de usuários.

**10.6 Tema escuro como default foi uma aposta contracultural vencedora.** Em 2015, o padrão da indústria era tema claro. O Discord escolheu escuro por alinhamento com gamers — e acertou. Pesquisa subsequente (Palmén et al., CHI 2023) demonstrou que usuários preferem modo escuro mesmo quando leem mais devagar nele. A decisão influenciou uma geração de aplicativos.

**10.7 Design de notificações é modelagem de atenção, não modelagem de dados.** O sistema de notificações do Discord não é um banco de regras — é uma árvore de decisão que espelha como a atenção humana funciona. A supressão cross-device (desktop ativo → push mobile suprimido), os breakpoints de interrupção (DM vs @role vs @everyone), e as horas silenciosas agendadas reconhecem que o problema não é técnico — é cognitivo.

**10.8 Optimistic UI é a ferramenta mais subestimada de percepção de performance.** Mensagens aparecem antes da confirmação do servidor. Erros revertem com indicador visual. Esta técnica reduz latência percebida em até 40% sem mudar uma linha de backend. A migração para Components V2 (2025) mostra que o Discord continua investindo em flexibilidade composicional de UI mesmo depois de 10 anos.

---

## 11. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Discord |
| **Fundação** | 2012 (Hammer & Chisel). Lançamento: 13 de maio de 2015 |
| **Fundadores** | Jason Citron (CEO), Stanislav Vishnevskiy (CTO) |
| **Categoria** | Comunicação em Tempo Real / Plataforma Social |
| **MAUs** | 150 milhões+ |
| **Mensagens/dia** | 4 bilhões |
| **Eventos WebSocket/s** | 26 milhões (pico) |
| **Equipe de chat infra** | 5 engenheiros |
| **Mensageria** | Elixir/Erlang (BEAM): guild processes, session processes, relays, supervision trees |
| **Fanout** | Manifold (3 estágios, open-source), passive sessions (90% redução), relays (15K sessões cada) |
| **Storage** | ScyllaDB (72 nós). Rust Data Service: gRPC, request coalescing (10-50×), consistent hash |
| **Cache** | FastGlobal (0,33µs lookup, 20× mais rápido que ETS). Tornou-se persistent_term no OTP 22 |
| **NIFs** | Rust SortedSet: 0,4µs insert, 160× pior caso. Rustler crate |
| **Push** | GenStage: 1M+/min, backpressure, load shedding |
| **Permissões** | Bitmask RBAC, role_version (O(1) invalidação), 8-layer algorithm |
| **Presença** | Subscription scoping (5× redução), batching 5s, consistência eventual |
| **Voz/Vídeo** | SFU C++ customizado. DAVE E2EE (MLS 1.0, WebRTC Encoded Transform, libdave C++) |
| **Mobile** | React Native (iOS desde 2015, Android desde 2022 com Hermes) |
| **Design System** | Mana: React, TypeScript, React Native, Swift, Kotlin. Auto Theme, Design Lint (open-source) |
| **Cor primária** | Blurple `#5865F2` (desde 2021). Original: `#7289DA` |
| **Fonte** | gg sans (2022, proprietária). Anterior: Whitney (2015-2022). Brand: Ginto Discord Nord |
| **Tema** | Dark por default (4 temas gratuitos desde 2025: Light, Ash, Dark, Onyx). 3 densidades de UI |
| **Acessibilidade** | Sem VPAT público. Em litígio (EDNY 1:24-cv-04380). Runtime accessibility checker interno |
| **Concorrentes** | Telegram, Slack, Guilded (Roblox), Microsoft Teams |

---

## 12. Linha do Tempo

```
2012 — Hammer & Chisel fundada. Foco: jogos para tablet.
2014 — Fates Forever fracassa. Pivot para ferramenta de comunicação.
2015 Mai 13 — Discord lançado. MongoDB para mensagens. Tema escuro como padrão. Logo Clyde no balão.
2015 — Migração de MongoDB para Cassandra (12 nós, bilhões de mensagens).
2016 — GenStage implementado para push notifications (1M+/min).
2017 — Manifold open-source. FastGlobal. Rust SortedSet NIF. 5M CCU.
2020 — Todos os bancos exceto mensagens migrados para ScyllaDB. Slogan: "Chat for Communities and Friends".
2021 Mai — Primeiro grande rebrand: novo Blurple #5865F2, Clyde libertado do balão, Ginto Discord Nord, "Imagine a Place".
2022 Mai — Cassandra → ScyllaDB: 177→72 nós, migrador Rust 3,2M msg/s, 9 dias.
2022 Ago — App Android reescrito em React Native para unificar com iOS.
2022 Set — Canais de Fórum lançados.
2022 Dez — Teste de stress: Copa do Mundo. gg sans substitui Whitney.
2023 — MaxJourney: Midjourney 1M+ CCU. Sistema de Onboarding nativo unificado. Redesign mobile (reação negativa).
2024 — Streaming Zstandard compression (40% redução tráfego WebSocket). App Directory. Quests/Orbs.
2024 Set — DAVE protocol: E2EE via MLS 1.0. libdave open-source. Trail of Bits audit.
2024 Mai — Discord reverte parcialmente redesign mobile (Mensagens voltam ao topo).
2024 — Ação judicial ADA (EDNY). Runtime accessibility checker. Slider de contraste adicionado.
2025 Fev — Reforma desktop controversa: overlay com widgets móveis, 4 temas gratuitos, 3 densidades de UI.
2025 — Components V2 (display components) lançados. 150M+ MAUs. 4B mensagens/dia.
```

---

## 13. Fontes

### Engenharia e Arquitetura
- [Discord Engineering Blog — How Discord Stores Trillions of Messages (Bo Ingram, Mar 2023)](https://discord.com/blog/how-discord-stores-trillions-of-messages)
- [Discord Engineering Blog — Maxjourney: Pushing Discord's Limits with a Million+ Online Users in a Single Server (Yuliy Pisetsky, Out 2023)](https://discord.com/blog/maxjourney-pushing-discords-limits-with-a-million-plus-online-users-in-a-single-server)
- [Discord Engineering Blog — How Discord Scaled Elixir to 5,000,000 Concurrent Users (2017)](https://blog.discord.com/scaling-elixir-f9b8e1e7c29b)
- [Discord Engineering Blog — How Discord Handles Push Request Bursts of Over a Million Per Minute with Elixir's GenStage (2016)](https://discord.com/blog/how-discord-handles-push-request-bursts)
- [Elixir Lang Blog — Real-time communication at scale with Elixir at Discord (2020)](https://elixir-lang.org/blog/2020/10/08/real-time-communication-at-scale-with-elixir-at-discord/)
- [InfoQ — Discord Scales to 1 Million+ Online MidJourney Users in a Single Server (Jan 2024)](https://www.infoq.com/news/2024/01/discord-midjourney-performance/)
- [InfoQ — Interfacing Elixir with Rust to Improve Performance: Discord's Story](https://www.infoq.com/news/2019/07/rust-elixir-performance-at-scale/)
- [ScyllaDB Tech Talk — How Discord Migrated Trillions of Messages from Cassandra to ScyllaDB](https://www.scylladb.com/tech-talk/how-discord-migrated-trillions-of-messages-from-cassandra-to-scylladb/)
- [GitHub — discord/manifold](https://github.com/discord/manifold)
- [GitHub — discord/fastglobal](https://github.com/discord/fastglobal)
- [GitHub — discord/sorted_set_nif](https://github.com/discord/sorted_set_nif)
- [GitHub — discord/dave-protocol](https://github.com/discord/dave-protocol)
- [DAVE Protocol Whitepaper](https://daveprotocol.com/protocol.md)
- [Hacker News — Maxjourney: Pushing Discord's Limits](https://news.ycombinator.com/item?id=38014682)

### Design Visual, Design System e Tipografia
- [Discord Engineering Blog — Light Theme, Redeemed](https://discord.com/blog/light-theme-redeemed)
- [Discord Engineering Blog — Building Open-Source Design Tools to Improve Discord's Design Workflow](https://discord.com/blog/building-open-source-design-tools-to-improve-discords-design-workflow)
- [GitHub — destefanis/auto-theme (Figma plugin)](https://github.com/destefanis/auto-theme)
- [GitHub — destefanis/design-lint (Figma linter)](https://github.com/destefanis/design-lint)
- [Envato Tuts+ — What Fonts Does Discord Use?](https://design.tutsplus.com/articles/what-fonts-does-discord-use--cms-108779)
- [ABCDinamo — Discord font custom work](https://abcdinamo.com/custom/discord)
- [Studio Moth — Discord visual direction](https://moth.studio/projects/discord)
- [BetterDiscord Documentation — Theme Environment & CSS Variables](https://docs.betterdiscord.app/themes/introduction/environment.html)
- [Discord Brand Guidelines (RelayTo)](https://relayto.com/discord)

### UX, Cognição e Arquitetura da Informação
- [Kim et al. (2025) — Discord's Design Encourages Third Place (arXiv 2501.09951)](https://export.arxiv.org/pdf/2501.09951)
- [Smashing Magazine — The Thumb Zone: Designing For Mobile Users (Steven Hoober)](https://www.smashingmagazine.com/2016/09/the-thumb-zone-designing-for-mobile-users/)
- [Remio.ai — Discord Redesign Backlash Reveals Mobile App Tradeoffs (2024)](https://www.remio.ai/post/discord-redesign-backlash-reveals-mobile-app-tradeoffs)
- [Enigmacamp — UI/UX Case Study: Revamping Discord Mobile Flow](https://enigmacamp.com/blog/discord-revamp)
- [TRTC.io — Discord Push Notifications Architecture](https://trtc.io/blog/discord-push-notifications)
- [Iqbal & Bailey — Breakpoints Research (CHI 2005-2008)](https://dl.acm.org/doi/10.1145/1054972.1055016)
- [Talypova, Lingler & Wintersberger (MUM 2023) — Attention Management Systems Vignette Study](https://dl.acm.org/doi/10.1145/3626705.3627767)
- [PanKUN Blog — Designing Discord Server Operations & IA](https://pankun.github.io/)
- [Page Flows — Discord Onboarding User Flow Recordings](https://pageflows.com/)
- [Academic thesis (NTNU) — Domestication theory applied to Discord](https://ntnuopen.ntnu.no/ntnu-xmlui/handle/11250/2784578)

### Acessibilidade
- [American Foundation for the Blind — Discord Accessibility Review (2023)](https://afb.org/aw/23/12/18148)
- [CSUSM — Creating an Accessible Discord Server](https://www.csusm.edu/iits/services/accessibility/guides/socialmedia/discord.html)
- [SIBR — Improving Accessibility: Contrast in Discord Team Roles](https://sibr.dev/papers/files/Improving_Accessibility__Contrast_in_Discord_Team_Roles_1.2.pdf)
- [CourtListener — ADA Case 1:24-cv-04380 (EDNY)](https://storage.courtlistener.com/recap/gov.uscourts.nyed.438000/gov.uscourts.nyed.438000.1.0_1.pdf)
- [GitNation — Brandon Dail, "Accessibility at Discord"](https://gitnation.com/contents/accessibility-at-discord)
- [W3C — Guidance on Applying WCAG 2.2 to Mobile](https://www.w3.org/TR/wcag2mobile-22/)
- [W3C — Mobile Accessibility Techniques M27](https://w3c.github.io/Mobile-A11y-TF-Note/Techniques/M027)

### Performance e UI Técnica
- [Mobile Vitals — Supercharging Discord Mobile: Journey to a Faster App](https://mobile-vitals.com/article/1482-discord-supercharging-discord-mobile-our-journey-to-a-faster-app)
- [Discord Engineering Blog — How Discord Reduced Outgoing Traffic by 40% with zstd](https://discord.com/blog/how-discord-reduced-websocket-traffic-by-40-percent)
- [React Native Radio (RNR 343) — Discord's Journey to React Native with Chas Jhin](https://infinite.red/react-native-radio/rnr-343-discords-journey-to-react-native-with-chas-jhin)
- [GitHub — discord/focus-rings](https://github.com/discord/focus-rings)
- [GitHub — discord/react-native-bottom-sheet](https://github.com/discord/react-native-bottom-sheet)
- [Discord.Net Docs — Components V2](https://docs.discordnet.dev/guides/components_v2/intro.html)

### Ciência da Cor e Percepção Visual
- [Palmén, Gilbert & Crossland (CHI 2023) — How Bold Can We Be? Font Grade Impact on Readability in Light/Dark](https://dl.acm.org/doi/fullHtml/10.1145/3544548.3581552)
- [APCA Readability Criterion (WCAG 3 Candidate) — Contrast for Self-Illuminated Displays](https://git.apcacontrast.com/documentation/APCA_in_a_Nutshell.html)
- [Brettel-Viénot-Mollon Algorithm (1997) — Color Blindness Simulation](https://github.com/dmccreary/automating-instructional-design/blob/main/docs/sims/color-blindness-simulator/index.md)
- [W3C CSSWG Issue #5169 — Add blurple as an official CSS color](https://github.com/w3c/csswg-drafts/issues/5169)

### Gestos, Ergonomia e Mobile
- [PMC — Memory Load and Touchscreen Gesture Learning (2022)](https://pmc.ncbi.nlm.nih.gov/articles/PMC9233678/)
- [OsloMet — Khadka (2021) Navigation Study: Bottom Tabs vs. Hamburger](https://oda.oslomet.no/oda-xmlui/bitstream/handle/11250/2777325/khadka-mauu2021.pdf)
- [ACM — Geven, Sefelin & Tscheligi (2006) Navigation on Mobile Devices](https://dl.acm.org/doi/epdf/10.1145/1152215.1152248)
- [Springer — Chuan, Sivaji & Ahmad (2015) Usability Heuristics for Gestural Interaction](https://link.springer.com/chapter/10.1007/978-3-319-20886-2_14)
