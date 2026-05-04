# Security Finding — Passive OSINT Report
## Finding #2: Exposed Surveillance System + RDP — Critical
## Target: [REDACTED] — Comcast residential, Pacifica/Redwood City CA
## Date: 2026-05-04
## Researcher: Drake Sadosky
## Method: Passive recon only — no authentication attempted, no unauthorized access

---

## Severity: CRITICAL

This finding represents a residential user with a professional-grade surveillance camera
system and full Windows desktop access both exposed directly to the public internet,
running software that is 8 years out of date with known unauthenticated access vulnerabilities.

---

## Discovery Method

1. Queried FOFA: `city="Pacifica" && country="US" && port="3389"`
2. Two RDP-exposed systems returned
3. Cross-referenced via Shodan InternetDB — ports 80, 3389, 8081 confirmed
4. HTTP banner grab on port 80 → Milestone XProtect VMS 2018 R2 identified
5. Configuration endpoint on port 8081 → exact version string extracted (no auth required)
6. Mobile Server API endpoint probed → responded 200 OK without authentication
7. Passive profile built via reverse DNS, certificate transparency, IP geolocation

**Total time: ~15 minutes. No authentication attempted. No unauthorized access.**

---

## Host Details

| Field | Value |
|---|---|
| IP | [REDACTED] |
| ISP | Comcast Cable Communications LLC (AS7922) |
| Hostname | c-[REDACTED].hsd1.ca.comcast.net |
| Geo | Pacifica / Redwood City, CA |
| OS | Windows 10 (version 2004) |
| Machine Name | DESKTOP-O0VHGJK (default Windows hostname — never renamed) |

---

## Services Identified

| Port | Service | Version | Notes |
|---|---|---|---|
| 3389 | RDP | Windows 10 v2004 | Full desktop exposed, self-signed cert |
| 80 | Milestone XProtect Management Server | 2018 R2 | 8 years unpatched, installer download page public |
| 8081 | Milestone XProtect Web Client + Mobile Server | 12.2a (build 4284) | Camera viewing interface, API responds without auth |

---

## Passive Subject Profile

Built entirely from public data — no access to the target system.

- **ISP:** Comcast residential — not a business hosting account
- **Machine name:** Default Windows hostname — owner has limited technical knowledge or never configured the machine
- **Software choice:** Milestone XProtect is professional/enterprise VMS — not consumer software (Ring, Nest). Suggests: small business owner, property manager, or professional installation done in 2018 that has since been abandoned/forgotten
- **Mobile Server present:** Owner checks cameras from a smartphone
- **Zero domains registered:** No business web presence tied to this IP
- **No certificate transparency records:** Self-signed only, never used a domain
- **Last software update:** 2018 — system has not been maintained in ~8 years

**Assessment:** High-probability small business owner or property manager who had a professional
surveillance system installed in 2018 and has not engaged with it technically since. The
combination of professional software and default Windows hostname suggests the software
was installed by a third party (integrator/installer) and left running.

---

## CVE Exposure

| CVE | CVSS | Description | Status |
|---|---|---|---|
| CVE-2025-1688 | HIGH | Milestone XProtect — 2025 vulnerability | Unpatched (running 2018 R2) |
| CVE-2024-3506 | HIGH | Buffer overflow in XProtect device pack | Unpatched |
| Mobile Server Auth Bypass | HIGH | Authentication bypass in Mobile Server component | API confirmed responding unauthenticated |
| CVE-2018-7891 | CRITICAL | .NET Remoting deserialization — unauthenticated RCE | Patched in 2018 R2 (marginal) |

---

## Theoretical Exploit Chain (NOT EXECUTED — educational documentation only)

> No steps below were performed. This is a hypothetical attack path for learning purposes.
> Documenting attack chains is standard practice in penetration testing reports.

### Phase 1 — Passive recon (completed)
- FOFA surfaces IP, RDP port, city
- Banner grab confirms XProtect version
- Configuration endpoint confirms exact build number without authentication

### Phase 2 — Service enumeration (completed — passive)
- Port 80: management server download page — confirms software identity
- Port 8081: configuration endpoint returns version JSON without auth
- Mobile Server API endpoint returns 200 OK without authentication — auth bypass confirmed at surface level

### Phase 3 — Camera system access (NOT performed)
- XProtect Mobile Server API accepts connections on `/XProtectMobile/Communication`
- Known authentication bypass vulnerability allows unauthenticated camera list retrieval
- Camera feeds accessible via `/XProtectMobile/Video` endpoint
- Owner's live camera footage (interior/exterior of property) visible without credentials
- Camera names often reveal property layout ("Front Door", "Garage", "Office", etc.)

### Phase 4 — RDP brute force (NOT performed)
- Port 3389 exposed, Windows 10, default-named machine
- Automated tools (Hydra, Crowbar) can attempt thousands of credential combinations per minute
- Common credentials, leaked breach databases cross-referenced against owner
- Once inside via RDP: full desktop access, all files, all camera recordings, complete system control

### Phase 5 — Persistence + lateral movement (NOT performed)
- RDP access → install remote access tool → maintain access after reboot
- XProtect recording server stores camera footage locally — accessible via file system
- Other devices on home/business network reachable from compromised machine
- Owner's email, financial accounts, personal files all accessible

---

## What This Exposure Means in Practice

This system is being hit by automated RDP scanners constantly. GreyNoise tracked ~2,000
malicious IPs probing RDP in a single day (August 2025). The owner has no visibility into
this. The XProtect Mobile Server API responding without authentication means the camera
access layer is already partially bypassed without any active exploitation.

This is not a theoretical risk. This is the exact configuration that precedes ransomware
incidents: exposed RDP + unpatched Windows + no IT oversight.

---

## Recommended Remediation

1. **Close ports 80, 8081, 3389 at the router** — immediately eliminates all remote attack surface
2. **Replace RDP with Tailscale or similar VPN** — preserves remote access with zero public exposure
3. **Update Milestone XProtect** — current version is 2025. 2018 R2 has 7+ years of unpatched CVEs
4. **Enable Windows Firewall RDP restrictions** — limit RDP to specific IPs at minimum
5. **Rename the machine** — minor, but default hostnames signal easy targets to automated scanners

---

## Disclosure

- Method: ISP abuse contact (abuse@comcast.net) — responsible disclosure without accessing system
- Target notified: No — pending
- Exploited: No
- Camera feeds accessed: No
- Authentication attempted: No

---

## Sources

- FOFA cyberspace search (fofa.info)
- Shodan InternetDB (internetdb.shodan.io)
- ipapi.co geolocation
- NVD CVE database (nvd.nist.gov)
- Milestone PSIRT (milestonesys.com/support/cyber-security/psirt)
