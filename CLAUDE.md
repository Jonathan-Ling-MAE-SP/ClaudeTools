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
