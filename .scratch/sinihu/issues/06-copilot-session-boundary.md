# Copilot Session Boundary

Type: grilling
Status: resolved

## Question

Specify how SiNiHu authorizes and routes AI-session operations between an authenticated account, its project, and the shared backend Copilot runtime. Define ownership checks for create, resume, use, and delete; session identifiers and persistence; behavior when provider credentials expire; and serialization for multiple tabs acting on the same user's session. Use the agreed shared runtime with `mode: "empty"`, explicit capabilities, and project-scoped session retention; do not rely on SDK session IDs as authorization.

## Answer

### Identity and authorization

- An AI session is bound server-side to one Account, stable Project ID, and Provider connection configuration. On first authorized use, the backend records the Project ID's Account owner; reject attempts by another Account to claim or access that Project ID. A browser-supplied Project ID is not proof of ownership.
- SiNiHu assigns each AI session an opaque ID. For every create, resume, use, and delete request, authenticate the Account, look up the session record, derive its Project and Provider connection from that record, and verify the Account owns it. SDK session IDs are internal runtime identifiers and never authorize access.
- A Project may have multiple AI sessions per Provider connection. Resuming one requires its explicit SiNiHu AI session ID.

### Runtime capabilities and context

- Use the shared backend Copilot runtime with `mode: "empty"` and explicitly grant only the capabilities needed for the authorized Project: read relevant deck context and return a Proposal. The runtime cannot directly mutate the deck, access other Projects or provider secrets, or use shell, filesystem, or arbitrary network capabilities.
- Supply the minimum relevant context by default. Let the user narrow context to selected slides or explicitly include the whole deck when needed. Applying a Proposal remains an explicit user action.

### Persistence and lifecycle

- Persist the runtime state required to resume an AI session; that state may include conversation context. Do not maintain a separate transcript. Retain state without idle expiry until the AI session, Provider connection, Project, or Account is deleted.
- Project deletion sends a server-side deletion tombstone. Once received, make the Project's AI sessions inaccessible immediately and retry runtime cleanup until complete. Queue tombstones while offline. Provide Account-scoped session listing and deletion, grouped by Project ID and limited to session metadata, so users can remove sessions if a tombstone is lost.
- Account deactivation blocks AI-session operations and cancels active work, but retains inaccessible session state. Reactivation requires the Provider connection to be reauthorized before resuming. Permanent Account deletion removes all associated AI session state, as specified in issue 01.
- Deleting a Provider connection prevents new provider calls and deletes its associated AI session state. Deletion blocks new operations, cancels active generation, and retries runtime cleanup.

### Provider credentials and configuration

- On definitive credential rejection, stop the call and mark the Provider connection `Needs attention`; preserve the AI session. Resume only after the same connection is reauthorized. Do not silently switch connections or credentials. Transient provider or network failures do not change connection status, as specified in issue 05.
- Bind each AI session to its Provider connection's provider, endpoint, and model configuration. Credential renewal or rotation may continue that session when the configuration is unchanged. A provider, endpoint, or model change requires a new AI session; do not resume an existing session against changed configuration.

### Concurrency and recovery

- Enforce one in-flight operation per AI session in the backend. Reject overlapping operations and submissions based on a stale server-side session sequence with a retryable conflict; clients refresh before retrying.
- Make create and delete requests idempotent. Give each use operation an ID and return its recorded result or status on retry rather than blindly repeating a possibly completed AI turn.
- If creation or resumption fails, record the operation status and clean up any partial runtime state. Never silently start a fresh conversation under the same SiNiHu AI session ID. Preserve the record as failed or unavailable and offer creation of a new AI session when resumption cannot be recovered.