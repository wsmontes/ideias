# Liberation Client — Identity & Keychain Architecture

> **Version:** 1.0.0  
> **Status:** Draft for Cryptographic Review  
> **Security Level:** HIGH — controls all protocol-level authentication  
> **Last Updated:** 2026-07-01

---

## Table of Contents

1. [Scope & Threat Model](#1-scope--threat-model)
2. [Part 1: Key Generation & Derivation Strategy](#part-1-key-generation--derivation-strategy)
3. [Part 2: Hardware-Backed Key Storage](#part-2-hardware-backed-key-storage)
4. [Part 3: Backup & Recovery System](#part-3-backup--recovery-system)
5. [Part 4: Identity Linking (Cross-Protocol)](#part-4-identity-linking-cross-protocol)
6. [Part 5: Key Rotation & Compromise Handling](#part-5-key-rotation--compromise-handling)
7. [Part 6: Implementation Specifications](#part-6-implementation-specifications)
8. [Appendix: References & Rationale](#appendix-references--rationale)

---

## 1. Scope & Threat Model

### 1.1 Trust Assumptions

| Layer | Trust Model |
|---|---|
| **Device OS** | Trusted for secure enclave operations, keychain APIs, and biometric authentication. OS compromise is **out of scope** (the app cannot protect against a rooted/jailbroken OS). |
| **Secure Enclave / TEE** | Trusted to generate and protect private key material from app-level compromise. `SecureEnclave` (iOS) and `StrongBox` (Android) are considered **hardware roots of trust**. |
| **App Process** | NOT trusted for long-lived private keys. Key material must never reside in app heap for longer than the operation requiring it. |
| **Network** | NOT trusted. All protocol authentication occurs over TLS 1.3. |
| **User** | Trusted but fallible. The design assumes the user may forget passphrases, lose devices, or fail to back up. All recovery paths must account for user error. |

### 1.2 Attack Vectors

| Threat | Severity | Mitigation |
|---|---|---|
| **Device theft** | Critical | Biometric gate on all key operations; Secure Enclave Keychain with biometric binding; remote wipe capability |
| **Malware reading app sandbox** | High | Keys never stored in app heap; all operations through SE/KeyStore; memory locking in Rust |
| **Backup compromise** | High | Tier 3 cloud backup is encrypted with Argon2id + user passphrase; mnemonic is never stored digitally unless user explicitly chooses to |
| **Phishing (user tricked into signing)** | Medium | Clear UX indication of what is being signed; per-protocol domain allowlists; signing requires biometric confirmation |
| **Side-channel (timing, power)** | Medium | All signing operations use constant-time implementations (Rust `constant_time_eq`, dalek libraries); Secure Enclave operations are inherently hardened |
| **Protocol-level key substitution** | Critical | Identity binding: every signature includes a domain separator binding to the protocol and the user's other verified identities (cross-signing); see Part 4 |
| **AI-generated crypto code flaws** | Medium | Formal verification of signature aggregation; property-based testing for all cryptographic operations; external audit required before 1.0 |

---

## Part 1: Key Generation & Derivation Strategy

### 1.1 Recommendation: Option A — Single Master Seed

**We choose Option A: BIP-39 single master seed with deterministic derivation.**

#### 1.1.1 Justification

| Criterion | Option A (Master Seed) | Option B (Per-Protocol) |
|---|---|---|
| **Backup UX** | Single 24-word mnemonic | Up to 6 independent keys + tokens + credentials to track |
| **Security isolation** | Good if derivation domains are strongly separated; compromised child key does not reveal master (hardened derivation) | Stronger: compromised Nostr key says nothing about Matrix key |
| **Compromise response** | Rotate seed -> rotate EVERYTHING (massive blast radius) | Rotate per protocol; fine-grained |
| **Recovery complexity** | One mnemonic -> full restore | Must back up each protocol separately or build a separate SSKR/backup system |
| **Cross-protocol identity linking** | Natural: all keys provably derive from same seed -> cryptographic identity proof | Requires manual or heuristic linking |
| **Implementation complexity** | Moderate (one derivation engine, 6 protocol adapters) | Higher (6 independent key storage + backup paths) |
| **Audit surface** | Single audit of derivation engine | 6 independent audits |

**Decision Rationale:** The UX win of "one phrase to back up everything" is decisive for a mass-market consumer app. The blast-radius concern (compromised seed exposes all identities) is mitigated by:

1. **Hardened derivation** (`m/44'/{coin}'`) ensures child key compromise does not leak the master.
2. **Secure Enclave storage** of the master seed makes extraction infeasible without biometrics + physical device.
3. **Optional isolation mode** for high-security users who can enable per-protocol independent seeds (see Section 1.5).

**However**, we acknowledge the critical limitation: **Matrix Megolm session keys and IRC SASL passwords are NOT derivable from a master seed in any standard way**. These require specialized handling:

- **Matrix Megolm:** Session keys are ephemeral and protocol-generated. Matrix's `m.secret_storage` (SSSS) is used to back up the Megolm key backup passphrase, which IS derived from the master seed.
- **IRC SASL:** The account password is server-chosen. We derive an **app-specific high-entropy password** from the seed and store it in the protocol's key-value store. The user sets this as their NickServ password.
- **XMPP OMEMO keys:** These ARE derivable from a master seed (Curve25519 keypair via SLIP-10 or similar).

### 1.2 Derivation Paths

All paths follow BIP-44 / SLIP-44 convention with protocol-specific coin types:

```
Master Seed (256-bit entropy, BIP-39 mnemonic, no passphrase)
│
├── Purpose 44' (BIP-44)
│   │
│   ├── Coin 1237' (Nostr, per NIP-06 / SLIP-44 registration)
│   │   ├── Account 0'
│   │   │   ├── External 0
│   │   │   │   ├── Index 0  → Nostr nsec (secp256k1)
│   │   │   │   ├── Index 1  → Nostr nsec (secondary)
│   │   │   │   └── ...
│   │   │   └── Internal 1
│   │   │       └── ... (change addresses, not used in Nostr)
│   │   └── Account 1' (optional)
│   │
│   ├── Coin 1000' (AT Protocol placeholder registration)
│   │   └── Account 0'
│   │       └── External 0
│   │           └── Index 0  → ATP signing key (secp256k1 or P-256)
│   │
│   ├── Coin 2345' (Matrix placeholder registration)
│   │   └── Account 0'
│   │       └── External 0
│   │           └── Index 0  → Matrix master cross-signing key (Ed25519)
│   │           └── Index 1  → Matrix self-signing key
│   │           └── Index 2  → Matrix user-signing key
│   │
│   ├── Coin 3456' (XMPP placeholder registration)
│   │   └── Account 0'
│   │       └── External 0
│   │           └── Index 0  → OMEMO identity key (Curve25519)
│   │           └── Index 1  → OMEMO signed pre-key
│   │           └── Index 2..N → OMEMO one-time pre-keys
│   │
│   ├── Coin 111' (SSSS — Matrix Secret Storage)
│   │   └── Account 0'
│   │       └── External 0
│   │           └── Index 0  → SSSS "passphrase" (32 bytes)
│   │
│   └── Authenticated encryption sub-key (app-internal)
│       └── Coin 0x7FFFFFFF' (testing/private use)
│           └── Path for IRC password vault key
│
└── Purpose 2' (multi-account hierarchy, future)
```

#### 1.2.1 Detailed Path Specifications

| Protocol | Key Type | Derivation Path | Curve | Bytes |
|---|---|---|---|---|
| **Nostr** (NIP-06) | Signing key | `m/44'/1237'/0'/0/0` | secp256k1 | 32 bytes private scalar |
| Nostr (alt) | Signing key | `m/44'/1237'/0'/1/0` | secp256k1 | 32 bytes |
| **AT Protocol** | Signing key | `m/44'/1000'/0'/0/0` | secp256k1 or P-256 | 32 bytes |
| **Matrix** MSK | Master signing key | `m/44'/2345'/0'/0/0` | Ed25519 | 32 bytes seed |
| **Matrix** SSK | Self-signing key | `m/44'/2345'/0'/0/1` | Ed25519 | 32 bytes seed |
| **Matrix** USK | User-signing key | `m/44'/2345'/0'/0/2` | Ed25519 | 32 bytes seed |
| **XMPP** OMEMO IKey | Identity key | `m/44'/3456'/0'/0/0` | Curve25519 | 32 bytes |
| **XMPP** OMEMO SPK | Signed pre-key | `m/44'/3456'/0'/0/1` | Curve25519 | 32 bytes |
| **XMPP** OMEMO OPK | One-time pre-keys | `m/44'/3456'/0'/0/2..100` | Curve25519 | 32 bytes each |
| **Matrix SSSS** | Encryption key | `m/44'/111'/0'/0/0` | AES-256-GCM | 32 bytes |
| **App Vault** | Symmetric key | `m/44'/0x7FFFFFFF'/0'/0/0` | AES-256-GCM | 32 bytes |

**Note on coin type registrations:** Nostr has SLIP-44 coin type 1237. The others (ATP 1000, Matrix 2345, XMPP 3456) are **placeholder registrations**. Before mainnet, these MUST be registered with SLIP-44 or a Liberation-specific BIP-43 purpose. Until then, use `m/44'/0x80000001'` thru `0x80000004'` (testing range).

### 1.3 Non-Derivable Credentials

The following credentials CANNOT be derived from the master seed and require their own storage path:

| Credential | Source | Storage Mechanism |
|---|---|---|
| **IRC SASL password** | Server-chosen (or user-chosen on registration) | Encrypted in app vault using derived `m/44'/0x7FFFFFFF'/0'/0/0` key; optional Argon2id-wrapped backup |
| **ActivityPub OAuth2 token** | Server-issued (OAuth2 authorization code flow) | Native keychain (Part 2), encrypted at rest |
| **Matrix access token** | Server-issued (login flow) | Native keychain (Part 2), encrypted at rest |
| **Matrix device ID & keys** | Per-device, generated at login | Native keychain + Rust `CryptoStore` |
| **Matrix Megolm inbound sessions** | Per-room, per-session, ephemeral | Rust `CryptoStore` (SQLCipher in app sandbox), backed up via Matrix SSSS |
| **XMPP SASL credentials** | User-chosen password | Encrypted in app vault + optional SCRAM stored key (if server supports) |

### 1.4 Derivation Engine Design

#### 1.4.1 BIP-39 Mnemonic Generation

```text
Entropy Source:   SecureEnclave (iOS) / StrongBox (Android) TRNG
                  → 256 bits (24 words) or 128 bits (12 words)
                  → 24 words RECOMMENDED, 12 words ACCEPTABLE but flagged

BIP-39 Wordlist:  English (mandatory for standardization)
                  Future: localized wordlists for non-English speakers

Passphrase:       EMPTY by default
                  Optional passphrase → prompts for it on every signing op
                  (high-security mode)
```

#### 1.4.2 Key Derivation Function

```text
seed = mnemonic_to_seed(mnemonic, passphrase = "")  // PBKDF2-HMAC-SHA512, 2048 rounds
                                                      // BIP-39 standard

For each protocol key at derivation path P:
  key_data = CKD(seed, P)  // BIP-32 CKD (HMAC-SHA512)

  // Post-processing per protocol curve:
  if curve == secp256k1:
    private_key = parse256k1(key_data % n)  // BIP-32 secp256k1 tweak
  if curve == Ed25519:
    private_key = sha512(key_data)[0..32]    // Ed25519 clamp + hash (SLIP-0010)
  if curve == Curve25519:
    private_key = clamp25519(sha512(key_data)[0..32])  // SLIP-0010 w/ clamp
  if curve == P-256:
    private_key = key_data[0..32]  // FIPS 186-4 B.4.1 rejection sampling
```

#### 1.4.3 Important Variants

For **Ed25519** and **Curve25519**, BIP-32 raw derivation does not work directly (not a prime-order group for standard BIP-32 CKD). We use **SLIP-0010** which replaces CKD with a modified HMAC-SHA512 chain for Edwards and Montgomery curves:

```
// SLIP-0010 for Ed25519 / Curve25519
I = HMAC-SHA512(Key = chain_code, Data = 0x00 || parent_private_key || index)
   for hardened paths; index >= 0x80000000

priv_key_derived = I[0..32]
chain_code      = I[32..64]
```

### 1.5 Optional High-Security Mode: Per-Protocol Independent Seeds

For users who explicitly opt in (Tier 2+ security awareness), allow:

```text
Mode: "Independent" (flagged in settings as "high security, high complexity")

- Each protocol generates its own BIP-39 seed independently
- Each seed stored separately in Secure Enclave
- Backup: each seed must be backed up separately, OR wrapped into a single
  encrypted seed vault protected by a master passphrase
- Recovery: must restore each seed individually
```

This mode is NOT the default. The onboarding flow defaults to Single Master Seed. Independent mode is offered only after a warning dialog about backup complexity.

### 1.6 Seed Lifetime & Renewal

| Event | Action |
|---|---|
| **First launch** | Generate seed, display mnemonic, force backup confirmation |
| **App reinstall (same device)** | Read from keychain (survives app deletion if Keychain access group persists) |
| **New device** | Must enter mnemonic or restore from cloud backup |
| **Seed compromise event** | Generate new seed, rotate all protocol keys, notify user of each account's new identity |
| **Seed lockout (forgot mnemonic)** | Offer cloud backup restore (Tier 3) or social recovery (Tier 2) if configured; otherwise IRRECOVERABLE — user must create new identity |

---

## Part 2: Hardware-Backed Key Storage

### 2.1 Platform Capabilities

#### 2.1.1 iOS Security Enclave

| Capability | Available Since | Notes |
|---|---|---|
| `SecureEnclave` key generation | A7 (iPhone 5S, 2013) | ECDSA secp256r1 only; NO Ed25519, NO Curve25519, NO secp256k1 |
| `SecureEnclave` biometric binding | A7 + Touch ID | `.biometryCurrentSet` or `.userPresence` |
| Keychain `.AccessControl` | iOS 7+ | Application password, biometric, or both |
| Keychain `.kSecAttrAccessible` | iOS 4+ | `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` |
| `LAContext` biometric caching | iOS 9+ | `.localizedReason`, `.touchIDAuthenticationAllowableReuseDuration` |

**Critical Limitation:** iOS Secure Enclave does NOT support secp256k1, Ed25519, or Curve25519 key generation. It only supports:
- ECDSA secp256r1 (P-256)
- ECDSA secp384r1 (P-384)
- ECDSA secp521r1 (P-521)
- EC Diffie-Hellman (P-256, P-384, P-521)
- AES (limited, for key wrapping)
- HMAC-SHA256

**Implication for this design:** Keys requiring secp256k1 (Nostr, AT Protocol), Ed25519 (Matrix), or Curve25519 (XMPP OMEMO, Matrix Olm) cannot be generated or stored inside the Secure Enclave. They must:
1. Be generated from the seed (outside SE, in Rust)
2. Be stored in the **Keychain** (not SE) with `.AccessControl` biometric gate
3. Use the SE only for **wrapping the key** with a P-256 key that never leaves the SE

#### 2.1.2 Android StrongBox / TEE

| Capability | Available Since | Notes |
|---|---|---|
| `AndroidKeyStore` with `PURPOSE_SIGN` | API 18+ | Supports RSA, EC, AES |
| StrongBox key storage | API 28+ (Pixel 2+, Samsung S8+) | Dedicated security chip; supports Ed25519 on some devices |
| `setUserAuthenticationRequired(true)` | API 30+ | Biometric timeout with `setUserAuthenticationValidityDurationSeconds` |
| `setInvalidatedByBiometricEnrollment(true)` | API 24+ | Keys invalidated when biometrics change |
| `setIsStrongBoxBacked(true)` | API 28+ | Requests StrongBox; can throw `StrongBoxUnavailableException` |

**Android KeyStore supports:** RSA, DSA, EC (P-256, P-384, P-521), AES, HMAC, **Ed25519** (API 28+ StrongBox), **X25519** (API 28+ StrongBox).

**Critical Limitation:** secp256k1 is NOT supported by `AndroidKeyStore` on any known device. As with iOS, secp256k1 keys must be software-generated and stored with key wrapping.

### 2.2 Storage Strategy: Hybrid Wrapped-Key Architecture

Given platform limitations (no secp256k1, limited Curve25519 support in SE), we adopt a **Hybrid Wrapped-Key Architecture**:

```
┌──────────────────────────────────────────────────────────────────┐
│                    Secure Enclave / StrongBox                     │
│                                                                  │
│  P-256 Key Pair (static, per-app-install)                        │
│  - Purpose: Key Wrapping Key (KWK)                                │
│  - Never exportable                                               │
│  - Biometric gated                                                │
│  - Used ONLY for ECIES wrapping of protocol keys                 │
│  - Algorithm: `kSecAlgorithmECIESEncryptionCofactorVariableIV`   │
│    (iOS) or `EC` with `PURPOSE_WRAP_KEY` (Android)               │
└──────────────────────┬───────────────────────────────────────────┘
                       │
                       │ Wraps/unwraps
                       ▼
┌──────────────────────────────────────────────────────────────────┐
│                      Keychain / KeyStore                         │
│                                                                  │
│  Wrapped Protocol Keys (stored as encrypted blobs):              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌───────┐  │
│  │ Nostr    │ │ ATP      │ │ Matrix   │ │ XMPP    │ │ IRC   │  │
│  │ secp256k1│ │ secp256k1│ │ Ed25519  │ │X25519   │ │Vault  │  │
│  │ CBOR enc │ │ CBOR enc │ │ CBOR enc │ │CBOR enc │ │CBOR   │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └───────┘  │
│                                                                  │
│  Access Token (unwrapped, plaintext in keychain):                │
│  ┌────────────────────┐ ┌──────────────┐ ┌────────────────┐      │
│  │ Matrix access token│ │ Mastodon     │ │ XMPP SASL      │      │
│  │                    │ │ OAuth2 token │ │ credentials    │      │
│  └────────────────────┘ └──────────────┘ └────────────────┘      │
└──────────────────────────────────────────────────────────────────┘
```

**Why wrapping instead of storing raw keys in the keychain?** The Keychain and AndroidKeyStore already provide OS-level encryption at rest. However, wrapping with a Secure-Enclave-bound key adds an additional layer of biometric gating on EVERY key operation: even if the device is unlocked, the app cannot unwrap keys without biometric authentication. This is architecturally significant.

### 2.3 Credential Storage Matrix

| Credential | iOS Storage | Android Storage | Biometric Binding | Exportable? |
|---|---|---|---|---|
| **Master seed / BIP-39 mnemonic** | Keychain: `kSecClassGenericPassword`, `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` + `SecAccessControlCreateWithFlags(.biometryCurrentSet \| .andPrivateKeyUsage)` — Wrapped with SE P-256 key | `EncryptedSharedPreferences` (AES-256 GCM) OR `AndroidKeyStore` `PURPOSE_ENCRYPT`/`PURPOSE_DECRYPT` with `KEY_PURPOSE_WRAP`; wrapped with StrongBox P-256 key | YES — biometric required for all signing operations; cached for 60 seconds max for bulk operations during active session | NO — never leaves device except via Recovery system (Tiers 2-4). Migration: user enters mnemonic on new device |
| **Nostr nsec (secp256k1)** | Keychain: wrapped blob under SE KWK; tagged with `com.liberation.nostr.signing` | `AndroidKeyStore`: stored as encrypted blob under StrongBox P-256 wrapping key | YES | YES — only as mnemonic-derived (never export raw nsec); export via Settings > Export Key |
| **ATP signing key (secp256k1 / P-256)** | Same as Nostr, tagged `com.liberation.atp.signing`; if P-256, can use SE directly (no wrapping needed) | `AndroidKeyStore`: if P-256: `ECKeyPairGenerator` in StrongBox; if secp256k1: wrapped blob | YES | Only as part of master seed recovery (never export raw key) |
| **Matrix access token** | Keychain: `kSecClassGenericPassword`, `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`, NO biometric (tokens are short-lived and app-initiated); token is an opaque server secret | `EncryptedSharedPreferences`, plain AES-256, NO biometric | NO (access tokens are ephemeral server credentials, not long-lived keys) | YES — transferable with device migration; encrypted in transit |
| **Matrix cross-signing keys (Ed25519 MSK/SSK/USK)** | Keychain: wrapped blob under SE KWK; tagged `com.liberation.matrix.msk` etc. | `AndroidKeyStore` with StrongBox: if device supports Ed25519 in StrongBox, store directly; else wrapped blob | YES — biometric required; cross-signing operations are infrequent (device verification) | NO — only via SSSS backup (Matrix protocol feature); or full seed recovery |
| **Matrix device keys (Curve25519 for Olm/Megolm)** | Keychain: wrapped blob under SE KWK | `AndroidKeyStore` with StrongBox: if device supports X25519, store directly; else wrapped blob | YES for device key; NO for session keys (too many) | Device keys: NO; Session keys: via Matrix key backup |
| **Matrix Megolm inbound sessions** | Rust `CryptoStore` (SQLCipher) in app sandbox; NOT in keychain (too many, too ephemeral) | Same — SQLCipher in app sandbox | NO | Via Matrix E2EE key backup (encrypted with SSSS key) |
| **Mastodon OAuth2 token** | Keychain: `kSecClassGenericPassword`, `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`, NO biometric | `EncryptedSharedPreferences` | NO | YES — migration via `migration.json` export (encrypted) |
| **XMPP SASL credentials** | Keychain: wrapped under app vault key (seed-derived); `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` | `EncryptedSharedPreferences` with key from seed-derived vault key | OPTIONAL — user choice at credential creation | YES — as part of seed recovery (vault is unlocked by seed) |
| **XMPP OMEMO keys (Curve25519)** | Keychain: wrapped blob under SE KWK | `AndroidKeyStore` with StrongBox | YES — biometric for identity key; NO for pre-keys (too many) | NO — only via seed recovery |
| **IRC SASL / NickServ password** | Keychain: wrapped under app vault key | `EncryptedSharedPreferences` with vault key | OPTIONAL | YES — vault key derives from seed; store encrypted blob in migration |
| **SSSS key (Matrix Secret Storage)** | Keychain: wrapped blob under SE KWK; tagged `com.liberation.matrix.ssss` | Same pattern | YES — biometric on first unlock per session; then cached 60s | NO — only via seed recovery (derived from seed) |
| **Login session state** | `UserDefaults` (non-sensitive) or `CoreData` for structured state | `DataStore` / `SharedPreferences` | NO | N/A |

### 2.4 Keychain Access Control — iOS Detail

```swift
// MARK: - Keychain Access Control Specifications

/// Wrapped protocol key (e.g., Nostr secp256k1, Matrix Ed25519)
func wrappedProtocolKeyQuery(tag: String) -> CFDictionary {
    var access: SecAccessControl?
    let error: Unmanaged<CFError>?
    
    access = SecAccessControlCreateWithFlags(
        nil,                                        // allocator
        kSecAttrAccessibleWhenUnlockedThisDeviceOnly, // accessibility
        .biometryCurrentSet | .andPrivateKeyUsage,   // flags
        &error
    )
    
    return [
        kSecClass: kSecClassKey,
        kSecAttrApplicationTag: tag.data(using: .utf8)!,
        kSecAttrAccessControl: access!,
        kSecAttrKeyType: kSecAttrKeyTypeEC,          // wrapping key is P-256
        kSecUseDataProtectionKeychain: true,
    ] as CFDictionary
}

/// Access token (no biometric)
func accessTokenQuery(account: String) -> CFDictionary {
    return [
        kSecClass: kSecClassGenericPassword,
        kSecAttrAccount: account,
        kSecAttrAccessible: kSecAttrAccessibleWhenUnlockedThisDeviceOnly,
        kSecUseDataProtectionKeychain: true,
    ] as CFDictionary
}

/// Biometric caching policy
/// - We set `LAContext.touchIDAuthenticationAllowableReuseDuration = 60`
/// - This means after first biometric unlock, subsequent key operations
///   within 60 seconds do NOT prompt again
/// - Rationale: bulk operations (message decryption, signing multiple events)
///   should not spam the user with biometric prompts
/// - Security: P60 second window is standard for password managers; the
///   device must be unlocked (PIN/biometric already satisfied)
```

### 2.5 Android KeyStore Detail

```kotlin
// MARK: - Android KeyStore Specifications

fun createWrappingKey(context: Context) {
    val keyStore = KeyStore.getInstance("AndroidKeyStore").apply { load(null) }
    
    if (!keyStore.containsAlias("liberation_kwk")) {
        val keyGenerator = KeyPairGenerator.getInstance(
            KeyProperties.KEY_ALGORITHM_EC, "AndroidKeyStore"
        )
        val spec = KeyGenParameterSpec.Builder("liberation_kwk")
            .setAlgorithmParameterSpec(ECGenParameterSpec("secp256r1"))
            .setDigests(KeyProperties.DIGEST_SHA256)
            .setKeySize(256)
            .setUserAuthenticationRequired(true)  // biometric needed
            .setUserAuthenticationParameters(
                60,  // 60-second timeout
                KeyProperties.AUTH_BIOMETRIC_STRONG
            )
            .setIsStrongBoxBacked(true)  // request StrongBox (may throw)
            .setKeyPurpose(KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT)
            .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
            .build()
        keyGenerator.initialize(spec)
        keyGenerator.generateKeyPair()
    }
}

fun storeWrappedProtocolKey(
    tag: String,
    plaintextKey: ByteArray,
    context: Context
) {
    val kwkPublicKey = // get KWK public key from AndroidKeyStore
    val encrypted = ECIES.encrypt(kwkPublicKey, plaintextKey)
    // Store encrypted blob in EncryptedSharedPreferences
    EncryptedSharedPreferences.create(
        "liberation_crypto_store",
        MasterKey.DEFAULT_MASTER_KEY_ALIAS,
        context,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    ).edit().putString(tag, Base64.encodeToString(encrypted)).apply()
}

fun unwrapProtocolKey(
    tag: String,
    context: Context
): ByteArray {
    val encrypted = // read from EncryptedSharedPreferences
    val kwkPrivateKey = // get KWK private key from AndroidKeyStore
    // This triggers biometric on first use or after 60s timeout
    return ECIES.decrypt(kwkPrivateKey, encrypted)
}
```

### 2.6 Wrapping Algorithm: ECIES

Both platforms use ECDH + AES-256-GCM + HKDF-SHA256 for key wrapping:

```text
Algorithm: ECIES-WRAP-256 (custom, not a standard, but follows SECG/ISO 18033-2)

1. Ephemeral key generation:
   - Generate ephemeral P-256 key pair (ephem_sk, ephem_pk) from SE/StrongBox TRNG
   
2. ECDH key agreement:
   - shared_secret = ECDH(ephem_sk, KWK_pk)
   - KWK_pk is the static wrapping key inside Secure Enclave / StrongBox

3. KDF:
   - enc_key = HKDF-SHA256(
       salt = "Liberation-ECIES-V1" || protocol_tag,
       ikm  = shared_secret || ephem_pk_x || ephem_pk_y,
       info = protocol_tag,
       length = 32
     )

4. AEAD:
   - ciphertext, tag = AES-256-GCM-Encrypt(
       key   = enc_key,
       nonce = random_12_bytes (stored alongside),
       aad   = protocol_tag || 0x00
     )

5. Output blob (CBOR):
   {
     version: 1,
     ephem_pk: { x: 32 bytes, y: 32 bytes },
     nonce: 12 bytes,
     ciphertext: variable,
     tag: 16 bytes,
     protocol_tag: string
   }

Total overhead: ~100 bytes per wrapped key
```

### 2.7 Biometric Binding Policies

| Operation | Biometric Required? | Caching | Rationale |
|---|---|---|---|
| **Sign a Nostr event** | YES | 60s | Cryptographic identity; needs user intent |
| **Sign an ATP post** | YES | 60s | Same as Nostr |
| **Decrypt Matrix message** | NO | N/A | Too frequent; device unlock is sufficient |
| **Verify Matrix device** | YES | 60s | High-security operation |
| **Unlock Matrix SSSS** | YES | Session (app in memory) | Accesses all secret storage |
| **Read Mastodon timeline** | NO | N/A | Token-based, read-only |
| **Post Mastodon status** | OPTIONAL (user config) | Session | User can disable for convenience |
| **IRC connect** | NO | N/A | Password-based |
| **XMPP connect** | NO | N/A | SASL credentials |
| **Export private key** | YES (MANDATORY) | 0s (no cache) | High-risk operation |
| **Display mnemonic** | YES (MANDATORY) | 0s | Single most sensitive data |
| **Seed recovery / import** | YES (MANDATORY) | 0s | New device setup |

### 2.8 Export / Migration Strategy

| Scenario | Mechanism |
|---|---|
| **Device-to-device migration (same platform)** | OS-native encrypted backup (iCloud / Google Drive) transfers keychain; user still needs biometric on new device. Keychain items with `ThisDeviceOnly` flag do NOT transfer — only seed recovery works for these. |
| **Cross-platform migration (iOS <-> Android)** | Mnemonic entry + cloud backup download (Tier 3). Access tokens must be re-authenticated (OAuth2 refresh / Matrix login). |
| **Full backup export** | Settings > Advanced > Export Encrypted Backup → file protected with user passphrase (Argon2id). Contains: wrapped keychain blobs, all tokens, IRC/XMPP vault. Does NOT contain Megolm session keys (separate Matrix key backup). |
| **Emergency key export** | Individual protocol keys can be exported in protocol-native format: Nostr nsec (bech32), Matrix device keys (JSON), etc. Each export requires fresh biometric. Logged as security event. |

---

## Part 3: Backup & Recovery System

### 3.1 Tier Architecture

```
Tier 1 ──────────────────────────────────────────────────────────────
  BIP-39 Mnemonic (MANDATORY)
  ├─ 24-word BIP-39 English phrase
  ├─ Displayed ONCE during onboarding
  ├─ User MUST write down / save to password manager
  └─ Recovery: enter 24 words → regenerate all protocol keys
       Exception: Matrix Megolm sessions (need key backup)

Tier 2 ──────────────────────────────────────────────────────────────
  Social Recovery (OPTIONAL, power users)
  ├─ 3-of-5 Shamir's Secret Sharing over BIP-39 seed
  ├─ Shards sent to trusted guardians
  └─ Recovery: collect 3 shards → reconstruct seed

Tier 3 ──────────────────────────────────────────────────────────────
  Encrypted Cloud Backup (OPTIONAL, convenience)
  ├─ Master seed encrypted with Argon2id KDF + user passphrase
  ├─ Stored in iCloud / Google Drive / Matrix SSSS
  └─ Recovery: download file + enter passphrase

Tier 4 ──────────────────────────────────────────────────────────────
  Protocol-Specific Recovery (SUPPLEMENTAL)
  ├─ Matrix SSSS → cross-signing keys + Megolm key backup
  ├─ AT Protocol → PLC recovery key (did:plc)
  └─ Nostr → NIP-06 derivation (always from seed)
```

### 3.2 Tier 1: BIP-39 Mnemonic (Mandatory)

#### 3.2.1 Onboarding Flow

```
Screen 1: Welcome
  └─ "Liberation Client uses a master key to secure all your accounts."
  └─ "One phrase to remember. All your identities."

Screen 2: Seed Generation
  └─ Device generates 256 bits TRNG → BIP-39 24-word phrase
  └─ Shows seed entropy animation (visualizing randomness)
  └─ WARNING: "This is the ONLY copy. We cannot show it again."

Screen 3: Write Down
  └─ Words displayed one at a time (user taps to advance)
  └─ Or: grid display with "I have written this down" checkbox
  └─ Button: "Copy to Clipboard" (warns: "Clipboard is readable by other apps")
  └─ Button: "Save to Password Manager" (iOS: SFBetterSafariViewController to
     system password manager prompt)
  └─ QR code option (for air-gapped transfer to hardware wallet)
  └─ REQUIRED: Show message "Seed is never stored digitally on this device
     unless you explicitly save to password manager"

Screen 4: Verification Quiz
  └─ User must enter 3 random words from their phrase
  └─ e.g., "What is word #7? What is word #14? What is word #22?"
  └─ 3 attempts max; on failure, return to Screen 3

Screen 5: Confirmation
  └─ "Your seed is your identity. If you lose it, your accounts are gone."
  └─ Checkboxes:
     [ ] I have written down all 24 words
     [ ] I understand that Liberation Client cannot recover my seed
     [ ] I will store this in a safe place, not on this device
  └─ All checked → Enable "Continue" button → proceed to protocol setup
```

#### 3.2.2 Recovery Flow

```
Screen 1: Welcome Back
  └─ "Enter your recovery phrase to restore your identities."

Screen 2: Word Entry
  └─ BIP-39 autocomplete keyboard (word suggestions as user types)
  └─ 24 fields (12 fields for 12-word mode)
  └─ Paste detection: if 24+ words detected in paste buffer, auto-fill
     (warns if pasting on device where clipboard may have been read)
  └─ Validation: checksum verification on each word (BIP-39 checksum)

Screen 3: Verification
  └─ Checksum valid → proceed
  └─ Checksum invalid → highlight incorrect word(s), allow correction

Screen 4: Protocol Recovery
  └─ "Recovering your identities..."
  └─ Regenerate all protocol keys from seed
  └─ For protocols needing server auth (Matrix, Mastodon, XMPP, IRC):
     └─ "Access tokens cannot be recovered from seed alone."
     └─ Offer: "Sign in to [protocol] with your username/password"
     └─ Matrix-specific: "Restore encrypted messages from key backup?"
        [Use SSSS key from seed] or [Skip, messages are lost]
  └─ Nostr / ATP: keys restored from seed, ready to use immediately
     (these are deterministic keypairs)

Screen 5: Result
  └─ Shows which protocols were fully restored (Nostr, ATP)
  └─ Shows which need re-authentication (Matrix, Mastodon, XMPP, IRC)
  └─ Shows Megolm session loss warning (if no key backup)
  └─ "Your identities are restored. Welcome back."
```

### 3.3 Tier 2: Social Recovery (Optional, Power Users)

#### 3.3.1 Scheme: SLIP-0039 (Shamir's Secret Sharing)

**We use SLIP-0039** (Shamir's Secret Sharing for BIP-39 mnemonics) instead of raw SSS, because:
- Standardized: interop with hardware wallets (Trezor, KeepKey)
- Includes verification shares (detect tampering)
- Supports passphrase-protected shares
- Group thresholds (e.g., 2-of-3 from family + 2-of-3 from friends = 4-of-6 total)

**We also evaluate BUSS (Bottom-Up Secret Sharing)** for comparison but choose SLIP-0039 for hardware wallet compatibility and audit maturity.

#### 3.3.2 Parameters

```text
Default scheme: 3-of-5 (threshold = 3, total shares = 5)

Share structure per SLIP-0039:
- Master Secret: 128 bits (12-word equivalent; seed is derived from this)
  -> Note: SLIP-0039 uses 128/256-bit master secret, NOT the full BIP-39 seed.
     The master secret IS the entropy that generates the BIP-39 mnemonic.
- Share length: 20 words each (BIP-39 wordlist, 10 bits/word)
- Identifier: random 15-bit value, same across all shares
- Checksum: 7 bits per share (tamper detection)

Share format:
  "label" + " " + identifier (3 words) +  " " + share_data (N words) + checksum
  Example: "theory romance hint theory ... census"

Guardian assignment:
  Guardian 1: Close friend (in person)
  Guardian 2: Close friend (in person)
  Guardian 3: Family member (in person or video call)
  Guardian 4: Secondary device (hardware wallet or old phone)
  Guardian 5: Self (safe deposit box or fireproof safe)
```

#### 3.3.3 Guardian Enrollment Flow

```
Step 1: User initiates Social Recovery setup
  └─ Settings > Backup & Recovery > Social Recovery > Set Up

Step 2: Choose parameters
  └─ "How many guardians?" (default: 5)
  └─ "How many are needed to recover?" (default: 3)
  └─ Warning: "Choose at least 3 guardians you trust not to collude"
  └─ "Choose guardians in different locations (not all the same family/friend group)"

Step 3: Generate shares
  └─ Device generates SLIP-0039 shares from master secret
  └─ Each share is a 20-word BIP-39 phrase
  └─ Device encrypts each share for its guardian before transmission

Step 4: Distribute shares
  └─ For each guardian, the app guides:

     OPTION A: In Person (SECURE)
       └─ User opens Guardian QR code on their device
       └─ Guardian scans QR with their own Liberation Client device
         (guest mode / no setup required)
       └─ Guardian's device stores the share in THEIR keychain
         (encrypted recipient: guardian's device KWK)
       └─ Guardian sees: "You are a recovery guardian for [user display name].
          Only act on this if they contact you directly."

     OPTION B: Encrypted Message (moderate security)
       └─ Share encrypted with guardian's Nostr public key (NIP-44 encryption)
       └─ Or: encrypted Signal message / Matrix DM
       └─ Guardian receives encrypted blob, imports into their Liberation Client
       └─ Transport note: we encrypt the share with the guardian's known public
          key; the transport itself (DMs, email) is untrusted

     OPTION C: Print / Write Down (self-guardian)
       └─ Display the 20-word share on screen
       └─ User writes it down and stores in safe deposit box
       └─ "Store this separately from your master seed"

Step 5: Verification
  └─ After enrollment, device performs a DRY RUN verification:
  └─ Temporarily reconstructs master secret from a random 3-subsets of shares
     (in-memory only, never persisted)
  └─ Validates: reconstructed seed == original seed
  └─ "Social recovery verified. Your guardians can help you recover."

Step 6: Guardian list display
  └─ Shows all 5 guardians (name / relationship / share ID prefix)
  └─ Shows last verification date
  └─ "Contact these guardians if you need to recover your identity"
```

#### 3.3.4 Recovery Flow (Social)

```
Step 1: User initiates recovery
  └─ "I lost my device" / "I forgot my seed"
  └─ "Contact your guardians to request their recovery shares."

Step 2: Collect shares
  └─ User visits guardians in person or calls them
  └─ Guardian opens Liberation Client > Account Recovery
     > "I have a recovery request" > Shows QR code or share phrase
  └─ Alternative: Guardian reads their 20-word share over phone
     (user types it in; lower security)

Step 3: Reconstruct
  └─ After 3 shares collected, device reconstructs SLIP-0039 master secret
  └─ Derives BIP-39 seed from master secret
  └─ "Identity recovered. You can now restore your accounts."

Step 4: Invite guardians to rest
  └─ "Your recovery is complete. Your guardians' shares have served
     their purpose. Should they keep their shares for future recovery?"
  └─ If yes: guardian's share remains valid (with same identifier)
  └─ If no: guardian deletes their share
```

### 3.4 Tier 3: Encrypted Cloud Backup (Optional, Convenience)

#### 3.4.1 Encryption Scheme

```text
Input: master_seed (64 bytes — BIP-39 seed, NOT mnemonic)
Output: encrypted_backup_file

1. Derive encryption key:
   salt = random_16_bytes
   key = Argon2id(
       password = user_passphrase,
       salt = salt,
       time = 3,          // OWASP recommended minimum 2, we use 3
       mem = 64 * 1024,   // 64 MB (OWASP recommended 19 MiB min; higher is better)
       parallelism = 4,
       key_length = 32    // AES-256
   )

2. Encrypt with AEAD:
   nonce = random_12_bytes
   ciphertext, tag = AES-256-GCM-Encrypt(
       key = key,
       nonce = nonce,
       plaintext = master_seed || protocol_derivation_paths_hash,
       aad = "LiberationClient-Backup-v1" || salt || Argon2id_params
   )

3. Package:
   {
     version: 1,
     salt: salt,
     nonce: nonce,
     ciphertext: ciphertext,
     tag: tag,
     argon2_params: { t: 3, m: 65536, p: 4 },
     backup_type: "master_seed",
     created_at: ISO8601_timestamp
   }

4. Serialize as CBOR → file extension .lcbak
```

#### 3.4.2 Passphrase Strength Estimation

During creation, app shows real-time strength meter:

```text
< 8 characters:        RED    "Too short. Use at least 12 characters."
8-10 characters:       ORANGE "Weak. Add more words or characters."
10-14 characters:      YELLOW "Moderate."
14-20 characters:      LIGHT  "Strong."
20+ characters:        GREEN  "Very strong."

Additionally:
- Passphrase entropy estimation (zxcvbn algorithm)
- Checks against 100K common password list
- "Avoid: birthdays, names, common phrases"
- "Use a passphrase: 4-5 random words is stronger than 12 random characters"
- "Write this down SEPARATELY from your seed phrase"
```

#### 3.4.3 Cloud Storage Integration

| Cloud Provider | Storage Location | Platform |
|---|---|---|
| **iCloud Drive** | `Liberation Client/backup-v1.lcbak` | iOS, macOS |
| **Google Drive** | `Apps/LiberationClient/backup-v1.lcbak` | Android |
| **Matrix SSSS** | `m.secret_storage.default_key` — encrypted seed stored as Matrix account data event | All platforms (Matrix account required) |
| **Local file export** | User-chosen location (USB drive, file server) | All platforms |

#### 3.4.4 Cloud Backup Recovery Flow

```
Screen 1: "Restore from Cloud Backup?"
  └─ Detects backup file in iCloud/Google Drive (or user selects file)

Screen 2: Enter Passphrase
  └─ Text field with show/hide toggle
  └─ "Enter the passphrase you chose when creating this backup."
  └─ On submit:
     └─ Derive key from passphrase + salt with Argon2id
     └─ Attempt AES-256-GCM decryption
     └─ On failure: "Incorrect passphrase. [N attempts remaining]"
     └─ On success: proceed to protocol recovery (same as Tier 1 recovery from here)

Screen 3: Delete Remote Backup?
  └─ "Your seed has been restored. Should we delete the cloud backup now?"
  └─ [Keep for Future] or [Delete Backup]
  └─ Note: if choosing Delete, file is moved to trash (recoverable for 30 days)
```

### 3.5 Tier 4: Protocol-Specific Recovery

#### 3.5.1 Matrix SSSS — Cross-Signing Keys & Key Backup

Matrix SSSS (`m.secret_storage`) stores encrypted secrets on the Matrix homeserver as account data events. We use the SSSS key derived from the master seed (`m/44'/111'/0'/0/0`) as the default key.

```text
SSSS Key ID:   "liberation-master-key"
SSSS Algorithm: "m.secret_storage.v1.aes-hmac-sha2"

Secrets stored in SSSS:
  ├─ m.cross_signing.master     → Matrix master signing key (Ed25519)
  ├─ m.cross_signing.self_signing → Matrix self-signing key
  ├─ m.cross_signing.user_signing → Matrix user-signing key
  └─ m.megolm_backup.v1        → Megolm key backup private key (Curve25519)

Recovery flow:
  1. User logs into Matrix with username/password (access token NOT from seed)
  2. App derives SSSS key from seed path m/44'/111'/0'/0/0
  3. SSSS key decrypts the 4 secrets from server
  4. Cross-signing keys restored → device verified
  5. Megolm backup key restored → decrypt key backup download → message history restored
```

#### 3.5.2 AT Protocol — PLC Directory Recovery

For `did:plc` DIDs, the PLC directory supports a rotation key separate from the signing key:

```text
PLC operation recovery:
  If user has the PLC recovery key (derived from seed at path m/44'/1000'/1'/0/0):
    ├─ Can rotate the signing key in DID document
    ├─ Can change the associated handle (if handle changed by attacker)
    └─ Can recover control of the DID

  If user does NOT have PLC recovery key (lost seed):
    └─ DID is unrecoverable — must create new AT Protocol identity
```

#### 3.5.3 Nostr — Always From Seed

Nostr keys are always derived via NIP-06. There is no "recovery" beyond the seed. If a user wants to change their Nostr key, they must create a new identity (there is no key rotation on Nostr without NIP-26 delegation — see Part 5).

### 3.6 What Happens to Matrix Megolm Sessions?

This is the single most important UX/recovery trade-off in the design:

```text
┌────────────────────────────────────────────────────────────────────┐
│                   Megolm Session Recovery Matrix                    │
├────────────────────┬──────────────┬───────────────────────────────┤
│ Backup configured? │ Recovered?   │ User Experience               │
├────────────────────┼──────────────┼───────────────────────────────┤
│ YES (SSSS backup   │ YES          │ All messages decrypt.         │
│ of megolm backup   │              │ Seamless recovery.            │
│ private key)       │              │                               │
├────────────────────┼──────────────┼───────────────────────────────┤
│ YES (SSSS)         │ YES, but     │ Messages from before backup   │
│                    │ some sessions│ were lost. New messages from   │
│                    │ deleted on   │ post-backup decrypt.          │
│                    │ server       │                               │
├────────────────────┼──────────────┼───────────────────────────────┤
│ NO (never set up   │ NO           │ All past messages show as     │
│ key backup)        │              │ "Unable to decrypt"           │
│                    │              │ User must ask contacts to     │
│                    │              │ re-send or accept loss.       │
│                    │              │ New messages will decrypt.    │
├────────────────────┼──────────────┼───────────────────────────────┤
│ NO (backup set up  │ PARTIAL      │ Without SSSS key, megolm      │
│ but SSSS key lost) │              │ backup key is unrecoverable.  │
│                    │              │ Same as "no backup" above.    │
├────────────────────┼──────────────┼───────────────────────────────┤
│ Device was logged  │ N/A          │ Old device has the sessions.  │
│ into Matrix        │              │ If old device accessible,     │
│ concurrently       │              │ export sessions via Matrix    │
│                    │              │ self-verification process.    │
└────────────────────┴──────────────┴───────────────────────────────┘

Recommendation:
  ONBOARDING: During Matrix account setup, STRONGLY recommend enabling
  key backup. Show the Megolm session loss warning prominently if declined.
  Make it sticky: nag monthly until enabled.
```

---

## Part 4: Identity Linking (Cross-Protocol)

### 4.1 Local Identity Model

Each user has one or more **Personas**. A Persona is a local abstraction that groups protocol-level identities together.

```text
Persona
├── display_name: String (user-chosen, e.g., "Work", "Personal")
├── avatar: Data? (local-only; not published)
├── created_at: Timestamp
├── is_primary: Bool (default persona for new contacts)
│
├── Protocol Identities (1..N)
│   ├── NostrIdentity
│   │   ├── npub: String
│   │   ├── nsec_derivation_path: "m/44'/1237'/0'/0/0"
│   │   ├── nip05: String?
│   │   ├── relays: [String]
│   │   ├── display_name: String (from kind:0 metadata)
│   │   └── connected: Bool
│   │
│   ├── ATPIdentity
│   │   ├── did: String (did:plc:... or did:key:...)
│   │   ├── handle: String (user.bsky.social)
│   │   ├── signing_key_path: "m/44'/1000'/0'/0/0"
│   │   ├── pds_endpoint: String
│   │   └── connected: Bool
│   │
│   ├── ActivityPubIdentity (Mastodon / Pleroma / etc.)
│   │   ├── instance: String (mastodon.social)
│   │   ├── username: String
│   │   ├── acct: String (@user@instance)
│   │   ├── oauth_token_hash: String (SHA-256 of token for dedup)
│   │   └── connected: Bool
│   │
│   ├── MatrixIdentity
│   │   ├── mxid: String (@user:server)
│   │   ├── device_id: String
│   │   ├── homeserver: String
│   │   ├── access_token_hash: String
│   │   └── connected: Bool
│   │
│   ├── XMPPIdentity
│   │   ├── jid: String (user@domain)
│   │   ├── sasl_mechanism: String ("SCRAM-SHA-256" | "PLAIN")
│   │   └── connected: Bool
│   │
│   └── IRCIdentity
│       ├── nickname: String
│       ├── network: String (irc.libera.chat)
│       ├── sasl: Bool
│       └── connected: Bool
│
├── Linked Contacts (local address book)
│   ├── Contact
│   │   ├── local_uuid: UUID
│   │   ├── display_name: String
│   │   ├── notes: String?
│   │   ├── last_interaction: Timestamp
│   │   ├── trust_level: enum { unknown, observed, cross-signed, manual }
│   │   │
│   │   ├── Protocol References
│   │   │   ├── Nostr: npub1...
│   │   │   ├── ATP: did:plc:...
│   │   │   ├── AP: @user@instance
│   │   │   ├── Matrix: @user:server
│   │   │   ├── XMPP: user@domain
│   │   │   └── IRC: nick@network
│   │   │
│   │   └── identity_links: [LinkEvidence] (see 4.3)
│   │
│   └── ...
│
└── identity_links: [LinkEvidence] (cross-protocol proofs for OWN identities)
```

### 4.2 Local Persistence

```sql
-- SQLite schema for identity store (app sandbox, raw keychain access guarded)

CREATE TABLE personas (
    id              TEXT PRIMARY KEY,      -- UUID v4
    display_name    TEXT NOT NULL DEFAULT 'Default',
    avatar_path     TEXT,
    created_at      INTEGER NOT NULL,      -- Unix timestamp
    is_primary      INTEGER NOT NULL DEFAULT 0,
    seed_derivation INTEGER NOT NULL DEFAULT 0  -- 0 = master seed path 0
);

CREATE TABLE protocol_identities (
    id              TEXT PRIMARY KEY,      -- UUID v4
    persona_id      TEXT NOT NULL REFERENCES personas(id) ON DELETE CASCADE,
    protocol        TEXT NOT NULL,         -- 'nostr' | 'atp' | 'activitypub' | 'matrix' | 'xmpp' | 'irc'
    local_label     TEXT,                  -- user-chosen nickname for this identity
    identifier      TEXT NOT NULL,         -- npub, DID, acct, MXID, JID, nick@net
    display_name    TEXT,                  -- last known display name from protocol
    avatar_url      TEXT,
    connected       INTEGER NOT NULL DEFAULT 0,
    last_sync       INTEGER,
    created_at      INTEGER NOT NULL,
    UNIQUE(protocol, identifier)
);

CREATE TABLE cross_protocol_links (
    id              TEXT PRIMARY KEY,
    persona_id      TEXT NOT NULL REFERENCES personas(id) ON DELETE CASCADE,
    identity_a_id   TEXT NOT NULL REFERENCES protocol_identities(id),
    identity_b_id   TEXT NOT NULL REFERENCES protocol_identities(id),
    link_method     TEXT NOT NULL,         -- 'manual' | 'heuristic' | 'cryptographic' | 'cross_post'
    confidence      REAL NOT NULL,         -- 0.0 to 1.0
    evidence_blob   TEXT,                  -- JSON: proof data (signature, URL, etc.)
    created_at      INTEGER NOT NULL,
    confirmed_by_user INTEGER NOT NULL DEFAULT 0,
    UNIQUE(identity_a_id, identity_b_id)
);

CREATE TABLE contacts (
    id              TEXT PRIMARY KEY,
    display_name    TEXT NOT NULL,
    avatar_url      TEXT,
    notes           TEXT,
    trust_level     INTEGER NOT NULL DEFAULT 0,  -- 0=unknown, 1=observed, 2=cross-signed, 3=manual
    first_seen      INTEGER NOT NULL,
    last_interaction INTEGER,
    is_blocked      INTEGER NOT NULL DEFAULT 0,
    is_muted        INTEGER NOT NULL DEFAULT 0
);

CREATE TABLE contact_protocol_refs (
    id              TEXT PRIMARY KEY,
    contact_id      TEXT NOT NULL REFERENCES contacts(id) ON DELETE CASCADE,
    protocol        TEXT NOT NULL,
    identifier      TEXT NOT NULL,         -- npub, DID, etc.
    last_verified   INTEGER,
    UNIQUE(protocol, identifier)
);

CREATE TABLE identity_link_evidence (
    id              TEXT PRIMARY KEY,
    link_id         TEXT NOT NULL REFERENCES cross_protocol_links(id) ON DELETE CASCADE,
    evidence_type   TEXT NOT NULL,         -- 'signature' | 'dns_record' | 'cross_post' | 'nip05' | 'manual'
    proof_data      TEXT NOT NULL,         -- JSON with type-specific evidence
    verified_at     INTEGER NOT NULL,
    expires_at      INTEGER
);
```

### 4.3 Heuristic Matching Engine

The app continuously (background, daily) attempts to link identities across protocols using the following heuristics:

```text
HEURISTIC 1: Handle / Username Match (Low confidence)
  ──────────────────────────────────────────────────
  If Nostr NIP-05 (user@domain) == Bluesky handle (user.domain) OR
     Nostr display_name == Bluesky display_name:
       → Suggest link with confidence 0.3
       → Requires user confirmation

HEURISTIC 2: DNS Cross-Check (Medium confidence)
  ─────────────────────────────────────────────
  If Nostr NIP-05 domain == Mastodon instance domain:
    → Fetch Nostr domain's .well-known/nostr.json
    → Fetch Mastodon domain's .well-known/webfinger
    → If both reference the same user identifier:
      → Suggest link with confidence 0.6

HEURISTIC 3: Cross-Post Detection (Medium-High confidence)
  ──────────────────────────────────────────────────────
  Look for posts across protocols that contain:
    "My [protocol] is [identifier]"
  → NLP parse of recent posts
  → If bidirectional (A posts about B AND B posts about A):
    → Confidence 0.8
  → Requires user confirmation for trust < manual

HEURISTIC 4: Direct Message / Private Reference (High confidence)
  ─────────────────────────────────────────────────────────────
  If the user has DMed the same person on Nostr and Matrix
  and both conversations reference the same real-world information:
    → Not automated. User can manually link contacts they know
      are the same person.
    → Trust level: "manual"

HEURISTIC 5: Cryptographic Proof (Highest confidence)
  ─────────────────────────────────────────────────
  User explicitly publishes a cross-protocol proof:
    Nostr event signed with nsec:
      "My AT Protocol DID is did:plc:abc... My Matrix ID is @user:server"
    → Any client can verify the Nostr signature
    → Cross-link confidence: 1.0 (deterministic)
    → This is the GOLD STANDARD (see 4.4)
```

### 4.4 Proof-of-Identity Cross-Posting

The user can optionally generate and publish cryptographic identity proofs:

```text
FORMAT: Signed Statement (protocol-native)
─────────────────────────────────────────

Nostr → Attest to other identities:
  Kind: 31400 (kind to be proposed: "identity attestation")
  Content: {
    "attestations": [
      { "protocol": "atp", "identifier": "did:plc:abc123", "verified_at": 1744243200 },
      { "protocol": "matrix", "identifier": "@user:server",
        "device_id": "DEVICEABC", "verified_at": 1744243200 },
      { "protocol": "ap", "identifier": "@user@instance.social" }
    ],
    "version": 1
  }
  Tags: [["p", "<npub>"]]
  → Signed with Nostr nsec
  → Published to Nostr relays

AT Protocol → Attest to other identities:
  Record type: app.liberation.id.attestation (lexicon to be registered)
  {
    "attestations": [
      { "protocol": "nostr", "identifier": "npub1...", "relays": ["wss://relay.damus.io"] },
      { "protocol": "matrix", "identifier": "@user:server" }
    ],
    "signature": "signature over this record with ATP signing key"
  }
  → Published to PDS

Matrix → Attest to other identities:
  Account data event: im.liberation.id.attestation
  {
    "attestations": [
      { "protocol": "nostr", "identifier": "npub1..." },
      { "protocol": "atp", "identifier": "did:plc:abc123" }
    ]
  }
  → Signed with Matrix cross-signing master key
  → Stored in Matrix account data

PRIVACY NOTE:
  All attestations are OPT-IN. Off by default.
  User must explicitly choose to publish each attestation.
  Cross-protocol linking on-device is ALWAYS private;
  attestations are published ONLY when user triggers "Publish identity proof."
```

### 4.5 Identity Linking UX: "The Identity Cloud"

A visual interface showing the user's linked identities:

```text
┌──────────────────────────────────────────────────────────────────────┐
│                     Your Identity Cloud                             │
│                                                                      │
│              ┌─────────┐                                             │
│              │ Primary  │                                             │
│              │ Persona  │ ◄── display name + avatar                  │
│              └────┬────┘                                             │
│                   │                                                  │
│     ══════════════╪═══════════════════════                           │
│     ┌──────────┐  │  ┌──────────┐  ┌──────────┐                     │
│     │ ⚡ Nostr  │──┼──│ 🔷 AT    │  │ ◉ Matrix  │                     │
│     │ npub1...  │  │  │ did:plc  │  │ @user:srv │                     │
│     │ ✅ CONN   │  │  │ ✅ CONN  │  │ ⚠ RECONN  │                     │
│     └──────────┘  │  └──────────┘  └──────────┘                     │
│                   │                                                  │
│     ┌──────────┐  │  ┌──────────┐  ┌──────────┐                     │
│     │ ☁ Mastodon│──┼──│ 💬 XMPP │  │ #️⃣  IRC   │                     │
│     │ @user@ins │  │  │ user@dom│  │ nick@net │                     │
│     │ ✅ CONN   │  │  │ ❌ DISCON│  │ ✅ CONN  │                     │
│     └──────────┘  │  └──────────┘  └──────────┘                     │
│                   │                                                  │
│     ┌──────────────────────────────────────────────────────────────┐ │
│     │  Cross-Protocol Links                                       │ │
│     │  ┌──────────────────────────────────────────────────────┐   │ │
│     │  │ Nostr npub1...  ←── crypto proof ──→  did:plc:abc   │   │ │
│     │  │                    ✓ verified 2026-06-15             │   │ │
│     │  ├──────────────────────────────────────────────────────┤   │ │
│     │  │ Nostr npub1...  ←── heuristic ──→  @user@instance   │   │ │
│     │  │                    ? confirm (tap to confirm)        │   │ │
│     │  └──────────────────────────────────────────────────────┘   │ │
│     └──────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  [Publish Identity Proof]  [Set Primary Persona]  [Manage Recovery]  │
└──────────────────────────────────────────────────────────────────────┘
```

### 4.6 Privacy Guarantees

```text
1. LOCAL-FIRST: All identity linking data lives in the app's SQLite database
   in the sandbox. It NEVER syncs to a server.

2. NO TELEMETRY: No crash reports, analytics, or usage data contains any
   identity information.

3. OPT-IN PUBLICATION: Cross-protocol attestations are NEVER published
   unless the user explicitly taps "Publish Identity Proof."

4. VERIFIABLE BUT PRIVATE: Published attestations are public (they're on
   Nostr relays, PDS, or Matrix server). The user is informed:
   "This proof is public and permanent. Anyone can see it."

5. CONTACT LINKING IS LOCAL: When you link a contact's Nostr npub to their
   Matrix MXID, that link is ONLY stored on your device. Not shared.

6. EXPORT CONTROL: The full identity database can be exported (encrypted)
   and imported on another device. This export contains all local links.
```

---

## Part 5: Key Rotation & Compromise Handling

### 5.1 Rotation Capabilities by Protocol

| Protocol | Key Rotation? | Mechanism | Impact |
|---|---|---|---|
| **Nostr** | NO (native key is identity) | NIP-26 delegation (separate key for signing, master offline) | npub is the identity; can't change without losing followers |
| **AT Protocol** | YES | DID document update (PLC rotation key) | DID stays the same; handle may need re-verification |
| **Matrix** | PARTIAL | Cross-signing + new device login | MXID stays; old messages unreadable unless key backup |
| **ActivityPub** | YES (token refresh) | OAuth2 refresh token flow; revoke old token | Account stays; app re-authenticates |
| **XMPP** | YES | Change password on server | Account stays; OMEMO keys re-generated (trust decisions lost) |
| **IRC** | YES | Change NickServ password | Account stays |

### 5.2 Nostr: NIP-26 Delegation Architecture

Since Nostr native keys cannot be rotated (the npub IS the identity), we implement NIP-26 delegation as the primary mitigation:

```text
                  OFFLINE                         ONLINE
              ┌──────────────┐             ┌──────────────────┐
              │ Master Key    │             │ Delegated Key     │
              │ (seed derived)│             │ (device-local)    │
              │ npub_master   │──delegate──►│ npub_delegate     │
              │              │              │                  │
              │ Used only    │              │ Used for daily   │
              │ for:         │              │ signing:         │
              │ - delegation │              │ - text notes     │
              │ - revocation │              │ - reactions      │
              │ - rare ops   │              │ - DMs            │
              └──────────────┘             └──────────────────┘
                      │                              │
                      │  NIP-26 delegation event:    │
                      │  {kind: 22242,               │
                      │   pubkey: npub_master,        │
                      │   content: {                  │
                      │     delegator: npub_master,   │
                      │     delegate: npub_delegate,  │
                      │     conditions: {             │
                      │       kind_whitelist: [1,7,4],│
                      │       since: 1744243200,      │
                      │       until: 1775779200       │
                      │     }                         │
                      │   },                          │
                      │   sig: sig_master}             │
                      │                              │
                      │  Revocation: publish new      │
                      │  delegation with              │
                      │  until: <past_timestamp>      │
                      └──────────────────────────────┘

Implementation:
  - On device setup: generate delegated keypair (from same seed, separate path)
  - Publish NIP-26 delegation event from master key to delegated key
  - Delegated key stored in keychain with biometric (no wrapping needed;
    it's the daily-use key)
  - Master key requires biometric + explicit "Advanced Operation" flow
  - Daily signing operations use delegated key only
  - If delegated key is compromised: generate new delegate, publish new
    delegation, revoke old one
  - If master key is compromised: we are in a critical state
```

**NIP-26 delegation conditions:**

```json
{
  "delegator": "<npub_master>",
  "delegate": "<npub_delegate>",
  "conditions": {
    "kind_whitelist": [1, 3, 4, 5, 6, 7, 20, 9734, 9802, 30023],
    "pubkey_whitelist": ["<npub_delegate>"],
    "since": 1744243200,
    "until": 1775779200
  }
}
```

Non-delegated operations (require master key):
- Publishing new delegation events
- Publishing delegation revocation
- Kind 0 (metadata) — optional, can be delegated
- Kind 10000 (mute list) — optional, can be delegated
- Kind 22242 (auth) — MUST use master key for authentication

### 5.3 Compromise Detection

#### 5.3.1 Signs of Compromise

```text
SUSPICIOUS ACTIVITY (detected by the app):
  ─────────────────────────────────────────
  - New device on Matrix session list (check daily)
  - Unexpected Nostr events with user's npub that app didn't create
    (requires running a verification scan against known events)
  - AT Protocol: unexpected PDS event log entries
  - Mastodon: "login from new device" email notification
    (app could prompt: "Did you just log in from [location]?")
  - Unexpected key changes on Matrix cross-signing

USER-REPORTED COMPROMISE:
  ─────────────────────────
  - "People are receiving messages I didn't send"
  - "My posts I didn't write are appearing"
  - "Someone changed my profile"
  - "I can't log in"
  - "My backups are gone"
```

#### 5.3.2 Automated Checks (Background, Daily)

```text
1. MATRIX SESSION AUDIT:
   → Query /_matrix/client/v3/keys/changes with old device list
   → If unknown devices added in past 24h: alert user

2. NOSTR EVENT VERIFICATION:
   → Query user's events from relays for past 24h
   → Cross-reference with locally-signed event hashes
   → If events exist that app didn't sign: HIGH alert

3. ATP PDS LOG:
   → Query PDS audit log (if available)
   → Check for unexpected DID document changes

4. MASTODON TOKEN AUDIT:
   → Query /api/v1/apps/verify_credentials (checks token validity)
   → Check last login time if available via API
```

### 5.4 Compromise Response Flow

```text
COMPROMISE DETECTED ──────────────────────────────────────────────────
  │
  ▼
STEP 1: LOCKDOWN
  ├─ Disconnect all protocols (revoke tokens where possible)
  ├─ Clear in-memory key cache
  ├─ Require biometric re-authentication for any further action
  └─ Show warning screen with recommended actions

STEP 2: INVESTIGATE
  └─ "What happened?" wizard:
      ├─ "Did someone get your device?" → go to Device Theft
      ├─ "Did someone get your seed phrase?" → go to Seed Compromise
      ├─ "Did you enter your credentials on a fake site?" → go to Phishing
      ├─ "Did you see suspicious activity?" → go to Suspicious Activity
      └─ "I'm not sure" → show all possible scenarios

STEP 3: EXECUTE PROTOCOL-SPECIFIC RESPONSE

  ┌─ NOSTR ──────────────────────────────────────────────────────────
  │  Scenario A: Delegated key compromised
  │    1. Generate new delegated keypair (from seed, new index)
  │    2. Publish new NIP-26 delegation event (from master key)
  │    3. Revoke old delegation (publish with until: <past>)
  │    4. Update app to use new delegated key
  │
  │  Scenario B: Master key compromised (SEED COMPROMISE)
  │    1. Generate new seed
  │    2. Derive new Nostr master key
  │    3. Publish NIP-01 "Key migration" event (kind: unspecified,
  │       proposed: kind 30001?) pointing to new npub
  │    4. Followers must manually re-follow new npub
  │    ⚠ This is equivalent to losing your Nostr identity
  │
  └────────────────────────────────────────────────────────────────

  ┌─ AT PROTOCOL ────────────────────────────────────────────────────
  │  Scenario A: Signing key compromised, recovery key safe
  │    1. Use PLC rotation key to update DID document
  │    2. Set new signing key (from seed, new derivation index)
  │    3. Re-verify handle
  │    4. Old key revoked at DID document level
  │
  │  Scenario B: All keys compromised
  │    1. Use PLC emergency recovery (7-day delay, if available)
  │    2. Or: create new DID (identity lost)
  │
  └────────────────────────────────────────────────────────────────

  ┌─ MATRIX ─────────────────────────────────────────────────────────
  │  Scenario A: Device key compromised
  │    1. Login from the device (if still accessible)
  │    2. Go to device management → sign out all other devices
  │    3. Verify new device with cross-signing
  │    4. Reset key backup (if old backup was compromised)
  │        → Matrix spec: delete old backup, create new one
  │        → Tell contacts: "I've reset my keys. Please re-verify."
  │
  │  Scenario B: Cross-signing master key compromised
  │    1. Use SSSS to restore cross-signing keys from backup
  │    2. If SSSS also compromised:
  │       a. Reset cross-signing (Matrix spec MSC 2734)
  │       b. Tell contacts to re-verify
  │       c. Old messages with old keys are lost
  │
  └────────────────────────────────────────────────────────────────

  ┌─ ACTIVITYPUB (MASTODON) / XMPP / IRC ──────────────────────────
  │  These are password/token based:
  │    1. Revoke access token (Mastodon: "authorized apps")
  │    2. Change password on server
  │    3. Generate new token from app (re-authenticate)
  │    4. For XMPP with OMEMO: generate new OMEMO keys,
  │       re-establish trust with contacts
  │
  └────────────────────────────────────────────────────────────────

STEP 4: DAMAGE ASSESSMENT
  ├─ Check if any cross-protocol identity proofs were published
  │  (an attacker could have linked a fake identity)
  ├─ If yes: revoke proofs by publishing nullifying event or
  │   waiting for attestation expiry
  └─ Ask user: "Do you want to notify your contacts?"
      └─ Option: Broadcast a "compromised identity" warning
         (Nostr: signed event; Matrix: broadcast message to rooms)

STEP 5: RECOVERY
  ├─ If seed was compromised:
  │   ├─ Generate new seed (requires entering old seed first to prove ownership)
  │   ├─ Derive new protocol keys
  │   ├─ Social recovery: generate new SLIP-0039 shares
  │   └─ Cloud backup: create new encrypted backup
  │
  ├─ If seed was NOT compromised:
  │   └─ Just rotate the affected protocol keys
  │
  └─ "Your identity has been restored. Review your linked identities."

STEP 6: POST-MORTEM
  └─ "How did this happen?" survey (optional, helps improve security)
  └─ "Enable these additional protections?"
      ├─ Require biometric for ALL posts (not just key ops)
      ├─ Enable NIP-26 delegation if not already
      ├─ Enable social recovery (if not configured)
      └─ Paper wallet generation (air-gapped backup)
```

---

## Part 6: Implementation Specifications

### 6.1 Rust Crate Structure

```text
libs/identity/
├── Cargo.toml                          # crate definition
├── src/
│   ├── lib.rs                          # crate root, re-exports
│   ├── seed.rs                         # BIP-39 seed generation & derivation
│   ├── derivation.rs                   # derivation engine (BIP-32 / SLIP-10)
│   ├── wrapping.rs                     # ECIES key wrapping / unwrapping
│   ├── protocol_keys.rs                # protocol-specific key derivations
│   │
│   ├── nostr.rs                        # Nostr key ops
│   ├── atp.rs                          # AT Protocol key ops
│   ├── matrix.rs                       # Matrix key ops
│   ├── xmpp.rs                         # XMPP OMEMO key ops
│   ├── irc.rs                          # IRC password vault ops
│   ├── activitypub.rs                  # Mastodon OAuth token ops (thin)
│   │
│   ├── ssss.rs                         # Matrix SSSS encryption/decryption
│   ├── sharding.rs                     # SLIP-0039 Shamir's Secret Sharing
│   ├── cloud_backup.rs                 # Tier 3 encrypted backup format
│   │
│   ├── identity_graph.rs               # local identity linking engine
│   ├── proof.rs                        # cross-protocol attestation signatures
│   ├── heuristic.rs                    # heuristic matching
│   │
│   ├── compromise.rs                   # compromise detection & response
│   ├── rotation.rs                     # key rotation logic
│   │
│   ├── ffi/                            # foreign function interface boundary
│   │   ├── mod.rs
│   │   ├── keychain.rs                 # native keychain access (via uniffi callback)
│   │   ├── biometric.rs                # biometric gate (via uniffi callback)
│   │   └── platform.rs                 # platform detection & feature flags
│   │
│   └── util/
│       ├── crypto.rs                   # constant-time utils, secure memory
│       ├── bech32.rs                   # bech32 encoding (npub, nsec)
│       └── cbor.rs                     # CBOR serialization helpers
│
├── tests/
│   ├── unit/
│   │   ├── derivation_test.rs          # test all derivation paths
│   │   ├── wrapping_test.rs            # test ECIES wrap/unwrap roundtrip
│   │   ├── nostr_test.rs               # NIP-06 compliance
│   │   ├── sharding_test.rs            # SLIP-0039 compliance
│   │   ├── ssss_test.rs                # Matrix SSSS spec compliance
│   │   ├── cloud_backup_test.rs        # backup format roundtrip
│   │   └── proof_test.rs               # attestation format & verification
│   │
│   ├── integration/
│   │   ├── seed_to_key_test.rs         # full seed → all protocol keys
│   │   ├── recovery_test.rs            # mnemonic → full recovery scenario
│   │   ├── social_recovery_test.rs     # SLIP-0039 split → collect → reconstruct
│   │   └── compromise_test.rs          # rotate all keys scenario
│   │
│   └── fixtures/
│       ├── test_seed_24.txt            # deterministic test seed words
│       ├── test_seed_12.txt            # 12-word test seed
│       └── test_vector_nostr.json      # NIP-06 test vectors
│
├── benches/
│   ├── derivation_bench.rs             # key derivation throughput
│   ├── wrapping_bench.rs               # ECIES wrap/unwrap performance
│   └── sharding_bench.rs               # SSS split/reconstruct performance
│
└── uniffi.toml                         # UniFFI config for FFI bindings
```

### 6.2 Cargo.toml

```toml
[package]
name = "liberation-identity"
version = "0.1.0"
edition = "2021"
license = "AGPL-3.0-only"
description = "Liberation Client identity & keychain management core"

[features]
default = ["std"]
std = []
# Enable for enclave simulator testing (no real hardware required)
enclave_simulator = []

[dependencies]
# BIP-39 / BIP-32 / SLIP-10
bip39 = { version = "2", features = ["std"] }
slip10 = "0.3"
ed25519-dalek = { version = "2", features = ["hazmat"] }
x25519-dalek = "2"
# secp256k1
secp256k1 = { version = "27", features = ["hashes", "rand", "serde"] }
# P-256 (NIST)
p256 = { version = "0.13", features = ["ecdsa", "pem"] }

# Symmetric / AEAD
aes-gcm = "0.10"
hkdf = "0.12"
rand = "0.8"
zeroize = { version = "1", features = ["zeroize_derive"] }
# Memory protection
memsec = "0.7"

# Serialization
serde = { version = "1", features = ["derive"] }
serde_json = "1"
ciborium = "0.2"              # CBOR
bs58 = "0.5"                   # base58 for various protocol encodings
bech32 = { version = "0.10", features = ["std"] }

# SSS / SLIP-0039
vsss-rs = { version = "0.2", features = ["std", "serde"] }  # SLIP-0039

# Argon2id
argon2 = { version = "0.5", features = ["std", "simd"] }

# UniFFI (for FFI to Swift/Kotlin)
uniffi = { version = "0.28", features = ["cli"] }

# Protocol-specific crypto
matrix-sdk-crypto = { version = "0.9", optional = true, default-features = false, features = ["native-tls"] }
# Nostr-specific (use our own, not full nostr lib)
# AT Protocol (our own light implementation)

# Async runtime for protocol calls
tokio = { version = "1", features = ["rt-multi-thread", "macros"], optional = true }

[dev-dependencies]
hex = "0.4"
hex-literal = "0.4"
criterion = "0.5"
proptest = "1"
quickcheck = "1"

[build-dependencies]
uniffi = { version = "0.28", features = ["build"] }

[lib]
crate-type = ["lib", "staticlib", "cdylib"]
name = "liberation_identity"
```

### 6.3 Core Traits / Interfaces

```rust
// ──────────────────────────────────────────────
// lib.rs — Crate root
// ──────────────────────────────────────────────

pub mod seed;
pub mod derivation;
pub mod wrapping;
pub mod protocol_keys;
pub mod nostr;
pub mod atp;
pub mod matrix;
pub mod xmpp;
pub mod irc;
pub mod ssss;
pub mod sharding;
pub mod cloud_backup;
pub mod identity_graph;
pub mod proof;
pub mod heuristic;
pub mod compromise;
pub mod rotation;
pub mod ffi;
pub mod util;

use zeroize::Zeroize;

/// Opaque handle to a derived protocol key.
/// The actual key material lives in the Secure Enclave wrapper.
pub struct KeyHandle {
    pub id: String,
    pub protocol: ProtocolKind,
    pub wrapped_blob: Vec<u8>,    // ECIES-wrapped key material (CBOR)
    pub public_key: Vec<u8>,      // cached public key (derived from private)
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum ProtocolKind {
    Nostr,
    AtProtocol,
    ActivityPub,
    Matrix,
    Xmpp,
    Irc,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct KeychainContext {
    /// Platform callback for keychain read
    pub keychain_read: fn(tag: &str) -> Result<Vec<u8>, KeychainError>,
    /// Platform callback for keychain write
    pub keychain_write: fn(tag: &str, data: &[u8]) -> Result<(), KeychainError>,
    /// Platform callback for biometric authentication
    pub biometric_authenticate: fn(reason: &str) -> Result<bool, BiometricError>,
    /// Platform callback for ECIES decrypt using SE-protected KWK
    pub se_unwrap: fn(ciphertext: &[u8]) -> Result<Vec<u8>, KeychainError>,
    /// Platform callback for ECIES encrypt using SE-protected KWK
    pub se_wrap: fn(plaintext: &[u8]) -> Result<Vec<u8>, KeychainError>,
}

// ──────────────────────────────────────────────
// seed.rs
// ──────────────────────────────────────────────

pub struct MasterSeed {
    entropy: [u8; 32],        // 256 bits, zeroized on drop
    mnemonic: bip39::Mnemonic,
    seed: [u8; 64],           // BIP-39 seed (PBKDF2 output), zeroized on drop
}

impl MasterSeed {
    /// Generate a new master seed from platform TRNG
    pub fn generate() -> Result<Self, SeedError> { ... }

    /// Restore from BIP-39 mnemonic phrase
    pub fn from_mnemonic(phrase: &str) -> Result<Self, SeedError> { ... }

    /// Restore from entropy bytes (for recovery from SLIP-0039)
    pub fn from_entropy(entropy: &[u8; 32]) -> Result<Self, SeedError> { ... }

    /// Get the mnemonic phrase (display only, biometric-gated)
    pub fn mnemonic(&self) -> &str { ... }

    /// Get the raw seed bytes (for derivation engine)
    pub fn seed_bytes(&self) -> &[u8; 64] { ... }

    /// Derive a key at the given BIP-32 path
    pub fn derive_key(&self, path: &DerivationPath) -> Result<KeyHandle, DerivationError> { ... }
}

impl Drop for MasterSeed {
    fn drop(&mut self) {
        self.entropy.zeroize();
        self.seed.zeroize();
    }
}

// ──────────────────────────────────────────────
// derivation.rs
// ──────────────────────────────────────────────

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct DerivationPath {
    pub purpose: u32,
    pub coin_type: u32,
    pub account: u32,
    pub change: u32,
    pub index: u32,
}

pub trait KeyDerivation {
    /// Derive a private key for the given curve at the given path
    fn derive(
        &self,
        seed: &MasterSeed,
        path: &DerivationPath,
    ) -> Result<(Vec<u8>, Vec<u8>), DerivationError>;  // (private, public)

    /// Derive just the public key (for read-only wallets)
    fn derive_public(
        &self,
        seed: &MasterSeed,
        path: &DerivationPath,
    ) -> Result<Vec<u8>, DerivationError>;
}

pub struct Bip32Derivation;   // secp256k1 via BIP-32 CKD
pub struct Slip10Derivation; // Ed25519 / Curve25519 via SLIP-0010
pub struct P256Derivation;   // P-256 via FIPS 186-4 B.4.1

// ──────────────────────────────────────────────
// wrapping.rs
// ──────────────────────────────────────────────

pub struct WrappedKey {
    pub version: u8,
    pub ephem_pk: (Vec<u8>, Vec<u8>),    // x, y
    pub nonce: [u8; 12],
    pub ciphertext: Vec<u8>,
    pub tag: [u8; 16],
    pub protocol_tag: String,
}

pub trait KeyWrapper {
    fn wrap(
        &self,
        context: &KeychainContext,
        plaintext: &[u8],
        protocol_tag: &str,
    ) -> Result<Vec<u8>, WrappingError>;

    fn unwrap(
        &self,
        context: &KeychainContext,
        wrapped: &[u8],
        protocol_tag: &str,
    ) -> Result<Vec<u8>, WrappingError>;
}

// ──────────────────────────────────────────────
// protocol_keys.rs
// ──────────────────────────────────────────────

/// Trait implemented by each protocol's key module
#[async_trait]
pub trait ProtocolKeyManager: Send + Sync {
    /// The protocol this manager handles
    fn protocol(&self) -> ProtocolKind;

    /// Derive the root keypair for this protocol from the master seed.
    /// Returns a KeyHandle that can be used for signing.
    fn derive_root_key(
        &self,
        seed: &MasterSeed,
        context: &KeychainContext,
    ) -> Result<KeyHandle, KeyError>;

    /// Sign data with this protocol's signing key
    fn sign(
        &self,
        handle: &KeyHandle,
        context: &KeychainContext,
        data: &[u8],
    ) -> Result<Vec<u8>, KeyError>;

    /// Verify a signature with a public key
    fn verify(
        &self,
        public_key: &[u8],
        data: &[u8],
        signature: &[u8],
    ) -> Result<bool, KeyError>;

    /// Export public key in protocol-native format
    fn export_public(&self, handle: &KeyHandle) -> Result<String, KeyError>;

    /// Export private key in protocol-native format (warns user first)
    fn export_private(&self, handle: &KeyHandle, context: &KeychainContext)
        -> Result<String, KeyError>;
}

// ──────────────────────────────────────────────
// nostr.rs
// ──────────────────────────────────────────────

pub struct NostrKeyManager {
    derivation_path: DerivationPath,
}

impl NostrKeyManager {
    pub const DERIVATION_PATH: DerivationPath = DerivationPath {
        purpose: 44,
        coin_type: 1237,
        account: 0,
        change: 0,
        index: 0,
    };

    pub const DELEGATED_PATH: DerivationPath = DerivationPath {
        purpose: 44,
        coin_type: 1237,
        account: 0,
        change: 0,
        index: 1,
    };

    /// Generate a NIP-26 delegation from master key to delegated key
    pub fn create_delegation(
        &self,
        master_handle: &KeyHandle,
        delegate_handle: &KeyHandle,
        context: &KeychainContext,
        conditions: &DelegationConditions,
    ) -> Result<Nip26Delegation, KeyError> { ... }

    /// Publish the delegation event to Nostr relays
    pub async fn publish_delegation(
        &self,
        delegation: &Nip26Delegation,
        relays: &[String],
    ) -> Result<(), NostrError> { ... }
}

#[async_trait]
impl ProtocolKeyManager for NostrKeyManager { ... }

// ──────────────────────────────────────────────
// matrix.rs
// ──────────────────────────────────────────────

pub struct MatrixKeyManager;

impl MatrixKeyManager {
    /// Derive Matrix cross-signing keys from seed
    pub fn derive_cross_signing_keys(
        seed: &MasterSeed,
        context: &KeychainContext,
    ) -> Result<MatrixCrossSigningKeys, KeyError> { ... }

    /// Derive device-specific Olm keys (from seed-derived seed for the device)
    pub fn derive_device_keys(
        device_id: &str,
        context: &KeychainContext,
    ) -> Result<OlmKeys, KeyError> { ... }

    /// Store keys in Matrix SSSS
    pub async fn store_in_ssss(
        keys: &MatrixCrossSigningKeys,
        ssss_key: &[u8; 32],
        client: &MatrixClient,
    ) -> Result<(), MatrixError> { ... }
}

// ──────────────────────────────────────────────
// sharding.rs — SLIP-0039 Social Recovery
// ──────────────────────────────────────────────

pub struct Slip0039Sharding;

impl Slip0039Sharding {
    /// Split master entropy into N shares with threshold T
    pub fn split(
        entropy: &[u8; 32],
        threshold: u8,     // e.g., 3
        total: u8,         // e.g., 5
        passphrase: Option<&str>,
    ) -> Result<Vec<String>, ShardingError> { ... }

    /// Reconstruct master entropy from shares
    pub fn reconstruct(
        shares: &[String],
        passphrase: Option<&str>,
    ) -> Result<[u8; 32], ShardingError> { ... }
}

// ──────────────────────────────────────────────
// cloud_backup.rs — Tier 3 Backup
// ──────────────────────────────────────────────

pub struct CloudBackup;

impl CloudBackup {
    /// Encrypt master seed with user passphrase
    pub fn encrypt(
        seed: &[u8; 64],
        passphrase: &str,
    ) -> Result<Vec<u8>, BackupError> { ... }  // returns CBOR bytes

    /// Decrypt backup file
    pub fn decrypt(
        data: &[u8],
        passphrase: &str,
    ) -> Result<[u8; 64], BackupError> { ... }

    /// Estimate passphrase strength (zxcvbn-like without the full lib)
    pub fn estimate_strength(passphrase: &str) -> PasswordStrength { ... }
}

// ──────────────────────────────────────────────
// identity_graph.rs — Cross-Protocol Identity Linking
// ──────────────────────────────────────────────

pub struct IdentityGraph {
    db: Connection,    // SQLite connection (managed by app, passed in)
}

impl IdentityGraph {
    pub fn new(db: Connection) -> Self { ... }

    /// Add a protocol identity
    pub fn add_identity(
        &mut self,
        persona_id: &str,
        protocol: ProtocolKind,
        identifier: &str,
        display_name: Option<&str>,
    ) -> Result<String, GraphError> { ... }

    /// Create a cross-protocol link
    pub fn link_identities(
        &mut self,
        persona_id: &str,
        identity_a: &str,
        identity_b: &str,
        method: &str,
        confidence: f64,
        evidence: Option<&str>,
    ) -> Result<(), GraphError> { ... }

    /// Run heuristic matching for a persona
    pub fn run_heuristics(
        &self,
        persona_id: &str,
    ) -> Vec<LinkSuggestion> { ... }

    /// Get all identity proofs to publish for a persona
    pub fn get_unsigned_proofs(
        &self,
        persona_id: &str,
    ) -> Vec<UnsignedAttestation> { ... }
}
```

### 6.4 FFI Boundary

The FFI boundary separates Rust cryptographic operations from platform-specific keychain and biometric operations.

```text
┌──────────────────────────────────────────────────────────────────────┐
│                    Native Layer (Rust)                               │
│                                                                      │
│  Seed Generation        ✓ (pure Rust — bip39 crate)                  │
│  BIP-32/SLIP-10 Derivation ✓ (pure Rust, no platform calls)          │
│  ECIES Wrapping/Unwrapping ✓ (pure Rust, but calls SE for ECDH)      │
│  SSS Split/Reconstruct  ✓ (pure Rust — vsss-rs crate)              │
│  AEAD Encrypt/Decrypt   ✓ (pure Rust — aes-gcm crate)                │
│  Argon2id KDF           ✓ (pure Rust — argon2 crate)                 │
│  Signature Operations   ✓ (pure Rust — dalek, secp256k1 crates)     │
│  CBOR/JSON Serialization ✓ (pure Rust)                              │
│  Identity Graph Logic   ✓ (pure Rust — rusqlite)                     │
│  Heuristic Matching     ✓ (pure Rust)                                │
│  Key Verification       ✓ (pure Rust)                                │
│                                                                      │
└──────────────────────────┬───────────────────────────────────────────┘
                           │ FFI (UniFFI)
                           ▼
┌──────────────────────────────────────────────────────────────────────┐
│                    Platform Layer (Swift / Kotlin)                   │
│                                                                      │
│  TRNG Entropy Source      → iOS: `SecRandomCopyBytes`                │
│                             Android: `SecureRandom`                  │
│                                                                      │
│  Secure Enclave ECDH      → iOS: `SecKeyCopyKeyExchangeResult`       │
│                             Android: `KeyAgreement` w/ StrongBox key │
│                                                                      │
│  Biometric Authentication → iOS: `LAContext.evaluatePolicy`          │
│                             Android: `BiometricPrompt`               │
│                                                                      │
│  Keychain Read/Write      → iOS: `SecItemCopyMatching` / `SecItemAdd`
│                             Android: `EncryptedSharedPreferences`    │
│                                                                      │
│  Platform TRNG binding    → Called from Rust during seed gen         │
│  SE ECDH binding           → Called from Rust during wrap/unwrap     │
│  Biometric binding         → Called from Rust during key operations  │
│                                                                      │
│  UI Layer:                                                           │
│    - Seed display & verification screens                             │
│    - Biometric prompts                                               │
│    - Device-to-device pairing for social recovery                    │
│    - Recovery flows (mnemonic entry, cloud backup download)          │
│    - Identity Cloud display                                          │
│    - Compromise alerts & recovery wizard                             │
└──────────────────────────────────────────────────────────────────────┘
```

#### 6.4.1 UniFFI Interface Definition

```text
// liberation_identity.udl — UniFFI definitions

namespace liberation_identity {
  // Seed operations
  MasterSeed generate_seed();
  MasterSeed restore_from_mnemonic(string phrase);
  string get_mnemonic(MasterSeed seed);

  // Recovery
  [Throws=BackupError]
  sequence<u8> encrypt_backup(MasterSeed seed, string passphrase);
  [Throws=BackupError]
  MasterSeed decrypt_backup(sequence<u8> encrypted, string passphrase);

  // Sharding
  [Throws=ShardingError]
  sequence<string> split_entropy(MasterSeed seed, u8 threshold, u8 total);
  [Throws=ShardingError]
  MasterSeed reconstruct_from_shares(sequence<string> shares);

  // Key operations
  [Throws=KeyError]
  KeyHandle derive_nostr_key(MasterSeed seed);
  [Throws=KeyError]
  KeyHandle derive_atp_key(MasterSeed seed);
  [Throws=KeyError]
  KeyHandle derive_matrix_cross_signing_keys(MasterSeed seed);
  [Throws=KeyError]
  KeyHandle derive_xmpp_omemo_key(MasterSeed seed);

  [Throws=KeyError]
  sequence<u8> sign(KeyHandle handle, sequence<u8> data);
  [Throws=KeyError]
  boolean verify(sequence<u8> public_key, sequence<u8> data, sequence<u8> signature);
  string export_public_key(KeyHandle handle);
  [Throws=KeyError]
  string export_private_key(KeyHandle handle);  // biometric required

  // Identity graph
  [Throws=GraphError]
  void link_identities(string persona_id, string id_a, string id_b, string method);
  sequence<LinkSuggestion> run_heuristics(string persona_id);
  sequence<UnsignedAttestation> get_unsigned_proofs(string persona_id);
};

dictionary KeyHandle {
  string id;
  ProtocolKind protocol;
  sequence<u8> public_key;
};

enum ProtocolKind { "Nostr", "AtProtocol", "ActivityPub", "Matrix", "Xmpp", "Irc" };

dictionary LinkSuggestion {
  string identity_a_id;
  string identity_b_id;
  string method;
  f64 confidence;
};

dictionary UnsignedAttestation {
  string source_protocol;
  string source_identifier;
  sequence<AttestedIdentity> targets;
};

dictionary AttestedIdentity {
  string protocol;
  string identifier;
};

[Error]
enum KeyError {
  "DerivationFailed",
  "SigningFailed",
  "VerificationFailed",
  "BiometricRequired",
  "KeyNotFound",
  "KeychainWriteFailed",
};

[Error]
enum BackupError {
  "EncryptionFailed",
  "DecryptionFailed",
  "InvalidFormat",
  "WrongPassphrase",
};
```

#### 6.4.2 Operations That MUST Stay in Rust

```text
CRITICAL: NEVER move to platform layer:
  - BIP-39 mnemonic generation & entropy management
  - BIP-32 / SLIP-10 key derivation (accepts seed, produces keys)
  - All raw cryptographic operations (signing, hashing, AEAD)
  - Argon2id key derivation
  - SSS split & reconstruction
  - Identity graph logic & heuristic matching
  - Backup format parsing & validation
  - Cross-protocol proof generation

OPERATIONS THAT CALL PLATFORM (via FFI callback):
  - Keychain read/write (platform handles persistence)
  - Biometric authentication (platform handles UI)
  - SE-ECDH key agreement (platform wraps SE access)
  - TRNG read (SE/StrongBox provides hardware RNG; Rust consumes)

WHY:
  - Moving crypto logic to Swift/Kotlin introduces risk of
    implementation divergence, timing leaks, and memory unsafety.
  - Rust's zeroize, constant-time comparison, and memory safety
    are essential for key management.
  - Platform layer is large and complex; minimizing the TCB
    (Trusted Computing Base) in platform code reduces audit surface.
```

### 6.5 Testing Strategy

#### 6.5.1 Unit Tests

```text
COVERAGE TARGET: 95%+ of cryptographic code paths.

  derivation_test.rs:
    ✓ BIP-32 secp256k1 derivation matches test vectors (BIP-32 spec)
    ✓ SLIP-0010 Ed25519 derivation matches test vectors (SLIP-10 spec)
    ✓ SLIP-0010 Curve25519 derivation matches test vectors
    ✓ P-256 FIPS 186-4 B.4.1 rejection sampling converges within 255 iterations
    ✓ All 6 protocol paths produce keys of correct type and length
    ✓ Hardened derivation protects master key (derive child, verify master safe)
    ✓ Invalid path rejection (out-of-range indices)

  wrapping_test.rs:
    ✓ ECIES wrap/unwrap roundtrip for 32-byte key
    ✓ ECIES wrap/unwrap roundtrip for 64-byte seed
    ✓ Tampered ciphertext → AEAD authentication failure
    ✓ Nonce reuse detection (nonce is random)
    ✓ Monkey testing: 10,000 random keys → all roundtrip correctly

  nostr_test.rs:
    ✓ NIP-06 test vectors: seed → nsec → npub
    ✓ NIP-06: 12-word seed compliance
    ✓ NIP-06: 24-word seed compliance
    ✓ Schnorr signature generation & verification (BIP-340)
    ✓ NIP-26 delegation event creation & verification
    ✓ NIP-26 delegation conditions enforcement (kind whitelist)
    ✓ NIP-26 revocation

  atp_test.rs:
    ✓ Signing key derivation matches AT Protocol spec
    ✓ DID key encoding (did:key)
    ✓ PLC operation signing format

  matrix_test.rs:
    ✓ Cross-signing key derivation matches Matrix spec
    ✓ SSSS key derivation
    ✓ Olm/Megolm key generation (if matrix-sdk-crypto feature enabled)
    ✓ Device key format per Matrix spec

  ssss_test.rs:
    ✓ AES-HMAC-SHA2 encrypt/decrypt roundtrip
    ✓ Matrix account data event format

  sharding_test.rs:
    ✓ SLIP-0039 test vectors (from spec)
    ✓ 3-of-5: any 3 shares reconstruct correctly
    ✓ 3-of-5: 2 shares fail to reconstruct
    ✓ Invalid share rejection
    ✓ Passphrase-protected shares

  cloud_backup_test.rs:
    ✓ Argon2id + AES-GCM roundtrip
    ✓ Wrong passphrase → decryption failure
    ✓ Tampered backup → AEAD failure
    ✓ Version 1 format backward compatibility

  proof_test.rs:
    ✓ Nostr→ATP attestation format
    ✓ ATP→Nostr attestation format
    ✓ Matrix→Nostr attestation format
    ✓ Signature verification across protocols
    ✓ Private key → public key derivation consistency
```

#### 6.5.2 Integration Tests

```text
  seed_to_key_test.rs:
    ✓ Full flow: generate seed → derive 6 protocol keys → all usable
    ✓ Determinism: same seed → same keys every time
    ✓ Cross-platform: seed generated on iOS test → reconstructed on Android test

  recovery_test.rs:
    ✓ Full Tier 1 flow: generate → backup mnemonic → wipe → enter mnemonic
      → reconstruct all keys → match original
    ✓ Full Tier 3 flow: generate → encrypt with passphrase → wipe
      → decrypt with passphrase → reconstruct → match original
    ✓ Full Tier 2 flow: generate → SSS split (3-of-5) → wipe
      → collect 3 shares → reconstruct → match original

  social_recovery_test.rs:
    ✓ Guardian enrollment simulation: generate shares, encrypt for guardian,
      guardian stores, guardian returns share
    ✓ 3-of-5 with 5 different guardians
    ✓ Collusion resistance with 2 corrupt guardians
    ✓ Long-term storage: share format stability across versions

  compromise_test.rs:
    ✓ Nostr delegated key compromise → generate new delegate → revoke old
    ✓ AT Protocol key rotation flow
    ✓ Scenario: seed compromise → generate new seed → new protocol identities
    ✓ Recovery from cloud backup after full device wipe
```

#### 6.5.3 Enclave Simulator Tests

```rust
// When compiled with --features enclave_simulator:
// Replace SE/StrongBox calls with software implementations for CI testing.

#[cfg(feature = "enclave_simulator")]
mod sim {
    pub struct SimulatedEnclave {
        wrapping_key: p256::SecretKey,
        trng: OsRng,
    }

    impl SimulatedEnclave {
        pub fn new() -> Self { ... }
        pub fn ecdh_agreement(&self, ephemeral_pk: &p256::PublicKey) -> Result<SharedSecret, ...> { ... }
        pub fn random_bytes(&mut self, buf: &mut [u8]) { ... }
    }
}
```

#### 6.5.4 Penetration Testing

```text
BEFORE 1.0 RELEASE, commission:

  1. THIRD-PARTY AUDIT:
     - Full source code review of libs/identity/
     - Focus: constant-time operations, memory zeroization, side channels
     - Must verify all 6 derivation paths are cryptographically sound
     - Must verify ECIES wrapping is secure (no KDF reuse, no padding oracle)

  2. RED TEAM EXERCISE:
     - Attempt to extract key material from app sandbox (privileged access)
     - Attempt to bypass biometric gate (presentation attack, replay)
     - Attempt to recover seed from keychain backup (iTunes backup decryption)
     - Attempt to downgrade derivation paths (algorithm confusion)

  3. FUZZ TESTING:
     - Fuzz BIP-39 mnemonic parser with 1M invalid phrases (no crashes)
     - Fuzz CBOR parser for wrapped keys
     - Fuzz SSS share parser (malformed shares must not panic)
     - Fuzz cloud backup format parser

  4. COMPLIANCE TESTING:
     - NIP-06 test vector compliance
     - SLIP-0039 test vector compliance
     - BIP-32 test vector compliance
     - BIP-39 test vector compliance (all languages available)
```

### 6.6 Security Checklist (Pre-Release)

```text
□ All private key material zeroized on drop (zeroize crate confirmed)
□ All stack buffers for secrets using ::zeroize() on return
□ heap allocations for secrets using mlprotect() or mlock() where possible
□ No secret material in CoreData, UserDefaults, or SharedPreferences
□ Keychain items tagged ThisDeviceOnly for non-transferable keys
□ Biometric required for all signing key operations (cached 60s max)
□ Biometric required for seed display (NO caching)
□ No timing side-channels in comparison operations (constant_time_eq)
□ All protocol signature formats verified against spec test vectors
□ NIP-26 delegation conditions properly enforced by verifier
□ SSS share distribution uses end-to-end encryption (guardian pubkey)
□ Cloud backup uses Argon2id (time=3, mem=64MB)
□ Passphrase strength estimator flags weak passphrases
□ Remote wipe capability: clear all keys from keychain
□ Rate limiting on biometric attempts (iOS handles this natively)
□ Screen recording / screenshot blocking on seed display screen
□ Clipboard cleared after seed copy (or warn user)
□ No sensitive data in crash logs or analytics
□ Cross-protocol attestation publishing is OPT-IN with confirmation
```

---

## Appendix: References & Rationale

### A.1 Standards Referenced

| Standard | Use | Version |
|---|---|---|
| BIP-32 | Hierarchical Deterministic Wallets | 2012 |
| BIP-39 | Mnemonic code for generating deterministic keys | 2013 |
| BIP-43 | Purpose Field for Deterministic Wallets | 2014 |
| BIP-44 | Multi-Account Hierarchy for Deterministic Wallets | 2014 |
| SLIP-0010 | Universal key derivation for Ed25519/Curve25519 | 2016 |
| SLIP-0039 | Shamir's Secret Sharing for BIP-39 | 2019 |
| SLIP-0044 | Registered coin types for BIP-44 | 2017 |
| NIP-06 | Basic key derivation from mnemonic seed | Nostr |
| NIP-26 | Delegated Event Signing | Nostr |
| NIP-44 | Encrypted Payloads (Versioned) | Nostr |
| FIPS 186-4 | Digital Signature Standard (P-256 key gen B.4.1) | 2013 |
| OWASP Argon2 | Password hashing recommendations | 2023 |
| Matrix Spec | Cross-signing, SSSS, Olm/Megolm | v1.7+ |

### A.2 Coin Type Registration (Pending)

| Protocol | Suggested SLIP-44 Coin Type | Status |
|---|---|---|
| Nostr | 1237 | Registered |
| AT Protocol | 1000 (placeholder) | Not registered — use test range until registered |
| Matrix | 2345 (placeholder) | Not registered |
| XMPP | 3456 (placeholder) | Not registered |

Until official registration: use `m/44'/0x80000001'` through `m/44'/0x80000004'` for ATP, Matrix, XMPP, and SSSS respectively.

### A.3 Security Rationale for Single Master Seed

While Option B (per-protocol keys) offers superior isolation, the decision to use Option A is driven by:

1. **UX research data**: 80%+ of users will not back up 6 independent keys. They will lose access. A system that is cryptographically perfect but unusable is insecure in practice because users will find workarounds (storing keys in plaintext notes).

2. **Recovery matrix**: With per-protocol keys, recovering from device loss requires re-establishing 6 independent identities. This is a significant user support burden for an app with no support team.

3. **Hardened derivation**: The `'` (apostrophe) in derivation paths denotes hardened derivation, which prevents child key compromise from revealing the master seed. Security isolation is strong even within a single seed hierarchy.

4. **Optional isolation**: Power users can enable per-protocol independent seeds. This satisfies the long-tail of high-security users without burdening the mainstream.

5. **Identity linking**: A single seed provides cryptographic binding between all identities. Cross-protocol proofs can reference the seed-derived keys with mathematical certainty that they belong to the same person.

### A.4 Key Research Mitigations (2025)

**Black Hat 2025 — Nostr attacks mitigated by:**
- Identity binding domain separator in signature format (cannot reuse signature across protocols)
- NIP-26 delegation for daily use (master key offline)
- Biometric confirmation for ALL key operations
- Local contact graph cross-checks npub with other protocol identifiers before trusting

**CVE-2025-62176 — Mastodon token bypass mitigated by:**
- Token scope enforcement in app (do not request more scope than needed)
- Token stored in platform keychain (not accessible to other apps)
- OAuth2 PKCE flow ensures only our app can use the token

**CVE-2022-39248 — Matrix protocol confusion mitigated by:**
- Strict type checking on Olm vs Megolm message types
- Domain separation in key derivation (different paths for different key types)
- All key material tagged with protocol identifier in wrapped blob

**Bitchat incident — AI-generated crypto code flaws mitigated by:**
- Formal verification of signature logic (property-based testing)
- Every cryptographic function has a "known answer test" (KAT) from spec test vectors
- No "custom crypto" — all cryptography uses audited, standard libraries
- Code review requirement for ALL crypto code changes
