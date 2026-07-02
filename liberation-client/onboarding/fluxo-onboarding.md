# Liberation Client — Fluxo de Onboarding & Solucao Cold Start

## Blueprint da Primeira Experiencia

> Um cliente social serverless, multiprotocolo que rejeita explicitamente feeds
> algoritmicos, otimizacao de engajamento e a economia da atencao.
> **Iteracao 2** — 2026-07-01

---

## Indice

1. [Filosofia de Onboarding](#1-filosofia-de-onboarding)
2. [Parte 1: O Fluxo de Onboarding (5 Telas)](#2-parte-1-o-fluxo-de-onboarding-5-telas)
   - [Tela 1: Boas-Vindas & Filosofia](#tela-1-boas-vindas--filosofia)
   - [Tela 2: Configuracao de Identidade](#tela-2-configuracao-de-identidade)
   - [Tela 3: O Content Bootstrap (A Tela Critica)](#tela-3-o-content-bootstrap-a-tela-critica)
   - [Tela 4: O Tasting Menu](#tela-4-o-tasting-menu)
   - [Tela 5: Primeiro Post / Introducao](#tela-5-primeiro-post--introducao)
3. [Parte 2: Descoberta Continua (Sem Algoritmos)](#3-parte-2-descoberta-continua-sem-algoritmos)
4. [Parte 3: Consideracoes Especificas por Protocolo](#4-parte-3-consideracoes-especificas-por-protocolo)
5. [Parte 4: Design dos Estados Vazios](#5-parte-4-design-dos-estados-vazios)
6. [Parte 5: O Conceito do Tasting Menu](#6-parte-5-o-conceito-do-tasting-menu)
7. [Parte 6: Metricas de Sucesso](#7-parte-6-metricas-de-sucesso)
8. [Apendice: Variante Brasileira (Localizada)](#8-apendice-variante-brasileira-localizada)

---

## 1. Filosofia de Onboarding

### 1.1 O Problema Central

O #2 killer risk de qualquer app social e o **empty feed problem**: usuario
abre o app, ve uma tela vazia, nao entende o que fazer, fecha e nunca mais
volta. Em 30 segundos, o app morre.

Num app que rejeita algoritmos, o problema e amplificado: nao podemos
simplesmente jogar conteudo populado por ML na tela. Nao podemos ter "For
You", "Discover", ou "Trending". O feed vazio e uma consequencia direta da
nossa postura etica -- e precisamos resolve-lo com **design, nao com
engenharia de engajamento**.

### 1.2 Nossa Abordagem: Onboarding como Ritual

Onboarding tradicional e uma barreira a ser transposta o mais rapido possivel
para "chegar ao produto de verdade". Nossa abordagem e o oposto: **o
onboarding e a primeira manifestacao dos principios do app**. Cada tela
demonstra um valor fundamental.

| Tela | Valor Demonstrado |
|---|---|
| Tela 1 | Transparencia radical: o app declara o que NAO faz |
| Tela 2 | Autonomia de identidade: voce escolhe quem quer ser |
| Tela 3 | Intencao sobre engajamento: voce constroi seu feed com agencia |
| Tela 4 | Beleza como antidoto a addiction: experiencia composta e finita |
| Tela 5 | Criacao sobre consumo: o primeiro ato e produzir, nao consumir |

### 1.3 Principios de Design do Onboarding

| Principio | Manifestacao no Onboarding |
|---|---|
| **Intent over Engagement** | Cada tela pergunta "O que voce quer?" em vez de "Deixe a gente te guiar" |
| **Friction is a Feature** | Pausas intencionais para reflexao. Toque longo para publicar. |
| **Transparency is Non-Negotiable** | O app explica exatamente o que faz com seus dados (nada) |
| **Ownership by Default** | Toda identidade e configuracao fica local. Nunca no nosso servidor. |
| **Design for Departure** | O onboarding termina com "Pronto, va viver sua vida", nao "Continue rolando" |

---

## 2. Parte 1: O Fluxo de Onboarding (5 Telas)

### Tela 1: Boas-Vindas & Filosofia

**Estado:** Tela cheia, sem rolagem. Background escuro e sereno (#1a1a2e).

```
+--------------------------------------------------+
|                                                  |
|                                                  |
|     ┌──────────────────────────────────┐          |
|     │                                  │          |
|     │      ◆  ◆  ◆  ◆  ◆  ◆          │          |
|     │         L I B E R A T I O N      │          |
|     │                                  │          |
|     └──────────────────────────────────┘          |
|                                                  |
|     Isto nao e um aplicativo de redes sociais.   |
|                                                  |
|     E uma ferramenta para conversas que          |
|     voce escolhe ter.                            |
|                                                  |
|     Nenhum algoritmo decide o que voce ve.       |
|     Voce decide.                                 |
|                                                  |
|     Nenhum numero diz o quanto voce vale.        |
|     Pessoas importam, nao contagens.             |
|                                                  |
|     Seis protocolos. Um lugar.                   |
|     Seu dispositivo. Suas regras.                |
|                                                  |
|     ┌──────────────────────────────────┐          |
|     │         COMECAR                   │          |
|     └──────────────────────────────────┘          |
|                                                  |
|     Ja conhece o projeto? [Pular filosofia]      |
+--------------------------------------------------+
```

**Design Rationale:**

- **Nao e um feature tour.** Nenhum screenshot, nenhum "Veja como e bonito".
  E uma declaracao de intencao. O usuario precisa saber NO QUE esta entrando.
- **Seis pontos no logo** representam os seis protocolos, sem identifica-los
  individualmente. O usuario descobre isso ao longo do fluxo.
- **Texto escaneavel.** Frases curtas, uma por linha. 3-5 segundos para ler.
- **CTA unico "COMECAR".** Nao ha "Criar conta" porque nao ha conta no
  sentido tradicional.
- **Link "Pular filosofia"** no rodape para early adopters, beta testers, ou
  quem ja conhece o projeto.
- **Nenhuma metrica coletada.** O app nem sabe que o usuario chegou ate aqui
  (sem event tracking remoto).
- **Transicao:** Toque em "COMECAR" faz crossfade lento (600ms) para Tela 2.
  Usuarios que pulam a filosofia veem um toast: "Bem-vindo de volta. Os
  principios estao em Ajustes > Sobre o Liberation."

---

### Tela 2: Configuracao de Identidade

**Estado:** Tela central com 3 cards de peso visual igual. O primeiro
esquerdo/superior e o padrao (mais rico em experiencia).

```
+--------------------------------------------------+
|                                                  |
|  Quem e voce aqui?                               |
|  Escolha como comecar. Tudo local.               |
|                                                  |
|  ┌──────────────────────────────────────────┐    |
|  │  ◆  JÁ TENHO CONTAS                      │    |
|  │                                          │    |
|  │  Conecte identidades existentes no       │    |
|  │  Mastodon, Bluesky, Nostr, Matrix,       │    |
|  │  IRC ou XMPP. Importe sua rede.          │    |
|  │  Tudo fica no seu dispositivo.           │    |
|  │  [Escolher este →]                       │    |
|  └──────────────────────────────────────────┘    |
|                                                  |
|  ┌──────────────────────────────────────────┐    |
|  │  ○  SOU NOVO NISSO                      │    |
|  │                                          │    |
|  │  Crie identidades frescas. Nenhum        │    |
|  │  servidor central. Suas chaves, suas     │    |
|  │  regras. 2 minutos e pronto.              │    |
|  │  [Escolher este →]                       │    |
|  └──────────────────────────────────────────┘    |
|                                                  |
|  ┌──────────────────────────────────────────┐    |
|  │  ○  SO QUERO EXPLORAR                   │    |
|  │                                          │    |
|  │  Modo convidado. Veja o que esta         │    |
|  │  rolando sem conectar nada. Tudo         │    |
|  │  efemero. Nada e salvo.                   │    |
|  │  [Escolher este →]                       │    |
|  └──────────────────────────────────────────┘    |
|                                                  |
|  [Continuar] (habilitado apos escolha)           |
+--------------------------------------------------+
```

**Design Rationale:**

- **Tres caminhos com peso visual igual.** O usuario nao e empurrado para
  nenhum caminho. Cada um tem seu valor.
- **"Ja tenho contas"** e o primeiro (superior/esquerdo) porque oferece a
  experiencia mais rica: feed preenchido imediatamente via importacao.
- **"Sou novo nisso"** e o centro: para usuarios sem nenhuma identidade
  descentralizada.
- **"So quero explorar"** e o modo convidado: descartavel, ideal para
  "experimentar antes de instalar".
- **Nenhuma pergunta sobre email, nome real, ou senha.** O app nao tem conta
  centralizada.
- **Nao ha "Criar conta" em lugar nenhum.** O app nunca sera um "servico" que
  "cria contas".

---

#### Sub-fluxo A: "Ja Tenho Contas" (DETALHADO)

**Passo A1: Selecionar protocolos**

```
+--------------------------------------------------+
|  Quais contas voce quer conectar?                |
|                                                  |
|  ☑ Mastodon (ActivityPub)                       |
|     Rede federada. Sua instancia em qualquer     |
|     servidor.                                     |
|                                                  |
|  ☐ Bluesky (AT Protocol)                        |
|     Rede em PDS. Handle + app password.          |
|                                                  |
|  ☐ Nostr                                        |
|     Rede relay-based. Chave criptografica.       |
|                                                  |
|  ☐ Matrix                                       |
|  ☐ IRC                                          |
|  ☐ XMPP                                         |
|     (Redes de chat — opcionais, pode adicionar   |
|      depois em Ajustes > Chat)                   |
|                                                  |
|  Recomendamos comecar com 1 ou 2 protocolos.     |
|  Voce pode adicionar mais a qualquer momento.    |
|                                                  |
|  [Continuar]                                     |
+--------------------------------------------------+
```

- **Mastodon pre-selecionado** (maior base de usuarios, mais familiar).
- **Maximo 2 selecionados inicialmente** para evitar paralisia por excesso de
  escolha. O usuario pode voltar e adicionar mais.
- **Matrix/IRC/XMPP agrupados como "chat"** -- secundarios, nao feed social.

**Passo A2: Conectar cada protocolo**

**Mastodon:**
```
+--------------------------------------------------+
|  Conectar Mastodon                               |
|                                                  |
|  Qual sua instancia (servidor)?                   |
|  ┌──────────────────────────────────────┐        |
|  │  exemplo@mastodon.social             │        |
|  └──────────────────────────────────────┘        |
|                                                  |
|  [Nao sei minha instancia — me ajude]            |
|  [Conectar via OAuth]                            |
|                                                  |
|  Apos conectar, podera importar:                 |
|  • Quem voce segue                               |
|  • Suas listas curadas                            |
|  • Contas silenciadas e bloqueadas                |
|  Tudo fica no seu dispositivo.                   |
+--------------------------------------------------+
```

**Ajuda para escolher instancia:**
- Diretorio curado de ~10 instancias, organizadas por:
  - Tema (geral, tecnologia, arte, Brasil)
  - Idioma (portugues, ingles, espanhol)
  - Politica de moderacao (liberal, moderada, restritiva)
- Cada instancia mostra: nome, descricao, numero aproximado de usuarios,
  linguas faladas

**Bluesky:**
```
+--------------------------------------------------+
|  Conectar Bluesky                                |
|                                                  |
|  Handle:                                         |
|  ┌──────────────────────────────────────┐        |
|  │  usuario.bsky.social                 │        |
|  └──────────────────────────────────────┘        |
|                                                  |
|  App Password (gerado em bsky.app):              |
|  ┌──────────────────────────────────────┐        |
|  │  xxxx-xxxx-xxxx-xxxx                │        |
|  └──────────────────────────────────────┘        |
|                                                  |
|  [O que e um app password?]                      |
|  [Conectar]                                      |
+--------------------------------------------------+
```

- App password e obrigatorio. O app nunca recebe a senha real do Bluesky.
- Link direto: "Gerar em bsky.app" (abre navegador).

**Nostr:**
```
+--------------------------------------------------+
|  Conectar Nostr                                  |
|                                                  |
|  Cole sua chave privada (nsec):                  |
|  ┌──────────────────────────────────────┐        |
|  │  nsec1...                            │        |
|  └──────────────────────────────────────┘        |
|                                                  |
|  Ou:                                              |
|  [Importar de backup]                            |
|  [So quero ler (apenas npub)]                    |
|                                                  |
|  Sua chave fica armazenada com seguranca no      |
|  keychain do seu dispositivo.                    |
+--------------------------------------------------+
```

- Se o usuario fornecer nsec, pode assinar e postar.
- Se fornecer apenas npub, modo leitura.
- Chave armazenada no iOS Keychain / Android Keystore.
- Opcao "Importar de backup" aceita BIP-39 mnemonic (12 palavras).

**Passo A3: Importar grafo social**

```
+--------------------------------------------------+
|  Sua rede esta conectada!                        |
|                                                  |
|  Que tal importar quem voce ja segue?            |
|                                                  |
|  ┌──────────────────────────────────────┐        |
|  |  Mastodon: @voce@mastodon.social     │        |
|  |  ☑ Seguindo (143 contas)            │        |
|  |  ☑ Listas (3 listas)               │        |
|  |  ☑ Contas silenciadas (12)         │        |
|  |  ☐ Contas bloqueadas (2)           │        |
|  |  ☑ Hashtags seguidas (8)           │        |
|  └──────────────────────────────────────┘        |
|                                                  |
|  ┌──────────────────────────────────────┐        |
|  |  Bluesky: @voce.bsky.social         │        |
|  |  ☑ Seguindo (89 contas)            │        |
|  |  ☑ Starter Packs (3 packs)         │        |
|  └──────────────────────────────────────┘        |
|                                                  |
|  ┌──────────────────────────────────────┐        |
|  |  Nostr: npub1...                    │        |
|  |  ☑ Contact list (67 contas)         │        |
|  └──────────────────────────────────────┘        |
|                                                  |
|  Seus dados NUNCA saem do seu dispositivo.       |
|  Esta e uma importacao unica.                    |
|                                                  |
|  [Importar Tudo] [Revisar antes de importar]     |
+--------------------------------------------------+
```

**Regras de importacao:**

- **Mastodon:** GET /api/v1/following (paginated), GET /api/v1/lists,
  GET /api/v1/list_accounts, GET /api/v1/mutes, GET /api/v1/blocks
- **Bluesky:** com.atproto.repo.getRecord para follow records e starter
  pack memberships
- **Nostr:** kind 3 (contact list) de 3 relays diferentes para maximizar
  cobertura
- **One-time import.** Nao ha sync continuo. O usuario pode reimportar
  manualmente em Ajustes.
- **Tudo local.** Nenhum dado cruza a fronteira do dispositivo.

**Apos importacao:**
> "143 contas importadas. Seu feed ja tem conteudo. Quer ver?
> [Ver feed] [Organizar em listas] [Continuar]"

---

#### Sub-fluxo B: "Sou Novo Nisso" (DETALHADO)

**Passo B1: Criar primeira identidade**

```
+--------------------------------------------------+
|  Vamos criar sua primeira identidade             |
|                                                  |
|  Por onde voce quer comecar?                     |
|                                                  |
|  ┌──────────────────────────────────────┐        |
|  │  ◆  Nostr (Recomendado)              │        |
|  │  Nao precisa de servidor. Gera       │        |
|  │  sua chave em segundos.               │        |
|  └──────────────────────────────────────┘        |
|                                                  |
|  ┌──────────────────────────────────────┐        |
|  │  ○  Mastodon                         │        |
|  │  Rede federada. Precisa de um        │        |
|  │  servidor (instancia).               │        |
|  └──────────────────────────────────────┘        |
|                                                  |
|  ┌──────────────────────────────────────┐        |
|  │  ○  Bluesky                         │        |
|  │  Precisa de um PDS. Vamos te guiar  │        |
|  │  ate o bsky.app.                     │        |
|  └──────────────────────────────────────┘        |
|                                                  |
|  Voce pode criar mais identidades depois.        |
+--------------------------------------------------+
```

**Nostr e o caminho recomendado** para usuarios novos porque:

1. Nao precisa escolher servidor (ao contrario de Mastodon)
2. Nao precisa de PDS (ao contrario de Bluesky)
3. Gera chave localmente em segundos
4. Pode comecar a seguir e postar imediatamente
5. Zero atrito de configuracao de rede

**Fluxo de geracao de chave Nostr:**

```
+--------------------------------------------------+
|  Sua Identidade Nostr                           |
|                                                  |
|  Gerando chave criptografica...                  |
|  (2 segundos)                                    |
|                                                  |
|  ┌──────────────────────────────────────┐        |
|  │  Sua frase de recuperacao:           │        |
|  │                                      │        |
|  │  casa flor montanha rio sol lua      │        |
|  │  livro ponte nave vento pedra mar    │        |
|  │                                      │        |
|  │  [Copiar] [Salvar no cofre]          │        |
|  └──────────────────────────────────────┘        |
|                                                  |
|  ⚠  Esta frase e a UNICA maneira de             │
|     recuperar sua identidade.                    │
|     Sem ela, voce perde acesso para sempre.      │
|                                                  |
|  ✓  Salvei minha frase de recuperacao            |
|                                                  |
|  Voce tambem pode conectar relays:               |
|  [Usar relays recomendados] [Configurar manual]  |
+--------------------------------------------------+
```

**Backup:** BIP-39 style mnemonic de 12 palavras. O usuario DEVE confirmar
que salvou antes de prosseguir. A chave e armazenada no keychain.

**Relays:** O usuario NUNCA ve a palavra "relay". O app usa um pool curado de
5 relays de alta qualidade:

- wss://relay.damus.io
- wss://nos.lol
- wss://relay.nostr.band
- wss://purplepag.es
- wss://relay.snort.social

O pool se expande automaticamente via NIP-65 relay lists de contatos
conhecidos. Em Ajustes > Rede, o usuario pode ver "Servidores de conexao
ativa" (nunca "Relays").

**Mastodon (alternativa):**
Se o usuario escolher Mastodon, oferecer diretorio simplificado:
- `mastodon.social` — Geral, global
- `bolha.one` — Brasil, portugues
- `techhub.social` — Tecnologia
- `mastodon.art` — Artes
- `mastodon.online` — Geral, Europa
- "Outra..." — campo de busca

**Bluesky (alternativa):**
Redirecionar para bsky.app para criacao de conta. O app nao pode criar PDS
localmente.

**Passo B2: Conteudo imediato**

Apos criar identidade, o usuario ve:

```
+--------------------------------------------------+
|  O que te interessa?                             |
|                                                  |
|  Escolha topicos para comecar. Sua timeline      |
|  vai se encher enquanto voce escolhe.            |
|                                                  |
|  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐  |
|  │  💻  │ │  🎵  │ │  🎨  │ │  🔬  │ │  🌍  │  |
|  │  Tec │ │  Mus │ │  Art │ │  Cie │ │  Pol │  |
|  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘  |
|                                                  |
|  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐  |
|  │  ⚽  │ │  📚  │ │  🌱  │ │  🇧🇷  │ │  🏳️  │  |
|  │  Esp │ │  Lit │ │  Nat │ │  Bra │ │  LGBTQ│  |
|  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘  |
|                                                  |
|  Voce pode escolher varios.                      |
|                                                  |
|  [Continuar]                                     |
+--------------------------------------------------+
```

Apos selecionar topicos, o app mostra Starter Packs relevantes:

```
+--------------------------------------------------+
|  Starter Packs para voce                         |
|                                                  |
|  Baseado nos topicos que escolheu:               |
|                                                  |
|  ┌──────────────────────────────────────┐        │
|  │  Musica Brasileira                   │        │
|  │  28 contas · Curado por @joao        │        │
|  │  Preview: @mpb, @samba, @bossa       │        │
|  │  [Seguir Pack] [Ver contas]          │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  ┌──────────────────────────────────────┐        │
|  │  Fotografia Urbana                   │        │
|  │  15 contas · Curado por @maria       │        │
|  │  [Seguir Pack] [Ver contas]          │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  [Ver mais packs] [Pular, ja chega]              │
+--------------------------------------------------+
```

**Meta:** Seguir 20-50 contas na primeira sessao.

---

#### Sub-fluxo C: "So Quero Explorar" (DETALHADO)

```
+--------------------------------------------------+
|  Modo Convidado                                  |
|                                                  |
|  Voce esta no modo exploracao. Nada e salvo.     |
|  Sem identidade, sem historico.                  |
|                                                  |
|  ┌──────────────────────────────────────┐        |
|  │  ★  Tasting Menu — 20 posts         │        |
|  │  Selecao curada da semana.           │        |
|  │  Pronto para ler agora.              │        |
|  └──────────────────────────────────────┘        |
|                                                  |
|  ┌──────────────────────────────────────┐        │
|  │  Ver firehose (timelines publicas)  │        │
|  └──────────────────────────────────────┘        │
|                                                  |
|  ┌──────────────────────────────────────┐        │
|  │  Navegar Starter Packs              │        │
|  └──────────────────────────────────────┘        │
|                                                  |
|  Quer salvar seu progresso?                      |
|  [Criar identidade]                              |
+--------------------------------------------------+
```

Modo convidado:
- Feed populado com Tasting Menu (ver Parte 5) + amostras de timelines
  publicas de cada protocolo
- Nao pode postar, seguir, ou curtir
- Dados descartados ao fechar o app
- Banner sutil no topo: "Modo Convidado — nada e salvo"
- A qualquer momento pode converter para identidade real

---

### Tela 3: O Content Bootstrap (A Tela Critica)

**Esta e a tela mais importante de todo o onboarding.** E aqui que o "empty
feed" e preenchido — sem algoritmos, sem engenharia de engajamento.

**Layout:**

```
+--------------------------------------------------+
|                                                  |
|  Monte seu feed                                  |
|  Nenhum algoritmo. So suas escolhas.             |
|                                                  |
|  ┌──────────────────────────────────────────┐    │
|  │  ★  TASTING MENU                        │    │
|  │  20 posts selecionados por curadores     │    │
|  │  humanos. Pronto pra ler.                │    │
|  │  [Experimentar →]                        │    │
|  └──────────────────────────────────────────┘    │
|                                                  |
|  ┌──────────────────────────────────────────┐    │
|  │  A  Importar Seguidores                  │    │
|  │  Sua rede do Mastodon/Bluesky/Nostr.     │    │
|  │  Ja conectado: 143 contas disponiveis.   │    │
|  │  [Importar]                              │    │
|  └──────────────────────────────────────────┘    │
|                                                  |
|  ┌──────────────────────────────────────────┐    │
|  │  B  Starter Packs                        │    │
|  │  Colecoes curadas por topicos.           │    │
|  │  Tecnologia, Arte, Musica, Ciencia...    │    │
|  │  [Explorar]                              │    │
|  └──────────────────────────────────────────┘    │
|                                                  |
|  ┌──────────────────────────────────────────┐    │
|  │  C  Seguir Hashtags / Topicos            │    │
|  │  Posts sobre o que voce ama.             │    │
|  │  [Escolher]                              │    │
|  └──────────────────────────────────────────┘    │
|                                                  |
|  ┌──────────────────────────────────────────┐    │
|  │  D  Ver o Firehose                       │    │
|  │  Amostras de timelines publicas.         │    │
|  │  [Explorar]                              │    │
|  └──────────────────────────────────────────┘    │
|                                                  |
|  ┌──────────────────────────────────────────┐    │
|  │  E  Prefiro comecar do Zero              │    │
|  │  Vazio por enquanto. Por design.         │    │
|  │  [Ir para feed vazio]                    │    │
|  └──────────────────────────────────────────┘    │
|                                                  |
+--------------------------------------------------+
```

#### Opcao A: Importar Seguidores (detalhamento)

Se o usuario conectou contas na Tela 2, mostra preview dos seguidores
encontrados:

```
+--------------------------------------------------+
|  Importar sua Rede                               |
|                                                  |
|  ┌──────────────────────────────────────┐        │
|  │  Mastodon: 143 contas                │        │
|  │  Preview: @joao, @maria, @pedro...   │        │
|  │  [Importar Todos] [Revisar Lista]    │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  ┌──────────────────────────────────────┐        │
|  │  Bluesky: 89 contas                  │        │
|  │  Preview: @fulano, @ciclano...       │        │
|  │  [Importar Todos] [Revisar Lista]    │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  ┌──────────────────────────────────────┐        │
|  │  Nostr: 67 contas                    │        │
|  │  [Importar Todos] [Revisar Lista]    │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  [Importar Tudo]                                  │
+--------------------------------------------------+
```

**Apos importar:**
> "299 contas importadas! Seu feed tem conteudo. Quer ver? [Ver feed]"
> Acompanhado de: "Quer organizar essas contas em listas? [Organizar] [Pular]"

#### Opcao B: Starter Packs (detalhamento)

```
+--------------------------------------------------+
|  Starter Packs                                   |
|                                                  |
|  Colecoes curadas pela comunidade. Nenhum        |
|  algoritmo. So pessoas recomendando pessoas.      |
|                                                  |
|  ┌──────────────────────────────────────┐        │
|  │  Buscar packs...                      │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  Categorias:                                      │
|  [Tecnologia] [Arte] [Musica] [Ciencia]          │
|  [Esportes] [Brasil] [Noticias] [Fotografia]      │
|                                                  │
|  ┌──────────────────────────────────────┐        │
|  │  ◆  Musica Brasileira               │        │
|  │  28 contas · Curado por @joao       │        │
|  │  Protocolos: Mastodon + Nostr       │        │
|  │  Preview: @mpb, @samba, @bossa      │        │
|  │  [Seguir Pack] [Ver Contas]          │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  ┌──────────────────────────────────────┐        │
|  │  ◆  Rust Lang                       │        │
|  │  15 contas · Curado por @ferris     │        │
|  │  Protocolo: Nostr                    │        │
|  │  [Seguir Pack] [Ver Contas]          │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  [Criar meu proprio Pack >]                      │
+--------------------------------------------------+
```

**Fontes de Starter Packs (sem servidor proprio):**
1. **Bluesky Starter Packs** — via AT Protocol API
   (com.atproto.graph.getStarterPack)
2. **Mastodon Packs** — via FEP (fediverse enhancement proposal) quando
   disponivel. Fallback: listas publicas do Mastodon
3. **Nostr** — lists curadas (kind 30000) e bookmark collections
4. **Diretorio proprio** — fallback com curadoria comunitaria, armazenado
   como arquivo estatico assinado (verificado por chave publica)

**Um toque "Seguir Pack" faz:**
1. Segue todas as contas do pack no protocolo correspondente
2. Adiciona marcador interno "Starter Pack: [nome]"
3. Segue hashtags associadas ao pack
4. Mostra confirmacao: "28 contas adicionadas. Quer remover alguma?
   [Revisar] [Ir para o feed]"

**Design anti-algoritmo:**
- Packs sao ordenados por categorias, nao por popularidade
- Nao ha "packs recomendados para voce"
- Curadores sao identificados por handle, nao por "numero de seguidores"

#### Opcao C: Seguir Hashtags / Topicos

```
+--------------------------------------------------+
|  Seguir Hashtags e Topicos                       |
|                                                  |
|  Posts sobre o que voce ama, agregados de        |
|  todos os protocolos. So o que voce escolhe.      |
|                                                  |
|  ┌──────────────────────────────────────┐        │
|  │  Buscar hashtag...                    │        │
|  └──────────────────────────────────────┘        │
|                                                  |
|  Sugeridas:                                       │
|  #photography · #musica · #tecnologia            │
|  #opensource · #fotografia · #literatura         │
|  #ciencia · #brasil · #art                       │
|                                                  |
|  Pacotes de topicos (multihashtag):               |
|  ┌──────────────────────────────────────┐        │
|  │  Musica Brasileira                  │        │
|  │  #mpb + #samba + #musicabrasileira  │        │
|  │  [Seguir Pacote]                     │        │
|  └──────────────────────────────────────┘        │
+--------------------------------------------------+
```

**Mecanismo de "seguir hashtag":**
- Cria feed virtual que agrega posts com aquela hashtag de todos os
  protocolos
- ActivityPub: busca de hashtag via Mastodon API
- AT Protocol: feeds de hashtag ou custom feeds
- Nostr: busca por tags #t (NIP-12)
- Badge "#" no feed unificado
- Usuario pode deixar de seguir a qualquer momento

#### Opcao D: Firehose (Exploracao com Intencao)

```
+--------------------------------------------------+
|  O que esta acontecendo agora?                   |
|                                                  |
|  Janelas para o mundo. Nao um feed infinito.     |
|                                                  |
|  ┌──────────────────────────────────────┐        │
|  │  ◆  Mastodon Timeline Publica       │        │
|  │  Amostra: 20 posts aleatorios       │        │
|  │  Do servidor: mastodon.social       │        │
|  │  [Ver Amostra]                       │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  ┌──────────────────────────────────────┐        │
|  │  ◆  Bluesky Firehose               │        │
|  │  Amostra: 20 posts aleatorios       │        │
|  │  [Ver Amostra]                       │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  ┌──────────────────────────────────────┐        │
|  │  ◆  Nostr Public Timeline           │        │
|  │  Amostra: 20 posts aleatorios       │        │
|  │  [Ver Amostra]                       │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  ⏱  Explorando ha 0 min                         │
|  Este e um modo de exploracao. Quando            │
|  encontrar algo interessante, siga a pessoa      │
|  ou o topico.                                     │
+--------------------------------------------------+
```

**Design anti-addiction:**
- **Cada janela mostra no maximo 20 posts.** Sem "carregar mais". Se quiser
  ver mais, abre nova janela.
- **Timer visivel** no rodape.
- **Apos 10 min:** "Quer seguir alguem que voce viu?"
- **Apos 15 min:** "Que tal compor seu primeiro post?"
- **Apos 20 min:** "Voce explorou bastante. O mundo nao vai acabar se voce
  desligar agora."
- **Nao ha pull-to-refresh.** Amostras sao estaticas ate "Nova amostra".
- Posts do firehose nunca entram no feed do usuario.

#### Opcao E: "Prefiro comecar do zero"

```
+--------------------------------------------------+
|  Tudo bem. Seu feed, suas regras.                |
|                                                  |
|  Seu feed esta vazio por enquanto. Isso e        |
|  proposital. Voce escolhe quando e como          |
|  preenche-lo.                                    |
|                                                  |
|  Enquanto isso, aqui estao algumas portas:       |
|                                                  |
|  ┌──────────────────────────────────────┐        │
|  │  Buscar alguem pelo handle           │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  ┌──────────────────────────────────────┐        │
|  │  Explorar Starter Packs              │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  ┌──────────────────────────────────────┐        │
|  │  Escrever meu primeiro post          │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  [Ir para o feed vazio >]                         |
+--------------------------------------------------+
```

O usuario cai no estado Fresh Empty (ver Parte 4).

---

### Tela 4: O Tasting Menu

Esta tela e acessada via Opcao ★ na Tela 3. Ver descricao completa na
[Parte 5](#6-parte-5-o-conceito-do-tasting-menu).

---

### Tela 5: Primeiro Post / Introducao

```
+--------------------------------------------------+
|                                                  |
|  Apresente-se ao mundo                           |
|                                                  |
|  Seu primeiro post e um convite. Conte quem       |
|  voce e e o que espera encontrar por aqui.      |
|                                                  |
|  ┌──────────────────────────────────────┐        │
|  │                                      │        │
|  │  Ola! Sou o [nome], sou              │        │
|  │  designer e fotografo. Estou         │        │
|  │  aqui para conversar sobre           │        │
|  │  arte, tecnologia e musica           │        │
|  │  brasileira.                         │        │
|  │                                      │        │
|  │  [0/500]                             │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  Publicar em:                                     │
|  [◆ Nostr] [□ Mastodon] [□ Bluesky]             │
|                                                  │
|  Hashtag: #Introduction (adicionada               │
|  automaticamente em protocolos compativeis)       │
|                                                  │
|  ┌──────────────────────────────────────┐        │
|  │  Publicar (toque longo)             │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  O toque longo e intencional. Esta postagem     │
|  sera visivel para todos nos protocolos         │
|  selecionados.                                  │
+--------------------------------------------------+
```

**Design Rationale:**

- **O primeiro ato e criar, nao consumir.** O usuario termina o onboarding
  tendo contribuido com algo para a rede.
- **Toque longo para publicar** e "friction is a feature" em acao.
- **#Introduction** adicionado automaticamente em protocolos que suportam
  hashtags (Mastodon, Bluesky, Nostr como #t).
- **Multi-protocol publishing:** o usuario pode escolher postar em 1, 2 ou 3
  protocolos simultaneamente.
- **Preview adaptado:** o usuario ve como o post aparece em cada protocolo.
- **Nao ha metricas de "alcance".** O post e enviado e pronto.

**Apos publicar:**

```
+--------------------------------------------------+
|  Pronto!                                         |
|                                                  |
|  Seu primeiro post esta no ar.                   |
|                                                  |
|  Voce esta no comando agora.                      |
|                                                  |
|  ┌──────────────────────────────────────┐        │
|  │  Ver meu feed                       │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  ┌──────────────────────────────────────┐        │
|  │  Explorar mais topicos              │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  ┌──────────────────────────────────────┐        │
|  │  Ajustar configuracoes              │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  Voce pode rever esta introducao em              │
|  Ajustes > Onboarding.                            |
+--------------------------------------------------+
```

Onboarding concluido. O usuario entra no feed principal.

---

## 3. Parte 2: Descoberta Continua (Sem Algoritmos)

Apos o onboarding, o usuario precisa descobrir novo conteudo ao longo do
tempo -- sem recomendacoes algoritmicas.

### 3.1 "Starter Packs Que Sua Rede Esta"

**Mecanismo:** "Pessoas que voce segue fazem parte destes packs..."

```
+--------------------------------------------------+
|  Descobrir Packs                                 |
|                                                  |
|  Pessoas que voce segue estao nestes packs:      |
|                                                  |
|  ┌──────────────────────────────────────┐        │
|  │  ◆  Fotografia Urbana               │        │
|  │  Voce segue 3 pessoas neste pack    │        │
|  │  12 pessoas no total                │        │
|  │  [Ver Pack] [Seguir Restante (9)]   │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  ┌──────────────────────────────────────┐        │
|  │  ◆  Devs Brasileiros               │        │
|  │  Voce segue 2 pessoas neste pack    │        │
|  │  8 pessoas no total                 │        │
|  │  [Ver Pack] [Seguir Restante (6)]   │        │
|  └──────────────────────────────────────┘        │
+--------------------------------------------------+
```

**Anti-algoritmo:** Nao e "pessoas como voce tambem seguem" (algoritmico).
E "pessoas que voce ja segue fazem parte de" (fato). O app escaneia packs
conhecidos e encontra interseccao com follows do usuario. Tudo local.

### 3.2 Hashtag Expansion

**Mecanismo:** "Quem usa #fotografia tambem usa #streetphotography"

```
+--------------------------------------------------+
|  Expandindo seus topicos                         |
|                                                  |
|  Voce segue #photography. Postagens que usam    |
|  essa tag tambem usam:                           |
|                                                  |
|  #streetphotography · #portraitphotography       |
|  #analogphotography · #belasartes                |
|  #fotografiaderua                                |
|                                                  |
|  [Seguir Selecionadas] [Ver Todas]               |
|  [Nao mostrar de novo]                           |
+--------------------------------------------------+
```

**Mecanismo local:** Co-ocorrencia de hashtags nos posts ja baixados no
dispositivo. Nao e "trending" -- e analise local do corpus do usuario.

### 3.3 Cross-Protocol Identity Discovery ("Also On...")

**Mecanismo:** "Voce segue @pessoa no Mastodon. Ela tambem esta no Bluesky."

```
+--------------------------------------------------+
|  Conexoes entre Protocolos                       |
|                                                  |
|  @joao@mastodon.social parece estar tambem       |
|  no Bluesky como @joao.bsky.social               |
|                                                  |
|  [Seguir no Bluesky] [Ver Perfil]                |
|  [Nao e a mesma pessoa] [Confirmar identidade]   |
+--------------------------------------------------+
```

**Mecanismo:**
- Matching local baseado em similaridade de handle, bio, links de perfil
- Usa Levenshtein distance para handles
- "Mesma pessoa?" e sugestao local, nao afirmacao
- Pareamento confirmado fica salvo localmente
- Nenhum dado enviado para servidor externo

### 3.4 List Sharing & Community Curation

Usuarios podem criar listas (grupos de contas) e compartilha-las como packs.

- "Minha lista de 50 contas essenciais sobre fotografia"
- Exportada como JSON assinado criptograficamente
- Publicada em qualquer protocolo como post
- Outros usuarios importam colando o JSON ou clicando em link
- Navegavel por topico, idioma, curador
- Nao ha descoberta algoritmica -- as listas sao compartilhadas organicamente

### 3.5 "Import Following Of" (Com Permissao)

```
+--------------------------------------------------+
|  Importar seguidores de alguem                   |
|                                                  |
|  Cole o handle de alguem cujo gosto voce         |
|  confia:                                         |
|                                                  |
|  ┌──────────────────────────────────────┐        │
|  │  @usuario@mastodon.social            │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  [Ver seguidores] [Importar Todos]               |
|  [Revisar antes de importar]                     |
+--------------------------------------------------+
```

So funciona com contas cuja following list e publica (Mastodon, Nostr).
Bluesky requer autenticacao. Preview antes de importar.

### 3.6 Instance/Neighbor Discovery (ActivityPub)

```
+--------------------------------------------------+
|  Servidores Proximos                             |
|                                                  |
|  Voce esta em mastodon.social. Pessoas em        |
|  servidores vizinhos postam sobre:               |
|                                                  |
|  ┌──────────────────────────────────────┐        │
|  │  ◆  fotografia.art.br               │        │
|  │  45 pessoas · Topicos: fotografia    │        │
|  │  [Explorar servidor]                 │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  ┌──────────────────────────────────────┐        │
|  │  ◆  ciencia.rodada.social           │        │
|  │  23 pessoas · Topicos: ciencia       │        │
|  │  [Explorar servidor]                 │        │
|  └──────────────────────────────────────┘        │
+--------------------------------------------------+
```

Usa API de instancias federadas do Mastodon.

---

## 4. Parte 3: Consideracoes Especificas por Protocolo

### 4.1 Nostr

| Aspecto | Decisao de Design |
|---|---|
| **Relay management** | O usuario NUNCA ve a palavra "relay". Substituir por "servidores de conexao" ou "pontos de rede". |
| **Pool default** | Pool curado de 5 relays de alta qualidade (ver secao 2, Sub-fluxo B). |
| **NIP-65** | Relay lists de npubs conhecidos expandem o pool automaticamente. |
| **Geracao de chave** | BIP-39 mnemonic de 12 palavras. "Frase de recuperacao" (nao "seed phrase"). |
| **Importar contact list** | kind 3 de 3 relays diferentes. |
| **Postar** | Plain text (cultura Nostr). Nao enviar HTML. |
| **Discovery** | Busca por tags #t. Sem "trending". |
| **Perfis** | kind 0. Fallback: npub truncado. |
| **Eventos** | NIP-42 (auth) e NIP-59 (gift wrap) suportados mas nao expostos. |

**Regra de ouro:** O usuario nao precisa saber que Nostr e "diferente". So
precisa saber que funciona.

### 4.2 ActivityPub (Mastodon)

| Aspecto | Decisao de Design |
|---|---|
| **Instance selection** | 3 caminhos: (1) "Ja sei minha instancia", (2) "Me ajude a escolher" (diretorio curado), (3) "Tanto faz" (mastodon.social default) |
| **Importar follows** | GET /api/v1/following (paginated) |
| **Importar lists** | GET /api/v1/lists + /api/v1/list_accounts. Criar lists locais. |
| **Importar mutes/blocks** | GET /api/v1/mutes, GET /api/v1/blocks |
| **OAuth** | Preferir OAuth app registration. Fallback: access token manual. |
| **Instance info** | Mostrar na ficha: "Conectado via mastodon.social" |
| **Mastodon Packs** | Integrar via FEP (v4.6+). Fallback: listas publicas. |

### 4.3 AT Protocol (Bluesky)

| Aspecto | Decisao de Design |
|---|---|
| **Autenticacao** | Handle + app password. Nunca senha real. |
| **Criacao de conta** | Redirecionar para bsky.app. |
| **Importar follows** | com.atproto.repo.getRecord para app.bsky.graph.follow |
| **Starter Packs** | com.atproto.graph.getStarterPack |
| **Feeds customizados** | Listar feeds que o usuario ja segue. Nao sugerir novos. |
| **PDS self-hosted** | Configuracao avancada. "Tenho meu proprio PDS." |

### 4.4 Matrix / IRC / XMPP

Estes protocolos sao chat, nao feed. Aparecem em secao separada.

| Aspecto | Decisao de Design |
|---|---|
| **Separacao UX** | Secao "Redes de Chat" separada do feed principal. |
| **Onboarding** | Opcional. "Adicionar rede de chat depois." |
| **Matrix** | Homeserver + login/senha. Suporte SSO. |
| **IRC** | Servidor + porta + nickname. Opcional SASL. |
| **XMPP** | JID + senha. Suporte OMEMO (E2EE). |
| **Descoberta** | Salas recomendadas por topico (curadas). |
| **Notificacoes** | Tratadas como "mensagens", nao "posts". Notificacao imediata. |

**UX de adicao pos-onboarding:**

```
+--------------------------------------------------+
|  Adicionar Rede de Chat                          |
|                                                  |
|  ┌──────────────────────────────────────┐        │
|  │  ◆  Matrix                          │        │
|  │  Conversas criptografadas em grupos  │        │
|  │  [Conectar]                           │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  ┌──────────────────────────────────────┐        │
|  │  ◆  IRC                             │        │
|  │  Canais de texto em tempo real      │        │
|  │  [Conectar]                           │        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  ┌──────────────────────────────────────┐        │
|  │  ◆  XMPP                            │        │
|  │  Mensagens instantaneas federadas   │        │
|  │  [Conectar]                           │        │
|  └──────────────────────────────────────┘        │
+--------------------------------------------------+
```

**Salas recomendadas (curadas):**
- Matrix: #liberation:matrix.org, #softwarelivre:matrix.org
- IRC: #python, #debian, #socorro (Libera.Chat)
- XMPP: MUCs publicos de tecnologia

---

## 5. Parte 4: Design dos Estados Vazios

### 5.1 Estado 1: Fresh Empty (Apos Onboarding, < 5 Seguidores)

```
+--------------------------------------------------+
|                                                  |
|                                                  |
|     ┌──────────────────────────────────┐          |
|     │        ○  ○  ○  ○               │          |
|     │        ○  ○  ○  ○               │          |
|     │        ○  ○  ○  ○               │          |
|     └──────────────────────────────────┘          |
|                                                  |
|                                                  |
|     Sua timeline esta vazia. Isso e por          |
|     design.                                       |
|                                                  |
|     Voce ainda nao escolheu o que quer           |
|     ver por aqui.                                 |
|                                                  |
|     ┌──────────────────────────────────────┐     │
|     │  Buscar alguem para seguir           │     │
|     └──────────────────────────────────────┘     │
|                                                  │
|     ┌──────────────────────────────────────┐     │
|     │  Explorar Starter Packs              │     │
|     └──────────────────────────────────────┘     │
|                                                  │
|     ┌──────────────────────────────────────┐     │
|     │  Escrever um post                    │     │
|     └──────────────────────────────────────┘     │
|                                                  │
|     ┌──────────────────────────────────────┐     │
|     │  Refazer Tasting Menu                │     │
|     └──────────────────────────────────────┘     │
|                                                  |
+--------------------------------------------------+
```

**Elementos:**
- Ilustracao abstrata de "potencial" (nao de "vazio")
- Mensagem positiva: "Isso e por design"
- Acoes que o usuario ja conhece do onboarding
- Nao ha "popular", "trending", ou "recomendado"

### 5.2 Estado 2: Caught Up (Leu Tudo)

```
+--------------------------------------------------+
|                                                  |
|     ════════════════════════════════════         |
|                                                  |
|     Voce esta em dia.                            |
|                                                  |
|     Nao ha nada de novo desde sua                |
|     ultima visita.                               |
|                                                  |
|     A internet pode esperar.                     |
|                                                  |
|     --- Fim do conteudo de 01/07/2026 ---       |
|                                                  |
|     ┌──────────────────────────────────────┐     │
|     │  Escrever um post                    │     │
|     └──────────────────────────────────────┘     │
|                                                  │
|     ┌──────────────────────────────────────┐     │
|     │  Explorar topicos                    │     │
|     └──────────────────────────────────────┘     │
|                                                  │
|     ┌──────────────────────────────────────┐     │
|     │  Ver posts salvos / "Ler depois"     │     │
|     └──────────────────────────────────────┘     │
|                                                  |
+--------------------------------------------------+
```

**Elementos:**
- Linha "--- End of content from [date] ---" (anti-infinite scroll)
- Tom satisfeito, nao ansioso
- "Verificar novamente" como botao explicito (nao pull-to-refresh)

### 5.3 Estado 3: Offline (Sem Conexao)

```
+--------------------------------------------------+
|                                                  |
|     ┌──────────────────────────────────┐          |
|     │   ○    ○    ○                  │          |
|     │     ○    ○    ○                │          |
|     │   ○    ○    ○                  │          |
|     └──────────────────────────────────┘          |
|                                                  |
|     Voce esta offline.                           |
|     Mas seus dados estao aqui.                   |
|                                                  |
|     ┌──────────────────────────────────────┐     │
|     │  Ler posts salvos (N salvos)        │     │
|     └──────────────────────────────────────┘     │
|                                                  │
|     ┌──────────────────────────────────────┐     │
|     │  Escrever rascunhos                 │     │
|     └──────────────────────────────────────┘     │
|                                                  │
|     ┌──────────────────────────────────────┐     │
|     │  Gerenciar contatos / listas        │     │
|     └──────────────────────────────────────┘     │
|                                                  |
+--------------------------------------------------+
```

**Elementos:**
- Ilustracao de conexoes quebradas, ainda bonita
- App funcional offline: rascunhos, cache, contatos
- "Seus dados estao aqui" (ownership)

### 5.4 Estado 4: Protocol Down (Um Protocolo Inacessivel)

```
+--------------------------------------------------+
|                                                  |
|  ◆ Mastodon: Desconectado                       │
|  ───────────────────────────────────────         |
|                                                  |
|  O servidor mastodon.social esta                 |
|  inacessivel. Seus outros protocolos             |
|  continuam funcionando.                           |
|                                                  |
|  Causas: servidor em manutencao, problema        |
|  de rede, ou bloqueio regional.                   |
|                                                  |
|  [Tentar reconectar] [Ignorar e continuar]       |
|  [Alterar servidor Mastodon]                     |
|                                                  |
|  Voce ainda tem 3 protocolos conectados:         |
|  Bluesky · Nostr · Matrix                        |
+--------------------------------------------------+
```

**Elementos:**
- Protocol badge em estado de erro (vermelho suave)
- Causas possiveis listadas (transparencia)
- Posts desse protocolo somem do feed unificado ate reconectar
- Feed unificado continua com outros protocolos
- Banner, nao modal -- nao interrompe

**No feed unificado:**
```
[Post Bluesky]                               ◆
[Post Nostr]                                 ◆
--- [Mastodon unavailable — 12 posts hidden] ---
[Post Bluesky]                               ◆
```

---

## 6. Parte 5: O Conceito do Tasting Menu

### 6.1 Visao Geral

O **Tasting Menu** e a inovacao central do onboarding anti-algoritmo. E uma
experiencia composta, finita, curada por humanos, que da ao usuario algo belo
para ler desde o segundo 1 -- sem configuracao, sem importacao, sem escolhas.

### 6.2 Mecanismo

```
+--------------------------------------------------+
|                                                  |
|  Tasting Menu: 20 Vozes da Rede Aberta          |
|                                                  |
|  Uma selecao curada de posts de toda a rede      |
|  aberta. Renovada toda semana.                   |
|                                                  |
|  ┌──────────────────────────────────────────┐    │
|  │  1/20                                    │    │
|  │                                          │    │
|  │  "A fotografia analogica me ensinou      │    │
|  │  a esperar. Hoje esperar e um ato        │    │
|  │  politico."                              │    │
|  │                                          │    │
|  │  — @joao@fotografia.art.br              │    │
|  │    ◆ Mastodon · Tema: Fotografia        │    │
|  │                                          │    │
|  │  [Seguir autor] [Seguir topico]         │    │
|  │                                [Proximo]│    │
|  └──────────────────────────────────────────┘    │
|                                                  │
|  ┌──────────────────────────────────────────┐    │
|  │  5/20                                    │    │
|  │                                          │    │
|  │  "Se voce nao controla seus servidores   │    │
|  │  de conexao, voce nao controla sua       │    │
|  │  timeline."                              │    │
|  │                                          │    │
|  │  — npub1... (anonimizado)               │    │
|  │    ◆ Nostr · Tema: Tecnologia           │    │
|  │                                          │    │
|  │  [Seguir autor anonimo] [Seguir tag]    │    │
|  │                                [Proximo]│    │
|  └──────────────────────────────────────────┘    │
|                                                  │
|  ...                                            │
|                                                  │
|  ┌──────────────────────────────────────────┐    │
|  │  20/20                                   │    │
|  │                                          │    │
|  │  "A cidade a noite tem uma luz que       │    │
|  │  nenhum app consegue replicar."          │    │
|  │                                          │    │
|  │  — @noite@bsky.social                   │    │
|  │    ◆ Bluesky · Tema: Fotografia         │    │
|  │                                          │    │
|  │  [Seguir autor] [Seguir topico]         │    │
|  │                               [Concluir]│    │
|  └──────────────────────────────────────────┘    │
|                                                  │
|  Apos concluir:                                   │
|                                                  │
|  ┌──────────────────────────────────────────┐    │
|  │  Voce ouviu 20 vozes da rede aberta.    │    │
|  │                                          │    │
|  │  Seguiu: 4 pessoas                       │    │
|  │  Seguiu: 3 topicos                       │    │
|  │                                          │    │
|  │  Seu feed agora tem conteudo.            │    │
|  │                                          │    │
|  │  Quer construir sua propria experiencia? │    │
|  │  [Ir para o feed] [Explorar mais]        │    │
|  └──────────────────────────────────────────┘    │
+--------------------------------------------------+
```

### 6.3 Principios do Tasting Menu

| Principio | Implementacao |
|---|---|
| **Curadoria humana** | Posts selecionados por curadores convidados. Cada semana, um curador diferente. Curadores sao identificados. |
| **Diverso por design** | Maximo 2 posts do mesmo protocolo, 2 do mesmo topico, 2 do mesmo curador por edicao. |
| **Finitude** | Exatamente 20 posts. Sem "carregar mais 20". Acabou, acabou. |
| **Acionavel** | Cada post tem "Seguir autor" e "Seguir topico". O menu e porta de entrada, nao destino. |
| **Temporal** | Edicao semanal. expira em 7 dias. Senso de "edicao", nao de "feed infinito". |
| **Protocol-aware** | Cada post mostra protocolo de origem. O usuario entende o multi-protocolo pela experiencia. |
| **Nao personalizado** | Mesmo menu para todos os usuarios na mesma semana. Diversidade curada, nao derivada de perfil. |

### 6.4 Estrategia de Curadoria

**Onde os posts vem:**
1. Curadores convidados selecionam posts manualmente
2. Qualidade intrinseca, nao engajamento
3. Diversidade de protocolos, idiomas, topicos, perspectivas
4. Curadores sao membros da comunidade, nao funcionarios do app

**Formato do arquivo de curadoria:**
```json
{
  "edition": 12,
  "week": "2026-07-01",
  "curator": {
    "name": "Maria Silva",
    "handle": "@maria@mastodon.art.br"
  },
  "posts": [
    {
      "protocol": "nostr",
      "author": "npub1...",
      "content_hash": "sha256:...",
      "topic": "arte",
      "curator_note": "Ensaio sobre arte generativa e imperfeicao"
    }
  ]
}
```

O arquivo de curadoria e:
- Assinado com a chave do curador (verificavel pelo app)
- Distribuido via protocolos (evento kind 30000 no Nostr, post no Mastodon)
- Baixado e verificado localmente
- Nao passa por servidor central

### 6.5 Ciclo de Vida do Tasting Menu

1. **Segunda-feira:** Nova edicao publicada. Notificacao silenciosa no app.
2. **Durante a semana:** Usuario le no seu ritmo. Posts sao cards individuais.
3. **Ao completar:** Resumo do que seguiu. Convite para construir feed proprio.
4. **Domingo:** Ultimo dia. Aviso: "Ultimo dia desta edicao. Nova edicao amanha."
5. **Segunda-feira:** Edicao anterior arquivada localmente. Nova edicao disponivel.

### 6.6 "E se o usuario nao quiser?"

O Tasting Menu e a opcao ★ (destaque) na Tela 3, mas nao e obrigatorio:

- Opcao A: Importar seguidores
- Opcao B: Starter Packs
- Opcao C: Hashtags
- Opcao D: Firehose
- Opcao E: Feed vazio

O Tasting Menu e o caminho de menor resistencia, nunca imposto.

---

## 7. Parte 6: Metricas de Sucesso

### 7.1 Principios de Medicao

1. **Tudo e opt-in.** Nenhuma metrica coletada sem consentimento.
2. **Tudo e local.** Metricas computadas no dispositivo, nunca enviadas.
3. **Tudo e anonimo.** Se compartilhado, e agregado e anonimizado.
4. **Nao metricas de engajamento.** Nao medimos tempo no app, scroll depth,
   sessoes por dia.
5. **Metricas de saude, nao de crescimento.** Medimos se o app funciona para
   o usuario.

### 7.2 Metricas Primarias

| Metrica | Como e Medida | O Que Indica |
|---|---|---|
| **Taxa de conclusao do onboarding** | Quantos passos o usuario completou antes de sair | Se o onboarding e claro |
| **Dia 1: contas seguidas** | Quantas contas o usuario segue apos 24h | Se o content bootstrap foi eficaz |
| **Semana 1: retencao** | O usuario abriu o app em ate 7 dias? | Se o usuario encontrou valor |
| **Diversidade de protocolos** | Quantos protocolos o usuario conectou apos 30 dias | Se o valor multi-protocolo e percebido |
| **Criacao de listas** | Quantas listas criadas em 30 dias | Se o usuario organiza ativamente o feed |
| **Publicacao D1** | Usuario postou no primeiro dia? | Se a barreira de publicacao esta baixa |

### 7.3 Metricas Secundarias

| Metrica | Como e Medida | O Que Indica |
|---|---|---|
| **Uso do Tasting Menu** | Quantos experimentaram o TM; quantos seguiram a partir dele | Eficacia do TM como porta de entrada |
| **Uso de Starter Packs** | Packs visualizados e seguidos | Utilidade dos packs para descoberta |
| **Cross-protocol follows** | Vezes que seguiu alguem em protocolo diferente | Eficacia da descoberta cross-protocolo |
| **Shares de listas** | Listas exportadas/compartilhadas | Contribuicao do usuario ao ecossistema |

### 7.4 O Que NAO Medimos

- Tempo total no app por sessao
- Scroll depth
- Sessoes por dia
- Taxa de cliques
- Ultima atividade
- Funil de conversao
- MAU/DAU

### 7.5 Tela de Metricas para o Usuario (Local)

```
+--------------------------------------------------+
|  Suas estatisticas (so voce ve)                  |
|                                                  |
|  Voce usa o Liberation ha 12 dias.               |
|                                                  |
|  Protocolos conectados: 3                         |
|  ◆ Mastodon · ◆ Nostr · ◆ Bluesky               |
|                                                  |
|  Contas que voce segue: 42                        |
|  Listas que voce criou: 4                         |
|  Hashtags que voce segue: 7                       |
|  Starter Packs que voce entrou: 3                |
|  Tasting Menus que voce explorou: 2              |
|  Posts que voce escreveu: 8                       |
|                                                  |
|  ┌──────────────────────────────────────┐        │
|  │  Compartilhar estatisticas (anonimas)│        │
|  └──────────────────────────────────────┘        │
|                                                  │
|  ┌──────────────────────────────────────┐        │
|  │  Limpar todos os dados locais       │        │
|  └──────────────────────────────────────┘        │
+--------------------------------------------------+
```

### 7.6 Feedback Loop Etico

Uma vez por mes, o app pergunta:
- "Como esta sendo sua experiencia?"
- 3 opcoes: Satisfeito / Neutro / Frustrado
- Campo de texto opcional
- Local-only. Se quiser compartilhar, exporta e envia.

---

## 8. Apendice: Variante Brasileira (Localizada)

### 8.1 Conteudo Localizado

**Starter Packs sugeridos para o Brasil:**
- "Musica Brasileira" (#mpb #samba #musicabrasileira #bossa)
- "Arte e Cultura Nacional" (#arte #cultura #literatura #cinemanacional)
- "Tech Brasil" (#tecnologia #programacao #opensource)
- "Fotografia Brasil" (#fotografia #brasil #natureza)
- "Politica e Sociedade" (#politica #sociedade #direitos)

**Hashtags brasileiras:**
- #futebol, #brasileirao, #livros, #leitura
- Topicossazonais: #saojoao, #carnaval

### 8.2 Tom e Voz

- "Comecar" em vez de "Iniciar"
- "Feed" em vez de "Timeline"
- Tratamento por "voce"
- "Servidor de conexao" em vez de "relay"
- Evitar "fediverso" no onboarding ("rede social aberta")

---

## 9. Resumo Visual do Fluxo

```
                    ┌─────────────────┐
                    │  Tela 1         │
                    │  Filosofia      │
                    └────────┬────────┘
                             │
                    ┌─────────────────┐
                    │  Tela 2         │
                    │  Identidade     │
                    └───┬───┬───┬─────┘
                        │   │   │
            ┌───────────┘   │   └──────────┐
            ▼               ▼              ▼
     ┌───────────┐   ┌───────────┐   ┌──────────┐
     │Ja tenho   │   │Sou novo   │   │Convidado │
     │contas     │   │aqui       │   │(efemero) │
     └─────┬─────┘   └─────┬─────┘   └────┬─────┘
           │               │              │
           └───────┬───────┘              │
                   │                      │
                   ▼                      ▼
            ┌─────────────────┐
            │  Tela 3         │
            │  Content        │
            │  Bootstrap      │
            └───┬───┬───┬─────┘
                │   │   │
        ┌───────┘   │   └────────┐
        ▼           ▼            ▼
   ┌────────┐ ┌────────┐  ┌──────────┐
   │A:Import│ │B:Packs │  │C:Hashtags│
   └────────┘ └────────┘  └──────────┘
   ┌────────┐ ┌────────┐  ┌──────────┐
   │D:Fire  │ │E:Vazio │  │★:Tasting │
   │hose    │ │        │  │  Menu    │
   └────────┘ └────────┘  └──────────┘
                   │
                   ▼
            ┌─────────────────┐
            │  Tela 4 (via ★) │
            │  Tasting Menu   │
            └────────┬────────┘
                     │
                     ▼
            ┌─────────────────┐
            │  Tela 5         │
            │  Primeiro Post  │
            └────────┬────────┘
                     │
                     ▼
            ┌─────────────────┐
            │  Feed Principal │
            │  (Fim do        │
            │  Onboarding)   │
            └─────────────────┘
```

---

## 10. Checklist de Implementacao

### P0 (Ship-blocking)

- [ ] Tela 1: Boas-Vindas e Filosofia (texto + CTA)
- [ ] Tela 2: Configuracao de Identidade (3 caminhos)
- [ ] Tela 2A: Importacao Mastodon (OAuth + follows + lists + mutes)
- [ ] Tela 2A: Importacao Bluesky (handle + app password + follows)
- [ ] Tela 2A: Importacao Nostr (nsec/npub + contact list)
- [ ] Tela 2B: Geracao de chave Nostr (secp256k1 + BIP-39 mnemonic)
- [ ] Tela 2B: Diretorio de instancias Mastodon (6 opcoes curadas)
- [ ] Tela 3: Content Bootstrap (5 opcoes)
- [ ] Tela 3B: Navegador de Starter Packs (categorias + busca)
- [ ] Tela 3C: Seguir hashtags / topicos
- [ ] Tela 3D: Firehose com timer e limite de 20 posts
- [ ] Tela 3E: "Comecar do zero"
- [ ] Tela 4: Tasting Menu (20 posts, navegacao um-por-vez)
- [ ] Tela 5: Primeiro Post / Introducao (com #Introduction + toque longo)
- [ ] Estado vazio Fresh Empty (< 5 seguidores)
- [ ] Estado Caught Up (leu tudo)

### P1 (Core Experience)

- [ ] Importacao de lists do Mastodon
- [ ] Importacao de starter packs do Bluesky
- [ ] Tasting Menu: renovacao semanal automatica
- [ ] Estado offline (cache + rascunhos)
- [ ] Estado protocol down (degradacao graciosa)
- [ ] Hashtag expansion (co-ocorrencia local)
- [ ] Metricas locais (tela de estatisticas)
- [ ] Feedback loop mensal (3 emojis)

### P2 (Delight & Depth)

- [ ] Cross-protocol identity discovery
- [ ] List sharing (exportar packs como JSON assinado)
- [ ] "Import following of" (com permissao)
- [ ] Instance/neighbor discovery (ActivityPub)
- [ ] Matrix / IRC / XMPP onboarding secundario
- [ ] Curadores brasileiros para Tasting Menu
- [ ] Localizacao completa PT-BR

### P3 (Stretch)

- [ ] Modo convidado completo (dados efemeros)
- [ ] NIP-65 relay list auto-expansao
- [ ] Starter packs via Nostr kind 30000
- [ ] Arquivo de curadoria assinado criptograficamente
- [ ] Exportacao de packs como JSON assinado

---

## 11. Glossario de Termos (Interno)

| Termo do App (Usuario Ve) | Termo Tecnico (Dev Ve) |
|---|---|
| Servidor de conexao | Relay (Nostr) |
| Servidor / Instancia | Instance (ActivityPub) |
| Identidade | Keypair (Nostr) |
| Frase de recuperacao | BIP-39 mnemonic / seed phrase |
| Rede social aberta | Fediverso |
| Ponto de rede | Relay URL |
| Importar rede | Import social graph |
| Modo convidado | Ephemeral session |
| Menu de degustacao | Tasting Menu |
| Pack | Starter Pack / Curated list |

---

*Este documento e o blueprint da primeira experiencia do Liberation Client.
Cada decisao traca de volta a um principio fundamental do projeto: Intent over
Engagement, Chronological is Neutral, Friction is a Feature, Transparency is
Non-Negotiable, Numbers Abstract People, Ownership by Default. Se uma
funcionalidade de onboarding nao serve a esses principios, ela nao pertence
aqui.*

**Proximo passo:** Testar o fluxo com prototipos de papel. Cronometrar cada
path. Validar com usuarios reais antes de implementar.
