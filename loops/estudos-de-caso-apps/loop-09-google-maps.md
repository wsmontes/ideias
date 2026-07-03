# Estudo de Caso 09 — Google Maps: O App Que Mapeou o Mundo Inteiro (E Virou Utilidade Pública)

> **Data:** 2026-07-02
> **Loop:** 9 de ∞
> **Categoria:** Mapas / Navegação / Geospatial / Infraestrutura
> **Tema:** 8 de fevereiro de 2005. O Google lança o Google Maps nos EUA. Foram necessárias TRÊS aquisições em outubro de 2004 (Where2 Technologies, Keyhole, ZipDash) + um time interno (Google Local) + 4 meses de integração frenética. A Where2 — fundada pelos irmãos dinamarqueses Lars e Jens Rasmussen em Sydney, Austrália — inventou o "slippy map": em vez de recarregar a página inteira a cada movimento (como MapQuest), carregava TILES de 256×256 pixels ao redor da área visível. O usuário ARRASTAVA o mapa como se fosse papel. Essa inovação RIDÍCULA de simples mudou TUDO. Os tiles de mapa eram pré-renderizados e cacheados em CDN, então o servidor NEM PRECISAVA trabalhar. Hoje: 2 bilhões de usuários mensais, 780 bilhões de km navegados por ano, 170 bilhões de imagens de Street View, 250 milhões de lugares indexados. Esta é a história do app que transformou "onde fica isso?" em "você chegou ao seu destino."

---

## 0. A Linhagem: Como a Humanidade Deixou de se Perder

```
Mapas de papel (séc. 6 AEC → presente): estáticos. Escala fixa. Você se LOCALIZA.
      ↓
MapQuest (1996): primeiro mapa digital na web. Mas cada movimento RECARREGAVA a página.
      ↓
GPS para civis (2000): Clinton remove "selective availability." Precisão sai de ~100m para ~5m.
      ↓
Where2 Technologies (2003-2004): "slippy maps." Tiles. AJAX. Sem reload. REVOLUÇÃO.
      ↓
Google Maps (2005): Where2 + Keyhole (satélite) + ZipDash (tráfego) + Google Local (busca).
      ↓
Google Maps Navigation (2009, Android): GPS turn-by-turn GRÁTIS. Matou a Garmin e a TomTom.
      ↓
Waze (adquirido 2013): crowdsourcing de trânsito. Comunidade.
      ↓
Google Maps hoje (2026): 2B MAUs. AI (Gemini). Immersive View. Live View (AR). 20 anos.
```

O Google Maps não inventou o mapa digital (MapQuest), nem o GPS (governo americano), nem o tráfego crowdsourced (Waze). Mas inventou o MAPA DIGITAL QUE FUNCIONA — rápido, fluido, global, gratuito, em qualquer dispositivo. E construiu, ao longo de 20 anos, a MAIOR e MAIS PRECISA base de dados geográficos já criada pela humanidade.

---

## 1. A Origem: Dinamarqueses na Austrália e 3 Aquisições em 1 Mês

### Where2 Technologies: Os Irmãos Que Dormiam no Sofá da Mãe

**2003.** Sydney, Austrália. Os irmãos dinamarqueses **Lars e Jens Rasmussen** fundam a **Where2 Technologies** com os australianos **Noel Gordon** e **Stephen Ma**.

- Foram demitidos no estouro da bolha. Jens dormia no SOFÁ DA MÃE na Dinamarca.
- Lars foi para a Austrália, juntou-se a Gordon. Trabalhavam num QUARTO DE HÓSPEDES alugado em Hunters Hill, Sydney.
- **9h às 17h, fins de semana OFF** — condição para usar o quarto.
- Cartões de crédito ESTOURADOS. Ma trabalhava num posto de gasolina. Gordon cortava tecido na fábrica do sogro.

O protótipo se chamava **Expedition** — um app desktop em C++.

### A Inovação: "Slippy Maps" (Mapas que Deslizam)

O MapQuest (líder em 2003) funcionava assim: você via um mapa. Clicava numa seta. A página INTEIRA recarregava. Experiência HORRÍVEL.

Os Rasmussen criaram uma solução GENIAL:
- Dividir o mapa em **tiles de 256×256 pixels.**
- Carregar NÃO SÓ os tiles visíveis, mas também os tiles AO REDOR (fora da tela).
- Quando o usuário arrastava o mapa, os tiles já estavam PRONTOS. Zero loading.
- **AJAX puro**, antes do termo "AJAX" existir.

O mapa parecia um PAPEL infinito que você DESLIZAVA com o mouse. Revolucionário.

### A Aquisição (Outubro de 2004)

A Where2 estava quebrada. O VC **Ram Shriram** conectou os irmãos ao Google. Larry Page foi DIRETO:

> *"We like the web."*

Page queria um app WEB, não um app desktop (Expedition era C++). Deu 3 SEMANAS para a Where2 fazer a versão web. Eles fizeram. O Google comprou.

No MESMO MÊS, o Google comprou a **Keyhole** (Earth Viewer, imagens de satélite, parcialmente financiada pela CIA — In-Q-Tel) e a **ZipDash** (dados de tráfego em tempo real). Internamente, tinha o time do **Google Local** (busca por localização, liderada por Dan Egnor).

**4 times. 3 aquisições. 1 produto. 4 meses para integrar TUDO.**

### 8 de Fevereiro de 2005

Google Maps é lançado nos EUA. Sem satélite ainda (chegaria em abril). Em 4 meses, um time de ~50 pessoas processou TERABYTES de dados de satélite. Sergey Brin, ao ver a lista de cidades a comprar:

> *"Why don't we do all of it?"*

Compraram TODAS as imagens de satélite disponíveis no mercado.

### 2009: O Golpe de Misericórdia nos GPS

**Google Maps Navigation** para Android. Turn-by-turn GPS. DE GRAÇA. As ações da Garmin e da TomTom DESPENCARAM. Um app de celular tinha MATADO uma indústria de bilhões de dólares.

---

## 2. A Filosofia do Produto: O Mapa É o Protagonista

### "Map-Centric Design" (2024-2025)

A filosofia de UX mais recente do Google Maps é CLARA e SIMPLES: **o mapa SEMPRE visível.**

Toda decisão de design em 2024-2025 segue este princípio:
- **Bottom sheets flutuantes** em vez de menus full-screen. O mapa SEMPRE aparece por trás.
- **Cantos arredondados** nas sheets. O mapa "vaza" pelas bordas.
- **Ações fixas no rodapé.** Directions, Start, Share — nunca scrollam para longe.
- **Três níveis de sheet**: minimizada (só a search bar) → meia-altura (mapa visível) → altura total.

Como resumiu o Android Authority: *"Google Maps feels like it's more about the actual map."*

### Os Pilares do Produto

| Pilar | O que significa |
|---|---|
| **Precisão** | Se o mapa diz "vire à direita em 200m", a rua PRECISA estar lá. A distância PRECISA estar certa. |
| **Velocidade** | Tiles em CDN. Roteamento em <100 microssegundos (Contraction Hierarchies). Sub-second rendering. |
| **Cobertura** | 250M+ lugares. 170B+ imagens Street View. 16M+ milhas mapeadas. 87 países no Street View. |
| **Gratuidade** | O Maps é GRÁTIS. Para TODO MUNDO. Monetizado indiretamente (ads localizados, API paga para empresas). |
| **Onipresença** | Android, iOS, Web, Car (Android Auto), Watch (Wear OS, Apple Watch), TV. A mesma experiência, adaptada. |

---

## 3. As Inovações Que o Google Maps Trousse ao Mundo

### 3.1 Slippy Maps / Tiles (2005)

A inovação FUNDACIONAL. O mapa não "carrega" — ele DESLIZA. Tiles de 256px pré-carregados ao redor da área visível.

- **20+ níveis de zoom.** z0 = mundo inteiro em 1 tile. z20 = detalhe de calçada (10¹² tiles).
- **Pré-renderizados até z14** (~268M tiles). Níveis mais profundos sob demanda.
- **>95% de cache hit na CDN.** Os mesmos tiles servem MILHÕES de usuários.

### 3.2 Street View (2007)

Larry Page teve a ideia: "e se a gente FOTOGRAFASSE cada rua do mundo?"

Hoje:
- **170 bilhões de imagens panorâmicas.**
- **16 milhões de milhas de estrada.**
- **87 países.**
- Carros com câmera 360° + LiDAR + GPS. Também mochilas, bikes, snowmobiles, CAMELOS (deserto).

**Como as imagens viram panoramas:**
1. Múltiplas câmeras fotografam simultaneamente.
2. Optical flow warping alinha imagens sobrepostas (corrige parallax, calibração).
3. Spline-based flow field faz blending suave.
4. Ceres Solver (open-source Google) resolve os parâmetros.

### 3.3 Google Maps Navigation Gratuito (2009)

Antes de 2009, GPS turn-by-turn custava CARO (Garmin: $200-500 + atualizações de mapa PAGAS). O Google Maps Navigation para Android era GRÁTIS. Com atualizações GRATUITAS. Com tráfego em tempo real.

A Garmin perdeu 70% do valor de mercado. A TomTom perdeu 80%. Um app matou uma indústria.

### 3.4 Live View (AR Walking Navigation, 2019)

Realidade Aumentada para PEDESTRES. Aponte a câmera para a rua. Setas GIGANTES indicam para onde ir. Nomes de rua flutuam no ar.

Usa **VPS (Visual Positioning Service)** — o Google compara o que a câmera VÊ com sua base de dados de Street View para determinar EXATAMENTE onde você está (precisão de CENTÍMETROS). Muito mais preciso que GPS em "urban canyons."

### 3.5 Immersive View (2022-2023)

Combine **Street View + imagens de satélite + AI** para criar um MODELO 3D FOTORREALISTA de uma cidade. Você SOBREVOA o trajeto ANTES de fazer. Vê o trânsito PREVISTO para a hora que você vai sair. Vê o TEMPO previsto.

**Como funciona**: Neural Radiance Fields (NeRF) + bilhões de imagens. AI reconstrói geometria 3D com iluminação e sombras realistas.

### 3.6 AI / Gemini Integration (2024-2025)

- **"Ask about this place"**: botão Gemini permanente no rodapé. "Esse restaurante é bom para crianças?" "Tem estacionamento?"
- **AI-powered search**: "rolê de sábado à noite em Pinheiros que não seja caro." Linguagem NATURAL.
- **Resumos de lugares**: AI condensa milhares de reviews em parágrafos sobre "ambiente", "comida", "serviço."

---

## 4. A Linha do Tempo do Design Visual

```
2005-2008 — Web. Mapa bege + azul. Estradas coloridas (inspiradas em mapas rodoviários dinamarqueses).
  ↓
2009-2012 — Mobile. Android: app nativo. iOS: app nativo (substituiu pré-instalado da Apple em 2012).
  ↓
2013-2014 — Material Design 1.0. Cards brancos. FAB. Ícone de pin colorido.
  ↓
2015-2019 — Refinamento incremental. Timeline (histórico de localização). Offline maps.
  ↓
2020-2023 — Material You (cores dinâmicas no logo, NÃO no app). Live View (AR). Immersive View.
  ↓
2024 — Bottom bar reduzida (5→3 tabs). Sheets redesign. TEAL substitui AZUL. Ícone de pin.
  ↓
2025 — Material 3 Expressive. Gradient icon. "Maps 20th anniversary." Logo preto/branco.
```

### Identidade Visual Atual

| Elemento | Especificação |
|---|---|
| **Cor primária** | **Teal** (verde-azulado). Substituiu o azul tradicional em 2024. Considerado "mais moderno, mais suave para os olhos." |
| **Cores do ícone** | Pin com as 4 cores Google (azul, vermelho, amarelo, verde). 2025: gradiente AI-era. |
| **Fundo** | Mapa: tons pastéis (bege, verde claro, azul água). Clean. Legível. |
| **Tipografia** | **Google Sans** (Android, Web). **SF Pro** (iOS). System fonts. |
| **Ícones** | Material Design 3 icons. Outline (light mode), filled (selecionados). |
| **Bottom bar** | 3 tabs: Explore, You, Contribute. Teal para o ativo. |

### O Pin: De Ícone de UI a Ícone Cultural

O pin do Google Maps — uma gota invertida com um círculo no topo — é um dos ícones mais RECONHECÍVEIS do mundo. Não precisa de texto. Não precisa de contexto. Você VÊ o pin e sabe: "mapa, localização, aqui."

A versão de 2025 moderniza: pin mais fino, círculo interno MAIOR, gradiente de 4 cores substituindo as partições diagonais azuis.

---

## 5. Anatomia do App Mobile Google Maps (2025-2026)

### 5.1 Estrutura

```
┌──────────────────────────────┐
│  🔍 Search here           👤│  ← Search bar (sticky no topo)
│                             │
│  ┌─────────────────────────┐│
│  │                         ││
│  │      🗺️ MAPA            ││  ← Mapa domina 80% da tela
│  │     (teal + pastel)      ││     Tiles vetoriais. Fluid. Zoomável.
│  │                         ││
│  │  📍 pins de lugares      ││
│  │  🚗 tráfego (verde/      ││
│  │     laranja/vermelho)    ││
│  │                         ││
│  └─────────────────────────┘│
│                             │
│  ┌─────────────────────────┐│  ← Bottom sheet (meia-altura)
│  │ Nearby places           ││     Cantos arredondados.
│  │ 🍔 🛒 ⛽ 🏨 ☕        ││     Mapa visível por trás.
│  └─────────────────────────┘│
│                             │
├──────────────────────────────┤
│ 🧭 Explore │ 👤 You │ ✏️ Contribute│ ← Bottom bar (3 tabs)
└──────────────────────────────┘
```

### 5.2 As 3 Tabs (Redução de 5 para 3 em 2024)

| Tab | Função |
|---|---|
| **🧭 Explore** | Descobrir lugares. Busca, categorias, recomendações. |
| **👤 You** | Seu histórico, listas, reviews, Timeline. Pessoal. |
| **✏️ Contribute** | Adicionar lugares, reviews, fotos, edits. Comunidade. |

### 5.3 O Sheet (Folha) Como Paradigma

A grande inovação de UX de 2024-2025 é o **sheet** (folha) que flutua sobre o mapa:

- **Minimizado**: você vê o mapa + search bar.
- **Meia-altura**: lugares próximos. Categorias. Puxou para cima = abre um pouco mais.
- **Altura total**: detalhes do lugar. Reviews. Fotos. Botão "Directions" FIXO no rodapé.

O mapa NUNCA desaparece completamente. Você SEMPRE vê onde está.

---

## 6. Arquitetura Técnica: Contraction Hierarchies e Tiles

### Stack

| Componente | Tecnologia |
|---|---|
| **Tiles** | Mapbox Vector Tile (Protobuf, delta-compressed). Pré-renderizados até z14. CDN global. |
| **Roteamento** | **Customizable Contraction Hierarchies (CCH)**. Sub-100 microssegundos. |
| **Busca** | Elasticsearch (~250M places). Edge-ngram autocomplete. |
| **Geospatial** | Bigtable, Spanner, geospatial DBs proprietários. |
| **Tráfego em tempo real** | GPS probes (bilhões/dia) → Kafka → Flink → edge weights (1-5 min). |
| **ML / ETA** | GNN (DeepMind, 2020). Transformer (Uber-style). Correção residual. |
| **Armazenamento** | S3/GCS para tiles. Bigtable para dados geo. Spanner para dados transacionais. |

### O Algoritmo de Roteamento (A Alma do Maps)

O Google Maps precisa calcular a rota entre DOIS pontos em QUALQUER lugar do planeta. Em menos de 1 milissegundo.

**Como eles fazem isso:**

**1. Contraction Hierarchies (CH)**:
- **Pré-processamento (semanal)**: os nós do grafo (intersecções) são ordenados por importância. Os MENOS importantes são "contraídos" — atalhos (shortcut edges) são criados para pular sobre eles.
- **Customização (a cada 1-5 min)**: pesos das arestas são atualizados com tráfego EM TEMPO REAL.
- **Query**: busca bidirecional "para cima" (só segue nós mais importantes). Quando os dois caminhos se encontram, a rota está pronta.

**2. GNN/DeepMind para ETA (2020)**:
- A rede rodoviária é dividida em "Supersegments."
- Um Graph Neural Network com message passing prevê o tempo de travessia para cada segmento.
- **+97% de precisão de ETA.** O GNN reduziu erros remanescentes em 50%+.
- Treinado em 20+ anos de dados históricos + tráfego em tempo real.

**3. ML como correção residual**:
- O motor de roteamento dá um ETA base.
- Modelos de ML corrigem esse ETA usando padrões históricos + features em tempo real.
- Latência total: **poucos milissegundos.**

### Como o Tráfego é Detectado

1. **Bilhões de GPS probes** de telefones Android (anônimos, opt-in) + sensores municipais.
2. **Kafka** ingere os dados em tempo real.
3. **Flink** agrega velocidades por segmento de rua.
4. **Pesos das arestas** são atualizados no CCH a cada 1-5 minutos.
5. **O mapa fica VERDE (livre), LARANJA (moderado), VERMELHO (congestionado).**

### Por Que Tiles Vetoriais?

Google Maps migrou de raster tiles (PNG) para **vector tiles** (Protobuf):
- **2-5× menores.** Menos banda. Mais rápido.
- **Rotação, tilt, restyling no CLIENTE.** Sem round-trip ao servidor.
- **Versionados e imutáveis** por dataset version. Cache invalidation = mudar a URL prefix.

---

## 7. Público-Alvo e Mercado

### Os Números

| Métrica | Valor |
|---|---|
| **MAUs** | 2 bilhões+ |
| **Km navegados/ano** | 780 bilhões |
| **Lugares indexados** | 250 milhões+ |
| **Street View imagens** | 170 bilhões+ |
| **Milhas Street View** | 16 milhões+ |
| **Países Street View** | 87 |
| **Reviews bloqueadas (fraude)** | 170 milhões+ (2023) |
| **Preço** | Gratuito. API paga para empresas (Maps Platform). |

### Quem Usa?

**TODO MUNDO.** Literalmente. O Google Maps é uma UTILIDADE PÚBLICA:
- Motoristas: navegação turn-by-turn.
- Pedestres: Live View AR.
- Ciclistas: rotas de bike.
- Transporte público: horários, conexões.
- Viajantes: explorar lugares, restaurantes, hotéis.
- Empresas: API para logística, delivery, ride-hailing.
- Governos: planejamento urbano, resposta a desastres.

---

## 8. Críticas e Pontos de Dor

| Problema | Detalhe |
|---|---|
| **Privacidade / Location History** | O Google SABE onde você esteve. Timeline liga os pontos. "Útil" ou "assustador"? |
| **Street View desatualizado** | A maioria das imagens tem 1-3 anos. Áreas rurais podem ter 5+ anos. |
| **Dados crowdsourced = viés** | Editores locais podem adicionar informações ERRADAS. Reviews FALSAS. |
| **Qualidade variável por país** | Alemanha = Street View PRECISO. Coreia do Norte? "Aqui há dragões." |
| **Waze foi comprado... e esquecido?** | Google comprou Waze (2013, $1B) mas NUNCA integrou de verdade. Waze ainda existe separado. |
| **Complexidade crescente** | 250M lugares. Categorias. Reviews. Fotos. Horários. AI. O app está PESADO. |
| **No dynamic color** | Maps é um dos POUCOS apps Google que NÃO adotou Material You dynamic color. Usa teal estático. |

---

## 9. Lições do Google Maps para Quem Constrói Apps Mobile

### 11.1 O Mapa É o Protagonista — Não a UI

O Google Maps entendeu (finalmente, em 2024) que a UI NÃO PODE competir com o mapa. Sheets flutuantes. Cantos arredondados. O mapa SEMPRE visível.

**Lição**: em apps de DADOS VISUAIS (mapas, gráficos, dashboards, canvas), a INTERFACE precisa SER os dados. Menos chrome. Mais conteúdo.

### 11.2 Pré-Compute Tudo que Puder

Tiles pré-renderizados. Contraction Hierarchies pré-processadas. CDN com >95% de cache hit. O Maps é RÁPIDO porque 99% do trabalho já foi feito ANTES de você perguntar.

**Lição**: compute offline. Sirva online. Tudo que pode ser pré-processado DEVE ser pré-processado.

### 11.3 "Grátis" Mata Indústrias

O Google Maps Navigation GRATUITO matou Garmin e TomTom. O Google Maps GRATUITO matou MapQuest. Quando um gigante oferece DE GRAÇA o que você VENDE, você precisa de um plano B.

**Lição**: se seu modelo de negócios depende de COBRAR por algo que um gigante pode oferecer DE GRAÇA como loss leader, você está EM PERIGO.

### 11.4 Separe Dados ESTÁTICOS de DINÂMICOS

Tiles são versionados e IMUTÁVEIS (CDN, cache infinito). Dados de tráfego são atualizados a cada 1-5 MINUTOS (streaming, real-time). O caminho de leitura é COMPLETAMENTE separado.

**Lição**: dados que mudam em velocidades DIFERENTES precisam de arquiteturas DIFERENTES. Não coloque tiles e tráfego na mesma pipeline.

### 11.5 "Why don't we do ALL of it?"

Quando mostraram a lista de cidades para comprar imagens de satélite, Sergey Brin disse: "por que não TODAS?" O Google comprou TUDO.

**Lição**: às vezes, a decisão certa é "vamos fazer TUDO." Não "vamos testar em 3 cidades." Mapas são um problema de ESCALA. Meio mapa não serve para NADA.

---

## 12. Ficha Técnica do App

| Atributo | Valor |
|---|---|
| **Nome** | Google Maps |
| **Desenvolvedor** | Google LLC |
| **Lançamento** | 8 de fevereiro de 2005 (web). 2008 (Android). 2012 (iOS próprio). |
| **Aquisições-chave** | Where2 (2004), Keyhole (2004), ZipDash (2004), Waze (2013, $1B) |
| **Categoria** | Mapas / Navegação / Geospatial |
| **Plataformas** | Android, iOS, Web, Android Auto, CarPlay, Wear OS, Apple Watch, TV |
| **MAUs** | 2 bilhões+ |
| **Preço** | Gratuito. API: Maps Platform (paga para empresas). |
| **Design System** | Material Design 3 Expressive. Teal accents. Bottom sheets. Vector tiles. |
| **Cores** | Teal (primário), Verde/Vermelho/Laranja (tráfego), Pastel (mapa base) |
| **Tipografia** | Google Sans (Android/Web), SF Pro (iOS) |
| **Tech Stack** | CCH routing, Bigtable, Spanner, Elasticsearch, Kafka, Flink, GNN, CDN global |
| **Tamanho do app** | ~150-300MB (Android, offline maps + cache) |
| **Avaliação** | 4.2★ (App Store), 4.3★ (Google Play) |
| **Concorrentes** | Apple Maps, Waze (também Google), HERE Maps, OpenStreetMap, TomTom |

---

## 13. Linha do Tempo Visual

```
2003 — Where2 Technologies fundada em Sydney. "Slippy maps."
2004 — Outubro: Google compra Where2 + Keyhole + ZipDash. "Façam versão web em 3 semanas."
2005 — 8 de fevereiro: Google Maps lançado nos EUA.
2005 — Abril: Satélite adicionado. "Why don't we do ALL of it?" — Sergey Brin.
2007 — Street View lançado. Carros com câmera 360°.
2008 — App Android. Mapa no bolso.
2009 — Google Maps Navigation (GPS turn-by-turn GRÁTIS). Garmin e TomTom DESPENCAM.
2012 — Apple Maps lança (DESASTRE). Google Maps para iOS vira app independente.
2013 — Google compra Waze ($1B). Crowdsourced traffic.
2014 — Material Design 1.0.
2017 — Timeline (histórico de localização).
2019 — Live View (AR walking navigation).
2020 — DeepMind GNN para ETA. -50% erro residual.
2022 — Immersive View (NeRF 3D cities).
2023 — Gemini AI no Maps. "Ask about this place."
2024 — Bottom bar reduzida (5→3). Teal accent. Bottom sheets redesign.
2025 — 20 anos. Gradient icon. Maps pin redesign. M3 Expressive.
2026 — AI-first Maps. Immersive View expandido. "Maps as a platform."
```

---

## Fontes e Referências

- [The Guardian — Google Maps: a decade of transforming the mapping landscape](https://www.theguardian.com/technology/2015/feb/08/google-maps-10-anniversary-iphone-android-street-view)
- [Tech Guide — Google Maps is 20 years old — developed in Australia](https://www.techguide.com.au/news/internet-news/google-maps-is-20-years-old-but-did-you-know-it-was-developed-in-australia/)
- [Brisbane Times — Revealed: the whiteboard doodles that became Google Maps](https://www.brisbanetimes.com.au/technology/revealed-the-whiteboard-doodles-that-became-google-maps-20140826-108een.html)
- [Bluewin — From your desk to your pocket: 20 years of Google Maps](https://www.bluewin.ch/en/news/from-your-desk-to-your-pocket-20-years-of-google-maps-li.2550826)
- [9to5Google — New Google Maps bottom bar with 3 tabs (2024)](https://9to5google.com/2024/05/16/new-google-maps-bottom-bar/)
- [9to5Google — Google Maps redesign goes all in on sheets](https://9to5google.com/2025/01/21/google-maps-redesign-sheets/)
- [Android Authority — Google Maps Sheets UI: 'mappier than ever'](https://www.androidauthority.com/google-maps-sheets-3548003/)
- [Gadgets360 — Google Maps Update With Redesigned Layout](https://www.gadgets360.com/ai/news/google-maps-update-android-new-bottom-sheet-layout-interface-6111422)
- [PhoneArena — Google Maps teal color update](https://www.phonearena.com/news/google-maps-for-android-buttons-ui-elements-new-color-update-rolling-out_id165781)
- [WebProNews — Google Maps Android UI Update: Map-Centric Design and Teal Accents](https://www.webpronews.com/google-maps-android-ui-update-map-centric-design-and-teal-accents/)
- [Hook Design — Branding the 20th Anniversary of Google Maps](https://byhook.com/work/branding-the-20th-anniversary-of-google-maps)
- [Android Authority — Google Maps teal accent static not dynamic color](https://www.androidauthority.com/google-maps-android-teal-3507581/)
- [Scrap.io — Google Maps Route Planning Technology (2026)](https://scrap.io/google-maps-route-planning-technology-20-years)
- [GrokkingTheSystemDesign — Google Maps System Design Guide](https://grokkingthesystemdesign.com/guides/google-maps-system-design/)
- [Educative — Detailed Design of Google Maps](https://www.educative.io/interview-prep/system-design/detailed-design-of-google-maps)
- [Gizmodo — The Secret Inner Workings of Google Maps](https://gizmodo.com/the-secret-inner-workings-of-google-maps-5941270)

---

> **Próximo loop:** Estudo de Caso 10 — a definir. Sugestões: **Tinder** (dating/swipe), **Duolingo** (educação/gamificação), **Strava** (fitness/social), **Netflix** (streaming), **YouTube** (vídeo), **Nubank** (fintech), **Telegram** (mensageria), **Slack** (produtividade empresarial).
>
> A sequência natural continua com **Tinder** (dating/swipe) no loop-10.
