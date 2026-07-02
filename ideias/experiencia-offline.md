# 📡 A Experiência Offline: O App Quando Não Tem Internet

> Iteração 3 do loop de produto. Foco no APP.
> 2026-07-01

---

## O insight

Todo app social é **inútil offline**. Abre, mostra um spinner, diz "sem conexão", e morre.

O Liberation Client é **local-first**. O dispositivo é a fonte da verdade. A internet é um mirror. Isso significa que o app **funciona melhor offline do que qualquer app social funciona online.**

O que você pode fazer sem internet?

---

## 1. Ler TUDO Que Já Foi Sincronizado

Sua timeline dos últimos 30 dias está no SQLite local. Todos os posts. Todas as respostas. Todos os perfis.

**Quando offline:**
- Timeline funciona NORMAL. Posts, respostas, mídia (cacheada).
- Um banner sutil no topo: "📡 Offline. Mostrando conteúdo salvo no dispositivo."
- Nada de spinner. Nada de "sem conexão". O conteúdo está LÁ.

**O que você NÃO vê offline:**
- Posts de pessoas que você segue mas que ainda não foram sincronizados (só os últimos 30 dias por padrão)
- Mídia não cacheada
- Perfis que você nunca visitou

**O design resolve isso:**
- Posts não disponíveis offline mostram um placeholder: "Disponível quando online."
- Você pode marcar contas como "Prioridade" — o conteúdo delas é sincronizado por MAIS TEMPO (90 dias) e com MAIS PROFUNDIDADE (mídia sempre cacheada)

---

## 2. Escrever, Compor, Rascunhar

Sem internet, o botão de publicar vira "Enviar quando online".

**O que acontece:**
- Você escreve normalmente. Mídia anexa normalmente.
- O post vai para a **Offline Queue** (tabela `outbox` no SQLite)
- Uma badge sutil: "📤 3 posts na fila"
- Quando a internet voltar, os posts são enviados na ordem. Você recebe uma notificação silenciosa: "3 posts publicados."

**Estados visuais na Offline Queue:**
```
┌─────────────────────────────────────────┐
│ 📤 Outbox                               │
│                                         │
│ ⏳ "Meu post sobre descentralização..." │
│    Mastodon · aguardando conexão        │
│                                         │
│ ⏳ "GM Nostr!"                          │
│    Nostr · aguardando conexão           │
│                                         │
│ ✅ "Thread sobre protocolos"            │
│    Mastodon + Bluesky · publicado 14:32 │
│                                         │
│ ❌ "Teste"                              │
│    Mastodon · falhou (rate limit)       │
│    [Tentar novamente] [Editar] [Deletar]│
└─────────────────────────────────────────┘
```

---

## 3. Responder e Interagir

Você pode responder a QUALQUER post que já foi sincronizado.

**Respostas offline:**
- Você escreve a resposta normalmente
- Ela entra na Offline Queue
- Quando online, é publicada no protocolo correto
- A thread local JÁ MOSTRA sua resposta (otimista). Com uma badge: "⏳ sua resposta · será publicada quando online"

**Reações offline:**
- Você pode "acknowledge" (like), "pensar" (💡), ou "presentear" (🎁) offline
- Entra na Offline Queue
- A UI local já reflete. Com badge sutil: "⏳".

**Follow offline:**
- Você pode seguir alguém offline
- Entra na queue
- O follow acontece quando online

---

## 4. Navegar e Explorar o Que Já Está Salvo

**Busca offline:**
- FTS5 do SQLite funciona OFFLINE
- Busca em todos os posts, perfis e conversas que já foram sincronizados
- Mesma experiência da busca online, só que limitada ao conteúdo local

**Explorar perfis offline:**
- Perfis que você já visitou estão no cache local
- Dá pra ler posts antigos, ver a bio, ver identidades linkadas
- Só não atualiza (óbvio)

**Ler conversas offline:**
- DMs e conversas de grupo estão no SQLite
- Dá pra ler tudo. Dá pra escrever (vai para a queue).

---

## 5. Organizar, Curar, Limpar

Offline é o momento PERFEITO para tarefas de curadoria:

### Compostagem
- Rever posts antigos e decidir o que compostar (mover para arquivo pessoal)
- "Este post é de janeiro. Quer compostar? Vai virar húmus para novas ideias."
- Organizar seu jardim digital

### Listas
- Criar e editar listas de pessoas
- Reorganizar o que você segue
- "Você segue 200 pessoas. 30 não postam há 3 meses. Quer mover para uma lista 'adormecidos'?"

### Favoritos e Salvos
- Organizar bookmarks
- Reler posts salvos
- "Você salvou 47 posts. Quer organizar por tópico?"

### Limpeza de Cache
- "Seu cache de mídia está com 2.1 GB. Quer reduzir para 1 GB? (mídias antigas serão removidas, posts continuam)"
- Gerenciar quais contas têm prioridade de sync

---

## 6. O Modo Avião Como Feature

**"Modo Profundo"** (Deep Mode):
- Você ativa manualmente (não precisa estar offline de verdade)
- O app SIMULA estar offline por X horas
- Nada entra. Nada sai. Você só interage com o que já está lá.
- Ideal para: escrever sem distração, ler com calma, refletir.

É o oposto do "modo avião" como limitação. É o "modo avião" como **intenção**.

---

## 7. A Reconexão

Quando a internet volta, o app:
1. Detecta conectividade (NetworkMonitor)
2. Flush da Offline Queue (posts, respostas, follows, reactions)
3. Incremental sync de cada protocolo (posts novos desde o último sync)
4. Atualiza a UI silenciosamente

**A experiência:**
```
┌─────────────────────────────────────────┐
│ 📡 Online de volta!                     │
│                                         │
│ ✅ 3 posts publicados                   │
│ ✅ 2 respostas enviadas                 │
│ ✅ 1 follow confirmado                  │
│                                         │
│ 📥 47 novos posts desde sua última      │
│    conexão (há 2 horas)                 │
│                                         │
│ [Ver o que perdi] [Depois]              │
└─────────────────────────────────────────┘
```

Nenhuma notificação push. Nenhum "você perdeu 847 coisas!". Um resumo tranquilo. Você decide se quer ver agora ou depois.

---

## Por que isso é revolucionário

Nenhum app social funciona offline. NENHUM. Nem o Mastodon, nem o Bluesky, nem o Nostr (clientes existentes mostram tela branca sem internet).

O Liberation Client **não é um app que funciona offline. É um app que FUNCIONA. A internet é opcional.**

Isso é particularmente relevante para:
- **Brasil**: conexões instáveis, metrô sem sinal, áreas rurais, planos de dados limitados
- **Viajantes**: aviões, trens, roaming caro
- **Áreas de conflito/censura**: a internet pode cair a qualquer momento
- **Qualquer pessoa**: às vezes você só quer silêncio

---

*Iteração 3 do loop de produto. Foco no APP.*
