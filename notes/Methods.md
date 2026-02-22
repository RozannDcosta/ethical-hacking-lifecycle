# CEH Footprinting Methods and Tools - Complete Reference

## Overview
Footprinting is the first phase of ethical hacking, involving the collection of information about a target system, network, or organization before launching an attack. This document covers all CEH footprinting methods and associated tools organized by category.

---

## 1. Search Engine Reconnaissance

Using search engines to find sensitive information indexed online.

### Google Hacking / Google Dorks
Advanced search operators to locate specific information.

| Operator | Purpose |
|:---------|:--------|
| `site:` | Limit results to a specific domain |
| `filetype:` | Find specific file types (PDF, XLS, DOC) |
| `intitle:` | Find pages with specific terms in title |
| `inurl:` | Find pages with specific terms in URL |
| `cache:` | View cached version of a page |
| `link:` | Find pages linking to target |
| `related:` | Find similar sites |

**Tools for Google Dorking:**
- **Pagodo** - Automates Google dork searches
- **Google Hacking Database (GHDB)** - Repository of dork queries

### IoT/Device Search Engines
Specialized search engines for internet-connected devices.

| Tool | Purpose |
|:-----|:--------|
| **Shodan** | Search for internet-connected devices and services |
| **Censys** | Search for certificates, hosts, and websites |
| **Thingful** | Search for IoT devices specifically |

---

## 2. Web Services Footprinting

### Netcraft
Website investigation tool that provides:
- Site background information
- Network details
- Hosting history
- Subdomain discovery
- Technology identification

### DNSdumpster
DNS reconnaissance tool that maps subdomains and provides visual representation of domain infrastructure.

### Wappalyzer
Browser extension that identifies technologies used on websites including:
- Web servers (Apache, Nginx, IIS)
- CMS (WordPress, Joomla, Drupal)
- JavaScript frameworks
- Analytics tools
- E-commerce platforms

### BuiltWith
Technology profiler similar to Wappalyzer with historical data.

### Archive.org (Wayback Machine)
Access historical versions of websites to find:
- Old vulnerabilities
- Removed content
- Development/staging sites
- Configuration files that were briefly exposed

---

## 3. DNS Footprinting

Gathering information about domain names and network infrastructure through DNS queries.

### DNS Record Types

| Record | Purpose |
|:-------|:--------|
| **A** | IPv4 address mapping |
| **AAAA** | IPv6 address mapping |
| **MX** | Mail exchange servers |
| **NS** | Name servers |
| **TXT** | Text records (SPF, verification) |
| **SOA** | Start of Authority |
| **CNAME** | Canonical name (aliases) |
| **SRV** | Service records |

### DNS Tools

| Tool | Command Example | Description |
|:-----|:----------------|:------------|
| **nslookup** | `nslookup example.com` | Interactive/non-interactive DNS lookup |
| **dig** | `dig example.com ANY` | Detailed DNS queries |
| **host** | `host -t mx example.com` | Simple DNS lookup |
| **dnsrecon** | `dnsrecon -d example.com` | DNS enumeration tool |
| **fierce** | `fierce --domain example.com` | DNS reconnaissance |
| **DNSRecon** | `dnsrecon -d example.com -t axfr` | DNS enumeration with zone transfer |

### Zone Transfer Attempt
```bash
# Linux
dig -t AXFR @nameserver example.com

# Windows
nslookup
> server nameserver
> ls -d example.com
