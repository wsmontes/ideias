# Estudo de Caso 19 — Discord: O Produto Que Começou Como Chat de Jogo e Virou a Infraestrutura Social da Internet

> **Data:** 2026-07-03
> **Loop:** 19 de ∞ (Reescrita — Fase 2)
> **Categoria:** Comunicação / Comunidade / Plataforma Social
> **Tema:** 2015. Jason Citron, que havia vendido sua plataforma de jogos sociais OpenFeint por US$ 104 milhões, está queimando o dinheiro da venda num estúdio de games chamado Hammer & Chisel. O primeiro jogo do estúdio, Fates Forever — uma tentativa de criar o primeiro MOBA para tablets — fracassa comercialmente. Mas durante o desenvolvimento, Citron e seu cofundador Stanislav Vishnevskiy percebem algo estranho: a melhor parte do jogo não era o jogo. Era o sistema de chat e voz que eles construíram para os testadores se comunicarem. Enquanto jogavam Final Fantasy XIV e League of Legends, os dois usavam Skype (que consumia CPU e caía), TeamSpeak (que exigia servidor próprio) e IRC (que era só texto). "Todas as ferramentas de voz para gamers são horríveis", concluíram. Citron demitiu um terço da equipe, abandonou o desenvolvimento de jogos, e passou seis meses reconstruindo a empresa em torno de uma única ideia: um aplicativo de comunicação por voz e texto que simplesmente funcionasse. Discord foi lançado em 13 de maio de 2015. No dia do lançamento, um desconhecido postou um link do Discord num subreddit de Final Fantasy XIV dizendo "tem um lugar para conversar sobre a nova expansão". Citron e Vishnevskiy entraram na conversa pessoalmente. Em poucas semanas, subreddits de Diablo, World of Warcraft e League of Legends estavam trocando seus links de IRC por links do Discord. Não houve campanha de marketing. A comunidade de jogos simplesmente adotou o produto porque ele resolvia um problema que todo gamer conhecia intimamente. Em 2021, a Microsoft teria oferecido US$ 12 bilhões pela empresa. Citron recusou. Hoje, o Discord tem 150 milhões de usuários ativos mensais e processa 4 bilhões de mensagens por dia, com uma equipe de infraestrutura de chat de apenas cinco engenheiros.

---

## 0. A Linhagem: Como Gamers Se Comunicavam Antes de Existir um Lugar Que Simplesmente Funcionasse

```
Jogos em LAN party (1990s): você gritava para o amigo do lado. Latência zero. Escala: 4 pessoas.
      ↓
IRC (1988): canais de texto. Persistente. Mas sem voz, sem busca, sem histórico offline.
      ↓
TeamSpeak (2001), Ventrilo (2002), Mumble (2005): voz de baixa latência. Exigiam servidor próprio.
      ↓
Skype (2003): voz + texto. Fácil de usar. Mas consumia CPU, caía, e vazava IP (DDoS).
      ↓
Discord (2015): voz + texto + persistência + servidores gratuitos + zero configuração.
      ↓
Discord hoje (2026): 150M MAUs. 4B mensagens/dia. Servidores, canais, bots, Nitro, streaming.
```

Nenhuma tecnologia na coluna da esquerda era nova em 2015. Voz sobre IP existia desde os anos 1990. WebRTC era um padrão aberto desde 2011. O que o Discord fez foi empacotar voz, texto, canais, permissões e persistência em uma experiência integrada que exigia zero configuração — sem alugar servidor, sem abrir portas no roteador, sem compartilhar endereço IP. Para o usuário, era como entrar numa sala. Para o operador, era invisível.

---

## 1. A Origem: Um Jogo Que Fracassou, Uma Ferramenta Que Sobreviveu

O caminho de Jason Citron até o Discord não foi linear. Em 2008, ele lançou Aurora Feint, um jogo para iPhone que não vendeu bem, mas cujos recursos sociais — chat, leaderboards, perfis — formaram uma comunidade engajada. Citron extraiu esses recursos e os transformou no OpenFeint, uma plataforma que permitia a qualquer desenvolvedor de jogos mobile adicionar recursos sociais com algumas linhas de código. Em seu pico, o OpenFeint era usado por aproximadamente 30% dos jogos da App Store. Em 2011, Citron vendeu a empresa para a japonesa GREE por US$ 104 milhões.

Com o dinheiro da venda, fundou a Hammer & Chisel em 2012. A tese era ambiciosa: construir jogos para tablets que rivalizassem com a profundidade dos jogos de PC. O primeiro título, Fates Forever, era um MOBA — League of Legends para iPad. Críticos gostaram. Jogadores não apareceram. O jogo foi um fracasso comercial.

Durante o desenvolvimento, porém, a equipe havia construído um sistema interno de chat e voz para coordenar testes. Citron e Vishnevskiy, que jogavam Final Fantasy XIV e League of Legends juntos, perceberam que usavam essa ferramenta interna muito mais do que qualquer software de comunicação disponível no mercado. "A melhor coisa sobre Fates Forever era o chat", Citron diria depois.

A decisão de pivotar foi dolorosa. Citron demitiu cerca de um terço da equipe e passou seis meses reestruturando a empresa. O produto resultante foi lançado em 13 de maio de 2015 sob o nome Discord — uma palavra que "soa legal, tem a ver com conversa, é fácil de falar e soletrar, e estava disponível para registro de marca". A piada interna era que o produto existia para resolver a "discórdia na comunidade de games".

No dia do lançamento, um usuário anônimo postou um link do Discord num subreddit de Final Fantasy XIV. Citron e Vishnevskiy entraram na conversa e passaram horas falando com os novos usuários. O padrão se repetiu: alguém postava um link do Discord num subreddit de Diablo, de World of Warcraft, de League of Legends. A comunidade de cada jogo substituía seu servidor IRC por um servidor Discord. Não havia orçamento de marketing. Havia um produto que resolvia um problema real para um público que já estava organizado em comunidades online.

O financiamento inicial veio do incubador YouWeb (US$ 1,2 milhão), complementado por rodadas da Benchmark Capital e da Tencent. Em 2016, o Discord tinha 11 milhões de usuários registrados. Em 2017, 45 milhões. Em 2018, 130 milhões. A pandemia de 2020 acelerou a adoção para além dos games: escolas, clubes do livro, grupos de estudo e comunidades de criptomoedas migraram para a plataforma. Em 2021, a Microsoft teria oferecido US$ 12 bilhões. Citron recusou. A empresa continua privada, com rumores de IPO em 2025.

---

## 2. A Filosofia do Produto: O "Terceiro Lugar" Digital

Em 1989, o sociólogo Ray Oldenburg publicou *The Great Good Place*, um livro que definia o conceito de "third place" — um espaço social que não é a casa (primeiro lugar) nem o trabalho (segundo lugar), mas um local onde as pessoas se reúnem voluntariamente, de forma informal, para conversar. Oldenburg identificou oito características que definem um third place: terreno neutro, nivelador social, conversa como atividade principal, acessibilidade, presença de frequentadores regulares, perfil baixo, atmosfera lúdica, e a sensação de estar "em casa fora de casa".

Em janeiro de 2025, pesquisadores da Universidade de Washington publicaram um estudo no arXiv analisando se o Discord funcionava como um third place digital. Após 25 entrevistas em profundidade com usuários que formaram amizades na plataforma, a resposta foi afirmativa. Os pesquisadores identificaram 21 elementos de design do Discord alinhados com as oito características de Oldenburg e os agruparam em quatro princípios:

1. **Espaços temáticos para interações repetidas** — servidores e canais persistentes que funcionam como lugares, não como feeds transitórios. Um canal de voz que você frequenta toda noite é um lugar. Um feed algorítmico que muda a cada refresh não é.
2. **Autonomia e customização do usuário** — estrutura de servidor definida pelo criador, cargos com permissões granulares, bots que automatizam moderação e entretenimento. Cada servidor é um espaço com regras próprias, construído por seus membros.
3. **Atividades mutuamente envolventes** — integração com jogos (Rich Presence mostra o que você está jogando), streaming de tela, bots de música, Activities (mini-jogos embutidos). As pessoas não estão apenas conversando; estão fazendo coisas juntas.
4. **Interação casual de baixa pressão** — o fluxo de chat é efêmero (rola para cima e some), não há algoritmo de engajamento empurrando conteúdo, não há contagem de seguidores, não há feed de notícias. O perfil de usuário é contextual: diferente em cada servidor.

Os entrevistados descreveram servidores do Discord como "praça da cidade", "café", "sala de jogos" e "uma festa que nunca acaba". Essas metáforas não são acidentais. Elas revelam que o Discord ocupa um espaço conceitual diferente das redes sociais tradicionais. O Facebook é um lugar onde você vai para ver o que seus amigos estão fazendo. O Twitter é um lugar onde você vai para ver o que está acontecendo. O Discord é um lugar onde você vai para **estar com pessoas**.

Essa distinção tem implicações de produto profundas. Em uma rede social tradicional, a unidade atômica é o post — uma unidade de conteúdo que compete por atenção num feed. No Discord, a unidade atômica é o servidor — um espaço persistente que acumula história, normas, piadas internas e relacionamentos. Você não "consome" um servidor. Você pertence a ele.

Em 2024, o Discord tentou expandir essa visão para públicos além dos games, posicionando-se como "third place para todos". A tentativa foi recebida com ceticismo pelo mercado e resultou em 170 demissões. A empresa recuou para seu núcleo: "o melhor lugar para conversar e passar tempo antes, durante e depois de um jogo". O recuo não foi uma derrota estratégica; foi o reconhecimento de que o terceiro lugar do Discord só funciona porque o "jogo" não é apenas uma atividade — é o contexto compartilhado que torna a conversa significativa.

---

## 3. Arquitetura Técnica: O Motor Que Roda em Elixir e Rust

O Discord processa 4 bilhões de mensagens por dia e mantém 12 milhões de usuários simultâneos, com uma equipe de infraestrutura de chat de cinco engenheiros. Essa eficiência é produto de escolhas arquiteturais feitas no início da empresa e iteradas ao longo de uma década.

### 3.1 O Modelo de Atores em Elixir/Erlang

O backend de mensageria do Discord é construído sobre a máquina virtual BEAM, que executa código Elixir e Erlang. A BEAM implementa o modelo de atores: cada entidade do sistema — uma guilda (servidor), uma sessão de usuário, uma chamada de voz, uma conexão WebSocket — é um ator independente com sua própria caixa de correio. Isso produz três propriedades que são particularmente valiosas para um sistema de chat em tempo real:

- **Isolamento de estado**: atores não compartilham memória. Se o ator responsável por uma guilda específica falha, o supervisor o reinicia sem afetar nenhuma outra guilda. Isso permite que o Discord isole falhas no nível de servidores individuais, em vez de experimentar quedas globais.
- **Comunicação por mensagem**: toda interação entre atores é uma mensagem assíncrona. Isso torna o sistema intrinsecamente rastreável (cada mensagem tem origem e destino definidos) e intrinsecamente distribuível (atores não precisam estar na mesma máquina).
- **Concorrência massiva**: a BEEM gerencia milhões de processos leves (não threads do SO) com troca de contexto de microssegundos. O Discord roda entre 400 e 500 máquinas executando a BEAM.

### 3.2 O Pipeline de Mensagens

Quando um usuário envia uma mensagem em um canal, o fluxo é:

1. **Gateway**: conexão WebSocket entre o cliente e o servidor. O Discord usa a biblioteca Cowboy (Erlang) para gerenciar milhões de conexões WebSocket simultâneas. O gateway aplica rate limiting — aproximadamente 26 milhões de eventos por segundo em pico.
2. **Validação e persistência**: a mensagem é validada (tamanho, permissões do usuário no canal), passa por filtros de spam, e é persistida no banco de dados.
3. **Fanout**: o servidor determina quais membros do canal estão online e entrega a mensagem a cada um via WebSocket. A latência média do gateway à entrega é inferior a 50 milissegundos. O pico de throughput é de 500 mil mensagens por segundo.

Uma otimização crítica: o Discord desabilita notificações para sessões passivas (usuários que estão online mas não interagiram recentemente), o que reduziu o custo do fanout em 90%.

### 3.3 Cassandra → ScyllaDB

O Discord armazenava mensagens no Apache Cassandra, usando uma chave de partição que combinava `channel_id` com um bucket de 10 dias. Servidores populares — como o do Midjourney, com 19 milhões de membros — criavam "hot partitions": leituras concentradas em uma única partição, causando degradação de latência para todos os usuários daquele canal.

A migração para ScyllaDB — um banco de dados compatível com Cassandra escrito em C++ — resolveu o problema. ScyllaDB oferece sharding por núcleo de CPU (em vez de por instância), elimina pausas de garbage collection, e implementa coalescimento de requisições: se 100 usuários solicitam a mesma mensagem simultaneamente, apenas uma consulta ao banco é executada.

### 3.4 Rust no Data Service

O Discord construiu uma camada de middleware em Rust — o Data Service Library — que intercepta requisições ao banco de dados. Sua função principal é coalescer requisições duplicadas em voo. O resultado foi uma redução de 10 a 50 vezes na taxa de consultas inbound ao banco. Em benchmarks internos, a latência de leitura no percentil 95 caiu de 250ms para 42ms.

### 3.5 A Infraestrutura de Voz: WebRTC com SFU Customizado

O sistema de voz do Discord não usa WebRTC puro no modelo peer-to-peer — que funciona bem para duas ou três pessoas, mas se torna exponencialmente inviável conforme o número de participantes cresce. Em vez disso, o Discord implementa um modelo cliente-servidor via SFU (Selective Forwarding Unit):

- **Plano de controle (Elixir)**: gerencia autenticação, permissões, configuração de conexão, IDs de stream, chaves de criptografia, e indicadores de quem está falando.
- **Plano de dados (C++ customizado)**: um servidor de mídia que recebe streams de áudio e vídeo de cada participante e os reencaminha aos demais, sem decodificar ou recodificar. Cada participante envia um stream ao servidor; o servidor replica para todos os outros. Adicionar um décimo participante requer apenas 10 conexões de recebimento adicionais, não 90.

O Discord fez modificações significativas na biblioteca WebRTC padrão. Eliminou ICE/STUN — já que todo o tráfego passa pelo servidor, a negociação ICE é desnecessária, o que simplifica a travessia de NAT e protege o endereço IP do usuário. Substituiu DTLS+SRTP por Salsa20 para criptografia mais rápida. Implementou supressão de silêncio para evitar o envio de pacotes quando o usuário está quieto. Reduziu a troca SDP ao mínimo necessário: endereço do servidor, chave de criptografia, codec (Opus) e IDs de stream.

A infraestrutura de voz opera em 850 servidores físicos distribuídos em 13 regiões geográficas e mais de 30 datacenters. Em pico, atende 2,5 milhões de usuários simultâneos em voz, gerando 220 Gbps de tráfego e 120 milhões de pacotes por segundo.

---

## 4. Modelo de Negócios: Nitro Como Âncora

O Discord monetiza através do Discord Nitro, uma assinatura que oferece uploads maiores, streaming em alta definição, emojis customizados globais, perfis personalizados e "boosts" de servidor. O preço é US$ 9,99 por mês (Nitro Basic custa US$ 2,99). A empresa não vende anúncios e não coleta dados de usuários para publicidade.

Três fatores predizem a conversão para Nitro: tempo gasto no Discord, tempo em canais de voz, e número de amigos que já possuem Nitro. O terceiro fator é particularmente revelador: Nitro tem um componente social. Ter amigos com Nitro expõe o usuário a emojis animados e perfis customizados, criando um ciclo de desejo aspiracional. O Discord Shop — onde usuários compram itens cosméticos para seus avatares — funciona como um canal de monetização complementar, mas permanece secundário em relação ao Nitro.

Em 2021, a empresa tentou diversificar receita com Stages (eventos de áudio ao vivo), mas a adoção foi limitada. Em 2024, introduziu Quests — um sistema inspirado em games onde usuários completam tarefas (como assistir a um trailer ou jogar um jogo específico) para ganhar recompensas. É publicidade, mas apresentada na linguagem da plataforma.

---

## 5. Lições de Produto

### 5.1 A melhor feature do seu produto pode ser algo que você construiu para uso interno

O sistema de chat e voz que se tornou o Discord não foi projetado para ser um produto. Foi construído para que a equipe da Hammer & Chisel pudesse testar Fates Forever. Citron e Vishnevskiy só perceberam seu valor quando se viram usando-o mais do que qualquer ferramenta externa. Isso ecoa um padrão recorrente em produtos de comunicação: o Slack nasceu do chat interno da Tiny Speck durante o desenvolvimento do Glitch. O Google Docs começou como uma ferramenta interna do Google. A lição não é "construa ferramentas internas" — é "preste atenção em quais ferramentas internas sua equipe adota voluntariamente". O que as pessoas usam quando ninguém está obrigando é o que tem valor real.

### 5.2 A unidade atômica do seu produto define tudo

No Facebook, a unidade atômica é o post — uma unidade de conteúdo que compete por atenção em um feed. No Discord, a unidade atômica é o servidor — um espaço persistente que acumula história, normas e relacionamentos. Essa diferença fundamental produz ecossistemas radicalmente diferentes. Um servidor Discord tem dono, regras, cargos, canais temáticos, bots e uma cultura interna. É uma instituição em miniatura, não uma unidade de conteúdo. Produtos que organizam sua experiência em torno de espaços (servidores, canais, projetos) produzem comunidades; produtos que organizam sua experiência em torno de conteúdo (posts, stories, reels) produzem audiências. A diferença é profunda e persistente.

### 5.3 O recuo estratégico não é derrota

Em 2024, o Discord demitiu 170 pessoas e abandonou a ambição de ser um "third place para todos", retornando ao posicionamento centrado em games. A leitura superficial vê fracasso; a leitura estratégica vê disciplina. O Discord entendeu que o contexto compartilhado do jogo não era um nicho do qual escapar, mas o fundamento sobre o qual o third place funcionava. Sem o contexto compartilhado do jogo, um servidor Discord vira um grupo de WhatsApp com canais. É o "antes, durante e depois do jogo" — nas palavras do CTO Stanislav Vishnevskiy — que cria o pretexto para a conversa. Expandir para além desse contexto significava diluir a razão pela qual as pessoas estavam lá em primeiro lugar.

### 5.4 Cinco engenheiros podem operar infraestrutura de 4 bilhões de mensagens diárias

A equipe de chat infrastructure do Discord tem cinco pessoas. Isso só é possível porque as escolhas arquiteturais foram feitas com a escala como requisito de primeiro ordem, não como otimização futura. O modelo de atores em Elixir/Erlang, a migração de Cassandra para ScyllaDB, o middleware de coalescimento em Rust, a separação entre plano de controle (Elixir) e plano de dados (C++) para voz — cada uma dessas decisões foi tomada não porque era a opção mais rápida de implementar, mas porque era a opção que escalaria sem exigir mais engenheiros. A dívida técnica que o Discord evitou nos primeiros anos está sendo paga diariamente na forma de uma equipe que consegue operar uma das maiores infraestruturas de comunicação do mundo sem crescer.

---

## 6. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Discord |
| **Fundação** | 2012 (Hammer & Chisel). Lançamento do Discord: 13 de maio de 2015. |
| **Fundadores** | Jason Citron (CEO), Stanislav Vishnevskiy (CTO) |
| **IPO** | Não. Privado. Microsoft ofereceu ~US$ 12B (2021). Valuation estimado: ~US$ 15B. |
| **Categoria** | Comunicação / Comunidade / Plataforma Social |
| **Plataformas** | Windows, macOS, Linux, iOS, Android, Web |
| **MAUs** | 150 milhões (2024) |
| **Mensagens/dia** | 4 bilhões |
| **Usuários simultâneos** | 12 milhões+ |
| **Preço** | Gratuito. Nitro Basic: US$ 2,99/mês. Nitro: US$ 9,99/mês. |
| **Tech Stack** | Elixir/Erlang (BEAM), Python (API), Rust (Data Service), C++ (WebRTC SFU), ScyllaDB, Redis, Elasticsearch |
| **Concorrentes** | Telegram, Slack, TeamSpeak, Guilded (Roblox), Microsoft Teams |

---

## 7. Linha do Tempo

```
2008 — Jason Citron lança Aurora Feint. Cria OpenFeint como spin-off.
2011 — OpenFeint vendido para GREE por US$ 104M.
2012 — Citron funda Hammer & Chisel.
2014 — Fates Forever é lançado. Fracassa.
2015 — Citron demite 1/3 da equipe. Seis meses de reestruturação.
2015 Mai 13 — Discord é lançado. Post em subreddit de FFXIV traz primeiros usuários.
2016 — 11M de usuários registrados.
2017 — 45M registrados. Nitro é lançado.
2018 — 130M registrados. Primeiros servidores não-gaming aparecem.
2020 — Pandemia. 100M+ MAUs. Escolas migram para a plataforma.
2021 — Microsoft oferece ~US$ 12B. Citron recusa. Discord tenta ser "third place para todos".
2024 — 170 demissões. Recuo estratégico para games. Discord Shop e Quests.
2025 Jan — Paper da UW: "Discord's Design Encourages Third Place Social Media Experiences".
2025 — Rumores de IPO com JP Morgan Chase.
```

---

## Fontes

- [Wikipedia — Discord](https://en.m.wikipedia.org/wiki/Hammer_%26_Chisel,_Inc.)
- [Gigazine — How Discord created the future of the Internet (2021)](https://gigazine.net/gsc_news/en/20210207-discord-invent-internet-future)
- [Greylock — The State of Gaming with Discord CEO Jason Citron](https://news.greylock.com/the-state-of-gaming-with-discord-ceo-jason-citron-e604479f10bd)
- [Elixir Lang Blog — Real-time communication at scale with Elixir at Discord (2020)](https://elixir-lang.org/blog/2020/10/08/real-time-communication-at-scale-with-elixir-at-discord/)
- [HackerNoon — Inside Discord's Architecture at Scale (2024)](https://hackernoon.com/lite/inside-discords-architecture-at-scale)
- [InfoQ — Discord Scales to 1M+ Online MidJourney Users (2024)](https://www.infoq.com/news/2024/01/discord-midjourney-performance/)
- [WebRTC.org.cn — Discord WebRTC Architecture (tradução do original)](https://webrtc.org.cn/20180922-webrtc-vedio/)
- [arXiv:2501.09951 — Discord's Design Encourages "Third Place" Social Media Experiences (Kim et al., 2025)](https://export.arxiv.org/abs/2501.09951)
- [GameDeveloper — Discord's rumored IPO revenue strategy (2025)](https://www.gamedeveloper.com/business/with-an-ipo-on-the-way-let-s-dig-into-discord-s-revenue-strategy)
- [Britannica — Discord Overview](https://www.britannica.com/topic/Discord)
