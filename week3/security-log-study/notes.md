# Security Log Study — Notes

## Objective
Study sample security logs and identify common indicators of compromise (IOCs), using log evidence from a self-directed Splunk/Sysmon SOC home lab built prior to this internship (June–July 2026). The lab environment itself has since been decommissioned; the log evidence and screenshots below are preserved from that work.

## Log Source
- **Log type:** Windows Sysmon (System Monitor) event logs, exported and ingested into Splunk
- **Sourcetype:** `sysmon_export`
- **Volume:** 1,001 total events in the dataset (`index=main sourcetype=sysmon_export`)
- **Collection method:** Sysmon installed on a Windows 10 target VM, logs exported and forwarded into Splunk for search and analysis

## Understanding Sysmon Logs

Sysmon logs system-level activity in far more detail than default Windows Event Logs, using specific Event IDs for different activity types:

| Sysmon Event ID | Meaning |
|---|---|
| 1 | Process creation |
| 3 | Network connection |
| 11 | File created |
| 13 | Registry value set |

Each event includes structured fields — process name/path, parent process, command line, user, hashes, and timestamps — which is what makes Sysmon far more useful for detection than basic OS logging.

## Sample Log Analysis — PowerShell Activity

**Search used:** `index=main sourcetype=sysmon_export "powershell"` — returned 12 matching events.

**Sample event (Event ID 11 — File Created):**
```
Message: "File created"
RuleName: -
ProcessGuid: {7ad35eba-e1a7-6a5d-dc12-00000000f00}
ProcessId: 7332
Image: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
TargetFilename: C:\Users\Muhammad Usman\AppData\Local\Temp\__PSScriptPolicyTest_kpxpnatu.qgs.ps1
User: DESKTOP-TM6C394\Muhammad Usman
```

### What "normal" looks like
PowerShell itself is a legitimate, heavily-used administrative tool built into Windows — its presence in logs is not inherently suspicious. Most PowerShell activity involves running signed scripts or interactive commands with no unusual temp-file behavior.

### What this log shows
This specific event is a **`__PSScriptPolicyTest_*.ps1`** temp file — a well-documented artifact that PowerShell automatically creates in the background whenever it evaluates its script execution policy (e.g. when a script is invoked, especially via automation, encoding, or non-interactive execution). Seeing this artifact repeatedly, especially correlated with encoded or obfuscated command-line activity, is a useful **behavioral IOC** for PowerShell-based execution techniques — including encoded/obfuscated PowerShell commands, a technique commonly used to evade basic string-based detection.

### IOC identified
- **Indicator:** Creation of `__PSScriptPolicyTest_*.ps1` temp files by `powershell.exe`
- **Significance:** A byproduct artifact of PowerShell script-policy evaluation; useful as a supporting indicator (not standalone proof) that PowerShell was invoked to run a script, which is worth correlating with process command-line logging (Event ID 1) to see exactly what was executed.
- **MITRE ATT&CK mapping:** T1059.001 — Command and Scripting Interpreter: PowerShell

## Reflection
Log volume alone (1,001 raw events from a few hours of activity) illustrates why manual log review doesn't scale — this is exactly the motivation for the structured search/correlation workflow built in Deliverable #2, and the dashboard in Deliverable #4.

## Evidence
Screenshots saved in `../screenshots/`:
`07-detections-sysmon-events-table.png` (log volume/structure overview), `07-detection-powershell.png` (PowerShell IOC detail)
