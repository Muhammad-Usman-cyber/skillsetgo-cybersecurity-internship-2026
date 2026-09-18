# Linux & System-Security Exercises — Notes

## Objective
Audit a Linux system (Kali Linux) for common security-relevant configuration: user/group privileges, SUID binaries, running services/open ports, firewall posture, and authentication logs.

## 1. Users & Groups

**Command:** `whoami && id && cat /etc/passwd | grep -E '/bin/bash|/bin/sh'` and `cat /etc/group | grep sudo`

**Observed:**
- Primary interactive user has standard desktop/security-tooling group memberships (sudo, wireshark, kismet-related groups, etc.).
- Two service accounts have interactive login shells: a PostgreSQL admin account and a Splunk service account, both set to `/bin/bash`.
- Only one user is in the `sudo` group — no unexpected privilege escalation paths via group membership.

**Finding:** Service accounts (PostgreSQL, Splunk) should generally use `/usr/sbin/nologin` rather than `/bin/bash`, since they don't need interactive shell access. Leaving them as `/bin/bash` is a minor hardening gap — if either account's credentials were ever compromised, an attacker gets an interactive shell rather than being blocked outright.

**Recommendation:** Change service-account shells to `nologin` unless interactive access is specifically required for administration.

## 2. File Permissions — SUID Binaries

**Command:** `find / -perm -4000 -type f 2>/dev/null`

**Observed:** A long list of SUID binaries, including standard system tools (`passwd`, `sudo`, `su`, `mount`, `ssh-keygen`, `pkexec`, `gpasswd`, `chfn`) alongside several Kali-specific wireless capture helpers (`kismet_cap_*`) and application sandboxes (`chrome-sandbox`).

**Analysis:** This is a larger-than-usual SUID list because Kali ships with many pentesting tools pre-installed (wireless capture tools in particular need elevated privileges to access network interfaces directly). Each SUID binary is a potential privilege-escalation target if it has a vulnerability, since it always executes with the file owner's (often root's) privileges regardless of who runs it.

**Recommendation:** SUID binaries should be periodically reviewed against a known-good baseline (e.g. a fresh Kali install) so any *new* SUID binary — which could indicate a backdoor or misconfigured install — stands out immediately.

## 3. Running Services & Open Ports

**Commands:** `systemctl list-units --type=service --state=running` and `ss -tulpn`

**Observed:**
- ~26 active services, mostly expected desktop/system services (NetworkManager, cron, dbus, ssh, docker, containerd, udisks2, wpa_supplicant, etc.).
- `ss -tulpn` shows SSH listening on `0.0.0.0:22` (all interfaces), a service on `127.0.0.1:46859` (localhost-only — low risk), a service on `0.0.0.0:7070`, and a UDP listener on `0.0.0.0:50001`.
- `anydesk.service` is running — third-party remote-desktop software.

**Finding:** SSH bound to `0.0.0.0` means it accepts connections from any network the host is on, not just localhost. Combined with AnyDesk running, this system has two independent remote-access paths. Each is a legitimate attack surface if not intentionally configured and secured (strong auth, key-based SSH, AnyDesk access-control settings).

**Recommendation:** Confirm both are intentional. If SSH isn't needed for external access, bind it to `127.0.0.1` or restrict via firewall rules to trusted source IPs. Confirm AnyDesk requires strong authentication and isn't using a default/weak access password.

## 4. Firewall Rules

**Command:** `sudo iptables -L -v`

**Observed:** UFW-managed chains are present. The `INPUT` chain's default policy is `DROP`, and `FORWARD` is also `DROP`; `OUTPUT` defaults to `ACCEPT`. Docker has also inserted its own chains (`DOCKER`, `DOCKER-BRIDGE`, `DOCKER-CT`).

**Analysis:** A default-deny `INPUT` policy is good practice — unsolicited inbound traffic is blocked unless explicitly allowed. This is a positive finding. However, the specific `ufw` allow rules (e.g. whether port 22 is explicitly allowed from anywhere) aren't visible in this raw `iptables` output.

**Recommendation:** Run `sudo ufw status verbose` for a human-readable view of exactly which ports/sources are allowed, and confirm SSH access is scoped as narrowly as possible.

## 5. Authentication / System Logs

**Command:** `sudo tail -n 50 /var/log/auth.log`

**Observed:** Normal desktop session activity — `lightdm` greeter sessions, user login/logout, a `pkexec` privilege-escalation call for a display-brightness helper (legitimate desktop function), and routine `cron` sessions running as root on schedule. No failed password attempts or unrecognized users appear in this window.

**Finding:** No signs of brute-force attempts or unauthorized access in the reviewed log window. This is a clean baseline — useful to compare against later if suspicious activity needs investigating.

## Summary

| Check | Result | Action needed |
|---|---|---|
| Users/groups | Sudo access scoped correctly | Set service accounts to `nologin` |
| SUID binaries | Elevated but consistent with Kali's toolset | Baseline for future comparison |
| Services/ports | SSH + AnyDesk both remotely reachable | Verify intentional, restrict if not |
| Firewall | Default-deny inbound (good) | Confirm explicit allow rules with `ufw status verbose` |
| Auth logs | No suspicious activity | None — clean baseline |

## Evidence
Screenshots for each check are saved in `screenshots/` in this folder:
`01-users-groups.png`, `02-suid-binaries.png`, `03-services-ports.png`, `04-firewall-rules.png`, `05-auth-system-logs.png`
