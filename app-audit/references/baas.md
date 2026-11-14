# Backend-as-a-service

Load when the app uses Supabase, Firebase, Appwrite, or a similar platform where the client talks to the database or storage directly. This is the classic breach in AI-built apps: the UI enforces the rules and the backend enforces nothing.

The core question for every table, collection, bucket, and function: if an attacker takes the public key out of the client and calls the platform directly, skipping the app entirely, what can they read and write.

## Keys in the client

- Which keys ship in the web bundle or mobile app. A public or anonymous key is designed to be exposed, and its safety depends entirely on the access rules behind it.
- An administrative or service-level key in any client, repo, or public env var is a Critical: it bypasses the access rules altogether. Check bundles, config files, env files with public prefixes, and mobile builds.
- Third-party keys (payment, email, AI providers) called from the client instead of from a server function.

## Database access rules

- Row-level security or security rules enabled on every table or collection that the client can reach. One table without them is fully open to anyone with the public key.
- Rules that exist but allow everything: a policy whose condition is literally true, a rule that checks only that the caller is signed in, or a rule on read but not on write.
- Rules that trust client-supplied values: a tenant id, role, or owner id taken from the request or from user-editable profile data instead of from the verified identity.
- Separate rules per operation: select, insert, update, and delete each need their own. Update rules must constrain both which rows can be changed and what they can be changed to, or a user reassigns a row to themselves or raises their own role.
- Rules in the repo versus rules in the platform dashboard. If the policies are not in migrations or a rules file, you cannot audit them: record that as an open gap and say what export would settle it.
- Views, functions, and stored procedures that run with elevated rights and bypass row-level rules. Each one is a privileged endpoint and needs its own checks.
- Columns the client can read that it should not: internal flags, other users' emails, tokens, and billing fields.

## Auth configuration

- Open signup when the product is invite-only, email confirmation disabled, and weak password policy.
- Roles and claims: who can set them, and whether a user can edit the profile field the rules rely on.
- Anonymous sign-in enabled, which makes "signed in" a meaningless condition.
- OAuth redirect allowlists with wildcards.

## Storage

- Bucket visibility and listing. A public bucket holding user documents is a breach waiting for a URL.
- Storage rules that mirror the database rules: tenant and owner checked on read, write, and delete.
- Upload constraints (size and type) enforced by the platform, not only by the client.

## Server functions

- Edge and cloud functions that do not verify the caller's token, or verify it and then use an administrative client without re-applying the caller's limits.
- Functions callable by anyone that perform privileged work: sending email, creating users, charging cards, or calling a paid API.
- Secrets used by functions: stored in the platform's secret store, not in the repo.

## Realtime and subscriptions

- Channels and subscriptions authorized per tenant and user. A client that can subscribe to any channel name sees other tenants' events.
- Broadcast and presence features carrying personal data.

## Logic that lives only in the client

- Business rules, price calculation, entitlement checks, and state transitions implemented in the app and written straight to the database. Anything the client computes, an attacker can write directly.
- "Admin" screens hidden in the UI while the underlying tables accept the same writes from any user.

## What you can and cannot conclude

Reading rules in the repo is implementation evidence for those rules only. Whether the live project matches them is unverified unless the human authorizes a check. State that limit in the report.
