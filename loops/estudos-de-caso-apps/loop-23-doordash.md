# Estudo de Caso 23 — DoorDash: O DeepRed (MIP + Gurobi), O OWA-RL (Double DQN + CQL, ICML 2026, 100M Inferências/Dia, Switchback Em 4.000 Regiões) e a Engine de ETA Com Mixture of Experts (DeepNet + CrossNet + Transformer) + Distribuição Weibull

> **Data:** 2026-07-03
> **Loop:** 23 de ∞ (Reescrita)
> **Categoria:** Food Delivery / Marketplace / Otimização Combinatória / RL

---

## 0. Linhagem

```
Pizza delivery (1960s-) — Domino's. Único segmento com logística própria.
Grubhub/Seamless (2004-) — marketplace: conecta mas não entrega.
DoorDash (2013) — logística própria. Motoristas gerenciados. Subúrbios.
DoorDash hoje (2026) — 10B+ pedidos. DeepRed MIP. OWA-RL. ETA MoE + Weibull.
```

O DoorDash não inventou delivery de comida. Internalizou a logística — contratando motoristas, roteirizando entregas e gerenciando a experiência ponta a ponta — para criar um mercado que não existia: delivery em subúrbios americanos onde densidade é baixa e nenhum restaurante tem frota própria.

---

## 1. Origem

Quatro estudantes de Stanford — Tony Xu, Andy Fang, Stanley Tang e Evan Moore — entrevistaram donos de pequenos negócios em Palo Alto para um projeto de faculdade. Uma loja de macarons mostrou uma pilha de pedidos de entrega que não conseguia atender. Construíram um protótipo em um dia: PaloAltoDelivery.com, 8 restaurantes, cardápios em PDF. Em 45 minutos, o primeiro pedido: comida tailandesa. Tony Xu dirigiu até o restaurante, pagou em dinheiro e entregou. Os fundadores fizeram os primeiros 200 pedidos pessoalmente — alternando entre código, telefone e direção. Fazer duzentas entregas pessoalmente ensinou mais sobre os pontos de falha da logística de última milha do que qualquer pesquisa de mercado.

---

## 2. Arquitetura Técnica

### 2.1 DeepRed: O Motor de Dispatch Com Mixed-Integer Programming e Gurobi

O problema central do DoorDash é atribuir N pedidos a M Dashers disponíveis, minimizando três objetivos em conflito: **ASAP** (consumer-facing latency — tempo até a comida chegar), **XCAT** (excess courier-side effort — tempo ocioso do motorista) e **batching efficiency** (agrupar múltiplos pedidos no mesmo trajeto). Esperar mais para agrupar ↑ batching mas ↑ ASAP. Despachar imediatamente ↓ ASAP mas ↓ batching e ↑ XCAT. O trade-off ótimo varia por loja, horário e condições de mercado.

**DeepRed** resolve isso com uma arquitetura de duas camadas:

**Camada ML (prediction).** Modelos na plataforma Sibyl ML predizem os inputs do otimizador: tempo de preparo do restaurante (histórico da loja, hora do dia, volume de pedidos), tempo de deslocamento (tráfego, distância, clima) e probabilidade de aceitação do Dasher (histórico individual, distância, valor do pedido). Modelos produzem intervalos de confiança, não estimativas pontuais — crítico para que o otimizador tome decisões conservadoras quando a incerteza é alta.

**Camada MIP (optimization).** Um solver de **mixed-integer programming (Gurobi)** recebe os inputs e calcula a atribuição ótima entre pedidos e Dashers. O MIP modela restrições como parte do problema: um Dasher só pode estar em um lugar por vez, um pedido não pode esperar mais que T minutos, um Dasher carrega no máximo K pedidos simultâneos. A função objetivo é uma soma ponderada de scores de qualidade (velocidade, on-time delivery) e eficiência (utilização do Dasher, batching).

**Três desafios de integração ML-MIP:** (1) **Garbage-in/garbage-out**: se o modelo prediz 5 min de preparo mas o restaurante leva 20, o MIP otimiza sobre dados errados. Solução: calibrar incerteza. (2) **Overfitting de parâmetros**: pesos da função objetivo ajustados manualmente — um peso ótimo às 12h em Manhattan é péssimo às 20h em Columbus. Solução: Bayesian optimization + OWA-RL. (3) **Cascading variability**: atrasar dispatch esperando batching propaga atraso — restaurante começa mais tarde, Dasher chega mais tarde, consumidor recebe mais tarde. Solução: penalty terms limitam atraso máximo de dispatch.

### 2.2 OWA-RL: Reinforcement Learning Offline Que Ajusta Pesos em Produção

O paper *"Multi-Agent Reinforcement Learning from Delayed Marketplace Feedback for Objective-Weight Adaptation in Three-Sided Dispatch"* (ICML 2026 Workshop on RLxF, Haochen Wu, Yi Hou, Shiguang Xie — DoorDash) descreve um sistema de RL em produção.

**Arquitetura de duas camadas aninhadas:**

- **Inner layer — Combinatorial Assignment Optimizer (DeepRed).** Tratado como black box pelo agente RL.
- **Outer layer — OWA-RL.** Um agente que seleciona um **multiplicador discreto** (valores: 0.8, 0.9, 1.0, 1.1, 1.2) aplicado ao peso ASAP no objetivo do otimizador — deslocando o trade-off entre velocidade e batching. Espaço de ação pequeno e discreto limita risco de ações catastróficas.

**Treinamento: Double DQN + Conservative Q-Learning (CQL) offline.** Double DQN reduz overestimation bias usando behavior-network para action selection e target-network para evaluation. CQL penaliza Q-values de ações fora da distribuição de treinamento — `log Σ exp Q(s,a') - Q(s,a)`, com parâmetro de força ajustável η. Crítico porque o agente nunca encontra estados OOD em treinamento offline.

**Treinamento centralizado, execução descentralizada**: uma única value function treinada com dados agregados de todas as lojas. Cada loja tem sua própria policy que observa features locais e seleciona ação.

**State features**: contagem de entregas pendentes na loja, tempo mediano de espera do courier nos últimos 30 minutos, pressão de oferta localizada (razão Dashers/pedidos na região).

**Reward**: combinação ponderada de ASAP e XCAT, agregada regionalmente para lidar com feedback de marketplace atrasado — uma decisão de dispatch leva dezenas de minutos para produzir feedback observável.

**Produção: global switchback experiment** — ~4.000 regiões geográficas, 2 semanas, intervalos de 2h alternando OWA-RL vs. baseline (pesos estáticos). CUPED para redução de variância. **~100 milhões de inferências por dia**, cadência de **20 segundos** por loja.

| Métrica | Efeito | Significância |
|---|---|---|
| Batching rate | +0,50 pp | p < 0,001 |
| Courier Wait Time (CWT) | −0,86 s | p = 0,004 |
| Courier Active Time (CAT) | −1,26 s | p = 0,019 |
| ASAP (customer latency) | +0,97 s (n.s.) | p = 0,264 |
| % 20-min late | Sem mudança | — |

O sistema melhorou eficiência do courier sem degradar qualidade de entrega para o consumidor.

### 2.3 A Engine de ETA: Mixture of Experts Com Transformer e Distribuição Weibull

O sistema de previsão de ETA precisa prever três durações distintas (preparo do restaurante, deslocamento até o restaurante, deslocamento até o consumidor), cada uma com distribuição estatística diferente e respondendo a sinais de mercado diferentes. A arquitetura atual (outubro 2024, *"Precision in Motion"*, DoorDash Engineering Blog) substituiu modelos baseados em árvore por uma **MLP-gated Mixture of Experts (MoE)** com três encoders paralelos:

**DeepNet Encoder.** Múltiplas camadas fully-connected capturando relações não-lineares de features numéricas, embeddings categóricos e séries temporais agregadas. Processa bucketized numerical features (travel duration, order subtotal) com robustness a outliers.

**CrossNet Encoder.** Inspirado no DCN v2 (recomendação). Parâmetros de crossing aprendíveis como matrizes de baixo rank com mecanismo de gating. Modela interdependências entre features temporais, espaciais e de tipo de pedido — ex.: "hora do rush" × "distância" × "fast food vs. serviço completo".

**Transformer Encoder.** Self-attention sobre séries temporais de sinais de mercado em granularidade de 5 minutos: volume de pedidos na loja, relação oferta/demanda de Dashers, velocidade média de tráfego. Embeddings posicionais aprendíveis permitem capturar "volume cresceu 3× nos últimos 15 minutos" — crítico para responder a mudanças rápidas.

Os três encoders rodam em paralelo. Um **MLP decoder** combina outputs atuando como mecanismo de gating implícito — sem rede de gating separada (não melhorava performance).

**Features**: embeddings para high-cardinality categoricals (stores, time buckets, tipos de loja, taxonomias de itens). Embeddings espaciais via **H3 hexagon indexing** para pick-up/drop-off locations em múltiplas granularidades. Time series features com positional embeddings aprendíveis em buckets de 5 minutos.

**Modelagem probabilística com Weibull.** Tempos de entrega seguem distribuição de cauda longa — a maioria em 20-30 min, minoria em 45-60 min. A distribuição **Weibull** (forma k, escala λ, localização γ) captura essa cauda; Gaussiana não. **Interval regression** resolve o problema da likelihood produzir γ negativo. O modelo prevê os três parâmetros Weibull, produzindo uma distribuição completa de probabilidade, não apenas estimativa pontual.

**Multi-task learning**: shared foundation layer (3 encoders) + task-specific lightweight heads para cada caso de uso (homepage, store page, checkout, Dasher delivery, consumer pickup). Casos infrequentes (pickup) beneficiam de transfer learning.

**Resultado**: melhoria relativa de **20% na acurácia de ETA**, melhor responsividade a condições dinâmicas de mercado (especialmente eventos de undersupply de Dashers).

---

## 3. Inovações

**3.1 OWA-RL como outer loop de otimização.** Em vez de substituir o otimizador MIP por ML end-to-end, o OWA-RL atua como camada externa que modula os pesos do otimizador. O MIP garante restrições; o RL adapta o trade-off. É uma arquitetura que preserva as garantias do otimizador enquanto adiciona adaptabilidade.

**3.2 RL offline seguro para produção financeira.** Espaço de ação discreto (5 valores), Double DQN + CQL, sem exploração em produção, switchback em 4.000 regiões. Demonstra que RL pode ser deployado com segurança em sistemas que movimentam dinheiro real.

**3.3 MoE com Transformer para ETA probabilístico.** Três encoders especializados — DeepNet para padrões não-lineares, CrossNet para interações de features, Transformer para dinâmica temporal — combinados com saída probabilística Weibull. Reconhece que diferentes componentes do ETA têm estruturas estatísticas diferentes e exigem mecanismos de aprendizado diferentes.

---

## 4. Lições de Engenharia

### 4.1 Dispatch é um problema de otimização com ML embedado, não de ML

ML prediz inputs; MIP otimiza atribuição. Separar responsabilidades permite que o MIP garanta restrições (um Dasher não está em dois lugares) enquanto o ML lida com incerteza (tempo de preparo). Sistemas ML end-to-end frequentemente produzem soluções que violam restrições.

### 4.2 RL offline com espaço de ação restrito é seguro para produção

OWA-RL: 5 ações discretas, Double DQN + CQL offline, switchback global. RL em produção não requer resolver exploração segura — requer restringir o problema.

### 4.3 Weibull é a distribuição correta para tempos de entrega

A diferença entre 25 min e 35 min não importa para o consumidor. A diferença entre 35 min e 60 min define se ele pede de novo. Modelar a cauda direita — o que a Weibull faz e a Gaussiana não — previne as experiências que causam churn.

---

## 5. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | DoorDash |
| **Fundação** | 2013 (PaloAltoDelivery). IPO: dez 2020 (NYSE: DASH) |
| **Fundadores** | Tony Xu (CEO), Andy Fang (CTO), Stanley Tang (Product) |
| **Categoria** | Food Delivery / Marketplace / Otimização Logística |
| **Dispatch** | DeepRed: ML layer + MIP layer (Gurobi solver) |
| **RL** | OWA-RL: Double DQN + CQL offline, 100M inferências/dia, cadência 20s, switchback 4.000 regiões |
| **ETA** | MLP-gated MoE: DeepNet + CrossNet + Transformer, distribuição Weibull, H3 embeddings, multi-task, +20% acurácia |
| **Concorrentes** | Uber Eats, Grubhub, Instacart |

---

## Fontes

- [ICML 2026 Workshop on RLxF — Multi-Agent RL from Delayed Marketplace Feedback for OWA in Three-Sided Dispatch (Wu, Hou, Xie, DoorDash)](https://arxiv.org/abs/2606.13604)
- [DoorDash Engineering Blog — Using ML and Optimization to Solve DoorDash's Dispatch Problem (DeepRed, MIP, Gurobi)](https://careersatdoordash.com/blog/using-ml-and-optimization-to-solve-doordashs-dispatch-problem/)
- [DoorDash Engineering Blog — Precision in Motion: Deep Learning for Smarter ETA Predictions (MoE, Weibull, H3, Out 2024)](https://careersatdoordash.com/blog/deep-learning-for-smarter-eta-predictions/)
- [DoorDash Engineering Blog — Improving ETAs with Multi-Task Models, Deep Learning, and Probabilistic Forecasts (Mar 2024)](https://careersatdoordash.com/blog/improving-etas-with-multi-task-models-deep-learning-and-probabilistic-forecasts/)
