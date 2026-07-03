# UX/UI Loop 12 — Dark Mode: O Modo Escuro Não É Só "Preto Em Vez De Branco"

> **Data:** 2026-07-02
> **Loop:** 12 de ∞
> **Tema:** Dark mode virou feature de marketing em 2019. iOS 13. Android 10. "Dark mode salva bateria!" "Dark mode protege seus olhos!" Mas a pesquisa CIENTÍFICA conta outra história. Dark mode PIORA a leitura para ~30% das pessoas (astigmatismo). Dark mode AUMENTA o tempo de leitura. Dark mode é MELHOR para OLED, PIOR para compreensão de texto. E quase ninguém implementa direito.

---

## 0. A História: De Necessidade Técnica a Feature de Marketing

### Anos 1970-80: Terminais (CRT)
Os primeiros monitores CRT mostravam **texto claro sobre fundo escuro** POR NECESSIDADE. O fósforo era escuro e acendia quando atingido pelo feixe de elétrons. Verde, âmbar, branco sobre preto.

### Anos 1980-90: WYSIWYG e a Metáfora do Papel
Os processadores de texto "What You See Is What You Get" (Apple Macintosh, 1984) inverteram: **texto preto sobre fundo branco.** Por quê? Porque PAPEL é branco. Tinta é preta. A metáfora do "desktop" exigia que a tela PARECESSE papel.

### 2018-2019: A "Revolução" Dark Mode
| OS | Ano |
|---|---|
| macOS Mojave | 2018 |
| iOS 13 | Set 2019 |
| Android 10 | Set 2019 |
| Windows 10 | 2016 (Anniversary Update) |

De repente, dark mode virou OBRIGATÓRIO. Todo app PRECISAVA ter dark mode. Mas a maioria só INVERTEU as cores. E isso NÃO É dark mode.

---

## 1. A Fisiologia: O Que Acontece Com Seus Olhos

### 1.1 Pupila: Por Que Dark Mode Pode PIORAR a Leitura

Em AMBIENTE CLARO (light mode):
- A pupila CONTRAI (fica menor)
- A imagem na retina fica mais NÍTIDA (maior profundidade de campo)
- Detalhes finos (texto pequeno) ficam mais FÁCEIS de ler

Em AMBIENTE ESCURO (dark mode):
- A pupila DILATA (fica maior)
- A imagem na retina fica mais BORRADA (menor profundidade de campo)
- Texto claro sobre fundo escuro sofre de **halation** (o texto parece "brilhar" e borrar)

**Susanne Mayr (Universidade de Passau), 6 estudos:**
> "In all of our studies, participants were better performing in the positive polarity condition. They detected more errors and/or read faster when dark text was presented on a light background."

### 1.2 Astigmatismo: ~1 em 3 Pessoas Sofre

Pessoas com astigmatismo têm a córnea IRREGULAR. Em dark mode, a pupila dilatada + córnea irregular = texto CLARO sobre fundo ESCURO fica SIGNIFICATIVAMENTE mais borrado. **Dark mode pode ser HOSTIL para 30% dos seus usuários.**

### 1.3 Luz Azul: O Mito

"Dark mode reduz luz azul e melhora o sono." Parcialmente verdade. Mas:
- A luz azul de telas é MÍNIMA comparada à luz do dia
- O que AFETA o sono é o BRILHO TOTAL e a DURAÇÃO da exposição, não o comprimento de onda
- Night Shift (iOS) / Night Light (Android) são mais eficazes que dark mode para reduzir luz azul

---

## 2. OLED: A Única Vantagem INQUESTIONÁVEL do Dark Mode

Em telas OLED/AMOLED, pixels PRETOS são pixels DESLIGADOS. Não consomem energia.

**Google (Pixel, OLED):**
- Google Maps em dark mode: **até 63% menos consumo** que light mode
- Brilho máximo, branco puro = **~6x mais energia** que preto puro

**Porém:**
- **Isso só vale para OLED.** iPhone XR, iPhone 11, iPads (LCD) = ZERO economia de bateria.
- **Preto PURO (#000000) economiza mais que cinza escuro (#121212).** Mas a economia do cinza é só ~1% menor.
- **A economia REAL depende do brilho da tela.** Brilho baixo = diferença pequena. Brilho máximo = diferença GRANDE.

**Conclusão:** se seu app é para dispositivos OLED, dark mode salva bateria. Se é para LCD, dark mode é puramente ESTÉTICO.

---

## 3. Design de Dark Mode: Não É Só Inverter Cores

### 3.1 O ERRO da inversão simples
```css
/* ERRADO: só inverter */
body { background: #000; color: #FFF; }
```

Resultado: texto BRANCO PURO sobre PRETO PURO. Contraste MÁXIMO. Isso MACHUCA os olhos.

### 3.2 O CERTO: Material Design Dark Theme

| Elemento | Light Mode | Dark Mode |
|---|---|---|
| **Background** | `#FFFFFF` | `#121212` (NÃO #000) |
| **Surface (cards)** | `#FFFFFF` | `#1E1E1E` |
| **Text primary** | `#000000` (87% opacity) | `#FFFFFF` (87% opacity) |
| **Text secondary** | `#000000` (60% opacity) | `#FFFFFF` (60% opacity) |
| **Error** | `#B00020` | `#CF6679` (mais CLARO) |

**Princípios:**
1. **Nunca use preto puro (#000).** Use cinza escuro (#121212). O contraste é suficiente sem ser AGRESSIVO.
2. **Nunca use branco puro (#FFF).** Use off-white (#E1E1E1). Reduz a halation.
3. **Reduza saturação.** Cores que funcionam em light mode VIBRAM demais em dark mode. Baixe 15-20%.
4. **Sem sombras.** Em dark mode, sombras NÃO são visíveis. Use SOBREPOSIÇÕES mais claras para elevação.
5. **Tipografia mais leve.** Texto claro parece mais GROSSO em fundo escuro (irradiação). Reduza o peso em 1 step (500 → 400). Aumente letter-spacing (~0.02em). Aumente line-height (1.5 → 1.6).

---

## 4. Acessibilidade: Para Quem Dark Mode REALMENTE Funciona

| Condição | Dark Mode | Light Mode |
|---|---|---|
| **Astigmatismo** (~30% da população) | ❌ PIORA (halation) | ✅ Melhor |
| **Fotofobia / enxaqueca** | ✅ Reduz brilho total | ❌ Pode desencadear |
| **Catarata / baixa visão** | ✅ Menos luz = melhor | ❌ Glare |
| **Ambiente escuro** | ✅ Confortável | ❌ "Farol nos olhos" |
| **Ambiente claro / sol** | ❌ Ileível | ✅ Melhor |
| **Leitura longa** | ❌ Mais lento | ✅ Mais rápido |
| **Consumo de mídia** | ✅ Melhor imersão | N/A |
| **Programação (código)** | ✅ Melhor syntax highlighting | ❌ Cansa mais |

**A lição:** dark mode NÃO É "melhor." É CONTEXTUAL. O usuário precisa de ESCOLHA.

---

## 5. Implementação Prática: Design Tokens para Dark Mode

### 5.1 A abordagem CORRETA: tokens semânticos com temas

```json
{
  "color-surface": {
    "$value": "#FFFFFF",        // light
    "$extensions": {
      "dark": "#121212"         // dark
    }
  },
  "color-text-primary": {
    "$value": "rgba(0,0,0,0.87)",
    "$extensions": {
      "dark": "rgba(255,255,255,0.87)"
    }
  }
}
```

A COR do token é a MESMA. O VALOR muda. O componente NÃO SABE se está em dark ou light mode.

### 5.2 O CSS mágico

```css
:root {
  --color-surface: #FFFFFF;
  --color-text: rgba(0,0,0,0.87);
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-surface: #121212;
    --color-text: rgba(255,255,255,0.87);
  }
}
```

**`prefers-color-scheme`** é a media query que lê a preferência do SISTEMA. O usuário escolheu dark mode no iOS? Seu site SABE. Automaticamente.

---

## 6. O Paradoxo: Dark Mode Aumenta o Tempo de Uso

**Twitter descobriu:** usuários em dark mode passam MAIS tempo no app. Por quê? Porque dark mode é mais CONFORTÁVEL para scrollar à noite. Você fica MAIS TEMPO porque dói MENOS.

**Isso é uma faca de dois gumes éticos:**
- ✅ O usuário está mais confortável
- ❌ O usuário está MAIS TEMPO no feed

Dark mode pode ser uma ferramenta de BEM-ESTAR (menos glare) ou de EXPLORAÇÃO (mais retenção). Depende de QUEM projeta.

---

## 7. Critérios de Teste

| Teste | Por quê |
|---|---|
| Alternar tema sem reiniciar o app | Usuário não pode perder estado |
| Texto legível em 5-100% de brilho | O usuário ajusta o brilho |
| Contraste WCAG AA (4.5:1 corpo) | Acessibilidade BÁSICA |
| Imagens com transparência (PNG) | Logo escuro sobre fundo escuro = INVISÍVEL |
| Cores semânticas (erro, sucesso) | Vermelho claro vibra no escuro |
| OLED vs LCD | Testar nos DOIS |
| Preferência do sistema | `prefers-color-scheme` deve ser respeitado |

---

## Referências

- Susanne Mayr (Universidade de Passau) — 6 estudos, positive polarity vence
- Google — Material Design Dark Theme guidelines (#121212, elevação por overlay)
- Apple HIG — Dark Mode (iOS 13+), SF Symbols, dynamic colors
- WCAG 2.2 — 4.5:1 corpo, 3:1 texto grande
- APCA (WCAG 3.0 draft) — perceptual contrast para negative polarity
- Google OLED battery research (2019) — até 63% economia no Maps
- Nielsen Norman Group — dark mode reduz velocidade de leitura

---

*Fim do Loop UX/UI 12. Dark mode: não é "preto em vez de branco." É fisiologia, acessibilidade e ética. Cron edf71e22 segue.*
