# Cross-Platform Security Standards

Load for every review. The platform files list the APIs specific to each language. This file gives the categories to check, based on the **OWASP Top 10:2025**. Also consult the **CWE Top 25** for weakness names when describing findings.

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low. These are default severities. Adjust them with the impact × likelihood rules in [../severity.md](../severity.md).

## A01 Broken Access Control (includes SSRF)
- `[C]` New endpoint, action, page or API with no authentication or authorization check, while similar existing ones have one.
- `[C]` IDOR: an object is loaded by an ID from the request with no check that the caller owns it or may see it.
- `[H]` Access control enforced only in the UI (hidden button, hidden menu) but not on the server or data layer.
- `[H]` Path traversal: a user-supplied file name joined to a base directory without normalizing it and checking that it stays inside the base.
- `[H]` SSRF: the server fetches a URL from user input with no allowlist. Also check for redirects and for internal and metadata addresses (`169.254.169.254`, `localhost`, private ranges).
- `[H]` CORS that reflects any origin while allowing credentials.
- `[M]` Open redirect from a `returnUrl`, `next` or `redirect` parameter.

## A02 Security Misconfiguration
- `[C]` Debug mode, a dev exception page or an interactive debugger enabled in production config.
- `[H]` Default credentials, or sample or admin endpoints left enabled.
- `[H]` XML parsers with DTDs or external entities enabled (see [xml.md](xml.md)).
- `[M]` Stack traces or internal details returned to clients.
- `[M]` Missing security headers on new web apps (CSP, HSTS, `X-Content-Type-Options`, frame protection); cookies without `Secure`, `HttpOnly` or `SameSite`.

## A03 Software Supply Chain Failures
- `[H]` A new dependency, or a version bump, to a release with known CVEs. Run the ecosystem's audit tool if one is available: `npm audit`, `pip-audit`, `dotnet list package --vulnerable`, OWASP Dependency-Check, `mvn org.owasp:dependency-check-maven:check`.
- `[H]` Dependency fetched over plain HTTP, or from an unofficial registry or Git URL without pinning.
- `[M]` Manifest changed without updating the lockfile; unpinned or floating versions (`*`, `latest`, `LATEST`, `RELEASE`).
- `[M]` CI actions or images referenced by a mutable tag (`@main`, `:latest`) where the project otherwise pins by commit SHA or digest.
- `[M]` A new dependency that looks like a typosquat, or that is unmaintained, or that duplicates something the project already has.

## A04 Cryptographic Failures
- `[C]` Hardcoded secrets, keys, tokens or connection-string passwords in source or committed config.
- `[C]` TLS certificate or hostname validation disabled.
- `[H]` Passwords hashed with a fast hash (MD5, SHA-1, SHA-256 without a KDF). Use Argon2id, scrypt, bcrypt or PBKDF2.
- `[H]` A non-cryptographic random generator used for tokens, IDs, OTPs or reset links.
- `[H]` ECB mode, a static or reused IV or nonce, home-grown crypto, or deprecated algorithms (DES, 3DES, RC4).
- `[M]` Sensitive data stored or sent unencrypted when the rest of the system encrypts it.

## A05 Injection
- `[C]` SQL, NoSQL, OS command, LDAP, XPath or expression-language queries built by concatenating or interpolating untrusted input.
- `[H]` XSS: untrusted data rendered without context-appropriate encoding, or through a raw-HTML escape hatch.
- `[H]` Server-side template injection: user input used as a template, not as template data.
- `[M]` Log injection: unsanitized CR/LF from user input written to logs.

## A06 Insecure Design
- `[H]` The server trusts values the client computed: prices, totals, roles, `isAdmin`, quantities.
- `[H]` Business-logic gaps, such as negative quantities, skipping a workflow step, or replaying a one-time action.
- `[M]` No rate limiting or lockout on login, OTP, password-reset or other expensive endpoints.

## A07 Authentication Failures
- `[C]` JWTs accepted without verifying the signature, or with the `none` algorithm allowed, or decoded but not verified.
- `[H]` Session ID not rotated at login; tokens or credentials placed in URLs.
- `[H]` Secrets or tokens compared with a non-constant-time equality check.
- `[M]` Password policy or MFA flow weakened compared to the existing code.

## A08 Software or Data Integrity Failures
- `[C]` Deserializing untrusted data with a serializer that can create any type (see the platform files).
- `[H]` Mass assignment: request bodies bound directly to persistence entities, so a caller can set fields like `role` or `ownerId`.
- `[H]` Updates, plugins or scripts downloaded and run without verifying a signature or checksum.

## A09 Security Logging & Alerting Failures
- `[H]` Passwords, tokens, full card numbers or other sensitive personal data written to logs.
- `[M]` Security-relevant events not logged when sibling code logs them: failed logins, access denials, privilege changes.

## A10 Mishandling of Exceptional Conditions
- `[H]` Fail-open: an exception in an auth, validation or payment check results in the request being allowed.
- `[H]` Partial failure leaves inconsistent state: no transaction, no compensation, no idempotency on retry.
- `[M]` Exceptions swallowed (`catch {}` or `except: pass`); return values or error codes ignored.
- `[M]` Error paths that leak resources or skip cleanup.
