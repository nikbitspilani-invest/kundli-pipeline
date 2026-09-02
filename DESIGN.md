# Kundli Pipeline â€” Design Tokens

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

Table widths must be normalised to the full text column: 10440â€“10466 DXA.

## Fonts

- Noto Serif Devanagari (headings + body)
- Noto Sans Devanagari (secondary)

Use static TTFs from `fonts-noto-core`. Do NOT instantiate variable fonts via
fontTools â€” unnecessary and slower.

Noto Devanagari lacks bullet glyphs (U+2022, U+00B7, U+25C6).
Safe separators: `â€“` `â€”` `|` `à¥¤` `à¥¥`

## Decorative images

Full-page images, one per page. Sizes below are the prototype's exact extents.

| File | Role | EMU (cx Ã— cy) | Pixels (Ã· 9525) |
|---|---|---|---|
| `01_cover.png` | Cover â€” page 1 | 6219825 Ã— 9334500 | 653 Ã— 980 |
| `02_part1_kundli_vishleshan.png` | Part 1 divider â€” à¤•à¥à¤‚à¤¡à¤²à¥€ à¤µà¤¿à¤¶à¥à¤²à¥‡à¤·à¤£ | 6286500 Ã— 8591550 | 660 Ã— 902 |
| `03_part2_dasha.png` | Part 2 divider â€” à¤¦à¤¶à¤¾ à¤µà¤¿à¤¶à¥à¤²à¥‡à¤·à¤£ | 6286500 Ã— 8801100 | 660 Ã— 924 |
| `04_part3_dosh.png` | Part 3 divider â€” à¤¦à¥‹à¤· à¤µà¤¿à¤šà¤¾à¤° | 6286500 Ã— 8896350 | 660 Ã— 934 |
| `05_part4_barah_vishay.png` | Part 4 divider â€” à¤¬à¤¾à¤°à¤¹ à¤µà¤¿à¤·à¤¯ | 6286500 Ã— 8705850 | 660 Ã— 914 |
| `06_qa_prashn_uttar.png` | Q&A divider â€” à¤ªà¥à¤°à¤¶à¥à¤¨ à¤à¤µà¤‚ à¤‰à¤¤à¥à¤¤à¤° (OPTIONAL) | 6219825 Ã— 9334500 | 653 Ã— 980 |
| `07_conclusion.png` | Conclusion divider | 6286500 Ã— 8886825 | 660 Ã— 933 |
| `08_contact.png` | Contact page â€” final page | 6219825 Ã— 9334500 | 653 Ã— 980 |

`06_qa_prashn_uttar.png` is used only when the client's report includes the
à¤ªà¥à¤°à¤¶à¥à¤¨ à¤à¤µà¤‚ à¤‰à¤¤à¥à¤¤à¤° section. Omit both divider and section when not requested.

## Conversion constants

```
EMU  -> pixels :  EMU / 9525
DXA  -> pixels :  DXA / 15
ImageRun transformation.width/height takes PIXELS at 96 DPI, not DXA.
Chart images:  jpegDims() scaled to  CW * frac / 15
```

## Layout rules (frozen â€” these are bug fixes, do not regress)

1. `ImageRun` paragraphs must OMIT the `line` key from `spacing`.
   LibreOffice clips the image vertically if it is present.
   1b. This includes INHERITED line spacing. Do NOT set
   `styles.default.document.paragraph.spacing.line` on the `Document` â€”
   it cascades into ImageRun paragraphs and clips every decorative PNG to
   a sliver. Set line spacing per-paragraph in `P()` / `LI()` instead.
2. Never use `PageBreak` as a `TextRun` child â€” it emits a spurious blank page.
   Use `pageBreakBefore: true` on the paragraph instead.
3. Full-page decorative images: `pageBreakBefore: true`, except the FIRST
   image (cover), which uses `false` to avoid a blank leading page.
4. `H1()` forces `pageBreakBefore: true`. Sub-headings that belong inside a
   parent section must use `H2()`, or you get spurious half-empty pages.
5. `dividerPage()` returns an array â€” callers must spread: `add(...arr)`.
6. The cover must carry no page number. Use `titlePage: true` in the section
   properties with `footers: { default: pageFooter(), first: blankFooter() }`.

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
- "à¤¯à¤¹ à¤•à¥à¤¯à¤¾ à¤¹à¥ˆ" sub-headings are rewritten to directly answer
  "X à¤•à¥à¤‚à¤¡à¤²à¥€/à¤¦à¥‹à¤· à¤•à¥à¤¯à¤¾ à¤¹à¥‹à¤¤à¥€/à¤¹à¥‹à¤¤à¤¾ à¤¹à¥ˆ?"
- Chart identification: numbers-only = Ashtakvarga; letters and numbers =
  Chandra; the remaining three are titled within the image itself.
- Page numbers on every non-cover page, centered, `â€” N â€”`, gold.

## Word budget (calibrated)

Calibrated against the Anshi build (31 pages, all passing the gate). Figures are
Devanagari words of body text, counted per printed page.

**Full-page capacity** â€” words that fill a page to fill = 1.00:

| Page composition | Capacity |
|---|---|
| Prose only (H1 + 2â€“4 H2 + paragraphs) | ~820 |
| + one `gridTable` (7 rows) | ~645 |
| + one `kvTable` (7 rows) | ~570 |
| + one chart image at `frac 0.56` | ~530 |

**Write to these targets** (fill â‰ˆ 0.85 â€” clears the 0.75 gate with margin,
without spilling onto the next page):

| Section type | Target words |
|---|---|
| Prose-only section | **~700** |
| Chart analysis (Lagna / Chandra / Chalit / Navamsa / Ashtakvarga) | **~450** |
| Section with a 6â€“8 row `gridTable` | **~550** |
| Section with a 7-row `kvTable` | **~490** |
| à¤®à¥‚à¤² à¤µà¤¿à¤µà¤°à¤£ (16-row kvTable + 9-row gridTable) | **~160** |

**Vertical cost of non-prose elements**, in prose-word equivalents. Subtract
these from ~820 to get the word target for any page composition:

| Element | Cost |
|---|---|
| Chart image, `frac 0.56` | 290 |
| Chart image, `frac 0.44` (Chandra, square) | 230 |
| `gridTable` | ~25 per row (incl. header row) |
| `kvTable` | ~35 per row |
| `H2()` heading | ~30 |
| `LI()` / `LIB()` item | its own words + ~15 |

### Worked example

Sade Sati page: H1 + 3 H2 + a 7-row `gridTable`.
820 âˆ’ (3 Ã— 30) âˆ’ (8 Ã— 25) = **530 words**. Built at 588 â†’ fill 0.908. Correct.

### Notes

- Devanagari runs ~15% fewer words per line than Latin at the same point size.
  These figures are Devanagari-specific; do not reuse them for English output.
- Aim at 0.85, not 0.75. Landing on the gate leaves no room for the reflow that
  LibreOffice introduces during PDF conversion.
- Fill above 0.98 means the section is about to spill. Trim rather than let a
  section run three lines onto a fresh page.
- Writing to these targets on the first pass removes the expand-and-remeasure
  loop, which was the second-largest token cost of the Anshi run (4 passes).

## Section order

Cover â†’ à¤®à¥‚à¤² à¤µà¤¿à¤µà¤°à¤£ table + à¤µà¥ˆà¤¦à¤¿à¤• à¤¸à¤¾à¤° â†’ Part 1 divider â†’ Lagna, Chandra, Chalit,
Navamsa, Ashtakvarga â†’ Part 2 divider â†’ Vimshottari + Yogini dasha â†’
Part 3 divider â†’ Sade Sati, Mangal, Kalsarpa, Pitru/Grahan/Guru-Chandal,
Rajyoga â†’ Part 4 divider â†’ twelve life topics, transits, annual reports â†’
puja recommendations â†’ [optional Q&A divider + Q&A] â†’ personal guidance +
disclaimer â†’ contact page.


