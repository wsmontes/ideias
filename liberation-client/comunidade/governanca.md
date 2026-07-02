# Modelo de Curadoria e Governanca Comunitária
## Liberation Client -- Curadoria Humana, Escala Comunitária

> **Documento original em portugues.**  
> English translation available at `../community/governance.md`  
> *"Devagar e sempre, com calor humano."*

---

## Indice

1. [Filosofia de Curadoria](#1-filosofia-de-curadoria)
2. [Sistema de Curadoria de Starter Packs](#2-sistema-de-curadoria-de-starter-packs)
3. [Sistema de Curadoria do Tasting Menu](#3-sistema-de-curadoria-do-tasting-menu)
4. [Estrutura de Governanca Comunitária](#4-estrutura-de-governanca-comunitaria)
5. [Infraestrutura Tecnica (Minima)](#5-infraestrutura-tecnica-minima)
6. [Inicializacao da Comunidade (Bootstrapping)](#6-inicializacao-da-comunidade-bootstrapping)
7. [A Voz Brasileira](#7-a-voz-brasileira)

---

## 1. Filosofia de Curadoria

### 1.1 O que e curadoria humana?

Curadoria humana no Liberation Client **nao e algoritmo**. Nao e otimizacao de engajamento. Nao e "o que viraliza". E **selecao intencional** feita por pessoas, para pessoas, com criterios explicitos e transparentes.

Curadoria humana e:

- **Um ato de cuidado** -- cada pack e cada menu sao montados pensando em quem esta chegando.
- **Um servico a comunidade** -- curators dedicam tempo para que outros encontrem o que e relevante sem serem manipulados por metricas de atencao.
- **Uma ponte entre mundos** -- curators conectam protocolos, linguas e topicos que normalmente nao se encontram.
- **Um exercicio de diversidade** -- a curadoria intencionalmente busca o que esta fora do centro, nao apenas o popular.

### 1.2 Escala humana

| Dimensao | Numero |
|----------|--------|
| Curadores ativos simultaneos (target) | 30-50 |
| Packs por curador (max recomendado) | 5 ativos |
| Posts selecionados por curador/semana | 20 (para o Tasting Menu) |
| Nominacoes por pessoa/semana | 3 max |
| Curadores do Tasting Menu por semana | 1-2 |

Nao queremos escalar para milhares de curadores. Queremos uma comunidade de curadores qualificados, rotativos, com diversidade de perspectivas.

### 1.3 O que faz um "bom" Starter Pack?

Um bom Starter Pack e:

- **Tematicamente coerente** -- as 15-30 contas tem uma relacao clara entre si (topic, regiao, lingua, vibe).
- **Acolhedor para iniciantes** -- inclui contas que introduzem o tema, nao apenas especialistas.
- **Diverso internamente** -- dentro do tema, busca variacao de perspectivas, nao apenas o consenso.
- **Atualizado** -- contas ativas nos ultimos 30 dias, atualizado a cada 90 dias.
- **Justificado** -- titulo e descricao explicam por que este pack existe e para quem e.

Um pack **ruim** e: lista de contas aleatorias sem conexao, spam, auto-promocao, contas mortas, ou apenas os seguidores de uma pessoa.

### 1.4 O que faz uma "boa" postagem no Tasting Menu?

Uma postagem no Tasting Menu deve atender a **pelo menos um** destes criterios:

- **Informativa** -- ensina algo novo, explica um conceito, traduz conhecimento.
- **Bela** -- visualmente impressionante (fotografia, arte, design, video).
- **Instigante** -- provoca reflexao, debate construtivo, nova perspectiva.
- **Representativa** -- captura um momento cultural, um movimento, uma comunidade.

E **nunca** deve ser: desinformacao, discurso de odio, auto-promocao explicita, clickbait, ou conteudo gerado por IA sem atribuicao.

### 1.5 Diversidade por desenho

A diversidade nao e acidental -- e **exigencia de curadoria**:

- **Topicos:** minimo 5 topicos diferentes por Tasting Menu.
- **Linguas:** minimo 3 linguas por Tasting Menu.
- **Protocolos:** minimo 3 protocolos diferentes por Tasting Menu.
- **Regioes:** packs e menus devem representar o Sul Global proporcionalmente.
- **Perspectivas:** curadores sao orientados a buscar vozes marginalizadas dentro de cada tema.

### 1.6 Transparencia radical

Cada pack e cada Tasting Menu exibe obrigatoriamente:

- Nome do curador (vinculado a identidade em protocolo)
- Data de criacao e da ultima atualizacao
- Criterios usados na selecao
- Numero de pessoas que seguiram o pack (anonimizado)
- Historico de versoes (para packs atualizados)

### 1.7 Responsabilidade rotativa

Nao existe uma classe permanente de curadores:

- Curadores do Tasting Menu sao selecionados semanalmente e so podem se candidatar novamente apos 4 semanas.
- Packs tem validade de 90 dias -- curadores precisam revalidar ou o pack e marcado como "nao mantido".
- Senioridade e reconhecimento, nao poder permanente.
- O sistema e desenhado para que **novas pessoas entrem continuamente** no circuito de curadoria.

---

## 2. Sistema de Curadoria de Starter Packs

### 2.1 Ciclo de Vida do Pack

```
Criacao → Revisao → Publicacao → Manutencao → (Atualizacao ou Depreciacao)
```

#### 2.1.1 Criacao

**Qualquer pessoa pode criar um pack**, desde que atenda aos requisitos minimos:

| Requisito | Detalhe |
|-----------|---------|
| Titulo | Obrigatorio, max 100 caracteres |
| Descricao | Obrigatoria, max 500 caracteres -- explica o proposito do pack |
| Tags de categoria | 1-3 obrigatorias (ex: "tecnologia", "cultura-brasileira", "fotografia") |
| Tag de lingua | Obrigatoria (ex: "pt", "en", "es", "ja") |
| Contas | 15-30 obrigatorias, todas ativas nos ultimos 30 dias |
| Feeds customizados | Opcional |
| Hashtags | Opcional, max 5 |

**Modelo "support-first"** (inspirado no Celo Engage Hub):  
Para criar seu primeiro pack, voce deve ter **interagido com 3 packs existentes** -- usado, avaliado ou revisado. Isso garante que novos curadores conhecem o sistema antes de contribuir.

**Anti-spam:**
- Conta no Liberation Client precisa ter no minimo 7 dias de idade.
- Conta vinculada a pelo menos 1 protocolo verificado (NIP-05, DID, ou dominio Mastodon).
- Limite de 1 pack novo por dia por pessoa.

#### 2.1.2 Revisao

**Modelo de auto-publicacao com sinalizacao comunitária:**

- Packs sao **auto-publicados** imediatamente apos a criacao -- nao ha fila de aprovacao central.
- A comunidade pode sinalizar packs por: spam, contas mortas (>50%), enganoso, conteudo improprio.
- **Sinalizacoes sao publicas** -- qualquer pessoa ve quem sinalizou e o motivo.
- Um pack com 3+ sinalizacoes em 24h entra em **revisao comunitária**:
  - 3 revisores aleatorios (Contributors+)
  - Decisao em 48h: manter, suspender, ou remover.
  - Revisores sao anonimos entre si mas identificados publicamente apos a decisao.

#### 2.1.3 Publicacao

Apos criacao, o pack e publicado imediatamente e aparece em:
- Busca por categoria, lingua, protocolo, curador
- "Packs novos" (filtro de recencia)
- Perfil do curador

#### 2.1.4 Manutencao

- Packs expiram apos **90 dias sem atualizacao**.
- 7 dias antes do vencimento, o curador recebe notificacao para revalidar.
- Curador pode: atualizar o pack (remover contas inativas, adicionar novas) ou marcar como "nao mantido".
- Packs nao mantidos ainda aparecem em buscas, mas com indicador claro de "desatualizado".

#### 2.1.5 Depreciacao

Um pack pode ser depreciado por:

| Motivo | Quem inicia | Processo |
|--------|-------------|----------|
| Contas mortas (>50%) | Sistema ou comunidade | Notifica curador; 14 dias para atualizar |
| Spam | Comunidade (3 sinalizacoes) | Revisao comunitária em 48h |
| Enganoso (titulo/descricao nao correspondem) | Comunidade | Revisao comunitária |
| Abandonado (sem atualizacao por 90+ dias) | Sistema | Marca como "desatualizado" |
| Violacao de conduta | Steward | Remocao imediata + registro publico |

### 2.2 Sinais de Qualidade do Pack

Nao usamos notas numericas publicas ou "rating". Usamos **sinais qualitativos**:

| Sinal | Descricao | Privacidade |
|-------|-----------|-------------|
| Uso | Numero de pessoas que seguiram o pack | Anonimizado (apenas total) |
| Retencao | Quantas continuam seguindo apos 30 dias | Anonimizado |
| Diversidade | Distribuicao de topicos, linguas, protocolos | Calculo local, agregado |
| Reputacao do curador | Baseada na qualidade dos packs do curador ao longo do tempo | So o curador ve seu proprio score |
| "Este pack me ajudou" | Botao opt-in, local, anonimizado | Nao sai do dispositivo |
| Comentarios | Revisoes textuais opcionais | Publicas com nome |

**Nao ha "nota media" exibida publicamente.** O objetivo e evitar gaming do sistema e competicao toxica. Os sinais sao usados internamente para ordenacao e recomendacao.

### 2.3 Descoberta de Packs

#### Navegacao

- Por **categoria** (tecnologia, arte, musica, noticias, etc.)
- Por **lingua**
- Por **protocolo** (Nostr, AT Protocol, ActivityPub, etc.)
- Por **curador**
- Por **popularidade** (numero de seguidores, anonimizado)
- Por **recencia**

#### Busca

- Por **palavra-chave** no titulo e descricao
- Por **topico** (tags)
- Por **handle de conta** (encontre packs que incluem uma conta especifica)

#### Recomendacao Social (sem ML)

- **"Packs que podem ser uteis para voce"**: baseado em packs seguidos por pessoas que voce segue. Nao ha modelo de machine learning -- e puramente prova social: "3 pessoas que voce segue usam este pack."
- **"Curadores em que voce pode confiar"**: curadores cujos packs foram seguidos por pessoas que voce segue ou por packs que voce ja usou.

Toda recomendacao social e **local** -- calculada no dispositivo do usuario, sem envio de dados para servidor central.

---

## 3. Sistema de Curadoria do Tasting Menu

### 3.1 Ciclo Semanal

```
Segunda-Terca:  Chamada de nominacoes
Quarta:         Curadoria (selecao dos 20 posts)
Quinta (00:00 UTC): Publicacao
Quarta seguinte: Reflexao e feedback
```

#### 3.1.1 Chamada de Nominacoes (Segunda a Terca)

Qualquer pessoa com papel **Contributor** ou superior pode nominar posts.

**Regras:**

- Qualquer post de qualquer protocolo conectado.
- Deve incluir justificativa de 1-2 frases explicando por que o post merece destaque.
- Limite de **3 nominacoes por pessoa por semana**.
- Post deve ser dos **ultimos 14 dias**.
- Nao pode nominar postagem propria.

**Nominacoes sao publicas** -- todos veem quem nominou o que e o motivo.

#### 3.1.2 Curadoria (Quarta)

O(s) curador(es) da semana selecionam **exatamente 20 posts** das nominacoes recebidas.

**Requisitos obrigatorios:**

| Requisito | Minimo |
|-----------|--------|
| Protocolos diferentes | 3 |
| Linguas diferentes | 3 |
| Topicos diferentes | 5 |
| Posts do Sul Global | 5 |
| Freshness | Posts dos ultimos 14 dias |

**Criterios de qualidade** (cada post selecionado deve atender a pelo menos um):

- **Informativo** -- ensina algo novo.
- **Belissimo** -- impacto visual.
- **Instigante** -- provoca reflexao ou debate construtivo.
- **Representativo** -- captura um momento cultural ou comunitario.

**Regras estritas:**

- Curador **nao pode selecionar postagem propria**.
- Curador **nao pode selecionar postagem de parente, parceire, ou pessoa com quem tenha relacao profissional contratual**.
- Curador **deve declarar conflitos de interesse** antes de iniciar a curadoria.
- Se um curador conhece pessoalmente quem nominou, deve declarar e o post pode ser mantido, mas isso fica registrado.

#### 3.1.3 Publicacao (Quinta, 00:00 UTC)

O Tasting Menu e publicado como um conjunto de 20 posts com:

- Post embeddado ou link para o post original
- Justificativa do curador para cada selecao (1-2 frases)
- Nome do curador e data
- Link para as nominacoes completas (transparencia)

O menu aparece em uma aba dedicada no app e e distribuido via evento nostr.

#### 3.1.4 Reflexao (Quarta Seguinte)

Antes do novo ciclo comecar:

- **Enquete comunitária**: "Qual post desta semana mais resonou com voce?" (opt-in, local).
- **O curador publica uma breve reflexao** (max 500 palavras) sobre suas escolhas, desafios de diversidade, tendencias que observou.
- A reflexao e armazenada juntamente ao menu como metadado.

### 3.2 Selecao de Curadores

#### 3.2.1 Modelo Rotativo

- **1-2 curadores por semana.**
- Mandato de **1 semana**.
- Pode se candidatar novamente apos **4 semanas de intervalo**.

#### 3.2.2 Elegibilidade

Para se candidatar, a pessoa deve:

- Ser usuaria ativa ha **30+ dias**.
- Ter **2+ protocolos conectados**.
- Ter submetido **5+ nominacoes de qualidade** (nao rejeitadas por spam ou baixa qualidade).
- Nao estar em suspensao ou processo disciplinar.

#### 3.2.3 Processo de Selecao

**Duas faixas:**

1. **Sorteio comunitário** (80% das vagas):  
   Entre todos os elegiveis que se candidataram, um sorteio aleatorio seleciona o curador da semana. O sorteio usa uma semente publica (hash do block do Bitcoin ou Nostr event aleatorio) para garantir imparcialidade.

2. **Curadoria Convidada** (20% das vagas):  
   Stewards podem convidar curadores especificos para trazer diversidade de perspectiva (ex: alguem de uma regiao sub-representada, especialista em um topico especifico).

#### 3.2.4 Mentorias

**Primeira vez como curador:**

- Curador de primeira viagem e **sempre pareado com um Senior Curator** como mentor.
- Mentor revisa a selecao antes da publicacao e da feedback.
- Mentor nao tem poder de veto -- apenas orientacao.
- Curador de primeira viagem pode optar por co-curadoria com o mentor (os 20 posts sao selecionados em conjunto).

### 3.3 Medidas Anti-Gaming

| Ameaca | Mitigacao |
|--------|-----------|
| **Brigading** (grupo coordena nominacoes para dominar o menu) | Nominacao requer conta com 30+ dias e atividade recente. Curadores tem liberdade para ignorar nominacoes coordenadas. |
| **Sybil attack** (multiplas contas falsas) | Identidade vinculada a protocolo verificado (NIP-05, DID, Mastodon verificado). Contas sem verificacao tem peso menor em eleicoes. |
| **Conflito de interesse** | Curador nao pode selecionar posts proprios ou de pessoas com quem tem relacao. Declaracao obrigatoria pre-curadoria. |
| **Captura do menu** (mesmos curadores sempre) | Sorteio aleatorio + intervalo de 4 semanas + limite de 1 semana seguida. |
| **Spam de nominacoes** | Limite de 3 por pessoa/semana. Nominacoes de baixa qualidade reduzem elegibilidade futura. |

### 3.4 Transparencia

Tudo e publico e auditavel:

- Todas as nominacoes (quem, o que, quando, por que)
- Todas as selecoes (quem selecionou, quais posts, por que)
- Identidade do curador (vinculada a protocolo)
- Decisoes de conflito de interesse
- Reflexao pos-publicacao
- Historico completo de menus

---

## 4. Estrutura de Governanca Comunitária

### 4.1 Filosofia de Governanca

O Liberation Client nao tem dono. Nao tem CEO. A governanca e:

- **Policentrica** -- diferentes grupos tomam decisoes em diferentes niveis.
- **Transparente** -- toda decisao e registrada publicamente.
- **Inclusiva** -- barreiras baixas para participacao, barreiras altas para concentracao de poder.
- **"Devagar e sempre"** -- decisoes significativas levam tempo, exigem consulta ampla.

### 4.2 Papeis e Permissoes

| Papel | Permissoes | Como obter | Remocao |
|-------|------------|------------|---------|
| **User** | Usar o app, seguir packs, avaliar packs (opt-in), nominar posts (limitado) | Default ao criar conta | -- |
| **Contributor** | Criar packs, nominar para Tasting Menu, votar em eleicoes de curadores, sinalizar packs | 30 dias ativo + 1 protocolo conectado | Inatividade > 90 dias |
| **Curator** | Curar Tasting Menu semanal, revisar packs sinalizados | Candidatura + sorteio ou convite | Fim do mandato (1 semana) |
| **Senior Curator** | Mentorar novos curadores, mediar disputas, gerenciar categorias de packs | 3+ semanas de curadoria bem-sucedidas | Reavaliacao anual por stewards |
| **Steward** | Gerenciar infraestrutura comunitária, resolver conflitos, guiar visao, decisoes de emergencia | Eleicao por Contributors (2/3 maioria) | 1 ano de mandato, max 2 termos consecutivos |

#### 4.2.1 Transparencia de Papeis

Todos os papeis sao **publicos e auditaveis**:

- Lista de Contributors e publica (opt-in para exibicao).
- Historico de curators e publico (inclui mentorias).
- Stewards sao identificados publicamente com justificativa de eleicao.
- Toda mudanca de papel e registrada como evento no backbone Nostr.

#### 4.2.2 Stewards: Criterios e Limites

**Quem pode ser Steward:**

- Contributor ha 6+ meses.
- Experiencia previa como Senior Curator.
- Participacao consistente em discussoes comunitárias.
- Aprovacao por 2/3 dos Contributors em votacao.

**Limites de poder:**

- Stewards atuam em **colegiado** -- nenhum steward tem poder unilateral (exceto emergencia).
- Decisoes unilaterais de emergencia exigem **justificativa publica em 48 horas**.
- Qualquer steward pode ser destituido por voto de 3/4 dos Contributors.
- Maximo de 7 stewards simultaneos.

### 4.3 Tomada de Decisao

| Tier | Tipo de decisao | Processo | Quorum |
|------|-----------------|----------|--------|
| **Tier 1 (Operacional)** | Selecao do Tasting Menu, revisao de packs sinalizados, atualizacao de packs | Curadores decidem autonomamente dentro das diretrizes | Decisao individual ou pareamento |
| **Tier 2 (Politica)** | Novas categorias de packs, mudancas no processo de curadoria, adicao de protocolos | Discussao comunitária (7 dias) + votacao dos Stewards | 2/3 dos Stewards |
| **Tier 3 (Constitucional)** | Mudanca no modelo de governanca, remocao de steward, alteracao do Codigo de Conduta, decisoes de funding | Discussao comunitária (14 dias) + votacao ampla | 4/5 dos Stewards + maioria dos Contributors |
| **Emergencia** | Remocao imediata de conteudo ilegal, banimento por ameaca a seguranca, resposta a crise | Steward age unilateralmente | Justificativa publica pos-hoc em 48h + ratificacao por colegiado em 7 dias |

#### 4.3.1 Votacao

- Votacao Tier 2 e Tier 3 e feita via eventos Nostr (kind:1022 ou similar).
- Cada Contributor tem 1 voto.
- Votacao e nominal (votos sao publicos) -- exceto quando o assunto e sensivel (ex: remocao de membro), quando pode ser anonima.
- Periodo de votacao: minimo 7 dias (Tier 2), 14 dias (Tier 3).

### 4.4 Resolucao de Conflitos

```
Passo 1: Conversa direta entre as partes
Passo 2: Mediacao por um Steward
Passo 3: Painel comunitario (3 membros aleatorios + 1 Steward)
Passo 4: Decisao final pelo conselho de Stewards (2/3)
```

**Principios:**

- Boa-fe presumida -- assume-se que todos agem de boa-fe ate prova em contrario.
- Privacidade nas etapas iniciais -- as partes podem optar por mediacao privada.
- Transparencia nas etapas finais -- decisoes do passo 3 e 4 sao publicas (exceto quando envolvem seguranca pessoal).
- Apelavel -- decisoes do passo 3 podem ser apeladas ao conselho de Stewards (passo 4).

### 4.5 Codigo de Conduta

#### 4.5.1 Nosso Compromisso

Como membros desta comunidade, comprometemo-nos a criar um ambiente acolhedor, respeitoso e seguro para todas as pessoas, independentemente de idade, corpo, deficiencia, etnia, identidade de genero, nivel de experiencia, nacionalidade, aparencia pessoal, raca, religiao, ou orientacao sexual.

#### 4.5.2 Comportamentos Esperados

- **Linguagem inclusiva** -- usar linguagem que respeita a diversidade de identidades e experiencias.
- **Empatia ativa** -- presumir boa-fe, fazer perguntas antes de julgar, oferecer ajuda.
- **Critica construtiva** -- feedback e bem-vindo quando especifico, fundamentado e respeitoso.
- **Responsabilidade de curadoria** -- curators reconhecem seu poder de amplificar vozes e exercem esse poder com cuidado.
- **Transparencia** -- declarar conflitos de interesse, erros, e limitacoes.

#### 4.5.3 Comportamentos Inaceitaveis

- **Assedio** -- comentarios indesejados, intimidacao, perseguicao, contato nao solicitado.
- **Discriminacao** -- qualquer forma de preconceito ou exclusao baseada em caracteristicas protegidas.
- **Desinformacao intencional** -- criacao ou propagacao deliberada de informacoes falsas.
- **Manipulacao do sistema** -- gaming de nominacoes, votacoes, ou metricas.
- **Auto-promocao predatória** -- uso do sistema de curadoria para beneficio pessoal ou comercial.
- **Ameacas** -- violencia, doxxing, ou qualquer forma de intimidacao.
- **Apropriacao de curadoria** -- usar o papel de curator para promover agenda pessoal ou silenciar vozes.

#### 4.5.4 Responsabilidade de Curadoria e Poder

Curators tem um papel unico de amplificacao. Com esse poder vem responsabilidade:

- Curators devem priorizar vozes marginalizadas e sub-representadas dentro de seus temas.
- Curators nao devem usar o Tasting Menu ou packs para promover aliados politicos, amigos, ou interesses comerciais.
- Curators devem corrigir publicamente erros de curadoria quando identificados.
- Curators devem declarar afiliacoes que possam influenciar suas escolhas.

#### 4.5.5 Conflito de Interesse

Qualquer pessoa em posicao de curadoria (pack ou Tasting Menu) deve declarar:

- Relacoes pessoais significativas com pessoas cujo conteudo esta sendo selecionado.
- Afiliacoes institucionais que possam influenciar escolhas.
- Interesses financeiros ou comerciais em topicos ou contas selecionadas.
- Participacao em comunidades que possam ser beneficiadas pela curadoria.

A declaracao de conflito nao impede a selecao, mas a torna transparente e auditavel.

#### 4.5.6 Processo de Apelacao

Qualquer pessoa pode apelar de uma decisao de curadoria ou disciplina:

1. Apelacao escrita ao conselho de Stewards (max 500 palavras).
2. Stewards tem 7 dias para responder.
3. Se a apelacao for negada, pode-se solicitar revisao pelo painel comunitario (Passo 3 da resolucao de conflitos).
4. Decisao do painel comunitario e final, exceto em casos de vicio processual.

#### 4.5.7 Escada de Enforcement

| Nivel | Violacao | Consequencia |
|-------|----------|--------------|
| **1 - Advertencia** | Linguagem inapropriada leve, desrespeito menor, erro de curadoria sem dolo | Advertencia privada registrada + educacao sobre o Codigo de Conduta |
| **2 - Advertencia Publica** | Violacao moderada, repeticao de nivel 1, conflito de interesse nao declarado | Advertencia publica + restricao de curadoria por 30 dias |
| **3 - Suspensao Temporaria** | Assedio, discriminacao, gaming do sistema, desinformacao intencional | Suspensao de 30-90 dias. Perda de permissoes de curadoria. Notificacao publica. |
| **4 - Remocao Permanente** | Violacao grave (ameacas, doxxing, assedio sexual), reincidencia de nivel 3, destruicao de confianca comunitária | Banimento permanente. Remocao de todos os packs e contribuicoes curatoriais. Registro publico da decisao. |

**Progressao:** As violacoes sobem na escala, mas podem pular niveis dependendo da gravidade. Uma ameaca de violencia vai diretamente para nivel 4.

#### 4.5.8 Aplicacao do Codigo

- Stewards sao responsaveis pela aplicacao do Codigo de Conduta.
- Decisoes de nivel 1-2 podem ser tomadas por um Steward individual.
- Decisoes de nivel 3-4 requerem consenso de 2/3 dos Stewards.
- Qualquer membro da comunidade pode reportar violacoes, inclusive anonimamente.
- O registro de violacoes e mantido em canal privado entre Stewards, com resumo publico anual (sem identificar individuos).

---

## 5. Infraestrutura Tecnica (Minima)

### 5.1 Principio

O Liberation Client **nao tem servidor central**. Toda a infraestrutura comunitária deve funcionar com armazenamento local, protocolos descentralizados, e comunicacao peer-to-peer.

**Backbone recomendado: Nostr** -- e o protocolo mais "serverless-friendly" para este caso de uso.

### 5.2 Esquema de Eventos Nostr

| Recurso | Tipo de Evento | Descricao |
|---------|----------------|-----------|
| **Starter Pack** | Kind:30000 (parametrized replaceable) | Metadados do pack: titulo, descricao, tags, lista de contas, curador, datas |
| **Tasting Menu** | Kind:30001 (parametrized replaceable) | Menu semanal: posts selecionados, justificativas, curador, data |
| **Reputacao do Curador** | Kind:30002 (parametrized replaceable) | Historico de curadoria, packs criados, mentorias |
| **Nominacao** | Kind:30003 (parametrized replaceable) | Nominacao de post para Tasting Menu |
| **Votacao** | Kind:30004 (parametrized replaceable) | Eleicoes e votacoes comunitárias |
| **Sinalizacao de Pack** | Kind:30005 (parametrized replaceable) | Sinalizacao de pack por spam/abuso |
| **Reacao/Avaliacao** | Kind:7 (reaction) | "Este pack me ajudou", reacoes a posts |
| **Reflexao do Curador** | Kind:30006 (parametrized replaceable) | Reflexao semanal pos-publicacao |

### 5.3 Starter Pack como Evento Nostr

```
{
  "kind": 30000,
  "pubkey": "<chave do curador>",
  "created_at": <timestamp>,
  "tags": [
    ["d", "<identificador-unico-do-pack>"],
    ["title", "Fotografia Brasileira Contemporanea"],
    ["summary", "Fotografos brasileiros ativos no fediverso"],
    ["category", "fotografia"],
    ["category", "cultura-brasileira"],
    ["language", "pt"],
    ["protocol", "nostr"],
    ["protocol", "atproto"],
    ["account", "<nostr:pubkey1>"],
    ["account", "<atproto:did1>"],
    ["account", "<activitypub:uri1>"],
    ["curator", "<pubkey do curador>"],
    ["created", "<timestamp>"],
    ["updated", "<timestamp>"],
    ["expiry", "<timestamp + 90 dias>"]
  ],
  "content": "Descricao do pack em texto livre..."
}
```

### 5.4 Tasting Menu como Evento Nostr

```
{
  "kind": 30001,
  "pubkey": "<chave do curador da semana>",
  "created_at": <timestamp>,
  "tags": [
    ["d", "tasting-menu-2026-07-02"],
    ["curator", "<pubkey do curador>"],
    ["mentor", "<pubkey do mentor (se aplicavel)>"],
    ["week", "2026-W27"],
    ["post", "<nostr:event-id>", "justificativa"],
    ["post", "<atproto:uri>", "justificativa"],
    ["post", "<activitypub:uri>", "justificativa"],
    ["protocol-count", "3"],
    ["language-count", "3"],
    ["topic-count", "5"]
  ],
  "content": "Menu desta semana no Liberation Client. Curador: @fulano. 20 posts de 3 protocolos e 3 linguas diferentes."
}
```

### 5.5 Identidade do Curador

- **Nao ha cadastro separado.** A identidade do curador e sua chave publica Nostr + verificacoes vinculadas (NIP-05, DID, Mastodon).
- O perfil do curador e o perfil Nostr (kind:0) com badges e metadados de curadoria.
- A reputacao e derivada dos eventos kind:30002 associados a chave publica.

### 5.6 Votacao e Avaliacao

- **Reacoes:** Nostr kind:7 para "este pack me ajudou", curtidas, etc.
- **Votacoes comunitárias:** kind:30004 com opcoes, periodo, e resultados.
- **Avaliacoes sao locais por padrao** -- o dispositivo do usuario armazena "packs que usei" e "packs que me ajudaram" sem enviar para nenhum servidor.
- Dados agregados e anonimizados podem ser publicados como eventos kind:30000 atualizados (contagem de seguidores, etc.)

### 5.7 Comunicacao Comunitária

| Canal | Uso | Implementacao |
|-------|-----|---------------|
| **Matrix** | Discussoes gerais, anuncions, suporte entre usuarios | Matrix room publico (#liberation-client:matrix.org) |
| **Nostr NIP-28** | Chat publico em tempo real dentro do app | Canal kind:28 vinculado a comunidade |
| **Eventos kind:1** | Anuncions oficiais (novos menus, eleicoes, mudancas) | Publicados na conta oficial da comunidade |
| **Git (sourcehut / GitHub)** | Documentacao de governanca, propostas formais (Tier 3) | Issues + Pull Requests no repositorio da comunidade |

### 5.8 Armazenamento Local

O app mantem localmente:

- Cache de packs recentes (ultimos 500)
- Historico de Tasting Menus (ultimos 3 meses)
- Lista de curadores conhecidos
- Preferencias de curadoria do usuario (packs seguidos, avaliacoes)

Tudo sincronizado via Nostr quando online, funcional offline com dados em cache.

---

## 6. Inicializacao da Comunidade (Bootstrapping)

### 6.1 Fase 1: Fundacao (0-100 usuarios)

**Duracao estimada:** 1-3 meses apos lancamento.

**Quem faz a curadoria:**
- Equipe fundadora do projeto.
- Curadoria centralizada e transparente.
- Decisoes registradas publicamente mesmo quando tomadas por poucas pessoas.

**Starter Packs:**
- **Primeiros 10 packs** criados pela equipe fundadora antes do lancamento publico.
- Cobertura intencional: 3 linguas, 5 topicos, 3 protocolos.
- Packs exemplares que servem como modelo de qualidade.

**Tasting Menu:**
- **Primeiros 4 menus** pre-selecionados pela equipe fundadora.
- Cada menu documentado com criterios e justificativas.
- Servem como "curadoria de referencia" para futuros curadores.

**Governanca:**
- Stewards sao membros da equipe fundadora (max 3).
- Decisoes Tier 2 e 3 ainda sao centralizadas, mas registradas publicamente.
- Documentacao de governanca e codigo de conduta ja estao em vigor.

**Infraestrutura:**
- Relays Nostr configurados e testados.
- Matrix room criado (#liberation-client:matrix.org).
- Repositorio de governanca ativo.

### 6.2 Fase 2: Crescimento (100-1000 usuarios)

**Duracao estimada:** 3-12 meses.

**Transicao de curadoria:**
- Primeiros membros da comunidade sao convidados a se tornar curators.
- Recrutamento ativo em comunidades do Mastodon, Bluesky, Nostr.
- Convites direcionados para diversidade: Brasil, Africa, Asia, Europa nao-anglofona.
- Mentorias estabelecidas (fundadores mentoram primeiros curators da comunidade).

**Starter Packs:**
- Comunidade comeca a criar packs (modelo support-first ativado).
- Sinalizacao comunitária entra em operacao.
- Primeiros casos de revisao e depreciacao.

**Tasting Menu:**
- Curadores da comunidade alternam com fundadores.
- Primeiros sorteios comunitários acontecem.
- Sistema de nominacoes amadurece.

**Governanca:**
- Contributors comecam a ser identificados.
- Primeiras eleicoes de Stewards (alguns membros da comunidade se juntam aos fundadores).
- Decisoes Tier 2 comecam a ser tomadas por votacao.

### 6.3 Fase 3: Maturidade (1000+ usuarios)

**Duracao:** Indefinida.

**Curadoria:**
- 100% comunitária.
- Fundadores sao stewards como quaisquer outros (se eleitos).
- Sistema de curadoria funciona autonomamente.

**Governanca:**
- Stewards eleitos pela comunidade.
- Decisoes Tier 2 e 3 seguem processo completo.
- Codigo de Conduta aplicado por Stewards eleitos.
- Conselho de Stewards com diversidade de regioes e protocolos.

**Evolucao:**
- Propostas de mudanca no modelo de curadoria sao bem-vindas e seguem processo Tier 2/3.
- A comunidade pode decidir por novos protocolos, novas categorias, novas formas de curadoria.
- O modelo e vivo e evolve por decisoes comunitárias.

### 6.4 Recrutamento Inicial de Curadores

**Onde encontrar:**

- **Mastodon:** Usuarios ativos em timelines comunitárias, moderadores de instancias pequenas, pessoas que ja curam listas manualmente.
- **Bluesky:** Criadores de Starter Packs com boa qualidade, moderadores de feeds, usuarios com listas publicas bem curadas.
- **Nostr:** Operadores de relays, criadores de conteudo com curadoria explicita, usuarios ativos em NIP-28 chats.
- **Outros:** Usuarios do Reddit que curam subreddits, editores de newsletters curadas, bibliotecarios, arquivistas.

**Perfil ideal:**
- Ja faz curadoria manualmente (listas, feeds, newsletters).
- Esta insatisfeito com algoritmos de engajamento.
- Tem diversidade de interesses e lingua.
- Prefere "slow and steady" a "move fast and break things".

### 6.5 Incentivos (Base Nao-Monetaria)

O Liberation Client e **anti-comercial** nesse aspecto -- nao ha pagamento para curadores. Os incentivos sao:

- **Reconhecimento:** "Curator do Mes" destacado no app e nos canais comunitários.
- **Badges de perfil:** badges visiveis no perfil Nostr/AT Protocol indicando papel e contribuicoes.
- **Grafico de contribuicao:** timeline visual de contribuicoes (packs criados, menus curados, mentorias).
- **Voz na governanca:** curators ativos tem peso em decisoes Tier 2 e 3.
- **Acesso antecipado:** curadores veem tendencias e conteudo novo antes da publicacao geral.
- **Satisfacao pessoal:** contribuir para um ecossistema saudavel e diverso.

**Nao havera:**
- Pagamento em dinheiro ou crypto.
- Tokens de governanca negociáveis.
- Rewards por engajamento.
- Qualquer mecanismo que transforme curadoria em extracao de valor.

### 6.6 Crescimento e Saude da Comunidade

**Metricas de saude (publicas, nao gamificadas):**

| Metrica | O que mede | Alvo |
|---------|------------|------|
| **Diversidade de curadores** | Quantos curadores diferentes em 30 dias | Minimo 15 unicos por mes |
| **Rotacao de curadores** | Novos curadores como % do total | Minimo 30% novos a cada trimestre |
| **Diversidade de protocolos** | Protocolos representados no Tasting Menu | Minimo 3 por menu |
| **Diversidade de linguas** | Linguas representadas | Minimo 3 por menu |
| **Cobertura de categorias** | Categorias com packs ativos | Minimo 15 categorias |
| **Engajamento em votacoes** | Participacao em eleicoes de Stewards | Minimo 20% dos elegiveis |
| **Tempo de resposta** | Tempo entre sinalizacao e revisao de pack | Max 48h |
| **Satisfacao de curadores** | Pesquisa semestral anonima | >80% satisfeitos ou neutros |

---

## 7. A Voz Brasileira

### 7.1 Origem e Identidade

O Liberation Client nasce no Brasil. Isso nao e detalhe -- e parte fundamental da identidade do projeto. Nao somos mais uma plataforma do Vale do Silicio. Nosso modelo de governanca reflete valores brasileiros e do Sul Global.

### 7.2 "Calor Humano" Como Valor de Governanca

"Calor humano" significa:

- **Relacoes antes de processos** -- a comunidade prioriza conexao humana. Conflitos sao resolvidos com conversa, nao com robotico cumprimento de regras.
- **Acolhimento de quem chega** -- newcomers sao recebidos com paciencia e orientacao, nao com paredes de documentacao.
- **Flexibilidade empatica** -- as regras sao guias, nao correntes. Excecoes sao possiveis quando o contexto humano justifica.
- **Celebracao das contribuicoes** -- reconhecer publicamente o trabalho dos outros e pratica esperada, nao opcional.

### 7.3 "Devagar e Sempre" vs "Move Fast and Break Things"

Rejeitamos explicitamente a filosofia do Vale do Silicio de "mova-se rapido e quebre coisas."

**"Devagar e sempre":**

- Mudancas na governanca levam tempo -- 14 dias minimo para decisoes Tier 3.
- Decisoes sao tomadas apos consulta ampla, nao por urgencia fabricada.
- A estabilidade do sistema e mais importante que a velocidade de inovacao.
- Erros sao admitidos e corrigidos, nao escondidos ou acelerados.
- O projeto cresce no ritmo da comunidade, nao no ritmo de investidores.

### 7.4 Documentacao em Portugues Primeiro

- Este documento e escrito em portugues como **versao original e autoritativa**.
- Traducoes para outras linguas sao derivadas e claramente marcadas como tal.
- Discussoes comunitárias formais (propostas Tier 2 e 3) aceitas em portugues, ingles, e espanhol.
- Documentos de governanca sao traduzidos para ingles e espanhol, mas questoes de interpretacao serao decididas pela versao em portugues.

### 7.5 Fusos Horários Amigaveis

- Reunioes comunitárias (chamadas de Stewards, assembleias) sao agendadas em horarios que funcionem para Americas.
- Horario de referencia: Brasilia (UTC-3) e Sao Paulo (UTC-3).
- Quando possivel, rotacao entre horarios para incluir Asia-Pacifico.
- Comunicacao assincrona e priorizada -- reunioes ao vivo sao excecao, nao regra.

### 7.6 Categorias e Perspectivas do Sul Global

As categorias de curadoria incluem explicitamente:

- **Cultura brasileira** e culturas lusofonas
- **Culturas indigenas** (Americas, Africa, Asia)
- **Movimentos sociais do Sul Global**
- **Tecnologia comunitária** e infraestrutura digital autônoma
- **Ciencias e saberes tradicionais**
- **Arte periférica e marginal**
- **Linguas em perigo de extincao**

Os curators sao incentivados a buscar ativamente conteudo e contas do Sul Global, nao como "exotismo" mas como parte central da diversidade que o Liberation Client se propoe a servir.

### 7.7 Independencia do Vale do Silicio

- Nao aceitamos venture capital.
- Nao otimizamos para crescimento de usuarios como metrica primária.
- Nao usamos modelos de negocios baseados em extracao de dados ou atencao.
- Nossa sustentabilidade e comunitária, nao corporativa.
- A tecnologia serve a comunidade, nao o inverso.

---

## Apendices

### A. Glossario

| Termo | Significado |
|-------|-------------|
| **Curator** | Pessoa selecionada para curar o Tasting Menu de uma semana |
| **Contributor** | Membro da comunidade com permissoes ampliadas (criar packs, nominar, votar) |
| **Starter Pack** | Conjunto curado de 15-30 contas organizado por tema/lingua/regiao |
| **Tasting Menu** | 20 posts curados semanalmente, diversos em protocolos, linguas e topicos |
| **Senior Curator** | Curator experiente que mentoriza novos curadores |
| **Steward** | Membro do conselho de governanca, eleito pela comunidade |
| **Pack "nao mantido"** | Pack sem atualizacao ha mais de 90 dias |
| **Sinalizacao** | Alerta da comunidade sobre problema em um pack |
| **Support-first** | Modelo que exige interacao com packs existentes antes de criar o proprio |
| **Calor humano** | Valor de governanca que prioriza relacoes e acolhimento |

### B. Template para Criacao de Starter Pack

```
Titulo: [max 100 chars]
Descricao: [max 500 chars -- explique o proposito e para quem e]
Categoria(s): [1-3, ex: tecnologia, cultura-brasileira]
Lingua: [ex: pt, en, es, ja]
Contas (15-30):
  - [handle/protocolo]
  - [handle/protocolo]
  - ...
Feeds customizados (opcional): [URLs]
Hashtags (opcional, max 5): [#tag1, #tag2]
```

### C. Template para Nominacao ao Tasting Menu

```
Post: [link ou ID do post]
Protocolo: [nostr | atproto | activitypub | ...]
Justificativa: [1-2 frases explicando por que merece destaque]
Criterio atendido: [informativo | belissimo | instigante | representativo]
```

### D. Template para Reflexao do Curador

```
Semana: [data]
Curador: [nome/handle]
Numero de nominacoes recebidas: [N]
Desafios de diversidade: [texto]
Tendencias observadas: [texto]
O que eu aprendi: [texto]
Agradecimentos especiais: [opcional]
```

---

> **Este documento e vivo.** Propostas de alteracao seguem o processo Tier 3 (constitucional).  
> Versao atual: 1.0 -- Julho 2026.  
> Proximo cronograma de revisao: Janeiro 2027.
>
> *"Devagar e sempre, com calor humano."*
