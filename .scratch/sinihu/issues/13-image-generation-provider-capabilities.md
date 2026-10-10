# Image Generation Provider Capabilities

Type: research
Status: resolved

## Question

Using primary sources, determine which image-generation routes are supportable through the selected user-configured provider options: GitHub Copilot SDK, Azure-hosted models, and generic OpenAI-compatible endpoints. Document official request/response formats, authentication/configuration requirements, and important limitations; distinguish documented SDK capability from provider-specific availability. Recommend the smallest honest first-release support matrix and save cited findings in `docs/idea/image-generation-provider-research.md`, including this ticket path as context.

## Answer

The SDK documents image input and application-defined tools, not a native image-generation endpoint. Azure OpenAI and OpenAI each document image-generation APIs; a generic OpenAI-compatible endpoint must be validated for that specific capability. Recommendation: support Azure and OpenAI Images API routes, and enable generic endpoints for image generation only after validation. The cited findings and request/auth details are recorded in [Image Generation Provider Capabilities Research](../../../docs/idea/image-generation-provider-research.md).