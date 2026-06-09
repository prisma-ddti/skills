# WCAG Color Contrast Reference

Source: [WCAG 2.2 — SC 1.4.3 Contrast (Minimum)](https://www.w3.org/TR/WCAG22/#contrast-minimum)

## SC 1.4.3 Contrast (Minimum) — Level AA

The visual presentation of text and images of text must meet a minimum contrast ratio against its background:

| Content     | Minimum ratio |
| ----------- | ------------- |
| Normal text | **4.5 : 1**   |
| Large text  | **3 : 1**     |

**Large text** = at least **18 point (≈ 24px)**, or **14 point bold (≈ 18.66px bold)**, or the equivalent size for CJK fonts.

### Exceptions (no contrast requirement)

- **Incidental** — text in inactive/disabled UI components, pure decoration, text not visible to anyone, or text that's part of a picture containing significant other visual content.
- **Logotypes** — text that is part of a logo or brand name.

## SC 1.4.6 Contrast (Enhanced) — Level AAA

A stricter target when you need it:

| Content     | Minimum ratio |
| ----------- | ------------- |
| Normal text | **7 : 1**     |
| Large text  | **4.5 : 1**   |

## SC 1.4.11 Non-text Contrast — Level AA (related)

Not part of 1.4.3, but the same discipline applies to non-text: UI component boundaries (input borders, focus indicators, toggle states) and meaningful graphics need **3 : 1** against adjacent colors. Worth checking whenever you style controls.

## How the ratio is computed

Contrast ratio ranges from 1:1 (identical) to 21:1 (black on white). It's based on relative luminance:

```
ratio = (L1 + 0.05) / (L2 + 0.05)
```

where `L1` is the lighter color's relative luminance and `L2` the darker's. Relative luminance of an sRGB color: linearize each channel, then `L = 0.2126·R + 0.7152·G + 0.0722·B`. You don't compute this by hand — use a tool.

## Tools

- Browser DevTools: the color picker in the Styles panel shows the live contrast ratio and AA/AAA pass marks.
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- Automated audits: Lighthouse, axe DevTools, WAVE.

## Best-practice rules

- **Don't eyeball contrast** — compute it. Borderline pairs (mid-grays on white, light text on color) fail more often than they look.
- **Never convey meaning by color alone.** Required fields, errors, chart series, and links must also be distinguishable by text, icon, underline, or pattern — this helps color-blind users and satisfies SC 1.4.1.
- **Watch state and theme variants.** Placeholder text, disabled-but-meaningful text, hover/focus colors, and dark-mode tokens each need their own contrast check.
- **Text over images/gradients** needs a scrim, overlay, or text shadow so the ratio holds across the whole background.
- **Bake thresholds into design tokens.** Define color pairs that pass 4.5:1 once, then reuse them, rather than re-checking ad hoc.

## Quick reference

| Situation                                | Target            |
| ---------------------------------------- | ----------------- |
| Body text, labels, links                 | 4.5 : 1           |
| Headings ≥ 24px or ≥ 18.66px bold        | 3 : 1             |
| Input borders, focus rings, icon buttons | 3 : 1 (SC 1.4.11) |
| Disabled controls, logos                 | exempt            |
| Aiming for AAA body text                 | 7 : 1             |
