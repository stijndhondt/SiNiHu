# Research: exporting a JSON deck to PPTX and PDF

Ticket: [../issues/04-export-pptx-pdf.md](../issues/04-export-pptx-pdf.md)
Date: 2026-11-19
Scope: options for a self-hosted Node (NestJS) backend that turns a structured JSON deck (Cards of Elements: text, images, shapes, charts, tables) into PPTX and PDF, the fidelity limits, and what each option assumes about the Card model.

Facts not checked against a primary source are marked **UNVERIFIED**.

---

## (a) PPTX generation from Node

### 1. PptxGenJS (pure JS, runs in Node and browser)

- **Repo / license:** https://github.com/gitbrent/PptxGenJS. MIT, "Copyright © 2015-present Brent Ely". About 6.4k stars and 3.4k commits. The latest release is **4.0.1** (releases page shows "26 Jun" and does not show the year; 4.0.0 was "04 May"): https://github.com/gitbrent/PptxGenJS/releases. It is actively maintained.
- **Runtime:** "dual ESM and CJS builds". It outputs base64, Blob, Buffer or Node stream. JSZip is its only dependency (README).
- **Geometry:** every object takes `x, y, w, h` in **inches**, or as a percentage string of the slide: https://gitbrent.github.io/PptxGenJS/docs/api-text/. The library is **absolute-position only** and has no layout engine.
- **Text:** produces native, editable text boxes. It supports runs with mixed formatting, bullets, `indentLevel`, `paraSpaceBefore/After`, `lineSpacing` / `lineSpacingMultiple`, `fontFace`, `fit: none | shrink | resize` and `autoFit` ("Fit to Shape"). It also supports RTL and Asian fonts (README).
  - `fit: 'shrink'` only writes the OOXML autofit flag. PowerPoint does the actual shrinking, because the spec stores the result as `<a:normAutofit fontScale lnSpcReduction>` values that the authoring app computes: https://learn.microsoft.com/en-us/dotnet/api/documentformat.openxml.drawing.normalautofit. PptxGenJS does not measure text. **UNVERIFIED:** whether PowerPoint recomputes the scale on open or only when the text is edited. Reports conflict, so test it.
- **Charts:** produces **native** DrawingML charts. `gen-charts.ts` writes `<c:chartSpace xmlns:c=".../drawingml/2006/chart">` and builds a per-chart embedded workbook at `ppt/embeddings/Microsoft_Excel_Worksheet<id>.xlsx`, linked through `<c:externalData>`: https://raw.githubusercontent.com/gitbrent/PptxGenJS/master/src/gen-charts.ts. Users can edit the data in PowerPoint. Supported types are area, bar, bar3d, bubble, bubble3d, doughnut, line, pie, radar, scatter, plus combo charts: https://gitbrent.github.io/PptxGenJS/docs/api-charts/.
- **Tables:** produces native tables with `colspan`/`rowspan`, `colW`, and `rowH` per row. If row heights are omitted, "rows take only the space their contents need" (PowerPoint sizes them).
  - `autoPage: true` creates new slides "as tables overflow", with `autoPageRepeatHeader` and `newSlideStartY`. The docs say auto-paging is imprecise and needs tuning through `autoPageCharWeight` / `autoPageLineWeight`: https://gitbrent.github.io/PptxGenJS/docs/api-tables/.
  - `tableToSlides(id)` converts an HTML `<table>` into slides (README).
- **Images / media:** supports PNG/JPG, SVG, animated GIF and YouTube embeds (README).
- **Fonts:** sets `fontFace` by name only. **There is no font embedding in core.** Embedding has been discussed in PR #1302 (mirror: https://gh.r-mj.com/gitbrent/PptxGenJS/pull/1302), and a third-party wrapper exists: https://github.com/liyao1520/pptx-embed-fonts (**UNVERIFIED** quality). Without embedding, the viewer substitutes a font when it isn't installed.
- **Masters:** supports custom Slide Masters (`defineSlideMaster`).

### 2. python-pptx via a sidecar (Python process or microservice)

- **Repo / license:** https://github.com/scanny/python-pptx. MIT, latest version **1.0.2** (2024-08-07), Python ≥ 3.8: https://pypi.org/project/python-pptx/. Releases are infrequent (the last one is over two years old).
- **Capabilities:** reads and writes (round-trips) PPTX, so it can **start from a .pptx template** and fill placeholders. It supports text boxes, tables, pictures, and charts that are "a graphical (DrawingML) object", with "most chart types other than 3D": https://python-pptx.readthedocs.io/en/latest/user/charts.html. **UNVERIFIED** from docs: whether the embedded chart workbook is written with XlsxWriter (the source suggests it is).
- **Text fitting:** `TextFrame.fit_text()` computes a best-fit point size using a **TrueType font file** for measurement (`font_file`, or an installed font). It then writes explicit run sizes and sets `auto_size = NONE`: https://python-pptx.readthedocs.io/en/latest/api/text.html, https://python-pptx.readthedocs.io/en/latest/dev/analysis/txt-fit-text.html. This is the one option here that measures text itself, though only roughly.
- **Fonts:** no embedding API (**UNVERIFIED**: not documented).
- **Geometry:** EMU/Length, absolute positioning. Same canvas assumption as PptxGenJS.
- **Cost for SiNiHu:** a second language runtime plus IPC/HTTP. You would choose it mainly for template round-tripping.

### 3. JVM / commercial: docx4j (pptx4j), Apache POI, Aspose.Slides

- **docx4j / pptx4j:** Apache 2.0 Java library "for creating and manipulating Microsoft Open XML (Word docx, Powerpoint pptx, and Excel xlsx) files": https://www.docx4java.org/trac/docx4j. It is low level (JAXB over OOXML) and would need a JVM sidecar. **UNVERIFIED:** current maintenance cadence.
- **Aspose.Slides for Node.js via Java:** commercial. It runs on a JVM through a `java` bridge that npm compiles at install time (needs a JDK, Python and a C++ toolchain). Unlicensed output gets an evaluation watermark. It reads and writes PPT/PPTX/ODP and **exports to PDF, SVG, HTML and images**, so one engine could do both formats: https://docs.aspose.com/slides/nodejs-java/. **UNVERIFIED:** pricing, and its font-embedding API (not on that page).
- Apache POI (XSLF) is another Apache-2.0 JVM option (**UNVERIFIED**; not researched further).

### 4. HTML/DOM to PPTX (measure the rendered DOM, emit native objects)

- **dom-to-pptx** (MIT, built on PptxGenJS): https://github.com/atharva9167j/dom-to-pptx.
  - It walks the DOM, reads computed styles, and does not interpret flex or grid. Instead it "measures each element's final x, y, width, height relative to the slide root" and places each one absolutely. It recommends authoring at 1920×1080 and scaling to 16:9.
  - It emits native text boxes (with mixed-run rich text), shapes and shadows. Gradients become SVG. SVG is rasterised unless `svgAsVector`. Pseudo-elements are rendered natively or through an html2canvas snapshot.
  - `autoEmbedFonts` embeds `@font-face` fonts; when CORS blocks a font it falls back to Arial.
  - It runs in the browser, and a headless CLI (`dom-to-pptx export`) exists.
  - Its changelog says it uses `getBoundingClientRect()` coordinates (https://cdn.jsdelivr.net/npm/dom-to-pptx@2.1.1/CHANGELOG.md).
  - It is a small, single-maintainer project (**UNVERIFIED**: long-term maintenance).
- Similar tools: `html-to-pptx`, `llm-dom-to-pptx` (https://cdn.jsdelivr.net/npm/llm-dom-to-pptx@1.2.7/README.md) and GX-Alex/html2pptx (https://github.com/GX-Alex/html2pptx). All of them are **UNVERIFIED** for quality.
- **The pattern matters more than the library:** render in Chromium, collect each element's box and computed style, then emit absolutely-positioned native objects. SiNiHu can implement this pattern itself against its own Element types instead of generic DOM, and still use PptxGenJS as the writer.

### 5. Flattened PPTX (one image per slide)

- Screenshot each Card (see b) and place it as a full-bleed picture with PptxGenJS. The result matches the screen exactly and nothing is editable. It is a common fallback mode.

### Summary table (PPTX)

| Option | Native editable text | Native charts | Native tables | Font embedding | Layout input it needs | License / runtime |
|---|---|---|---|---|---|---|
| PptxGenJS | yes | yes (with embedded xlsx) | yes, plus imprecise autoPage | no (core) | absolute x/y/w/h in inches | MIT, Node |
| python-pptx | yes | yes (no 3D) | yes | no (UNVERIFIED) | absolute EMU; `fit_text` needs a TTF | MIT, Python sidecar |
| docx4j / POI | yes | yes (low level) | yes | UNVERIFIED | absolute | Apache 2.0, JVM |
| Aspose.Slides | yes | yes | yes | UNVERIFIED | absolute | commercial, JVM bridge |
| DOM to PPTX (dom-to-pptx or in-house) | yes | only if mapped to PptxGenJS charts; otherwise image/SVG | yes, if mapped | dom-to-pptx: yes (@font-face) | measured boxes from a browser render | MIT, browser or headless |
| Flattened images | no | no | no | n/a (pixels) | screenshot | any |

---

## (b) PDF generation

### 1. Headless Chromium print-to-PDF (Puppeteer `page.pdf()` / Playwright / CDP `Page.printToPDF`)

- **Options (Puppeteer):** `format` (overrides width and height), `width`/`height` (number or unit string), `preferCSSPageSize` ("Give any CSS `@page` size declared in the page priority"), `printBackground` (default false), `omitBackground`, `scale` (0.1–2), `pageRanges`, `tagged` (default true; accessible PDF, experimental), `outline` (experimental): https://pptr.dev/api/puppeteer.pdfoptions. The CDP equivalent takes `paperWidth`/`paperHeight` in inches plus `preferCSSPageSize`, `generateTaggedPDF` and `generateDocumentOutline`: https://chromedevtools.github.io/devtools-protocol/tot/Page/#method-printToPDF.
- **Playwright:** uses print media by default and needs `emulateMedia({media:'screen'})` for screen styles. It also needs `-webkit-print-color-adjust` to keep exact colours: https://playwright.dev/docs/api/class-page#page-pdf. PDF export works **only in Chromium**: non-Chromium engines throw "PDF generation is only supported for Headless Chromium" (https://github.com/microsoft/playwright-python/issues/2909).
- **Page sizing:** CSS `@page { size: <w> <h> }` with absolute lengths sets the page box (https://developer.mozilla.org/en-US/docs/Web/CSS/@page/size). A 16:9 deck uses 13.333in × 7.5in, the PowerPoint widescreen size, with `preferCSSPageSize: true` or explicit `width`/`height`. You force one Card per page with `break-after: page`. Print **paginates flowing content**: a Card taller than the page splits across pages unless it is constrained (`break-inside: avoid` only helps when the content fits).
- **Vector text and fonts:** Chrome's PDF output goes through Skia's PDF backend. It embeds fonts, subsets TrueType with HarfBuzz ("Chrome uses the HarfBuzz subsetter"), and uses glyph-ID encoding. Non-TrueType fonts and fonts flagged do-not-embed become Type3: https://skia.org/docs/dev/design/pdftheory/. So text is selectable and searchable. CSS effects such as filters, `backdrop-filter` and some blends may be rasterised (**UNVERIFIED**; test against SiNiHu themes).
- **Charts:** an SVG chart library gives vector charts in the PDF. A canvas-based one (e.g. Chart.js default) gives bitmaps.

### 2. Screenshot to PDF

- `page.screenshot({clip, type, quality, omitBackground})` per Card (https://pptr.dev/api/puppeteer.screenshotoptions), with the viewport `deviceScaleFactor` set for resolution. You then assemble the images into a PDF (e.g. with pdf-lib, **UNVERIFIED** choice). The output is pixel-identical to the editor, has no selectable text, produces large files, and gets blurry when zoomed. Its one benefit is that it avoids print-mode differences.

### 3. Alternatives

- **Gotenberg:** MIT, "A Docker-based API for PDF". It wraps headless Chromium (HTML/URL/Markdown) and LibreOffice (office documents): https://github.com/gotenberg/gotenberg. It is a ready-made sidecar for both paths. One route is to generate PPTX first and then convert it to PDF with LibreOffice, so PDF and PPTX come from one source. The trade-off is that LibreOffice's PPTX rendering fidelity limits the PDF (**UNVERIFIED** for SiNiHu content).
- **Aspose.Slides** PPTX to PDF (commercial; see a.3).
- Programmatic PDF libraries (pdfkit, pdf-lib, react-pdf) would need a second layout implementation. They are not researched further (**UNVERIFIED**).

### Docker cost (Chromium)

- Puppeteer publishes `ghcr.io/puppeteer/puppeteer` (Chrome for Testing plus dependencies). It needs `--cap-add=SYS_ADMIN` for the sandbox and `--init`: https://pptr.dev/guides/docker. The troubleshooting guide installs `fonts-freefont-ttf` and others "to support major charsets". It runs as a non-root user "so we don't need --no-sandbox", and says no-sandbox is "strongly discouraged": https://github.com/puppeteer/puppeteer/blob/main/docs/troubleshooting.md.
- Practical costs (**UNVERIFIED**, general experience): an image of roughly 1 GB or more, a few hundred MB RAM per browser, small `/dev/shm` problems (`--shm-size` or `--disable-dev-shm-usage`), and the need to bundle **every theme font** in the image or load it through `@font-face`. A pool of reusable browser instances keeps latency down.

---

## (c) Fidelity trade-offs

- **Editable versus flattened PPTX.** Native objects can be edited in PowerPoint and Google Slides, but they are re-rendered by **PowerPoint's** text engine. Line breaks, kerning and line height then differ from Chromium, so text can wrap differently and overflow its box. Flattened images look exactly right and are useless for editing. A hybrid is common: native text, tables and charts, with decorative elements (complex SVG, CSS effects, gradients) rasterised.
- **Fonts.**
  - PDF: Chromium embeds subsets automatically.
  - PPTX: PptxGenJS core cannot embed fonts. PowerPoint embedding requires TTF/OTF whose licence embedding level permits it ("Non-embeddable, Preview/Print, Editable, Installable"). The macOS and Windows desktop apps support it (https://support.microsoft.com/en-us/office/benefits-of-embedding-custom-fonts-cb3982aa-ea76-4323-b008-86670f222dbc).
  - Without embedding, themes should map to fonts likely to be installed, or accept substitution.
- **Charts.** Native PPTX charts (PptxGenJS, python-pptx) are editable but styled by PowerPoint. The browser chart library's look (animations, custom labels) will not match exactly. Exporting the chart as SVG or an image keeps the look and loses editability.
- **Tables.** Native table row heights are decided by PowerPoint at render time. PptxGenJS `autoPage` guesses at overflow using character-weight heuristics.
- **Variable-height (flowing) content on a fixed 16:9 slide.** There are four strategies:
  1. **Scale to fit:** measure the rendered Card height H. When H is greater than the 16:9 height, apply the uniform scale 1080/H. In PDF this is a CSS transform. In PPTX every box and font size is multiplied, so fonts can become tiny.
  2. **Split across slides:** break at Element boundaries (or table rows) wherever the cumulative height exceeds the slide. This needs measured heights. It is what print pagination does automatically and what PptxGenJS `autoPage` approximates for tables.
  3. **Overflow:** keep a fixed canvas and let content clip, or rely on OOXML `normAutofit`. Text shrink is applied by PowerPoint and not by the generator (`fontScale` / `lnSpcReduction` are stored values).
  4. **Prevent at authoring time:** the editor constrains Cards to a 16:9 frame (fixed canvas) or warns on overflow, as Gamma-style "fit to slide" does (**UNVERIFIED** how Gamma does it).

---

## (d) What each option assumes about the Card model

| Option | Assumes | Fit with a flowing Card |
|---|---|---|
| PptxGenJS, python-pptx, docx4j/POI, Aspose (writers) | **Fixed canvas, absolute boxes** (inches or EMU) for every object; no layout engine | A layout pass must first produce boxes |
| DOM to PPTX (dom-to-pptx or in-house) | Any HTML/CSS layout, converted by **measuring the browser render** | Natural fit; the browser is the layout engine |
| Chromium print-to-PDF | Flowing HTML, paginated by print rules | Natural fit, but splitting and scaling need explicit CSS |
| Screenshot to PDF / PPTX | Any rendered pixels | Natural fit; no editability |
| LibreOffice / Aspose PPTX to PDF | Whatever PPTX was produced | Inherits the PPTX layout |

**Producing editable PPTX from a flowing model** requires SiNiHu to compute a set of **measured layout boxes** per Card, typically in headless Chromium using the same Angular renderer and theme CSS. The list of what to compute:

1. **Card frame:** the rendered Card at a fixed reference width (e.g. 1920px for 16:9) and its natural height.
2. **Fit decision:** scale factor, split points, or both (see c), applied before or after measuring.
3. **Element boxes:** `getBoundingClientRect()` for each Element, relative to the Card, converted px → inches (13.333in / reference width).
4. **Text runs:** computed font family, size, weight, colour, line-height and alignment per run or paragraph. Optionally also the browser's line breaks. These can be emitted as explicit line breaks or a slightly enlarged box, so PowerPoint's different wrapping doesn't overflow.
5. **Tables:** column widths and row heights from the render.
6. **Charts:** the chart's box plus its **source data** from the JSON model (not the DOM), so it can be emitted as a native chart. Otherwise use an SVG or image snapshot.
7. **Rasterised fallbacks:** screenshots clipped to the boxes of elements that have no native mapping (effects, complex SVG, custom components).
8. **Z-order and backgrounds:** paint order and Card background or theme.

With a **fixed-canvas** model, steps 1–3 come straight from the stored JSON (x, y, w, h already exist), and only text wrapping can still differ between the browser and PowerPoint. With a **hybrid** model (flowing layout inside fixed 16:9 frames or regions), the measured boxes are bounded by the frame, so overflow becomes an authoring-time concern instead of an export-time one.

---

## Implications for SiNiHu

These are facts and options only. The decision belongs to the grilling ticket "The Card model: flowing, fixed canvas or hybrid".

- **PDF is largely model-agnostic.** Headless Chromium print-to-PDF with `@page { size: 13.333in 7.5in }` gives vector text and subset-embedded fonts from the same renderer the editor uses. The remaining questions are what to do with Cards taller than 16:9 (scale, split or clip), and running Chromium in Docker (image size, sandbox capability, bundled fonts).
- **Editable PPTX is not model-agnostic.** Every PPTX writer (PptxGenJS, python-pptx, docx4j, Aspose) needs absolute boxes.
  - A fixed-canvas model supplies these boxes directly.
  - A flowing model needs a browser measurement pass that produces boxes, text styles, table metrics and fallback screenshots. That is the dom-to-pptx pattern, which SiNiHu could reimplement against its own Element types.
  - A hybrid model needs the same measurement pass, but the frame bounds its output.
- **PptxGenJS (MIT, Node-native, active, v4.0.1)** is the natural writer for a Node backend. It provides native editable text, native charts with embedded xlsx, native tables (with heuristic auto-paging), images and SVG. Its gaps are no text measurement and no core font embedding.
- **Text reflow mismatch** between Chromium and PowerPoint exists under every model. Overflow in editable PPTX is a fidelity risk even when boxes are exact. The mitigations are padding, explicit line breaks, `fit: 'shrink'`/`normAutofit` (applied by PowerPoint, not by the generator), or a font set that is safe to embed or likely installed.
- **Charts:** emitting native PPTX charts means generating them from the JSON chart data, not from the browser rendering. Their styling will differ from the in-app chart library. The alternative is SVG or image charts, which keep the look and lose data editing.
- **Escape hatches under any model:** flattened image PPTX, and a single-source "PPTX first, then LibreOffice or Gotenberg to PDF" pipeline. The second trades browser fidelity for consistency between the two formats.
