# Estudo de Caso 24 — Roblox: A Plataforma Onde Crianças Constroem Jogos, Ganham Dinheiro, e o Sistema de Moderação Processa 6 Bilhões de Mensagens Por Dia

> **Data:** 2026-07-03
> **Loop:** 24 de ∞ (Reescrita — Fase 2)
> **Categoria:** Gaming / UGC / Plataforma Social
> **Tema:** 1989. David Baszucki e Erik Cassel fundam a Knowledge Revolution, uma empresa de software educacional. Seu produto principal, Interactive Physics, é um laboratório de simulação de física 2D usado em escolas. Os dois observam algo que define o resto de suas carreiras: os alunos não querem usar as simulações prontas. Querem construir suas próprias. Em 1998, vendem a empresa por US$ 20 milhões. Em 2004, começam a trabalhar num protótipo chamado DynaBlocks — a visão de um mundo 3D onde usuários criam suas próprias experiências, com avatares, física e interação social. O nome muda para Roblox — "robots" + "blocks" — em janeiro de 2005. Erik Cassel morre de câncer em 2013, aos 45 anos, sem ver o que o Roblox se tornaria. Em 2021, a empresa abre capital via listagem direta na NYSE, avaliada em US$ 41 bilhões. Em 2025, atinge 111 milhões de usuários ativos diários, processa 6,1 bilhões de mensagens de chat por dia, e opera uma infraestrutura de moderação que combina 200 sistemas de AI com 3.000 moderadores humanos. Em 2026, anuncia o Roblox Reality — uma arquitetura híbrida que combina o motor de jogo tradicional com modelos de vídeo generativos para produzir visuais fotorrealistas.

---

## 0. A Linhagem: De Sims de Física a Metaverso

```
LEGO físico (1932-): construir com tijolos. Físico. Individual.
      ↓
Interactive Physics (1989): simulação 2D. Alunos constroem experimentos.
      ↓
Second Life (2003): mundo virtual persistente. Adultos. Conteúdo gerado por usuário.
      ↓
Minecraft (2011): blocos. Sobrevivência. Criação. Mas não é plataforma — é jogo.
      ↓
Roblox (2006): plataforma UGC. Motor proprietário. Criadores são desenvolvedores.
      ↓
Roblox hoje (2026): 111M DAU. 40M+ experiências. Economia de criadores. Híbrido game/AI.
```

O Roblox não é um jogo — é uma plataforma onde os usuários constroem os jogos. Ele não compete com Fortnite ou Minecraft como experiência de entretenimento; compete com eles como destino de atenção. Mas a analogia mais precisa para o Roblox é uma combinação de App Store (distribuição), YouTube (criadores geram o conteúdo) e Unity (motor de jogo) — tudo operado por uma única empresa que também controla a moeda virtual (Robux), a moderação e as regras da economia interna.

---

## 1. A Origem: Física Educacional, Dois Fundadores e o Que Os Alunos Realmente Queriam

A Knowledge Revolution, fundada por Baszucki e Cassel em 1989, era uma empresa de software educacional. O Interactive Physics permitia que alunos simulassem experimentos de física — colisões, pêndulos, planos inclinados — em um ambiente 2D. O software vendeu milhões de cópias e foi traduzido para nove idiomas.

Mas a observação que moldou o futuro dos dois fundadores não foi sobre educação. Foi sobre comportamento. Os alunos passavam mais tempo construindo suas próprias simulações do que usando as prontas. Eles queriam criar coisas — carros que explodiam, casas que desabavam, máquinas de Rube Goldberg — e mostrar para os amigos. A lição era clara, mas a tecnologia para executá-la em escala ainda não existia: as pessoas querem construir, e querem que outras pessoas vejam o que construíram.

Em 1998, a MSC.Software adquiriu a Knowledge Revolution por US$ 20 milhões. Baszucki e Cassel passaram alguns anos na empresa adquirida. Em 2002, Baszucki saiu para fundar uma firma de investimentos-anjo. Em dezembro de 2003, os dois começaram a trabalhar em um protótipo que capturasse a dinâmica que observaram no Interactive Physics — mas em 3D, online e social.

O protótipo se chamava DynaBlocks. O nome não durou. Em janeiro de 2004, virou Roblox — uma aglutinação de "robots" e "blocks". A empresa foi formalmente constituída em Menlo Park, Califórnia. O produto foi lançado publicamente em 1º de setembro de 2006.

Erik Cassel morreu de câncer em 11 de fevereiro de 2013. Tinha 45 anos. Seus filhos construíram um memorial dentro do Roblox — um lugar chamado "Erik Cassel Memory Place" — que ainda existe.

O IPO do Roblox, em março de 2021, não foi um IPO tradicional. Foi uma listagem direta na NYSE, com preço de referência de US$ 45 por ação. As ações abriram a US$ 64,50, avaliando a empresa em aproximadamente US$ 41 bilhões. Baszucki reteve 70,1% do poder de voto através de uma estrutura de ações de classe dupla.

---

## 2. A Economia do Criador: Robux, DevEx e o Problema dos Incentivos

A economia do Roblox é lastreada em Robux, uma moeda virtual que os usuários compram com dinheiro real. Criadores de experiências e itens de avatar ganham Robux quando os usuários gastam dentro de suas criações. Através do Developer Exchange (DevEx), criadores podem converter Robux de volta para dólares quando acumulam pelo menos 30.000 Robux — aproximadamente US$ 105. A divisão de receita varia: criadores podem receber até 70% em itens de avatar, mas apenas 10% de comissão em alguns casos.

A economia é real. O Roblox pagou mais de US$ 900 milhões a criadores em 2024. Os dez maiores desenvolvedores ganham, em média, mais de US$ 30 milhões por ano cada. Um adolescente de 16 anos construiu um jogo chamado "Grow a Garden" em três dias — plantar, crescer, vender, melhorar — e ganhou entre US$ 1,2 e US$ 1,8 milhão em um mês. Esses números são reais e documentados.

Mas a economia de criadores do Roblox também gera tensões que nenhuma plataforma UGC resolveu completamente. Um paper apresentado na conferência ACM CHI 2025 entrevistou vinte criadores do Roblox e documentou como o modelo de monetização — que recompensa tempo de sessão, compras repetidas e mecânicas de acaso — incentiva designs de jogo que maximizam extração de valor em vez de qualidade de experiência. Mecânicas de loot box, rodas da fortuna e "mystery boxes" são explicitamente recomendadas no Creator Hub do Roblox. Para um adulto, esses mecanismos são familiares e regulados. Para uma plataforma onde 40% dos usuários têm menos de 13 anos, eles são o centro de um debate regulatório e ético que está longe de ser resolvido.

---

## 3. Arquitetura Técnica: Física Distribuída e o Motor Que Roda em 26 Datacenters

O Roblox executa um motor de jogo proprietário escrito em C++, com scripts de criadores executados em uma sandbox Lua. Experiências de topo contêm mais de 50.000 linhas de código Lua. A plataforma opera vinte e seis datacenters de borda distribuídos globalmente, executando milhões de instâncias de jogos simultaneamente.

O modelo de rede é servidor autoritativo: o servidor mantém o estado oficial do jogo; os clientes usam predição especulativa para reduzir a latência percebida. Quando o servidor envia uma correção, o cliente faz rollback e re-simula a partir do estado corrigido. Objetos e jogadores distantes não geram atualizações de rede (interest management). Apenas dados modificados são transmitidos (delta state updates).

O sistema de física é a peça mais interessante da arquitetura. O Roblox usa um modelo de física distribuída: o servidor delega a simulação física de objetos próximos a um jogador para o cliente desse jogador, mantendo autoridade sobre objetos que nenhum jogador está perto. A carga é balanceada dinamicamente: a região de simulação de cada jogador expande ou contrai com base na capacidade de processamento e largura de banda disponíveis. A simulação física roda a 240 Hz — fixa, independente da taxa de quadros de renderização. Esse modelo permite que o Roblox dimensione para "um número quase infinito de corpos fisicamente simulados", nas palavras de um paper técnico da empresa.

O Roblox também desenvolveu o SLIM (Scalable Lightweight Interactive Models), um sistema que cria representações geométricas simplificadas de objetos 3D complexos automaticamente. O Harmony, sistema complementar, monitora os recursos disponíveis em cada dispositivo (memória, GPU, CPU, largura de banda) e ajusta a qualidade do streaming de conteúdo em tempo real. Juntos, permitem que uma experiência rode tanto num PC gamer de US$ 3.000 quanto num iPhone com dez anos de idade.

O anúncio mais ambicioso veio em 2026: o Roblox Reality, uma arquitetura híbrida que combina o motor de jogo tradicional com Video World Models — modelos generativos de vídeo treinados para produzir visuais fotorrealistas. O motor de jogo continua responsável pela simulação determinística (física, colisões, lógica de jogo, estado persistente). Os Video World Models, executados em GPUs H200/B200 em datacenters adjacentes, geram detalhes visuais estocásticos: texturas de alta fidelidade, grama que se move com o vento, reflexos, poeira, nuvens. A separação entre "o que é verdade" (motor) e "o que é verossímil" (modelo de vídeo) resolve uma tensão fundamental em jogos multiplayer: simulação determinística garante justiça; geração estocástica garante beleza.

---

## 4. Moderação: 200 Sistemas de AI, 3.000 Humanos e Seis Bilhões de Mensagens Por Dia

O Roblox processa 6,1 bilhões de mensagens de chat por dia e 1,1 milhão de horas de comunicação por voz. Entre fevereiro e dezembro de 2024, os usuários enviaram aproximadamente um trilhão de peças de conteúdo. Moderar essa escala sem degradar a experiência do usuário é, em termos de engenharia, um dos problemas mais difíceis da internet.

A plataforma opera aproximadamente duzentos sistemas de AI para segurança. O filtro de texto — baseado em transformers — processa 750.000 requisições por segundo. Um filtro de informações pessoais identificáveis opera a 370.000 requisições por segundo em pico. O sistema de moderação de voz — um classificador de AI que analisa conversas em oito idiomas — foi aberto ao público como open source. O Roblox Sentinel, também open source, detecta padrões de comportamento de longo prazo que evoluem para violações de política — por exemplo, grooming que começa com conversa inócua e escala ao longo de semanas.

A moderação humana complementa a AI para casos ambíguos, raros ou complexos. Três mil moderadores revisam conteúdos sinalizados por máquinas, investigam denúncias e processam apelações. A filosofia declarada é que AI só é implantada quando supera humanos em precisão e recall para uma categoria específica de moderação.

Em novembro de 2024, o Roblox introduziu uma série de mudanças que endureceram significativamente o acesso de menores a funcionalidades sociais. Contas de usuários com menos de 13 anos foram automaticamente bloqueadas de mensagens diretas fora de jogos. Jogos focados em socialização foram restritos a maiores de 13 anos. Espaços com quartos ou banheiros foram reclassificados como 17+. Controles parentais passaram a ser ativados por padrão para todas as contas de menores.

Essas mudanças não ocorreram no vácuo. Desde 2018, pelo menos trinta pessoas foram presas nos Estados Unidos por abduzir ou abusar sexualmente de crianças que conheceram no Roblox. O procurador-geral da Louisiana processou a empresa em 2025, alegando que a plataforma "prioriza crescimento de usuários, receita e lucros sobre a segurança infantil". Vários países do Oriente Médio baniram o Roblox. A tensão entre crescimento e segurança é o problema de produto mais complexo que a empresa enfrenta — e não tem solução puramente técnica.

---

## 5. Lições de Produto

### 5.1 A moeda virtual é o produto mais importante da plataforma — e o mais perigoso

O Robux resolveu o problema de monetização para uma plataforma onde o conteúdo é gerado por usuários: criadores ganham Robux, consumidores gastam Robux, e o Roblox controla a taxa de câmbio entre Robux e dólares. Isso cria um circuito fechado onde o valor é gerado e capturado dentro do ecossistema. Mas também cria incentivos perversos: quando criadores são pagos com base em quanto tempo os usuários passam em suas experiências e quanto Robux gastam, o design de jogo converge para mecânicas de extração — loot boxes, grind, FOMO. Regular esse sistema sem sufocar a economia de criadores que o alimenta é o dilema central de qualquer plataforma UGC.

### 5.2 A moderação em escala é um problema de engenharia de sistemas, não apenas de policy

Seis bilhões de mensagens por dia não podem ser moderadas por humanos. Também não podem ser moderadas exclusivamente por máquinas, porque máquinas não entendem contexto, ironia, grooming de longo prazo ou novas táticas de evasão. A solução do Roblox — AI para triagem em escala, humanos para casos complexos, sistemas de detecção de padrões de comportamento para ameaças que evoluem ao longo do tempo — é provavelmente a arquitetura de moderação mais sofisticada em operação em qualquer plataforma social. Também é insuficiente, como os processos judiciais demonstram. A lição é que moderação não tem "pronto": é uma corrida armamentista perpétua contra adversários que se adaptam.

### 5.3 Física distribuída resolve um problema que a maioria das plataformas nem sabe que tem

O modelo de física distribuída do Roblox — onde clientes processam a simulação de objetos próximos enquanto o servidor mantém autoridade sobre o resto — permite que o sistema dimensione para milhões de corpos físicos simultâneos sem que o servidor se torne o gargalo. É uma arquitetura contraintuitiva (o dogma de jogos multiplayer é "nunca confie no cliente"), mas funciona porque o Roblox não é um jogo competitivo — é uma plataforma social onde a precisão da física importa menos do que a percepção de responsividade.

---

## 6. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Roblox |
| **Fundação** | 2004 (DynaBlocks). Lançamento público: setembro de 2006. |
| **Fundadores** | David Baszucki (CEO), Erik Cassel (falecido em 2013) |
| **IPO** | 10 de março de 2021. Listagem direta na NYSE (RBLX). Valuation inicial: ~US$ 41B. |
| **DAU** | 111,8 milhões (Q2 2025) |
| **Horas jogadas** | Dezenas de bilhões por ano |
| **Criadores ativos** | Milhões |
| **Pago a criadores (2024)** | US$ 900 milhões+ |
| **Moderação** | 200 sistemas de AI, 3.000 moderadores humanos. 6,1B mensagens/dia. |
| **Preço** | Gratuito. Robux: compra com dinheiro real. Premium: US$ 4,99-19,99/mês. |
| **Tech Stack** | C++ (engine), Lua (scripting), física distribuída, SLIM, Harmony, Roblox Reality (2026) |
| **Concorrentes** | Fortnite (UEFN), Minecraft, Rec Room |

---

## Fontes

- [Roblox Newsroom — Hybrid Architecture: Democratizing Photorealistic Multiplayer Gaming (2026)](https://d3fel7ao8ljmgc.cloudfront.net/en-au/newsroom/2026/04/roblox-reality-hybrid-architecture-democratizing-photorealistic-multiplayer-gaming)
- [Roblox Newsroom — SLIM: Scalable Lightweight Interactive Models (2025)](https://d3fel7ao8ljmgc.cloudfront.net/en-au/newsroom/2025/12/introducing-roblox-slim-scalable-lightweight-interactive-models)
- [Roblox Newsroom — Tech Talks: Update On Safety Initiatives (2025)](https://d3fel7ao8ljmgc.cloudfront.net/newsroom/2025/08/tech-talks-update-on-roblox-safety-initiatives)
- [Roblox Newsroom — How Roblox Uses AI to Moderate Content on a Massive Scale (2025)](https://d3fel7ao8ljmgc.cloudfront.net/newsroom/2025/07/roblox-ai-moderation-massive-scale)
- [Game Developer — Distributed Physics: The Key to Online Game Simulation (2012)](https://www.gamedeveloper.com/programming/distributed-physics-the-key-to-online-game-simulation)
- [ACM DL — "The System is Made to Inherently Push Child Gambling": Creator Economy & Monetization on Roblox (CHI 2025)](https://dl.acm.org/doi/10.1145/3706598.3713170)
- [Wikipedia — Child safety on Roblox](https://en.wikipedia.org/wiki/Child_safety_on_Roblox)
- [InvestorPlace — Roblox IPO Guide (2020)](https://investorplace.com/2020/12/heres-what-you-need-to-know-heading-into-the-roblox-ipo/)
