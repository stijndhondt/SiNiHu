# Define the canonical deck model and package

Status: resolved

## Problem

The manual editor, Deck AI, Slide AI, online presenter, and PPTX exporter need a shared editable deck model and portable package contract.

## Acceptance criteria

- Define the manifest, slide and element model, revision history, checkpoints, and package format.
- Provide a machine-readable schema and document package validation and compatibility requirements.

## Resolution

The canonical model and package contract are documented in [deck-schema-and-package.md](../../../docs/deck-schema-and-package.md), with the manifest schema in [deck.schema.json](../../../schemas/deck.schema.json) and the architectural decision in [ADR 0001](../../../docs/adr/0001-canonical-deck-model-and-package.md).
