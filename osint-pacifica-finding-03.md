# Security Finding — Passive OSINT Report
## Finding #3: Critical CVE on Local Business Web Server
## Target: Axle Surgeons of California, Inc. — axlerepair.com
## Date: 2026-05-04
## Researcher: Drake Sadosky
## Method: Passive recon only — no exploitation, no unauthorized access

---

## Severity: CRITICAL

Business web and email infrastructure running OpenSSH with a known unauthenticated
remote code execution vulnerability (CVSS 8.1), MySQL end-of-life, and 18 total CVEs
on a shared hosting server that also handles business email.

---

## Target Profile

| Field | Value |
|---|---|
| Business | Axle Surgeons of California, Inc. |
| Address | 690 Roberts Rd #210, Pacifica, CA 94044 |
| Phone | (650) 738-0945 |
| Email | info@axlerepair.com |
| Website | axlerepair.com |
| Hosting | Bluehost shared hosting |
| Server IP | [REDACTED] |
| Hostname | box5424.bluehost.com |

---

## Discovery Method

1. Retrieved business listing from Pacifica Chamber of Commerce / pacificaonline.us
2. Searched business name → found axlerepair.com
3. DNS resolution: `dig axlerepair.com +short` → IP resolved
4. Cross-referenced IP against Shodan InternetDB → 18 CVEs returned
5. DNS banner grab confirmed mail server co-located on same IP
6. SPF record pulled → weak policy confirmed

**Total time: ~5 minutes. Zero active probing.**

---

## Services Identified

| Port | Service | Version | Notes |
|---|---|---|---|
| 22 | OpenSSH | 8.7 | **CVE-2024-6387 — unauthenticated RCE** |
| 21 | FTP (Pure-FTPd) | — | Unencrypted file transfer exposed |
| 25/587/465 | Exim SMTP | 4.98.2 | Business email server |
| 80/443 | Apache HTTP | — | Business website |
| 3306 | MySQL | 5.7.44 | **End-of-life Oct 2023 — permanently unpatched** |
| 5432 | PostgreSQL | — | Database exposed |
| 2082/2083 | cPanel/WHM | — | Hosting control panel |
| 110/143/993/995 | IMAP/POP3 | — | Email retrieval ports |

---

## CVE Summary

| CVE | CVSS | Description |
|---|---|---|
| CVE-2024-6387 | **8.1 HIGH** | regreSSHion — unauthenticated RCE via SSH race condition. Root access, no credentials required. OpenSSH 8.6–9.7 affected. Patched in 9.8. |
| CVE-2023-48795 | HIGH | Terrapin attack — SSH protocol downgrade, weakens encryption |
| CVE-2023-38408 | HIGH | OpenSSH ssh-agent RCE via forwarded agent |
| CVE-2023-51385 | MEDIUM | OS command injection via shell metacharacters in hostname |
| MySQL 5.7 EOL | N/A | No security patches issued after Oct 2023 — permanent exposure |
| **Total CVEs** | **18** | Full list in Shodan InternetDB record |

---

## Risk Analysis

### Primary risk — CVE-2024-6387 (regreSSHion)
An unauthenticated attacker sends crafted packets to port 22. Exploiting the signal
handler race condition in OpenSSH 8.7 results in remote code execution as root.
Root on a shared hosting server means access to every hosted site, every database,
and every email account on that physical machine — not just axlerepair.com.

### Email exposure
Business email (mail.axlerepair.com) resolves to the same server IP. A compromised
server means all inbound and outbound business email is readable. Customer inquiries,
vendor contracts, invoices — full business communication intercepted.

### SPF policy weakness
`v=spf1 a mx ptr include:bluehost.com ?all`
The `?all` qualifier is neutral — it does not instruct receiving servers to reject
spoofed email. An attacker can send email appearing to be from @axlerepair.com
without triggering SPF failures. Enables phishing of customers and vendors.

### Data exposure scope
- Customer contact records and vehicle service history
- Business invoices and payment records
- Any stored credit card or payment data (PCI-DSS implications)
- Vendor and supplier communications

---

## Theoretical Exploit Chain (NOT EXECUTED)

1. Port scan confirms SSH port 22, OpenSSH 8.7 banner
2. CVE-2024-6387 exploit triggers race condition during failed auth window
3. Root shell obtained on Bluehost shared server
4. MySQL 5.7 databases accessible — dump customer/business records
5. Email spool readable — intercept all current and incoming email
6. cPanel credentials extractable — full hosting account control
7. Persistence: SSH key added, backdoor installed
8. Lateral: other sites on shared server now accessible

---

## Disclosure

- **Responsible disclosure email sent:** 2026-05-04
- **Sent to:** info@axlerepair.com
- **Action recommended:** Contact Bluehost support, reference CVE-2024-6387
- **Exploited:** No
- **Data accessed:** No

---

## Sources

- Shodan InternetDB (internetdb.shodan.io)
- NVD CVE-2024-6387 (nvd.nist.gov/vuln/detail/CVE-2024-6387)
- Qualys Security Advisory — regreSSHion
- pacificaonline.us business directory
