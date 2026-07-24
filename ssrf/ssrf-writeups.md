# SSRF Writeups — The Ones That Matter

> Only writeups with real technique breakdowns, bypass details, and transferable skills. No empty HackerOne shells.

## The Must-Reads

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 1 | SSRF in the Wild: Analysis of 200+ Real-World SSRF Vulnerabilities | The Bug Hunter | Data-driven patterns from 217 disclosed reports, filter bypass stats, cloud metadata abuse trends | https://www.thebughunter.blog/case-studies/ssrf-analysis |
| 2 | Just Gopher It — Blind SSRF to RCE for $15k | SirLeeroyJenkins | Gopher protocol exploitation, scheme-dropping bypass, IMDSv1 credential theft, post-exploitation with AWS CLI | https://sirleeroyjenkins.medium.com/just-gopher-it-escalating-a-blind-ssrf-to-rce-for-15k-f5329a974530 |
| 3 | Five Bounties, One Bug — Five SSRF Bypass Techniques | Kayra Öksüz | Validate/execute mismatch, IP-encoding fuzzing, retesting after fixes, bypass chain persistence | https://medium.com/@oksuzkayra16/five-bounties-one-bug-exploiting-the-same-ssrf-via-five-unique-techniques-3f0adb7965d6 |
| 4 | SSRF to Server Takeover PoC | Malvin Valerian | GCP metadata endpoint without required header, Network tab recon, source_file param abuse | https://medium.com/@malvinval/ssrf-to-server-takeover-poc-bug-bounty-writeup-82d6715e333d |
| 5 | Breaking SSRF Defenses: Real-World Bypass Techniques | Rahul Dhawan | Interactive lab walkthrough, IPv6 mapped bypass, DNS rebinding TOCTOU, redirect chaining, IMDSv2 bypass via internal service chaining | https://rahuldhawan.me/breaking-ssrf-defenses-real-world-bypass-techniques |
| 6 | SSRF worth $4,913 — Dropbox/HelloSign | Sayaan Alam | File import SSRF discovery, not giving up after 404, pattern recognition for SSRF-shaped features | https://medium.com/techfenix/ssrf-server-side-request-forgery-worth-4913-my-highest-bounty-ever-7d733bb368cb |
| 7 | Bypassing SSRF Protection to Exfiltrate AWS Metadata from LarkSuite | Jacob Krut | DNS rebinding to bypass SSRF protections, Lark wiki attachment exfil, full AWS credential extraction | https://sirleeroyjenkins.medium.com/bypassing-ssrf-protection-to-exfiltrate-aws-metadata-from-larksuite-bf99a3599462 |
| 8 | A New Era of SSRF — Exploiting URL Parsers | Orange Tsai | URL parser confusion across languages, localhost representation tricks, the foundational research most bypasses are based on | https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf |
| 9 | Full SSRF to read AWS Metadata | Louis Shyers | Webhook test functionality abuse, open redirect to metadata endpoint, critical classification criteria | https://medium.com/@l_s_/full-read-ssrf-leading-to-aws-metadata-service-b62823e5b93a |
| 10 | SSRF to Internal AWS Metadata — A Bug Bounty Case Study | khandal.tech | Open redirect + blind SSRF chain, IAM credential extraction, EC2 metadata service walkthrough | https://khandal.tech/blogs/ssrf-aws-metadata/ |

## Cloud Metadata & Escalation

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 11 | Exploitation of an SSRF vulnerability against EC2 IMDSv2 | Yassine Aboukir | IMDSv2 bypass via internal service header forwarding, chained SSRF attack flow | https://www.yassineaboukir.com/blog/exploitation-of-an-ssrf-vulnerability-against-ec2-imdsv2/ |
| 12 | SSRF to Cloud Metadata to RCE | Bug Bounty Playbook | Full attack flow diagram, AWS/GCP/Azure metadata endpoints, credential extraction and IAM enumeration | https://bugbounty.info/Chains/SSRF-to-Cloud-RCE |
| 13 | Steal EC2 Metadata Credentials via SSRF | Hacking The Cloud | AWS IMDSv1 vs IMDSv2, detection, mitigation, the canonical reference for cloud SSRF | https://hackingthe.cloud/aws/exploitation/ec2-metadata-ssrf |
| 14 | SSRF → reach internal Redis → write SSH key → RCE | Attack Paths | Full kill-chain: SSRF → gopher → Redis CONFIG SET → authorized_keys write → SSH login | https://www.attackpaths.org/en/paths/db-redis-ssrf-rce |
| 15 | Bug Bounty Story: Escalating SSRF to RCE on AWS | hg8 | SSRF to RCE chain on real AWS infrastructure, timeline and patching process | https://hg8.sh/posts/bugbounty/ssrf-to-rce-aws |
| 16 | Exploiting Lambda SSRF to Access S3 Buckets | kariarce2377 | Lambda function SSRF, S3 bucket enumeration, IAM role abuse | https://medium.com/@kariarce2377/cybr-academy-exploiting-lambda-ssrf-to-access-s3-buckets-walkthrough-198b04cc25b1 |

## Filter Bypass & Defense Evasion

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 17 | SSRF against cloud metadata in 2026 — IMDSv2, parser confusion, and DNS rebinding | Ashfaqul Haq | Modern IMDSv2 bypasses, parser confusion attacks, DNS rebinding against cloud metadata | https://0xashfaq.github.io/blog/web-exploitation/ssrf-cloud-metadata-modern-bypasses |
| 18 | SSRF via DNS Rebinding That Leads to Several Clouds Access | Berserker1337 | DNS rebinding across AWS/GCP/Azure, multi-cloud exploitation | https://medium.com/@Berserker1337/ssrf-via-dns-rebinding-that-leads-to-several-clouds-access-arabic-db9f9a9c530c |
| 19 | DNS Rebinding Explained with Real Demos | M Square LLC | Singularity tool setup, DNS cache flushing, Google Home exploitation, router attacks | https://msquarellc.net/blog/dns-rebinding-explained |
| 20 | DNS rebinding attacks explained | GitHub Security Lab | DNS rebinding without CORS, browser caching behavior, real-world vulnerability exploitation | https://github.blog/security/application-security/dns-rebinding-attacks-explained-the-lookup-is-coming-from-inside-the-house/ |
| 21 | SSRF DNS Rebinding: Bypassing Protection with CVE-2025-69660 | Aydin Yunus | CVE-based DNS rebinding, zero-second TTL technique | https://aydinnyunus.github.io/2026/03/14/ssrf-dns-rebinding-vulnerability/ |
| 22 | Circumventing Common SSRF Defenses | cyberw1ng | Strategy-level bypass techniques, defense-by-defense breakdown | https://infosecwriteups.com/circumventing-common-ssrf-defenses-a-deep-dive-into-strategies-and-techniques-c9607f1bb61e |

## PDF, HTML Renderers & Non-Obvious Surfaces

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 23 | Exploiting HTML-to-PDF Converters through HTML Imports | mhmdiaa | HTML import exploitation in PDF generators, file:// protocol abuse, WeasyPrint/Puppeteer attack surface | https://mhmdiaa.com/blog/exploiting-html-imports/ |
| 24 | SSRF in PDF HTML Injection: Basic and Blind | jbince | PDF generation SSRF, both in-band and blind exploitation | https://infosecwriteups.com/exploiting-ssrf-in-pdf-html-injection-basic-and-blind-047fec5317ae |
| 25 | SSRF To Internal Data Access Via PDF Print Feature | bishal0x01 | PDF print feature as SSRF surface, internal data exfiltration | https://infosecwriteups.com/ssrf-to-internal-data-access-via-pdf-print-feature-b8e6a912844a |
| 26 | Exploiting SSRF in an API | Dana Epp | API-specific SSRF patterns, OWASP API Top 10 context | https://danaepp.com/exploiting-ssrf-in-an-api |
| 27 | SSRF via Gateway actuator | Intigriti Bug Bytes | Spring Boot actuator as SSRF surface, internal service enumeration | https://www.intigriti.com/researchers/blog/bug-bytes/bug-bytes-152-ssrf-via-gateway-actuator-flickr-account-takeover-writeup-of-nsos-imessage-rce |

## Recon & Discovery Methodology

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 28 | From Wayback Machine to AWS Metadata: SSRF in Production Within 5 Minutes | gguzelkokar | Wayback Machine recon → SSRF → cloud metadata, speed and efficiency | https://medium.com/@gguzelkokar.mdbf15/from-wayback-machine-to-aws-metadata-uncovering-ssrf-in-a-production-system-within-5-minutes-2d592875c9ab |
| 29 | How I Discovered SSRF on HackerOne Program | kerstan | GraphQL as SSRF surface, DNS rebinding bypass, port scanning via SSRF | https://readmedium.com/how-i-discovered-ssrf-on-hackerone-program-7bbe72334f74 |
| 30 | How to find SSRF, Bypass Cloudflare, and extract AWS metadata | anontriager | Cloudflare bypass, metadata extraction workflow | https://anontriager.medium.com/how-to-find-ssrf-bypass-cloudflare-and-extract-aws-metadata-46d1ee6d1857 |
| 31 | Find SSRF, LFI, XSS using httpx, waybackurls, gf, gau, qsreplace | kerstan | Automation pipeline for SSRF discovery at scale | https://medium.com/@kerstan/how-use-6-step-to-find-ssrf-bug-bounty-tuesday-acc44d806c08 |
| 32 | SSRF leads to access AWS metadata | Akash Patil | Proxy endpoint SSRF discovery, AWS metadata extraction | https://infosecwriteups.com/ssrf-leads-to-access-aws-metadata-21952c220aeb |
| 33 | SSRF via DNS Rebinding (CVE-2022–4096) | Unknown | CVE-based DNS rebinding, Ceye for DNS callback | https://infosecwriteups.com/ssrf-via-dns-rebinding-cve-2022-4096-b7bf75928bb2 |

## Technique Collections & Guides

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 34 | Mastering Server-Side Request Forgery Exploitation in 2025 | SQUID SEC | Complete technique catalog, every bypass family, tool comparison | https://squidhacker.com/2025/05/mastering-server-side-request-forgery-ssrf-exploitation-in-2025 |
| 35 | SSRF in 2026: Attack Patterns, Cloud Metadata Exploitation | AppSec Brief | Modern attack patterns, defense-in-depth, Python/Node.js secure code | https://appsecbrief.com/articles/ssrf-prevention-guide-2026 |
| 36 | Complete SSRF Guide | ssrf.page | End-to-end reference: detection, exploitation, bypass, cloud, blind | https://ssrf.page/ |
| 37 | SSRF Attacks — KaliRange Labs | KaliRange | Hands-on lab walkthrough, Gopher payloads, Redis exploitation | https://kalirange.com/v2/labs/ssrf-attacks.html |
| 38 | Exploiting SSRF Vulnerabilities: From Detection to RCE | Pzod Security | Detection techniques, cloud metadata, Redis/Memcached exploitation | https://www.pzod.io/blog/ssrf-to-rce |
| 39 | SSRF Types and Ways to Exploit It | SaN ThosH | URL scheme exploitation, cloud metadata attacks, foundational guide | https://medium.com/@madrobot/ssrf-server-side-request-forgery-types-and-ways-to-exploit-it-part-1-29d034c27978 |
| 40 | SSRF to RCE: A Case Study in Exploiting Chained Vulnerabilities | Land2Cyber | Chained exploitation methodology | https://medium.com/@Land2Cyber/ssrf-to-rce-a-case-study-in-exploiting-chained-vulnerabilities-78f290ae9011 |
| 41 | SSRF Hunting in the Cloud | Land2Cyber | Cloud-specific SSRF hunting methodology | https://medium.com/@Land2Cyber/ssrf-hunting-in-the-cloud-exploiting-misconfigured-services-in-cloud-environments-c0d4604fa06c |

## Real-World Case Studies

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 42 | $300 Bounty: SSRF to Cloud Metadata | Monika Sharma | Phabricator SSRF, meme generation feature abuse, IMDSv1 exploitation | https://infosecwriteups.com/300-bounty-ssrf-to-cloud-metadata-4c6a7dda9818 |
| 43 | The $900 Bug: My Journey Through SSRF and LFI Exploits | myselfakash20 | SSRF + LFI chaining, practical impact escalation | https://osintteam.blog/the-900-bug-my-journey-through-ssrf-and-lfi-exploits-222feb276deb |
| 44 | SSRF to Cloud Metadata Exploitation | CSPI | Blind SSRF detection methodology, timing and OOB channels, URL filter bypass | https://cybersecpentesting.com/blog/ssrf-cloud-metadata-exploitation.html |
| 45 | SSRF via Gateway actuator | Intigriti | Spring Boot actuator exploitation, Flickr account takeover chain | https://www.intigriti.com/researchers/blog/bug-bytes/bug-bytes-152-ssrf-via-gateway-actuator-flickr-account-takeover-writeup-of-nsos-imessage-rce |
| 46 | Understanding SSRF: Real-World Exploitation | Shah Kaif | Step-by-step AWS metadata abuse, blacklist vs whitelist bypass, XXE-based SSRF | https://infosecwriteups.com/understanding-ssrf-real-world-exploitation-of-server-side-request-forgery-792fadb3350f |
| 47 | SSRF to Internal Data Access Via PDF Print Feature | bishal0x01 | PDF print as SSRF vector, internal network access | https://infosecwriteups.com/ssrf-to-internal-data-access-via-pdf-print-feature-b8e6a912844a |
| 48 | SSRF in Search.gov via ?url= parameter | HackerOne #514224 | Government target, simple URL parameter SSRF, VDP scope rules | https://hackerone.com/reports/514224 |
| 49 | Blind SSRF (external service interaction) Via Uploading QR Code Image | Manojchy | QR code image as SSRF trigger, blind detection technique | https://medium.com/@Manojchy/blind-ssrf-external-service-interaction-dns-and-http-via-uploading-qr-code-image-49c8022375ee |
| 50 | SSRF in Exchange leads to ROOT access in all instances | HackerOne #341876 | Shopify Exchange SSRF to root, the canonical high-impact SSRF report | https://hackerone.com/reports/341876 |
