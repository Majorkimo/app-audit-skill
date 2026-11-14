# Tenancy, async work, and lifecycle

Load when the system has more than one tenant or org, or any queue, cron, export, or cache. Work from the data-access inventory.

## Tenant isolation

- Where the tenant identity comes from on each request: a verified session or token, never a header, body field, or URL segment the client controls without a membership check.
- Every row in the data-access inventory has a tenant filter or a stated reason it does not need one. Global and shared data is fine when it is deliberate: say which tables are shared on purpose.
- Enforcement point: a filter each developer must remember, a repository or ORM layer that applies it, or the database itself (row-level security). The weaker the mechanism, the harder you check every call.
- Raw queries, reporting queries, search indexes, and admin screens, which commonly bypass the scoped data layer.
- Joins and includes that pull related records without re-scoping them.
- Uniqueness and lookups: an email, slug, or external id that is unique globally when it should be unique per tenant, which leaks existence across tenants.
- Cross-tenant references: a record in tenant A that points at an id in tenant B, accepted because only the parent was checked.

## Caches

- Cache keys that include tenant and, where the data is per user, the user. A key built only from the resource id serves one tenant's data to another.
- Invalidation when membership, role, or permission changes.
- Shared caches and CDNs holding private responses.
- Whether attacker-influenced content can be stored in a shared cache and served to others.

## Queues, cron, and delayed work

A passing check on the request does not cover what runs later.

- For every job: where tenant and user identity come from. A payload field set by the producer is a claim, and anything that can enqueue can forge it.
- Whether authority is checked again at execution time. A job enqueued by a user who has since been removed should not run with their old access.
- Retries, dead-letter replays, and duplicate delivery against operations that must happen once.
- Worker context reuse: tenant state held in a module-level variable or a reused connection leaking into the next job.
- Cron and system jobs that iterate over all tenants: one tenant's bad data must not stop or corrupt the run for the others.
- Who can enqueue: queue or broker endpoints reachable without authentication, and admin UIs for the queue exposed.

## Exports and generated files

- Follow request, generation, storage, and retrieval separately. Who may ask, whose data goes in, where the file lands, and who can fetch it.
- Export links: guessable, long-lived, or still valid after the requester lost access.
- Aggregates and reports that keep tenant ownership through transformation.

## Lifecycle

Walk each transition and ask what observation proves it took effect:

- Logout and token revocation that invalidate server-side.
- Role or membership change: sessions, caches, pending jobs, and open websocket connections.
- User removal from a tenant: API keys, personal access tokens, scheduled jobs, and shared links they created.
- Tenant offboarding: active records, caches, queued work, exports, search indexes, and backups.
- Restore from backup: must not revive deleted data or revoked access, and restoring one tenant must not expose another.

## Administration and support access

- Tenant admin versus platform admin: tenant administration must not grant platform-wide authority.
- Support impersonation: scoped to one tenant, limited to stated operations, expiring, revocable, visibly logged, and attributed to the real operator.
- Tenant settings that affect shared behavior: a template, webhook, or policy change in one tenant reaching another through a shared dependency.

## Shared capacity

- What bounds one tenant's demand on shared workers, storage, and downstream services. Ingress rate limits do not prove worker fairness.
- Expensive operations (exports, imports, reports, AI calls) that one tenant can trigger without limit.
