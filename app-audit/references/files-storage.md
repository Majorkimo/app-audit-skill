# File upload and storage

Load when users can upload, preview, download, or export files. Follow upload, processing, preview, and download as four separate paths. A safe upload path says nothing about the preview path.

## Upload

- Who may upload, and whether the endpoint is authenticated and rate limited.
- Type validation on the server by content, not by extension or the client-supplied MIME type. An allowlist of types, not a blocklist.
- Size limits enforced before the file is fully read into memory, and limits on count and total quota per user and tenant.
- Filenames: never used as a path or key as supplied. Check for traversal, overwrite of existing objects, and control characters.
- Direct-to-storage uploads (presigned URLs): what the signature constrains (key, size, content type, expiry), and whether the client can choose the destination key or bucket.

## Malware and hostile content

- Whether uploads are scanned before they are served to other users or processed. If nothing scans them, say so plainly and rate it by who can download them.
- Files quarantined until scanned, or reachable immediately.
- Formats that carry active content: SVG and HTML (script), office documents (macros), PDFs (script and external references), and archives.
- Archive handling: zip bombs, path traversal in entry names, and symlinks.
- Image and document processing libraries run on untrusted input: versions, sandboxing, and resource limits.
- Metadata that leaks personal data (location in image EXIF) passed on to other users.

## Serving and preview

- Uploads served from the app's own origin execute in the app's security context. Check for a separate origin or storage domain, `Content-Disposition: attachment` where inline rendering is not needed, a correct `Content-Type`, and `X-Content-Type-Options: nosniff`.
- Preview renderers that execute what they display: inline HTML or SVG, PDF viewers, markdown with raw HTML, and document converters.
- Thumbnails and derived copies: same access rules as the original, and removed when the original is.

## Access control on stored objects

- Buckets and containers: public versus private, listing enabled or not, and policies that grant more than intended.
- Object keys: guessable or sequential keys on a public bucket are equivalent to no access control.
- Signed URLs: lifetime, whether they survive revocation of the user's access, and whether they are logged or shared in ways that leak them.
- Authorization on download endpoints: tenant and owner checked on every fetch, not only at upload.
- Row-level or policy-based storage rules where the platform supports them. See `baas.md`.

## Storage hygiene

- Encryption at rest and in transit, and who holds the keys.
- Deletion: whether a deleted file is removed from storage, derived copies, caches, CDN, and backups, or merely unlinked in the database.
- Orphans: files left behind by failed or abandoned uploads, growing without limit.
- Temporary files on local disk: location, permissions, cleanup, and predictable names.
- Backups of file storage: where they live and who can read them.
- Storage credentials: scoped to the bucket and operations the app needs, not account-wide, and never in the client.
