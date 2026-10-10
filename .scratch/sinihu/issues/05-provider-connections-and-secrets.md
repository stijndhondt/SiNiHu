# Provider Connections and Secrets

Type: grilling
Status: resolved

## Question

Define the user-facing lifecycle for connecting, validating, replacing, refreshing, and deleting GitHub Copilot access, Azure-hosted BYOK models, and generic OpenAI-compatible BYOK endpoints. Specify which provider settings and secrets are required, how authentication expiry or revocation is handled, and how the UI communicates connection state without ever returning stored secret values. SiNiHu stores secrets server-side and does not include them in project archives.

## Answer

### Connections and settings

- Each Account explicitly authorizes its own GitHub Copilot connection, separately from any GitHub sign-in identity. SiNiHu uses the person's existing Copilot entitlement; license purchasing or assignment is out of scope.
- Azure BYOK connections require the user's endpoint, API key, and deployment or model identifier. Generic OpenAI-compatible BYOK connections require a base URL, API key, and model identifier; support only request formats SiNiHu has validated.
- An Account may have multiple named connections of each provider type. Names are unique among that Account's connections of the same provider type.
- Store provider secrets server-side. Never return or display stored secret values or fragments, and do not include secrets in project archives.

### Validation, replacement, and status

- A connection may be saved before it is tested. An explicit test must validate the configured endpoint, credential, and model or deployment together. Mark the connection Ready only after that test succeeds.
- A definitive credential or configuration rejection marks a connection Needs attention. A transient network or provider outage does not change an existing status. On a first test, a definitive rejection marks the connection Needs attention; a transient failure leaves it Not tested.
- Show the status, last successful test time, and sanitized actionable errors. Errors must not expose secret values or fragments.
- Test replacement credentials before applying them. A failed test preserves the existing credential and connection configuration.

### Renewal and deletion

- SiNiHu manages Copilot OAuth token renewal when the chosen authorization flow supports it. If renewal is unsupported or fails, or authorization is revoked, ask the user to reauthorize.
- Deleting a connection requires confirmation, removes its server-stored credentials and configuration, and prevents new provider calls. For Copilot, attempt provider-side authorization revocation when supported, but do not make local deletion depend on that attempt succeeding.
- When deleting an Azure or generic BYOK connection, tell the user to revoke the provider-issued API key separately; deleting the connection removes only SiNiHu's stored copy.
- Behavior for active or resumable AI sessions belongs to issue 06. Personal defaults and behavior when a connection is unavailable belong to issue 07.