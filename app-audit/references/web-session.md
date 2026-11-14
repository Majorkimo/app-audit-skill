# Web client, sessions, and federated login

Load when there is a browser client, cookie or JWT sessions, OAuth, OIDC, SAML, or SSO.

## Sessions and cookies

- Cookie flags on every session and refresh cookie: `HttpOnly`, `Secure`, a deliberate `SameSite` value, and a scoped domain and path.
- Session lifetime: idle and absolute expiry, rotation on login and privilege change, and no session fixation.
- Logout and revocation that invalidate server-side, not just delete the cookie. Check "log out everywhere", password change, and account disable.
- Tokens in browser storage. A token in `localStorage` is readable by any script on the origin, so every XSS becomes account takeover. Note where tokens live and what can read them.

## CSRF

For every state-changing route that authenticates by cookie: what stops a cross-site request. Trace the actual conditions (cookie `SameSite` value, a token, an origin check, or a custom header requirement) before calling it vulnerable or safe. A JSON API is not automatically safe, and `SameSite=Lax` does not cover state changes on GET.

## JWT

- Algorithm pinned on verification, no `none`, and no confusion between symmetric and asymmetric keys.
- Signing secret strength and where it lives. A short or hardcoded secret is a Critical.
- Expiry, audience, and issuer all checked. Long-lived access tokens with no revocation path.
- Claims trusted without re-checking: a role or tenant id in the token that outlives the user's actual role or membership.
- Refresh tokens: rotation, reuse detection, and binding to the client.

## OAuth, OIDC, and SSO

- `redirect_uri` matched exactly, not by prefix or wildcard.
- `state` present and verified, and PKCE for public clients (browser and mobile).
- Authorization code flow, not implicit. Tokens never in URLs or logs.
- ID token validation: signature, issuer, audience, expiry, and nonce.
- Account linking: an attacker who controls an email at one provider must not take over an existing local account with the same email. Check whether the email is verified by the provider and how the link is made.
- SAML, where present: signature validation on the assertion itself, audience restriction, and replay protection.
- Just-in-time provisioning: which tenant and role a first-time SSO user lands in, and who controls that mapping.

## Browser-side attack surface

- XSS sinks: raw HTML rendering, markdown and rich-text renderers, `href` and `src` built from user input, and inline event handlers. Follow user-controlled text into another user's view, especially an admin's.
- Content Security Policy: present, and not defeated by `unsafe-inline`, `unsafe-eval`, or wildcard sources. Treat CSP as a mitigation, not as proof that XSS is absent.
- Clickjacking: `frame-ancestors` or `X-Frame-Options` on pages with sensitive actions.
- Open redirects in login, logout, and "return to" parameters, which also weaken OAuth flows.
- CORS: an origin reflected from the request combined with credentials is equivalent to no same-origin policy. Check the allowlist logic, not just the config value.
- Third-party scripts: which run with full access to the page, who can change them after review, and whether integrity hashes pin them. Tag managers count.
- `postMessage` handlers that do not check the sender's origin.

## Caching and exposure

- Private responses marked non-cacheable, and no personal data served through a shared cache or CDN keyed only by URL.
- Sensitive data left in the browser after logout: cached pages, storage, and service-worker caches.
- Secrets or internal endpoints in the client bundle and source maps published to production.
- Environment variables exposed to the client by a public prefix convention. List them and confirm each one is meant to be public.
- Server-side rendering and server actions: every server function callable from the client is a public endpoint and needs its own authentication and authorization, whatever the page around it checks.

## Security headers

HSTS, `X-Content-Type-Options`, `Referrer-Policy`, and a `Permissions-Policy` that matches what the app uses. Low severity on their own, but note what is absent.
