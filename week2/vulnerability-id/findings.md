# Vulnerability Identification — Findings

## Objective
Identify vulnerabilities in a legal practice environment (DVWA — Damn Vulnerable Web Application, run locally via Docker) through manual testing.

## Target
- **Application:** DVWA (Damn Vulnerable Web Application) v1.10 *Development*
- **Environment:** Local Docker container, `localhost:8080`
- **Security level:** Low (deliberately lowered to demonstrate vulnerabilities clearly)

## Finding 1 — SQL Injection

**Where:** SQL Injection module — User ID field

**Evidence:** Submitting the payload `%' or '1'='1` into a field intended to look up a single user by ID returned **all 5 users** in the database (admin, Gordon Brown, Hack Me, Pablo Picasso, Bob Smith) instead of one.

**Analysis:** The application concatenates user input directly into a SQL `WHERE` clause without parameterization. The payload's `OR '1'='1'` clause is always true, so the query matches every row in the table regardless of the intended filter.

**OWASP mapping:** A03:2021 — Injection

**Potential impact:** An attacker could enumerate the entire user table, and depending on the query's structure elsewhere, potentially extend this into extracting other tables (usernames, password hashes) via UNION-based injection.

**Recommended fix:** Use parameterized queries/prepared statements for all database access; never concatenate raw user input into SQL strings.

## Finding 2 — Command Injection

**Where:** Command Injection module — "Ping a device" IP address field

**Evidence:** Submitting `127.0.0.1 && whoami` returned the normal ping output for `127.0.0.1`, followed by an extra line: `www-data` — the output of the injected `whoami` command.

**Analysis:** The application passes the IP address field directly into a system shell command (likely `ping <input>`) without validating that the input is actually a well-formed IP address, or without stripping shell metacharacters (`&&`, `;`, `|`). This allows arbitrary command chaining.

**OWASP mapping:** A03:2021 — Injection (OS Command Injection)

**Potential impact:** Since the output revealed the web server runs as `www-data`, an attacker could potentially execute any command that user has permissions for — reading files, establishing a reverse shell, or pivoting further into the host, depending on further privilege boundaries.

**Recommended fix:** Avoid passing user input to shell commands entirely where possible; if unavoidable, use strict allow-list validation (e.g. regex-match a valid IPv4/IPv6 format) and use language-level APIs that avoid shell interpretation (e.g. passing arguments as an array rather than a concatenated string).

## Summary Table

| # | Finding | OWASP Category | Severity |
|---|---------|-----------------|----------|
| 1 | SQL Injection (User ID lookup) | A03:2021 Injection | High |
| 2 | OS Command Injection (ping field) | A03:2021 Injection | Critical |

## Tools Used
- Manual payload testing via DVWA's own input forms
- Browser (no additional tooling required for these two findings)

## Evidence
Screenshots saved in `screenshots/`: SQL Injection result, Command Injection result (`www-data` output following ping).
