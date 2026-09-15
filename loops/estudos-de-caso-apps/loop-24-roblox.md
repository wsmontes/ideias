# Estudo de Caso 24 — Roblox: 30,6M CCU Com 24 Edge DCs + TACO Tuesday, Redis Federado a 1,38B QPS (6.000 Nós, Proxy Envoy, cgroups Colocation), Luau (JIT x64/ARM64, Native Codegen 1,5-2,5×, Semantic Subtyping) e Matchmaking de 4 Bilhões de Combinações/Segundo

> **Data:** 2026-07-03
> **Loop:** 24 de ∞ (Reescrita)
> **Categoria:** Gaming / UGC / Infraestrutura Distribuída

---

## 0. Linhagem

```
LEGO (1932-) — blocos físicos. Construção livre. O modelo mental.
Interactive Physics (1989) — simulação 2D educacional. Alunos preferiam criar.
Roblox (2006) — mundos 3D com física, avatares e scripting.
Roblox hoje (2026) — 85,3M DAU. 30,6M CCU. 24 edge DCs. TACO Tuesdays. Redis 1,38B QPS. Luau.
```

O Roblox não é um jogo — é uma plataforma bilateral: criadores constroem experiências 3D com Roblox Studio e scripting em Luau; jogadores as consomem gratuitamente. A economia é baseada em Robux, com repasse a criadores. É o maior empregador de jovens desenvolvedores de jogos do mundo.

---

## 1. Arquitetura Técnica

### 1.1 Infraestrutura: 24 Edge DCs, 30,6M CCU, TACO Tuesdays

**Topologia.** 24 edge data centers globais rodam servidores de jogo — usuários são roteados ao edge mais próximo. 2 core data centers rodam serviços centralizados (website, recomendações, safety filters, economia virtual, publishing). Rede privada global interconecta tudo. Edges atuam como firewall protegendo cores.

**Cloud bursting.** Quando capacidade física satura, "virtual edge data centers" são provisionados em cloud pública. "There's no way to rack and stack servers in 12 hours, so we work with our cloud partners." Arquitetura **cellular**: matchmaking abstraído de saber se célula é física ou cloud.

**Escala.** Pico de **30,6 milhões de jogadores concorrentes** (junho 2025), dobrando de 13,9M em abril 2024. **Grow a Garden**: 21,6M CCU (recorde Guinness). Dead Rails: 1M CCU em um único jogo. 85,3M DAU (Q4 2024). **1.600+ microserviços**, 1.000+ engenheiros.

**Ciclo semanal de capacidade:**
- **Segunda**: incident reviews
- **Terça (TACO Tuesday — Test Actual Capacity On)**: estrangulamento intencional de serviços em produção. "Every Tuesday, engineers take a few of their 1,600+ microservices, intentionally constrain their capacity in production, observe attributes, and fix issues before the weekend."
- **Quinta**: review de capacidade para updates de criadores
- **Sexta**: provisionamento de cloud para pico do fim de semana
- **Contínuo**: plataforma de **chaos testing** injeta falhas aleatoriamente em produção — "randomly injects faults, exhausts resources, and terminates processes in production"

**Capacidade prevista com até 2 anos de antecedência.** **C3 (Continuous Capacity Correctness)** dashboards: cada time prediz e gerencia sua própria CPU capacity. **"Dark" capacity** mantida para cortes de cabo de rede e falhas inesperadas. **Executivos participam de on-call nos fins de semana** — incluindo o CEO.

**Matchmaking.** Redesenhado para "thundering herds" — **não faz throttle** de joins. Avalia até **4 bilhões de combinações possíveis por segundo**. Design target: 10 milhões de joins em 10 segundos. Text-filter system: 250K req/s em pico, 300+ AI inference pipelines. Analytics pipeline: 2 trilhões de eventos/ano, pode ver 30% de surge de tráfego de um único update popular.

### 1.2 Cache Redis Federado: 1,38 Bilhão de QPS

**O problema.** Redis usa protocolo Gossip para health/status/shard mapping. Acima de ~400-500 nós por cluster, "the gossip chatter consumes substantial CPU and network resources, degrading latency." AWS ElastiCache: limite de 500 nós.

**A solução.** Arquitetura federada: **client reverse proxy** (baseado em Envoy) na frente de múltiplos clusters Redis independentes (15+), cada um limitado a 400 nós. Proxy apresenta todos como "a singular, unified, and scalable caching service."

| Métrica | Valor |
|---|---|
| Peak cluster-wide cache QPS | **1,38 bilhão** |
| Single logical cluster peak | 100M QPS |
| Largest deployment | **6.000+ nós Redis**, 15+ clusters |
| Previous single-cluster peak | 10M QPS |
| Per-cluster node limit | 400 (Gossip ceiling) |

**Grow a Garden surge.** Abril 2025: jogo explodiu de 2,8M para **21,6M CCU** em 3 meses. **10× mais tráfego** no maior cluster Redis, **3× mais tráfego** no caching total. "Not part of our long-term capacity plan."

**Otimizações durante o surge:** Novo scheduling de memória entre grupos de máquinas físicas. Fine-tuning de container sizes para reduzir fragmentação. Autoscaling de proxy com target CPU mais agressivo (~10% CPU improvement). Otimização de health checks — eliminação de heartbeats desnecessários. **Colocation Redis+Envoy via cgroups container-level isolation**: Redis (memory-bound) e Envoy (compute-bound) na mesma pool de máquinas, reduzindo capacidade total em **25%**.

**Protocolo de migração de 3 estágios:** dual-writing (proxy escreve em ambos os clusters) → parity + read switch (TTL-bounded, sync natural) → decommissioning.

**Futuro**: migração para **ValKey** multi-tenant — "next-generation multitenant caching service built on ValKey" para melhor resource sharing, isolation e cost efficiency.

### 1.3 Luau: Lua Fork Com Native Codegen, Type System e Sandbox

**Por que fork?** Lua padrão não escala para milhões de linhas de código e milhares de scripts simultâneos. Roblox considerou LuaJIT mas rejeitou: "limited platform support (especially video game consoles) and maintenance challenges." Construiu Luau "largely from scratch."

**Performance.** Intérprete comparável ao modo interpretador do LuaJIT. **Native code generation (outubro 2023)**: compila bytecode Luau para código de máquina em **x64 e ARM64** (incluindo Apple Silicon). Speedup: **1,5-2,5×** para código compute-intensive. Compilador processa ~**950.000 linhas/s** em um core Ryzen 5900X. Otimizações: constant folding, dead code elimination, peephole optimization, inline caching.

**Progressive type system.** Tipagem gradual: anotações adicionadas incrementalmente. Type inference automática. Dois modos: `--!strict` (preciso, erros antes do runtime) e não-strito (permissivo, minimiza falsos positivos). **Structural typing** (não nominal) — verifica campos de tabelas, não nomes de classes. **Primeira linguagem a implementar semantic subtyping** para larga base de usuários. Inspirado em TypeScript, Python e LuaJIT.

**Sandbox.** Biblioteca padrão sem filesystem/OS. `safeenv`: isola global tables de scripts individuais e previne monkey-patching de built-in libraries. Resource limits: memória e interrupção de código long-running.

**Adoção externa (2024-2025):** Remedy Entertainment (Alan Wake 2 — removeu 80K linhas de legacy code), Digital Extremes (Warframe — melhor GC e memory efficiency), Giants Software (Farming Simulator 2025 — migrou de LuaJIT), Linden Lab (Second Life SLua, dezembro 2025).

### 1.4 Sentinel: Contrastive Learning Para 6,1 Bilhões de Mensagens/Dia

O Roblox processa **6,1 bilhões de mensagens de chat por dia**. O **Sentinel** (open-source, agosto 2025) usa **contrastive learning** inspirado no SimCLR: Positive Index (histórico de chat sem violações), Negative Index (~13.000 exemplos confirmados de child-endangerment). Cada mensagem vira embedding; **similaridade de cosseno** mede distância aos dois índices; **skewness estatística** (não média) detecta outliers — mensagens raras mas perigosas em meio a centenas de benignas.

Text filter: **750K+ req/s** em GPU. Filtro PII: 370K RPS. Transição CPU→GPU com quantização/distilação: 4× capacidade, 30% menos falsos positivos, 25% mais detecção. H1 2025: ~1.200 reports ao NCMEC. **35% proativos** — detectados antes de qualquer report de usuário.

---

## 2. Inovações

**2.1 Federated Redis com proxy reverso.** Resolver o limite de 400-500 nós do Gossip com uma camada de indireção, sem modificar o Redis. 6.000+ nós operando como serviço unificado. Mesmo padrão do DocDB do Stripe e Data Service do Discord.

**2.2 TACO Tuesday como disciplina de engenharia.** Testar capacidade em produção toda terça-feira, estrangulando serviços intencionalmente. Produz conhecimento sobre limites reais que nenhum teste de staging conseguiria.

**2.3 Workload colocation Redis+Envoy via cgroups.** 25% de redução de capacidade sem degradação. Otimização não "sexy" que economiza milhões em escala.

**2.4 Semantic subtyping em linguagem de script para milhões de usuários.** Luau é a primeira linguagem a implementar semantic subtyping para uma base de usuários massiva — permitindo type checking estrutural em código criado por crianças e adolescentes.

---

## 3. Lições de Engenharia

### 3.1 Federar middleware atrás de proxy é mais barato que reescrever

Limite de 400-500 nós do Gossip resolvido com proxy reverso — sem modificar Redis. É o padrão de indireção como mecanismo de escala.

### 3.2 TACO Tuesday é capacity planning como disciplina semanal

Estrangular serviços em produção revela limites reais. Requer cultura que tolera degradação controlada.

### 3.3 Colocation Redis+Envoy via cgroups reduz capacidade em 25%

Memória-bound + compute-bound na mesma máquina, isolados via container-level cgroups. Economia de milhões em escala de 6.000+ nós.

---

## 4. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Roblox |
| **Lançamento** | 2006. IPO: março 2021 (NYSE: RBLX, US$ 41B) |
| **Categoria** | Gaming / UGC / Plataforma Social |
| **DAU** | 85,3M (Q4 2024) |
| **Pico CCU** | 30,6M (junho 2025). Recorde: 21,6M em único jogo |
| **Infra** | 24 edge DCs + 2 core DCs. 1.600+ microserviços. Cloud bursting. TACO Tuesdays |
| **Cache** | Redis federado: 1,38B QPS, 6.000+ nós, 15+ clusters, proxy Envoy, cgroups (25% capacity reduction). Futuro: ValKey |
| **Matchmaking** | 4B combinações/segundo. Target: 10M joins/10s |
| **Luau** | Lua fork, native codegen x64/ARM64 (1,5-2,5×), type system progressivo, semantic subtyping, sandbox |
| **Segurança** | Sentinel: contrastive learning, skewness, 6,1B msgs/dia, open-source |
| **Concorrentes** | Minecraft, Fortnite Creative, Unity |

---

## Fontes

- [Roblox Newsroom — The Infrastructure Supporting Record-Breaking Experiences (Jun 2025): 24 edge DCs, 30,6M CCU, TACO Tuesdays, 4B matchmaking](https://d3fel7ao8ljmgc.cloudfront.net/en-nz/newsroom/2025/06/roblox-infrastructure-supporting-record-breaking-games)
- [Roblox Newsroom — How Roblox's Cache Sustained 1.38B QPS Beyond Redis Limits (Mar 2026): 6.000+ nós, federated proxy Envoy, cgroups, Grow a Garden 10×](https://d3fel7ao8ljmgc.cloudfront.net/en-au/newsroom/2026/03/roblox-cache-sustained-138b-qps-beyond-redis-limits)
- [Luau.org — Performance: native codegen 1,5-2,5×, 950K linhas/s, x64/ARM64](https://luau.org/performance/)
- [GitHub — luau-lang/luau: MIT License, semantic subtyping, progressive typing, sandbox](https://github.com/luau-lang/luau)
- [Roblox Newsroom — Open-Sourcing Roblox Sentinel: Preemptive Risk Detection (Ago 2025)](https://d3fel7ao8ljmgc.cloudfront.net/en-au/newsroom/2025/08/open-sourcing-roblox-sentinel-preemptive-risk-detection)
- [Fast Company — How Roblox handles millions of players on viral games (Nov 2025)](https://www.fastcompany.com/91357216/how-roblox-handles-millions-of-players-on-viral-games-like-grow-a-garden)
