# SOC Overview — Home Lab Dashboard Summary

**Source:** Personal SOC home lab (Kali attacker VM → Windows 10 target, Sysmon + Splunk)
**Lab status:** Decommissioned — data below preserved from 2026-07-20 session
**Program:** Skill Set Go Cybersecurity Internship — Week 3

## Key Metrics

| Metric | Value |
|---|---|
| Sysmon events ingested | 1,001 |
| Attack techniques simulated | 3 |
| Fully evidenced detections | 1 |
| MITRE ATT&CK techniques mapped | 3 |

## Simulated Activity & Detection Status

| Technique | MITRE ATT&CK | Attack Evidence | Detection Evidence |
|---|---|---|---|
| Network Reconnaissance (Nmap full port scan) | T1046 | ✅ Confirmed | ⚠️ Gap |
| SMB Brute-Force Authentication (Hydra) | T1110.001 | ✅ Confirmed | ⚠️ Gap |
| Encoded PowerShell Execution | T1059.001 / T1027 | ✅ Confirmed | ✅ Confirmed |

## Fully Evidenced Detection — Encoded PowerShell Execution

- **Sourcetype:** `sysmon_export` — **Event ID:** 11 (FileCreate)
- **Image:** `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`
- **TargetFilename:** `...\Temp\__PSScriptPolicyTest_kpxpnatu.qgs.ps1`
- **User:** `DESKTOP-TM6C394\Muhammad Usman`

This temp-file artifact is a documented byproduct of PowerShell evaluating script execution policy — a useful supporting IOC for script-based execution, correlated here with the attacker-side encoded `-enc` command by timestamp and host. Full walkthrough in `../security-log-study/notes.md`.

## Known Evidence Gap

The Nmap and Hydra techniques were executed and confirmed attacker-side, but the corresponding host-side detection queries (Sysmon Event ID 3 for the port scan; repeated authentication-failure events for the brute force) were not run and saved before the lab environment was decommissioned. This gap is recorded here deliberately rather than fabricated — full reasoning in `../suspicious-activity-analysis/writeup.md`.

## Evidence
Screenshot: `../screenshots/08-dashboard-soc-overview.png` (raw Splunk log view from the home lab).
