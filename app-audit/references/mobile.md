# Mobile clients

Load when there is a mobile client: Flutter, React Native, Capacitor or another WebView shell, or native. The first rule of mobile: the client is in the attacker's hands. Device possession, an unlocked screen, the signed-in account, and backend permission are four separate facts, and a client-side check is never an authorization decision.

## Secrets and the bundle

- Secrets, private keys, or internal endpoints baked into the bundle, retrievable by anyone who unzips the APK or IPA. Obfuscation slows this down and does not prevent it.
- Environment and build config: debug flags, verbose logging, test endpoints, and developer menus left on in release builds.
- Third-party keys that should be called from a server.

## Transport

- Cleartext traffic allowed, or TLS validation weakened: custom trust managers, disabled hostname checks, and broad transport-security exceptions.
- Certificate pinning where the threat model calls for it, and a plan for rotation so pinning does not brick the app.

## Local storage

- Tokens or personal data in plain preferences, local files, or an unencrypted local database instead of the platform keystore or keychain.
- Keychain and keystore accessibility settings: available only when unlocked, and not synced or backed up unless intended.
- App data included in device or cloud backups.
- Sensitive data in logs, crash reports, the clipboard, screenshots, and the app-switcher snapshot.
- Cached API responses, images, and WebView storage left behind after logout.

## Entry points from outside the app

- Deep links and custom URL schemes accepted without validation, letting another app or a web page drive navigation, pass identifiers, or trigger actions. Custom schemes can be claimed by any app: prefer verified app links and universal links for anything sensitive, including OAuth redirects.
- Android exported activities, services, receivers, and content providers reachable by other apps without a permission.
- Intents and share targets carrying attacker-controlled data into internal screens.
- Push notifications: content shown on the lock screen, stale notifications after an account switch, and whether tapping one re-checks the current account's authority.

## WebView and hybrid shells

- JavaScript bridges and native plugins exposed to web content. Any script that runs in the WebView can call them, so one XSS reaches native capabilities.
- WebViews that load remote or user-influenced URLs with the bridge enabled, and navigation not restricted to an allowlist.
- File access and universal access from file URLs enabled.
- For Capacitor and similar: the allowed navigation list, cleartext settings, and which plugins are installed but unused.

## Authentication and account state

- Biometric or PIN gates that are local only: they protect the screen, not the token.
- Token refresh and revocation across offline periods: when the device learns it was revoked, and what pending work does on rejection.
- Account switch and sign-out: the previous user's data, queued offline actions, cached files, and notifications. A queued action created by account A must not be sent as account B.
- Offline queues and sync: each action carries its creator, target, and base revision, and the server authorizes it at upload time, not at creation time.

## Platform hardening

- Root and jailbreak detection, tamper detection, and code obfuscation, only where the threat model justifies them (payments, regulated data). Report absence as a note, not a vulnerability, unless the app needs it.
- Minimum OS versions and deprecated APIs with known weaknesses.

## Permissions and store compliance

- Permissions requested but never used, and permissions requested before the user sees why. Both are a privacy problem and a store-review risk.
- Third-party SDKs: what each collects and where it sends it, since the app is accountable for them.
- Store privacy declarations (privacy labels, data-safety forms, and required-reason or privacy manifests) compared with what the code and SDKs actually collect. A mismatch is a finding.
- In-app account deletion, where the app allows account creation. Verify the current store policy before citing it.
- Children as users: stricter rules apply to data collection, advertising SDKs, and consent. See `privacy-compliance.md`.

## Mobile-specific AI failures

Generated mobile code tends to: validate in the form and trust it on the server, store tokens in plain preferences because that is the shortest example, hardcode a base URL and key per environment, leave a debug banner or logging on, and request every permission a plugin's README mentions.
