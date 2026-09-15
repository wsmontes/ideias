# Estudo de Caso 63 — BeReal: A Arquitetura De Notificação Simultânea (2 Minutos, Dual Camera), O Feed Cronológico Anti-Algoritmo e a Aquisição Pela Voodoo (2024)

> **Data:** 2026-07-03
> **Loop:** 63 de ∞ (Reescrita)
> **Categoria:** Rede Social / Autenticidade / Anti-Algoritmo

---

## 0. Linhagem

```
Instagram (2010) — feed algorítmico. Curadoria. Filtros. Performance.
Snapchat (2011) — efemeridade. Stories. Câmera primeiro.
BeReal (2020, França) — notificação simultânea. 2 minutos. Dual camera. Sem filtros.
Voodoo (2024) — aquisição por €500M. Fim da era independente.
```

---

## 1. Arquitetura Técnica

### 1.1 O Mecanismo de Notificação Simultânea

O BeReal envia uma notificação push simultânea para todos os usuários em um horário aleatório diferente a cada dia. O usuário tem **2 minutos** para capturar e postar uma foto usando as duas câmeras simultaneamente (frontal + traseira). Após o timer expirar, ainda pode postar, mas marcado como "late."

**Infraestrutura de push**: Firebase Cloud Messaging (Android) + APNs (iOS) para entrega simultânea para milhões de dispositivos. O desafio técnico é coordenar o envio para dezenas de milhões de dispositivos dentro de uma janela de poucos segundos — um problema de **thundering herd** que o Discord e o Roblox também enfrentam, mas com latência ainda mais crítica.

**Dual camera capture**: iOS AVFoundation (AVCaptureMultiCamSession) e Android Camera2/CameraX para captura simultânea das câmeras frontal e traseira em um único frame. Postagem com layout picture-in-picture: foto traseira como fundo, frontal como sobreposição.

### 1.2 O Feed Cronológico Anti-Algoritmo

Diferentemente de toda rede social desde 2016, o feed do BeReal é **estritamente cronológico**. Sem algoritmo de engajamento. Sem curadoria. Sem anúncios (na era independente). O feed "My Friends" mostra posts na ordem em que foram publicados. O feed "Discovery" mostra posts públicos globais, também cronológicos.

**RealMojis**: reações via selfie — o usuário tira uma foto de sua expressão facial que aparece como "emoji" no post do amigo. É essencialmente uma mini-foto de reação, processada e enviada como imagem.

### 1.3 Aquisição Pela Voodoo (2024)

Junho 2024: Voodoo ( publisher francês de jogos mobile ) adquire BeReal por **€500 milhões**. A Voodoo é conhecida por monetização agressiva via anúncios e in-app purchases em jogos hyper-casual — essencialmente o oposto da filosofia anti-anúncio e anti-algoritmo do BeReal.

---

## 2. Lições de Engenharia

### 2.1 Notificação simultânea para dezenas de milhões é thundering herd como serviço

Coordenar push notifications para milhões de dispositivos dentro da mesma janela de segundos exige infraestrutura de entrega massivamente paralela. Firebase e APNs são os backbones; o servidor BeReal coordena o timing.

### 2.2 Dual camera capture é um problema de API de SO que muda a cada versão

AVCaptureMultiCamSession no iOS e Camera2/CameraX no Android têm comportamentos diferentes entre versões de SO e modelos de dispositivo. Manter captura dual-camera consistente em centenas de dispositivos é o principal desafio de engenharia mobile do app.

### 2.3 Anti-algoritmo é um posicionamento de produto, não uma vantagem competitiva

Ser "anti-Instagram" funcionou como narrativa de crescimento, mas não como modelo de negócio. Sem algoritmo de engajamento, sem anúncios direcionados, sem monetização — o BeReal provou que autenticidade gera downloads, não receita.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | BeReal |
| **Fundação** | 2020 (França). Fundadores: Alexis Barreyat, Kévin Perreau |
| **Aquisição** | Voodoo: €500M (Jun 2024) |
| **Categoria** | Rede Social / Autenticidade / Foto |
| **Core** | Push simultâneo (FCM+APNs). Dual camera (AVCaptureMultiCamSession/Camera2). 2 min timer |
| **Feed** | Cronológico. Sem algoritmo. Sem filtros. RealMojis (reação via selfie) |
| **Concorrentes** | Instagram, Snapchat, TikTok Now (descontinuado) |

---

## Fontes

- [Voodoo acquisition announcement — BeReal acquired for €500M (Jun 2024)](https://www.voodoo.io/)
- [MobileAppCircular — Cost to Build an App like BeReal (2025, architecture overview)](https://mobileappcircular.com/cost-to-build-an-app-like-bereal-in-2025-complete-guide-990342766b63)
