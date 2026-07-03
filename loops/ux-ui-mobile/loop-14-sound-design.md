# UX/UI Loop 14 — Sound Design: A Camada Sonora Que Nenhum Designer Projeta

> **Data:** 2026-07-02
> **Loop:** 14 de ∞
> **Tema:** Bauhaus, Rams, Ive, Material Design — todos são VISUAIS. Mas o mobile tem FALANTES. E microfones. E o som é a camada de UX mais NEGLIGENCIADA do design digital. Um "ding" bem projetado pode definir uma marca. Um "bzzz" mal projetado pode fazer o usuário desinstalar seu app. E quase NINGUÉM tem um sound designer na equipe.

---

## 0. O Som É UX. Não É "Produção Musical."

| O que o som FAZ na interface | Exemplo |
|---|---|
| **Confirma** uma ação | "Clique" ao enviar mensagem |
| **Alerta** sobre um evento | Notificação de mensagem nova |
| **Orienta** espacialmente | "Tock" do scroll no date picker |
| **Expressa a marca** | "Ta-dum" da Netflix |
| **Dá personalidade** | "Swoosh" do Mail enviado (Apple) |
| **Acalma ou excita** | Som ambiente do Headspace vs. som de vitória do Duolingo |

**O som NÃO É cosmético.** Ele CARREGA informação. Ele CRIA emoção. Ele define a PERSONALIDADE do produto. E, na maioria dos apps, é um AFTERTHOUGHT.

---

## 1. Earcons vs Auditory Icons: Os Dois Tipos de Som de Interface

### 1.1 Auditory Icons (Gaver, 1989)
Sons que imitam o mundo REAL. Têm vínculo SEMÂNTICO com o que representam.

- 🗑 Som de papel amassando = "deletar arquivo"
- 📸 Clique de obturador = "foto tirada"
- 🔒 Estalo metálico = "trancado"

**Vantagem**: intuitivo. Não precisa aprender.
**Desvantagem**: pode ser ambíguo. O som de papel amassando pode significar "lixo" OU "rascunho descartado."

### 1.2 Earcons (Blattner, Sumikawa & Greenberg, 1989)
Sons ABSTRATOS, sintéticos, musicais. A associação é APRENDIDA.

- "Ta-dum" da Netflix (duas notas descendentes)
- "Ding" do WhatsApp (tom único)
- "Whoosh" do Twitter pull-to-refresh

**Vantagem**: ÚNICO. Não confunde com som real. Pode ser BRANDING.
**Desvantagem**: precisa ser APRENDIDO. Não é intuitivo.

### 1.3 Qual é melhor?
**Garzonis et al. (CHI 2009)** — estudo longitudinal com mobile notifications:

| Medida | Vencedor |
|---|---|
| Intuitividade | **Auditory Icons** |
| Aprendizado | **Auditory Icons** |
| Memorização | **Auditory Icons** |
| Preferência do usuário | **Auditory Icons** |

**Auditory icons vencem em TUDO.** Mas earcons são melhores para BRANDING (são mais ÚNICOS). A tendência atual é o **NEOMÓRFICO** = real + abstrato. Exemplo: Apple mistura som de papel (real) com tom sintético (abstrato).

---

## 2. A Gramática do Som de Interface

### 2.1 Pitch (Altura): Para Onde o Som "Vai"

| Direção | Significado |
|---|---|
| ⬆ **Ascendente** (grave → agudo) | Positivo: "sucesso", "abrir", "conectar" |
| ⬇ **Descendente** (agudo → grave) | Negativo: "erro", "fechar", "desconectar" |
| ➡ **Plano** (mesma nota) | Neutro: "notificação", "alerta" |

### 2.2 Ritmo: O "Quando" do Som

| Ritmo | Significado |
|---|---|
| **Regular** (toc-toc-toc) | Seguro, estável, normal |
| **Sincopado** (toc... toooc) | Erro, alerta, "algo estranho" |
| **Rápido** (toc-toc-toc-toc) | Urgente, "agora!" |
| **Lento** (tooooc... tooooc) | Calmo, resolvido, "está tudo bem" |

### 2.3 Timbre: A "Cor" do Som

| Timbre | Personalidade |
|---|---|
| **Sine wave** (puro, simples) | Clean, moderno, tech |
| **Madeira** (xilofone, marimba) | Quente, orgânico, amigável |
| **Metal** (sino, triângulo) | Preciso, claro, "importante" |
| **Granular / noise** | Áspero, erro, "cuidado" |

**Brewster et al. (1994):** o timbre é o FATOR MAIS IMPORTANTE para reconhecimento de earcons. Instrumentos musicais com harmônicos RICOS são muito mais fáceis de distinguir do que beeps de senoide pura.

### 2.4 Duração

| Tipo | Duração Ideal |
|---|---|
| **Micro-interação** (clique, toggle) | 50-150ms |
| **Feedback de ação** (enviar, deletar) | 150-300ms |
| **Notificação** (mensagem, alerta) | 300-600ms |
| **Alarme** (despertador, timer) | 1-3 segundos |
| **Ringtone** | ~20 segundos |

---

## 3. Sonic Branding: O Som Que Define a Marca

### 3.1 A "Paleta de Cores Sônicas" (Made Music Studio)

Assim como um design system tem color tokens, um sound design system deveria ter "tokens sonoros":

| Token | Tipo de Som |
|---|---|
| **Feedback positivo** | Ascendente, brilhante, leve |
| **Feedback negativo** | Descendente, grave, dissonante |
| **Notificação** | Neutro, rítmico, reconhecível |
| **Navegação** | Cliques, swipes, transições |
| **Brand sting** | O "logo sonoro" (Netflix: ta-dum) |

### 3.2 Exemplos Icônicos

| Marca | Som | Por que funciona |
|---|---|---|
| **Netflix** | "Ta-dum" (2 notas) | Reconhecível em 0.5s. Minimalista. |
| **Apple Pay** | "Ding" + haptic | Confirmação de PAGAMENTO. Confiança. |
| **Slack** | "Knock brush" | Único. Amigável. Não é agressivo. |
| **WhatsApp** | "Ding" (tom de sino) | Simples. Audível. Mas IRRITANTE depois de 50x/dia. |

---

## 4. O Erro Mais Comum: Som SEM Controle

### Regra de OURO do UX Sound Design:
**NUNCA toque som sem que o usuário possa SILENCIAR.**

- O app deve respeitar o silent switch do iOS (físico, na lateral)
- O app deve oferecer toggle de som NAS configurações
- O app NUNCA deve tocar som em autoplay SEM consentimento

**Anti-padrões mortais:**
- ❌ Vídeo com som LIGADO por default ao abrir
- ❌ Som de notificação que NÃO RESPEITA o silent switch
- ❌ Som em loop que não pode ser pausado
- ❌ Volume que IGNORA o volume do sistema
- ❌ Propaganda com som (a Apple REJEITA apps que fazem isso)

---

## 5. Haptics + Som: O Casamento Perfeito

O ideal é que som e tato estejam SINCRONIZADOS:

| Tempo | O que acontece |
|---|---|
| **t=0** | Dedo toca a tela |
| **t=10-20ms** | Haptic engine vibra |
| **t=20-30ms** | Som toca |

**Por que o haptic antes do som?** O tato é processado mais RÁPIDO que a audição (~20ms vs ~50ms). Se o som vier ANTES do haptic, o usuário SENTE uma discrepância — "algo está errado" — mesmo sem saber o quê.

---

## 6. Acessibilidade: Som Não É "Plus" — É NECESSIDADE

Para usuários cegos ou com baixa visão, o som NÃO é cosmético — é CANAL PRIMÁRIO:
- **VoiceOver (iOS) / TalkBack (Android)**: o som É a interface
- **Confirmações sonoras**: "mensagem enviada", "pagamento confirmado"
- **Earcons de navegação**: "você está na tela inicial", "abriu o menu"

**Mas cuidado:** para usuários SURDOS, som NÃO funciona. Toda informação sonora precisa ter equivalente VISUAL + TÁTIL.

---

## 7. Como Criar um Sound Style Guide

### 7.1 O documento que TODO app deveria ter:

```
SOUND STYLE GUIDE — [Nome do App]

1. PALETA TIMBRÍSTICA
   - Instrumento base: [marimba / sino / sintetizador]
   - Faixa de pitch: [C3 a C5]
   - Caráter: [quente / frio / orgânico / tech]

2. CATÁLOGO DE SONS
   - send_message.wav: tom ascendente, Dó→Mi, 200ms
   - error.wav: duas notas dissonantes, Ré→Ré♭, 150ms
   - notification.wav: tom neutro, Sol, 300ms
   - brand_sting.wav: sequência de 3 notas, 1.5s

3. REGRAS DE USO
   - Som NUNCA é o único canal de feedback
   - Todo som tem versão SILENCIOSA (haptic-only)
   - Sons frequentes (>15x/hora) variam pitch ±5%
   - Volume: -23 a -16 LUFS (UI) / -18 a -16 LUFS (notificações)
   - Formatos: AAC (.m4a) iOS / OGG Android
```

### 7.2 Especificações Técnicas

| Parâmetro | Valor |
|---|---|
| **Sample rate** | 44.1kHz (mobile) |
| **Bit depth** | 16-bit |
| **Loudness (UI)** | -23 a -20 LUFS |
| **Loudness (notificação)** | -18 a -16 LUFS |
| **iOS** | AAC (.m4a) |
| **Android** | OGG Vorbis |

---

## Referências

- Gaver, W. (1989) — "The SonicFinder: An Interface That Uses Auditory Icons" — auditory icons
- Blattner, Sumikawa & Greenberg (1989) — "Earcons and Icons: Their Structure and Common Design Principles" — earcons
- Garzonis et al. (CHI 2009) — "Auditory Icon and Earcon Mobile Service Notifications" — earcons vs auditory icons
- Brewster et al. (1994) — timbre is the most important grouping factor for earcons
- Google Design — "Method: Episode 13" (Conor O'Sullivan, Pixel Sound Design Lead)
- Material Design — "About Sound" (m2.material.io)
- Huawei Universal Design Guide — "Sound" (prompt tones, interactive tones, ringtones, alarms)
- BlackBerry 10 — "Earconography Sound Design" (3-layer framework: X/Y/Z)
- SFX Engine — practical mobile sound design specs (44.1kHz, LUFS targets)
- Made Music Studio — "Sonic Color Palette" approach

---

*Fim do Loop UX/UI 14. Som: a camada de UX que Bauhaus nunca sonhou. Mas que todo app PRECISA. Cron edf71e22 segue.*
