# UX/UI Loop 35 — Micro-Interactions & Animation: Os 200ms Que Fazem a Interface Parecer Viva

> **Data:** 2026-07-02
> **Loop:** 35 de ∞
> **Tema:** Uma interface sem animação é uma interface MORTA. Você toca um botão. Nada acontece (visualmente). O botão não "afunda." A tela não "desliza." A lista não "quica." O resultado: a interface parece QUEBRADA — mesmo que esteja funcionando. Micro-interações são o TECIDO CONJUNTIVO da UX. E seguem os MESMOS princípios que Walt Disney usava para dar vida a personagens em 1930.

---

## 0. Por Que Animação É UX (Não "Decoração")

**Animação NÃO É cosmética.** Ela TEM funções de UX:

| Função | Exemplo |
|---|---|
| **Feedback** | Botão "afunda" ao toque. Você SABE que o toque foi registrado |
| **Orientação espacial** | Tela desliza da direita. Você SABE que "voltou" |
| **Hierarquia** | Modal sobe de baixo. Você SABE que é uma camada TEMPORÁRIA |
| **Continuidade** | Card se expande para tela cheia. Você SABE que é o MESMO conteúdo |
| **Personalidade** | O "bounce" do iOS. O "ripple" do Material. Cada um tem CARÁTER |
| **Redução de percepção de espera** | Skeleton screen com shimmer. A espera PARECE mais curta (L23) |

---

## 1. Os 12 Princípios de Disney Aplicados a UI

Em 1981, Ollie Johnston e Frank Thomas (animadores da Disney) publicaram *The Illusion of Life* com 12 princípios de animação. Em 2026, eles são o MANUAL SECRETO de toda grande UI.

### 1. Squash & Stretch (Achatamento e Estiramento)
Dar PESO e FLEXIBILIDADE a objetos.

**UI:** Um botão que "afunda" ao toque (squash). Uma lista que "estica" no pull-to-refresh (stretch). O scroll com bounce no iOS.

### 2. Anticipation (Antecipação)
Preparar o espectador para a ação que VAI acontecer.

**UI:** Hover state em botões. O ícone que muda de cor ANTES de você clicar. O card que "levanta" (elevação) ANTES de se expandir.

### 3. Staging (Encenação)
Dirigir a ATENÇÃO para o que é importante.

**UI:** O modal que escurece o fundo (o foco vai para o modal). O botão CTA que PULSA. O item novo que "brilha."

### 4. Straight Ahead & Pose to Pose
Dois métodos de animação. Frame a frame ou poses-chave.

**UI:** CSS transitions = pose to pose. JavaScript requestAnimationFrame = straight ahead.

### 5. Follow Through & Overlapping Action
Partes do objeto continuam se movendo DEPOIS que o objeto parou. E partes diferentes se movem em TEMPOS diferentes.

**UI:** O "bounce" no fim do scroll (follow through). Elementos da lista que aparecem em CASCATA (overlapping). O efeito "parallax" onde camadas se movem em velocidades diferentes.

### 6. Slow In & Slow Out (Easing)
Movimentos ACELERAM e DESACELERAM. Nada se move em velocidade constante.

**UI:** Curvas de easing. `ease-out` (desacelera) para elementos ENTRANDO. `ease-in` (acelera) para elementos SAINDO. `cubic-bezier()` é o pincel do animador de UI.

### 7. Arc (Arco)
Movimentos naturais seguem ARCOS, não linhas retas.

**UI:** O FAB que se expande em LEQUE. O menu que "explode" em arco. Não em linha reta.

### 8. Secondary Action (Ação Secundária)
Ações menores que REFORÇAM a ação principal.

**UI:** O ícone de "enviar" que vira um AVIÃOZINHO e "voa" para fora da tela. O like que "explode" em corações menores.

### 9. Timing
A VELOCIDADE da ação define seu CARÁTER.

**UI:**
- 100ms: instantâneo (o usuário NÃO percebe)
- 200-300ms: rápido (feedback de clique)
- 300-500ms: transição (navegação entre telas)
- 500-1000ms: dramático (onboarding, celebração)

### 10. Exaggeration (Exagero)
Aumentar a REALIDADE para torná-la mais LEGÍVEL.

**UI:** O "bounce" do iOS que quica MAIS do que a física real. O badge de notificação que PULSA. O ícone que "treme" no erro de senha.

### 11. Solid Drawing (Desenho Sólido)
Entender o OBJETO em 3D — mesmo quando desenhado em 2D.

**UI:** Elevação do Material Design (sombra = profundidade). O "vidro fosco" do iOS (translucidez). A interface ENTENDE que é um espaço 3D, mesmo em tela plana.

### 12. Appeal (Apelo)
O design tem CARISMA. O usuário QUER olhar.

**UI:** O dinossauro do Chrome (jogo offline). O coração do Twitter que "explode." A animação de "pago!" do Nubank.

---

## 2. Os 4 Tipos de Micro-Interação (Dan Saffer)

Dan Saffer (*Microinteractions*, 2013) categorizou toda micro-interação em 4 partes:

| Parte | Pergunta | Exemplo |
|---|---|---|
| **Trigger** | O que INICIA? | Toque, swipe, notificação, sistema |
| **Rules** | O que ACONTECE? | "Se swipe para direita, arquivar" |
| **Feedback** | O que o usuário VÊ/SENTE/OUVE? | Animação, haptic, som |
| **Loops & Modes** | O que muda com REPETIÇÃO? | "Você já arquivou 100 emails!" |

---

## 3. O Dicionário de Durações

| Duração | Uso | Curva |
|---|---|---|
| **100ms** | Toggle, checkbox, hover | `ease-out` |
| **200ms** | Clique em botão, ripple | `ease-out` |
| **250ms** | Transição entre telas (iOS) | `ease-in-out` |
| **300ms** | Modal aparecendo, card expandindo | `ease-out` + spring |
| **400ms** | Notificação entrando, toast | `ease-out` |
| **500ms** | Splash screen, onboarding | `ease-in-out` |
| **800ms-1s** | Celebração, conquista | Spring generoso |

---

## 4. O "Easing" Como Linguagem

**`ease-out`** (desacelera): objetos ENTRANDO na tela. Como um carro freando SUAVEMENTE.

**`ease-in`** (acelera): objetos SAINDO da tela. Como um carro ARRANCANDO.

**`ease-in-out`** (acelera + desacelera): objetos se movendo DENTRO da tela. O padrão mais NATURAL.

**Spring (mola):** objetos que "quicam" além do ponto final e voltam. Dá sensação de PESO e FÍSICA. iOS ama spring. Material Design prefere ease-out mais contido.

---

## 5. As Regras de OURO da Animação de UI

1. **Toda animação tem um PORQUÊ.** Se a animação não comunicar feedback, orientação, hierarquia ou personalidade — REMOVA.

2. **Respeite `prefers-reduced-motion`.** ~15% dos usuários ativam "Reduzir Movimento" (iOS) ou "Remover animações" (Android). Enxaqueca, vertigem, autismo. Sua animação NÃO pode causar dor.

3. **Duração < 400ms para interações.** Se o usuário precisa ESPERAR a animação terminar, ela é longa demais.

4. **Performance é TUDO.** 60fps. Sem jank. Sem dropped frames. Uma animação a 30fps PARECE mais lenta que uma estática.

5. **Animação NUNCA é o único canal.** Sempre combine com cor, texto, ícone. O usuário pode ter `prefers-reduced-motion` ativado.

---

## Referências

- Ollie Johnston & Frank Thomas — *The Illusion of Life: Disney Animation* (1981). 12 princípios.
- Dan Saffer — *Microinteractions* (O'Reilly, 2013)
- Apple HIG — Animation. Spring animations. `prefers-reduced-motion`.
- Material Design — Motion. Easing curves. Duration tokens.
- Disney → UI lineage: Johnston/Thomas → UI animators → toda interface moderna

---

*Fim do Loop UX/UI 35. Micro-interações: os 200ms que fazem a interface parecer viva. Cron d942e0d4 segue.*
