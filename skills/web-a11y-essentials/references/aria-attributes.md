# ARIA Attributes Reference

Source: [MDN — ARIA Attributes](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes)

## The first rule of ARIA

> ARIA only modifies the **accessibility tree** — how assistive technology presents content. It does **not** change an element's function or behavior.

Consequences:

- A `role` or `aria-*` makes a screen reader _announce_ something, but adds zero behavior. `<div role="button">` is announced as a button yet isn't focusable and doesn't fire on Enter/Space — you'd have to add `tabindex`, key handlers, and click logic yourself.
- Therefore: **use the native element first.** Only build an ARIA widget when no native element fits, and then you own its full keyboard interaction and state management.
- A wrong ARIA value is worse than none — it actively misleads users. Don't add ARIA "just in case."

## States vs. properties

- **States** reflect a _current, changeable_ condition: `aria-expanded`, `aria-checked`, `aria-pressed`, `aria-selected`, `aria-disabled`, `aria-invalid`, `aria-busy`. Keep these in sync with JavaScript as the UI changes.
- **Properties** define a _more fixed_ characteristic: `aria-label`, `aria-labelledby`, `aria-describedby`, `aria-controls`, `aria-haspopup`, `aria-required`.

## Most-used attributes

| Attribute          | Purpose                                        | Example                                                |
| ------------------ | ---------------------------------------------- | ------------------------------------------------------ |
| `aria-label`       | Accessible name when no visible text           | `<button aria-label="Close dialog">×</button>`         |
| `aria-labelledby`  | Name taken from other element(s) by id         | `<section aria-labelledby="t"><h2 id="t">Billing</h2>` |
| `aria-describedby` | Link supplementary description / help / error  | `<input aria-describedby="pw-help">`                   |
| `aria-hidden`      | Remove decorative content from a11y tree       | `<svg aria-hidden="true">`                             |
| `aria-expanded`    | Disclosure/menu/accordion open state           | `<button aria-expanded="false" aria-controls="menu">`  |
| `aria-controls`    | Element this control affects                   | `<button aria-controls="panel-1">`                     |
| `aria-current`     | Current item within a set                      | `<a href="/" aria-current="page">Home</a>`             |
| `aria-pressed`     | Toggle-button state                            | `<button aria-pressed="true">Bold</button>`            |
| `aria-checked`     | Checkbox/radio/switch state (custom)           | `<div role="switch" aria-checked="true">`              |
| `aria-selected`    | Selection in lists/tabs/grids                  | `<div role="tab" aria-selected="true">`                |
| `aria-disabled`    | Perceivable-but-inert control                  | `<button aria-disabled="true">`                        |
| `aria-invalid`     | Field failed validation                        | `<input aria-invalid="true">`                          |
| `aria-required`    | Field is required                              | `<input aria-required="true">`                         |
| `aria-haspopup`    | Control opens a menu/dialog/listbox            | `<button aria-haspopup="menu">`                        |
| `aria-live`        | Announce dynamic updates                       | `<div aria-live="polite">Saved</div>`                  |
| `aria-atomic`      | Announce the whole region, not just the change | `<div aria-live="polite" aria-atomic="true">`          |

## Attribute categories (MDN)

- **Global** (allowed on any element regardless of role): includes `aria-label`, `aria-labelledby`, `aria-describedby`, `aria-hidden`, `aria-current`, `aria-controls`, `aria-disabled`, `aria-haspopup`, `aria-live`, `aria-busy`, `aria-roledescription`, and more.
- **Widget**: `aria-checked`, `aria-expanded`, `aria-pressed`, `aria-selected`, `aria-disabled`, `aria-invalid`, `aria-required`, `aria-valuemin/max/now/text`, `aria-sort`, etc.
- **Live region**: `aria-live`, `aria-atomic`, `aria-relevant`, `aria-busy`.
- **Relationship**: `aria-controls`, `aria-describedby`, `aria-labelledby`, `aria-owns`, `aria-activedescendant`, `aria-posinset`, `aria-setsize`, `aria-colindex`/`rowindex`, etc.

## Naming rules

- `aria-label` and `aria-labelledby` are **not allowed** on `role="presentation"` / `role="none"`.
- A visible `<label for>` is preferable to `aria-label` — it benefits sighted users too and is more robust.
- `aria-labelledby` wins over `aria-label` when both are present; `aria-label` overrides visible content. Use deliberately.

## Common mistakes to flag

- Adding a `role` that the element already has (`<button role="button">`, `<nav role="navigation">`) — redundant.
- `aria-hidden="true"` on a focusable element — keyboard users still reach it, but it's gone from the a11y tree → confusion.
- Building a custom widget with ARIA but no keyboard support (no `tabindex`, no Arrow/Enter/Escape handling).
- ARIA states that drift out of sync with the actual UI state.
- Using ARIA to patch markup that should just be a native element.

## Pattern: accessible custom widget (when truly needed)

For non-native widgets (tabs, comboboxes, custom selects), follow the [ARIA Authoring Practices Guide (APG)](https://www.w3.org/WAI/ARIA/apg/patterns/) keyboard and role conventions rather than inventing your own — users expect standard key behavior (Arrows to move, Enter/Space to activate, Escape to dismiss).
