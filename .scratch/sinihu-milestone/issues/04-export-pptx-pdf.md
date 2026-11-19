# Exporting a JSON document model to PPTX and PDF

Type: research
Status: resolved
Blocked by:

## Question

What are the options for turning a structured JSON deck into (a) PPTX (pptxgenjs, python-pptx, others) and (b) PDF (headless Chromium print or screenshot, others) in a self-hosted Node backend?

Which fidelity limits matter: editable text vs flattened images, fonts, charts and tables as native objects, variable-height content on fixed-size slides?

What does each option assume about the Card model (fixed canvas vs flowing)?

## Answer

Full findings: [../research/export-pptx-pdf.md](../research/export-pptx-pdf.md)

- **PPTX:** PptxGenJS (MIT, Node-native, v4.0.1, active) is the main option. It produces native editable text, native charts with an embedded xlsx, and native tables with heuristic auto-paging. It does not measure text and does not embed fonts in core. python-pptx (Python sidecar, template round-trip), docx4j/POI (JVM) and Aspose.Slides (commercial, JVM bridge, also exports PDF) are alternatives.
- **PDF:** headless Chromium print-to-PDF gives vector text with subset-embedded fonts and 16:9 pages through `@page size` / `preferCSSPageSize`. Screenshot-to-PDF is pixel-exact but not selectable. Gotenberg (Chromium + LibreOffice) is a ready-made Docker sidecar. Running Chromium in Docker needs the sandbox capability, bundled fonts and a sizeable image.
- **Fidelity:** editable PPTX is re-wrapped by PowerPoint's text engine, so line breaks and overflow can differ from the browser. Flattened images match exactly but cannot be edited. Native charts get PowerPoint styling.
- **Variable-height content on a 16:9 slide** can be scaled to fit, split at Element or row boundaries, left to overflow (PowerPoint applies `normAutofit`, not the generator), or prevented in the editor.
- **Card model:** every PPTX writer needs absolute boxes (a fixed canvas). Chromium PDF accepts flowing layout. Editable PPTX from a flowing or hybrid model therefore needs a headless-browser measurement pass (the dom-to-pptx pattern) that produces per-Element boxes, text styles, table metrics, chart data from the JSON, and screenshots for anything with no native mapping.
