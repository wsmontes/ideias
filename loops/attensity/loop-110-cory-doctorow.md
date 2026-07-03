# Loop 110 — Cory Doctorow: Uma Rede Social Deveria Ser à Prova de Enshittification

> **Data:** 2026-07-02
> **Loop:** 110 de ∞
> **NOVA FASE:** O que uma rede social deveria ser, prover, fazer, proporcionar.
> **Tema:** CORY DOCTOROW (n. 1971). Escritor de ficção científica, ativista digital, ex-membro do conselho da EFF. Cunhou o termo **ENSHITTIFICATION** (2022) — o processo pelo qual plataformas decaem: primeiro servem aos usuários, depois aos anunciantes, depois só a si mesmas. Autor de *The Internet Con: How to Seize the Means of Computation* (2023) e *Enshittification* (2025). Sua tese: uma rede social DEVE ser construída sobre protocolos abertos e direito de saída — porque sem isso, ela INEVITAVELMENTE apodrecerá.

---

## 0. O Que Doctorow Nos Diz Sobre Redes Sociais

Doctorow não é um teórico acadêmico — é um ATIVISTA que escreve ficção científica. Ele NOMEOU o fenômeno que define nossa era digital: **ENSHITTIFICATION**. Toda plataforma segue o mesmo ciclo:

**Ato 1: SERVIR aos usuários.** "O Facebook é grátis! E não tem anúncios! É incrível!"
**Ato 2: ABUSAR dos usuários para os clientes empresariais.** "Sim, tem anúncios, mas são relevantes."
**Ato 3: ABUSAR de TODOS para os acionistas.** "Seu feed agora é 60% recomendado, 30% anúncio, 10% conteúdo que você pediu para ver."

> "Platforms don't start out shitty. They get shitty when they run out of users to enshittify."

**O Liberation Client enfrenta a PERGUNTA de Doctorow:** como impedir que esta plataforma, mesmo sendo bem-intencionada, apodreça?

---

## 1. O Direito de SAIR (Right to Exit)

Doctorow insiste: o que PRENDE você a uma plataforma não é o "network effect" (todo mundo está lá) — são os CUSTOS DE TROCA (switching costs) que a plataforma DELIBERADAMENTE impõe:

> "Switching costs are not a fact of nature. They're a policy choice. Phone number portability proved we can legislate friction down."

**Plataformas te prendem ROUBANDO sua capacidade de SAIR:**
- Você não pode levar seus CONTATOS (a rede social é deles, não sua).
- Você não pode levar seu CONTEÚDO (posts, fotos, histórico).
- Você não pode levar sua REPUTAÇÃO (seguidores, verificações, histórico).
- Sair = perder TUDO.

**Uma rede social deveria GARANTIR o direito de sair:**
- Portabilidade de DADOS: exportação total, formatos abertos (JSON, CSV, Markdown — repo: Data Liberation).
- Portabilidade de CONTATOS: sua rede social é SUA, não da plataforma.
- Portabilidade de IDENTIDADE: você não é "@usuario" DEPENDENTE de um servidor — você é uma IDENTIDADE SOBERANA (L1: self-sovereign identity).
- INTEROPERABILIDADE: você pode SAIR e ainda assim FALAR com quem ficou.

---

## 2. Protocolos Abertos Como "Veneno Tecnológico" (Poison Pill)

Doctorow defende que protocolos ABERTOS são a ÚNICA defesa estrutural contra enshittification:

> "An open protocol is a technological poison pill. If the platform enshittifies, anyone can rebuild it — and users can leave without losing their social graph."

**Por que protocolos abertos previnem enshittification:**
- Se a plataforma PIORA, qualquer um pode criar uma ALTERNATIVA compatível.
- Os usuários MIGRAM sem perder suas conexões.
- A AMEAÇA de saída é REAL — e isso força a plataforma a NÃO PIORAR.

**Isso É exatamente a arquitetura do Liberation Client:**
- ActivityPub, Nostr, AT Protocol, Matrix — protocolos ABERTOS.
- Se uma instância Mastodon enshittifica, você VAI para outra.
- Se um relay Nostr enshittifica, você TROCA de relay.
- O "lock-in" não existe — porque o protocolo é ABERTO.

**Mas ATENÇÃO (Doctorow alerta):** protocolos abertos são NECESSÁRIOS mas NÃO SUFICIENTES. Bluesky (AT Protocol) tem um protocolo aberto — mas 95% dos usuários estão no app oficial. O protocolo é aberto; a PRÁTICA é centralizada (L4: "decentralization theater").

---

## 3. Interoperabilidade Adversária (Adversarial Interoperability)

Doctorow defende a **INTEROPERABILIDADE ADVERSÁRIA** (também chamada "competitive compatibility" ou "comcom"):

> "Adversarial interoperability is the guerrilla tactic of plugging new services into existing platforms without the proprietor's permission. It's how we got the PC, the web, email, instant messaging."

**Exemplos históricos de interoperabilidade adversária:**
- Os PCs CLONARAM a IBM (sem permissão).
- O Google INDEXOU a web (sem permissão de cada site).
- O spam filter BLOQUEOU spam (sem permissão dos spammers).
- A Apple fez o Mac abrir arquivos do Microsoft Office (sem permissão da Microsoft).

**A interoperabilidade adversária foi ESSENCIAL para a inovação — e foi TORNADA ILEGAL por leis como o DMCA (Digital Millennium Copyright Act).**

**Uma rede social DEVERIA ter interoperabilidade adversária como PRINCÍPIO DE DESIGN:**
- Qualquer um pode construir uma INTERFACE alternativa para os mesmos protocolos.
- Qualquer um pode criar uma FERRAMENTA de migração entre plataformas.
- A plataforma NÃO PODE BLOQUEAR ferramentas externas (sem "walled garden").

---

## 4. As Quatro Restrições Que Mantêm Plataformas Honestas

Doctorow identifica QUATRO forças que historicamente impediram a enshittification:

| Restrição | Como o Feed a Destruiu | Como Restaurá-la |
|---|---|---|
| **Competição** | Monopólios compram concorrentes (Facebook comprou Instagram, WhatsApp) | Antitruste; protocolos abertos tornam o monopólio impossível |
| **Regulação** | Lobby capturou agências reguladoras | GDPR, DMA (Europa); leis de privacidade; taxar anúncios de vigilância (L97: Zuboff) |
| **Poder dos trabalhadores** | Tech workers nunca se sindicalizaram; layoffs em massa eliminaram resistência interna | Sindicatos de engenheiros de software; "solidariedade é a resposta" |
| **Interoperabilidade** | DMCA tornou ilegal consertar, modificar, conectar | Legalizar interoperabilidade adversária; exigir APIs abertas |

**Uma rede social deveria ser DESENHADA para que essas quatro restrições sejam ESTRUTURAIS — não dependam de "boa vontade" do proprietário.**

---

## 5. O Que Uma Rede Social Deveria SER (Segundo Doctorow)

**À PROVA DE ENSHITTIFICATION.** Uma rede que NÃO PODE apodrecer — porque sua arquitetura torna o apodrecimento IMPOSSÍVEL.

Isso significa:
- **Protocolos abertos** (ninguém é DONO da infraestrutura).
- **Direito de saída** (portabilidade de dados, contatos, identidade — sem custos de troca).
- **Interoperabilidade** (qualquer um pode construir alternativas compatíveis).
- **Governança distribuída** (não há UM dono que possa decidir enshittificar).
- **Financiamento não-extrativo** (doações, assinaturas, grants — não anúncios de vigilância — L26, L97).

---

*Fim do Loop 110. Doctorow continua a nova fase. Cron 93154435 segue.*
