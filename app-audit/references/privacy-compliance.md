# Privacy and compliance mapping

Load when personal data is processed, or when a compliance mapping was requested. This file has two parts with different rules: the privacy engineering review always runs when loaded, and the framework mapping runs only on request.

This is an engineering review. It supports privacy and legal work and does not replace it. Never state that the system is compliant, that a law applies, or that consent is valid. Those are legal conclusions.

## Part 1: Privacy engineering review

Build a data inventory in `inventory.md`:

`data category | whose data | where collected | where stored | who can read it | which third parties receive it | retention | deletion path`

Then check:

- Collection. Data collected that no feature uses, and stable identifiers that link a person across contexts.
- Sensitive categories. Health, biometric, financial, location, government ids, and children's data. Identify them from the schema and forms, and check they get stronger handling than ordinary fields.
- Logs, traces, and error reporters. Personal data, tokens, and full request bodies written to places with wide access and long retention. Cite the file and line, never the values.
- Analytics and telemetry. What each SDK and event sends, to whom, and whether it starts before any consent the product claims to collect.
- Third-party recipients. Every processor that receives personal data: payment, email, support, analytics, AI providers, and hosting. This list is what a privacy notice and a processing agreement must match.
- Location of data. Where databases, backups, and processors are hosted, as far as config shows. Residency and cross-border transfer rules depend on this.
- Access. Who inside the company and the app can read personal data, and whether that access is logged.
- Retention. Whether anything is ever deleted, or everything is kept forever by default.
- Deletion and export. Whether a user's deletion request reaches the primary store, caches, search indexes, analytics, third parties, and backups. Whether a user's data can be exported. Soft delete that keeps the data readable is not deletion.
- Consent and preference records. Where they are stored, and whether the code actually obeys them.
- Minors. If children can be users, check age handling, parental consent flows, advertising and tracking SDKs, and what data is collected by default.
- Breach readiness. Whether logging and monitoring would let the team establish what was accessed, which most notification duties depend on.

Missing documentation is not a privacy defect. A flow you could not trace is a gap, not a finding.

## Part 2: Framework mapping (only on request)

Map existing findings and gaps to the frameworks the owner names. Ask which jurisdictions and frameworks matter if that is unclear, instead of assuming one.

Commonly requested:

- Application security: OWASP Top 10, OWASP API Security Top 10, OWASP ASVS, OWASP MASVS for mobile, and OWASP Top 10 for LLM Applications.
- Management systems and assurance: ISO/IEC 27001 Annex A, ISO/IEC 27701, ISO/IEC 42001 for AI, SOC 2, and NIST CSF.
- European Union: GDPR and NIS2.
- United States: CCPA/CPRA and other state privacy laws, COPPA for children, and sector rules such as HIPAA or PCI DSS where the data calls for them.
- Saudi Arabia: the Personal Data Protection Law, the NCA Essential Cybersecurity Controls, and the SAMA Cyber Security Framework for financial entities.
- United Arab Emirates: the federal Personal Data Protection Law, plus free-zone regimes (DIFC and ADGM) where the entity sits in one.
- Egypt: the Personal Data Protection Law.
- Other Gulf states: each has its own data protection law. Confirm which applies.

Rules for the mapping:

1. Map from findings to controls, never the reverse. A grid of controls with no finding behind it hides the fire.
2. Do not cite article, clause, or control numbers from memory. Verify each reference against the current official text, or write the control topic in words and mark the number as unverified. Regulations and their implementing rules change.
3. Mark each mapped item as: finding (evidence of a gap), observed control (evidence it exists), or not assessed. Most organizational controls (policies, training, and vendor contracts) are not assessable from a repo, and the mapping must say so.
4. Applicability is the owner's and counsel's call. Record the assumption you mapped under.

## Output

Add a "Privacy" section to the report with the data inventory summary, the findings, and the open questions for the owner. Put any framework mapping in a separate `compliance.md`, so it never dilutes the ranked findings.
