# SiNiHu First-Release Specification

## Destination

An implementation-ready product and architecture specification for SiNiHu's first usable release, covering its core user workflows, domain behavior, security boundaries, and homelab deployment.

## Notes

- Domain: a family-operated slide-deck application hosted in a Docker homelab; Angular with Optimus UI; TypeScript backend; Copilot SDK is a first-class integration.
- Sessions should consult the `grilling` and `domain-modeling` skills; use other skills named by a ticket when applicable.
- Charting baseline: Microsoft, GitHub, and Google sign-in; invitation-only accounts with a one-time first-admin bootstrap code; each user connects their own Copilot entitlement and BYOK providers; personal model default; browser-local deck data with ZIP portability; one project per deck; AI edits are proposals requiring user approval; image generation is in scope.
- Keep projects private. Cross-account sharing and remote-access setup are not part of this effort. Assume a trusted browser/OS profile; do not claim browser-local account namespaces are a security boundary.
- Existing research notes: `docs/idea/copilot-sdk-multiuser-research.md` and `docs/idea/gamma-presentation-mode-research.md`.

## Decisions so far

<!-- Closed child tickets are indexed here; the decision detail lives only in each ticket. -->

- [Image Generation Provider Capabilities](issues/13-image-generation-provider-capabilities.md): Azure and OpenAI document image APIs; generic compatible endpoints require capability validation, and Copilot SDK has no documented native generation endpoint.
- [First Admin and Invitations](issues/01-first-admin-and-invitations.md): bootstrap, invitation, sign-in identity linking, account recovery and removal policies are specified, including the boundary around browser-local deck data.
- [Layouts, Templates, and Themes](issues/04-layouts-templates-and-themes.md): Deck templates provide starting content, structure, and backgrounds; layouts and reusable slides are reusable independently; themes control deck-wide colors and typography; personal reusable content stays private to its owner.

## Not yet specified

<!-- No additional in-scope fog identified yet. -->

## Out of scope

- Public remote access and configuration of Cloudflare Tunnels or Traefik; this will be handled separately if needed.
- Server-backed cross-account sharing, live co-editing, and cloud synchronization.
- Purchasing or assigning Copilot licenses; SiNiHu connects to each user's existing entitlement.