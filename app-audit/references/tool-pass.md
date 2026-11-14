# Authorized tool pass, git history, and dynamic proof

Load only when the human has authorized one or more of these. The default audit is read-only and executes nothing. Each item below needs its own explicit approval, naming the command or the target. Record in the report exactly what ran, where, and what did not.

Before asking, say what you want to run, why, what it touches, and whether it needs the network. Then wait.

## 1. Git history

Secrets removed from the current tree are still leaked if they sit in history.

- Use read-only git commands only: log, show, and diff on explicit revisions. Do not fetch, check out, or run hooks, and do not use project-provided git wrappers.
- Search history for added and removed lines that look like keys, tokens, connection strings, and private key headers, and for env and key files that were once committed.
- Report by commit, file, and type with a placeholder. Never print the value.
- A secret found in history is live until rotated. The fix is rotation, not rewriting history, and the report says so.
- Also useful from history: large single-commit features (where review was thinnest), and security code that was removed or weakened.

## 2. Scanners

Scanners find the mechanical cases quickly and miss logic flaws. They support the audit and do not replace it.

Categories, with examples of widely used tools. Use what is already installed or what the human prefers:

- Secret scanning of tree and history (for example gitleaks or trufflehog).
- Dependency vulnerability scanning against advisory databases (for example osv-scanner, or the ecosystem's own audit command).
- Static analysis (for example semgrep with a maintained ruleset).
- Container image and IaC scanning (for example trivy).
- Mobile binary analysis (for example MobSF) when a build artifact is supplied.

Rules:

- Do not install tools or run package-manager commands inside the project without approval. Installs execute code.
- Never run a script, linter, or task the project itself provides as a way of scanning it.
- Dependency scanners usually need the network. Say so when asking.
- Triage every result. Each scanner hit is a candidate that goes through Phase 4 like any other finding. Do not paste raw scanner output into the report, and do not inflate the count with unreachable or dev-only issues.
- Record tool, version, ruleset, and date, so a re-audit can repeat it.

## 3. Dynamic proof

Turns "Likely" into "Confirmed" by observing behavior. This is the only part of the audit that touches a running system.

Hard limits:

- Local or dedicated test environments only. Never production, never shared staging with real data, and never a third party's system.
- Test accounts and synthetic data only.
- Proof of access, not exploitation. Stop at the first observation that settles the question. No data extraction beyond a single marker record, no destructive actions, and no denial-of-service testing.
- The human names the target, the accounts, and the time window.

The highest-value checks:

- Two-tenant test. Create tenant A and tenant B with one user each. As A, request B's objects by id through every route in the inventory that takes an id: read, update, delete, export, and bulk. Pass: every request is denied and nothing changes. Fail: any response returns B's data or changes B's state. Look at content and effects, not only status codes.
- Role test. As the lowest role, call every route the authorization matrix reserves for higher roles.
- Direct platform access, for backend-as-a-service: using only the public key from the client, read and write each table and bucket directly, bypassing the app.
- Lifecycle test. Revoke access, then replay a saved token, a signed URL, an export link, and a queued job.
- Unauthenticated sweep. Call every route with no credentials.

For each check, write down before running it: the request, what a pass looks like, what a fail looks like, and what would make it inconclusive. Afterwards record the observation and update the finding's status from "not run" to "passed", "failed", or "inconclusive".

## Reporting

Add a "What ran" section to the report: each authorized action, the exact command or request set, the environment, and the result. Everything not listed there remains code-reading evidence only.
