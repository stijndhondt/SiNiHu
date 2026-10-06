# Copilot SDK Multi-User Boundary

**Research date:** 2026-10-06  
**Scope:** Angular/browser-local slide app; GitHub Copilot SDK in a backend; family users and explicit sharing.

## Conclusion

The SDK supports concurrent independent sessions on a shared runtime, with per-session state and optional per-session GitHub credentials. It does not provide SiNiHu's user authentication, authorization, project ownership, sharing rules, credential lifecycle, or locking for concurrent access to the same session. IndexedDB gives SiNiHu local structured storage and transaction coordination, but not separation between accounts logged into the same app origin, nor synchronization/sharing across browsers. [GitHub multi-tenancy](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/multi-tenancy) [GitHub scaling](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/scaling) [IndexedDB 3.0](https://w3c.github.io/IndexedDB/#database-connection) [Storage Standard](https://storage.spec.whatwg.org/#storage-keys)

## Verified Facts

### SDK sessions, tenancy, and concurrency

- GitHub documents three deployment patterns: one CLI/runtime per user for strongest isolation; a shared runtime with unique session IDs for logical session isolation; and multiple users sharing one session for collaboration. It characterizes the shared runtime as lighter but weaker isolation than per-user runtimes. [Scaling: session isolation patterns](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/scaling#session-isolation-patterns)
- A shared runtime supports multiple independent sessions, and GitHub says one CLI server can handle many concurrent sessions. The guide does not publish a numeric capacity or a throughput/concurrency guarantee; its `maxConcurrent = 50` is a sample application-side limit, not a stated SDK limit. [Scaling: vertical scaling](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/scaling#vertical-scaling)
- Session state is stored per session ID under `COPILOT_HOME/session-state/{sessionId}` by default. `baseDirectory` selects `COPILOT_HOME` for a runtime instance; when connecting to an existing runtime, configure storage on that runtime instead. The SDK also exposes `sessionFs` for routing session file operations to application-managed storage. [Multi-tenancy: `baseDirectory` and `sessionFs`](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/multi-tenancy#basedirectory) [TypeScript client source](https://github.com/github/copilot-sdk/blob/main/nodejs/src/client.ts)
- For shared multi-user servers, GitHub identifies `mode: "empty"` as the safe baseline: the application explicitly selects tools, MCP servers, skills, and workspace paths. In the default `copilot-cli` mode, ambient host capabilities can expose shared host filesystem access. [Multi-tenancy: `mode: "empty"`](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/multi-tenancy#mode-empty) [Session-level isolation](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/multi-tenancy#session-level-isolation-guarantees)
- The SDK does not check SiNiHu ownership of a session ID. GitHub explicitly assigns authentication, routing, ownership metadata, and authorization checks to the application, including checks before resume, delete, or any operation selected by a client-provided session ID. A unique or user-prefixed ID is useful metadata, not an access-control mechanism. [Multi-tenancy: shared-runtime pattern](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/multi-tenancy#pattern-2-shared-cli-with-mode-empty) [Session IDs and shared-runtime access control](https://docs.github.com/en/copilot/how-tos/copilot-sdk/features/session-persistence#deployment-patterns)
- The SDK does not provide built-in session locking. GitHub requires application-level serialization if multiple clients may write to the same session. The scaling guide also identifies load balancing as external and runtime session state as file-based; horizontal replicas need sticky routing or shared session storage. [Scaling: shared sessions](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/scaling#pattern-3-shared-sessions-collaborative) [Scaling: limitations](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/scaling#limitations)
- `disconnect()` releases active in-memory resources but preserves persisted session state; `deleteSession()` removes it. BYOK provider credentials are not persisted with the session and must be supplied again when resuming. [Session persistence: lifecycle and BYOK resume](https://docs.github.com/en/copilot/how-tos/copilot-sdk/features/session-persistence#using-byok-bring-your-own-key-with-resumed-sessions)

### Authentication and credentials

- GitHub Copilot authentication is separate from SiNiHu application login. In SDK multi-user mode, SiNiHu authenticates the incoming app request and authorizes its app user; the SDK then receives the Copilot/provider credential selected for that session. GitHub describes the application as owning authentication and authorization in backend multi-user mode. [Backend services](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/backend-services) [Multi-tenancy overview](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/multi-tenancy)
- GitHub Copilot user authentication can use a signed-in CLI identity, explicit GitHub OAuth/App user tokens, or environment tokens (`COPILOT_GITHUB_TOKEN`, `GH_TOKEN`, `GITHUB_TOKEN`). For OAuth, the app performs the OAuth flow and passes a GitHub user token; the SDK does not manage token storage, refresh, or expiration. A Copilot subscription is required for this route. [Authentication](https://docs.github.com/en/copilot/how-tos/copilot-sdk/auth/authenticate) [GitHub OAuth setup](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/github-oauth#token-lifecycle)
- The SDK distinguishes process/client credentials from per-session identity. A client-level token authenticates the runtime process; `gitHubToken` or a rotating token provider on a session scopes GitHub Copilot access to that session. External runtimes own process-level auth and storage, so a shared backend should not assume an app user's identity is implicitly propagated. [Multi-tenancy: per-session token](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/multi-tenancy#per-session-githubtoken) [Authentication: rotating tokens](https://docs.github.com/en/copilot/how-tos/copilot-sdk/auth/authenticate#rotating-session-scoped-github-tokens)
- BYOK is a different route: provider configuration supplies an endpoint and API key or bearer-token provider, and bypasses GitHub Copilot authentication. GitHub documents OpenAI-compatible providers, Azure/Microsoft Foundry, Anthropic, and local endpoints. Its Azure managed-identity guide composes the SDK with Azure Identity and a bearer-token callback; that is provider authentication, not SiNiHu app login. [BYOK](https://docs.github.com/en/copilot/how-tos/copilot-sdk/auth/byok) [Azure managed identity with BYOK](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/azure-managed-identity)

### IndexedDB and sharing

- IndexedDB databases are associated with a storage key; the Storage Standard currently defines a storage key as an origin. The database name distinguishes databases within that storage key. Logging different family accounts into one SiNiHu origin does not itself create separate IndexedDB storage partitions. [IndexedDB: database and connection](https://w3c.github.io/IndexedDB/#database-connection) [Storage: storage keys](https://storage.spec.whatwg.org/#storage-keys)
- The origin boundary prevents other origins from accessing that site's database; it is not a per-app-account authorization system. IndexedDB has no pathname-level database access control. [IndexedDB: security considerations](https://w3c.github.io/IndexedDB/#security) [MDN: same-origin behavior](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Basic_Terminology#key-characteristics)
- IndexedDB transactions provide atomic writes and coordinate overlapping read/write scopes; they do not implement concurrent editing policy, merge semantics, or app-level ownership. The standard allows multiple database clients, including pages and workers, and specifies transaction scheduling. [IndexedDB: transaction scheduling](https://w3c.github.io/IndexedDB/#transaction-scheduling) [IndexedDB: transaction lifecycle](https://w3c.github.io/IndexedDB/#transaction-lifecycle)
- IndexedDB does not synchronize local records with a server database. Data stored there is available within the browser's storage context; making a deck available to another account or device requires SiNiHu-defined transfer or synchronization. [MDN: IndexedDB limitations](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Basic_Terminology#limitations)

## Recommendations for SiNiHu

These are design recommendations, not guarantees supplied by the SDK or browser:

- Treat Microsoft/Google/GitHub sign-in as **SiNiHu identity**. Maintain a stable internal user ID and authorize every project, deck, template, AI session, credential record, and share operation against it. Do not accept a client-supplied user/session/project ID as proof of ownership.
- Start with one backend-managed Copilot runtime in `mode: "empty"`, unique opaque session IDs, explicit tool/workspace allowlists, and per-session GitHub token/provider configuration. Persist an app-side session-to-owner/project mapping. Consider a runtime per user if the shared runtime's host/storage boundary is insufficient; GitHub documents that as stronger isolation with higher resource cost.
- Serialize writes to any session shared by multiple users. Define whether family sharing means read-only, edit, or co-edit access, and define conflict/turn ordering in SiNiHu. SDK support for connecting clients to a session is not a locking or authorization policy.
- Keep BYOK and GitHub OAuth tokens out of IndexedDB deck records, ZIP exports, and logs. Store credentials in a backend-controlled secret store (encrypted at rest, access-controlled); retrieve/decrypt only for authorized use and pass the correct provider config/token to the session. Re-provide provider credentials when resuming BYOK sessions.
- Use IndexedDB for device-local drafts and revisions, but key/namespace records by stable app user ID and enforce the active account in every repository query. This is organizational separation only: same-origin client code can access the origin's databases. On logout/account switch, close editors and prevent accidental reuse; do not represent local namespaces as a hard security boundary.
- Implement explicit sharing as an app-owned operation: for portable offline handoff, export/import a deck package and omit secrets; for live cross-device sharing, add server-backed deck metadata/content with owner/share ACLs and a defined synchronization/conflict strategy. IndexedDB itself supplies neither workflow.
- Test multi-tab editing and schema upgrades using IndexedDB transactions and `versionchange` handling; expose storage quota/persistence status and provide reliable export/backup, since browser storage may be cleared or evicted. [IndexedDB upgrade coordination](https://w3c.github.io/IndexedDB/#database-connection) [Storage: persistence, quota, and storage pressure](https://storage.spec.whatwg.org/#persistence)

## Practical Checklist

- [ ] Separate SiNiHu app identity from GitHub Copilot/provider credentials.
- [ ] Choose shared-runtime versus per-user-runtime isolation; document the residual trust boundary.
- [ ] Use `mode: "empty"`; explicitly scope tools, paths, MCP servers, and skills.
- [ ] Store session ownership in SiNiHu; authorize create, resume, send, list, and delete operations.
- [ ] Use unique session IDs and per-session auth; implement refresh/expiration handling.
- [ ] Add locking or queueing for a session that more than one user may mutate.
- [ ] Keep provider secrets server-side; re-provide BYOK configuration after session resume.
- [ ] Namespace local IndexedDB data by app user, but do not rely on that for security.
- [ ] Specify explicit share permissions and either export/import or server-backed synchronization.
- [ ] Verify persistence, quota/eviction, backup/export, multi-tab writes, and schema upgrade behavior.

## Open Questions

- Which login providers are required for SiNiHu app identity, and should GitHub Copilot use each user's GitHub subscription or a deliberate service-account model?
- Is local-only deck storage acceptable when users switch devices, or should explicit sharing introduce a server-owned canonical deck?
- Does family collaboration require simultaneous editing of one deck/session, or only owner-granted access with one active editor at a time?
- What separation is required between family members on a shared browser/OS account, and is browser-local data acceptable under that threat model?

## Sources

Primary sources reviewed: [GitHub Copilot SDK setup guides](https://docs.github.com/en/copilot/how-tos/copilot-sdk/setup/choosing-a-setup-path), [official SDK repository](https://github.com/github/copilot-sdk), [TypeScript client implementation](https://github.com/github/copilot-sdk/blob/main/nodejs/src/client.ts), [TypeScript session implementation](https://github.com/github/copilot-sdk/blob/main/nodejs/src/session.ts), [Indexed Database API 3.0](https://w3c.github.io/IndexedDB/), and [WHATWG Storage Standard](https://storage.spec.whatwg.org/). MDN is used only as an authoritative browser-platform guide for IndexedDB's same-origin behavior and lack of built-in synchronization.