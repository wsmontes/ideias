# Estudo de Caso 48 — Candy Crush Saga: O Motor de Level Design Com AI Bots Que Testam 18.700+ Fases, 65 Novas Por Semana, 95% Menos Ajustes Manuais e 50% Mais Rápido

> **Data:** 2026-07-03
> **Loop:** 48 de ∞ (Reescrita)
> **Categoria:** Casual Gaming / AI-Assisted Design / Live Ops

---

## 0. Linhagem

```
Bejeweled (2001) — match-3. O gênero ancestral.
Candy Crush Saga (2012) — Facebook → mobile. Freemium. Níveis progressivos. Vidas.
King (2016) — Activision Blizzard: US$ 5,9B. Microsoft (2023): US$ 68,7B.
Candy Crush hoje (2026) — 18.700+ níveis. US$ 20B+ receita vitalícia. 5B+ downloads.
```

---

## 1. Arquitetura de Level Design

### 1.1 AI Bots Que Testam Cada Nível Milhares de Vezes

A King desenvolveu um sistema de **AI playtesting bots** que jogam cada nível milhares de vezes antes do lançamento, medindo dificuldade, "shuffles" (resets de tabuleiro), taxa de vitória e outras métricas. A aquisição da startup **Peltarion** em 2022 trouxe a tecnologia de AI que acelerou este pipeline.

**Resultados**: 50% mais rápido para alterar níveis. 95% de redução em ajustes manuais. Os bots fornecem aos designers insights altamente precisos antes do shipping.

**AI co-pilot para designers**: ferramenta que realiza tweaking automático por AI. Designers definem critérios; a AI sugere refinamentos. Designers anotam sugestões boas/ruins; o sistema aprende com o feedback.

### 1.2 A Filosofia de Dificuldade: "Difficult ≠ Fun"

Pesquisadores de data science da King descobriram que dificuldade e diversão são conceitos distintos. Medem "fun" via combinação de **time to abandon** (tempo até desistir) e **time to pass** (tempo até vencer). Níveis difíceis podem ser divertidos; níveis fáceis podem ser entediantes. **Níveis longos têm menor probabilidade de serem divertidos**. Se um nível é muito difícil, deve ser muito curto.

**Player skill profiling**: a King mapeia habilidade individual do jogador (baseada em vitórias, derrotas, tentativas) contra designs de nível para entender como diferentes jogadores experienciam o mesmo nível. Dificuldade é **personalizada** por jogador — não é a mesma para todos.

**Constant pruning**: a King identifica os 100 níveis menos divertidos do jogo e os corrige continuamente. "Very significant uplift in engagement." "Crazy hard levels never pay off" no longo prazo — retenção sempre vence.

### 1.3 65 Níveis Por Semana, 18.700+ Total

A King lança **65 novos níveis por semana**. Já lançou o **300º client version**. Manter 18.700+ níveis existentes enquanto adiciona 65 por semana exige um pipeline de **level tweaks automation**: ajustar parâmetros como move count, candy colors, blocker types ou objectives. É uma operação contínua que toca simultaneamente dificuldade, engajamento, retenção e monetização.

---

## 2. Lições de Engenharia

### 2.1 AI testing bots são o único caminho para testar 18.700 níveis

Teste manual seria impossível. Bots que jogam milhares de partidas por nível e retornam métricas de dificuldade e diversão são o que permite shipping de 65 níveis por semana com qualidade consistente.

### 2.2 Dificuldade não é diversão — e medir a diferença é o trabalho do data scientist

"Time to abandon" vs. "time to pass" como métricas separadas revelam que jogadores toleram níveis difíceis se forem curtos. Níveis longos e difíceis são onde a retenção quebra.

### 2.3 Live ops de 12 anos exige pipeline de tweaks automatizado

Manter 18.700 níveis enquanto adiciona 65 por semana requer um sistema de tweaking que mexe em parâmetros de difficulty, engagement e monetização simultaneamente — sem intervenção manual por nível.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Candy Crush Saga (King / Microsoft) |
| **Lançamento** | Abril 2012 (mobile) |
| **Categoria** | Casual Gaming / Match-3 / Live Ops |
| **Receita vitalícia** | US$ 20B+ |
| **Downloads** | 5B+ |
| **Níveis** | 18.700+ (65 novos/semana) |
| **Testes** | AI bots (milhares de partidas/nível). Peltarion (2022). 95% menos ajustes manuais |
| **Design** | Player skill profiling. Personalized difficulty. Constant pruning (100 piores níveis) |
| **Adquirida** | Activision Blizzard: US$ 5,9B (2016). Microsoft: US$ 68,7B (2023) |

---

## Fontes

- [MobileGamer.biz — How King balances human and AI-powered design in Candy Crush Saga (GDC 2025)](https://mobilegamer.biz/how-king-balances-human-and-ai-powered-design-in-candy-crush-saga/)
- [MobileGamer.biz — How King defines a 'good' Candy Crush Saga level (GDC 2025)](https://mobilegamer.biz/how-king-defines-a-good-candy-crush-saga-level-and-why-it-constantly-prunes-the-bad-ones/)
- [PocketGamer.biz — AI joins the team at King (2025)](https://www.pocketgamer.biz/ai-joins-the-team-at-king-our-challenge-is-to-create-the-right-amount-of-challenge/)
- [Neurohive.io — How AI Helped King Studio Develop 13,755 Levels (Peltarion acquisition, 50% faster, 95% reduction)](https://neurohive.io/en/ai-apps/how-ai-helped-king-studio-develop-13-755-levels-for-candy-crush-saga/)
- [MichaelChiu.io — Level Tweaks Automation at Scale (2024-2025 case study)](https://michaelchiu.io/case-study/content-engine)
