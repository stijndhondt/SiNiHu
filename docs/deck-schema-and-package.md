# Deck schema and package

This document defines the canonical editable deck model and portable ZIP package contract shared by the manual editor, Deck AI, Slide AI, online presenter, and PPTX exporter. The machine-readable manifest schema is [deck.schema.json](../schemas/deck.schema.json). Terms use the vocabulary in [GLOSSARY.md](../GLOSSARY.md).

## Model invariants

- A deck contains an ordered array of active slides. Array order is presentation order.
- Each slide has a stable opaque ID and points to its current immutable slide revision.
- Each revision is a complete snapshot of one slide, not an edit delta. Its elements form an ordered flat array; array order is back-to-front paint order. Nested groups are not supported in schema version 1.
- Deck, slide, element, revision, and checkpoint IDs are stable opaque strings. Editing retains an element ID; creating or copying an element assigns a new ID.
- Slide geometry is expressed in inches in a deck-declared coordinate space. The default is 13⅓ × 7.5 in (16:9); custom positive dimensions are supported. An element's `x` and `y` locate its top-left corner, and `width` and `height` are positive.
- Core fields and element types are schema-defined. Unknown namespaced extension data is preserved verbatim, not interpreted. An unsupported core element type is rejected, not treated as an extension.

## Manifest and content

The ZIP root `deck.json` contains the package-format version, deck-schema version, and deck state. Schema version 1 is defined in [deck.schema.json](../schemas/deck.schema.json). The manifest has:

- `packageFormatVersion` and `deckSchemaVersion`, independent integer versions;
- a `deck` with identity, title, timestamps, slide dimensions, active ordered slides, retained revisions, and checkpoints;
- a current-revision reference on each active slide.

Each revision stores complete slide content: ordered elements and optional slide-level speaker notes. Every element has a stable ID, geometry, and typed style properties. The supported core types are:

| Type | Content |
| --- | --- |
| `text` | Structured paragraphs containing text runs. Runs support bold, italic, underline, and HTTPS hyperlinks; paragraphs support alignment and ordered/unordered lists. HTML is not accepted. |
| `image` | An image referenced by a package-relative asset path, with optional alt text and a decorative flag. |
| `shape` | A basic rectangle, ellipse, line, arrow, or triangle. |
| `table` | Editable rows and cells. Cell content uses the structured paragraph/run model and can have typed styling. |
| `chart` | Editable data-backed chart with categories and numeric series. Schema version 1 supports `bar`, `column`, `line`, and `pie`. |
| `externalResource` | A `video` or provider `embed` represented by a credential-free HTTPS URL and optional provider metadata. No binary media, arbitrary HTML, or scripts are embedded. |

The typed style vocabulary covers fill, stroke, opacity, font family, font size, font weight, color, alignment, and vertical alignment. Font size and stroke width are typographic points (`pt`), independent of the slide's inch-based geometry. Colors use sRGB hexadecimal `#RRGGBB` or `#RRGGBBAA` notation, with alpha as the final byte; `fill` additionally accepts `none`. Element opacity multiplies any color alpha. Arbitrary CSS is not part of the model. Optional alt text and an explicit decorative flag are supported for images and external resources; decorative assets need not have alt text.

Elements can include an `extensions` object with namespaced JSON values for future or application-specific data. Readers and writers that do not understand a namespace preserve its values unchanged. This does not permit adding unrecognized core element types.

## Revisions and checkpoints

- A committed edit transaction creates an immutable slide revision; keystrokes and no-op changes do not. A revision records its slide, creation time, source, complete content snapshot, and optional base revision. AI-authored revisions identify their AI session without storing prompts or responses.
- An edit supplies its expected base revision. A stale base is a conflict that must be surfaced to the caller; writers must not silently overwrite newer content.
- Each slide retains its latest 10 ordinary revisions. Revisions referenced by a checkpoint are pinned in addition to that rolling set. Removing a checkpoint requires an explicit user action; an unpinned revision outside the latest 10 may then be pruned.
- A manual checkpoint is created only on explicit user action and may have an optional user-provided label.
- At the end of each AI session that changed the deck, the system creates one checkpoint labeled `AI`. It captures the full ordered active-deck mapping from slide IDs to revision IDs, including slides the session did not change. It records checkpoint/session IDs and timestamps, not the AI conversation.
- A checkpoint is a full-deck state reference, not a copy of slide content. It may reference a revision for a slide that has since been removed from the active deck.

## ZIP package contract

A deck package is a ZIP archive containing exactly one manifest at the root and assets beneath the root `assets/` directory:

```text
deck.json
assets/
  <package-relative image files>
```

The package preserves current slides, retained slide revisions, checkpoints, and images referenced by current or retained revisions. Paths inside the archive use `/` separators regardless of the host operating system. Image references are relative paths beneath `assets/`; absolute paths, traversal outside that directory, and machine-local paths are invalid.

Video and provider embed resources are HTTPS references, not bundled binaries. Their URLs must have a host and may have a path, but must not contain user information, a query, or a fragment. This deliberately excludes signed URLs and query-based provider URLs; authentication belongs in runtime configuration or a separate sign-in flow. Credentials, cookies, AI prompts/responses, and executable HTML/scripts must never be serialized into the archive.

Before opening, the package validator checks the entire archive and manifest. It rejects malformed manifests, unsupported core types, missing or unsafe asset references, duplicate or ambiguous normalized entries, path traversal, and unsupported versions with actionable diagnostics. Image references must consist of non-empty slash-separated path segments beneath `assets/`, with a non-empty final filename; `.` and `..` segments and backslashes are invalid. It must not silently drop content, substitute missing assets, or partially load an invalid package. The validator rejects duplicate IDs within each reference scope: active slide IDs, revision IDs, and checkpoint IDs must each be unique within their respective collections. Each checkpoint must also contain at most one mapping for a given slide ID. These uniqueness checks are package-validator requirements; JSON Schema validation alone does not establish them. The validator checks revision and checkpoint references and verifies that active slide mappings resolve to revisions belonging to those slides.

## Versioning and compatibility

- `packageFormatVersion` describes the ZIP/container contract; `deckSchemaVersion` describes the manifest data model. They evolve independently.
- A reader applies explicit migrations for older deck schema versions it supports. Migrations preserve unknown namespaced extensions.
- An unsupported package format or newer unsupported deck schema is rejected clearly; best-effort interpretation is not a compatibility strategy.
- Changes to the manifest update the JSON Schema and document migration behavior. The migrated manifest is validated before acceptance.

## PPTX export

The exporter produces editable native PowerPoint objects where supported, including text, shapes, images, tables, and supported charts. External resources remain credential-free links where PowerPoint permits. Every conversion, degradation, or omission is reported per item; content is never silently discarded.

## Validation boundary

JSON Schema validates manifest structure, field types, supported element discriminators, and local constraints. Package validators MUST apply Draft 2020-12 format assertion when evaluating the schema's `date-time` and `uri` formats. A validator that cannot enable that vocabulary MUST perform equivalent RFC 3339 date-time and RFC 3986 URI validation and reject invalid values. It cannot prove that IDs resolve to the intended revision, checkpoint mappings are coherent, ZIP paths remain safe after normalization, archive entries are unique, or referenced files exist. These are required checks for the ZIP package validator, as specified above.
