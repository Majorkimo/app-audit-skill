# Infrastructure and deployment

Load when the repo contains IaC, Docker or compose files, Kubernetes manifests, server provisioning scripts, reverse-proxy config, or hosting-platform config.

Scope: this is a review of configuration in the repo. It says what the config would produce, not what is running. Anything about the live environment is unverified unless the human authorizes a check, and that limit goes in the report.

## Network exposure

- Services bound to all interfaces or published to the internet that should be internal: databases, caches, queues, search engines, admin UIs, metrics, and debug ports.
- Datastores with no authentication or default credentials. A cache or queue open on a public address is a Critical on its own.
- Firewall and security-group rules: default-deny or default-allow, and how wide the allowed sources are.
- Internal services trusting any caller on the private network.
- Egress: whether an application server can reach anything on the internet and the cloud metadata address, which matters the moment there is an SSRF.

## Hosts and access

- SSH: password login, root login, and shared keys. Who has access and how it is removed.
- Patching: base images and OS versions pinned to something old, and no update process.
- Deploy users and service accounts with more rights than the deploy needs.

## Containers

- Running as root, privileged mode, host network or PID namespace, and the container runtime socket mounted into a container.
- Secrets baked into images or passed as build arguments, which persist in image layers and history.
- Base images unpinned (`latest`) or from unknown publishers.
- Writable root filesystems, missing resource limits, and broad Linux capabilities.
- Compose and manifest files that publish internal ports to the host.
- Kubernetes, where present: default service accounts with cluster rights, secrets in plain manifests, missing network policies, and workloads without security context.

## TLS and the edge

- TLS termination: where it happens, and whether traffic behind it travels in cleartext across an untrusted network.
- Protocol and certificate handling in proxy config, HTTP to HTTPS redirect, and HSTS.
- Proxy trust: `X-Forwarded-For` and host headers trusted from anywhere, which breaks rate limiting, IP allowlists, and generated links.
- Request size and timeout limits at the proxy.
- Security headers set at the edge versus in the app. Note which layer owns them.

## Secrets and configuration

- Secrets in the repo, in IaC variables files, in CI config, or in state files (Terraform state holds secrets in plaintext).
- One set of credentials shared across environments.
- Environment variables exposed to client bundles by the hosting platform's public prefix.
- Debug mode, verbose errors, or seed and admin endpoints enabled by production config.

## Cloud and platform identity

- IAM roles and policies with wildcards on actions or resources.
- Long-lived access keys where short-lived roles are available.
- Storage buckets: public access, listing, and policies. See `files-storage.md`.
- Serverless and edge functions: public by default, and their environment secrets.
- Preview and branch deployments: publicly reachable, connected to production data, or carrying production secrets.

## Environments and delivery

- Real separation between development, staging, and production: accounts, data, and credentials.
- Production data copied into lower environments without masking.
- How a change reaches production, and whether one person or one compromised token can push it alone. See `supply-chain.md`.

## Resilience and detection

- Backups: what is covered, where they live, whether they are encrypted, who can read or delete them, and whether a restore has ever been tested. An untested backup is a hope.
- Single points of failure the product's availability claims do not allow for.
- Logging and monitoring: logs shipped off the host, retained, and protected from the app's own credentials. Alerting on authentication failures, privilege changes, and unusual data access.
- Whether anyone would notice an attack in progress, and how long it would take.

## DNS and domains

- Records pointing at decommissioned services or unclaimed cloud resources, which allow subdomain takeover.
- Wildcard records, and email authentication records (SPF, DKIM, and DMARC) for domains that send mail.
