# IDOR / Access Control Writeups — The Ones That Matter

> Only writeups with real technique breakdowns, bypass details, and transferable skills. No empty shells.

## The Must-Reads

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 1 | IDOR in the Wild: A Comprehensive Analysis of 250 Real-World Vulnerabilities | The Bug Hunter | Data-driven patterns from 250 disclosed reports, $129K+ bounties analyzed, 36.4% rated High/Critical, write vs read operation breakdown, import/export pipeline blind spots | https://www.thebughunter.blog/case-studies/idor-analysis |
| 2 | IDOR Hunting — Zero to Advanced (v.2026) | NR Shafi | Complete field guide: OWASP 2025-aligned, UUIDv4/v7 enumeration, GraphQL alias batching, AI-augmented proxy workflow, two-account methodology | https://nrshafi.github.io/idor-guide/ |
| 3 | Broken Access Control — The Complete Bug Bounty Guide (2025/2026) | NR Shafi | Every BAC class, 40 CWEs mapped, reporting playbook for max bounty, 32,654 mapped CVEs, avg bounty $1.5K-$50K+ | https://nrshafi.github.io/broken-access-control-guide/ |
| 4 | BugQuest 2026: 31 Days of Broken Access Control | Intigriti | 31-day structured course covering discovery methods, reporting impact, common config files, content discovery with ffuf, real lab exercises | https://www.intigriti.com/researchers/blog/hacking-tools/bugquest-2026-31-days-of-broken-access-control |
| 5 | Exploiting IDOR: The Hidden Door to Sensitive Data | Elcazad0r | Invoice IDOR real case, alternate parameters, encoding tricks, parameter pollution, case sensitivity bypasses | https://elcazad0r.medium.com/exploiting-idor-the-hidden-door-to-sensitive-data-c45555fdf22c |
| 6 | Testing and Bypassing Technique for IDOR | Vicky (evilox) | Array wrapping, JSON object wrapping, HTTP parameter pollution, encoding tricks, PortSwigger lab walkthrough | https://infosecwriteups.com/testing-and-bypassing-technique-for-idor-9ee03f28f4e1 |
| 7 | IDOR - Broken Authentication | RedMethod | Array/object wrapping bypass, HTTP parameter pollution, JSON parameter pollution, multi-endpoint authorization logic flaws | https://redmethod.hashnode.dev/idor-broken-authentication |

## Real-World Case Studies with Real Bounties

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 8 | IDOR in HackerOne Bugs Overview — $20,000 | GitLab Case | Private project theft via crafted tarball import, sequential ID enumeration, import/export pipeline bypass, the highest-paid single IDOR bounty on record | https://hackerone.com/reports/663431 |
| 9 | Snapchat Spotlight — $15,000 | prickn9 (Sahil Saxena) | Delete any creator's monetized content via IDOR, write/delete operations as higher severity than read | https://hackerone.com/reports/1819832 |
| 10 | PayPal Business Account — $10,500 | HackerOne | Add unauthorized users to PayPal business accounts, horizontal privilege escalation in financial system | https://hackerone.com/reports/1016122 |
| 11 | Uber 3-Bug Chain — $5,750 | HackerOne #1145428 | Chain: IDOR #1 enumerate voucher IDs + IDOR #2 modify voucher target + auth bypass apply to any account = arbitrary charges to any U4B business credit card | https://hackerone.com/reports/1145428 |
| 12 | GitLab Project Import — $20,000 | HackerOne #743953 | Steal any project's private issues, MRs, notes via crafted project.json in tarball, AttributeCleaner fix, sequential ID exploitation | https://hackerone.com/reports/743953 |
| 13 | McDonald's McHire — 64M Records Exposed | Ian Carroll & Sam Curry | Abandoned test account with password "123456" from 2019, IDOR on applicant IDs, Paradox.ai AI hiring system, 30-minute full compromise | https://ian.sh/mcdonalds |
| 14 | Broken Access Control $5,000 — Fintech Platform | WebAsha | `/api/v1/tickets/view` IDOR, alternate endpoint versions, client-side vs server-side access control, PII disclosure on banking app | https://www.webasha.com/blog/what-is-a-real-hackerone-broken-access-control-exploit-worth-5000-full-writeup-inside |
| 15 | IDOR to Account Takeover | 0xsparsh | Password reset flow IDOR, sequential user IDs in reset token generation, full account compromise chain | https://0xsparsh.medium.com/idor-to-account-takeover-ebdab0f62017 |
| 16 | Blind IDOR — Change Employee Personal Details | sengarharshit1 | Blind IDOR without response body, modifying other users' personal information, impact escalation | https://medium.com/hackcura/blind-idor-leads-to-change-personal-details-of-the-companys-employees-acc2e0701155 |
| 17 | How I Found 4 IDORs in the Same Target | Ahmed0x00 | Multiple IDORs on one target, pattern recognition, methodical endpoint coverage | https://medium.com/@Ahmed0x00/how-i-found-4-idors-in-the-same-target-73db8760a4be |
| 18 | IDOR on Reacted.com — Bug Hunter's Tale | jeetpal2007 | Real-world IDOR discovery process, 123 likes on writeup, detailed reproduction steps | https://infosecwriteups.com/uncovering-an-idor-on-reacted-com-a-curious-bug-hunters-tale-%EF%B8%8F-%EF%B8%8F-99f2bcf742af |
| 19 | IDOR in Payment Flow — Order Manipulation & PII Exposure | System Weakness | Missing authorization in payment API, access pending orders, place unauthorized orders, retrieve PII without authentication | https://systemweakness.com/how-a-simple-idor-in-a-payment-flow-led-to-order-manipulation-and-pii-exposure-ec3ed410b08d |
| 20 | Attention to Details: Finding Hidden IDORs | aseem-shrey | 1,108 likes, detailed hidden IDOR discovery methodology, attention to subtle parameter differences | https://aseem-shrey.medium.com/attention-to-details-a-curious-case-of-multiple-idors-5a4417ba8848 |
| 21 | Caught an IDOR on a Private Program — Earned a Bounty | Cybersecurity Write-ups | Method tampering for IDOR, testing POST/PUT/DELETE alongside GET, invoice/report endpoints as high-value targets | https://cybersecuritywriteups.com/caught-an-idor-vulnerability-on-a-private-program-earned-a-bounty-a99d3ac6602b |
| 22 | IDOR "My First P1 in Bugbounty" | jedus0r | First P1 finding process, realistic approach to IDOR hunting | https://infosecwriteups.com/idor-insecure-direct-object-references-my-first-p1-in-bugbounty-fb01f50e25df |

## BOLA — API Authorization Failures

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 23 | The Red Agent POV: Exploiting BOLA in an Airline GraphQL API | Wiz (Gal Nagli) | AI agent exploiting BOLA in GraphQL, bypassing backend resolvers, full airline booking database exposure in 15 minutes | https://www.wiz.io/blog/red-agent-pov-bola |
| 24 | BOLA Vulnerabilities: How to Find and Fix | Red Team Worldwide | Complete BOLA lifecycle, Burp Autorize workflow, array wrapping/JSON manipulation/HTTP parameter pollution bypass, server-side ownership validation | https://www.redteamworldwide.com/broken-object-level-authorization/ |
| 25 | How I Found a BOLA in a Real Production App | 0x7Nulll | Cross-account cart manipulation using client-controlled cart identifier, real production BOLA discovery | https://medium.com/@0x7Nulll/how-i-found-a-broken-object-level-authorization-bola-in-a-real-production-app-808a12aac664 |
| 26 | API Pentesting: Broken Object Level Authorization (BOLA) | hackerdevil | BOLA basics for APIs, bouncer analogy, authentication vs authorization distinction, practical testing methodology | https://infosecwriteups.com/api-pentesting-broken-object-level-authorization-bola-2227be0069dc |
| 27 | API Attack Awareness: BOLA — Why It Tops OWASP API Top 10 | Wallarm (Tim Erlin) | 95% of API attacks from authenticated sources, Wallarm Q2 2025 ThreatStats, Sapphos dating app shutdown after BOLA, stateful vulnerability detection challenges | https://securityboulevard.com/2025/10/api-attack-awareness-broken-object-level-authorization-bola-why-it-tops-the-owasp-api-top-10/ |
| 28 | How to Fix Broken Object Level Authorization in Laravel (CVE-2026-34456) | ApiPosture | OAuth flow BOLA, email-based account linking bypass, complete account takeover via social login | https://www.apiposture.com/remediation/laravel/how-to-fix-broken-object-level-authorization-in-laravel-cve-2026-34456 |

## Privilege Escalation & Vertical BAC

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 29 | Privilege Escalation: How Broken Access Control Led to Full Account Takeover | Abhishek Gupta | Broken access control → full ATO chain, practical privilege escalation methodology | https://infosecwriteups.com/privilege-escalation-how-broken-access-control-led-to-full-account-takeover-c7b42bb9f47b |
| 30 | From Zero to Hero: Critical Privilege Escalation Flaw — $500 | Krish_cyber | RBAC testing methodology, SaaS platform privilege escalation, step-by-step bounty discovery | https://cybersecuritywriteups.com/from-zero-to-hero-how-i-uncovered-a-critical-privilege-escalation-flaw-and-earned-500-c6be96484090 |
| 31 | $800 Bounty: Privilege Escalation via API — Scheduler to Team Admin | InfoSec Write-ups | Low-privileged Scheduler role → editor-level admin via direct API calls, Broken Access Control on SaaS platform | https://infosecwriteups.com/800-bounty-privilege-escalation-via-api-from-scheduler-to-team-admin-810bb8401a0f |
| 32 | Access Control Bug Bounty Reports | R-Rad Hasan | Exploiting PATCH method for privilege escalation, 8+ hours of JavaScript analysis, real report examples | https://rradhasan.medium.com/access-control-bug-bounty-reports-b12ded6aece7 |
| 33 | Exploiting Access Control Misconfiguration: Privilege Escalation via Improper PATCH Method | R-Rad Hasan | PATCH method bypass for access control, JavaScript analysis for endpoint discovery | https://rradhasan.medium.com/access-control-bug-bounty-reports-b12ded6aece7 |

## Bypass Techniques & Defense Evasion

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 34 | Bug Bounty Bootcamp #34: IDOR Beyond GET | Aman Sharma | Modifying, deleting, method switching for maximum impact, POST/PUT/DELETE IDOR, turning read access into full account takeover | https://infosecwriteups.com/bug-bounty-bootcamp-34-idor-beyond-get-modifying-deleting-and-method-switching-for-maximum-159554377462 |
| 35 | Exploiting IDOR: The Hidden Door — Advanced Bypass | Elcazad0r | Alternate parameters (id vs email vs username), case sensitivity tricks, URL encoding, double encoding, null byte injection | https://elcazad0r.medium.com/exploiting-idor-the-hidden-door-to-sensitive-data-c45555fdf22c |
| 36 | IDOR Checklist 2025 | mohaned0101 | Structured checklist for systematic IDOR testing, parameter enumeration, authorization check verification | https://medium.com/@mohaned0101/idor-checklist-2025-443575a389d4 |
| 37 | EXPOSED! The Authorization Blind Spot | YuvaSec | Missing line of code leading to catastrophic consequences, developer mindset vs attacker mindset, real-world case studies | https://dev.to/yuvasec/exposed-the-authorization-blind-spot-that-makes-idor-attacks-unstoppable-4o88 |

## Methodology & Tooling

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 38 | IDOR Account Takeover Case Studies — Prompt Hunting | LifeJiggy | Pattern recognition: 45% sequential integers, 60% missing ownership checks, 25% client-supplied user context, hunting methodology with Burp Autorize/Param Miner/ffuf | https://github.com/LifeJiggy/Prompt-Hunting/blob/main/Real-World-Case-Studies/01-IDOR-Account-Takeover-Case-Studies.md |
| 39 | Bug Bounty Roadmap 2025 — IDOR Section | GitHub | Structured learning path for IDOR, from basics to advanced exploitation | https://github.com/l3tchupkt/Bug-Bounty-Roadmap-2025/tree/main/vulnerabilities/idor |
| 40 | Broken Access Control: A Deep Dive | Mohamed Salem (0xd3vourer) | PortSwigger lab walkthroughs, robots.txt admin panel discovery, IDOR privilege escalation, UUID exposure, response sanitization, comprehensive 14-page PDF | https://0xd3vourer.github.io/assets/Write-Ups/Broken_Access_Control.pdf |
| 41 | Mastering Access Control Vulnerabilities and Privilege Escalation | Securityium | Vertical vs horizontal vs context-dependent controls, URL manipulation, HTTP method tricks, header spoofing, deny-by-default best practices | https://www.securityium.com/mastering-access-control-vulnerabilities-and-privilege-escalation/ |
| 42 | IDOR — The Bug Bounty Goldmine Hidden in Plain Sight | Priyansh | Hunter's perspective on IDOR, realistic exploitation flow, parameter identification methodology, from intercept to impact | https://secpriyansh.medium.com/idor-the-bug-bounty-goldmine-hidden-in-plain-sight-1c80eb9d546b |

## Real Disclosed HackerOne Reports with Technique

| # | Title | What You Learn | URL |
|---|-------|----------------|-----|
| 43 | IDOR in HackerOne Bugs Overview — Private Program Data Access | Access to private program data via IDOR on HackerOne itself | https://hackerone.com/reports/663431 |
| 44 | IDOR Exposing Order Information on Affirm | Financial data exposure via order ID manipulation | https://hackerone.com/reports/1323406 |
| 45 | IDOR to View User Order History on Bohemia Interactive | Gaming platform order history access | https://hackerone.com/reports/287789 |
| 46 | Broken Access Control — Insecure Function Level Access Control | Function-level access control bypass, admin functionality exposure | https://hackerone.com/reports/895172 |

## Key Numbers to Remember

| Metric | Value | Source |
|--------|-------|--------|
| IDOR reports on HackerOne (2017-2025) | 250 disclosed | thebughunter.blog |
| Total disclosed bounties | $129,408+ | thebughunter.blog |
| Highest single IDOR bounty | $20,000 | GitLab Project Import |
| IDORs rated High or Critical | 36.4% | thebughunter.blog |
| Peak IDOR reports in one year | 50 (2022) | thebughunter.blog |
| BOLA rank in OWASP API Top 10 | API1:2023 | OWASP |
| BAC rank in OWASP Top 10 | A01:2025 | OWASP |
| Mapped CVEs for BAC | 32,654 | nrshafi.github.io |
| Avg BAC bounty range | $1,500 - $50,000+ | nrshafi.github.io |
| % of API attacks from authenticated sources | 95% | Salt Security / Wallarm |
