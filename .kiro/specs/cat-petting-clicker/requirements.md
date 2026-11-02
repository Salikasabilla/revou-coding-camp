# Requirements Document

## Introduction

"Kiro — Cute Cat Petting Clicker" is a single-page interactive web application where users click or tap an adorable vector-style cat to accumulate pets, trigger expressive animations and reactions, earn milestone celebrations, and optionally enjoy synthesized sounds. The app is delivered as a single `index.html` file using HTML5, Tailwind CSS (CDN Play script), custom CSS, and Vanilla JavaScript — no external frameworks, image assets, or audio files.

## Glossary

- **App**: The single-page cat-petting clicker web application delivered as one `index.html` file.
- **Cat**: The large, interactive, vector-style cat element at the center of the page, built exclusively with CSS, inline SVG, or HTML shapes — no external image assets and no `<img>` tags.
- **Pet_Counter**: The running total of valid user interactions (clicks, taps, or keyboard activations) on the Cat within the current browser session, initialized to 0 on page load.
- **Reaction**: A distinct animated state the Cat enters after each valid interaction, chosen from the eight defined types: Happy, Purring, Blushing, Surprised, Sleepy, Zoomies, Playful, Grumpy.
- **Reaction_Message**: A short text string displayed below the Cat that corresponds to the active Reaction, selected from the per-Reaction message pool without repeating the immediately preceding displayed string.
- **Particle**: A short-lived DOM element (heart, star, paw print, sparkle, exclamation mark, steam cloud, speed line, or "Zzz" text) spawned during a Reaction that animates and is removed from the DOM inside its own `animationend` handler.
- **Milestone**: A Pet_Counter threshold (10, 25, 50, 100) that triggers a one-time-per-session celebration when reached for the first time.
- **Milestone_Flag**: A boolean stored at runtime that records whether a given Milestone has already been triggered in the current session.
- **Sound_Engine**: The Web Audio API subsystem that synthesizes all in-app sounds without external audio files, `<audio>` elements, or Base64-encoded audio data.
- **Sound_Toggle**: The `<button>` that enables or disables the Sound_Engine for subsequent interactions.
- **Reset_Button**: The `<button>` that resets the Pet_Counter and active Reaction state after a two-step confirmation, without resetting Milestone_Flags.
- **Session_State**: The runtime JavaScript object `{ petCount: number, soundEnabled: boolean, milestones: { 10: boolean, 25: boolean, 50: boolean, 100: boolean } }`.
- **Reduced_Motion_Mode**: The App's behaviour when the OS-level `prefers-reduced-motion: reduce` media query is active.

---

## Requirements

### Requirement 1: Visual Theme and Layout

**User Story:** As a user, I want a kawaii, pastel glassmorphism interface with animated background decorations, so that the app feels cozy and delightful from the moment it loads.

#### Acceptance Criteria

1. THE App SHALL render a full-viewport CSS gradient background that transitions from `#FFDEED` to `#B5FFFC`, covering 100 vw × 100 vh with `overflow: hidden` on both axes so no scrollbars appear and no gap exists between the gradient and the viewport edges.
2. THE App SHALL display floating background decoration elements — a minimum of 3 stars, 2 clouds, 3 sparkles, and 2 glowing circles — each animated continuously at a speed that completes one cycle in no less than 4 s, positioned and animated such that none occlude the Cat bounding box or any interactive UI control at any frame of their animation cycle, at any supported viewport size from 320 px to 2560 px wide.
3. THE App SHALL load the Google Font "Quicksand" via a `<link rel="preconnect">` and a `<link rel="stylesheet">` tag in `<head>`, and SHALL list "Varela Round" and a generic `sans-serif` as CSS `font-family` fallbacks, applying this font stack as the default typeface for all visible text.
4. THE App SHALL display the heading "Kiro's Cozy Corner" and the subheading "Give your little friend some love!" such that each achieves a WCAG 2.1 AA contrast ratio of at least 4.5:1 measured against the gradient background color at the element's position.
5. THE App SHALL apply the Tailwind CSS classes `bg-white/30`, `backdrop-blur-lg`, and `border border-white/40` to all UI card panels and control containers, combined with a border-radius of at least `1rem` (`rounded-2xl` or larger).
6. THE App SHALL render without horizontal overflow at any viewport width from 320 px to 2560 px and without vertical overflow at any viewport height from 568 px to 1440 px, verified by absence of scrollbars and absence of clipped content.
7. THE App SHALL apply `user-select: none` to the Cat element and to all background decoration elements, and SHALL apply `touch-action: manipulation` and `-webkit-tap-highlight-color: transparent` to the Cat element to suppress default browser touch behaviors during Cat interactions.

---

### Requirement 2: Cat Element

**User Story:** As a user, I want a large, adorable, expressive cat rendered entirely from code at the center of the page, so that I have a clear and charming interaction target.

#### Acceptance Criteria

1. THE Cat SHALL be built exclusively from inline CSS, inline SVG elements, or HTML shape elements — zero `<img>` tags, zero references to external image files, and zero Base64-encoded image data.
2. THE Cat SHALL include all of the following rendered features using soft colors and rounded shapes: two eyes capable of a blink state and a widened state, two rounded ears, one nose, two whisker groups (minimum 2 whiskers each side), and one mouth capable of at least two distinct expressions (smile and frown).
3. THE Cat SHALL continuously animate in its idle state with: (a) a vertical float cycle using the `float` keyframe that completes one up-and-down cycle in 3 s ± 0.5 s, (b) an eye blink triggered at a random interval between 2 s and 6 s, and (c) a scale-pulse breathing effect using the `float` or a dedicated keyframe that varies scale between 0.97 and 1.03.
4. THE App SHALL apply the Tailwind classes `aspect-square`, `flex`, `items-center`, and `justify-center` to the Cat's container element to maintain a square aspect ratio and centered layout.
5. THE Cat SHALL appear above a soft circular shadow or pastel-colored cushion element rendered as a CSS shape that reinforces the visual floating illusion.
6. WHEN the Cat element receives keyboard focus, THE App SHALL display a focus ring that achieves a minimum 3:1 contrast ratio between the ring color and adjacent background colors, meeting WCAG 2.1 AA for UI components.

---

### Requirement 3: Interaction and Pet Counter

**User Story:** As a user, I want to click or tap the cat and immediately see the pet count increase with satisfying visual feedback, so that every interaction feels instant and rewarding.

#### Acceptance Criteria

1. WHEN a user clicks or taps the Cat, THE Pet_Counter SHALL increment by exactly 1, regardless of how rapidly events are delivered, with no double-counting of a single physical gesture.
2. THE App SHALL display the Pet_Counter in the format "🐾 Total Pets: {n}" where `{n}` is the current integer value of Pet_Counter, initialized to 0 on page load.
3. WHEN a user clicks or taps the Cat, THE Cat SHALL immediately apply the `squash` CSS animation as press-feedback before any Reaction-specific animation begins, with the `squash` animation completing in 150 ms or less.
4. THE App SHALL update Session_State.petCount to equal the new Pet_Counter value within the same event-loop tick as the increment, so the displayed value and stored value are never out of sync for longer than one tick.
5. THE App SHALL display the static label "Tap the kitty to make it happy ♡" at all times; this label SHALL NOT be hidden, removed, or obscured during normal use.
6. WHEN a user presses the Enter key or Space key while the Cat has keyboard focus, THE App SHALL treat the keypress as a valid interaction, increment Pet_Counter by exactly 1, apply the `squash` animation, and trigger the full Reaction sequence identically to a pointer interaction.
7. WHEN a new valid interaction occurs while a Reaction animation is in progress, THE App SHALL remove all active Reaction CSS classes and all orphaned Particle elements from the DOM within 16 ms, apply the `squash` animation, and begin the new Reaction sequence.

---

### Requirement 4: Reaction System

**User Story:** As a user, I want the cat to react differently each time I pet it using one of eight distinct reactions, so that the experience stays varied and entertaining over many interactions.

#### Acceptance Criteria

1. WHEN a valid interaction occurs, THE App SHALL select one of the eight Reaction types (Happy, Purring, Blushing, Surprised, Sleepy, Zoomies, Playful, Grumpy) using a weighted-random or round-robin algorithm defined in the design document.
2. THE App SHALL implement all eight Reaction types, each producing a distinct facial expression, a distinct CSS animation on the Cat, at least one Particle type, and at least one Reaction_Message string.
3. WHEN a Reaction is active, THE App SHALL display the corresponding Reaction_Message below the Cat; the selected string SHALL NOT be identical to the immediately preceding displayed Reaction_Message string. On the very first interaction of a session, when no preceding Reaction_Message exists, THE App SHALL select any message from the pool freely without applying the non-repeat constraint.
4. WHEN a Reaction animation reaches its natural `animationend` event, THE Cat SHALL return to the idle floating state with all Reaction CSS classes and inline animation style overrides removed.
5. IF a new valid interaction occurs while a Reaction animation is in progress, THEN THE App SHALL cancel the active Reaction within 16 ms by removing all Reaction CSS classes and all Particle elements owned by that Reaction from the DOM before starting the next Reaction.
6. THE App SHALL ensure that zero Particle elements remain attached to the DOM after their owning Reaction ends naturally or is cancelled.

---

### Requirement 5: Happy Reaction

**User Story:** As a user, I want the cat to smile, bounce, and spawn floating hearts when I pet it during a Happy reaction, so that I feel the cat is joyful.

#### Acceptance Criteria

1. WHEN the Happy Reaction is triggered, THE Cat SHALL display a smile expression and an eye-blink state, and SHALL apply the `bounce` keyframe with a total animation duration between 600 ms and 1000 ms.
2. WHEN the Happy Reaction is triggered, THE App SHALL spawn between 3 and 6 heart-shaped Particle elements that each translate upward by at least 60 px, rotate between −30° and 30°, scale to between 1.2× and 1.8×, then fade to opacity 0 before being removed from the DOM.
3. WHEN the Happy Reaction is triggered, THE App SHALL display a Reaction_Message selected randomly from exactly the following pool: "Yay! ♡", "Best human ever!", "More cuddles, please!", "I love you!".

---

### Requirement 6: Purring Reaction

**User Story:** As a user, I want the cat to show gently closed eyes and sway slowly with floating hearts when I pet it during a Purring reaction, so that I feel the cat is deeply content.

#### Acceptance Criteria

1. WHEN the Purring Reaction is triggered, THE Cat SHALL display a gently closed-eye expression and SHALL apply the `wiggle` keyframe at a horizontal amplitude no greater than 6 px and a cycle duration between 1.2 s and 2 s.
2. WHEN the Purring Reaction is triggered, THE App SHALL spawn between 2 and 4 heart-shaped Particle elements that translate upward at least 50 px and fade to opacity 0 before being removed from the DOM.
3. WHEN the Purring Reaction is triggered, THE App SHALL display a Reaction_Message selected randomly from exactly the following pool: "Purrrrr... ♡", "This is the life ♡", "Purrr...".

---

### Requirement 7: Blushing Reaction

**User Story:** As a user, I want the cat to show pink cheeks and tilt its head with hearts and sparkles when I pet it during a Blushing reaction, so that the cat appears shy and endearing.

#### Acceptance Criteria

1. WHEN the Blushing Reaction is triggered, THE Cat SHALL display two semi-transparent pink oval cheek overlay elements with opacity between 0.4 and 0.7, and SHALL apply the `blush` keyframe that tilts the Cat between 10° and 20° from vertical with a total duration between 600 ms and 1200 ms.
2. WHEN the Blushing Reaction is triggered, THE App SHALL spawn a minimum of 2 heart Particle elements and a minimum of 2 sparkle Particle elements, each translating upward at least 50 px and fading to opacity 0 before being removed from the DOM.
3. WHEN the Blushing Reaction is triggered, THE App SHALL display a Reaction_Message selected randomly from exactly the following pool: "You're making me blush!", "Stop, I'm blushing!", "Eep! You're too sweet!".

---

### Requirement 8: Surprised Reaction

**User Story:** As a user, I want the cat to show wide eyes and an open mouth with a sharp bounce and exclamation particles when I pet it during a Surprised reaction, so that the cat appears startled in a cute way.

#### Acceptance Criteria

1. WHEN the Surprised Reaction is triggered, THE Cat SHALL display widened eyes and an open "O"-shaped mouth, and SHALL apply the `bounce` keyframe with a scale factor of at least 1.2× at peak and a total duration between 400 ms and 700 ms.
2. WHEN the Surprised Reaction is triggered, THE App SHALL spawn a minimum of 2 exclamation mark ("!") text Particle elements and a minimum of 2 sparkle Particle elements that translate outward at least 40 px from the Cat center and fade to opacity 0 before being removed from the DOM.
3. WHEN the Surprised Reaction is triggered, THE App SHALL display a Reaction_Message selected randomly from exactly the following pool: "Meow?!", "Oh! You got me!", "!".

---

### Requirement 9: Sleepy Reaction

**User Story:** As a user, I want the cat to appear drowsy with closed eyes and staggered floating Zzz bubbles when I pet it during a Sleepy reaction, so that the cat feels cozy and relatable.

#### Acceptance Criteria

1. WHEN the Sleepy Reaction is triggered, THE Cat SHALL apply the `sleepy-tilt` keyframe that scales the Cat to 90% of its base size and tilts it to −6° from vertical, with closed eyes rendered, and a total animation duration between 800 ms and 1400 ms.
2. WHEN the Sleepy Reaction is triggered, THE App SHALL spawn between 3 and 5 "Zzz" text Particle elements with staggered `animation-delay` values between 150 ms and 300 ms apart, each translating upward at least 60 px and fading to opacity 0 before being removed from the DOM.
3. WHEN the Sleepy Reaction is triggered, THE App SHALL display a Reaction_Message selected randomly from exactly the following pool: "Zzz... just five more minutes", "Five more minutes...", "Zzz...".

---

### Requirement 10: Zoomies Reaction

**User Story:** As a user, I want the cat to dash horizontally with speed lines when I pet it during a Zoomies reaction, so that the interaction feels explosive and funny.

#### Acceptance Criteria

1. WHEN the Zoomies Reaction is triggered, THE Cat SHALL apply the `zoomies` keyframe that moves the Cat element horizontally left-to-right (or equivalent rapid oscillation) with a total animation duration no greater than 600 ms, accompanied by a minimum of 3 speed-line sibling DOM elements rendered as horizontal CSS shapes.
2. WHEN the Zoomies Reaction is triggered, THE App SHALL spawn a minimum of 3 horizontal-streak Particle elements with animation durations no greater than 500 ms each, translating horizontally at least 80 px and fading to opacity 0 before being removed from the DOM.
3. WHEN the Zoomies Reaction is triggered, THE App SHALL display a Reaction_Message selected randomly from exactly the following pool: "ZOOMIES!!!", "NYOOOM!", "Can't catch me!".

---

### Requirement 11: Playful Reaction

**User Story:** As a user, I want the cat to jump high with a spin and spawn colorful stars and paw prints when I pet it during a Playful reaction, so that the cat appears bursting with energy.

#### Acceptance Criteria

1. WHEN the Playful Reaction is triggered, THE Cat SHALL apply the `bounce` keyframe at a vertical travel distance at least 2× the base bounce amplitude, combined with a 360° rotation and a `wiggle` sequence, with a total animation duration between 800 ms and 1400 ms.
2. WHEN the Playful Reaction is triggered, THE App SHALL spawn a minimum of 2 star Particle elements and a minimum of 2 paw-print Particle elements using at least 2 distinct fill colors, each translating upward at least 70 px and fading to opacity 0 before being removed from the DOM.
3. WHEN the Playful Reaction is triggered, THE App SHALL display a Reaction_Message selected randomly from exactly the following pool: "Catch me if you can!", "Let's play!", "Catch me!".

---

### Requirement 12: Grumpy Reaction

**User Story:** As a user, I want the cat to look annoyed with a rapid shake and steam effects when I pet it during a Grumpy reaction, so that the experience has humorous variety.

#### Acceptance Criteria

1. WHEN the Grumpy Reaction is triggered, THE Cat SHALL display a grumpy facial expression — furrowed brows and a downturned mouth — and SHALL apply the `wiggle` keyframe at a horizontal amplitude between 8 px and 16 px with a cycle duration no greater than 200 ms, and a total animation duration between 600 ms and 1000 ms.
2. WHEN the Grumpy Reaction is triggered, THE App SHALL spawn a minimum of 2 steam-cloud or angry-symbol Particle elements that translate upward at least 50 px and fade to opacity 0 before being removed from the DOM.
3. WHEN the Grumpy Reaction is triggered, THE App SHALL display a Reaction_Message selected randomly from exactly the following pool: "Hey! Gentle pets, please!", "Hmph!", "Do NOT disturb!".

---

### Requirement 13: Reaction Messages

**User Story:** As a user, I want to see varied, reaction-specific text messages with smooth transitions each time I pet the cat, so that the app remains fresh and engaging over many sessions.

#### Acceptance Criteria

1. THE App SHALL maintain a per-Reaction message pool for all eight Reaction types; the combined pool across all Reactions SHALL contain at least 15 distinct strings.
2. WHEN a Reaction_Message is selected, THE App SHALL verify that the selected string is not identical to the immediately preceding displayed Reaction_Message; IF the initial random selection is identical, THE App SHALL re-select from the remaining pool entries until a non-identical string is found.
3. THE App SHALL display each incoming Reaction_Message using a `fade-in` CSS animation with a duration of at least 150 ms, and SHALL remove the outgoing message using a `fade-out` CSS animation with a duration of at least 150 ms before the replacement string is inserted into the DOM.
4. THE combined message pool SHALL include all of the following strings: "Yay! ♡", "Best human ever!", "More cuddles, please!", "Purrrrr... ♡", "This is the life ♡", "Eep! You're too sweet!", "Stop, I'm blushing!", "Meow?!", "Oh! You got me!", "Five more minutes...", "Can't catch me!", "NYOOOM!", "Catch me!", "Let's play!", "Hmph!".

---

### Requirement 14: Milestone Celebrations

**User Story:** As a user, I want special one-time celebrations when I reach certain pet-count thresholds, so that I feel rewarded for sustained interaction.

#### Acceptance Criteria

1. WHEN Pet_Counter reaches 10 for the first time in a session, THE App SHALL spawn a minimum of 8 heart Particle elements originating from the center of the Cat bounding box, each translating outward at least 80 px in varying directions before fading to opacity 0.
2. WHEN Pet_Counter reaches 25 for the first time in a session, THE App SHALL display a banner element containing exactly the text "25 Pets! You're on a roll! 🌟" that auto-dismisses by removing itself from the DOM after exactly 3000 ms ± 200 ms.
3. WHEN Pet_Counter reaches 50 for the first time in a session, THE App SHALL spawn a minimum of 20 Particle elements of at least 3 distinct types, distributed across a large-scale or full-screen area, each animating and then being removed from the DOM.
4. WHEN Pet_Counter reaches 100 for the first time in a session, THE App SHALL display an achievement overlay element containing exactly the text "🏆 100 Pets! You're the ultimate cat friend!" that remains visible in the DOM for a minimum of 4000 ms before auto-dismissing.
5. THE App SHALL set the corresponding Milestone_Flag to `true` upon triggering each Milestone celebration, and SHALL NOT trigger the same Milestone celebration a second time during the same browser session regardless of subsequent Pet_Counter values.
6. WHEN Pet_Counter is reset to 0 via the Reset_Button, THE App SHALL NOT modify any Milestone_Flag; all previously triggered Milestone_Flags SHALL remain `true` for the remainder of the session.

---

### Requirement 15: CSS Animations

**User Story:** As a user, I want smooth, polished animations throughout the app, so that every interaction and background element feels satisfying.

#### Acceptance Criteria

1. THE App SHALL define CSS `@keyframes` blocks in `<style>` for all of the following named animations: `float`, `bounce`, `wiggle`, `zoomies`, `blush`, `fade-in`, `fade-out`, `float-up`, `sleepy-tilt`, `squash`.
2. WHEN a valid Cat interaction occurs, THE App SHALL apply the `squash` keyframe to the Cat element before any Reaction-specific animation class is applied.
3. THE App SHALL apply `animation-fill-mode: forwards` to all one-shot Reaction animations on the Cat element so the Cat does not visually revert to its pre-animation state between `animationend` and class removal.
4. THE App SHALL attach a single `animationend` event listener to each Particle element at spawn time and SHALL remove the Particle from the DOM inside that handler, ensuring no Particle persists beyond its animation duration.
5. WHEN a Reaction animation class is removed from the Cat, THE App SHALL also remove any inline `animation` or `transform` style overrides added during that Reaction cycle, restoring the Cat's style state for the next idle cycle.
6. WHEN the `prefers-reduced-motion: reduce` media query is active (Reduced_Motion_Mode), THE App SHALL replace all non-essential animations — idle `float`, Particle animations, and all Reaction-specific keyframes — with instantaneous state changes or static visual feedback, while preserving all functional state changes (counter increments, Reaction_Message updates, Milestone_Flag updates).

---

### Requirement 16: Sound System

**User Story:** As a user, I want optional synthesized sounds that match each cat reaction, so that the experience is more immersive when I choose to enable it.

#### Acceptance Criteria

1. THE Sound_Engine SHALL synthesize all sounds exclusively using the Web Audio API — zero `<audio>` elements, zero external audio file references, and zero Base64-encoded audio data.
2. THE App SHALL initialize with sound disabled; Session_State.soundEnabled SHALL be `false` on page load and the Sound_Toggle SHALL reflect the disabled state on first render.
3. WHEN the user activates the Sound_Toggle while sound is disabled, THE App SHALL set Session_State.soundEnabled to `true` and produce sound on all subsequent valid interactions.
4. WHEN the user activates the Sound_Toggle while sound is enabled, THE App SHALL set Session_State.soundEnabled to `false`, cease any currently playing Sound_Engine output within 50 ms, and produce no sound on subsequent interactions.
5. WHEN a valid interaction occurs and Session_State.soundEnabled is `true`, THE Sound_Engine SHALL play a synthesized pop or meow-like tone using an oscillator node with a duration between 80 ms and 300 ms.
6. WHEN the Happy Reaction is triggered and Session_State.soundEnabled is `true`, THE Sound_Engine SHALL play an ascending tone sequence of at least 3 distinct notes, each note no longer than 200 ms.
7. WHEN the Purring Reaction is triggered and Session_State.soundEnabled is `true`, THE Sound_Engine SHALL play a continuous oscillation with a fundamental frequency between 60 Hz and 120 Hz for the full duration of the Purring Reaction animation.
8. WHEN the Zoomies Reaction is triggered and Session_State.soundEnabled is `true`, THE Sound_Engine SHALL play a pitch sweep (ascending then descending) with a total duration between 300 ms and 700 ms.
9. WHEN the Surprised Reaction is triggered and Session_State.soundEnabled is `true`, THE Sound_Engine SHALL play a staccato ascending sequence of at least 2 notes with a total duration under 500 ms.
10. THE App SHALL create the `AudioContext` on the first user gesture (pointer or keyboard event on any interactive element), and WHEN the `AudioContext.state` is `"suspended"`, THE App SHALL call `AudioContext.resume()` before attempting to play any sound, without throwing an unhandled exception.

---

### Requirement 17: Sound Toggle and Reset Controls

**User Story:** As a user, I want clearly labeled controls to toggle sound and safely reset my progress, so that I can manage my session without accidentally losing my pet count.

#### Acceptance Criteria

1. THE App SHALL render a Sound_Toggle `<button>` element with the attribute `aria-label="Toggle sound"` that displays a 🔊 icon when Session_State.soundEnabled is `true` and a 🔇 icon when Session_State.soundEnabled is `false`, updating the displayed icon within one event-loop tick of each toggle.
2. THE App SHALL render a Reset_Button `<button>` element with the attribute `aria-label="Reset pet counter"`.
3. WHEN the user activates the Reset_Button and no confirmation prompt is currently displayed, THE App SHALL display a confirmation prompt containing the text "Are you sure?" without modifying Pet_Counter or any Milestone_Flag.
4. WHEN the user activates the Reset_Button a second time while the confirmation prompt is visible, THE App SHALL set Pet_Counter to 0, remove all active Reaction CSS classes and all Particle elements from the DOM, update the displayed counter to "🐾 Total Pets: 0", and hide the confirmation prompt.
5. WHEN the user activates any element other than the Reset_Button while the confirmation prompt is visible, THE App SHALL dismiss the confirmation prompt without modifying Pet_Counter.
6. THE App SHALL automatically dismiss the confirmation prompt and restore the Reset_Button to its initial label after 3000 ms ± 200 ms if neither the confirm nor cancel action has been taken.
7. WHEN Pet_Counter is reset to 0, THE App SHALL leave all Milestone_Flags unchanged at their current values.

---

### Requirement 18: Technical and Accessibility Standards

**User Story:** As a developer and user, I want the app to be a single, self-contained, accessible HTML file with no memory leaks or external JavaScript dependencies, so that it is trivial to distribute and runs reliably on any modern browser.

#### Acceptance Criteria

1. THE App SHALL be delivered as a single `index.html` file; all HTML markup, all CSS, and all JavaScript SHALL be inline within that file — zero linked `.css` files and zero linked `.js` files are permitted.
2. THE App SHALL use semantic HTML5 elements including `<main>`, `<header>`, `<section>`, and `<button>`, and SHALL provide a descriptive `aria-label` attribute on every interactive element.
3. THE App SHALL load Tailwind CSS exclusively via the official Tailwind Play CDN `<script>` tag placed in `<head>` — zero other external JavaScript frameworks or CSS libraries are permitted.
4. THE App SHALL load the chosen Google Font via a `<link rel="preconnect">` and a `<link rel="stylesheet">` pair in `<head>` — zero other external font services are permitted.
5. THE App SHALL organize all JavaScript into named, single-responsibility functions, each accompanied by an inline comment of at least one sentence describing its purpose.
6. THE App SHALL remove all `animationend` event listeners from Particle DOM elements before or immediately after removing those elements from the DOM, ensuring zero detached-node listener leaks.
7. THE App SHALL meet WCAG 2.1 AA color-contrast requirements: a minimum 4.5:1 contrast ratio for normal text and a minimum 3:1 contrast ratio for large text and UI component boundaries against their immediate backgrounds.
8. THE App SHALL include `<meta name="viewport" content="width=device-width, initial-scale=1">` in `<head>`.
9. WHEN the `prefers-reduced-motion: reduce` media query is active, THE App SHALL disable all non-essential CSS animations, satisfying WCAG 2.1 Success Criterion 2.3.3.

---

### Requirement 19: Keyboard and Focus Management

**User Story:** As a keyboard-only user, I want full access to every interactive control using only the Tab and Enter keys, so that the app is completely usable without a pointer device.

#### Acceptance Criteria

1. THE App SHALL ensure all interactive elements — the Cat, the Sound_Toggle, and the Reset_Button — are reachable via sequential Tab key navigation in a logical DOM order matching their visual order.
2. THE App SHALL ensure that no interactive element that is functionally available has `tabindex="-1"` or is hidden via `display: none` or `visibility: hidden` in a way that removes it from the Tab focus order.
3. WHEN the confirmation prompt for the Reset_Button is displayed, THE App SHALL programmatically move keyboard focus to the confirmation prompt element or its first focusable child within one event-loop tick.
4. WHEN the confirmation prompt is dismissed (whether confirmed or cancelled), THE App SHALL programmatically return keyboard focus to the Reset_Button within one event-loop tick.
5. THE App SHALL ensure that Milestone celebration overlays do not trap keyboard focus; users SHALL be able to Tab past them and SHALL be able to dismiss them by pressing the Escape key.

---

### Requirement 20: Performance Constraints

**User Story:** As a user on a mid-range mobile device, I want the app to remain smooth and responsive during rapid repeated interactions, so that the experience is not degraded by lag or dropped frames.

#### Acceptance Criteria

1. THE App SHALL limit the maximum number of simultaneously live Particle DOM elements to 20 at any given moment; IF spawning a new Particle would cause the live count to exceed 20, THE App SHALL remove the oldest existing Particle before inserting the new one.
2. THE App SHALL implement all Particle and Cat animation keyframes using only the `transform` and `opacity` CSS properties to ensure compositor-thread execution, with zero usage of `top`, `left`, `width`, or `height` inside any animation keyframe block.
3. WHEN more than 10 valid Cat interactions occur within any 100 ms window, THE App SHALL batch Pet_Counter DOM text-node updates using `requestAnimationFrame`, producing no more than 1 DOM text update per animation frame during the burst.
4. THE App SHALL attach all repetitive event listeners (pointer events and keyboard events on the Cat) exactly once during initialization and SHALL NOT re-attach any of these listeners during subsequent interaction cycles.

---

### Requirement 21: Session State

**User Story:** As a developer, I want session state to be represented as a well-defined runtime object with predictable defaults and consistent update semantics, so that the app's internal state is always synchronized with its displayed values.

#### Acceptance Criteria

1. THE App SHALL represent Session_State as a plain JavaScript object with exactly the following structure: `{ petCount: number, soundEnabled: boolean, milestones: { 10: boolean, 25: boolean, 50: boolean, 100: boolean } }` — no additional top-level fields are permitted.
2. WHEN Session_State is read back from the runtime object immediately after being written, THE App SHALL produce values identical to those that were written, satisfying the round-trip property: `read(write(state)) === state` for all valid state values.
3. IF the runtime Session_State object is missing one or more expected fields at initialization, THEN THE App SHALL substitute default values — `petCount = 0`, `soundEnabled = false`, all Milestone_Flags = `false` — without throwing an uncaught exception.
4. IF the runtime Session_State object contains a `petCount` value that is not a non-negative integer, THEN THE App SHALL substitute `petCount = 0` before rendering the Pet_Counter display.
5. THE App SHALL update Session_State.petCount atomically after each Pet_Counter increment such that the in-memory Session_State value and the displayed Pet_Counter value are never out of sync for longer than one event-loop tick; IF a Session_State update is delayed or fails, THE App SHALL force Session_State.petCount to match the currently displayed Pet_Counter value rather than reverting the displayed value to the stale state.
