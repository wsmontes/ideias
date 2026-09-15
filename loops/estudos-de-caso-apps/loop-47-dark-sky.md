# Estudo de Caso 47 — Dark Sky: O Algoritmo de Nowcasting Que Previa Chuva Com Precisão de Minutos Usando Radar, Optical Flow e GPU — Até a Apple Matá-lo e Integrá-lo ao WeatherKit

> **Data:** 2026-07-03
> **Loop:** 47 de ∞ (Reescrita)
> **Categoria:** Clima / Previsão / Data Visualization

---

## 0. Linhagem

```
Previsão do tempo tradicional — meteorologistas + TV. "30% de chance de chuva hoje."
Dark Sky (2012) — previsão hiperlocal minuto a minuto. "Chuva começando em 17 minutos."
Apple Weather (2023) — Dark Sky integrado. WeatherKit API. App original desligado.
```

---

## 1. Arquitetura Técnica

### 1.1 O Algoritmo de Nowcasting em 5 Etapas

O Dark Sky não tentava prever chuva com modelos atmosféricos complexos. Usava **nowcasting**: extrapolação de observações atuais para previsão de curtíssimo prazo (0-60 minutos).

**1. Noise Filtering com FANN.** Dados de radar das 140+ estações NOAA contêm ruído: reflexos de solo, pássaros, insetos. Descartar todo sinal de baixa intensidade remove bordas de tempestade (valiosas). Dark Sky treinou uma **Fast Artificial Neural Network (FANN)** para reconhecer a "textura" distinta do ruído, atingindo 90-95% de identificação com near-zero falsos positivos.

**2. Computer Vision para Storm Velocity.** Sistemas meteorológicos se comportam aproximadamente de forma linear em escalas de minutos. Um algoritmo de CV extrai vetores de movimento de tempestades a partir de imagens de radar consecutivas — essencialmente **optical flow** aplicado a precipitação.

**3. GPU-Based Prediction.** Campos de velocidade alimentam a GPU para extrapolar movimento e renderizar a animação de nowcast para os próximos 60 minutos.

**4. Error Monitoring.** Previsões são continuamente comparadas com radar real para trackear erro por estação em tempo real.

**5. Hyperlocal GPS Adjustment.** A previsão é ajustada para fatores microclimáticos: elevação, inclinação, distância de corpos d'água, efeitos de ilha de calor urbana.

**Limitação**: "Our system cannot predict conditions beyond 6 hours." Além de 60 minutos, a previsão degrada rapidamente — o caos atmosférico torna impossível prever chuva com precisão de minutos usando apenas extrapolação de radar.

### 1.2 O App e a API

O app iOS (US$ 3,99) foi lançado em 2012. A **Dark Sky API** (2012-2023) permitia a qualquer desenvolvedor acessar as mesmas previsões hiperlocais. Processava milhões de requisições por hora, servindo apps de viagem, agricultura, construção civil e milhares de apps de clima.

### 1.3 Aquisição e Morte (Apple, 2020-2023)

Março 2020: Apple adquire por valor não divulgado. App Android descontinuado imediatamente. Setembro 2022: app iOS removido da App Store. Janeiro 2023: app para de funcionar. Março 2023: API desligada. Tecnologia integrada ao Apple Weather (iOS 16+) e exposta como **WeatherKit API** (Swift + REST, pricing de US$ 49,99/mês para 1M chamadas).

A transição não foi suave: múltiplas outages do Apple Weather em 2023-2024, atribuídas à complexidade de integrar múltiplas fontes de dados globais (NOAA, ECMWF, JMA) com o motor do Dark Sky.

---

## 2. Lições de Engenharia

### 2.1 Nowcasting é um problema de visão computacional, não de modelagem atmosférica

Extrapolar movimento de células de precipitação visíveis no radar usando optical flow é mais eficaz para 0-60 minutos que qualquer modelo atmosférico complexo. Você não precisa entender física da atmosfera para prever onde uma célula de chuva estará em 15 minutos.

### 2.2 Construir seu negócio sobre uma API de terceiro é risco existencial

Milhares de apps dependiam da Dark Sky API. Quando a Apple a desligou, cada um precisou refatorar seu backend de clima — com degradação de qualidade.

### 2.3 Aquisições por big tech frequentemente matam produtos, não os salvam

A Apple comprou o Dark Sky para extrair a tecnologia, não para operá-lo. App desligado, API descontinuada, time absorvido. Destino padrão de startups adquiridas por plataformas.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Dark Sky |
| **Lançamento** | 2012. App: US$ 3,99 |
| **Fundadores** | Adam Grossman, Jack Turner |
| **Aquisição** | Apple: março 2020. Shutdown: jan 2023 |
| **Tecnologia** | FANN noise filtering + optical flow + GPU extrapolation + hyperlocal GPS adj. |
| **API** | Milhões de req/hora. Desligada mar 2023 |
| **Sucessor** | Apple Weather (iOS 16+) + WeatherKit API |

---

## Fontes

- [Dark Sky blog (archive.org) — How Dark Sky works](https://web.archive.org/web/20221201000000*/https://blog.darksky.net/)
- [TechCrunch — Apple acquires Dark Sky (Mar 2020)](https://techcrunch.com/2022/09/13/as-apples-weatherkit-launches-dark-sky-for-ios-to-wind-down-operations-by-year-end/)
- [MacRumors — Dark Sky shuts down (Jan 2023)](https://www.macrumors.com/2023/01/01/dark-sky-shuts-down-tomorrow/)
- [Weather4Cast 2024 — Optical flow + cGAN for nowcasting (Deshpande et al.)](https://arxiv.org/pdf/2412.00451)
