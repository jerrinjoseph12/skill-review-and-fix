# HTML / CSS / Accessibility Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

Applies to plain HTML and CSS, and to the markup in templates: JSP, Thymeleaf, Razor (`.cshtml`, `.razor`), JSX/TSX, Vue, Angular and Handlebars.

## Authorities
- WHATWG HTML Living Standard
- **WCAG 2.2 Level AA** (the usual legal and contract baseline)
- WAI-ARIA 1.2 and the ARIA Authoring Practices Guide
- OWASP XSS Prevention Cheat Sheet
- Content Security Policy Level 3
- MDN Web Docs

## Tooling to look for and run
- Markup: `html-validate` or the Nu HTML Checker
- Accessibility: axe-core, pa11y or Lighthouse
- CSS: `stylelint`
- JSX: `eslint-plugin-jsx-a11y`
- Angular: `@angular-eslint/template` accessibility rules

## Security
- `[C]` or `[H]` Untrusted data rendered through a raw-HTML escape hatch:
  - `innerHTML`, `outerHTML`, `document.write`
  - `dangerouslySetInnerHTML`, `v-html`, `[innerHTML]`
  - Thymeleaf `th:utext`, Razor `@Html.Raw`, Handlebars `{{{ }}}`, Jinja `|safe`
- `[H]` User data placed in `href` or `src` without blocking `javascript:` URLs; user data inside inline `<script>` blocks or event-handler attributes.
- `[M]` Third-party `<script>` or `<link>` from a CDN without `integrity` (SRI) and `crossorigin`.
- `[M]` Untrusted content in an `<iframe>` without `sandbox`; forms posting to `http://`.
- `[M]` Inline scripts or `on*=` handlers in an app that enforces, or plans to enforce, CSP.
- `[L]` `target="_blank"` without `rel="noopener"`. Modern browsers imply `noopener`, so this is Low.

## Accessibility (WCAG 2.2 AA)
Rate it **High** if it blocks keyboard or screen-reader users from completing a task. Otherwise rate it **Medium**.
- `[H]` `<img>` without `alt`. Use `alt=""` for decorative images. (1.1.1)
- `[H]` Form controls without an associated `<label>`, `aria-label` or `aria-labelledby`. A placeholder isn't a label. (1.3.1, 4.1.2)
- `[H]` Clickable `<div>` or `<span>` without `role`, `tabindex="0"` and Enter/Space key handling. Use `<button>`. (2.1.1)
- `[H]` Focus indicator removed (`outline: none` with no visible replacement) (2.4.7); focused element hidden behind sticky headers (2.4.11, new in 2.2).
- `[H]` `user-scalable=no` or `maximum-scale=1` in the viewport meta tag. This blocks zoom. (1.4.4)
- `[H]` `aria-hidden="true"` on a focusable element or an ancestor of one; keyboard traps in modals with no Esc or close button. (2.1.2)
- `[M]` Text contrast below 4.5:1 (3:1 for large text and for UI components or focus indicators). (1.4.3, 1.4.11)
- `[M]` Missing `<html lang>`; skipped heading levels or several `<h1>` used for visual styling; no landmarks (`<main>`, `<nav>`).
- `[M]` Click targets smaller than 24×24 CSS px without spacing (2.5.8, new in 2.2); drag-only interactions with no alternative (2.5.7).
- `[M]` Information shown by color alone (1.4.1); dynamic updates not announced (no `aria-live` or role `status`/`alert`).
- `[M]` Re-asking for data the user already gave in the same flow (3.3.7); login that relies on a cognitive test or blocks pasting passwords (3.3.8).
- `[M]` Positive `tabindex` values; ARIA roles that conflict with native semantics (`<button role="link">`); redundant ARIA where native HTML would do.

## Markup correctness
- `[H]` `<button>` inside a `<form>` with no `type`. It defaults to `submit` and submits the form unexpectedly.
- `[M]` Duplicate `id` values, which break `label for`, ARIA references and scripts; interactive elements nested inside each other (`<a>` inside `<button>`).
- `[M]` Missing `<!doctype html>`, `<meta charset="utf-8">` or the responsive viewport meta tag; unclosed or wrongly nested elements.
- `[M]` Form inputs without `name`, or with the wrong `type` or `autocomplete` (`email`, `tel`, `current-password`, `one-time-code`).
- `[L]` `<a>` without `href` used as a button; deprecated elements and attributes (`<center>`, `<font>`, `align`).

## Performance
- `[M]` `<img>` or `<video>` without `width` and `height` or `aspect-ratio`, which causes layout shift (CLS); large images below the fold without `loading="lazy"`.
- `[M]` Render-blocking `<script>` in `<head>` without `defer`, `async` or `type="module"`.
- `[L]` Large inline base64 assets; unused CSS shipped site-wide.

## CSS
- `[M]` Fixed `px` font sizes or fixed-height text containers that clip text at 200% zoom or with text-spacing overrides (1.4.4, 1.4.12).
- `[M]` Content that carries meaning placed only in `::before` or `::after`; `display: none` used on content meant for screen readers (use a visually-hidden utility).
- `[M]` Animations that ignore `prefers-reduced-motion`.
- `[L]` `!important` piling up, magic-number `z-index`, hand-written vendor prefixes when Autoprefixer is configured, colors not taken from the project's design tokens or variables.
