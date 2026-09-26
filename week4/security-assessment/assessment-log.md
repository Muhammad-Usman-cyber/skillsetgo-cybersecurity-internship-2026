# Structured Security Assessment — Log

## Objective
Perform a full structured security assessment consolidating all vulnerabilities identified and validated across the internship's practical lab work (Weeks 1–2), presented as a single formal assessment.

## Scope

| Target | Environment | Weeks Tested |
|---|---|---|
| OWASP Juice Shop | Local Docker container | Week 1 |
| DVWA (Damn Vulnerable Web Application) | Local Docker container | Week 2 |

**Assessment type:** Black-box and grey-box (source-reviewed) web application testing, legal practice environments only.
**Out of scope:** Any production system; denial-of-service testing.

## Methodology
1. Reconnaissance — identify application, version, and available attack surface.
2. Manual input testing — craft payloads to test how untrusted input is handled (injection points, session logic).
3. Source code review — where available (DVWA), confirm root cause rather than relying on observed behavior alone.
4. Evidence capture — screenshot every successful test.
5. Risk rating — likelihood × impact, mapped to OWASP Top 10:2025.

## Consolidated Findings Log

| # | Finding | Target | OWASP 2025 | Severity |
|---|---------|--------|------------|----------|
| 1 | SQL Injection (search endpoint) | Juice Shop | A05 — Injection | High |
| 2 | Missing Content-Security-Policy header | Juice Shop | A02 — Security Misconfiguration | Medium |
| 3 | DOM-based XSS (search field) | Juice Shop | A05 — Injection | High |
| 4 | SQL Injection (User ID lookup) | DVWA | A05 — Injection | High |
| 5 | OS Command Injection (ping utility) | DVWA | A05 — Injection | Critical |
| 6 | Predictable Session ID generation | DVWA | A07 — Authentication Failures / A04 — Cryptographic Failures | High |

*(Full technical detail for each finding is preserved in its original week's documentation: `../../week1/vulnerability-id/findings.md`, `../../week1/security-assessment/assessment.md`, `../../week2/vulnerability-id/findings.md`, `../../week2/auth-session-analysis/notes.md`.)*

## Cross-Application Pattern Analysis

Looking at all 6 findings together (rather than per-application, as they were originally documented) reveals a pattern that wasn't as visible week-to-week:

- **4 of 6 findings are Injection-class** (SQLi ×2, OS Command Injection, DOM XSS) — despite testing two completely independent applications with different codebases. This suggests input validation/output encoding is a systemic weak point in poorly-secured applications generally, not a one-off bug in either app specifically.
- **The two non-injection findings (Missing CSP, Predictable Session IDs)** both represent *missing defense-in-depth* rather than a direct exploit path — they don't grant access on their own, but they remove a safety net that would have limited the blast radius of the injection findings if properly configured.
- **Only one finding (Command Injection) is rated Critical** — it's also the only one providing direct OS-level code execution rather than data exposure or session compromise, which is why it sits a tier above the rest despite all 6 sharing a similar root cause category.

## Overall Risk Posture

Both applications, as tested, would be considered **high risk** for production deployment in their current state — not because of any single finding, but because multiple independent Injection vulnerabilities alongside missing browser-level and session-level protections compound each other. An attacker succeeding at any one injection point faces almost no secondary barriers (no CSP to contain XSS, no strong session model to limit lateral access).

## Evidence
See individual findings documents linked above; original screenshots preserved in each week's respective folders.
