# Incident-Response Procedure — Suspicious Encoded PowerShell Execution

## Scenario
This procedure is built around a simulated attack scenario from this internship's SOC home lab: a host generates a Sysmon FileCreate event showing `powershell.exe` creating a `__PSScriptPolicyTest_*.ps1` temp file — an artifact consistent with base64-encoded PowerShell execution (MITRE ATT&CK T1059.001, T1027), a common technique for evading simple keyword-based detection.

## 1. Preparation
*(Controls that should already be in place before an incident occurs)*
- Sysmon deployed on endpoints with a configuration that logs process creation (Event ID 1) and file creation (Event ID 11), forwarded to a central log platform (Splunk in this lab).
- A documented, saved search/alert for PowerShell-related temp-file artifacts and `-enc`/`-EncodedCommand` usage in process command lines.
- A designated on-call analyst or escalation path for triaging alerts.

## 2. Identification
**Trigger:** A Sysmon Event ID 11 alert fires for a file matching the pattern `__PSScriptPolicyTest_*.ps1` in a user's Temp directory, OR a Sysmon Event ID 1 process-creation event shows `powershell.exe` invoked with `-enc`/`-EncodedCommand`.

**Analyst actions:**
1. Pull the full event: `ProcessGuid`, `Image`, `TargetFilename`, `User`, `UtcTime`.
2. Correlate by `ProcessGuid` to find the parent process-creation event and recover the encoded command line if command-line logging is enabled.
3. Decode any Base64 command found (`-enc` payloads are UTF-16LE Base64) to determine actual intent.
4. Check whether this activity is expected (e.g. a known admin script, a scheduled task) or has no legitimate explanation for this user/host.
5. Classify: if unexplained, escalate to a confirmed incident. If explained, document as a false positive and close.

## 3. Containment
**Short-term:**
- Isolate the affected host from the network (disable network adapter or apply an isolation ACL) to prevent lateral movement or C2 communication while investigation continues.
- Suspend or disable the involved user account if credential compromise is suspected.

**Long-term:**
- Block the specific encoded command hash / associated indicators (destination IPs/domains if the decoded command references any) at the network perimeter.

## 4. Eradication
- Terminate the malicious PowerShell process if still running.
- Remove the dropped temp file and any additional payloads/persistence mechanisms it may have created (scheduled tasks, registry run keys, new services) — check these locations specifically since encoded PowerShell is commonly used to establish persistence.
- Reset credentials for the affected account.

## 5. Recovery
- Re-enable the host on the network only after confirming no persistence mechanisms remain and endpoint protection signatures are current.
- Re-enable the user account with a new password and, if available, enforce MFA re-enrollment.
- Monitor the host and account for a defined period (e.g. 72 hours) with heightened alerting sensitivity.

## 6. Lessons Learned
*(Post-incident review questions)*
- Was the encoded PowerShell execution detected in a timely manner, or only found during retrospective log review? *(In this lab, detection relied on a manually-run targeted search rather than a pre-configured alert — see Week 3's `../../week3/log-monitoring-workflow/workflow.md` for the related workflow gap identified.)*
- Was full command-line logging enabled to allow decoding the actual payload, or only the file-creation artifact?
- Should PowerShell Constrained Language Mode or script block logging be enabled to reduce the effectiveness of this technique going forward?

## Reference
This procedure directly builds on evidence and gaps documented in Week 3:
- `../../week3/security-log-study/notes.md` — original IOC identification
- `../../week3/suspicious-activity-analysis/writeup.md` — the technique's full attack-to-detection analysis, including the honestly-documented gap in detection evidence for the other two simulated techniques (Nmap, Hydra), which this procedure's "Preparation" section is designed to help close going forward.
