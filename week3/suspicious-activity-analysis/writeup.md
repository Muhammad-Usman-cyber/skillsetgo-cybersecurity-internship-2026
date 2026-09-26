# Simulated Suspicious Activity Analysis

## Objective
Analyze simulated attack activity and the corresponding host-side logs/alerts to identify suspicious activity patterns, using evidence from a personal SOC home lab (Kali attacker VM against a Windows 10 target, Sysmon telemetry forwarded to Splunk). Note: the lab environment (VirtualBox VMs, Sysmon, Splunk) has since been decommissioned; analysis below is based on preserved screenshots from that work.

## Simulated Technique 1 — Network Reconnaissance (Nmap)

**Attacker-side evidence:**
```
nmap -sV -sC 192.168.56.103
Nmap scan report for 192.168.56.103
Host is up (0.0033s latency).
All 1000 scanned ports on 192.168.56.103 are in ignored states.
Not shown: 1000 filtered tcp ports (no-response)
```

**Technique:** Full TCP port scan with service/version detection against the target host.

**MITRE ATT&CK mapping:** T1046 — Network Service Discovery

**Expected defender-side signature:** A properly tuned detection would flag this via a high volume of distinct destination ports contacted from a single source IP within a short time window — typically surfaced through Sysmon Event ID 3 (Network Connection) events or firewall/IDS logs, aggregated by source IP and counted by unique destination port.

**Detection status:** ⚠️ **Evidence gap** — the attack was executed and confirmed via the attacker-side terminal output, but a corresponding host-side Sysmon/Splunk detection query specific to this scan (e.g. filtered by Event ID 3 and the scan's timestamp) was not captured before the lab environment was decommissioned. This is noted transparently rather than fabricated — a real assessment should never claim detection evidence that wasn't actually captured.

## Simulated Technique 2 — SMB Brute-Force Authentication (Hydra)

**Attacker-side evidence:**
```
hydra -l administrator -P /usr/share/wordlists/rockyou.txt smb://192.168.56.103
[DATA] max 1 task per server, overall 1 task, 14344399 login tries
[DATA] attacking smb://192.168.56.103:445/
```

**Technique:** Automated credential brute-forcing against the SMB service using a large common-password wordlist.

**MITRE ATT&CK mapping:** T1110.001 — Brute Force: Password Guessing

**Expected defender-side signature:** A high volume of failed authentication attempts against the same account/service in a short window — normally surfaced via Windows Security Event Log 4625 (failed logon) rather than Sysmon directly, or via repeated Sysmon Event ID 3 connections to port 445 from a single source at high frequency.

**Detection status:** ⚠️ **Evidence gap** — same limitation as Technique 1: the attack execution is fully evidenced attacker-side, but the corresponding host-side failed-logon or repeated-connection detection query was not preserved from the lab before it was decommissioned.

## Simulated Technique 3 — Encoded PowerShell Execution

**Attacker-side evidence:**
```
PS C:\Users\Muhammad Usman> powershell.exe -enc VwByAGkAdAB1AC0ASABvAHMAdAAgACIASABlAGwAbABvACAAZgByAG8AbQAgAGUAbgBjAG8AZABlAGQAIABQAG8AdwBlAHIAUwBoAGUAbABsACIA
Hello from encoded PowerShell
```

**Technique:** Base64-encoded PowerShell command execution (`-enc` flag) — a common technique for evading simple string/keyword-based detection, since the malicious command text isn't visible in plaintext on the command line.

**MITRE ATT&CK mapping:** T1059.001 — Command and Scripting Interpreter: PowerShell; T1027 — Obfuscated Files or Information

**Defender-side evidence (Sysmon, via Splunk):**
```
Event ID 11 (FileCreate)
Image: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
TargetFilename: C:\Users\Muhammad Usman\AppData\Local\Temp\__PSScriptPolicyTest_kpxpnatu.qgs.ps1
User: DESKTOP-TM6C394\Muhammad Usman
```

**Detection status:** ✅ **Fully evidenced** — this is the one technique with a complete attack-to-detection story. The `__PSScriptPolicyTest_*.ps1` temp file creation is a documented byproduct of PowerShell evaluating script execution policy, and its appearance here — correlated by timestamp and `ProcessGuid` with the attacker-side encoded command — confirms the host-side telemetry successfully captured evidence of the technique, even though the encoded command content itself isn't visible in this specific event (it would require also pulling the corresponding Event ID 1 process-creation log with full command-line logging enabled to see the raw encoded string).

## Analysis Summary

| # | Technique | MITRE ATT&CK | Attack Evidence | Detection Evidence |
|---|-----------|--------------|-------------------|----------------------|
| 1 | Network Reconnaissance (Nmap) | T1046 | ✅ Confirmed | ⚠️ Not preserved |
| 2 | SMB Brute-Force (Hydra) | T1110.001 | ✅ Confirmed | ⚠️ Not preserved |
| 3 | Encoded PowerShell Execution | T1059.001, T1027 | ✅ Confirmed | ✅ Confirmed |

## Reflection

This analysis highlights a real, common gap in SOC work: an attack being executed is not the same as an attack being detected and evidenced. Technique 3 succeeded end-to-end because the specific log-filtering query (`"powershell"`) was run and captured at the time; Techniques 1 and 2 lacked an equivalent targeted query before the lab was torn down. The practical lesson: **capture and export detection evidence immediately after each simulated technique**, rather than assuming the raw log volume alone will be sufficient to reconstruct findings later — 1,001 raw events without a targeted, saved search is not the same as a documented detection.

## Evidence
Screenshots saved in `../screenshots/`: `06-attacks-nmap.png`, `06-attacks-bruteforce.png`, `06-attacks-powershell-encoded.png`, `03-windows-sysmon-running.png` (Sysmon installation/collection confirmation), `05-logs-confirmed.png` (overall log ingestion covering the attack timeframe).
