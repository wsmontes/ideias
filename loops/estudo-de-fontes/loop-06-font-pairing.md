# 🔤 Estudo de Fontes Loop 06 — Font Pairing: A Arte de Combinar Fontes (E Quando Quebrar Todas as Regras)

> **Data:** 2026-07-02
> **Loop:** 6 de ∞
> **Tema:** Como combinar fontes — as 10 regras de ouro, a ciência por trás da harmonia tipográfica, os combos clássicos que funcionam há 100 anos (Garamond + Helvetica, Baskerville + Futura), os combos modernos que dominam 2025 (Inter + Fraunces, Playfair + Lato), superfamílias que resolvem o problema de uma vez, e os piores pairings que você NUNCA deve fazer.

---

## 0. O Problema: 300.000 Fontes. Você Precisa de DUAS.

Escolher UMA fonte já é difícil. Escolher DUAS que funcionem JUNTAS é exponencialmente mais complexo. O espaço de combinações possíveis entre 300.000 fontes é de ~45 BILHÕES de pares. Você precisa de um método.

A maioria dos designers faz font pairing por INSTINTO — "essa parece bonita com essa." Funciona às vezes. Mas quando falha, falha FEIO: a página parece amadora, a hierarquia some, o texto vira ruído.

Este loop te dá **o método**. As 10 regras. Os combos testados. A ciência. E a licença para quebrar tudo com intenção.

---

## 1. As 10 Regras de Ouro do Font Pairing

### Regra 1: Contraste com Propósito (NÃO "Contraste por Contraste")

O erro #1 de iniciante: achar que "contraste" significa "duas fontes MUITO DIFERENTES." Não. Contraste significa **diferença com harmonia.**

| Contraste FUNCIONA | Contraste FALHA |
|---|---|
| Serif + Sans-Serif (categorias diferentes, mesma proporção) | Script floreada + Blackletter gótica (duas personalidades extremas) |
| Bold + Light (pesos diferentes, MESMA família) | Bold condensado + Light expandido (largura + peso conflitantes) |
| Geométrica + Humanista (estrutura diferente, mesmo tom) | Display decorativa + Outra display decorativa (competem) |

**A fórmula mágica:** **Diferença de CATEGORIA + semelhança de PROPORÇÃO.**

Exemplo: **Playfair Display (Didone) + Lato (Humanista Sans)**
- Categorias DIFERENTES (serif vs sans)
- Mesma x-height (~68% da cap height em ambas)
- Mesmo tom (elegante mas não frio)

FUNCIONA. E funciona há 10 anos.

### Regra 2: X-Height Igual é Harmonia Invisível

O fator mais SUBESTIMADO no font pairing é a **x-height**. Duas fontes com a mesma x-height parecem "irmãs" mesmo sendo de categorias diferentes. Duas fontes com x-heights diferentes parecem "brigando" mesmo sendo da mesma categoria.

**O teste do "x":** Coloque `xxXX` nas duas fontes, mesmo tamanho de ponto, lado a lado. Se os 'x' minúsculos têm alturas muito diferentes, você vai precisar de TAMANHOS diferentes para harmonizá-las.

| Combinação | x-height relativa | Verdict |
|---|---|---|
| **Inter + Georgia** | Inter > Georgia | Precisam de tamanhos diferentes |
| **Inter + Source Serif** | Similar | Harmonizam naturalmente |
| **Garamond + Helvetica** | Helvetica >> Garamond | Precisam de compensação (Helvetica menor) |
| **Baskerville + Futura** | Similar | Harmonizam naturalmente |

**A regra prática:** se as x-heights são muito diferentes, COMPENSE com tamanho de ponto. Helvetica a 14px com Garamond a 17px pode equilibrar.

### Regra 3: No Máximo 2-3 Fontes

| Quantidade | Quando Usar |
|---|---|
| **1 fonte (vários pesos)** | UI limpa, dashboards, apps, sistemas |
| **2 fontes** | Sites, landing pages, branding, editorial |
| **3 fontes** | Revistas, layouts complexos, hierarquia tripla (display + heading + body) |
| **4+** | Caos. Só em pôster experimental ou se você é a Paula Scher |

**Se você ACHA que precisa de 3+ fontes, pergunte-se:** "Posso resolver isso com mais PESOS da mesma fonte?" A resposta quase sempre é SIM.

### Regra 4: Uma Fonte é a Estrela. A Outra é o Suporte.

NUNCA bote duas fontes "protagonistas" juntas. Uma vai ser a ESTRELA (headlines, hero, identidade) e a outra vai ser o WORKHORSE (body, UI, labels).

| Estrela | Suporte | Contexto |
|---|---|---|
| **Playfair Display** (Didone dramática) | **Lato** (humanista neutra) | Editorial, moda |
| **Futura** (geométrica icônica) | **Georgia** (transicional para tela) | Portfolio, design |
| **Bebas Neue** (condensada impactante) | **Inter** (neo-grotesca limpa) | Landing page, hero |
| **Recursive** (variável criativa) | **Recursive** (monoespaçada neutra) | App criativo, code blocks |

Se as DUAS fontes "gritam," o usuário não sabe para onde olhar. Se as DUAS "sussurram," não há hierarquia.

### Regra 5: Combine a Personalidade, Não Só a Forma

> "Se essas fontes fossem roupas, elas pertenceriam ao mesmo evento?"

| Contexto | Personalidade | Combinação |
|---|---|---|
| **Luxo, moda** | Elegante, dramático | **Bodoni + Montserrat Light** |
| **Tech, SaaS** | Limpo, neutro, funcional | **Inter + Source Serif** |
| **Editorial, leitura longa** | Confortável, clássico | **Georgia + Inter** ou **Baskerville + Proxima Nova** |
| **Criativo, ousado** | Único, expressivo | **Fraunces (SOFT: 80) + Inter** |
| **Corporativo, confiável** | Sólido, tradicional | **Times New Roman + Arial** (sim, funciona para relatórios) |

### Regra 6: A Mesma Fundição/Família Resolve Tudo

Se você NÃO quer arriscar, use uma **superfamília** — fontes desenhadas JUNTAS para funcionarem JUNTAS:

| Superfamília | O Que Inclui | Melhor Para |
|---|---|---|
| **Roboto + Roboto Serif + Roboto Mono** | Sans + Serif + Mono | Android, Material Design, sistemas |
| **IBM Plex Sans + Serif + Mono** | Sans + Serif + Mono | Corporativo, dados, dashboards |
| **Source Sans 3 + Source Serif 4 + Source Code Pro** | Sans + Serif + Mono | Open source, Adobe, web |
| **Inter + Inter Display** (v4.0) | Sans UI + Sans Display | Aplicativos, dashboards |
| **DM Sans + DM Serif Display + DM Mono** | Sans + Serif + Mono | Branding, editorial, sites |
| **Noto Sans + Noto Serif** | Sans + Serif (800+ línguas) | Produtos globais, multilíngue |
| **FF Meta + FF Seria** | Sans + Serif | Editorial europeu |
| **Freight Sans + Freight Text** | Sans + Serif | Editorial premium, branding |

**Vantagem de superfamília:** x-height idêntica, DNA compartilhado, zero risco de clash. A desvantagem: menos "surpresa" — pode parecer previsível.

### Regra 7: Teste com Conteúdo REAL

Lorem Ipsum MENTE. A palavra "Project" tem formas diferentes de "Exemplification." Teste seu pairing com:

- Headline real: "We Build Products That People Love"
- Subtítulo real: "Discover how our platform helps teams ship faster"
- Body text real: 3-4 linhas de parágrafo
- Números, @, #, &, % — caracteres "estressantes"

**Teste especialmente em MOBILE.** Um pairing que funciona a 48px em desktop pode desmoronar a 16px num iPhone SE.

### Regra 8: Use a Roda Tipográfica (Como a Roda de Cores)

Assim como existe uma roda de cores, existe uma "roda" de fontes:

```
                    SERIF
                   /      \
            Old Style    Didone
           /                  \
    Humanista                  Slab
         |                      |
    Geométrica              Grotesca
         \                  /
          \                /
           Neo-Grotesca  Humanista Sans
                    |
                  SANS
```

**Fontes OPOSTAS na roda tendem a se complementar.** Fontes ADJACENTES tendem a colidir.

- **OPOSTAS (funciona):** Old Style Serif + Humanista Sans (Garamond + Frutiger)
- **OPOSTAS (funciona):** Didone + Neo-Grotesca (Bodoni + Helvetica)
- **ADJACENTES (cuidado):** Geométrica Sans + Neo-Grotesca (Futura + Helvetica — muito próximas)
- **ADJACENTES (cuidado):** Slab Serif + Grotesca (Rockwell + Franklin Gothic)

### Regra 9: O Par Perfeito Pode Ser a MESMA Fonte

O pairing mais subestimado: **UMA fonte, múltiplos pesos.**

**Inter** sozinha, com Display Black para headlines e Regular para body, é um "pairing" que funciona em 80% dos aplicativos modernos. A Apple usa **SF Pro** com pesos diferentes para TUDO no iOS. O Google usa **Product Sans** para tudo.

Você NÃO PRECISA de duas fontes. Às vezes, a resposta certa é UMA — com 5 pesos.

### Regra 10: Conheça as Regras para Quebrá-las

Estas regras são MAPAS, não LEIS. **David Carson** (Ray Gun magazine) usava fontes que "brigavam" DE PROPÓSITO — para criar tensão visual. **Paula Scher** (Pentagram) empilha 5+ fontes em um pôster e funciona.

A diferença entre "quebrar regras com maestria" e "erro de iniciante" é: **INTENÇÃO.** Se você sabe POR QUE está quebrando a regra, não é um erro — é uma escolha.

---

## 2. A Ciência do Pairing (Nature, 2024)

Um estudo publicado na **Nature — Scientific Reports** (Vol. 14, 2024) analisou **22.000+ usos reais de fontes** em 19 mídias diferentes e identificou **3 dimensões morfológicas fundamentais** que governam o font pairing:

| Dimensão | O Que Mede |
|---|---|
| **1. Serif vs Sans-Serif** | O eixo PRIMÁRIO de contraste |
| **2. Básico vs Decorativo** | Quanto "ornamento" a fonte carrega |
| **3. Leve vs Bold** | Peso visual e densidade |

**A descoberta:** pares do mundo real seguem padrões consistentes e quantificáveis — validando as "regras de polegar" que designers usam intuitivamente há décadas. **Font pairing não é só "gosto" — tem matemática por trás.**

---

## 3. Os Combos Clássicos (Testados Por Décadas)

### 3.1 Garamond + Helvetica — O Casamento Perfeito

| Papel | Fonte | Por Quê |
|---|---|---|
| **Headlines** | Helvetica Bold | Neutra, limpa, impacto |
| **Body** | Garamond (Adobe) | Elegante, quente, legível |

**Por que funciona:** Helvetica é neo-grotesca fria e uniforme. Garamond é Garalde quente com modulação. São opostos em quase TODAS as dimensões — categoria, contraste, história, personalidade. Isso cria contraste MÁXIMO com harmonia.

**Onde usar:** Editorial, branding, sites de luxo, livros.

**Ajuste:** Garamond tem x-height MUITO menor que Helvetica. Compense: Garamond a 17px ≈ Helvetica a 14px em altura percebida.

### 3.2 Baskerville + Futura — O Intelectual e o Modernista

| Papel | Fonte | Por Quê |
|---|---|---|
| **Headlines** | Futura Bold | Geométrica, impactante, Bauhaus |
| **Body** | Baskerville | Transicional, refinada, elegante |

**Por que funciona:** Baskerville é transicional — contraste alto, eixo vertical, serifas com bracket sutil. Futura é geométrica — círculos, zero modulação, apex pontiagudo. O contraste é MÁXIMO em categoria e construção. Mas as x-heights são surpreendentemente próximas — o que cria harmonia invisível.

**Onde usar:** Portfolio de design, moda, lifestyle, capas de livro.

### 3.3 Bodoni + Futura — O Glamour Encontra a Geometria

| Papel | Fonte | Por Quê |
|---|---|---|
| **Headlines** | Bodoni Bold | Didone dramática, contraste extremo |
| **Body** | Futura Book | Geométrica limpa e neutra |

**Por que funciona:** O pairing "Vogue" — Bodoni é a fonte da moda de luxo, Futura é a contraparte moderna. Contraste entre o GLAMOUR do séc. XIX e a GEOMETRIA do séc. XX.

**Onde usar:** Moda, revistas, branding de luxo, perfumes.

### 3.4 Caslon + Franklin Gothic — O Clássico Americano

| Papel | Fonte |
|---|---|
| **Headlines** | Franklin Gothic Bold |
| **Body** | Caslon |

**Por que funciona:** Ambas são "americanas" — Caslon (inglesa, mas adotada pelos EUA — a fonte da Declaração de Independência) e Franklin Gothic (Morris Fuller Benton, ATF, 1902). Compartilham DNA histórico mas são categorias diferentes.

### 3.5 Times New Roman + Helvetica — O "Padrão" Que Funciona

| Papel | Fonte |
|---|---|
| **Headlines** | Helvetica Bold |
| **Body** | Times New Roman |

**Por que funciona:** Times é transicional — desenhada para jornal (The Times, 1932), compacta, econômica. Helvetica é neo-grotesca. É o pairing "default" do mundo corporativo. Não é CRIATIVO — mas é FUNCIONAL. Funciona em relatórios, documentos, sistemas.

---

## 4. Os Combos Modernos (2024-2025)

### 4.1 Inter + Fraunces — O Combo do Momento

| Papel | Fonte | Peso |
|---|---|---|
| **Headlines** | Fraunces | `wght: 600-900, SOFT: 60-80` |
| **Body** | Inter | `wght: 400-450` |

**Por que funciona:** Fraunces é uma serif VARIÁVEL com eixo `SOFT` (0-100%) que vai de nítida a arredondada. Em `SOFT: 70`, ela tem uma personalidade editorial "vintage-moderna" que contrasta LINDO com a neutralidade digital da Inter.

**Onde usar:** Branding, landing pages, sites de produto, editorial digital.

### 4.2 Playfair Display + Lato — O Elegante Acessível

| Papel | Fonte |
|---|---|
| **Headlines** | Playfair Display Bold |
| **Body** | Lato Regular |

**Por que funciona:** Playfair é Didone (contraste extremo, glamour). Lato é humanista sans (quente, acessível). O pairing é "luxo democrático" — elegante mas não elitista.

**Onde usar:** Sites institucionais, portfolios, blogs de moda/lifestyle.

### 4.3 Inter + Source Serif — O Editorial Digital

| Papel | Fonte |
|---|---|
| **Headlines** | Source Serif 4 Bold |
| **Body** | Inter Regular |

**Por que funciona:** Source Serif é a parceira serif da Source Sans (Adobe). Inter é a neo-grotesca digital padrão. Ambas são open-source. Ambas têm x-heights próximas. Ambas são "de trabalho" — fontes para LER, não para EXIBIR.

**Onde usar:** Blogs, documentação, newsletters, sites de conteúdo.

### 4.4 DM Serif Display + DM Sans — A Superfamília "Cool"

| Papel | Fonte |
|---|---|
| **Headlines** | DM Serif Display |
| **Body** | DM Sans |

**Por que funciona:** MESMA família. MESMO DNA. DM Serif Display é uma transicional elegante. DM Sans é uma geométrica limpa. Elas foram DESENHADAS para funcionar juntas. Zero risco.

**Onde usar:** Sites, branding, editorial, landing pages.

### 4.5 Montserrat + Merriweather — O Robusto Acolhedor

| Papel | Fonte |
|---|---|
| **Headlines** | Montserrat Bold |
| **Body** | Merriweather Regular |

**Por que funciona:** Montserrat é geométrica (Futura-like, mas mais amigável). Merriweather é transicional desenhada para TELA. Ambas são "robustas" — strokes grossos, boa legibilidade. Funcionam bem em sites de conteúdo pesado.

### 4.6 Bricolage Grotesque + Inter — O Quirky Corporativo

| Papel | Fonte |
|---|---|
| **Headlines** | Bricolage Grotesque Bold |
| **Body** | Inter Regular |

**Por que funciona:** Bricolage Grotesque é uma grotesca contemporânea com personalidade — "excêntrica mas profissional." Inter é a workhorse neutra. Bricolage injeta CARÁTER onde Inter sozinha seria "genérica demais."

---

## 5. Superfamílias: O Caminho Sem Erro

Se você NÃO quer pensar em pairing, use uma superfamília. Elas incluem sans + serif + mono desenhadas JUNTAS.

### 5.1 IBM Plex — A Superfamília Corporativa Definitiva

**IBM Plex** (Mike Abbink, 2017) é a superfamília mais COMPLETA do mercado:

| Variante | Uso |
|---|---|
| **IBM Plex Sans** | Body text, UI, labels |
| **IBM Plex Serif** | Headlines, editorial, branding |
| **IBM Plex Mono** | Código, dados, tabelas |
| **IBM Plex Condensed** | Espaços apertados, dashboards |
| **IBM Plex Arabic, Devanagari, Thai, CJK...** | Cobertura global |

**DNA compartilhado:** Todas as variantes têm o mesmo "esqueleto" — x-height idêntica, proporções consistentes, mesma filosofia de DESIGN. Você pode trocar entre sans, serif e mono e o "ritmo" permanece.

### 5.2 Source — A Superfamília Open-Source da Adobe

| Variante | Uso |
|---|---|
| **Source Sans 3** | Body UI, web |
| **Source Serif 4** | Editorial, headings |
| **Source Code Pro** | Código |

A Source Serif 4 é variável com 5 tamanhos ópticos discretos (8, 12, 18, 36, 72).

### 5.3 Roboto + Roboto Serif + Roboto Mono — Android

| Variante | Uso |
|---|---|
| **Roboto Flex** (variável) | UI Android, apps |
| **Roboto Serif** | Editorial, headings |
| **Roboto Mono** | Código |

A família Roboto é o "Material Design" em forma de fonte.

### 5.4 Noto — A Superfamília GLOBAL

**Noto** (Google, 2012+) é a tentativa de cobrir TODAS as línguas do mundo com fontes harmonicamente compatíveis. **800+ línguas, 100.000+ caracteres.** Noto Sans + Noto Serif são as workhorses; as variantes cobrem do cirílico ao tamil.

**Se seu produto é multilíngue, Noto é a ÚNICA resposta.**

---

## 6. Anti-Pairings: Combinações Que Você NUNCA Deve Fazer

### 6.1 Duas da Mesma Categoria

| Erro | Por Que Falha |
|---|---|
| **Helvetica + Arial** | Indistinguíveis para 99% das pessoas. Parece ERRO, não escolha. |
| **Futura + Montserrat** | Duas geométricas. Uma delas "sobra." |
| **Raleway + Montserrat** | Duas geométricas com peso similar — flat, sem contraste. |
| **Playfair Display + Libre Baskerville** | Duas serif de alto contraste — competem em vez de complementar. |

### 6.2 Personalidades Conflitantes

| Erro | Por Que Falha |
|---|---|
| **Roboto + Playball** | Utilitária vs. Script romântica. Sem DNA compartilhado. |
| **Archivo Black + Cardo** | Bold moderno vs. acadêmico clássico — crise de identidade. |
| **Alfa Slab One + Lato** | Uma GRITA, a outra sussurra. Desbalanceamento total. |

### 6.3 Ritmo Incompatível

| Erro | Por Que Falha |
|---|---|
| **Futura + Times New Roman** | Diferença de largura EXTREMA. Futura é wide, Times é narrow. Discordantes. |
| **Playfair Display + Quicksand** | Serif dramática com sans "fofa" — o tom não casa. |

---

## 7. O Método de 5 Minutos Para Escolher Um Par

1. **Defina o tom** (1min): Luxo? Tech? Editorial? Criativo?
2. **Escolha a ESTRELA** (1min): A fonte que vai nos headlines/hero. Pode ser expressiva.
3. **Escolha o SUPORTE** (2min): A fonte do body. DEVE ser legível a 14-16px. Priorize legibilidade sobre personalidade.
4. **Teste o 'x'** (30s): Bote `xxXX` nas duas lado a lado. Mesmo tamanho. As x-heights são próximas?
5. **Teste com conteúdo real** (30s): Headline + subtítulo + 3 linhas de parágrafo. Funciona em mobile?

**Se depois de 5 minutos você não está CONFIANTE, escolha uma superfamília e vá ser feliz.**

---

## 8. Tabela-RESUMO: 30 Combos Testados

| # | Headlines (Estrela) | Body (Suporte) | Estilo | Melhor Para |
|---|---|---|---|---|
| 1 | **Playfair Display** | **Lato** | Didone + Humanista | Editorial, moda, luxo |
| 2 | **Fraunces** | **Inter** | Serif Variável + Neo-Grotesca | Branding, landing pages |
| 3 | **DM Serif Display** | **DM Sans** | Transicional + Geométrica (superfamília) | Sites, branding |
| 4 | **Garamond** | **Helvetica** | Garalde + Neo-Grotesca | Editorial, livros |
| 5 | **Baskerville** | **Futura** | Transicional + Geométrica | Portfolio, capas |
| 6 | **Bodoni** | **Montserrat Light** | Didone + Geométrica | Moda, luxo |
| 7 | **Montserrat Bold** | **Merriweather** | Geométrica + Transicional | Sites de conteúdo |
| 8 | **Inter Display** | **Source Serif 4** | Neo-Grotesca + Transicional | Blogs, docs |
| 9 | **Bebas Neue** | **Inter** | Display Condensada + Neo-Grotesca | Hero sections |
| 10 | **Bricolage Grotesque** | **Inter** | Grotesca Quirky + Neo-Grotesca | Apps com personalidade |
| 11 | **Recursive (CASL:1)** | **Recursive (CASL:0, MONO:0)** | Sans Criativa + Sans Neutra (superfamília) | UI criativa |
| 12 | **IBM Plex Serif** | **IBM Plex Sans** | Transicional + Neo-Grotesca (superfamília) | Corporativo |
| 13 | **Roboto Serif** | **Roboto Flex** | Slab + Neo-Grotesca (superfamília) | Android, apps |
| 14 | **Source Serif 4** | **Source Sans 3** | Transicional + Humanista (superfamília) | Open source, web |
| 15 | **Space Grotesk** | **Inter** | Geométrica Contemporânea + Neo-Grotesca | SaaS, startups |
| 16 | **Georgia** | **Inter** | Transicional Screen + Neo-Grotesca | Editorial web |
| 17 | **Merriweather** | **Open Sans** | Transicional + Humanista | Blogs, conteúdo |
| 18 | **Lora** | **Montserrat** | Humanista Serif + Geométrica | Sites acadêmicos, arte |
| 19 | **Newsreader** | **Inter** | Serif Editorial + Neo-Grotesca | Newsletters |
| 20 | **Crimson Pro** | **Lato** | Garalde + Humanista | Leitura longa |
| 21 | **Oswald** | **Noto Sans** | Display Condensada + Humanista Global | Multilíngue |
| 22 | **EB Garamond** | **Fira Sans** | Garalde + Humanista | Editorial open source |
| 23 | **Poppins** | **Source Sans 3** | Geométrica + Humanista | Moderno, minimal |
| 24 | **Abril Fatface** | **Inter Light** | Didone Display + Neo-Grotesca | Hero luxo |
| 25 | **Manrope** | **Inter** | Geométrica Moderna + Neo-Grotesca | Fintech, dashboards |
| 26 | **Lexend** | **Inter** | Legibilidade + Neo-Grotesca | Educação, acessibilidade |
| 27 | **Atkinson Hyperlegible** | **Inter** | Acessibilidade + Neo-Grotesca | Acessibilidade máxima |
| 28 | **Freight Text** | **Freight Sans** | Garalde + Humanista (superfamília) | Editorial premium |
| 29 | **Noto Serif** | **Noto Sans** | Transicional + Humanista (superfamília) | Produtos globais |
| 30 | **SF Pro Display** | **SF Pro Text** | Neo-Grotesca + Neo-Grotesca (mesma família) | iOS nativo |

---

## 9. O Pairing É Um RITMO, Não Uma FÓRMULA

Font pairing é como compor música. Você não quer duas melodias IDÊNTICAS (monótono) nem duas melodias COMPLETAMENTE DIFERENTES (ruído). Você quer duas vozes que CANTAM JUNTAS — diferentes o suficiente para serem distintas, similares o suficiente para estarem na mesma TONALIDADE.

**O melhor pairing é aquele que o usuário NÃO PERCEBE.** Quando funciona, o leitor só pensa no CONTEÚDO — não nas fontes. A tipografia desaparece. Só a mensagem fica.

E quando alguém PERGUNTA "que fontes são essas?", você sabe que acertou.

---

## Fontes Citadas Neste Loop

| Fonte | Designer/Ano | Papel Típico |
|---|---|---|
| **Inter** | Rasmus Andersson, 2017 | Body UI, suporte neutro |
| **Fraunces** | Undercase Type, 2020 | Headlines editorial com SOFT |
| **Playfair Display** | Claus Eggers Sørensen, 2012 | Headlines Didone |
| **Lato** | Łukasz Dziedzic, 2010 | Body humanista quente |
| **DM Serif Display + DM Sans** | Colophon, 2019 | Superfamília |
| **Garamond (Adobe)** | Robert Slimbach, 1989 | Body editorial clássico |
| **Helvetica** | Max Miedinger, 1957 | Headlines neutro, suporte |
| **Baskerville** | John Baskerville, 1757 | Headlines transicional |
| **Futura** | Paul Renner, 1927 | Headlines geométrico |
| **Bodoni** | Giambattista Bodoni, 1798 | Headlines Didone luxo |
| **Montserrat** | Julieta Ulanovsky, 2011 | Headlines geométrico |
| **Merriweather** | Eben Sorkin, 2010 | Body transicional screen |
| **Source Serif 4** | Adobe, 2021 | Headlines editorial |
| **Source Sans 3** | Adobe, 2021 | Body UI |
| **IBM Plex Sans + Serif** | Mike Abbink, 2017 | Superfamília corporativa |
| **Roboto Flex + Roboto Serif** | Google, 2011-2021 | Superfamília Android |
| **Noto Sans + Noto Serif** | Google, 2012+ | Superfamília global |
| **Bricolage Grotesque** | Mathieu Triay, 2022 | Headlines quirky |
| **Bebas Neue** | Ryoichi Tsunekawa, 2010 | Display condensado |
| **Space Grotesk** | Florian Karsten, 2018 | Headlines geométrico contemporâneo |
| **Recursive** | Arrow Type, 2019 | Superfamília variável criativa |
| **Georgia** | Matthew Carter, 1993 | Body serif screen |
| **Lora** | Cyreal, 2011 | Body humanista serif |
| **Oswald** | Vernon Adams, 2011 | Display condensado |
| **Manrope** | Mikhail Sharanda, 2018 | Headlines fintech |
| **Lexend** | Bonnie Shaver-Troup, 2019 | Body acessibilidade |
| **Atkinson Hyperlegible** | Braille Institute, 2020 | Body acessibilidade máxima |
| **Freight Sans + Text** | Joshua Darden, 2005 | Superfamília editorial premium |
| **SF Pro** | Apple, 2014 | Body/Display UI nativa |

---

> **Próximo loop sugerido:** Serif a fundo — Humanista, Garalde, Transicional, Didone, Slab. A história, os detalhes, e por que a serif NÃO morreu (e nunca vai morrer). Ou: **Fontes para Apps Mobile** — San Francisco, Inter, Roboto, Product Sans. Como as big techs escolhem e customizam suas fontes.
