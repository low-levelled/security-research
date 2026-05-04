# Security Finding — Passive OSINT Report
## Target: [REDACTED] — Unknown residential user, Pacifica CA
## Date: 2026-05-03
## Researcher: Drake Sadosky
## Method: Passive recon only — no active probing, no unauthorized access

---

## Summary

During passive OSINT research of publicly indexed infrastructure in Pacifica, CA,
a residential AT&T connection was identified running a Plex Media Server exposed
directly to the public internet with no reverse proxy, no VPN, and a self-signed
certificate. Version information is publicly visible and cross-referenceable against
known CVEs.

**No packets were sent to the target. All data sourced from FOFA passive index.**

---

## Finding Details

| Field | Value |
|---|---|
| IP | [REDACTED] |
| ISP | AT&T Enterprises LLC (AS7018) |
| Location | Pacifica, CA (FOFA geo) |
| Open Ports | 80, 443, 8080, 8443, 32400 |
| Primary Service | Plex Media Server |
| Plex Version | 1.43.1.10561 |
| Web Server | OpenResty + F5 NGINX |
| Frontend | AngularJS |
| Certificate | Self-signed (no domain) |
| Tags | self-signed |

---

## Discovery Method

1. Queried FOFA (fofa.info) for `city="Pacifica" && country="US"`
2. Four IPs returned with exposed HTTP services
3. Cross-referenced each IP against Shodan InternetDB (internetdb.shodan.io)
4. Port 32400 identified as Plex default port on [REDACTED]
5. Version string extracted from Shodan banner data
6. CVE database cross-referenced against version

**Total time: ~10 minutes. Zero active probing.**

---

## Risk Assessment

### What an attacker learns from passive recon alone

- Exact software version → direct CVE lookup
- 5 open ports → expanded attack surface beyond just Plex
- Self-signed cert → no domain, no CDN, no WAF in front
- Residential AT&T IP → home user, likely unmonitored, no SOC

### CVE Cross-Reference

| CVE | CVSS | Affected Versions | Description |
|---|---|---|---|
| CVE-2025-69415 | 7.1 HIGH | ≤ 1.42.2.10156 | Device token not invalidated on account removal — `/myplex/account` accessible with stale token |
| CVE-2025-69414 | HIGH | ≤ 1.42.2.10156 | Permanent access token retrievable via transient token via `/myplex/account` |
| CVE-2025-69416 | HIGH | ≤ 1.43.0.10389 | Incorrect resource transfer — server owner credentials exposed via `/myplex/account` |

**Current version 1.43.1.10561 appears patched for the above CVEs.**
However: version is still publicly visible, enabling attacker to search for any
unreported or newly disclosed vulns specific to 1.43.1.x.

---

## Theoretical Exploit Chain (NOT EXECUTED — educational documentation only)

> This section documents what an attacker *could* do with this information.
> No steps below were performed. This is a hypothetical attack path for learning purposes.

### Phase 1 — Passive recon (completed)
- FOFA query surfaces IP and port
- InternetDB confirms version string
- CVE cross-reference identifies relevant vulnerabilities

### Phase 2 — Active fingerprinting (NOT performed)
- Connect to `http://[REDACTED]:32400/web` — Plex web UI loads without auth by default
- Pull full server info from `http://[REDACTED]:32400/` — Plex XML response includes
  server name, machine ID, platform version
- Server name often reveals owner's name or hostname (e.g. "Drake's Plex" or "DESKTOP-XYZ")

### Phase 3 — Account enumeration (NOT performed, requires CVE ≤ 1.42.2)
- CVE-2025-69415: obtain a device token (possible via prior Plex auth flow)
- Call `/myplex/account` with stale device token
- Response contains account email, username, subscription status

### Phase 4 — Credential pivot (NOT performed)
- Email from Plex account → search in known breach databases (HaveIBeenPwned)
- If password reused → account takeover on other services
- If Plex has shared libraries → access to other users' accounts

### Phase 5 — Lateral movement (NOT performed)
- Plex server is on a home network
- Other devices on same network reachable via Plex server if compromised
- NAS, home router admin panel, other IoT devices potentially reachable

---

## Recommended Remediation

If notifying this user, the recommended fixes in priority order:

1. **Put Plex behind Tailscale** — close port 32400 entirely, access only via VPN.
   Zero public exposure, full remote access preserved.

2. **Close unused ports** — 8080 and 8443 appear unnecessary. Reduce surface.

3. **Use a real certificate** — Let's Encrypt via Caddy or nginx-proxy-manager.
   Self-signed cert = no identity verification for the owner either.

4. **Enable Plex auth token expiry** — ensure device tokens are invalidated on logout.

---

## Disclosure

- Reported via: ISP abuse contact (abuse@att.net) [optional]
- Target notified: No
- Exploited: No
- Data retained: Version number and open ports only — no PII collected

---

## What This Demonstrates

This finding was produced entirely through passive OSINT:
- No vulnerability scanners run against target
- No packets sent to target IP
- No authentication attempted
- No data exfiltrated

All information sourced from publicly indexed third-party databases (FOFA, Shodan InternetDB, NVD).
This is the standard first phase of any authorized penetration test or bug bounty engagement.
