# Risk Register & Recommended Security Controls

## Objective
Document identified risks from the structured security assessment (`../security-assessment/assessment-log.md`) in a formal risk register, and recommend appropriate security controls for each.

## Risk Register

| Risk ID | Description | Likelihood | Impact | Risk Rating | Existing Controls | Recommended Controls | Status |
|---|---|---|---|---|---|---|---|
| RISK-01 | SQL Injection allows unauthorized data access/enumeration via unsanitized search input (Juice Shop) | High | High | **High** | None observed | Parameterized queries; least-privilege DB account; WAF as defense-in-depth | Open |
| RISK-02 | Missing Content-Security-Policy header allows unrestricted script execution if injection occurs | High | Medium | **Medium** | None observed | Implement strict CSP (`script-src 'self'`); avoid inline scripts | Open |
| RISK-03 | DOM-based XSS via unsanitized search parameter enables session/credential theft | High | High | **High** | None observed | Output encoding (e.g. DOMPurify); avoid `innerHTML` with untrusted data; combine with RISK-02's CSP fix | Open |
| RISK-04 | SQL Injection in user lookup enables full user table enumeration (DVWA) | High | High | **High** | None observed | Parameterized queries/prepared statements; input validation | Open |
| RISK-05 | OS Command Injection allows arbitrary command execution as the web server user | High | Critical | **Critical** | None observed | Eliminate shell invocation from user input; allow-list validation; run service with minimal OS privileges | Open |
| RISK-06 | Predictable session IDs (MD5 of incrementing counter) allow session hijacking without credentials | High | High | **High** | None observed | Cryptographically secure random token generation; `HttpOnly`/`Secure`/`SameSite` cookie flags | Open |

## Risk Heat Map (Likelihood × Impact)

| | Low Impact | Medium Impact | High Impact | Critical Impact |
|---|---|---|---|---|
| **High Likelihood** | — | RISK-02 | RISK-01, RISK-03, RISK-04, RISK-06 | RISK-05 |
| **Medium Likelihood** | — | — | — | — |
| **Low Likelihood** | — | — | — | — |

*(All 6 risks cluster in the "High Likelihood" row, since every finding was successfully and repeatably exploited during testing with no rate-limiting or detection observed — this itself is a notable pattern worth flagging to stakeholders: exploitability was never in question for any finding, only severity varied.)*

## Recommended Control Priorities

| Priority | Control | Risk(s) Addressed | Control Type |
|---|---|---|---|
| 1 | Parameterized queries / ORM for all database access | RISK-01, RISK-04 | Preventive |
| 2 | Eliminate shell command construction from user input | RISK-05 | Preventive |
| 3 | Cryptographically secure session token generation | RISK-06 | Preventive |
| 4 | Output encoding / sanitization for all rendered user input | RISK-03 | Preventive |
| 5 | Content-Security-Policy header | RISK-02, RISK-03 | Detective/Compensating |
| 6 | Secure cookie flags (`HttpOnly`, `Secure`, `SameSite`) | RISK-06 | Preventive |
| 7 | Centralized logging/alerting on repeated injection-pattern requests | All | Detective (ties to Week 3 monitoring workflow) |

## Reflection
Structuring these findings as a formal risk register (rather than a flat findings list) surfaces something the findings-by-week format didn't: every single risk here shares the same "High Likelihood" rating, because none of the tested applications had any preventive or detective control in place to slow an attacker down. In a real environment, this pattern — total absence of layered defense — would itself be escalated as a standalone governance finding, separate from any individual vulnerability.
