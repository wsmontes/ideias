# UX/UI Loop 25 — Dark Patterns & Design Ético: O Que NÃO Fazer (E Por Que)

> **Data:** 2026-07-02
> **Loop:** 25 de ∞
> **Tema:** 24 loops sobre COMO fazer design. Este é sobre COMO NÃO fazer. Harry Brignull cunhou "dark patterns" em 2010 — padrões de interface PROJETADOS para enganar. Eles estão em TODO lugar: "aceitar cookies", "só mais 3 vagas!", cancelar assinatura requer 7 telas. Designers éticos precisam SABER reconhecer dark patterns — e RECUSAR implementá-los.

---

## 0. O Que São Dark Patterns

> Harry Brignull (2010): "A dark pattern is a user interface that has been carefully crafted to trick users into doing things they did not mean to do."

**A diferença entre "UX ruim" e "dark pattern":**
- UX ruim: o designer TENTOU fazer algo bom, mas FALHOU (incompetência)
- Dark pattern: o designer FEZ algo ruim DE PROPÓSITO para beneficiar a empresa (má-fé)

**O resultado é o mesmo:** o usuário sofre. Mas a ÉTICA é completamente diferente.

---

## 1. A Taxonomia dos Dark Patterns (Brignull + Expansões)

### 1.1 Confirmshaming
Fazer o usuário se sentir CULPADO por não aceitar algo.

> "Assine nossa newsletter!" → [Sim, quero ser mais inteligente!] [Não, prefiro continuar ignorante]

**Por que é dark:** a opção "não" ENVERGONHA o usuário. Você não está "escolhendo" — está sendo HUMILHADO por escolher.

### 1.2 Roach Motel
ENTRAR é fácil. SAIR é quase impossível.

**Exemplo clássico:** cancelar assinatura da Amazon Prime. Requer navegar por 6 telas, confirmar 3 vezes, e o botão "cancelar" é cinza-claro sobre fundo branco.

**Exemplo mobile:** cancelar conta no app. O botão "deletar conta" está em Settings → Account → Advanced → (scroll) → Delete. E ainda pede para "entrar em contato com o suporte."

### 1.3 Privacy Zuckering
Compartilhar MAIS dados do que o usuário pretendia.

**Exemplo:** Facebook (Mark Zuckerberg, daí o nome). Configurações de privacidade que MUDAM sozinhas. "Público" como default. Checkboxes pré-marcados.

**Exemplo mobile:** "Compartilhar contatos?" com "SIM" em destaque e "Depois" em cinza. O usuário CLICA no botão colorido sem ler.

### 1.4 Bait and Switch
O usuário CLICA em uma coisa. Acontece OUTRA.

**Exemplo:** popup de "Fechar" (X) que na verdade ABRE o anúncio. O X é FALSO.

**Exemplo mobile:** botão "Pular" no onboarding que na verdade é "Aceitar e continuar."

### 1.5 Disguised Ads
Anúncios disfarçados de CONTEÚDO ou NAVEGAÇÃO.

**Exemplo mobile:** "Download" (botão falso — é um anúncio). O verdadeiro botão de download é minúsculo e está em outro lugar.

### 1.6 Forced Continuity
Assinatura que NÃO TERMINA quando você acha que terminou.

**Exemplo:** "Free trial por 7 dias." Você cancela no dia 6. Mas a cobrança JÁ FOI FEITA no dia 5. (A "semana" deles tem 5 dias.)

### 1.7 Trick Questions
Perguntas formuladas para CONFUNDIR.

**Exemplo:** "Marque esta caixa se você NÃO QUER receber nossos e-mails." (Dupla negação.)

### 1.8 Hidden Costs
Preços revelados APENAS no ÚLTIMO passo.

**Exemplo mobile:** "Pedir Uber — R$25." No fim: +R$5 taxa, +R$3 pedágio, +R$2 "outros." Total: R$35.

### 1.9 Sneak Into Basket
Itens adicionados ao carrinho SEM consentimento.

**Exemplo:** "Comprar passagem." No fim: seguro de viagem JÁ MARCADO. Você precisa DESMARCAR para não pagar.

---

## 2. Dark Patterns Específicos de Mobile

### 2.1 Notificações Como Recapture
O app te manda notificação DEPOIS que você parou de usar. "Fulano postou algo!" (Você abre. Não tem nada.)

### 2.2 "Pseudo-Urgência"
"🔥 Só mais 2 vagas!" (para sempre). "⏰ Oferta termina em 00:59!" (reseta quando zera).

### 2.3 Permissões Abusivas
App de lanterna pedindo acesso a CONTATOS. App de calculadora pedindo LOCALIZAÇÃO. O pedido aparece quando você está DISTRAÍDO.

### 2.4 Dark Pattern de Assinatura
"R$9,90 por SEMANA" (em letras minúsculas: cobrança ANUAL de R$514,80).

### 2.5 Gamificação Viciante
Streaks (ofensivas), placares, "você é o 3º melhor!" — mecânicas de jogo para MAXIMIZAR engajamento, não para DIVERTIR.

---

## 3. Quem Regula (E Como)

| Legislação | Onde | O que cobre |
|---|---|---|
| **GDPR** (2018) | Europa | Consentimento REAL. Checkbox NÃO pode ser pré-marcado |
| **DSA** (2024) | Europa | Dark patterns PROIBIDOS em plataformas digitais |
| **EAA** (2025) | Europa | Acessibilidade obrigatória. Multas até €1M |
| **ADA** | EUA | Acessibilidade. 4.000+ processos em 2024 |
| **LGPD** (2020) | Brasil | Consentimento. Inspirado no GDPR |
| **ANPD** | Brasil | Autoridade Nacional de Proteção de Dados |

---

## 4. O Código de Ética do Designer

### 4.1 O Juramento de Ética do UX Designer

> "Eu NÃO usarei meu conhecimento de psicologia, design e tecnologia para ENGANAR usuários. Não criarei interfaces que CONFUNDEM propositalmente. Não esconderei opções que prejudicam o usuário. Não farei o 'cancelar' mais difícil que o 'assinar.' Lembrarei que do outro lado da tela há um SER HUMANO — com pressa, cansaço, distração."

### 4.2 Princípios do Design Ético

| Princípio | O que significa |
|---|---|
| **Transparência** | O usuário SABE o que está acontecendo |
| **Consentimento REAL** | "Sim" significa SIM. Checkbox NUNCA pré-marcado |
| **Simetria** | Cancelar é TÃO FÁCIL quanto assinar |
| **Acessibilidade** | O app funciona para TODOS os corpos |
| **Privacidade por default** | Compartilhar é OPÇÃO. Não compartilhar é DEFAULT |
| **Tempo bem gasto** | O app MELHORA a vida do usuário, não a DRENA |

### 4.3 O "Red Flag Checklist" (Antes de Lançar)

- [ ] Cancelar é tão fácil quanto assinar?
- [ ] O botão "não" tem o MESMO peso visual que o "sim"?
- [ ] Checkboxes NÃO são pré-marcados?
- [ ] Preços são visíveis ANTES do checkout?
- [ ] Notificações podem ser desligadas com 1 TOQUE?
- [ ] O app funciona sem permissões ABUSIVAS (lanterna não pede contatos)?
- [ ] Contagem regressiva é REAL (não reseta)?
- [ ] "Grátis" é GRÁTIS (sem "free trial que vira cobrança automática")?

---

## 5. O Que Fazer Quando Te Pedem Um Dark Pattern

**1. Mostre os DADOS.** "Isso vai gerar reclamações. Vai aumentar churn. Vai gerar processo (LGPD, DSA)."

**2. Mostre a ALTERNATIVA ÉTICA.** "Em vez de 'confirmshaming', podemos usar copy positivo."

**3. Mostre os RISCOS LEGAIS.** GDPR: multa de 4% do faturamento GLOBAL. DSA: dark patterns PROIBIDOS.

**4. Recuse.** Designers têm RESPONSABILIDADE ÉTICA. "Não vou implementar isso." É uma frase DIFÍCIL. Mas NECESSÁRIA.

---

## 6. O Design Ético É BOM PARA O NEGÓCIO

**Dados (Level Access, Forrester, McKinsey):**
- Empresas com design inclusivo têm 2.3x mais receita
- Apps com dark patterns têm MAIOR churn (usuários se SENTEM enganados)
- Multas de GDPR já ultrapassaram €4 bilhões
- 88% dos usuários NÃO VOLTAM após uma experiência ruim

**Design ético NÃO É "caridade." É ESTRATÉGIA DE LONGO PRAZO.**

---

## Referências

- Harry Brignull — darkpatterns.org (2010). Taxonomia original.
- GDPR (2018) — consentimento real, multas de 4% do faturamento global
- DSA / Digital Services Act (2024) — dark patterns proibidos na Europa
- EAA / European Accessibility Act (jun 2025) — multas até €1M
- LGPD (2020) — Brasil
- Forrester / McKinsey — ROI de design ético e inclusivo

---

*Fim do Loop UX/UI 25. Dark patterns: conhecer para RECUSAR. O designer é o guardião ético da interface. Cron edf71e22 segue.*
