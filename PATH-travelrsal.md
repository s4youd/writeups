# Path Traversal Writeups — The Ones That Matter

> Path traversal is the bug that keeps paying. GitLab alone has paid $60K+ across multiple reports. The key is encoding, encoding, encoding — and knowing what files to read once you're in.

## The Must-Reads

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 1 | Directory Traversal & Path Traversal — Complete Exploitation Guide | SHADOW120 | Full bypass taxonomy, target file lists for Linux/Windows/PHP/Node/Java/Python, ffuf automation, escalation paths, report template | https://hacklido.com/blog/1479-directory-traversal-path-traversal-complete-exploitation-guide |
| 2 | Path Traversal Beyond ../ — Encoding Tricks That Bypass Input Filters | Roi Abitboul (Raven.io) | "The gap between filter processing and filesystem processing is where every bypass lives" — URL encoding, double encoding, Unicode normalization, overlong UTF-8 | https://ravenio.org/blog/path-traversal-beyond-dotdot |
| 3 | YesWeHack — Beyond '../': Practical Guide to Path Traversal | YesWeHack | File resolution mechanics, JSON API path parameters, chaining with SSRF, high-value target file identification | https://www.yeswehack.com/learn-bug-bounty/practical-guide-path-traversal-attacks |
| 4 | Complete LFI Techniques Cheat Sheet | Shaker-maker | Dot-dot variations, encoding matrix, PHP wrappers, null byte, path truncation, mixed slashes, extension bypass — all in one reference | https://github.com/Shaker-maker/ctf-writeups/blob/main/cheatsheets/lfi-techniques.md |
| 5 | PayloadsAllTheThings — Directory Traversal | swisskyrepo | Complete encoding reference table, `dotdotpwn` fuzzing tool, all encoding variants in one place | https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Directory%20Traversal/README.md |

## Real-World Bounty Writeups

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 6 | GitLab Path Traversal to RCE — $12,000 Bounty | saltyyolk | URL-encoded traversal in Maven package registry, `grape` router URL decoding behavior, `curl --path-as-is`, wrote to `~/.ssh/authorized_keys`, sanitization BEFORE URL decoding = bypass | https://hackerone.com/reports/733072 |
| 7 | GitLab Arbitrary File Read via UploadsRewriter — $20,000 | HackerOne #827052 | Path traversal during issue move/transfer via UploadsRewriter component — traversal in file move logic is a less-tested surface | https://hackerone.com/reports/827052 |
| 8 | GitLab Bulk Imports UploadsPipeline — $29,000 | HackerOne #1439593 | Arbitrary file read via bulk import pipeline — CI/CD and import/export features are high-value attack surfaces | HackerOne Report #1439593 |
| 9 | Mozilla VPN Path Traversal to RCE — $6,000 | HackerOne #2995025 | Path traversal in desktop/VPN clients — not limited to web apps | HackerOne Report #2995025 |
| 10 | $500-$10K Path Traversal Advanced Methodology | It4chis3c | `curl --path-as-is` for API testing, file upload/download traversal, package registry endpoints, ZIP extraction, document sharing, image uploaders | https://infosecwriteups.com/500-10k-worth-path-traversal-advanced-methodology-dd80c18c5539 |
| 11 | 145 HackerOne Disclosed Path Traversal Reports | ajaysenr | Sorted by bounty: $29K GitLab, $20K GitLab, $16K GitLab, $12K GitLab, $6K Mozilla VPN, $5K PortSwigger, $4K Apache HTTP Server | https://github.com/ajaysenr/HackerOne-Disclosed-Reports/blob/main/by-weakness/path-traversal.md |

## Filter Bypass Techniques

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 12 | Path Traversal Filter Bypass Techniques | StackExchange | 16-bit Unicode (`%u002e`), double URL encoding (`%252e`), overlong UTF-8 (`%c0%2e`), null byte at end of filename, start-of-path variations | https://security.stackexchange.com/questions/96736/path-traversal-filter-bypass-techniques |
| 13 | Directory Traversal: Exploiting and Bypassing Protections | 1337rce | URL encoding, double encoding, UTF-8, combined encodings, null byte injection, LFI to RCE chaining | https://medium.com/@1337rce/directory-traversal-attacks-exploiting-and-bypassing-protections-741da2ccdbea |
| 14 | Athena OS Codex — LFI Filter Bypasses | Athena OS | Non-recursive strip bypass (`....//`), partial encoding (`%2e%2e/`), combined bypasses (`./languages/....//....//....//....//etc%2fpasswd`) | https://codex.athenaos.org/codex/web/lfi/filter-bypasses |
| 15 | PortSwigger Path Traversal All Labs Walkthrough | dollarboysushil | All 6 labs: simple, absolute path, non-recursive strip, double URL encoding, start-of-path validation, null byte extension | https://infosecwriteups.com/portswigger-path-traversal-all-labs-walkthrough-bug-bounty-prep-by-dollarboysushil-85ab64d6106a |

## LFI to RCE Techniques

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 16 | From LFI to RCE: Practical Escalation Techniques | Yunolay | 5 paths: log poisoning (Apache/Nginx/SSH/mail), `data://` wrapper, PHP filter chain (Synacktiv technique — byte-by-byte payload), PHP session files, `/proc/self/environ` and `/proc/self/fd` | https://yunolay.com/lfi-to-rce/ |
| 17 | RCE via Apache Log Poisoning — Step-by-Step | Sailor Security | LFI on one port, Apache logs on another port — poison logs on one service, include via LFI on different service. Reverse shell spawning | https://sailorsecurity.ca/2024/03/23/rce-via-apache-log-poisoning |
| 18 | The Hacker Recipes — LFI to RCE Log Poisoning | The Hacker Recipes | SSH auth log poisoning (`ssh '<?php phpinfo(); ?>'@target`), Apache/Nginx access log poisoning, SMTP/mail log poisoning via telnet RCPT TO, Windows log paths | https://www.thehacker.recipes/web/inputs/file-inclusion/lfi-to-rce/logs-poisoning |
| 19 | LFI to RCE via Apache Log Poisoning — PoC Repository | Th3G0df4th3xr | Working exploit script, vulnerable PHP endpoint, Apache config — full working PoC | https://github.com/Th3G0df4th3xr/LFI-LOG-Poisoning |
| 20 | Complete LFI/RFI Methodology | aw-junaid | Phase 1: 16+ parameter names to test. Phase 2: Burp Intruder. Phase 3: Bypass appended extensions. Phase 4: PHP wrappers, log poisoning, truncation. CVE-2020-16152 Aerohive: unauthenticated root RCE | https://github.com/aw-junaid/bug-bounty/blob/main/methodologies/web%20penetration/LFI%20and%20RFI.md |

## Client-Side Path Traversal (CSPT)

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 21 | Client-Side Path Traversal: From XSS to RCE | NullSecurityX | CSPT in React/Next.js/Vue via router manipulation, backslash traversal, CSPT2CSRF chaining, framework-specific (Next.js dynamic routes trust params) | https://nullsecurityx.codes/Client-side-path-traversal-for-bug-bounty-hunters |
| 22 | CSPT2CSRF — Client-Side Path Traversal to CSRF | Doyensec (Maxence Schmitt) | CSPT reroutes API requests through front-end routing, GET→POST chaining, found in Mattermost/Rocket.Chat, Burp extension | https://blog.doyensec.com/2024/07/02/cspt2csrf.html |

## WAF Bypass & Advanced Techniques

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 23 | NASA Path Traversal and XSS — WAF Bypass | incogbyte | Path traversal + XSS on NASA subdomains with CloudFlare WAF bypass, AWS infrastructure exposure | https://incogbyte.github.io/posts/2024-07-10-nasa-pathtraversal-xss-tricks |
| 24 | WAFFLED — Exploiting Parsing Discrepancies to Bypass WAFs | Academic (2025) | Multipart boundary mutations, JSON/XML structural manipulation, parameter type confusion between WAF and backend, Content-Type confusion | https://arxiv.org/html/2503.10846v1 |
| 25 | ModSecurity Path Normalization Bypass (CVE-2024-1019) | Security Research | ModSecurity v3 decoded URL paths before separating query string, percent-encoded payloads in path hidden from rules while backend interprets correctly | Referenced in WAF evasion research |

## Comprehensive Guides & Methodology

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 26 | File Inclusion Vulnerabilities for Bug Bounty | Herish | Path traversal basics, PHP wrappers deep dive, log poisoning, parameter discovery methodology (page, file, include, template, path, lang, view) | https://herish.me/blog/file-inclusion-vulnerabilities-bug-bounty/ |
| 27 | Ehxb — Path Traversal: From Discovery to Automation | Ehxb | 5 PortSwigger lab scenarios with Python automation code | https://infosecwriteups.com/ehxb-path-traversal-vulnerabilities-from-discovery-to-automation-569b64ce46ac |
| 28 | The Bug Bounty Playbook — Path Traversal | BugBounty.info | Testing checklist: parameter discovery → basic traversal → encoding → null byte → escalation to SSH keys/config/DB creds | https://bugbounty.info/Attack-Surface/Web/File-Operations/Path-Traversal |
| 29 | IntelligenceX — LFI Complete Guide | IntelligenceX | Any service that logs user-controlled input is a poisoning candidate — SSH, FTP, SMTP, web logs. Truncation techniques for extension bypass | https://blog.intelligencex.org/local-file-inclusion-lfi-vulnerabilities-complete-guide |
| 30 | Aikido Security — Path Traversal in 2024: Year Unpacked | Aikido | Path traversal trends in open source — container orchestration (Kubernetes secrets, Docker configs) is a growing attack surface | https://www.aikido.dev/blog/path-traversal-in-2024-the-year-unpacked |

## Key Numbers to Remember

| Metric | Value | Source |
|--------|-------|--------|
| GitLab total path traversal bounties | $60K+ across 4 reports | HackerOne |
| GitLab bulk import traversal | $29,000 | HackerOne #1439593 |
| GitLab UploadsRewriter traversal | $20,000 | HackerOne #827052 |
| GitLab Maven registry traversal to RCE | $12,000 | HackerOne #733072 |
| Mozilla VPN traversal to RCE | $6,000 | HackerOne #2995025 |
| Meta CSPT chain to RCE | $111,750 | NullSecurityX |
| PortSwigger traversal labs | $5,000 | HackerOne |
| Apache HTTP Server traversal | $4,000 | HackerOne |
| Non-recursive strip bypass | `....//` → `../` | Athena OS |
| PHP filter chain technique | Byte-by-byte, no file write needed | Synacktiv 2022 |

## Bypass Technique Matrix

| Filter Defense | Bypass Technique | Payload |
|---|---|---|
| Blocks `../` | Non-recursive strip | `....//....//....//etc/passwd` |
| Blocks `../` | URL encoding | `%2e%2e%2f%2e%2e%2fetc/passwd` |
| Blocks `../` | Double URL encoding | `%252e%252e%252f%252e%252e%252fetc/passwd` |
| Blocks `../` | Unicode/UTF-8 | `%c0%ae%c0%ae%c0%afetc%c0%afpasswd` |
| Appends `.php` extension | Null byte (legacy) | `../../etc/passwd%00` |
| Appends `.php` extension | Path truncation | `../../etc/passwd/./././` × 4096 |
| Blocks traversal entirely | Absolute path | `/etc/passwd` |
| Requires base dir prefix | Prefix + traversal | `/var/www/uploads/../../../etc/passwd` |
| Blocks all encoding | PHP wrappers | `php://filter/convert.base64-encode/resource=...` |
| `allow_url_include=Off` | Log poisoning | Poison User-Agent → include access log |
| No file write + no logs | PHP filter chain | `php://filter/convert.iconv.UTF8.CSISO2022KR\|...` |
| Standard traversal blocked | Double extension | `shell.php.jpg` |
| All web filters block | WAF bypass via JSON/XML | Embed traversal in structured data |
| Standard filters | Client-side traversal | Backslash `\` in SPA router params |

## Target Files to Read After Exploitation

| File | What You Get |
|------|-------------|
| `/etc/passwd` | User list, home directories, shell info |
| `/etc/shadow` | Password hashes (if readable) |
| `/etc/hostname` | Server hostname |
| `/proc/self/environ` | Environment variables, often with API keys |
| `/proc/self/cmdline` | Current process command line |
| `/proc/self/fd/0..30` | File descriptors — iterate for open logs |
| `/var/log/apache2/access.log` | Apache logs (for poisoning) |
| `/var/log/nginx/access.log` | Nginx logs (for poisoning) |
| `/var/log/auth.log` | SSH auth logs (for poisoning) |
| `/var/log/mail.log` | Mail logs (for poisoning) |
| `/var/www/html/config.php` | Database credentials |
| `/var/www/html/.env` | API keys, secrets |
| `~/.ssh/authorized_keys` | SSH key write target |
| `~/.ssh/id_rsa` | Private SSH key |
| `/opt/app/config/database.yml` | Rails DB config |
| `/proc/net/tcp` | Active network connections |
