# Nessus - Full Vulnerability Assessment

## Objective
To perform a full automated vulnerability assessment against a legal target (`scanme.nmap.org`) using Nessus Essentials, identifying and documenting all discovered vulnerabilities by severity, understanding their risk implications, and demonstrating the use of an industry-standard vulnerability scanner as part of the Scanning and Enumeration phase of the ethical hacking lifecycle.

---

## Tools Used
- Nessus Essentials (Tenable) - web-based GUI, localhost:8834

---

## Lab Environment
- Operating System: Kali Linux
- Tool: Nessus Essentials
- Target: `scanme.nmap.org` (45.33.32.156)
- Scan Policy: Basic Network Scan
  
---

## Background
Nessus is one of the most widely used vulnerability assessment platforms in the world, developed by Tenable. It works by sending targeted probes to a host across all open ports, then cross-referencing the responses against a continuously updated database of over 100,000 plugins - each representing a known vulnerability, misconfiguration, or security weakness. Unlike Nmap, which discovers ports and services, Nessus goes a step further by actively testing whether those services are vulnerable to specific known exploits, misconfigurations, or outdated software versions.

Each finding is assigned a CVSS (Common Vulnerability Scoring System) score on a scale of 0–10, categorized as Critical (9.0–10.0), High (7.0–8.9), Medium (4.0–6.9), Low (0.1–3.9), or Informational. Nessus Essentials, the free tier used in this lab, supports scanning up to 16 IP addresses and provides the full plugin set available in the commercial product.

In the ethical hacking lifecycle, Nessus sits at the boundary between Scanning and Enumeration and Vulnerability Analysis - it automates what would otherwise require a penetration tester to manually cross-reference every discovered service version against CVE databases and known exploit repositories.

---

## Steps Performed

### Step 1: Scan Configuration
```
Nessus > New Scan > Basic Network Scan
Target: 45.33.32.156
Policy: Basic Network Scan
Severity Base: CVSS v3.0
```
<img width="967" height="544" alt="image" src="https://github.com/user-attachments/assets/1f4426ec-5ec6-44e5-b0da-2257c9913549" />


**Finding:** The scan was configured targeting `45.33.32.156` directly using the raw IP to ensure reliable resolution. The Basic Network Scan policy was selected, which enables host discovery, port scanning, and the full vulnerability plugin set. Severity scoring was set to CVSS v3.0, the current industry standard for vulnerability scoring.

---

### Step 2: Completed Scan - Host Summary
```
Nessus > My Scans > scan > Hosts tab
```
<img width="1245" height="618" alt="image" src="https://github.com/user-attachments/assets/0c71f37a-1e0b-498c-b964-ec401952f224" />


**Finding:** The scan completed successfully after 2 hours. Host `45.33.32.156` was confirmed reachable and fully assessed. The vulnerability bar showed a breakdown across all severity categories. Scan details confirmed:

| Field | Value |
|---|---|
| Policy | Basic Network Scan |
| Status | Completed |
| Severity Base | CVSS v3.0 |
| Scanner | Local Scanner |
| Start | Today at 3:50 AM |
| End | Today at 5:24 AM |
| Elapsed | 2 hours |

Total vulnerabilities discovered: **40** across 1 host.

---

### Step 3: Vulnerability Severity Overview
```
Nessus > scan > Vulnerabilities tab (summary donut chart)
```
<img width="334" height="236" alt="image" src="https://github.com/user-attachments/assets/79b66fce-aa56-4a3d-a651-0df7db1ed5db" />


**Finding:** The severity breakdown across all 40 findings:

| Severity | Count |
|---|---|
| Critical | 1 |
| High | 1 |
| Medium | 5 |
| Low | 3 |
| Info | 30 |
| **Total** | **40** |

The donut chart confirmed the dominant finding category is Informational (30), followed by Medium (5). The presence of 1 Critical and 1 High finding on a public-facing host represents significant real-world risk.

---

### Step 4: Full Vulnerabilities List
```
Nessus > scan > Vulnerabilities tab (sorted by severity)
```

<img width="1496" height="848" alt="image" src="https://github.com/user-attachments/assets/c2d5c248-50af-4985-8b73-15e42454ab18" />

**Finding:** The full vulnerabilities tab showed 24 grouped entries (some grouping multiple sub-findings) sorted by severity.

---

### Step 5: Critical Finding - Canonical Ubuntu Linux SEoL (14.04.x)
```
Nessus > scan > Vulnerabilities > Canonical Ubuntu Linux SEoL (14.04.x)
Plugin #201408 | CVSS v3.0: 10.0
```

<img width="1266" height="752" alt="image" src="https://github.com/user-attachments/assets/422f0575-4862-46c1-9f49-b537ea7f9f08" />

**Finding:** Plugin 201408 flagged the target as running **Ubuntu Linux 14.04 (Trusty Tahr)**, which reached Security End of Life on **April 25, 2019** - over 6 years ago at the time of this scan.

| Field | Value |
|---|---|
| Severity | Critical |
| Plugin ID | 201408 |
| CVSS v3.0 Score | 10.0 |
| CVSS v2.0 Score | 10.0 |
| Risk Factor | Critical |
| Family | General |
| Type | Combined |
| Published | July 3, 2024 |
| CPE | cpe:/o:canonical:ubuntu_linux |
| Unsupported by vendor | True |

**Output confirmed:**
- OS: Ubuntu Linux 14.04
- Security End of Life: April 25, 2019
- Time since Security End of Life: >= 6 years

**Solution:** Upgrade to a currently supported version of Canonical Ubuntu Linux.

**Why this matters:** An end-of-life operating system receives no security patches from the vendor. Every new CVE discovered after April 2019 that affects Ubuntu 14.04 will remain permanently unpatched on this host. This is rated CVSS 10.0 - the maximum possible score - because the attack surface is unbounded and cannot be remediated without a full OS upgrade.

---

### Step 6: High Finding - DNS Server Spoofed Request Amplification DDoS
```
Nessus > scan > ISC Bind (Multiple Issues) > DNS Server Spoofed Request Amplification DDoS
Plugin #35450 | CVSS v3.0: 7.5 | VPR: 3.6 | EPSS: 0.369
```

<img width="1505" height="858" alt="image" src="https://github.com/user-attachments/assets/1dda377e-78b7-4a32-b75f-abeec114b5a7" />

**Finding:** The DNS service running on the target is vulnerable to **DNS Spoofed Request Amplification**, a technique used in Distributed Denial of Service attacks. The DNS server responds to recursive queries from external hosts, which allows an attacker to use it as a reflector - sending small forged DNS requests that cause the server to send much larger responses to a victim IP address.

| Field | Value |
|---|---|
| Severity | High |
| CVSS v3.0 | 7.5 |
| VPR Score | 3.6 |
| EPSS Score | 0.369 (36.9% exploitation probability) |
| Family | DNS |

Also present in the ISC Bind group:
- **DNS Server Recursive Query Cache Poisoning Weakness** - Medium, CVSS 5.0, EPSS 0.0132
- **DNS Server hostname.bind Map Hostname Disclosure** - Info

**Why this matters:** DNS amplification is one of the most common techniques used in large-scale DDoS attacks. A server with recursive queries enabled and no rate limiting becomes a weapon that can be turned against third-party victims without the server owner's knowledge. An EPSS score of 0.369 means there is a 36.9% probability this vulnerability will be exploited in the wild within the next 30 days.

---

### Step 7: SSH Vulnerability Group
```
Nessus > scan > SSH (Multiple Issues) - 6 findings
```

<img width="1507" height="847" alt="image" src="https://github.com/user-attachments/assets/111e0390-875a-4cad-ba8f-198fed531132" />

**Finding:** The SSH service (OpenSSH 6.6.1p1 on port 22) produced 6 vulnerability findings:

| Severity | Name | CVSS | EPSS |
|---|---|---|---|
| Medium | SSH Weak Algorithms Supported | 4.3 | - |
| Low | SSH Server CBC Mode Ciphers Enabled | 3.7 | 0.0307 |
| Low | SSH Weak Key Exchange Algorithms Enabled | 3.7 | - |
| Low | SSH Weak MAC Algorithms Enabled | 2.6 | - |
| Info | SSH Algorithms and Languages Supported | N/A | - |
| Info | SSH SHA-1 HMAC Algorithms Supported | N/A | - |

The SSH service is configured to support weak and deprecated cryptographic algorithms including CBC mode ciphers, weak key exchange algorithms, weak MAC algorithms, and SHA-1 HMAC. These weaknesses allow downgrade attacks where an attacker in a man-in-the-middle position forces the SSH session to negotiate weaker encryption, potentially enabling session decryption or integrity violations.

---

### Step 8: DNS Vulnerability Group
```
Nessus > scan > DNS (Multiple Issues) - 4 findings
```

<img width="1513" height="871" alt="image" src="https://github.com/user-attachments/assets/37a3fae9-d572-4409-8942-3a906ba1757d" />

**Finding:** The DNS service produced 4 grouped findings:

| Severity | Name | CVSS | EPSS |
|---|---|---|---|
| Medium | DNS Server Cache Snooping Remote Information Disclosure | 5.3 | - |
| Medium | DNS Server Zone Transfer Information Disclosure (AXFR) | 5.0 | 0.8277 |
| Info | DNS Server Detection | N/A | - |
| Info | DNS Server Version Detection | N/A | - |

**DNS Cache Snooping (CVSS 5.3):** Allows an attacker to query the DNS server's cache to determine which domains have recently been resolved by hosts on the network - revealing browsing patterns and internal infrastructure without authentication.

**DNS Zone Transfer (AXFR) (CVSS 5.0, EPSS 0.8277):** The DNS server may permit unauthorized zone transfer requests, which would allow an attacker to download the complete DNS zone file - revealing all hostnames, IP addresses, and subdomains associated with a domain. An EPSS of 0.8277 means an 82.77% probability of exploitation in the wild - the highest EPSS score found in this scan.

---

### Step 9: OpenSSH Terrapin Vulnerability
```
Nessus > scan > Openbsd Openssh (Multiple Issues) - 2 findings
```

<img width="1517" height="837" alt="image" src="https://github.com/user-attachments/assets/ee3301fc-a491-4ed0-9fbe-977ba95b130c" />

**Finding:** The OpenSSH group revealed 2 findings including a named CVE:

| Severity | Name | CVSS | VPR | EPSS |
|---|---|---|---|---|
| Medium | SSH Terrapin Prefix Truncation Weakness (CVE-2023-48795) | 5.9 | 6.1 | 0.5786 |
| Info | OpenSSH Detection | N/A | - | - |

**CVE-2023-48795 (Terrapin Attack):** A prefix truncation vulnerability in the SSH protocol's Binary Packet Protocol (BPP). An attacker performing a man-in-the-middle attack can silently truncate or modify the beginning of the SSH handshake, potentially downgrading security features negotiated during connection setup. EPSS score of 0.5786 indicates a 57.86% exploitation probability - one of the most actively exploited findings in this scan.

---

### Step 10: Full Vulnerability Report - Exported Summary
```
Nessus > scan > Export > PDF Report
```

<img width="676" height="808" alt="image" src="https://github.com/user-attachments/assets/4f70237f-11ea-4820-b784-6ab1bb78db7d" />

**Finding:** The exported report confirmed the complete finding set for `45.33.32.156` with totals of 1 Critical, 1 High, 5 Medium, 3 Low, and 30 Informational findings - 40 total.

---

## Commands Used

| Action | Details |
|---|---|
| Start Nessus service | `sudo systemctl start nessusd` |
| Access Nessus UI | `https://localhost:8834` |
| Scan policy | Basic Network Scan |
| Target | `45.33.32.156` |
| Severity base | CVSS v3.0 |
| Export format | PDF Report |

---

## Observations

The 2-hour Basic Network Scan of `45.33.32.156` produced **40 total findings** across 5 severity levels. Key observations:

**Critical (1):** The host is running Ubuntu Linux 14.04, which reached end of life in April 2019 - over 6 years before this scan. With a CVSS score of 10.0, this is the most severe possible finding. No security patches will ever be released for this OS version, meaning every new vulnerability discovered affecting Ubuntu 14.04 is permanently exploitable on this host.

**High (1):** The DNS service accepts recursive queries from external hosts, making it exploitable as a DDoS amplification reflector. The EPSS score of 0.369 indicates active exploitation in the wild.

**Medium (5):** Findings span three distinct service areas - SSH cryptographic weaknesses (weak ciphers, key exchange, and MAC algorithms), a named CVE in OpenSSH (CVE-2023-48795 Terrapin Attack, EPSS 0.5786), and DNS service misconfigurations including cache snooping and potential zone transfer exposure (EPSS 0.8277).

**Low (3):** All three low findings relate to deprecated SSH cryptographic configurations - CBC mode ciphers, weak key exchange algorithms, and weak MAC algorithms - each enabling potential downgrade attacks.

**Info (30):** Informational findings confirmed service versions, detection data, CPE entries, and backported patch detection across SSH, Apache HTTP, and DNS services. While not directly exploitable, these findings provide an attacker with a precise fingerprint of every service version running on the host.

**EPSS Highlights:** Three findings showed high real-world exploitation probability - DNS Zone Transfer (82.77%), SSH Terrapin CVE-2023-48795 (57.86%), and DNS Amplification DDoS (36.9%) - indicating these are not theoretical risks but actively targeted vulnerabilities.

---

## Security Relevance

**End-of-Life Operating Systems:** Running an unsupported OS is the single highest-risk configuration a server can have. Ubuntu 14.04 has not received security updates since 2019. Every new CVE published after that date that affects the Linux kernel, OpenSSH, Apache, or any installed package on this host is permanently unpatched. In a real engagement, this finding alone would be escalated as a critical remediation priority requiring immediate OS upgrade.

**CVSS vs EPSS - Understanding Real Risk:** CVSS scores measure the theoretical severity of a vulnerability based on attack vector, complexity, and impact. EPSS (Exploit Prediction Scoring System) measures the real-world probability of exploitation within 30 days based on threat intelligence data. The DNS Zone Transfer finding (CVSS 5.0, EPSS 0.8277) is a perfect example - its theoretical severity is medium, but its real-world exploitation probability is 82.77%, making it far more urgent than its CVSS score suggests. Defenders should prioritize remediation using both scores together.

**DNS Amplification as a Weapon:** A DNS server accepting recursive queries from the public internet can be weaponized by attackers to amplify DDoS traffic against third-party victims. The attacker sends small forged UDP packets to the open resolver with a spoofed source IP (the victim), and the DNS server sends much larger responses to the victim. This makes the server an unwitting participant in attacks against others, creating legal and reputational risk for its operator in addition to technical risk.

**SSH Cryptographic Weaknesses:** Multiple deprecated cryptographic algorithms remain enabled in the SSH configuration - CBC mode ciphers, weak key exchange, weak MAC algorithms, and SHA-1 HMAC. While exploiting these requires a man-in-the-middle position, they represent a defense-in-depth failure. Modern SSH hardening requires disabling all CBC ciphers, SHA-1 HMAC, and weak Diffie-Hellman key exchange groups in `sshd_config`.

**CVE-2023-48795 (Terrapin):** This is a relatively recent CVE (published 2023) affecting the SSH Binary Packet Protocol, demonstrating that even well-maintained services can carry recent vulnerabilities. OpenSSH 6.6.1p1 predates the fix. The combination of an EOL OS and an old OpenSSH version means this CVE cannot be patched without the OS upgrade.

**Nessus in the Pentest Workflow:** Nessus sits between Scanning/Enumeration and Vulnerability Analysis in the ethical hacking lifecycle. Its output directly feeds the next phase - an attacker would take the Critical and High findings and search for public exploits (via Exploit-DB, Metasploit, or CVE databases) to move into the Exploitation phase. The plugin IDs and CVE references in Nessus output are directly searchable in Metasploit's module database.

---

## Disclaimer
This lab was conducted exclusively against `scanme.nmap.org`, a host maintained by the Nmap project specifically for legal, authorized scanning and testing. Nessus Essentials was used under its free license for educational purposes. No exploitation of any discovered vulnerability was attempted. All techniques documented here are for educational purposes only.
