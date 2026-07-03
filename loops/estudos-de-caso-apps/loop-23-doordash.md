# Estudo de Caso 23 — DoorDash: O Marketplace de Três Lados Que Usa Reinforcement Learning Para Balancear Velocidade e Custo

> **Data:** 2026-07-03
> **Loop:** 23 de ∞ (Reescrita — Fase 2)
> **Categoria:** Food Delivery / Marketplace / Logística
> **Tema:** Fevereiro de 2013. Quatro estudantes de Stanford — Tony Xu, Andy Fang, Stanley Tang e Evan Moore — entrevistam donos de pequenos negócios em Palo Alto para um projeto de faculdade. Uma loja de macarons mostra uma pilha de pedidos de entrega que não consegue atender. A proprietária não tem volume para contratar um motorista dedicado, mas perde vendas todos os dias porque não entrega. Os quatro constroem um protótipo em um dia: PaloAltoDelivery.com, com cardápios em PDF de oito restaurantes que não autorizaram o uso de seus nomes. Em 45 minutos, o primeiro pedido chega — comida tailandesa — e Tony Xu pega seu carro para entregar. Os fundadores fazem os primeiros duzentos pedidos pessoalmente, alternando entre programar, atender telefone e dirigir. Em março de 2013, entram no Y Combinator. Na demo day, rebatizam o projeto como DoorDash. Tony Xu passa dez semanas apresentando para investidores sem sucesso — o dinheiro está acabando. Keith Rabois (Khosla Ventures) e Saar Gur (CRV) finalmente lideram uma rodada seed de US$ 2,4 milhões. O insight que diferencia o DoorDash de Grubhub e Seamless é estrutural, não cosmético: em vez de apenas listar restaurantes que já tinham entrega própria, o DoorDash contrata e gerencia seus próprios motoristas — os Dashers. Isso abre um mercado que os concorrentes ignoravam: restaurantes de subúrbio, lanchonetes, pequenos comércios que nunca tiveram entrega. Em 2020, o IPO avalia a empresa em US$ 60 bilhões. Hoje, o DoorDash processa mais de dez bilhões de pedidos, opera robôs autônomos (Dot) e drones (Wing) na mesma plataforma de logística que os Dashers humanos, e treina agentes de reinforcement learning para ajustar os pesos do otimizador de dispatch em tempo real.

---

## 0. A Linhagem: Como o Subúrbio Americano Ficou Sem Entrega Até 2013

```
Pizza delivery (1960s-): a única comida que chegava na sua porta. Domino's. 30 minutos ou grátis.
      ↓
Grubhub/Seamless (2004-): marketplaces que listavam restaurantes. Mas o restaurante fazia a entrega.
      ↓
Postmates (2011), Caviar (2012): entrega de qualquer coisa. Premium. Cidades grandes.
      ↓
DoorDash (2013): entrega gerenciada. Motoristas próprios. Subúrbio. O mercado que ninguém via.
      ↓
DoorDash hoje (2026): 10B+ pedidos. Restaurantes, mercado, farmácia, flores, conveniência.
```

O DoorDash não inventou o delivery de comida. O que ele fez foi perceber que o modelo de marketplace — onde a plataforma conecta cliente e restaurante, mas não toca na logística — só funciona em cidades densas onde os restaurantes já têm frota própria. Nos subúrbios americanos, onde a densidade é baixa e quase nenhum restaurante entrega, o modelo de marketplace é inútil. O DoorDash resolveu isso internalizando a logística: contratando motoristas, roteirizando entregas, gerenciando a experiência ponta a ponta. Isso é mais caro de operar, mas abre um mercado que os concorrentes estruturalmente não conseguem acessar.

---

## 1. A Origem: Um Site Feito em Um Dia e Duzentos Pedidos Entregues Pelos Fundadores

Tony Xu nasceu em Nanjing, China, e imigrou para os Estados Unidos aos quatro anos. Sua mãe era médica na China, mas seu diploma não foi reconhecido nos EUA. Ela trabalhou em três empregos — incluindo lavar pratos num restaurante onde Tony, ainda criança, ajudava — por doze anos até conseguir pagar a recertificação. A carta de abertura do IPO do DoorDash, em dezembro de 2020, é endereçada a ela.

O projeto de faculdade que virou o DoorDash começou com uma observação trivial: pequenos comerciantes de Palo Alto recebiam pedidos de entrega que não podiam atender. A dona da loja de macarons guardava uma pilha de pedidos não realizados. Ela não tinha escala para pagar um motorista dedicado, mas aqueles pedidos representavam receita que ela estava deixando na mesa.

O protótipo que os quatro estudantes construíram em fevereiro de 2013 — PaloAltoDelivery.com — era primitivo. Oito restaurantes, cardápios em PDF, um número de Google Voice. A primeira refeição entregue foi comida tailandesa. Tony Xu dirigiu até o restaurante, pagou com dinheiro, colocou a sacola no banco do passageiro e entregou. Durante meses, os fundadores alternavam entre escrever código e fazer entregas. Atendiam o telefone durante a aula e saíam para dirigir entre os turnos. Fazer duzentas entregas pessoalmente ensinou mais sobre os problemas de logística de última milha do que qualquer pesquisa de mercado.

O Y Combinator aceitou o projeto no verão de 2013. Na demo day, o nome PaloAltoDelivery foi substituído por DoorDash. Tony Xu passou dez semanas apresentando o negócio para investidores. Ninguém queria investir. O dinheiro estava acabando. Keith Rabois, da Khosla Ventures, e Saar Gur, da CRV, lideraram uma rodada seed de US$ 2,4 milhões — valor modesto para o que se tornaria uma empresa de US$ 60 bilhões.

O ativo estratégico que o DoorDash construiu nos primeiros anos não era tecnológico — era geográfico. A empresa focou em subúrbios e cidades médias que Grubhub e Uber Eats ignoravam por terem densidade baixa demais para o modelo de marketplace tradicional. Nessas regiões, o DoorDash era frequentemente a única opção de entrega. Quando os concorrentes tentaram entrar, já era tarde: os restaurantes estavam integrados, os Dashers estavam na rua, e os consumidores tinham o aplicativo instalado. O mercado não era "delivery de comida nos EUA" — era "delivery no subúrbio de Columbus, Ohio". E nesse mercado, o DoorDash era o incumbente.

---

## 2. O Problema Técnico Central: Otimização Multi-Objetivo em Tempo Real

O problema de engenharia que o DoorDash resolve não é de banco de dados, nem de escalabilidade de servidores. É um problema de otimização combinatória com três funções objetivo em conflito, resolvido sob restrições de tempo real e com informação incompleta.

Quando um consumidor faz um pedido, o sistema precisa decidir qual Dasher vai atendê-lo. Essa decisão envolve prever quanto tempo o restaurante vai levar para preparar o pedido, quanto tempo o Dasher vai levar para chegar ao restaurante, quanto tempo a entrega vai levar até o consumidor, qual o impacto de agrupar este pedido com outros no trajeto do Dasher, e como todas essas variáveis interagem com as metas do marketplace: velocidade de entrega (importa para o consumidor), taxa de ocupação do Dasher (importa para o motorista), e volume de pedidos processados (importa para o restaurante).

A arquitetura que o DoorDash desenvolveu para resolver esse problema — publicada em um paper no ICML 2026 — tem duas camadas. A camada interna é um otimizador combinatório determinístico: dado um conjunto de pedidos, motoristas, restrições e pesos de objetivo, ele calcula a atribuição ótima. A camada externa é um agente de reinforcement learning (chamado OWA-RL) que ajusta os pesos do otimizador em tempo real, por loja. O agente foi treinado offline com dados históricos de marketplace e aprendeu a modular o trade-off entre velocidade de entrega e eficiência de agrupamento (batching) sem degradar a qualidade percebida pelo consumidor. Em experimentos de produção, o sistema aumentou a taxa de batching em 0,5 pontos percentuais e reduziu o tempo de espera do motorista em aproximadamente 0,86 segundos — ganhos marginais que, na escala do DoorDash, representam economias de dezenas de milhões de dólares.

A previsão de ETA é outro componente crítico. O DoorDash precisa prever três durações diferentes — preparo do restaurante, deslocamento do Dasher até o restaurante, e deslocamento até o consumidor — com informação parcial (o restaurante pode estar mais lotado do que o sistema sabe, o trânsito pode mudar). Modelos de machine learning treinados em bilhões de entregas históricas alimentam essas previsões, que por sua vez alimentam o otimizador de dispatch.

---

## 3. Dot e a Plataforma de Entrega Autônoma: Por Que o Robô Não É o Produto

Em setembro de 2025, o DoorDash anunciou o Dot — um robô de entrega autônomo de 1,37 metros de altura, 160 quilos, capaz de transportar quinze quilos de carga a 32 km/h. Mas o produto que o DoorDash construiu não é o robô. É a **Autonomous Delivery Platform (ADP)** — uma camada de orquestração que decide, para cada pedido, qual método de entrega é ótimo: Dasher humano, Dot, drone Wing, ou robô de calçada da Coco Robotics.

A ADP é uma extensão natural do otimizador de dispatch. A diferença é que agora a matriz de decisão inclui uma nova dimensão — o tipo de veículo — com restrições próprias. Um drone pode carregar pouco peso e tem alcance limitado. Um Dot pode operar em subúrbios planos mas não em prédios. Um humano é caro mas flexível. O sistema precisa decidir, em menos de cem milissegundos, qual combinação de veículos atende um conjunto de pedidos com o menor custo total respeitando todas as restrições de qualidade.

O Dot foi projetado para ser "bom o suficiente" — não o robô mais avançado do mundo, mas o robô que faz sentido econômico para entregar um tubo de pasta de dente ou um pacote de fraldas. É pequeno o suficiente para calçadas, rápido o suficiente para manter a comida quente, e barato o suficiente para ser viável em escala. A aposta do DoorDash é que a vantagem competitiva em entrega autônoma não está em construir o melhor robô, mas em construir a melhor plataforma de orquestração que utilize múltiplos tipos de robôs e humanos como recursos intercambiáveis.

---

## 4. Lições de Produto

### 4.1 O mercado que você cria é mais valioso do que o mercado que você disputa

O DoorDash não competiu com Grubhub pelo mercado de delivery em Manhattan. Criou o mercado de delivery em subúrbios onde ele não existia. Quando os concorrentes perceberam o valor desse mercado, o DoorDash já havia integrado os restaurantes, recrutado os motoristas e educado os consumidores. A lição é que mercados novos são menos contestados do que mercados existentes, e a vantagem de ser o primeiro em um mercado novo é estrutural: você define as expectativas de preço, velocidade e qualidade antes que exista uma referência.

### 4.2 O problema de dispatch não é um problema de banco de dados — é um problema de otimização combinatória

A maioria das startups de marketplace constrói um CRUD com matching simples e itera a partir daí. O DoorDash entendeu desde cedo que o problema central do negócio era matemático: atribuir N pedidos a M motoristas minimizando tempo de espera e maximizando eficiência, com informação incompleta e restrições de tempo real. Investir em otimização — primeiro com heurísticas, depois com machine learning, depois com reinforcement learning — não foi um luxo. Foi a diferença entre um marketplace que funciona e um que perde dinheiro em cada entrega.

### 4.3 Entregar os primeiros pedidos pessoalmente é a melhor pesquisa de produto que você pode fazer

Tony Xu e seus cofundadores fizeram duzentas entregas antes de contratar o primeiro Dasher. Isso é ineficiente como operação, mas extraordinariamente eficiente como pesquisa. Cada entrega era uma observação direta dos pontos de falha do sistema: o restaurante demorou mais do que o esperado, o endereço estava errado, o cliente não atendia. Nenhuma ferramenta de analytics substitui o fundador carregando uma sacola de comida tailandesa no banco do passageiro e descobrindo, em primeira pessoa, o que está quebrado.

---

## 5. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | DoorDash |
| **Fundação** | 2013 (PaloAltoDelivery). Y Combinator verão 2013. |
| **Fundadores** | Tony Xu (CEO), Andy Fang (CTO), Stanley Tang (Product), Evan Moore (saída em 2014) |
| **IPO** | 9 de dezembro de 2020 (NYSE: DASH). Preço: US$ 102. Valor de mercado: ~US$ 60B. |
| **Categoria** | Food Delivery / Marketplace / Logística |
| **Pedidos processados** | 10 bilhões+ |
| **Receita de comerciantes** | US$ 230 bilhões+ (acumulado) |
| **Dashers** | Milhões. Ganhos acumulados: US$ 80 bilhões+. |
| **Preço** | Gratuito. DashPass: US$ 9,99/mês. Comissão para restaurantes: 15-30%. |
| **Concorrentes** | Uber Eats, Grubhub, Instacart |

---

## Fontes

- [TechCrunch — Y Combinator-Backed DoorDash Delivers Food Quickly In South Bay (2013)](https://techcrunch.com/2013/07/26/y-combinator-backed-doordash-delivers-food-quickly-in-south-bay-hopes-to-expand-beyond-food/)
- [Forbes — Andy Fang, Co-Founder Of DoorDash (2022)](https://www.forbes.com/sites/joannechen/2022/07/31/american-dreamers-andy-fang-co-founder--cto-of-doordash-on-turning-a-school-project-into-feeding-america/)
- [Business Insider — Tony Xu said DoorDash seed round the hardest to raise (2020)](https://www.businessinsider.com/tony-xu-said-doordash-seed-round-the-harest-to-raise-2020-3)
- [arXiv:2606.13604 — Multi-Agent RL for Three-Sided Dispatch (ICML 2026)](https://arxiv-org.ezproxy.obspm.fr/html/2606.13604v1)
- [DoorDash Engineering — Engineering Autonomy: Building Dot and the ADP (2025)](https://careersatdoordash.com/blog/doordash-engineering-autonomy-for-local-commerce-dot-and-autonomous-delivery-platform/)
- [DoorDash Engineering — Unleashing the power of LLMs at DoorDash (2024)](https://careersatdoordash.com/blog/unleashing-the-power-of-large-language-models-at-doordash-for-a-seamless-shopping-adventure/)
- [O'Reilly — Spotlight on Data: DoorDash Global Optimality](https://www.oreilly.com/live-events/spotlight-on-data-how-doordash-solved-the-global-optimality-problem-with-raghav-ramesh/0636920284246/0636920284239/)
- [DoorDash — Dash Forward 2025: Delivering The Future of Local Commerce](https://about.doordash.com/en-us/news/doordash-unveils-dot)
