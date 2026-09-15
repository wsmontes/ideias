# Estudo de Caso 45 — Todoist: O Parser de Linguagem Natural Quick Add Que Interpreta "Reunião Amanhã 15h @trabalho !!1", o Sistema Karma Anti-Exploração e 18 Anos Bootstrapped

> **Data:** 2026-07-03
> **Loop:** 45 de ∞ (Reescrita)
> **Categoria:** Produtividade / NLP / Bootstrapping

---

## 0. Linhagem

```
Listas de tarefas em papel — o método original. Sem sync. Sem lembretes.
Wunderlist (2011), Remember The Milk (2005) — primeiros apps. Cloud.
Todoist (2007) — NLP Quick Add. Karma. Bootstrapped. Zero VC.
Todoist hoje (2026) — 52M signups. US$ 25M ARR. 96 funcionários. 18 anos autofinanciado.
```

---

## 1. Origem

Amir Salihefendić nasceu na Bósnia e fugiu da guerra nos anos 90. Adolescência na Dinamarca, ciência da computação na Aarhus University. Em 2007, sobrecarregado com aulas + 2 empregos + projetos pessoais, construiu o Todoist como ferramenta pessoal. Um post de blog gerou cobertura do Lifehacker, Digg e ReadWriteWeb.

Em 2008, co-fundou o Plurk (rede social popular na Ásia). O Todoist ficou abandonado por 4 anos — apenas bugs corrigidos, servidor rodando. Quando voltou em 2011, tinha 300.000 usuários. O produto cresceu sozinho, sem founder, sem time, sem marketing. Fundou a Doist em 2012. Recusou US$ 500.000 de um VC que queria substituí-lo como CEO. 18 anos depois, a empresa é 100% bootstrapped, lucrativa, 96 funcionários em 28+ países.

---

## 2. Arquitetura Técnica

### 2.1 O Quick Add Parser: NLP Para Input de Tarefas

A feature mais tecnicamente distintiva do Todoist é o **Quick Add** — um parser de linguagem natural que processa entrada de texto e extrai data, hora, projeto, labels e prioridade. É uma das implementações de NLP mais antigas em produção contínua (2007).

**Pipeline**: tokenização por espaço → reconhecimento de padrões (formatos de data como "amanhã", "segunda", "15h", prefixos de projeto como "#trabalho", labels como "@home", prioridade como "!!1" a "!!4") → resolução de ambiguidade ("amanhã 15h" = data+hora; "toda segunda 9h" = recorrência; "next Friday" = ambiguidade cultural EUA vs. Europa) → extração (tokens reconhecidos removidos, restante = título da tarefa).

**Por que NLP em vez de date picker**: um date picker (clique no calendário, selecione dia, selecione hora) leva 5-10 segundos. O Quick Add processa tudo em uma string de texto em <1 segundo. Para usuários que adicionam dezenas de tarefas por dia, essa diferença de latência de input é a diferença entre usar o app e abandoná-lo.

**Suporte multi-idioma**: 20+ idiomas, cada um com convenções de data, formatos de hora e expressões idiomáticas próprias. O parser precisa resolver "next Friday" (inglês americano = próxima sexta; inglês britânico = sexta da próxima semana) em milissegundos.

### 2.2 O Sistema Karma: Gamificação Anti-Exploração

O Karma do Todoist é um estudo de caso em gamificação que não explora:

- **Pontos por tarefas concluídas**, não por tarefas criadas. O incentivo é terminar, não acumular.
- **Pontos maiores para tarefas prioritárias** (p1 > p4) e tarefas com data limite.
- **Streaks por dias consecutivos** atingindo meta diária.
- **Níveis cosméticos** (Beginner → Enlightened).
- **Sem leaderboard, sem competição, sem recompensas tangíveis.**

A filosofia: gamificação deve reforçar comportamento desejado (concluir tarefas importantes consistentemente), não substituir motivação intrínseca por caça a pontos. O sistema dá pontos pelo que você termina, não pelo que você cria — prevenindo a armadilha de "produtividade de mentira" que afeta a maioria dos apps de hábitos.

### 2.3 Arquitetura de Sync: 10+ Plataformas

O Todoist sincroniza tarefas entre Web, iOS, Android, macOS, Windows, Linux, Apple Watch, Wear OS, plugins de navegador e plugins de email. A arquitetura usa consistência eventual com resolução de conflitos last-writer-wins: cada cliente mantém SQLite local (mobile) ou IndexedDB (web); mudanças são enviadas como operações com timestamps; o servidor aplica em ordem cronológica e transmite o estado resultante.

---

## 3. Lições de Engenharia

### 3.1 Um produto que resolve dor real sobrevive ao abandono do founder

0 → 300.000 usuários em 4 anos sem founder ativo. Isso só é possível quando o problema (gestão de tarefas) é perene e universal, o produto é funcional mesmo sem updates, e o modelo freemium não exige vendas ativas.

### 3.2 NLP para input é superior a forms quando a frequência de uso é alta

Date pickers são mais precisos, mas 5-10× mais lentos. Para apps de alta frequência (dezenas de inputs/dia), velocidade de input é requisito de retenção, não luxo.

### 3.3 Gamificação que mede conclusão, não criação, é anti-frágil

Se o Karma desse pontos por tarefas criadas, os usuários criariam tarefas falsas. Como dá pontos por conclusão, o incentivo é terminar trabalho real — alinhando a gamificação com o objetivo do produto.

---

## 4. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | Todoist (Doist) |
| **Fundação** | 28 janeiro 2007 |
| **Fundador** | Amir Salihefendić |
| **Categoria** | Produtividade / Gestão de Tarefas |
| **Signups** | 52M+. ARR: ~US$ 25M |
| **Funcionários** | 96 (28+ países, 100% remoto) |
| **NLP** | Quick Add: 20+ idiomas, data/hora/projeto/label/prioridade em <1s |
| **Karma** | Conclusão, não criação. Streaks. Sem leaderboard |
| **Sync** | Consistência eventual. 10+ plataformas |
| **Concorrentes** | TickTick, Microsoft To Do, Things |

---

## Fontes

- [SaaS Club — He Abandoned His Freemium SaaS for 4 Years and It Still Grew (Amir Salihefendić)](https://saasclub.io/podcast/amir-salihefendic-doist-part1/)
- [SaaS Unbound — 17 Years of Bootstrapping with $200K Revenue Per Employee](https://saas.group/podcasts/saas-unbound-interview-amir-salihefendic-doist/)
- [Trophy.so — Todoist's Gamification Strategy: A Case Study (2024)](https://trophy.so/blog/todoist-gamification-case-study)
- [Fast Company — With 40 People in 20+ Countries, This Startup Wants to Make Physical Offices Irrelevant](https://www.fastcompany.com/3048812/with-40-people-spread-across-20-countries-this-startup-wants-to-make-physical-of)
