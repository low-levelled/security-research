# Security Finding — Passive OSINT Report
## Finding #5: HIPAA-Regulated Dental Practice — 26 CVEs, OpenSSH RCE, EOL Database
## Target: [REDACTED — responsible disclosure sent 2026-05-04, 90-day embargo active]
## Sector: Healthcare / Dental Practice
## Date: 2026-05-04
## Researcher: Drake Sadosky
## Method: Passive recon only — no exploitation, no unauthorized access

---

## Severity: CRITICAL

HIPAA-regulated dental practice with patient data hosted on a shared server running
OpenSSH 7.4 with 26 documented CVEs including a remote code execution vulnerability.
MySQL 5.7 end-of-life since October 2023. No security patches received in over a year.

---

## Target Profile

| Field | Value |
|---|---|
| Sector | Healthcare — dental practice |
| Regulation | HIPAA — protected health information (PHI) in scope |
| Hosting | Bluehost shared hosting |
| Server IP | [REDACTED] |

---

## Discovery Method

1. Search for small healthcare businesses via public directory
2. DNS resolution of practice website domain
3. Shodan InternetDB cross-reference — 26 CVEs returned immediately
4. CVE cross-reference against NVD for most critical findings

**Total time: ~5 minutes. Zero active probing.**

---

## Services & Vulnerabilities

| Port | Service | Version | Status |
|---|---|---|---|
| 22 | OpenSSH | 7.4 | **CVE-2023-38408 — ssh-agent RCE, affects all versions < 9.3p2** |
| 21 | FTP Pure-FTPd | — | Unencrypted file transfer exposed |
| 3306 | MySQL | 5.7.23 | **End-of-life Oct 2023 — permanently unpatched** |
| 5432 | PostgreSQL | — | Database port exposed |
| 25/465/587 | Exim SMTP | 4.99.1 | Business email on same server |
| 2082/2083 | cPanel/WHM | — | Hosting control panel |
| **Total CVEs** | **26** | | Full list in Shodan InternetDB |

---

## Critical CVEs

| CVE | CVSS | Description |
|---|---|---|
| CVE-2023-38408 | HIGH | ssh-agent remote code execution — affects OpenSSH < 9.3p2. Exploitable if agent forwarding is used. |
| CVE-2023-48795 | HIGH | Terrapin attack — SSH protocol downgrade weakening encryption |
| CVE-2023-51385 | MEDIUM | OS command injection via hostname metacharacters |
| CVE-2018-15473 | MEDIUM | Username enumeration via timing attack |
| MySQL 5.7 EOL | N/A | No patches issued after Oct 2023 — permanent exposure |

---

## Why Healthcare Is Higher Risk

HIPAA mandates:
- Breach notification to affected patients within 60 days
- Notification to HHS Office for Civil Rights
- Fines: $100–$50,000 per violation, up to $1.9M per year per violation category
- Potential class action liability from patients

A dental practice server compromise is not just an IT incident — it triggers a federal
regulatory response. The combination of 26 unpatched CVEs on a server hosting patient
communication and data represents material HIPAA non-compliance.

---

## Passive Recon Chain

```
1. Public directory search → practice domain identified
2. dig [domain] +short → IP resolved
3. internetdb.shodan.io/[IP] → 26 CVEs, OpenSSH 7.4, MySQL 5.7 EOL
4. NVD lookup → CVE-2023-38408 confirmed RCE on OpenSSH 7.4
5. HIPAA context applied → regulatory liability identified
```

---

## Disclosure

- **Responsible disclosure email sent:** 2026-05-04
- **Sent to:** [REDACTED] — practice contact email
- **Action recommended:** Contact Bluehost, reference CVE-2023-38408, migrate to patched infrastructure
- **Embargo expires:** 2026-08-02
- **Exploited:** No
- **Unauthorized access:** No

---

## Key Lesson

Small healthcare practices are among the highest-value targets for attackers and the
most underserved by the security industry. They hold regulated patient data, face
significant breach liability, and are almost never assessed by security professionals.

A 5-minute passive recon found 26 CVEs and a named RCE vulnerability on a HIPAA-regulated
server. The practice has no visibility into this. This is the gap the market doesn't serve.

---

## Sources

- Shodan InternetDB (internetdb.shodan.io)
- NVD CVE-2023-38408 (nvd.nist.gov)
- HHS HIPAA enforcement (hhs.gov/hipaa/for-professionals/compliance-enforcement)
