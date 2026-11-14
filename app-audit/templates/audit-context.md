# Audit context

Copy this into the root of the repo being audited (or fold it into `CLAUDE.md` or `AGENTS.md`) and fill it in. The audit reads it first, treats it as the team's claim about the system, and verifies it against the code. Keep it short. Do not put secrets in it.

## Product

What the app does, in two or three sentences, and who uses it.

## Data

What data the app holds and how sensitive it is: personal data, payment data, credentials, health or financial records, children's data, and compliance or governance records.

## Tenancy and access model

- Single tenant, multi-tenant, or B2B with organizations.
- The roles that exist, and what each is meant to be able to do.
- How a request is tied to a tenant and a user.
- Any support, admin, or impersonation access, and who holds it.

## Architecture

- Clients (web, mobile, and others) and backends.
- Datastores, queues, caches, and file storage.
- Third-party services that receive data.
- AI features: models called, tools exposed, and retrieval sources.
- Where it is hosted and how it is deployed.

## Known constraints and decisions

Things an auditor would otherwise flag that are deliberate: public endpoints that are public on purpose, shared tables, accepted risks, and legacy areas scheduled for replacement.

## Design system

Where the design system or component library lives, if there is one.

## Jurisdictions and frameworks

Where the product is sold and which regulations or standards the owner cares about. Leave empty if no compliance mapping is wanted.

## Out of scope

Directories, services, or concerns the audit should skip, and why.

## Previous audits

Path to the last `audit/<date>/findings.json`, if any.
