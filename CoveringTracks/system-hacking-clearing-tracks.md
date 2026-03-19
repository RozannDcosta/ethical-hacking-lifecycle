**Note:** This lab is Part 3 of 3 Continues from the persistent access established in 
> - Phase 5 – [`system-hacking-maintaining-access.md`](../MaintainingAccess/system-hacking-maintaining-access.md)
---

System Hacking: Clearing Tracks
---

## Objective

Using the existing root shell on the target, remove or zero out evidence of the intrusion by clearing bash command history, authentication logs, system logs, and login records. Demonstrate the anti-forensics techniques an attacker uses to reduce the forensic footprint left by the exploitation, persistence, and post-exploitation activities performed in Phases 3 and 4.

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Bash built-ins (history, cat, echo) | History and log manipulation |
| /var/log/ system logs | Target log files cleared |
| wtmp / btmp | Login record files zeroed |

---

## Lab Environment

| Role | Machine | IP Address |
|------|---------|-----------|
| Attacker | Kali Linux | 192.168.23.130 |
| Target | Metasploitable2 | 192.168.23.131 |

All clearing operations were executed from within the root shell session established in Phase 3. Root access was required for clearing protected log files under `/var/log/`.

---

## Background

After gaining access and establishing persistence, attackers attempt to reduce their forensic footprint to delay or prevent detection and incident response. On Linux systems, the primary evidence sources include shell command history, authentication logs, syslog, and binary login records (`wtmp`, `btmp`). Clearing or zeroing these files removes evidence of the attacker's commands, login events, and shell activity.

Note: True anti-forensics on a production system is significantly more complex. Logs may be shipped to a remote SIEM in real-time, making local clearing ineffective after the fact. This lab demonstrates the technique against a locally logging system as documented in CEH Module 6.

---

## Steps Performed

### Step 1 — Clear Bash Command History

The in-memory command history for the current session was cleared, and the `.bash_history` file on disk was wiped to prevent recovery of commands run during the intrusion.

```bash
history -c
cat /dev/null > ~/.bash_history
```

<img width="700" height="400" alt="history -c and cat /dev/null to bash_history commands executed in root shell" src="YOUR_SCREENSHOT_URL" />

**Result:** In-memory history flushed with `history -c`. On-disk history file `/root/.bash_history` overwritten with zero bytes via `/dev/null` redirect.

---

### Step 2 — Clear Authentication and System Logs

Authentication logs (`auth.log`) and system daemon logs (`syslog`) were zeroed out. These files contain records of the FTP connection, shell spawn, and any sudo activity.

```bash
cat /dev/null > /var/log/auth.log
cat /dev/null > /var/log/syslog
```

<img width="700" height="400" alt="cat /dev/null redirected to auth.log and syslog zeroing both files" src="YOUR_SCREENSHOT_URL" />

**Result:** Both `/var/log/auth.log` and `/var/log/syslog` reduced to 0 bytes. FTP authentication events, shell spawn records, and session activity removed.

---

### Step 3 — Zero Login Record Files

The binary `wtmp` and `btmp` files — which record successful and failed login events respectively — were zeroed out to remove evidence of any login sessions during the attack window.

```bash
echo "" > /var/log/wtmp
echo "" > /var/log/btmp
```

<img width="700" height="400" alt="echo empty string redirected to wtmp and btmp zeroing both binary login record files" src="YOUR_SCREENSHOT_URL" />

**Result:** `wtmp` and `btmp` both reduced to 1 byte (newline). The `last` and `lastb` commands will no longer return historical login records from the attack window.

---

### Step 4 — Verify Log State

The `/var/log/` directory was listed to confirm which files were successfully zeroed and to document the state of remaining logs.

```bash
ls -la /var/log/
```

<img width="700" height="400" alt="ls -la /var/log showing auth.log syslog wtmp and btmp all at 0 or 1 bytes after clearing" src="YOUR_SCREENSHOT_URL" />

**Result:** Confirmed:

| Log File | Size After Clearing | Status |
|---------|-------------------|--------|
| auth.log | 0 bytes | Cleared |
| syslog | 0 bytes | Cleared |
| wtmp | 1 byte | Zeroed |
| btmp | 1 byte | Zeroed |

Other logs such as `kern.log`, `daemon.log`, `messages`, `vsftpd.log`, and `debug` remained present and unmodified — these were not targeted in this lab. A complete track-clearing engagement would address all relevant log files.

---

## Commands Used

```bash
# Clear in-memory bash history
history -c

# Wipe on-disk bash history file
cat /dev/null > ~/.bash_history

# Zero authentication log
cat /dev/null > /var/log/auth.log

# Zero system daemon log
cat /dev/null > /var/log/syslog

# Zero successful login records
echo "" > /var/log/wtmp

# Zero failed login records
echo "" > /var/log/btmp

# Verify log state after clearing
ls -la /var/log/
```

---

## Observations

- All targeted log files were successfully zeroed from within the root shell. Root access is a prerequisite for modifying files under `/var/log/` on this system.
- The `ls -la /var/log/` output confirmed `auth.log` and `syslog` at 0 bytes and `wtmp`/`btmp` at 1 byte each, consistent with successful clearing.
- Several logs were observed to remain uncleared, including `kern.log` (315KB), `daemon.log` (79KB), `messages` (227KB), and `vsftpd.log` (1.3KB). In a thorough anti-forensics scenario these would also be addressed or selectively trimmed.
- The vsftpd.log file in particular would contain timestamped evidence of the backdoor FTP connection and was noted as remaining on disk.
- On systems with remote syslog forwarding or a SIEM agent, local log clearing is ineffective for events already shipped to the remote collector. This is a critical limitation of local anti-forensics techniques.
- Zeroing files rather than deleting them avoids the file-not-found anomaly that deletion would create and preserves normal file system structure.

---

## Security Relevance

| Risk | Detail |
|------|--------|
| Local log clearing is trivially simple with root access | Any attacker with root can zero logs in seconds |
| Binary login records are easily corrupted | wtmp/btmp have no integrity protection — zeroing them removes all login history |
| bash_history is unreliable as an evidence source | It can be suppressed entirely via HISTSIZE=0 or unset HISTFILE before any commands are run |
| Remaining logs (vsftpd.log, kern.log) still contained evidence | Track clearing in this lab was partial, not complete |

Defenders should forward logs in real-time to an immutable remote SIEM or log aggregator (e.g., Splunk, Elastic, Wazuh). Auditd with rules targeting writes to `/var/log/` and modifications to `.bash_history` will generate alerts before clearing is complete. File integrity monitoring on log files using AIDE or similar tools can detect unexpected zero-byte conditions. The presence of a 0-byte `auth.log` or `syslog` on a running system is itself a high-confidence indicator of compromise.

---

## Disclaimer

This lab was performed exclusively on Metasploitable2, an intentionally vulnerable virtual machine designed for security training. Both attacker and target machines were isolated on a host-only internal network with no internet routing. No unauthorized systems were targeted at any point. All techniques demonstrated are for educational purposes only and ethical hacking portfolio development. Performing these techniques against systems without explicit written authorization is illegal under the Computer Fraud and Abuse Act (CFAA) and equivalent legislation in other jurisdictions.
