# Nmap - Network Scanning & Port Discovery

## Objective
To perform active network scanning and port discovery against a legal target (`scanme.nmap.org`) using a comprehensive range of Nmap techniques - from basic host discovery to stealth scans, firewall evasion, OS fingerprinting, vulnerability detection, and banner grabbing - documenting what each technique reveals about the target's attack surface.

---

## Tools Used
- Nmap 7.95 (command-line, Kali Linux)

---

## Lab Environment
- Operating System: Kali Linux
- Tool: Nmap 7.95
- Target: `scanme.nmap.org` (45.33.32.156)
- Authorization: Nmap officially permits scanning of `scanme.nmap.org` for testing and educational purposes

---

## Background
Nmap (Network Mapper) is the industry-standard open-source tool for network discovery and security auditing. It works by sending crafted packets to a target and analyzing responses to determine which hosts are alive, which ports are open, what services and versions are running, and what operating system the target is likely running. Nmap's NSE (Nmap Scripting Engine) extends this capability to include vulnerability detection, banner grabbing, brute force, and more. In the ethical hacking lifecycle, Nmap is the cornerstone tool of the Scanning & Enumeration phase - it transforms the domain names and IP ranges discovered during Reconnaissance into a detailed map of exploitable entry points.

---

## Scans Performed

### Scan 1: Ping Scan - Host Discovery
```bash
nmap -sn scanme.nmap.org
```

<img width="658" height="128" alt="image" src="https://github.com/user-attachments/assets/713280ad-62fd-42fe-8a58-81141b2ad3a4" />

**Finding:** Host confirmed up at `45.33.32.156` with a latency of `0.00055s`. The scan completed in 6.92 seconds and confirmed the target is reachable before running any deeper scans. An IPv6 address (`2600:3c01::f03c:91ff:fe18:bb2f`) was also detected but not scanned.

---

### Scan 2: Default Scan - Top 1000 Ports
```bash
nmap scanme.nmap.org
```

<img width="554" height="49" alt="image" src="https://github.com/user-attachments/assets/ea6065e3-700a-40a4-951c-6e877c676119" />
<img width="670" height="176" alt="image" src="https://github.com/user-attachments/assets/47977507-561e-4261-97e6-e4d898a16552" />

**Finding:** The default scan (no sudo, no flags) resolved the target to `45.33.32.156` and scanned the top 1000 TCP ports. Results revealed 3 open ports - **22/tcp (SSH)**, **80/tcp (HTTP)**, and **9929/tcp (nping-echo)** - with 507 ports closed and 490 filtered. The scan took 6448.16 seconds due to network throttling, producing retransmission warnings.

---

### Scan 3: SYN Stealth Scan + Version Detection
```bash
sudo nmap -sS -sV scanme.nmap.org
```

<img width="905" height="542" alt="image" src="https://github.com/user-attachments/assets/49c2cdb8-1f46-44bf-8920-1c22bbe5ee4d" />

**Finding:** The SYN stealth scan (half-open scan) discovered 4 open ports with version information:

| Port | State | Service | Version |
|---|---|---|---|
| 22/tcp | open | ssh | OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 |
| 80/tcp | open | http | Apache httpd 2.4.7 (Ubuntu) |
| 9929/tcp | open | nping-echo | Nping echo |
| 31337/tcp | open | tcpwrapped | - |

Service Info confirmed: `OS: Linux; CPE: cpe:/o:linux:linux_kernel`. The scan generated retransmission warnings, indicating rate-limiting or packet filtering at the network level. SYN scans are stealthier than full TCP connects as they never complete the three-way handshake, reducing the chance of being logged by applications.

---

### Scan 4: Aggressive Scan - OS + Version + Scripts + Traceroute
```bash
sudo nmap -A scanme.nmap.org
```

<img width="305" height="39" alt="image" src="https://github.com/user-attachments/assets/78f826b6-b64e-4469-9c1c-a4bbda0a8c48" />
<img width="834" height="470" alt="image" src="https://github.com/user-attachments/assets/545f21d6-9f8d-4259-95da-b3d834557af6" />

**Finding:** The aggressive scan confirmed all 4 open ports with full detail:

| Port | Service | Version |
|---|---|---|
| 22/tcp | SSH | OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 |
| 80/tcp | HTTP | Apache httpd 2.4.7 (Ubuntu) |
| 9929/tcp | nping-echo | Nping echo |
| 31337/tcp | tcpwrapped | - |

**SSH Host Keys exposed:**
- 1024-bit DSA: `ac:00:a0:1a:82:ff:cc:55:99:dc:67:2b:34:97:6b:75`
- 2048-bit RSA: `20:3d:2d:44:62:2a:b0:5a:9d:b5:b3:05:14:c2:a6:b2`
- 256-bit ECDSA: `96:02:bb:5e:57:54:1c:4e:45:2f:56:4c:4a:24:b2:57`
- 256-bit ED25519: `33:fa:91:0f:e0:e1:7b:1f:6d:05:a2:b0:f1:54:41:56`

**HTTP:** Apache 2.4.7 with server header `Apache/2.4.7 (Ubuntu)`. HTTP title: *Go ahead and ScanMe!*. HTTP favicon identified as Nmap Project.

**OS Detection:** Fingerprint was non-ideal due to no UDP response. Top guesses included Actiontec MI424WR-GEN3I WAP (95%), DD-WRT v24-sp2 Linux 2.4.37 (95%), Linux 3.2 (93%), Linux 4.4 (91%). CPE confirmed `cpe:/o:linux:linux_kernel`.

**Traceroute:** 2 hops - Hop 1: `192.168.23.2` (0.18ms), Hop 2: `scanme.nmap.org / 45.33.32.156` (0.19ms).

Scan completed in **5656.61 seconds** with multiple retransmission warnings indicating aggressive filtering on many ports.

---

### Scan 5: Default NSE Script Scan
```bash
sudo nmap -sC scanme.nmap.org
```

<img width="261" height="37" alt="image" src="https://github.com/user-attachments/assets/81070026-b5cc-4e82-b4c2-fe2f6d713517" />
<img width="669" height="500" alt="image" src="https://github.com/user-attachments/assets/15f62b40-c5a6-44ca-90e1-d87511636c6c" />


**Finding:** NSE scripts returned rich enumeration data alongside port discovery:

| Port | State | Service |
|---|---|---|
| 22/tcp | open | ssh |
| 25/tcp | filtered | smtp |
| 80/tcp | open | http |
| 135/tcp | filtered | msrpc |
| 139/tcp | filtered | netbios-ssn |
| 445/tcp | filtered | microsoft-ds |
| 514/tcp | filtered | shell |
| 1080/tcp | filtered | socks |
| 9929/tcp | open | nping-echo |
| 31337/tcp | open | Elite |

SSH host keys were enumerated (same 4 keys as Scan 4). HTTP title returned: *"Go ahead and ScanMe!"*. HTTP favicon identified as Nmap Project. The presence of filtered ports for SMB (139, 445), MSRPC (135), and SOCKS (1080) indicates active firewall rules blocking those services rather than no service being present. Scan completed in 277.27 seconds.

---

### Scan 6: UDP Scan - Top 20 Ports
```bash
sudo nmap -sU --top-ports 20 scanme.nmap.org
```

<img width="379" height="40" alt="image" src="https://github.com/user-attachments/assets/5a70a0ab-f278-43f5-bc61-1591150f71b4" />
<img width="664" height="451" alt="image" src="https://github.com/user-attachments/assets/45508404-2a1c-4747-9119-c105e0e4757f" />

**Finding:** UDP scan of the top 20 ports completed in 9.01 seconds. Results:

| Port | State | Service |
|---|---|---|
| 53/udp | open\|filtered | domain (DNS) |
| 67/udp | open\|filtered | dhcps |
| 68/udp | open\|filtered | dhcpc |
| 69/udp | open\|filtered | tftp |
| **123/udp** | **open** | **ntp** |
| 135/udp | open\|filtered | msrpc |
| 137/udp | open\|filtered | netbios-ns |
| 138/udp | open\|filtered | netbios-dgm |
| 139/udp | open\|filtered | netbios-ssn |
| 161/udp | open\|filtered | snmp |
| 162/udp | open\|filtered | snmptrap |
| 445/udp | open\|filtered | microsoft-ds |
| 500/udp | open\|filtered | isakmp |
| 514/udp | open\|filtered | syslog |
| 520/udp | open\|filtered | route |
| 631/udp | open\|filtered | ipp |
| 1434/udp | open\|filtered | ms-sql-m |
| 1900/udp | open\|filtered | upnp |
| 4500/udp | open\|filtered | nat-t-ike |
| 49152/udp | open\|filtered | unknown |

Port **123/udp (NTP)** was the only confirmed open UDP port. The `open|filtered` state on all remaining 19 ports indicates Nmap could not determine conclusively whether those ports are open or filtered - UDP gives no response on filtered ports, making it indistinguishable from an open port with no data to send.

---

### Scan 7: Firewall Evasion - Fragmented Packets
```bash
sudo nmap -f scanme.nmap.org
```

<img width="298" height="38" alt="image" src="https://github.com/user-attachments/assets/d56d6707-3d47-4047-b6c1-0523723e0c60" />
<img width="685" height="166" alt="image" src="https://github.com/user-attachments/assets/61f4b96d-23f7-4196-b67b-c3011c30b9b0" />

**Finding:** The fragmented packet scan broke IP packets into 8-byte fragments to attempt to bypass stateless packet inspection firewalls and IDS/IPS systems. Results were consistent with the default scan - **4 open ports: 22, 80, 9929, 31337** - with 662 filtered and 334 closed. The identical results confirm the target's firewall is stateful and reassembles fragments before inspection, rather than evaluating each fragment independently. This technique remains effective against older or simpler stateless firewalls.

---

### Scan 8: Null Scan
```bash
sudo nmap -sN scanme.nmap.org
```

<img width="665" height="193" alt="image" src="https://github.com/user-attachments/assets/976a30d1-5437-424d-a431-15fa10a0fe86" />

**Finding:** The Null scan sends TCP packets with no flags set. RFC 793 specifies that a closed port should respond with RST, while an open port gives no response. Result: **all 1000 scanned ports returned `open|filtered`** with no RST responses, meaning the target's stateful firewall is dropping the flagless packets entirely. This behavior is expected on Linux targets running modern `iptables` or `nftables` rulesets. Scan completed in 10.75 seconds.

---

### Scan 9: FIN Scan
```bash
sudo nmap -sF scanme.nmap.org
```

<img width="679" height="195" alt="image" src="https://github.com/user-attachments/assets/b9f4051b-4b7d-47e4-879a-3f072b60a144" />

**Finding:** The FIN scan sends TCP packets with only the FIN flag set - a technique used to bypass non-stateful firewalls that only block SYN packets. Like the Null scan, **all 1000 ports returned `open|filtered`** with no RST responses. This confirms the Linux kernel is silently dropping malformed TCP flag packets that do not belong to an established connection. Completed in 10.80 seconds.

---

### Scan 10: Xmas Scan
```bash
sudo nmap -sX scanme.nmap.org
```

<img width="666" height="193" alt="image" src="https://github.com/user-attachments/assets/122031b8-cc34-4eb1-b758-5d00407c25a1" />

**Finding:** The Xmas scan sets the FIN, PSH, and URG flags simultaneously. According to RFC 793, closed ports should reply with RST. Result: **all 1000 ports returned `open|filtered`**, consistent with Scan 8 and Scan 9. The uniformity across all three stealth scan types confirms the target is running a stateful Linux firewall that drops packets not belonging to an established connection, rather than relying on per-flag rules. Completed in 10.77 seconds.

---

### Scan 11: Vulnerability Scan - NSE Vuln Scripts
```bash
sudo nmap --script vuln scanme.nmap.org
```

<img width="702" height="572" alt="image" src="https://github.com/user-attachments/assets/49cf95cf-7609-425a-8413-8c0ae8bad6da" />

**Finding:** The vulnerability scan ran a full suite of NSE vuln-category scripts against all discovered open ports. Completed in 932.01 seconds. Key results:

**HTTP (port 80):**
- `http-dombased-xss`: No DOM-based XSS vulnerabilities found
- `http-stored-xss`: No stored XSS vulnerabilities found
- `http-enum`: Discovered `/images/` directory with **listing enabled on Apache 2.4.7 (Ubuntu)** — a misconfiguration that exposes directory contents to unauthenticated users
- `http-csrf`: Spider (maxdepth=3, maxpages=20) found **2 potential CSRF vulnerability paths**:
  - `http://scanme.nmap.org:80/` — Form ID: `nst-head-search`, Action: `/search/`
  - `http://scanme.nmap.org:80/` — Form ID: `nst-foot-search`, Action: `/search/`

**Other ports noted:** 25/tcp (SMTP) filtered, 139/tcp and 445/tcp (SMB) filtered, 514/tcp filtered, 1080/tcp filtered.

No critical CVEs were triggered, but the directory listing misconfiguration and CSRF form findings are legitimate low-to-medium severity findings on a production web server.

---

### Scan 12: Banner Grabbing
```bash
sudo nmap --script banner scanme.nmap.org
```

<img width="911" height="499" alt="image" src="https://github.com/user-attachments/assets/f775cb76-d41b-4311-9ba1-66de75fe5c1f" />

**Finding:** Raw service banners were successfully retrieved:

| Port | Banner |
|---|---|
| 22/tcp | `SSH-2.0-OpenSSH_6.6.1p1 Ubuntu-2ubuntu2.13` |
| 9929/tcp | Raw binary nping-echo banner (non-printable bytes) |

The SSH banner explicitly discloses the exact software version (`OpenSSH 6.6.1p1`) and OS distribution (`Ubuntu 2ubuntu2.13`). This is highly valuable to an attacker for identifying version-specific CVEs without running a single exploit. HTTP port 80 did not return a banner via this script but version was confirmed through other scans as `Apache/2.4.7 (Ubuntu)`. Scan completed in 6882.97 seconds due to retransmission timeouts on filtered ports.

---

### Scan 13: Full Aggressive Scan - Saved Output (All Formats)
```bash
sudo nmap -A scanme.nmap.org -oA nmap_full_output
```

<img width="911" height="729" alt="image" src="https://github.com/user-attachments/assets/0d8a4d4d-879e-4fa3-9a9c-314265d6645b" />

**Finding:** Full results saved to three output files - `nmap_full_output.nmap` (human-readable), `nmap_full_output.xml` (machine-readable, importable into Metasploit), and `nmap_full_output.gnmap` (grepable format for scripting and pipeline use). Results confirmed 4 open ports with full version strings, SSH host keys, OS fingerprint, and traceroute data. Aggressive OS guesses included Linux 4.4 (91%) and Linux 3.2 (93%). Network distance confirmed at 2 hops. These output formats are standard practice in professional penetration testing engagements for reporting, evidence preservation, and integration with exploitation frameworks.

---

## Commands Used

| # | Command | Purpose |
|---|---|---|
| 1 | `nmap -sn scanme.nmap.org` | Host discovery / ping scan |
| 2 | `nmap scanme.nmap.org` | Default top-1000 port scan |
| 3 | `sudo nmap -sS -sV scanme.nmap.org` | SYN stealth scan + version detection |
| 4 | `sudo nmap -A scanme.nmap.org` | Aggressive scan (OS + version + scripts + traceroute) |
| 5 | `sudo nmap -sC scanme.nmap.org` | Default NSE script scan |
| 6 | `sudo nmap -sU --top-ports 20 scanme.nmap.org` | UDP scan — top 20 ports |
| 7 | `sudo nmap -f scanme.nmap.org` | Firewall evasion via packet fragmentation |
| 8 | `sudo nmap -sN scanme.nmap.org` | Null scan (no TCP flags) |
| 9 | `sudo nmap -sF scanme.nmap.org` | FIN scan |
| 10 | `sudo nmap -sX scanme.nmap.org` | Xmas scan (FIN+PSH+URG flags) |
| 11 | `sudo nmap --script vuln scanme.nmap.org` | NSE vulnerability scan |
| 12 | `sudo nmap --script banner scanme.nmap.org` | Service banner grabbing |
| 13 | `sudo nmap -A scanme.nmap.org -oA nmap_full_output` | Full aggressive scan saved to all output formats |

---

## Observations

Across all 13 scans, the following was confirmed about `scanme.nmap.org (45.33.32.156)`:

**Open Ports:**
- **22/tcp - SSH:** OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (protocol 2.0). Banner explicitly disclosed via banner grabbing. Four SSH host keys enumerated (DSA, RSA, ECDSA, ED25519).
- **80/tcp - HTTP:** Apache httpd 2.4.7 (Ubuntu). HTTP title: *"Go ahead and ScanMe!"*. Directory listing enabled on `/images/`. Two potential CSRF-vulnerable search forms detected.
- **9929/tcp - nping-echo:** Nmap's own echo service, expected on this host.
- **31337/tcp - tcpwrapped/Elite:** Port 31337 is listed as tcpwrapped - the service accepted the TCP connection but did not respond with identifiable data.

**Filtered Ports:** 25 (SMTP), 135 (MSRPC), 139 (NetBIOS-SSN), 445 (SMB), 514 (Shell), 1080 (SOCKS) - all actively blocked by firewall.

**UDP:** Only port 123 (NTP) confirmed open. All other top-20 UDP ports returned `open|filtered`.

**Stealth Scans (Null/FIN/Xmas):** All 1000 ports returned `open|filtered` across all three scan types - consistent behavior of a stateful Linux firewall dropping malformed TCP packets not belonging to an established connection.

**Firewall Evasion:** Fragmented packet scan (-f) produced identical results to a standard scan, confirming the target firewall is stateful and reassembles fragments before inspection.

**Vulnerability Findings:** No critical CVEs triggered. Directory listing on `/images/` confirmed as a real misconfiguration. Two CSRF-susceptible form paths detected on the web interface. No XSS found.

**OS:** Confirmed Linux kernel. CPE: `cpe:/o:linux:linux_kernel`. OS detection non-ideal due to missing UDP response - top guess was Linux 4.4 at 91%.

**Traceroute:** Only 2 hops to target - local gateway (192.168.23.2) and scanme.nmap.org directly, indicating minimal network distance.

---

## Security Relevance

This lab demonstrates the full information-gathering capability of Nmap from both attacker and defender perspectives.

**Version Disclosure:** OpenSSH 6.6.1p1 is an older version (released ~2014). In a real engagement this version string would be immediately cross-referenced against CVE databases. The SSH banner broadcasting the exact version and OS distribution is a hardening failure — production servers should suppress or obscure version banners using `DebianBanner no` in `sshd_config` and `ServerTokens Prod` in Apache configuration.

**Apache Directory Listing:** The `/images/` directory being listable is a direct misconfiguration. Directory listing allows attackers to enumerate files without needing to guess paths, potentially exposing backup files, configuration files, or sensitive assets that were not intended to be public. It should be disabled with `Options -Indexes` in Apache configuration.

**CSRF Findings:** Both search forms lacked CSRF tokens, making them potential targets for cross-site request forgery attacks where an attacker tricks an authenticated user's browser into submitting unintended requests to the server on their behalf.

**SSH Host Key Fingerprinting:** All four SSH host keys were extracted without authentication. This is by design for SSH (to allow client verification), but in a red team scenario this data can be used to fingerprint a specific server instance and track it across IP changes or infrastructure migrations.

**Stealth Scan Behavior:** The consistent `open|filtered` response to Null, FIN, and Xmas scans confirms stateful firewall protection. However, these same techniques are effective against misconfigured or legacy firewalls that only inspect the SYN flag, making them standard tools in a penetration tester's enumeration phase.

**Firewall Evasion (-f):** Packet fragmentation is a classic IDS evasion technique. While it did not bypass this target's defenses, it remains relevant against older or simpler security appliances and demonstrates why modern IDS/IPS systems must perform full fragment reassembly before inspection.

**Port 31337:** Historically associated with malware and backdoors (referred to as "Elite" in hacker culture). Its presence here is intentional and educational, but seeing tcpwrapped on unusual high ports in a real engagement would warrant immediate investigation and traffic analysis.

---

## Disclaimer
This lab was conducted exclusively against `scanme.nmap.org`, a host maintained by the Nmap project specifically for legal, authorized scanning and testing. As stated on their website: *"We set up this machine to help folks learn about Nmap and also to test and make sure that their Nmap installation is working correctly."* No unauthorized systems were scanned. All techniques documented here are for educational purposes only.
