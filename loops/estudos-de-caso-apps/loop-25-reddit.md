# Estudo de Caso 25 — Reddit: A Rede Social Que Não É Sobre Pessoas (É Sobre Comunidades)

> **Data:** 2026-07-03
> **Loop:** 25 de ∞ (Fase 2: Comunidade / Social)
> **Categoria:** Rede Social / Comunidades / Discussão
> **Tema:** Junho de 2005. Steve Huffman e Alexis Ohanian, recém-formados na University of Virginia, participam da PRIMEIRA turma do Y Combinator. Paul Graham rejeitou a ideia original deles (um app de pedido de comida por SMS chamado "My Mobile Menu"). Mas gostou DOS DOIS. "Voltem com outra ideia." Numa sessão de brainstorming com Graham, nasce o conceito: **"a front page of the internet."** Um site onde usuários SUBMETEM links e a comunidade VOTA. O que é bom SOBE. O que é ruim DESCE. Huffman codou a primeira versão em LISP. Por um MÊS, ele e Ohanian criaram contas FALSAS para postar conteúdo — para o site não parecer VAZIO. Venderam para a Condé Nast por ~$15M em 2006 — e Huffman depois chamou de "erro." Em 2015, ele VOLTOU como CEO para salvar a empresa. Hoje: 110M+ DAUs, 100.000+ subreddits, IPO de $28B (2024), e a MAIOR fonte de conhecimento humano orgânico da internet — tanto que o Google paga $60M/ANO para treinar AI nos dados do Reddit. Esta é a história da plataforma que NÃO é sobre VOCÊ — é sobre o QUE você tem a dizer.

---

## 1. A Origem: Contas Falsas, LISP e um Mascote Chamado Snoo

### Os Fundadores

| Fundador | Background |
|---|---|
| **Steve Huffman** (u/spez) | UVA. Programador. Codou o Reddit original em LISP. Vendeu. Saiu. Voltou como CEO em 2015. |
| **Alexis Ohanian** (u/kn0thing) | UVA. Marketing, comunidade, "a cara do Reddit." Marido da Serena Williams. |

### Y Combinator, Turma #1 (2005)

- Ideia original: **My Mobile Menu** — pedir comida por SMS. Paul Graham: "NÃO."
- Graham: "Mas vocês são bons. Voltem com OUTRA ideia."
- Brainstorm: "E se as pessoas SUBMETESSEM links e VOTASSEM?" Graham: "ISSO."

Huffman codou o Reddit inteiro em **LISP.** 2005. Um computador.

### O Primeiro Mês: Contas Falsas

Reddit lançou. Zero usuários. Ninguém postava.

Huffman e Ohanian criaram **contas FALSAS** e postavam links ELES MESMOS. Por um MÊS inteiro. Para o site não parecer um deserto.

> *"The day we didn't have to submit anything was awesome — because we'd set a tone and people didn't hate it."* — Ohanian

### O Nome Que Quase Foi

Queriam chamar de **"Snew."** Daí a piada: "What's snew?" → "Exactly!" (What's new?)

Desistiram. O mascote alienígena, **Snoo**, é o fantasma desse nome.

### A Venda e o Retorno

- **2006**: Condé Nast compra por **~$15M.**
- Huffman depois: *"Foi um erro vender tão cedo."*
- **2009**: Huffman e Ohanian SAEM.
- **2015**: Ellen Pao renuncia sob polêmica. **Huffman VOLTA como CEO.**
- **2024**: IPO na NYSE: RDDT. Valuation: **$28B.** 20 anos depois.

---

## 2. A Filosofia: "A Rede Anti-Social"

### Interest-First, Não Identity-First

O Reddit NÃO tem:
- Foto de perfil (avatars custom, mas sem foto REAL).
- Contagem de seguidores (existe mas ninguém OLHA).
- Feed algorítmico de "amigos."

O Reddit TEM:
- **Subreddits**: comunidades sobre TÓPICOS. r/brasil. r/programming. r/aww. r/AskHistorians.
- **Upvote/Downvote**: o que é BOM sobe. O que é RUIM desce. O que é CONTROVERSO (100↑, 100↓) desaparece.
- **Comentários em ÁRVORE**: threads aninhadas. Debates profundos.

> *"Reddit is not about who you ARE. It's about what you have to SAY."* — Steve Huffman

### "Depth Over Scrolls"

Enquanto TikTok e Instagram otimizam para SCROLL INFINITO (velocidade), o Reddit otimiza para IMERSÃO: clicar numa thread, LER argumentos, PARTICIPAR. O tempo no Reddit é MAIS PROFUNDO.

---

## 3. As Inovações do Reddit

### 3.1 Upvote/Downvote: O Algoritmo de Confiança

O Reddit NÃO ordena por "quem tem mais upvotes." Usa um **intervalo de confiança estatístico:**

- Um comentário com 10↑/1↓ pode rankear ACIMA de um com 100↑/50↓.
- Isso favorece conteúdo GENUINAMENTE útil, não polarizador.
- O controverso (muitos ↑ E muitos ↓) AFUNDA.

### 3.2 Subreddits: O Átomo da Comunidade

100.000+ comunidades auto-organizadas. Cada uma com:
- **Regras PRÓPRIAS.**
- **Moderadores VOLUNTÁRIOS.**
- **Cultura ÚNICA.** r/AskHistorians é acadêmico. r/wallstreetbets é o CAOS.

"Estados dentro de uma federação."

### 3.3 AMA ("Ask Me Anything")

Barack Obama. Bill Gates. Cientistas. Celebridades. Pessoas NORMAIS com histórias INCRÍVEIS.

"Eu sou um ex-presidiário. AMA." → 10.000 comentários. O Reddit INVENTOU a entrevista coletiva da internet.

### 3.4 Reddit Answers (2024): AI Que Busca no Reddit

Em vez de Google + "reddit", o Reddit criou seu PRÓPRIO buscador AI. Resume discussões. Cita fontes. "Qual é a melhor cafeteira custo-benefício?" → Resposta baseada em 47 threads.

Google paga **$60M/ano** para treinar AI nos dados do Reddit. OpenAI também.

---

## 4. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Reddit |
| **Fundação** | Junho de 2005 |
| **Fundadores** | Steve Huffman, Alexis Ohanian |
| **IPO** | Março de 2024 (NYSE: RDDT). $28B. |
| **DAUs** | 110.4M (Q2 2025) |
| **Subreddits** | 100.000+ |
| **Preço** | Gratuito. Premium: $5.99/mês. |
| **Concorrentes** | X (Twitter), Quora, Discord, Stack Overflow |

---

## 5. Lições do Reddit

### 5.1 Conteúdo > Identidade

Reddit é a ÚNICA rede social grande onde QUEM você é não importa. Importa o que você DIZ. Não tem foto de perfil. Não tem seguidores. Não tem "influenciador."

**Lição**: existe espaço para redes sociais anônimas/pseudônimas. Nem tudo precisa ser sobre IDENTIDADE.

### 5.2 Crie Contas Falsas No Começo (Sério)

Huffman e Ohanian passaram UM MÊS fingindo ser usuários. Postando links. Criando "atividade." É o "cold start" mais HONESTO da história: "a gente fingiu até virar verdade."

**Lição**: no começo, ninguém quer ser o PRIMEIRO. Seja você o primeiro. Depois as pessoas VÊM.

### 5.3 "Vender Cedo Demais" É Um Erro Real

Huffman vendeu por $15M em 2006. O Reddit vale $28B hoje. Ele se ARREPENDE.

**Lição**: se você acredita no longo prazo do seu produto, NÃO venda no primeiro cheque. Espere.

---

## Fontes e Referências

- [ZDNet — Reddit turns 20 (2025)](https://www.zdnet.com/article/reddit-turns-20-its-incredible-journey-from-scrappy-startup-to-the-heart-of-the-internet/)
- [The Verge — Reddit origin story](https://on.theverge.com/2012/11/30/3709702/watch-this-reddit-origin)
- [Just Another PM — Reddit Product Case Study](https://www.justanotherpm.com/blog/reddit-product-case-study)
- [Koder.ai — Reddit's Distribution Engine](https://koder.ai/blog/reddits-distribution-engine-communities-moderation-ugc-learning)
- [Digital Trends — Reddit new features 2025](https://www.digitaltrends.com/phones/reddit-new-feature-post-content-check-rule-community-recommend-mobile-desktop/)
