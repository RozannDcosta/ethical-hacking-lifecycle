# WHOIS Lookup – Public Domain Information Gathering

## Objective
To perform passive reconnaissance by gathering publicly available domain registration details using WHOIS lookup, and to understand what information an attacker or security professional can extract before any direct interaction with a target system.

---

## Tools Used
- `whois` (Kali Linux CLI)
- WHOIS Web Service (whois.com) — used for cross-verification

---

## Lab Environment
- Operating System: Kali Linux
- Target Domain: `example.com` (public test domain)

---

## Steps Performed

1. Opened the terminal in Kali Linux.
2. Ran the following command to perform a WHOIS lookup via CLI:
   ```bash
   whois example.com
   ```
<img width="675" height="282" alt="image" src="https://github.com/user-attachments/assets/4e518f05-c2bd-4ad6-981c-eeacd66f9807" />

   
3. Cross-verified results using the web-based WHOIS service at [whois.com](https://www.whois.com).
<img width="802" height="557" alt="image" src="https://github.com/user-attachments/assets/3858766f-e5f2-4742-9bcd-6156e45b5b6a" />

4. Reviewed the full output including registration dates, registrant details, nameserver information, and registrar data.

---

## Commands Used

```bash
whois example.com
```

---

## Observations

The WHOIS lookup revealed the following key details about `example.com`:

- **Registration Date:** 1995-08-14 — one of the oldest registered domains, indicating a well-established entity
- **Expiration Date:** 2026-01-16 — a domain nearing expiration can be a target for domain hijacking attempts
- **Nameservers:** `elliott.ns.cloudflare.com` and `hera.ns.cloudflare.com`

The use of Cloudflare nameservers is a significant finding. It indicates that DNS management and traffic routing are handled through Cloudflare's infrastructure, meaning the real origin IP of the web server is likely masked behind Cloudflare's proxy. From an attacker's perspective, this means direct IP-based attacks would hit Cloudflare's network rather than the actual target server — making origin IP discovery a necessary next step in a real engagement.

---

## Security Relevance

WHOIS data is entirely public and requires no special access to retrieve, making it a first-stop resource for both attackers and defenders.

Key risks exposed by WHOIS data:

- **Hosting provider / DNS provider identification** — reveals infrastructure stack (e.g., Cloudflare usage)
- **Domain expiration dates** — expired or soon-to-expire domains are targets for domain hijacking
- **Registrant contact details** — when privacy protection is not enabled, names, emails, and phone numbers are exposed, enabling social engineering attacks
- **Registrar information** — can be used to craft convincing phishing emails impersonating the registrar

Organizations should use **WHOIS privacy protection** services to mask registrant details, and should actively monitor domain expiration dates to prevent hijacking.

---

## Learning Outcome

This lab demonstrated that before touching a single packet on a target network, a significant amount of infrastructure intelligence can be gathered passively through WHOIS. In a real penetration testing engagement, this data feeds directly into the next steps — DNS interrogation, infrastructure mapping, and social engineering recon.

Key takeaway: the nameserver discovery (Cloudflare) immediately tells a tester that standard IP-based scanning won't reach the origin server. The next step in a real engagement would be to attempt origin IP discovery through techniques like checking historical DNS records, SSL certificate transparency logs, or email header analysis.

---

## Disclaimer
This lab was performed solely on a public test domain (`example.com`) for educational purposes as part of CEH (Certified Ethical Hacker) training. No unauthorized systems were accessed. All information gathered is publicly available.
