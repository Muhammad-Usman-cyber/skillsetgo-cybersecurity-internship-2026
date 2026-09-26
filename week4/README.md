# Week 4 — Cybersecurity Project (Capstone)

**Deadline:** 13 October 2026 (confirm with program given Week 1's delay)

## Objective
Perform a full structured security assessment, document risks and controls, create an incident-response procedure, and submit a complete capstone project with a professional security report.

## Deliverables & Status

| # | Task | Evidence type | File(s) | Status |
|---|------|----------------|---------|--------|
| 1 | Structured security assessment | Assessment log | `security-assessment/assessment-log.md` | ✅ Done |
| 2 | Risk & controls documentation | Risk register document | `risk-controls/risk-register.md` | ✅ Done |
| 3 | Incident-response procedure | IR procedure document | `incident-response/ir-procedure.md` | ✅ Done |
| 4 | CAPSTONE: Complete Cybersecurity Project | Full security report (PDF) + supporting evidence | `capstone-report/report.pdf` | ✅ Done |

## Scope
This capstone consolidates evidence from the full internship rather than starting a new target from scratch:
- **Week 1:** OWASP Juice Shop — SQL Injection, Missing CSP, DOM-based XSS
- **Week 2:** DVWA — SQL Injection, OS Command Injection, Predictable Session IDs
- **Week 3:** Personal SOC home lab — Nmap recon, SMB brute-force, encoded PowerShell execution (1 fully evidenced detection)

## Tools used
- OWASP Juice Shop, DVWA (Docker)
- Splunk/Sysmon (home lab, preserved evidence)
- Findings and reports from Weeks 1–3

## Result / Reflection

This capstone pulled together everything from Weeks 1–3 into a single formal deliverable rather than treating each week as isolated work. The most valuable insight only emerged at this consolidation stage: 4 of 6 vulnerabilities across two completely unrelated applications were Injection-class, and every risk in the register carried the same "High Likelihood" rating due to a total absence of layered defenses. Individually, each week's findings looked like isolated bugs — together, they told a much stronger governance-level story about systemic weak points, which is the actual value a real security consultant would be expected to deliver, not just a list of vulnerabilities.
