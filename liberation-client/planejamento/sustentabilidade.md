# Liberation Client — Sustentabilidade Financeira

> Como financiar um projeto de libertação sem se vender.
> **Lacuna 1** — 2026-07-01

---

## 1. O Problema

Projetos open-source de infraestrutura social morrem por três razões: **burnout do mantenedor**, **falta de dinheiro**, ou **captura pelo capital**. Este documento existe para prevenir as três.

O Liberation Client rejeita explicitamente:
- Capital de risco (VC)
- Publicidade
- Coleta e venda de dados
- Modelos de engajamento que otimizam tempo de tela

A pergunta é: **o que sobra?**

---

## 2. Modelos Compatíveis com os Princípios

### 2.1 Doações (Tier 1 — Fundação)

O modelo mais alinhado: usuários e apoiadores doam voluntariamente.

**Canais**:
| Canal | Taxa | Público |
|-------|------|---------|
| **Open Collective** | 10% | Internacional, transparente, fiscal host |
| **GitHub Sponsors** | 0% (Microsoft cobre) | Desenvolvedores |
| **Pix recorrente** | 0% | Brasil — essencial para a base local |
| **Liberapay** | 0% | Europa, alinhado com valores open-source |
| **BTCPay Server** | 0% | Bitcoin/Lightning, autocustódia |
| **Stripe** | 2.9% + $0.30 | Cartão de crédito internacional |

**Meta de doações**:
- **Sustentável (1 dev)**: $3,000/mês = 300 doadores de $10/mês
- **Estável (2 devs)**: $8,000/mês = 400 doadores de $20/mês
- **Crescimento (3-4 devs + infra)**: $20,000/mês = 500 doadores média $40/mês

**Transparência radical**: Relatório financeiro público mensal. Cada doação (anônima ou não) registrada. Gastos detalhados por categoria.

### 2.2 Assinatura Ética (Tier 2 — Opcional)

Nem todo modelo de assinatura é extrativo. Uma assinatura pode ser um **ato de sustentação**, não de extração.

**Liberation Plus** — $5/mês ou $50/ano:
- Suporte ao desenvolvimento (o produto é idêntico ao gratuito)
- Badge "Sustainer" no perfil (opt-in, não visível por padrão)
- Voto em prioridades do roadmap (governança compartilhada)
- Acesso a builds beta (TestFlight/Play Store beta)
- Sem features exclusivas. Sem paywall. Sem "premium".

**O que NUNCA será cobrado**:
- ❌ Protocolos (cada protocolo adicional é um direito, não um privilégio)
- ❌ Limite de contas (multi-conta é liberdade)
- ❌ Funcionalidades de privacidade (E2EE é um direito)
- ❌ Temas ou customização
- ❌ Exportação de dados

### 2.3 Grants & Financiamento Institucional (Tier 3)

**Fundações que financiam infraestrutura digital aberta**:

| Programa | Foco | Ticket Médio |
|----------|------|-------------|
| **NLnet Foundation** (NGI Zero) | Internet aberta, privacidade, descentralização | €50K-€150K |
| **Open Technology Fund** (OTF) | Anti-censura, liberdade de expressão | $50K-$300K |
| **Mozilla Foundation** | Internet saudável, privacidade, open-source | $10K-$100K |
| **EU Horizon Europe** | Infraestrutura digital, soberania de dados | €200K-€2M (consórcio) |
| **Protocol Labs** (IPFS/Filecoin) | Descentralização, P2P | $25K-$100K |
| **Sovereign Tech Fund** (Alemanha) | Infraestrutura digital de código aberto | €50K-€500K |

**Estratégia de grants**:
1. **Fase 1** (pré-MVP): NLnet NGI Zero — construção do Nostr adapter + modelo de dados
2. **Fase 2** (MVP): OTF — anti-censura, suporte a Tor, resistência a bloqueios
3. **Fase 3** (crescimento): STF — infraestrutura de código aberto, sustentabilidade
4. **Fase 4** (maturidade): consórcio EU Horizon — pesquisa em social media descentralizada

### 2.4 Cooperativismo (Tier 4 — Visão de Longo Prazo)

A estrutura jurídica ideal não é uma startup (Delaware C-Corp mirando exit), mas uma **cooperativa de trabalhadores** ou **fundação sem fins lucrativos**.

**Modelos de referência**:
| Organização | Estrutura | O que aprendemos |
|-------------|-----------|-----------------|
| **Mastodon gGmbH** | GmbH sem fins lucrativos (Alemanha) | Financiado por doações + grants + Mastodon Hosting |
| **Signal Foundation** | 501(c)(3) (EUA) | Financiado por doação inicial de $50M + grants + doações |
| **Element / Matrix.org Foundation** | UK Foundation + empresa | Governança separada do produto comercial |
| **Bluesky PBLLC** | Public Benefit LLC (EUA) | PBC obriga missão acima de lucro. Financiado por VC inicialmente. |
| **Plataformas.coop** (Brasil) | Cooperativa de tecnologia | Modelo brasileiro, autogestão, assembleias |

**Recomendação para o Liberation Client**:
- **Fase inicial** (ano 1-2): Pessoa jurídica individual no Brasil (MEI → Ltda) + Open Collective para transparência
- **Fase intermédia** (ano 2-3): Fundação sem fins lucrativos (modelo Mastodon gGmbH) na Alemanha ou Holanda
- **Fase madura** (ano 4+): Cooperativa de trabalhadores com governança comunitária (modelo Plataformas.coop)

### 2.5 Serviços Complementares (NÃO no app)

Serviços que financiam o desenvolvimento sem poluir o produto:

- **Liberation Hosting**: Hospedagem gerenciada de soju (bouncer IRC) e bridges Matrix. $3/mês.
- **Consultoria**: Empresas e organizações que querem migrar para comunicação federada.
- **Treinamentos**: Workshops sobre descentralização, protocolos abertos, segurança digital.

O app permanece gratuito. Os serviços são separados, opcionais, e opacos ao usuário do app.

---

## 3. Orçamento por Fase

### Fase 1: Protótipo (Ano 1)
| Item | Custo mensal | Custo anual |
|------|-------------|------------|
| 1 desenvolvedor Rust/Flutter (Brasil) | R$ 15.000 | R$ 180.000 |
| Infraestrutura (domínio, CI, servidores de teste) | R$ 500 | R$ 6.000 |
| Taxas de app store (Apple Developer + Google Play) | R$ 200 | R$ 2.400 |
| **Total** | **R$ 15.700** | **R$ 188.400** |
| **≈ USD** | **$3,000** | **$36,000** |

### Fase 2: MVP Público (Ano 2)
| Item | Custo mensal | Custo anual |
|------|-------------|------------|
| 2 desenvolvedores | R$ 30.000 | R$ 360.000 |
| 1 designer UX (part-time) | R$ 5.000 | R$ 60.000 |
| Infraestrutura | R$ 1.500 | R$ 18.000 |
| Community manager | R$ 4.000 | R$ 48.000 |
| **Total** | **R$ 40.500** | **R$ 486.000** |
| **≈ USD** | **$8,000** | **$96,000** |

### Fase 3: Crescimento (Ano 3+)
| Item | Custo mensal | Custo anual |
|------|-------------|------------|
| 4 desenvolvedores | R$ 60.000 | R$ 720.000 |
| 1 designer + 1 community manager | R$ 12.000 | R$ 144.000 |
| Infraestrutura + auditoria de segurança | R$ 5.000 | R$ 60.000 |
| Reserva legal / contingência | R$ 3.000 | R$ 36.000 |
| **Total** | **R$ 80.000** | **R$ 960.000** |
| **≈ USD** | **$16,000** | **$192,000** |

---

## 4. O Plano Brasileiro

O projeto nasce no Brasil. Isso traz desafios e vantagens específicas:

**Desafios**:
- Câmbio: receber em USD/EUR e pagar em BRL = exposição cambial
- Meios de pagamento: Pix é essencial, mas invisível para doadores internacionais
- Burocracia: abrir e manter pessoa jurídica no Brasil é caro e lento

**Vantagens**:
- Custo de desenvolvimento mais baixo que EUA/Europa
- Comunidade tech vibrante e engajada
- Identidade cultural forte (diferenciação do Vale do Silício)
- Lei do Bem e leis de incentivo à inovação (isenção fiscal)
- Potencial de ser o primeiro grande projeto open-source brasileiro de infraestrutura social

**Estratégia cambial**:
- Doações internacionais via Open Collective (recebe em USD/EUR, repassa para Brasil)
- Doações nacionais via Pix (custo zero, instantâneo)
- Reserva de 6 meses de custos operacionais em USD (proteção cambial)

---

## 5. O Que NUNCA Faremos

| Anti-padrão | Por que não |
|-------------|------------|
| **VC / Venture Capital** | Incompatível com missão. VC exige exit (venda ou IPO) ou crescimento infinito. |
| **Publicidade** | Incompatível com anti-engajamento. Ads otimizam para atenção, não para bem-estar. |
| **Venda de dados** | Incompatível com privacidade. Dados são do usuário, não nossos. |
| **Tokens / ICO / "Web3"** | Incompatível com simplicidade. Criptomoedas próprias são esquemas Ponzi com marketing descentralizado. |
| **Features pagas no app** | Incompatível com equidade. Funcionalidades de comunicação são direitos, não privilégios. |
| **Donation shaming** | Incompatível com respeito. Nunca mostrar popup de "você ainda não doou". |
| **Exclusividade de protocolo** | Incompatível com interoperabilidade. Nunca cobrar por protocolo adicional. |

---

## 6. Métricas de Sustentabilidade

| Métrica | Alvo Ano 1 | Alvo Ano 2 | Alvo Ano 3 |
|---------|-----------|-----------|-----------|
| Doadores recorrentes | 50 | 300 | 1,000 |
| Doação média mensal | R$ 30 | R$ 40 | R$ 35 |
| Receita mensal de doações | R$ 1.500 | R$ 12.000 | R$ 35.000 |
| Grants aprovados | 1 | 2 | 3 |
| Receita de grants (anual) | €50K | €150K | €300K |
| Assinantes Liberation Plus | — | 200 | 1,500 |
| Receita de assinaturas | — | R$ 10.000 | R$ 75.000 |
| **Receita total mensal** | **R$ 3.500** | **R$ 22.000** | **R$ 110.000** |
| **Custo total mensal** | **R$ 15.700** | **R$ 40.500** | **R$ 80.000** |
| **Gap mensal** | **-R$ 12.200** | **-R$ 18.500** | **+R$ 30.000** |

**Nota**: O gap do Ano 1 e 2 é coberto por grants (não mensais, mas lump sums). O Ano 3 é autossustentável com doações + assinaturas, sem grants.

---

## 7. Transparência Radical

Todo mês, publicamos:

```
═══════════════════════════════════════
  RELATÓRIO FINANCEIRO — JULHO 2026
═══════════════════════════════════════

ENTRADAS
  Doações Open Collective     $2,450
  Doações GitHub Sponsors     $1,200
  Doações Pix                 R$ 4.800
  Grant NLnet (parcela 3/6)   €8,300
  Assinaturas Liberation Plus R$ 2.100
  ─────────────────────────────────────
  Total                       ≈ R$ 31.000

SAÍDAS
  Desenvolvedor 1            R$ 15.000
  Desenvolvedor 2            R$ 12.000
  Infraestrutura              R$ 1.200
  Taxas Open Collective       R$ 1.500
  Contabilidade               R$ 800
  Reserva                     R$ 500
  ─────────────────────────────────────
  Total                       R$ 31.000

CAIXA
  Saldo anterior             R$ 45.000
  Saldo atual                R$ 45.000
  Meses de runway                2.9

═══════════════════════════════════════
```

Publicado no site, no repositório, e no feed do projeto. Transparência como princípio, não como marketing.

---

*Documento gerado na Lacuna 1 do /loop de finalização.*
