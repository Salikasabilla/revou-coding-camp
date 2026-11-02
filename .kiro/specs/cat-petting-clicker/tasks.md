# Implementation Plan: Kiro — Cute Cat Petting Clicker

## Overview

Build a single `index.html` file at `d:\revou-coding-camp\index.html` containing all HTML, CSS, and JavaScript inline. No build system, no package manager, no external assets. The file delivers a kawaii cat-petting clicker with eight reactions, a particle system, a Web Audio sound engine, milestone celebrations, and full WCAG 2.1 AA accessibility.

## Tasks

---

### Phase 1 — Project Foundation

- [x] 1. Create `index.html` with correct `<head>` structure
  - Create `d:\revou-coding-camp\index.html`
  - Add `<meta charset="UTF-8">` and `<meta name="viewport" content="width=device-width, initial-scale=1">` tags
  - Add `<link rel="preconnect" href="https://fonts.googleapis.com">` and `<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>` tags
  - Add `<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Quicksand:wght@400;600;700&display=swap">` tag
  - Add Tailwind Play CDN `<script src="https://cdn.tailwindcss.com"></script>` tag in `<head>`
  - Add `<style>` block placeholder comment with section markers for custom CSS and `@keyframes`
  - Set `font-family: 'Quicksand', 'Varela Round', sans-serif` as default typeface in the `<style>` block
  - Add `<title>Kiro's Cozy Corner</title>`
  - _Satisfies: Req 1.3, Req 18.1, Req 18.3, Req 18.4, Req 18.8_

- [x] 2. Set up full-viewport gradient background
  - Add CSS `body` rule with `background: linear-gradient(135deg, #FFDEED, #B5FFFC)`, `min-height: 100vh`, `overflow: hidden` on both axes
  - Ensure the gradient covers 100 vw × 100 vh with no gap at viewport edges
  - Define CSS custom properties (color tokens) in `:root`: `--bg-start`, `--bg-end`, `--cat-body`, `--cat-inner-ear`, `--cat-eye`, `--cat-pupil`, `--cat-nose`, `--cat-mouth`, `--cat-whisker`, `--cat-cheek`, `--cat-cushion`, `--panel-text`, `--heading-text`, `--particle-heart`, `--particle-star`, `--particle-paw`, `--particle-sparkle`, `--particle-steam`, `--speed-line`
  - _Satisfies: Req 1.1, Req 1.6_

- [x] 3. Add semantic HTML skeleton
  - Add `<header>` containing `<h1>` and subheading `<p>` inside `<body>`
  - Add `<main>` containing `<section id="cat-panel">` and `<section id="controls-panel">`
  - Add `<div id="particle-container" aria-hidden="true">` inside `<main>` with `position: absolute` CSS
  - Add `<div id="bg-decorations" aria-hidden="true">` as first child of `<body>` (before `<header>`)
  - _Satisfies: Req 18.2_

- [x] 4. Add heading and subheading text with WCAG contrast
  - Set heading text to "Kiro's Cozy Corner" using `color: var(--heading-text)` (`#2D1B1B`) — verify ≥4.5:1 contrast against gradient
  - Set subheading text to "Give your little friend some love!" using `color: var(--panel-text)` (`#3D2C2C`) — verify ≥4.5:1 contrast
  - Apply Tailwind typography classes: `text-3xl sm:text-4xl font-bold` on `<h1>`, `text-base sm:text-lg` on subheading `<p>`
  - _Satisfies: Req 1.4, Req 18.7_

- [x] 5. Add inline `<script>` block with Session_State and module scaffolding
  - Add `<script>` tag at end of `<body>` with clearly commented section headers: `// === CONSTANTS ===`, `// === STATE MODULE ===`, `// === REACTION REGISTRY ===`, `// === REACTION SYSTEM ===`, `// === PARTICLE SYSTEM ===`, `// === SOUND ENGINE ===`, `// === MILESTONE SYSTEM ===`, `// === DOM UPDATER ===`, `// === EVENT HANDLERS ===`, `// === ANIMATION ORCHESTRATION ===`, `// === ACCESSIBILITY ===`, `// === INIT ===`
  - Define `Session_State` object with exactly: `{ petCount: 0, soundEnabled: false, milestones: { 10: false, 25: false, 50: false, 100: false } }`
  - Implement `initState()` that resets `Session_State` to safe defaults
  - Implement `validateAndNormalizeState(raw)` substituting typed defaults for missing/invalid fields
  - Detect reduced motion: `const REDUCED_MOTION = window.matchMedia('(prefers-reduced-motion: reduce)').matches;`
  - _Satisfies: Req 21.1, Req 21.3, Req 21.4, Req 15.6_

---

### Phase 2 — Cat Element

- [x] 6. Build cat body and ears using CSS shapes
  - Add `<div id="cat-wrapper">` with Tailwind classes `aspect-square flex items-center justify-center relative` inside `#cat-panel`
  - Add `<div id="cat-cushion" aria-hidden="true">` inside `#cat-wrapper` for the shadow/cushion
  - Add `<div id="cat" role="button" tabindex="0" aria-label="Pet the cat">` inside `#cat-wrapper`
  - Add `<div id="cat-body">` inside `#cat`
  - Add two ear elements `.cat-ear.cat-ear--left` and `.cat-ear.cat-ear--right` with `.cat-ear-inner` children using `clip-path: polygon(50% 0%, 0% 100%, 100% 100%)` triangle shapes
  - Style `#cat-body` with `border-radius: 50% 50% 45% 45% / 55% 55% 45% 45%`, `background: var(--cat-body)`, responsive width (120 px mobile, 150 px sm, 180 px md+)
  - Style `#cat-cushion` with `border-radius: 50%`, `background: var(--cat-cushion)`, `filter: blur(8px)`, positioned absolutely below cat body
  - _Satisfies: Req 2.1, Req 2.4, Req 2.5_

- [x] 7. Add cat face, eyes, nose, mouth, and whiskers
  - Add `<div id="cat-face">` inside `#cat-body`
  - Add two `.cat-eye` elements (`.cat-eye--left`, `.cat-eye--right`) each containing `.cat-pupil`, `.cat-eye-blink`, `.cat-eye-wide` child divs
  - Add `.cat-cheek.cat-cheek--left` and `.cat-cheek.cat-cheek--right` pink oval overlays with `opacity: 0` default, `border-radius: 50%`, `background: var(--cat-cheek)`
  - Add `<div id="cat-nose">` with `border-radius: 50% 50% 40% 40%`, `background: var(--cat-nose)`
  - Add `<div id="cat-mouth">` containing `.cat-mouth-smile`, `.cat-mouth-frown` (hidden by default), `.cat-mouth-open` (hidden by default)
  - Add `.cat-whiskers.cat-whiskers--left` and `.cat-whiskers.cat-whiskers--right` each with 3 `.cat-whisker` children using `background: var(--cat-whisker)`, angled via `transform: rotate()`
  - Implement CSS expression classes on `#cat-face`: `.face--blink` (show blink overlay, hide pupil), `.face--wide` (scale eyes 1.25×), `.face--closed` (show blink overlay, hide pupil), `.face--blushing` (cheek opacity 0.55)
  - Style smile as bottom half-circle border, frown as top half-circle border, open mouth as filled oval
  - _Satisfies: Req 2.2_

- [-] 8. Implement idle cat animations and interaction properties
  - Define `@keyframes float` in `<style>`: 0%/100% `translateY(0) scale(1.00)`, 50% `translateY(-12px) scale(1.03)` — 3 s cycle
  - Apply `animation: float 3s ease-in-out infinite` to `#cat-body` via `.idle-float` class
  - Implement JS `startBlinkLoop()` function using `setTimeout` with random interval `2000 + Math.random() * 4000` ms; adds `.face--blink` to `#cat-face` for 200 ms then removes it and reschedules
  - Apply `user-select: none` to `#cat` and all `.deco` background elements
  - Apply `touch-action: manipulation` and `-webkit-tap-highlight-color: transparent` to `#cat`
  - _Satisfies: Req 1.7, Req 2.3_

- [-] 9. Add WCAG-AA focus ring for cat keyboard focus
  - Add CSS rule `#cat:focus-visible { outline: 3px solid #5C4033; outline-offset: 4px; border-radius: 50%; }` ensuring ≥3:1 contrast between ring color and adjacent background
  - Verify focus ring is visible at all supported viewport sizes
  - _Satisfies: Req 2.6, Req 18.7_

---

### Phase 3 — Pet Counter and Interaction

- [x] 10. Add counter display and instruction label
  - Add `<div id="counter-display" aria-live="polite" aria-atomic="true">🐾 Total Pets: <span id="pet-count">0</span></div>` inside `#cat-panel`
  - Add `<p id="instruction-label">Tap the kitty to make it happy ♡</p>` inside `#cat-panel`, always visible
  - Implement `updateCounterDisplay()` setting `document.getElementById('pet-count').textContent = Session_State.petCount`
  - _Satisfies: Req 3.2, Req 3.5_

- [x] 11. Implement pet count increment and state management
  - Implement `incrementPetCount()`: increment `Session_State.petCount += 1`, call `scheduleCounterUpdate()`, call `checkMilestones()` — all in the same event-loop tick
  - Implement RAF-batched `scheduleCounterUpdate()` with `_rafPending` flag: only one `requestAnimationFrame` queued at a time, calls `updateCounterDisplay()` then clears flag
  - Implement `validateAndNormalizeState()` called at init to ensure `petCount` is a non-negative integer
  - _Satisfies: Req 3.1, Req 3.4, Req 20.3, Req 21.2, Req 21.4, Req 21.5_

- [-] 12. Wire pointer and keyboard event listeners on cat
  - Attach `pointerdown` listener on `#cat` exactly once at init: calls `handleCatInteraction(event)`, uses `event.pointerId` guard to prevent double-count of same gesture
  - Attach `keydown` listener on `#cat` exactly once at init: fires `handleCatInteraction()` only for `e.key === 'Enter'` or `e.key === ' '`, calls `e.preventDefault()` to suppress page scroll
  - Call `ensureAudioContext()` on first gesture
  - _Satisfies: Req 3.1, Req 3.6, Req 20.4_

- [-] 13. Implement `squash` keyframe and press feedback
  - Define `@keyframes squash` in `<style>`: 0% `scale(1,1)`, 40% `scale(1.20,0.75)`, 70% `scale(0.90,1.10)`, 100% `scale(1,1)` — total duration 130 ms
  - Add CSS class `.anim--squash { animation: squash 130ms ease-out forwards; }`
  - In `handleCatInteraction()`: add `.anim--squash` to `#cat` immediately, remove after 140 ms via `_squashTimeout`
  - _Satisfies: Req 3.3, Req 15.2_

---

### Phase 4 — Reaction System Core

- [~] 14. Define `REACTION_REGISTRY` with all 8 reaction entries
  - Create `REACTION_REGISTRY` array in `// === REACTION REGISTRY ===` section with one object per reaction
  - Each entry must include: `name`, `cssClass`, `expression: { eyes, mouth, cheeks }`, `animation`, `animDuration`, `particles: { types, countMin, countMax }`, `messages[]`, `sound`
  - Entries: Happy (bounce, 800 ms, hearts, 3–6), Purring (wiggle/5px, 1600 ms, hearts, 2–4), Blushing (blush, 900 ms, heart+sparkle, 4–6), Surprised (bounce-surprised, 500 ms, exclamation+sparkle, 4–6), Sleepy (sleepy-tilt, 1100 ms, zzz, 3–5), Zoomies (zoomies, 550 ms, speed-line, 3–5), Playful (bounce-playful, 1200 ms, star+paw, 4–6), Grumpy (wiggle/12px, 800 ms, steam, 2–4)
  - Include per-reaction message pools matching Req 5.3, 6.3, 7.3, 8.3, 9.3, 10.3, 11.3, 12.3 exactly
  - _Satisfies: Req 4.1, Req 4.2, Req 5–12 (all reaction definitions)_

- [~] 15. Implement reaction selection algorithm
  - Define `REACTION_WEIGHTS = [20, 15, 12, 10, 12, 10, 12, 9]` matching registry order
  - Implement `selectReaction()`: sum weights, pick `Math.random() * total`, iterate subtracting weights until `r <= 0`, return that registry entry; always returns a non-null entry (fallback to index 0)
  - _Satisfies: Req 4.1_

- [~] 16. Implement `applyExpression(expression)` and `cancelActiveReaction()`
  - Implement `applyExpression({ eyes, mouth, cheeks })`:
    - Clear all expression classes from `#cat-face`
    - Map `eyes` value to class: `'blink'` → `.face--blink`, `'wide'` → `.face--wide`, `'closed'` → `.face--closed`
    - Show/hide `.cat-mouth-smile`, `.cat-mouth-frown`, `.cat-mouth-open` based on `mouth` value
    - Toggle `.face--blushing` on `#cat-face` based on `cheeks` boolean
  - Implement `cancelActiveReaction()`: clear `_squashTimeout`, remove reaction CSS class and expression classes from `#cat-body` and `#cat-face`, remove inline `animation`/`--wiggle-x`/`--bounce-y` style properties, remove all `_purgingIds` particles from DOM (with listener removal), reset `_purgingIds = []`, stop purring oscillator if active, set `_activeReaction = null`
  - _Satisfies: Req 3.7, Req 4.5, Req 15.5_

- [~] 17. Implement `startReaction()` and `cleanupReaction()` lifecycle
  - Implement `startReaction(reaction)`: set `_activeReaction = reaction`, add `.anim--squash` to `#cat`, set `_squashTimeout` for 140 ms, inside timeout: remove squash, call `applyExpression()`, set CSS custom properties (`--wiggle-x`, `--bounce-y`) on `#cat-body` if needed, add `reaction.cssClass` to `#cat-body`, set inline `animation` style, call `spawnReactionParticles(reaction)`, call `showReactionMessage(reaction)`, attach single `animationend` listener on `#cat-body` that calls `cleanupReaction()` if `_activeReaction === reaction`
  - Implement `cleanupReaction(reaction)`: remove `reaction.cssClass` from `#cat-body`, remove expression classes from `#cat-face`, remove inline animation/transform overrides, call `restoreIdleAnimation()`, set `_activeReaction = null`
  - Implement `restoreIdleAnimation()`: remove inline `animation` property from `#cat-body` so the `.idle-float` class rule reactivates
  - _Satisfies: Req 4.4, Req 15.3, Req 15.5_

- [~] 18. Add reaction message element and fade transitions
  - Add `<div id="reaction-message" aria-live="polite" aria-atomic="true" role="status"></div>` to `#cat-panel`
  - Implement `showReactionMessage(reaction)`: filter pool to exclude `_lastMessage` (if pool.length > 1), pick random candidate, update `_lastMessage`, if `REDUCED_MOTION` set `textContent` directly, else apply `fade-out 160ms` animation then swap text and apply `fade-in 200ms` animation after 160 ms timeout
  - Define `@keyframes fade-in` (from opacity 0 + translateY(4px) to opacity 1) and `@keyframes fade-out` (inverse) with duration ≥150 ms each
  - _Satisfies: Req 4.3, Req 13.1, Req 13.2, Req 13.3, Req 13.4_

---

### Phase 5 — All Keyframes and Reaction CSS

- [x] 19. Define all `@keyframes` in `<style>` block
  - `float`: `0%/100%` `translateY(0) scale(1.00)` → `50%` `translateY(-12px) scale(1.03)`
  - `squash`: `0%` `scale(1,1)` → `40%` `scale(1.20,0.75)` → `70%` `scale(0.90,1.10)` → `100%` `scale(1,1)`
  - `bounce`: uses `--bounce-y` custom property, 0%→30%→55%→75%→100% per design
  - `bounce-surprised`: peaks at `scale(1.25) translateY(-20px)` at 30%
  - `bounce-playful`: translateY(-80px) + rotate(360deg) combined
  - `wiggle`: uses `--wiggle-x` custom property, 0%/100%→20%→50%→80%
  - `zoomies`: translateX oscillation left-right within 600 ms
  - `blush`: rotate 0→15deg→−8deg→10deg→0 within 600–1200 ms
  - `sleepy-tilt`: scale(0.90) rotate(−6deg) over 800–1400 ms
  - `float-up`: uses `--p-dy`, `--p-rot-start`, `--p-rot-end`, `--p-scale` properties
  - `speed-line`: horizontal streak with `translateX` and `scaleX`
  - `fade-in` and `fade-out` (if not already added in task 18)
  - Verify all keyframes use ONLY `transform` and `opacity` properties (no `top`, `left`, `width`, `height`)
  - Apply `animation-fill-mode: forwards` to all one-shot reaction animations
  - _Satisfies: Req 15.1, Req 15.3, Req 20.2_

- [ ] 20. Add per-reaction CSS classes and reduced-motion block
  - Add CSS classes `.reaction--happy`, `.reaction--purring`, `.reaction--blushing`, `.reaction--surprised`, `.reaction--sleepy`, `.reaction--zoomies`, `.reaction--playful`, `.reaction--grumpy` on `#cat-body` defining the animation shorthand for each reaction
  - Add `@media (prefers-reduced-motion: reduce)` block: `#cat, .particle, #bg-decorations *, #cat-body, #reaction-message { animation: none !important; transition: none !important; }`
  - Call `applyReducedMotionDefaults()` at init to add `.reduced-motion` class to `<html>` if needed
  - _Satisfies: Req 15.6, Req 18.9_

---

### Phase 6 — Particle System

- [~] 21. Implement `PARTICLE_RENDERERS` and `spawnParticle()`
  - Define `PARTICLE_RENDERERS` object with inline SVG/text for: `heart` (SVG path), `star` (SVG polygon), `paw` (SVG circles + ellipse), `sparkle` (SVG 4-point star path), `exclamation` (text span), `steam` (SVG circle cluster), `speed-line` (CSS div with linear-gradient), `zzz` (text span)
  - Implement `spawnParticle(type, originX, originY, opts)`:
    - Enforce 20-particle cap: `while (container.children.length >= 20)` remove oldest child (and its listener) first
    - Create `<div class="particle particle--{type}">`, set `position: absolute`, `left`/`top` from originX/originY, set CSS custom properties `--p-dy`, `--p-rot-start`, `--p-rot-end`, `--p-scale`, `--p-color`, `animation` shorthand
    - Inject `PARTICLE_RENDERERS[type]` as innerHTML
    - Attach `animationend` handler that removes listener then removes element from DOM, also splices from `_purgingIds`
    - Store handler as `el._cleanupHandler`, push `el` to `_purgingIds`
    - Return element
  - Define `PARTICLE_COLORS` map for all 8 particle types
  - _Satisfies: Req 20.1, Req 15.4, Req 18.6_

- [~] 22. Implement per-reaction particle spawn helpers
  - Implement `spawnReactionParticles(reaction)`: return early if `REDUCED_MOTION`; read cat bounding box via `getBoundingClientRect()`; compute origin relative to `#particle-container`; pick `randInt(countMin, countMax)` count; cycle through `reaction.particles.types`; for `speed-line` type call `spawnSpeedLine(ox, oy, i)`, otherwise call `spawnParticle()` with appropriate opts
  - Implement `spawnSpeedLine(ox, oy, idx)`: create particle with `speed-line` keyframe, set `--sl-dx` to −80 to −120 px, stagger by idx
  - Apply Sleepy stagger: `delay: i * 200` for zzz particles (Req 9.2)
  - _Satisfies: Req 5.2, Req 6.2, Req 7.2, Req 8.2, Req 9.2, Req 10.1, Req 10.2, Req 11.2, Req 12.2_

---

### Phase 7 — Sound Engine

- [~] 23. Implement lazy AudioContext init and base `playTone()`
  - Implement `ensureAudioContext()`: create `AudioContext` on first call (lazy), catch exceptions if API unavailable (`_audioCtx = null`), call `_audioCtx.resume()` if state is `'suspended'`, return context
  - Implement `playTone(freq, duration, type, gain, startDelay)`: guard on `!Session_State.soundEnabled`, call `ensureAudioContext()`, create oscillator + gain node, wire envelope (attack 10 ms, release at `duration`), call `osc.start()` / `osc.stop()`
  - Call `ensureAudioContext()` on first user gesture (inside `handleCatInteraction`)
  - _Satisfies: Req 16.1, Req 16.5, Req 16.10_

- [~] 24. Implement per-reaction sound functions and dispatcher
  - Implement `playGenericPet()`: two sine tones (180 Hz/100 ms, 260 Hz/80 ms offset 80 ms)
  - Implement `playHappySound()`: three ascending tones C5 (523 Hz/160 ms), E5 (659 Hz/160 ms), G5 (784 Hz/180 ms), staggered at 0/170/340 ms
  - Implement `playPurringSound(durationMs)`: sawtooth oscillator at 80 Hz, gain 0.15, auto-stop at `durationMs`; store as `_purringOscillator` / `_purringGain`
  - Implement `stopPurring()`: try-stop oscillator, disconnect, null refs
  - Implement `playZoomiesSound()`: sawtooth sweep 200→800→200 Hz over 500 ms
  - Implement `playSurprisedSound()`: two staccato square tones (400 Hz/120 ms, 640 Hz/120 ms offset 150 ms), total <500 ms
  - Define `SOUND_FUNCTIONS` map: `{ happy, purring, surprised, zoomies, generic }`
  - Implement `applySoundEffect(reaction)`: guard `!soundEnabled`, look up `SOUND_FUNCTIONS[reaction.sound]` with fallback to `playGenericPet`, call function
  - Call `applySoundEffect()` inside `startReaction()` after expression is applied
  - _Satisfies: Req 16.2, Req 16.3, Req 16.4, Req 16.5, Req 16.6, Req 16.7, Req 16.8, Req 16.9_

---

### Phase 8 — Milestone System

- [~] 25. Implement `checkMilestones()` and milestone flag logic
  - Implement `checkMilestones()`: iterate `[10, 25, 50, 100]`, for each threshold check `Session_State.petCount === threshold && !Session_State.milestones[threshold]`, if true set flag to `true` then call `triggerMilestone(threshold)`
  - Ensure `checkMilestones()` is called synchronously inside `incrementPetCount()` after the state update
  - Verify `resetSession()` does NOT reset any milestone flags
  - _Satisfies: Req 14.5, Req 14.6, Req 17.7_

- [~] 26. Implement milestone 10 and milestone 25 celebrations
  - Implement `triggerMilestone10()`: return early if `REDUCED_MOTION`; read cat bounding box; spawn 8 heart particles radiating in all directions (`angle = (360/8)*i`), each with `dy: -(80 + random*40)`, stagger `delay: i*50 ms`
  - Implement `triggerMilestone25()`: create `<div>` with `textContent = "25 Pets! You're on a roll! 🌟"`, `role="status"`, `aria-live="polite"`, `class="milestone-banner glass-panel"`, append to `<main>`, `setTimeout` remove after 3000 ms
  - _Satisfies: Req 14.1, Req 14.2_

- [~] 27. Implement milestone 50 and milestone 100 celebrations
  - Implement `triggerMilestone50()`: return early if `REDUCED_MOTION`; spawn 20 particles cycling through types `['heart', 'star', 'sparkle', 'paw']` (≥3 distinct types), distributed around cat center with varying radius 30–90 px, staggered `delay: i*40 ms`
  - Implement `triggerMilestone100()`: create overlay `<div>` with `textContent = "🏆 100 Pets! You're the ultimate cat friend!"`, `role="dialog"`, `aria-label="Achievement unlocked"`, `aria-modal="false"`, append to `<main>`, auto-dismiss after 4200 ms
  - Add global `keydown` Escape listener attached when overlay appears, removed on dismiss via `dismissOverlay(el, escHandler)`
  - Ensure overlay does NOT trap focus (`aria-modal="false"`, no `tabindex` manipulation)
  - _Satisfies: Req 14.3, Req 14.4, Req 19.5_

---

### Phase 9 — Controls: Sound Toggle and Reset

- [~] 28. Add Sound Toggle button and wire `toggleSound()`
  - Add `<button id="sound-toggle" aria-label="Toggle sound" aria-pressed="false">🔇</button>` inside `#controls-panel`
  - Implement `toggleSound()`: flip `Session_State.soundEnabled`, call `updateSoundIcon()`
  - Implement `updateSoundIcon()`: set button `textContent` to `'🔊'` or `'🔇'`, update `aria-pressed` attribute — all within one event-loop tick
  - Attach `click` listener to `#sound-toggle` calling `toggleSound()` once at init
  - Initialize as `soundEnabled = false` on page load (button shows 🔇)
  - _Satisfies: Req 16.2, Req 16.3, Req 16.4, Req 17.1_

- [~] 29. Add Reset Button with two-step confirmation flow
  - Add `<button id="reset-button" aria-label="Reset pet counter">Reset</button>` inside `#controls-panel`
  - Add `<div id="reset-confirm" role="dialog" aria-label="Confirm reset" aria-modal="true" style="display:none">` containing `<span>Are you sure?</span>`, `<button id="reset-confirm-btn" aria-label="Confirm reset">Yes</button>`, `<button id="reset-cancel-btn" aria-label="Cancel reset">No</button>`
  - Implement `displayConfirmPrompt()`: show `#reset-confirm`, move focus to `#reset-confirm-btn` via `requestAnimationFrame`, start 3000 ms auto-dismiss `_confirmTimer`
  - Implement `hideConfirmPrompt()`: `clearTimeout(_confirmTimer)`, hide `#reset-confirm`, return focus to `#reset-button`
  - Implement `resetSession()`: set `Session_State.petCount = 0`, call `cancelActiveReaction()`, call `updateCounterDisplay()` — milestone flags unchanged
  - Wire: first `#reset-button` click → `displayConfirmPrompt()`; `#reset-confirm-btn` click → `resetSession()` then `hideConfirmPrompt()`; `#reset-cancel-btn` click → `hideConfirmPrompt()`; click outside prompt while visible → `hideConfirmPrompt()`
  - _Satisfies: Req 17.2, Req 17.3, Req 17.4, Req 17.5, Req 17.6, Req 17.7_

---

### Phase 10 — Background Decorations

- [~] 30. Build background decoration elements
  - Inside `#bg-decorations`, add (using CSS shapes, no `<img>`):
    - 3 `.deco.deco--star` elements: small rotated squares or polygon stars, positioned at `top: 8%/left: 5%`, `top: 72%/left: 88%`, `top: 45%/left: 92%` with `animation-duration: 6s/5s/7s`
    - 2 `.deco.deco--cloud` elements: overlapping oval CSS shapes, positioned at `top: 15%/left: 80%`, `top: 80%/left: 3%` with `animation-duration: 9s/8s`
    - 3 `.deco.deco--sparkle` elements: 4-point star shapes via `clip-path`, positioned at `top: 5%/left: 45%`, `top: 88%/left: 55%`, `top: 55%/left: 2%` with `animation-duration: 5s/6s/7s`
    - 2 `.deco.deco--glow` elements: soft radial-gradient circles, positioned at `top: 30%/left: 95%`, `top: 65%/left: 1%` with `animation-duration: 10s/8s`
  - All decorations use `position: fixed`, `pointer-events: none`, `user-select: none`, `z-index: 1`, `animation: float linear infinite`
  - Each decoration uses `translateY` only in its animation (no horizontal drift) so it cannot intersect the center cat panel zone
  - All animation cycles complete in ≥4 s
  - _Satisfies: Req 1.2, Req 1.7_

---

### Phase 11 — Accessibility and Keyboard Management

- [~] 31. Verify and enforce Tab order and focus management
  - Confirm DOM order places `#cat` before `#sound-toggle` before `#reset-button` to match visual/logical Tab order
  - Implement programmatic focus move in `displayConfirmPrompt()`: `requestAnimationFrame(() => document.getElementById('reset-confirm-btn').focus())` (within one event-loop tick)
  - Implement focus return in `hideConfirmPrompt()`: `document.getElementById('reset-button').focus()` immediately after hiding
  - Ensure no interactive element has `tabindex="-1"` or `display: none` that would remove it from Tab order during normal operation
  - _Satisfies: Req 19.1, Req 19.2, Req 19.3, Req 19.4_

- [~] 32. Verify `aria-live` regions and keyboard equivalence
  - Confirm `#counter-display` has `aria-live="polite" aria-atomic="true"`
  - Confirm `#reaction-message` has `aria-live="polite" aria-atomic="true" role="status"`
  - Confirm Enter/Space on focused `#cat` calls `handleCatInteraction()` identically to pointer, increments counter by 1, applies squash, triggers full reaction sequence
  - Confirm Escape key dismisses milestone overlays without trapping focus
  - Add `aria-label` attributes to every interactive element not already covered
  - _Satisfies: Req 3.6, Req 18.2, Req 19.5_

---

### Phase 12 — Responsive Layout and Final Polish

- [~] 33. Apply mobile-first responsive layout
  - Apply to `<body>`: `display: flex; flex-direction: column; align-items: center; justify-content: center; min-height: 100vh`
  - Apply to `#cat-panel` and `#controls-panel`: glassmorphism classes `bg-white/30 backdrop-blur-lg border border-white/40 rounded-2xl` plus `border-radius: 1.25rem`
  - Responsive cat container: Tailwind `w-32 h-32 sm:w-40 sm:h-40 md:w-48 md:h-48` (or equivalent CSS breakpoints)
  - Responsive cat body: 120 px mobile, 150 px sm (640 px+), 180 px md (1024 px+)
  - Verify no horizontal scrollbar appears at 320 px viewport width
  - Verify no vertical scrollbar at viewport heights 568–1440 px
  - _Satisfies: Req 1.1, Req 1.5, Req 1.6_

- [~] 34. Final quality audit and memory leak verification
  - Audit that all `animationend` listeners are removed before or immediately after removing particle elements from the DOM (zero detached-node listener leaks)
  - Verify all event listeners on `#cat` are attached exactly once at init (not re-attached in reaction cycles)
  - Confirm `#particle-container` child count never exceeds 20 after rapid interaction burst
  - Confirm no inline `.js` files or `.css` files are linked — everything is inside `index.html`
  - Final pass: verify combined message pool contains all 15 required strings from Req 13.4
  - _Satisfies: Req 18.1, Req 18.5, Req 18.6, Req 20.1, Req 20.4_

- [~] 35. Checkpoint — verify complete feature before delivery
  - Open `index.html` in a modern browser (Chrome/Edge/Firefox) and confirm:
    - Page loads with no console errors
    - Cat is visible, floats, and blinks
    - Clicking/tapping cat increments counter and triggers a reaction
    - All 8 reactions produce distinct facial expressions, animations, and particles
    - Milestone celebrations fire at 10, 25, 50, and 100 pets (each only once)
    - Sound toggle enables/disables audio
    - Reset button two-step flow works; counter resets to 0; milestone flags stay
    - Tab navigation reaches all controls; Enter/Space pet the cat
    - No horizontal scrollbar at 320 px width
  - Fix any issues found before completing this checklist item
  - _Satisfies: All requirements_

---

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation
- The design document's Correctness Properties section defines 16 properties; property-based tests are listed as optional sub-tasks in the relevant phases
- All CSS keyframes must use only `transform` and `opacity` to remain on the compositor thread (Req 20.2)

---

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1", "2"] },
    { "id": 1, "tasks": ["3", "4", "5"] },
    { "id": 2, "tasks": ["6", "10"] },
    { "id": 3, "tasks": ["7", "11", "19"] },
    { "id": 4, "tasks": ["8", "9", "12", "13", "20"] },
    { "id": 5, "tasks": ["14", "21"] },
    { "id": 6, "tasks": ["15", "16", "22"] },
    { "id": 7, "tasks": ["17", "18", "23"] },
    { "id": 8, "tasks": ["24", "25"] },
    { "id": 9, "tasks": ["26", "28", "30"] },
    { "id": 10, "tasks": ["27", "29"] },
    { "id": 11, "tasks": ["31", "32"] },
    { "id": 12, "tasks": ["33"] },
    { "id": 13, "tasks": ["34"] },
    { "id": 14, "tasks": ["35"] }
  ]
}
```
