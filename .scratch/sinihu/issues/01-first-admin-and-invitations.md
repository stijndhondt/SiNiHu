# First Admin and Invitations

Type: grilling
Status: resolved

## Question

Specify the account bootstrap and invitation lifecycle for a private household deployment. The initial admin is established with a one-time code generated through the Docker console, and subsequent account creation is invitation-only. Decide code and invitation expiry, revocation and recovery; which admin actions exist; how users are removed; and how a person links Microsoft, GitHub, or Google sign-in methods to one SiNiHu account without account takeover or accidental duplication.

## Answer

### Initial admin bootstrap

- The Docker operator can generate a cryptographically random, single-use bootstrap code only while no SiNiHu Accounts exist.
- The code expires after 30 minutes. Generating a replacement invalidates any previously generated code.
- The person claiming the code must authenticate with a supported sign-in provider. Code consumption and creation of the first admin Account must be atomic so the code cannot establish more than one initial admin.
- Once the first Account exists, bootstrap is permanently closed through the normal application flow.

### Invitations

- Account creation after bootstrap is invitation-only.
- Each invitation is bound to one verified email address and can be claimed once by a person who proves control of that address through a supported sign-in provider. An unverified matching address is insufficient.
- Invitations expire after seven days. An admin may revoke an invitation before it is claimed; expired or revoked invitations cannot be restored and must be reissued.

### Admins and security events

- Multiple Accounts may hold the admin role. Admins may issue, revoke, and reissue invitations; promote or demote other Accounts; and deactivate or permanently delete Accounts.
- The last admin cannot be removed or demoted. An admin may remove their own Account only when another admin remains. Members request removal from an admin.
- Record the actor, affected Account, action, and timestamp for bootstrap and recovery, invitation changes, role changes, sign-in identity linking or unlinking, and Account removal. Make these security events visible to admins. Do not include codes, tokens, provider secrets, or deck content. Retain the history until the deployment owner removes it.

### Sign-in identities and recovery

- An Account may link Microsoft, GitHub, or Google sign-in identities. Linking requires an authenticated Account and proof of control of the new identity. A provider identity may belong to only one Account; matching email addresses never trigger automatic Account merging.
- If a provider identity already belongs to another Account, reject the link. Unlinking is allowed only when another usable sign-in identity remains.
- A member who loses every linked sign-in method may recover the existing Account through an admin after out-of-band verification. The new identity must be authenticated and linked to that Account, and the recovery must be recorded. If verification is not possible, the admin deactivates the old Account and invites the person to create a new one.
- If an admin loses every linked sign-in method, the Docker operator can generate a single-use recovery code valid for 15 minutes. Generating a replacement invalidates any earlier code. The code can bind a newly authenticated sign-in identity to the existing admin Account or designate a replacement admin; record the action without recording the code.

### Deactivation and permanent deletion

- Both deactivation and permanent deletion are admin-controlled. The affected person must be warned that browser-local deck data cannot be erased remotely or transferred to another Account. The owner can export or delete that local data and must explicitly acknowledge this limitation during removal.
- Deactivation immediately disables sign-in but retains the Account and its linked sign-in identities for possible admin-approved reactivation. Delete its stored provider credentials; reconnecting those providers is required after reactivation.
- Permanent deletion is irreversible. Delete the Account, its linked sign-in identities, provider credentials, and associated server-managed AI session data. The released provider identities may be linked to a newly invited Account later. Browser-local decks remain in the browser unless the owner deletes them.
