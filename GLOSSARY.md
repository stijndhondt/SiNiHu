# SiNiHu

SiNiHu is a household slide-deck authoring and presentation application. This glossary defines the product concepts used across its planning and implementation.

## People and Access

**Account**:
A person's identity in SiNiHu, associated with one or more sign-in methods and owning that person's projects, reusable content, and provider connections.
_Avoid_: Login, profile

**Account-local data**:
Projects and reusable content held in a browser and associated with a SiNiHu Account. This association is organizational and does not make same-origin browser storage a security boundary.
_Avoid_: Account storage (which may imply isolation or protection)

**Unassigned local content**:
Browser-held projects or reusable content that has not been associated with a SiNiHu Account.
_Avoid_: Orphaned content

**Sign-in identity**:
An identity from an external provider, such as Microsoft, GitHub, or Google, that a person uses to authenticate to a SiNiHu Account.
_Avoid_: Provider connection

**Invitation**:
A single-use authorization for a person to create a SiNiHu Account, bound to a verified email address and issued by an admin.
_Avoid_: Invite link

**Bootstrap code**:
A one-time code used to establish the first admin Account in a SiNiHu deployment.
_Avoid_: Setup password

**Account deactivation**:
A reversible suspension of an Account that disables access while preserving the Account and its linked sign-in identities.
_Avoid_: Account deletion

**Account deletion**:
The irreversible removal of a SiNiHu Account and its server-managed identity and session data.
_Avoid_: Deactivation

**Provider connection**:
A user's authorization or configuration for using an AI provider, such as GitHub Copilot or a BYOK model service.
_Avoid_: License, API key (unless referring specifically to the secret)

**Sharing**:
An explicit grant that lets another account access a project or reusable asset.
_Avoid_: Collaboration (unless simultaneous editing is meant)

## Deck Authoring

**Project**:
A user's work container for one deck and its related content.
_Avoid_: Workspace, presentation (when referring to the editable project)

**Deck**:
An ordered collection of slides prepared for online presentation or export.
_Avoid_: Project (when referring only to slide content)

**Slide layout**:
A reusable arrangement of content on a slide, including fixed placements and structured objects that reflow as their contents change.
_Avoid_: Template

**Reusable slide**:
A saved, populated slide that an owner can insert into another deck, preserving its original styling or adopting the destination deck's theme.
_Avoid_: Slide template

**Template**:
A reusable deck-level starting point that may provide slide content, structure, and backgrounds, and may contain only one slide.
_Avoid_: Theme

**Theme**:
A set of colors and typography that can be applied across a deck independently of its template, content, structure, and backgrounds.
_Avoid_: Template

## AI Work

**AI session**:
A user's scoped interaction with an AI provider for work on a project.
_Avoid_: Chat (when referring to the persisted interaction)

**Proposal**:
A candidate deck or slide change produced by AI and awaiting the user's review and explicit application.
_Avoid_: Update (unless the change has been applied)