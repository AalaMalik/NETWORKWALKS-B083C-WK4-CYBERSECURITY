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

  ## 4. Findings and Proof of Exploitation

### Finding 1: SQL Injection in Patient Portal Authentication
* **Location:** `/patient/login.php`
* **Severity:** Critical
* **Description:** User-supplied input in the username field was improperly concatenated directly into backend database queries without parameterization, returning verbose MySQL syntax errors and permitting authentication bypass via payload manipulation (e.g., `admin' -- -`).
* **Evidence:** 
  
  ![Screenshot: Patient Portal MySQL Error / Successful Bypass Dashboard](docs/images/placeholder_sqli.png)

### Finding 2: Insufficient Password Protection of Patient Reports
* **Location:** Downloaded patient report files (`patient_report_*.pdf`)
* **Severity:** High
* **Description:** Patient pathology files were protected using weak document-level passwords susceptible to dictionary and brute-force attacks once extracted using tools like `pdf2john` and cracked using expanded wordlists.
* **Evidence:** 
  
  ![Screenshot: PDF Hash Extraction and Password Recovery Results](docs/images/placeholder_pdf_crack.png)

### Finding 3: Sensitive Metadata Exposure in PDF Files
* **Location:** `patient_report_3.pdf`
* **Severity:** Medium
* **Description:** Metadata analysis using `exiftool` revealed internal author accounts (`j.malik`) and developer comments pointing to backup migration paths (`DB backup moved to /old before site migration`).
* **Evidence:** 
  
  ![Screenshot: Exiftool Metadata Output Displaying IT Notes](docs/images/placeholder_exiftool.png)

### Finding 4: Publicly Accessible Database Backup & Directory Listing
* **Location:** `/old/` directory (`/old/mediroza_db_backup_2019.sql`)
* **Severity:** Critical
* **Description:** The legacy directory was explicitly disclosed via `robots.txt` and permitted public directory browsing, exposing an unauthenticated raw SQL database dump containing confidential staff salaries and shareholder records.
* **Evidence:** 
  
  ![Screenshot: Directory Listing of /old/ and SQL Backup File](docs/images/placeholder_dir_listing.png)

---

## 5. Risk Rating Summary

| Finding ID | Vulnerability Description | Location | Risk Level | Exploited |
| :--- | :--- | :--- | :--- | :--- |
| **F-01** | SQL Injection Authentication Bypass | `/patient/login.php` | Critical | Yes |
| **F-02** | Weak PDF Document Passwords | Patient Reports | High | Yes |
| **F-03** | Sensitive PDF Metadata Disclosure | `patient_report_3.pdf` | Medium | Yes |
| **F-04** | Public Database Backup Exposure | `/old/` | Critical | Yes |

---

## 6. Recommendations and Remediation

* **SQL Injection:** Rewrite authentication logic to use parameterized queries and prepared statements rather than raw string concatenation. Suppress verbose database error messages from returning to client interfaces.
* **Document Security:** Ensure sensitive medical records are stored behind robust portal access controls rather than relying solely on weak, easily crackable client-side PDF document passwords.
* **Metadata Sanitization:** Strip all identifying internal metadata (authors, comments, operational notes) from released documents prior to distribution using tools like `exiftool -all=`.
* **Directory & Backup Hygiene:** Immediately remove backup archives from web-accessible directories, enforce strict file-system permissions, and disable directory indexing across all web servers.
