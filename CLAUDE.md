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

## Feature Plan

Living plan for the portal, tracked by phase so it stays useful as the
collection grows. Mark a phase's checklist items `[x]` as they're built;
once a whole phase is shipped, its detailed notes can be pruned down to a
one-line summary so this section stays skimmable.

### Architecture (applies to every phase)
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

### Phase 1 — Portal shell + first two tools
Status: `[ ]` not started

**Tools:** Reflex Tester (reaction-time test), Aim Tester (click-accuracy
trainer).

**Data model**
- `portal.theme`: `'light' | 'dark'`
- `portal.reflex.stats`: `{ best: msOrNull, attempts: [{ms, falseStart, ts}] }`
- `portal.aim.stats`: `{ best: {accuracyPct, avgMs, totalMs, ts} | null, attempts: [...] }`

**Key flows**
- Reflex Tester: `idle → waiting → ready → result`, with a `falseStart`
  branch off `waiting` (click too early). Randomized 1.5–4s delay before
  the cue; `performance.now()` for timing; `unmount()` clears the pending
  timer so navigating away mid-wait can't leak a stale callback.
- Aim Tester: fixed 20-target round; targets positioned by percentage
  inside a bounded play area; one delegated click handler on the play area
  scores hits vs. misses; round end reports accuracy %, avg time/target,
  total time.
- Portal shell: welcome panel is the default/home route; sidebar
  highlights the active tool; unknown hashes get a graceful fallback.

**Deliverable:** `index.html` implementing the shell plus both tools per
the flows above.

### Phase 2+ — future tools
Status: not started; not yet scoped.
- Adding a tool is additive: write a self-contained IIFE with
  `mount`/`unmount`, call `ToolRegistry.register(...)`. No shell/router
  changes required.
- Candidate tools: TBD.
