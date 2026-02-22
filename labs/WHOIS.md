# WHOIS Lookup Lab – Public Domain Information Gathering

## Objective
The objective of this lab is to practice passive reconnaissance by gathering publicly available domain registration details using WHOIS lookup services.

## Tools Used
- WHOIS Web Service (whois.com)

## Lab Environment
- Operating System: Kali Linux
- Target Domain: example.com (public test domain)

## Steps Performed
1. Opened a web browser in Kali Linux.
2. Navigated to a WHOIS lookup service.
3. Entered the target domain (example.com).
4. Reviewed the output, including domain registration date, expiration date, and name server information.

## Observations
The WHOIS lookup revealed that example.com was registered on 1995-08-14 and is set to expire on 2026-01-16.  
The domain uses Cloudflare name servers (elliott.ns.cloudflare.com and hera.ns.cloudflare.com), indicating that DNS management and traffic protection are handled through Cloudflare services.

## Learning Outcome
This exercise demonstrated how WHOIS data can reveal important information such as domain age, expiration timeline, and infrastructure details like DNS providers. Understanding this data helps security professionals assess a domain’s exposure and digital footprint.

## Security Relevance
Public WHOIS information can be leveraged by attackers to identify hosting providers, infrastructure patterns, and domain lifecycle details. Organizations should monitor their public domain records and use privacy protection services when appropriate to reduce unnecessary exposure.

## Disclaimer
This lab was performed solely on a public test domain (example.com) for educational purposes. No unauthorized systems were accessed.
