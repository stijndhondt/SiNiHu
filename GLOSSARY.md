# SiNiHu

SiNiHu is a household slide-deck authoring and presentation application. This glossary defines the product concepts used across its planning and implementation.

## People and Access

**Account**:
A person's identity in SiNiHu, associated with one or more sign-in methods and owning that person's projects and provider connections.
_Avoid_: Login, profile

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
A reusable structural arrangement of content on a slide.
_Avoid_: Template

**Template**:
A reusable starting point that supplies slide content or structure for a new deck or slide.
_Avoid_: Theme

**Theme**:
A set of visual styling choices that can be applied independently of a template's starting content.
_Avoid_: Template

## AI Work

**AI session**:
A user's scoped interaction with an AI provider for work on a project.
_Avoid_: Chat (when referring to the persisted interaction)

**Proposal**:
A candidate deck or slide change produced by AI and awaiting the user's review and explicit application.
_Avoid_: Update (unless the change has been applied)