# Kundli Pipeline — Design Tokens

Extracted from `Prototype_Short_Kundli.docx`. These are authoritative.
The prototype is no longer needed for a build; everything it defined lives here.

## Palette

| Role | Hex |
|---|---|
| Headings (maroon) | `7A1F1F` |
| Underlines, footer, page numbers (gold) | `8A5A00` |
| Body text | `1A1A1A` |
| Secondary / accent | `553311` |

## Page geometry (A4)

| Value | DXA |
|---|---|
| Page width | 11906 |
| Page height | 16838 |
| Margins (all four) | 720 (0.5 inch) |
| Header / footer | 708 |
| **Content width (CW)** | **10466** |

Table widths must be normalised to the full text column: 10440–10466 DXA.

## Fonts

- Noto Serif Devanagari (headings + body)
- Noto Sans Devanagari (secondary)

Use static TTFs from `fonts-noto-core`. Do NOT instantiate variable fonts via
fontTools — unnecessary and slower.

Noto Devanagari lacks bullet glyphs (U+2022, U+00B7, U+25C6).
Safe separators: `–` `—` `|` `।` `॥`

## Decorative images

Full-page images, one per page. Sizes below are the prototype's exact extents.

| File | Role | EMU (cx × cy) | Pixels (÷ 9525) |
|---|---|---|---|
| `01_cover.png` | Cover — page 1 | 6219825 × 9334500 | 653 × 980 |
| `02_part1_kundli_vishleshan.png` | Part 1 divider — कुंडली विश्लेषण | 6286500 × 8591550 | 660 × 902 |
| `03_part2_dasha.png` | Part 2 divider — दशा विश्लेषण | 6286500 × 8801100 | 660 × 924 |
| `04_part3_dosh.png` | Part 3 divider — दोष विचार | 6286500 × 8896350 | 660 × 934 |
| `05_part4_barah_vishay.png` | Part 4 divider — बारह विषय | 6286500 × 8705850 | 660 × 914 |
| `06_qa_prashn_uttar.png` | Q&A divider — प्रश्न एवं उत्तर (OPTIONAL) | 6219825 × 9334500 | 653 × 980 |
| `07_conclusion.png` | Conclusion divider | 6286500 × 8886825 | 660 × 933 |
| `08_contact.png` | Contact page — final page | 6219825 × 9334500 | 653 × 980 |

`06_qa_prashn_uttar.png` is used only when the client's report includes the
प्रश्न एवं उत्तर section. Omit both divider and section when not requested.

## Conversion constants

```
EMU  -> pixels :  EMU / 9525
DXA  -> pixels :  DXA / 15
ImageRun transformation.width/height takes PIXELS at 96 DPI, not DXA.
Chart images:  jpegDims() scaled to  CW * frac / 15
```

## Layout rules (frozen — these are bug fixes, do not regress)

1. `ImageRun` paragraphs must OMIT the `line` key from `spacing`.
   LibreOffice clips the image vertically if it is present.
2. Never use `PageBreak` as a `TextRun` child — it emits a spurious blank page.
   Use `pageBreakBefore: true` on the paragraph instead.
3. Full-page decorative images: `pageBreakBefore: true`, except the FIRST
   image (cover), which uses `false` to avoid a blank leading page.
4. `H1()` forces `pageBreakBefore: true`. Sub-headings that belong inside a
   parent section must use `H2()`, or you get spurious half-empty pages.
5. `dividerPage()` returns an array — callers must spread: `add(...arr)`.

## Page-fill quality gate

Measured on the body region, excluding top ~5% and bottom ~7% margins:

```
body-area fill      >= 0.75
mean ink density    >= 0.002      (blank-page detection)
```

Measure with `pdftoppm -jpeg -r 60` then NumPy pixel darkness analysis
(`mapsec.py`). Regenerate rasters after every rebuild.

## Content rules

- Future-only. No past dates or events; the birth year is the sole exception.
- The future-only rule must NEVER appear as visible text in the report.
- "यह क्या है" sub-headings are rewritten to directly answer
  "X कुंडली/दोष क्या होती/होता है?"
- Chart identification: numbers-only = Ashtakvarga; letters and numbers =
  Chandra; the remaining three are titled within the image itself.
- Page numbers on every non-cover page, centered, `— N —`, gold.

## Section order

Cover → मूल विवरण table + वैदिक सार → Part 1 divider → Lagna, Chandra, Chalit,
Navamsa, Ashtakvarga → Part 2 divider → Vimshottari + Yogini dasha →
Part 3 divider → Sade Sati, Mangal, Kalsarpa, Pitru/Grahan/Guru-Chandal,
Rajyoga → Part 4 divider → twelve life topics, transits, annual reports →
puja recommendations → [optional Q&A divider + Q&A] → personal guidance +
disclaimer → contact page.

