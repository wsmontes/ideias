# Estudo de Caso 40 — 1Password: A Arquitetura Zero-Knowledge Com 2SKD (PBKDF2 650K + HKDF + XOR), SRPv6, AES-256-GCM, Rust Core + WASM, Watchtower k-Anonymity e Passkeys/Recovery Codes

> **Data:** 2026-07-03
> **Loop:** 40 de ∞ (Reescrita)
> **Categoria:** Segurança / Criptografia / Local-First

---

## 0. Linhagem

```
Senhas em post-it — o estado pré-digital.
Keychain (Apple, 1999), KeePass (2003) — local. Sem sync. Sem zero-knowledge.
LastPass (2008) — cloud. Breach em 2022.
1Password (2006) — AgileBits. 2SKD. SRP. Zero-knowledge. 20 anos sem breach.
1Password hoje (2026) — US$ 6,8B. Rust core + WASM. Passkeys. Watchtower.
```

O 1Password não é um gerenciador de senhas — é uma **arquitetura de segurança zero-knowledge**: nem a AgileBits, nem um atacante que comprometa os servidores, pode acessar os dados dos usuários. Essa propriedade é garantida por design criptográfico, não por política de privacidade.

---

## 1. Arquitetura Técnica

### 1.1 O Modelo de Segurança: 2SKD, SRPv6, AES-256-GCM

O **Two-Secret Key Derivation (2SKD)** é o coração criptográfico do 1Password. Dois segredos independentes — nenhum suficiente sozinho — são combinados para derivar todas as chaves:

**Account Password** (o que o usuário sabe). Passa por `normalize()` e `trim()`, depois **650.000 rounds de PBKDF2-SHA256** com salt preparado via HKDF. O fator de trabalho torna bruteforce proibitivamente caro.

**Secret Key** (128 bits de entropia, ~2¹²⁸ possibilidades, 26 caracteres). Gerada localmente no dispositivo durante sign-up. Nunca sai do dispositivo. Nunca conhecida pelo servidor. Armazenada no Emergency Kit para recuperação.

**Processo de derivação (8 passos, Section 8.2.1 do White Paper v0.5.1):**

1. `p ←` Account Password do usuário
2. `(k_A, e, I, s) ←` Secret Key, email, ID, salt do storage local
3. `p ← trim(p)` → `p ← normalize(p)`
4. `s ← HKDF(s, version, e, 32)` — preparação do salt
5. `k_m ← PBKDF2(p, s, 650.000)` — slow hashing
6. `k_A ← HKDF(k_A, version, I, |k_m|)` — expansão da Secret Key
7. `k_m ← k_m ⊕ k_A` — **XOR** do material derivado da password com o material derivado da Secret Key
8. `k_m ← JWKify(k_m)` — formatação como JSON Web Key

**Propriedade de segurança crucial:** um atacante que rouba dados do servidor não pode sequer **tentar** adivinhar a password sem a Secret Key. Tentativas com password correta e password incorreta falham de forma idêntica sem ela — indistinguibilidade perfeita.

**SRPv6 (Secure Remote Password).** Zero-knowledge proof para autenticação: o servidor nunca aprende a password ou a Secret Key. Armazena apenas um **verifier** `v` — matematicamente inútil para recuperar as credenciais. Mútuo authentication + session key agreement.

**AES-256-GCM.** Criptografia autenticada para todos os vault data. End-to-end: servidores 1Password não podem decriptar.

### 1.2 Rust Core + WASM: Arquitetura Local-First Multi-Plataforma

**Rust core:** engine compartilhado contendo data layer, networking, sincronização, resolução de conflitos e toda a criptografia. Compilado para binário nativo (desktop/mobile) e para **WebAssembly (WASM)** no web client. Garante implementação criptográfica memory-safe e consistente entre plataformas.

**Thin clients:** UIs específicas por plataforma — TypeScript + React (Electron desktop), Swift (iOS), Kotlin (Android). Comunicação com Rust core via FFI: UI envia invocations; core processa e retorna resultados via callbacks.

**Sync local-first:** pull-first, push-second. Cliente puxa último estado do servidor, resolve conflitos localmente (servidor não pode — dados são criptografados). Notifier microservice: WebSocket push para todos os clientes conectados.

### 1.3 Watchtower: k-Anonymity e Privacy-Preserving Breach Detection

Watchtower verifica passwords contra bases de breaches conhecidas (Have I Been Pwned) usando **k-anonymity**: apenas um **prefixo parcial do hash SHA-1** (5 caracteres) é enviado ao servidor. O servidor retorna todos os sufixos de hash que correspondem àquele prefixo; o cliente verifica localmente se o hash completo existe na lista. O servidor nunca aprende a password real nem o hash completo.

**7 categorias de risco monitoradas:** compromised passwords, weak, reused, unsecured HTTP, missing 2FA, expiring items, passkeys available. Database de breaches baixada localmente — todas as comparações são client-side.

### 1.4 Passkeys e Recovery Codes (2024-2025)

**Passkeys** como método de autenticação alternativo ao 2SKD tradicional. **Recovery Codes**: chaves de 256 bits que fornecem acesso quando credenciais são perdidas. Usam split similar ao 2SKD: cryptographic step (recovery key) + identity verification step (email verification). Recovery keys geram três subkeys via HKDF: authentication, encryption e identifier.

**Windows 11 system-level passkey provider** (novembro 2025): 1Password registrado como provider do sistema. **CXP/CXF standards** (FIDO Alliance): 1Password co-iniciou o Credential Exchange Protocol para portabilidade de passkeys entre providers.

---

## 2. Lições de Engenharia

### 2.1 Zero-knowledge é propriedade matemática, não promessa

2SKD + SRPv6 + AES-256-GCM + client-side encryption garantem que nem a AgileBits, nem atacante com acesso total aos servidores, pode ler vaults. Security by architecture, not by policy.

### 2.2 Rust core via WASM é o padrão para apps multi-plataforma com segurança crítica

Um codebase Rust para toda lógica sensível + thin clients nativos. WASM permite até o web client usar o mesmo core. Elimina bugs de criptografia por implementação divergente.

### 2.3 k-anonymity permite cloud-backed breach intelligence sem sacrificar privacidade

5 caracteres de hash SHA-1 não revelam nada sobre a password. O design prova que inteligência de segurança cloud-backed é compatível com zero-knowledge.

---

## 3. Ficha Técnica

| Atributo | Valor |
|---|---|
| **Nome** | 1Password (AgileBits) |
| **Fundação** | 2006. Valuation: US$ 6,8B (2025) |
| **Categoria** | Segurança / Password Manager / Criptografia |
| **Segurança** | 2SKD (PBKDF2 650K + HKDF + XOR), SRPv6, AES-256-GCM, zero-knowledge, 20 anos sem breach |
| **Core** | Rust → nativo (desktop/mobile) + WASM (web). FFI com TypeScript/Swift/Kotlin |
| **Sync** | Local-first: pull-first, push-second. Notifier WebSocket |
| **Watchtower** | k-anonymity (SHA-1 5-char prefix), Have I Been Pwned, 7 risk categories |
| **Passkeys** | Recovery Codes (256-bit), Windows 11 system provider, CXP/CXF standards |
| **Concorrentes** | Bitwarden, LastPass, Dashlane, Apple Passwords |

---

## Fontes

- [1Password Security Design White Paper v0.5.1 (Fev 2025, agilebits.github.io): 2SKD, SRPv6, 8-step derivation, AES-256-GCM](https://agilebits.github.io/security-design/)
- [Syntax.fm #776 — How 1Password Uses WASM and Rust for Local First Dev (Andrew Burkhart, Mai 2024)](https://syntax.fm/show/776/how-1password-uses-wasm-and-rust-for-local-first-dev-with-andrew-burkhart)
- [1Password Support — About the security of unlocking 1Password with a passkey (Recovery Codes, 256-bit)](https://support.1password.com/passkey-security/)
- [Crypto StackExchange — What Makes a 2SKD Secure?](https://crypto.stackexchange.com/questions/114029/what-makes-a-2skd-two-secret-key-derivation-secure)
