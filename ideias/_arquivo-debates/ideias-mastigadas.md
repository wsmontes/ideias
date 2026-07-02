# Ideias Mastigadas — Liberation Client

## O que realmente ficou dos 12 debates

---

## Quem venceu cada debate (sem meio-termo)

| # | Debate | Venceu | Por quê |
|---|--------|--------|---------|
| 1 | Um protocolo ou vários? | **Linus** | Seis é suicídio. A pergunta real é qual UM primeiro. |
| 2 | Qual protocolo? | **Steve** | ActivityPub. 10M de pessoas > 500K. Pessoas > arquitetura. |
| 3 | Construir ou contribuir? | **Linus** | A biblioteca é o legado. O app é demo. Ninguém precisa de mais um cliente. |
| 4 | Feed unificado ou separado? | **Steve** | Unificado. O usuário não quer saber de protocolo. |
| 5 | Offline-first: como vender? | **Linus** | Performance + privacidade. "Offline liberation" é lorota. |
| 6 | Zero algoritmo? | **Linus** | Cronológico não é neutro, é viés de recência. Ferramentas de curadoria > purity. |
| 7 | Compositor automático? | **Linus** | Auto-truncagem é problema editorial. Humano decide. Preview manual. |
| 8 | E2EE em todo lugar? | **Steve** | Não reinventa PGP. Dois estados. Não aterroriza o usuário. |
| 9 | Design ou função bruta? | **Steve** | Design é função. App feio = app deletado. Mas 60fps primeiro. |
| 10 | Biblioteca ou app como produto? | **Empate** | A biblioteca é o legado. O app é a história. Precisamos dos dois. |
| 11 | Teclado ou touch? | **Steve** | Mobile-first. A libertação é pra todo mundo, não pra dev Linux. |
| 12 | Privacidade radical? | **Linus** | "Seus dados nunca saem do dispositivo" é mentira no momento que você publica. |

**Placar: Linus 6 × Steve 5 × Empate 1.** Linus venceu por pouco. Isso diz algo sobre o projeto: a visão é do Steve, mas a execução viável é do Linus.

---

## As 7 decisões que realmente importam (consistentes entre si)

### 1. O MVP é ActivityPub. Só.

Não são 2 protocolos. Não é "ActivityPub + Nostr". É **UM**. ActivityPub. 10 milhões de pessoas estão lá. O diferencial multi-protocolo entra depois. O MVP prova que o app funciona. Sem isso, não há produto.

O que implementar: Mastodon API (timeline, post, reply, boost, favorite), streaming WebSocket, OAuth, HTML sanitization.

### 2. O produto de verdade é a biblioteca `libs/protocol-adapters`

O Liberation Client é o cliente de referência — funcional, não bonito. Se 100.000 pessoas usarem, ótimo. Se 100 usarem, a biblioteca sobrevive. A biblioteca é o que outros desenvolvedores vão usar para construir os clientes bonitos (Damus 2.0, Ivory multi-protocolo, etc).

Licença: MIT ou Apache 2.0. Sem copyleft. Adoção máxima.

### 3. O feed é unificado. Fim.

Sem abas de protocolo. Sem badges coloridos. O protocolo é um ponto de 6px no avatar. Toque longo para ver detalhes. 90% dos usuários nunca precisam saber que existem protocolos diferentes.

Exceção: quando algo falha. Aí sim — transparência total. "Este post é do Mastodon, que está offline. Mostrando versão em cache de 2h atrás."

### 4. Cronológico como default. Ferramentas de curadoria como poder.

Sem "For You". Sem algoritmo de engajamento. A timeline é o que as pessoas postaram, na ordem.

Mas — e isso é o que o debate 6 decidiu — o usuário PRECISA de ferramentas: listas, mute de palavras, contas prioritárias, filtros por tom. Isso não é algoritmo. É agência do usuário.

Algoritmo opcional futuro: open-source, configurável, transparente. Mas só depois que as ferramentas manuais estiverem perfeitas.

### 5. Offline = performance + privacidade. Nada de "libertação da internet."

SQLite local é cache de performance. 60fps garantido porque os dados já estão no dispositivo. Isso é a verdade. "Funciona sem internet" é consequência, não feature.

Staleness é honesta: "Este post é de 3 dias atrás. A conversa pode ter continuado." Storage budget é transparente: "App usando 2.1 GB. Limite: 5 GB."

### 6. Design essencial. Zero decoração. 60fps é requisito.

Animações só quando comunicam informação: fade-in de conteúdo novo, parallax de hierarquia espacial. Zero bounce, zero googly eyes, zero "delight". 

O app não compete com Ivory em polish visual. Compete em FUNCIONAR. Rápido. Simples. Confiável.

### 7. Honestidade radical sobre privacidade

O app não tem servidores. Seus rascunhos, histórico e buscas nunca saem do dispositivo. Isso É privacidade real.

Mas — e isso é inegociável — o momento que você publica algo, aquilo está em servidores de TERCEIROS. O admin da sua instância Mastodon pode ler seus DMs. O app deixa isso CLARO. "Esta conversa NÃO é privada. O administrador de mastodon.social pode lê-la."

E2EE onde o protocolo já tem (Matrix, Nostr). Transparente. Sem emoji. Sem PGP caseiro.

---

## O que NÃO entrou no MVP (por enquanto)

- ❌ AT Protocol, Matrix, IRC, XMPP → roadmap pós-MVP
- ❌ Compositor multi-protocolo → só ActivityPub no lançamento
- ❌ Offline queue com sync conflito → complexo demais. Vai para o roadmap.
- ❌ Algoritmo configurável → primeiro as ferramentas manuais
- ❌ Modo power-user com multi-coluna → mobile-first
- ❌ E2EE caseiro → usar só o que o protocolo já oferece
- ❌ Data import do Twitter/Instagram → roadmap

---

## O MVP em 1 frase

> **Um cliente ActivityPub que funciona offline, não tem algoritmo, e conta a verdade sobre privacidade. O código é aberto. A arquitetura suporta múltiplos protocolos no futuro.**
