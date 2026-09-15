# Estudo de Caso 26 — Twitch: O Pipeline de Vídeo Com RTMP, Transcodificação, HLS+ABR+LL-HLS, CDN e o Chat Com IRC Sobre WebSocket, Redis Pub/Sub (<5ms) + Kafka Dual-Write, HyperLogLog e Rate Limiting Via Redis Sorted Sets

> **Data:** 2026-07-03
> **Loop:** 26 de ∞ (Reescrita)
> **Categoria:** Live Streaming / Real-Time Chat / Infraestrutura de Mídia

---

## 0. Linhagem

```
Televisão ao vivo — broadcast unidirecional. Sem feedback. Sem chat.
Justin.tv (2007) — lifecasting 24/7. Fracassa. Pivota para UGC streaming.
Twitch.tv (2011) — spin-off focado em gaming. Chat como diferenciador.
Twitch hoje (2026) — Amazon. ~100 PoPs de ingestão. Redis Pub/Sub + Kafka chat.
```

O Twitch não inventou streaming ao vivo. Emparelhou vídeo com chat em tempo real de forma que a audiência se tornasse parte do conteúdo — o chat não é complemento ao vídeo, é o produto.

---

## 1. Arquitetura Técnica

### 1.1 O Pipeline de Vídeo: Ingestão, Transcodificação e Distribuição

**Ingestão RTMP e SRT.** Streamers enviam vídeo via **RTMP** (Real-Time Messaging Protocol) para um dos ~100 PoPs globais. O ingest edge autentica a stream key, verifica IP e user agent, e retransmite via backbone privado para o cluster de transcodificação. **SRT** (Secure Reliable Transport) ganha tração como alternativa para melhor packet-loss resilience em broadcasters profissionais.

**Intelligest routing.** Substituindo HAProxy: **Capacitor** monitora recursos de compute nos origins; **The Well** monitora backbone. Randomized greedy algorithm alcança 100% compute utilization em pico global.

**Transcodificação Multi-Bitrate.** O estágio mais caro computacionalmente: decodificar stream de entrada e re-encodificar em múltiplos níveis de qualidade. Single FPGA: 120 fps VP9 real-time vs 4 fps em CPU (30× speedup). TwitchTranscoder custom: 65% mais rápido que FFmpeg, 80% menos TCO. Migração Rust (2024): latência de 220ms→85ms (-60%), custo -30%.

**Enhanced Broadcasting (2024):** multi-encode vai para GPU do streamer (NVENC) — até 5 streams concorrentes, reduzindo carga no cluster de transcodificação.

**ABR ladder típica:** Source 1080p (6.000 kbps) → 720p (4.500) → 480p (1.500) → 360p (800) → 160p (400).

**HLS + ABR + CDN.** Cada rendition é segmentada em chunks de 2-6 segundos (`.ts` para HLS) e enviada para object storage (S3-compatible) como origin. Manifest `.m3u8` lista níveis de qualidade + URLs de segmentos. Player executa ABR logic: monitora velocidade, faz upgrade/downgrade dinâmico com histerese.

**CDN:** Origin Shield (cache regional) → Edge Nodes globais (cache hit rate >99%). GeoDNS roteia viewers ao edge mais próximo. Multi-CDN para redundância e otimização de custo.

**Baixa latência.** HLS tradicional: 15-30s. Twitch usa abordagem custom **`#EXT-X-PREFETCH`** (não LL-HLS padrão Apple): persistent HTTP/1.1 + chunked transfer encoding, ~3-6s latency. WebRTC: <500ms mas não escala para milhões. Twitch opera a 5-8s para partners, ~15s para não-partners.

### 1.2 O Pipeline de Chat: IRC Sobre WebSocket, Redis Pub/Sub e Kafka Dual-Write

**Protocolo e Conexão.** Chat usa **IRC sobre WebSocket** (`wss://irc-ws.chat.twitch.tv:443`). Conexões persistentes e bidirecionais eliminam overhead de polling HTTP. Tags IRCv3 customizadas: badges (mod, subscriber), cor do nome, emotes, flags de moderação. Um servidor gerencia ~10.000 conexões (limite de file descriptors/memória).

**Fanout Cross-Server Com Redis Pub/Sub.** Para stream com 100K viewers em 10+ servidores, uma mensagem no Servidor 1 precisa chegar aos viewers nos Servidores 2-10. **Redis Pub/Sub**: cada chat server subscreve a `stream:ID:chat`. Quando mensagem é publicada, Redis entrega a todos os inscritos simultaneamente em **<5ms**. Fire-and-forget — sem persistência, sem replay.

**Kafka Para Durabilidade e Replay.** Em paralelo, cada mensagem é escrita em **tópico Kafka** particionado por stream ID. Retenção configurável (tipicamente 3h). Fornece: durabilidade, replay para late-joining viewers (fetch last N messages), processamento assíncrono (moderação ML, analytics, logging), cross-region replication via **Kafka MirrorMaker 2.0**. Latência: 50-100ms.

**O Dual-Write Pattern.** Redis para live delivery (<5ms), Kafka para durabilidade (50-100ms). Nenhum resolve ambos sozinho. Dual-write é o padrão canônico. Fan-out multiplication: 1 incoming message → milhões de outgoing deliveries (read/write ratio >10.000:1 em pico).

**Contagem de Espectadores Com HyperLogLog.** `SELECT COUNT(DISTINCT user_id)` não escala. **Redis HyperLogLog**: estrutura probabilística com **12KB de memória constante**, erro de ±0,81%. `PFADD` no heartbeat, `PFCOUNT` para contagem. Chaves rotacionadas a cada minuto para rolling window.

**Rate Limiting e Moderação.** Sliding window via **Redis sorted sets** (ZREMRANGEBYSCORE + ZCARD): ex. 2 msg/s por usuário. Slow mode: Redis key com TTL. Subscriber-only: verificação contra Redis set. Emote-only: regex validation.

**Pipeline de moderação:** keyword filter (Redis SET, O(1) lookup) → regex patterns → **classificador ML assíncrono** (Kafka consumer) → mensagens podem ser retroativamente deletadas via "retract" events. Shadow banning: usuário vê suas próprias mensagens; outros não.

**Multi-Region.** GeoDNS → clusters regionais → Redis Pub/Sub local (<5ms) + Kafka brokers locais. Kafka MirrorMaker 2.0 replica cross-region (~80-120ms). Consistência: eventual — aceitável para chat.

---

## 2. Lições de Engenharia

### 2.1 Redis Pub/Sub + Kafka dual-write é o padrão correto para chat em tempo real

Redis resolve fanout com <5ms. Kafka resolve durabilidade com retenção. Nenhum resolve ambos sozinho. Mesmo padrão do Discord (ETS + ScyllaDB).

### 2.2 HyperLogLog é a estrutura de dados correta para contagem de cardinalidade em streaming

12KB de memória constante, <1% de erro. COUNT DISTINCT não escala além de poucos milhares sem sharding.

### 2.3 Custo de banda define toda arquitetura de vídeo

50-70% das despesas operacionais. Cada decisão — codec, qualidade, edge topology, ABR aggressiveness — é tomada à sombra do custo de banda.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Twitch (Amazon) |
| **Fundação** | Justin.tv: 2007. Twitch: junho 2011. Aquisição Amazon: US$ 970M (2014) |
| **Categoria** | Live Streaming / Gaming / Real-Time Chat |
| **Ingestão** | ~100 PoPs globais. RTMP + SRT. Intelligest routing (Capacitor + The Well) |
| **Transcodificação** | Enhanced Broadcasting (NVENC streamer-side, até 5 streams). FPGA VP9 120fps (30× CPU). TwitchTranscoder (Rust, 220→85ms, -60%) |
| **CDN** | Origin Shield + Edge Nodes. Cache hit >99%. Multi-CDN. HLS via `#EXT-X-PREFETCH` (~3-6s) |
| **Chat** | IRC sobre WebSocket. Redis Pub/Sub (<5ms) + Kafka (durabilidade). HyperLogLog (12KB, ±0,81%). Rate limiting via Redis sorted sets |
| **Concorrentes** | YouTube Live, Kick, Facebook Gaming |

---

## Fontes

- [System Design — Live Chat at Scale: YouTube Live, Twitch, and 100k Concurrent Viewers (2026)](https://malukenho.github.io/post/2026/06/02/system-design-web-live-chat.html)
- [Grokking the System Design — Live Comments System Design (2025)](https://grokkingthesystemdesign.com/guides/live-comments-system-design/)
- [DeepWiki — twitch4j: Twitch API library (IRC, Helix, EventSub WebSocket)](https://deepwiki.com/twitch4j/twitch4j)
- [Ably — Definitive Guide to Building Live Streaming Chat at Scale (PDF, 2024-2025)](https://pages.ably.com/hubfs/Resources/ably-defininitive-guide-to-building-live-streaming-chat-at-scale.pdf)
- [GeeksforGeeks — Design a Live Streaming App like Twitch (Jul 2025)](https://www.geeksforgeeks.org/system-design/design-a-live-streaming-app-like-twitch-system-design/)
