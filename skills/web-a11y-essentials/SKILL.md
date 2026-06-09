---
name: web-a11y-essentials
description: Web accessibility best practices for semantic HTML, ARIA attributes, and WCAG color contrast. Use this skill whenever the user writes, reviews, or fixes HTML/JSX/TSX/Svelte markup or UI components, builds page structure, navigation, forms, buttons, modals, or tabs, picks text/background colors or theme tokens, or mentions accessibility, a11y, semantic markup, screen readers, ARIA, roles, labels, alt text, focus order, contrast ratio, or WCAG. Apply it proactively when generating or styling markup even if accessibility is not explicitly requested. Do not use it for non-web accessibility.
---

# Web A11y Essentials

Best practices for building accessible web interfaces, grounded in three sources:

- Semantic HTML — [MDN: Semantics](https://developer.mozilla.org/en-US/docs/Glossary/Semantics)
- ARIA states & properties — [MDN: ARIA Attributes](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes)
- Color contrast — [WCAG 2.2 SC 1.4.3 Contrast (Minimum)](https://www.w3.org/TR/WCAG22/#contrast-minimum)

## Quick Start

Work in this order — it prevents most accessibility defects before they appear:

1. **Choose the markup that describes the data**, not the markup that looks right. Reach for a semantic element first. See [references/semantic-html.md](references/semantic-html.md).
2. **Add ARIA only to fill gaps native HTML cannot.** If you typed `role=` or `aria-*`, pause and check whether a native element already does it. See [references/aria-attributes.md](references/aria-attributes.md).
3. **Verify color contrast** for any text or meaningful UI you style. See [references/wcag-contrast.md](references/wcag-contrast.md).
4. **Run the pre-ship checklist** at the bottom before considering the work done.

## The Single Most Important Rule

> Prefer native HTML. ARIA does not add behavior — it only changes how assistive technology _describes_ an element. A `<div role="button">` looks like a button to a screen reader but has no keyboard handling, no focusability, and no click-on-Enter until you add all of that in JavaScript. A `<button>` gives you everything for free.

This is the "first rule of ARIA": if a native element with the semantics and behavior you need exists, use it instead of repurposing a `<div>`/`<span>` and bolting on a role. Custom ARIA widgets are a real cost — only pay it when no native element fits.

## Pillar 1 — Semantic HTML

Markup should represent the _meaning_ of the content; presentation is CSS's job alone. A `<span style="font-size:32px">` may look like a heading but carries no meaning for screen readers, search engines, or other developers.

Core habits:

- Use landmarks to structure the page: `<header>`, `<nav>`, `<main>` (one per page), `<aside>`, `<footer>`. These let screen-reader users jump between regions.
- Use one `<h1>` per page and never skip heading levels for visual size — style with CSS instead.
- Use `<button>` for actions and `<a href>` for navigation. Don't swap them.
- Use `<ul>`/`<ol>`/`<li>` for lists, `<table>` with `<th scope>` for tabular data, `<figure>`/`<figcaption>` for captioned media.
- Use real form elements: `<label for>` tied to every input, `<fieldset>`/`<legend>` for groups.

Full element guide and anti-patterns: [references/semantic-html.md](references/semantic-html.md).

## Pillar 2 — ARIA Attributes

ARIA states (current condition, e.g. `aria-expanded`, `aria-checked`, `aria-pressed`) and properties (fixed characteristics, e.g. `aria-label`, `aria-labelledby`, `aria-describedby`) describe elements to assistive technology when native HTML can't.

The attributes you'll reach for most:

- **Naming:** `aria-label`, `aria-labelledby` (give an accessible name when no visible `<label>`/text exists). Not allowed on `role="presentation"`/`none`.
- **Describing:** `aria-describedby` (link help text or error messages to a field).
- **Hiding:** `aria-hidden="true"` (remove purely decorative content from the accessibility tree — never on focusable elements).
- **Widget state:** `aria-expanded`, `aria-pressed`, `aria-checked`, `aria-selected`, `aria-disabled`, `aria-invalid`, `aria-required`, `aria-haspopup`, `aria-controls`.
- **Navigation:** `aria-current="page"` for the active link.
- **Dynamic updates:** `aria-live` (`polite`/`assertive`) with `aria-atomic` for content that changes without a reload.

If you build a custom widget, you own its keyboard interaction and must keep ARIA states in sync with JavaScript. Full attribute reference and patterns: [references/aria-attributes.md](references/aria-attributes.md).

## Pillar 3 — WCAG Color Contrast

WCAG 2.2 SC 1.4.3 (Level AA) is the baseline for readable text:

| Content                                                  | Minimum ratio (AA) |
| -------------------------------------------------------- | ------------------ |
| Normal text                                              | **4.5 : 1**        |
| Large text (≥ 18pt / 24px, or ≥ 14pt / 18.66px **bold**) | **3 : 1**          |

Exceptions: incidental text (disabled controls, pure decoration, invisible text) and logotypes have no requirement. AAA (SC 1.4.6) raises these to 7:1 and 4.5:1.

Don't eyeball it — compute the ratio (see the reference for the formula and tools), and never rely on color alone to convey meaning (pair it with text, icons, or underlines). Details and the related non-text contrast guidance: [references/wcag-contrast.md](references/wcag-contrast.md).

## Pre-Ship Checklist

- [ ] Page has one `<h1>`, logical heading order, and landmark regions (`main`, `nav`, etc.).
- [ ] Every interactive control is a native `<button>`/`<a>`/form element, or — if custom — is focusable, keyboard-operable, and has a role + synced ARIA state.
- [ ] Every input has an associated `<label>` (or `aria-label`/`aria-labelledby`); errors use `aria-describedby` + `aria-invalid`.
- [ ] Every meaningful `<img>` has descriptive `alt`; decorative images have `alt=""` or `aria-hidden`.
- [ ] No `aria-*` that duplicates what a native element already provides.
- [ ] Text contrast meets 4.5:1 (3:1 for large text); meaning is never conveyed by color alone.
- [ ] Keyboard-only pass: everything reachable and operable, focus visible, order logical.
