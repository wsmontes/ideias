# ⌨️ O Modo Power User: Teclado, Colunas, Velocidade

> Iteração 5 do loop de produto. Foco no APP.
> 2026-07-01

---

## O insight

90% dos usuários usam o app no celular, de forma casual. Mas os 10% que usam NO DESKTOP, COM TECLADO, são os que produzem a maior parte do conteúdo, moderam comunidades, e fazem curadoria. Se o app é HOSTIL a power users, ele perde seus melhores contribuidores.

O que seria um app social **otimizado para teclado**?

---

## 1. Navegação Total por Teclado

Cada ação no app tem um atalho. Sem exceção.

```
Navegação:
  j / k         → próximo post / post anterior
  Enter         → abrir post em foco
  Esc           → voltar / fechar
  g t           → ir para timeline
  g n           → ir para notificações
  g m           → ir para mensagens
  g e           → ir para explorar
  g s           → ir para configurações
  1-6           → filtrar por protocolo (1=Mastodon, 2=Bluesky, 3=Nostr...)

Interações:
  r             → responder ao post em foco
  Shift+R       → responder como identidade alternativa
  l             → acknowledge (like)
  b             → boost/repost
  s             → salvar/bookmark
  Shift+S       → compartilhar link
  m             → silenciar autor
  Shift+M       → silenciar conversa
  p             → abrir perfil do autor

Compositor:
  c             → novo post
  Shift+C       → novo post com identidade alternativa
  Ctrl+Enter    → publicar (com confirmação)
  Ctrl+S        → salvar rascunho
  Tab           → navegar entre campos (texto, mídia, protocolos)

Thread:
  t             → expandir thread
  n / p         → próxima / anterior resposta na thread
  Shift+N/P     → pular para próxima / anterior conversa paralela

Timeline:
  f             → "check for new posts"
  Shift+F       → voltar ao topo
  /             → buscar
  .             → carregar mais posts (próxima página)
```

Nenhum app social tem navegação completa por teclado. O TweetDeck chegou perto. O Liberation Client deve ir além.

---

## 2. Layout Multi-Coluna

No desktop, o layout não é uma coluna. É um **dashboard configurável**.

```
┌────────────┬──────────────────┬──────────────────┐
│ Coluna 1   │ Coluna 2         │ Coluna 3         │
│            │                  │                  │
│ Timeline   │ Notificações     │ Mensagens        │
│ [Mast+Nos] │ [Todas]          │ [Todas]          │
│            │                  │                  │
│ ┌────────┐ │ ┌──────────────┐ │ ┌──────────────┐ │
│ │ Post 1 │ │ │ @bob replied │ │ │ Eve: "Hey..."│ │
│ └────────┘ │ └──────────────┘ │ └──────────────┘ │
│ ┌────────┐ │ ┌──────────────┐ │ ┌──────────────┐ │
│ │ Post 2 │ │ │ @carol ➕     │ │ │ #opensource  │ │
│ └────────┘ │ └──────────────┘ │ └──────────────┘ │
│ ┌────────┐ │ ┌──────────────┐ │                  │
│ │ Post 3 │ │ │ @alice 💡     │ │                  │
│ └────────┘ │ └──────────────┘ │                  │
└────────────┴──────────────────┴──────────────────┘
```

**Cada coluna é independente:**
- Pode mostrar um protocolo específico, uma lista, uma busca salva, ou uma conversa
- Scrolla independentemente
- Pode ser redimensionada, movida, fechada
- Nova coluna: `Ctrl+T` (timeline), `Ctrl+N` (notificações), `Ctrl+1-6` (protocolo)

**Presets salvos:**
- "Modo Leitura": 1 coluna larga. Apenas timeline.
- "Modo Dashboard": 3 colunas (timeline, notificações, mensagens)
- "Modo Curadoria": 2 colunas (timeline, busca/explorar)
- "Modo Foco": 1 coluna. Apenas o compositor ou o post atual.

---

## 3. Quick Search (Spotlight-Style)

`Ctrl+K` abre uma barra de busca estilo Spotlight/Command Palette:

```
┌─────────────────────────────────────────────┐
│ ▸ @alice                                    │
│                                             │
│ ── PESSOAS ──────────────────────────────── │
│ @alice@mastodon.social     ● Mastodon       │
│ @alice.bsky.social         ◆ Bluesky        │
│ npub1abc...                ⬡ Nostr          │
│                                             │
│ ── CONVERSAS ────────────────────────────── │
│ "Descentralização..."      💬 47 mensagens  │
│                                             │
│ ── AÇÕES ────────────────────────────────── │
│ Ir para perfil de @alice                    │
│ Enviar mensagem para @alice                 │
│ Buscar posts de @alice contendo "protocolo" │
│                                             │
└─────────────────────────────────────────────┘
```

Funciona como Spotlight: busca tudo (pessoas, conversas, hashtags, comandos) em tempo real.

---

## 4. Custom Saved Searches (Agents)

Você cria uma busca salva que funciona como um **agente**:

```
┌─────────────────────────────────────────────┐
│ 🔍 "nostr" E "activitypub"                  │
│                                             │
│ Atualizado a cada 2h                        │
│ ┌─────────────────────────────────────────┐ │
│ │ ⬡ "Comparing Nostr and ActivityPub..."  │ │
│ │ npub1xyz... · 34min                     │ │
│ └─────────────────────────────────────────┘ │
│ ┌─────────────────────────────────────────┐ │
│ │ ● "Why I prefer ActivityPub over Nostr" │ │
│ │ @user@mastodon.social · 1h              │ │
│ └─────────────────────────────────────────┘ │
│                                             │
│ [Editar] [Pausar] [Deletar]                 │
└─────────────────────────────────────────────┘
```

Cada busca salva pode ser:
- Uma coluna no dashboard
- Um filtro rápido (pill no topo)
- Um "agente" que te notifica quando encontra algo novo

---

## 5. Post Templates (para quem publica muito)

Se você publica regularmente (curador, moderador, criador), você pode criar **templates**:

```
┌─────────────────────────────────────────────┐
│ Templates                                   │
│                                             │
│ ┌─────────────────────────────────────────┐ │
│ │ 📰 "Weekly Roundup"                     │ │
│ │ "Esta semana na comunidade:             │ │
│ │  🔗 Link 1                              │ │
│ │  🔗 Link 2                              │ │
│ │  🔗 Link 3                              │ │
│ │  Obrigado a @mention1, @mention2..."    │ │
│ │ Postar em: ● Mastodon                   │ │
│ └─────────────────────────────────────────┘ │
│                                             │
│ ┌─────────────────────────────────────────┐ │
│ │ 🌱 "Nova ideia"                         │ │
│ │ "[Título da ideia]                      │ │
│ │  [Contexto]                             │ │
│ │  [Por que importa]                      │ │
│ │  [Pergunta para a comunidade]"          │ │
│ │ Postar em: ● Mastodon ◆ Bluesky        │ │
│ └─────────────────────────────────────────┘ │
│                                             │
│ [+ Novo template]                           │
└─────────────────────────────────────────────┘
```

`Ctrl+Shift+T` → abre menu de templates. Seleciona um. Preenche os campos. Publica.

---

## 6. Batch Operations

Para curadores e moderadores:

- Selecionar múltiplos posts (Shift+J/K) e aplicar ação em lote:
  - "Salvar todos como referências"
  - "Adicionar todos à lista X"
  - "Silenciar todos os autores"
- Selecionar múltiplas notificações e "Marcar todas como vistas"
- Selecionar múltiplos DMs e "Arquivar todos"

---

## 7. O Teclado Como Interface Primária

O diferencial do power user mode: **tudo pode ser feito sem o mouse.**

| Ação | Atalho |
|------|--------|
| Navegar posts | j/k |
| Abrir/fechar | Enter/Esc |
| Responder | r |
| Compor | c |
| Buscar | Ctrl+K |
| Filtrar protocolo | 1-6 |
| Nova coluna | Ctrl+T |
| Fechar coluna | Ctrl+W |
| Próxima coluna | Ctrl+→ |
| Modo leitura | Ctrl+Shift+R |
| Alternar Simple Mode | Ctrl+Shift+S |
| Ajuda (mostrar todos) | ? |

---

## Por que isso importa

O desktop NÃO É uma versão maior do mobile. É uma ferramenta diferente para um uso diferente:
- **Mobile** = consumo rápido, resposta curta, estar presente
- **Desktop** = curadoria, criação, moderação, pesquisa, deep work

O Liberation Client precisa ser excelente nos DOIS.

---

*Iteração 5 do loop de produto. Foco no APP.*
