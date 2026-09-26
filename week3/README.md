# Week 3 — SOC & Threat Detection

**Deadline:** 06 October 2026 (confirm with program given Week 1's delay)

## Objective
Study security logs and identify IOCs, build a log-monitoring workflow, analyze simulated suspicious activity, and build a security monitoring dashboard.

## Deliverables & Status

| # | Task | Evidence type | File(s) | Status |
|---|------|----------------|---------|--------|
| 1 | Security log study | Notes document | `security-log-study/notes.md` | ✅ Done |
| 2 | Log-monitoring workflow | Workflow document + sample outputs | `log-monitoring-workflow/workflow.md` | ✅ Done |
| 3 | Simulated suspicious activity analysis | Analysis write-up | `suspicious-activity-analysis/writeup.md` | ✅ Done |
| 4 | Security monitoring dashboard | Dashboard file/screenshots | `monitoring-dashboard/` | ✅ Done |

## Tools used
- Splunk Enterprise (existing SOC home lab: Kali attacker VM to Windows 10 target)
- Sysmon log ingestion
- MITRE ATT&CK mapping

## Setup
Reusing and extending the existing personal SOC home lab (Splunk ingesting Sysmon logs, 3 simulated attack techniques: Nmap recon, SMB brute-force, encoded PowerShell execution).

## Result / Reflection

This week's work highlighted the difference between running an attack and actually proving it was detected. Of 3 simulated techniques (Nmap recon, SMB brute-force, encoded PowerShell execution), only the PowerShell one has a complete attack-to-detection story, because a targeted Splunk search was run and captured for it at the time — the other two lacked an equivalent saved query before the lab was decommissioned. The lesson: capture and export detection evidence immediately after each simulated technique rather than assuming raw log volume can be reconstructed into findings later. This gap is documented transparently across the log study, workflow, and suspicious-activity docs rather than papered over, which is itself realistic SOC practice — not every alert gets fully investigated in time, and knowing what wasn't covered is as important as what was.
