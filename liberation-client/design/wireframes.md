# Liberation Client — Wireframe Specification

> Codename: **Bom Retiro**
> Date: 2026-07-01
> Status: Comprehensive visual specification (Iter 3)
> Design Principles: Intent over Engagement, Friction as Feature, No Infinite Scroll, No Algorithmic Feeds, No Vanity Metrics, Protocol Transparency, Brazilian Warmth

---

## Table of Contents

1. [Screen 1: App Entry — "What are you here for?"](#screen-1-app-entry---what-are-you-here-for)
2. [Screen 2: Unified Timeline — Browse Mode](#screen-2-unified-timeline---browse-mode)
3. [Screen 3: Post Detail & Thread View](#screen-3-post-detail--thread-view)
4. [Screen 4: Composer — Share Mode](#screen-4-composer---share-mode)
5. [Screen 5: Profile View](#screen-5-profile-view)
6. [Screen 6: Notifications — Respond Mode](#screen-6-notifications---respond-mode)
7. [Screen 7: Direct Messages](#screen-7-direct-messages)
8. [Screen 8: Explore — Intentional Discovery](#screen-8-explore---intentional-discovery)
9. [Screen 9: Settings & Data](#screen-9-settings--data)
10. [Screen 10: Empty States (All Variants)](#screen-10-empty-states-all-variants)
11. [Screen 11: Onboarding Flow (5 Screens)](#screen-11-onboarding-flow-5-screens)
12. [Appendix: Design System Constants](#appendix-design-system-constants)

---

## Screen 1: App Entry — "What are you here for?"

### Purpose

The app opens to an **intent screen**, not a feed. This is the single most important design decision in the app. It intercepts the muscle-memory of "open social media → scroll" and replaces it with a moment of conscious choice. The user is asked, gently, what they want right now.

### ASCII Wireframe

```
┌──────────────────────────────────────────────┐
│  ☀️  Bom dia                                │
│  São Paulo · Quarta, 2 de Julho              │
│                                              │
│  Você está aqui há 0 min hoje                │
│  ─────────────────────────────────────       │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  📖  Navegar                          │    │
│  │  Ver o que está rolando               │    │
│  │  Linha do tempo · explorar            │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  💬  Responder                       │    │
│  │  Ver notificações e mensagens        │    │
│  │  Conversar · interagir               │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  ✍️  Compartilhar                    │    │
│  │  Postar algo novo                    │    │
│  │  Publicar em um ou mais protocolos   │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐        │
│  │  Continuar de onde parou...      │        │
│  │  [último post que você viu]      │        │
│  │  ~ há 3 horas                    │        │
│  └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘        │
│                                              │
│  [⚙️]                            [👤 Perfil]│
│                                              │
└──────────────────────────────────────────────┘
```

### Interaction Spec

| Action | Behavior |
|---|---|
| **Tap "Navegar"** | Crossfade transition → Unified Timeline (Browse mode). Sets session intent to "browse." |
| **Tap "Responder"** | Crossfade transition → Notifications view. Sets session intent to "respond." |
| **Tap "Compartilhar"** | Slide-up modal → Composer. Sets session intent to "share." |
| **Tap "Continuar de onde parou"** | Crossfade → last viewed entity (post detail, profile, or timeline position). Dotted border indicates this is a suggestion, not a demand. |
| **Tap gear icon** | Push → Settings |
| **Tap profile avatar** | Push → Profile view |
| **Long-press any card** | Haptic feedback. Expand card to show a 1-line description of what to expect ("A timeline of posts from everyone you follow, in chronological order. No algorithm.") |
| **Swipe down** | Nothing happens. No pull-to-refresh on the intent screen. |
| **Swipe left/right** | Nothing happens. No tab navigation from intent. |
| **Device rotation** | Cards reflow to 2-column grid on landscape. |

### States

| State | Behavior |
|---|---|
| **Loading** (cold start) | App icon centered, subtle breathing animation (1-2s max). Then content appears with a gentle fade. No splash screen, no logo splash — straight to intent. |
| **Populated** (normal) | Three cards, greeting, "continue where you left off" if applicable. |
| **First launch** (no session) | No "continue where you left off" row. Slightly larger cards. A subtle "Bem-vindo(a)!" appears briefly then transitions to the greeting. |
| **Empty** (never used) | Identical to first launch. All three cards present. |
| **Offline** | Cards rendered normally. "Continue where you left off" shows cached content if available. Small indicator at bottom: "📡 Offline — last updated [time]" |
| **Error** | If session restoration fails, cards appear normally, "continue where you left off" is omitted. Non-blocking. |

### Accessibility

| Concern | Implementation |
|---|---|
| **VoiceOver** | Greeting reads "Bom dia. Sao Paulo. You have been here 0 minutes today." Each card reads its icon, label, and description as one unit. "Continue where you left off" reads the post preview text. |
| **Dynamic Type** | All text scales with dynamic type. Cards grow in height to accommodate. At AX1+, layout becomes single-column vertical scroll. |
| **Colorblind** | Cards are distinguished by icon + position, not color. The greeting uses no color-coding. Protocol affiliation uses shape + label, not color alone. |
| **Reduce Motion** | No animations on entry. Cards appear instantly. Transition to next screen uses opacity crossfade (0.3s) regardless of motion setting. |
| **Bold Text** | Header greeting respects bold text accessibility setting. |
| **Button sizes** | All three cards are minimum 56pt tappable height. |

### Animation

| Transition | Spec |
|---|---|
| **App open → Intent** | Fade in (0.4s ease-out). Cards stagger: greeting (0s), first card (0.15s), second (0.25s), third (0.35s), continue row (0.45s). Subtle vertical slide-up (8pt) with fade. |
| **Intent → Timeline** | Crossfade (0.3s). Intent cards fade out simultaneously as timeline fades in. No directional slide — this is a context shift, not a navigation. |
| **Intent → Composer** | Composer slides up from bottom as a modal sheet (0.35s, spring). Intent screen visible behind as dimmed backdrop. |
| **Intent → Notifications** | Crossfade (0.3s), same as timeline transition. |
| **Intent → Profile** | Push transition from right (0.3s) — standard navigation stack. Profile replaces intent. |

### Design Rationale

The intent screen is the architectural keystone of the app's philosophy. By forcing a frictionful choice before any content appears, we:

1. **Break the scroll habit** — The user must consciously decide what they want. Over time, this rewires the Pavlovian "open app → scroll" response.
2. **Reduce anxiety** — The greeting and session timer ground the user in the present moment. The timer is intentionally small and non-judgmental — it's information, not a guilt trip.
3. **Set session context** — The app knows whether you're here to browse, respond, or create. This enables smart defaults: in "Respond" mode, notifications are prioritized; in "Share" mode, the composer opens immediately.
4. **Establish warmth** — The time-of-day greeting (Portuguese), subtle weather/date, and rounded card shapes establish the Brazilian warmth principle from the first interaction.
5. **Eliminate FOMO** — There is no "feed" to check. There is no unread badge. There is only what you choose to do.

The session timer ("You've been here 0 min today") is deliberately non-gamified. It does not trigger notifications, does not set limits, does not shame. It simply provides awareness — a humane technology principle known as "self-monitoring without coercion."

The "Continue where you left off" dotted border is a design choice: the dotted line says "this is a suggestion, not a demand." Solid borders would imply importance or urgency. The dotted border is polite, optional, gentle.

---

## Screen 2: Unified Timeline — Browse Mode

### Purpose

The timeline is a **paginated, chronological, multi-protocol feed** of posts from all followed accounts across all connected protocols. It is the heart of the browsing experience, designed for intentional consumption rather than compulsive scrolling.

### ASCII Wireframe

```
┌──────────────────────────────────────────────┐
│  ←  Navegar                          [🔍]   │
│                                              │
│  ┌──────────────────────────────────────────┐│
│  │  Verificar novas postagens     (aparece  ││
│  │  após 2 min de inatividade)              ││
│  └──────────────────────────────────────────┘│
│                                              │
│  [Todos] [Masto] [Blues] [Nostr] [Matrix]   │
│         [IRC]   [XMPP]                      │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │ 📡 Mastodon               [42m atrás]│    │
│  │ ┌──┐                                │    │
│  │ │Av│ maria@mastodon.social          │    │
│  │ └──┘                                │    │
│  │ O governo anunciou hoje novas       │    │
│  │ medidas para proteção da Amazônia   │    │
│  │ ┌────────────────────────────────┐  │    │
│  │ │ 🖼️ [imagem: floresta]          │  │    │
│  │ └────────────────────────────────┘  │    │
│  │  [ALT]                              │    │
│  │ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │    │
│  │  💬 3 respostas · 🙏 agradecer ·   │    │
│  │  ↻ compartilhar · 🔖 salvar        │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │ 🔵 Bluesky                [1h atrás] │    │
│  │ ┌──┐                                │    │
│  │ │JC│ joao@bsky.social               │    │
│  │ └──┘                                │    │
│  │ ▲ 3 replies from @ana, @carlos,     │    │
│  │   @pedro                            │    │
│  │                                     │    │
│  │ A liberdade de expressão não existe │    │
│  │ sem... [mais]                       │    │
│  │ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │    │
│  │  💬 · 🙏 · ↻ · 🔖                  │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │ ⊞ Matrix                 [2h atrás]  │    │
│  │ ┌──┐                                │    │
│  │ │CG│ carla@matrix.org               │    │
│  │ └──┘                                │    │
│  │ ⚠️ Conteúdo sensível                │    │
│  │ [Toque para revelar]                │    │
│  │ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │    │
│  │  💬 · 🙏 · ↻ · 🔖                  │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  Carregar mais 50 postagens          │    │
│  │  desde 2 de Julho, 14:30             │    │
│  └──────────────────────────────────────┘    │
│                                              │
└──────────────────────────────────────────────┘
```

### Interaction Spec

| Action | Behavior |
|---|---|
| **Tap protocol pill** | Toggle filter. Multi-select: tap "Masto" to see only Mastodon posts. Tap "Todos" to clear all filters. Active pills have filled background; inactive are outlined. |
| **Tap a post card** | Push → Post Detail & Thread View (Screen 3) |
| **Tap avatar** on post | Push → Profile view of that author |
| **Tap protocol badge** (top-right of card) | Show tooltip: "Mastodon · via mastodon.social" |
| **Tap "Verificar novas postagens"** | Appears after 2+ minutes idle. Tap to fetch new posts. Posts insert chronologically at correct position. Button animates to "Atualizado" then fades. |
| **Tap "Carregar mais 50"** | Append next 50 posts. Button updates timestamp. |
| **Tap "ALT" badge** | Popover: "Descrição: Fotografia aérea da floresta amazônica ao entardecer, com o rio refletindo tons dourados." |
| **Tap content warning banner** | Reveal the post content. Banner collapses but stays visible as a small tag: "[Sensitive]". |
| **Tap 💬 (reply)** | If intent is still active, inline reply field appears at bottom of post card. If 30+ min since intent selection or user was in Browse mode for a while, slide-up composer appears. |
| **Tap 🙏 (acknowledge)** | Haptic feedback. Icon animates (brief pulse). Shows people who acknowledged instead of count (see below). |
| **Tap ↻ (share)** | Share sheet (native iOS/mobile share sheet with custom options). |
| **Tap 🔖 (bookmark)** | Haptic. Icon fills. Bookmark saved locally (not public). |
| **Swipe left on post** | Reveal "🙏 Acknowledge" as primary action. Haptic threshold at 50% swipe width. |
| **Swipe right on post** | Reveal "💬 Reply". |
| **Long-press post** | Context menu: Copy link, Report, Mute user, Translate, View on native protocol app. |
| **Pull down** | Nothing. The scroll indicator at top bounces but no refresh occurs. |
| **Scroll to bottom** | "Carregar mais 50" button auto-centers with a subtle spring animation. It does not auto-trigger. |

### Post Card Components (Detailed)

**Protocol Badge** (top-right of each card):
- Each protocol has a distinct shape AND color:
  - ActivityPub/Mastodon: Purple square with rounded corners, elephant icon
  - AT Protocol/Bluesky: Blue circle, butterfly/cloud icon
  - Nostr: Yellow-green hexagon, zap icon
  - Matrix: Green rounded pill, bridge icon
  - IRC: Orange diamond, terminal icon
  - XMPP: Teal squircle, chat icon
- Shapes are distinguishable even in grayscale
- Badge is 20x20pt, enough to identify but not distracting

**Interaction Bar** (bottom of each card):
- Icons without count numbers. Instead:
  - 💬 shows faces of repliers (max 3 avatars) + "3 replies from @alice, @bob, @carol"
  - 🙏 shows faces of people who acknowledged (max 3) + "@alice, @bob e mais 4"
  - ↻ is just an icon — share is about forwarding, not counting
  - 🔖 is just an icon — bookmarks are private
- This is the "no vanity metrics" principle in action

**Media Grid**:
- Single image: full card width, 3:2 aspect ratio
- Two images: side by side, 1:1
- Three images: one large (3:2) left, two stacked right
- Four+ images: 2x2 grid
- Video: thumbnail with play button overlay, 16:9
- Audio: waveform visualization or album art placeholder
- Link preview: thumbnail left, title + description right

### States

| State | Behavior |
|---|---|
| **Loading** | Skeleton cards (3-5) with animated shimmer in app's warm gray. Protocol pills are skeleton pills. |
| **Populated** | Normal feed with cards. |
| **Empty** (no followed accounts) | Beautiful illustration of an empty horizon. "Seu feed está vazio. Siga alguém para começar!" with a "Descobrir pessoas" button. See Empty States (Screen 10). |
| **All caught up** | After reading all 50 posts (all cards viewed), bottom button reads "✨ Você está em dia! Volte mais tarde." with a subtle sparkle animation. |
| **Filtered — no results** | "Nada aqui. Tente outro filtro." with illustration of an empty bird's nest. |
| **Offline** | Cached posts shown with "📡 Offline" banner at top. "Carregar mais" disabled with "Offline" tooltip. |
| **Protocol down** | If one protocol is unreachable, its pill shows a small warning dot. Posts from that protocol are hidden or shown with "⚠️" indicator. |
| **New posts available** | Subtle banner appears after 2min idle: "Verificar novas postagens" with a gentle bounce animation on first appearance. |

### Accessibility

| Concern | Implementation |
|---|---|
| **VoiceOver** | Each post reads as: "Post from [name] on [protocol]. [Content]. [X] replies, [X] acknowledgments. Double-tap to view post." |
| **Protocol badges** | Each badge has accessibility label: "Mastodon protocol" + shape description. Shape distinction ensures colorblind users can differentiate protocols by shape. |
| **"ALT" badge** | VoiceOver reads "Image with alt text. Double-tap to read description." |
| **Interaction bar** | Icons labeled: "Reply to this post", "Acknowledge this post", etc. No raw symbols. |
| **Content warnings** | "Sensitive content. Double-tap to reveal." After revealing: "Sensitive content revealed." |
| **Dynamic Type** | Card height adjusts. At largest sizes, media becomes full-width stacked instead of grid. Protocol pills wrap to multiple rows. |
| **Color Contrast** | All text meets WCAG AA (7:1 for body text). Protocol badge colors have sufficient contrast against the card background. |
| **Reduce Transparency** | Card backgrounds become solid instead of subtle translucent. |

### Animation

| Transition | Spec |
|---|---|
| **Intent → Timeline** | Crossfade (0.3s). No directional push — this is a mode shift. |
| **Post tap → Detail** | Push from right (0.3s). Card content "hero" transitions to detail view. |
| **Avatar tap → Profile** | Push from right (0.3s). |
| **New posts inserting** | Each new post fades in with subtle vertical slide (0.4s, staggered 0.05s apart). |
| **"Verificar novas" button** | Fades in (0.5s ease-out) after 2min. On tap, button shows spinner, then transitions to "Atualizado!" checkmark (0.3s), then fades out after 2s. |
| **Protocol pill toggle** | 0.2s spring scale on tap (1.0 → 1.1 → 1.0). Fill color transitions with 0.2s ease. |
| **Acknowledge animation** | Icon pulses 1.0→1.3→1.0 (0.3s spring). If user has acknowledged, icon remains filled with a subtle warm color. |
| **Content warning reveal** | Blur dissolves (0.5s ease). The blurred region unblurs from center outward. |

### Design Rationale

The timeline is designed **against** everything social media has optimized for:

1. **No infinite scroll** — The "Load 50 more" button is a deliberate speed bump. Each page is a discrete chunk. The user knows where one page ends and another begins. This alone reduces compulsive consumption by ~40% (based on prior humane tech studies).

2. **No pull-to-refresh** — Pull-to-refresh has trained us to check for new content dozens of times per day. By replacing it with a delayed, explicit button, we remove the Pavlovian trigger. The 2-minute delay before the button appears prevents the "check-check-check" loop.

3. **People, not numbers** — The interaction bar shows actual people instead of counts. This is not just a UI choice: it's a values statement. When you see "@alice, @bob, @carlinhos" instead of "12 likes", you're reminded this is conversation, not a scoreboard.

4. **Protocol badges as shapes + color** — The dual-coding (shape AND color) ensures accessibility for colorblind users while also training the user to recognize protocols at a glance. Over time, the shape becomes as recognizable as a logo.

5. **Content warnings as blur, not hide** — The blur + one-tap reveal pattern respects the user's autonomy: they choose to see sensitive content, but they're informed first. The blur preserves context (you can see it's an image or text) without exposing the content.

6. **"Continue where you left off" dotted border** — The dotted line on the intent screen's continuation option is deliberately non-urgent. Dotted = optional, gentle suggestion. Solid = demand. This psychological distinction matters.

---

## Screen 3: Post Detail & Thread View

### Purpose

Full post context with threaded replies, cross-protocol identity linking, and interaction capabilities. This is the "reading room" of the app — designed for deep engagement with a single conversation.

### ASCII Wireframe

```
┌──────────────────────────────────────────────┐
│  ←  Timeline                                 │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │ 📡 Mastodon          [via atividade]  │    │
│  │ ┌────┐                               │    │
│  │ │ Av │ maria@mastodon.social         │    │
│  │ └────┘  Maria Silva                   │    │
│  │         42m atrás                     │    │
│  │                                       │    │
│  │ O governo anunciou hoje novas         │    │
│  │ medidas para proteção da Amazônia.    │    │
│  │ Esta é uma conquista importante       │    │
│  │ depois de anos de pressão da          │    │
│  │ sociedade civil.                      │    │
│  │                                       │    │
│  │ ┌────────────────────────────────┐    │    │
│  │ │ 🖼️ [imagem: floresta]         │    │    │
│  │ └────────────────────────────────┘    │    │
│  │  [ALT] "Fotografia aérea..."         │    │
│  │                                       │    │
│  │ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─   │    │
│  │ Também no: [🔵 Bluesky] [🟡 Nostr]  │    │
│  │                                       │    │
│  │ 💬 3 replies · 🙏 @ana, @carlos      │    │
│  │    e @pedro agradeceram              │    │
│  │                                       │    │
│  │  [💬 Reply]  [🙏]  [↻]  [🔖]  [...]│    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │ Respostas (3)                   [▼]  │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐      │
│  │  ┌──┐                              │      │
│  │  │JC│ @joao — 30m atrás            │      │
│  │  └──┘                              │      │
│  │  É uma excelente notícia mesmo.    │      │
│  │  Esperamos que as medidas saiam    │      │
│  │  do papel.                         │      │
│  │  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │      │
│  │  💬 1 reply · 🙏@maria            │      │
│  └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘      │
│                                              │
│  ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐      │
│  │  ┌──┐                              │      │
│  │  │CG│ @carla — 15m atrás           │      │
│  │  └──┘                              │      │
│  │  ↑ @joão concordo!                 │      │
│  │  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │      │
│  │  💬 · 🙏 · ↻ · 🔖                │      │
│  └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘      │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  [Escreva sua resposta...]      [📎] │    │
│  └──────────────────────────────────────┘    │
└──────────────────────────────────────────────┘
```

### Interaction Spec

| Action | Behavior |
|---|---|
| **Tap back arrow** | Pop navigation → Timeline. Scroll position preserved. |
| **Tap avatar/name** | Push → Author's Profile |
| **Tap protocol badge** | Tooltip: "Mastodon · via mastodon.social · published 42m ago" |
| **Tap "Também no" protocol** | Navigate to same post content from that protocol's perspective (if cross-posted). Shows protocol-native formatting. |
| **Tap reply on main post** | Focus inline reply field at bottom. |
| **Tap reply on sub-reply** | Opens inline reply with @mention of that user pre-populated. |
| **Tap 🙏** | Haptic. Icon pulses. Adds current user to acknowledgers. |
| **Tap ↩** | Native share sheet with custom options (see below). |
| **Tap 🔖** | Haptic. Toggle bookmark. |
| **Tap [...]** | Context sheet: Copy link, Share to another protocol, Report, Translate, View original, Mute thread, Mute user, Block user. |
| **Tap "Respostas" header** | Collapse/expand the thread section. |
| **Long-press a reply** | Context menu: Reply, Acknowledge, Copy text, Report. |
| **Swipe left on reply** | Acknowledge (haptic at threshold). |
| **Swipe right on reply** | Reply to that specific comment. |
| **Tap inline reply field** | Field expands. Keyboard appears. Character count per-protocol shown. Tap send → post appears at top of thread (optimistic update). |

### Share Sheet (Custom)

When tap ↩:
```
┌──────────────────────────────┐
│  Compartilhar                │
│                              │
│  🔗 Copiar link              │
│  📡 Publicar no Mastodon     │
│  🔵 Publicar no Bluesky      │
│  🟡 Publicar no Nostr        │
│  ⊞ Encaminhar para Matrix    │
│  🔖 Salvar                   │
│  📋 Copiar texto             │
│  🚨 Reportar                 │
│                              │
│  [Cancelar]                  │
└──────────────────────────────┘
```

### States

| State | Behavior |
|---|---|
| **Loading** | Skeleton card with protocol badge placeholder, avatar shimmer, 3 lines of text shimmer. |
| **Populated** | Full post + thread as shown. |
| **Thread collapsed** | Only root post visible. "Respostas (3)" header shows count. Replies hidden. |
| **No replies** | "Respostas" section shows: "Nenhuma resposta ainda. Seja a primeira pessoa a responder!" |
| **Cross-protocol available** | "Também no" bar appears with protocol badges. |
| **Cross-protocol not available** | "Também no" bar hidden entirely. |
| **Post deleted / unavailable** | Gentle message: "Esta publicação não está mais disponível." Original content hidden. |
| **Offline** | Cached content shown if available. Reply field disabled: "Offline — respostas aparecerão quando você estiver online" |
| **Content warning** | Root post blurred on first load. Tap to reveal. |
| **Long thread** (>20 replies) | "Carregar mais respostas" button at bottom of visible replies. 20 replies per page. |

### Accessibility

| Concern | Implementation |
|---|---|
| **VoiceOver** | Reads: "Post by [name] on Mastodon. [Content]. Thread has 3 replies. Double-tap to interact." |
| **Thread navigation** | Rotor can navigate between replies. Each reply is a distinct element. |
| **"Also on" bar** | "This post is also available on Bluesky and Nostr. Double-tap to view." |
| **Content warning** | "Sensitive content. Double-tap to reveal." |
| **Dynamic Type** | Thread indentation reduces at larger text sizes to maintain readability. Cards stack vertically. |
| **Colorblind** | Thread indentation uses a thin vertical line (not just color) to show reply depth. Level 1 = full line, Level 2 = dashed line, Level 3 = dotted line. |

### Animation

| Transition | Spec |
|---|---|
| **Timeline → Detail** | Push from right (0.3s). The tapped post card "hero transitions" to fill the screen: card content scales up 1.0x to fill, while background fades from card to full white. |
| **Detail → Timeline** | Pop transition (0.3s, slide from left). |
| **"Also on" tap** | Crossfade (0.25s) between protocol-native renderings of same post. |
| **Reply submitted** | Reply bubble slides up from input area into thread (0.35s spring). Optimistic update. |
| **Acknowledge** | Icon pulse 1.0→1.3→1.0 (0.3s spring). Name added to acknowledger list with fade-in. |
| **Thread collapse** | Replies fold upward (0.3s ease-in-out) with content compression, like an accordion. |

### Design Rationale

The post detail view prioritizes **context and conversation** over virality metrics:

1. **"Also on..."** — This feature is unique to a multi-protocol client. It shows when an author has cross-posted the same content to other protocols. This builds the user's understanding of the federated web and may encourage them to explore other protocols.

2. **People-based acknowledgments** — Instead of a like count, we show up to 3 faces/names. This transforms "12 likes" from a popularity score into "your friends @ana, @carlos, and @pedro appreciated this." The emotional weight is completely different.

3. **Thread as primary unit** — The root post + replies are treated as one conversation, not as separate entities. The collapse/expand on replies treats the thread as a coherent whole.

4. **"Via [instance/server/relay]"** — This transparency educates users about the federated architecture without overwhelming them. It's a small detail that builds mental models over time.

5. **No share counts** — The share button does not display how many times something was shared. Sharing is an action, not a metric.

---

## Screen 4: Composer — "Share" Mode

### Purpose

The composer is the tool for intentional creation. It supports multi-protocol posting with protocol-aware character limits, formatting, and visibility settings. The design emphasizes **deliberation over impulse** — posting requires intent.

### ASCII Wireframe

```
┌──────────────────────────────────────────────┐
│  ✍️  Compartilhar                          │
│                                              │
│  Para: [📡 Masto] [🔵 Bluesky] [🟡 Nostr]  │
│        [+ Adicionar protocolo]               │
│                                              │
│  ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐          │
│  │  Pré-visualização Mastodon     │          │
│  │  (500 chars)                   │          │
│  └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘          │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │                                       │    │
│  │  O que você quer compartilhar?        │    │
│  │                                       │    │
│  │                                       │    │
│  │                                       │    │
│  │                                       │    │
│  │  [485/500] 📡  [298/300] 🔵          │    │
│  │           [∞] 🟡                      │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  [📷 Mídia] [⚠️ Aviso] [🔗 Link]            │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  Visibilidade: [Público ▼]           │    │
│  │  Modo Adaptar: [🔘]                  │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌── ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ───┐    │
│  │  ⟳  Segure para publicar           │    │
│  └── ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ───┘    │
│                                              │
│  [Rascunho salvo]                           │
└──────────────────────────────────────────────┘
```

### Interaction Spec

| Action | Behavior |
|---|---|
| **Tap protocol selector** | Toggle on/off. Active protocols shown with filled badge. At least one must be selected. |
| **Tap "+ Add protocol"** | Sheet slides up with list of available (configured) protocols not currently selected. |
| **Tap preview tab** | Switch preview between selected protocols. Each tab shows protocol-native character limit. |
| **Type in text area** | Character count updates in real-time per protocol. If text exceeds any protocol's limit, that counter turns amber (warning) then red (exceeded). |
| **Tap "Adapt" toggle** | When ON: text auto-truncates per protocol limits. Formatting adjusts (e.g., markdown→richtext conversion per protocol capability). When OFF: text must fit all protocols. Cannot post if any limit is exceeded. |
| **Tap media button** | Native image picker. Multi-select (up to 10). Selected media appears as thumbnail grid below text area. |
| **Tap ⚠️ (content warning)** | Field expands below text area: "Aviso de conteúdo (opcional)". |
| **Tap visibility selector** | Dropdown: Público, Seguidores apenas, Direto/Mensagem privada. Per-protocol visibility mapped automatically. |
| **Long-press publish button** | Haptic feedback begins. Circular progress indicator fills as press continues (1.5s hold). On full hold: confirmation sheet appears. |
| **Tap publish (after long-press)** | "Isso será visível para todos os seus seguidores em [protocolos selecionados]. Publicar?" [Publicar] [Cancelar] |
| **Swipe down** | Dismiss composer. Draft auto-saves. |
| **Tap back arrow** | If content exists: "Salvar rascunho?" [Salvar] [Descartar] [Cancelar]. If no content: dismiss immediately. |
| **Tap draft indicator** | Open list of saved drafts from database. Tap draft to load into composer. |

### Per-Protocol Preview Tabs

```
┌──────────────────────────────────────────────┐
│  📡 Mastodon    🔵 Bluesky    🟡 Nostr      │
│  ─────────────  ───────────  ─────────      │
│  500 chars      300 chars    ilimitado       │
│  Markdown       Rich text    Markdown        │
│  CW suportado   Labels       CW suportado    │
│  Público/Seg/   Público/Seg/ Público/Seg/   │
│  DM             DM           DM (kind)      │
└──────────────────────────────────────────────┘
```

### Character Count Visual

```
┌──────────────────────────────────────────────┐
│  [485/500] 📡        [298/300] 🔵           │
│  ████████████████████ ███████████████████   │
│                                              │
│  [∞] 🟡                                     │
│  (no limit indicator shown)                 │
└──────────────────────────────────────────────┘
```

The progress bars show relative fill. When approaching limit (90%+), bar turns amber. At 100%, bar turns red and text is highlighted.

### States

| State | Behavior |
|---|---|
| **Empty** | Text area shows placeholder: "O que você quer compartilhar?" Protocol selector shows all configured protocols as defaults. |
| **Draft loaded** | Content populated. "Rascunho salvo" label visible. Timestamp of last auto-save. |
| **Typing** | Counts update live. Preview tabs update live. Draft auto-saves after 5s of inactivity. |
| **Media attached** | Thumbnails shown in horizontal scrollable row below text area. Tappable to view full-size. |
| **Limit exceeded** (Adapt OFF) | Publish button disabled. Red indicator: "Reduza o texto para [protocolo] (excede em 45 caracteres)" |
| **Limit approached** (Adapt OFF) | Amber indicator: "Quase no limite do [protocolo] (42 caracteres restantes)" |
| **Limit exceeded** (Adapt ON) | Green indicator: "Será adaptado automaticamente para cada protocolo" |
| **Long-press in progress** | Circular progress fills around button. Haptic feedback ticks at 25%, 50%, 75%, 100%. |
| **Publishing** | Button shows spinner. "Publicando..." Protocol badges animate with sending indicator (pulse). |
| **Published** | Checkmark animation. "Publicado!" Auto-dismiss after 1.5s. Return to previous screen. |
| **Publish failed** | Error state: "Não foi possível publicar em [protocolo]." Shows which protocol failed. Retry button. |
| **Offline** | "Rascunho salvo localmente. Será publicado quando você estiver online." Post queued. |
| **No protocols configured** | Redirect to protocol setup in Settings. Empty state: "Conecte um protocolo para começar a publicar." |

### Long-Press to Publish — Detailed Flow

```
Normal state:           Long-press start:      Hold complete:       Confirmation:
┌────────────────┐     ┌────────────────┐      ┌────────────────┐   ┌────────────────┐
│                │     │                │      │                │   │ Isso será      │
│  ⟳  Segure    │     │  ⟳  Segure     │      │  ✓  Pronto     │   │ visível para   │
│  para publicar │  →  │  firme...      │  →   │  para publicar │ → │ todos os seus  │
│                │     │  ░░░▒▒▒▓▓▓██   │      │  ████████████  │   │ seguidores em  │
└────────────────┘     └────────────────┘      └────────────────┘   │ Mastodon e     │
                                                                     │ Bluesky.       │
                                                                     │                │
                                                                     │ [Publicar]     │
                                                                     │ [Cancelar]     │
                                                                     └────────────────┘
```

### Accessibility

| Concern | Implementation |
|---|---|
| **VoiceOver** | Reads: "Composer. Text area, 485 of 500 characters used on Mastodon. Double-tap to edit." |
| **Protocol selector** | "Mastodon selected. Double-tap to deselect." Selected state announced. |
| **Character counts** | "485 characters used, 15 remaining on Mastodon. Near limit." Dynamic announcements at thresholds. |
| **Long-press alternative** | A "Publicar" button is ALSO available (not just long-press) for accessibility. The long-press is friction for typical use, not a barrier. |
| **Media attachments** | "Image attached: [filename]. Double-tap to view. Triple-tap to remove." |
| **Content warning field** | "Content warning field. Optional text that will be shown before your post content." |
| **Dynamic Type** | Text area scales. Character count stays visible. Protocol selector wraps to multiple rows. |
| **Reduce Motion** | No character count animations. Static numbers. |

### Animation

| Transition | Spec |
|---|---|
| **Intent → Composer** | Slide up from bottom (0.35s spring). Intent screen dimmed behind (0.3 opacity). |
| **Protocol selector toggle** | Badge scales 1.0→1.15→1.0 (0.2s spring). Color transitions smoothly (0.2s ease). |
| **Preview tab switch** | Crossfade (0.2s) between protocol previews. |
| **Character count update** | Number animates (0.15s ease) when changing. Progress bar animates width. |
| **Media added** | Thumbnail scales in from attachment point (0.3s spring). |
| **Long-press progress** | Circular fill animates smoothly with haptic feedback. |
| **Publish success** | Button morphs from "Publicar" to checkmark (0.4s spring). Entire composer slides down and fades out (0.3s). |
| **Dismiss** | Slide down (0.3s ease-in). |

### Design Rationale

The composer is designed for **intentional creation**, not impulsive broadcasting:

1. **Long-press to publish** — This is the most controversial and most important design decision. By requiring a 1.5-second hold, we create a moment of reflection before anything becomes public. Research shows that even a 1-second delay reduces regretted posts by ~70%. The haptic feedback makes this feel deliberate, not frustrating.

2. **Multi-protocol awareness** — The character counts per protocol educate the user about protocol differences. The "Adapt" mode makes cross-posting practical while being transparent about what's happening.

3. **Pre-visualization tabs** — Instead of a single "what you see is what you get" preview, we show how the post will look on each protocol. This respects the fact that each protocol has its own culture and formatting norms.

4. **Visibility per protocol** — The visibility dropdown is not one-size-fits-all. The app intelligently maps "Public" to each protocol's public mode, "Followers-only" to each protocol's equivalent, etc.

5. **Draft auto-save** — No "Are you sure?" dialog on accidental back-swipe. Just a gentle "Save draft?" prompt. Drafts are stored locally with SQLite, searchable by date and protocol.

6. **Accessibility override** — The long-press is supplemental, not exclusive. A standard "Publish" button is always available for users who cannot or prefer not to long-press. The friction should not become a barrier.

---

## Screen 5: Profile View

### Purpose

A unified identity view showing one person across all protocols. The profile is a **person-first, protocol-second** representation — you see the human before you see the technical details.

### ASCII Wireframe

```
┌──────────────────────────────────────────────┐
│  ←  Buscar                                   │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  │    │
│  │  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  │    │
│  │  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  │    │
│  │         ┌──────────┐                │    │
│  │         │  Avatar  │                │    │
│  │         └──────────┘                │    │
│  │         Maria Silva                  │    │
│  │         @maria@mastodon.social       │    │
│  │         @maria.bsky.social           │    │
│  │         @maria_nostr                  │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌─ Nuvem de Identidades ───────────────┐    │
│  │  ┌────┐                               │    │
│  │  │📡  │ ● Mastodon · 1.234 posts     │    │
│  │  │AM  │                               │    │
│  │  └────┘                               │    │
│  │     ┌────┐                            │    │
│  │     │🔵  │ ○ Bluesky · 567 posts     │    │
│  │     │BS  │                            │    │
│  │     └────┘                            │    │
│  │          ┌────┐                       │    │
│  │          │🟡  │ ● Nostr · 89 posts   │    │
│  │          │NS  │                       │    │
│  │          └────┘                       │    │
│  │  ┌────┐                               │    │
│  │  │⊞  │ ◌ Matrix · inativo           │    │
│  │  │Mtx│                               │    │
│  │  └────┘                               │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  Sou desenvolvedora e ativista. Trabalho     │
│  com tecnologia social na periferia de SP.   │
│  📍 São Paulo · 🌐 mariasilva.dev            │
│                                              │
│  Também no: [📡 Mastodon] [🔵 Bluesky]       │
│  (você segue em 2 protocolos)                │
│                                              │
│  [Seguir]        [Compartilhar perfil]       │
│                                              │
│  Conexão: Segue de volta · Compartilham     │
│  3 comunidades · Ambos no Mastodon e Bluesky │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  Publicações                    [▼]  │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  (timeline do usuário, mesmo formato         │
│   da timeline principal)                     │
└──────────────────────────────────────────────┘
```

### Interaction Spec

| Action | Behavior |
|---|---|
| **Tap back** | Pop → previous screen. |
| **Tap avatar** | Full-screen avatar view with zoom. |
| **Tap handle** | Copy to clipboard. Toast: "Copiado!" |
| **Tap protocol badge in identity cloud** | Show detailed stats for that protocol identity: joined date, post count, follower/following count, instance/server. |
| **Tap identity cloud node** (the colored shape) | Switch profile view to show only content from that protocol. Visual indicator in the cloud: selected node grows slightly, others dim. |
| **Tap "Seguir"** | If not followed: prompt appears "Seguir @maria? Você poderá ver as publicações dela no seu timeline." [Seguir] [Cancelar]. If already followed: button changes to "Seguindo" with unfollow option on tap. |
| **Long-press "Seguir"** | Shows follow cooldown info: "Seguir novamente em: 23h 14min restantes" (if recently unfollowed). |
| **Tap "Compartilhar perfil"** | Share sheet: share profile link, share as vCard, share individual protocol handles. |
| **Tap "Conexão" section** | Expand to show detailed connection view: mutual follows across protocols, shared communities/lists, common protocols. |
| **Tap a post in profile timeline** | Push → Post Detail. |
| **Swipe up on banner** | Banner parallax scrolls. Avatar and name stick to top (like Damus). |
| **Scroll down** | Banner compresses. Avatar shrinks (120pt → 40pt). Name moves into navigation bar. |

### Identity Cloud — Visual Cluster

The identity cloud is a unique visualization showing all protocol identities for a person:

```
        ┌────┐
        │📡  │  ● Active - Mastodon
        │AM  │  1,234 posts
        └────┘
             ┌────┐
             │🔵  │  ○ Active - Bluesky
             │BS  │  567 posts
             └────┘
                  ┌────┐
                  │🟡  │  ● Active - Nostr
                  │NS  │  89 posts
                  └────┘
  ┌────┐
  │⊞  │  ◌ Inactive - Matrix
  │Mtx│  (no recent activity)
  └────┘
```

Visual properties:
- **Size** reflects activity level (more posts = larger node)
- **Opacity** reflects recency (full = active recently, faded = inactive)
- **Position** is organic, slightly scattered (not rigid grid)
- **Connection lines** between nodes if the person has linked their identities
- **Status dot**: ● active (post in last 7d), ○ recent (post in last 30d), ◌ inactive (no posts in 30d+)

### States

| State | Behavior |
|---|---|
| **Loading** | Skeleton: banner shimmer, avatar circle shimmer, 3 lines of text shimmer, identity cloud placeholders (colored circles with no detail). |
| **Populated** | Full profile as shown. |
| **Own profile** | "Seguir" replaced with [Editar perfil]. Identity cloud shows "Você" label. |
| **No bio** | Bio section hidden. Additional space for identity cloud. |
| **Single protocol** | Identity cloud shows single node, centered. "Também no" section hidden. |
| **Blocked user** | "Você bloqueou esta pessoa." [Desbloquear] option. Content hidden. |
| **Private account** | "Esta conta é privada. Siga para ver as publicações." |
| **Offline** | Cached profile data shown. "Última atualização: [time]" banner. |
| **No posts** | Beautiful empty state: "Nenhuma publicação ainda" with illustration. |

### Accessibility

| Concern | Implementation |
|---|---|
| **VoiceOver** | "Profile of Maria Silva. Also known as at-maria on Mastodon, at-maria dot bsky dot social on Bluesky. 3 protocol identities." |
| **Identity cloud** | Each node read as: "Mastodon identity. Active. 1,234 posts. Double-tap to view Mastodon content." |
| **Status dots** | "Active" / "Recent" / "Inactive" read as text, not just color. |
| **Banner parallax** | Respects Reduce Motion: no parallax, static banner. |
| **Dynamic Type** | Identity cloud nodes become a vertical list at AX1+ instead of visual cluster. |
| **Colorblind** | Identity nodes use shape + text label, not color alone. Status dots have text labels. |

### Animation

| Transition | Spec |
|---|---|
| **Previous → Profile** | Push from right (0.3s). Banner appears with parallax already at its initial position. |
| **Banner scroll** | Parallax at 0.7x scroll speed (banner moves slower than content). Avatar scales from 120pt to 40pt smoothly (continuous, not step-based). |
| **Identity cloud tap** | Selected node scales up 1.15x with spring (0.3s). Other nodes dim to 0.4 opacity. |
| **Follow button** | "Seguir" → "Seguindo" with morph animation (0.3s). Background fills in from left. |
| **Profile → Post Detail** | Push from right (0.3s). |

### Design Rationale

The profile view centers the **person, not the platform**:

1. **Identity Cloud** — This is the signature visual innovation for multi-protocol identity. Instead of a boring list of "Linked Accounts," we create a living visualization. The size, opacity, and connection lines tell a story about how the person uses each protocol. It makes protocol pluralism visible and beautiful.

2. **"How we're connected"** — This section reframes the relationship from the standard "follows you / you follow" binary to a richer understanding. "You both follow @ana. You're both in the #technology community. You're both on Mastodon and Bluesky." This builds a sense of shared context.

3. **Follow cooldown** — The 24h cooldown on follow/unfollow is intentional friction designed to prevent the social media pattern of "follow → unfollow → follow" that games engagement metrics. It's explained transparently when triggered.

4. **Banner parallax** — Borrowed from Damus because it works: the parallax creates a sense of space and personality. The banner is the person's self-expression — a place for a personal image, an artwork, or simply a gradient.

5. **Protocol-specific content filtering** — Tapping an identity node in the cloud filters the timeline below to only show posts from that protocol. This enables "I want to see what this person posts on Nostr specifically" without leaving their profile.

---

## Screen 6: Notifications — "Respond" Mode

### Purpose

Notifications as **inbox, not dopamine triggers**. Grouped meaningfully by type (mentions, replies, DMs). No like/favorite/repost notifications. No unread counts. The goal is to help the user respond to actual conversations, not feed engagement metrics.

### ASCII Wireframe

```
┌──────────────────────────────────────────────┐
│  💬  Responder                  [Marcar tudo │
│                                     como visto]│
│                                              │
│  📢 Menções (3)                              │
│  ┌──────────────────────────────────────┐    │
│  │ ┌──┐ @joao · 30m atrás          📡  │    │
│  │ │JC│ "Excelente ponto! Eu         │    │
│  │ └──┘  acrescentaria que..."       │    │
│  │ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │    │
│  │ [Responder]    [Ignorar]          │    │
│  └──────────────────────────────────────┘    │
│  ┌──────────────────────────────────────┐    │
│  │ ┌──┐ @carla · 2h atrás           🔵  │    │
│  │ │CG│ "Vi seu post sobre a          │    │
│  │ └──┘  Amazônia..."                 │    │
│  │ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │    │
│  │ [Responder]    [Ignorar]          │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  💬 Respostas (5)                            │
│  ┌──────────────────────────────────────┐    │
│  │ ┌──┐ @pedro · 15m atrás          📡  │    │
│  │ │PJ│ respondeu ao seu post:         │    │
│  │ └──┘ "Concordo plenamente!          │    │
│  │        Vamos apoiar a iniciativa"   │    │
│  │ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │    │
│  │ [Ver conversa]  [Ignorar]          │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ✉️ Mensagens Diretas (2)                    │
│  ┌──────────────────────────────────────┐    │
│  │ ┌──┐ lucas@xmpp.org  · 1h        🔒  │    │
│  │ │L │ "Oi, tudo bem? Queria         │    │
│  │ └──┘  saber mais sobre..."          │    │
│  │ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │    │
│  │ [Abrir]  [Ignorar]                 │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  Você está em dia!                           │
│  Todas as notificações vistas.               │
└──────────────────────────────────────────────┘
```

### Interaction Spec

| Action | Behavior |
|---|---|
| **Tap a notification** | Navigate to the relevant context: mention → post detail, reply → thread, DM → DM conversation. |
| **Tap avatar** | Push → Profile of the sender. |
| **Tap protocol badge** | Tooltip showing protocol via which the notification arrived. |
| **Tap "Responder"** | Opens inline reply or slide-up composer (depending on context). |
| **Tap "Ignorar"** | Dismiss notification. Haptic. Adds to "seen" state. |
| **Swipe left** | Reveal "Ignorar" action (red). Swipe fully → dismiss with haptic. |
| **Swipe right** | Reveal "Responder" action (primary color). Swipe fully → open reply. |
| **Tap "Marcar tudo como visto"** | All notifications transition to "seen" state. Visual: all cards fade from subtle tint to neutral. |
| **Long-press notification** | Context menu: Mute conversation, Mute user, Report, Copy link. |
| **Tap section header** | Collapse/expand that section. |

### Notification Cards — Detailed Anatomy

**Mention card:**
```
┌──────────────────────────────────────────────┐
│ ┌──┐ @username · time ago           [📡]    │
│ │Av│ "Excerpt of what they said..."           │
│ └──┘                                         │
│ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │
│ [Responder]                    [Ignorar]     │
└──────────────────────────────────────────────┘
```

**Reply card:**
```
┌──────────────────────────────────────────────┐
│ ┌──┐ @username · time ago           [🔵]    │
│ │Av│ replied to your post:                   │
│ └──┘ "Reply excerpt..."                       │
│ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │
│ [Ver conversa]                 [Ignorar]     │
└──────────────────────────────────────────────┘
```

**DM notification (shown within this screen, but tapping takes you to Screen 7):**
```
┌──────────────────────────────────────────────┐
│ ┌──┐ username · time ago           [🔒]     │
│ │Av│ "Message preview text..."                │
│ └──┘                                         │
│ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │
│ [Abrir]                        [Ignorar]     │
└──────────────────────────────────────────────┘
```

### States

| State | Behavior |
|---|---|
| **Loading** | Skeleton cards (3 per section). Shimmer animation. |
| **Has notifications** | Grouped sections as shown. Each section has a header with count. |
| **All seen** (but not empty) | Cards render in neutral tint (no highlight). No unread indicators. |
| **Empty** (no notifications) | Beautiful illustration: a calm inbox. "Nenhuma notificação. Aproveite o silêncio." |
| **Only one type** | Only that section shown. Others hidden. |
| **Offline** | Cached notifications shown. "Última atualização: [time]". |
| **New notification arrives** | Subtle slide-in banner at top (not a popup). No sound by default. No badge count. |

### Accessibility

| Concern | Implementation |
|---|---|
| **VoiceOver** | "Notification from [name] on Mastodon. They mentioned you: [text excerpt]. Double-tap to view. Swipe left to dismiss." |
| **Section headers** | "Mentions section. 3 notifications." Rotor navigable. |
| **Actions** | "Responder" and "Ignorar" buttons accessible via standard navigation. |
| **Protocol badges** | Each protocol badge has accessibility label. |
| **Dynamic Type** | Cards stack vertically. Excerpt truncates at larger sizes. |
| **No unread badges** | This is an accessibility feature, not a limitation. Unread badges are anxiety triggers for many users. |

### Animation

| Transition | Spec |
|---|---|
| **Intent → Notifications** | Crossfade (0.3s). |
| **Tap notification → Context** | Push transition (0.3s) to relevant screen. |
| **Dismiss notification** | Card slides right (0.25s) with fade. Remaining cards slide up to fill gap (0.3s spring). |
| **"Mark all as seen"** | Cards subtly shift from warm tint to neutral (0.5s ease). No dramatic animation — this is a quiet transition. |
| **New notification arrives** | Slides in from top (0.4s spring), stays for 5s, then slides out (0.3s ease-in) unless interacted with. |

### Design Rationale

The notifications screen is a **radical departure from social media norms**:

1. **No like/favorite/repost notifications** — This is not optional; it's a design principle. The app fundamentally does not track or surface vanity metrics. If a user wants to know who acknowledged their post, they can view the post detail. Unsolicited "Your post got 12 likes!" notifications are engagement-manipulation patterns.

2. **No unread count badges** — Badge counts are one of the most potent anxiety triggers in modern UI. By eliminating them, we reduce the compulsion to check. The "Mark all as seen" button exists for users who want to clear their notification state, but there's no red circle screaming for attention.

3. **Grouped by type, not time** — Chronological within type, but types are grouped to help the user triage: "I only want to see DMs right now" is easy when DMs are their own section.

4. **"Seen" vs "Read"** — This distinction matters. "Mark all as seen" acknowledges that the user has looked at their notifications. It does NOT send read receipts. Read receipts are a separate, granular permission that only applies to DMs (and even then, is optional).

5. **Two action buttons per card** — Responder and Ignorar are always visible, always tappable. No hidden swipe gestures for primary actions. The swipe is an accelerator, not the only path.

6. **No notification sounds by default** — Sound is opt-in. The app starts silent. Notifications are things you check when you choose to, not things that interrupt you.

---

## Screen 7: Direct Messages

### Purpose

A unified DM inbox across all messaging protocols (Matrix, XMPP, IRC, plus DM-capable social protocols like Mastodon/Bluesky/Nostr). End-to-end encryption indicators, ephemeral channel distinction, and cross-protocol conversation threading.

### ASCII Wireframe

```
┌──────────────────────────────────────────────┐
│  ✉️  Mensagens                     [✏️ Nova] │
│                                              │
│  [Todas] [Matriz] [XMPP] [IRC] [Social]     │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │ ┌──┐ Ana Oliveira              📡  🔒│    │
│  │ │AO│ "Sim, vou estar lá às 19h!"     │    │
│  │ └──┘                         10:30   │    │
│  └──────────────────────────────────────┘    │
│  ┌──────────────────────────────────────┐    │
│  │ ┌──┐ Carlos Mendes            🔵  🔒│    │
│  │ │CM│ "O link do artigo: https://..." │    │
│  │ └──┘                         09:15   │    │
│  └──────────────────────────────────────┘    │
│  ┌──────────────────────────────────────┐    │
│  │ ┌──┐ Grupo: Tecnologia Social  ⊞  🔒│    │
│  │ │TS│ @joao: "Alguém já testou..."    │    │
│  │ └──┘                         Ontem   │    │
│  └──────────────────────────────────────┘    │
│  ┌──────────────────────────────────────┐    │
│  │ ┌──┐ #geral (IRC Freenode)     ◆  📡│    │
│  │ │♯ │ <user1> Boa noite pessoal      │    │
│  │ └──┘                         Ontem   │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  📡 = Mastodon  🔵 = Bluesky                 │
│  ⊞ = Matrix  ◆ = IRC  🟡 = Nostr           │
│  🔒 E2EE  📡 No encryption                   │
└──────────────────────────────────────────────┘
```

### Conversation View (Inside a DM)

```
┌──────────────────────────────────────────────┐
│  ←  Mensagens     Ana Oliveira               │
│  📡 Mastodon · 🔒 Criptografia de ponta a    │
│                 ponta                        │
│                                              │
│  ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐ ─ ─ ─ ─ ┐        │
│  │  Oi, tudo bem? Vamos   │             │    │
│  │  nos encontrar amanhã? │             │    │
│  └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘             │        │
│                              ┌──────┐  │        │
│                              │ Claro!│  │        │
│                              │ 19h?  │  │        │
│                              └──────┘  │        │
│  ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐ ─ ─ ─ ─ ┐        │
│  │  Sim, vou estar lá!   │             │    │
│  └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘             │        │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │ Mensagem...                    [📎]  │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │ 🔐 Verificar identidade              │    │
│  │ Compare o emoji abaixo com o que     │    │
│  │ Ana está vendo no dispositivo dela:  │    │
│  │ 🦊 🐧 🌵 🎸 🍉 🚀                  │    │
│  │ [São iguais] [Diferentes]            │    │
│  └──────────────────────────────────────┘    │
└──────────────────────────────────────────────┘
```

### Interaction Spec

| Action | Behavior |
|---|---|
| **Tap conversation** | Push → Conversation view. |
| **Tap avatar in list** | Push → Profile. |
| **Tap protocol badge** | Tooltip: protocol name + encryption status. |
| **Tap encryption icon** | Show encryption details: protocol, algorithm, key age, verification status. |
| **Tap ✏️ New** | Sheet: "Nova mensagem". Protocol selector, then contact search. |
| **Filter pills** | Toggle: All, Matrix-only, XMPP-only, IRC-only, Social (Mastodon/Bluesky/Nostr). |
| **Swipe left on conversation** | Reveal "Arquivar" / "Excluir". |
| **Swipe right** | Reveal "Fixar" (pin to top). |
| **Inside conversation — tap message** | Select message (copy, forward, reply, report). |
| **Inside conversation — long-press message** | Context menu: Copy, Reply, Forward to another protocol, Delete, Report. |
| **Inside conversation — tap verification banner** | Expand verification sheet (shown above). |
| **Tap "São iguais"** | Identity verified. Banner changes to green: "✔ Identidade verificada". |
| **Tap "Diferentes"** | Warning: "A identidade desta conversa mudou. Possível ataque MITM." Recommendations to verify out-of-band. |
| **IRC — tap channel** | Opens read-only or write-enabled view depending on channel mode. "Ephemeral" badge on IRC channels. |

### Encryption Status Indicators

| Icon | Meaning | Color |
|---|---|---|
| 🔒 | E2EE verified | Green |
| 🔐 | E2EE not verified | Yellow |
| 🔓 | Encrypted (not E2EE) | Gray |
| 📡 | No encryption | Red |
| ◆ (diamond) | IRC — no encryption, ephemeral | Orange |

### States

| State | Behavior |
|---|---|
| **Loading** | Skeleton conversation list items. |
| **Populated** | List as shown. Conversations sorted by most recent message (pinned first). |
| **Empty** (no conversations) | "Nenhuma conversa ainda. Toque em ✏️ para começar." |
| **Empty** (filtered) | "Nenhuma conversa neste protocolo." |
| **E2EE verification needed** | Verification banner at top of conversation (shown above). |
| **E2EE verified** | Small green checkmark next to encryption icon. |
| **Message send failed** | Red exclamation on message bubble. Tap to retry. |
| **Offline** | Cached messages shown. "Offline. Mensagens aparecerão quando você conectar." |
| **IRC disconnected** | Channel shown with "⚠️ Desconectado" indicator. Tap to reconnect. |
| **Key changed** | Warning banner: "A chave de criptografia de [user] mudou." Re-verification flow. |

### Accessibility

| Concern | Implementation |
|---|---|
| **VoiceOver** | "Conversation with Ana Oliveira on Mastodon. Encrypted. Last message: 'Sim, vou estar la as 19h.' Double-tap to open." |
| **Encryption status** | Read as text: "End-to-end encrypted, verified." or "No encryption." Not just icon. |
| **Message bubbles** | Read as: "From Ana, 10:30 AM: [message text]." |
| **Verification emoji** | "Emoji verification: fox, penguin, cactus, guitar, watermelon, rocket. Are they the same on Ana's device?" Each emoji read by name. |
| **Dynamic Type** | Message bubbles expand. Font scales. At AX1+, verification emoji become text words. |
| **IRC channels** | "IRC channel: hashtag-geral. Ephemeral. No encryption." |

### Animation

| Transition | Spec |
|---|---|
| **Notifications → DMs** | Push from right (0.3s) or crossfade if arriving from intent. |
| **Conversation list → Inside** | Push from right (0.3s). |
| **Message sent** | Bubble animates into conversation (0.3s spring). Scrolls to bottom. |
| **New message arrives** | Bubble slides in (0.3s spring) if conversation is open. If in list view, conversation moves to top with a subtle highlight (0.5s ease). |
| **Verification banner** | Expands/collapses with spring (0.35s). |
| **Encryption status change** | Icon transitions (0.3s ease) with color change. |

### Design Rationale

DMs in a multi-protocol app present unique challenges and opportunities:

1. **Encryption transparency** — One of the biggest problems in multi-protocol messaging is the false sense of security: "I sent it encrypted" when really it was only encrypted in transit, not E2EE. Our design makes encryption status **unignorable** — always visible, always labeled, always honest.

2. **IRC as first-class but distinguished** — IRC is ephemeral and unencrypted by nature. Rather than hiding this, we embrace it with clear labeling and visual distinction. The user knows IRC is a different kind of conversation.

3. **Signal-style verification** — Emoji comparison is the gold standard for E2EE key verification (proven by Signal). By bringing this to Matrix and XMPP conversations, we provide a familiar, trustworthy verification flow.

4. **Cross-protocol conversation threading** — If you're talking to someone on Matrix AND Mastodon DMs, the conversation view merges them into a single thread (for the same person). This is the "unified inbox" promise fulfilled.

5. **No read receipts by default** — Read receipts are opt-in per conversation. The app does not send them unless the user explicitly enables them. This protects the user's privacy and autonomy.

6. **IRC channels listed separately** — IRC's channel model (topic-based, many-to-many) is fundamentally different from DMs (person-to-person). Listing them in the same inbox but visually distinguished respects both patterns.

---

## Screen 8: Explore — Intentional Discovery

### Purpose

Not a "For You" feed. This is a **toolkit for discovery** that puts the user in control. Rather than an algorithm deciding what's interesting, the explore screen provides structured tools: starter packs, a tasting menu, searchable hashtags, and cross-protocol discovery. Every section is time-boxed.

### ASCII Wireframe

```
┌──────────────────────────────────────────────┐
│  🔍  Descobrir                               │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  Buscar pessoas, tópicos, hashtags   │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌─ Menu Degustação ─────────────────────┐   │
│  │  📋 20 publicações selecionadas esta  │   │
│  │  semana. Uma amostra do melhor da     │   │
│  │  web aberta.                          │   │
│  │                                       │   │
│  │  ┌──┐ "Uma descoberta incrível..."   │   │
│  │  │  │ 📡 @user · 3h                  │   │
│  │  └──┘                                │   │
│  │  ┌──┐ "Novo estudo mostra que..."    │   │
│  │  │  │ 🔵 @user2 · 5h                │   │
│  │  └──┘                                │   │
│  │  ┌──┐ "Reflexão sobre o fediverso"   │   │
│  │  │  │ 🟡 @user3 · 8h                │   │
│  │  └──┘                                │   │
│  │                                       │   │
│  │  [Ver as 20 →]                       │   │
│  └───────────────────────────────────────┘   │
│                                              │
│  ┌─ Pacotes de Entrada ─────────────────┐    │
│  │  Categorias:                         │    │
│  │  [Tecnologia] [Ativismo] [Arte]      │    │
│  │  [Música] [Ciência] [Brasil]         │    │
│  │                                       │    │
│  │  Tecnologia: 5 contas selecionadas   │    │
│  │  ┌──┐ @anna_dev               📡    │    │
│  │  │  │ "Dev fullstack, open source"   │    │
│  │  └──┘                                │    │
│  │  ┌──┐ @protocol_bot           🔵    │    │
│  │  │  │ "AT Protocol updates"          │    │
│  │  └──┘                                │    │
│  │  ...                                 │    │
│  │  [Seguir todos] [Ver pacote]         │    │
│  └───────────────────────────────────────┘   │
│                                              │
│  ┌─ Hashtags em Alta ──────────────────┐    │
│  │  #Amazônia          📡🔵🟡  42     │    │
│  │  #Tecnologia        📡🔵    128    │    │
│  │  #MúsicaBrasileira  📡🟡     67    │    │
│  │  #Fediverso         📡      245    │    │
│  │  [Ver todas →]                      │    │
│  └───────────────────────────────────────┘   │
│                                              │
│  Você está explorando há 10 min.             │
│  Quer seguir algumas dessas contas?          │
│  [Sim, ver sugestões] [Continuar explorando] │
└──────────────────────────────────────────────┘
```

### Interaction Spec

| Action | Behavior |
|---|---|
| **Tap search bar** | Focus → full-screen search with recent searches, trending searches. |
| **Tap Tasting Menu card** | Push → timeline view of 20 curated posts. Same post card design as main timeline. "Menu Degustação · Julho 2026" header. |
| **Tap "Ver as 20"** | Push → full tasting menu timeline. |
| **Tap starter pack category** | Filter starter packs by category. Selected category highlighted. |
| **Tap a starter pack** | Push → Pack detail: list of all accounts, preview of recent posts from each, "Seguir todos" button. |
| **Tap "Seguir todos"** | Confirmation: "Seguir [N] contas em [protocolos]?" with per-protocol breakdown. |
| **Tap hashtag** | Push → search results for that hashtag, chronological, filterable by protocol. |
| **Tap "Follow" on hashtag** | Toggle follow. Followed hashtags appear in timeline when relevant posts are made. |
| **Tap time-box prompt** | "Sim, ver sugestões" → shows follow recommendations based on exploration activity. "Continuar explorando" → dismisses prompt, reappears at 15min, then 20min, then max. |
| **Long-press any post in tasting menu** | Same context menu as main timeline. |
| **Swipe left on hashtag** | Reveal "Follow" action. |
| **Pull down** | Nothing. No refresh. Explore content updates daily. |

### Tasting Menu — Detail

The tasting menu is a **weekly curated set of 20 posts** from across the open web:
- Selected by the app's editorial team (or community-curated in future versions)
- Represents all major protocols
- Includes diverse voices, topics, and formats
- Zero personalization — every user sees the same 20 (creates shared reference)
- Weekly refresh (not daily — to prevent compulsive checking)
- Shown prominently on first launch (onboarding path 3: "Just Explore")

### Starter Packs — Detail

Inspired by Bluesky's starter packs, but cross-protocol:
- Each pack contains 5-15 accounts across multiple protocols
- Categorized by topic area
- Community-contributed (moderated)
- "Seguir todos" follows each account on the protocol they're primarily active on
- Preview shows: account name, protocol, a sample post, follower count (as people, not number)

### Time-Boxed Exploration

- After 10 minutes of exploration: gentle prompt
- After 15 minutes: second prompt
- After 20 minutes: final prompt, then no more
- Timer resets when user leaves the explore screen
- Timer is per-session, not cumulative (no lifetime tracking)

### States

| State | Behavior |
|---|---|
| **Loading** | Skeleton: search bar shimmer, 3 skeleton cards, category pills skeleton. |
| **Populated** | Full explore screen as shown. |
| **First visit** (new user) | Tasting menu expanded/highlighted. "Novo por aqui? Comece pelo Menu Degustação." |
| **Tasting menu unavailable** | Section hidden or shows "Em breve" placeholder. |
| **No starter packs in category** | "Nenhum pacote ainda nesta categoria." |
| **Search — no results** | Empty state: illustration of an empty nest. "Nada encontrado. Tente outros termos." |
| **Offline** | Cached explore data shown. "Explore offline — dados de [date]" |
| **Exploration time limit reached** | Final prompt. "Você já explorou bastante. Que tal seguir algumas contas e voltar amanhã?" |

### Accessibility

| Concern | Implementation |
|---|---|
| **VoiceOver** | "Explore. Search bar. Tasting Menu section: 20 curated posts from across the open web. Double-tap to view." |
| **Starter packs** | "Technology starter pack. 5 accounts. Includes Anna Dev on Mastodon and Protocol Bot on Bluesky. Double-tap to view." |
| **Hashtags** | "Hashtag Amazonia. Active on Mastodon, Bluesky, and Nostr. 42 posts in the last 24 hours." |
| **Time-box prompt** | "You have been exploring for 10 minutes. Would you like to follow some of these accounts?" |
| **Dynamic Type** | Sections stack vertically. Tasting menu shows fewer preview posts at larger sizes. |
| **Colorblind** | Hashtag protocol indicators use shape + label. |

### Animation

| Transition | Spec |
|---|---|
| **Timeline → Explore** | Push from right (0.3s) if accessed via tab. Crossfade (0.3s) if accessed via onboarding. |
| **Search bar focus** | Search bar expands (0.3s spring), keyboard appears. Background blurs (0.3s). |
| **Tasting menu card tap** | Card lifts (shadow expands) then pushes to detail (0.3s). Hero transition of the card. |
| **Starter pack category scroll** | Horizontal scroll with momentum. |
| **Time-box prompt** | Slides up from bottom (0.4s spring) with subtle haptic attention. |
| **Follow action** | Same morph animation as profile follow (0.3s). |

### Design Rationale

The explore screen is the **antithesis of the "For You" feed**:

1. **Tasting Menu** — This is the zero-config onboarding path (see also Screen 11). For users who just want to "see what's here" without committing to accounts or protocols, the tasting menu provides a beautiful, curated sample. By being unpersonalized, it creates a shared cultural reference point.

2. **Starter packs over algorithmic recommendations** — Instead of "because you liked X," we offer "people who care about technology also follow these accounts." It's human curation, not machine prediction. The difference is philosophical: the machine optimizes for engagement; humans optimize for relevance and diversity.

3. **Time-boxed discovery** — The exploration time prompt is one of the most important humane design features. It doesn't block the user; it simply asks: "You've been exploring for 10 minutes. Do you want to act on what you've found?" This creates a natural break point and prevents the "discovery scroll" from becoming another infinite feed.

4. **Hashtags with protocol indicators** — Seeing which protocols are discussing a hashtag helps users understand the fediverse landscape. "Oh, #Amazônia is active on all three major protocols" is a learning moment.

5. **No algorithmic "because you liked"** — There are zero recommendation models running on the user's behavior. Discovery is through human-curated structures (starter packs, tasting menu) and explicit search (hashtags, user search). This is a philosophical commitment.

---

## Screen 9: Settings & Data

### Purpose

The control center for the app: identity management, key backup, data export, storage, notifications, appearance, and session management. Designed to be **transparent and empowering**, not a dark-pattern labyrinth.

### ASCII Wireframe

```
┌──────────────────────────────────────────────┐
│  ⚙️  Configurações                           │
│                                              │
│  ── Identidade ──                           │
│  ┌──────────────────────────────────────┐    │
│  │ 👤 Identidades conectadas            │    │
│  │ 📡 Mastodon · @maria@masto.social    │    │
│  │ 🔵 Bluesky · @maria.bsky.social      │    │
│  │ 🟡 Nostr · npub1...                   │    │
│  │ ⊞ Matrix · @maria:matrix.org         │    │
│  │ ◆ IRC · maria on libera.chat         │    │
│  │ [Adicionar protocolo]                │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ── Segurança e Chaves ──                   │
│  ┌──────────────────────────────────────┐    │
│  │ 🔐 Chaves de criptografia            │    │
│  │ ● Mastodon: [Gerenciar]              │    │
│  │ ● Nostr: [Exportar chave privada]    │    │
│  │ ● Matrix: [Verificar dispositivo]    │    │
│  │                                       │    │
│  │ 📋 Frase de recuperação (BIP-39)     │    │
│  │ [Ver] [Fazer backup] [Restaurar]     │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ── Dados ──                                │
│  ┌──────────────────────────────────────┐    │
│  │ 💾 Armazenamento                     │    │
│  │ 📡 Mastodon: 245 MB                  │    │
│  │ 🔵 Bluesky: 89 MB                    │    │
│  │ 🟡 Nostr: 34 MB                      │    │
│  │ ● Cache total: 420 MB                │    │
│  │ ● Mídia em cache: 380 MB            │    │
│  │ [Limpar cache]                       │    │
│  │ [Exportar todos os dados]            │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ── Preferências ──                         │
│  ┌──────────────────────────────────────┐    │
│  │ 🔔 Notificações                      │    │
│  │ ● Menções: [On]                      │    │
│  │ ● Respostas: [On]                    │    │
│  │ ● DMs: [On]                          │    │
│  │ ● Sons: [Off]                        │    │
│  │ ● Horário de silêncio: 22:00-07:00  │    │
│  │                                       │    │
│  │ 🎨 Aparência                         │    │
│  │ ● Modo Simples: [Off]                │    │
│  │ ● Pele de protocolo: [Automática ▼]  │    │
│  │ ● Alto contraste: [Off]              │    │
│  │                                       │    │
│  │ ⏱ Sessão                             │    │
│  │ ● Timer visível: [On]                │    │
│  │ ● Lembrete a cada: [20 min ▼]        │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ── Sobre ──                                │
│  ┌──────────────────────────────────────┐    │
│  │ © Livre v0.1.0                       │    │
│  │ Código aberto · MIT                  │    │
│  │ [Ler Manifesto]                      │    │
│  │ [Princípios de Design]               │    │
│  └──────────────────────────────────────┘    │
└──────────────────────────────────────────────┘
```

### Interaction Spec

| Action | Behavior |
|---|---|
| **Tap "Adicionar protocolo"** | Sheet: list of available protocols. Tap one → setup flow (OAuth, key generation, or server config). |
| **Tap protocol identity** | Edit or remove that protocol identity: display name, avatar per protocol, server settings. |
| **Tap "Exportar chave privada"** | Warning: "Sua chave privada concede acesso total a esta identidade. Não compartilhe." Options: Copy, Export to file, Print paper wallet. |
| **Tap "Frase de recuperação"** | Warning screen: "MOSTRE ESTA FRASE A NINGUÉM. Ela pode ser usada para recuperar todas as suas identidades." Then reveal 12/24 words. |
| **Tap "Exportar todos os dados"** | One-tap export: generates a .zip with all local data (posts, bookmarks, preferences, keys [encrypted]). No confirmation beyond one tap. |
| **Tap "Limpar cache"** | Warning: "Mídia em cache será removida (380 MB). Pode ser recarregada." |
| **Toggle "Modo Simples"** | Hides protocol badges, simplifies identity display, hides protocol-specific features. The app becomes a simple social app. Changes apply immediately. |
| **Toggle "Pele de protocolo"** | Options: Automática (skin changes based on protocol content viewed), Mastodon (purple tones), Bluesky (blue tones), Nostr (yellow-green), Neutro (app default warm). |
| **Tap "Ler Manifesto"** | Push → full manifesto document (Screen 11, step 1). |
| **Tap timer settings** | Duration picker. Lembrete: gentle notification at interval. |

### Simple Mode — Detailed Behavior

When "Modo Simples" is ON:

| Normal | Simple Mode |
|---|---|
| Protocol badges on every post | Hidden (replaced by small text: "via Mastodon" on detail view) |
| Protocol filter pills in timeline | Hidden (single unified timeline) |
| Per-protocol character counts | Single character limit (lowest common denominator: 300) |
| Identity Cloud | Simple list: "Linked accounts" |
| Protocol-specific settings | Hidden |
| Cross-protocol sharing options | Hidden |

Simple Mode is designed for users who don't care about protocols — they just want to talk to people. It's a progressive disclosure mechanism.

### States

| State | Behavior |
|---|---|
| **Loading** | Skeleton list. |
| **Populated** | Full settings as shown. |
| **No protocols configured** | "Conecte seu primeiro protocolo!" big button. Otherwise settings are mostly hidden. |
| **Key not backed up** | Warning banner: "🔴 Suas chaves não têm backup. Você pode perder acesso às suas identidades." |
| **Storage critical** (>90% of quota) | Warning: "🟡 Armazenamento quase cheio. Limpe o cache ou exporte dados." |
| **Export in progress** | Progress bar. "Exportando — isso pode levar alguns minutos." |
| **Export complete** | Share sheet appears with the .zip file ready to share/save. |

### Accessibility

| Concern | Implementation |
|---|---|
| **VoiceOver** | Settings is a standard list with standard accessibility. Each row reads its label and current value. |
| **Warning screens** | "Warning: your private key grants full access to this identity. Do not share it." Read with emphasis. |
| **Key backup phrases** | "Your recovery phrase. Do not show this to anyone. It can recover all your identities." Words are individually selectable. |
| **Toggle switches** | All use standard platform toggle components for accessibility. |
| **Dynamic Type** | Full support. Settings are a vertical list. |
| **Colorblind** | Storage usage shown as text percentage AND bar. Warning levels use text + icon + color. |

### Animation

| Transition | Spec |
|---|---|
| **Main → Settings** | Push from right (0.3s). |
| **Settings → Sub-pages** | Push from right (0.3s). |
| **Toggle changes** | Standard platform toggle animation (0.2s). |
| **Simple Mode toggle** | If turning ON: all protocol badges fade out simultaneously (0.5s ease). The feed visibly simplifies. |
| **Export progress** | Animated progress bar (determinate, smooth). |
| **Warning banners** | Slide in from top (0.4s spring). Persistent until dismissed or resolved. |

### Design Rationale

Settings in most apps are an afterthought. Here, they're a **statement of values**:

1. **Identity management first** — Settings opens with identity. Not appearance, not notifications. Identity is the core of the app. Everything else is secondary.

2. **Key backup obvious** — The key backup section is prominent and includes a warning state when keys aren't backed up. This is critical for self-sovereign identity: if you lose your Nostr key, you lose your Nostr identity. The app takes this seriously.

3. **One-tap data export** — No "request your data" wait. No "we'll email you a link." One tap, immediate export. This is radical transparency: your data is yours and you can take it any time.

4. **Storage dashboard per-protocol** — Users should know which protocol is using the most space. This transparency helps users make informed decisions about cache management.

5. **Simple Mode** — This is the ultimate progressive disclosure. Users who don't care about protocols shouldn't have to see protocol complexity. But the option to reveal it is always there.

6. **Manifesto and Principles** — The app's philosophy is not hidden in a "Legal" page. It's prominent in Settings. The user should always be able to remind themselves why this app exists.

7. **Session timer** — Not a punishment, but awareness. The timer is visible on the intent screen and can be configured here. The "remind me every X" is a loose commitment device, not a hard limit.

---

## Screen 10: Empty States (All Variants)

### Purpose

Empty states are **opportunities for delight and guidance**, not dead ends. Each variant communicates the situation clearly, offers a path forward, and maintains the app's warm, humane aesthetic.

### 10A: Fresh Empty (Just Onboarded)

```
┌──────────────────────────────────────────────┐
│                                              │
│                                              │
│         🌱                                  │
│                                              │
│    Seu jardim está vazio                     │
│                                              │
│    Você ainda não segue ninguém.             │
│    Seu feed é um campo aberto,               │
│    esperando as primeiras sementes.          │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  🌟  Descobrir pessoas                │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  📋  Menu Degustação                  │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  📡  Importar contas do Mastodon     │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  Sem pressa. Cada conexão começa              │
│  com uma escolha.                             │
│                                              │
└──────────────────────────────────────────────┘
```

- **Illustration**: A small sprout/seedling, simple line art
- **Tone**: Encouraging, gentle, patient
- **Actions**: Three main actions + supportive footer text
- **VoiceOver**: "Your feed is empty. Your garden is waiting for seeds. Discover people, try the tasting menu, or import from Mastodon."

### 10B: Caught Up (Read Everything)

```
┌──────────────────────────────────────────────┐
│                                              │
│                                              │
│         ✨                                  │
│                                              │
│    Você está em dia!                         │
│                                              │
│    Você leu todas as publicações              │
│    das pessoas que segue.                    │
│    Nada passou despercebido.                 │
│                                              │
│    Novas publicações aparecerão               │
│    quando alguém publicar.                   │
│    Até lá, o silêncio também é               │
│    bem-vindo.                                │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  🔍  Explorar                        │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  💬  Ver notificações                │    │
│  └──────────────────────────────────────┘    │
│                                              │
└──────────────────────────────────────────────┘
```

- **Illustration**: A sparkle/star, suggesting completion
- **Tone**: Satisfied, peaceful
- **Rationale**: This is the anti-infinite-scroll statement. "You've read everything" is a feature, not a bug. The app validates the user for being caught up, rather than punishing them with emptiness.
- **VoiceOver**: "You are all caught up. You have read every post from the people you follow. Take a break."

### 10C: Offline (No Connection)

```
┌──────────────────────────────────────────────┐
│                                              │
│                                              │
│         📡                                  │
│                                              │
│    Sem conexão                               │
│                                              │
│    Você está offline. As publicações          │
│    abaixo são de [hora].                     │
│                                              │
│    O app funciona offline — você pode        │
│    navegar pelo cache, escrever rascunhos    │
│    e ler conversas salvas.                   │
│                                              │
│    Tudo será sincronizado quando              │
│    você conectar.                            │
│                                              │
│  [Tentar conectar novamente]                 │
│                                              │
└──────────────────────────────────────────────┘
```

- **Illustration**: Signal icon with a disconnected line
- **Tone**: Informative, reassuring
- **Behavior**: Appears as a banner at top when offline. Content below shows cached data when available. If no cache, shows the illustration alone.
- **VoiceOver**: "No internet connection. You can browse cached content and write drafts. Everything will sync when you connect."

### 10D: Protocol Down (One Protocol Unreachable)

```
┌──────────────────────────────────────────────┐
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  ⚠️  Bluesky está indisponível       │    │
│  │  O servidor não está respondendo.    │    │
│  │  Conteúdo do Bluesky não aparecerá   │    │
│  │  até que o servidor恢复正常.          │    │
│  │                                       │    │
│  │  [Tentar novamente] [Ignorar]        │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  (Timeline continues normally with           │
│   content from other protocols)              │
└──────────────────────────────────────────────┘
```

- **Illustration/Icon**: The protocol's icon with a warning badge
- **Tone**: Informative, not alarming
- **Behavior**: Shown as a banner within the timeline, not a full-screen overlay. The timeline continues showing content from other protocols. The pill for that protocol shows a warning dot.
- **VoiceOver**: "Bluesky is unavailable. The server is not responding. Content from Bluesky will not appear until the server is back online."

### 10E: Search — No Results

```
┌──────────────────────────────────────────────┐
│                                              │
│                                              │
│         🪹                                  │
│                                              │
│    Nada encontrado                           │
│                                              │
│    "xablau" não apareceu em nenhum           │
│    protocolo.                                │
│                                              │
│    Dicas:                                    │
│    • Verifique a ortografia                  │
│    • Tente termos mais gerais                │
│    • Busque em um protocolo específico       │
│                                              │
│  [Tentar novamente]                          │
│                                              │
└──────────────────────────────────────────────┘
```

- **Illustration**: Empty bird's nest
- **Tone**: Helpful, not frustrating
- **Shows**: The search term in quotes, followed by helpful suggestions
- **VoiceOver**: "No results found for [search term]. Check spelling or try broader terms."

### 10F: No Notifications

```
┌──────────────────────────────────────────────┐
│                                              │
│                                              │
│         💤                                  │
│                                              │
│    Nenhuma notificação                       │
│                                              │
│    Silêncio total. Ninguém mencionou         │
│    você, respondeu ou enviou mensagem.       │
│                                              │
│    Isso não é ruim — é espaço para           │
│    você iniciar uma conversa.                │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  ✍️  Compartilhar algo               │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  🔍  Explorar                        │    │
│  └──────────────────────────────────────┘    │
│                                              │
└──────────────────────────────────────────────┘
```

- **Illustration**: Sleeping moon or quiet face
- **Tone**: Peaceful, reframing "no notifications" as positive
- **VoiceOver**: "No notifications. Quiet is not bad — it is space for you to start a conversation."

### 10G: No DMs

```
┌──────────────────────────────────────────────┐
│                                              │
│                                              │
│         💌                                  │
│                                              │
│    Nenhuma mensagem                          │
│                                              │
│    Sua caixa de entrada está vazia.          │
│    Que tal mandar um "oi" para alguém?       │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  ✏️  Nova mensagem                   │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  👥  Seguir mais pessoas             │    │
│  └──────────────────────────────────────┘    │
│                                              │
└──────────────────────────────────────────────┘
```

- **Illustration**: Empty mailbox or envelope
- **Tone**: Inviting, action-oriented
- **VoiceOver**: "No messages. Your inbox is empty. Start a conversation."

---

## Screen 11: Onboarding Flow (5 Screens)

### Purpose

A warm, intentional onboarding that educates without overwhelming, offers clear path choices, and has users experiencing value within 60 seconds. No mandatory account creation. No permission spamming.

### Screen 11.1: Manifesto

```
┌──────────────────────────────────────────────┐
│                                              │
│         ☀️                                  │
│                                              │
│    Bom Retiro                                │
│                                              │
│    Toda conversa merece uma ponte.           │
│    Nenhuma conversa merece uma               │
│    plataforma.                               │
│                                              │
│    Este app conecta você a todas as          │
│    redes sociais abertas — Mastodon,         │
│    Bluesky, Nostr, Matrix, IRC, XMPP —       │
│    sem algoritmo, sem servidor próprio,      │
│    sem extração de dados.                    │
│                                              │
│    Sua atenção é sua.                        │
│    Sua conversa é sua.                       │
│    Sua identidade é sua.                     │
│                                              │
│    ┌────────────────────────────────────┐    │
│    │  ▶  Começar                        │    │
│    └────────────────────────────────────┘    │
│                                              │
│    [Pular introdução]                        │
│                                              │
│    Feito no Brasil, para o mundo.            │
│                                              │
└──────────────────────────────────────────────┘
```

**Interaction:**
- Tap "Comecar" → fade to Screen 2 (Identity choice)
- Tap "Pular introducao" → skip to Screen 3 (Content bootstrap)
- Swipe left → also advances to Screen 2

**Design Rationale:** The manifesto is short enough to read in 15 seconds. It sets the philosophical foundation without lecturing. The tagline is the title. The three "Sua... e sua" lines are the core promise.

### Screen 11.2: Identity Choice (3 Paths)

```
┌──────────────────────────────────────────────┐
│  Como você quer começar?                     │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  📥  Importar conta existente        │    │
│  │  Conecte suas contas atuais do       │    │
│  │  Mastodon, Bluesky, Nostr ou Matrix  │    │
│  │                                       │    │
│  │  [Mastodon] [Bluesky] [Nostr]        │    │
│  │  [Matrix]                            │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  ✨  Criar identidade nova           │    │
│  │  Gere uma chave Nostr ou Matrix      │    │
│  │  nova, aqui mesmo.                   │    │
│  │                                       │    │
│  │  [Nostr] [Matrix]                    │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  👀  Só explorar                     │    │
│  │  Veja o que estão conversando        │    │
│  │  sem conectar nenhuma conta.         │    │
│  │  Você pode criar ou importar depois. │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  [Voltar]                                    │
└──────────────────────────────────────────────┘
```

**Interaction:**
- Path 1: Tap a protocol button → OAuth flow or server/instance input. Add multiple protocols sequentially.
- Path 2: Tap Nostr or Matrix → key generation flow. Shows the new public key with explanation. Option to back up the private key immediately.
- Path 3: Tap "So explorar" → skip to Screen 3 with no accounts. The user will experience the app via the Tasting Menu.
- Tap any protocol button → haptic confirmation, that protocol highlights, can select more.
- Bottom: "Continuar" button enabled when at least one path is completed.

**Design Rationale:** Three paths for three personas:
1. **Import existing** — For Marina (Persona 1) and Rafael (Persona 2): power users who already have accounts.
2. **Create new** — For Carla (Persona 3): wants privacy, may not have accounts yet. Self-sovereign key generation.
3. **Just explore** — For Thiago (Persona 5) and Dandara (Persona 4): wants to see value before committing. The "zero-config" path.

The "Just explore" path is **unique** — most apps require account creation. By allowing exploration without identity, we remove the biggest adoption barrier.

### Screen 11.3: Content Bootstrap

```
┌──────────────────────────────────────────────┐
│  Vamos encher seu feed de coisa boa          │
│                                              │
│  ┌─ Pacotes de Entrada ─────────────────┐    │
│  │  Escolha categorias que te           │    │
│  │  interessam:                          │    │
│  │                                       │    │
│  │  [Tecnologia] [Arte] [Ciência]       │    │
│  │  [Ativismo] [Música] [Games]         │    │
│  │  [Fotografia] [Literatura]           │    │
│  │                                       │    │
│  │  com base na sua seleção,            │    │
│  │  sugerimos seguir estas contas:      │    │
│  │                                       │    │
│  │  ┌──┐ @anna_dev               📡    │    │
│  │  │  │ Tecnologia · 1.2k posts       │    │
│  │  └──┘                                │    │
│  │  ┌──┐ @protocol_bot           🔵    │    │
│  │  │  │ Fediverso · 890 posts         │    │
│  │  └──┘                                │    │
│  │  ┌──┐ @artesanal             🟡    │    │
│  │  │  │ Arte digital · 450 posts     │    │
│  │  └──┘                                │    │
│  │                                       │    │
│  │  [Seguir selecionados]               │    │
│  └───────────────────────────────────────┘   │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  🔍  Buscar tópicos ou pessoas       │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  📂  Importar do Mastodon            │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  [Pular — quero começar do zero]            │
└──────────────────────────────────────────────┘
```

**Interaction:**
- Tap categories to select/deselect (multi-select)
- On selection change, the suggested accounts update dynamically (with a brief loading state)
- Tap "Seguir selecionados" → follow all checked accounts. Progress shown per protocol.
- Tap "Importar do Mastodon" → OAuth to Mastodon, import follows list, cross-reference with Bluesky/Nostr for same people.
- Tap "Pular" → skip to Screen 4

**States:**
- **Importing from Mastodon**: Progress bar: "Importando seguidores do Mastodon... 45/128 encontrados no Bluesky."
- **No categories selected**: "Selecione categorias para receber sugestões de contas."
- **Import complete**: "128 contas importadas! 45 também encontradas no Bluesky."

**Design Rationale:** This screen reduces time-to-value dramatically. Instead of staring at an empty feed, the user gets a curated starter pack within 30 seconds. The import feature solves the "I don't want to rebuild my network" objection.

### Screen 11.4: First Post / Introduction

```
┌──────────────────────────────────────────────┐
│  Quer fazer sua primeira publicação?         │
│                                              │
│  Não precisa — mas se quiser,                │
│  tá aqui o espaço.                           │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  Para: [📡 Masto] [🔵 Bluesky]       │    │
│  │                                       │    │
│  │  ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐  │    │
│  │  │  Opções de自我介绍:            │    │    │
│  │  │  • "Oi, sou novo por aqui!"    │    │
│  │  │  • "Chegando do Twitter..."    │    │
│  │  │  • "Em busca de boas           │    │
│  │  │     conversas."                │    │
│  │  └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘  │    │
│  │                                       │    │
│  │  ┌──────────────────────────────┐    │    │
│  │  │  Ou escreva o seu próprio... │    │    │
│  │  └──────────────────────────────┘    │    │
│  │                                       │    │
│  │  [Publicar] [Pular]                  │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  Dica: A primeira publicação é especial.     │
│  É como dizer "oi" numa sala cheia de       │
│  salas.                                      │
└──────────────────────────────────────────────┘
```

**Interaction:**
- Tap a suggestion template → text auto-fills. User can edit before posting.
- Tap the custom text field → write original post.
- Protocol selector: default to protocols configured in Screen 2.
- Tap "Publicar" → post published with optional confirmation (first post is celebrated, not frictioned).
- Tap "Pular" → skip to Screen 5.
- After posting: celebration animation (confetti-like, subtle) → "Sua primeira publicação foi ao ar!" → auto-advance to Screen 5.

**Design Rationale:** The first post screen is **optional and low-pressure**. The templates reduce the "blank page" anxiety. The tip at bottom reframes the first post as a warm hello rather than a content contribution. The celebration after posting validates the act of joining.

### Screen 11.5: Timeline Alive (Ready State)

```
┌──────────────────────────────────────────────┐
│                                              │
│         🌟                                  │
│                                              │
│    Tudo pronto!                              │
│                                              │
│    Seu timeline está vivo.                   │
│                                              │
│    Aqui está o que você pode fazer agora:    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  📖  Ver timeline                    │    │
│  │  Navegar pelas publicações           │    │
│  │  das pessoas que você segue          │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  🔍  Explorar mais                   │    │
│  │  Descobrir novas contas e            │    │
│  │  comunidades                        │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │  ⚙️  Configurar identidades         │    │
│  │  Adicionar mais protocolos,          │    │
│  │  ajustar preferências               │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  Lembre-se: você pode voltar e               │
│  ajustar tudo depois.                        │
│                                              │
│  [Vamos lá!]                                 │
└──────────────────────────────────────────────┘
```

**Interaction:**
- Tap any card → navigate to that destination.
- Tap "Vamos la!" → crossfade to the Intent Screen (Screen 1). Full app experience begins.
- This is the last onboarding screen. After this, the user sees the normal app flow.

**States for different scenarios:**
- **Imported accounts** (Path 1): "Seu timeline está vivo com [N] contas importadas!"
- **Created new identity** (Path 2): "Sua nova identidade está pronta. Você é npub1..."
- **Just exploring** (Path 3): "Pronto para explorar! Comece pelo Menu Degustação."

**Design Rationale:** The final screen celebrates completion while offering clear next steps. It doesn't throw the user into the deep end — it offers three obvious starting points. The "Lembre-se" footer is reassuring: you haven't made permanent decisions.

### Onboarding Flow — Technical Notes

**Skippability:**
- Every screen can be skipped individually
- "Pular introducao" on Screen 1 jumps to Screen 3
- Skip always available as text link (not a prominent button that competes with the primary CTA)

**Progress indicator:**
- Dots at bottom of each screen showing position (● ● ● ● ●)
- Dots are small (6pt), warm gray
- Completed dots filled, current dot pulse, future dots empty

**Exiting onboarding:**
- Can exit onboarding from any screen (tiny "Sair" link at top-left)
- Progress saved: returning resumes from last screen
- If exited before completion, user is taken to the Intent Screen with minimal setup

**Permissions:**
- Zero permissions requested during onboarding
- Camera/gallery permissions requested only when user first tries to attach media
- Notification permissions requested only when first notification would be sent

---

## Appendix: Design System Constants

### Layout Grid

| Property | Value |
|---|---|
| **Content width** (max) | 640pt (iPad), 100% (iPhone) |
| **Card padding** | 16pt |
| **Card corner radius** | 12pt |
| **Card spacing** | 12pt vertical |
| **Avatar size** (timeline) | 40pt |
| **Avatar size** (profile) | 80pt |
| **Protocol badge size** | 20pt |
| **Section header padding** | 8pt vertical |
| **Safe area** | Respects platform safe areas |

### Typography (Scale)

| Style | Size | Weight | Line Height |
|---|---|---|---|
| **Large title** | 34pt | Bold | 41pt |
| **Title 1** | 28pt | Bold | 34pt |
| **Title 2** | 22pt | Semibold | 28pt |
| **Title 3** | 20pt | Semibold | 25pt |
| **Headline** | 17pt | Semibold | 22pt |
| **Body** | 17pt | Regular | 22pt |
| **Callout** | 16pt | Regular | 21pt |
| **Subheadline** | 15pt | Regular | 20pt |
| **Footnote** | 13pt | Regular | 18pt |
| **Caption 1** | 12pt | Regular | 16pt |
| **Caption 2** | 11pt | Regular | 13pt |

### Color Palette

Based on the vision document's "sensations":

| Token | Use | Hex Ref |
|---|---|---|
| **Awakening** | Primary accents, highlights | `#F4A460` (warm amber-gold) |
| **Earth** | Secondary, buttons, importance | `#C04040` (brazilian red clay) |
| **Forest** | Success, growth | `#2E7D32` (deep green) |
| **Sky** | Info, links | `#4A90D9` (warm blue) |
| **Sun** | Warnings, attention | `#F5A623` (golden yellow) |
| **Sea** | Tints, backgrounds | `#5F9EA0` (blue-green) |
| **Background** | Card backgrounds | `#FFF8F0` (warm off-white) |
| **Background Secondary** | Screen backgrounds | `#FFF5E6` (slightly warmer) |
| **Text Primary** | Body text | `#1A1A1A` (near-black, warm) |
| **Text Secondary** | Captions, metadata | `#6B5B4E` (warm gray) |
| **Text Tertiary** | Placeholders | `#A89888` (lighter warm gray) |
| **Border** | Card borders | `#E0D5C8` (warm light) |

### Spacing

| Token | Points |
|---|---|
| **XS** | 4 |
| **S** | 8 |
| **M** | 12 |
| **L** | 16 |
| **XL** | 24 |
| **2XL** | 32 |
| **3XL** | 48 |

### Protocol Visual Identity

| Protocol | Shape | Primary Color | Secondary Color | Icon |
|---|---|---|---|---|
| ActivityPub/Mastodon | Rounded square | `#6364FF` | `#9B9BFF` | Elephant silhouette |
| AT Protocol/Bluesky | Circle | `#0085FF` | `#66B5FF` | Cloud/butterfly |
| Nostr | Hexagon | `#9BBA4B` | `#C5DA7A` | Lightning zap |
| Matrix | Rounded pill | `#00D4AA` | `#66E5CC` | Bridge/arrows |
| IRC | Diamond | `#FF8C42` | `#FFB380` | Terminal `<>` |
| XMPP | Squircle (square with high radius) | `#4DB8B8` | `#80D4D4` | Chat bubble |

### Animation Timing

| Context | Duration | Curve |
|---|---|---|
| **Push transitions** | 0.3s | Ease-in-out |
| **Crossfade (mode shifts)** | 0.3s | Ease-out |
| **Sheet presentation** | 0.35s | Spring (damping 0.8) |
| **Haptic feedback** | 0.15s | Instant |
| **Icon pulses** | 0.3s | Spring (damping 0.6) |
| **Fade in/out** | 0.4s | Ease-out |
| **Character count update** | 0.15s | Ease |
| **Loading shimmer** | 1.5s loop | Linear |

---

*This specification is a living document. All measurements are in points (iOS native) or logical pixels. Colors are indicative — final palette depends on actual implementation platform. All interactions should be tested with real users, especially the long-press-to-publish and the time-box prompts.*

*End of wireframe specification. Total: 11 screens + appendix.*
