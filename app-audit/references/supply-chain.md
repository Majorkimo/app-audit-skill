# Supply chain and CI/CD

Load on every full audit. Inspect by reading manifests, lockfiles, and pipeline definitions. Never install, build, or run anything to inspect it.

## Dependencies

- Lockfile present, committed, and consistent with the manifest. No lockfile means every install can resolve differently.
- Version ranges that float (`*`, `latest`, or wide ranges) on direct dependencies.
- Hallucinated and typosquatted packages. AI agents invent plausible package names, and attackers register them. For every dependency you do not recognize, check that it is the package the code thinks it is: publisher, age, purpose, and whether the imported API actually exists in it. Without network access, list the unrecognized names as items to verify.
- Dependencies pulled from git URLs, tarballs, or private registries without pinning to a commit or hash.
- Dependency confusion: internal package names that could be claimed on a public registry, and registry config that does not scope them.
- Install and lifecycle scripts (`postinstall` and equivalents) in direct dependencies.
- Abandoned or single-maintainer packages on a critical path (auth, crypto, parsing, and payments).
- Known vulnerabilities. Reading the lockfile does not tell you which versions have advisories. Unless a scanner was authorized (see `tool-pass.md`), report this as not assessed, and never guess CVEs from memory.
- Vendored and copied-in code: version unknown, patches unknown.
- Unused dependencies, which add attack surface for no benefit.
- Licenses that conflict with how the product is distributed, as a Quality note.

## Build integrity

- Build scripts that download and execute remote code (`curl | sh`), or fetch unpinned tools.
- Generated code and build output committed to the repo, where it can drift from its source.
- Reproducibility: whether the artifact deployed can be traced to a commit.

## CI/CD pipelines

- Secrets available to workflows triggered by pull requests from forks or untrusted branches. This is the most common pipeline breach.
- Workflow triggers that run untrusted code with write permissions or deployment credentials.
- Script injection: untrusted values (branch names, PR titles, and issue bodies) interpolated into shell steps.
- Third-party actions and plugins referenced by a mutable tag instead of a pinned commit.
- Token and permission scope: default write access to the repo, packages, or cloud when read would do.
- Self-hosted runners shared between trusted and untrusted jobs, and runners with persistent state.
- Secrets echoed to logs, written to artifacts, or cached.
- Artifacts and caches that an untrusted job can poison for a trusted one.

## Path to production

- Branch protection: required reviews, required checks, and whether administrators can bypass them.
- Whether one person or one leaked token can push to production alone.
- Deploy credentials: long-lived versus short-lived, and scoped to the deploy or to the whole account.
- Security checks in the pipeline (tests, linting, scanning) that are present but set to never fail the build.
- Manual deploy paths that bypass the pipeline entirely.

## Mobile and client release

- Signing keys and keystores in the repo or in CI variables readable by any workflow.
- Store upload credentials and their scope.
- Over-the-air update mechanisms: who can push an update, whether it is signed, and whether it can change native behavior.
- Source maps and debug symbols published with production builds.

## Repository hygiene

- Secrets in the current tree. History needs authorization: see `tool-pass.md`.
- `.gitignore` covering env files, keys, and build output, and whether any of those were committed before the ignore rule existed.
- Who has write and admin access, deploy keys, and installed integrations, where that information is in the repo. Otherwise record it as a question for the owner.
