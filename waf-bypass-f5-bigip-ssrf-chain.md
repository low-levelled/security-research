# F5 ASM WAF Bypass + Backend Infrastructure Enumeration via BIG-IP Persistence Cookie

## Overview

During authorized bug bounty research, I identified multiple chained vulnerabilities on a major gaming platform's infrastructure, escalating from an F5 BIG-IP persistence cookie information disclosure to a demonstrated WAF bypass, backend server pinning, full infrastructure enumeration, and unauthenticated VPN configuration disclosure.

**Reported via:** HackerOne  
**Status:** Submitted  
**Date:** 2026-05-31  

---

## Finding 1: F5 ASM WAF Bypass via IP Encoding

### Description
The F5 ASM WAF blocks plain RFC 1918 IP addresses in a server-side proxy endpoint's URL parameter. However, the WAF fails to normalize alternative IP encodings (hexadecimal, octal, mixed), allowing attacker-controlled internal IP targets to bypass the WAF and reach the application layer.

### Proof of Bypass

The WAF returns `error.htm?support_id=...` (F5 ASM violation page) when it blocks a request. Bypassed requests return `/Error.htm` (application error page with no support_id). This behavioral difference proves the request passed through the WAF unfiltered.

| Input Format | Response | Result |
|-------|----------|--------|
| Plain decimal (`10.x.x.x`) | `error.htm?support_id=...` | WAF blocked |
| Loopback (`127.0.0.1`) | `error.htm?support_id=...` | WAF blocked |
| Link-local (`169.254.169.254`) | `error.htm?support_id=...` | WAF blocked |
| IPv6-mapped (`[::ffff:10.x.x.x]`) | `error.htm?support_id=...` | WAF blocked |
| Hex dotted (`0x0a.0x5a.0x2a.0xNN`) | `/Error.htm` | **WAF bypassed** |
| Hex integer (`0x0a5a2aNN`) | `/Error.htm` | **WAF bypassed** |
| Octal (`012.0132.052.0NNN`) | `/Error.htm` | **WAF bypassed** |
| Mixed (`10.0x5a.42.NN`) | `/Error.htm` | **WAF bypassed** |

### Additional Finding
The WAF blocks hex-encoded localhost (`0x7f0000NN`) but misses hex-encoded RFC 1918 addresses (`0x0a5a2aNN`). This demonstrates inconsistent normalization — the WAF was updated to catch some hex patterns but not all.

### Impact
The F5 ASM SSRF protection rule is ineffective against standard IP encoding techniques. An attacker who chains this with any server-side URL-fetching functionality can target internal backend servers while evading WAF detection and logging.

---

## Finding 2: Backend Server Pinning via Persistence Cookie Manipulation

### Description
The F5 BIG-IP load balancer sets unencrypted persistence cookies that encode the backend server IP address in hexadecimal. An attacker can craft these cookies to deterministically route requests to any specific backend server, bypassing load balancer distribution.

### Technique

The persistence cookie format encodes IPv4 addresses as hex:
```
cookie=rd101o00000000000000000000ffff[HEX_IP]o[PORT]
```

By modifying the hex IP portion, requests can be pinned to arbitrary backend servers. Different servers respond with distinct application fingerprints (verified via response hash), confirming that routing is controlled by the attacker.

### Proof
Four different backend servers were confirmed via distinct response hashes when pinning to different IPs within the backend subnet.

### Impact
An attacker can target individual backend servers, bypass per-server rate limiting, and perform targeted attacks against potentially vulnerable instances.

---

## Finding 3: Full Backend Pool Enumeration

### Description
By scanning a /24 subnet range via crafted persistence cookies, the complete backend server pool was enumerated. All 254 addresses in the subnet were tested, revealing 10 unique application server instances identifiable by distinct response hashes.

### Impact
Complete internal network topology of the backend pool is exposed. An attacker knows the exact number of servers and can individually target each one.

---

## Finding 4: Unauthenticated VPN Configuration Disclosure

### Description
A Palo Alto GlobalProtect VPN endpoint exposes its prelogin configuration without authentication, leaking the VPN gateway's real IP address, SSO provider details, application identifiers, and SAML configuration.

### Information Disclosed

| Field | Type |
|-------|------|
| VPN Server IP | Real gateway IP (bypasses DNS protections) |
| VPN Type | Palo Alto GlobalProtect |
| SSO Provider | Okta |
| SSO App ID | Okta application identifier |
| SAML ACS URL | Full SAML assertion consumer service URL |
| SAML Issuer | Service provider issuer |
| Organization | Parent company name |
| SSL Cert | Wildcard certificate with validity dates |
| Auth Method | SAML POST binding |

### Impact
This disclosure provides an attacker with the exact VPN gateway IP, SSO provider and application ID for targeted phishing, SAML endpoint details for potential replay/injection attacks, and organizational identity for social engineering.

---

## Finding 5: Outdated Server Software

### Description
An origin server (leaked via response header) runs Apache 2.4.37 (released Oct 2018) with OpenSSL 1.1.1k (released Mar 2021). Both have known CVEs:

- **Apache 2.4.37:** CVE-2019-0211 (privilege escalation), CVE-2019-0215 (TLS access control bypass), CVE-2019-0217 (auth bypass)
- **OpenSSL 1.1.1k:** CVE-2021-3449 (DoS), CVE-2021-3450 (CA check bypass). The 1.1.1 branch is EOL.

---

## Attack Chain Summary

```
Unencrypted persistence cookie
  → Decode backend IPs (information disclosure)
  → Craft cookies to pin to specific servers (server pinning)
  → Scan subnet to map all backends (infrastructure enumeration)
  → Use WAF bypass to target internal IPs via proxy (WAF evasion)
  → Combined with any SSRF-capable endpoint = internal network access
```

This target previously had three separate SSRF vulnerabilities in an emblem editor (since removed), each paying $1,500 via HackerOne. The WAF bypass demonstrated here would have defeated the SSRF IP filtering on those endpoints and remains exploitable against any future server-side URL-fetching functionality.

---

## Remediation Recommendations

1. **Encrypt F5 BIG-IP persistence cookies** to prevent IP extraction and cookie crafting
2. **Update F5 ASM WAF rules** to normalize hex/octal/integer IP encodings before applying RFC 1918 blocklists
3. **Validate persistence cookies server-side** — reject cookies pointing to IPs outside the actual pool
4. **Disable unauthenticated VPN prelogin endpoint** or redact sensitive fields
5. **Update Apache and OpenSSL** to current stable versions

---

## Methodology

All research was conducted passively using standard tools (curl, browser DevTools) within the scope of the target's public bug bounty program. No active exploitation, destructive testing, or unauthorized access was performed.

**Tools:** curl, Python 3, browser DevTools  
**Duration:** ~4 hours  
**Approach:** Cookie analysis → WAF differential testing → infrastructure enumeration → adjacent system discovery
