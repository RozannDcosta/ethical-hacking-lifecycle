# theHarvester - Email & Subdomain Harvesting

## Objective
To perform passive reconnaissance using theHarvester to enumerate publicly available email addresses, subdomains, and hosts associated with a target domain, and to understand how attackers leverage OSINT tools to map an organization's digital footprint.

---

## Tools Used
- `theHarvester` (Kali Linux CLI)

---

## Lab Environment
- Operating System: Kali Linux
- Target Domain: `microsoft.com` (public domain used for educational purposes)

---

## Steps Performed

1. Ran theHarvester against `microsoft.com` using all available sources:
   ```bash
   theHarvester -d microsoft.com -b all
   ```
   <img width="345" height="36" alt="image" src="https://github.com/user-attachments/assets/859d64a2-8f93-49d3-875b-a92c898be303" />

2. Observed the tool querying multiple sources including Baidu, Certspotter, CRTsh, Hackertarget, Bing, DuckDuckGo, Yahoo, Sitedossier, and more.

3. Reviewed the emails discovered in the output.

   <img width="238" height="103" alt="image" src="https://github.com/user-attachments/assets/94aef9ba-d9dc-4bf2-afa5-c02596f02e33" />

4. Reviewed the total hosts count returned by the tool.

   <img width="160" height="24" alt="image" src="https://github.com/user-attachments/assets/238b492b-6ea0-459c-a5bb-59974823874f" />

---

## Commands Used

- `theHarvester -d microsoft.com -b all` - runs theHarvester against the target domain using all available data sources

---

## Observations

**Emails Found: 5**

The tool returned the following emails:

| Email | Notes |
|---|---|
| `azure-noreply@microsoft.com` | Automated service email |
| `mayuresh.kambli@microsoft.com` | Real employee email |
| `scottgu@microsoft.com` | Real employee email - Scott Guthrie is a well known Microsoft Executive VP |
| `abc@microsoft.com` | Likely a junk/test result |
| `abc@microsoft.commicrosoft.com` | Malformed result - parsing error by the tool |

Two of the emails (`abc@microsoft.com` and `abc@microsoft.commicrosoft.com`) appear to be malformed or junk results, which is common when using `-b all` across multiple sources with varying data quality. 

**Hosts Found: 4243**

The tool discovered 4243 subdomains associated with `microsoft.com`, including notable ones such as `azure.microsoft.com`, `careers.microsoft.com`, `account.microsoft.com`, `answers.microsoft.com`, and `billing.microsoft.com`. This gives a broad view of Microsoft's infrastructure and attack surface.


---

## Security Relevance

theHarvester demonstrates how much information about an organization is publicly accessible without ever interacting with their systems directly.

Key risks exposed through email and subdomain harvesting:

- **Employee emails** - real employee emails like `scottgu@microsoft.com` can be used for spear phishing or targeted social engineering attacks against high-value individuals
- **Subdomain enumeration** - 4243 discovered hosts reveal a massive attack surface. Each subdomain is a potential entry point - forgotten dev environments, old login portals, or misconfigured services
- **Infrastructure mapping** - subdomains like `billing.microsoft.com`, `account.microsoft.com`, and `azure.microsoft.com` reveal business-critical services that attackers would prioritize
- **Email format discovery** - even a few real emails reveal the organization's email naming convention (e.g., `firstname.lastname@microsoft.com`), allowing attackers to generate valid email addresses for any employee

Organizations should monitor their subdomain exposure regularly, use email security controls like DMARC, DKIM, and SPF to prevent spoofing, and conduct periodic OSINT assessments on their own domains to understand what attackers can see.

---

## Disclaimer
This lab was performed solely on a public domain (`microsoft.com`) for educational purposes as part of CEH (Certified Ethical Hacker) training. No unauthorized systems were accessed. All information gathered is publicly available through passive reconnaissance only.
