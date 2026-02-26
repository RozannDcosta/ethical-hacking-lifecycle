## Objective  
To performed passive reconnaissance using nslookup to gather IP addresses and DNS records for a target domain.

## Background  
In this exercise, I used nslookup to gather publicly available DNS information. 

## Steps Performed  
1. I ran 'nslookup example.com' to retrieve the domain’s IPv4 addresses. 
<img width="230" height="187" alt="image" src="https://github.com/user-attachments/assets/af4762bd-20eb-417c-90e9-ac6ba42a31f6" />

2. I then performed a reverse lookup by running 'nslookup <IP address>'. In this step, the result returned "NXDOMAIN," meaning no domain was found for that IP. 
<img width="415" height="36" alt="image" src="https://github.com/user-attachments/assets/a0ff6235-55b6-435f-9147-8faea6ab0eae" /> 

3. To verify further, I used 'dig' command for a reverse lookup as well, which confirmed that the IP did not resolve back to a domain.
<img width="738" height="329" alt="image" src="https://github.com/user-attachments/assets/0d49783f-071f-4f48-8390-81f96ceccef4" />

4. After that, I ran 'nslookup -type=any example.com' to gather all types of DNS records.
<img width="685" height="660" alt="image" src="https://github.com/user-attachments/assets/42f44d57-bea9-4a1f-9ce9-3762382f3b9c" />


## Commands Used  
- `nslookup example.com`: Retrieves the domain’s A (IPv4) record.  
- `nslookup <IP>`: Performs a reverse lookup to find a domain from an IP address.  
- `dig <IP>`: Used to validate the reverse lookup and cross-check IP-to-domain mapping.
- `nslookup -type=any example.com`: Retrieves all DNS record types.  

## Findings  
The domain resolved to multiple IP addresses, both IPv4 and IPv6. However, during the reverse lookup, no domain was associated with the IP (NXDOMAIN). Used dig to reaffirmed that the IP did not map back to a domain name. By using the '-type=any' command, I gathered all DNS records, including A, AAAA, MX, SOA, DS, and DNSKEY, which gave me a full view of the domain’s infrastructure
