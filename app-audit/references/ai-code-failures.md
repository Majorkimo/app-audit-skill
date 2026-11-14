# AI-generated code failures

Run this first, on every audit. Most code today is written fast by an AI agent under loose supervision. AI failures are not random bugs: they have a signature, they cluster in predictable places, and an AI mistake often is the vulnerability.

For each one you find, state what the AI clearly intended, what it actually produced, and the gap between the two. That framing is what makes the finding land.

## Silent and fabricated correctness

- Functions that look complete but never get called, dead handlers wired to nothing, and routes defined but never registered. AI generates plausible scaffolding and forgets to connect it. The route inventory exposes these: a handler with no trigger, or a guard class that no route uses.
- Hallucinated APIs, methods, env vars, or config keys that do not exist in the installed version of a dependency. Check calls against the versions in the manifest and lockfile, not against what the method "should" be called.
- Hallucinated packages. A dependency name that looks right but resolves to something unrelated, abandoned, or newly registered. See `supply-chain.md`.
- Error handling that swallows everything: `catch` blocks that log and continue, or return a success shape on failure. AI hides failures to keep the happy path clean.
- Stubs presented as features: functions that return hardcoded values, `TODO` behind a working-looking UI, mock data left in a production path, and feature flags that are permanently on or off.
- Tests that assert nothing, test the mock, or were edited to pass. A green suite written by the same agent that wrote the bug proves little.
- Validation on the client only. The form validates, the server trusts it blindly. AI builds the visible guard and skips the real one.

## Security theatre and broken guards

- Auth checks that are present but ineffective: a middleware that reads the role but never enforces it, a guard on the list route but not the detail route, and ownership checked in the UI but not in the query.
- Inconsistent tenant scoping. AI scopes nine queries and misses the tenth, and the tenth is the breach. The data-access inventory is the check: every row, not most rows.
- Secrets, API keys, tokens, or connection strings hardcoded or committed, including in client and mobile bundles that ship to every user. Also check example and seed files, which AI fills with real-looking and sometimes real values.
- Permissive defaults: CORS set to `*`, JWT with no expiry or a weak or hardcoded secret, password hashing with a fast algorithm or none, and raw HTML injection.
- Security code copied from a tutorial pattern: a default admin account, a debug or seed endpoint left routable, and a "temporary" bypass keyed on an env var or a magic header.
- Comments that claim a control the code does not implement ("validated above", "admin only"). The comment is not evidence.

## Logic and state errors

- Race conditions and missing transactions where two writes must be atomic, common because AI generates each operation in isolation.
- Off-by-one and pagination errors, wrong or absent rate limiting, and no idempotency on retried operations such as payments and webhooks.
- Inconsistent data models: the type says one thing, the schema another, and the migration a third. AI drifts across files because it does not hold the whole model.
- Copy-paste duplication where one fix had to land in five places and landed in one. Duplicated logic that has diverged is a live bug.
- Two implementations of the same concern living side by side (two auth helpers, two HTTP clients, two validation layers), where only one is hardened.
- Money and time handled naively: floats for currency, local time for expiry, and client-supplied timestamps or prices trusted by the server.

## How to hunt

- Start from the inventories. Empty cells and inconsistent rows are where generated code breaks.
- Compare siblings. For any guarded route, read the neighbouring routes in the same controller. For any scoped query, read the other queries on the same entity.
- Follow one feature end to end, UI to database, and note every place a layer assumes another layer did the checking.
- Read the newest and largest commits first if git history is available. Big single-commit features are where supervision was loosest.
