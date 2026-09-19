# Basic Security Assessment — OWASP Juice Shop

## 1. Scope & Objective

**Target:** OWASP Juice Shop (local Docker instance, `localhost:3000`)
**Assessment type:** Black-box, unauthenticated web application assessment
**Objective:** Identify security weaknesses through structured manual and tool-assisted testing, assess their risk, and provide remediation guidance — as a controlled, legal-lab exercise (Juice Shop is an intentionally vulnerable training application).

**Out of scope:** No denial-of-service testing performed; no testing against any system other than the local Juice Shop container.

## 2. Methodology

1. **Reconnaissance** — Identified the application, version (via error disclosure), and technology stack (Node.js/Express).
2. **Enumeration** — Reviewed HTTP response headers for security-relevant configuration (CSP, X-Frame-Options, etc.).
3. **Manual testing** — Probed the search functionality with both SQL injection and XSS payloads to test input handling.
4. **Automated scanning** — Ran a custom Python vulnerability scanner against the target as a supplementary check.
5. **Documentation** — Recorded each finding with evidence, root cause, and business impact.

This mirrors a standard lightweight assessment workflow: recon → enumerate → test → validate → report.

## 3. Findings & Risk Ratings

| # | Finding | Risk | Likelihood | Impact |
|---|---------|------|-----------|--------|
| 1 | SQL Injection (error-based) on product search | **High** | High — trivially triggered with a single character | High — potential data exposure/manipulation |
| 2 | Missing Content-Security-Policy header | **Medium** | High — present on every page load | Medium — amplifies impact of any XSS |
| 3 | DOM-based XSS via search parameter | **High** | High — no authentication required to exploit | High — session/credential theft potential |

*(Full technical detail, payloads, and evidence for each finding are documented in `../vulnerability-id/findings.md`.)*

## 4. Root Cause Analysis

All three findings trace back to a single underlying pattern: **user input is trusted and used unsafely** — passed into SQL queries without parameterization (Finding 1), and inserted into the DOM without sanitization (Finding 3) — combined with an absent browser-level safety net (Finding 2's missing CSP would otherwise have partially mitigated Finding 3's impact).

## 5. Recommendations (Prioritized)

| Priority | Recommendation | Addresses |
|----------|-----------------|-----------|
| 1 | Use parameterized queries / an ORM for all database access; never concatenate user input into SQL strings | Finding 1 |
| 2 | Sanitize/encode all user-supplied data before DOM insertion (e.g. DOMPurify, avoid `innerHTML`) | Finding 3 |
| 3 | Implement a strict Content-Security-Policy header (`script-src 'self'`, no `unsafe-inline`) | Finding 2 |
| 4 | Disable verbose/stack-trace error messages in production; return generic error pages instead | Finding 1 |

## 6. Conclusion

This assessment identified 3 vulnerabilities in the tested application — 2 High risk and 1 Medium risk — all stemming from insufficient input validation/output encoding and a lack of defense-in-depth browser controls. None require authentication to exploit, which raises overall risk since any unauthenticated visitor could trigger them. Addressing Findings 1 and 3 (both input-handling issues) should be treated as the immediate priority, with the CSP header added as a supporting control.

## Evidence
See `screenshots/` in this folder and the cross-referenced detail in `../vulnerability-id/findings.md`.
