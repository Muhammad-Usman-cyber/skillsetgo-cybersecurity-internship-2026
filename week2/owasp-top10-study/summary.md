# OWASP Top 10:2025 — Study Summary

## Objective
Summarize each OWASP Top 10 (2025 edition) category with a safe conceptual example and defensive recommendations, based on structured study and legal-lab practice (OWASP Juice Shop, DVWA).

> **Note:** OWASP released the Top 10:2025 edition on 6 November 2025, replacing the 2021 list. Key changes: Server-Side Request Forgery (SSRF) was merged into Broken Access Control, and two new categories were added — Software Supply Chain Failures and Mishandling of Exceptional Conditions.

## A01:2025 — Broken Access Control
**What it is:** Restrictions on what authenticated users are allowed to do aren't properly enforced — e.g. a user can access or modify data belonging to another user simply by changing an ID in a URL or request. This category now also absorbs SSRF (a server making requests to unintended destinations based on unvalidated user input).
**Safe example:** Changing `?user_id=104` to `?user_id=105` in a request and receiving another user's data, when the app should have checked ownership.
**Defense:** Enforce access-control checks server-side on every request; deny by default; validate/allow-list any server-side request destinations; never rely on hiding a link/button as the only protection.

## A02:2025 — Security Misconfiguration
**What it is:** Insecure default settings, unnecessary features enabled, verbose error messages, or missing security headers.
**Safe example (validated in Week 1):** Juice Shop's root response was missing a `Content-Security-Policy` header, leaving no browser-level restriction on script execution.
**Defense:** Harden configurations before deployment; disable unused features; set security headers (CSP, X-Frame-Options, HSTS); never expose stack traces to users.

## A03:2025 — Software Supply Chain Failures *(new category)*
**What it is:** Risks introduced through third-party libraries, dependencies, CI/CD pipelines, and build tooling — broader than just "using an outdated library" (the old A06:2021), this covers the entire chain of trust from source to deployment.
**Safe example:** A CI/CD pipeline that pulls a dependency from an unverified source with no integrity/signature check, allowing a compromised package to be silently included in a build.
**Defense:** Maintain a software bill of materials (SBOM); use dependency-scanning tools; verify package signatures/checksums; secure the CI/CD pipeline itself.

## A04:2025 — Cryptographic Failures
**What it is:** Sensitive data (passwords, tokens, personal data) is exposed because it's transmitted or stored without proper encryption, or with weak/outdated algorithms.
**Safe example (validated in Week 2):** DVWA's Weak Session IDs module generates session tokens using `md5()` of a simple incrementing counter — a weak, predictable use of a hash function instead of proper cryptographic randomness.
**Defense:** Enforce TLS everywhere; use strong, modern hashing for passwords (bcrypt/Argon2 with salting); use cryptographically secure random generators for tokens, never predictable input.

## A05:2025 — Injection
**What it is:** Untrusted input is passed to an interpreter (SQL, OS commands, LDAP) as part of a command/query, allowing an attacker to alter its meaning. Includes Cross-Site Scripting (XSS).
**Safe example (validated in Week 1 & 2):** Juice Shop's search field returned a raw `SQLITE_ERROR` from a single-quote payload; DVWA's ping field allowed `127.0.0.1 && whoami` to execute an arbitrary OS command.
**Defense:** Use parameterized queries/prepared statements; validate and encode all input; escape output based on context (HTML, JS, SQL, shell).

## A06:2025 — Insecure Design
**What it is:** Security flaws baked into the architecture itself — missing threat modeling, missing rate-limiting on sensitive actions, business logic that can be abused — rather than a single implementation bug.
**Safe example:** A password reset flow with no rate limit, allowing unlimited guesses at a reset code.
**Defense:** Threat-model features during design, not after; build in rate limiting, account lockout, and abuse-case testing from the start.

## A07:2025 — Authentication Failures
**What it is:** Weaknesses in how the app verifies identity — weak password policies, exposed session identifiers, missing multi-factor authentication, predictable session tokens.
**Safe example (validated in Week 2):** DVWA's session ID generation logic uses no random component even at its "High" security setting — an attacker who sees one valid session can predict others.
**Defense:** Enforce strong password policies, offer MFA, rotate/invalidate session tokens on logout, use secure session cookie flags (`HttpOnly`, `Secure`, `SameSite`).

## A08:2025 — Software or Data Integrity Failures
**What it is:** Code or infrastructure that relies on plugins, updates, or deserialization without verifying integrity — e.g. auto-update mechanisms that don't verify a digital signature.
**Safe example:** An application that pulls and executes an update package from a remote source with no signature/checksum verification.
**Defense:** Verify digital signatures on updates and dependencies; avoid deserializing untrusted data.

## A09:2025 — Logging & Alerting Failures
**What it is:** Insufficient logging of security-relevant events, or logs that generate no alerts, delaying breach detection.
**Safe example:** A system where repeated failed login attempts from the same source generate no alert and aren't rate-limited.
**Defense:** Log authentication events and access-control failures; centralize logs (SIEM); set up alerting for anomalous patterns — this connects directly to Week 3's SOC/monitoring work.

## A10:2025 — Mishandling of Exceptional Conditions *(new category)*
**What it is:** Applications that fail unsafely when something unexpected happens — an error, a timeout, an edge case — instead of failing closed/securely. Includes verbose error messages that leak internal details.
**Safe example:** An application error page that reveals the internal file path or framework version instead of showing a generic error (this occurred during Week 2 testing when DVWA's Insecure CAPTCHA module disclosed its config file path in an error message).
**Defense:** Design explicit error-handling paths that fail safely (deny access, generic message) rather than relying on default framework error output; never expose stack traces or internal paths to users.

## Summary Table

| # | Category | Status in this internship |
|---|----------|----------------------------|
| A01 | Broken Access Control | Conceptual only |
| A02 | Security Misconfiguration | ✅ Validated (missing CSP, Week 1) |
| A03 | Software Supply Chain Failures | Conceptual only |
| A04 | Cryptographic Failures | ✅ Validated (weak session ID hashing, Week 2) |
| A05 | Injection | ✅ Validated (SQLi + Command Injection, Weeks 1–2) |
| A06 | Insecure Design | Conceptual only |
| A07 | Authentication Failures | ✅ Validated (predictable session IDs, Week 2) |
| A08 | Software/Data Integrity Failures | Conceptual only |
| A09 | Logging/Alerting Failures | Ties into Week 3 |
| A10 | Mishandling of Exceptional Conditions | ✅ Validated (config path disclosure, Week 2) |

## Sources
- OWASP Top 10:2025 official project documentation (owasp.org/Top10/2025/)
- Own validated findings from Week 1 (Juice Shop) and Week 2 (DVWA)
