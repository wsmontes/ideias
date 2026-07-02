# ⚡ 10 Features Que Nenhum App Social Faz

> Iteração 1 do loop de produto. Foco no APP. O que seria mágico de usar?
> 2026-07-01

---

## 1. Thread Unificada Cross-Protocolo

**O que é:** Você posta no Mastodon. Alguém responde pelo Bluesky. Outra pessoa responde pelo Nostr. Você vê TUDO na mesma thread, com badges de protocolo em cada resposta. O app busca replies em todos os protocolos conectados e monta a árvore da conversa.

**Por que é mágico:** Hoje, se você posta em 2 protocolos, tem que checar 2 apps diferentes pra ver as respostas. O Liberation Client **derruba a parede entre protocolos**. A conversa é uma só. Os protocolos são transparentes.

**Como funciona:** O app detecta quando você cross-posta (mesmo conteúdo em múltiplos protocolos) e cria um "thread group" local. Quando busca replies, consulta todos os protocolos do grupo. Deduplica por similaridade de conteúdo. Monta a thread unificada com indicadores visuais de protocolo.

**Desafio técnico:** Sincronizar timelines diferentes. Um reply no Mastodon aparece em segundos (streaming). No Nostr, depende do relay. A thread pode ter "buracos" temporários.

---

## 2. Compositor Universal com Adapt Mode

**O que é:** Você escreve UMA vez. O app mostra previews ao vivo de como o post vai aparecer em cada protocolo. Com um toggle, ele adapta automaticamente:
- Trunca para o limite de caracteres do Bluesky (300)
- Adiciona hashtags para descoberta no Mastodon
- Converte @menções para o formato de cada protocolo
- Remove formatação para IRC
- Extrai tags para Nostr

**Por que é mágico:** Cross-posting hoje é trabalho braçal: copia, cola, adapta, posta em cada app. O compositor universal faz isso em UM lugar, com previews em tempo real.

**Interface:**
```
┌─────────────────────────────────────────┐
│ Composer                                │
│ ┌─────────────────────────────────────┐ │
│ │ Escreva uma vez, publique em todos  │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ [● Mastodon] [◆ Bluesky] [⬡ Nostr]    │ ← protocolos selecionados
│                                         │
│ ● Mastodon: 234/500 ████████░░░        │ ← barra de progresso por protocolo
│ ◆ Bluesky:  234/300 ██████████████░░   │
│ ⬡ Nostr:    234/∞   ████░░░░░░░░░░   │
│                                         │
│ ☐ Adapt Mode: auto-ajustar para cada   │
│                                         │
│ [Preview Mastodon ▾]                    │
│ ┌─────────────────────────────────────┐ │
│ │ 🌿 Meu jardim esta manha. #manha    │ │
│ │ #jardim #natureza                   │ │
│ │ 📎 garden.jpg [ALT: jardim com sol] │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ [Preview Bluesky ▸]                     │
│ [Preview Nostr ▸]                       │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │     SEGURE PARA PUBLICAR            │ │ ← friction by design
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

---

## 3. Identity Switcher

**O que é:** Ao responder a um post, o app detecta automaticamente qual identidade usar (mesmo protocolo do post original). Mas você pode trocar com UM toque. "Responder como @alice@mastodon.social" → "Responder como @alice.bsky.social" → "Responder como npub1abc..."

**Por que é mágico:** Hoje, multi-identidade é um pesadelo de login/logout. Aqui, todas as suas identidades estão sempre disponíveis. Trocar é um gesto.

**Interface:**
```
┌─────────────────────────────────────────┐
│ Reply to @bob's post                    │
│                                         │
│ Replying as:                            │
│ ● @alice@mastodon.social     [padrão]  │
│ ○ @alice.bsky.social                   │
│ ○ npub1abc... (Nostr)                  │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ Write your reply...                 │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

---

## 4. Post Durability Selector

**O que é:** Antes de publicar, você escolhe quanto tempo o post vai durar:

- 🦋 **Borboleta** (1 semana): depois vira composto (arquivo pessoal)
- 🕯️ **Vela** (24 horas): some completamente. Sem arquivo.
- 🪨 **Pedra** (para sempre): fica público até você decidir compostar
- 🌱 **Semente** (7 dias + crescimento): as pessoas contribuem, vira jardim permanente
- 💨 **Vento** (até a conversa morrer): some 3 dias após a última resposta

**Por que é mágico:** Nenhuma rede social te dá controle sobre o TEMPO DE VIDA do que você publica. Stories somem em 24h. Posts ficam para sempre. Não há meio-termo.

**O que muda:** Você pode ser mais vulnerável numa borboleta (some depois). Mais curador numa pedra (fica para sempre). Mais experimental num vento (morre naturalmente).

---

## 5. Feed com Eixos Múltiplos

**O que é:** A timeline não é só uma lista vertical. Tem filtros rápidos:
- **Por protocolo**: "só Mastodon", "só Nostr", "todos"
- **Por tipo**: posts públicos, mensagens de grupo, DMs
- **Por tom**: "só conversas calmas", "debates", "anúncios"
- **Por tempo**: "última hora", "hoje", "esta semana", "mês passado"

Tudo com UM toque. Nada de configurar "listas" ou "algoritmos". Filtros simples, poderosos, transparentes.

**Por que é mágico:** Você controla o que vê sem precisar de algoritmo. Não é "Para Você". É "O Que Você Quer Agora".

---

## 6. Busca Unificada com Proveniência

**O que é:** Um campo de busca que pesquisa EM TODOS OS PROTOCOLOS ao mesmo tempo. Resultados mostram:
- De qual protocolo veio (badge)
- Quem postou
- Quando
- Em qual conversa/thread
- "Este resultado veio do Mastodon (via sua instância) e do Nostr (via relay X)"

**Por que é mágico:** Hoje, buscar em redes descentralizadas é PÉSSIMO. Mastodon desabilita busca por padrão. Nostr depende de relays que implementam NIP-50. Aqui, a busca é local (FTS5 no SQLite) + remota (APIs de cada protocolo). Resultados unificados. Proveniência visível.

---

## 7. Status de Presença sem Vigilância

**O que é:** Você pode ver QUANTAS pessoas estão online na sua rede, mas não QUEM — a menos que elas escolham se mostrar. É um "clima" da rede, não uma lista de alvos.

**Interface:**
```
┌─────────────────────────────────────────┐
│ 🌤️ Sua rede está tranquila              │
│ 12 pessoas online · 3 conversas ativas  │
│                                         │
│ Nas últimas 24h:                        │
│ 47 posts · 23 conversas · 8 novos links │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ ☕ Boteco da Esquina — 5 pessoas    │ │
│ │ "Alguém testou o novo protocolo?"  │ │
│ │ Última mensagem: 2min               │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

**Por que é mágico:** É o oposto do "visto por último às 22:47". Dá sensação de comunidade sem vigilância.

---

## 8. Modo Leitura (Reading Mode)

**O que é:** Um modo onde você vê UM post por vez. Sem timeline. Sem notificações. Sem distrações. Apenas o post e suas respostas. Tela cheia. Fonte grande. Fundo tranquilo. Você avança quando quiser.

**Para quê serve:** Ler threads longas. Consumir conteúdo com calma. Não ser sugado pelo scroll.

**Por que é mágico:** É o equivalente digital de "sentar num banco de praça e ler". Nenhum app social tem isso. Todo app social é sobre "consumir rápido". Este é sobre "consumir com presença".

---

## 9. "Quem Mais Leu" — Métrica Invertida

**O que é:** Em vez de mostrar quantas pessoas curtiram, o post mostra quantas pessoas **leram até o fim**. Não é um número. É uma lista de pessoas que dedicaram tempo ao que você escreveu.

**Como funciona:** O app detecta quando alguém passa mais de X segundos num post (baseado no comprimento do texto) e scrolla até o final. Isso conta como "leitura". A pessoa pode escolher se aparece ou não na lista.

**Por que é mágico:** "Likes" medem impulso. "Leituras" medem atenção. Num app anti-engajamento, a métrica de sucesso não é quantas pessoas clicaram num botão. É quantas dedicaram tempo real ao que você disse.

---

## 10. "O Que Eu Perdi?" — Resumo Inteligente, Não Ansioso

**O que é:** Quando você volta depois de um tempo offline, em vez de um feed infinito de "tudo que você perdeu" (ansiedade), o app mostra:
- **3 conversas que tiveram mais participação** (pessoas que você conhece)
- **2 posts que geraram mais respostas** (na sua rede)
- **1 coisa que alguém disse sobre você** (menção ou resposta direta)
- **N pessoas novas no seu circuito** (quem começou a te seguir ou apareceu nas conversas)

Sempre 3-2-1-N. Sempre contável nos dedos. Sempre acolhedor, não acusatório.

**Por que é mágico:** "Você perdeu 847 posts" é uma dívida. "Aqui está o que importa enquanto você esteve fora" é um presente.

---

## Menções Honrosas (features que não couberam nas 10)

- **Cross-Protocol Identity Discovery**: "Você segue @alice no Mastodon. Ela também está no Bluesky como @alice.bsky.social. Quer segui-la lá também?"
- **Saved Search Agents**: buscas salvas que rodam periodicamente e te mostram resultados novos. "Me avise quando alguém mencionar 'Nostr' E 'ActivityPub' na mesma conversa."
- **Protocol Health Dashboard**: indicadores sutis de saúde de cada protocolo. "Mastodon: 🟢 | Nostr: 🟢 | Bluesky: 🟡 (lento)".
- **Post Version History**: ver como um post foi editado ao longo do tempo. Cada edição é uma versão. Transparente.
- **Conversation Export**: exportar uma conversa inteira como Markdown, PDF, ou JSON. Para guardar. Para citar. Para publicar em outro lugar.

---

*Iteração 1 do loop de produto. Pesquisa de referências em andamento.*
