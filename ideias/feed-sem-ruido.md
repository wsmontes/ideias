# 🎨 O Feed Multi-Protocolo sem Ruído Visual

> Iteração 4 do loop de produto. Foco no APP.
> 2026-07-01

---

## O problema

Abre o OpenVibe ou o Flare. O feed é uma sopa de badges coloridos, ícones de protocolo, formatos inconsistentes. Um post do Mastodon parece diferente de um post do Bluesky. O Nostr é plain text. O Matrix é mensagem de chat. O resultado: **poluição visual.** O usuário sente que está usando 4 apps diferentes colados com fita adesiva.

**Como fazer 6 protocolos parecerem UM app?**

---

## Princípio 1: O post é o protagonista. O protocolo é uma nota de rodapé.

Em todo app multi-protocolo existente, o badge do protocolo GRITA. É colorido, grande, impositivo.

No Liberation Client:
- O badge do protocolo é um **ponto colorido de 6px** no canto superior direito do avatar
- Não tem texto. Não tem nome. Não tem ícone.
- Toque longo no badge para ver: "Mastodon · via mastodon.social"
- No dia a dia, você NEM PERCEBE o badge. Só vê o conteúdo.

```
┌─────────────────────────────────────────┐
│ ┌────┐                                  │
│ │ A  │● Alice Chen · 2min                │
│ └────┘                                  │
│                                         │
│ A descentralização não é um fim em si   │
│ mesma. É um meio para a autonomia.      │
│                                         │
│ 💬 Bob, Carol e +2     🔖              │
└─────────────────────────────────────────┘
```
O ponto colorido no avatar é a ÚNICA indicação de protocolo. Invisível no uso diário. Acessível quando necessário.

## Princípio 2: Cores de protocolo são SUAVES, não vibrantes

Cores padrão de cada protocolo (iteração 1 do projeto):
- ActivityPub: roxo (#6364FF)
- AT Protocol: azul (#208BFE)
- Nostr: verde menta (#4ADE80)

Essas cores são para BADGES e FILTROS. Não para o feed.

No feed, posts de protocolos diferentes têm cores NEUTRAS. O fundo do post é sempre o mesmo. A tipografia é a mesma. Não há "post do Mastodon" e "post do Nostr". Há POSTS.

## Princípio 3: Filtros de protocolo são PROATIVOS, não REATIVOS

Você não precisa VER de qual protocolo cada post veio. Você precisa PODER filtrar quando quiser.

**Os pills de filtro no topo:**
```
┌─────────────────────────────────────────┐
│ Timeline  [Todos ▾]                     │
│ ┌──────┐ ┌──────┐ ┌──────┐             │
│ │ Todos│ │ Mast │ │ Nostr│ → mais       │
│ └──────┘ └──────┘ └──────┘             │
└─────────────────────────────────────────┘
```

"Todos" é o default. Os pills são pequenos, monocromáticos, com texto. Você toca para filtrar. Não são coloridos — só ficam com cor SUAVE quando selecionados.

**Comportamento:** 
- Toque em "Mastodon" → só posts do Mastodon. Os outros somem com fade.
- Toque em "Todos" → voltam.
- Toque longo em qualquer pill → "Destacar Mastodon" (os posts de Mastodon ficam normais, os outros ficam levemente esmaecidos)

## Princípio 4: Tipos de conteúdo têm TRATAMENTO visual, não badges

| Tipo de conteúdo | Tratamento visual |
|-----------------|-------------------|
| Post público (Mastodon, Bluesky, Nostr) | Card normal, texto + mídia |
| Resposta em thread | Card levemente indentado à esquerda |
| Mensagem de grupo (Matrix, IRC) | Card com fundo sutilmente diferente (0.5% mais escuro) |
| DM | Só aparece na aba de Mensagens. NUNCA no feed. |
| Post com content warning | Blur + razão do CW. Um toque revela. |
| Post com mídia | Imagem com cantos arredondados, badge ALT |
| Post longo (+500 chars) | Truncado com "Ler mais". Expansão inline. |

**NUNCA mostramos:** "Este é um post do Mastodon" como texto. O usuário não precisa saber. Ele só precisa LER.

## Princípio 5: A transição entre protocolos é INVISÍVEL

Quando você está rolando o feed e passa de um post do Mastodon para um do Nostr:

**O que NÃO acontece:**
- ❌ Não há um divider "── Nostr ──"
- ❌ Não há mudança de cor de fundo
- ❌ Não há ícone de protocolo gigante
- ❌ Não há diferença de tipografia

**O que ACONTECE:**
- Os posts fluem naturalmente, como se fossem do mesmo lugar
- A ÚNICA diferença é o ponto colorido de 6px no avatar
- Se você não está prestando atenção nos pontos, você NEM PERCEBE que mudou de protocolo

Isso é deliberado. A proposta de valor é: **"Um app para todas as suas redes."** Não: **"Um app que te lembra o tempo todo que você está usando redes diferentes."**

## Princípio 6: O "Protocolo Viewer" — disclosure progressivo

Se você QUISER saber mais sobre o protocolo de um post:

1. **Nível 1 (sempre visível):** ponto colorido no avatar. Sem texto.
2. **Nível 2 (toque no ponto):** tooltip: "Mastodon · mastodon.social"
3. **Nível 3 (toque longo no post):** menu de contexto com opção "Ver detalhes do protocolo"
4. **Nível 4 (tela de detalhes):** instância/relay, latência, saúde da conexão, link para o post original

**A regra:** cada nível de detalhe só aparece quando o usuário PEDE. O default é o Nível 1 (o ponto). 90% dos usuários nunca passam do Nível 2. E está tudo bem.

## Princípio 7: O "Modo Simples" esconde TUDO

Lembra do Simple Mode da especificação de UX (Iteração 4 do loop anterior)?

No Modo Simples:
- Pontos de protocolo SOMEM
- Pills de filtro SOMEM
- Todos os posts parecem vir DO MESMO LUGAR
- A experiência é de UM app, UMA rede
- As configurações de protocolo ficam em "Avançado"

**Por que isso importa:** Minha mãe não quer saber o que é ActivityPub. Ela quer ver o que os amigos dela postaram. O Modo Simples é para ela.

---

## O que acontece quando dá problema?

### Protocolo fora do ar
Se o Mastodon cair, os posts do Mastodon NÃO somem do feed. Eles ficam com uma opacidade levemente reduzida (0.6) e um indicador sutil: "Mastodon indisponível · mostrando cache".

### Post que falhou ao carregar
Se um post específico não carregou (mídia quebrada, link morto no relay), ele mostra um placeholder: "🔗 Link indisponível · toque para tentar novamente"

### Protocolo lento
Se o Nostr está lento (relay com alta latência), os posts do Nostr aparecem com um pequeno relógio no canto. "Nostr está lento · posts podem demorar para aparecer." Mas os posts que JÁ estavam no cache local aparecem normalmente. Só os NOVOS demoram.

---

## A diferença visual entre "Todos" e "Filtrado"

### Modo "Todos" (default)
```
┌─────────────────────────────────────────┐
│ Timeline  [Todos ▾]  Mast | Nostr | ATP │
│                                         │
│ ┌────┐                                  │
│ │ A  │● Alice · 2min                    │
│ └────┘ "Descentralização..."            │
│                                         │
│ ┌────┐                                  │
│ │ B  │⬡ Bob · 5min                      │
│ └────┘ "GM Nostr! ☕"                   │
│                                         │
│ ┌────┐                                  │
│ │ C  │◆ Carol · 8min                    │
│ └────┘ "Just shipped v2..."            │
│                                         │
│ Posts fluem naturalmente. Só pontos     │
│ coloridos indicam protocolo.            │
└─────────────────────────────────────────┘
```

### Modo "Só Mastodon" (filtrado)
```
┌─────────────────────────────────────────┐
│ Timeline  [Mastodon ▾]  Todos | Nostr | │
│                                         │
│ ┌────┐                                  │
│ │ A  │● Alice · 2min                    │
│ └────┘ "Descentralização..."            │
│                                         │
│ ┌────┐                                  │
│ │ D  │● Dave · 4min                     │
│ └────┘ "New blog post..."              │
│                                         │
│ ┌────┐                                  │
│ │ E  │● Eve · 12min                     │
│ └────┘ "Thinking about protocols..."   │
│                                         │
│ Modo limpo. Posts de outros protocolos  │
│ ocultos. Nenhuma indicação de que       │
│ existem outros — mas fácil de voltar.   │
└─────────────────────────────────────────┘
```

---

## Por que isso importa

O maior risco de um app multi-protocolo é a **fadiga cognitiva**: o usuário sente que está gerenciando sistemas diferentes, não interagindo com pessoas.

A solução não é "esconder que são protocolos diferentes" (isso seria desonesto). É **mostrar só quando necessário, e de forma que não atrapalhe.**

O feed do Liberation Client deve ser o feed mais CALMO que você já usou. Independente de quantos protocolos estejam por trás.

---

*Iteração 4 do loop de produto. Foco no APP.*
