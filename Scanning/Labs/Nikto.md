# Nikto - Web Server Vulnerability Scanning

## Objective
To perform comprehensive web server vulnerability scanning against a legal target (`scanme.nmap.org`) using Nikto, identifying misconfigurations, outdated software, dangerous HTTP methods, missing security headers, and exposed files - and to demonstrate multiple scanning techniques including tuning options, evasion methods, output formats, and Nmap integration.

---

## Tools Used
- Nikto v2.5.0 (command-line, Kali Linux)

---

## Lab Environment
- Operating System: Kali Linux
- Tool: Nikto v2.5.0
- Target: `scanme.nmap.org` (45.33.32.156)

---

## Background
Nikto is an open-source web server scanner that performs comprehensive tests against web servers, checking for over 6,700 potentially dangerous files and programs, outdated server software, version-specific problems, and server configuration issues. Unlike network-level scanners such as Nmap or Nessus, Nikto operates exclusively at the HTTP application layer - it communicates directly with the web server using crafted HTTP requests and analyzes the responses for signs of vulnerability or misconfiguration.

Nikto works by sending HTTP GET, HEAD, POST, and OPTIONS requests to a target web server and comparing responses against its built-in database of known vulnerable files, dangerous configurations, and insecure headers. Each finding references an OSVDB (Open Source Vulnerability Database) identifier or external resource. Nikto also supports tuning options that restrict or focus the scan to specific vulnerability categories, evasion techniques to bypass web application firewalls and IDS systems, and multiple output formats for reporting.

In the ethical hacking lifecycle, Nikto sits firmly in the Scanning and Enumeration phase, focused specifically on the web application attack surface. It is typically run after Nmap has confirmed port 80 or 443 is open, providing a web-specific layer of enumeration before manual exploitation begins.

---

## Scans Performed

### Scan 1: Version Check
```bash
nikto -Version
```

<img width="201" height="65" alt="image" src="https://github.com/user-attachments/assets/50a40177-db0c-44af-83ed-3781c75231d3" />

**Finding:** Nikto v2.5.0 confirmed installed and ready. Version verification is performed before any lab to confirm the tool is up to date and to document the exact version used, which is required for professional engagement reports.

---

### Scan 2: Basic Scan
```bash
nikto -h scanme.nmap.org
```

<img width="893" height="604" alt="image" src="https://github.com/user-attachments/assets/aee1d059-39c4-4511-bf41-8baa65065186" />

**Finding:** The basic scan ran 8049 requests over 2430 seconds (40.5 minutes) and reported **8 items** on the remote host. Key findings:

| Finding | Detail |
|---|---|
| Server | Apache/2.4.7 (Ubuntu) |
| Missing Header | `X-Frame-Options` not present - clickjacking risk |
| Missing Header | `X-Content-Type-Options` not set - MIME sniffing risk |
| Outdated Software | Apache/2.4.7 is outdated - current is at least Apache/2.4.54 |
| Uncommon Header | `tcn` header found at `/index` with contents: list |
| Misconfiguration | Apache `mod_negotiation` enabled with MultiViews - enables filename brute force |
| HTTP Methods | OPTIONS allowed: GET, HEAD, POST, OPTIONS |
| Directory Indexing | `/images/` directory listing enabled |
| Exposed File | `/icons/README` - Apache default file found |

- Start Time: 2026-03-12 00:24:44 (GMT-4)
- End Time: 2026-03-12 01:05:14 (GMT-4)
- Duration: 2430 seconds
- Requests: 8049
- Errors: 0

---

### Scan 3: SSL Scan (Port 443)
```bash
nikto -h scanme.nmap.org -ssl
```

<img width="613" height="144" alt="image" src="https://github.com/user-attachments/assets/a8987879-9d6f-44e0-8b2b-763894e95712" />

**Finding:** The SSL scan returned **0 hosts tested** - Nikto could not connect on port 443. This confirms that `scanme.nmap.org` does not expose HTTPS on port 443, which is consistent with the Nmap findings from Phase 2 Lab 1 where only port 80 was found open for HTTP. The absence of HTTPS on a public-facing web server is itself a security finding - all traffic between clients and the server is transmitted unencrypted, exposing session data, form submissions, and page content to interception.

---

### Scan 4: Specific Port Scan
```bash
nikto -h scanme.nmap.org -p 80
```

<img width="830" height="505" alt="image" src="https://github.com/user-attachments/assets/60cb3b3c-2250-4d9a-a747-7dc30855d386" />

**Finding:** Explicitly targeting port 80 produced identical results to the basic scan - **8 items** reported across 8050 requests in 2405 seconds. Findings were consistent:
- Apache/2.4.7 (Ubuntu) confirmed on port 80
- All 8 findings from Scan 2 reproduced exactly
- No additional findings were returned by explicitly specifying the port

Pinning a scan to a specific port is standard practice in engagements where a host runs multiple web services on non-standard ports (e.g., 8080, 8443, 8888). In this case it confirmed port 80 as the sole HTTP attack surface.

---

### Scan 5: Verbose Output Scan
```bash
nikto -h scanme.nmap.org -Display V
```

<img width="843" height="1005" alt="image" src="https://github.com/user-attachments/assets/93dabf74-37d1-4dcf-92c1-fbb3d6d316cd" />

**Finding:** The `-Display V` flag enabled verbose output, printing every HTTP request Nikto sent and the response code received. The overwhelming majority of responses were **404 Not Found** - confirming that Nikto systematically tested thousands of known vulnerable file paths, CGI scripts, backup files, and configuration files, none of which exist on this server. This is expected and meaningful behavior - the absence of hits on common vulnerable paths (such as `/phpinfo.php`, `/admin/`, `/.git/`, `/wp-admin/`) confirms the server is not running PHP, WordPress, or other commonly exploited web frameworks. The few positive findings that did surface matched those from the basic scan.

---

### Scan 6: Save Output to Text File
```bash
nikto -h scanme.nmap.org -o nikto_output.txt -Format txt
cat nikto_output.txt
```

<img width="847" height="522" alt="image" src="https://github.com/user-attachments/assets/655ea34d-5990-4791-9650-5b071ff34669" />

<img width="841" height="321" alt="image" src="https://github.com/user-attachments/assets/3cc93b8d-ec1d-49f4-bdfc-a07912a081d7" />

**Finding:** Output was successfully saved to `nikto_output.txt` using the `-Format txt` flag. The `cat` output confirmed all findings were written correctly with HTTP method prefixes (GET, HEAD, OPTIONS) prepended to each finding, showing which HTTP method triggered the discovery. The saved file format differs slightly from the terminal output - each finding is prefixed with the HTTP verb used to discover it, providing additional context useful for manual verification and reporting. Saving scan output is mandatory practice in professional engagements for evidence preservation and report generation.

---

### Scan 7: Nmap Integration - Pipe Open Ports into Nikto
```bash
nmap -p 80,443 scanme.nmap.org -oG - | nikto -h -
```

<img width="631" height="88" alt="image" src="https://github.com/user-attachments/assets/2b245594-29ae-4406-ad2f-3fb9e787bc6f" />

**Finding:** The Nmap-to-Nikto pipe returned **0 hosts tested**. This occurred because Nmap's grepable output format (`-oG`) requires specific open port status strings for Nikto's stdin parser to recognize and process the host. When ports are filtered or the grepable output does not contain an `open` state for the web port in the expected format, Nikto does not initiate a scan. This is a documented behavior of Nikto's `-h -` stdin mode. In practice, this integration works most reliably when Nmap confirms at least one web port as definitively open in the grepable output.

---

### Scan 8: Tuning 23 - Interesting Files and Misconfigurations
```bash
nikto -h scanme.nmap.org -Tuning 23
```

<img width="826" height="473" alt="image" src="https://github.com/user-attachments/assets/03f435bf-72e9-4124-b714-c5401d1f5f5c" />

**Finding:** Tuning option `23` restricts the scan to three specific test categories - interesting files (2), misconfigurations (3), and information disclosure (no category number, included by default). The focused scan completed significantly faster - **693 seconds** vs 2430 seconds for the full scan - running only 2173 requests and returning **7 items**:

- `X-Frame-Options` header missing
- `X-Content-Type-Options` header missing
- Apache/2.4.7 outdated
- `mod_negotiation` MultiViews enabled - filename brute force risk
- OPTIONS methods allowed: GET, HEAD, POST, OPTIONS
- `/images/` directory indexing enabled
- `/icons/README` Apache default file exposed

Tuning reduces scan noise and time by focusing on the most relevant vulnerability categories for a specific assessment goal. In a time-constrained engagement, tuned scans allow a tester to prioritize the most impactful findings first.

---

### Scan 9: Tuning 5 - Authentication Bypass Checks
```bash
nikto -h scanme.nmap.org -Tuning 5
```

<img width="843" height="460" alt="image" src="https://github.com/user-attachments/assets/717cc85c-d293-4d89-8d0c-33fdd927d7a8" />

**Finding:** Tuning `5` restricts the scan to authentication bypass tests - checking for paths that return 200 OK despite requiring authentication, HTTP method overrides that bypass access controls, and similar authorization weaknesses. The scan completed in **269 seconds** across 765 requests and returned **6 items** - all of which were the standard header and configuration findings (missing `X-Frame-Options`, missing `X-Content-Type-Options`, outdated Apache, MultiViews, OPTIONS methods, mod_negotiation). No authentication bypass vulnerabilities were found, confirming the server does not expose any protected paths that can be accessed without credentials using standard bypass techniques.

---

### Scan 10: Evasion Technique - Random URI Encoding
```bash
nikto -h scanme.nmap.org -evasion 1
```

<img width="840" height="390" alt="image" src="https://github.com/user-attachments/assets/6db7fe9d-33f0-4edd-8eb4-8b79f7043bde" />

**Finding:** The `-evasion 1` flag enabled Random URI encoding (non-UTF8), which encodes request URIs using random character encodings to attempt to bypass web application firewalls and intrusion detection systems that pattern-match on known Nikto request signatures. The scan header confirmed: `Using Encoding: Random URI encoding (non-UTF8)`.

Results: **3 items** found across 8048 requests in 2589 seconds - fewer findings than the basic scan (8 items). The reduction in findings is expected when using evasion - some tests that rely on exact URI formatting may not trigger the same server responses when the URI is encoded differently. The findings that did surface were the most fundamental ones (header misconfigurations and outdated software), which are detectable regardless of URI encoding. This demonstrates the trade-off between evasion and scan completeness - stealth comes at the cost of some detection capability.

- Start Time: 2026-03-12 00:32:56 (GMT-4)
- End Time: 2026-03-12 01:16:05 (GMT-4)
- Duration: 2589 seconds

---

### Scan 11: Increased Timeout Scan - Saved to File
```bash
nikto -h scanme.nmap.org -timeout 20 -o nikto_timeout_scan.txt -Format txt
cat nikto_timeout_scan.txt
```

<img width="739" height="35" alt="image" src="https://github.com/user-attachments/assets/dbc8b67d-028e-4b74-a4f9-41eb9fa7900f" />

<img width="836" height="355" alt="image" src="https://github.com/user-attachments/assets/8f0db60c-bfc5-471e-9f86-3541a721e8a5" />

**Finding:** The `-timeout 20` flag increased the per-request timeout from the default 10 seconds to 20 seconds, allowing Nikto to wait longer for slow server responses before marking a request as timed out. Output was saved to `nikto_timeout_scan.txt`. The `cat` output confirmed findings consistent with the basic scan - all 8 items captured including the `X-Frame-Options` missing header, `X-Content-Type-Options` missing header, outdated Apache version, MultiViews misconfiguration, OPTIONS methods, `/images/` directory indexing, and `/icons/README` file. Increasing the timeout is used in engagements where the target is geographically distant, under load, or behind rate-limiting infrastructure - ensuring slow responses are not incorrectly treated as timeouts and missed in the final report.

---

## Commands Used

| # | Command | Purpose |
|---|---|---|
| 1 | `nikto -Version` | Confirm installed version |
| 2 | `nikto -h scanme.nmap.org` | Full default scan |
| 3 | `nikto -h scanme.nmap.org -ssl` | SSL scan on port 443 |
| 4 | `nikto -h scanme.nmap.org -p 80` | Explicit port 80 scan |
| 5 | `nikto -h scanme.nmap.org -Display V` | Verbose output showing all HTTP requests |
| 6 | `nikto -h scanme.nmap.org -o nikto_output.txt -Format txt` | Save output to text file |
| 7 | `nmap -p 80,443 scanme.nmap.org -oG - \| nikto -h -` | Nmap integration via stdin pipe |
| 8 | `nikto -h scanme.nmap.org -Tuning 23` | Focused scan - interesting files and misconfigurations |
| 9 | `nikto -h scanme.nmap.org -Tuning 5` | Authentication bypass checks |
| 10 | `nikto -h scanme.nmap.org -evasion 1` | Random URI encoding evasion |
| 11 | `nikto -h scanme.nmap.org -timeout 20 -o nikto_timeout_scan.txt -Format txt` | Increased timeout scan saved to file |

---

## Observations

Across all 11 scans of `scanme.nmap.org` (45.33.32.156) port 80, Nikto consistently identified the following findings:

**Missing Security Headers:**
- `X-Frame-Options` not present - the browser is not instructed to prevent the page from being loaded in an iframe, enabling clickjacking attacks where an attacker overlays a transparent malicious frame on top of the legitimate site
- `X-Content-Type-Options` not set - the browser may attempt MIME type sniffing, potentially rendering a file as a different content type than intended, enabling content injection attacks

**Outdated Web Server Software:**
- Apache/2.4.7 (Ubuntu) is confirmed outdated - the current stable release is at least Apache/2.4.54. Apache 2.2.34 is the EOL for the 2.x branch. Running outdated server software means known CVEs for intermediate versions may be present and unpatched

**Apache Misconfiguration - MultiViews:**
- `mod_negotiation` is enabled with the MultiViews option, which allows Apache to serve files based on partial filename matches. This enables filename brute force attacks where an attacker can discover files by requesting partial names and observing whether Apache negotiates a match

**HTTP Methods Exposed:**
- OPTIONS method confirmed allowed, returning: GET, HEAD, POST, OPTIONS. While OPTIONS itself is not dangerous, advertising allowed methods reveals the full HTTP attack surface of the server to an attacker

**Directory Indexing:**
- `/images/` directory listing is enabled, allowing unauthenticated enumeration of all files in that directory

**Default Apache Files:**
- `/icons/README` - the Apache default README file is publicly accessible, confirming the server was deployed from a standard Apache package without removing default files

**No HTTPS:**
- Port 443 returned 0 hosts tested - the server has no HTTPS, meaning all HTTP traffic is unencrypted

**Evasion Comparison:**
- Basic scan found 8 items; evasion scan found only 3 items - demonstrating that WAF/IDS evasion techniques reduce scan completeness and should be used only when stealth is specifically required

---

## Security Relevance

**Missing Security Headers:** The absence of `X-Frame-Options` and `X-Content-Type-Options` are both listed in the OWASP Top 10 under Security Misconfiguration. These headers are single-line additions to the Apache configuration that take minutes to implement but prevent entire classes of client-side attacks. Their absence on a production server indicates the web server was deployed without a security hardening checklist.

**Outdated Apache Version:** Apache/2.4.7 was released in 2013. Over a decade of security patches have been released since then. Each unpatched version between 2.4.7 and the current release may carry known CVEs exploitable against this server. Combined with the Ubuntu 14.04 EOL finding from the Nessus lab, the entire stack - OS and web server - is running without vendor security support.

**mod_negotiation MultiViews:** This is a less commonly known but significant misconfiguration. When MultiViews is enabled, Apache will attempt to serve a file that best matches a requested URI even if the exact file does not exist. An attacker can use this behavior to enumerate filenames by requesting partial names and observing 200 vs 404 responses, effectively brute forcing the file system through the web server without triggering standard directory traversal detections.

**Directory Indexing on /images/:** Directory listing allows an attacker to enumerate every file stored in a directory without needing to guess filenames. This is particularly dangerous if backup files, configuration files, or sensitive documents are accidentally placed in a web-accessible directory.

**No HTTPS:** Running a public web server without HTTPS in 2026 means all data transmitted between the server and visitors - including any form submissions, session tokens, or personal information - travels in plaintext across the internet, subject to interception by any party with access to the network path.

**Evasion Techniques in Real Engagements:** The `-evasion 1` flag demonstrates an important concept in web application penetration testing - many organizations deploy web application firewalls (WAF) or IDS systems that detect and block known scanning tool signatures. Nikto's evasion options allow testers to assess whether the WAF correctly blocks scanning activity or whether it can be bypassed. A WAF that blocks standard Nikto but allows encoded requests provides false security - attackers use the same encoding techniques.

**Nikto vs Nessus:** While Nessus identified OS-level and network service vulnerabilities, Nikto specifically targeted the HTTP application layer. The two tools are complementary - Nessus found the Ubuntu EOL and SSH weaknesses while Nikto found the web server misconfigurations and missing headers. A complete vulnerability assessment requires both network-level and application-level scanning.

---

## Disclaimer
This lab was conducted exclusively against `scanme.nmap.org`, a host maintained by the Nmap project specifically for legal, authorized scanning and testing. All Nikto scans were performed for educational purposes only. No vulnerabilities were exploited and no data was accessed or modified. All techniques documented here are for educational purposes only.
