# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A static showcase of 19 self-contained login form templates (pure HTML/CSS/JS — no build step, no framework, no package manager). Each form lives in `forms/<name>/` and is meant to be copy-paste portable into other projects. The root `index.html` is the gallery page that links to every form. Live demos are published via GitHub Pages at `https://puikinsh.github.io/login-forms/forms/<name>/`.

## Running Locally

There is no build, no test runner, and no lint config. To preview:

```bash
# from the repo root
python3 -m http.server 8000
# then open http://localhost:8000/ for the gallery
# or http://localhost:8000/forms/<name>/ for a specific form
```

Opening files via `file://` also works for individual forms — they have no cross-origin dependencies.

## Architecture

### Per-form structure

Every directory under `forms/` follows the same layout:

```
forms/<name>/
├── index.html    # markup
├── style.css     # all visual styling (no shared stylesheet)
├── script.js     # form-specific class extending shared utilities
└── README.md     # per-form notes
```

Forms are intentionally **independent** — there is no shared CSS. Visual styles, color systems, and animations live entirely in each form's `style.css` so a user can copy one folder into another project without dragging anything else along.

### Shared JavaScript layer

The one cross-form dependency is [shared/js/form-utils.js](shared/js/form-utils.js), a static `FormUtils` class with reusable helpers:

- `validateEmail` / `validatePassword` — return `{ isValid, message }`
- `showError` / `clearError` / `showSuccess` — DOM helpers that target `.form-group` and `#<fieldName>Error` by convention
- `simulateLogin(email, password)` — fake 2-second async login (rejects only for `admin@demo.com` / `wrongpassword`)
- `setupFloatingLabels` / `setupPasswordToggle` — wire up common UI patterns
- `addSharedAnimations` — injects `@keyframes shake/slideIn/slideOut/spin/...` once into `<head>`
- `showNotification(message, type, container)` — toast-style messages

Each form's `script.js` defines a class (e.g. `LoginForm1`, `NeonLogin`) instantiated on `DOMContentLoaded`. The class composes `FormUtils` helpers and adds form-specific behavior. The shared **DOM contract** every form follows:

- `<form id="loginForm">` containing `#email`, `#password`, optional `#remember`
- error spans with id `<fieldName>Error` inside `.form-group` wrappers
- inputs wrapped in `.input-wrapper` (used by focus state and success border)
- `#passwordToggle` button with a `.eye-icon` child
- `#successMessage` element toggled with `.show` class on successful login

If you add a new form, mirror this contract so `FormUtils` works without modification.

### Gallery page

[index.html](index.html) is a single self-contained file with inline `<style>` (no external CSS). It hard-codes the list of forms, their categories, and links. When adding/removing/renaming a form, update both this file and [README.md](README.md) (the README's gallery table and category lists).

## Conventions

- **No build tooling.** Don't introduce npm, bundlers, preprocessors, or task runners. Keep everything copy-paste portable.
- **No external runtime dependencies.** No CDN scripts, no Google Fonts loaded from forms (the gallery page is allowed to use system fonts via `font-family` stack only). Forms should work fully offline.
- **Self-contained styling.** Don't extract shared CSS across forms — each form must remain droppable into another codebase as a single folder.
- **`script.js` extends, doesn't replace, `FormUtils`.** Validation and notification logic should call into `FormUtils` rather than reimplementing it.
- **Accessibility expectations** (already in place across forms): `aria-label` on icon buttons, `autocomplete="email"` / `autocomplete="current-password"`, `novalidate` on the form (validation is handled in JS), keyboard-accessible password toggle.

## Adding a New Form

1. Create `forms/<name>/` with `index.html`, `style.css`, `script.js`, `README.md`.
2. Match the DOM contract above so `FormUtils` works out of the box.
3. Reference `../../shared/js/form-utils.js` before `script.js` in `index.html`.
4. Add a screenshot to `assets/screenshots/<name>.png`.
5. Add the form to both the gallery in [index.html](index.html) and the table/category lists in [README.md](README.md).

## Notes

- `CLAUDE.md`, `.DS_Store`, `login-form-screenshots/`, and `colorlib-blog-post.html` are in [.gitignore](.gitignore) — don't commit them.
- The repo's main branch is `main`. There is no CI configured.
