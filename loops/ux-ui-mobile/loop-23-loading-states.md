# UX/UI Loop 23 — Loading States: O Design da Espera

> **Data:** 2026-07-02
> **Loop:** 23 de ∞
> **Tema:** Nenhum app é instantâneo. Tudo CARREGA. E o que o usuário vê durante a espera define se ele FICA ou SAI. Um spinner genérico diz: "espere." Uma skeleton screen diz: "está quase pronto." Uma animação inteligente diz: "isto é tão rápido que você nem percebeu." O loading state é o design do TEMPO.

---

## 0. A Espera É Inevitável — O Sofrimento É Opcional

**Leis que TODO designer deveria conhecer:**

1. **O usuário NÃO espera.** 1 segundo = "o app está lento." 3 segundos = "travou." 10 segundos = app fechado.

2. **O cérebro PERCEBE o tempo, não o MEDE.** Uma espera com FEEDBACK parece mais curta que uma espera SEM feedback. (Psicologia: "occupied time feels shorter than unoccupied time" — William James, 1890.)

3. **A espera mais dolorosa é a INCERTA.** Spinner girando + "carregando..." = você não sabe SE vai carregar, QUANDO vai carregar, ou SE TRAVOU. 10 segundos de spinner = ABANDONO.

---

## 1. A Evolução do Loading State

| Era | Padrão | Exemplo |
|---|---|---|
| **1990s** | Barra de progresso determinada | "Installing... 47%" |
| **2000s** | Spinner (beach ball, roda dentada) | Mac OS X beach ball of death |
| **2010s** | Spinner mobile + "pull-to-refresh" | iOS UIActivityIndicatorView |
| **2015** | Skeleton screens | Facebook, LinkedIn, YouTube |
| **2020+** | Shimmer + skeleton + animação de marca | Reddit, Notion, Stripe |

**A tendência:** do spinner GENÉRICO (décadas) para a skeleton screen PERSONALIZADA (2015+) para animações com PERSONALIDADE de marca (2020+).

---

## 2. Spinner: O Mais Simples, O Mais Frustrante

O spinner (roda girando) é o loading state MAIS COMUM — e o PIOR.

**Por que spinners são RUINS:**
- ❌ Não mostram PROGRESSO (você não sabe se falta 10% ou 90%)
- ❌ Não mostram ESTRUTURA (você não sabe O QUE está carregando)
- ❌ São GENÉRICOS (não carregam marca, personalidade, contexto)
- ❌ Causam ANSIEDADE ("travou? deu erro? está carregando?")

**Quando usar spinners (RARAMENTE):**
- ✅ Operações < 1 segundo (salvar, enviar)
- ✅ Áreas minúsculas (um botão, um ícone)
- ✅ Pull-to-refresh (o gesto já é o feedback; o spinner é SECUNDÁRIO)

---

## 3. Skeleton Screens: A Revolução de 2015

**Skeleton screen** = um "esqueleto" visual do conteúdo que está carregando. Blocos cinza no formato APROXIMADO do conteúdo final.

**Por que funciona:**
- ✅ Mostra ESTRUTURA (você SABE o que vai aparecer)
- ✅ Dá sensação de PROGRESSO (o shimmer dá movimento)
- ✅ Reduz a percepção do tempo (a tela "já está lá", só falta o conteúdo)
- ✅ Carrega a MARCA (cores, formas, personalidade)

**Quem usa bem:**
- **Facebook** (2015): pioneiro. Skeletons no feed
- **LinkedIn**: skeletons nos cards de vaga
- **YouTube**: skeleton do player + descrição
- **Notion**: skeleton das páginas
- **Stripe**: skeleton dos dashboards

---

## 4. Loading States Com Personalidade

### 4.1 O "Skeleton" Que Conta Uma História
**Headspace:** em vez de spinner, uma animação de respiração. "Respire enquanto carregamos." Meditação no loading.

**Duolingo:** a coruja pisca. O personagem da marca INTERAGE com você durante a espera.

**Google Chrome:** o dinossauro (jogo offline). O "erro de conexão" vira DIVERSÃO.

### 4.2 O "Skeleton" Que ENSINA
**Airbnb:** enquanto carrega, mostra dicas de viagem. O loading vira CONTEÚDO.

### 4.3 O "Skeleton" Que DISTRAI
**Snapchat:** animação do fantasma dançando. Você nem PERCEBE que está carregando.

---

## 5. Estratégias: Como Fazer o Usuário ESQUECER Que Está Esperando

| Estratégia | Exemplo | Psicologia |
|---|---|---|
| **Otimismo** | "Quase lá..." "Preparando algo incrível..." | Expectativa positiva > ansiedade |
| **Educação** | "Dica: você pode usar atalhos..." | Aprender > Esperar |
| **Humor** | "Calibrando os fluxos quânticos..." (falso, engraçado) | Rir > Esperar |
| **Progresso real** | "47% completo" (com barra DETERMINADA) | Saber > Não saber |
| **Distração** | Joguinho, animação, easter egg | Jogar > Esperar |
| **Antecipação** | Skeleton screen do conteúdo que VAI aparecer | Ver o futuro > Esperar o presente |
| **Beleza** | Animação hipnotizante | Contemplar > Esperar |

---

## 6. O Erro: "Algo Deu Errado"

O loading state que FALHA é o pior de todos.

**O que NÃO fazer:**
- ❌ "Erro 500." (o usuário NÃO É desenvolvedor)
- ❌ "Algo deu errado." (O QUÊ? O que eu FAÇO?)
- ❌ Silêncio total. (O app TRAVOU? FECHOU? ESTÁ CARREGANDO?)

**O que FAZER:**
- ✅ "Não foi possível carregar seu feed. Verifique sua conexão."
- ✅ Botão "Tentar novamente" (GRANDE, VISÍVEL, ÓBVIO)
- ✅ "Tentar novamente" NÃO perde o estado (o que você já escreveu NÃO some)
- ✅ Se possível, mostrar conteúdo em CACHE enquanto tenta reconectar

---

## 7. O Tempo Percebido vs O Tempo Real

**Estudo (Doherty Threshold, IBM, 1982):** quando o sistema responde em <400ms, o usuário mantém o FLUXO de pensamento. Produtividade DISPARA.

**Estudo (NN/g):** 
- 0.1s = "instantâneo" (usuário não percebe)
- 1s = "fluido" (usuário nota mas mantém fluxo)
- 10s = "limite" (usuário PERDE o foco)

**Hack de percepção:** mostre ALGO imediatamente. Uma skeleton screen em 100ms PARECE mais rápida que uma tela branca + spinner por 800ms — mesmo que o conteúdo REAL demore o mesmo tempo nos dois casos.

---

## Referências

- William James (1890) — "occupied time feels shorter than unoccupied time"
- Doherty Threshold (IBM, 1982) — <400ms mantém fluxo
- Facebook (2015) — pioneiro em skeleton screens
- Apple HIG — UIActivityIndicatorView, UIProgressView
- Material Design — LinearProgressIndicator, CircularProgressIndicator
- NN/g — percepção de tempo em interfaces

---

*Fim do Loop UX/UI 23. Loading states: o design do TEMPO. Cron edf71e22 segue.*
