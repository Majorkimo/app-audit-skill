# API, authentication, and authorization

Load for any backend API. Work from the route inventory and the authorization matrix, not from memory of which routes exist.

## Authentication

- Credential handling: password storage with a slow, salted algorithm, no credentials in logs, and no plaintext reset or invite secrets.
- Login protection: brute-force and credential-stuffing limits per account and per source, and no user enumeration through different errors or timings on login, signup, and reset.
- MFA: present where the data warrants it, enforced server-side, and not bypassable through an alternate login path, an API token, or account recovery.
- Account recovery and invites: token entropy, expiry, single use, and binding to the account. Recovery is the most common way around a strong login.
- API keys and service tokens: scoped, revocable, stored hashed, and never accepted where a user session is expected.
- Every route in the inventory with "none found" under authentication must be public on purpose. Say which are, and why.

## Authorization

- A real authorization layer (RBAC or ABAC) versus role checks scattered through controllers. Scattered checks drift.
- Object level: can this caller touch this specific record. Change an id in a read, an update, a delete, a bulk operation, and an export, and trace what stops it. A passing read check proves nothing about update or export.
- Function level: a normal user reaching an admin or internal operation because only the UI hides it.
- Property level: fields the caller may read versus fields returned, and fields the caller may write versus fields accepted. Mass assignment of `role`, `tenantId`, `ownerId`, `price`, or `isVerified` is the classic.
- Privilege changes: who can grant roles, whether a user can raise their own, and whether a role change takes effect on existing sessions.
- Each mismatch between the intended and enforced matrix is a finding candidate.

## Input validation and injection

- Server-side validation is the source of truth. Schema validation on every body, query, param, and header that reaches logic.
- SQL and NoSQL injection, including raw query fragments, dynamic `ORDER BY`, and operator injection through JSON bodies.
- Command injection, path traversal, template injection, unsafe deserialization, and prototype pollution.
- XSS: stored and reflected output that reaches another user's browser, especially a more privileged viewer such as an admin panel.
- Header and host injection in generated links, redirects, and emails.

## Resource exhaustion and abuse

- Rate limits where they matter: auth, reset, search, export, messaging, and anything that costs money per call.
- Unbounded work: pagination without a maximum, queries without limits, regular expressions on user input, large or deeply nested payloads, and expensive operations callable in a loop.
- GraphQL: introspection in production, query depth and complexity limits, batching and aliasing used to bypass rate limits, and field-level authorization on every resolver.
- Enumeration: sequential ids, predictable slugs, and endpoints that confirm whether an account, email, or resource exists.

## Business logic

The bugs no scanner finds. For each money or state-changing workflow, write down the invariants, then try to break them:

- Price, quantity, discount, or currency supplied by the client. Negative and zero quantities. Coupon reuse and stacking.
- Workflow steps skipped or repeated by calling later endpoints directly.
- Replay and duplicates: the same request twice, concurrently, or after a timeout. Idempotency on payments, refunds, credits, and webhooks.
- Entitlements: a feature or limit enforced in the UI but not in the API, and subscription state trusted from the client.
- State machines: transitions that should be impossible but are only blocked in the UI.

## Webhooks and outbound calls

- Inbound webhooks: signature verification against the raw body, timestamp or nonce replay protection, and no trust in a payload field that says who or which tenant it is for.
- Any caller-supplied URL (callbacks, imports, avatars, link previews): destination allowlist, blocking of internal and metadata addresses, redirect handling, and what the worker can reach from where it runs. A configurable URL is not yet SSRF: trace what the fetch can reach and what comes back to the caller.
- Outbound payloads: what data leaves, to whom, and whether the response is trusted to change state.

## Errors, logging, and configuration

- Verbose errors, stack traces, and debug endpoints reachable in production.
- Logs sufficient to investigate an incident (who, what, which tenant, when, from where), protected from tampering, and free of secrets, tokens, and full personal records.
- Secrets out of code and out of env dumps, least-privilege service and cloud roles, and separate credentials per environment.
- Admin and internal tooling: reachable from where, protected by what, and audited or not.
