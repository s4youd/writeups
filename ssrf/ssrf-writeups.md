# SSRF Mastery Reference — Foundations + Write-ups + Skills Extracted

> **Honest framing first:** nobody is ever "100% done" with a bug class. SSRF bypass techniques evolve as fast as cloud providers and frameworks patch the last one. What *is* achievable is having the full reference layer memorized, a working set of tools/fuzzing habits, and a standing habit of reading new disclosures. This document gives you all three: the living reference pages (Part A), 13 real write-ups broken down for *every* transferable skill — not just the SSRF payload (Part B), and a consolidated cheat sheet (Part C). Read Part A first; it's what every write-up in Part B is quietly drawing from.

---

# Part A — Foundational References (bookmark these, not just this file)

These are **living documents** — they get updated as new bypasses are discovered. This file will go stale; these mostly won't, because the community keeps them current.

| # | Resource | Why it matters |
|---|----------|-----------------|
| A1 | **HackTricks — SSRF main page** ([link](https://hacktricks.wiki/en/pentesting-web/ssrf-server-side-request-forgery/index.html)) | The single most complete free bypass catalog: redirect abuse, DNS rebinding, SNI-based SSRF, Referrer-header SSRF, TLS/AIA cert-issuer SSRF, gopher protocol. Treat this as your primary reference — check it before every SSRF engagement. |
| A2 | **HackTricks — Cloud SSRF page** ([link](https://hacktricks.wiki/en/pentesting-web/ssrf-server-side-request-forgery/cloud-ssrf.html)) | IMDSv1 vs IMDSv2 differences, GCP identity-token theft (with `audience` parameter for IAP-protected backends), Azure Managed Identity token theft, EKS/pod-identity credential theft via env vars. The page to read *after* you've confirmed SSRF and need to know exactly what to request next, per cloud provider. |
| A3 | **HackTricks — SSRF Vulnerable Platforms page** ([link](https://hacktricks.wiki/en/pentesting-web/ssrf-server-side-request-forgery/ssrf-vulnerable-platforms.html)) | A running catalog of *specific* software/platforms with known SSRF footguns (Next.js Image Optimization, webhook workers in Grafana OnCall/Gogs/Soft Serve/Firecrawl). If your target runs a name-brand framework, check this page for a pre-built primitive. |
| A4 | **PortSwigger Web Security Academy — SSRF topic** ([link](https://portswigger.net/web-security/ssrf)) | The clearest conceptual explanation of *why* SSRF is dangerous (trust relationships, network topology), plus free hands-on labs to actually practice each variant instead of only reading about it. |
| A5 | **PayloadsAllTheThings — Server Side Request Forgery** ([github.com/swisskyrepo/PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)) | Raw payload reference — every IP-encoding trick, every protocol handler, every cloud metadata URL, kept in one grep-able repo. This is what you paste into Burp Intruder, not what you read start-to-finish. |
| A6 | **The Hacker Recipes — SSRF** ([link](https://www.thehacker.recipes/web/inputs/ssrf/)) | Sharpest one-page distinction between SSRF and file-inclusion vulnerabilities, plus a tight tool list (SSRFmap, Gopherus). Good for building correct mental models before you start hunting. |
| A7 | **"Book of Bug Bounty Tips" — SSRF page** ([gowsundar.gitbook.io](https://gowsundar.gitbook.io/book-of-bugbounty-tips/ssrf)) | A raw, unfiltered dump of hundreds of hunters' individual tweeted tips — pure trick density, no narrative padding. The page to skim when you're stuck and need "one more thing to try." |
| A8 | **YesWeHack — The Ultimate Guide to Exploiting SSRF** ([link](https://www.yeswehack.com/learn-bug-bounty/server-side-request-forgery-ssrf)) | The best *methodology* piece in this whole list — explicitly covers the Burp extensions **Param Miner** and **Collaborator Everywhere** for discovering SSRF primitives you'd never find by manual testing. Summarized in Part C below. |
| A9 | **OWASP SSRF Prevention Cheat Sheet** ([cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)) | Read this from the *defender's* side — tells you exactly which mitigations are considered "correct" (allow-lists over deny-lists, disabling redirects, network segmentation). Knowing the correct fix tells you which half-measures to expect and target. |

---

# Part B — 13 Write-ups, Every Skill Extracted

Each entry below covers **five things**, not just the vulnerability: what happened, the recon/discovery skill, the fuzzing/testing skill (called out explicitly since most "top writeups" lists skip this entirely), the tools used, and the SSRF-specific trick. This is the part most compilations miss.

---

## 1. SSRF worth $4,913 — Dropbox/HelloSign
**Author:** Sayaan Alam · [Medium](https://medium.com/techfenix/ssrf-server-side-request-forgery-worth-4913-my-highest-bounty-ever-7d733bb368cb)

- **What happened:** HelloSign's "import from Dropbox/GDrive/Box/OneDrive/EverNote" feature took a `file_reference` param. First payload got a 404 — looked patched.
- **Recon/discovery skill:** Recognized *any* "import a document from another service" feature as SSRF-shaped the moment they saw it — didn't need a hint, it's a pattern to actively hunt for.
- **Fuzzing/testing skill:** None needed here — this is a case of *not giving up* after one failed test rather than a fuzzing technique. Worth noting because giving up after the first blocked payload is the single most common reason hunters miss real SSRF.
- **Tools used:** Burp Collaborator (to confirm out-of-band interaction), a self-hosted PHP redirect page.
- **SSRF trick:** 303-redirect bypass, sourced from reading a *different* company's disclosed HackerOne report first.
- **Mindset lesson:** When a direct payload is blocked, the next move isn't more creativity on your own — it's checking Hacktivity for the exact same primitive on other programs.

---

## 2. SSRF in Webhooks → AWS Private Keys — Omise (Hacktivity)
**Author:** honoki · [HackerOne #508459](https://hackerone.com/reports/508459)

- **What happened:** Omise's webhook feature followed 303 redirects even though ordinary 301/302 were filtered — landed on the AWS metadata IAM credentials endpoint.
- **Recon/discovery skill:** Identified the webhook feature as the SSRF surface simply because "the server has to make an outbound request to register it" — the same recognition move as write-up #1, applied independently.
- **Fuzzing/testing skill:** Effectively fuzzed *redirect status codes* by hand (301 → blocked, tried 303 → worked) rather than payload content — a reminder that fuzzing isn't only about wordlists, it's about systematically varying any input dimension, including HTTP status codes.
- **Tools used:** A minimal PHP redirect script — no automation needed; a reminder that expensive tooling isn't a prerequisite for a critical finding.
- **SSRF trick:** The original source of the 303-bypass technique reused in write-up #1 — read this one directly for the precise language the researcher used to explain *why* 303 slipped past validation logic that checked 301/302 only.
- **Mindset lesson:** Report clarity mattered here — the researcher explicitly asked the triager to justify a severity downgrade rather than silently accepting it, and got the rating corrected. Advocate for accurate severity, respectfully.

---

## 3. SSRF to Server Takeover — Streaming Feature
**Author:** Malvin Valerian · [Medium](https://medium.com/@malvinval/ssrf-to-server-takeover-poc-bug-bounty-writeup-82d6715e333d)

- **What happened:** An audio-streaming feature's `source_file` param, pointed at the GCP metadata endpoint, leaked SSH configuration details — escalated toward full takeover.
- **Recon/discovery skill:** Opened the browser Network tab while using a completely unrelated feature (pressing "play" on an audio player) instead of only inspecting obvious forms — SSRF surfaces hide in features that don't look like "import a URL" on the surface.
- **Fuzzing/testing skill:** None explicit, but implicitly tested multiple cloud metadata endpoint formats until GCP's responded — a reminder to try all three major cloud metadata URL formats rather than assuming AWS.
- **Tools used:** Browser DevTools Network tab (an underrated recon tool — most hunters jump straight to Burp and skip this).
- **SSRF trick:** GCP metadata endpoint (`computeMetadata/v1`) leaking data despite the provider's own documentation saying it requires a `Metadata-Flavor: Google` header — a reminder that documented "protections" aren't always actually enforced server-side.
- **Mindset lesson:** Watch the Network tab on *every* feature during normal use, not just ones with an obvious "paste a URL here" field.

---

## 4. $25,000 SSRF in HackerOne's Own PDF Generator
**Author:** Monika Sharma (OSINT Team) · [Medium](https://osintteam.blog/25-000-ssrf-in-hackerones-analytics-reports-b9a5b3aa3d6e)

- **What happened:** An unsanitized template value rendered into HTML before PDF conversion — injecting an `<iframe>` triggered a server-side fetch to AWS metadata.
- **Recon/discovery skill:** Recognized PDF/report/export generators as a distinct SSRF category most hunters never test, because they don't contain an obvious URL parameter at all.
- **Fuzzing/testing skill:** Tried multiple HTML injection payload shapes (`<iframe>`, `<img src>`) to see which ones the PDF renderer would actually fetch server-side — a form of manual payload fuzzing against a rendering engine rather than a text field.
- **Tools used:** Manual testing only — no automation; the discovery came from understanding *how PDF renderers work internally* (they fetch external resources like a browser does) rather than from a scanner.
- **SSRF trick:** HTML-to-PDF SSRF — treat any document/report/invoice generator as "a headless browser under the hood," because that's usually literally what it is (wkhtmltopdf, Puppeteer, etc.).
- **Mindset lesson:** Understanding the *implementation* of a feature (rendering engines fetch external resources) beats blind payload-spraying — conceptual knowledge of how the backend works finds bugs scanners can't.

---

## 5. Five Bounties, One Bug — Five SSRF Bypass Techniques
**Author:** Kayra Öksüz · [Medium](https://medium.com/@oksuzkayra16/five-bounties-one-bug-exploiting-the-same-ssrf-via-five-unique-techniques-3f0adb7965d6)

- **What happened:** A blind SSRF got "fixed" four times; each fix was bypassed with a new IP-encoding or redirect-chaining trick.
- **Recon/discovery skill:** Retested the *same* endpoint repeatedly over time rather than moving on after the first fix shipped — patch-retesting as an ongoing recon habit, not a one-time check.
- **Fuzzing/testing skill:** Systematically fuzzed alternate representations of `127.0.0.1` (dotted variants, octal) against the filter until one slipped through — genuine input-space fuzzing, just done by hand with a mental list rather than a tool.
- **Tools used:** Timing/behavioral analysis only (this was blind SSRF start to finish) — no direct response reading at all, which makes the persistence here more impressive.
- **SSRF trick:** Validate/execute mismatch — the filter normalized and checked one representation of the IP while the HTTP client fetched a different, unnormalized representation.
- **Mindset lesson:** A single "fixed" report isn't necessarily closed forever — logged bypass attempts against previously-reported bugs are a legitimate, repeatable bounty source.

---

## 6. Just Gopher It — Blind SSRF to RCE for $15k
**Author:** SirLeeroyJenkins · [Medium](https://sirleeroyjenkins.medium.com/just-gopher-it-escalating-a-blind-ssrf-to-rce-for-15k-f5329a974530)

- **What happened:** A "logo grabber" feature, chained through a self-hosted redirector, reached internal Redis via the gopher protocol — full RCE via a cron-job payload.
- **Recon/discovery skill:** Ran subdomain enumeration *specifically to find more targets for the same SSRF primitive* — sprayed the same vulnerable request against every enumerated corporate subdomain, not just the main target.
- **Fuzzing/testing skill:** Genuine port fuzzing — redirected the SSRF sequentially at a fixed list of common internal ports (3306, 9000, 11211, 6379, 10050, 25) and used response timing as the oracle (fast = open, slow/timeout = closed). Real blind-fuzzing methodology, done manually with a redirect script.
- **Tools used:** Self-hosted Python redirect server, **Gopherus** (auto-generates gopher payloads for Redis/Memcached/MySQL/FastCGI/SMTP/Zabbix), Netcat listener.
- **SSRF trick:** Gopher protocol escalation from read-only SSRF to full command execution via Redis.
- **Mindset lesson:** The researcher was *seconds from giving up* (mouse literally on the terminal's close button) right before the payload landed — persistence past the point where a finding looks dead is explicitly called out as the difference-maker.

---

## 7. Escalating SSRF to RCE on AWS — Private Program
**Author:** hg8 · [hg8.sh](https://hg8.sh/posts/bugbounty/ssrf-to-rce-aws/)

- **What happened:** A "proxify" endpoint found during recon led to IMDSv1 credential theft after dropping the `http://` scheme bypassed a regex filter.
- **Recon/discovery skill:** Found the vulnerable endpoint purely through *subdomain enumeration surfacing an unusually-named endpoint* ("proxify") — not from an obvious feature, from noticing a suspicious name during routine recon.
- **Fuzzing/testing skill:** Minimal — mostly confirmed via `interact.sh` out-of-band interaction rather than fuzzing per se, but did iteratively test scheme variations (`http://`, no scheme, `https://`) against the filter.
- **Tools used:** `interact.sh` (self-hosted, free alternative to Burp Collaborator), AWS CLI (to actually use the stolen credentials afterward), **CloudGoat** + **Pacu** for practicing the post-exploitation half in a legal lab.
- **SSRF trick:** Scheme-dropping to bypass a URL-pattern filter; IMDSv1 plain-GET credential theft.
- **Mindset lesson:** Practicing the *post-SSRF* AWS exploitation chain (what to actually do with stolen credentials) in CloudGoat before you need it live is explicitly recommended — don't let unfamiliarity with AWS CLI/IAM cost you impact on a real finding.

---

## 8. Top 25 SSRF Reports — Pattern Compilation
**Author:** Cristian Cornea · [Medium](https://corneacristian.medium.com/top-25-server-side-request-forgery-ssrf-bug-bounty-reports-136928356eca)

- **What happened:** A curated round-up organized by *pattern* — SSRF+XXE combos, internal admin panels on non-standard ports.
- **Recon/discovery skill:** Frames SSRF as a pivot into *non-public attack surface*, not just a metadata-leak bug — a mental model shift worth adopting wholesale.
- **Fuzzing/testing skill:** N/A (this is an aggregator, not a single hunt) — but implicitly recommends port-fuzzing the internal IP space once SSRF is confirmed.
- **Tools used:** N/A (reference piece).
- **SSRF trick:** SSRF as an internal-admin-panel-discovery primitive — public app on 443, internal dashboard on 8008, invisible to normal recon but reachable once the server itself makes the request for you.
- **Mindset lesson:** Read aggregator/compilation pieces like this specifically to build a *mental catalog of shapes*, not to memorize any one incident — the value is pattern recognition across 25 cases at once.

---

## 9. zseano's SSRF Methodology — BugBountyHunter.com
**Author:** zseano · [BugBountyHunter.com](https://www.bugbountyhunter.com/vulnerability/?type=ssrf)

- **What happened:** Not one bug — a from-scratch hunting methodology from a well-known bug bounty educator.
- **Recon/discovery skill:** Explicit habit: **bank every open redirect you find**, even "low severity" ones, specifically to use later as an SSRF-filter bypass chain — a discipline of *not discarding* small findings.
- **Fuzzing/testing skill:** Recommends testing a *known common port* (Jira's default `:8080`) against a *random unlikely port* (`:1337`) and comparing timing/behavior as the impact-proof technique for blind SSRF — a clean, minimal-effort fuzz strategy (2 data points, not 65,000).
- **Tools used:** None specific — pure methodology.
- **SSRF trick:** Same-domain trust bypass via chained open redirect.
- **Mindset lesson:** Google `"<company> SSRF AWS"` before you start — prior disclosed reports on the *exact same target* are free, target-specific training data.

---

## 10. Full-Read SSRF via `url=` Param — DoD (Hacktivity)
**Source:** HackerOne Reports (DoD VDP) · [#1624140](https://hackerone.com/reports/1624140) · [#1623685](https://hackerone.com/reports/1623685)

- **What happened:** A `download-url` endpoint with zero filtering — raw metadata URL worked immediately.
- **Recon/discovery skill:** None needed — included specifically as the calibration case for what "no defenses at all" looks like next to the bypass-heavy write-ups above.
- **Fuzzing/testing skill:** None needed.
- **Tools used:** Browser/Burp Repeater only.
- **SSRF trick:** None — the lesson *is* "try it raw first."
- **Mindset lesson:** A meaningful fraction of real SSRF findings, especially on government/enterprise VDP programs, need zero bypass technique. Don't over-engineer your first attempt.

---

## 11. Sunshine CTF 2025 — Header Fuzzing to Unlock an SSRF Tool
**Author:** kokhant · [Medium](https://khantnyeinko.medium.com/sunshine-ctf-2025-writeups-web-misc-and-forensics-challenges-6339e6d7b72b)

- **What happened:** A `/fetch` endpoint (an SSRF-testing feature) returned 403 "missing or incorrect SSRF access header." `robots.txt` hinted a specific header was required, but not its name.
- **Recon/discovery skill:** Checked `robots.txt` for hints before doing anything else — a habit worth having on *every* target, CTF or real-world.
- **Fuzzing/testing skill:** This is the clearest **header-name fuzzing** example in this whole set — built a custom header wordlist and ran:
  `ffuf -w headers.txt -u 'https://target/fetch' -H 'FUZZ: true' -fc 403`
  filtering out the known 403 response so only headers that changed behavior showed up. Two hits came back (`Allow` and `allow`) — proving the app accepted the header case-insensitively.
- **Tools used:** `ffuf` for header fuzzing, a custom (LLM-assisted) header-name wordlist rather than a generic one.
- **SSRF trick:** Once past the header gate, the actual SSRF was trivial — `http://127.0.0.1/admin` returned an internal admin response directly.
- **Mindset lesson:** This is the exact "did they fuzz in a smart way" example worth studying closely: the fuzz target wasn't the SSRF payload itself, it was the **access-control gate in front of** the SSRF feature. Fuzzing headers, not just parameters or paths, is an underused technique — most hunters only fuzz the URL and body.

---

## 12. Blind SSRF Chains Research — Assetnote
**Referenced via HackTricks' SSRF Vulnerable Platforms page** ([link](https://hacktricks.wiki/en/pentesting-web/ssrf-server-side-request-forgery/ssrf-vulnerable-platforms.html))

- **What happened:** Not a single bug — a research methodology for turning a *blind* SSRF (no visible response at all) into a confirmed, high-impact finding by bouncing the request through known internal software that itself makes a second outbound request to your out-of-band listener.
- **Recon/discovery skill:** Deriving likely internal hostnames from DNS records, TLS certificates, load-balancer names, and application error messages, then guessing standard internal tool names (`jira`, `jenkins`, `grafana`, `solr`, `consul`, `redis`, `gitlab`, `argo`, `vault`, `kibana`) rather than blindly scanning the entire private IP range.
- **Fuzzing/testing skill:** "Spray platform-specific canaries instead of only probing `/` on random ports" — fuzz with *known health-check/status paths* for each guessed internal service (e.g. `/v1/status/leader` for Consul, `/_cat/health` for Elasticsearch, `/solr/admin/info/system` for Solr) rather than generic paths.
- **Tools used:** Interactsh / Burp Collaborator / webhook.site for OAST confirmation.
- **SSRF trick:** Chaining through internal platforms that themselves perform a second outbound fetch — this both confirms reachability *and* fingerprints exactly which internal software you hit, even when you can never see a direct response.
- **Mindset lesson:** When SSRF is blind, don't just try to prove "a request happened" — try to prove *which specific internal service* it reached, since that tells you what to target next.

---

## 13. Twitter/X Tip Compilation — Real Encoding & Protocol Tricks
**Compiled in:** [Book of Bug Bounty Tips — SSRF](https://gowsundar.gitbook.io/book-of-bugbounty-tips/ssrf) — original tips from multiple named researchers on X/Twitter

- **What happened:** Not one story — a running collection of one-line tips from working hunters, each a bypass discovered independently on real programs.
- **Selected skills/tricks worth internalizing:**
  - IP-encoding fuzzing: encoding only *one or two* octets of an internal IP instead of the whole address (this exact technique reportedly contributed to a $160,000 payout).
  - `http://127.1` instead of `http://127.0.0.1` — shorthand IP notation that resolves identically but confuses naive string-match filters.
  - Protocol downgrade: submitting `HTTP/0.9` and removing the `Host` header entirely has defeated several real SSRF fixes.
  - File-scheme quirks: `file:///etc/passwd` blocked, but a mixed-slash-encoding variant working instead.
  - UNC path SSRF/LFI on Windows hosts — pointing a file-loading parameter at an attacker-controlled UNC share path to capture an SMB/NTLM hash via Responder.
  - Excel/XLSX-to-PDF conversion features evaluating web-service formulas server-side — an unexpected SSRF vector inside document-conversion pipelines (same family as write-up #4's PDF renderer bug).
  - AWS Lambda-specific: with RCE/SSRF *inside* a Lambda function, requesting the local runtime invocation endpoint reveals the function's invocation data and headers.
  - ECS-specific: reading `/proc/self/environ` for a container UUID, then querying the ECS task metadata endpoint (distinct from the EC2 one) to extract the task's IAM role credentials.
- **Mindset lesson:** No single hunter invented all of these — this compilation exists *because* individual hunters tweet single tricks as they find them. Following a handful of active bug bounty researchers on X/Twitter is a legitimate, low-effort way to keep your trick list current between major write-ups.

---

# Part C — Master Cheat Sheet (Everything Above, Consolidated)

### Discovery — where to look
- Any "import from URL / Dropbox / Drive / OneDrive" feature (#1, #2)
- Webhooks / callback registration (#2)
- Media fetchers — audio/video/image/subtitle/logo/favicon URL fields (#3, #6)
- PDF/report/invoice/document/spreadsheet generators — treat as headless browsers or formula engines under the hood (#4, #13)
- Suspiciously-named endpoints surfaced during subdomain/endpoint recon — "proxify," "fetch," "logoGrabber" (#6, #7)
- `robots.txt` — check it before testing anything else, it sometimes leaks hints about hidden gates (#11)
- **Passive discovery while doing unrelated testing:** install Burp's **Collaborator Everywhere** extension so every header/param in your normal traffic gets a live OAST payload automatically — you'll catch SSRF primitives you'd never think to test directly (A8)

### Fuzzing — concrete commands worth having ready
```
# Header-name fuzzing to find hidden access-control gates in front of an SSRF feature (#11)
ffuf -w headers.txt -u 'https://target/fetch' -H 'FUZZ: true' -fc 403

# Parameter-name fuzzing to find undocumented URL-taking parameters
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
     -u "https://target.com/profile?FUZZ=http://your-oast-domain" -mc 200

# Port fuzzing an internal host once SSRF is confirmed (via Burp Intruder position or a loop)
http://127.0.0.1:§8080§   ->  payload set: numbers 1-65535, compare response time/size

# Param Miner (Burp extension) - automates the above two fuzz types together,
# using its own curated wordlists, and flags any param/header that changes server response (A8)
```
- Wordlists to keep on hand: `SecLists` (the default), `burp-parameter-names.txt` specifically for parameter fuzzing, `OneListForAll` when you're short on time and need one merged list, Jhaddix's LFI list (doubles for SSRF path-traversal-style payloads).

### Bypass techniques, roughly in order of "try this first"
1. **Try it completely raw first** — no bypass at all (#10).
2. **Redirect bypass** — test all of 301/302/303/307/308, not just one (#1, #2, #6).
3. **Alternate IP encodings** — `127.0.1.3`, `127.1`, octal (`0177.0.0.1`), decimal-integer form, partial-octet encoding, hex (`0xA9FEA9FE`), `0.0.0.0`, IPv6 `::1` (#5, #13).
4. **Chain through your own or a public open redirect** (#5, #9).
5. **Drop the URL scheme** if a filter regex-matches full URLs (#7).
6. **Protocol downgrade** — try `HTTP/0.9`, drop the `Host` header (#13).
7. **Mixed slash-encoding on `file://`** schemes (#13).
8. **Gopher protocol** once you can reach internal hosts — use `Gopherus` for Redis/Memcached/MySQL/FastCGI/SMTP/Zabbix payloads (#6).
9. **DNS rebinding** when a filter resolves-and-checks the domain once, then the app resolves it again later — a rebinding service can return a public IP on the first check and an internal IP on the actual fetch (A1).
10. **SNI-based SSRF** on misconfigured TLS-terminating proxies that use the SNI field directly as the backend address (A1).

### Confirming and proving impact when blind
- `interact.sh` or Burp Collaborator for OAST confirmation — free, self-hostable alternative if you don't have Burp Pro (#7, #12).
- Timing-based port scan: redirect to `internal-ip:port`, compare response time (#6, #9).
- Known-common-port vs random-port comparison — `:8080` (Jira) vs `:1337` (#9).
- Derive likely internal hostnames from DNS, TLS certs, load-balancer names, error messages — then spray platform-specific canary paths (`/v1/status/leader`, `/_cat/health`, `/solr/admin/info/system`) rather than blind-scanning the whole private range (#12).

### Cloud metadata endpoints, by provider
| Provider | Endpoint | Notes |
|---|---|---|
| AWS EC2 (IMDSv1) | `http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>` | Plain GET — works directly via basic SSRF (#7, A2) |
| AWS EC2 (IMDSv2) | Requires PUT + token header first | Much harder to trigger via basic SSRF — check which version is in use before assuming success (A2) |
| AWS ECS (task role) | `http://169.254.170.2/v2/credentials/` | Distinct from the EC2 endpoint — needs the container's task UUID, often read from `/proc/self/environ` (#13) |
| AWS Lambda | Local runtime invocation endpoint on port 9001 | Only reachable if you already have RCE/SSRF *inside* the function (#13) |
| GCP | `http://169.254.169.254/computeMetadata/v1/` | Normally needs `Metadata-Flavor: Google` header — test anyway, some proxies leak without it (#3); also grab an **identity token** with `?audience=` for IAP-protected backends (A2) |
| Azure | `http://169.254.169.254/metadata/instance?api-version=2021-02-01` | Normally needs `Metadata: true` header (A2) |
| EKS pod identity | Read the container-authorization-token env var, use it against the credentials-URI env var | Relevant when SSRF is combined with local file/env read inside a pod (A2) |

### After confirming SSRF, escalate beyond metadata
- Common internal admin/service ports: `8080`, `8008`, `8443`, `9090`, `9200` (Elasticsearch), `6379` (Redis), `5984` (CouchDB), `2375` (Docker), `8500` (Consul) (#8, #12).
- Internal admin panels are frequently bound only to `localhost`/`127.0.0.1` on a non-public port of the *same* application host (#8).
- Gopher-reachable services worth targeting for full RCE: Redis, Memcached, MySQL, FastCGI, SMTP, Zabbix (#6).

### Practice labs (no live target needed)
- **PortSwigger Web Security Academy** — free SSRF labs, all variants (A4).
- **CloudGoat**'s `ec2_ssrf` scenario + **Pacu** for the post-credential-theft half (#7).
- **SSRF_Vulnerable_Lab** (GitHub, `incredibleindishell`) — sample vulnerable code to run locally (A1).

### The one meta-lesson tying everything together
The single most repeated root cause across every write-up above: **validation and execution use different representations of the same input.** The app checks one form (resolved IP, decoded URL, normalized hostname) but the actual HTTP client fetches a *different* form (raw string, post-redirect target, re-encoded value). Every bypass technique in this document — IP encoding tricks, redirect chaining, scheme dropping, protocol downgrades — is a variation on exploiting that specific gap. Reading disclosed reports for *any* target, not just the one you're actively testing, keeps paying off because this exact mismatch keeps reappearing in completely unrelated codebases.

---

# Part D — Keeping This Current (since Part A/B will age)

- Follow HackTricks' GitHub repo directly (`HackTricks-wiki/hacktricks`) — the SSRF pages get PRs from active researchers regularly; watching the repo is more current than any snapshot.
- Set a recurring habit: 15-20 minutes on HackerOne Hacktivity filtered to `SSRF`, sorted by disclosure date, weekly — not to read everything, just to catch new bypasses as programs disclose them.
- Follow a handful of researchers who post one-off tips on X/Twitter (the source of Part B, entry #13) — individual tricks surface there months before they get compiled anywhere.
- Re-read the OWASP Prevention Cheat Sheet (A9) periodically — as defenses mature (more programs correctly implement allow-lists and disable redirects), the "easy" bypasses in this document will stop working, and you'll need to track what replaces them.

---
*Compiled from HackTricks, PortSwigger, PayloadsAllTheThings, The Hacker Recipes, Book of Bug Bounty Tips, YesWeHack, Medium, independent blogs, and HackerOne Hacktivity — 4 July 2026.*
