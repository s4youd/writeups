# WordPress Bug Bounty Hunting — Writeups, Methodology & Tips

A curated collection for hunting WordPress (core + plugins + themes), since WP powers a huge chunk of the web and runs on PHP + MySQL — same stack, tons of attack surface.

---

## 1. Why WordPress is a good target

- 60,000+ plugins in the official repo alone, many written by junior/solo devs → weak input validation is common.
- Attack surface = **Core** (rarely vulnerable, heavily audited) + **Plugins** (biggest source of bugs) + **Themes** (less common, but happens).
- Programs like **Wordfence** and **Patchstack** pay specifically for plugin/theme vulns — separate from a company's main HackerOne/YesWeHack program if they happen to run WordPress.

---

## 2. Recon: confirm it's WordPress & fingerprint the stack

1. **Confirm CMS** — Wappalyzer, or manually:
   - `/wp-login.php`, `/wp-admin/`, `/wp-content/`, `/wp-json/` all responding
   - View source for `wp-content/themes/` or `wp-content/plugins/` paths
   - `curl -I target.com` → check `X-Powered-By`, generator meta tag on homepage (`<meta name="generator" content="WordPress X.X">`)
2. **Enumerate plugins & themes** (this is where the money is):
   - `wpscan --url https://target.com -e vp,vt,u` (vulnerable plugins, vulnerable themes, users)
   - Manually browse `/wp-content/plugins/<name>/readme.txt` for version numbers
   - Check JS/CSS asset query strings (`?ver=1.2.3`) — reveals plugin/theme versions even if WPScan can't enumerate them
3. **Enumerate users**:
   - `/wp-json/wp/v2/users` (REST API often leaks usernames)
   - `/?author=1` redirect trick
   - `xmlrpc.php` (also a target itself — see below)
4. **Version-match against CVEs**: once you know plugin name + version, check the [WPScan Vulnerability Database](https://wpscan.com/), [Patchstack DB](https://patchstack.com/database), and [Wordfence Intelligence](https://www.wordfence.com/threat-intel/) for known n-day bugs — huge % of real-world WP compromises are just unpatched known CVEs.

**Tools worth having:** WPScan, CMSMap, wp2shell-style scanners, Burp, and just plain source-diffing plugin versions on `plugins.svn.wordpress.org` to find what a patch actually fixed (classic technique: diff vN vs vN-1, the diff often *is* the vulnerability).

---

## 3. Where to actually test (attack surface checklist)

| Area | What to try |
|---|---|
| `wp-admin/admin-ajax.php` | Any custom AJAX action registered by a plugin (`wp_ajax_*` / `wp_ajax_nopriv_*`) — the `nopriv` ones are unauthenticated and a top source of bugs |
| REST API (`/wp-json/...`) | Custom plugin REST routes, missing `permission_callback`, IDOR on object IDs |
| `xmlrpc.php` | SSRF via `pingback.ping`, brute-force amplification, still enabled on many sites |
| File upload fields (forms, media, import tools) | Extension bypass → RCE via webshell (see Section 5) |
| Shortcodes & widgets | Reflected/stored XSS — shortcode attributes rarely sanitized properly |
| `$wpdb->query()` / `$wpdb->prepare()` misuse | SQLi — look for direct string concatenation instead of `%s`/`%d` placeholders |
| Plugin settings pages / import-export features | CSRF (missing nonce checks), SSRF (URL fetchers), PHP object injection (`unserialize()`) |
| Login / password reset flows | Auth bypass, user enumeration, weak reset tokens |
| Multisite installs | Cross-site privilege escalation between subsites |
| `wp-config.php` backups, `.git`, `debug.log` | Info disclosure via misconfigured exposed files |

---

## 4. Common vulnerability classes in WordPress (with root cause)

- **SQL Injection** — plugin builds a raw query with `.$_GET[...].` instead of `$wpdb->prepare()`. Grep source for `.$` / `".$` next to `$wpdb->` calls — this is literally how researchers find these bugs (see the Peng Zhou writeup below).
- **XSS (stored/reflected)** — shortcode attributes, comment fields, custom meta boxes in wp-admin, unescaped `echo $_GET[...]`.
- **Arbitrary File Upload → RCE** — form builders, import tools, media handlers that check extension client-side only or have a blacklist you can bypass (`.pHp5`, `.phtml`, double extensions, null byte tricks depending on server config).
- **CSRF** — missing `wp_nonce_field()` / `check_admin_referer()` on state-changing admin actions.
- **PHP Object Injection / Insecure Deserialization** — plugin calls `unserialize()` on user input (cookies, POST params).
- **Broken Access Control / IDOR** — REST routes registered without a proper `permission_callback`, or capability checks only on the UI, not the endpoint.
- **SSRF** — URL-fetching features (oEmbed, `pingback.ping` via xmlrpc, import-from-URL tools).
- **Auth bypass / privilege escalation** — logic flaws in role-checking functions, or REST endpoints that trust a client-supplied `role` parameter.

---

## 5. Practical write-ups (mixed skill level — start from the top)

**Big-picture methodology & resource hubs:**
- [Mastering WordPress Pentesting: The Ultimate Resource Guide](https://medium.com/@RaunakGupta1922/mastering-wordpress-pentesting-the-ultimate-resource-guide-423bc1e1ddef) — Raunak Gupta. Curated list of blogs, vulnerable WordPress CTF labs, tools, and common vulnerable plugins. Best single starting point.
- [HackTricks — WordPress Pentesting](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/wordpress.html) — the canonical checklist: enumeration commands, plugin/theme attack notes, common misconfig list.
- [Mastering WordPress Bug Hunting — InfoSec Writeups](https://infosecwriteups.com/mastering-wordpress-bug-hunting-a-complete-guide-for-security-researchers-3ff7ee4413a2) — breaks down Core vs Themes vs Plugins as attack surface and common vuln types.
- [Hunting WordPress Vulnerabilities — Z3DX (Medium)](https://medium.com/@ksreemosmar/hunting-wordpress-vulnerabilities-a-bug-bounty-4324c6782f1f) — walkthrough of methodology + fingerprinting.
- [Complete WordPress Bug Bounty Guide — Patchstack](https://patchstack.com/articles/wordpress-bug-bounty/) — explains the different WP-specific bounty programs (WordPress.org via HackerOne, Patchstack Alliance, Wordfence) and how reports/payouts work.
- [Splashing Around in the Shallow End — Stealthcopter](https://sec.stealthcopter.com/splashing-around-in-the-shallow-end/) — real journey of someone who reported ~300 WordPress plugin vulns for ~$27k; links to his public [wordpress-hacking GitHub repo](https://github.com/stealthcopter/wordpress-hacking) of resources and disclosed reports. **Very relevant for a beginner** — shows the actual grind and process.

**SQL Injection specific:**
- [How I Make Two SQL Injections Exploitable Under Magic Restricts in WordPress — Peng Zhou](https://medium.com/@zpbrent/how-i-make-two-sql-injections-exploitable-under-the-magic-restricts-in-wordpress-817cd60dc80a) — excellent technique: grep plugin source for `'.$` and `".$` next to `$wpdb->` calls to find raw-concatenated queries, then details how to bypass WP's built-in character filtering. Two real CVEs walked through.
- [SQL Injection in WordPress Plugin Huge IT Video Gallery — Uber HackerOne report](https://hackerone.com/reports/125932) — real disclosed report, shows exact reporting format/style.
- [Grab: SQLi in Formidable Pro plugin — HackerOne report](https://hackerone.com/reports/273946)
- [WooCommerce SQL Injection via coupon_codes param — Automattic HackerOne report](https://hackerone.com/reports/3198980) — recent (2025), shows a union-based SQLi with sleep() blind confirmation in a live admin report endpoint.
- [Top SQLi reports across HackerOne (includes multiple WordPress ones)](https://github.com/reddelexc/hackerone-reports/blob/master/tops_by_bug_type/TOPSQLI.md) — big list, several WP plugin SQLi reports mixed in with general web SQLi; good for pattern-matching report style and payloads.
- [WordPress Core `WP_Query` SQL Injection — CVE-2022-21661 (Exploit-DB)](https://www.exploit-db.com/exploits/50663) — rare Core-level SQLi, unauthenticated, good to understand even though core bugs are less common.

**File Upload → RCE specific:**
- [File Upload Bypass to RCE == $$$$ — Sagar Sajeev](https://sagarsajeev.medium.com/file-upload-bypass-to-rce-76991b47ad8f) — not WP-specific but directly applicable: 3 different extension-filter bypasses (case randomization like `.pHp5`, etc.) chained to RCE, 3 separate bounties on the *same* bug after each "fix."
- Patchstack vulnerability database entries — good for seeing real disclosed patterns:
  - [Import WP plugin — Arbitrary File Upload → RCE](https://patchstack.com/database/vulnerability/jc-importer/wordpress-import-wp-plugin-2-4-5-arbitrary-file-upload-vulnerability-leading-to-remote-code-execution-rce)
  - [Simple File List plugin — Unauthenticated Arbitrary File Upload → RCE](https://patchstack.com/database/vulnerability/simple-file-list/wordpress-simple-file-list-plugin-4-2-2-unauthenticated-arbitrary-file-upload-vulnerability-leading-to-remote-code-execution-rce)
  - [Super Forms plugin — Arbitrary File Upload → RCE](https://patchstack.com/database/vulnerability/super-forms/wordpress-super-forms-premium-plugin-4-9-700-arbitrary-file-upload-leading-to-remote-code-execution-rce-vulnerability)
- [50,000 Sites Exposed — Ninja Forms File Upload RCE (Wordfence blog)](https://www.wordfence.com/blog/2026/04/50000-wordpress-sites-affected-by-arbitrary-file-upload-vulnerability-in-ninja-forms-file-upload-wordpress-plugin/) — recent (2026) real disclosure, good root-cause breakdown (missing filename sanitization → path traversal in webroot).
- [Critical Thinking Podcast — HackerNotes Ep. 55: Popping WordPress Plugins](https://blog.criticalthinkingpodcast.io/p/hn-55-wordpress-plugins-common-design-flaws-code-review-methodology) — interview with Ram Gall (Wordfence) on common WP plugin code-review methodology and design flaws; great for source-review approach rather than pure black-box.

**Other real-world / miscellaneous WordPress bugs (from the resource-hub list above — worth reading in order):**
- "P1 Bug Hunting: Exploiting Common WordPress Vulnerabilities" — The Gray Area (Medium)
- "How to Get a Reverse Shell from Any WordPress" — System Weakness
- "Pwning WordPress Passwords" — InfoSec Writeups
- "Hacking WordPress Server Database" — System Weakness
- "Leaking WordPress CSRF Tokens" — ahussam.me
- "ATO of WordPress Website: 4-Digit Bounty in 5 Minutes" — Ritesh Gohil (Medium)
- "Error-Based SQL Injection on a WordPress Website — Extracted 150k+ User Details" — Ynoof (Medium)
- "5 Minutes, 3 Sites, 1 WordPress Vulnerability" — Markaz Gasimov (Medium) — Blind SSRF via `xmlrpc.php`'s pingback feature found on 3 separate WP sites in one evening; good example of a low-effort, repeatable bug class.

*(These last ones are aggregated from a curated resource list rather than independently verified by me — worth searching the exact title if a link ever breaks.)*

---

## 6. Beginner-friendly practice labs (before hunting live targets)

- **DVWP** (Damn Vulnerable WordPress) — GitHub, intentionally vulnerable WP install to practice locally.
- Patchstack & Wordfence monthly bounty programs — real plugins, real (small) cash, structured triage feedback — good for a beginner because the scope is "any WP plugin with X installs," so you're not stuck waiting for a specific program invite.
- WPScan's own vulnerability DB — read *closed* reports to learn what "good" looks like before you submit your own.

---

## 7. Practical tips for a beginner hunter

1. **Go for plugins, not core.** Core is audited constantly; plugins by small devs are where the real density of bugs is.
2. **Diff plugin versions.** Download vN and vN-1 from the SVN repo, diff them. If a "security fix" changelog entry exists, the diff often shows you exactly what the bug was — great for learning patterns even if you can't reproduce the same bug elsewhere.
3. **Grep for anti-patterns in source** if you have plugin code (most are open-source in the WP repo): `$wpdb->query(` without `prepare`, raw `unserialize(`, `move_uploaded_file(` without extension whitelisting, `wp_ajax_nopriv_` handlers doing sensitive things.
4. **Unauthenticated `nopriv` AJAX/REST endpoints are gold** — always check what they do without a login.
5. **Read a handful of *closed* Patchstack/Wordfence reports** for report-writing style before submitting your first one — clear repro steps matter as much as the bug itself for getting paid quickly.
6. **Track install counts.** Wordfence generally only accepts plugins with 50k+ installs; Patchstack accepts wider scope — pick your target program accordingly.
