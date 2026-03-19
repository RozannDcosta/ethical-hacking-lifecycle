# Shodan - Internet Facing Device Reconnaissance

## Objective
To perform passive reconnaissance using Shodan to discover internet - exposed devices, open ports, live camera feeds, and vulnerable infrastructure, without directly interacting with any target system.

---

## Tools Used
- Shodan (shodan.io - browser-based)

---

## Lab Environment
- Operating System: Kali Linux
- Browser: Any web browser (fire fox)
- Target: CCTV-related internet-facing devices (global and India-specific)

---

## Background
Shodan is a search engine for internet-connected devices. Unlike Google, which indexes web content, Shodan indexes **banners** — raw responses from servers, routers, webcams, industrial systems, and IoT devices. It reveals what services are running, what ports are open, and what information devices are exposing to the public internet - all without sending a single packet to the target.

---

## Searches Performed

### 1. `org:"cctv"`
Basic search for devices belonging to organizations with "cctv" in their name.

![WhatsApp Image 2026-03-03 at 21 47 58](https://github.com/user-attachments/assets/910883e6-aebc-4841-9f99-221f5888588f)

**Finding:** Returned **736 total results** globally. India topped the list with 407 results, followed by the United States (187) and China (77). Top ports included 80 (HTTP), 161 (SNMP), 443 (HTTPS), 2000, and 22 (SSH). Notable organizations included Rugged CCTV, Wefe Telecom And Cctv Services, and Fortis Central CCTV Platform. SNMP banners were visible on several devices, revealing device uptime, engine data, and MikroTik router details - all without authentication.

---

### 2. `org:"cctv" port:80`
Filters CCTV organization devices exposing port 80 (HTTP - unencrypted web interface).

![WhatsApp Image 2026-03-03 at 21 47 20](https://github.com/user-attachments/assets/def641c3-12fc-4c1e-bb97-e3faee8ee8bc)

**Finding:** Returned **77 results**. India dominated with 61 results. Several devices showed "Default Web Site Page" - meaning the web server is running but not configured, which is a common misconfiguration. One device from Wefe Telecom And Cctv Services in Bhubaneswar, India was actively serving content with no caching headers, suggesting a live web interface. Default pages indicate devices that have never been properly secured after installation.

---

### 3. `org:"cctv" http.title:"login"`
Searches for CCTV devices exposing a login page on their web interface.

![WhatsApp Image 2026-03-03 at 21 22 29 (2)](https://github.com/user-attachments/assets/032d29ea-8eb8-4b24-bdf3-feb1a440728d)

**Finding:** Returned **20 results**. Multiple devices from Rugged CCTV in Dallas, United States were found exposing WHM (Web Host Manager) login panels publicly on the internet. SSL certificates were issued by Let's Encrypt. Exposed login panels are direct targets for brute force attacks, credential stuffing, and unauthorized access attempts.

---

### 4. `org:"cctv" product:"Hikvision"`
Searches for Hikvision — the world's most widely deployed CCTV brand - within CCTV organizations.

![WhatsApp Image 2026-03-03 at 21 28 14](https://github.com/user-attachments/assets/15b43044-5f0a-4499-9584-851538a3f88f)

**Finding:** Returned **20 results** with the United States (16) and India (4) as top countries. Rugged CCTV appeared as the top organization with 16 devices. Hikvision devices are historically associated with multiple critical CVEs including default credential vulnerabilities and remote code execution flaws, making their exposure on the public internet a significant security risk.

---

### 5. `has_screenshot:true webcam`
Searches for internet-exposed webcams where Shodan has already captured a live screenshot.

![WhatsApp Image 2026-03-03 at 21 22 29 (3)](https://github.com/user-attachments/assets/7ac6fac9-a502-4344-bb2f-222f1e3220ab)

**Finding:** Returned **80 results** globally across the United States, Germany, South Korea, United Kingdom, and Italy. A live camera feed from Yokohama, Japan (KDDI Corporation) was captured by Shodan running MJPG-Streamer - a lightweight streaming server. The feed was accessible with no authentication (`HTTP/1.0 200 OK`), meaning anyone with the IP address could view the live stream. Cache-Control headers showed `no-store, no-cache` confirming a live feed.

---

### 6. `cctv country:"IN"`
Searches for CCTV-related devices specifically in India.

![WhatsApp Image 2026-03-03 at 21 22 29 (4)](https://github.com/user-attachments/assets/a4453afa-8dc4-4494-8ab0-57e476110c1a)

**Finding:** Returned **77 results** across Indian cities — Kalyān (10), Bengaluru (7), Bhopāl (5), Chennai (5), and Mumbai (4). Top ports included 80, 8000, 135, 3389, and 443. Port 3389 (RDP) was open on 6 devices, which is a critical finding as exposed RDP is one of the most commonly exploited attack vectors for ransomware and unauthorized remote access. One device revealed a MikroTik router with SNMP data including engine ID, uptime, and enterprise details - all exposed without authentication.

---

## Commands Used

| Query | Purpose |
|---|---|
| `org:"cctv"` | Find all devices from CCTV organizations |
| `org:"cctv" port:80` | Find CCTV devices with unencrypted HTTP interfaces |
| `org:"cctv" http.title:"login"` | Find exposed login panels |
| `org:"cctv" product:"Hikvision"` | Find Hikvision camera devices |
| `has_screenshot:true webcam` | Find cameras with live screenshots captured |
| `cctv country:"IN"` | Find CCTV-related devices in India |

---

## Observations

Across all six searches, the lab demonstrated that a significant number of CCTV-related devices are publicly accessible on the internet with little to no authentication or security hardening. Key findings include:

- **736 devices** found under CCTV organizations globally, with India being the largest contributor
- **Live camera feeds** accessible with no authentication, captured by Shodan in real time
- **Exposed RDP (port 3389)** on Indian CCTV devices — one of the highest-risk findings, as RDP exposure is a primary ransomware entry point
- **Default web pages** on multiple devices indicating unconfigured and likely unmonitored systems
- **Hikvision devices** exposed publicly, a brand with a documented history of critical CVEs
- **SNMP banners** leaking device details including engine IDs, uptime, and hardware information without requiring any credentials
- **WHM login panels** publicly exposed, presenting brute force and credential stuffing opportunities

---

## Security Relevance

Shodan demonstrates that the internet is filled with devices that were never designed to be publicly exposed or were deployed without proper security configuration. CCTV systems are a prime example of this problem.

Key risks exposed through this lab:

- **No authentication on live feeds** — anyone with the IP can view live camera footage, posing serious physical security and privacy risks
- **Exposed RDP** — port 3389 open on internet-facing devices is one of the most exploited entry points for ransomware groups and APT actors
- **Default configurations** — unconfigured web servers and default pages indicate devices that were deployed and forgotten, with no ongoing security management
- **SNMP exposure** — SNMP without authentication leaks detailed device information including hardware identifiers and uptime, useful for fingerprinting and targeted attacks
- **Hikvision vulnerabilities** — publicly exposed Hikvision devices are frequently targeted due to well-known CVEs including default credential exploitation and remote code execution

Organizations deploying CCTV and IoT infrastructure should place devices behind firewalls and VPNs rather than exposing them directly to the internet, disable SNMP or use SNMPv3 with authentication, change default credentials immediately on deployment, and conduct regular Shodan searches against their own IP ranges to identify unintended exposure.

---

## Disclaimer
This lab was performed solely using Shodan's passive indexing service for educational purposes . No devices were accessed, connected to, or interacted with in any way. All information was gathered passively through Shodan's publicly available search interface.
