# Estudo de Caso 39 — Shopify: O Monólito Modular de 2.8M Linhas de Ruby Com Pod Architecture (100+ Pods), Packwerk+Sorbet, YJIT 3.4 (92% Mais Rápido) e Ghostferry Com TLA+

> **Data:** 2026-07-03
> **Loop:** 39 de ∞ (Reescrita)
> **Categoria:** E-commerce / Infraestrutura / Modular Monolith

---

## 0. Linhagem

```
Lojas físicas — vitrine, balcão, dinheiro. Sem internet.
OsCommerce (2000), Magento (2008) — PHP. Complexos. Pesados.
Shopify (2006) — Ruby on Rails. SaaS. "Sua loja online em minutos."
Shopify hoje (2026) — 5.5M+ merchants. 173B requests/dia. Pod architecture. YJIT. Packwerk.
```

O Shopify fez a aposta contrária ao consenso da indústria em 2015-2020: enquanto startups reescreviam monolitos em microserviços, o Shopify investiu em modularização interna, isolamento de infraestrutura via pods e compilação JIT de Ruby. O resultado é um "Majestic Monolith" que na Black Friday 2024 processou US$ 5B em GMV com 173 bilhões de requisições, 45 milhões de leituras de banco por segundo e 66 milhões de mensagens Kafka por segundo.

---

## 1. Arquitetura Técnica

### 1.1 O "Redismageddon" e a Pod Architecture

Em 2015, depois de shardear horizontalmente o MySQL, todos os shards ainda dependiam de uma **única instância compartilhada de Redis** para cache, sessões e filas de jobs. Quando essa instância caiu, **cada loja na plataforma caiu junto** — storefronts, checkouts, tudo. O incidente — apelidado "Redismageddon" — ensinou uma lição: evitar qualquer recurso compartilhado entre todas as lojas.

A resposta foi a **pod architecture**: particionar infraestrutura em unidades independentes. Cada pod contém seu próprio **MySQL**, **Redis** e **Memcached** — completamente isolados. Pods nunca se comunicam entre si. Se o Pod A falha, apenas as lojas no Pod A são afetadas. Cada pod tem um par ativo + recovery datacenter, com failover de ~1 minuto.

**Sorting Hat**: script Lua rodando em nginx/OpenResty. Consulta o domínio da loja na request, injeta header `X-Sorting-Hat-PodId` com o pod alvo, e o application worker lê o header para conectar ao MySQL/Redis/Memcached corretos.

**Escala**: 100+ pods, 5.5M+ merchants, 284M edge requests/minuto em pico Black Friday.

### 1.2 Packwerk + Sorbet: Fronteiras Aplicadas Estaticamente

O Shopify desenvolveu o **Packwerk** (open-source) — ferramenta de análise estática que impõe fronteiras de dependência entre módulos do monólito:

- Desenvolvedores declaram quais módulos podem conversar com quais
- Dependências cross-module não-intencionais são flaggadas automaticamente
- Módulos só interagem via **interfaces públicas definidas**
- `package_todo.yml` como mecanismo de "bankruptcy" — impede novas violações enquanto existentes são corrigidas
- Combinado com **Sorbet** (type checker gradual): assinaturas Sorbet nas interfaces públicas dão a Packwerk "dentes mais afiados"

**Packwerk v3.0**: removeram privacy checks — estavam transformando Packwerk em ferramenta de API design não-intencional e quebrando convenções Rails. **Retrospectiva**: zero-violation isolation é extremamente difícil; dependências não-rastreáveis (rotas, fixtures, initializers, constantes dinâmicas) são pontos cegos.

### 1.3 YJIT 3.4: 92% Mais Rápido Que o Interpretador CRuby

Shopify investiu massivamente em performance de Ruby. **YJIT** (Yet Another JIT) — compilador JIT construído em Rust, mergeado no CRuby — atingiu em janeiro 2025:

- **92% mais rápido que o interpretador CRuby** em x86-64 nos benchmarks headline
- 5-7% mais rápido que YJIT 3.3.6 com **menos uso de memória**
- Melhorias: inlining (56,3% de chamadas C inline no benchmark lobsters, 82,5% no liquid-render), register allocator mais sofisticado, lazy frame pushing especulativo, serialização binária
- **Black Friday 2024**: YJIT 3.4 pre-release rodando em todos os StoreFront Renderers — processando 80M+ requests/minuto

**ZJIT** (anunciado RubyKaigi 2025): sucessor method-based JIT com SSA-based IR. Usa `call`/`ret` da CPU para chamadas JIT-to-JIT. Foco: maintainability, extensibility, persistência de código compilado entre deploys. Early prototype: 0.010s vs YJIT 0.016s em Fibonacci recursivo.

### 1.4 Ghostferry: Migração Zero-Downtime Com TLA+

**Ghostferry** (Go, open-source, MIT) é a ferramenta de migração de dados entre instâncias MySQL. Design formalmente especificado em **TLA+** para verificação de invariantes de correção.

**Fluxo**: binlog tailing (Ghostferry simula MySQL slave) → batch copy por primary key ranges → wait for catch-up → cutover com **MRSW lock** (multi-reader-single-writer, via Redis) → routing update via Sorting Hat.

**Downtime**: ~2,5 segundos médios por loja. Centenas de milhares de shops movidas por ano para rebalanceamento. **Pod Balancer**: classifica lojas por consumo de recursos, identifica pods sobrecarregados, move lojas pesadas para pods menos carregados. Merchants de alto tráfego (flash sales de celebridades) recebem pods dedicados.

---

## 2. Lições de Engenharia

### 2.1 Modularização interna substitui separação física

Rails Engines + Packwerk fornecem isolamento de domínio. Pods fornecem isolamento de infraestrutura. O resultado combina simplicidade operacional de monólito com isolamento de falhas de arquitetura distribuída — sem complexidade de rede de microserviços.

### 2.2 YJIT é um investimento de 10 anos em performance de linguagem

Em vez de reescrever em Go/Rust, Shopify investiu em tornar Ruby mais rápido. YJIT 3.4 é 92% mais rápido que o interpretador. ZJIT é o próximo passo. A aposta: melhorar a linguagem é mais barato que reescrever o código.

### 2.3 Pods são a resposta ao problema de noisy neighbor em SaaS multi-tenant

Sem isolamento de infraestrutura, uma loja com flash sale degrada todas as outras. Com pods, a loja pesada vai para um pod dedicado. O Pod Balancer automatiza isso continuamente.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Shopify |
| **Fundação** | 2006. IPO: 2015 (NYSE: SHOP) |
| **Categoria** | E-commerce / Plataforma |
| **Monólito** | Ruby on Rails: 2.8M linhas, 500K+ commits. Pod architecture (100+ pods) |
| **Modularização** | Packwerk (static analysis) + Sorbet (type checker). Rails Engines |
| **JIT** | YJIT 3.4: 92% faster que CRuby. ZJIT: SSA-based, experimental em Ruby 4.0 |
| **Migração** | Ghostferry (Go, TLA+). Downtime ~2.5s. Centenas de milhares de shops/ano |
| **Black Friday 2024** | US$ 5B GMV. 173B requests. 45M DB reads/s. 66M Kafka msg/s. 97% CDN cache hit |
| **Infra** | GCP. Kubernetes (2.4M cores). MySQL 8, Redis, Memcached isolados por pod |

---

## Fontes

- [Shopify Engineering — A Pods Architecture To Allow Shopify To Scale (2018)](https://shopify.engineering/a-pods-architecture-to-allow-shopify-to-scale)
- [Shopify Engineering — Shard Balancing: Moving Shops Confidently with Zero-Downtime at Terabyte-scale](https://shopify.engineering/mysql-database-shard-balancing-terabyte-scale)
- [Shopify Engineering — Enforcing Modularity in Rails Apps with Packwerk (2020)](https://shopify.engineering/enforcing-modularity-rails-apps-packwerk)
- [Shopify Engineering — A Packwerk Retrospective (2024, railsatscale.com)](https://shopify.engineering/a-packwerk-retrospective)
- [Rails at Scale — YJIT 3.4: Even Faster and More Memory-Efficient (Jan 2025)](https://railsatscale.com/2025-01-10-yjit-3-4-even-faster-and-more-memory-efficient/)
- [RubyKaigi 2025 — ZJIT: Building a Next Generation Ruby JIT (Maxime Chevalier-Boisvert, Shopify)](https://www.slideshare.net/slideshow/zjit-building-a-next-generation-ruby-jit/278807093)
- [GitHub — Shopify/ghostferry: MySQL migration tool (Go, TLA+, MRSW locks)](https://github.com/Shopify/ghostferry)
- [Sujeet Jaiswal — Shopify: Pod Architecture for Multi-Tenant Isolation at Scale](https://sujeet.pro/articles/system-design/real-world-case-studies/shopify-pod-architecture)
- [ByteByteGo — Shopify Tech Stack (Black Friday 2024 metrics)](https://blog.bytebytego.com/p/shopify-tech-stack)
- [GitHub — Shopify/yjit-bench: Official YJIT benchmarks](https://github.com/Shopify/yjit-bench)
