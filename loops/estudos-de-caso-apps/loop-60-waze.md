# Estudo de Caso 60 — Waze: A Arquitetura Server-Side Routing Com A* Time-Dependent, Kafka + DBSCAN, RNN/LSTM Traffic Prediction e Community Map Editor

> **Data:** 2026-07-03
> **Loop:** 60 de ∞ (Reescrita)
> **Categoria:** Navegação / Crowdsourcing / Real-Time Data

---

## 0. Linhagem

```
Mapas de papel — estáticos. Sem tráfego. Sem rerouting.
GPS dedicado (Garmin, TomTom) — routing offline. Sem crowdsourcing.
Google Maps (2005) — navegação. Dados históricos + satélite.
Waze (2008, Israel) — crowdsourced. Real-time. Community map editing. Google (2013).
Waze hoje (2026) — arquitetura distinta do Google Maps. Cloud-only. 2,4× mais reports/dia.
```

---

## 1. Arquitetura Técnica

### 1.1 Server-Side Routing Com A* Time-Dependent

Diferentemente do Google Maps (offline vector tiles + on-device routing), o Waze é **cloud-native e server-dependent**. Toda rota é calculada no servidor; o cliente só tem fallback routing offline.

**Algoritmo**: A* com edge weights time-dependent. Cada segmento de estrada tem peso baseado em: real-time speed data de GPS probes, **historical speed data** em janelas de 10-15 minutos, e **turn delay tracking** — o tempo de travessia é separado por direção de saída (virar à esquerda vs. direita vs. reto), evitando que congestionamento de uma direção penalize incorretamente o tráfego que segue reto.

**Junction penalty**: 5 segundos para novas estradas sem dados reais. Cai assim que dados de direção real se acumulam. **Route caching**: rotas frequentes cacheadas. Se conhece B→C e você pede A→C, calcula A→B e concatena. Live Map updates podem levar até 24 horas.

**Rerouting agressivo**: recalcula automaticamente quando tráfego aumenta ~15% acima do esperado — sem perguntar ao usuário. Google Maps sugere a ~25% e requer tap.

### 1.2 Kafka + DBSCAN Para Crowdsourcing Pipeline

- **Kafka-based event streaming** processa incident reports (acidentes, polícia, hazards, road closures)
- **Geospatial clustering (DBSCAN)** agrupa múltiplos reports do mesmo incidente no espaço/tempo
- **Vector-based map matching** para precisão de posição GPS
- Dados de velocidade de **GPS probes** como input primário para tempo de travessia
- **Sub-10 segundo reroute latency**
- **Protocol Buffers** como formato de dados
- **Feedback loop**: reports são confirmados/refutados por usuários subsequentes. Condições de estrada atualizadas em minutos

### 1.3 ML Para Predição de Tráfego e ETA

- **RNNs e LSTMs** para dependências sequenciais em dados de tráfego
- **Graph-based neural networks** + **spatiotemporal embeddings** para encoding da rede viária
- **Hybrid framework**: dados históricos (baseline) + dados real-time (adaptativo). Spatial features: classificação de estradas, densidade de interseções, speed limits, elevação. Temporal features: hora, dia da semana, sazonalidade, eventos
- **Cloud-based ML inference** (Google Maps usa TensorFlow Lite on-device)

### 1.4 Community Map Editor (WME)

**Segment-based road model**: estradas divididas em segmentos entre junções. Mínimo 5m. **Elevation layers**: túneis com atributo `-1`. **Turn restrictions**: editores configuram viradas permitidas/restritas. **Waze Beacons**: dispositivos Bluetooth em túneis para localização quando GPS indisponível.

**NCDOT integration (2022-2024)** : 36.000+ road closure incidents automatizados via feed JSON/XML. Waze Reverse Geocoding API para road name matching. Voluntários como QA layer.

### 1.5 Waze vs Google Maps: Arquiteturas Distintas

| Dimensão | Waze | Google Maps |
|---|---|---|
| Routing | Server-side A* | On-device + server fallback |
| Rerouting | Auto (~15%) | Sugestão (~25%, tap) |
| ML | Cloud RNN/LSTM | On-device TensorFlow Lite |
| Offline | Zero (cloud-only) | Full vector tiles |
| Dados | Apenas crowdsourced | Histórico + satélite + crowd |
| Reports/dia | 2,4× mais | Menos, com sensor fusion |

---

## 2. Lições de Engenharia

### 2.1 Server-side routing permite iteração de algoritmo sem update de cliente

Cada melhoria no A* time-dependent é deployada no servidor e disponível para todos os usuários instantaneamente. Google Maps precisa de update de app.

### 2.2 DBSCAN clustering resolve N reports do mesmo incidente

Múltiplos usuários reportando o mesmo acidente geram N eventos. Geospatial clustering agrupa reports próximos em um incidente verificado, reduzindo ruído.

### 2.3 Turn delay tracking por direção de saída é a feature mais subestimada

Separar tempo de travessia por direção evita que congestionamento de uma direção contamine as outras. Segmentos curtos demais quebram esse tracking.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Waze (Google) |
| **Fundação** | 2008 (Israel). Google: 2013 (US$ 1,1B) |
| **Categoria** | Navegação / Crowdsourcing / Real-Time |
| **Routing** | Server-side A* time-dependent. Reroute <10s. Junction penalty 5s |
| **Pipeline** | Kafka + DBSCAN + vector map matching. Protobuf |
| **ML** | RNN/LSTM + GNN + spatiotemporal embeddings. Cloud-based |
| **Map Editor** | Segment-based. Community volunteers. Beacons Bluetooth |
| **Concorrentes** | Google Maps, Apple Maps, TomTom |

---

## Fontes

- [Waze Discuss — Routing Server (community documentation, algorithm details)](https://www.waze.com/discuss/t/routing-server/379517)
- [NCDOT STIC Final Report — Automating Waze Road Closures (36K incidents, JSON/XML feed, 2022-2024)](https://connect.ncdot.gov/groups/NCSTIC/STIC%20Projects/STIC%20Incentive%20Final%20Report%20-%20Automating%20Waze%20Road%20Closures.pdf)
- [HAL Science / PeerJ 2024 — ETA Prediction Survey (RNN/LSTM, GNN, hybrid framework)](https://hal.science/hal-05314114v1/file/peerj-cs-3259.pdf)
- [News.lavx.hu — Waze vs. Google Maps: Technical Deep Dive (2024)](https://news.lavx.hu/article/waze-vs-google-maps-technical-deep-dive-into-navigation-algorithms-and-features)
