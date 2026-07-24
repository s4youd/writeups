# SQL Injection Writeups — The Ones That Matter

> Only writeups with real technique breakdowns, bypass details, and transferable skills. No empty shells.

## The Must-Reads

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 1 | SQL Injection 2026: Blind, Time-Based, ORM Bypass, and WAF Evasion | Hive Security | The complete 2026 reference: all 5 SQLi types, PostgreSQL CVE-2025-1094 breach chain (BeyondTrust → US Treasury), ORM `.raw()` escape hatches, least-privilege DB accounts, full payloads per database engine | https://hivesecurity.gitlab.io/blog/sql-injection-complete-guide-2026 |
| 2 | SQL Injection Cheat Sheet 2026: Payloads, Bypasses, Defenses | Vulnsy | 2025 academic mutation strategies (BWAFSQLi ACM TOSEM), PostgreSQL CVE-2025-1094 encoding-confusion, JSON-syntax WAF bypass, real CVEs through 2026, context-aware payloads | https://www.vulnsy.com/cheat-sheets/sql-injection |
| 3 | Advanced SQL Injection: WAF Bypass Techniques That Still Work | HackerXone | 2026 Verizon DBIR: injection = 23% of web breaches, modern WAF detection mechanisms, encoding bypass, HTTP/2 desync, real-world attack scenarios | https://hackerxone.com/advanced-sql-injection-waf-bypass-techniques-that-still-work/ |
| 4 | A Complete Conquest of SQL Injection Attacks [2026] | Tomodahinata | PortSwigger-aligned: UNION, blind, time-based, sqlmap, WAF bypass — hands-on granularity with real command output | https://tomodahinata.com/en/blog/sql-injection-attack-techniques-union-blind-sqlmap-waf-bypass-guide |
| 5 | Mastering SQLMap and Ghauri: Practical Guide to WAF Bypass | Lostsec | Step-by-step SQLMap/Ghauri usage, WAF detection and evasion, real 2026 methodology, 13-minute deep read | https://infosecwriteups.com/mastering-sqlmap-and-ghauri-a-practical-guide-to-waf-bypass-techniques-1aaa9eee9d32 |
| 6 | Mastering the SQL Injection Test: Advanced Techniques, CVE Case Studies | Penligent | 3-element validation framework (reachability, semantic alteration, observability), MOVEit CVE-2023-34362 case study, async background job injection, AI-generated code risks | https://www.penligent.ai/hackinglabs/mastering-the-sql-injection-test-advanced-techniques-cve-case-studies-and-ai-defense/ |

## Blind SQLi & Time-Based

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 7 | Time-Based SQL Injection: Complete Real-World Bug Bounty Guide | Monika Sharma (OSINT Team) | Full end-to-end: target approach, Burp Repeater workflow, true/false condition testing, repeatability proof, database engine identification (MySQL/PostgreSQL/MSSQL/Oracle), JSON API testing, header-based injection, automation only after manual proof | https://osintteam.blog/time-based-sql-injection-complete-real-world-bug-bounty-guide-0d38311a4adf |
| 8 | Blind SQL Injection Explained: A 2026 Hacker's Guide | HackerDNA | Boolean-based vs time-based vs OOB comparison, character-by-character extraction methodology, SLEEP/WAITFOR/pg_sleep/dbms_pipe per engine, network jitter handling, OOB DNS/SMB exfiltration | https://hackerdna.com/blog/blind-sql-injection |
| 9 | What is Blind SQL Injection? Ways to Exploit, Examples and Impact | Jsmon (Inderjeet Singh) | Boolean-based with response comparison, time-based with SLEEP(), character extraction loop, identification methods, prevention strategies | https://blogs.jsmon.sh/what-is-blind-sql-injection-ways-to-exploit-examples-and-impact/ |
| 10 | Critical Blind SQL Injection leads to $4,134 (7/30 Days) | 0day stories | Real bounty writeup: blind SQLi discovery, exploitation timeline, $4,134 payout on private program | https://infosecwriteups.com/critical-blind-sql-injection-leads-to-4-134-7-30-days-d8918ff3d2d0 |
| 11 | How Blind SQL Injection Works — CEH Scenario with Time-Based Example | Web Asha Technologies | Training-grade breakdown: SLEEP function mechanics, WAITFOR DELAY, BENCHMARK, login form blind injection scenario | https://www.webasha.com/blog/how-blind-sql-injection-works-scenario-with-time-based-example |

## WAF Bypass & Filter Evasion

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 12 | WAF Bypass Techniques: Encoding, HTTP/2 Desync (2026) | DecryptionDigest | 73% of attacks attempt WAF evasion first (Imperva 2025), HTTP/2 cleartext injection, CRS rule tuning, double-encoding, smuggling headers, SIEM integration, quarterly bypass testing | https://www.decryptiondigest.com/blog/waf-bypass-techniques-detection-rule-tuning |
| 13 | Advanced-SQL-Injection-Cheatsheet: WAF Bypass Techniques | kleiton0x00 | Taxonomy of bypass techniques, NULL-based alternatives, AND 0 alternatives, alternative injection vectors, comprehensive GitHub reference | https://deepwiki.com/kleiton0x00/Advanced-SQL-Injection-Cheatsheet/3-waf-bypass-techniques |
| 14 | WAF Bypass with SQL Injection Techniques | Mahbod Farajian (Gist) | Copy-ready payloads: obfuscation, encoding techniques, basic and intermediate WAF bypass methods | https://gist.github.com/Mahbod-Farajian/da5ed49c5d6649fc9f812d94d79642b9 |
| 15 | The Art of Bypassing WAFs: Techniques, Testing & Defense Playbook | Penligent | WAFFLED 2025 study: 1,207 real bypass instances across AWS/Azure/Cloudflare/Google/ModSecurity, parsing inconsistencies, content-type loose handling, safe automation practices | https://www.penligent.ai/hackinglabs/the-art-of-bypassing-wafs-techniques-testing-defense-playbook/ |
| 16 | OWASP: SQL Injection Bypassing WAF | OWASP Community | Official OWASP reference: encoding tricks, case manipulation, inline comments, alternative syntax, HTTP parameter pollution for SQLi WAF bypass | https://github.com/OWASP/www-community/blob/master/pages/attacks/SQL_Injection_Bypassing_WAF.md |
| 17 | WAF Bypass: The Ultimate Guide | WAF.is / Claroty Team82 | JSON-based SQLi WAF bypass (Team82 research), JSON support review across organizations, generic WAF bypass methodology | https://waf.is/articles/waf-bypass-guide |

## CVEs & Real-World Breaches

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 18 | PostgreSQL CVE-2025-1094: BeyondTrust → US Treasury Breach | OX Security / Hive Security | Encoding-confusion SQLi under BIG5 client + EUC_TW server, PQescapeLiteral failure, full breach chain from PostgreSQL to US Treasury, libpq patch versions | https://www.ox.security/blog/lessons-from-the-postgresql-cve-2025-1094-exploitation/ |
| 19 | CVE-2026-42208: LiteLLM AI-Gateway Pre-Auth SQLi (CVSS 9.3) | NVD | Pre-authentication SQL injection in AI gateway, exploited in-the-wild 26 hours after advisory, LLM infrastructure as SQLi target | https://nvd.nist.gov/vuln/detail/CVE-2026-42208 |
| 20 | CVE-2026-40906: ElectricSQL Error-Based SQLi (CVSS 9.9) | NVD | Error-based SQLi in /v1/shape endpoint against Postgres, authenticated user can read/write/destroy database, sync-gateway as under-tested surface | https://nvd.nist.gov/vuln/detail/CVE-2026-40906 |
| 21 | CVE-2026-42087: OpenC3 COSMOS Time-Series DB SQLi (CVSS 9.6) | NVD | Critical SQLi in telemetry/command-and-control deployments, TSDB front-end injection | https://nvd.nist.gov/vuln/detail/CVE-2026-42087 |
| 22 | CVE-2026-27681: SAP Product-Family Critical SQLi | NVD | Classic-shape SQLi in enterprise ERP integrations, enterprise SaaS still shipping injection in 2026 | https://nvd.nist.gov/vuln/detail/CVE-2026-27681 |
| 23 | FG-IR-25-151: Fortinet GUI Unauthenticated SQLi | NVD | Unauthenticated SQLi in perimeter security device management GUI, high-value target class | https://nvd.nist.gov/vuln/detail/FG-IR-25-151 |
| 24 | MOVEit Transfer CVE-2023-34362: When SQLi Testing Fails | Penligent | Why shallow testing missed backend API injection, async automation endpoint, human2.aspx webshell deployment, global impact | https://www.penligent.ai/hackinglabs/mastering-the-sql-injection-test-advanced-techniques-cve-case-studies-and-ai-defense/ |
| 25 | CVE-2026-3164: SQL Injection in News Portal Applications | WAF Insider | Real-world impact of inadequate SQLi defense, 7 essential defense strategies, WAF rules for news portal SQLi | https://waf.is/articles/sql-injection-defense-cve-2026-3164-strategies |

## Out-of-Band, Second-Order & Advanced Techniques

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 26 | SQL Injection 2026: Out-of-Band (OOB) Injection | Hive Security | MySQL LOAD_FILE DNS exfil, MSSQL xp_dirtree SMB exfil, database-initiated outbound connections, async background job injection | https://hivesecurity.gitlab.io/blog/sql-injection-complete-guide-2026 |
| 27 | SQL Injection 2026: Second-Order Injection | Hive Security | Payload stored in DB, executed later in different query context, ORMs with raw queries, stored procedures as injection surface | https://hivesecurity.gitlab.io/blog/sql-injection-complete-guide-2026 |
| 28 | SQL Injection 2026: ORM Bypass | Hive Security | `.raw()` escape hatches in Django/Rails/Sequelize, string concatenation in ORM queries, why ORMs reduce but don't eliminate risk | https://hivesecurity.gitlab.io/blog/sql-injection-complete-guide-2026 |
| 29 | JSON-Based SQLi WAF Bypass (Team82) | Claroty Team82 | JSON syntax abuse to bypass WAF SQLi rules, generic bypass across WAF vendors, JSON support gaps in WAF parsing | https://waf.is/articles/waf-bypass-guide |
| 30 | PostgreSQL libpq Encoding-Confusion (CVE-2025-1094) | Vulnsy | BIG5/EUC_TW encoding confusion, PQescapeLiteral failure, `\xa1' OR 1=1--` payload, patched in libpq 17.3/16.7/15.11/14.16/13.19 | https://www.vulnsy.com/cheat-sheets/sql-injection |

## Injection Points & Attack Surfaces in 2025-2026

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 31 | Modern SQLi Entry Points: Beyond HTML Forms | Penligent | JSON APIs & GraphQL, HTTP headers (User-Agent, X-Forwarded-For), file imports (CSV/XML/XLSX), background jobs, AI-assisted query builders | https://www.penligent.ai/hackinglabs/mastering-the-sql-injection-test-advanced-techniques-cve-case-studies-and-ai-defense/ |
| 32 | Time-Based SQLi: Testing APIs & JSON Requests | Monika Sharma | JSON body injection: `{"filter": "test' AND IF(1=1, SLEEP(5), 0)--"}`, test every key, test numbers as strings, test nested objects, APIs skip WAF | https://osintteam.blog/time-based-sql-injection-complete-real-world-bug-bounty-guide-0d38311a4adf |
| 33 | Time-Based SQLi: Header-Based Injection | Monika Sharma | User-Agent, X-Forwarded-For, Referer as SQLi vectors, logging systems inserting headers directly into SQL, Burp testing workflow | https://osintteam.blog/time-based-sql-injection-complete-real-world-bug-bounty-guide-0d38311a4adf |
| 34 | SQLi in AI-Generated Code | Penligent | AI coding assistants using string concatenation for queries, hallucinated "safe" wrapper functions, treating AI-generated DB code as untrusted | https://www.penligent.ai/hackinglabs/mastering-the-sql-injection-test-advanced-techniques-cve-case-studies-and-ai-defense/ |

## Detection & Defense

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 35 | SQL Injection 2026: Detection Queries | Hive Security | MySQL general query log, PostgreSQL log_statement, monitoring for UNION/SLEEP/information_schema from app accounts, red flags in DB logs | https://hivesecurity.gitlab.io/blog/sql-injection-complete-guide-2026 |
| 36 | SQL Injection 2026: Parameterized Queries — The Only Real Fix | Hive Security | `cursor.execute("SELECT * FROM users WHERE user = %s", (user,))`, safe ORM patterns, why parameterization is the only true fix | https://hivesecurity.gitlab.io/blog/sql-injection-complete-guide-2026 |
| 37 | SQL Injection 2026: Least Privilege Database Accounts | Hive Security | Creating restricted app users, GRANT SELECT/INSERT/UPDATE only, never GRANT ALL/FILE/SUPER/EXECUTE, blast radius reduction | https://hivesecurity.gitlab.io/blog/sql-injection-complete-guide-2026 |
| 38 | SQL Injection Test: A Visibility Framework | Penligent | Classification by signal type: error-based, union-based, boolean-blind, time-based, OOB — which technique for which environment | https://www.penligent.ai/hackinglabs/mastering-the-sql-injection-test-advanced-techniques-cve-case-studies-and-ai-defense/ |

## Cheat Sheets & References

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 39 | Vulnsy SQL Injection Cheat Sheet (2026) | Vulnsy | Complete payload reference: context-aware, per-database-engine, academic mutation strategies, modern bypasses 2023-2026, real CVEs | https://www.vulnsy.com/cheat-sheets/sql-injection |
| 40 | OWASP SQL Injection | OWASP Foundation | Official reference: attack types, prevention cheat sheet, parameterized queries, stored procedures, escape functions | https://owasp.org/www-community/attacks/SQL_Injection |
| 41 | PortSwigger SQL Injection | PortSwigger Web Security Academy | Interactive labs, UNION-based, blind, second-order, out-of-band, oracle brute-force | https://portswigger.net/web-security/sql-injection |
| 42 | kleiton0x00 Advanced SQL Injection Cheatsheet | GitHub | Comprehensive bypass techniques, NULL alternatives, AND 0 alternatives, WAF bypass taxonomy, alternative injection methods | https://github.com/kleiton0x00/Advanced-SQL-Injection-Cheatsheet |
| 43 | RootMe SQL Injection Challenges | nh4ttruong/emtoor | Writeups for 6+ SQLi challenges: authentication bypass, error-based, UNION-based, blind, time-based, GBK encoding | https://nh4ttruong.github.io/emtoor/SQL-Injection |

## Key Numbers to Remember

| Metric | Value | Source |
|--------|-------|--------|
| Injection share of web app breaches (2026) | ~23% | Verizon DBIR 2026 |
| SQLi share of all web app attacks | Two-thirds | Dark Reading |
| WAF evasion attempts before payload | 73% of attacks | Imperva 2025 |
| WAF bypass instances in WAFFLED study | 1,207 | WAFFLED 2025 |
| LiteLLM AI-gateway pre-auth SQLi CVSS | 9.3 | CVE-2026-42208 |
| ElectricSQL error-based SQLi CVSS | 9.9 | CVE-2026-40906 |
| OpenC3 COSMOS SQLi CVSS | 9.6 | CVE-2026-42087 |
| PostgreSQL CVE-2025-1094 impact | BeyondTrust → US Treasury | OX Security |
| Blind SQLi bounty (7/30 days) | $4,134 | 0day stories |
| MOVEit Transfer global impact | Thousands of organizations | Penligent |
| Freepik breach records | 8.3 million | BleepingComputer |
| 2025 academic SQLi mutation strategies | 13 obfuscations extended | BWAFSQLi ACM TOSEM |
