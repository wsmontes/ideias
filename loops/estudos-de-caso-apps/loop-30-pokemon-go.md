# Estudo de Caso 30 — Pokémon GO: O Jogo Que Mapeou o Mundo em 3D Enquanto Você Caçava Pikachu

> **Data:** 2026-07-03
> **Loop:** 30 de ∞ (Reescrita — Fase 2)
> **Categoria:** AR / Gaming / Infraestrutura Geoespacial
> **Tema:** 2010. John Hanke — o homem que fundou a Keyhole (adquirida pelo Google em 2004 e transformada em Google Earth) — convence Larry Page a deixá-lo montar um grupo interno no Google explorando a interseção de mapas, telefones celulares e jogos. O grupo se chama Niantic Labs, nome de um navio baleeiro que encalhou em San Francisco durante a Corrida do Ouro de 1849 e foi soterrado — uma metáfora para a ideia de que "há coisas incríveis escondidas sob a superfície". O primeiro produto da Niantic é o Field Trip (2011), um app que notifica o usuário sobre pontos históricos próximos — interessante, mas não viral. O segundo é o Ingress (2012), um jogo de realidade aumentada onde jogadores visitam locais físicos para capturar portais. O Ingress acumula 15 milhões de downloads e, crucialmente, 5 milhões de localizações submetidas por usuários — um banco de dados geoespacial que se tornaria a infraestrutura do Pokémon GO. Em 1º de abril de 2014, o Google Maps publica um vídeo de April Fools' mostrando pessoas caçando Pokémon no Google Maps. O vídeo tem 18 milhões de visualizações. Tsunekazu Ishihara, CEO da The Pokémon Company, é um jogador de Ingress nível 11 que joga com a esposa todos os dias. Ele assiste ao vídeo. Hanke assiste ao vídeo. A reunião entre Niantic e The Pokémon Company parece "inevitável". Em agosto de 2015, a Niantic se separa do Google. Google, Nintendo e The Pokémon Company investem US$ 20 milhões. Em 6 de julho de 2016, o Pokémon GO é lançado. Em um mês, fatura US$ 200 milhões. Os servidores caem repetidamente. O mundo para.

---

## 0. A Linhagem: Como Um Mapa Virtual do Mundo Real se Tornou o Tabuleiro de Jogo Mais Valioso Já Construído

```
Google Earth/Keyhole (2001-2005): imagens de satélite. Visualização. Passivo.
      ↓
Field Trip (2011): notificações baseadas em localização. Interessante, não viral.
      ↓
Ingress (2012): jogo AR. Portais em locais reais. 5M de pontos submetidos por usuários.
      ↓
Pokémon GO (2016): Ingress + Pokémon + timing perfeito. Fenômeno global.
      ↓
Niantic Lightship (2021-): plataforma de AR para desenvolvedores. VPS.
      ↓
Large Geospatial Model (2024): AI espacial treinada em bilhões de imagens de pedestres.
```

Pokémon GO não foi um jogo que deu certo. Foi uma infraestrutura — cinco anos de dados geoespaciais coletados por jogadores de Ingress — que encontrou a propriedade intelectual certa no momento em que smartphones tinham capacidade de processamento suficiente para AR. O jogo era a camada visível. O mapa era o ativo.

---

## 1. A Origem: Google Earth, April Fools' e Um CEO Que Jogava Ingress

John Hanke não era um desenvolvedor de jogos. Era um construtor de mapas. Sua empresa Keyhole — cujo nome era uma referência aos satélites espiões KH da Guerra Fria — havia sido financiada em parte pela In-Q-Tel, o braço de venture capital da CIA. Quando o Google adquiriu a Keyhole em 2004, o Earth Viewer se tornou Google Earth. Hanke passou seis anos dentro do Google gerenciando as divisões de Maps e Earth, crescendo a equipe de trinta para mais de mil pessoas.

Em 2010, ele estava inquieto. Larry Page deu permissão para um experimento: um pequeno grupo dentro do Google explorando mapas, celulares e jogos. O nome Niantic veio de uma placa que Hanke viu em San Francisco — Niantic era um navio baleeiro que encalhou durante a Corrida do Ouro. Sobreviveu como armazém e hotel. Quando queimou, foi soterrado. Anos depois, arqueólogos escavaram o local. O navio estava lá o tempo todo, debaixo da cidade. A metáfora era perfeita: há camadas de história e significado sob cada ponto do mundo físico. O trabalho da Niantic era revelá-las.

O Ingress foi o laboratório. Dois times — Enlightened e Resistance — disputando portais em locais reais. Jogadores precisavam estar fisicamente presentes em um local para capturá-lo. O jogo não era um sucesso comercial massivo, mas construiu dois ativos extraordinários: uma comunidade de jogadores dispostos a viajar para lugares aleatórios para capturar pontos virtuais, e um banco de dados de 5 milhões de portais — submetidos pelos próprios jogadores, verificados por outros jogadores, cobrindo o planeta inteiro. Cada portal tinha coordenadas GPS, foto, descrição e categoria.

Em 1º de abril de 2014, Tatsuo Nomura — um engenheiro do Google Maps — publicou um vídeo de April Fools' mostrando um "Pokémon Challenge" no Google Maps. Dezoito milhões de visualizações. A piada revelou uma demanda real. Hanke e Nomura começaram a explorar a ideia seriamente. A reunião com Ishihara foi decisiva. Ishihara era um jogador de Ingress — nível 11, jogava diariamente, conhecia o produto melhor que a maioria dos funcionários da Niantic. Ele entendeu imediatamente o potencial. O acordo foi assinado.

Quando a Niantic se separou do Google em agosto de 2015 — o Alphabet estava reorganizando a empresa e a Niantic não se encaixava em nenhuma unidade de negócios — Hanke escolheu investidores estratégicos em vez de VCs. Google, Nintendo e The Pokémon Company investiram US$ 20 milhões. Andreessen Horowitz e Kleiner Perkins avaliaram o negócio em US$ 150 milhões e passaram. Subestimaram o valor do banco de dados de portais do Ingress. Subestimaram o valor de uma franquia de vinte anos com um bilhão de fãs. Subestimaram o apelo primitivo de ver um Pikachu no seu quintal.

O Pokémon GO foi construído sobre um motor completamente novo — não o motor do Ingress. Hanke insistiu nisso. O motor V1 do Ingress teria sido "absolutamente esmagado" pela escala de lançamento. A reconstrução levou meses adicionais, mas significou que o jogo aguentou — mal, com quedas constantes de servidor, mas aguentou — a demanda explosiva. Os PokéStops e Gyms foram extraídos diretamente do banco de dados de portais do Ingress: os melhores portais viraram Gyms, o resto virou PokéStops. Os habitats das espécies foram definidos por sobreposição de dados geográficos: corpos d'água geravam Pokémon aquáticos, parques geravam Pokémon de grama, dados climáticos e de solo influenciavam distribuições.

Em um mês, o jogo faturou US$ 200 milhões. O valuation da Niantic saltou para mais de US$ 3 bilhões.

---

## 2. A Infraestrutura Que o Jogo Construiu: VPS, Lightship e o Modelo Geoespacial

O Pokémon GO foi o caso de uso que financiou a construção de uma infraestrutura de AR muito mais ambiciosa do que o jogo sugeria. O **Visual Positioning System (VPS)** é um serviço de nuvem que localiza um dispositivo com precisão centimétrica — não por GPS, que tem erro de vários metros em áreas urbanas, mas por comparação da imagem da câmera com um mapa 3D global construído a partir de scans enviados por jogadores. Um único frame comprimido da câmera é suficiente para determinar a posição e orientação do dispositivo com seis graus de liberdade.

O mapa 3D que alimenta o VPS é construído por crowdsourcing. Jogadores que optam por escanear PokéStops contribuem clipes de quinze a trinta segundos — aproximadamente trezentos frames cada. O pipeline de processamento divide os scans em segmentos, corrige deriva de GPS, reconstrói geometria 3D via "Many-Depth" (estimativa de profundidade por frame), aplica segmentação semântica (árvore, prédio, chão, céu, água) e otimiza globalmente o resultado via bundle adjustment. O resultado são malhas 3D texturizadas e semanticamente anotadas de locais reais. Em 2024, a Niantic tinha mais de dez milhões de locais escaneados globalmente, um milhão de locais ativados para VPS, e recebia aproximadamente um milhão de novos scans por semana.

O **Lightship ARDK** é a plataforma de desenvolvimento que expõe essa infraestrutura para terceiros. A versão 3.0, lançada em 2024, oferece meshing em tempo real usando apenas a câmera RGB — sem LiDAR — e segmentação semântica com mais de vinte classes. Suporta co-localização multiplayer (até dez jogadores no mesmo espaço AR, sincronizados via VPS).

O **Large Geospatial Model (LGM)** , anunciado em novembro de 2024, é a aposta mais ambiciosa. Treinado em bilhões de imagens de perspectiva de pedestre — coletadas via Pokémon GO, Ingress e Scaniverse — o LGM aprende características comuns entre localizações. Reconhece uma igreja vista por trás mesmo que todas as imagens de treinamento mostrem a fachada. Prevê a geometria de ambientes a partir de visões parciais. É um modelo de fundação para espaço físico: o equivalente geoespacial do que GPT é para texto.

---

## 3. Lições de Produto

### 3.1 O jogo pode ser a interface para construir infraestrutura

Pokémon GO gerou bilhões em receita. Mas o ativo mais valioso que produziu não foram os Pokémon — foi o mapa 3D do mundo construído por jogadores que escaneavam PokéStops. A Niantic entendeu que um jogo de sucesso não é um fim em si mesmo; é um motor de coleta de dados que financia a construção de infraestrutura para a próxima plataforma. O mesmo padrão se aplica a Tesla (carros coletam dados para direção autônoma) e Google (buscas coletam dados para treinar modelos de AI).

### 3.2 O banco de dados de localizações pré-existente era o verdadeiro produto

O Pokémon GO não teria funcionado sem os 5 milhões de portais do Ingress. Esses portais não foram criados por uma equipe de curadores — foram submetidos por jogadores ao longo de anos, de graça, por diversão. O banco de dados de localizações era o fosso competitivo que nenhum concorrente conseguiria replicar rapidamente. A lição é que dados geoespaciais gerados por usuários são extraordinariamente difíceis de copiar e extraordinariamente valiosos quando combinados com a propriedade intelectual certa.

### 3.3 O timing de uma piada pode revelar um mercado

O vídeo de April Fools' do Google Maps não era um estudo de mercado — era uma piada. Mas dezoito milhões de visualizações são um sinal tão forte quanto qualquer pesquisa de mercado. A Niantic tratou o sinal como real e construiu o produto.

---

## 4. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Pokémon GO |
| **Lançamento** | 6 de julho de 2016 |
| **Desenvolvedor** | Niantic, Inc. (spin-off do Google, 2015) |
| **Fundador** | John Hanke (CEO) |
| **IPO** | Não (privada). Valuation: ~US$ 9B (pico). |
| **Downloads** | 1 bilhão+ |
| **Receita vitalícia** | US$ 6 bilhões+ |
| **Locais VPS** | 10M+ escaneados, 1M ativados |
| **Scans/semana** | ~1 milhão |
| **Preço** | Gratuito. In-app purchases (PokéCoins). |
| **Concorrentes** | Jurassic World Alive, Minecraft Earth (RIP), Monster Hunter Now |

---

## Fontes

- [Game Developer — Tracing Pokémon GO's roots back to Meridian 59](https://www.gamedeveloper.com/design/tracing-i-pok-mon-go-i-s-roots-back-to-the-90s-mmorpg-i-meridian-59-i-)
- [CBC News — How Pokémon Go went from Google prank to mobile gaming phenomenon](https://www.cbc.ca/news/science/pokemon-google-origins-1.3690769)
- [Mashable — How the gurus behind Google Earth created Pokémon GO](https://mashable.com/article/john-hanke-pokemon-go)
- [GamesBeat — The accidental history of Niantic's Pokémon GO, as told by John Hanke](https://gamesbeat.com/the-accidental-history-of-niantics-pokemon-go-as-told-by-john-hanke/)
- [Niantic Labs — Engineering Pokémon Playgrounds: VPS](https://nianticlabs.com/news/pokemon-playgrounds)
- [Niantic Labs — Lightship VPS Part 2: Building Our 3D Map](https://nianticlabs.com/news/vps-part-2)
- [Game Developer — Niantic's new AI model built by Pokémon GO players (2024)](https://www.gamedeveloper.com/business/niantic-pokemon-go-ai-model)
- [PCMag — Pokémon GO players helped train AI for spatial intelligence (2024)](https://uk.pcmag.com/ai/155407/ever-played-pokemon-go-you-helped-train-an-ai-for-spatial-intelligence)
