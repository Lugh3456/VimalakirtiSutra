# Architecture — Vimalakirti Sutra

## Overview

Vimalakirti Sutra is a **static multi-page mini site** hosted on GitHub Pages. It is a spoke in the hub-and-spoke model under The Dharma Gate portal, following the same architecture as DiamondSutra.

- The **hub** is [The Dharma Gate](https://lugh3456.github.io/DharmaGate/)
- This **spoke** is a standalone repository with its own index and 14 section pages
- The hub links to this site (card symbol 维); this site links back via portal bar and footer

HTML + external CSS only. No frameworks, no build tools.

---

## Folder Structure

```
VimalakirtiSutra/
  index.html          <- chapter grid (14 cards)
  section1.html       <- Ch 1: 佛国品
  section2.html       <- Ch 2: 方便品
  section3.html       <- Ch 3: 弟子品
  section4.html       <- Ch 4: 菩萨品
  section5.html       <- Ch 5: 文殊师利问疾品
  section6.html       <- Ch 6: 不思议品
  section7.html       <- Ch 7: 观众生品
  section8.html       <- Ch 8: 佛道品
  section9.html       <- Ch 9: 入不二法门品
  section10.html      <- Ch 10: 香积佛品
  section11.html      <- Ch 11: 菩萨行品
  section12.html      <- Ch 12: 见阿閦佛品
  section13.html      <- Ch 13: 法供养品
  section14.html      <- Ch 14: 嘱累品
  css/
    style.css         <- all styles (design tokens, components, responsive)
  docs/
    readme.md         <- project overview + chapter table
    architecture.md   <- this file
    ai-context.md     <- instructions for AI assistants
    roadmap.md        <- delivery checklist
    audit-config.json <- audit system config
    reference.py      <- authoritative source texts (VS dict, 14 chapters)
```

---

## Page Structure — Index

```
<portal-bar>           <- link back to DharmaGate
<header.site-header>   <- "Vimalakirti Sutra" title, Chinese subtitle, tradition label
<section.intro>        <- brief description of the sutra
<main.chapter-grid>    <- 14 chapter cards (responsive grid, 14 links)
  a.chapter-card       <- chapter number (saffron) + Chinese title (serif)
<footer>
```

The index grid uses `repeat(auto-fill, minmax(180px, 1fr))` at desktop and `repeat(2, 1fr)` on mobile (≤600px). The chapter-grid layout is scoped in a `<style>` tag on `index.html` (not in `style.css`).

---

## Page Structure — Section Pages

```
<portal-bar>              <- link back to DharmaGate
<header.section-header>   <- chapter name, ← prev / 目录 Contents / next → nav
<main.section-container>
  section.full-text       <- full Chinese text + 🔊 聆听经文 button
  section.line-explanation
    h2                    <- "逐句解释"
    .explanation-card     <- one per key phrase:
      p.line              <- Chinese phrase (bold)
      p.explanation-zh    <- short Chinese explanation (concept-focused)
      button.toggle-btn   <- "English ▾"
      .detail-content     <- hidden by default:
        p.translation     <- English translation (red)
        p.explanation     <- full English commentary (grey)
  section.summary
    h2                    <- "总结 · Summary"
    p.summary-zh          <- Chinese summary
    button.toggle-btn     <- "English ▾"
    .detail-content       <- hidden by default:
      p.explanation       <- English summary
<script>                  <- toggleDetail() + speak()
<footer>
```

---

## Full Text Block — Important Difference from DiamondSutra

Chapters with multiple paragraphs or verse blocks use a `<div id="chineseText">` container rather than a single `<p>`:

```html
<div id="chineseText">
  <p>...prose paragraph...</p>
  <div class="verse-block">
    line one　　line two　　line three　　line four<br>
    ...
  </div>
  <p>...more prose...</p>
</div>
```

The audit script auto-detects the actual element tag from the HTML — no config field is needed.

---

## CSS Architecture

All styles are in `css/style.css`, copied verbatim from DiamondSutra. Organised into labelled sections:

```
DESIGN TOKENS          <- CSS custom properties (:root)
SKIP LINK              <- accessibility
RESET & BASE
PORTAL BAR
HEADER — INDEX
HEADER — SECTION PAGE
INTRO — INDEX
SECTION CARD GRID      <- (not used on index; retained from shared CSS)
SECTION PAGE CONTAINER
FULL TEXT BLOCK
LINE-BY-LINE EXPLANATION
EXPAND / COLLAPSE TOGGLE
SUMMARY
FOOTER
RESPONSIVE             <- <= 600px
```

---

## Design Tokens

| Token | Value | Usage |
|-------|-------|-------|
| `--bg` | `#f7f0e6` | Page background (warm cream) |
| `--surface` | `#ffffff` | Card backgrounds |
| `--ink` | `#1c0f0f` | Header, footer, portal bar backgrounds |
| `--red` | `#b83232` | Primary accent |
| `--red-light` | `#d44e4e` | Hover states, heading colour |
| `--saffron` | `#c8820a` | Chapter number colour |
| `--saffron-lt` | `#e09c2a` | Portal bar link colour |
| `--text` | `#2d1f1f` | Body text, Chinese explanations |
| `--muted` | `#7a6060` | English explanations, toggle button |
| `--border` | `#e0d0c8` | Card borders, dividers |

---

## Expand/Collapse Pattern

Each explanation card and the summary section use a simple toggle:

- A `<button class="toggle-btn">` sits after the Chinese content
- Its sibling `<div class="detail-content" hidden>` holds the English content
- `toggleDetail(btn)` flips the `hidden` attribute and swaps ▾/▴
- `aria-expanded` is updated for accessibility

No external JS libraries. Plain vanilla JavaScript.

---

## Speech Synthesis

Each section page has a 🔊 聆听经文 button that reads the full Chinese text aloud using the Web Speech API. It targets Apple's Ting-Ting voice on iOS Safari (with fallback to any `zh-CN` voice) and runs at 0.9x speed for clarity.

---

## External Dependencies

Google Fonts (loaded via `<link>`):
- Noto Serif SC — headings, Chinese phrases
- Noto Sans SC — body text, buttons, descriptions

No other external dependencies.

---

## Responsive Breakpoints

| Breakpoint | Change |
|-----------|--------|
| > 600px | Default layout — auto-fill grid on index |
| <= 600px | 2-column grid on index, single-column cards on section pages, reduced padding |

---

## Navigation

Each section page has a `<nav class="section-nav">` with three links:
- `← 第N-1章` — previous chapter (or absent on chapter 1)
- `目录 Contents` — back to index
- `第N+1章 →` — next chapter (or absent on chapter 14)

---

## Hosting

GitHub Pages via the `lugh3456` account.
Deployment: push files to repository root on `main` branch.
Live URL: `https://lugh3456.github.io/VimalakirtiSutra/`
