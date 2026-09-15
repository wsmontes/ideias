# Ralph+Lisa: Estudos de Caso de Apps

## Referência de qualidade

O benchmark é `loop-01-gmail.md` (~853 linhas, 53KB, 15 seções). Leia-o integralmente antes de começar QUALQUER trabalho. Ele define o padrão de completude, densidade técnica e profundidade analítica que TODOS os documentos devem atingir.

## Fluxo de trabalho (UM documento por ciclo Ralph+Lisa)

1. Leia o arquivo atual do loop APENAS para saber qual app é
2. Execute 4 CICLOS MÍNIMOS OBRIGATÓRIOS de pesquisa. Cada ciclo deve produzir resultados VISÍVEIS (links, dados, citações) antes de avançar. NÃO escreva o documento antes de completar os 4 ciclos
3. Só então reescreva o documento
4. Ao terminar, output <promise>CHECKLIST_PASSED</promise> — Lisa validará cada requisito

## Os 4 ciclos de pesquisa (OBRIGATÓRIOS, SEQUENCIAIS)

**REGRA FUNDAMENTAL**: Cada ciclo deve ALIMENTAR o próximo. Buscas paralelas no mesmo ciclo são permitidas, mas contam como UM ciclo — mesmo que você tenha feito 10 buscas simultâneas, aquilo é o Ciclo 1. Após analisar os resultados, você usa o que descobriu para formular as buscas do Ciclo 2, e assim por diante. Isso garante profundidade progressiva. NÃO adianta fazer 4 buscas simultâneas e chamar de 4 ciclos.

### Ciclo 1 — Mapeamento
2-3 buscas genéricas para mapear o terreno técnico do app: engineering blog, stack, arquitetura, decisões conhecidas

### Ciclo 2 — Aprofundamento
Buscas específicas nos 3-4 threads mais promissores. Encontrar URLs de engineering blogs, white papers, talks técnicas

### Ciclo 3 — Extração
WebFetch dos 2-3 posts originais mais relevantes para extrair detalhes técnicos completos

### Ciclo 4 — Preenchimento de lacunas
Buscar palavras-chave descobertas nos ciclos anteriores. Preencher TODO o resto. Nada superficial

### Ciclos 5-6 (recomendados)
WebFetch adicional para tópicos que ainda ficaram superficiais

## Seções obrigatórias (nesta ordem)

0. **Linhagem** — ASCII tree mostrando a categoria antes e depois do app
1. **Origem** — fundadores, contexto histórico, primeiras versões, dados internos da empresa
2. **Filosofia do Produto** — princípios que definem as decisões do app
3. **Arquitetura Técnica** — a seção MAIS EXTENSA. Stack completa, protocolos, data models, infraestrutura, decisões de engenharia, gambiarras e genialidades
4. **Inovações** — o que o app trouxe que não existia antes, com detalhes de implementação
5. **Linha do Tempo do Design Visual** — evolução visual ano a ano
6. **UX Mobile e Ergonomia** — anatomia da interface, gestos, componentes, one-handed usability
7. **Paleta de Cores e Identidade Visual** — cores oficiais com HEX, tipografia, Design System
8. **Público-Alvo e Mercado** — demografia, posicionamento, concorrência
9. **Críticas** — problemas, pontos de dor, limitações técnicas e de UX
10. **Lições** — insights TÉCNICOS não-óbvios para builders de apps
11. **Ficha Técnica** — tabela com todos os atributos do app
12. **Linha do Tempo** — cronologia completa ano a ano
13. **Fontes** — todas as URLs consultadas

## Viés TÉCNICO (OBRIGATÓRIO)

- Foco ABSOLUTO em engenharia: arquitetura, protocolos, algoritmos, infraestrutura, decisões técnicas
- IGNORE artigos de business, VC, funding, valuation, IPO, "história de sucesso"
- Se mencionar receita ou valuation, limite a UMA linha na Ficha Técnica
- Conhecimento interno das empresas é o ouro: engineering blogs, talks, papers, post-mortems, RFCs

## Estilo (OBRIGATÓRIO)

- Tom ANALÍTICO, não motivacional
- Densidade de informação alta — cada parágrafo contém informação substantiva
- Alvo: 500-850 linhas
- PROIBIDO: "O App Que [verbo]", emojis como bullet points, parágrafos de 1 frase, CAIXA ALTA para ênfase
- As lições devem ser insights TÉCNICOS não-óbvios

## Sequência de processamento

```
loop-03-whatsapp.md → loop-04-uber.md → loop-05-airbnb.md →
loop-06-instagram.md → loop-07-notion.md → loop-08-tiktok.md → loop-09-google-maps.md →
loop-10-tinder.md → loop-11-duolingo.md → loop-12-strava.md → loop-13-netflix.md →
loop-14-youtube.md → loop-15-nubank.md → loop-16-telegram.md → loop-17-slack.md →
loop-18-figma.md → loop-19-discord.md → loop-20-canva.md → loop-21-wechat.md →
loop-22-stripe.md → loop-23-doordash.md → loop-24-roblox.md → loop-25-reddit.md →
loop-26-twitch.md → loop-27-snapchat.md → loop-28-zoom.md → loop-29-paypal.md →
loop-30-pokemon-go.md → loop-31-line.md → loop-32-kakaotalk.md → loop-33-revolut.md →
loop-34-calm.md → loop-35-capcut.md → loop-36-ifood.md → loop-37-google-photos.md →
loop-38-grab.md → loop-39-shopify.md → loop-40-1password.md → loop-41-linear.md →
loop-42-wise.md → loop-43-gojek.md → loop-44-procreate.md → loop-45-todoist.md →
loop-46-superhuman.md → loop-47-dark-sky.md → loop-48-candy-crush.md → loop-49-peloton.md →
loop-50-apple-health.md → loop-51-fortnite.md → loop-52-chatgpt.md → loop-53-venmo.md →
loop-54-kindle.md → loop-55-tripadvisor.md → loop-56-dropbox.md → loop-57-substack.md →
loop-58-yelp.md → loop-59-zillow.md → loop-60-waze.md → loop-61-patreon.md →
loop-62-evernote.md → loop-63-bereal.md
```

Processe UM documento por vez, em ordem. Não pule. Output <promise>CHECKLIST_PASSED</promise> ao final de cada documento.

## Critério de conclusão

Quando loop-63-bereal.md for reescrito com o mesmo padrão de qualidade do loop-01-gmail.md, o ciclo termina.

## Comece agora

**Loop atual: loop-03-whatsapp.md.** Loop-01 (Gmail) é o benchmark. Loop-02 (Spotify) já foi refeito. Execute 4 ciclos de pesquisa. Reescreva.
