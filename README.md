# Penetration Testing Report: Mediroza General Hospital

**Target:** `https://medirozahospital.com`  
**Client:** Mediroza General Hospital  
**Engagement Type:** Black-Box Penetration Test  
**Batch:** B082 | Week 4  

---

## 1. Executive Summary
A black-box penetration test was conducted against the simulated Mediroza General Hospital web environment to assess externally accessible infrastructure and determine whether an unauthorized external user could compromise sensitive assets. The engagement was structured around four core milestones, focusing on reconnaissance, authentication security, file encryption protection, and sensitive information exposure.

Testing identified multiple critical vulnerabilities, most notably an unauthenticated SQL injection vulnerability within the patient portal authentication mechanism. This allowed attackers to bypass authentication controls entirely, retrieve confidential patient PDF reports, perform offline password recovery, and ultimately access a publicly exposed database backup containing sensitive employee salary and shareholder data. Immediate remediation is strongly advised to secure authentication gates and purge legacy web assets.

---

## 2. Scope and Methodology
* **Target:** `https://medirozahospital.com` (IP: `199.188.201.16`)
* **Approach:** Black-box assessment simulating an external attacker with no prior knowledge of internal application architecture or source code.
* **Tools Utilized:** `nslookup`, `WhatWeb`, `WAFWOOF`, `Nmap`, `pdf2john`, `Hashcat`, `qpdf`, `exiftool`, web browser developer tools, and manual HTTP interception.
* **Limitations:** Rules of engagement explicitly prohibited Denial-of-Service (DoS) attacks and social engineering.

---

## 3. Reconnaissance & Discovery
* **Domain Resolution & Port Scanning:** Initial enumeration via `Nmap` identified open services including FTP (port 21), SSH, HTTP (port 80), HTTPS (port 443), and various mail daemons.
* **Robot File Analysis:** Inspecting `https://medirozahospital.com/robots.txt` revealed sensitive unindexed directories intended to be hidden from search engines:
  ```text
  Disallow: /patient/
  Disallow: /staff/
  Disallow: /old/
