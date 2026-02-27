# Google Dorking - Advanced Search Reconnaissance

## Objective
To perform passive reconnaissance using Google Dorking techniques to discover publicly indexed sensitive information, exposed files, admin panels, and login portals associated with a target domain - using only a web browser and advanced Google search operators.

---

## Tools Used
- Google Search

---

## Lab Environment
- Operating System: Windows
- Browser: Google
- Target Domain: `microsoft.com` (public domain used for educational purposes)

---

## Background
Google Dorking (also known as Google Hacking) uses advanced search operators to filter and find specific types of information indexed by Google. Unlike other recon techniques, Google Dorking requires no special tools and leaves no traces on the target's systems - making it a purely passive reconnaissance method. The results are entirely based on what Google has already crawled and indexed.

---

## Dorks Executed
`site:microsoft.com filetype:pdf`

Searches for publicly indexed PDF files hosted on microsoft.com.

<img width="938" height="492" alt="image" src="https://github.com/user-attachments/assets/d83f8d84-bacc-48f8-8b6b-3abfd59474ed" />

**Finding:** Returned publicly accessible PDF documents including FAQ pages and Terms of Use documents from `designer.microsoft.com`. PDFs can contain metadata such as author names, software versions, and internal file paths — all useful for building a target profile.

---
`filetype:csv site:microsoft.com`
Searches for publicly indexed CSV files hosted on microsoft.com.

<img width="941" height="258" alt="image" src="https://github.com/user-attachments/assets/35383afb-3fac-4be3-a69f-072ad74afb16" />

**Finding:** Returned a result from `admin.microsoft.com/UserManagement/Samples` - an "Import User Sample" CSV file. This is significant as it reveals an admin portal path (`admin.microsoft.com`) and suggests user management functionality that could be a target for further enumeration.

---
 
`site:microsoft.com inurl:login`
Searches for pages within microsoft.com that contain "login" in the URL.

<img width="944" height="443" alt="image" src="https://github.com/user-attachments/assets/3855a476-863b-42a0-b908-cea7261b6eeb" />

**Finding:** Returned login-related pages including SQL Server login documentation. While these are legitimate public pages, in a real engagement this dork helps identify authentication portals that could be tested for weak credentials or brute force vulnerabilities.

---

`site:microsoft.com inurl:admin`
Searches for pages within microsoft.com that contain "admin" in the URL.

<img width="944" height="444" alt="image" src="https://github.com/user-attachments/assets/72b37356-25ae-46ba-92dd-4f1d95903675" />

**Finding:** Returned results including Windows Admin Center and Microsoft 365 Admin Center pages. Discovering admin panel URLs is a critical recon finding as these are high value targets for attackers attempting unauthorized access.

---

`intitle:"index of" site:microsoft.com`
Searches for directory listing pages within microsoft.com , pages that expose file and folder structures.

<img width="948" height="434" alt="image" src="https://github.com/user-attachments/assets/f4a21c91-85e9-4051-9083-198e612fd935" />

**Finding:** Returned directory index pages including an archive directory at `cran.microsoft.com/src/contrib/Archive/startup` showing file listings with timestamps. Exposed directory listings allow attackers to browse server contents directly, potentially discovering sensitive files or backup data.

---

`site:microsoft.com "internal use only"`
Searches for pages within microsoft.com that contain the phrase "internal use only."

<img width="932" height="450" alt="image" src="https://github.com/user-attachments/assets/2c608f3b-17ff-4d1b-8825-f2e3a5543eaf" />

**Finding:** Returned two notable results - a Visual Studio page explicitly marked "MICROSOFT INTERNAL USE ONLY" from 2017, and a Microsoft Community Hub post discussing how to remove "Company Confidential - FOR INTERNAL USE ONLY" labels from PowerPoint files. Documents marked as internal that are publicly indexed represent a data exposure risk, as they may contain information not intended for public access.

---

## Commands Used

| Dork | Purpose |
|---|---|
| `site:microsoft.com filetype:pdf` | Find publicly indexed PDF files |
| `filetype:csv site:microsoft.com` | Find publicly indexed CSV files |
| `site:microsoft.com inurl:login` | Discover login portals |
| `site:microsoft.com inurl:admin` | Discover admin panels |
| `intitle:"index of" site:microsoft.com` | Find exposed directory listings |
| `site:microsoft.com "internal use only"` | Find potentially sensitive indexed content |

---

## Observations

All six dorks returned results, demonstrating that even a well-secured organization like Microsoft has publicly indexed content that could be leveraged during a recon phase. Key findings include:

- An admin portal path discovered via the CSV filetype dork (`admin.microsoft.com`)
- Exposed directory listings revealing file structures and timestamps
- Documents explicitly marked as internal that are publicly accessible via Google
- Admin and login panel URLs that could serve as entry points for further testing

---

## Security Relevance

Google Dorking requires no hacking tools, no special access, and leaves no logs on the target's systems, yet it can uncover sensitive information that organizations didn't realize was publicly accessible.

Key risks exposed through Google Dorking:

- **Exposed admin panels** - discovered via `inurl:admin`, these are high-value targets for credential attacks
- **Sensitive file exposure** - PDFs and CSVs can contain metadata, internal paths, employee names, and data not intended for public access
- **Directory listings** - exposed folder structures allow attackers to enumerate server contents without authentication
- **Internally marked documents** - content labeled "internal use only" that is publicly indexed represents a failure in data classification and access controls
- **Infrastructure enumeration** - login and admin URLs reveal the structure of an organization's web infrastructure

Organizations should regularly run Google Dorks against their own domains, use `robots.txt` and proper access controls to prevent sensitive content from being indexed, and conduct periodic audits of publicly accessible files for unintended metadata exposure.

---

## Disclaimer
This lab was performed solely on a public domain (`microsoft.com`) for educational purposes as part of CEH training. No unauthorized systems were accessed. All information gathered is publicly available through passive reconnaissance only. Google Dorking was used strictly for educational observation and not for any malicious purpose.
