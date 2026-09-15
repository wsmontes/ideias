# Relatorio de Pesquisa: Carga Cognitiva e Arquitetura da Informacao no Discord

## Uma analise aprofundada sob a otica da UX, ergonomia cognitiva e interacao humano-computador

---

## 1. Introducao: Por que o Discord e um Objeto de Estudo Unico

O Discord ocupa uma posicao singular no panorama das plataformas sociais. Nascido como ferramenta de comunicacao para gamers em 2015, evoluiu para um ecossistema que abriga comunidades de todos os tipos -- educacionais, profissionais, de hobbies, suporte tecnico e muito mais. Sua arquitetura, baseada em **servidores** como "casas" e **canais** como "comodos", oferece um laboratorio vivo para estudar como o design de interface afeta a cognicao dos usuarios.

Diferente de plataginas como Instagram ou TikTok, que priorizam feeds algoritmicos infinitos, o Discord adota um modelo **espacial e estruturado** que se alinha de forma quase intuitiva com a forma como o cerebro humano organiza informacoes. Esta pesquisa consolida descobertas academicas de diversas areas -- Psicologia Cognitiva, Interacao Humano-Computador (IHC), Ciencia da Informacao e Design de Experiencia do Usuario -- para entender por que essa arquitetura funciona (e onde ela falha).

---

## 2. Teoria da Carga Cognitiva Aplicada a Interfaces de Chat

### 2.1 Fundamentos: Sweller, Miller e Cowan

A **Teoria da Carga Cognitiva (Cognitive Load Theory -- CLT)** , desenvolvida por **John Sweller** em 1988, parte de uma constatacao fundamental: a memoria de trabalho humana e severamente limitada, enquanto a memoria de longo prazo e praticamente ilimitada. O design utilizavel e o processo de trabalhar dentro desse gargalo.

Tres tipos de carga cognitiva:

| Tipo | Definicao | Acao de Design |
|---|---|---|
| **Intrinseca** | Complexidade inerente a tarefa | Gerenciar -- sequenciar, fragmentar, dividir em etapas |
| **Extrinseca** | Esforco desperdicado por ma apresentacao | Eliminar -- poluicao visual, atencao dividida, redundancia |
| **Germana** | Esforco produtivo que constroi esquemas | Proteger orcamento -- liberar capacidade para aprendizado real |

Todas compartilham **um unico orcamento fixo** -- o tamanho da memoria de trabalho. Se a carga extrinseca e alta, nao sobra capacidade para o processamento germano.

**George Miller (1956)** sugeriu o classico numero magico de "7 ± 2" itens na memoria de curto prazo. Porem, **Nelson Cowan (2001)** refinou essa estimativa para **aproximadamente 4 blocos (chunks)** quando o ensaio mental e bloqueado -- numero hoje amplamente aceito na pesquisa contemporanea. Esses poucos itens decaem em cerca de **20 segundos** sem manutencao ativa.

**Fonte:** Cowan, N. (2001). The magical number 4 in short-term memory: A reconsideration of mental storage capacity. *Behavioral and Brain Sciences*, 24(1), 87-114. https://pubmed.ncbi.nlm.nih.gov/11515286/

### 2.2 O "Efeito Buraco de Fechadura" (Keyhole Effect)

Um artigo seminal de **Mohan Reddy (2025)** , "The Keyhole Effect: Why Chat Interfaces Fail at Data Analysis", oferece uma estrutura cognitiva que se aplica diretamente ao Discord. Reddy identifica **cinco mecanismos de falha** em interfaces puramente conversacionais:

1. **Deslocamento constante de conteudo** -- derrota os sistemas de memoria espacial do hipocampo (impossibilidade de formar mapas estaveis do tipo "onde esta aquilo?")
2. **Variaveis de estado ocultas** -- excedem a capacidade da memoria de trabalho (~4 blocos sob carga)
3. **Verbalizacao forcada** -- desencadeia ofuscamento verbal, degradando o reconhecimento visual de padroes
4. **Fluxos textuais lineares** -- bloqueiam acao epistemica e externalizacao cognitiva (ver e mais eficiente que lembrar)
5. **Penalidades de serializacao** -- dados multidimensionais forcados em texto 1D sofrem perda de informacao

Reddy formaliza a sobrecarga cognitiva como:

**O = max(0, m - v - W)**

Onde:
- **m** = itens relevantes a tarefa necessarios para raciocinar corretamente
- **v** = itens externamente visiveis (persistentes, consultaveis rapidamente)
- **W** = capacidade da memoria de trabalho (~4 blocos sob carga)

Quando **O > 0**, a probabilidade de erro aumenta e vieses analiticos (ancoragem, confirmacao, cegueira a mudancas) se amplificam.

**Por que o Discord e diferente:** Diferente de um chat linear como WhatsApp ou Messenger, o Discord oferece **navegacao espacial persistente** -- a coluna de servidores a esquerda, a lista de canais, o painel de membros. Essa estrutura fixa de 4 colunas funciona como um **trilho de estado** (state rail), mantendo visiveis as variaveis de navegacao e reduzindo drasticamente O na equacao de Reddy.

**Fonte:** Reddy, M. (2025). The Keyhole Effect: Why Chat Interfaces Fail at Data Analysis. arXiv:2602.00947. https://ar5iv.labs.arxiv.org/html/2602.00947

### 2.3 Aplicacao Direta ao Discord

O layout de 4 colunas do Discord (servidores > canais > conteudo > membros) e um exemplo pratico de **externalizacao cognitiva**. O usuario nao precisa manter na memoria de trabalho:

- Em que servidor esta (sempre visivel na primeira coluna)
- Que canais estao disponiveis (sempre visiveis na segunda coluna)
- Quem esta presente (sempre visivel na quarta coluna)

Isso libera capacidade da memoria de trabalho para o que realmente importa: o conteudo da conversa. Este principio e consistente com a pesquisa de **Zhang & Norman (1994)** sobre "representacoes externas" -- ferramentas cognitivas que reduzem a carga ao tornar informacoes persistentemente visiveis.

A pesquisa de **Ishii & Tobitani (2026)** , "Broadening Topic Breadth by Hiding the Latest Message in Text Chat", propos uma intervencao contra-intuitiva: **ocultar a mensagem mais recente** ate que o usuario envie sua resposta. Os resultados mostraram:

- Aumento significativo na **amplitude tematica percebida** (4.33 vs 3.22 em escala de 7 pontos)
- **Nenhum aumento na carga cognitiva subjetiva** -- o trabalho interpretativo foi experienciado como engajamento produtivo, nao como fardo

Isso sugere que o design do Discord, que propositalmente **segmenta conversas em canais** (criando "lacunas de informacao" intencionais), pode promover engajamento mais profundo em vez de sobrecarga.

**Fonte:** Ishii, Y. & Tobitani, K. (2026). Broadening Topic Breadth by Hiding the Latest Message in Text Chat. ACM CHI 2026. https://dl.acm.org/doi/10.1145/3772363.3798600

---

## 3. Modelos Mentais em Software Social: A Metafora Espacial

### 3.1 "Casa com Comodos" e a Cognicao Espacial

A metafora do Discord -- **servidores como casas, canais como comodos** -- nao e arbitraria. Ela explora o que os cientistas cognitivos chamam de **esquemas de imagem** (image schemas), estruturas pre-linguisticas derivadas da experiencia corporal no mundo fisico.

**Maglio & Matlock** demonstraram que mesmo usuarios novatos da web conceitualizam espacos de informacao como **espacos fisicos pelos quais se movem ativamente**. As pessoas **nao conseguem deixar de usar metaforas espaciais** para conceitualizar espacos informacionais.

**Fonte:** Maglio, P. & Matlock, T. (1999). Navigating in Information Spaces. In *Social Navigation of Information Space*. Springer.

**Hong et al. (2025)** , em "Prototyping Digital Social Spaces through Metaphor-Driven Design", descobriram que:

- Usuarios conceitualizam naturalmente espacos sociais digitais por meio de metaforas espaciais
- Distinguem consistentemente entre espacos **"abertos"** (energeticos, espontaneos, engajados) e espacos **"fechados"** (intimos, relaxados, de longa permanencia)
- Atributos como **tipos de mensagens, reacoes e identidade** foram os mais consistentemente alinhados com as expectativas dos usuarios
- A **autenticidade percebida** dependia de quao bem o fluxo de conteudo correspondia a metafora original

No Discord, isso se manifesta na distincao entre:
- **Canais de texto gerais** (espacos abertos para conversa fluida)
- **Canais de forum** (espacos semiestruturados para discussoes focadas)
- **Canais de voz** (espacos de copresenca sincrona)

**Fonte:** Hong, S. et al. (2025). Prototyping Digital Social Spaces through Metaphor-Driven Design. arXiv:2510.02759. https://browse-export.arxiv.org/pdf/2510.02759

### 3.2 Discord como "Terceiro Lugar" Digital

**Ray Oldenburg** (1989) introduziu o conceito de **"terceiros lugares"** (third places) em seu livro *The Great Good Place* -- espacos de reuniao informais e publicos, distintos do primeiro lugar (casa) e do segundo lugar (trabalho). As oito caracteristicas definidoras de Oldenburg:

1. **Terreno neutro** -- pessoas podem ir e vir livremente
2. **Nivelador** -- hierarquias sociais sao minimizadas
3. **Conversa como atividade principal**
4. **Acessibilidade e acomodacao**
5. **Frequentadores habituais** (regulars)
6. **Perfil baixo** -- sem pretensoes
7. **Tom ludico** -- descontraido e alegre
8. **Lar longe de casa**

**Kim et al. (2025)** , no artigo "Discord's Design Encourages Third Place Social Media Experiences", realizaram um estudo qualitativo com ~25 participantes e identificaram **21 elementos de design** no Discord que se alinham com as caracteristicas de Oldenburg. Achados principais:

- Usuarios descrevem espacos do Discord usando metaforas fisicas: **"praca central," "cafe," "sala de jogos"**
- A arquitetura de **servidor-e-canal** cria espacos tematicos para interacoes repetidas
- Recursos de **autonomia e personalizacao do usuario**, interacoes de baixa pressao e atividades compartilhadas promovem dinamicas de terceiro lugar
- O design espacial do Discord foca em **particionamento, presenca e conscientizacao** -- uma abordagem arquitetonica diferente de mundos virtuais 3D

**Fonte:** Kim, J. et al. (2025). Discord's Design Encourages "Third Place" Social Media Experiences. arXiv:2501.09951. https://export.arxiv.org/pdf/2501.09951

### 3.3 Participacao em Etapas (Tiered Engagement)

Um conceito crucial emergente da pesquisa e a **participacao em etapas** (tiered engagement). O Discord permite:

- **Lurking** (observacao silenciosa) -- sem pressao para contribuir
- **Peek-in** (espiar) -- entrar e sair sem compromisso
- **Join** (participar ativamente) -- comecar a contribuir quando confortavel

Essa estrutura reduz a **barreira de entrada cognitiva**, algo que a pesquisa sobre terceiros lugares digitais de **McArthur & White (2016)** , analisando mais de 3.100 tweets, identificou como critico para a vitalidade da comunidade.

**Fonte:** McArthur, J. A. & White, A. F. (2016). Twitter Chats as Third Places: Conceptualizing a Digital Gathering Site. *Social Media + Society*, 2(3). https://journals.sagepub.com/doi/10.1177/2056305116665857

---

## 4. Scent Information e Navegacao (Information Scent)

### 4.1 A Teoria do Forrageamento de Informacao

Desenvolvida por **Pirolli e Card (1999)** , a **Teoria do Forrageamento de Informacao (Information Foraging Theory -- IFT)** traca uma analogia entre animais forrageando por comida e usuarios buscando informacao em espacos digitais. Usuarios seguem **"scent information"** (pistas proximais como links, rotulos, cabecalhos) para alcancar metas informacionais distais, equilibrando o ganho de informacao contra o custo de acesso.

**Fonte:** Pirolli, P. & Card, S. (1999). Information foraging. *Psychological Review*, 106(4), 643-675.

### 4.2 Aplicacao ao Discord: O Cmd+K e a Hierarquia de Canais

O Discord implementa multiplos mecanismos de **scent information**:

1. **Hierarquia de servidores e canais** -- a arvore de navegacao a esquerda funciona como um mapa de scent: o nome do servidor, os icones, as cores dos canais, os emojis cabecalho. Cada elemento e uma pista sobre o que o usuario encontrara ali.

2. **Atalho Cmd+K (Quick Switcher)** -- implementa o que a pesquisa de IFT chama de **"search as navigation"** . O usuario pode "pular" diretamente para um canal, contato ou servidor conhecido, reduzindo o custo de navegacao. Isso corresponde ao que **ScentTrails** (um sistema de navegacao hibrido) demonstrou: integrar busca e navegacao reduz o custo de acesso a informacao.

3. **Canais de forum e tags** -- os forums do Discord implementam um sistema de **etiquetagem** que estende o scent information. Pesquisa da *Computers & Education* sobre sistemas de tagging social propoe um modelo de **"scent information estendido"** onde tanto estruturas de conhecimento individual quanto conhecimento coletivo (nuvens de tags, folksonomias) influenciam a selecao de links e o aprendizado incidental.

**Fonte:** Fu, W. (2012). The collective knowledge of social tags: Direct and indirect influences on navigation, learning, and information processing. *Computers & Education*, 60(1). https://www.sciencedirect.com/science/article/abs/pii/S036013151200200X

### 4.3 Navegacao Visual vs. Nao-Visual

**Upadhyay (2020)** , em pesquisa no CHI, descobriu que usuarios com visao normal adotam estrategias de consulta extensiva ou de seguimento extensivo de links, enquanto usuarios de leitores de tela otimizam de forma diferente: revisitam e reutilizam paginas de resultados, realizam varreduras abrangentes e formulam menos consultas. Isso tem implicacoes diretas para a acessibilidade do Discord: a navegacao por landmarks (6 regioes semanticas) e o suporte a ARIA sao cruciais para que usuarios cegos tambem possam se beneficiar do scent information.

**Fonte:** Upadhyay, S. (2020). Comparing Non-Visual and Visual Information Foraging on The Web. *ACM CHI 2020*. https://dl.acm.org/doi/fullHtml/10.1145/3334480.3383025

---

## 5. Paralisia de Decisao e Sobrecarga de Escolha

### 5.1 O Paradoxo da Escolha

**Barry Schwartz (2004)** , em *The Paradox of Choice: Why More Is Less*, argumentou que, enquanto alguma escolha e melhor que nenhuma, uma **abundancia excessiva de opcoes** pode levar a ansiedade, paralisia, insatisfacao e arrependimento. Dois tipos de decididores:

- **Maximizadores** -- buscam a "melhor opcao possivel". Tendem a ganhar mais, mas sao consistentemente menos felizes.
- **Satisfatores** (satisficers) -- aceitam "bom o suficiente". Sao consistentemente mais felizes e satisfeitos.

O Discord apresenta multiplos niveis de escolha:
- Quantos servidores participar?
- Quantos canais ler/participar?
- Quais conversas acompanhar?
- Quando silenciar ou sair?

**Fonte:** Schwartz, B. (2004). *The Paradox of Choice: Why More Is Less*. Ecco Press.

### 5.2 Limites Empiricos para Canais e Servidores

Pesquisa de **Reutskaja et al. (2018)** usando fMRI encontrou que, ao escolher entre 24 opcoes, a atividade cerebral no cortex cingulado anterior e no estriado **diminuiu marcadamente** em comparacao com 12 opcoes, indicando **desengajamento cognitivo**. O ponto ideal parece estar entre **8 e 15 opcoes**.

**Fonte:** Reutskaja, E. et al. (2018). Choice overload reduces neural signatures of choice quality. *Nature Communications*.

**Jones, Moldovan, Raban & Butler (CSCW 2008)** encontraram limites duros para interacao em grupo sincrono:
- Aproximadamente **40 postadores ativos por intervalo de 20 minutos** parece ser o teto para canais de chat sincrono (IRC), mesmo quando o total de usuarios cresce para centenas
- Isso e atribuido a **restricoes individuais de processamento de informacao** -- alem desse limite, a sobrecarga conversacional e a entropia da informacao tornam impossivel manter um discurso coerente

**Fonte:** Jones, Q. et al. (2008). Information Overload in Synchronous Chat. *ACM CSCW 2008*. https://dlnext.acm.org/doi/epdf/10.1145/1460563.1460616

### 5.3 Aplicacao ao Discord: Quantos Canais e Demais?

A pesquisa de design de comunidades sugere:

| Contexto | Faixa Otima | Limite Superior (ponto de sobrecarga) |
|---|---|---|
| Servidores pequenos/médios | 8-15 canais | ~25 canais |
| Servidores grandes | 20-120 canais (com categorias) | ~150+ requer governanca rigorosa |
| Canais de chat sincrono | ~40 participantes ativos | ~50 ativos |
| Opcoes de cargo (roles) | 5-10 por mensagem | >15 causa paralisia |

A pesquisa de pratica recomenda **comecar com 3 categorias e 8-15 canais** -- adicionar apenas quando um canal existente estiver consistentemente muito movimentado. Canais inativos apos 30 dias devem ser arquivados.

### 5.4 A Fadiga de Plataforma

**Fan et al. (2024)** , aplicando o modelo **Estimulo-Organismo-Resposta (SOR)** , descobriram que:

- A **sobrecarga de informacao** e a **sobrecarga de comunicacao** sao estimulos negativos diretos que levam a **fadiga online**, que por sua vez aumenta as **intencoes de troca de plataforma**
- Contudo, os **custos de troca** (investimentos de tempo, emocionais, financeiros) moderam negativamente a lacuna intencao-comportamento: intencoes fortes de troca nem sempre se traduzem em troca real se o custo percebido e alto
- Isso cria um estado de **"preso mas insatisfeito"** -- um paradoxo de retencao que as plataformas exploram

**Fonte:** Fan, W. et al. (2024). Consumers' platform switching behavior in social commerce. *Heliyon*. http://psasir.upm.edu.my/id/eprint/111524/

---

## 6. Divulgacao Progressiva (Progressive Disclosure)

### 6.1 O Conceito

**Divulgacao progressiva** e uma tecnica de UX que adia recursos e informacoes avancadas para componentes de interface secundarios, mantendo apenas o essencial na interface primaria. O objetivo e melhorar a usabilidade tanto para usuarios novatos quanto experientes, mostrando **o que eles precisam, quando precisam** (Interaction Design Foundation).

**Fonte:** Interaction Design Foundation. What is Progressive Disclosure? https://www.interaction-design.org/literature/topics/progressive-disclosure

### 6.2 Discord: Um Estudo de Caso em Divulgacao Progressiva

O Discord e um exemplo notavel de divulgacao progressiva em acao, funcionando em varios niveis:

**Nivel 1 -- Onboarding inicial:**
- O usuario ve uma interface complexa, mas as acoes iniciais sao simples
- O popup de "boas-vindas" oferece apenas algumas opcoes
- A barra de servidores mostra apenas alguns icones

**Nivel 2 -- Descoberta de recursos avancados:**
- Slash commands (/) sao descobertos gradualmente
- Menus de contexto do botao direito revelam funcionalidades ocultas
- Integracoes e bots aparecem conforme a necessidade

**Nivel 3 -- Recursos de comunidade:**
- Cargos e permissoes sao tipicamente configurados por admins, nao por membros comuns
- Canais de forum (introduzidos em 2022) representam um paradigma hibrido que o usuario descobre quando necessario
- Threads permitem desdobramentos temporarios sem criar canais permanentes

**Nivel 4 -- Personalizacao:**
- Temas (claro/escuro, cores de destaque)
- Modos de exibicao (compacto vs. sem espacamento)
- Configuracoes de notificacao (global > servidor > canal)

### 6.3 Eficacia da Divulgacao Progressiva

A pesquisa mostra que o **onboarding em etapas** supera significativamente o "tutorial-overload":

- **Duolingo** -- 3 perguntas iniciais (qual idioma, quanto tempo, comecar licao) alcanca **73% de conclusao** vs. 40% da media da industria
- O aplicativo medio perde **77% dos usuarios ativos diarios dentro de 3 dias** e ate **80% dos usuarios saem apenas devido ao mau onboarding**
- Mais de **30% das etapas de onboarding** em muitos produtos SaaS sao desnecessarias e poderiam ser removidas

**Fonte:** Digital Hill Multimedia. Why Simplicity Beats Features When Introducing Users to New Platforms. https://www.digitalhill.com/blog/why-simpler-onboarding-keeps-more-users-around/

### 6.4 Divulgacao Progressiva em Plataformas Sociais

A Interaction Design Foundation observa que aplicacoes sociais usam divulgacao progressiva para:
- Exibir inicialmente um feed simples e recursos basicos de postagem
- Revelar recursos extras (analises, atividades) mediante exploracao ou demanda
- Evitar sobrecarregar usuarios com opcoes que ainda nao precisam

Pesquisa IEEE analisou a experiencia de integracao em uma rede social ficticia onde os usuarios enfrentavam **17 escolhas de privacidade**. Achados:
- O **numero de telas e a abordagem de agrupamento** tiveram pouco impacto no sentimento/compreensao do usuario
- No entanto, **predefinicoes** influenciaram significativamente o comportamento
- A maioria dos participantes preferiu ver as escolhas divididas em varias telas em vez de uma tela longa

**Fonte:** IEEE (2025). Interface Design to Support Informed Choices When Users Face Numerous Privacy Decisions. https://ieeexplore.ieee.org/document/11095630

### 6.5 Consideracoes Culturais

Um estudo global de onboarding descobriu que a divulgacao progressiva funciona melhor em **mercados ocidentais** (America do Norte, Europa Ocidental, Australia) onde os usuarios valorizam velocidade e descobrem recursos ao longo do caminho. Ela **falha em culturas de alto contexto** como o Japao, onde os usuarios preferem prova social e veem o onboarding minimo como falta de confiabilidade.

---

## 7. Gerenciamento de Atencao e Design de Notificacoes

### 7.1 Atencao como Recurso Escasso

**Herbert Simon (1971)** estabeleceu o fundamento teorico: *"Uma riqueza de informacao cria uma pobreza de atencao."* Em um mundo rico em informacao, o recurso escasso nao e mais a informacao em si, mas a **atencao humana** -- a capacidade finita de processar essa informacao.

**Fonte:** Simon, H. A. (1971). Designing Organizations for an Information-Rich World. In *Computers, Communication, and the Public Interest*.

**Bruineberg (2025)** , em "Rethinking the Cognitive Foundations of the Attention Economy", examina criticamente as suposicoes de Simon:
- O modelo de "alocador centralizado" de Simon e desafiado pela ciencia cognitiva contemporanea, que sugere que a atencao emerge de **competicao distribuida** entre processos neurais
- A atencao como "recurso unificado e invariante" pode nao ter um referente estavel e singular

**Fonte:** Bruineberg, J. (2025). Rethinking the cognitive foundations of the attention economy. *Philosophical Psychology*. https://www.tandfonline.com/doi/full/10.1080/09515089.2025.2502428

### 7.2 A Hierarquia de Notificacoes do Discord

O Discord implementa uma **hierarquia sofisticada de notificacoes** que reflete principios de design de atencao baseados em pesquisa:

| Categoria | Prioridade | Mecanismo |
|---|---|---|
| Mensagens Diretas | Mais alta | Sempre enviadas a todos os dispositivos |
| @mencoes (tres niveis) | Alta/Media | @usuario > @cargo > @everyone/@here |
| Mensagens de servidor | Varia | Padrao muda por tamanho do servidor |
| Substituicoes por canal | Granular | Permitem silenciar por periodo (15min, 1h, 8h, 24h) |
| Eventos/Stage/Threads | Separado | Logica propria para novos tipos de conteudo |

### 7.3 Inteligencia Entre Dispositivos

Um detalhe de UX particularmente relevante: o Discord usa **supressao por atividade no desktop**: se voce esta usando ativamente o Discord no desktop, as notificacoes push no celular sao atrasadas ou suprimidas. Isso evita a "fadiga do duplo toque" -- um detalhe critico para reduzir a sobrecarga de notificacoes.

**Arvore de decisao de supressao do Discord** (antes de qualquer push ser disparado):
1. Usuario em Nao Perturbe? => Suprimir
2. E horario de silencio? => Suprimir
3. Usuario ativo no desktop? => Entrega apenas no desktop
4. Servidor/canal silenciado? => Suprimir
5. Aplicar substituicao de notificacao por canal
6. Verificar prioridade do tipo de evento
7. Mensagem ja foi lida em outro dispositivo? => Suprimir

### 7.4 O Modelo de "Agente de Notificacao Intencional"

Pesquisa academica comparou tres abordagens de gerenciamento de atencao:

| Abordagem | Descricao | Eficacia |
|---|---|---|
| **Apenas registro** | Gravacao passiva de intencoes | Baixa adocao, 68% de desconfianca |
| **Lembrete simples** | Notificacoes de lembrete estaticas | Eficacia moderada, facilmente ignorado |
| **INA (Agente Interativo de Notificacao)** | Notificacoes adaptativas e sensiveis ao contexto | **86% positivo** para realizacao de tarefas; **64%** para reducao de distracao |

**Fonte:** arXiv:2510.14513. Intentional Notification Agents. https://arxiv.org/pdf/2510.14513v1

### 7.5 Ciencia da Interrupcao

**Talypova et al. (2023)** , em estudo com 163 participantes, descobriram que:

- Usuarios querem que sistemas de gerenciamento de atencao **adiem interrupcoes para limites de tarefa ou momentos de baixa carga mental**
- Achado chave: **o gerenciamento de interrupcoes e desejado, desde que os usuarios sintam que estao no controle**
- Recursos recomendados: opcoes de substituicao de emergencia, avaliacao de urgencia baseada em contexto e auxilios de retomada

**Fonte:** Talypova, D. et al. (2023). User-Centered Investigation of Features for Attention Management Systems. *ACM MUM '23*. https://dl.acm.org/doi/fullHtml/10.1145/3626705.3627766

**Bolton et al. (2022)** testaram se pre-visualizacoes de notificacoes reduzem custos de interrupcao:
- Pre-visualizacoes **reduziram a carga mental subjetiva** (NASA-TLX)
- Mas **nao melhoraram o desempenho objetivo da tarefa**
- O efeito foi consistente em todos os niveis de carga de tarefa

**Fonte:** Bolton, T. et al. (2022). Can Previews Mitigate the Effect of Interruptions? *International Journal of Human-Computer Interaction*. https://www.tandfonline.com/eprint/V4FMXHISMZXFZEQMQJDC/full

**Wang (2025)** , em *Frontiers in Psychology* (N=240), examinou como a complexidade do design instrucional (CDI) e a interferencia de notificacoes pop-up (INP) interagem em salas de aula virtuais:
- Tanto CDI quanto INP prejudicam independentemente a atencao e a retencao
- **Combinados, produzem custos agravados** (partial eta² ~ 0.03)
- **Moderacao**: aprendizes com maior experiencia digital sao menos afetados por INP

**Fonte:** Wang (2025). Instructional Design Complexity and Notification Interference. *Frontiers in Psychology*. https://www.frontiersin.org/journals/psychology/articles/10.3389/fpsyg.2025.1618121

### 7.6 Perigo das Interrupcoes Emocionais

Uma pesquisa de 2026 no *International Journal of Human-Computer Studies* (N=18) usou rastreamento ocular (gaze + paragaze: tamanho da pupila, piscadas) durante tarefas de roleplay interrompidas:
- **Alta carga cognitiva** -> impacto prejudicial no tempo de conclusao
- **Baixa carga cognitiva** -> reinicio do tempo para tarefas primarias
- **Interrupcoes emocionais** (alta excitacao/valencia) alteraram o comportamento ocular no retorno, mostrando menos assimilacao de informacao visual sob alta carga
- Primeira evidencia experimental ligando **interrupcao afetiva** a mudancas na carga cognitiva

**Fonte:** IJHCS (2026). The eyes show it: Exploring eye behavior and impact on human mental state during task interruption. https://www.sciencedirect.com/science/article/pii/S1071581926000479

---

## 8. Acessibilidade e Inclusao

### 8.1 O Panorama Geral

Dados da OMS indicam que mais de **1 bilhao de pessoas** (aproximadamente **15% da populacao mundial**) vivem com alguma forma de deficiencia. Estudos de UX sugerem que **25-30%+ dos usuarios de internet** teem algum tipo de deficiencia -- visual, auditiva, cognitiva ou motora. Ignorar a acessibilidade nao e apenas uma falha etica, mas uma exclusao de uma parcela massiva da base de usuarios.

### 8.2 Recursos de Acessibilidade do Discord

| Deficiencia | Recursos Disponiveis |
|---|---|
| **Visual** | Compatibilidade com leitores de tela (NVDA, JAWS, VoiceOver, TalkBack); Modo de Alto Contraste; Role Colors; Suporte a High Contrast Mode do Windows; Texto alternativo (alt text) para imagens no mobile |
| **Auditiva** | Legendas ocultas (closed captions) em streams e videos; Indicadores visuais de audio (mudo/surdo) |
| **Cognitiva** | Modo para epilepsia (remove animacoes e blinking); Navegacao por landmarks e headings; Atalhos de teclado; Modo compacto para reduzir poluicao visual |

### 8.3 A Parceria com o American Council of the Blind (2021)

O Discord firmou uma parceria com o **American Council of the Blind (ACB)** para adotar as **WCAG 2.1 nivel AA** como padrao de acessibilidade digital. A empresa se comprometeu a trabalhar diretamente com a comunidade de pessoas cegas para melhorar a usabilidade.

**Fonte:** American Council of the Blind. Discord and ACB Announce Digital Accessibility Initiative. https://www.acb.org/discord-and-american-council-blind-announce-digital-accessibility-initiative

### 8.4 Navegacao por Leitores de Tela: 6 Landmarks Semanticas

A interface do Discord e dividida em **6 regioes (landmarks)**, conforme guia de uso com NVDA:

1. **Server Sidebar** -- arvore de servidores (navegacao entre "casas")
2. **Private/Server Channels** -- canais de texto/voz ("comodos")
3. **User Area** -- status, microfone, alto-falantes
4. **Channel Header** -- cabecalho contextual
5. **Channel Area** -- area principal de mensagens
6. **Members List** -- lista de membros

Esta estrutura de landmarks e consistente com as recomendacoes do WAI-ARIA para navegacao eficiente por teclado e leitores de tela.

**Fonte:** GitHub - PepperTheVixen/Discord-With-NVDA. Guia para uso do Discord com NVDA no Windows. https://github.com/PepperTheVixen/Discord-With-NVDA

### 8.5 Desafios Persistentes

Apesar dos avancos, a pesquisa de acessibilidade do Discord (Django Community Working Group, GitHub) aponta desafios:
- Alguns pop-ups e menus ainda sao de dificil acesso para leitores de tela
- A descoberta de funcionalidades de acessibilidade poderia ser melhor comunicada na interface
- A navegacao em servers muito grandes (150+ canais) continua sendo um desafio cognitivo para usuarios de leitores de tela

**Fonte:** GitHub - django/online-community-working-group. Accessibility review of the Django Discord. https://github.com/django/online-community-working-group/issues/25

---

## 9. Sintese: Implicacoes de Design

### 9.1 O Que o Discord Acerta

| Principio | Evidencia | Implementacao no Discord |
|---|---|---|
| **Externalizacao cognitiva** | Teoria da Carga Cognitiva (Sweller), Efeito Buraco de Fechadura (Reddy) | Layout fixo de 4 colunas que mantém estado de navegacao sempre visivel |
| **Metafora espacial** | Esquemas de imagem (Maglio & Matlock), Design guiado por metafora (Hong et al.) | Servidores como casas, canais como comodos |
| **Segmentacao de canais** | Sobrecarga de informacao em chat sincrono (Jones et al.) | Canais separados reduzem overflow cognitivo |
| **Participacao em etapas** | Terceiro lugar digital (Kim et al.) | Lurking, peek-in e join como modos de engajamento |
| **Hierarquia de notificacoes** | Ciencia da interrupcao (Talypova et al.), Fadiga de notificacao | Supressao por desktop, silenciamento granular, horarios de silencio |
| **Divulgacao progressiva** | CLT, pesquisa de onboarding (Interaction Design Foundation) | Recursos revelados gradualmente (slash commands, threads, forums) |

### 9.2 O Que Pode Melhorar

| Problema | Evidencia | Sugestao de Design |
|---|---|---|
| **Sobrecarga em servers grandes** | Paralisia de escolha (Schwartz), 15+ canais ideais | Onboarding com exposicao gradual de canais por roles |
| **Acessibilidade de leitores de tela** | Pesquisa de forrageamento nao-visual (Upadhyay) | Melhor comunicacao de recursos de acessibilidade; melhor ordenacao por landmarks |
| **Canais de forum vs. chat** | Estudo de usabilidade (Massey University) | Sinalizacao mais clara de quando usar cada tipo de canal |
| **Fadiga multiplataforma** | Fadiga online (Fan et al.) | Sistema de "modo foco" que consolide notificacoes |
| **Busca e descoberta** | Scent Information (Pirolli & Card) | Melhor indexacao de conversas antigas; descoberta de canais por interesse |

### 9.3 Limites Cognitivos: Um Resumo Pratico

| Aspecto | Limite Empirico | Referencia |
|---|---|---|
| Memoria de trabalho | ~4 blocos (chunks) | Cowan (2001) |
| Opcoes em uma escolha | 8-15 (ideal) | Reutskaja et al. (2018), Schwartz (2004) |
| Participantes ativos em chat sincrono | ~40 por intervalo de 20min | Jones et al. (2008) |
| Canais em servidor novo | 8-15 (inicial) | Pratica recomenda |
| Cargos por mensagem de reaction role | 5-10 | Pratica recomenda |
| Notificacoes viaveis sem fadiga | Dependente de controle granular | Talypova et al. (2023) |

---

## 10. Referencias Citadas

1. Almomani, A. (2024). Discord Affordances and Institutional Logics. PhD Thesis, University of Edinburgh. https://era.ed.ac.uk/bitstream/handle/1842/42160/Almomani2024.pdf

2. Bolton, T. et al. (2022). Can Previews Mitigate the Effect of Interruptions? *International Journal of Human-Computer Interaction*. https://www.tandfonline.com/eprint/V4FMXHISMZXFZEQMQJDC/full

3. Bruineberg, J. (2025). Rethinking the cognitive foundations of the attention economy. *Philosophical Psychology*. https://www.tandfonline.com/doi/full/10.1080/09515089.2025.2502428

4. Cowan, N. (2001). The magical number 4 in short-term memory. *Behavioral and Brain Sciences*, 24(1), 87-114. https://pubmed.ncbi.nlm.nih.gov/11515286/

5. Fan, W. et al. (2024). Consumers' platform switching behavior in social commerce. *Heliyon*. http://psasir.upm.edu.my/id/eprint/111524/

6. Fu, W. (2012). The collective knowledge of social tags. *Computers & Education*, 60(1). https://www.sciencedirect.com/science/article/abs/pii/S036013151200200X

7. Hong, S. et al. (2025). Prototyping Digital Social Spaces through Metaphor-Driven Design. arXiv:2510.02759. https://browse-export.arxiv.org/pdf/2510.02759

8. Interaction Design Foundation. What is Progressive Disclosure? https://www.interaction-design.org/literature/topics/progressive-disclosure

9. Ishii, Y. & Tobitani, K. (2026). Broadening Topic Breadth by Hiding the Latest Message in Text Chat. ACM CHI 2026. https://dl.acm.org/doi/10.1145/3772363.3798600

10. Jones, Q. et al. (2008). Information Overload in Synchronous Chat. *ACM CSCW 2008*. https://dlnext.acm.org/doi/epdf/10.1145/1460563.1460616

11. Kim, J. et al. (2025). Discord's Design Encourages "Third Place" Social Media Experiences. arXiv:2501.09951. https://export.arxiv.org/pdf/2501.09951

12. Maglio, P. & Matlock, T. (1999). Navigating in Information Spaces. In *Social Navigation of Information Space*. Springer.

13. McArthur, J. A. & White, A. F. (2016). Twitter Chats as Third Places. *Social Media + Society*, 2(3). https://journals.sagepub.com/doi/10.1177/2056305116665857

14. Oldenburg, R. (1989). *The Great Good Place: Cafes, Coffee Shops, Community Centers, Beauty Parlors, General Stores, Bars, Hangouts, and How They Get You Through the Day*. Paragon House.

15. Pirolli, P. & Card, S. (1999). Information foraging. *Psychological Review*, 106(4), 643-675.

16. Reddy, M. (2025). The Keyhole Effect: Why Chat Interfaces Fail at Data Analysis. arXiv:2602.00947. https://ar5iv.labs.arxiv.org/html/2602.00947

17. Reutskaja, E. et al. (2018). Choice overload reduces neural signatures of choice quality. *Nature Communications*.

18. Schwartz, B. (2004). *The Paradox of Choice: Why More Is Less*. Ecco Press.

19. Simon, H. A. (1971). Designing Organizations for an Information-Rich World. In *Computers, Communication, and the Public Interest*.

20. Talypova, D. et al. (2023). User-Centered Investigation of Features for Attention Management Systems. *ACM MUM '23*. https://dl.acm.org/doi/fullHtml/10.1145/3626705.3627766

21. Upadhyay, S. (2020). Comparing Non-Visual and Visual Information Foraging on The Web. *ACM CHI 2020*. https://dl.acm.org/doi/fullHtml/10.1145/3334480.3383025

22. Wang (2025). Instructional Design Complexity and Notification Interference. *Frontiers in Psychology*. https://www.frontiersin.org/journals/psychology/articles/10.3389/fpsyg.2025.1618121

23. IJHCS (2026). The eyes show it: Exploring eye behavior and impact on human mental state during task interruption. https://www.sciencedirect.com/science/article/pii/S1071581926000479

24. American Council of the Blind. Discord and ACB Announce Digital Accessibility Initiative. https://www.acb.org/discord-and-american-council-blind-announce-digital-accessibility-initiative

25. PepperTheVixen. Discord-With-NVDA: Guia para uso do Discord com NVDA. GitHub. https://github.com/PepperTheVixen/Discord-With-NVDA

26. Django Community. Accessibility review of the Django Discord. GitHub. https://github.com/django/online-community-working-group/issues/25

27. arXiv:2510.14513. Intentional Notification Agents. https://arxiv.org/pdf/2510.14513v1

28. Digital Hill Multimedia. Why Simplicity Beats Features When Introducing Users to New Platforms. https://www.digitalhill.com/blog/why-simpler-onboarding-keeps-more-users-around/

29. IEEE (2025). Interface Design to Support Informed Choices When Users Face Numerous Privacy Decisions. https://ieeexplore.ieee.org/document/11095630

30. Yang et al. (2023). Usability of Chat and Forum Discussion Tools in Higher Education. OzCHI 2023. https://dlnext.acm.org/doi/fullHtml/10.1145/3638380.3638392

---

## Nota Metodologica

Esta pesquisa foi realizada entre junho e julho de 2026, utilizando WebSearch para varredura de literatura academica em bases como ACM Digital Library, arXiv, ScienceDirect, IEEE Xplore, Semantic Scholar, PubMed, Taylor & Francis, Google Scholar e Frontiers. Foram priorizados artigos revisados por pares (conferencias e periodicos) e pre-prints de alta relevancia. Os top 30 resultados foram consolidados neste relatorio.

A analise e transversal a seis areas: Teoria da Carga Cognitiva (Sweller, Cowan), Modelos Mentais e Metaforas Espaciais (Maglio & Matlock, Hong et al.), Teoria do Terceiro Lugar (Oldenburg, Kim et al.), Teoria do Forrageamento de Informacao (Pirolli & Card), Paradoxo da Escolha (Schwartz) e Ciencia da Interrupcao (Simon, Talypova et al.).

---

*Documento gerado em 3 de julho de 2026.*
