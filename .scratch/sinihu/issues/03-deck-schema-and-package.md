# Deck Schema and Package

Type: grilling
Status: closed

## Question

Define the canonical editable deck and slide content model shared by the manual editor, Deck AI, Slide AI, online presenter, and PPTX exporter. Set the supported content structures, how per-slide revisions (latest 10, plus the stated AI-session and manual checkpoints) relate to a deck, and the versioning and compatibility rules for ZIP packages containing deck JSON and referenced image assets. Credentials must remain outside project archives.

## Resolution

The shared editable model is a versioned deck containing ordered slides. Each slide has a stable ID, slide-level speaker notes, and immutable full-content revisions. Each revision contains a flat ordered list of typed elements; list order is back-to-front paint order. Deck, slide, element, revision, and checkpoint IDs are stable opaque IDs. Editing preserves an element ID; creating or copying content assigns a new one.

Slides use declared dimensions in inches, defaulting to 13⅓ × 7.5 in (16:9), with custom positive dimensions allowed. Element positions and sizes use that coordinate space. Version 1 supports:

- text boxes with structured paragraphs and typed runs/marks for bold, italic, underline, HTTPS links, alignment, and ordered/unordered lists; no HTML;
- package-bundled images, basic shapes (rectangle, ellipse, line, arrow, triangle), editable tables, and data-backed bar, column, line, and pie charts;
- typed external video/provider-embed references using credential-free HTTPS URLs and metadata, without bundled video, arbitrary HTML, or scripts.

Styles are schema-defined (including fill, stroke, opacity, typography, and alignment), not arbitrary CSS. Namespaced extension values are preserved verbatim when unknown; unknown core element types are rejected. Images and external resources support optional alt text and an explicit decorative flag.

Each committed edit transaction creates a complete immutable slide revision; keystrokes and no-op changes do not. Writes include an expected base revision; stale writes are conflicts, not last-write-wins. Retain the latest 10 ordinary revisions per slide plus any revisions pinned by checkpoints. A manual checkpoint is created explicitly and may have a user label. Each AI session that changes the deck creates one full-deck checkpoint at session end, labeled `AI`. A checkpoint stores the ordered full-deck slide-to-revision mapping and metadata/session ID, not a conversation transcript. Removing a checkpoint requires explicit user action; revisions no longer pinned may be pruned when outside the latest 10.

The portable project package is a ZIP with one root `deck.json` and an `assets/` directory. It preserves current content, retained revisions, checkpoints, and referenced image files. Image paths are package-relative under `assets/`; external resources use credential-free HTTPS URLs. Credentials, cookies, tokens, prompts/responses, and executable embed code are never stored in the archive.

The manifest has independent integer `packageFormatVersion` and `deckSchemaVersion` fields. Readers explicitly migrate supported older schemas and clearly reject unsupported package formats or newer schemas. Validate the complete ZIP before opening: malformed manifests, unsupported core content, unsafe or missing asset references, duplicate/ambiguous paths, and broken revision/checkpoint references fail with actionable diagnostics and no partial or silent load. PPTX export uses editable native objects where supported, preserves external links where possible, and reports each degradation or omission.

The decisions are documented in `docs/deck-schema-and-package.md` and machine-readable `schemas/deck.schema.json`; canonical terms are in `GLOSSARY.md`, and the durable model/package choice is recorded in `docs/adr/0001-canonical-deck-model-and-package.md`.