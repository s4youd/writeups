# XSS Writeups — The Ones That Matter

> Only writeups with real technique breakdowns, bypass details, and transferable skills. No empty shells.

## The Must-Reads

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 1 | XSS WAF Bypass Techniques That Work in 2026 — A Bug Hunter's Guide | WolfSec1337 | Tags, events, attributes, WAF bypass, CSP exploitation, DOMPurify bypass, postMessage XSS — everything that actually fires in 2026 | https://wolfsec1337.medium.com/xss-is-not-just-script-alert-1-script-heres-what-actually-makes-it-fire-b599a0cb4578 |
| 2 | XSS Filter & WAF Bypass Techniques: Real-World Payloads | Payload Playground | Case variation, encoding tricks, alternative tags/events, attribute breakouts, JS obfuscation, polyglots, DOM sinks, mutation XSS — copy-ready payloads | https://payloadplayground.com/blog/xss-filter-bypass-techniques |
| 3 | YesWeHack: XSS Attacks & Exploitation — The Ultimate Guide | YesWeHack | Reflected, stored, DOM, blind XSS exploitation workflows, authenticated reflected XSS with CSRF chaining, OOB callback for blind XSS, full attack diagrams | https://www.yeswehack.com/learn-bug-bounty/xss-attacks-exploitation-ultimate-guide |
| 4 | XSS Cheat Sheet 2026 — Payloads, WAF Bypass | OffSecKit | Context-organized (HTML body, attribute, JS string, URL), polyglot payloads, copy-paste ready for each injection context | https://offseckit.com/blog/xss-payload-cheat-sheet |
| 5 | PortSwigger XSS Cheat Sheet — 2026 Edition | PortSwigger Research | The definitive reference, updated May 2026, vectors by event/tag/browser, proof of concept for every vector | https://portswigger.net/web-security/cross-site-scripting/cheat-sheet |
| 6 | XSS Trends in 2025-2026: DOM-Based Attacks and Trusted Types | BreachVex | Why reflected XSS is dying, DOM-based is the real target, Trusted Types adoption status, JSONP/Angular CDN CSP bypasses, 3-phase testing methodology | https://breachvex.com/blog/xss-research-2026 |
| 7 | Modern XSS Filter Bypass: Techniques and Defense (2026) | ZeroThreat | 80% of obfuscated payloads bypass WAFs, encoding layering, obscure event handlers (onauxclick, onbeforetoggle, onscrollend), polyglot payloads, mXSS | https://zerothreat.ai/blog/xss-attacks-bypassing-filter |

## DOM XSS & Client-Side Exploitation

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 8 | Hunting for DOM-based XSS: A Complete Guide | Intigriti (Ayoub) | DOM sources/sinks catalog, static code analysis methodology, runtime DOM interception, innerHTML exploitation, Function() sink, DOMPurify bypass contexts | https://www.intigriti.com/researchers/blog/hacking-tools/exploiting-dom-based-xss-vulnerabilities |
| 9 | Exploiting DOM-Based XSS: A Real-World Bug Bounty Case Study | Tony Moukbel | PostMessage handler exploitation, crafting malicious payloads, PII extraction via XHR, vulnerable `window.addEventListener('message')` patterns | https://undercodetesting.com/exploiting-dom-based-xss-a-real-world-bug-bounty-case-study |
| 10 | DOM-Based XSS: Exploiting JavaScript Sinks and Source Taint Flow | Payload Playground | Sources-to-sinks cheat sheet, taint-flow analysis, DOM clobbering, postMessage and mutation XSS, DOMPurify and Trusted Types prevention | https://payloadplayground.com/blog/xss-dom-based-exploitation |
| 11 | DOM XSS: Finding and Fixing Client-Side Injection | Safeguard | Why server scanners miss DOM XSS entirely, source-to-sink tracing in client code, SPA-specific testing methodology | https://safeguard.sh/resources/blog/dom-based-xss-finding-and-fixing |
| 12 | DOM Clobbering: Breaking Sanitizers Without a Single Script | Payload Playground | id/name attribute abuse, breaking HTML sanitizers, building gadget chains to DOM XSS, real-world impact, concrete defenses | https://payloadplayground.com/blog/dom-clobbering-attacks-guide |
| 13 | Account Takeover via DOM XSS with User Interaction | Google VRP | DOM XSS leading to full ATO, zero user interaction exploitation via malicious extensions, search field injection | https://bughunters.google.com/reports/vrp/fQawWHgBt |

## DOMPurify & Sanitizer Bypass (mXSS)

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 14 | Bypassing DOMPurify Again with Mutation XSS | Gareth Heyes (PortSwigger) | The foundational mXSS research: math/mtext/table/mglyph/style tag nesting, comment injection inside attributes, Chrome vs Firefox mutation differences, CDATA abuse | https://portswigger.net/research/bypassing-dompurify-again-with-mutation-xss |
| 15 | DOMPurify 3.2.3 Bypass (Non-Default Config) | ensy.zip | Incorrectly opened comment exception (`<! ... >`), mXSS via comment injection in attribute values, SAFE_FOR_XML regex gap, CVE-2025-26791 | https://ensy.zip/posts/dompurify-323-bypass |
| 16 | Mutation XSS via Namespace Confusion — DOMPurify < 2.0.17 | Securitum (Michał Bentkowski) | SVG namespace vs HTML namespace parsing difference, `<style>` tag behavior in SVG vs HTML, DOMPurify 2.0.17 bypass via MathML mutation | https://research.securitum.com/mutation-xss-via-mathml-mutation-dompurify-2-0-17-bypass/ |
| 17 | Bypassing DOMPurify: Namespace Confusion | Seokchan Yoon | HTML `<style>` treats inner text as TEXT, SVG `<style>` treats inner text as TAG — the namespace parsing gap that enables mXSS | https://new-blog.ch4n3.kr/bypassing-dompurify-possibilities/ |
| 18 | DOMPurify XSS via `selectedcontent` Re-clone (CVE-2026-47423) | Cure53 | High-severity (8.2 CVSS) DOMPurify bypass via re-cloning selectedcontent, affects string-input path with innerHTML insertion | https://github.com/cure53/DOMPurify/security/advisories/GHSA-87xg-pxx2-7hvx |
| 19 | DOMPurify Prototype Pollution XSS (CVE-2026-41238) | SentinelOne | Prototype pollution-based XSS bypass, injecting custom elements and event handlers via DOMPurify | https://www.sentinelone.com/vulnerability-database/cve-2026-41238/ |
| 20 | DOMPurify Bypass via Nesting-based mXSS (CVE-2024-47875) | NVD/Cure53 | Nesting-based mXSS, affects DOMPurify < 2.5.0 and < 3.1.3, sanitizer bypass via specific HTML nesting patterns | https://nvd.nist.gov/vuln/detail/CVE-2024-47875 |

## CSP Bypass

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 21 | CSP Bypasses: Advanced Exploitation Guide | Intigriti (Ayoub) | Finding CSP declarations (header vs meta tag), deconstructing directives, reporting-only bypass, whitelisted domain abuse, nonce-based policy weaknesses | https://www.intigriti.com/researchers/blog/hacking-tools/content-security-policy-csp-bypasses |
| 22 | A Bug Hunter's Guide to CSP Bypasses (Part 1) | Abhishek Meena | unsafe-inline catastrophe, wildcard abuse, JSONP endpoints on whitelisted domains, CDN library exploitation, $2,500 payout for 30-second check | https://infosecwriteups.com/a-bug-hunters-guide-to-csp-bypasses-part-1-69b606fd2699 |
| 23 | HackTricks: CSP Bypass | HackTricks | PHP warning-based CSP kill via max_input_vars, error page rewriting, SOME technique, comprehensive bypass catalog | https://hacktricks.wiki/en/pentesting-web/content-security-policy-csp-bypass/index.html |
| 24 | Kill CSP via max_input_vars (Headers Already Sent) | HackTricks/terjanq | PHP warnings emitted before CSP header → header() fails → CSP stripped entirely, fill response buffer with junk to prevent CSP header from being sent | https://hacktricks.wiki/en/pentesting-web/content-security-policy-csp-bypass/index.html |
| 25 | Researchers Bypassed CSP Using HTML-Injection | CyberPress | Browser caching + bfcache interaction with CSP nonces, disk cache abuse to reuse valid nonces across requests | https://cyberpress.org/bypassed-content-security-policy-html |
| 26 | CSP Bypass (DAST) — Nuclei Templates v10.1.5 | ProjectDiscovery | 204 automated templates for CSP bypass testing, sourced from CSPBypass repository, integrated into DAST workflows | https://projectdiscovery.io/blog/csp-bypass-dast-nuclei-templates-v10-1-5 |
| 27 | Samczsun: Reflected XSS + CORS/CSP Bypass on Twitter | samczsun | Full account takeover via link click on Twitter, multi-step CSP + CORS bypass chain, 528 likes, elite-level research | https://x.com/samczsun/status/1734790962331992172 |

## Account Takeover & XSS Chaining

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 28 | How Bug Hunters Chain XSS Into Account Takeover in 2025 | OSINT Team | Step-by-step ATO chain: stored XSS in wiki comments → cookie theft → session hijack → admin access, timing and persistence details | https://osintteam.blog/how-bug-hunters-chain-xss-into-account-takeover-in-2025-525d67f07746 |
| 29 | From Stored XSS to Account Takeover | CybaVerse | Stored XSS in support ticketing → weak session security → session hijack → MFA disable → email change → password reset → full lockout | https://www.cybaverse.co.uk/resources/from-stored-xss-to-account-takeover |
| 30 | Account Takeover via Cookie-Based Stored XSS — $1,650 Bounty | staphysec | Cookie persistence enabling ATO, redirection chain, full exploitation walkthrough with bounty details | https://medium.com/@staphysec/account-takeover-via-cookie-based-stored-xss-1650-bounty-817dcc57be65 |
| 31 | Chaining Bugs: Self XSS to Account Takeover | Diconium (Behnam Yazdanpanah) | Self-XSS escalation chain, Akamai WAF bypass on `/xhr_preferences/index/change/`, HTML injection → CSRF → full ATO, PHP-based target | https://diconium.com/en/blog/cybersecurity/chaining-bugs-from-self-xss-to-account-takeover |
| 32 | Day 3: XSS Deep Dive — From Alert(1) to Account Takeovers | Aman Sharma | Reflected XSS in contact form → $5,000 ATO chain, forgotten inputs (HTTP headers, PDF generators, APIs), User-Agent XSS in admin panel ($2,500) | https://infosecwriteups.com/day-3-xss-deep-dive-from-alert-1-to-account-takeovers-cf422ec57def |
| 33 | Account Takeover: An Epic Bug Bounty Story | Jaydev Ahire | Broken access control + XSS chaining, email change without verification, pre-ATO exploitation, detailed chain methodology | https://infosecwriteups.com/account-takeover-an-epic-bug-bounty-story-dd5468d5773d |
| 34 | Stealing JWT Tokens Using XSS and CSRF | InfoSec Write-ups | XSS → CSRF → JWT theft, token extraction without httpOnly bypass, full authentication compromise | https://infosecwriteups.com/stealing-jwt-tokens-using-xss-and-csrf-fb82c96ed8cf |

## Stored XSS & Blind XSS

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 35 | Mastering Stored XSS: Real-World Exploits and Advanced Bypass | Aman Sharma | Where stored XSS hides (comments, profiles, file uploads, PDF metadata), WAF bypass techniques, advanced payloads for real targets | https://infosecwriteups.com/mastering-stored-xss-real-world-exploits-and-advanced-bypass-techniques-1ff2ce3a4e1e |
| 36 | Blind XSS in Profile Input — $2,000 Reward | hm_92366 | Blind XSS payload in profile field, admin panel execution, OOB callback confirmation, full bounty writeup | https://medium.com/@hm_92366/blind-xss-vulnerability-in-profile-input-leads-to-2-000-reward-4641defd8d54 |
| 37 | Blind XSS to Admin Panel Takeover — $6,500 Bounty | Karthikdude/Advanced-XSS | Blind XSS in "purchase description" field, markdown parser exploitation, hex encoding to bypass WAF, PII extraction (SSNs, banking details) from admin DOM | https://github.com/Karthikdude/Advanced-XSS |
| 38 | Stored XSS via Profile Name Field | Markaz Gasimov | Simple profile name XSS, minimal payload, maximum impact, real-world stored XSS discovery | https://markazgasimov.medium.com/stored-xss-via-profile-name-field-4235fd476617 |

## WAF Bypass & Filter Evasion Techniques

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 39 | Breaking WAFs: Real-World XSS Payloads That Slip Through | Kalireddipalli | Function constructor with backticks, setTimeout/setInterval template literals, Firefox-specific SVG/MathML tricks, legacy tags (xmp, marquee, object) | https://medium.com/@kalireddipalli/breaking-wafs-real-world-xss-payloads-that-slip-through-the-cracks-6454946f9b6d |
| 40 | Advanced XSS Bypass Payloads — WAF Bypass Collection | ERO-HACK/bypassXSS | Encoding-based bypasses, event handler tricks, HTML5 weird tags, JS context escapes, framework-specific payloads (Angular, React, Vue), case studies | https://github.com/ERO-HACK/bypassXSS |
| 41 | WAF Bypass — The Bug Bounty Playbook | BugBounty.info | Baseline WAF bypass methodology flowchart, identify blocking token first, systematic alternates for each token, encoding/mutation chains | https://bugbounty.info/Attack-Surface/Web/Injection/XSS/WAF-Bypass |
| 42 | XSS Bypass Filter & WAF Techniques | Undercode Testing | Case manipulation, HTML encoding, SVG event handlers, JS pseudo-protocols, Base64 eval, document.write bypass, DOM XSS via fragment, Unicode/null bytes, HPP | https://undercodetesting.com/xss-bypass-filter-waf-techniques/ |
| 43 | XSS Filter Evasion: Why Filtering Alone Isn't Enough | Acunetix/Invicti | Why WAFs fail: DOM XSS invisible to servers, mismatched context sanitization, input vs output filter gaps, application-level filtering complexity | https://www.acunetix.com/blog/articles/xss-filter-evasion-bypass-techniques/ |
| 44 | Advanced XSS Techniques (2025-2026) | Karthikdude | WebAssembly XSS, Service Worker XSS, Import Maps exploitation, CSS injection to XSS, Trusted Types bypass, prototype pollution to XSS, AI agent weaponization | https://github.com/Karthikdude/Advanced-XSS |

## Methodology & Tooling

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 45 | 0toroot: XSS Filter Bypass Techniques | 0toroot | Structured learning: encoding tricks, browser quirks, polyglot payloads, 3 hands-on challenges (Filter Gauntlet, Parentheses Prison, WAF Warrior), quiz | https://0toroot.com/learn/web-security/xss-filter-bypass |
| 46 | Intigriti: Hunting for Reflected XSS — A Complete Guide | Intigriti (Ayoub) | Reflected XSS methodology, parameter testing workflow, HTTP headers as injection vectors, PDF generator XSS, API blindspot exploitation | https://www.intigriti.com/researchers/blog/hacking-tools/hunting-for-reflected-xss-vulnerabilities |
| 47 | Intigriti: Hunting for Blind XSS — A Complete Guide | Intigriti (Ayoub) | Blind XSS methodology, OOB callback setup, admin panel targeting, support ticket injection vectors | https://www.intigriti.com/researchers/blog/hacking-tools/hunting-for-blind-xss-vulnerabilities |
| 48 | How I Chained Three Bugs to XSS — IDOR + DOM Clobbering + DOMPurify Bypass | Prateek Pulastya | Intigriti CTF solution: IDOR → DOM clobbering → DOMPurify 3.0.9 bypass, three-vulnerability chain, verified in Chrome + Playwright headless | https://prateekpulastya.medium.com/how-i-chained-three-bugs-to-xss-an-intigriti-ctf-idor-dom-clobbering-dompurify-3-0-9-bypass-25b74fc7afc7 |

## The Complete Reference

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 49 | Advanced XSS Techniques — GitHub Repository | Karthikdude | Complete 2025-2026 reference: every XSS type, DOM sources/sinks, WAF bypass per vendor (Cloudflare/AWS/Akamai), framework payloads, mXSS, CSP evasion, 10+ real-world case studies | https://github.com/Karthikdude/Advanced-XSS |
| 50 | ERO-HACK/bypassXSS — Payload Repository | ERO-HACK | Copy-ready payload files organized by category: waf-bypass.txt, dom-based.txt, unicode-encodings.txt, framework-specific/ (angular, react, vue), csp-bypass.txt | https://github.com/ERO-HACK/bypassXSS |

## Key Numbers to Remember

| Metric | Value | Source |
|--------|-------|--------|
| Sophisticated XSS payloads bypassing WAFs | ~80% | ArXiv research 2025 |
| XSS share of newly disclosed CVEs | Significant (multiple high-severity per week) | ZeroThreat 2026 |
| DOMPurify CVE-2026-47423 CVSS | 8.2 (High) | Cure53/GitHub |
| DOMPurify CVE-2025-26791 CVSS | 6.1 (Medium) | NVD |
| CSP bypass Nuclei templates available | 204 | ProjectDiscovery |
| Samczsun Twitter XSS → ATO | 528 likes, elite research | X/Twitter |
| Self-XSS to ATO chain bounty | $5,000 | Aman Sharma |
| Blind XSS in profile input bounty | $2,000 | hm_92366 |
| Blind XSS to admin panel bounty | $6,500 | Karthikdude |
| Reflected XSS in contact form bounty | $2,500 | Aman Sharma |
| unsafe-inline CSP misconfiguration payout | $2,500 (30-second check) | Abhishek Meena |
