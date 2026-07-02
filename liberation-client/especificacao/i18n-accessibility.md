# Liberation Client — i18n & Accessibility Implementation Guide

> Como construir um app multilíngue e acessível desde o primeiro commit.
> **Iteração 9** — 2026-07-01

---

## Part 1: Internationalization (i18n)

### 1.1 Language Strategy

**Portuguese first, then the world.** All strings are authored in Brazilian Portuguese as the source language. Translations are contributed by the community.

**v1.0 target languages** (10 minimum):
| # | Language | Rationale |
|---|----------|-----------|
| 1 | 🇧🇷 Português (Brasil) | Source language, project origin |
| 2 | 🇺🇸 English | Universal |
| 3 | 🇪🇸 Español | Latin America + Spain |
| 4 | 🇫🇷 Français | Francophone Africa + Europe |
| 5 | 🇩🇪 Deutsch | European fediverse stronghold |
| 6 | 🇯🇵 日本語 | Japanese fediverse community |
| 7 | 🇸🇦 العربية | RTL support validation |
| 8 | 🇮🇳 हिन्दी | Largest non-Western internet population |
| 9 | 🇷🇺 Русский | Large decentralisation-interested community |
| 10 | 🇨🇳 简体中文 | Required for App Store China (even if sideloaded) |

### 1.2 Architecture: Dual-Layer i18n

```
┌─────────────────────────────────────────┐
│  UI Layer (Flutter)                      │
│  ┌──────────────────────────────────┐   │
│  │  Flutter ARB files               │   │
│  │  (app_pt.arb, app_en.arb, ...)   │   │
│  │  → All UI strings, widgets       │   │
│  └──────────────┬───────────────────┘   │
│                 │                        │
├─────────────────┼────────────────────────┤
│                 │                        │
│  Rust Core      │                        │
│  ┌──────────────▼───────────────────┐   │
│  │  Fluent (.ftl) files             │   │
│  │  → Error messages, log strings,  │   │
│  │    system notifications           │   │
│  └──────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

**Why two systems**:
- **Flutter ARB**: Standard for Flutter UI. Good tooling (gen_l10n). Widget-level.
- **Rust Fluent**: Standard for Rust. Good crate (`fluent-rs`). System-level strings (errors, logs, CLI output).

### 1.3 Flutter ARB Structure

```dart
// l10n/app_pt.arb — Source language (Brazilian Portuguese)
{
  "@@locale": "pt",
  "@@description": "Liberation Client — Brazilian Portuguese",

  // App Entry
  "entryGreetingMorning": "Bom dia.",
  "entryGreetingAfternoon": "Boa tarde.",
  "entryGreetingEvening": "Boa noite.",
  "@entryGreetingMorning": { "description": "Morning greeting on app entry screen" },

  "entryBrowse": "Ler e explorar",
  "entryBrowseDesc": "Sua timeline unificada, em ordem cronológica",
  "entryRespond": "Responder",
  "entryRespondDesc": "Notificações, menções e mensagens diretas",
  "entryShare": "Escrever",
  "entryShareDesc": "Publicar algo novo",

  // Timeline
  "timelineAllProtocols": "Todos",
  "timelineCheckNew": "Ver novos posts",
  "timelineLoadMore": "Carregar mais {count} posts",
  "timelineCaughtUp": "Você está em dia ✨",
  "timelineCaughtUpSubtitle": "A internet pode esperar.",
  "timelineEmptyTitle": "Sua timeline está vazia",
  "timelineEmptySubtitle": "Isso é por design. Você ainda não escolheu o que colocar nela.",

  // Post
  "postRepliedBy": "{people} respondeu",
  "postRepliedByPlural": "{people} e mais {count} responderam",
  "postAcknowledgedBy": "{people} reconheceu",
  "postRepostedBy": "{people} compartilhou",
  "postVia": "via {instance}",
  "postAltText": "ALT",
  "postSensitiveContent": "Conteúdo sensível",
  "postTapToReveal": "Toque para revelar",

  // Composer
  "composerPlaceholder": "O que você quer dizer?",
  "composerHoldToPublish": "SEGURE PARA PUBLICAR",
  "composerConfirmTitle": "Publicar",
  "composerConfirmBody": "Isso será visível para todos no {protocols}. Tem certeza?",
  "composerAltRequired": "Texto alternativo é obrigatório para imagens.",
  "composerAdaptMode": "Adaptar para cada protocolo",
  "composerDraftSaved": "Rascunho salvo",

  // Profile
  "profileFollow": "Seguir",
  "profileFollowing": "Seguindo",
  "profileFollowCooldown": "Seguindo em {hours}h. Cancelar?",
  "profileIdentityCloud": "Nuvem de Identidade",
  "profileHowConnected": "Como vocês estão conectados",
  "profileMutualFollows": "Seguem um ao outro",
  "profileSharedList": "Na lista '{listName}'",
  "profileBothOn": "Ambos no {protocol}",

  // Notifications
  "notificationsMentions": "MENÇÕES",
  "notificationsReplies": "RESPOSTAS",
  "notificationsDirectMessages": "MENSAGENS DIRETAS",
  "notificationsMarkSeen": "Marcar como visto",
  "notificationsNoLikes": "Sem notificações de curtidas ou compartilhamentos. Apenas conversas.",

  // Encryption
  "encryptionVerified": "Verificado ✓",
  "encryptionUnverified": "Criptografado (não verificado)",
  "encryptionNone": "Sem criptografia",
  "encryptionEphemeral": "Efêmero",
  "encryptionVerifyPrompt": "Verificar segurança desta conversa",
  "encryptionEmojiMatch": "Os emojis são iguais?",

  // Settings
  "settingsExportData": "Exportar todos os dados",
  "settingsExportConfirm": "Isso vai gerar um arquivo .zip com todos os seus dados. Tamanho estimado: {size}. Continuar?",
  "settingsSimpleMode": "Modo Simples",
  "settingsSimpleModeDesc": "Esconde a complexidade dos protocolos",
  "settingsSessionTimer": "Temporizador de sessão",
  "settingsAutoClose": "Fechar app após o tempo",
  "settingsManifesto": "Ler o Manifesto",

  // Empty States
  "emptyOffline": "Você está offline",
  "emptyOfflineSubtitle": "Mostrando o que está salvo no dispositivo.",
  "emptyProtocolDown": "{protocol} está indisponível",
  "emptyProtocolDownSubtitle": "Seus outros protocolos funcionam normalmente.",
  "emptySearchNoResults": "Nenhum resultado para '{query}'",
  "emptyNoNotifications": "Nada ainda",
  "emptyNoDMs": "Nenhuma mensagem",

  // Session
  "sessionTimerPrefix": "Você passou {minutes} min aqui hoje",
  "sessionExploreTime": "Você está explorando há {minutes} min.",
  "sessionExploreNudge": "Encontrou algo interessante?",
  "sessionExploreHardStop": "Que tal seguir algumas dessas pessoas e voltar depois?",

  // Onboarding
  "onboardingManifestoTitle": "Isto não é uma rede social.",
  "onboardingManifestoSubtitle": "É uma ferramenta para as conversas que você escolhe ter.",
  "onboardingPathImport": "JÁ TENHO CONTAS",
  "onboardingPathCreate": "SOU NOVO NISSO",
  "onboardingPathExplore": "SÓ QUERO EXPLORAR",
  "onboardingFirstPost": "Se apresente para a rede aberta",
  "onboardingDone": "Pronto. Vá viver sua vida."
}
```

### 1.4 English ARB (Translation)

```dart
// l10n/app_en.arb
{
  "@@locale": "en",
  "entryGreetingMorning": "Good morning.",
  "entryGreetingAfternoon": "Good afternoon.",
  "entryGreetingEvening": "Good evening.",
  "entryBrowse": "Browse",
  "entryBrowseDesc": "Your unified timeline, in chronological order",
  "entryRespond": "Respond",
  "entryRespondDesc": "Notifications, mentions, and direct messages",
  "entryShare": "Write",
  "entryShareDesc": "Publish something new",
  // ... full English translation ...
  "postRepliedBy": "{people} replied",
  "postRepliedByPlural": "{people} and {count} more replied",
  "timelineCaughtUp": "You're all caught up ✨",
  "composerHoldToPublish": "HOLD TO PUBLISH",
  "encryptionVerified": "Verified ✓",
  "emptyOffline": "You're offline",
  "onboardingDone": "Done. Go live your life."
}
```

### 1.5 Rust Fluent (System Strings)

```ftl
# locales/pt-BR/main.ftl

# Sync engine status messages
sync-started = Sincronizando {$protocol}...
sync-completed = {$protocol} sincronizado ({$count} itens)
sync-error = Erro ao sincronizar {$protocol}: {$error}
sync-offline = Dispositivo offline. {$count} posts na fila.

# Key management
key-backup-created = Frase de recuperação criada. Guarde em local seguro.
key-backup-verify = Confirme sua frase de recuperação:
key-social-recovery-setup = Recuperação social configurada ({$have} de {$need} guardiões)
key-compromised-warning = ATENÇÃO: Chave privada do {$protocol} pode ter sido comprometida.

# Security
security-unsigned-event-rejected = Evento Nostr rejeitado: assinatura inválida
security-nip04-blocked = NIP-04 bloqueado: use NIP-44 (ChaCha20-Poly1305)
security-xss-blocked = Conteúdo HTML malicioso removido de post {$id}
security-token-expired = Token expirado para {$protocol}. Reautenticando...

# Database
db-migration-applied = Migração {$version} aplicada em {$duration}ms
db-vacuum-completed = VACUUM concluído. {$freed} liberados.
db-corruption-detected = Corrupção detectada no banco. Tentando recuperação...
```

### 1.6 Plural Rules & RTL Support

```dart
// Flutter ARB handles plurals natively:
"postRepliedByPlural": "{people} e mais {count} responderam",
"@postRepliedByPlural": {
  "placeholders": {
    "people": { "type": "String" },
    "count": { "type": "int" }
  }
}
// For languages with complex plurals (Arabic, Russian),
// ARB supports ICU message format:
// "{count, plural, =1{1 pessoa} other{{count} pessoas}}"

// RTL layout is automatic with Flutter's Directionality widget.
// Tested with Arabic (ar) as the validation language.
// No hardcoded left/right — use start/end everywhere.
```

### 1.7 Community Translation Pipeline

```
1. String added to app_pt.arb (source of truth)
2. CI extracts new strings → creates issue in Weblate/Crowdin
3. Community translators submit translations
4. PR opened with translated .arb files
5. Review by 1 native speaker + 1 existing translator
6. Merge → CI generates localized app bundle
```

---

## Part 2: Accessibility Implementation Guide

### 2.1 Core Principle

> Accessibility is not a feature. It's a prerequisite for liberation.

If the app isn't accessible, its promise of "liberation from big tech" is a lie — it only liberates people without disabilities. Every screen spec'd in the wireframes (Iter 4) must work with VoiceOver, Dynamic Type, and assistive technologies.

### 2.2 Screen Reader (VoiceOver / TalkBack)

```dart
// Every interactive element MUST have semantic labels.
// Protocol badges — dual-encoded (color + shape) AND labeled.

// ❌ WRONG — invisible to screen readers
Container(
  decoration: BoxDecoration(color: ProtocolColors.nostr),
  child: Icon(NostrIcons.hexagon),
)

// ✅ CORRECT — semantic, translatable
Semantics(
  label: AppLocalizations.of(context)!.protocolBadgeNostr,
  hint: AppLocalizations.of(context)!.protocolBadgeHint,
  child: Container(
    decoration: BoxDecoration(color: ProtocolColors.nostr),
    child: Icon(NostrIcons.hexagon),
  ),
)

// Timeline announcements — tell user what changed
Semantics(
  label: '${postCount} posts from ${authorCount} accounts',
  hint: 'Double-tap to open post. Swipe right to reply. Swipe left to bookmark.',
  child: TimelineView(posts: posts),
)

// Post interaction bar — names, not numbers
// ❌ "3 replies" — abstract number
// ✅ "Alice, Bob, and Carol replied" — concrete people
Semantics(
  label: '${replyNames.join(', ')} ${repliedWord}',
  // PT-BR: "Alice, Bob e Carol responderam"
  // EN: "Alice, Bob, and Carol replied"
)

// ALT text on images — mandatory, navigable
Semantics(
  label: 'Image. ${attachment.altText}',
  hint: 'Double-tap to view full size',
  // Also make ALT badge navigable:
  onTap: () => showAltTextOverlay(attachment.altText),
)

// Encryption status on conversations
// Green lock, yellow lock, gray lock, red triangle — ALL need labels.
Semantics(
  label: encryptionVerified
      ? 'End-to-end encrypted. Verified.'
      : encryptionEnabled
          ? 'End-to-end encrypted. Not yet verified. Double-tap to verify.'
          : 'Not encrypted.',
)
```

### 2.3 Dynamic Type & Text Scaling

```dart
// All text must respect system font size settings.
// Never hardcode font sizes — use relative scaling.

// ✅ Text scales with system settings
Text(
  post.content,
  style: Theme.of(context).textTheme.bodyLarge, // Defined in theme, respects system.
  maxLines: null, // Don't truncate text.
  overflow: TextOverflow.visible, // Don't clip.
)

// ❌ Hardcoded sizes break accessibility
Text(post.content, style: TextStyle(fontSize: 14)), // IGNORES user's size preference.

// ❌ maxLines: 2 — truncates content for users who need larger text.
Text(post.content, maxLines: 2),

// Layouts must reflow when text scales up to 2x.
// Use Flexible, Expanded, Wrap instead of fixed widths.
// Test at: Default, Large (1.3x), Extra Large (1.5x), Accessibility (2x).
```

### 2.4 Color & Contrast

```dart
// All text must meet WCAG AA contrast ratio (4.5:1 for normal text, 3:1 for large).
// Protocol badge colors were chosen for contrast against both light and dark backgrounds.

// Primary text on background:
// #1A1A1A on #FAFAF5 = 14.2:1 ✅ (exceeds AAA 7:1)

// Protocol accent on card:
// #6364FF (ActivityPub) on #FFFFFF = 4.6:1 ✅ (meets AA)
// #9CA3AF (IRC) on #FFFFFF = 2.8:1 ❌ (fails AA)
//   → Darken IRC gray to #6B7280 for 4.9:1 ✅

// High Contrast Mode: toggle in Settings.
// When enabled:
//   - All text → pure black (#000) or pure white (#FFF)
//   - Background → pure white or pure black
//   - Borders added to differentiate cards (not just shadows)
//   - Link underlines ALWAYS visible (not just on hover)

if (settings.highContrast) {
  return HighContrastTheme(
    textColor: CupertinoColors.label,
    backgroundColor: CupertinoColors.systemBackground,
    // Override protocol colors with high-contrast equivalents.
    protocolColors: highContrastProtocolColors,
    // Add visible borders to all cards.
    cardBorder: Border.all(color: CupertinoColors.separator),
  );
}
```

### 2.5 Touch Targets & Interaction

```dart
// Minimum touch target: 44x44pt (Apple HIG) / 48x48dp (Material).
// Every tappable element must meet this.

// ❌ Tiny touch target
GestureDetector(
  onTap: () => openPost(post),
  child: Icon(Icons.reply, size: 16), // Only 16x16! Way too small.
)

// ✅ Minimum 44x44 touch target
GestureDetector(
  onTap: () => openPost(post),
  child: Padding(
    padding: EdgeInsets.all(14), // 16 + 28 = 44
    child: Icon(Icons.reply, size: 16),
  ),
)

// Long-press alternatives for users who can't long-press.
// Composer "Hold to Publish" — provide alternative:
//   - Hold for 1.5s (default)
//   - Triple-tap (accessibility alternative)
//   - "Publish" button appears after 3 seconds of composing (timeout)
```

### 2.6 Reduced Motion

```dart
// Respect system "Reduce Motion" accessibility setting.
final bool reduceMotion = MediaQuery.of(context).disableAnimations;

// Animations that MUST be disabled when Reduce Motion is on:
// - Timeline scroll physics (disable spring, use linear)
// - Parallax banner on profiles (disable, show static)
// - Crossfade transitions (disable, use instant cuts)
// - Composer slide-up (disable, navigate directly)

final pageTransition = reduceMotion
    ? PageTransitionsTheme(
        builders: { TargetPlatform.iOS: const NoAnimationPageTransitionsBuilder() },
      )
    : null;
```

### 2.7 Cognitive Accessibility

```dart
// Simple Mode — hides protocol complexity.
// Toggle in Settings → Appearance → Simple Mode.
// When ON:
//   - Protocol badges hidden
//   - Protocol filter pills hidden
//   - Identity Cloud collapsed to single "you"
//   - "Also on" sections hidden
//   - Composer posts to default protocol only
//   - All protocol-specific settings grouped under "Advanced"

// Session boundaries — reduce overwhelm.
// - Pagination: 50 posts per page, explicit "Load more"
// - "You're all caught up" — closure, not endlessness
// - After 200 posts read: gentle "Que tal uma pausa?"
// - Session timer: 15/30/60 min, auto-close option

// Reading mode — single post, no timeline noise.
// Tappable from any post. Shows only that post + its thread.
// No sidebar, no notifications, no badges. Just reading.
```

### 2.8 Accessibility Audit Checklist

Per-screen audit (from wireframes, Iter 4):

| Screen | VoiceOver Labels | Dynamic Type | Contrast | Touch Targets | Reduced Motion |
|--------|:---:|:---:|:---:|:---:|:---:|
| App Entry | ✓ | ✓ | ✓ | ✓ | ✓ |
| Timeline | ✓ | ✓ | ✓ | ✓ | ✓ |
| Post Detail | ✓ | ✓ | ✓ | ✓ | ✓ |
| Composer | ✓ | ✓ | ✓ | ✓ | ✓ |
| Profile | ✓ | ✓ | ✓ | ✓ | ✓ |
| Notifications | ✓ | ✓ | ✓ | ✓ | ✓ |
| DMs | ✓ | ✓ | ✓ | ✓ | ✓ |
| Explore | ✓ | ✓ | ✓ | ✓ | ✓ |
| Settings | ✓ | ✓ | ✓ | ✓ | ✓ |
| Empty States | ✓ | ✓ | ✓ | ✓ | ✓ |
| Onboarding | ✓ | ✓ | ✓ | ✓ | ✓ |

**Testing requirements**:
- VoiceOver/TalkBack: navigate entire app, every element reachable and meaningful
- Dynamic Type: smallest → accessibility largest, no text clipped
- Contrast: WCAG AA minimum verified with contrast checker
- Keyboard: full navigation without touch (Bluetooth keyboard on iOS, Keyboard on Android)
- Switch Control: every action accessible via switch (single button)

### 2.9 Developer Checklist (Per PR)

```markdown
## Accessibility PR Checklist

- [ ] New strings added to app_pt.arb (NOT hardcoded)
- [ ] Semantic labels on ALL interactive elements
- [ ] ALT text on ALL images (required in composer, auto-prompted)
- [ ] Touch targets ≥ 44x44pt on all tappable elements
- [ ] Tested with Dynamic Type at 2x scale
- [ ] Tested with VoiceOver/TalkBack on one screen
- [ ] Color contrast passes AA for all text
- [ ] Reduced Motion: animations have static fallbacks
- [ ] RTL: layout works with Arabic locale
```

---

*Documento gerado na Iteração 9 do /loop.*
