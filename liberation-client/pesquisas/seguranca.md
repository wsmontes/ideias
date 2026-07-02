# Pesquisa: Segurança em Sistemas Descentralizados (2025)

> Notas para Iteração 3 — Identity/Keychain + Threat Model
> Data: 2026-07-01

---

## Nostr: Primeira Avaliação de Segurança (Black Hat 2025)

Pesquisadores de NICT, Osaka University, NEC e University of Hyogo conduziram a primeira avaliação abrangente de segurança do Nostr.

### 5 Ataques Descobertos

| Ataque | Impacto | Causa Raiz |
|--------|---------|-----------|
| **Public Key Substitution** | Personificar qualquer usuário | Sem mecanismo de verificação de autenticidade de pubkey (falha de especificação) |
| **Signature Verification Flaws** | Alterar posts/perfis arbitrariamente | Validação de assinatura ausente em múltiplos clientes |
| **Encrypted DM Forgery** | Forjar qualquer DM criptografado | Reuso de chave entre subprotocolos (NIP-01, NIP-04, NIP-46) |
| **DM Plaintext Recovery** | Ler DMs confidenciais sem interação | CBC malleability + link preview automático no receptor |
| **Key Replace Attack** | Atacante substitui pubkey da vítima | Protocolo não vincula identidade à chave |

**8 cenários de ataque.** Patches sendo deployados progressivamente.

**Paper:** [Not in The Prophecies: Practical Attacks on Nostr](https://eprint.iacr.org/2025/1459) — IEEE EuroS&P 2025

---

## Matrix: Hierarquia de Chaves Cross-Signing

### Arquitetura de 3 Camadas
```
Master Key (msk, mpk)
  ├── assina → User-Signing Key (usk, upk) → assina master keys de outros usuários
  └── assina → Self-Signing Key (ssk, spk) → assina device keys próprias
```

### Protocolos de Ratchet
- **Olm**: Double Ratchet para 1:1 (3DH modificado)
- **Megolm**: Ratchet simétrico para grupos. Chaves de sessão distribuídas via Olm.

### Vulnerabilidade Notável
- **CVE-2022-39248**: Protocol confusion — mensagens to-device criptografadas com Megolm (em vez de Olm) eram aceitas, permitindo injeção de fake key backup secrets

---

## Social Recovery (2025)

### Bottom-Up Secret Sharing (BUSS)
- **Kate et al. (IACR 2025)**: Guardians derivam shares on-the-fly de suas próprias chaves — **sem armazenamento extra**
- Backup: ~146ms (WAN), Recovery: ~1.2ms (LAN) para 8 guardians

### MetaMask Social Login (2025)
- Google/Apple login para wallets self-custodial
- **TOPRF + Shamir's Secret Sharing**: chaves privadas divididas entre servidores independentes
- Nenhuma entidade única pode reconstruir. Password + social auth para recovery.

### IEEE 2025: Wallet-Layer Recovery
- Threshold SSS + DIDComm. Recovery em **menos de 30 segundos**
- Biometric gating + E2EE. Compatível com Hyperledger Indy e W3C DIDs.

---

## Mobile Keychain: iOS Secure Enclave vs Android StrongBox

| Feature | iOS | Android |
|---------|-----|---------|
| Hardware | Secure Enclave (iPhone 5s+) | TEE ou StrongBox (API 28+) |
| API | Keychain + SecKey | Android Keystore |
| Biometric | `.biometryCurrentSet` | `setInvalidatedByBiometricEnrollment(true)` |
| Export | Nunca possível | Nunca possível |
| Biometric Invalidation | Chave permanentemente inacessível | `KeyPermanentlyInvalidatedException` |

### Decisões-Chave
- **Chaves assimétricas não-exportáveis** (signing, key agreement) → enclave-backed
- **Chaves simétricas** (AES) → gerar dentro do enclave, ou wrap com chave assimétrica do Keystore
- **Segredos simples** (tokens) → Keychain/Keystore com acesso biométrico

### Common Pitfalls
- ❌ Storing tokens in UserDefaults / SharedPreferences
- ❌ Não checar `.biometryCurrentSet` → crash ou bypass
- ❌ Software-only Keystore em Android → checar `isInsideSecureHardware()`
- ❌ Hardcoding encryption keys → usar APIs de geração da plataforma

---

## CVEs Relevantes (2025)

| CVE | Plataforma | Severidade | Issue |
|-----|-----------|-----------|-------|
| CVE-2025-62176 | Mastodon | 4.3 Medium | Token scope bypass no streaming API |
| CVE-2025-24896 | Misskey | 8.1 High | Token persiste após logout |
| CVE-2025-46340 | Misskey | 7.2 High | CSS injection via UrlPreviewService |
| CVE-2023-52139 | Misskey | 9.1 Critical | API/WebSocket sem escopo de permissão |
| CVE-2022-39248 | Matrix | High | Protocol confusion Olm/Megolm |

---

## Implicações para o Liberation Client

1. **Nostr**: Verificar TODAS as assinaturas. Isolar chaves por subprotocolo. Usar NIP-44 (NÃO NIP-04). Vincular identidade à chave via NIP-05 + out-of-band.
2. **Matrix**: Usar matrix-rust-sdk com vodozemac. Não implementar criptografia do zero.
3. **Key Storage**: Sempre Secure Enclave / StrongBox. Nunca exportar chaves privadas.
4. **Recovery**: BIP-39 como Tier 1. Social Recovery (Shamir's BUSS) como Tier 2.
5. **Sanitização HTML**: Essencial para posts ActivityPub. Strict allowlist, sem <script>/<iframe>.
6. **Token Hygiene**: Short-lived tokens. Rotação em logout. Nunca em UserDefaults.
