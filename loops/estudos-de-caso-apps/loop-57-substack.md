# Estudo de Caso 57 — Substack: A Plataforma De Newsletters Com Node.js+Express, Cloudflare, Mailgun (SPF+DKIM+DMARC), Recommendation Network Que Gera 50% Das Assinaturas e Stripe Como Backbone De Pagamentos

> **Data:** 2026-07-03
> **Loop:** 57 de ∞ (Reescrita)
> **Categoria:** Publishing / Newsletters / Plataforma de Conteúdo

---

## 0. Linhagem

```
Blogs (2000s) — WordPress, Blogger. RSS. Sem monetização nativa.
Medium (2012) — publishing platform. Sem newsletters. Sem email lists.
Substack (2017) — newsletter + blog + payments. 10% fee. Recommendation network.
Substack hoje (2026) — 35M+ active subscriptions. ~2M paid. Node.js+AWS+Cloudflare.
```

---

## 1. Arquitetura Técnica

### 1.1 Tech Stack

**Cloud**: AWS (IaaS). **CDN/Proxy**: Cloudflare (SSL, DDoS, reverse proxy). **Backend**: Node.js + Express. **Banco de dados**: PostgreSQL + DynamoDB. **Storage**: AWS S3. **Email**: Mailgun (primário) + provider secundário. **Monitoramento**: Datadog. **Payments**: Stripe. **Analytics**: Cloudflare Insights.

**Email authentication configurada por escritor**: cada newsletter tem subdomínio próprio com SPF, DKIM e DMARC configurados. O SPF aponta para os servidores Mailgun/Substack. DKIM assina criptograficamente cada email. DMARC define política de enforcement (`p=none` → `p=quarantine` → `p=reject`).

### 1.2 O Recommendation Network

A feature mais arquiteturalmente distintiva do Substack é o **recommendation network** — um grafo de escritores e publicações que gera **~50% de todas as assinaturas** e **~30% das assinaturas pagas**. Cross-promotions, publisher recommendations, leaderboards e feeds por categoria formam um mecanismo de descoberta que nenhuma plataforma self-hosted (Ghost, WordPress) consegue replicar.

**Notes**: short-form social media integrado à plataforma, funcionando como camada de engajamento e descoberta.

### 1.3 Stripe Como Backbone De Pagamentos

Cada escritor conecta sua própria conta Stripe ao Substack. Leitores que assinam tornam-se customers na Stripe do escritor — o escritor retém ownership do billing relationship. O Substack cobra **10%** sobre assinaturas pagas; Stripe cobra ~2,9% + US$ 0,30 por transação.

**Apple IAP lock-in (2025)**: Substack passou a exigir Apple In-App Purchase para assinaturas iOS. O IAP impõe **30% Apple fee** e transfere o billing relationship para a Apple — transações IAP não aparecem na Stripe do escritor, impedindo portabilidade de assinantes pagos. É uma estratégia de platform lock-in: controle do app → dependency via dark patterns → sever do billing relationship.

### 1.4 Email Deliverability: SPF, DKIM, DMARC e IP Warming

**2024 watershed**: Google e Yahoo passaram a exigir SPF+DKIM+DMARC para qualquer sender com >5.000 mensagens/dia. Envio não-compliant começa a ser bounced parcialmente, com rejection rates aumentando gradualmente.

**Pipeline de deliverability do Substack por escritor**:
- Custom sending domain com SPF/DKIM/DMARC
- **IP warming**: novo escritor começa com volume gradual (semanas) para construir reputação de envio
- **Bounce monitoring**: remoção automática de endereços inválidos
- **Spam complaint rate** monitorado: <0,10% exigido. Google Postmaster Tools + Yahoo Sender Hub
- **RFC 8058 one-click unsubscribe**: obrigatório desde 2024
- **List hygiene**: double opt-in, sunset policies para inativos

---

## 2. Lições de Engenharia

### 2.1 O recommendation network é um fosso que nenhuma alternativa self-hosted consegue replicar

Ghost e WordPress oferecem 0% platform fee, mas não têm discovery network. O Substack gera 50% das assinaturas via recomendações — é o equivalente a um algoritmo de feed social para newsletters.

### 2.2 SPF+DKIM+DMARC deixaram de ser "boas práticas" para serem requisitos de entrega

Desde fevereiro 2024, sem os três configurados, emails >5.000/dia não chegam à caixa de entrada do Gmail ou Yahoo. É a maior mudança regulatória em email desde o CAN-SPAM Act.

### 2.3 A taxa de 10% é um seguro contra complexidade de infraestrutura

Manter infra própria (SPF, DKIM, IP warming, bounce monitoring, spam compliance, CDN, CMS, payments) custa mais que 10% da receita para a maioria dos escritores. O break-even só ocorre acima de ~US$ 10K/mês em subscription revenue.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Substack |
| **Fundação** | 2017. Fundadores: Chris Best (CEO), Hamish McKenzie, Jairaj Sethi |
| **Categoria** | Publishing / Newsletters / Plataforma |
| **Assinaturas** | 35M+ ativas. ~2M pagas |
| **Fee** | 10% + Stripe (~2,9% + US$ 0,30). Apple IAP: 30% adicional |
| **Stack** | Node.js+Express, PostgreSQL+DynamoDB, AWS S3, Cloudflare, Mailgun, Stripe, Datadog |
| **Email** | SPF+DKIM+DMARC por domínio. IP warming. <0,10% spam rate. RFC 8058 |
| **Concorrentes** | Ghost, beehiiv, WordPress+Memberships, Patreon |

---

## Fontes

- [WebTechSurvey — substack.com Technology Stack (2025)](https://webtechsurvey.com/website/substack.com)
- [Semafor — Substack network drives 50% of subscriptions, 30% of paid (2024)](https://www.semafor.com/)
- [DataConomy — Substack forces IAP, hits writers with 30% fee (Set 2025)](https://dataconomy.com/2025/09/01/substack-forces-iap-hits-writers-with-30-fee/)
- [Warmy — Email Deliverability Trends 2025 Guide (SPF/DKIM/DMARC enforcement by Google/Yahoo Feb 2024)](https://blog.warmy.io/)
- [RaftLabs — How to Build a Newsletter Platform Like Substack (architecture guide)](https://www.raftlabs.com/blog/how-to-build-app-like-substack)
