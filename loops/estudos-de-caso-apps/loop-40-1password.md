# Estudo de Caso 40 — 1Password: O App Que Começou Como "Projeto de 1 Mês" (E Virou o Cofre Digital de $6.8B)

> **Data:** 2026-07-03
> **Loop:** 40 de ∞ (Fase 2: Segurança & Utilidades)
> **Categoria:** Segurança / Senhas / Autenticação
> **Tema:** 2005. Dave Teare e Roustem Karimov — dois desenvolvedores web canadenses — estão CANSADOS de digitar senhas o dia TODO. Trabalham com e-commerce. Cada teste de formulário exige login. Cada login exige senha. Eles constroem uma ferramenta interna: auto-preenchimento de formulários. Chamam de **"OS X Form."** Era para ser um "projeto de UM MÊS." Mas adicionaram salvamento de senhas ("uma senha é só mais um campo de formulário"). Renomearam para **"1Passwd."** Em 19 de maio de 2006, subiram no MacUpdate. Na PRIMEIRA HORA, alguém COMPROU. Receita do primeiro ano: $80.000. Largaram os empregos. Viveram de miojo. **Zero investimento por 14 ANOS.** Bootstrapped. Lucrativos desde o ANO 1. Em 2019, primeirO investimento externo: $200M da Accel. Valuation: $2B. Em 2025: $6.8B. 4M+ passkeys armazenados. 15M+ autofills de passkey. Esta é a história do app que transformou "123456" em "nunca mais preciso lembrar uma senha" — e que agora quer MATAR a senha de vez com passkeys.

---

## 1. A Origem: "Só Mais Um Mês" (Por 20 Anos)

### Dave Teare e Roustem Karimov

| Fundador | Background |
|---|---|
| **Dave Teare** | Canadense. Desenvolvedor web. Ex-World Vision Canada. |
| **Roustem Karimov** | Desenvolvedor. Ex-World Vision Canada. |
| **Sara Teare e Natalia Karimov** | Esposas. Co-fundadoras. |

Os dois trabalhavam como consultores web. Construíam sites de e-commerce. Testar formulários o dia TODO significava digitar senhas O DIA TODO.

### "OS X Form" → "1Passwd" → "1Password"

- **2005**: ferramenta interna. Auto-preenchimento de formulários. "Projeto de 1 MÊS."
- Adicionaram salvamento de senhas: "uma senha é só mais um campo de formulário."
- **Maio de 2006**: upload no MacUpdate. Na PRIMEIRA HORA, alguém COMPROU.
- **Ano 1**: $80.000 de receita. Largaram os empregos.
- **14 ANOS sem investimento externo.** Bootstrapped. Lucrativos.
- **2019**: primeirO investimento: $200M da Accel. $2B valuation.
- **2025**: $6.8B valuation.

---

## 2. A Filosofia: "Segurança Só Funciona Se As Pessoas USAM"

### O Princípio Fundamental

> *"We're ultimately a security tool, but it's always been about ease of use and convenience."* — Anna Pobletts, Head of Passwordless

### Pilares de Design

| Pilar | Significado |
|---|---|
| **Simplicity over complexity** | Biometria = "tão simples quanto tocar o sensor de digital." |
| **Consistency & familiarity** | Nome "passkey" + ícone padrão em TODA parte. |
| **Portability** | Usuário NUNCA fica preso a um ecossistema. |
| **Defense in depth** | AES-GCM-256. Secret Key + Account Password. Dual-key encryption. |

### "Passwordless ≠ Password-Free" (2025)

O 1Password NÃO acredita que senhas vão DESAPARECER. "Passwordless não significa password-free." A transição é GRADUAL:
1. Armazene TUDO no cofre.
2. Descubra senhas fracas/reutilizadas (Watchtower).
3. Migre para passkeys ONDE DER.
4. Mantenha senhas para sistemas LEGADOS.

---

## 3. As Inovações do 1Password

### 3.1 Secret Key + Account Password (Autenticação Dual)

Sua conta é protegida por DOIS segredos: a **Secret Key** (128-bit, gerada no dispositivo, NUNCA enviada ao servidor) + sua **Account Password.** Mesmo que o servidor do 1Password seja hackeado, seus dados estão SEGUROS — porque a Secret Key NUNCA saiu do seu dispositivo.

### 3.2 Watchtower: O Radar de Senhas Vazadas

Monitora haveibeenpwned e outras fontes. "Sua senha do Netflix apareceu num vazamento. Troque."

### 3.3 Passkeys (2023-2025): A Aposta No Futuro Sem Senha

- **4M+ passkeys armazenados.** 15M+ autofills.
- **~30% dos usuários** já usam passkeys.
- **PRF (Pseudo-Random Function)**: inovação técnica que permite passkeys serem usadas para ENCRIPTAÇÃO (antes, passkey só servia para AUTENTICAÇÃO).

### 3.4 Travel Mode: "Apague Tudo Ao Viajar"

Ative o Travel Mode. Seus cofres SOMEM do dispositivo. Só Voltam quando você DESATIVAR. Para cruzar fronteiras com SEGURANÇA.

---

## 4. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | 1Password |
| **Fundação** | 2005. Lançamento: 19 de maio de 2006. |
| **Fundadores** | Dave Teare, Roustem Karimov, Sara Teare, Natalia Karimov |
| **Valuation** | $6.8 bilhões (2025) |
| **Passkeys armazenados** | 4M+ |
| **Preço** | Individual: $2.99/mês. Family: $4.99. Business: $7.99. |
| **Concorrentes** | LastPass, Bitwarden, Dashlane, Apple Passwords, Google Password Manager |

---

## 5. Lições do 1Password

### 5.1 "Projeto de 1 Mês" Há 20 Anos

Dave e Roustem diziam: "só mais um mês." 20 anos depois, ainda estão nisso. O "projeto interno" virou $6.8B.

**Lição**: as melhores empresas frequentemente começam como FERRAMENTAS INTERNAS. O que VOCÊ precisa pode ser o que o MUNDO precisa.

### 5.2 "Zero Investimento Por 14 Anos" — Bootstrapping FUNCIONA

1Password foi LUCRATIVO desde o ano 1. Só pegou investimento em 2019 — 14 ANOS depois. Valuation saltou de $0 para $6.8B.

**Lição**: você NÃO PRECISA de VC para construir um negócio BILIONÁRIO. LUCRO desde o dia 1 é melhor que valuation.

### 5.3 Segurança Sem Usabilidade É SEGURANÇA QUE NINGUÉM USA

O 1Password NÃO É "o cofre mais seguro do mundo." É "o cofre que as pessoas REALMENTE USAM." Porque a UX é TÃO BOA que ninguém TEM DESCULPA para não usar.

**Lição**: o produto de segurança mais SEGURO do mundo é INÚTIL se ninguém USA. Segurança = tecnologia + ADOÇÃO.

---

## Fontes e Referências

- [Lifehacker — Behind the App: The Story of 1Password (2014)](https://lifehacker.com/behind-the-app-the-story-of-1password-1643425238)
- [Crunchbase — 1Password Raises $200M Series A (2019)](https://news.crunchbase.com/venture/in-its-first-funding-in-14-years-1password-raises-200m-series-a-led-by-accel/)
- [1Password Security Design White Paper (2025)](https://agilebits.github.io/security-design/index.html)
- [1Password — Sierre Wolfkostin on passkey UX (2024)](https://1password.com/blog/sierre-wolfkostin-passkeys-interview)
- [1Password — World Password Day 2025: why passwordless ≠ password-free](https://1password.com/blog/why-the-future-is-passwordless-but-not-password-free)
- [Android Police — 1Password passkey integration balloons to millions (2024)](https://www.androidpolice.com/1password-passkeys-fast-growth-from-september-2023/)
