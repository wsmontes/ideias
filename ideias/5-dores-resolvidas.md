# 🔧 5 Dores Que Todo App Social Tem (e Como o Liberation Resolve)

> Iteração 2 do loop de produto. Foco no APP. Experiências concretas.
> 2026-07-01

---

## Dor 1: "Preciso de 4 apps diferentes para falar com as mesmas pessoas"

**O problema:** Você segue @pessoa no Mastodon, no Bluesky E no Nostr. Três apps. Três timelines. Três caixas de notificação. A conversa está fragmentada.

**Como o Liberation resolve:**

### A) Perfil Unificado
Quando você abre o perfil de alguém, o app mostra TODAS as identidades dessa pessoa que você segue, em todos os protocolos:
```
┌─────────────────────────────────────────┐
│ @alice                                  │
│                                         │
│ ● @alice@mastodon.social      ✓ online │
│ ◆ @alice.bsky.social          ✓ online │
│ ⬡ npub1abc...                 ✗ offline │
│                                         │
│ [Posts] [Em comum] [Todos protocolos ▾] │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ ● "Thinking about protocols..." 2h │ │ ← Mastodon
│ └─────────────────────────────────────┘ │
│ ┌─────────────────────────────────────┐ │
│ │ ◆ "Just shipped v2!" 5h            │ │ ← Bluesky
│ └─────────────────────────────────────┘ │
│ ┌─────────────────────────────────────┐ │
│ │ ⬡ "GM Nostr!" 1d                   │ │ ← Nostr
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

### B) Notificações Unificadas
Uma menção no Mastodon, um reply no Bluesky, um zap no Nostr — tudo na mesma lista de notificações. Com badge de protocolo. Sem contagem. Sem bolinha vermelha.

### C) "Also on..."
Quando você encontra alguém interessante no Mastodon, o app pergunta: "Esta pessoa também está no Bluesky e no Nostr. Quer segui-la lá também?" Um toque. Pronto.

---

## Dor 2: "Escrevi o post errado, editei, e ninguém viu a edição"

**O problema:** Você publica algo. Percebe um erro. Edita. Mas quem já viu o post NUNCA vê a edição. Ou pior: o Twitter/X não tem edição. Você deleta e reposta. Perde as respostas.

**Como o Liberation resolve:**

### Histórico de Versões Visível
Cada edição de um post gera uma versão. Quem quiser pode ver o que mudou:
```
┌─────────────────────────────────────────┐
│ Post de @alice                          │
│                                         │
│ Versão atual (editada há 2h):           │
│ "A descentralização não é um fim em si  │
│  mesma. É um meio para a autonomia."    │
│                                         │
│ [Ver versão original ▾]                 │
│ ┌─────────────────────────────────────┐ │
│ │ "A descentralização é um fim em si  │ │ ← riscado = removido
│ │  mesma. É um meio para a liberdade."│ │ ← sublinhado = adicionado
│ └─────────────────────────────────────┘ │
│                                         │
│ 3 pessoas responderam à v1.             │
│ Suas respostas continuam visíveis.      │
└─────────────────────────────────────────┘
```
- Edições são transparentes (dá pra ver o que mudou)
- Respostas à versão antiga NÃO somem
- O autor pode adicionar uma nota: "Corrigi o argumento sobre descentralização"

### Edição Cross-Protocolo
Se você cross-postou, editar no Liberation edita em TODOS os protocolos (onde edição é suportada: Mastodon ✓, Bluesky ✓, Nostr ✓ via NIP-33).

---

## Dor 3: "Quero responder algo mas não quero entrar numa discussão infinita"

**O problema:** Você vê um post. Tem algo a dizer. Mas sabe que se comentar, vai receber 50 notificações, entrar numa thread de 200 mensagens, e perder 2 horas.

**Como o Liberation resolve:**

### Modos de Resposta com Níveis de Engajamento

Ao responder, você escolhe o **nível de engajamento**:

| Modo | O que faz | Notificações |
|------|-----------|-------------|
| 🍃 **Sussurro** | Sua resposta aparece SÓ para o autor. Não é pública. | Só se o autor responder |
| 💬 **Resposta normal** | Resposta pública, aparece na thread | Respostas à sua resposta |
| 🔥 **Fogueira** | Resposta pública, mas a thread fecha em 24h | Silencia após 24h |
| 📬 **Carta** | Resposta longa (mín. 500 chars). Sem replies. | Zero notificações |

**Por que é mágico:** Você controla seu nível de exposição. Dá pra contribuir sem ser sugado.

### Sussurro: a feature mais subestimada
O "sussurro" resolve o problema de "quero agradecer mas não quero poluir a thread". É um comentário privado que só o autor vê. Como um DM, mas ancorado no post. O autor pode tornar público se quiser (com permissão de quem sussurrou).

---

## Dor 4: "Abri o app para uma coisa e saí 40 minutos depois sem fazer o que queria"

**O problema:** Você abriu o Instagram para responder uma mensagem. 40 minutos depois, você assistiu 15 Reels, curtiu 7 fotos, e ESQUECEU de responder a mensagem. O app sequestrou sua intenção.

**Como o Liberation resolve:**

### Intent-Based Entry (já especificado, mas aqui vai o fluxo real)

```
┌─────────────────────────────────────────┐
│ Bom dia.                                │
│                                         │
│ O que você quer fazer?                  │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ 📖 LER                              │ │
│ │ Sua timeline, em ordem cronológica  │ │
│ │ 47 posts não lidos                  │ │
│ └─────────────────────────────────────┘ │
│ ┌─────────────────────────────────────┐ │
│ │ 💬 RESPONDER                        │ │
│ │ 3 menções, 1 mensagem direta        │ │
│ └─────────────────────────────────────┘ │
│ ┌─────────────────────────────────────┐ │
│ │ ✍️ ESCREVER                         │ │
│ │ Publicar algo novo                  │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ⏱️ Você passou 0 min aqui hoje         │
└─────────────────────────────────────────┘
```

### Session Guardrails
- Se você entrou em "RESPONDER", o app só mostra notificações. Sem timeline. Sem scroll.
- Se você entrou em "LER", o app mostra a timeline. Após 50 posts: "Fim da página. [Carregar mais?]"
- Se você entrou em "ESCREVER", o app abre o compositor. Nada mais.
- **Cada modo é autocontido.** Você não "cai" na timeline sem querer.
- Timer visível: "Você está aqui há 12 min."

---

## Dor 5: "Gostei do post mas não tenho nada a dizer"

**O problema:** Like é vazio. "❤️ 47" não significa nada. Mas comentar "concordo!" também é inútil. Não há meio-termo entre o vazio e o ruído.

**Como o Liberation resolve:**

### Acknowledgement com Contexto

Em vez de um coração vazio, o "acknowledge" (não chamamos de like) tem **camadas de significado**:

| Gesto | Significado | Visível para |
|-------|-------------|-------------|
| 👁️ **Eu vi** | "Li seu post." | Só o autor |
| 💡 **Me fez pensar** | "Mudou algo na minha cabeça." | Autor + quem escolher |
| 🎁 **Presente** | "Isso me tocou. Aqui está algo em troca." (texto livre) | Só o autor |
| 🔖 **Salvar** | "Quero voltar nisso depois." | Só você |

**Como aparece para o autor:**
```
┌─────────────────────────────────────────┐
│ Seu post "Descentralização..."          │
│                                         │
│ 12 pessoas leram 👁️                      │
│ 7 pessoas marcaram 💡                    │
│ 3 pessoas presentearam 🎁                │
│                                         │
│ 💡 @bob: "Nunca tinha pensado nisso"    │
│ 💡 @carol: "Mudou minha perspectiva"    │
│ 🎁 @dave: "Aqui está um artigo que      │
│    conecta com o que você disse..."     │
│                                         │
│ [Ver todos]                             │
└─────────────────────────────────────────┘
```

Números NUNCA visíveis publicamente. O autor vê QUEM, não QUANTOS. O público não vê nada — só as respostas.

---

## Pra fechar: a dor zero

**"Não sei o que está acontecendo."**

Em todo app social, você está perdido. Não sabe por que vê o que vê. Não sabe o que o algoritmo escondeu. Não sabe quem viu ou deixou de ver seu post.

O Liberation tem um botão em toda tela: **"Por que estou vendo isso?"**

```
┌─────────────────────────────────────────┐
│ Por que este post está aqui?            │
│                                         │
│ ● Postado por @alice                    │
│ ● Você segue @alice desde março 2025    │
│ ● Protocolo: Mastodon                   │
│ ● Instância: mastodon.social            │
│ ● Ordem: cronológica (14:32 UTC)        │
│ ● Nenhum algoritmo influenciou esta     │
│   posição                               │
│                                         │
│ [Gerenciar: deixar de seguir @alice]    │
│ [Filtrar: menos posts assim?]           │
│ [Tudo certo]                            │
└─────────────────────────────────────────┘
```

Transparência radical. Sem caixa preta. Você sempre sabe o PORQUÊ.

---

*Iteração 2 do loop de produto. Foco no APP.
Pesquisa: 80+ apps analisados.*
