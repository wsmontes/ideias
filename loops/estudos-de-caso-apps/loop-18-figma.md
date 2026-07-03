# Estudo de Caso 18 — Figma: O App Que Colocou o Design No Navegador (E Forçou a Adobe a Desistir)

> **Data:** 2026-07-03
> **Loop:** 18 de ∞ (Reescrita — Fase 2)
> **Categoria:** Design / Colaboração / Ferramentas Web
> **Tema:** Abril de 2011. Evan Wallace, um TA de ciência da computação na Brown University, aparece no laboratório depois de um fim de semana de hackathon. Ele conseguiu recriar uma API de processamento de imagem do lado do servidor usando **WebGL** — uma interface gráfica de baixo nível que rodava direto no navegador. "Dá para construir ferramentas criativas inteiras aqui dentro", disse a Dylan Field, um aluno de CS e matemática que acompanhava seu trabalho. Durante o ano seguinte, os dois discutiram obsessivamente como o WebGL poderia tornar as ferramentas criativas acessíveis a qualquer pessoa, em qualquer lugar, sem baixar nada. Em 2012, Field ganhou a **Thiel Fellowship** — US$ 100 mil para largar a faculdade e abrir uma empresa. Sua inscrição original era para uma empresa de **drones**. Wallace o convenceu a abandonar a ideia: hardware tem ciclos de depuração longos demais, a regulamentação de espaço aéreo é imprevisível, e toda ideia de drone que eles tiveram ou violava privacidade ou causava dano físico. Os dois pivotaram para "Photoshop no navegador". Passaram meses sem foco — um editor de fotos, um gerador de memes, ferramentas 3D — até que Field sentou com a equipe inteira e forçou uma decisão. Cortaram animação, 3D, edição de foto, pintura. Escolheram uma coisa só: **design de interfaces.** Passaram quatro anos construindo antes de mostrar ao mundo. Em 27 de setembro de 2016, o **Figma** foi lançado publicamente. Designers odiaram. "Ferramenta de design no navegador é lenta, feia, limitada." Mas o Figma tinha algo que nem Photoshop nem Sketch tinham: **multiplayer de verdade.** Múltiplas pessoas no mesmo arquivo, editando ao mesmo tempo, com cursores ao vivo, sem precisar instalar nada. Em 2021, valia US$ 10 bilhões. Em 2022, a Adobe tentou comprá-lo por US$ 20 bilhões — e quando os reguladores barraram o negócio, a Adobe pagou US$ 1 bilhão de multa e **descontinuou o Adobe XD**, o produto que o Figma tornara obsoleto. Em julho de 2025, o Figma abriu capital na NYSE. As ações fecharam o primeiro dia valendo US$ 115,50 — 250% acima do preço do IPO — com um valor de mercado de US$ 56 bilhões. O aplicativo que começou com dois universitários discutindo WebGL no laboratório de Brown vale hoje três vezes o que a Adobe ofereceu.

---

## 0. A Linhagem: O Que Havia Antes do Design Colaborativo no Navegador

```
Mesa de luz e caneta (pré-digital): o designer trabalhava fisicamente, com réguas, letras transferíveis e nanquim.
      ↓
Adobe Photoshop (1990), Illustrator (1987), InDesign (1999): profissionais. Arquivos .psd e .ai.
      ↓
Sketch (2010): o primeiro app de design de interfaces nativo para Mac. Vetorial. Mais leve que a Adobe.
      ↓
Figma (2016): navegador. Multiplayer. WebGL + WebAssembly. Zero instalação.
      ↓
Adobe XD (2017-2023): a resposta da Adobe ao Figma. Lançado às pressas. Descontinuado após o bloqueio do acquisition.
      ↓
Figma IPO (2025): US$ 56 bilhões. 13 milhões de usuários ativos mensais. Dois terços não são designers.
```

O Figma não inventou o design de interfaces (Sketch), nem o software de desenho vetorial (Illustrator), nem a colaboração online (Google Docs). Inventou o **design de interfaces colaborativo no navegador** — uma categoria que só existia em arquivos salvos no disco rígido e enviados por email para stakeholders que não podiam abri‑los. Antes do Figma, design era uma ilha. Depois, virou uma mesa compartilhada.

---

## 1. A Origem: Drones, Memes, e um Processador de Imagem no Navegador

O Figma deveria ter sido uma empresa de drones. Dylan Field inscreveu‑se na Thiel Fellowship com um projeto de UAVs civis. Evan Wallace, que programava e montava drones de verdade, fez Field desistir em três argumentos: o ciclo de debug de hardware é longo demais, a FAA é um obstáculo imprevisível, e toda aplicação de drones em que eles conseguiam pensar ou feria a privacidade ou causava danos físicos.

A dupla pivotou para o que Wallace havia demonstrado em 2011: **reimplementar um processador de imagens do lado do servidor usando WebGL.** O navegador, até então, era território de HTML, CSS e JavaScript. O WebGL — uma API de renderização de baixo nível inspirada no OpenGL — permitia que código rodasse diretamente na GPU. O que Wallace mostrou foi que dava para fazer no navegador o que até então exigia um servidor com FFmpeg ou uma aplicação nativa.

Field e Wallace convenceram‑se de que essa capacidade técnica permitiria construir **ferramentas criativas** que rodassem no navegador com desempenho nativo, eliminando a necessidade de baixar e instalar software. A visão de Field era radical: "Desde que o Writely [Google Docs] foi lançado dez anos atrás, eu acredito que todo software deveria ser online, em tempo real e colaborativo. Ferramentas criativas não fizeram esse salto porque o navegador nunca teve potência suficiente. Agora, com o WebGL, tudo mudou."

### Os Primeiros Anos: Perdidos Entre Filtros e Memes

O problema inicial não foi técnico — foi de foco. A dupla passou 2012 e 2013 experimentando ideias diferentes:

- **Um editor de fotos** com filtros inspirados no Instagram.
- **Um gerador de memes** usando WebGL. Field lembra dessa fase como "a pior semana do Figma". Ele se olhou no espelho e pensou: "Eu larguei Brown para isso?"
- **Ferramentas de modelagem 3D**, edição de vídeo, animação.

John Lilly, sócio da Greylock, passou a rodada seed e foi honesto: *"Acho que vocês ainda não sabem o que estão fazendo."* Ele estava certo.

A virada ocorreu quando Field reuniu os fundadores e os poucos funcionários numa sala e forçou uma decisão de produto. O grupo listou todas as direções possíveis e começou a eliminar. Cortaram animação, 3D, edição de foto, pintura, geração de assets. Sobrou uma coisa: **design de interfaces.** Era a categoria com a necessidade de mercado mais clara e o melhor encaixe para a arquitetura baseada em navegador que estavam construindo. Designers de interface passavam o dia no Sketch e no Photoshop, salvando arquivos no disco, exportando PNGs, enviando por email. Era um fluxo de trabalho quebrado e a web era a plataforma natural para consertá‑lo.

### O Seed Que Quase Não Veio

Em 2013, Field e Wallace levantaram uma **seed de US$ 1,7 milhão** liderada por Danny Rimer (Index Ventures). Rimer conhecera Field quando este era um estagiário de 19 anos na Flipboard. Jeff Weiner, então CEO do LinkedIn, também entrou como anjo. Não era uma rodada óbvia: o consenso entre VCs era que design profissional jamais rodaria bem no navegador.

### Quatro Anos no Stealth

O Figma levou quatro anos para ser lançado publicamente. A decisão de construir o núcleo de renderização em C++ e compilá‑lo para WebAssembly ainda era inédita quando começaram. Em 2015, trouxeram **Sho Kuwamoto**, veterano de 20 anos na Adobe e Macromedia, para liderar a engenharia de produto. Kuwamoto entendia profundamente o domínio de ferramentas criativas e ajudou a moldar a transição de um protótipo técnico para um produto utilizável.

Outra contratação crucial no período foi a de **Yuhki Yamashita**, que entrou como designer de produto e subiria a CPO anos depois. Yamashita articulou o que se tornaria a filosofia do Figma: *"Multiplayer não é uma feature — é o paradigma. O objetivo não é colocar um botão de 'compartilhar'. É tornar o design acessível a todo o time de produto."*

### O Lançamento Que Ninguém Pediu

**27 de setembro de 2016**. O Figma foi lançado como um aplicativo web gratuito para design de interfaces. Designers odiaram. "Lento." "Limitado." "Quem vai querer fazer design no navegador?" O que eles não perceberam — e o que o Figma havia passado quatro anos construindo — era que o navegador não era uma limitação. Era a fundação sobre a qual o multiplayer funcionava **nativamente**.

A estratégia de adoção foi deliberada: fazer designers usarem o Figma para projetos pessoais primeiro. Criar uma conta. Brincar. Fazer um redesign do aplicativo favorito. Quando vissem como era ter um PM, um engenheiro e outro designer no mesmo arquivo, com cursores coloridos se movendo, sem exportar PNG, sem "versão final FINAL.sketch" — não conseguiriam mais voltar.

Em 18 meses, a resistência inicial virou evangelismo. Designers que odiaram o Figma passaram a exigi‑lo nos novos empregos.

---

## 2. A Filosofia do Produto: O Navegador Não É Uma Limitação — É O Fosso

A decisão de construir o Figma como uma aplicação web não foi um compromisso. Foi uma aposta estratégica que produziu três vantagens estruturais impossíveis de replicar por concorrentes nativos.

A primeira é **multiplayer por arquitetura, não por adição**. Sketch, Adobe XD e Photoshop tentaram adicionar colaboração a bases de código originalmente projetadas para um único usuário e um único arquivo local. O resultado foram experiências lentas, quebradiças, com conflitos de merge e arquivos corrompidos — porque a arquitetura subjacente tratava a colaboração como exceção. No Figma, cada arquivo nasce como um espaço compartilhado: o servidor mantém o estado autoritativo e cada cliente é uma projeção desse estado. Não há "salvar". Não há "enviar". O que existe é um fluxo contínuo de mudanças de propriedades que trafegam entre clientes e servidor via WebSocket.

A segunda é **acesso universal sem instalação**. O Figma funciona em qualquer sistema operacional com um navegador moderno — macOS, Windows, Linux, ChromeOS. Para um stakeholder que só precisa ver e comentar, nem conta é necessária: basta abrir uma URL. Isso eliminou o gargalo de distribuição que afligia o Sketch (Mac apenas) e o Adobe XD (Mac e Windows, mas com instalação).

A terceira é **um modelo de negócios orientado a distribuição, não a licenças**. O plano gratuito do Figma não limita o número de colaboradores que podem visualizar e comentar — apenas o número de editores. Isso significa que cada arquivo do Figma funciona como uma demonstração do produto. Um designer pago convida cinco pessoas gratuitas. Duas dessas pessoas decidem que também querem editar. Convertem‑se em pagantes. O arquivo é o vendedor.

Quando perguntaram a Dylan Field qual era a vantagem competitiva mais duradoura do Figma, ele respondeu com uma palavra: **"multiplayer"**. Não a tecnologia de sincronização, mas o fato de que uma vez que um time inteiro — designers, PMs, engenheiros, pesquisadores, redatores — passa a habitar o mesmo espaço de trabalho, o custo de troca se torna proibitivo. Você não está trocando de ferramenta de design. Está reorganizando como seu time de produto se comunica.

---

## 3. As Inovações Técnicas e de Produto Que Sustentam o Fosso

### 3.1 O Motor de Renderização: C++ → WebAssembly → WebGL

A decisão de escrever o núcleo do Figma em C++ e compilá‑lo para WebAssembly, em vez de usar a stack nativa da web, foi controversa nos primeiros anos. A maioria das aplicações web usava JavaScript para tudo. O problema é que JavaScript é uma linguagem com garbage collection gerenciado, o que significa que a qualquer momento o coletor de lixo pode pausar a execução por dezenas de milissegundos — uma eternidade quando se precisa manter 60 quadros por segundo durante uma operação de arrastar um retângulo com sombra, blur e opacidade aninhada.

O C++ permitiu ao Figma controle absoluto sobre alocação e desalocação de memória. Objetos do SceneGraph vivem em um buffer pré‑alocado; o garbage collector do JavaScript nunca os toca. O resultado é um motor de renderização que opera consistentemente dentro de 2× do desempenho nativo — próximo o suficiente para que o usuário não perceba a diferença.

O WebAssembly forneceu o caminho de compilação: código C++ é transformado pelo Emscripten em bytecode que o navegador executa em uma sandbox com desempenho próximo ao nativo. Comparado com o asm.js que o Figma usava originalmente, o WebAssembly reduziu o tempo de carregamento em **3×** e acelerou o parsing em **20×**.

Sobre essa camada de C++/WASM, o Figma construiu um motor de renderização 2D proprietário — batizado internamente de **TinyGPU** — que em vez de depender do DOM, `<canvas>` ou SVG do navegador, desenha cada pixel diretamente via **WebGL**. Isso deu ao Figma controle sobre anti‑aliasing, blend modes, máscaras, gradientes com dithering e composição de camadas — coisas que os renderizadores nativos dos navegadores ou não fazem, ou fazem de forma inconsistente entre plataformas.

Recentemente o time iniciou a migração para **WebGPU**, sucessor do WebGL. O WebGPU permite compute shaders que transferem trabalho da CPU para a GPU e elimina o estado global que tornava o WebGL propenso a bugs sutis e difíceis de reproduzir.

### 3.2 O Motor de Sincronização: CRDTs Parciais com Servidor Central

A maioria dos editores colaborativos usa **Operational Transformation** (OT), o algoritmo popularizado pelo Google Docs. O Figma rejeitou essa abordagem. OT exige que cada operação seja transformada contra todas as operações concorrentes, gerando uma explosão combinatória quando múltiplos clientes editam offline e sincronizam depois.

A alternativa que o Figma construiu é uma **variação inspirada em CRDTs** governada por um servidor central. A unidade atômica de mudança é uma **propriedade individual de um objeto**. Se o Cliente A move um retângulo (muda a propriedade `x`), e o Cliente B muda a cor do mesmo retângulo (muda a propriedade `fill`), não há conflito — porque são propriedades diferentes. Se dois clientes alteram a **mesma** propriedade simultaneamente, o servidor aplica **last‑writer‑wins**: a última alteração recebida pelo servidor é a que vale.

Isso é mais simples que OT e mais simples que CRDTs completos, mas funciona porque captura a realidade de como designers trabalham: raramente duas pessoas editam **exatamente o mesmo atributo** de **exatamente o mesmo objeto** ao mesmo tempo. Quando editam, LWW é uma política de resolução aceitável.

Para ordenação de camadas na árvore de objetos, o Figma usa **fractional indexing**. Cada objeto tem uma posição representada como uma fração (0,3; 0,5; 0,7). Inserir um novo objeto entre dois existentes é trivial: calcula‑se o ponto médio (0,4). Isso evita a necessidade de reindexar a lista inteira a cada inserção, que seria um gargalo em cenários multiplayer.

### 3.3 Multiplayer Server em Rust

O servidor de sincronização foi reescrito em **Rust**, substituindo a implementação original em TypeScript. Rust oferece garantias de segurança de memória em tempo de compilação e desempenho comparável ao C++. Cada documento do Figma é atribuído a um processo de servidor dedicado, que mantém o estado autoritativo do SceneGraph e transmite mudanças aos clientes conectados.

Uma otimização importante: o servidor **não reexecuta a lógica de renderização**. O que ele armazena é uma representação compacta do grafo de objetos com as propriedades de cada nó, permitindo que o estado de um documento com milhares de camadas caiba em poucos megabytes de RAM.

### 3.4 Componentes, Variants e Design Systems Como Infraestrutura

O Figma introduziu **Components** em 2018 — objetos reutilizáveis cujas instâncias herdam propriedades do mestre, mas permitem overrides locais. Em 2020, lançou **Variants** — um mecanismo para agrupar múltiplas variações de um componente (como estados de botão: default, hover, pressed, disabled) em um único container com propriedades mapeáveis.

Esses dois recursos, combinados com **Shared Libraries** (bibliotecas de componentes publicadas que times inteiros consomem), transformaram o Figma de uma ferramenta de desenho para uma **plataforma de infraestrutura de design**. Uma vez que uma empresa como Uber, Microsoft ou Spotify publica sua biblioteca de componentes no Figma, trocar de ferramenta significa reconstruir anos de ativos de design. Isso cria um custo de troca que nenhum concorrente consegue superar apenas com features.

---

## 4. A Arquitetura de Distribuição: Como o Figma Cresceu Sem Vendedores

O motor de crescimento do Figma não é força de vendas. É **product‑led growth** (PLG) ancorado em três mecanismos.

O primeiro é o **plano gratuito com colaboradores ilimitados**. Um designer assina o plano Professional (US$ 12/mês). Convida seu PM, seu engenheiro, seu pesquisador e seu redator para visualizar e comentar o arquivo. Nenhum deles paga. Mas todos passam a usar o Figma. Quando um desses stakeholders decide que também quer criar seus próprios arquivos — talvez o PM queira fazer wireframes, ou o engenheiro queira inspecionar o CSS gerado — converte‑se em editor pago. O custo de aquisição é zero. O vendedor é o arquivo.

O segundo mecanismo é a **Comunidade Figma**. Lançada em 2019, é um marketplace onde designers publicam arquivos, plugins, templates e wireframes que outros designers podem duplicar com um clique. Isso produz um ciclo de retroalimentação: criadores publicam para ganhar visibilidade, consumidores encontram recursos que aceleram seu trabalho, e cada recurso consumido é um arquivo Figma que prende o usuário ao ecossistema. A Comunidade também serve como canal de aprendizado: um designer júnior abre o arquivo de um designer sênior e vê exatamente como ele organiza camadas, componentes e auto layout.

O terceiro mecanismo é **Figma Config**, a conferência anual que atrai milhares de designers e desenvolvedores. Diferente de conferências de tecnologia tradicionais, a Config funciona como um evento da comunidade de design — com lançamentos de produto, oficinas e networking — mas com o efeito colateral de transformar cada participante em um evangelista.

Os números são inequívocos: em 2025, o Figma tinha 13 milhões de usuários ativos mensais e **dois terços não eram designers**. A penetração corporativa é quase total: 95% das empresas da Fortune 500 usam Figma. A receita anual é de aproximadamente US$ 749 milhões, com retenção líquida de receita acima de 150% — ou seja, cada coorte de clientes gasta 50% mais a cada ano que passa na plataforma.

---

## 5. A Linha do Tempo do Design Visual

```
2012-2015 — Stealth. Sem identidade pública. Canvas escuro. Protótipos internos.
2016 — Lançamento público. Interface escura. Roxo escuro como cor de marca. Minimalista.
2017 — Primeiros componentes reutilizáveis. Biblioteca de ícones em evolução.
2018 — Components como primitiva de produto. O canvas começa a ficar mais claro.
2019 — Comunidade Figma. Tipografia "Whyte" como fonte principal. Layout mais arejado.
2020 — Variants. Auto Layout. O produto amadurece visualmente.
2021 — FigJam. Uma segunda superfície de produto com identidade visual complementar.
2022 — Aquisição frustrada pela Adobe. Figma usa o US$ 1B de breakup fee para acelerar P&D.
2023 — Dev Mode. Primeira superfície de produto desenhada para um público não‑designer.
2024 — Rebrand completo: Figma Sans, Primitives, 29 cores. "Unapologetically jumbo."
2025 — IPO. Config 2025. Figma Make (AI). Figma Slides. Figma Sites. WebGPU.
```

### Identidade Visual (2024-2025): A Linguagem Que Substituiu o Sistema

O rebrand de 2024, liderado pelo diretor criativo Damien Correll, partiu de uma premissa pouco usual: *"Identidades visuais duradouras devem ser pensadas como **linguagens**, não como sistemas. Sistemas implicam regras rígidas e previsibilidade."*

A paleta de cores abandonou o roxo como "cor única" e expandiu-se para **29 cores** organizadas em três categorias: primárias vibrantes, neons elétricos e tons terrosos suaves. O sistema usa **Figma Variables** para adaptar-se automaticamente a temas claros e escuros. A tipografia migrou de Whyte para **Figma Sans**, uma "grotesca opinativa" encomendada à fundição suíça **Grilli Type**. É uma fonte variável com eixos de peso, largura e tamanho óptico, complementada por **Figma Condensed**, **Figma Mono** e **Figma Hand**.

A ilustração abandonou as metáforas de cursores que dominavam a identidade anterior e adotou **Primitives** — formas básicas (círculos, quadrados, manchas) que funcionam como tijolos de composição. Os nós de seleção são "jumbo" — quadrados enormes nos cantos das bounding boxes — uma referência direta aos controladores de seleção da interface do produto. A animação foi reduzida intencionalmente para **15 quadros por segundo**, criando uma sensação tátil e artesanal, oposta à fluidez polida de 60 fps que caracteriza a maioria dos produtos de tecnologia.

---

## 6. Anatomia do Produto (2025)

O Figma de 2025 não é mais uma ferramenta única. É uma plataforma com múltiplas superfícies de produto que compartilham o mesmo motor de renderização e o mesmo sistema de sincronização:

| Superfície | Função | Público Principal |
|---|---|---|
| **Figma Design** | Design de interfaces, prototipagem, design systems | Designers |
| **FigJam** | Quadro branco colaborativo para ideação e diagramação | Todos os stakeholders |
| **Dev Mode** | Inspeção de designs, extração de medidas e código | Desenvolvedores |
| **Figma Slides** | Criação de apresentações interativas dentro do canvas | Designers, PMs |
| **Figma Make** | Geração de protótipos funcionais via prompt de linguagem natural | Designers, PMs |
| **Figma Sites (beta)** | Publicação de designs como websites sem código | Designers, marketers |

Cada superfície compartilha o mesmo modelo de dados subjacente, o que significa que um componente criado no Figma Design pode ser reutilizado no Figma Slides, inspecionado no Dev Mode e publicado no Figma Sites, sem exportação ou conversão.

---

## 7. A Oferta de US$ 20 Bilhões e O Que Ela Revela Sobre a Adobe

Em setembro de 2022, a Adobe anunciou a aquisição do Figma por US$ 20 bilhões — metade em dinheiro, metade em ações. Seria a maior aquisição de uma empresa de software privada na história.

O anúncio provocou um colapso nas ações da Adobe: investidores interpretaram o preço como admissão de que o Adobe XD havia falhado e de que o modelo de negócios de licenças perpétuas da Adobe estava sendo corroído por ferramentas colaborativas baseadas em navegador. A reação regulatória foi imediata. O Departamento de Justiça dos EUA, a Comissão Europeia e a CMA do Reino Unido abriram investigações antitruste. O argumento central era simples: a Adobe era o único concorrente capaz de desafiar o Figma, e adquiri‑lo eliminaria a pressão competitiva no mercado de design de interfaces.

Em dezembro de 2023, com os três reguladores prestes a bloquear o negócio, Adobe e Figma anunciaram mutuamente o cancelamento. A Adobe pagou uma **taxa de rescisão de US$ 1 bilhão** e, meses depois, **descontinuou o Adobe XD**.

O episódio revelou três coisas. Primeiro, que o Figma havia se tornado estruturalmente difícil de competir: o Adobe XD, um produto nativo com anos de desenvolvimento, não conseguia igualar a experiência multiplayer do Figma porque sua arquitetura não fora projetada para isso. Segundo, que os reguladores estavam dispostos a intervir em aquisições de software de design — um mercado que a Adobe dominara por décadas via aquisições (Macromedia, 2005). Terceiro, que o valuation de US$ 20 bilhões, considerado absurdo em 2022, revelou‑se conservador: dois anos depois, o IPO do Figma valorizou a empresa em US$ 56 bilhões.

---

## 8. Lições de Produto

### 8.1 Onde os outros veem limitação técnica, veja uma vantagem arquitetural

O consenso em 2012 era que o navegador jamais rodaria ferramentas criativas com desempenho profissional. Field e Wallace ignoraram esse consenso porque entenderam algo que seus críticos não entendiam: o gargalo não era o navegador como plataforma — era a stack de desenvolvimento que as pessoas estavam usando nele. Ao abandonar o DOM e renderizar diretamente via WebGL com um núcleo em C++ compilado para WebAssembly, eles contornaram as limitações que todos os outros aceitavam como inevitáveis.

Isso é diferente de "persistência". É uma forma de pensamento de primeira ordem: em vez de perguntar "o navegador consegue rodar isso?", perguntar "o que seria necessário para o navegador rodar isso?".

### 8.2 O plano gratuito é sua força de vendas

O Figma entendeu antes dos concorrentes que o usuário pagante não é o cliente — é o patrocinador da conta. O verdadeiro motor de crescimento são os usuários gratuitos que ele convida. Cada stakeholder que recebe um link do Figma e abre no navegador é um lead qualificado que experimentou o produto sem atrito. O plano gratuito com colaboradores ilimitados não é generosidade — é aquisição de clientes com custo marginal zero.

### 8.3 Foco não é sobre o que você adiciona — é sobre o que você elimina

O Figma começou tentando ser Photoshop, gerador de memes, editor 3D e ferramenta de animação simultaneamente. O produto só encontrou tração quando Field cortou tudo e focou em uma única categoria. Essa disciplina de foco se manteve: o Figma nunca tentou ser editor de fotos, software de ilustração ou ferramenta de motion design. Ele faz design de interfaces e expande‑se para superfícies adjacentes (whiteboarding, slides, inspeção de código) a partir desse núcleo.

### 8.4 Componentes e design systems são uma âncora — use isso a seu favor

Uma empresa que publica sua biblioteca de componentes no Figma não consegue trocar de ferramenta sem perder anos de trabalho. O Figma entendeu isso e investiu pesadamente em Shared Libraries, Variants, branching e diffs de componentes. Cada melhoria nessa infraestrutura aumenta o custo de troca para o cliente. É um fosso competitivo construído sobre valor real entregue ao usuário — não sobre contratos ou lock‑in artificial.

### 8.5 A aquisição que não aconteceu foi melhor do que a aquisição que aconteceu

A Adobe ofereceu US$ 20 bilhões. Os reguladores disseram não. O Figma recebeu US$ 1 bilhão de multa, usou o dinheiro para acelerar desenvolvimento e abriu capital valendo US$ 56 bilhões. A lição não é "recuse aquisições" — é que o valuation de aquisição de um concorrente em pânico subestima o valor de uma plataforma que ainda está no início da sua curva de crescimento. A Adobe não estava comprando o Figma de 2022; estava tentando evitar o Figma de 2032.

---

## 9. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Figma |
| **Fundação** | 2012. Lançamento público: 27 de setembro de 2016. |
| **Fundadores** | Dylan Field (CEO), Evan Wallace (CTO) |
| **IPO** | 31 de julho de 2025 (NYSE: FIGM). Preço inicial: US$ 32. Fechamento dia 1: US$ 115,50. |
| **Valor de mercado** | US$ 56 bilhões (ações em circulação); ~US$ 65 bilhões (totalmente diluído) |
| **Categoria** | Design de Interfaces / Colaboração / Plataforma Criativa |
| **Plataformas** | Navegador (WebGL), Desktop (Electron), iOS, Android |
| **Usuários ativos mensais** | 13 milhões (2025) |
| **Usuários não‑designers** | ~67% do total |
| **Penetração Fortune 500** | 95% |
| **Receita anual (2025)** | ~US$ 749 milhões |
| **Retenção líquida de receita** | >150% |
| **Preços** | Starter (Grátis), Professional (US$ 12/editor/mês), Organization (US$ 45), Enterprise (US$ 75) |
| **Tech Stack** | C++ (core) → WebAssembly, WebGL (renderização), React/TypeScript (UI), Rust (servidor multiplayer) |
| **Design System** | Figma Sans (Grilli Type), 29 cores, Primitives, Variáveis de tema |
| **Funcionários** | ~1.400 (2025) |
| **Concorrentes** | Sketch, Adobe XD (descontinuado), Canva, Penpot (open‑source) |

---

## 10. Linha do Tempo Visual

```
Abr 2011 — Evan Wallace demonstra processamento de imagem via WebGL no lab de Brown.
2012 — Dylan Field recebe Thiel Fellowship (US$ 100K). Inscreve‑se com ideia de drones.
2012 — Field e Wallace abandonam drones. Começam a construir "ferramentas criativas no navegador".
2013 — Período de experimentação: editor de fotos, memes, 3D, animação. Nada ganha tração.
2013 — Field força decisão de foco. Cortam tudo exceto design de interfaces.
2013 — Seed de US$ 1,7M (Index Ventures, Jeff Weiner).
2015 — Sho Kuwamoto (ex‑Adobe/Macromedia) entra. Série A de US$ 14M (Greylock).
2016 Set — Lançamento público. Designers rejeitam. "Lento. Limitado."
2017 — Designers começam a migrar. KPCB lidera Série B.
2018 — Components. Primeira versão do sistema de design como primitiva de produto.
2019 — Comunidade Figma. Marketplace de plugins, templates, arquivos.
2020 — Variants. Auto Layout. O produto atinge maturidade funcional.
2021 — FigJam. Série E: valuation de US$ 10B.
2022 Set — Adobe anuncia aquisição por US$ 20B. Ações da Adobe caem.
2023 Dez — Aquisição bloqueada por reguladores. Adobe paga US$ 1B de multa.
2023 — Dev Mode. Primeira superfície para desenvolvedores.
2024 — Rebrand: Figma Sans, Primitives, 29 cores. Aquisição da Weavy (AI).
2024 — Figma Slides. Figma Sites (beta). OpenAI partnership.
2025 Jul 31 — IPO na NYSE. Ações fecham a US$ 115,50 (+250%). Market cap: US$ 56B.
2025 — Figma Make (AI). Buzz. MCP Server. WebGPU em migração.
```

---

## Fontes

As informações neste documento foram baseadas em:

- Figma Engineering Blog: ["Design Meet the Internet"](https://www.figma.com/blog/design-meet-the-internet/), ["WebAssembly cut Figma's load time by 3x"](https://www.figma.com/fr-fr/blog/webassembly-cut-figmas-load-time-by-3x/), ["How Figma draws inspiration from the gaming world"](https://www.figma.com/de/blog/how-figma-draws-inspiration-from-the-gaming-world/)
- Figma Blog: ["Figma on Figma: Our Latest Brand Refresh"](https://www.figma.com/blog/figma-on-figma-evolving-our-visual-language/), ["Just Our Type: The Story of Creating Figma Sans"](https://www.figma.com/blog/the-story-of-creating-figma-sans/), ["Updating Our Web System For A New Era"](https://www.figma.com/blog/updating-our-web-system-for-a-new-era/)
- Figma Brand Refresh coverage: [Yahoo Tech](https://tech.yahoo.com/general/articles/figmas-unapologetically-jumbo-identity-draws-130038130.html), [DesignTAXI](https://designtaxi.com/news/427148/Figma-Borrows-From-Playgrounds-To-Build-Its-Fun-Flexible-New-Visual-Identity/), [TYPE01](https://type-01.com/figma-rebrands-with-a-fresh-custom-font-by-grilli-type/)
- TechCrunch: ["Designers hated Figma at first, but grew to love it"](https://techcrunch.com/2023/06/25/figma-design-tool-history/)
- Quartr: ["Figma: Collaborative Design, Growth, and the Road After IPO"](https://quartr.com/insights/company-research/figma-collaborative-design-growth-and-the-road-after-IPO)
- Evan Wallace: ["How Figma's multiplayer technology works"](https://madebyevan.com/figma/how-figmas-multiplayer-technology-works/)
- Liveblocks: ["Understanding sync engines: How Figma, Linear, and Google Docs work"](https://liveblocks.io/blog/understanding-sync-engines-how-figma-linear-and-google-docs-work)
- VinVentures: ["Figma: A Decade of Revolution Leading to Tech's Biggest Design IPO"](https://www.vinventures.net/post/figma-a-decade-of-revolution-leading-to-tech-s-biggest-design-ipo)
- Ideaplan: ["How Figma Disrupted Adobe by Building for Collaboration"](https://www.ideaplan.io/case-studies/figma-collaborative-design)
- RBC Capital Markets: ["Redefining how organizations design, build and ship digital products"](https://www.rbccm.com/en/insights/2025/12/redefining-how-organizations-design-build)
- IPO coverage: Yahoo Finance, Quartz, multiple financial outlets (July 2025)
