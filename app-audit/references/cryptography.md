# Cryptography and secrets generation

Load when the app encrypts, hashes, signs, or generates tokens itself, instead of leaving all of it to a platform.

The first finding to look for is home-made cryptography: any custom cipher, custom construction, or "encryption" that is really encoding (Base64, hex, XOR, or ROT). Each one is a finding on its own.

## Passwords

- A slow, salted, purpose-built password hashing algorithm with sensible cost parameters. A fast general-purpose hash, salted or not, is a High.
- Cost parameters that are tunable and a path to rehash on login when they change.
- Passwords never encrypted reversibly, logged, or returned.
- Comparison of secrets in constant time.

## Randomness

- Tokens, reset links, invite codes, session ids, API keys, and one-time codes generated with a cryptographically secure generator. A general-purpose random function, a timestamp, a counter, or a hash of predictable input is guessable.
- Enough entropy for the lifetime and exposure of the token, and short numeric codes protected by attempt limits and expiry.
- UUIDs used as secrets: only random versions are unpredictable, and even those are identifiers first.

## Encryption

- Authenticated modes for symmetric encryption. Unauthenticated modes allow tampering, and ECB leaks structure.
- Nonces and IVs never reused with the same key, and never hardcoded.
- Key sizes and algorithms that are current, with no deprecated ciphers or hashes used for security purposes.
- Encrypt-then-what: data that is encrypted in the database but decrypted for every query, logged in plaintext, or cached in the clear gains little.
- Field-level encryption for the most sensitive data, where the threat model includes database compromise.

## Keys

- Keys out of the repo, out of the client, and out of config files. Stored in a secrets manager or KMS, with access limited to the service that needs them.
- One key doing several jobs (signing and encryption, or shared across environments and tenants).
- Rotation: possible without downtime, and actually planned. A key that cannot be rotated is a permanent liability once leaked.
- Derivation from passwords using a proper key-derivation function, not a single hash.
- Where the key sits relative to the data it protects. A key stored next to the ciphertext protects against very little.

## Signatures and integrity

- Signature and MAC verification on everything that claims to be signed: webhooks, tokens, update packages, and signed URLs. Verified against the raw bytes, with a constant-time comparison.
- Algorithm fixed by the verifier, never chosen by the message.
- What the signature covers. A signed token whose unsigned part is trusted is unsigned.
- Expiry and replay protection on signed messages.

## Transport

- TLS on every hop that crosses an untrusted network, including service-to-service and database connections.
- Certificate validation never disabled, in any environment's code path that could reach production.

## How to report

Name the exact primitive, mode, and parameters you saw, with file and line. "Weak crypto" is not a finding. State what an attacker gains: offline cracking, token prediction, tampering, or decryption after a database leak. If a library's defaults decide the outcome and you did not verify the installed version's defaults, grade the finding Likely and say so.
