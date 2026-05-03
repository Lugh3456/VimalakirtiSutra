# AI Context — Vimalakirti Sutra

## What this project is

A static mini site for studying the Vimalakirti Sutra (维摩诘所说经), one of nine scripture sites under The Dharma Gate portal. Same architecture as DiamondSutra — HTML + external CSS, no frameworks, 14 section pages.

Source text: Kumārajīva translation (鸠摩罗什译), T0475, CBETA digital edition. Traditional Chinese converted to Simplified via opencc.

## Current state (as of 2026-05-02)

- index.html: 14-card chapter grid — complete
- section1.html–section14.html: all 14 chapters complete with full Chinese source text, bilingual explanation cards (5–11 cards per section), speech synthesis, and bilingual summary
- docs/: readme.md, architecture.md, ai-context.md, roadmap.md, audit-config.json, reference.py — all complete
- **Audit: ✓ 0 issues across all 14 pages** (both audit_content.py and check_structure.py pass clean)
- Ready for deployment — next step is creating the GitHub repository

## Explanation card pattern

```html
<div class="explanation-card">
  <p class="line"><strong>Chinese phrase</strong></p>
  <p class="explanation-zh">Short Chinese explanation (2–3 sentences, concept-focused)</p>
  <button class="toggle-btn" onclick="toggleDetail(this)" aria-expanded="false">English ▾</button>
  <div class="detail-content" hidden>
    <p class="translation">English translation (red)</p>
    <p class="explanation">Full English commentary (grey, with analogies)</p>
  </div>
</div>
```

## Summary pattern

```html
<section class="summary">
  <h2>总结 · Summary</h2>
  <p class="summary-zh">Chinese summary</p>
  <button class="toggle-btn" onclick="toggleDetail(this)" aria-expanded="false">English ▾</button>
  <div class="detail-content" hidden>
    <p class="explanation">English summary</p>
  </div>
</section>
```

## Chapter card pattern (index.html)

```html
<a href="sectionN.html" class="chapter-card">
  <span class="chapter-num">第N章</span>
  <span class="chapter-zh">章名</span>
</a>
```

No English subtitle on the cards — removed by design.

## Full text block — important difference from DiamondSutra

Chapter 1 (and other chapters with multiple paragraphs or verse blocks) uses a `<div>` rather than `<p>` for the full text, because the content spans multiple paragraph breaks and a verse block:

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

Chapters that are pure prose with a single paragraph may use `<p id="chineseText">` instead.

The audit script auto-detects the actual element tag from the HTML — `source_text_element` in audit-config.json is no longer needed and has been removed.

## Verse reconstruction note

Chapter 1 contains a long verse (偈). In the CBETA PDF, pdfminer extracts the 4-column verse table column-by-column (all of column 1 top-to-bottom, then column 2, etc.). To reconstruct row-by-row reading order, each column string is split into 7-character lines and the lines are interleaved:

```python
verse_lines = ['　　'.join(col_lines[j][i] for j in range(4)) for i in range(n)]
```

Other verse blocks in later chapters should be checked for the same issue and reconstructed the same way.

## Special character substitutions

The CBETA PDF uses circled/special characters in place of standard CJK characters. Always apply these substitutions before OpenCC conversion:

```python
subs = [
    ('㆒', '一'), ('㆔', '三'), ('㆕', '四'),
    ('㆗', '中'), ('㆝', '天'), ('㆟', '人'), ('㆞', '地'),
]
```

Additionally, Chapter 12 (见阿閦佛品): the character 閦 (U+95A6) is rendered as ฀ (U+0E00) by pdfminer. Apply:

```python
text = re.sub(r'见阿฀佛', '见阿閦佛', text)
text = re.sub(r'阿฀佛', '阿閦佛', text)
```

## Chapter header locations in PDF

Chapters 1–4 use full headers (e.g. `维摩诘所说经佛国品第一`). Chapters 5 and 10 appear after volume markers and use shorter headers:

- Ch 5: search for `文殊师利问疾品第五` (after `维摩诘所说经卷中`)
- Ch 10: search for `香积佛品第十` (after `维摩诘所说经卷下`)

## Reference data

`docs/reference.py` contains all 14 chapters in a dict `VS`:

```python
VS = {
    1: "...",   # 佛国品       ~3011 chars
    2: "...",   # 方便品       ~1121 chars
    3: "...",   # 弟子品       ~3806 chars
    4: "...",   # 菩萨品       ~2980 chars
    5: "...",   # 文殊师利问疾品 ~2695 chars
    6: "...",   # 不思议品      ~1945 chars
    7: "...",   # 观众生品      ~2572 chars
    8: "...",   # 佛道品       ~1850 chars
    9: "...",   # 入不二法门品   ~1725 chars
   10: "...",   # 香积佛品      ~2184 chars
   11: "...",   # 菩萨行品      ~2323 chars
   12: "...",   # 见阿閦佛品    ~1579 chars
   13: "...",   # 法供养品      ~1634 chars
   14: "...",   # 嘱累品       ~838 chars
}
```

## Audit system

`docs/audit-config.json` + `docs/reference.py` power the annotated-text-audit skill.

Run from the skills-source copy (since the installed skill location is read-only):
```
python3 ~/Documents/Claude/Cowork/skills-source/annotated-text-audit/scripts/audit_content.py <project_folder>
python3 ~/Documents/Claude/Cowork/skills-source/annotated-text-audit/scripts/check_structure.py <project_folder>
```

Reference variable: `VS`.

Config notes:
- `check_card_coverage: false` — this site uses thematic section cards, not line-by-line annotation; coverage check is disabled
- `check_title_h1: false` — headings use Chinese chapter numbers (第N章), not "Chapter N"
- `source_text_element` is NOT set — the audit script now auto-detects the element tag from the HTML

## Design tokens

| Token | Value | Usage |
|-------|-------|-------|
| `--red` | `#b83232` | Primary accent |
| `--red-light` | `#d44e4e` | Hover states, headings |
| `--saffron` | `#c8820a` | Chapter number colour |
| `--saffron-lt` | `#e09c2a` | Portal bar links |
| `--ink` | `#1c0f0f` | Header/footer backgrounds |
| `--bg` | `#f7f0e6` | Page background |
| `--text` | `#2d1f1f` | Body text, Chinese explanations |
| `--muted` | `#7a6060` | English text, toggle buttons |
| `--border` | `#e0d0c8` | Card borders, dividers |

## Portal link

Portal bar and footer both link to: `https://lugh3456.github.io/DharmaGate/`

## GitHub repo

Live URL (when deployed): `https://lugh3456.github.io/VimalakirtiSutra/`

## Notable content highlights

- Ch 1 (佛国品): 随其心净则佛土净 — the pure land is the mind itself; Śāriputra's famous doubt resolved by 螺髻梵王
- Ch 2 (方便品): Vimalakirti feigns illness to teach the Dharma; all dharmas are like illusions
- Ch 3–4 (弟子品/菩萨品): Every disciple and bodhisattva declines to visit — each recalls being corrected by Vimalakirti
- Ch 5 (文殊师利问疾品): Manjushri finally visits; illness as upaya; non-arising and non-cessation
- Ch 6 (不思议品): The 不思议 liberation — 84,000 thrones fit into a tiny room; Mount Sumeru fits in a mustard seed
- Ch 7 (观众生品): The goddess scatters flowers; Śāriputra cannot shake them off; gender is empty of inherent existence
- Ch 8 (佛道品): Entering the Buddha-path through the crooked path; all defilements are the seeds of Buddhahood
- Ch 9 (入不二法门品): 32 bodhisattvas each give an answer on non-duality; Vimalakirti's thunderous silence is the ultimate answer
- Ch 10 (香积佛品): Fragrance accumulation Buddha-land; a ladle of fragrant rice feeds the entire assembly
- Ch 12 (见阿閦佛品): 见阿閦佛 — beholding Akshobhya Buddha; the entire Abhirati world appears in the assembly
- Ch 13 (法供养品): 法供养 — offering the Dharma surpasses all material offerings
