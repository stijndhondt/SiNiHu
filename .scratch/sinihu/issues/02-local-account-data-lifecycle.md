# Local Account Data Lifecycle

Type: grilling
Status: resolved
Blocked by: 01

## Question

Define how browser-local Projects and reusable content behave across sign-in, Account switching, sign-out, deactivation, and permanent deletion. Browser storage is scoped to the application origin, not to an Account, and Account namespacing is an organizational boundary rather than hard security. The application must reduce casual cross-Account exposure without implying it can protect local data from someone with access to the same browser profile, developer tools, or origin storage.

## Account ownership and selection

- Associate local records with the stable internal Account ID. Projects, Templates, Themes, referenced assets, and revision history belong to that Account; do not use a client-supplied display name or provider identity as the ownership key.
- After sign-in, require explicit confirmation of the Account whose local data is to be opened and fresh authentication for that Account. Never automatically open another Account's local data.
- Before switching Accounts, flush pending edits. If saving fails, keep the current Account active and report the failure.
- On successful switch or sign-out, close editors and clear the active Account/session state. Retain the local records, but do not make them available until the Account is explicitly confirmed and authenticated again.
- When the active Account changes or signs out in one tab, immediately lock Account-scoped editors in all other tabs and require explicit confirmation before reopening.
- Lock an active Account after 15 minutes of inactivity. Require authentication again before reopening its local data.
- Explain during first local-data setup and again before switching or removal that same-origin storage separation is not a hard security boundary.

## Existing unassigned local data

Keep records created before Account ownership is introduced unassigned. Do not attach them automatically to the first or currently signed-in Account. Offer explicit adoption into one Account; move the records into that Account's namespace and remove the unassigned originals only after migration succeeds. Provide recovery/reporting if migration fails rather than silently losing or partially adopting data.

## Lifecycle and disposition

| Event | Local-data behavior | Export and user action |
| --- | --- | --- |
| Sign-in | Do not open local records until the user explicitly confirms the Account and completes fresh authentication. | Existing Account-local records remain available after confirmation. |
| Account switch | Flush edits first; if flushing fails, do not switch. On success, close editors, lock the former Account, and lock its editors in other tabs. | Confirm and authenticate the target Account before opening its records. |
| Sign-out | Close editors, clear active Account/session state, and retain Account-local records in a locked state. | No signed-out export flow. |
| Account deactivation | Retain local records, but require the user to export them, delete them locally, or explicitly acknowledge retaining them in this browser before disabling sign-in. | Complete the disposition while authenticated, before sign-in is disabled. |
| Permanent Account deletion | Before deletion, require the user to export the local data, delete it locally, or explicitly acknowledge that it will remain in this browser. After deletion, any retained records are locked to cleanup/deletion only; do not offer export through a deleted Account. | Complete the disposition while authenticated, in the pre-deletion flow. State that remote Account deletion cannot erase browser-local data. |
| Explicit local-data deletion | Delete all app data in the Account namespace: Projects, reusable content, revisions, assets, caches, and pending AI proposals. | Explain that browser storage APIs cannot guarantee secure physical erasure. |

## ZIP export and import

- An Account export contains its Projects, Templates and Themes, referenced assets, and revision history. Do not include authentication material, provider credentials, or other secrets.
- Export is available while authenticated and during the deactivation or permanent-deletion disposition flow. Do not offer export while signed out or after permanent Account deletion.
- Importing another Account's ZIP is allowed. Treat imported content as new copies owned by the target Account; preserve only non-sensitive provenance and never import credentials.
- Keep export/import behavior consistent with the existing portable Project archive format and validate imported content before adding it to an Account namespace.

## Answer

Browser-local content is owned by the stable internal Account ID, including Projects, Templates, Themes, assets, revisions, caches, and pending AI proposals. Require explicit Account confirmation and fresh authentication before opening an Account's data. Flush edits before switching and do not switch if saving fails; sign-out or a successful switch closes editors and locks that Account's data across tabs. Lock after 15 minutes of inactivity and require authentication to reopen. Explain during setup and before sensitive lifecycle actions that same-origin storage is not a hard security boundary.

Keep pre-existing content unassigned until explicit adoption. Move it into the selected Account namespace and remove its unassigned originals only after successful migration.

On deactivation, retain local data but require export, local deletion, or explicit acknowledgement of retention before disabling sign-in. Before permanent deletion, require the same disposition. Any retained data after deletion is locked to cleanup/deletion only; it cannot be exported through the deleted Account. Local deletion covers all Account-namespaced app data and warns that browser storage cannot guarantee secure physical erasure.

Authenticated users and users in pre-removal flows can export all Account-local Projects, reusable content, assets, and revision history. ZIPs contain no credentials. Importing another Account's ZIP creates new copies owned by the target Account, preserving only non-sensitive provenance.