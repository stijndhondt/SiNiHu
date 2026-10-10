# Image Generation Provider Capabilities

Research date: 2026-10-06

Context: [Image Generation Provider Capabilities](../../.scratch/sinihu/issues/13-image-generation-provider-capabilities.md)

## Findings

- **GitHub Copilot SDK:** The documented SDK feature is image input for vision-capable models. The SDK also permits application-defined tools, so SiNiHu could expose its own image-generation operation to an agent; that would be SiNiHu's integration with a provider API, not a native Copilot SDK image-generation endpoint. [Image input](https://docs.github.com/en/copilot/how-tos/copilot-sdk/features/image-input) [SDK feature overview](https://docs.github.com/en/copilot/how-tos/copilot-sdk/features) [SDK repository and custom-tool overview](https://github.com/github/copilot-sdk)
- **Azure OpenAI / Microsoft Foundry:** Microsoft documents image generation through an Images API. The current guide shows `POST /openai/v1/images/generations?api-version=preview`, a deployed image model in the `model` field, API-key or Microsoft Entra authentication, and base64 image data in `data[].b64_json`. Availability depends on the selected model, deployment, region, access requirements, and quota. [Image generation guide](https://learn.microsoft.com/en-us/azure/foundry/openai/how-to/dall-e)
- **OpenAI API:** The Images API documents `POST /v1/images/generations` with bearer-token authentication, a model and prompt, and base64 image data in the response. The API also documents image editing. [Image generation guide](https://developers.openai.com/api/docs/guides/image-generation) [Create image API reference](https://developers.openai.com/api/reference/resources/images/methods/create)
- **Generic OpenAI-compatible endpoints:** “OpenAI-compatible” alone does not establish support for the Images API. An endpoint must specifically implement the image-generation route and the request/response fields SiNiHu uses; unsupported options and model availability vary by server.

## Recommendation

Treat image generation as a separate provider capability from Copilot chat. For a first release, support Azure's documented image API and OpenAI Images API routes, and advertise a generic compatible endpoint as image-capable only after its configuration has been validated. Make image requests through the backend so users' provider secrets are not exposed to the browser. Keep model names, access, quota, pricing, and supported parameters configurable rather than assuming they are uniform or permanent.