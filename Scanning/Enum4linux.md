# Enum4linux - SMB/Windows Enumeration

## Objective
To perform comprehensive SMB enumeration against a vulnerable target (Metasploitable2) using Enum4linux, extracting OS information, user accounts, SMB shares, group memberships, password policy, and SID/RID data through unauthenticated null session access - demonstrating the full information disclosure risk of misconfigured SMB services.

---

## Tools Used
- Enum4linux v0.9.1 (command-line, Kali Linux)
  
---

## Lab Environment
- Operating System: Kali Linux (attacker)
- Tool: Enum4linux v0.9.1
- Target: Metasploitable2 - `192.168.23.131`

---

## Background
Enum4linux is a Linux-based tool for enumerating information from Windows and Samba SMB servers. It is a wrapper around several Samba utilities - `rpcclient`, `net`, `nmblookup`, and `smbclient` - combining their output into a single structured enumeration run. It replicates the functionality of the Windows `enum.exe` tool and adds RID cycling capabilities for user discovery.

SMB (Server Message Block) is a network file sharing protocol used by Windows and Linux Samba servers for sharing files, printers, and other resources. When SMB is misconfigured to allow null sessions - unauthenticated connections using a blank username and password - an attacker can enumerate a significant amount of information about the target system without any credentials at all. This includes user account lists, group memberships, share names and access levels, password policies, and domain SID information.

In the ethical hacking lifecycle, SMB enumeration sits in the Scanning and Enumeration phase and directly feeds the Exploitation phase - the user list discovered via RID cycling can be used for password spraying, the share access levels reveal directly accessible file systems, and the weak password policy confirms that brute force attacks are likely to succeed.

---

## Scans Performed

### Scan 1: Version and Help Check
```bash
enum4linux -h
```

<img width="837" height="901" alt="image" src="https://github.com/user-attachments/assets/697c9f4b-8529-4c24-94bb-306998d61a60" />

**Finding:** Enum4linux v0.9.1 confirmed installed. The help output revealed the full flag set available:

| Flag | Purpose |
|---|---|
| `-U` | Get user list |
| `-M` | Get machine list |
| `-S` | Get share list |
| `-P` | Get password policy information |
| `-G` | Get group and member list |
| `-r` | Enumerate users via RID cycling |
| `-o` | Get OS information |
| `-a` | Run all simple enumeration (-U -S -G -P -r -o -n -i) |
| `-n` | nmblookup (similar to nbtstat) |
| `-v` | Verbose - show full commands being run |
| `-A` | Aggressive - do write checks on shares |

Enum4linux is described as a simple wrapper around the Samba package tools, requiring `rpcclient`, `net`, `nmblookup`, and `smbclient` as dependencies - all present on Kali Linux by default.

---

### Scan 2: OS Information Enumeration
```bash
enum4linux -o 192.168.23.131
```

<img width="838" height="855" alt="image" src="https://github.com/user-attachments/assets/e5ebd828-44ce-4736-a889-b651a5428f14" />

**Finding:** The OS scan successfully connected to the target and retrieved detailed system information through a null session:

| Field | Value |
|---|---|
| Target | 192.168.23.131 |
| RID Range | 500-550, 1000-1050 |
| Username | '' (null) |
| Password | '' (null) |
| Known Usernames | administrator, guest, krbtgt, domain admins, root, bin, none |
| Domain/Workgroup | WORKGROUP |
| Domain SID | NULL SID |
| OS Info (srvinfo) | METASPLOITABLE Wk Sv PrQ Unx NT SNT metasploitable server |
| Samba Version | Samba 3.0.20-Debian |
| Platform ID | 500 |
| OS Version | 4.9 |
| Server Type | 0x9a03 |

**Critical finding:** `[+] Server 192.168.23.131 allows sessions using username '', password ''` - null session access confirmed. This means the target accepts unauthenticated SMB connections, enabling all subsequent enumeration without any credentials. Samba 3.0.20 is a severely outdated version (released 2007) with multiple known critical CVEs including remote code execution vulnerabilities.

---

### Scan 3: User Enumeration
```bash
enum4linux -U 192.168.23.131
```

<img width="837" height="646" alt="image" src="https://github.com/user-attachments/assets/91e86499-7e9b-4ab2-92dd-5898f707a1f0" />

**Finding:** User enumeration via null session returned **35 user accounts** from the target. Key accounts discovered:

| Index | RID | Account | Description |
|---|---|---|---|
| 0x6 | 0xbba | user | just a user, 111 |
| 0x8 | 0x3e8 | root | root |
| 0xa | 0x4c0 | postgres | PostgreSQL administrator |
| 0x14 | 0x4c2 | mysql | MySQL Server |
| 0x18 | 0xbb8 | msfadmin | msfadmin |
| 0x19 | 0x3ee | telnetd | (null) |
| 0x20 | 0x4c4 | ftp | (null) |
| 0x21 | 0x4c4 | tomcat55 | (null) |
| 0x7 | 0x42a | www-data | www-data |
| 0xf | 0x4b2 | dhcp | (null) |
| 0xe | 0x4ca | proftpd | (null) |

The complete user list was extracted without any authentication. The presence of `root`, `msfadmin`, `postgres`, `mysql`, `tomcat55`, and `www-data` accounts reveals the full service stack running on the target. In a real engagement, this user list would be directly fed into password spraying and brute force attacks.

---

### Scan 4: Share Enumeration
```bash
enum4linux -S 192.168.23.131
```

<img width="846" height="523" alt="image" src="https://github.com/user-attachments/assets/09a27782-6950-440e-b60e-1f23f8fcdb8d" />

**Finding:** Share enumeration discovered **5 SMB shares** and tested access to each:

| Sharename | Type | Comment | Mapping | Listing |
|---|---|---|---|---|
| print$ | Disk | Printer Drivers | DENIED | N/A |
| tmp | Disk | oh noes! | OK | OK |
| opt | Disk | (none) | DENIED | N/A |
| IPC$ | IPC | IPC Service (metasploitable server) | N/A | N/A |
| ADMIN$ | IPC | IPC Service (metasploitable server) | DENIED | N/A |

**Critical finding:** The `tmp` share is **fully accessible without authentication** - Mapping: OK, Listing: OK. This means an unauthenticated attacker can browse the contents of the `/tmp` directory on the target server directly over SMB. The comment "oh noes!" is the Metasploitable2 developer's acknowledgment of this intentional vulnerability. Workgroup master confirmed as `METASPLOITABLE`.

---

### Scan 5: Group Enumeration
```bash
enum4linux -G 192.168.23.131
```

<img width="837" height="378" alt="image" src="https://github.com/user-attachments/assets/c8b69b5a-c61b-4800-a833-2a0743f4cd4d" />

**Finding:** The group enumeration scan completed without returning any group data - builtin groups, local groups, domain groups, and all associated memberships returned empty. This is consistent with Samba 3.0.20's behavior where group enumeration via `rpcclient` may not return results even when null sessions are permitted, depending on the specific Samba configuration. The absence of group data does not indicate the scan failed - it indicates the Samba service on this version does not expose group information through this enumeration method. User-to-group mappings were instead captured through RID cycling in Scan 7.

---

### Scan 6: Password Policy Enumeration
```bash
enum4linux -P 192.168.23.131
```

<img width="845" height="704" alt="image" src="https://github.com/user-attachments/assets/7751bdde-eb54-4c08-a131-0bc8a91cd7e8" />

**Finding:** Password policy was successfully retrieved via null session attachment to the IPC$ share using protocol 139/SMB. Two domains were identified - METASPLOITABLE and Builtin:

| Policy Setting | Value |
|---|---|
| Minimum Password Length | 5 (domain) / 0 (rpcclient) |
| Password History Length | None |
| Maximum Password Age | Not Set |
| Password Complexity Flags | 000000 (all disabled) |
| Domain Refuse Password Change | 0 |
| Domain Password Store Cleartext | 0 |
| Domain Password Lockout Admins | 0 |
| Domain Password No Clear Change | 0 |
| Domain Password No Anon Change | 0 |
| Domain Password Complex | 0 |
| Minimum Password Age | None |
| Reset Account Lockout Counter | 30 minutes |
| Locked Account Duration | 30 minutes |
| Account Lockout Threshold | None |
| Forced Log off Time | Not Set |
| Password Complexity (rpcclient) | Disabled |
| Minimum Password Length (rpcclient) | 0 |

**Critical finding:** Password complexity is completely disabled, there is no account lockout threshold, and the minimum password length is effectively 0. This means brute force and password spraying attacks can run indefinitely with no lockout risk, and passwords of any length and complexity (including blank passwords) are accepted by the system.

---

### Scan 7: RID Cycling - User Enumeration via SID/RID Brute Force
```bash
enum4linux -r 192.168.23.131
```

<img width="854" height="763" alt="image" src="https://github.com/user-attachments/assets/4bece43f-dbd1-4634-9a57-b4ccdb6d5ec1" />

**Finding:** RID cycling successfully discovered the domain SID and enumerated users and groups across RID ranges 500-550 and 1000-1050:

**Domain SID discovered:** `S-1-5-21-1042354039-2475377354-766472396`

Key accounts and groups enumerated via RID cycling:

| RID | Account | Type |
|---|---|---|
| 500 | METASPLOITABLE\Administrator | Local User |
| 501 | METASPLOITABLE\nobody | Local User |
| 512 | METASPLOITABLE\Domain Admins | Domain Group |
| 513 | METASPLOITABLE\Domain Users | Domain Group |
| 514 | METASPLOITABLE\Domain Guests | Domain Group |
| 1000 | METASPLOITABLE\root | Local User |
| 1001 | METASPLOITABLE\root | Domain Group |
| 1002 | METASPLOITABLE\daemon | Local User |
| 1004 | METASPLOITABLE\bin | Local User |
| 1006 | METASPLOITABLE\sys | Local User |
| 1008 | METASPLOITABLE\sync | Local User |
| 1012 | METASPLOITABLE\man | Local User |
| 1016 | METASPLOITABLE\mail | Local User |
| 1018 | METASPLOITABLE\news | Local User |
| 1020 | METASPLOITABLE\uucp | Local User |
| 1026 | METASPLOITABLE\proxy | Local User |

RID cycling works by iterating through numeric RID values appended to the discovered domain SID and requesting the name associated with each SID - essentially brute forcing the user and group list by SID number. This technique works independently of standard user enumeration and can recover accounts that are hidden from the `-U` flag output.

---

## Commands Used

| # | Command | Purpose |
|---|---|---|
| 1 | `enum4linux -h` | Version check and flag reference |
| 2 | `enum4linux -o 192.168.23.131` | OS and domain information |
| 3 | `enum4linux -U 192.168.23.131` | User account enumeration |
| 4 | `enum4linux -S 192.168.23.131` | SMB share enumeration and access testing |
| 5 | `enum4linux -G 192.168.23.131` | Group enumeration |
| 6 | `enum4linux -P 192.168.23.131` | Password policy retrieval |
| 7 | `enum4linux -r 192.168.23.131` | RID cycling - SID/RID brute force user enumeration |

---

## Observations

All scans were performed against Metasploitable2 at `192.168.23.131` running Samba 3.0.20-Debian. The following critical findings were confirmed:

**Null Session Access Confirmed:** The target accepted SMB connections with a blank username and blank password - the most severe SMB misconfiguration possible. All enumeration in this lab was performed with zero credentials.

**35 User Accounts Exposed:** The complete user list including `root`, `msfadmin`, `postgres`, `mysql`, `tomcat55`, `www-data`, and `proftpd` was extracted unauthenticated, revealing the entire service stack.

**Accessible SMB Share:** The `tmp` share was fully accessible without authentication - Mapping: OK, Listing: OK - providing direct unauthenticated read access to the server's `/tmp` directory.

**Critically Weak Password Policy:** No complexity requirements, no lockout threshold, minimum length of 0 - meaning brute force attacks can run indefinitely with no defensive response from the system.

**Domain SID Discovered:** `S-1-5-21-1042354039-2475377354-766472396` - the full domain SID was extracted, enabling targeted SID-based attacks and precise RID cycling across the full user and group space.

**RID Cycling Successful:** Local users and domain groups enumerated across RID ranges 500-550 and 1000-1050, confirming Administrator (RID 500), Domain Admins (RID 512), and dozens of local users and service accounts.

**Outdated Samba Version:** Samba 3.0.20 was released in 2007. This version is associated with critical CVEs including CVE-2007-2447 (Samba usermap script remote code execution) - one of the most commonly exploited vulnerabilities in Metasploit labs.

---

## Security Relevance

**Null Sessions - The Root Cause:** Every finding in this lab was possible because the Samba service was configured to allow null sessions - unauthenticated SMB connections. This configuration was common in older Windows NT and Samba deployments but has been disabled by default in all modern Windows versions since Windows XP SP2. On Linux Samba servers, null sessions are controlled by the `restrict anonymous` parameter in `smb.conf`. Setting `restrict anonymous = 2` prevents all anonymous enumeration.

**The Attack Chain from Enumeration to Exploitation:** The data gathered in this lab directly enables the next phase of an attack. The user list from `-U` and `-r` feeds directly into password spraying - trying common passwords like `msfadmin`, `password`, `123456` against each discovered account. The weak password policy (no lockout, no complexity) means this can be done aggressively. The accessible `tmp` share provides a staging area for uploading tools or payloads. The Samba 3.0.20 version maps directly to CVE-2007-2447 in Metasploit, providing a one-command remote root shell.

**RID Cycling vs Standard User Enumeration:** The `-U` flag uses standard SAM enumeration which can be restricted by server configuration. RID cycling (`-r`) is a lower-level technique that works by directly requesting SID translations - it bypasses some enumeration restrictions and can reveal accounts hidden from standard queries. In real engagements, both methods should always be run as they may return different results.

**Password Policy as an Attack Enabler:** The password policy findings directly inform attack strategy. No lockout threshold means brute force can run indefinitely. No complexity requirement means simple wordlists will work. Minimum length of 0 means blank passwords are technically valid. A defender seeing these policy settings should immediately enforce a minimum 12-character password, enable complexity requirements, and set an account lockout threshold of 5-10 attempts.

**SMB Shares as Lateral Movement Paths:** In a real network, accessible SMB shares are primary lateral movement vectors. An attacker with access to a writable share can plant malicious files, drop scripts for execution, or exfiltrate data. The `tmp` share being world-readable without authentication on a server running as root is a direct path to full system compromise when combined with the Samba RCE vulnerabilities present on this version.

**Samba 3.0.20 - CVE-2007-2447:** This version of Samba contains a critical remote code execution vulnerability in the username map script feature. When Samba is configured with a username map script, an attacker can inject shell metacharacters into the username field during authentication, causing the server to execute arbitrary commands as root. This CVE has a Metasploit module (`exploit/multi/samba/usermap_script`) and is one of the most practised exploitation exercises in CEH and OSCP training.

---

## Disclaimer
This lab was performed exclusively against Metasploitable2, a deliberately vulnerable virtual machine created by Rapid7 for penetration testing education and practice. Metasploitable2 is intentionally insecure and is designed to be used in isolated lab environments. No real-world systems, production infrastructure, or third-party networks were targeted at any point. All techniques documented here are for educational purposes only.
