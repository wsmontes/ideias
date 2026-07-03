# UX/UI Loop 27 — Onboarding: Os 30 Segundos Que Definem Seu App

> **Data:** 2026-07-02
> **Loop:** 27 de ∞
> **Tema:** ~25% dos apps são ABANDONADOS após o PRIMEIRO uso. A maioria das pessoas decide se vai CONTINUAR usando um app nos primeiros 30 SEGUNDOS. E a maioria dos apps FRACASSA no onboarding: tutoriais de 7 telas que ninguém lê, permissões pedidas antes do valor ser entregue, formulários de cadastro antes de mostrar o produto. Onboarding é a ARTE de ensinar sem ENSINAR.

---

## 0. O Funil da Morte

**Dados (Quettra, Adjust, Appsflyer):**
- ~25% dos apps são abandonados após 1 uso
- ~60% são desinstalados em 7 dias
- ~80% dos usuários NUNCA abrem o app uma segunda vez
- O onboarding É o momento de MAIOR CHURN

**A causa #1:** o app não entrega VALOR nos primeiros 30 segundos. O usuário não sabe O QUE o app faz, COMO usar, ou POR QUE deveria ficar. E vai embora.

---

## 1. As 3 Leis do Onboarding

### Lei 1: Mostre VALOR Antes de Pedir CADASTRO
O usuário acabou de ABRIR seu app. Ele NÃO confia em você. Você pede: "Crie sua conta. Nome, e-mail, senha, confirme senha, aceite termos."

**O usuário PENSANDO:** "Eu nem sei se este app PRESTA. Por que eu daria meus dados?"

**O CERTO:** mostre o PRODUTO primeiro. Deixe o usuário EXPERIMENTAR. Peça cadastro DEPOIS — quando ele JÁ VIU o valor.

**Exemplo:** Airbnb. Você busca destinos ANTES de criar conta. O valor é ENTREGUE antes do pedido de dados.

### Lei 2: Ensine FAZENDO, Não EXPLICANDO
O tutorial de 7 telas com ilustrações e bullets points — NINGUÉM LÊ. As pessoas deslizam até o fim e clicam "Começar" sem ter absorvido NADA.

**O CERTO:** ensine no CONTEXTO. O tooltip aparece QUANDO o usuário PRECISA daquela informação. "Para enviar, deslize para a direita" — mas só mostre isso quando ele estiver PARADO na tela de envio.

### Lei 3: Progresso > Perfeição
O primeiro uso NÃO precisa ser COMPLETO. Precisa ser RÁPIDO e SATISFATÓRIO. O usuário conseguiu FAZER ALGO? Conseguiu VER valor? Já está BOM. O resto ele aprende DEPOIS.

---

## 2. Os 4 Tipos de Onboarding

| Tipo | Exemplo | Ideal para |
|---|---|---|
| **Progressivo** (ensina no contexto) | Tooltips, coach marks, dicas sutis | Apps complexos com muitas features |
| **Demonstração** (mostra o valor) | Vídeo curto, animação, "veja como funciona" | Apps inovadores (conceito novo) |
| **Hands-on** (aprender fazendo) | "Deslize para começar" — e o usuário DESLIZA | Apps simples, gestuais |
| **Empty State** (tela vazia que ENSINA) | "Você ainda não tem mensagens. Que tal enviar uma?" + botão | Apps de conteúdo (mensagens, fotos, notas) |

---

## 3. O Empty State Como Superpoder

O "empty state" (tela VAZIA) é o momento MAIS SUBESTIMADO do onboarding. Quando não há DADOS ainda, você tem uma OPORTUNIDADE de ensinar:

**❌ Empty state RUIM:**
- "Sem mensagens." (tela branca. Só texto. Nenhuma ação.)

**✅ Empty state BOM:**
- Ilustração. Mensagem AMIGÁVEL: "Nada aqui ainda! Que tal enviar sua primeira mensagem?" + BOTÃO "Enviar mensagem" em destaque.

**Exemplos memoráveis:**
- **Dropbox** (primeiro uso): "Sua pasta está vazia. Arraste arquivos aqui." — a interface É o tutorial
- **Discord**: "Nenhum servidor. Crie um ou entre em um!" com ilustração e CTAs claros

---

## 4. Permissões: O Assassino do Onboarding

**O ERRO CLÁSSICO:** o usuário abre o app pela PRIMEIRA VEZ. Imediatamente:
- "O app gostaria de enviar notificações" [Permitir / Não permitir]
- "O app gostaria de acessar seus contatos" [OK / Não]
- "O app gostaria de acessar sua localização" [Permitir / Não permitir]

**3 popups nativos do sistema. Em 5 segundos.** O usuário clica "NÃO" em TODOS. E nunca mais volta.

**O CERTO:** peça permissão no MOMENTO em que o usuário ENTENDE por que precisa dela. "Para encontrar restaurantes perto de você, precisamos da sua localização." (O usuário JÁ ESTÁ na tela de busca de restaurantes. Ele ENTENDEU.) Agora SIM.

**Regra de ouro:** NUNCA peça permissão no primeiro uso. Espere o MOMENTO CERTO. Se o usuário diz "não", respeite.

---

## 5. A Regra dos 3 Toques

O usuário deve conseguir FAZER ALGO SIGNIFICATIVO em ATÉ 3 TOQUES após abrir o app.

| Toque | O que acontece |
|---|---|
| **1** | Abre o app → vê a tela principal + empty state ou valor imediato |
| **2** | Toca na ação principal (CTA) |
| **3** | Resultado: algo ACONTECEU. Valor foi ENTREGUE |

Se você precisa de MAIS de 3 toques para entregar valor, seu onboarding precisa ser REVISTO.

---

## 6. Sign In With Apple / Google: O Melhor Padrão de 2020+

O "Sign In With Apple" (2019) e "Sign In With Google" reduziram o onboarding a UM TOQUE.

**Antes:** Nome. E-mail. Senha. Confirmar senha. Aceitar termos. Verificar e-mail. (5-7 minutos. Altíssimo drop-off.)

**Depois:** "Entrar com Google" → UM toque. Pronto.

**Para o desenvolvedor:** o usuário já está DENTRO. Os dados vieram do Google/Apple. Zero formulários. Zero fricção.

---

## 7. O Que Medir No Onboarding

| Métrica | O que significa |
|---|---|
| **Time-to-value** (TTV) | Quanto tempo até o usuário EXPERIMENTAR o valor central? |
| **Activation rate** | % de usuários que completam a "ação mágica" (ex: enviam 1ª mensagem) |
| **Drop-off por tela** | Em qual tela do onboarding o usuário DESISTE? |
| **Permission opt-in rate** | % que ACEITAM notificações / localização |
| **Day 1 retention** | % que voltam no dia seguinte |
| **Day 7 retention** | % que voltam após 7 dias |

---

## 8. Exemplos Memoráveis

| App | Onboarding |
|---|---|
| **Duolingo** | Você começa a APRENDER imediatamente. Sem cadastro. Sem explicação. "Escolha um idioma" → primeira lição. |
| **TikTok** | Abre o app → vídeo rolando. Onboarding ZERO. Valor IMEDIATO. |
| **Calm** | "Respire fundo" — som de natureza, animação suave. Você EXPERIMENTA antes de decidir. |
| **Airbnb** | Busca destinos sem conta. Só pede login quando você vai RESERVAR. |
| **Robinhood** | Você vê o app FUNCIONANDO (com dados falsos) antes de criar conta. |

---

## Referências

- Quettra — 25% dos apps abandonados após 1º uso
- Adjust / Appsflyer — ~60% desinstalados em 7 dias
- Don Norman — "discoverability" como princípio (L26)
- Apple HIG — Sign In With Apple (2019), permissões contextuais
- Material Design — onboarding patterns, empty states

---

*Fim do Loop UX/UI 27. Onboarding: o funil que decide se seu app VIVE ou MORRE. Cron edf71e22 segue.*
