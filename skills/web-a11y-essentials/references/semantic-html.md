# Semantic HTML Reference

Source: [MDN — Semantics](https://developer.mozilla.org/en-US/docs/Glossary/Semantics)

## What "semantic" means

Semantics is the _meaning_ of an element — the role it plays — not how it looks by default. A semantic element tells the browser, search engines, and assistive technology what the content _is_. The guiding principle:

> HTML should represent the **data** it contains, not its presentation. How it looks is the sole responsibility of CSS.

When choosing an element, ask: "What best describes this content?" — a list? an article with sections? something needing a caption, header, or footer?

## Why it matters

| Benefit         | How semantics helps                                                                         |
| --------------- | ------------------------------------------------------------------------------------------- |
| Accessibility   | Screen readers expose landmarks, headings, and controls so users can navigate by structure. |
| SEO             | Search engines weigh semantic structure when interpreting and ranking content.              |
| Maintainability | Meaningful tags are easier to scan than a wall of `<div>`s.                                 |
| Communication   | Element names tell the next developer what data belongs there.                              |

## Element groups

### Document landmarks (structure)

- `<header>` — introductory content / banner for the page or a section.
- `<nav>` — major navigation blocks.
- `<main>` — the primary content. **One per page**, not nested in other landmarks.
- `<article>` — self-contained, independently distributable content (a post, card, comment).
- `<section>` — a thematic grouping, usually with a heading.
- `<aside>` — tangentially related content (sidebars, pull quotes).
- `<footer>` — footer for the page or a section.

Landmarks let assistive-tech users jump directly between regions, so use them instead of generic `<div>`s for top-level structure.

### Headings

- `<h1>`–`<h6>` convey document outline. Use **one `<h1>`** per page, and don't skip levels (no `<h1>` → `<h3>`) just to get a font size. Style with CSS.

### Content

- `<figure>` / `<figcaption>` — media with a caption.
- `<details>` / `<summary>` — native disclosure widget (expand/collapse) with no JS required.
- `<ul>` / `<ol>` / `<li>` — unordered / ordered lists.
- `<dl>` / `<dt>` / `<dd>` — description/definition lists.
- `<table>` with `<caption>`, `<thead>`, `<th scope="col|row">` — tabular data only, never layout.

### Text-level

- `<mark>` — highlighted/relevant text.
- `<time datetime="...">` — machine-readable dates/times.
- `<strong>` (importance) and `<em>` (emphasis) — prefer over `<b>`/`<i>`, which carry no semantic weight.
- `<abbr>`, `<code>`, `<kbd>`, `<q>`, `<blockquote>`, `<cite>`.

### Interactive

- `<button>` — triggers an action in the page. Focusable and keyboard-operable for free.
- `<a href>` — navigates to a URL. If there's no `href`, it's not a real link.
- `<form>`, `<label for>`, `<input>`, `<select>`, `<textarea>`, `<fieldset>`, `<legend>`.

## Anti-patterns to flag

- `<div>`/`<span>` styled to _look_ like a heading, button, or link. No meaning, no keyboard support.
- `<div onclick>` as a button — not focusable, not operable by keyboard, invisible to screen readers as a control.
- Using `<table>` for page layout.
- Skipping heading levels or using multiple `<h1>`s for visual hierarchy.
- An `<a>` used to run an action (use `<button>`); a `<button>` used to navigate (use `<a href>`).
- Inputs without an associated `<label>`.

## The litmus test

Strip away all CSS. If the bare HTML still reads as a sensible, well-structured document — headings, lists, controls, regions all making sense — the semantics are right.
