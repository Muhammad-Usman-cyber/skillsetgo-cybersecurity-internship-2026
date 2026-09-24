# Week 2 — Ethical Hacking & Web Security

**Deadline:** 29 September 2026 (confirm with program — Week 1 was delayed/moved to 25 Sep)

## Objective
Study the OWASP Top 10, identify vulnerabilities in a legal practice environment, analyze authentication/session weaknesses, and produce a structured penetration-testing style report.

## Deliverables & Status

| # | Task | Evidence type | File(s) | Status |
|---|------|----------------|---------|--------|
| 1 | OWASP Top 10 study | Summary document | `owasp-top10-study/summary.md` | ✅ Done |
| 2 | Vulnerability identification (legal practice env) | Findings log + screenshots | `vulnerability-id/findings.md`, `vulnerability-id/screenshots/` | ✅ Done |
| 3 | Authentication/session weakness analysis | Analysis notes | `auth-session-analysis/notes.md` | ✅ Done |
| 4 | Penetration-testing style report | Structured PDF report | `pentest-report/report.pdf` | ✅ Done |

## Tools used
- DVWA (Damn Vulnerable Web Application), local Docker container
- Browser DevTools (Network tab, source inspection)
- Manual payload testing (no automated scanner used this week)

## Setup
1. `docker run -d -p 8080:80 --name dvwa vulnerables/web-dvwa`
2. Initialize the database via the "Create/Reset Database" button, log in with `admin` / `password`, set DVWA Security to Low.
3. This week's testing continues from — but does not repeat — Week 1's Juice Shop work; see `../week1/vulnerability-id/findings.md` for that prior evidence (Injection, Security Misconfiguration).

## Findings summary (this week)
- SQL Injection (User ID field) — `vulnerability-id/findings.md`
- OS Command Injection (ping field) — `vulnerability-id/findings.md`
- Predictable session ID generation (`md5()` of an incrementing counter) — `auth-session-analysis/notes.md`

## Result / Reflection
Week 2 produced 3 validated vulnerabilities (SQL Injection, OS Command Injection, Predictable Session IDs) across DVWA, spanning 3 distinct OWASP Top 10:2025 categories (Injection, Authentication Failures, Cryptographic Failures). Combined with Week 1's Juice Shop findings, this internship has now demonstrated hands-on evidence across 4 different OWASP categories rather than repeating the same vulnerability class.

The most valuable lesson was in the session-ID finding specifically: reading the actual server-side source code (rather than just observing behavior) revealed the true root cause — an MD5 hash with no random input — which a black-box test alone might have only flagged as "suspicious" without being able to explain *why* it was weak or how confidently exploitable it was. Next time, I'd prioritize source review earlier in the process wherever it's available, rather than treating it as a final confirmation step.
