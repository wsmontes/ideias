# Loop 02: Halide — How Lux Optics Masters iOS Hardware

> A deep research document on how Halide pushes iPhone camera hardware to its limits through native APIs, gesture-first design, and a philosophy that treats the phone as a "real camera" — and what Feedmine can learn about building an app that exploits every hardware capability.

---

## 1. Executive Summary

Halide (by Lux Optics, now at Mark III) is the gold standard for how a third-party iOS app can use Apple hardware more creatively than Apple's own app. It won the 2022 Apple Design Award for "Delight and Fun," runs natively on iPhone and iPad, and has become the canonical example of an app that doesn't just use iOS — it pushes iOS hardware APIs further than anyone else.

**What makes it the reference standard:**

- **Deepest camera hardware API usage in the App Store.** AVCaptureSession, AVCaptureDevice manual controls, depth APIs, LiDAR scanner, ProRAW/DNG capture, 48MP sensor access, 14-bit RAW streaming — Halide uses everything Apple exposes and sometimes discovers undocumented capabilities.
- **Gesture-first interface.** Swipe to adjust exposure. Swipe to adjust focus. Tap to toggle auto/manual. The entire app can be operated without hunting through menus — the viewfinder IS the control surface.
- **Process Zero — an anti-computational-photography philosophy.** Instead of adding more ML processing, Halide built a mode that strips away all iPhone image processing, producing "digital negatives" at the sensor level. This is a product differentiator built entirely on understanding what Apple's camera pipeline DOES and choosing to bypass it.
- **Film simulation engine with Looks.** Mark III (2026) introduces a physically accurate film simulation system, co-developed with a Hollywood colorist, that processes at the RAW level — not a filter on a JPEG, but a custom development pipeline.
- **Hardware button integration.** Supports iPhone Action Button (15 Pro+) and Camera Control button (16+) as first-class triggers.

**The lesson for Feedmine:** Halide proves that understanding the hardware layer below the OS gives you capabilities no competitor can match. For a feed reader, the equivalent is understanding the scroll engine, the display refresh rate, the ProMotion system, and the haptic engine at a level deeper than UIKit defaults.

---

## 2. Platform Integration Inventory

### 2.1 Camera Hardware APIs (AVFoundation)

Halide's technical foundation is the AVFoundation framework — specifically:

| API | What Halide Does With It |
|-----|--------------------------|
| `AVCaptureDevice` manual controls | Full control over ISO, shutter speed, white balance, focus distance |
| `AVCapturePhotoOutput` | RAW (DNG), ProRAW, HEIF, JPEG capture |
| Depth data (`AVDepthData`) | Depth map capture, portrait matte generation |
| LiDAR scanner (ARKit depth) | Point cloud depth for non-portrait subjects |
| 48MP sensor access | Full-resolution Bayer RAW on iPhone 14 Pro+ |
| Focus peaking (custom) | Real-time edge detection overlay on live preview |
| Histogram (custom) | Live histogram rendered via Metal/CoreImage |
| Waveform monitor | Professional-grade exposure analysis in real-time |
| Exposure metering (separate API) | Uses device's metering independent of live feed |
| ProMotion 120Hz | Smooth viewfinder at max display refresh |

**Key insight from Mark III:** The live video feed on iPhone does NOT accurately represent the final captured photo — computational photography is baked into the preview. Halide discovered this and built a separate exposure meter using Apple's metering API, mimicking an analog camera's light meter. This is the kind of platform knowledge that comes from years of deep API exploration.

### 2.2 Process Zero — Bypassing the Image Pipeline

Process Zero is Halide's signature feature: a mode that captures photos at the raw sensor level, skipping Apple's entire computational photography stack:

- No Smart HDR
- No Deep Fusion
- No noise reduction
- No tone mapping
- No AI subject detection and selective brightening
- No sharpening

**What it produces:** A minimal, film-like photo that preserves natural lighting, skin tones, and dynamic range as the sensor sees it. It operates at the DNG/RAW level and processes photos with a custom pipeline that Lux Optics built from scratch.

**Technical implementation:** Process Zero uses `AVCapturePhotoOutput` with settings that disable all Apple processing, then applies Halide's own minimal development pipeline (custom tone curve, basic highlight recovery, color science calibration per-device-model).

**Feedmine analogy:** The equivalent for a feed reader would be a "Process Zero" for content: strip away all algorithmic ranking and show the raw chronological feed with zero ML intervention — as a user-controllable mode alongside the smart feed.

### 2.3 Film Simulation Engine (Mark III)

Halide Mark III introduces "Looks" — physically accurate film simulations:

| Look | Intent | Character |
|------|--------|-----------|
| Valencia | Landscapes, cityscapes | Thick contrast, deep saturation, color separation |
| Rembrandt | Portraits | Mid-tone contrast, warm low-end, convergent highlights |
| Nova | Colorful landscapes | Tight contrast, smooth peachy highlights |
| Zephyr | General/neutral | Subtle filmic contrast, rich skin tones |
| Chroma Noir | Black & white | Medium contrast, extra grain |

Each Look includes:
- Custom tone curve
- Film grain simulation (togglable)
- Halation simulation (togglable) 
- MTF (Modulation Transfer Function / micro-contrast)
- Vignetting
- Full HDR support (separate SDR and HDR versions developed for each)

**Developed with Hollywood colorist Cullen Kelly** — this isn't a hobbyist filter pack. It's professional color science applied at the RAW processing level.

### 2.4 iPhone Hardware Buttons

**Action Button (iPhone 15 Pro+):**
- Halide registers as an Action Button destination
- User configures: press Action Button → Halide opens and captures immediately (or opens to viewfinder)
- Multiple trigger modes: open app, open and capture, open to specific lens

**Camera Control Button (iPhone 16+):**
- Halide integrates with the new capacitive Camera Control
- Half-press for focus lock, full-press for capture
- Swipe on the button to adjust exposure or zoom

**Lock Screen camera shortcut (iOS 18+):**
- Users can replace Apple's Camera app on the Lock Screen shortcut with Halide
- Press and hold the camera shortcut → Halide launches directly

### 2.5 iPad Support

Halide for iPad (shipped 2021) is a native iPad app, not just a scaled-up iPhone version:
- Optimized for iPad Pro's camera system
- Full interface redesign for larger display (more controls visible simultaneously)
- Photo Lab editor takes advantage of iPad's screen real estate
- Mark III continues iPad-first design for editing

### 2.6 Apple Watch Integration

Halide includes an Apple Watch companion app:
- **Remote viewfinder** — see what the camera sees on your wrist
- **Remote shutter** — trigger capture from the watch
- **Timer controls** — set self-timer from the watch
- **Siri shortcut** — "Hey Siri, take a photo" triggers Halide's shutter

Use case: prop iPhone up for a group photo, frame on the watch, trigger from wrist.

### 2.7 Widgets

Halide supports Lock Screen and Home Screen widgets:
- Quick-access widget to launch directly into specific shooting mode
- Lock Screen widget for instant camera access

### 2.8 Siri Shortcuts & Share Extension

- Siri Shortcuts for hands-free capture ("Take a photo with Halide")
- Shortcut actions for specific modes (RAW, Process Zero, specific lens)
- Share Extension for receiving images to edit in Halide's Photo Lab

### 2.9 Metal & CoreImage Pipeline

The real-time viewfinder overlays (focus peaking, histogram, waveform, zebras, level grid) are rendered using:
- **Metal** for GPU-accelerated rendering at 120fps on ProMotion displays
- **CoreImage** for real-time filter chains on the camera preview
- Custom shaders for focus peaking edge detection

This ensures the viewfinder remains smooth even with multiple professional overlays active simultaneously.

### 2.10 Liquid Glass Adoption (iOS 26)

Mark III's design refresh "went hand-in-hand with Apple's release of their Liquid Glass design system":
- Dynamic toolbar items that appear/disappear based on context
- "..." menu paradigm for decluttering rarely-used controls
- Organic animations for UI state transitions
- Translucent glass effects on control surfaces over the camera preview

---

## 3. Interaction Design Patterns

### 3.1 Gesture-First Camera Control

The fundamental design philosophy: **the viewfinder is the control surface.**

| Gesture | Action | Why It Works |
|---------|--------|--------------|
| Swipe up/down on right edge | Adjust exposure compensation | Mimics an analog dial — muscle memory |
| Swipe up/down on left edge | Adjust manual focus (when active) | Same pattern, different axis |
| Tap AF/MF button | Toggle auto/manual focus | Single tap, mode switch |
| Tap anywhere on viewfinder | Focus + expose on that point | Mirrors Apple Camera behavior (familiar) |
| Pinch | Zoom (lens switch or digital) | System-standard gesture |

**Design principle:** "A camera is an extension of your body. It works best when it creates muscle memory. We need consistent gestures. We need to be flexible without changing buttons around all the time."

### 3.2 Single Yellow Highlight Color

Halide uses exactly ONE accent color (yellow — a homage to classic Nikon cameras) to indicate active state. Every control is either:
- Gray/white = inactive/available
- Yellow = currently active/enabled

This eliminates confusion about what's on vs. off. The team learned this lesson when a developer went on a trip and accidentally shot all photos NOT in RAW because the deactivated state was ambiguous.

### 3.3 Progressive Complexity

The UI layers complexity:
1. **Auto mode** — tap anywhere, snap photos. Feels like Apple Camera.
2. **Semi-manual** — tap AF to go manual focus, use focus peaking
3. **Full manual** — toggle manual exposure, control ISO + shutter separately
4. **Pro tools** — enable histogram, waveform, zebras, level grid

Each layer is accessible from the previous one with a single tap. No settings screens. No modal dialogs. Everything happens on the viewfinder.

### 3.4 The Photo Lab (Built-In Editor)

Mark III includes a streamlined RAW editor:
- **Quick Edit:** Audition different Looks, toggle HDR/film simulation, adjust exposure
- **Frame:** Crop, level, output resolution (defaults to 4K even for 48MP captures)
- **Exposure:** Histogram, tone fusion tool
- **Film:** Toggle individual simulation components (grain, halation, MTF, vignetting)
- **Balance:** White balance presets and custom

Design principle: "We didn't build a full-fledged editor. We wanted something better." — streamlined and approachable, not an airplane cockpit.

---

## 4. Development Philosophy

### 4.1 "We Didn't Make an App — We Made a Camera"

This philosophical framing drives every decision:
- Controls mimic physical camera ergonomics, not software UI patterns
- The viewfinder is always dominant (never obscured by chrome)
- Haptic and visual feedback mimics physical button/dial sensation
- Features are named after camera concepts (Looks, not "filters")

### 4.2 Excitement Without Intimidation

From Sebastiaan de With: "Give a child a camera and they'll play with the aperture ring and the dials and the switches. I thought, 'Maybe we can bring a semblance of that delight to an app on a piece of glass.'"

The team includes both professional photographers AND someone (Rebecca Slatkin) who came from a casual photography background — ensuring accessibility for all skill levels. "You know in bowling, where they put bumpers in the gutters to make sure you at least hit a pin? We support people like that."

### 4.3 Anti-AI Positioning

While the industry races toward AI photography, Halide goes the opposite direction:
- Process Zero explicitly removes ALL computational photography
- Film simulations are physics-based, not ML-generated
- The marketing is "zero-AI, minimally processed shots"
- This creates a clear product identity and attracts users tired of over-processed iPhone photos

### 4.4 Small Team, Focused Scope

Lux Optics is ~3 core people (Sebastiaan de With, Ben Sandofsky, Rebecca Slatkin) plus collaborators. They maintain:
- Halide (camera)
- Spectre (long exposure — App of the Year 2019)
- Kino (video)
- Orion (iPad as HDMI monitor)

Each app is laser-focused on one thing done exceptionally well.

### 4.5 Community-Driven Development

Mark III was developed over 18 months with exhaustive communication on Discord. The user community tested ideas, provided feedback, and validated the new design before launch. This is "build in public" for a premium native app.

---

## 5. What Feedmine Can Steal

### 5.1 Gesture-as-Primary-Interface

Halide proves that gestures can replace menus entirely for a core interaction loop. The swipe-to-adjust pattern creates muscle memory — users develop proprioceptive knowledge of where their thumb needs to go.

**For Feedmine:**
- Swipe right on an article → save/bookmark (one gesture, no menu)
- Swipe left → "not interested" / skip
- Long press → preview without fully opening
- Pull down from top → refresh
- Edge swipe on left → go back (system-standard)
- Vertical swipe on right edge → adjust font size (like Halide's exposure dial)

### 5.2 Single Accent Color as State Indicator

Halide's "yellow = active" system eliminates ambiguity. Every UI element's state is immediately clear.

**For Feedmine:** Pick one accent color. When a source filter is active, it's that color. When an article is bookmarked, the indicator is that color. When a sort mode is engaged, it's that color. Everything else is system gray/neutral.

### 5.3 The "Process Zero" Mindset for Feed Ranking

Halide's philosophy of offering a "zero-processing" mode alongside intelligent processing maps perfectly to Feedmine's "open algorithms" concept.

**For Feedmine:** Always offer a "Process Zero" mode — pure chronological feed with zero ranking, zero intelligence, zero personalization. Let the user see what the raw feed looks like, then appreciate what the algorithm does for them. This builds trust AND serves users who sometimes just want unfiltered content.

### 5.4 Metal/GPU Awareness for Smooth Scrolling

Halide uses Metal for 120fps viewfinder overlays. The equivalent for a feed reader is ensuring the scroll engine uses the GPU effectively.

**For Feedmine:**
- Pre-render article thumbnails on the GPU
- Use `CADisplayLink` for any custom animations (match ProMotion)
- Ensure scroll performance never drops below 120fps on capable devices
- Use Core Animation's Metal-backed layers for smooth list transitions

### 5.5 Hardware Button Integration

Halide uses Action Button and Camera Control as first-class inputs. Feed readers don't have the same urgency, but the principle applies.

**For Feedmine:**
- Action Button → open Feedmine to "Quick Read" mode (show top 3 articles immediately)
- Lock Screen shortcut → launch directly into today's briefing
- Spotlight suggestions → "Read your morning feed" appears at habitual reading times

### 5.6 Community as Development Partner

Halide's Discord community shaped Mark III over 18 months. For an open-source project like Feedmine, this is even more natural.

**For Feedmine:** Build on GitHub with transparent issues/discussions. Let power users beta test ranking algorithm changes. Make the development process visible.

### 5.7 Professional Aesthetic Without Complexity

Halide looks professional and feels serious without overwhelming the user. The interface is minimal yet powerful — complexity is layered, not imposed.

**For Feedmine:** The app should look like a professional tool (not a toy, not a social media feed). Clean typography. Generous whitespace. Information density that FEELS intentional — like a well-designed newspaper, not a cluttered news aggregator.

---

## 6. Anti-Patterns Halide Avoids

1. **No feature creep.** Halide is a camera. It doesn't do video (that's Kino). It doesn't do long exposure (that's Spectre). One app, one job.
2. **No "filter" mentality.** Looks are physics-based film simulations, not Instagram presets. The distinction matters for quality and user respect.
3. **No settings labyrinths.** Everything is on the viewfinder or one tap away. No "Settings > Camera > Advanced > Manual Controls > Focus Mode" paths.
4. **No AI branding.** Despite using ML in some features (Spectre, Instant RAW), the marketing is explicitly anti-AI for Process Zero.
5. **No social features.** Halide is about making photos, not sharing them.
6. **No cloud lock-in.** Photos go to your camera roll. No Halide account. No cloud gallery.

---

## 7. Technical Deep Dives

### 7.1 Focus Peaking Implementation

Focus peaking highlights in-focus edges in the live preview. Halide's implementation:
- Real-time edge detection (likely Sobel or Laplacian filter) on the camera preview frames
- Rendered as an overlay using Metal shaders
- Color-coded (user-selectable: red, yellow, blue) to distinguish from scene content
- Runs at full viewfinder framerate (up to 120fps)
- Must handle varying lighting conditions without false positives

### 7.2 Depth Capture System

Halide was one of the first third-party apps to exploit iPhone's depth APIs:
- **Dual-camera depth (iPhone 7+):** Stereo disparity from two lenses
- **TrueDepth (front camera):** Structured light depth for selfie portraits
- **LiDAR (iPhone 12 Pro+):** Time-of-flight depth scanning
- **Portrait Matte:** Apple's ML-generated foreground segmentation

Users can view and export depth maps as separate images — useful for 3D photography, VFX, and AR applications.

### 7.3 ProRAW vs. Bayer RAW

Halide gives users granular control over RAW format:
- **Bayer RAW (DNG):** Pure sensor data, no Apple processing. Smaller files, maximum editing latitude.
- **ProRAW (12-bit or 10-bit):** Apple's computational photography baked in but with RAW editability. Larger files, Apple's HDR/detail retained.
- **Process Zero:** Bayer RAW with Halide's minimal custom development.

Users can choose file size vs. quality tradeoff: 12-bit ProRAW (largest), 10-bit ProRAW, or Bayer RAW (smallest).

### 7.4 HDR Photo Processing

Mark III Looks include full HDR support:
- Each Look has separate SDR and HDR versions (doubled development effort)
- HDR captures preserve highlight detail beyond SDR range
- Photos display in EDR (Extended Dynamic Range) on capable displays
- The Photo Lab shows HDR content correctly during editing

---

## 8. Key Takeaways for Feedmine

1. **Push hardware APIs beyond their obvious use.** Halide discovered that the live preview doesn't match final capture and built a separate metering system. For Feedmine: explore what ProMotion's variable refresh rate can do for reading (lower refresh for static text = battery savings, 120Hz for scrolling).

2. **Make the interaction surface the content surface.** Halide's viewfinder IS the controls. For Feedmine: the article list IS the interaction surface — gestures on the list items directly manipulate them, no secondary UI needed.

3. **Offer "raw" mode alongside smart mode.** Process Zero proves users value transparency. Feedmine should always let users see the unranked chronological feed.

4. **Professional aesthetics earn trust.** Halide's camera-inspired design (etched-metal fonts, yellow accent, dark interface) makes users feel they're using a serious tool. Feedmine should feel like a professional reading tool, not a consumer social app.

5. **One app, one job, done perfectly.** Halide doesn't try to be a photo editor, social network, or cloud gallery. Feedmine is a feed reader. Not a podcast app. Not a note-taking app. Not a social network.

6. **Hardware awareness = competitive moat.** The apps that understand the hardware layer (GPU, display, haptic engine, sensors) deliver experiences that cross-platform frameworks cannot replicate.

---

*Document generated: 2026-07-06. Research rounds: Broad (app philosophy, Apple recognition, feature landscape), Focused (Mark III features, Process Zero, AVFoundation APIs), Gaps (hardware buttons, Metal rendering, depth APIs, accessibility). Sources: Apple Developer News, lux.camera blog, PetaPixel, DPReview, MacStories, Halide support docs.*
