# Authentication Writeups — The Ones That Matter

> Only writeups with real technique breakdowns, bypass details, and transferable skills. No empty shells.

## The Must-Reads

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 1 | Authentication Bypass Security Guide (2026) | chs.us | Complete taxonomy: brute force, default creds, SQLi in login, 2FA bypass, password reset, OAuth/SSO, JWT, session fixation — with CVEs and real-world case studies | https://chs.us/guides/authentication-bypass/ |
| 2 | Bypassing Authentication: Common Tricks Used in Bug Bounty | Bugv (Bikash Sharma) | 10 technique categories: brute force, default creds, weak session mgmt, SQLi in login, 2FA bypass, password reset, IDOR, OAuth/SSO, logic flaws, JWT — each with prevention | https://blog.bugv.io/bypassing-authentication-common-tricks-used-in-bug-bounty |
| 3 | SSO Bypass: How Attackers Circumvent Single Sign-On | Obsidian Security | 8 primary SSO bypass techniques: Golden SAML, Silver SAML, session token theft, MFA bypass via OAuth, legacy protocol exploitation, SAML signature bypass, IdP compromise, federated trust chain | https://www.obsidiansecurity.com/blog/sso-bypass-attack-techniques |
| 4 | Exploiting JWT Misconfigurations in Real-World Applications | bitsbyamg | Internal AppSec review patterns: alg:none, hardcoded/fallback secrets, algorithm confusion (RS→HS), missing claim validation, token forgery methodology, case study with `JWT_SECRET = os.getenv("JWT_SECRET", "s3cr3t")` | https://bitsbyamg.com/blog/post/2025/08/16/exploiting-jwt-in-realworld-applications |

## JWT Vulnerabilities & Bypass

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 5 | JWT Authentication Bypass: 4 Attack Techniques and How to Test | Nautillo Pro | alg:none bypass, weak secret brute-force, RS256→HS256 algorithm confusion, kid injection — with HTTP-level examples and testing checklist | https://nautillo.pro/blog/jwt-authentication-bypass |
| 6 | CVE-2026-29000: pac4j-jwt Auth Bypass (CVSS 10) | kernelzeroday / Penligent | JWE-wrapped PlainJWT bypasses signature verification, attacker needs only RSA public key, full Python token-forge CLI and library, writeup + PoC + lab | https://github.com/kernelzeroday/CVE-2026-29000 |
| 7 | CVE-2026-29000 Deep Dive: Encryption Into False Trust | Penligent | Root cause: `toSignedJWT()` returns null on PlainJWT, JWE wraps unsigned token, public key becomes sufficient to mint identity, detection difficulty (logs show "legitimate" sessions) | https://www.penligent.ai/hackinglabs/cve-2026-29000-the-pac4j-jwt-authentication-bypass-that-turns-encryption-into-a-false-sense-of-trust/ |
| 8 | Hacking JSON Web Tokens: How Attackers Exploit API Authentication | Sana Jalil | Algorithm confusion attack (RS256→HS256) step-by-step, none algorithm attack step-by-step, real-world API exploitation | https://infosecwriteups.com/hacking-json-web-tokens-how-attackers-exploit-api-authentication-19bb28010bd8 |
| 9 | JWT Authentication Bypass leads to Admin Control Panel | Hohky | Removing algorithm from JWT header, admin access via token manipulation, 208 likes, real bounty writeup | https://infosecwriteups.com/jwt-authentication-bypass-leads-to-admin-control-panel-dfa6efcdcbf5 |
| 10 | CVE-2025-64386: JWT Token Hijacking Auth Bypass | SentinelOne | JWT token hijacking during active sessions, modification of security parameters, session stealing technique | https://www.sentinelone.com/vulnerability-database/cve-2025-64386/ |
| 11 | CVE-2025-49151: JWT Auth Bypass — Forge Tokens Unauthenticated | SentinelOne | Unauthenticated JWT forging, unauthorized access without prior credentials | https://www.sentinelone.com/vulnerability-database/cve-2025-49151/ |
| 12 | CVE-2026-23993: JWT Bypass in Harbour Container Registry | chs.us | Algorithm handling in JWT validation, forged tokens for admin access, enterprise container registry exploitation | https://chs.us/guides/authentication-bypass/ |
| 13 | JWT Authentication Bypass via Weak Signing Key | Aditya Bhatt | Hashcat + Burp Suite JWT cracking, weak secret discovery, full admin access and user deletion PoC | https://github.com/AdityaBhatt3010/JWT-Authentication-Bypass-via-Weak-Signing-Key-for-Bug-Bounty |

## OAuth & SSO Attacks

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 14 | 1 Click, Zero Permission: How OAuth Mistakes Lead to ATO (Part 1) | Snyk Labs (David Bors) | 10 vulnerabilities across Langfuse/Authlib/Fastapi-SSO/Fastapi-Users, hardcoded cryptographic keys, predictable state tokens, 1-click ATO, CVE-2025-65107/CVE-2025-68158/CVE-2025-14546/CVE-2025-68481 | https://labs.snyk.io/resources/OAuth-mistake-takeover-part-one/ |
| 15 | Pre-Account Takeover via OAuth Misconfiguration | Ehtesham Ul Haq | Registering with victim's email via Google OAuth before they sign up, email ownership not verified, pre-ATO scenario | https://infosecwriteups.com/pre-account-takeover-via-oauth-misconfiguration-0e393cda1f7e |
| 16 | Pre-Account Takeover Through OAuth on Mailing Website | Harish | Two flaws: no email verification on signup + OAuth misconfiguration, full pre-ATO on mailing platform | https://infosecwriteups.com/pre-account-takeover-through-misconfigured-oauth-on-a-mailing-website-b906a5c118e9 |
| 17 | OAuth to Account Takeover — HackTricks | HackTricks | State parameter CSRF, redirect_uri open redirect, token leakage via happy paths/XSS/frames/postMessage, everlasting auth codes, AWS Cognito email overwrite, token not bound to client | https://blog.1nf1n1ty.team/hacktricks/pentesting-web/oauth-to-account-takeover |
| 18 | OAuth 2.0 Vulnerabilities: Stealing Tokens and Bypassing Auth | Payload Playground | Implicit grant token leakage, CSRF on auth endpoints, open redirect in redirect_uri, state bypass, PKCE bypass, ATO via misconfiguration | https://payloadplayground.com/blog/oauth-vulnerabilities-exploitation |
| 19 | Hidden OAuth Attack Vectors | PortSwigger Research | Advanced OAuth attack surface research, hidden vectors that bypass standard security controls | https://portswigger.net/research/hidden-oauth-attack-vectors |
| 20 | CVE-2026-40575: OAuth2 Proxy Auth Bypass via X-Forwarded | chs.us | X-Forwarded header manipulation bypasses OAuth2 proxy authentication, unauthorized access to protected apps | https://chs.us/guides/authentication-bypass/ |
| 21 | Flickr Account Takeover via AWS Cognito | HackTricks / Lauritz Holtmann | Cognito token has permissions to overwrite user data, change email address via API, full ATO | https://security.lauritz-holtmann.de/advisories/flickr-account-takeover/ |
| 22 | OAuth Misconfiguration: Abusing Other Apps' Tokens | Salt Security | Attacker creates OAuth app, victim logs in with Facebook on attacker's app, attacker reuses victim's token on target app, token not bound to client ID | https://salt.security/blog/oh-auth-abusing-oauth-to-take-over-millions-of-accounts |

## SAML & Federation Attacks

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 23 | Sign in as Anyone: Bypassing SAML SSO with Parser Differentials | GitHub Security Lab (Peter Stockli) | CVE-2025-25291/CVE-2025-25292 in ruby-saml, XML parser differential, one valid signature = forge any assertion, sign in as any user, 90-day disclosure timeline | https://github.blog/security/sign-in-as-anyone-bypassing-saml-sso-authentication-with-parser-differentials/ |
| 24 | SAML Signature Bypass via Samlify (CVE-2025-47949) | Obsidian Security | Signature wrapping attack, bypass signature validation entirely, unsigned assertions accepted as valid | https://www.obsidiansecurity.com/blog/sso-bypass-attack-techniques |
| 25 | Fortinet SSO Bypass via SAML Forgery (CVE-2025-59718/59719) | Obsidian Security / Arctic Wolf | Active exploitation Jan 2026, FortiCloud SSO bypass, generic account creation for persistence, VPN access enablement, firewall config exfiltration, all within seconds (automated) | https://www.obsidiansecurity.com/blog/sso-bypass-attack-techniques |
| 26 | Golden SAML: The Most Devastating SSO Bypass | Obsidian Security | Forged SAML tokens identical to legitimate ones, FireEye/SolarWinds supply chain, no credential theft or MFA bypass needed, concentrated attack surface | https://www.obsidiansecurity.com/blog/sso-bypass-attack-techniques |
| 27 | AuthSpark: Detecting Auth Bypass in Embedded Firmware | NDSS 2026 | Greybox fuzzer for firmware auth bypass, found 6 zero-days, 4 assigned CVEs, NTLM/Basic auth confusion in lighttpd, custom authentication module exploitation | https://www.ndss-symposium.org/wp-content/uploads/2026-f2757-paper.pdf |

## Client-Side Authentication Bypass

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 28 | Client-side Authentication Bypass: 3 Real-World Case Studies | Synack (Kuldeep Pandya) | 3 cases: full CRUD operations bypass, ACPV for inactive users, route guard reverse-engineering from JS — automated scanners almost never catch these | https://www.synack.com/exploits-explained/client-side-authentication-bypass-3-real-world-pentesting-case-studies/ |
| 29 | How Attackers Bypass 2FA with Response Tampering | Synack | 2FA bypass via response modification, tampering with authentication flow responses | https://www.synack.com/exploits-explained/how-attackers-bypass-2fa-with-response-tampering/ |

## Password Reset & Account Recovery

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 30 | Password Reset Exploits — Bugv | Bugv (Bikash Sharma) | Insecure token-based resets, email verification bypass, reset link interception, time-limited token failures | https://blog.bugv.io/bypassing-authentication-common-tricks-used-in-bug-bounty |
| 31 | Account Takeover via Broken Access Control + XSS Chaining | Jaydev Ahire | Email change without verification, broken access control on profile update, XSS chaining for impact, pre-ATO exploitation | https://infosecwriteups.com/account-takeover-an-epic-bug-bounty-story-dd5468d5773d |

## Session Management & Fixation

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 32 | Session Management Security — 92 Sources (2026 Updated) | chs.us | Complete session management reference: session fixation, session hijacking, token lifecycle, cookie security, session storage, 92 sources with 2026 CVEs | https://chs.us/guides/authentication-bypass/ |
| 33 | Session Token Theft and Replay | Obsidian Security | Session token theft as SSO bypass technique, replay attacks, token lifetime management, replay window minimization | https://www.obsidiansecurity.com/blog/sso-bypass-attack-techniques |

## CVEs & Real-World Breaches

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 34 | CVE-2026-24061: Telnetd Auth Bypass → Root | BleepingComputer / GreyNoise | Telnet IAC option negotiation injection, `USER=-f root` grants shell without auth, 18 attacker IPs, 1525 packets, 83.3% target root, SSH key persistence + Python malware deployment | https://www.bleepingcomputer.com/news/security/hackers-exploit-critical-telnetd-auth-bypass-flaw-to-get-root |
| 35 | CVE-2023-4473/4474: Zyxel NAS Authentication Bypass | BugProve | Embedded firmware auth bypass case study, routers/cameras/NAS affected, firmware analysis methodology | https://bugprove.com/knowledge-hub/cve-2023-4473-and-cve-2023-4474-authentication-bypass-and-multiple-blind-os-command-injection-vulnerabilities-in-zyxel-s-nas-326-devices |
| 36 | CVE-2025-53770: SharePoint Auth Bypass Risk | Fidelis Security | SharePoint authentication bypass, enterprise collaboration platform exploitation | https://fidelissecurity.com/vulnerabilities/cve-2025-53770 |
| 37 | Cisco IOS XE Web UI Auth Bypass (CVE-2023-20198) | NDSS/AuthSpark | Unauthenticated access to web management interface, privilege escalation to root, active in-the-wild exploitation, perimeter device as initial access | https://www.ndss-symposium.org/wp-content/uploads/2026-f2757-paper.pdf |

## Modern Attack Surfaces (2025-2026)

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 38 | Auth Bypass in AI-Generated Code | Penligent | AI coding assistants generating vulnerable auth code, hallucinated "safe" wrappers, treating AI-generated auth logic as untrusted | https://www.penligent.ai/hackinglabs/mastering-the-sql-injection-test-advanced-techniques-cve-case-studies-and-ai-defense/ |
| 39 | OAuth Consent Phishing: Bypassing MFA | Obsidian Security | OAuth consent as MFA bypass vector, delegated permissions abused, attack surface beyond traditional auth | https://www.obsidiansecurity.com/blog/sso-bypass-attack-techniques |
| 40 | Subdomain Takeover → SSO Bypass | Obsidian Security | Abandoned subdomains with CNAME to decommissioned SaaS, session/cookie leakage, federated trust chain exploitation | https://www.obsidiansecurity.com/blog/sso-bypass-attack-techniques |

## Key Numbers to Remember

| Metric | Value | Source |
|--------|-------|--------|
| CVE-2026-29000 (pac4j-jwt) CVSS | 10.0 | NVD |
| CVE-2025-65107 (Langfuse) impact | 1-click ATO | Snyk Labs |
| Snyk OAuth research: total vulns found | 10 across 4 projects | Snyk Labs |
| CVE-2026-24061 (telnetd) attacker IPs | 18 unique | GreyNoise |
| CVE-2026-24061 target user | 83.3% root | GreyNoise |
| Fortinet SSO bypass exploitation | Active, automated, seconds | Arctic Wolf |
| AuthSpark zero-days found | 6 (4 assigned CVEs) | NDSS 2026 |
| AuthSpark detection rate | 42/44 known vulns | NDSS 2026 |
| SAML parser differential impact | One valid signature = any user | GitHub Security Lab |
| Golden SAML supply chain impact | FireEye/SolarWinds 2020 | Obsidian Security |
| Salesloft-Drift OAuth breach blast radius | 700+ organizations, 10x previous | Obsidian Security |
