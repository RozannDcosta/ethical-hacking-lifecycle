# Netdiscover - ARP Host Discovery on LAN

## Objective
To perform active and passive ARP-based host discovery on a local area network using Netdiscover, identifying all live hosts by their IP address, MAC address and hardware vendor and to understand how this information is used during the Scanning and Enumeration phase of an ethical hacking engagement.

---

## Tools Used
- Netdiscover (command-line, Kali Linux)

---

## Lab Environment
- Operating System: Kali Linux
- Tool: Netdiscover
- Target: Local network - `192.168.23.0/24` and `192.168.60.0/24`
- Interfaces: eth0 (192.168.23.129), eth2 (192.168.60.129), eth3 (192.168.23.130)

---

## Background
Netdiscover is an ARP-based network scanning tool designed for host discovery on local area networks. Unlike TCP/UDP-based scanners such as Nmap, Netdiscover operates at Layer 2 of the OSI model by sending ARP (Address Resolution Protocol) requests and listening for ARP replies. Because ARP is a fundamental protocol required for LAN communication, hosts cannot silently ignore ARP requests the way they can ignore ICMP pings or TCP probes - making ARP-based discovery highly reliable on local networks.

Netdiscover operates in two modes. In active mode it broadcasts ARP requests across a specified subnet and maps every host that replies. In passive mode it simply listens to existing ARP traffic on the network without sending any packets, making it completely invisible to network monitoring tools that detect active scanning.

In the ethical hacking lifecycle, Netdiscover is used at the very start of the Scanning and Enumeration phase when the attacker has access to the local network segment. It provides a fast and reliable map of all live hosts before deeper scanning with tools like Nmap begins.

---

## Scans Performed

### Scan 1: Interface and Network Enumeration
```bash
ip a
```

<img width="839" height="440" alt="image" src="https://github.com/user-attachments/assets/4b76b6d7-8483-402d-ae94-6fd2b2ee4ac8" />

**Finding:** The Kali machine has four network interfaces active:

| Interface | IP Address | Subnet |
|---|---|---|
| lo | 127.0.0.1 | loopback |
| eth0 | 192.168.23.129 | 192.168.23.0/24 |
| eth1 | (no IPv4) | link-local only |
| eth2 | 192.168.60.129 | 192.168.60.0/24 |
| eth3 | 192.168.23.130 | 192.168.23.0/24 |

MAC address for eth0: `00:0c:29:c2:bf:01`. The machine has dual presence on the `192.168.23.0/24` subnet via both eth0 and eth3, and a separate presence on `192.168.60.0/24` via eth2. This multi-interface configuration is common in virtualized lab environments and penetration testing setups.

---

### Scan 2: Subnet Route Detection
```bash
ip route | grep -v default
```

<img width="680" height="98" alt="image" src="https://github.com/user-attachments/assets/d5ab23f4-b4c0-471f-a761-0a8d2ef6ef39" />

**Finding:** Three active routes confirmed without the default gateway:

| Network | Interface | Source IP | Metric |
|---|---|---|---|
| 192.168.23.0/24 | eth0 | 192.168.23.129 | 100 |
| 192.168.23.0/24 | eth3 | 192.168.23.130 | 103 |
| 192.168.60.0/24 | eth2 | 192.168.60.129 | 102 |

This command is used before scanning to precisely identify which subnets are reachable and via which interfaces, avoiding the need to guess the correct subnet range. In an engagement, knowing the exact subnet boundaries prevents wasted scan time and ensures no reachable hosts are missed.

---

### Scan 3: Passive Mode - ARP Traffic Listening
```bash
sudo netdiscover -p
```

<img width="649" height="134" alt="image" src="https://github.com/user-attachments/assets/3ed3eb22-afe0-4bbe-b04c-f6c811b96576" />

**Finding:** Passive mode captured **19 ARP request/reply packets from 2 hosts** with a total size of 1140 bytes - without sending a single packet:

| IP | MAC Address | Count | Vendor |
|---|---|---|---|
| 192.168.23.2 | 00:50:56:e7:21:f9 | 3 | VMware, Inc. |
| 192.168.23.1 | 00:50:56:c0:00:08 | 16 | VMware, Inc. |

Both hosts were actively generating ARP traffic on the network, making them visible even in completely passive mode. This is significant from a stealth perspective - passive scanning produces zero outbound packets and cannot be detected by any network monitoring system. `192.168.23.1` generated 16 packets, suggesting it is the most active host on the segment, consistent with a gateway or router role.

---

### Scan 4: Active Scan - Full Subnet
```bash
sudo netdiscover -r 192.168.23.0/24
```


<img width="650" height="197" alt="image" src="https://github.com/user-attachments/assets/5e9d5407-eeea-412b-a91d-4805edda24b4" />

**Finding:** Active scan of the full `/24` subnet discovered **5 live hosts** from 5 captured ARP packets:

| IP | MAC Address | Vendor |
|---|---|---|
| 192.168.23.1 | 00:50:56:c0:00:08 | VMware, Inc. |
| 192.168.23.2 | 00:50:56:e7:21:f9 | VMware, Inc. |
| 192.168.23.129 | 00:0c:29:c2:bf:1f | VMware, Inc. |
| 192.168.23.130 | 00:0c:29:c2:bf:1f | VMware, Inc. |
| 192.168.23.254 | 00:50:56:e2:2b:80 | VMware, Inc. |

All five hosts returned VMware OUI prefixes, confirming this is a fully virtualized lab environment running on VMware infrastructure. The active scan confirmed two additional hosts - `.129`, `.130`, and `.254` - that were not visible in passive mode due to their lower ARP traffic volume.

---

### Scan 5: Interface-Specific Scan
```bash
sudo netdiscover -i eth0 -r 192.168.23.0/24
```
<img width="629" height="184" alt="image" src="https://github.com/user-attachments/assets/ccf0199b-d3ef-4a6f-8ad8-482b1397e3e8" />

**Finding:** Specifying `eth0` explicitly produced identical results - **5 hosts discovered**, all VMware-vendor MACs, consistent with Scan 4. Pinning scans to a specific interface is important on multi-homed machines (machines with multiple network interfaces) to ensure the scan is sent from and received on the correct network segment. In a real engagement where an attacker has compromised a dual-homed host, this technique is used to pivot and discover hosts on otherwise unreachable internal segments.

---

### Scan 6: Fast Mode Scan
```bash
sudo netdiscover -r 192.168.23.0/24 -f
```
<img width="633" height="168" alt="image" src="https://github.com/user-attachments/assets/9c5ba75a-7cce-4bb0-b036-12e7d02534e5" />

**Finding:** Fast mode (`-f`) completed with **8 ARP packets from 3 hosts** - discovering `.1`, `.2`, and `.254`. The fast mode reduces the number of ARP retransmissions per host, trading thoroughness for speed. Hosts `.129` and `.130` (the Kali machine's own interfaces) were not returned, which is expected as a host typically does not ARP-reply to itself. Fast mode is useful for quick initial sweeps during time-limited engagements but may miss hosts with higher ARP response latency.

---

### Scan 7: Increased Sleep Time Between Requests
```bash
sudo netdiscover -r 192.168.23.0/24 -s 5
```

<img width="699" height="180" alt="image" src="https://github.com/user-attachments/assets/d1d9f3ee-0fd7-495d-9ea2-000ab05e084d" />

**Finding:** The `-s 5` flag introduced a 5-second sleep between each ARP request, resulting in **7 ARP packets captured from 5 hosts**. All 5 hosts were discovered - `.1`, `.2`, `.129`, `.130`, and `.254` - consistent with the full active scan. The slower pacing reduces the scan's ARP broadcast rate, making it less likely to trigger rate-limiting thresholds or IDS rules that alert on high-volume ARP floods. This technique is used in engagements where stealth and IDS evasion are a priority, accepting a longer scan duration in exchange for a lower detection profile.

---

### Scan 8: Repeated Count Scan
```bash
sudo netdiscover -r 192.168.23.0/24 -c 5
```

<img width="635" height="180" alt="image" src="https://github.com/user-attachments/assets/f6f23347-0a0e-4d38-88a9-950964a1d462" />

**Finding:** Running with `-c 5` (5 ARP retries per host) captured **36 ARP packets from 5 hosts** with a total size of 2160 bytes - significantly more packets than any previous scan. Host `.1` alone generated 16 ARP exchanges (960 bytes), indicating high responsiveness or multiple services relying on ARP at that address. The higher count confirms reliability of discovery at the cost of increased network noise. In a real engagement, higher retry counts are used on noisy or unreliable networks to ensure hosts behind intermittent firewalls or switches are not missed.

---

### Scan 9: Save Output to File
```bash
sudo netdiscover -r 192.168.23.0/24 -P > netdiscover_output.txt
cat netdiscover_output.txt
```

<img width="640" height="203" alt="image" src="https://github.com/user-attachments/assets/8f562bde-9516-4bcc-8d81-9374c5b44f8d" />

**Finding:** The `-P` flag outputs results in a parseable, non-interactive format suitable for redirection to a file. The saved `netdiscover_output.txt` contained the complete host table:

| IP | MAC Address | Count | Len | Vendor |
|---|---|---|---|---|
| 192.168.23.1 | 00:50:56:c0:00:08 | 1 | 60 | VMware, Inc. |
| 192.168.23.2 | 00:50:56:e7:21:f9 | 1 | 60 | VMware, Inc. |
| 192.168.23.129 | 00:0c:29:c2:bf:1f | 1 | 60 | VMware, Inc. |
| 192.168.23.130 | 00:0c:29:c2:bf:1f | 1 | 60 | VMware, Inc. |
| 192.168.23.254 | 00:50:56:e2:2b:80 | 1 | 60 | VMware, Inc. |

The file ended with the confirmation line: `-- Active scan completed, 5 Hosts found.` Saving scan output is standard practice in professional engagements for evidence preservation, report generation, and feeding results into downstream tools.

---

## Commands Used

| # | Command | Purpose |
|---|---|---|
| 1 | `ip a` | Enumerate all network interfaces and IP addresses |
| 2 | `ip route \| grep -v default` | Identify active subnets and routing interfaces |
| 3 | `sudo netdiscover -p` | Passive ARP listening - zero packets sent |
| 4 | `sudo netdiscover -r 192.168.23.0/24` | Active scan of full subnet |
| 5 | `sudo netdiscover -i eth0 -r 192.168.23.0/24` | Interface-specific active scan |
| 6 | `sudo netdiscover -r 192.168.23.0/24 -f` | Fast mode scan - fewer retries |
| 7 | `sudo netdiscover -r 192.168.23.0/24 -s 5` | Slow scan with 5s sleep between requests |
| 8 | `sudo netdiscover -r 192.168.23.0/24 -c 5` | Scan with 5 ARP retries per host |
| 9 | `sudo netdiscover -r 192.168.23.0/24 -P > netdiscover_output.txt` | Save output to file in parseable format |

---

## Observations

All scans were performed against the `192.168.23.0/24` subnet. The following 5 live hosts were consistently discovered across active scan modes:

| IP | MAC Address | Vendor | Likely Role |
|---|---|---|---|
| 192.168.23.1 | 00:50:56:c0:00:08 | VMware, Inc. | VMware virtual gateway / router |
| 192.168.23.2 | 00:50:56:e7:21:f9 | VMware, Inc. | VMware management / DHCP host |
| 192.168.23.129 | 00:0c:29:c2:bf:1f | VMware, Inc. | Kali Linux attacker machine (eth0) |
| 192.168.23.130 | 00:0c:29:c2:bf:1f | VMware, Inc. | Kali Linux attacker machine (eth3) |
| 192.168.23.254 | 00:50:56:e2:2b:80 | VMware, Inc. | VMware virtual network edge device |

**Passive scan** captured 2 hosts generating natural ARP traffic without any scanning activity, confirming that even silent reconnaissance reveals active hosts on a LAN.

**Fast mode** discovered only 3 hosts, demonstrating that speed optimizations can reduce discovery completeness on less-responsive hosts.

**Sleep mode** (`-s 5`) discovered all 5 hosts while generating significantly less ARP broadcast noise - a meaningful evasion trade-off.

**Retry count** (`-c 5`) produced 36 total ARP packets and confirmed all 5 hosts with high reliability, useful for unstable or lossy network environments.

---

## Security Relevance

**OUI and MAC Vendor Analysis**

The first 3 bytes (24 bits) of every MAC address are called the OUI - Organizationally Unique Identifier. These bytes are globally registered and assigned by the IEEE to hardware manufacturers. In every result from this lab, the OUI prefixes `00:50:56` and `00:0c:29` both belong to VMware, Inc.

In a real engagement, MAC vendor identification is a critical early step because it reveals the likely device type before any port scanning begins. An OUI of `00:50:56` immediately signals VMware virtualization infrastructure. Other common examples include `b8:27:eb` (Raspberry Pi Foundation), `00:17:88` (Philips Hue / IoT), `f4:f5:db` (Ubiquiti Networks / network hardware), and `3c:22:fb` (Apple Inc.). Knowing that a host is a Raspberry Pi, a Ubiquiti access point, or a Cisco router before running a single port scan informs which exploits, default credentials, and attack vectors are most likely to succeed.

**ARP as an Unavoidable Attack Surface**

ARP operates at Layer 2 and cannot be disabled on a functioning LAN - hosts must respond to ARP requests to communicate. This makes ARP-based host discovery fundamentally more reliable than ICMP ping sweeps or TCP scans, both of which can be blocked by host-based firewalls. A host running a software firewall that drops all ICMP and TCP traffic will still respond to ARP, making it visible to Netdiscover even when invisible to Nmap's default scans.

**Passive Scanning for Stealth**

The passive scan (`-p`) captured 2 live hosts by observing natural ARP traffic with zero outbound packets. This technique is completely undetectable by network intrusion detection systems because it generates no anomalous traffic. In real engagements, passive ARP listening is used after gaining initial access to a network segment to quietly map the environment before any active scanning begins.

**Identifying Dual-Homed Hosts**

The `ip a` and `ip route` output revealed the Kali machine has interfaces on both `192.168.23.0/24` and `192.168.60.0/24`. In a real engagement, a compromised dual-homed host is a pivot point - it provides access to network segments that are not directly reachable from the attacker's original position. Identifying these segments early in enumeration is essential for mapping the full scope of accessible infrastructure.

**Saving Output for Downstream Tools**

The `-P` flag and file redirection produce output that can be directly parsed by scripts, imported into frameworks like Metasploit, or used to automatically generate target lists for the next phase of scanning. Professional penetration testers always save raw tool output as evidence for client reports and for maintaining an accurate record of what was discovered and when.

---

## Disclaimer
This lab was performed exclusively on a controlled, isolated virtual lab environment owned and operated by the tester. No external networks, public infrastructure, or third-party systems were scanned at any point. ARP scanning is restricted to the local network segment by design - ARP packets do not cross routers. All techniques documented here are for educational purposes only.
