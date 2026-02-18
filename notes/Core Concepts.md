# Core Concepts & Terminology

## Reconnaissance

- The first phase of the ethical hacking methodology (and the cyber attack lifecycle). It involves gathering as much information as possible about a target system, network, or organization *before* launching an attack.

## Footprinting

- Footprinting is a part of reconnaissance process which is used for gathering possible information about a target computer system or network. 

## Types of Reconnaissance

| Feature | Passive Reconnaissance | Active Reconnaissance |
| :--- | :--- | :--- |
| **Interaction** | No direct interaction with the target. You never touch their systems. | Direct interaction with the target. You are touching their systems. |
| **Detection Risk** | **Very Low / Undetectable.** The target cannot see you. | **High.** Logs, IDS/IPS, and firewalls can detect your probes. |
| **Data Source** | Public information, search engines, social media, third-party databases, job sites. | Pinging the target, scanning ports, browsing the website, querying their DNS servers directly. |
| **Example** | Using Google to find a PDF on the company site, checking LinkedIn for employees, looking up the domain on `whois.net`. | Running `nslookup` against their DNS server, using `ping` to see if a host is alive, performing a port scan with Nmap. |

## Objectives of Footprinting

- Know Security Posture – The data gathered will help us to get an overview of the security posture of the company such as details about the presence of a 
firewall, security configurations of applications etc. 
- Reduce Attack Area – Can identify a specific range of systems and concentrate on particular targets only. This will greatly reduce the number of systems we are focussing on. 
- Identify vulnerabilities – we can build an information database containing the vulnerabilities, threats, loopholes available in the system of the target organization. 
- Draw Network map – helps to draw a network map of the networks in the target organization covering topology, trusted routers, presence of server and other information. 


#### Network Footprinting Objectives

| Objective | Description |
|:----------|:------------|
| **Access Control** | Authentication mechanisms, access policies, and security controls |
| **Accessible Systems** | Systems that can be reached from the internet/internal network |
| **DNS** | Domain Name System records, zone transfers, and naming conventions |
| **Firewall Vendors** | Identifying firewall brands, versions, and rule configurations |
| **IDS Systems** | Intrusion Detection System presence, types, and placements |
| **IP Networks** | Identifying network ranges, subnets, and IP address allocation |
| **Phone System (Analog/VoIP)** | Telephony infrastructure, VoIP servers, and phone exchange info |
| **Routing/Routed Protocols** | Dynamic routing protocols (OSPF, BGP, EIGRP) and static routes |
| **VPN Endpoints** | VPN server locations, protocols used, and entry points |
| **Websites** | Web servers, applications, and associated technologies |

---

#### Organization Footprinting Objectives

| Objective | Description |
|:----------|:------------|
| **Business Associations** | Partners, vendors, clients, and third-party relationships |
| **Company History** | Mergers, acquisitions, rebranding, and historical events |
| **Directory Information** | Employee directories, organizational charts, and contact lists |
| **Office Locations** | Physical addresses, branch offices, and remote sites |
| **Organization Structure** | Departments, hierarchy, subsidiaries, and business units |
| **Phone Numbers** | Main lines, direct dials, fax numbers, and VoIP extensions |
| **Websites** | Corporate sites, regional sites, and microsites |

---

#### Hosts Footprinting Objectives

| Objective | Description |
|:----------|:------------|
| **Enumerated Information** | Share names, session data, and system-specific details |
| **Internet Reachability** | Whether hosts are accessible from the internet |
| **Listening Services** | Active ports, running services, and daemons |
| **Mobile Devices** | BYOD policies, mobile device types, and management systems |
| **Operating System Versions** | OS types, versions, patch levels, and build numbers |
| **SNMP Information** | SNMP community strings, MIB data, and system info |
| **Users/Groups** | Local users, domain users, groups, and administrative accounts |



