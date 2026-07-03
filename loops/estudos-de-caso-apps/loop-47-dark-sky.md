# Estudo de Caso 47 — Dark Sky: O App de Clima Que a Apple Comprou, Matou (E Enterrou no iOS 16)

> **Data:** 2026-07-03
> **Loop:** 47 de ∞ (Fase 3: Utilidades & Design)
> **Categoria:** Clima / Previsão do Tempo / Data Visualization
> **Tema:** 2011. Adam Grossman e Jack Turner lançam um Kickstarter para um app de previsão do tempo. Arrecadam ~$40.000. Em 2012, lançam o **Dark Sky.** O diferencial: previsão **hiper-local,** minuto a minuto. "Vai chover daqui a 8 minutos." Não "30% de chance de chuva hoje." MINUTOS. O app usava MACHINE LEARNING — zero meteorologistas humanos. *"Humanos são HORRÍVEIS em prever o tempo."* O design era uma OBRA-PRIMA de visualização de dados. A tela principal mostrava EXATAMENTE o que você precisava saber num PISCAR DE OLHOS. Nada mais. Nada menos. Em março de 2020, a **Apple comprou.** Fechou o app Android. Fechou a API. Em janeiro de 2023, **matou o app iOS.** As features foram "absorvidas" pelo Apple Weather no iOS 16 — mas todo mundo que USOU o Dark Sky sabe: a ALMA se perdeu. Esta é a história do app que PROVOU que design de informação IMPORTA — e que quando a Apple te compra, ela compra sua TECNOLOGIA, não sua ALMA.

---

## 1. A Origem: Kickstarter, $40K e "Vai Chover Em 8 Minutos"

### Adam Grossman: O Fundador Que Odiou Meteorologistas

- Lançou o Dark Sky via **Kickstarter em 2011.** ~$40.000.
- **2012**: app no iOS. $3.99.
- Filosofia desde o dia 1: **"Humanos são HORRÍVEIS em prever o tempo. Deixe os COMPUTADORES fazerem isso."**

> *"When it comes to weather forecasting, it's best to leave it to the computers."* — Adam Grossman, 2014

### A Inovação: Previsão Hiper-Local, Minuto a Minuto

- "Vai chover daqui a 8 minutos." NÃO "30% de chance de chuva."
- GPS seguia sua localização EXATA. Previsão para o SEU quarteirão.
- **Project Quicksilver**: o mapa de temperatura de MAIS ALTA resolução do mundo. Cada pixel ≤12.5 milhas quadradas.

### Apple Compra (Março de 2020)

- Valor: NÃO DIVULGADO.
- Android: MORTO. API: MORTA. Website: MORTO.
- iOS: sobreviveu até **1º de janeiro de 2023.**
- Features foram para o Apple Weather (iOS 16). Mas o DESIGN... não.

---

## 2. A Filosofia: "O Que Você REALMENTE Quer Saber?"

### O "Glance" — Uma Olhadela de Menos de 1 Segundo

> *"O que um ciclista quer saber quando OLHA para o céu escuro e pega o telefone?"*

A tela principal mostrava:
1. **Temperatura atual** + ícone simples.
2. **Linha do tempo de chuva** (ANIMADA só se chuva fosse esperada).
3. **Próximas horas** — barras de temperatura. Tendência visível SEM ler números.
4. **Próximos dias** — barras horizontais. "Você não precisa LER os números para entender a TENDÊNCIA."

### Os Princípios de Design

| Princípio | Como o Dark Sky Aplicava |
|---|---|
| **Contexto > completude** | Mostrava SÓ o relevante AGORA. "Não TUDO." |
| **Expandir, não navegar** | Cada elemento expandia NO LUGAR. Sem abrir novas telas. |
| **Tendências > precisão** | Barras horizontais mostravam subida/descida sem precisar ler números. |
| **Cor com RESTRIÇÃO** | VERMELHO só para alertas de severidade do NWS. O ÚNICO splash de cor. |
| **Movimento = significado** | Timeline de chuva ANIMADA só quando chuva era esperada. |

---

## 3. O Legado: "Requiem For An Interface"

Em 2023, a ACM *interactions* publicou um artigo chamado **"Requiem for an Interface"** — uma EULOGIA ao design do Dark Sky.

> *"Dark Sky was not just an app; it was a paradigm shift in how we perceive and interact with weather data."*

> *"The interface embodied Vitruvian principles: commodity, firmness, and delight."*

O Apple Weather (iOS 16+) TEM a tecnologia do Dark Sky. Mas NÃO TEM a alma. "Larded with too much nerdy exactitude and blinking, flashing colors."

---

## 4. Lições do Dark Sky

### 5.1 "O Que o USUÁRIO Quer Saber?" > "O Que os DADOS Podem Mostrar?"

A tela principal do Dark Sky respondia UMA pergunta: "vai chover?" Em SEGUNDOS. O Apple Weather mostra TUDO.

**Lição**: design de INFORMAÇÃO é EDITAR. Cortar. Remover. "O que o usuário REALMENTE precisa?" Não "o que CABE na tela?"

### 5.2 Quando a Apple Te Compra, Ela Compra Sua TECNOLOGIA, Não Sua ALMA

A Apple absorveu a TECNOLOGIA do Dark Sky. Mas MATOU o design. O app morreu. A API morreu. O Android morreu.

**Lição**: se uma big tech te compra, entenda: ela quer seus DADOS, seus ALGORITMOS, seus ENGENHEIROS. Não sua INTERFACE.

### 5.3 Um App PODE Ser Uma Obra-Prima de Design

Dark Sky é estudado em cursos de HCI como EXEMPLO CANÔNICO de design para "situações de uso." "O que um ciclista precisa saber num GLANCE?"

**Lição**: design de interface NÃO é "deixar bonito." É entender PROFUNDAMENTE o contexto de uso.

---

## Fontes e Referências

- [Fast Company — Dark Sky Reinvents Weather Apps With Hyper-Local Forecasts (2011)](https://www.fastcompany.com/1665389/dark-sky-reinvents-weather-apps-with-hyper-local-forecasts)
- [Fast Company — How Dark Sky Is Changing Weather Forecasting With ML (2016)](https://www.fastcompany.com/3063991/how-dark-sky-is-changing-weather-forecasting-with-machine-learning)
- [ACM interactions — Requiem for an Interface (2023)](https://interactions.acm.org/archive/view/september-october-2023/requiem-for-an-interface)
- [Nightingale — A Eulogy for Dark Sky, a Data Visualization Masterpiece (2023)](https://nightingaledvs.com/dark-sky-weather-data-viz/)
- [BBC — Apple buys weather app Dark Sky (2020)](https://www.bbc.com/news/technology-52115095)
