# The Card model: flowing, fixed canvas or hybrid

Type: grilling
Status: resolved
Blocked by: 04, 05

## Question

Is a Card a flowing, variable-height block (Gamma-style), a fixed-aspect canvas with absolutely positioned Elements (PowerPoint-style), or a hybrid (e.g. flowing layout regions plus optional free-positioned Elements)?

The answer has to support drag/resize/reposition, Present mode, PDF and PPTX export, and later formats (documents, web pages).

Inputs: [Exporting a JSON document model to PPTX and PDF](04-export-pptx-pdf.md), [Gamma as reference behaviour](05-gamma-reference.md).

## Answer

Recorded as [ADR 0001](../../../docs/adr/0001-card-layout-regions-fixed-frame.md). Terms are in `GLOSSARY.md` (Frame, Layout, Region, Element, Compound Element, Free Element).

1. **Frame:** every Card has a Frame. The milestone ships only fixed 16:9, and the model already allows fluid and other ratios.
2. **Placement:** a Card uses a Layout of named Regions, and Elements flow in order inside each Region. The AI fills Regions and never produces coordinates. A Free Element sits at an absolute position relative to the Frame, remembers its home Region and can snap back.
3. **Overflow:** text shrinks automatically down to the Theme's minimum size, then the Card shows an overflow warning offering Split, Switch Layout or Shorten. Cards are never split silently. Generation is told to stay within each Region's capacity.
4. **Geometry:** all geometry is relative to the Frame. Layouts declare which Frames they support.
5. **No nesting:** Regions are one level deep. Structured content is a Compound Element.
6. **Export:** a headless-Chromium render service reuses the editor's Card renderer.
   - **PDF:** print to PDF.
   - **PPTX:** measure Element boxes in that render, then write with PptxGenJS. Text, shapes, images and tables are native objects.
   - **Images:** charts and anything else with no PowerPoint equivalent become images in the milestone.
7. **Present mode:** the Frame scales to fit the screen, letterboxed, with no scrolling.
