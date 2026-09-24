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
_(fill in once the Week 2 PDF report is complete)_
