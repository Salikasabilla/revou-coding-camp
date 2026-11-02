# Design Document

## Overview

"Kiro — Cute Cat Petting Clicker" is a single self-contained `index.html` file. All HTML markup, CSS, and JavaScript are inline. There are no linked stylesheets, no linked scripts, no external image assets, and no audio files. The app uses Tailwind CSS (CDN Play script), Vanilla JavaScript, and the Web Audio API.

A user loads the file in any modern browser, sees an animated kawaii cat on a pastel glassmorphism background, clicks or taps the cat, and receives immediate animated feedback. Each interaction increments a persistent session counter, triggers one of eight distinct cat reactions with particles and a message, and optionally plays a synthesized sound. Four milestone thresholds produce one-time celebrations.

---

## Architecture

### Single-File Structure

The file is organized in this exact order:

```
<html>
  <head>
    <!-- 1. Meta tags (charset, viewport) -->
    <!-- 2. Google Font preconnect + stylesheet links -->
    <!-- 3. Tailwind CDN Play <script> tag -->
    <!-- 4. <style> block: all custom CSS + @keyframes -->
  </head>
  <body>
    <!-- 5. Semantic HTML structure -->
    <script>
      <!-- 6. All JavaScript: constants, state, modules, init -->
    </script>
  </body>
</html>
```

### JavaScript Module Organization

All JavaScript lives in a single `<script>` tag divided into clearly commented sections. Each section is a logical module implemented as a group of named, single-responsibility functions:

```
// === CONSTANTS ===
// === STATE MODULE ===
// === REACTION REGISTRY ===
// === REACTION SYSTEM ===
// === PARTICLE SYSTEM ===
// === SOUND ENGINE ===
// === MILESTONE SYSTEM ===
// === DOM UPDATER ===
// === EVENT HANDLERS ===
// === ANIMATION ORCHESTRATION ===
// === ACCESSIBILITY ===
// === INIT ===
```

### Data Flow

```
User gesture (click / tap / Enter / Space)
        │
        ▼
handleCatInteraction()
        │
        ├─► incrementPetCount()       ← STATE MODULE
        │         │
        │         └─► updateCounterDisplay()   ← DOM UPDATER
        │         └─► checkMilestones()        ← MILESTONE SYSTEM
        │
        ├─► cancelActiveReaction()    ← REACTION SYSTEM
        │
        ├─► applySoundEffect()        ← SOUND ENGINE
        │
        └─► startReaction(type)       ← REACTION SYSTEM
                  │
                  ├─► applyExpression(reaction)
                  ├─► applyReactionClass(reaction)
                  ├─► spawnParticles(reaction)   ← PARTICLE SYSTEM
                  ├─► showReactionMessage(msg)   ← DOM UPDATER
                  └─► (animationend) → cleanupReaction() → restoreIdle()
```

### State Ownership

`Session_State` is the single source of truth. DOM elements are updated only via `DOM UPDATER` functions that read from `Session_State`. No DOM element is read back to infer application state.

---

## Components and Interfaces

### HTML Structure

```html
<body>
  <!-- Background decorations (purely visual, aria-hidden) -->
  <div id="bg-decorations" aria-hidden="true">
    <!-- Stars, clouds, sparkles, glowing circles -->
  </div>

  <header>
    <h1>Kiro's Cozy Corner</h1>
    <p>Give your little friend some love!</p>
  </header>

  <main>
    <!-- Central card panel (glassmorphism) -->
    <section id="cat-panel" class="bg-white/30 backdrop-blur-lg border border-white/40 rounded-2xl ...">

      <!-- Pet counter display -->
      <div id="counter-display" aria-live="polite" aria-atomic="true">
        🐾 Total Pets: <span id="pet-count">0</span>
      </div>

      <!-- Cat + cushion wrapper -->
      <div id="cat-wrapper" class="aspect-square flex items-center justify-center relative">

        <!-- Cushion (decorative, aria-hidden) -->
        <div id="cat-cushion" aria-hidden="true"></div>

        <!-- Interactive cat element -->
        <div
          id="cat"
          role="button"
          tabindex="0"
          aria-label="Pet the cat"
          class="aspect-square flex items-center justify-center"
        >
          <!-- Cat body -->
          <div id="cat-body">
            <!-- Ears -->
            <div class="cat-ear cat-ear--left">
              <div class="cat-ear-inner"></div>
            </div>
            <div class="cat-ear cat-ear--right">
              <div class="cat-ear-inner"></div>
            </div>

            <!-- Face -->
            <div id="cat-face">
              <!-- Eyes -->
              <div class="cat-eye cat-eye--left">
                <div class="cat-pupil"></div>
                <div class="cat-eye-blink"></div>  <!-- hidden by default -->
                <div class="cat-eye-wide"></div>   <!-- hidden by default -->
              </div>
              <div class="cat-eye cat-eye--right">
                <div class="cat-pupil"></div>
                <div class="cat-eye-blink"></div>
                <div class="cat-eye-wide"></div>
              </div>

              <!-- Cheek overlays (blushing, hidden by default) -->
              <div class="cat-cheek cat-cheek--left" aria-hidden="true"></div>
              <div class="cat-cheek cat-cheek--right" aria-hidden="true"></div>

              <!-- Nose -->
              <div id="cat-nose"></div>

              <!-- Mouth (smile default; frown via class) -->
              <div id="cat-mouth">
                <div class="cat-mouth-smile"></div>
                <div class="cat-mouth-frown" style="display:none"></div>
                <div class="cat-mouth-open" style="display:none"></div>
              </div>

              <!-- Whiskers -->
              <div class="cat-whiskers cat-whiskers--left">
                <div class="cat-whisker"></div>
                <div class="cat-whisker"></div>
                <div class="cat-whisker"></div>
              </div>
              <div class="cat-whiskers cat-whiskers--right">
                <div class="cat-whisker"></div>
                <div class="cat-whisker"></div>
                <div class="cat-whisker"></div>
              </div>
            </div><!-- /#cat-face -->
          </div><!-- /#cat-body -->
        </div><!-- /#cat -->
      </div><!-- /#cat-wrapper -->

      <!-- Reaction message -->
      <div
        id="reaction-message"
        aria-live="polite"
        aria-atomic="true"
        role="status"
      ></div>

      <!-- Static instructional label (always visible) -->
      <p id="instruction-label">Tap the kitty to make it happy ♡</p>

    </section><!-- /#cat-panel -->

    <!-- Controls panel (glassmorphism) -->
    <section id="controls-panel" class="bg-white/30 backdrop-blur-lg border border-white/40 rounded-2xl ...">
      <button
        id="sound-toggle"
        aria-label="Toggle sound"
        aria-pressed="false"
      >🔇</button>

      <button
        id="reset-button"
        aria-label="Reset pet counter"
      >Reset</button>

      <!-- Reset confirmation prompt (hidden by default) -->
      <div
        id="reset-confirm"
        role="dialog"
        aria-label="Confirm reset"
        aria-modal="true"
        style="display:none"
      >
        <span>Are you sure?</span>
        <button id="reset-confirm-btn" aria-label="Confirm reset">Yes</button>
        <button id="reset-cancel-btn" aria-label="Cancel reset">No</button>
      </div>
    </section>

    <!-- Particle container (positioned absolutely over cat-wrapper) -->
    <div id="particle-container" aria-hidden="true"></div>

    <!-- Milestone overlay (injected/removed dynamically, hidden by default) -->
    <!-- #milestone-overlay is created dynamically by the Milestone System -->

  </main>
</body>
```

### Key Element IDs

| ID | Purpose |
|---|---|
| `bg-decorations` | Background floating shapes |
| `cat` | Primary interactive element |
| `cat-body` | Receives reaction CSS classes |
| `cat-face` | Contains all facial features |
| `cat-cheek--left/right` | Blushing overlays |
| `cat-mouth-smile/frown/open` | Mouth expression switching |
| `cat-eye-blink` | Blink overlay (hidden by default) |
| `cat-eye-wide` | Wide-eye overlay (surprised) |
| `pet-count` | Text node updated by DOM updater |
| `counter-display` | aria-live region for screen readers |
| `reaction-message` | aria-live polite region |
| `particle-container` | All spawned particles appended here |
| `sound-toggle` | Sound on/off button |
| `reset-button` | First-step reset trigger |
| `reset-confirm` | Confirmation dialog (hidden initially) |

---

## Data Models

### Session_State

```javascript
// Exact structure — no additional top-level fields permitted (Req 21.1)
const Session_State = {
  petCount: 0,                         // non-negative integer
  soundEnabled: false,                 // boolean, false on init (Req 16.2)
  milestones: {
    10:  false,                        // boolean Milestone_Flag
    25:  false,
    50:  false,
    100: false
  }
};
```

### Reaction Registry Entry

```javascript
// One entry per reaction type; 8 total entries in REACTION_REGISTRY array
{
  name:       'Happy',                 // display name
  cssClass:   'reaction--happy',      // class applied to #cat-body
  expression: {
    eyes:     'blink',                // 'normal' | 'blink' | 'wide' | 'closed'
    mouth:    'smile',                // 'smile' | 'frown' | 'open'
    cheeks:   false                   // boolean — show cheek overlays
  },
  animation:  'bounce',               // keyframe name applied to #cat
  animDuration: 800,                  // ms
  particles: {
    types: ['heart'],
    countMin: 3,
    countMax: 6
  },
  messages: [
    'Yay! ♡', 'Best human ever!',
    'More cuddles, please!', 'I love you!'
  ],
  sound: 'happy'                      // key into SOUND_FUNCTIONS map
}
```

### Particle Descriptor

```javascript
{
  type:      'heart',                  // string key into PARTICLE_RENDERERS
  x:         number,                  // spawn origin x (px, relative to particle-container)
  y:         number,                  // spawn origin y (px, relative to particle-container)
  dx:        number,                  // horizontal offset delta
  dy:        number,                  // upward translate distance (negative = up)
  rotation:  number,                  // degrees, e.g. -30 to 30
  scale:     number,                  // end scale multiplier
  duration:  number,                  // animation ms
  delay:     number                   // animation-delay ms (for stagger)
}
```

---

## Visual System

### Color Palette

| Token | Hex | Usage |
|---|---|---|
| `--bg-start` | `#FFDEED` | Gradient start (pink) |
| `--bg-end` | `#B5FFFC` | Gradient end (cyan) |
| `--cat-body` | `#F9E4B7` | Cat fur, warm cream |
| `--cat-inner-ear` | `#F4A7B9` | Inner ear, pastel pink |
| `--cat-eye` | `#5C4033` | Iris, warm dark brown |
| `--cat-pupil` | `#1A1A2E` | Pupil, near-black |
| `--cat-nose` | `#F4A7B9` | Nose, pastel pink |
| `--cat-mouth` | `#5C4033` | Mouth lines |
| `--cat-whisker` | `#888` | Whiskers, mid-gray |
| `--cat-cheek` | `#FFB3C6` | Blush overlay, hot pink |
| `--cat-cushion` | `#E8D5F5` | Cushion, pastel lavender |
| `--panel-text` | `#3D2C2C` | Body text on panels — ≥4.5:1 on `bg-white/30` over gradient |
| `--heading-text` | `#2D1B1B` | Heading, darkest — ≥4.5:1 verified |
| `--particle-heart` | `#FF6B9D` | Heart particles |
| `--particle-star` | `#FFD700` | Star particles |
| `--particle-paw` | `#C084FC` | Paw print particles |
| `--particle-sparkle` | `#A5F3FC` | Sparkle particles |
| `--particle-steam` | `#94A3B8` | Steam / angry puff |
| `--speed-line` | `#CBD5E1` | Zoomies speed streaks |

### Typography Scale

| Element | Tailwind class | Size |
|---|---|---|
| `<h1>` heading | `text-3xl sm:text-4xl font-bold` | 1.875rem / 2.25rem |
| Subheading `<p>` | `text-base sm:text-lg` | 1rem / 1.125rem |
| Counter `#counter-display` | `text-xl sm:text-2xl font-semibold` | 1.25rem / 1.5rem |
| Reaction message | `text-lg sm:text-xl italic` | 1.125rem / 1.25rem |
| Instruction label | `text-sm text-center` | 0.875rem |
| Controls | `text-2xl` (emoji icons) | 1.5rem |

### Cat Anatomy in CSS

```css
/* ── Body ─────────────────────────────────────────── */
#cat-body {
  position: relative;
  width: 180px; height: 160px;
  background: var(--cat-body);
  border-radius: 50% 50% 45% 45% / 55% 55% 45% 45%;
  /* Responsive: 120px on mobile, 180px on md+ */
}

/* ── Ears ─────────────────────────────────────────── */
.cat-ear {
  position: absolute; top: -28px;
  width: 40px; height: 44px;
  background: var(--cat-body);
  clip-path: polygon(50% 0%, 0% 100%, 100% 100%); /* triangle */
  border-radius: 4px 4px 0 0;
}
.cat-ear--left  { left: 20px; transform: rotate(-10deg); }
.cat-ear--right { right: 20px; transform: rotate(10deg); }

.cat-ear-inner {
  /* smaller pink triangle inset inside ear */
  position: absolute; bottom: 4px; left: 8px;
  width: 24px; height: 28px;
  background: var(--cat-inner-ear);
  clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
}

/* ── Eyes (normal state) ──────────────────────────── */
.cat-eye {
  position: absolute; top: 52px;
  width: 28px; height: 28px;
  background: var(--cat-eye);
  border-radius: 50%;
}
.cat-eye--left  { left: 38px; }
.cat-eye--right { right: 38px; }

.cat-pupil {
  position: absolute; top: 6px; left: 6px;
  width: 16px; height: 16px;
  background: var(--cat-pupil);
  border-radius: 50%;
}

/* Eye states — toggled by JS adding class to #cat-face */
.cat-eye-blink {
  /* horizontal oval covering the eye circle */
  position: absolute; top: 50%; left: 0; transform: translateY(-50%);
  width: 100%; height: 8px;
  background: var(--cat-eye);
  border-radius: 50%;
  display: none; /* shown during blink / purring / sleepy */
}
.cat-eye-wide {
  /* pupil scaled up — parent circle gets scale(1.2) via class */
  display: none; /* shown during surprised */
}

/* Expression classes applied to #cat-face */
.face--blink   .cat-eye-blink  { display: block; }
.face--blink   .cat-pupil      { display: none; }
.face--wide    .cat-eye        { transform: scale(1.25); }
.face--closed  .cat-eye-blink  { display: block; }
.face--closed  .cat-pupil      { display: none; }

/* ── Cheeks ───────────────────────────────────────── */
.cat-cheek {
  position: absolute; top: 68px;
  width: 28px; height: 16px;
  background: var(--cat-cheek);
  border-radius: 50%;
  opacity: 0;              /* hidden by default */
  transition: opacity 0.2s;
}
.cat-cheek--left  { left: 22px; }
.cat-cheek--right { right: 22px; }
.face--blushing .cat-cheek { opacity: 0.55; } /* 0.4–0.7 range, Req 7.1 */

/* ── Nose ─────────────────────────────────────────── */
#cat-nose {
  position: absolute; top: 82px; left: 50%; transform: translateX(-50%);
  width: 14px; height: 10px;
  background: var(--cat-nose);
  border-radius: 50% 50% 40% 40%;
}

/* ── Mouth ────────────────────────────────────────── */
.cat-mouth-smile {
  position: absolute; top: 96px; left: 50%; transform: translateX(-50%);
  width: 28px; height: 14px;
  border: 3px solid var(--cat-mouth);
  border-top: none;
  border-radius: 0 0 20px 20px;
}
.cat-mouth-frown {
  position: absolute; top: 100px; left: 50%; transform: translateX(-50%);
  width: 28px; height: 14px;
  border: 3px solid var(--cat-mouth);
  border-bottom: none;
  border-radius: 20px 20px 0 0;
}
.cat-mouth-open {
  position: absolute; top: 94px; left: 50%; transform: translateX(-50%);
  width: 18px; height: 16px;
  background: var(--cat-mouth);
  border-radius: 50%;
}

/* ── Whiskers ─────────────────────────────────────── */
.cat-whiskers { position: absolute; top: 84px; }
.cat-whiskers--left  { left: -36px; }
.cat-whiskers--right { right: -36px; transform: scaleX(-1); }
.cat-whisker {
  width: 48px; height: 2px;
  background: var(--cat-whisker);
  border-radius: 1px;
  margin: 5px 0;
  transform-origin: right center;
}
.cat-whiskers--left  .cat-whisker:nth-child(1) { transform: rotate(10deg); }
.cat-whiskers--left  .cat-whisker:nth-child(2) { transform: rotate(0deg); }
.cat-whiskers--left  .cat-whisker:nth-child(3) { transform: rotate(-10deg); }

/* ── Cushion ──────────────────────────────────────── */
#cat-cushion {
  position: absolute; bottom: -16px;
  width: 160px; height: 40px;
  background: var(--cat-cushion);
  border-radius: 50%;
  filter: blur(8px);
  opacity: 0.7;
}
```

### Glassmorphism Panel Styles

```css
.glass-panel {
  background: rgba(255, 255, 255, 0.30);   /* Tailwind bg-white/30 */
  backdrop-filter: blur(16px);              /* Tailwind backdrop-blur-lg */
  -webkit-backdrop-filter: blur(16px);
  border: 1px solid rgba(255, 255, 255, 0.40); /* border-white/40 */
  border-radius: 1.25rem;                   /* rounded-2xl */
}
```

### Responsive Layout

Mobile-first. The layout is a vertical flex column, centered.

| Breakpoint | Cat body width | Card padding | Font scale |
|---|---|---|---|
| 320 px (default) | 120 px | `p-4` | base |
| 640 px (`sm:`) | 150 px | `p-6` | +1 step |
| 1024 px (`md:`) | 180 px | `p-8` | +1 step |
| 1280 px+ (`lg:`) | 200 px | `p-10` | stable |

```css
#cat-body {
  width: 120px; height: 108px;          /* mobile default */
}
@media (min-width: 640px) {
  #cat-body { width: 150px; height: 134px; }
}
@media (min-width: 1024px) {
  #cat-body { width: 180px; height: 160px; }
}
```

The `#cat-wrapper` is sized in Tailwind: `w-48 h-48 sm:w-56 sm:h-56 md:w-64 md:h-64`.

Background decorations use absolute/fixed positioning with `pointer-events: none` and `z-index: 0`; the cat panel has `z-index: 10`. Decoration elements are positioned at the edges and corners of the viewport so they cannot intersect the cat bounding box at any supported viewport size.

---

## CSS Keyframe Definitions

All defined inside the `<style>` block. All animation properties use only `transform` and `opacity` per Req 20.2. `animation-fill-mode: forwards` is applied to all one-shot reaction animations per Req 15.3.

```css
/* ── float ───────────────────────────────────────────
   Idle cat vertical oscillation. Duration: 3s (Req 2.3a). ─── */
@keyframes float {
  0%, 100% { transform: translateY(0px) scale(1.00); }
  50%       { transform: translateY(-12px) scale(1.03); }
}

/* ── squash ──────────────────────────────────────────
   Press feedback. Max duration 150ms (Req 3.3, 15.2). ─── */
@keyframes squash {
  0%   { transform: scale(1.00, 1.00); }
  40%  { transform: scale(1.20, 0.75); }
  70%  { transform: scale(0.90, 1.10); }
  100% { transform: scale(1.00, 1.00); }
}

/* ── bounce ──────────────────────────────────────────
   Used by Happy (800ms), Surprised (500ms), Playful (1200ms).
   --bounce-y controls amplitude: base = -40px, 2× = -80px.  ─── */
@keyframes bounce {
  0%   { transform: translateY(0px) scale(1.00); }
  30%  { transform: translateY(var(--bounce-y, -40px)) scale(1.05); }
  55%  { transform: translateY(0px) scale(0.98); }
  75%  { transform: translateY(calc(var(--bounce-y, -40px) * 0.4)) scale(1.02); }
  100% { transform: translateY(0px) scale(1.00); }
}

/* ── bounce-surprised ────────────────────────────────
   Peaks at scale 1.2× (Req 8.1). Applied via reaction class override. ─── */
@keyframes bounce-surprised {
  0%   { transform: scale(1.00); }
  30%  { transform: scale(1.25) translateY(-20px); }
  60%  { transform: scale(0.95) translateY(0); }
  100% { transform: scale(1.00); }
}

/* ── bounce-playful ──────────────────────────────────
   2× amplitude + 360° spin (Req 11.1). ─── */
@keyframes bounce-playful {
  0%   { transform: translateY(0px)   rotate(0deg)   scale(1.00); }
  30%  { transform: translateY(-80px) rotate(180deg) scale(1.05); }
  60%  { transform: translateY(-80px) rotate(360deg) scale(1.05); }
  80%  { transform: translateY(0px)   rotate(360deg) scale(0.97); }
  100% { transform: translateY(0px)   rotate(360deg) scale(1.00); }
}

/* ── wiggle ──────────────────────────────────────────
   Horizontal oscillation. --wiggle-x sets amplitude.
   Purring: amplitude ≤6px, cycle 1.2–2s (Req 6.1).
   Grumpy:  amplitude 8–16px, cycle ≤200ms (Req 12.1).
   Playful: wiggle appended after bounce-playful via animationend chaining. ─── */
@keyframes wiggle {
  0%, 100% { transform: translateX(0); }
  20%       { transform: translateX(calc(-1 * var(--wiggle-x, 8px))); }
  50%       { transform: translateX(var(--wiggle-x, 8px)); }
  80%       { transform: translateX(calc(-1 * var(--wiggle-x, 8px))); }
}

/* ── zoomies ─────────────────────────────────────────
   Horizontal dash, total ≤600ms (Req 10.1). ─── */
@keyframes zoomies {
  0%   { transform: translateX(0px)   scaleX(1.0); }
  20%  { transform: translateX(60px)  scaleX(1.2); }
  40%  { transform: translateX(-60px) scaleX(1.2); }
  60%  { transform: translateX(40px)  scaleX(1.0); }
  80%  { transform: translateX(-20px) scaleX(1.0); }
  100% { transform: translateX(0px)   scaleX(1.0); }
}

/* ── blush ───────────────────────────────────────────
   Head tilt 10–20°, total 600–1200ms (Req 7.1). ─── */
@keyframes blush {
  0%   { transform: rotate(0deg); }
  30%  { transform: rotate(15deg); }
  60%  { transform: rotate(-8deg); }
  80%  { transform: rotate(10deg); }
  100% { transform: rotate(0deg); }
}

/* ── sleepy-tilt ─────────────────────────────────────
   Scale to 90%, tilt −6°, 800–1400ms (Req 9.1). ─── */
@keyframes sleepy-tilt {
  0%   { transform: scale(1.00) rotate(0deg); }
  40%  { transform: scale(0.90) rotate(-6deg); }
  70%  { transform: scale(0.92) rotate(-4deg); }
  100% { transform: scale(0.90) rotate(-6deg); }
}

/* ── fade-in ─────────────────────────────────────────
   ≥150ms (Req 13.3). ─── */
@keyframes fade-in {
  from { opacity: 0; transform: translateY(4px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* ── fade-out ────────────────────────────────────────
   ≥150ms (Req 13.3). ─── */
@keyframes fade-out {
  from { opacity: 1; transform: translateY(0); }
  to   { opacity: 0; transform: translateY(-4px); }
}

/* ── float-up ────────────────────────────────────────
   Particle upward movement + fade. Used by most particle types. ─── */
@keyframes float-up {
  0%   { transform: translateY(0)   rotate(var(--p-rot-start, 0deg)) scale(1.0); opacity: 1; }
  60%  { transform: translateY(var(--p-dy, -70px)) rotate(var(--p-rot-end, 15deg)) scale(var(--p-scale, 1.4)); opacity: 0.9; }
  100% { transform: translateY(var(--p-dy, -70px)) rotate(var(--p-rot-end, 15deg)) scale(var(--p-scale, 1.4)); opacity: 0; }
}

/* ── speed-line ──────────────────────────────────────
   Horizontal streak for zoomies. ─── */
@keyframes speed-line {
  0%   { transform: translateX(0)     scaleX(0); opacity: 0; }
  20%  { transform: translateX(-10px) scaleX(1); opacity: 1; }
  100% { transform: translateX(var(--sl-dx, -80px)) scaleX(0.3); opacity: 0; }
}
```

### CSS Custom Properties on Particles

Each particle element receives inline CSS custom properties set at spawn time:

| Property | Purpose |
|---|---|
| `--p-dy` | Vertical translation distance (negative = up) |
| `--p-rot-start` | Initial rotation |
| `--p-rot-end` | Final rotation |
| `--p-scale` | Scale at animation peak |
| `--p-color` | Fill/text color |
| `--sl-dx` | Speed-line horizontal travel |
| `--wiggle-x` | Wiggle amplitude |
| `--bounce-y` | Bounce travel distance |

### Reduced Motion

```css
@media (prefers-reduced-motion: reduce) {
  /* Kill all non-essential animations (Req 15.6, 18.9) */
  #cat, .particle, #bg-decorations *, #cat-body {
    animation: none !important;
    transition: none !important;
  }
  /* Reaction messages still use class change but no CSS animation */
  #reaction-message { animation: none !important; }
}
```

The JS init function detects reduced motion with:
```javascript
const REDUCED_MOTION = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
```
This flag is checked before every particle spawn and every animation class application.

---

## State Management

### Functions

```javascript
// Initializes or resets Session_State to safe defaults (Req 21.3)
function initState() {
  Session_State.petCount    = 0;
  Session_State.soundEnabled = false;
  Session_State.milestones   = { 10: false, 25: false, 50: false, 100: false };
}

// Increments petCount by exactly 1 (Req 3.1)
// Updates display and checks milestones in the same tick (Req 3.4)
function incrementPetCount() {
  Session_State.petCount += 1;
  updateCounterDisplay();
  checkMilestones();
}

// Toggles soundEnabled; updates UI icon (Req 16.3, 16.4)
function toggleSound() {
  Session_State.soundEnabled = !Session_State.soundEnabled;
  updateSoundIcon();
}

// Resets petCount and clears reaction state; does NOT touch milestones (Req 14.6, 17.7)
function resetSession() {
  Session_State.petCount = 0;
  cancelActiveReaction();
  updateCounterDisplay();
}

// Evaluates milestones after each increment (Req 14.1–14.5)
function checkMilestones() {
  const n = Session_State.petCount;
  [10, 25, 50, 100].forEach(threshold => {
    if (n === threshold && !Session_State.milestones[threshold]) {
      Session_State.milestones[threshold] = true;
      triggerMilestone(threshold);
    }
  });
}
```

### State Validation on Init

```javascript
function validateAndNormalizeState(raw) {
  // Req 21.3: substitute defaults for missing fields
  const state = {
    petCount: (typeof raw.petCount === 'number' && Number.isInteger(raw.petCount) && raw.petCount >= 0)
                ? raw.petCount : 0,   // Req 21.4
    soundEnabled: typeof raw.soundEnabled === 'boolean' ? raw.soundEnabled : false,
    milestones: {
      10:  typeof raw?.milestones?.[10]  === 'boolean' ? raw.milestones[10]  : false,
      25:  typeof raw?.milestones?.[25]  === 'boolean' ? raw.milestones[25]  : false,
      50:  typeof raw?.milestones?.[50]  === 'boolean' ? raw.milestones[50]  : false,
      100: typeof raw?.milestones?.[100] === 'boolean' ? raw.milestones[100] : false
    }
  };
  return state;
}
```

### DOM Updater

```javascript
// Updates the counter text node (Req 3.2)
// Format: "🐾 Total Pets: {n}"
function updateCounterDisplay() {
  document.getElementById('pet-count').textContent = Session_State.petCount;
}

// Updates sound toggle icon (Req 17.1)
function updateSoundIcon() {
  const btn = document.getElementById('sound-toggle');
  btn.textContent = Session_State.soundEnabled ? '🔊' : '🔇';
  btn.setAttribute('aria-pressed', Session_State.soundEnabled ? 'true' : 'false');
}
```

### RAF Batching for Burst Inputs

```javascript
// Req 20.3: max 1 DOM text update per animation frame during rapid bursts
let _rafPending = false;
function scheduleCounterUpdate() {
  if (_rafPending) return;
  _rafPending = true;
  requestAnimationFrame(() => {
    updateCounterDisplay();
    _rafPending = false;
  });
}
```

---

## Reaction System Design

### Reaction Registry

```javascript
const REACTION_REGISTRY = [
  {
    name: 'Happy',
    cssClass: 'reaction--happy',
    expression: { eyes: 'blink', mouth: 'smile', cheeks: false },
    animation: 'bounce', animDuration: 800,
    particles: { types: ['heart'], countMin: 3, countMax: 6 },
    messages: ['Yay! ♡', 'Best human ever!', 'More cuddles, please!', 'I love you!'],
    sound: 'happy'
  },
  {
    name: 'Purring',
    cssClass: 'reaction--purring',
    expression: { eyes: 'closed', mouth: 'smile', cheeks: false },
    animation: 'wiggle', animDuration: 1600,  // uses --wiggle-x: 5px
    particles: { types: ['heart'], countMin: 2, countMax: 4 },
    messages: ['Purrrrr... ♡', 'This is the life ♡', 'Purrr...'],
    sound: 'purring'
  },
  {
    name: 'Blushing',
    cssClass: 'reaction--blushing',
    expression: { eyes: 'blink', mouth: 'smile', cheeks: true },
    animation: 'blush', animDuration: 900,
    particles: { types: ['heart', 'sparkle'], countMin: 4, countMax: 6 },
    messages: ["You're making me blush!", "Stop, I'm blushing!", "Eep! You're too sweet!"],
    sound: 'generic'
  },
  {
    name: 'Surprised',
    cssClass: 'reaction--surprised',
    expression: { eyes: 'wide', mouth: 'open', cheeks: false },
    animation: 'bounce-surprised', animDuration: 500,
    particles: { types: ['exclamation', 'sparkle'], countMin: 4, countMax: 6 },
    messages: ['Meow?!', 'Oh! You got me!', '!'],
    sound: 'surprised'
  },
  {
    name: 'Sleepy',
    cssClass: 'reaction--sleepy',
    expression: { eyes: 'closed', mouth: 'smile', cheeks: false },
    animation: 'sleepy-tilt', animDuration: 1100,
    particles: { types: ['zzz'], countMin: 3, countMax: 5 },
    messages: ['Zzz... just five more minutes', 'Five more minutes...', 'Zzz...'],
    sound: 'generic'
  },
  {
    name: 'Zoomies',
    cssClass: 'reaction--zoomies',
    expression: { eyes: 'wide', mouth: 'open', cheeks: false },
    animation: 'zoomies', animDuration: 550,
    particles: { types: ['speed-line'], countMin: 3, countMax: 5 },
    messages: ['ZOOMIES!!!', 'NYOOOM!', "Can't catch me!"],
    sound: 'zoomies'
  },
  {
    name: 'Playful',
    cssClass: 'reaction--playful',
    expression: { eyes: 'wide', mouth: 'smile', cheeks: false },
    animation: 'bounce-playful', animDuration: 1200,
    particles: { types: ['star', 'paw'], countMin: 4, countMax: 6 },
    messages: ['Catch me if you can!', "Let's play!", 'Catch me!'],
    sound: 'generic'
  },
  {
    name: 'Grumpy',
    cssClass: 'reaction--grumpy',
    expression: { eyes: 'normal', mouth: 'frown', cheeks: false },
    animation: 'wiggle', animDuration: 800,  // uses --wiggle-x: 12px, fast cycle
    particles: { types: ['steam'], countMin: 2, countMax: 4 },
    messages: ['Hey! Gentle pets, please!', 'Hmph!', 'Do NOT disturb!'],
    sound: 'generic'
  }
];
```

### Reaction Selection Algorithm

```javascript
// Weighted random selection (Req 4.1)
// Weights: Happy(20), Purring(15), Blushing(12), Surprised(10),
//          Sleepy(12), Zoomies(10), Playful(12), Grumpy(9) — total 100
const REACTION_WEIGHTS = [20, 15, 12, 10, 12, 10, 12, 9];

function selectReaction() {
  const total = REACTION_WEIGHTS.reduce((a, b) => a + b, 0);
  let r = Math.random() * total;
  for (let i = 0; i < REACTION_REGISTRY.length; i++) {
    r -= REACTION_WEIGHTS[i];
    if (r <= 0) return REACTION_REGISTRY[i];
  }
  return REACTION_REGISTRY[0]; // fallback
}
```

### Reaction Lifecycle

```
1. handleCatInteraction()
   │
   ├── cancelActiveReaction()          // abort any in-progress reaction within 16ms
   │
   ├── applyExpression(reaction)       // set face class on #cat-face
   │                                   // show/hide cheeks, set mouth variant
   │
   ├── [set CSS custom properties]     // --wiggle-x, --bounce-y on #cat-body
   │
   ├── applySoundEffect(reaction)      // Sound Engine
   │
   ├── [apply squash class]            // #cat gets class 'anim--squash'
   │
   ├── [wait 150ms via setTimeout]     // squash completes
   │
   ├── [apply reaction.cssClass]       // #cat-body gets e.g. 'reaction--happy'
   │
   ├── spawnParticles(reaction)        // Particle System
   │
   ├── showReactionMessage(reaction)   // DOM Updater
   │
   └── [animationend on #cat-body] ──► cleanupReaction()
                                         │
                                         ├── remove reaction CSS class
                                         ├── remove expression class from #cat-face
                                         ├── hide cheeks
                                         ├── remove inline style overrides
                                         └── restoreIdleAnimation()
```

### Cancellation Path

```javascript
// Module-level reference to active reaction state
let _activeReaction = null;
let _squashTimeout  = null;
let _purgingIds     = [];   // particle element refs for current reaction

function cancelActiveReaction() {
  // Remove squash timeout if still pending
  if (_squashTimeout) {
    clearTimeout(_squashTimeout);
    _squashTimeout = null;
  }

  if (!_activeReaction) return;

  const cat     = document.getElementById('cat');
  const catBody = document.getElementById('cat-body');
  const catFace = document.getElementById('cat-face');

  // Remove reaction CSS classes immediately (within 16ms)
  catBody.classList.remove(_activeReaction.cssClass, 'anim--squash');
  catFace.className = '';   // strip all expression classes
  cat.classList.remove('anim--squash');

  // Remove inline animation overrides (Req 15.5)
  catBody.style.removeProperty('animation');
  catBody.style.removeProperty('--wiggle-x');
  catBody.style.removeProperty('--bounce-y');

  // Remove all orphaned particles for this reaction (Req 4.5, 4.6)
  _purgingIds.forEach(el => {
    if (el.parentNode) {
      el.removeEventListener('animationend', el._cleanupHandler);
      el.parentNode.removeChild(el);
    }
  });
  _purgingIds = [];

  // Stop any running purring oscillator
  if (_purringOscillator) {
    stopPurring();
  }

  _activeReaction = null;
}
```

---

## Particle System Design

### `spawnParticle` Function

```javascript
/**
 * Creates a single particle DOM element and appends it to #particle-container.
 * Enforces max-20 live particle cap (Req 20.1).
 * Attaches animationend listener for self-cleanup (Req 15.4, 18.6).
 *
 * @param {string} type       - Particle type key
 * @param {number} originX    - Spawn x (px from left of particle-container)
 * @param {number} originY    - Spawn y (px from top of particle-container)
 * @param {object} opts       - { dy, rotStart, rotEnd, scale, duration, delay, color }
 */
function spawnParticle(type, originX, originY, opts = {}) {
  const container = document.getElementById('particle-container');

  // Enforce cap: remove oldest particle if at limit (Req 20.1)
  while (container.children.length >= 20) {
    const oldest = container.firstChild;
    oldest.removeEventListener('animationend', oldest._cleanupHandler);
    container.removeChild(oldest);
  }

  const el = document.createElement('div');
  el.className = `particle particle--${type}`;
  el.style.cssText = `
    position: absolute;
    left: ${originX}px;
    top:  ${originY}px;
    --p-dy:        ${opts.dy         ?? -70}px;
    --p-rot-start: ${opts.rotStart   ?? 0}deg;
    --p-rot-end:   ${opts.rotEnd     ?? 15}deg;
    --p-scale:     ${opts.scale      ?? 1.4};
    --p-color:     ${opts.color      ?? '#FF6B9D'};
    animation: float-up ${opts.duration ?? 800}ms ease-out
               ${opts.delay ?? 0}ms forwards;
    pointer-events: none;
    user-select: none;
  `;

  // Inject particle inner markup
  el.innerHTML = PARTICLE_RENDERERS[type] ?? '';

  // Self-cleanup on animationend (Req 15.4, 18.6)
  const handler = () => {
    el.removeEventListener('animationend', handler); // prevent leak (Req 18.6)
    if (el.parentNode) el.parentNode.removeChild(el);
    // Remove from _purgingIds if still tracked
    const idx = _purgingIds.indexOf(el);
    if (idx > -1) _purgingIds.splice(idx, 1);
  };
  el._cleanupHandler = handler;
  el.addEventListener('animationend', handler);

  container.appendChild(el);
  _purgingIds.push(el);

  return el;
}
```

### Particle Renderers

```javascript
const PARTICLE_RENDERERS = {
  // Inline SVG heart (Req 5.2)
  heart: `<svg width="20" height="18" viewBox="0 0 20 18" fill="var(--p-color)" aria-hidden="true">
    <path d="M10 17s-8-5.4-8-11a5 5 0 0110 0 5 5 0 0110 0c0 5.6-8 11-8 11z"/>
  </svg>`,

  // Inline SVG star (Req 11.2)
  star: `<svg width="20" height="20" viewBox="0 0 20 20" fill="var(--p-color)" aria-hidden="true">
    <polygon points="10,1 12.5,7.5 19,7.5 14,12 16,18.5 10,14.5 4,18.5 6,12 1,7.5 7.5,7.5"/>
  </svg>`,

  // Inline SVG paw print (Req 11.2)
  paw: `<svg width="22" height="22" viewBox="0 0 22 22" fill="var(--p-color)" aria-hidden="true">
    <circle cx="7" cy="6" r="3"/>
    <circle cx="15" cy="6" r="3"/>
    <circle cx="4" cy="11" r="2.2"/>
    <circle cx="18" cy="11" r="2.2"/>
    <ellipse cx="11" cy="16" rx="5.5" ry="4"/>
  </svg>`,

  // Inline SVG sparkle (Req 7.2, 8.2)
  sparkle: `<svg width="18" height="18" viewBox="0 0 18 18" fill="var(--p-color)" aria-hidden="true">
    <path d="M9 0 L10.2 7.8 L18 9 L10.2 10.2 L9 18 L7.8 10.2 L0 9 L7.8 7.8 Z"/>
  </svg>`,

  // Exclamation text (Req 8.2)
  exclamation: `<span style="font-size:24px;font-weight:bold;color:var(--p-color);line-height:1">!</span>`,

  // Steam cloud — SVG circle cluster (Req 12.2)
  steam: `<svg width="24" height="18" viewBox="0 0 24 18" fill="var(--p-color)" opacity="0.7" aria-hidden="true">
    <circle cx="6" cy="12" r="5"/>
    <circle cx="12" cy="9"  r="6"/>
    <circle cx="18" cy="12" r="5"/>
  </svg>`,

  // Speed line — horizontal CSS-only div (Req 10.2)
  // Uses speed-line keyframe instead of float-up
  'speed-line': `<div style="
    width:60px; height:3px;
    background: linear-gradient(to left, transparent, var(--p-color));
    border-radius: 2px;
  "></div>`,

  // Zzz text particle (Req 9.2)
  zzz: `<span style="font-size:20px;font-weight:bold;color:var(--p-color);letter-spacing:1px">Zzz</span>`
};
```

### Per-Reaction Particle Spawn Helper

```javascript
/**
 * Spawns all particles for a given reaction.
 * Reads cat bounding box to set origin.
 */
function spawnReactionParticles(reaction) {
  if (REDUCED_MOTION) return;   // Req 15.6

  const catEl  = document.getElementById('cat');
  const rect   = catEl.getBoundingClientRect();
  const container = document.getElementById('particle-container');
  const cRect  = container.getBoundingClientRect();
  const cx     = rect.left - cRect.left + rect.width  / 2;
  const cy     = rect.top  - cRect.top  + rect.height / 2;

  const count = randInt(reaction.particles.countMin, reaction.particles.countMax);
  const types = reaction.particles.types;

  for (let i = 0; i < count; i++) {
    const type  = types[i % types.length];
    const angle = (360 / count) * i;
    const r     = 20 + Math.random() * 20;
    const ox    = cx + Math.cos(toRad(angle)) * r;
    const oy    = cy + Math.sin(toRad(angle)) * r;

    // Speed-line particles use a different keyframe
    if (type === 'speed-line') {
      spawnSpeedLine(ox, oy, i);
      continue;
    }

    spawnParticle(type, ox, oy, {
      dy:       -(70  + Math.random() * 40),       // −70 to −110 px upward
      rotStart: -30 + Math.random() * 60,
      rotEnd:   -30 + Math.random() * 60,
      scale:    1.2 + Math.random() * 0.6,
      duration: 700 + Math.random() * 300,
      delay:    reaction.name === 'Sleepy' ? i * 200 : 0,  // stagger for Zzz (Req 9.2)
      color:    PARTICLE_COLORS[type]
    });
  }
}

function spawnSpeedLine(ox, oy, idx) {
  const container = document.getElementById('particle-container');
  const el = document.createElement('div');
  el.className = 'particle particle--speed-line';
  el.style.cssText = `
    position: absolute;
    left: ${ox}px; top: ${oy + idx * 10 - 15}px;
    --p-color: ${PARTICLE_COLORS['speed-line']};
    --sl-dx: ${-(80 + Math.random() * 40)}px;
    animation: speed-line 450ms ease-out forwards;
    pointer-events: none;
  `;
  el.innerHTML = PARTICLE_RENDERERS['speed-line'];
  const handler = () => {
    el.removeEventListener('animationend', handler);
    if (el.parentNode) el.parentNode.removeChild(el);
  };
  el._cleanupHandler = handler;
  el.addEventListener('animationend', handler);
  container.appendChild(el);
  _purgingIds.push(el);
}

const PARTICLE_COLORS = {
  heart:       '#FF6B9D',
  star:        '#FFD700',
  paw:         '#C084FC',
  sparkle:     '#A5F3FC',
  exclamation: '#FF6B6B',
  steam:       '#94A3B8',
  'speed-line':'#CBD5E1',
  zzz:         '#8B9DC3'
};
```

---

## Animation Orchestration

### Squash → Reaction Sequencing

```javascript
let _squashTimeout = null;

function startReaction(reaction) {
  _activeReaction = reaction;
  const cat     = document.getElementById('cat');
  const catBody = document.getElementById('cat-body');

  // Step 1: Apply squash immediately (Req 3.3, 15.2)
  cat.classList.add('anim--squash');
  // squash duration = 130ms; remove class after it completes
  _squashTimeout = setTimeout(() => {
    cat.classList.remove('anim--squash');

    // Step 2: Apply expression
    applyExpression(reaction.expression);

    // Step 3: Set animation CSS custom properties
    if (reaction.name === 'Purring') {
      catBody.style.setProperty('--wiggle-x', '5px');
    } else if (reaction.name === 'Grumpy') {
      catBody.style.setProperty('--wiggle-x', '12px');
    } else if (reaction.name === 'Playful') {
      catBody.style.setProperty('--bounce-y', '-80px');
    }

    // Step 4: Apply reaction animation class
    catBody.classList.add(reaction.cssClass);
    catBody.style.animation =
      `${reaction.animation} ${reaction.animDuration}ms ease-in-out forwards`;

    // Step 5: Spawn particles and message
    spawnReactionParticles(reaction);
    showReactionMessage(reaction);

    // Step 6: Wait for animationend → cleanup
    const endHandler = () => {
      catBody.removeEventListener('animationend', endHandler);
      if (_activeReaction === reaction) {
        cleanupReaction(reaction);
      }
    };
    catBody.addEventListener('animationend', endHandler);

  }, 140); // 140ms covers squash keyframe (≤150ms, Req 3.3)
}
```

### Idle Animation Restoration

```javascript
function restoreIdleAnimation() {
  const catBody = document.getElementById('cat-body');
  catBody.style.removeProperty('animation');
  // The .idle-float class (applied permanently) re-starts the `float` keyframe
  // via CSS: .idle-float { animation: float 3s ease-in-out infinite; }
  // Removing the inline style re-exposes the class-based rule.
}
```

### Reduced Motion Detection

```javascript
// Evaluated once at init; stored globally
const REDUCED_MOTION = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

function applyReducedMotionDefaults() {
  if (!REDUCED_MOTION) return;
  // Add a class that CSS @media rule already handles; also disable JS-spawned effects
  document.documentElement.classList.add('reduced-motion');
}
```

---

## Sound Engine Design

### AudioContext Lazy Init

```javascript
let _audioCtx = null;

// Called on first user gesture (Req 16.10)
function ensureAudioContext() {
  if (!_audioCtx) {
    _audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  }
  if (_audioCtx.state === 'suspended') {
    _audioCtx.resume(); // no await; fire-and-forget per spec guidance
  }
  return _audioCtx;
}
```

### Base Tone Function

```javascript
/**
 * Plays a single synthesized tone.
 * @param {number} freq       - Frequency in Hz
 * @param {number} duration   - Duration in ms
 * @param {string} type       - OscillatorNode type: 'sine' | 'triangle' | 'sawtooth' | 'square'
 * @param {number} gain       - Peak gain value (0–1)
 * @param {number} startDelay - Delay before note starts (s, default 0)
 */
function playTone(freq, duration, type = 'sine', gain = 0.3, startDelay = 0) {
  if (!Session_State.soundEnabled) return;
  const ctx = ensureAudioContext();
  const osc = ctx.createOscillator();
  const gn  = ctx.createGain();

  osc.connect(gn);
  gn.connect(ctx.destination);

  osc.type = type;
  osc.frequency.setValueAtTime(freq, ctx.currentTime + startDelay);

  // Envelope: quick attack, sustain, quick release
  gn.gain.setValueAtTime(0, ctx.currentTime + startDelay);
  gn.gain.linearRampToValueAtTime(gain, ctx.currentTime + startDelay + 0.01);
  gn.gain.linearRampToValueAtTime(0,    ctx.currentTime + startDelay + duration / 1000);

  osc.start(ctx.currentTime + startDelay);
  osc.stop(ctx.currentTime  + startDelay + duration / 1000 + 0.05);
}
```

### Per-Reaction Sound Functions

```javascript
// Generic pop/meow for any interaction (Req 16.5)
// Sine wave, 180Hz, 100ms
function playGenericPet() {
  playTone(180, 100, 'sine', 0.3);
  playTone(260, 80,  'sine', 0.2, 0.08);
}

// Happy: ascending 3-note sequence (Req 16.6)
// C5 → E5 → G5, each ≤200ms
function playHappySound() {
  playTone(523, 160, 'sine',     0.3, 0.00);   // C5
  playTone(659, 160, 'sine',     0.3, 0.17);   // E5
  playTone(784, 180, 'triangle', 0.3, 0.34);   // G5
}

// Purring: continuous LFO oscillation at 80Hz (Req 16.7)
// Oscillator is started and stored for the reaction duration
let _purringOscillator = null;
let _purringGain       = null;

function playPurringSound(durationMs) {
  if (!Session_State.soundEnabled) return;
  const ctx = ensureAudioContext();
  _purringOscillator = ctx.createOscillator();
  _purringGain       = ctx.createGain();
  _purringOscillator.connect(_purringGain);
  _purringGain.connect(ctx.destination);
  _purringOscillator.type = 'sawtooth';
  _purringOscillator.frequency.value = 80;   // 60–120 Hz (Req 16.7)
  _purringGain.gain.setValueAtTime(0.15, ctx.currentTime);
  _purringOscillator.start();
  // Auto-stop at reaction end
  _purringOscillator.stop(ctx.currentTime + durationMs / 1000);
}

function stopPurring() {
  if (_purringOscillator) {
    try { _purringOscillator.stop(); } catch (_) {}
    _purringOscillator.disconnect();
    _purringOscillator = null;
    _purringGain       = null;
  }
}

// Zoomies: pitch sweep ascending then descending (Req 16.8)
// Total 500ms
function playZoomiesSound() {
  const ctx = ensureAudioContext();
  if (!Session_State.soundEnabled) return;
  const osc = ctx.createOscillator();
  const gn  = ctx.createGain();
  osc.connect(gn);
  gn.connect(ctx.destination);
  osc.type = 'sawtooth';
  osc.frequency.setValueAtTime(200, ctx.currentTime);
  osc.frequency.linearRampToValueAtTime(800, ctx.currentTime + 0.25);  // ascending
  osc.frequency.linearRampToValueAtTime(200, ctx.currentTime + 0.50);  // descending
  gn.gain.setValueAtTime(0.2, ctx.currentTime);
  gn.gain.linearRampToValueAtTime(0, ctx.currentTime + 0.50);
  osc.start();
  osc.stop(ctx.currentTime + 0.55);
}

// Surprised: staccato ascending 2-note sequence (Req 16.9)
// Total <500ms
function playSurprisedSound() {
  playTone(400, 120, 'square', 0.25, 0.00);
  playTone(640, 120, 'square', 0.25, 0.15);
}

// Dispatcher called by handleCatInteraction
const SOUND_FUNCTIONS = {
  happy:     playHappySound,
  purring:   () => playPurringSound(1600),
  surprised: playSurprisedSound,
  zoomies:   playZoomiesSound,
  generic:   playGenericPet
};

function applySoundEffect(reaction) {
  if (!Session_State.soundEnabled) return;
  const fn = SOUND_FUNCTIONS[reaction.sound] ?? playGenericPet;
  fn();
}
```

---

## Milestone System

```javascript
function triggerMilestone(threshold) {
  if (REDUCED_MOTION) {
    // Still update flags; skip visual effects
    return;
  }

  switch (threshold) {
    case 10:
      triggerMilestone10();
      break;
    case 25:
      triggerMilestone25();
      break;
    case 50:
      triggerMilestone50();
      break;
    case 100:
      triggerMilestone100();
      break;
  }
}

// 10 pets: ≥8 hearts from cat center (Req 14.1)
function triggerMilestone10() {
  const catEl = document.getElementById('cat');
  const rect  = catEl.getBoundingClientRect();
  const container = document.getElementById('particle-container');
  const cRect = container.getBoundingClientRect();
  const cx = rect.left - cRect.left + rect.width  / 2;
  const cy = rect.top  - cRect.top  + rect.height / 2;

  for (let i = 0; i < 8; i++) {
    const angle = (360 / 8) * i;
    spawnParticle('heart', cx, cy, {
      dy:       -(80 + Math.random() * 40),
      rotStart: angle - 20,
      rotEnd:   angle + 20,
      scale:    1.5,
      duration: 900 + Math.random() * 200,
      delay:    i * 50,
      color:    '#FF6B9D'
    });
  }
}

// 25 pets: banner with exact text, auto-dismiss after 3000ms ±200ms (Req 14.2)
function triggerMilestone25() {
  const overlay = document.createElement('div');
  overlay.id = 'milestone-banner-25';
  overlay.setAttribute('role', 'status');
  overlay.setAttribute('aria-live', 'polite');
  overlay.textContent = '25 Pets! You\'re on a roll! 🌟';
  overlay.className = 'milestone-banner glass-panel';
  document.querySelector('main').appendChild(overlay);
  setTimeout(() => {
    if (overlay.parentNode) overlay.parentNode.removeChild(overlay);
  }, 3000);
}

// 50 pets: ≥20 particles of ≥3 types (Req 14.3)
function triggerMilestone50() {
  const types = ['heart', 'star', 'sparkle', 'paw'];
  const catEl = document.getElementById('cat');
  const rect  = catEl.getBoundingClientRect();
  const container = document.getElementById('particle-container');
  const cRect = container.getBoundingClientRect();
  const cx = rect.left - cRect.left + rect.width  / 2;
  const cy = rect.top  - cRect.top  + rect.height / 2;

  for (let i = 0; i < 20; i++) {
    const angle = (360 / 20) * i;
    const r     = 30 + Math.random() * 60;
    spawnParticle(types[i % types.length],
      cx + Math.cos(toRad(angle)) * r,
      cy + Math.sin(toRad(angle)) * r,
      {
        dy:       -(90 + Math.random() * 60),
        scale:    1.3 + Math.random() * 0.5,
        duration: 800 + Math.random() * 400,
        delay:    i * 40,
        color:    Object.values(PARTICLE_COLORS)[i % Object.values(PARTICLE_COLORS).length]
      });
  }
}

// 100 pets: achievement overlay, visible ≥4000ms (Req 14.4)
function triggerMilestone100() {
  const overlay = document.createElement('div');
  overlay.id = 'milestone-overlay-100';
  overlay.setAttribute('role', 'dialog');
  overlay.setAttribute('aria-label', 'Achievement unlocked');
  overlay.setAttribute('aria-modal', 'false'); // does not trap focus (Req 19.5)
  overlay.textContent = '🏆 100 Pets! You\'re the ultimate cat friend!';
  overlay.className = 'milestone-overlay glass-panel';
  // Escape key to dismiss (Req 19.5)
  const escHandler = (e) => {
    if (e.key === 'Escape') {
      dismissOverlay(overlay, escHandler);
    }
  };
  document.addEventListener('keydown', escHandler);
  document.querySelector('main').appendChild(overlay);
  setTimeout(() => dismissOverlay(overlay, escHandler), 4200); // ≥4000ms
}

function dismissOverlay(el, escHandler) {
  document.removeEventListener('keydown', escHandler);
  if (el.parentNode) el.parentNode.removeChild(el);
}
```

---

## Reaction Message System

```javascript
let _lastMessage = null;

function showReactionMessage(reaction) {
  const msgEl = document.getElementById('reaction-message');
  const pool  = reaction.messages;

  // Select a message that differs from the last (Req 4.3, 13.2)
  // Exception: first interaction has no prior message constraint
  let candidates = pool;
  if (_lastMessage !== null && pool.length > 1) {
    candidates = pool.filter(m => m !== _lastMessage);
  }
  const selected = candidates[Math.floor(Math.random() * candidates.length)];
  _lastMessage = selected;

  // Fade out current message, swap text, fade in (Req 13.3)
  if (REDUCED_MOTION) {
    msgEl.textContent = selected;
    return;
  }

  msgEl.style.animation = 'fade-out 160ms ease-out forwards';
  setTimeout(() => {
    msgEl.textContent = selected;
    msgEl.style.animation = 'fade-in 200ms ease-in forwards';
  }, 160);
}
```

---

## Interaction Flow

### Pointer / Keyboard Interaction

```
Pointer down or keydown(Enter/Space) on #cat
        │
        ├── [guard: prevent double-count for same gesture]
        │   pointerdown → set _pointerDownId = event.pointerId
        │   pointerup / keydown each fire handleCatInteraction once
        │
        ├── ensureAudioContext()           (first gesture only)
        │
        ├── cancelActiveReaction()
        │
        ├── incrementPetCount()
        │       └── updateCounterDisplay() (via RAF if burst, Req 20.3)
        │       └── checkMilestones()
        │
        ├── selectReaction()
        │
        └── startReaction(reaction)
              ├── [squash 140ms]
              ├── applyExpression()
              ├── applySoundEffect()
              ├── catBody.classList.add(reaction.cssClass)
              ├── spawnReactionParticles()
              ├── showReactionMessage()
              └── [animationend → cleanupReaction()]
```

### Interrupt Path

When `handleCatInteraction()` is called while `_activeReaction !== null`:

```
handleCatInteraction()
    └── cancelActiveReaction()   ← removes classes + particles within 16ms
    └── [proceeds with new reaction as above]
```

### Reset Confirmation Flow

```
User clicks #reset-button (first time)
    └── displayConfirmPrompt()
         ├── show #reset-confirm (display:block)
         ├── focus #reset-confirm-btn (Req 19.3)
         └── start 3000ms auto-dismiss timer

User clicks #reset-confirm-btn (confirm)
    └── resetSession()
         ├── Session_State.petCount = 0
         ├── cancelActiveReaction()
         ├── updateCounterDisplay()
         └── milestones unchanged (Req 14.6)
    └── hideConfirmPrompt()
         ├── display:none on #reset-confirm
         └── focus #reset-button (Req 19.4)

User clicks #reset-cancel-btn OR clicks anywhere else
    └── hideConfirmPrompt()     ← no petCount change
         └── focus #reset-button

Timer fires (3000ms, no action taken)
    └── hideConfirmPrompt()     (Req 17.6)
```

### Milestone Check Timing

`checkMilestones()` is called synchronously inside `incrementPetCount()`, immediately after `Session_State.petCount` is updated and `updateCounterDisplay()` is queued. This ensures the milestone fires on the exact tick of the threshold being crossed, before the next interaction can be processed.

---

## Background Decorations

### Generation Strategy

Background decorations are static HTML elements defined in `#bg-decorations`. No JavaScript generation is needed — the fixed count (3 stars, 2 clouds, 3 sparkles, 2 glowing circles) is sufficient and renders before any JS executes (Req 1.2).

```html
<div id="bg-decorations" aria-hidden="true">
  <!-- Stars -->
  <div class="deco deco--star" style="top:8%;  left:5%;  animation-delay:0s;   animation-duration:6s"></div>
  <div class="deco deco--star" style="top:72%; left:88%; animation-delay:2s;   animation-duration:5s"></div>
  <div class="deco deco--star" style="top:45%; left:92%; animation-delay:4s;   animation-duration:7s"></div>
  <!-- Clouds -->
  <div class="deco deco--cloud" style="top:15%; left:80%; animation-delay:1s;  animation-duration:9s"></div>
  <div class="deco deco--cloud" style="top:80%; left:3%;  animation-delay:3s;  animation-duration:8s"></div>
  <!-- Sparkles -->
  <div class="deco deco--sparkle" style="top:5%;  left:45%; animation-delay:0.5s; animation-duration:5s"></div>
  <div class="deco deco--sparkle" style="top:88%; left:55%; animation-delay:2.5s; animation-duration:6s"></div>
  <div class="deco deco--sparkle" style="top:55%; left:2%;  animation-delay:1.5s; animation-duration:7s"></div>
  <!-- Glowing circles -->
  <div class="deco deco--glow" style="top:30%; left:95%; animation-delay:0s;   animation-duration:10s"></div>
  <div class="deco deco--glow" style="top:65%; left:1%;  animation-delay:5s;   animation-duration:8s"></div>
</div>
```

### Positioning Strategy

All decorations are placed outside the center 60% of the viewport (where the cat panel resides), using `position: fixed` with percentage-based `top`/`left`. Their animations use `translateY` only — no horizontal drift — so they cannot drift into the center panel zone (Req 1.2).

```css
.deco {
  position: fixed;
  pointer-events: none;
  user-select: none;     /* Req 1.7 */
  z-index: 1;
  animation: float linear infinite;
}
/* Cat panel z-index: 10 — always above decorations */
```

---

## Accessibility Design

### Focus Management for Reset Confirmation

```javascript
function displayConfirmPrompt() {
  const prompt = document.getElementById('reset-confirm');
  prompt.style.display = 'block';
  // Move focus to confirmation button within same tick (Req 19.3)
  requestAnimationFrame(() => {
    document.getElementById('reset-confirm-btn').focus();
  });
  // Auto-dismiss timer
  _confirmTimer = setTimeout(hideConfirmPrompt, 3000);
}

function hideConfirmPrompt() {
  clearTimeout(_confirmTimer);
  const prompt = document.getElementById('reset-confirm');
  prompt.style.display = 'none';
  // Return focus to reset button (Req 19.4)
  document.getElementById('reset-button').focus();
}
```

### Milestone Overlay Focus

Milestone overlays use `aria-modal="false"` and do not intercept Tab navigation (Req 19.5). An `Escape` keydown listener is attached globally when an overlay appears and removed when it is dismissed. Focus is never moved to the overlay automatically.

### ARIA Live Regions

```html
<!-- Counter updates announced by screen readers (Req 3.2) -->
<div id="counter-display" aria-live="polite" aria-atomic="true">
  🐾 Total Pets: <span id="pet-count">0</span>
</div>

<!-- Reaction messages announced (Req 13.3) -->
<div id="reaction-message" aria-live="polite" aria-atomic="true" role="status"></div>
```

`aria-atomic="true"` ensures the full string is read rather than the changed text node alone.

### Keyboard Handler for Cat

```javascript
// Attached exactly once at init (Req 20.4)
document.getElementById('cat').addEventListener('keydown', (e) => {
  if (e.key === 'Enter' || e.key === ' ') {
    e.preventDefault();  // prevent page scroll on Space
    handleCatInteraction();
  }
});
```

---

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system — essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Pet count increment is exactly 1

*For any* valid Session_State and any single valid interaction (click, tap, Enter, or Space), `Session_State.petCount` after the interaction shall equal `Session_State.petCount` before plus exactly 1, regardless of how rapidly interactions arrive.

**Validates: Requirements 3.1, 3.4**

---

### Property 2: Counter display matches state

*For any* non-negative integer `n` stored in `Session_State.petCount`, the text content of `#counter-display` shall equal the string `"🐾 Total Pets: " + n`.

**Validates: Requirements 3.2, 21.2**

---

### Property 3: Keyboard and pointer interactions are equivalent

*For any* Session_State, a keyboard activation (Enter or Space on focused cat) shall produce exactly the same state delta as a pointer click: `petCount` increments by 1, a reaction is selected, expression classes are applied, and particles are spawned identically.

**Validates: Requirements 3.6**

---

### Property 4: Reaction selection stays within defined set

*For any* call to `selectReaction()`, the returned reaction object shall be one of the eight entries in `REACTION_REGISTRY` (Happy, Purring, Blushing, Surprised, Sleepy, Zoomies, Playful, Grumpy), and shall never be `null` or `undefined`.

**Validates: Requirements 4.1**

---

### Property 5: Consecutive reaction messages are non-identical

*For any* sequence of two or more reactions where each reaction's message pool contains more than one string, no two consecutively displayed `#reaction-message` strings shall be identical.

**Validates: Requirements 4.3, 13.2**

---

### Property 6: Zero particles remain after reaction ends

*For any* reaction that completes naturally (via `animationend`) or is cancelled (via `cancelActiveReaction()`), the `#particle-container` element shall contain exactly 0 child elements after cleanup, and no detached DOM nodes with `animationend` listeners shall exist.

**Validates: Requirements 4.6, 15.4, 18.6**

---

### Property 7: Reaction cancellation clears all state

*For any* in-progress reaction, calling `cancelActiveReaction()` shall: remove all reaction CSS classes from `#cat-body` and `#cat-face`, remove all particles belonging to that reaction from the DOM, and set `_activeReaction` to `null` — all within 16 ms of invocation.

**Validates: Requirements 3.7, 4.5**

---

### Property 8: Cat returns to clean idle state after every reaction

*For any* reaction that ends naturally via `animationend`, `#cat-body` shall have no reaction CSS classes, no inline `animation` overrides, and no inline `transform` overrides; the `float` idle animation shall be active via the class-based CSS rule.

**Validates: Requirements 4.4, 15.5**

---

### Property 9: Milestone flags are set exactly once and are reset-proof

*For any* milestone threshold T ∈ {10, 25, 50, 100}, `Session_State.milestones[T]` shall transition from `false` to `true` exactly once per session (when `petCount` first equals T), and shall remain `true` after any subsequent call to `resetSession()` or any number of additional `incrementPetCount()` calls.

**Validates: Requirements 14.5, 14.6, 17.7**

---

### Property 10: Milestone 10 spawns the required particle count

*For any* session where `petCount` reaches 10 for the first time, `triggerMilestone10()` shall spawn a minimum of 8 heart particle elements originating from the cat center bounding box, each translating outward at least 80 px.

**Validates: Requirements 14.1**

---

### Property 11: Milestone 50 spawns diverse particles

*For any* session where `petCount` reaches 50 for the first time, `triggerMilestone50()` shall spawn a minimum of 20 particle elements comprising at least 3 distinct particle types.

**Validates: Requirements 14.3**

---

### Property 12: Live particle count never exceeds 20

*For any* sequence of particle spawn operations, the number of child elements in `#particle-container` at any point in time shall never exceed 20; spawning a 21st particle shall first remove the oldest existing particle.

**Validates: Requirements 20.1**

---

### Property 13: Session_State round-trip integrity

*For any* valid `Session_State` object written to the runtime variable, immediately reading back `Session_State` shall produce a value deeply equal to the written value: `read(write(state)) === state` for all valid `petCount`, `soundEnabled`, and `milestones` combinations.

**Validates: Requirements 21.2**

---

### Property 14: Malformed petCount defaults to 0

*For any* value of `petCount` that is not a non-negative integer (including `undefined`, `null`, `NaN`, negative numbers, floats, and strings), `validateAndNormalizeState()` shall substitute `0` without throwing an exception, and the displayed counter shall show `"🐾 Total Pets: 0"`.

**Validates: Requirements 21.3, 21.4**

---

### Property 15: Sound toggle icon reflects state for all boolean values

*For any* value of `Session_State.soundEnabled`, the text content of `#sound-toggle` shall be `"🔊"` when `soundEnabled` is `true` and `"🔇"` when `soundEnabled` is `false`, updated within one event-loop tick of the toggle.

**Validates: Requirements 17.1**

---

### Property 16: Focus is always returned after reset prompt dismissal

*For any* dismissal of the reset confirmation prompt — whether by confirming, cancelling, clicking elsewhere, or auto-timeout — `document.activeElement` shall equal `document.getElementById('reset-button')` within one event-loop tick of dismissal.

**Validates: Requirements 19.4**

---

## Error Handling

### Malformed State

`validateAndNormalizeState()` is called during `initState()`. It substitutes typed defaults for any missing or invalid field without throwing (Req 21.3, 21.4). If `petCount` is not a non-negative integer, it becomes `0`. If `soundEnabled` is not a boolean, it becomes `false`. All milestone flags default to `false`.

### AudioContext Errors

The `ensureAudioContext()` function wraps `AudioContext` construction in a try-catch. If the Web Audio API is unavailable (rare, but possible in some environments), `_audioCtx` remains `null` and all `playTone` / sound functions return early after the `if (!Session_State.soundEnabled)` guard. No unhandled exceptions are thrown (Req 16.10).

### animationend Not Firing

All reaction cleanup is triggered by `animationend`. To guard against edge cases where `animationend` never fires (e.g., element removed before animation completes, reduced-motion override), the cancellation path in `cancelActiveReaction()` performs cleanup unconditionally on the next interaction. This ensures orphaned particles cannot accumulate indefinitely.

### Particle Cap Violation

If a bug causes particles to pile up faster than `animationend` fires, the `while (container.children.length >= 20)` guard in `spawnParticle()` removes oldest particles synchronously before each new spawn, bounding the DOM size at 20 regardless of other failures (Req 20.1).

---

## Testing Strategy

This feature is a single `index.html` application with no build pipeline. Testing is conducted using a test harness loaded alongside the app in a browser environment (e.g., Playwright or jsdom-based tests).

**Unit tests** cover specific examples, edge cases, and error conditions:
- Each of the 8 reaction registry entries has all required fields
- `validateAndNormalizeState()` handles specific malformed inputs
- The combined message pool contains all 15 required strings (Req 13.4)
- `selectReaction()` returns a valid registry entry for a fixed random seed
- Reset confirmation flow: confirm path, cancel path, auto-dismiss path
- Sound toggle state changes and icon updates
- Focus management: prompt open moves focus, prompt close returns focus

**Property-based tests** using [fast-check](https://github.com/dubzzz/fast-check) verify universal invariants. Each test runs a minimum of 100 iterations. Tests are tagged with feature and property references.

| Test tag | Property |
|---|---|
| `Feature: cat-petting-clicker, Property 1: pet count increment` | Property 1 |
| `Feature: cat-petting-clicker, Property 2: counter display` | Property 2 |
| `Feature: cat-petting-clicker, Property 3: keyboard/pointer equivalence` | Property 3 |
| `Feature: cat-petting-clicker, Property 4: reaction selection bounds` | Property 4 |
| `Feature: cat-petting-clicker, Property 5: non-repeat messages` | Property 5 |
| `Feature: cat-petting-clicker, Property 6: zero particles after cleanup` | Property 6 |
| `Feature: cat-petting-clicker, Property 7: reaction cancellation` | Property 7 |
| `Feature: cat-petting-clicker, Property 8: idle state restoration` | Property 8 |
| `Feature: cat-petting-clicker, Property 9: milestone flag idempotence` | Property 9 |
| `Feature: cat-petting-clicker, Property 10: milestone 10 particles` | Property 10 |
| `Feature: cat-petting-clicker, Property 11: milestone 50 diversity` | Property 11 |
| `Feature: cat-petting-clicker, Property 12: particle cap` | Property 12 |
| `Feature: cat-petting-clicker, Property 13: state round-trip` | Property 13 |
| `Feature: cat-petting-clicker, Property 14: malformed petCount` | Property 14 |
| `Feature: cat-petting-clicker, Property 15: sound toggle icon` | Property 15 |
| `Feature: cat-petting-clicker, Property 16: focus restoration` | Property 16 |
