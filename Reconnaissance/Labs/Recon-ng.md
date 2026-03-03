# Recon-ng – Automated OSINT Framework Reconnaissance

## Objective
To perform automated passive reconnaissance using Recon-ng to discover subdomains and hosts associated with a target domain, and to understand how a modular OSINT framework can systematically enumerate an organization's internet-facing infrastructure.

---

## Tools Used
- `recon-ng` v5.1.2 (Kali Linux CLI)

---

## Lab Environment
- Operating System: Kali Linux
- Target Domain: `microsoft.com` (public domain used for educational purposes)

---

## Background
Recon-ng is a full-featured web reconnaissance framework built into Kali Linux. It follows a modular architecture similar to Metasploit — operators load specific modules, set targets, and run automated information gathering across multiple data sources. Unlike running individual tools manually, Recon-ng allows chaining multiple modules together within a single workspace, building a comprehensive picture of a target's digital footprint systematically.

---

## Steps Performed

### 1. Launching Recon-ng
Launched Recon-ng from the terminal:
```bash
recon-ng
```

<img width="840" height="431" alt="image" src="https://github.com/user-attachments/assets/cd132ed6-09d0-4c46-9236-8d12a0153a52" />

Recon-ng v5.1.2 launched successfully. No modules were enabled by default — all modules must be installed from the marketplace before use.

---

### 2. Creating a Workspace
Created a dedicated workspace to keep results organized:
```
workspaces create exmaple

```
Workspaces in Recon-ng isolate results per engagement, allowing a tester to run multiple investigations without mixing data.

---

### 3. Module 1 – Hackertarget (Subdomain Discovery)
Installed and loaded the `hackertarget` module, which queries the HackerTarget API to retrieve subdomains for a target domain:
```
marketplace install recon/domains-hosts/hackertarget
modules load recon/domains-hosts/hackertarget
options set SOURCE microsoft.com
run
```

<img width="594" height="76" alt="image" src="https://github.com/user-attachments/assets/17667e2d-8641-44c7-aec3-9221c47452fd" />

**Result:**

<img width="354" height="67" alt="image" src="https://github.com/user-attachments/assets/63093c26-6f4a-45b8-a4f5-3d92462ba9ae" />

The hackertarget module returned **50 new hosts** for `microsoft.com`.

---

### 4. Module 2 – Brute Hosts (DNS Brute Force)
Installed and loaded the `brute_hosts` module, which performs DNS brute forcing by testing common subdomain wordlists against the target domain:
```
marketplace install recon/domains-hosts/brute_hosts
modules load recon/domains-hosts/brute_hosts
options set SOURCE microsoft.com
run
```

<img width="654" height="120" alt="image" src="https://github.com/user-attachments/assets/b8ddb834-52bc-4742-bec6-60b3dc63d5ed" />

**Result:**

<img width="342" height="64" alt="image" src="https://github.com/user-attachments/assets/cfd2db11-ffc1-403b-b241-ad08d1dac1c5" />

The brute_hosts module returned **958 total hosts (640 new)** — significantly expanding the host discovery beyond what hackertarget alone returned.

---

### 5. Dashboard Summary
Ran the `dashboard` command to view the full activity and results summary:
```
dashboard

```

<img width="404" height="509" alt="image" src="https://github.com/user-attachments/assets/8e37b5b9-697e-497e-9323-5e5d74e2a8e5" />


A total of **2 module runs** were completed, discovering **640 unique hosts** stored in the workspace database.

---

## Commands Used

| Command | Purpose |
|---|---|
| `recon-ng` | Launch the Recon-ng framework |
| `workspaces create <name>` | Create an isolated workspace for the engagement |
| `marketplace install <module>` | Install a module from the Recon-ng marketplace |
| `modules load <module>` | Load an installed module |
| `options set SOURCE <domain>` | Set the target domain |
| `run` | Execute the loaded module |
| `dashboard` | View activity and results summary |
| `show hosts` | Display all discovered hosts in the workspace |

---

## Observations

Two modules were used in this lab, each using a different discovery technique:

- **hackertarget** queried an external API to retrieve known subdomains, returning 50 hosts — fast but limited to what HackerTarget has indexed
- **brute_hosts** performed DNS brute forcing using a wordlist, returning 958 total hosts (640 new) — a far larger result set that significantly expanded the attack surface

The combination of both modules demonstrates why using multiple recon techniques matters. A single module would have missed 640 hosts that were only discovered through brute force enumeration. In a real engagement, each of these 640+ hosts represents a potential entry point that feeds directly into the next phase — scanning and enumeration.

---

## Security Relevance

Recon-ng demonstrates how an attacker can systematically automate the enumeration of an organization's entire internet-facing infrastructure using only publicly available data sources.

Key risks exposed through this lab:

- **958 discoverable subdomains** — each subdomain is a potential attack surface including forgotten dev environments, staging servers, old login portals, and misconfigured services
- **Automated enumeration** — Recon-ng can chain dozens of modules together, meaning a determined attacker can build a comprehensive target profile in minutes
- **DNS brute forcing** — common subdomain names (mail, vpn, remote, admin, dev, staging) are predictable and can be discovered without any special access
- **Workspace data persistence** — all discovered hosts are stored in a local database and can be fed into subsequent modules for deeper enumeration, making the framework highly efficient for multi-phase attacks

Organizations should conduct regular subdomain audits to identify and decommission forgotten or unused subdomains, implement DNS monitoring to detect unusual enumeration activity, and ensure all internet-facing subdomains are accounted for in their asset inventory.

---

## Disclaimer
This lab was performed solely on a public domain (`microsoft.com`) for educational purposes. No unauthorized systems were accessed. All information gathered is publicly available through passive reconnaissance only.
