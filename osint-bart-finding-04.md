# Security Finding — Passive OSINT Report
## Finding #4: Critical Infrastructure VPN — Fortinet FortiOS, Multiple CISA KEV CVEs
## Target: [REDACTED — CISA coordinated disclosure submitted 2026-05-04]
## Sector: Public Transportation / Critical Infrastructure
## Date: 2026-05-04
## Researcher: Drake Sadosky
## Method: Passive recon only — no exploitation, no unauthorized access

---

## Severity: CRITICAL — Critical Infrastructure

Fortinet FortiOS VPN endpoint identified on Bay Area public transit authority network.
Three CVSS 9.8 vulnerabilities from CISA's Known Exploited Vulnerabilities catalog apply
to the FortiOS product family. All three have mandatory remediation deadlines that have
already expired. Specific version and patch status unconfirmed — CISA disclosure submitted.

---

## Target Profile

| Field | Value |
|---|---|
| Sector | Public transportation — critical infrastructure |
| Location | Oakland, CA |
| Endpoint type | SSL VPN concentrator |
| Software | Fortinet FortiOS |
| Identification method | Shodan CPE record + SSL certificate identity verification |
| Identity confidence | HIGH — DigiCert wildcard cert issued to verified legal entity |

---

## Discovery Method

1. DNS subdomain enumeration — resolved VPN endpoint hostname to IP
2. Shodan InternetDB cross-reference — CPE record confirmed Fortinet FortiOS
3. SSL certificate inspection — confirmed legal entity identity via DigiCert cert
4. CISA KEV catalog cross-reference — three CVSS 9.8 CVEs identified

**Total time: ~20 minutes. Zero active probing. No credentials, no exploitation.**

---

## Vulnerabilities Identified

| CVE | CVSS | Description | KEV Deadline | Status |
|---|---|---|---|---|
| CVE-2024-55591 | **9.8 CRITICAL** | Unauthenticated super-admin access via WebSocket | Jan 21, 2025 | **EXPIRED** |
| CVE-2025-59718 | **9.8 CRITICAL** | SSO auth bypass via crafted SAML response | Dec 23, 2025 | **EXPIRED** |
| CVE-2026-24858 | **9.8 CRITICAL** | Multi-product auth bypass via FortiCloud SSO | Jan 30, 2026 | **EXPIRED** |

All three are on CISA's Known Exploited Vulnerabilities catalog — confirmed active
exploitation in the wild. Not theoretical. Attacker tooling exists for all three.

---

## Why This Is Critical Infrastructure Level

A successful authentication bypass on a transit authority VPN concentrator does not
give an attacker a machine — it gives them the network. For a transit operator, that
network may include:

- Employee systems and HR/payroll data
- Operational technology (OT) / SCADA systems
- Train control and signaling infrastructure
- Passenger data and fare systems
- Physical security systems

All three CVEs require no credentials. Attacker sends a crafted packet, gets admin.
Combined affected version range covers FortiOS 7.0.0 through 7.6.5 — the vast majority
of all deployed FortiOS versions.

---

## Passive Recon Chain

```
1. DNS: subdomain.target.gov → IP
2. Shodan InternetDB[IP] → CPE: cpe:/o:fortinet:fortios
3. SSL cert: openssl s_client → O=LEGAL ENTITY NAME, CN=*.target.gov
4. NVD + CISA KEV: fortios → CVE-2024-55591, CVE-2025-59718, CVE-2026-24858
5. All three: CVSS 9.8, actively exploited, KEV deadlines expired
```

This is the complete passive chain. No active probing required.

---

## What Is NOT Confirmed

- Specific FortiOS version running
- Whether applicable CVEs have been patched
- Whether compensating controls are in place
- Whether FortiCloud SSO is enabled (required for two of three CVEs)

This finding reports software identification + CVE cross-reference on critical
infrastructure. Version confirmation and patch status require privileged access
or direct coordination with the organization — hence CISA submission.

---

## Disclosure

- **Method:** CISA Coordinated Vulnerability Disclosure (cisa@cisa.dhs.gov)
- **Submitted:** 2026-05-04
- **Direct contact with target:** No — CISA handles notification to critical infrastructure operators
- **Exploited:** No
- **Unauthorized access:** No
- **Embargo:** None — CISA submission, no public embargo required

---

## Key Lesson

Three CVSS 9.8 CVEs with expired CISA KEV deadlines on a critical infrastructure
VPN endpoint. All found in 20 minutes using DNS, a public indexing database, and
a standard HTTPS connection to read a certificate.

Government and critical infrastructure organizations are not inherently more secure —
they are often less secure due to budget constraints, procurement cycles, and the
complexity of patching production systems that cannot have downtime.

The passive recon chain is identical for a home user and a transit authority.
The impact is not.

---

## Sources

- Shodan InternetDB (internetdb.shodan.io)
- CISA Known Exploited Vulnerabilities Catalog (cisa.gov/known-exploited-vulnerabilities-catalog)
- NVD CVE-2024-55591, CVE-2025-59718, CVE-2026-24858
- Fortinet PSIRT Advisories (fortiguard.com/psirt)
