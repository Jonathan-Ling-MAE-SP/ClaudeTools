# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Stack & Conventions

This project has two hard constraints that override any default Claude Code
behavior (such as splitting code into multiple files or reaching for a
framework/build tool):

1. **Single-file project.** The entire project must live in one `index.html`
   file, with all CSS and JavaScript inlined via `<style>` and `<script>`
   tags — never in separate `.css`/`.js` files or additional `.html` pages.
   Linking external images, CSS libraries, and JavaScript libraries (e.g.
   via `<link>`/`<script src>` to a CDN) is allowed. This constraint exists
   so the finished project can be copy-pasted as a single file for sharing
   in class and on single-file code platforms (e.g. CodePen, JSFiddle).
2. **Vanilla only, no build step.** Use plain HTML, CSS, and JavaScript
   only — no frameworks or libraries that require a build/bundling step
   (e.g. React, Vue, TypeScript, Sass, webpack/Vite). The file must run
   by simply opening it in a browser, with no compilation step.

## Tech stack (hard constraints — do not deviate)
- Vanilla HTML, CSS, and JavaScript only. No React, Vue, or any JS framework.
- Tailwind CSS for all styling (via CDN only).
- No backend, no database. Fully static site.
- A toggle for light and dark theme, with the choice remembered
  across visits.

## Working conventions
- Before implementing any non-trivial feature, ask clarifying
  questions about scope, edge cases, and constraints first —
  don't propose a plan until you've asked.

## Design Direction (hard constraint)

Standing visual language for the whole portal and every tool/page added
to it — chosen from three explored directions ("Playful Arcade"). Apply
it by default to anything built; don't introduce new colors, fonts, or
spacing values without updating this section first.

**Palette** (light theme; CSS custom properties)
- `--bg`: `oklch(0.98 0.012 300)` — page background, near-white violet tint
- `--surface`: `oklch(0.955 0.018 300)` — sidebar/panel background
- `--border`: `oklch(0.89 0.02 300)`
- `--ink`: `oklch(0.22 0.03 300)` — primary text
- `--muted`: `oklch(0.5 0.02 300)` — secondary text
- `--accent`: `#7C4DFF` (violet) — primary accent: active nav item, primary CTA
- `--accent-2`: `oklch(0.72 0.13 200)` (cyan) — secondary per-tool accent, used
  to visually distinguish one tool from another without adding unrelated hues
- Card surface: `#fff`, `box-shadow: 0 1px 2px oklch(0.2 0.02 300 / 0.06)`
- Dark theme: invert the same relationships (surface darker than bg, ink
  near-white, muted mid-gray); keep the ~300 hue family and both accents —
  don't invent a separate dark palette.

**Typography** (Google Fonts via CDN `<link>`, no self-hosting)
- Display/headings: **Space Grotesk** (weights 500/600/700)
- Body/UI text: **Plus Jakarta Sans** (weights 400/500/600/700)
- Mono accents (stats, timers, CTA labels): **Space Mono** (weights 400/700)
- At most these 3 fonts — don't add a 4th without updating this section.

**Spacing & shape**
- Border radius: 18px cards, 10–12px buttons/icons/nav pills — always rounded.
- Card padding ~22px; sidebar padding ~28px/20px; main content ~56px/64px.
- Grid/stack gaps: 20px between cards, 6–14px within a stack.
- Sidebar: fixed ~272px wide, `border-right: 1px solid var(--border)`.

**Tone**: playful, bold color, rounded shapes — matches the game-like
nature of the tools. New tools reuse this vocabulary (same radii,
spacing scale, font pairing) rather than introducing their own style.

## Feature Plan

Living plan for the portal, tracked by phase so it stays useful as the
collection grows. Mark a phase's checklist items `[x]` as they're built;
once a whole phase is shipped, its detailed notes can be pruned down to a
one-line summary so this section stays skimmable.

### Architecture (applies to every phase)
- Portal name: **SP Hair Monster Hunt** — tagline: "Sharpen your
  reflexes and aim — hunt down the Singapore Poly Hair Monster, one
  click at a time." Welcome copy and tool-card descriptions may use
  hair-monster flavor text (e.g. "Hunt →" instead of "Try it"); tool
  ids/names in the data model below (`reflex`, `aim`) are unaffected.
- Single `index.html`. Tools plug into a `ToolRegistry`
  (`{id, name, icon, mount(container), unmount()}`); the shell (sidebar +
  router) never changes when a tool is added — only a new registered IIFE.
- Sidebar lists tools from the registry; content pane swaps via
  `location.hash` routing (`#<toolId>`; `#home` = welcome panel; unknown
  hash = "not found" panel). Deep links and browser back/forward work via
  native `hashchange`.
- Tailwind loaded via CDN with `darkMode: 'class'`; theme toggle persisted
  to `localStorage['portal.theme']`; a pre-paint script applies the saved
  (or OS-preferred) theme to avoid a flash of the wrong theme.
- Shared localStorage convention: one JSON blob per tool at
  `portal.<toolId>.stats` = `{ best, attempts: [...] }` (history capped to
  the most recent ~20 attempts).
- Both tools blend two input modes — click (aim-trainer style) and type
  (TypeShark style) — chosen randomly per round/target. Shared
  `HAIR_MONSTER_WORDS` list + `randomWord()` at top-level script scope
  supply the type-mode words for both tools.
- Both tools run the same "hunt" session shape: up to 20 monsters, 3
  lives, ends on 20 kills (victory) or 0 lives (defeat). Monsters are a
  shared original inline-SVG illustration (`createMonsterEl()` /
  `buildMonsterSVG()`, procedurally varied fur color + eye/tuft jitter
  per spawn) that grows from far-away to close-up over a fixed
  per-monster time limit (`startApproach()`, frame-by-frame via
  `requestAnimationFrame` so the exact current scale is always known).
  Defeating it in time plays a death animation (`.monster-death`);
  letting the timer run out plays a reached animation (`.monster-reached`)
  and costs a life — both are short fixed-duration CSS `@keyframes`
  driven off a `--ms` custom property set to the monster's exact scale
  at hand-off, so there's no visual jump from the JS-driven approach.
  Shared `renderLives()` draws the heart-icon life indicator.
- Approach speed is adjustable: a shared Easy/Normal/Hard difficulty
  (`getDifficulty()`/`setDifficulty()`, persisted to
  `localStorage['portal.difficulty']`, `DIFFICULTY_MULTIPLIERS = {easy:
  1.5, normal: 1, hard: 0.75}`) scales each tool's own base approach
  time (`REFLEX_BASE_MS`/`AIM_BASE_MS`) at monster-spawn time. Shared
  `renderDifficultyPicker()` draws the picker; each tool hides it while
  a hunt is in progress and shows it again between hunts.
- High scores are resettable: shared `Storage.reset(toolId)` clears one
  tool's `localStorage` entry. Each tool has its own "Reset stats"
  button (behind a native `confirm()`); the welcome page has a "Reset
  all high scores" button that resets every registered tool via
  `ToolRegistry.all()` (so it covers future tools automatically).

### Phase 1a — Portal shell + placeholder tools
Status: `[x]` done

**Built:** `index.html` — sidebar + content-pane shell, `ToolRegistry`,
hash router (`#home`, `#<toolId>`, unknown-hash fallback with graceful
message, active-nav highlighting, deep links, native back/forward),
theme toggle persisted to `localStorage['portal.theme']` with a
pre-paint FOUC guard, Design Direction styling (Tailwind CDN + CSS
custom properties for light/dark). Two placeholder tools registered
(`reflex`, `aim`) with real `mount()`/`unmount()` lifecycle and working
navigation — each just shows a "not live yet" panel; no game logic yet.
Verified end-to-end with headless-browser automation (routing, history,
deep links, theme persistence all pass); Tailwind/Google Fonts loading
itself wasn't visually confirmed in the dev sandbox (network policy
there blocks the CDN hosts) but is standard `<script src>`/`<link>`
usage that will load normally wherever this file is actually opened.

### Phase 1b — Reflex Tester & Aim Tester logic
Status: `[x]` done

**Data model**
- `portal.reflex.stats`: `{ best: msOrNull, attempts: [{ms, falseStart, mode, ts}] }`
  — `mode` is `'click' | 'type'`.
- `portal.aim.stats`: `{ best: {accuracyPct, avgMs, totalMs, ts} | null, attempts: [...] }`
  (schema unchanged by the click/type mix — scoring is mode-agnostic).

**Key flows**
- Reflex Tester: `idle → waiting → ready → result`, with a `falseStart`
  branch off `waiting` (a stray click OR keypress before the cue both
  count). At cue time a mode is picked randomly: `'click'` shows a
  "CLICK NOW" prompt; `'type'` shows a random word from
  `HAIR_MONSTER_WORDS` that must be typed correctly (a document-level
  `keydown` listener, added in `mount()` and removed in `unmount()`).
  Randomized 1.5–4s delay before the cue; `performance.now()` for
  timing; `unmount()` also clears the pending timer.
- Aim Tester: fixed 20-target round; each target is independently
  `'click'` (a circle at a random position) or `'type'` (a word chip) —
  one delegated click handler plus one document `keydown` handler (both
  gated on `roundActive`) cover both. A wrong keystroke resets the typed
  buffer and counts as a miss, same as a stray click on empty space.
  Round end reports accuracy %, avg time/target, total time. Start/replay
  buttons call `stopPropagation()` so the click that starts a round
  doesn't also bubble into the play area's own click handler and log a
  phantom miss against the first target (caught during verification).

**Verified:** headless-browser run through several click- and type-mode
reflex attempts, both false-start paths (click and keypress), a full
20-target aim round mixing both modes to 100% accuracy, `localStorage`
schemas match the above, and mid-wait navigation away from Reflex
Tester doesn't leak its timer/keydown listener.

### Phase 1c — Hair Monster graphics, approach & lives
Status: `[x]` done

Replaced the static circle/word-chip targets in both tools with an
animated hair-monster encounter: the monster spawns far away and grows
larger as it approaches; defeat it (click or type, mode still random)
before a per-monster time limit and it dies; let the timer run out and
it reaches you, costing a life. Both tools now run the same 20-monster,
3-life hunt session (see Architecture above) instead of their previous
separate structures — Reflex Tester in particular changed from
"click to start each independent attempt" to a continuous hunt with a
lurking/false-start phase before each monster.

**Data model** (extends Phase 1b's schemas)
- `portal.reflex.stats` attempts gain a `timedOut` boolean (monster
  reached you; `ms` is `null` like a false start, but does cost a life
  and is a distinct history label — "Too slow" vs "False start").
- `portal.aim.stats` attempts are now
  `{ accuracyPct, avgMs: msOrNull, totalMs, kills, timeouts, misses, result: 'victory'|'defeat', ts }`.
  `accuracyPct` is now `kills / (kills + timeouts) * 100` (stray-click/
  typo misses are tracked but no longer factor into accuracy directly).
  `best` is chosen by most kills, then lowest `avgMs` as a tiebreaker.

**Verified:** headless-browser run through mixed click/type kills in
both tools, a false start in Reflex Tester (auto-resumes lurking after
a brief message, no life lost), deliberately timing out 3 monsters in
both tools to confirm lives drain, the heart indicator's color (not
just count) actually changes per life lost, both defeat-screens render
with correct stats, and navigating away from Reflex Tester mid-lurk
doesn't leak its wait timer into the next tool.

### Phase 1d — Difficulty control, resettable scores, monster redesign
Status: `[x]` done

Follow-up polish requested after trying Phase 1c: the fixed approach
speed felt too fast, there was no way to clear saved high scores, and
the monster art changed from a round shaggy blob to a toupee/wig
creature (googly eyes peeking out from under a hairpiece, with a
couple of swept comb-over strands and stubby legs) per user direction
("a toupee/wig with eyes"). No data-model changes — `buildMonsterSVG()`
is swapped internally, `startApproach()`'s duration argument is now
computed from difficulty rather than a fixed constant, and reset just
clears existing storage keys.

**Verified:** headless-browser check that the new monster SVG renders
with plausible structure across several spawns; that selecting a
difficulty persists to `localStorage` and actually changes the
approach duration; that the difficulty picker hides during a hunt and
reappears once it ends (confirmed correct once an early false read —
caused by the test's own polling overhead slowing the sandboxed
browser's timers — was ruled out with a clean, unpolled wait); and
that per-tool and global reset both clear the right `localStorage`
keys and update the UI immediately.

### Phase 2+ — future tools
Status: not started; not yet scoped.
- Adding a tool is additive: write a self-contained IIFE with
  `mount`/`unmount`, call `ToolRegistry.register(...)`. No shell/router
  changes required.
- Candidate tools: TBD.
