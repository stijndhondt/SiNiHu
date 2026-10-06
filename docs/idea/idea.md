# SiNiHu

My own version of an ai assisted slidedeck building application (think Gamma.app). own use running on a docker container
i want a web frontend in angular with optimus ui. and a backend in typescript or c# or Go. Copilot SDK integration is a first-class part of the application, not an optional add-on.

## Initial product decisions

- Use IndexedDB as the initial browser store for editable deck/project data and per-slide revisions. Provide a portable ZIP import/export containing the deck JSON and its referenced image assets. Keep the latest 10 revisions per slide for rollback. Create one rollback version when an AI chat session begins; group subsequent AI requests in that session as the same logical change. Provide a manual Save Version action for an in-between checkpoint. Consider Storage Buckets later if separating or managing the lifecycle of larger asset collections becomes useful. Do not include OAuth, Copilot, or BYOK credentials in project archives.
- Treat provider credentials separately from deck data. Let users add, replace, or delete credentials without ever returning stored credential values to the UI. Use HTTPS in transit and standard server-side encryption at rest with encryption keys kept outside the source repository and project archives. If the SDK runs on the backend, credentials must be decrypted in backend memory when used; do not build a custom rolling cipher. Avoid storing long-lived provider secrets in browser session storage.
- Verify the Copilot SDK's multi-user/session capabilities before choosing the backend integration topology. Regardless of SDK support, SiNiHu must isolate each app user's projects, AI sessions, model configuration, and credentials.
- Keep projects and reusable assets private by default. Sharing must be an explicit user action with clear visibility into who has access; enforce ownership and share permissions in SiNiHu rather than relying on Copilot SDK sessions or IndexedDB boundaries.
- Support manual authoring as well as two chat-like AI modes, similar to Copilot Chat. **Deck AI** creates or updates a deck from deck-level guidance and slide structure. **Slide AI** creates or updates the selected slide from slide-specific instructions and context. Both use the Copilot SDK and translate user information into the app's structured visual language for editing, presentation, and export.
- AI changes use proposal mode: show the proposed deck or slide changes for review and require the user to apply them, rather than silently overwriting manual work.
- Treat basic slide layouts as first-class building blocks shared by the manual editor, Deck AI, and Slide AI. Users can choose a layout when adding a slide; AI proposals should use the same structured layouts so the result remains editable in the editor. Templates can provide starting content and styling on top of those layouts.
- Provide an online presentation mode that allows scrolling through long slide content. Export a PPTX in a Gamma-like way without automatically splitting overflowing content across slides. The user can adjust the exported deck in PowerPoint; SiNiHu does not track those later edits.

This is the documentation for choosing a path

https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/choosing-a-setup-path



| What you need                                     | Guide                                                                                                                   |
| ------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Getting started quickly                           | [Default setup (bundled CLI)  ](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/bundled-cli)               |
| Use your own CLI binary or server                 | [Local CLI setup](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/local-cli)                               |
| Users sign in with GitHub                         | [GitHub OAuth setup](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/github-oauth)                         |
| Use your own model keys (OpenAI, Azure, and more) | [BYOK (bring your own key)](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/azure-managed-identity)        |
| Azure BYOK with Managed Identity (no API keys)    | [Azure managed identity with BYOK](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/azure-managed-identity) |
| Run the SDK on a server                           | [Backend services setup](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/backend-services)                 |
| Configure SDK options for concurrent users        | [Multi-tenancy and server deployments](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/multi-tenancy)      |
| Serve multiple users / scale horizontally         | [Scaling and multi-tenancy](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/scaling)                       |


This is used be al people in my house (family so multiuser). we deploy inside a homelab (docker) all management should be done with the angular webinterface. user log in with miscorosft, github, google.
They then see their custom Portal/Dashboard with projects they have created, reusable slides / templates. and here they manage the copilot license, the byok and models.

You can see a POC for a dashboard start in D:\Projects\SiNiHuOld\studio

we store as much as possible inside the browser(be carefull about credentials for copilot itself here)
Each user should be able toprovide there own BYOK (own models or azure models and github copilot license).

We can create slides, rearagne slides, inject slides between, generate images, select images form the pc, or from an url.
a helper prompt for image generation.
```
You generate images that visually support the user's content.

For each image:
- Identify the main subject, key idea, audience, and tone from the surrounding content.
- Create an image that makes that specific idea easier to understand or feel. Prefer concrete, relevant imagery over generic decoration.
- Match the image's mood, lighting, color palette, and level of realism to the content and the app's visual theme.
- Use a clear focal point and a composition that fits the requested placement and aspect ratio. Leave negative space when text may appear beside or over the image.
- Keep a consistent visual direction across images in the same document, while varying compositions enough to avoid repetition.
- Do not include captions, typography, logos, watermarks, UI mockups, or unrelated decorative elements unless explicitly requested.
- Avoid clichés and generic stock-photo scenes. Do not invent factual details about real people, organizations, or events.

Return a concise image-generation prompt describing the subject, setting, composition, lighting, palette, and style. Also return brief alt text that describes the image's relevant meaning.
```

when creating a slide deck with the ai the user provides two pieces of info

## 1 A general guidance

Steering information on what we want to do set overal style and approach. 
Things to avoid things to include

See the generalGuidance.md file



## 2 and a slide structure in markdown. each --- denotes a new slide

A structure in markdown that details thigns about what that slide should tell. this is not a one on one copy but this is specific guidance about that slide. what do we want to tell with that slide. is thjere speicific visuals we want to include( next to the genral image guideline)

See the slideExample.md file

