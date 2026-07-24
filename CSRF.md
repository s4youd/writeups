# CSRF Writeups — The Ones That Matter

> CSRF isn't dead — it evolved. The same-site cookie era changed the game, but new bypasses surface every month. The classic email-change → password-reset → ATO chain still pays.

## The Must-Reads

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 1 | Comprehensive CSRF Guide (2026) | chs.us | 18-section reference: 7-token harness tests per endpoint, method/content-type/header/SameSite testing, PoC generation, anti-patterns that always fail | https://chs.us/guides/csrf/ |
| 2 | CSRF — PortSwigger (Official) | PortSwigger | Token validation bypass taxonomy, SameSite bypass taxonomy, method override, double-submit cookie, cookie tossing from subdomains | https://portswigger.net/web-security/csrf |
| 3 | CSRF Bug Bounty 2026 — Complete Guide | Security Elites | 6 token bypass techniques, SameSite=Lax gaps, ATO chain methodology, "token present ≠ token validated" principle | https://securityelites.com/day-19-csrf-bug-bounty/ |
| 4 | OWASP Testing Guide — CSRF | OWASP | JSON CSRF via `enctype="text/plain"`, standard form exploitation, session management failures | https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/06-Session_Management_Testing/05-Testing_for_Cross_Site_Request_Forgery |
| 5 | The Bug Bounty Guide to CSRF | YesWeHack | Referer bypass, method override (Laravel/Symfony/Express), synchronizer token vs double-submit, SameSite defense layers | https://www.yeswehack.com/learn-bug-bounty/ultimate-guide-csrf-vulnerabilities |

## CSRF to Account Takeover (Email Change Chains)

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 6 | CSRF Account Takeover via Email Change — $1,300 Bounty | Gautam Sharma | No CSRF token on email change, no SameSite on session cookie, immediate email change with no verification, chain to password reset | https://medium.com/@gautam2001sharma17/account-takeover-via-csrf-on-the-email-change-endpoint-1-300-bounty-957e46f30b5f |
| 7 | CSRF to ATO: One POST Request — Target.com | Shah Kaif | Profile update with zero CSRF protection, response codes (111=success vs 2=exists) confirming state change, auto-submitting form with `history.pushState` | https://infosecwriteups.com/csrf-to-ato-how-i-took-over-accounts-on-target-com-with-one-post-request-3ab95112900c |
| 8 | CSRF Leading to ATO — $600 Bounty | Levi Ackerman | Hidden email field accepted by server but not shown in UI, Burp CSRF PoC generator, email change → password reset | https://levi4.medium.com/csrf-to-account-takeover-83cacd848903 |
| 9 | CSRF Account Takeover — Rejected Then Rewarded | Bhavishthakral | Third-party SaaS email change lacks CSRF, initially rejected as "third-party issue" — Red Team confirmed, Director paid discretionary bounty | https://medium.com/@bhavishthakral123/csrf-account-takeover-rejected-by-the-security-team-rewarded-by-the-security-director-d00022676d87 |

## CSRF + IDOR Chains

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 10 | CSRF + IDOR → Full ATO — $4,500 Bounty | Prabhakar | CSRF on phone update + read-only IDOR + backup contact password reset flow = critical chain. Global Master Admin Dashboard affected | https://medium.com/@psaibtech/how-i-turned-a-worthless-csrf-idor-into-a-full-account-takeover-eb8d448d7a58 |
| 11 | CSRF + IDOR: From Minor Glitches to Major Wins | Akash Ghosh | CSRF on `/updateUser` + IDOR on `user_id` parameter, step-by-step attack workflow with screenshots for report clarity | https://osintteam.blog/from-minor-glitches-to-major-wins-how-i-chained-csrf-and-idor-for-a-critical-exploit-0b110170cd9c |
| 12 | CSRF Bypass + IDOR → ATO | Omar Alzughaibi | Token capture trick: send update request, capture token, DROP request — token remains valid for reuse. XSS enumerates user_ids for IDOR | https://medium.com/@omarzzu/csrf-bypass-combined-with-idor-to-complete-account-takeover-f4995c5946d3 |

## CSRF Protection Bypass Techniques

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 13 | Complete CSRF Bypass on Django Crypto Exchange | Hacker MD | Django double-submit: any 32-char string works as both cookie and form field. Server doesn't verify against `SECRET_KEY`. Superuser creation PoC: `<input name="is_superuser" value="true">` | https://infosecwriteups.com/how-i-discovered-a-complete-csrf-protection-bypass-on-a-major-crypto-exchange-and-what-happened-10c7fc794324 |
| 14 | CSRF in 2025 — "Solved" But Still Bypassable | Vivek PS | `SameSite=None` + CORS misconfig, CSRF tokens leaked in API responses, OAuth missing `state`, `postMessage()` blind acceptance, mobile app weak CSRF | https://infosecwriteups.com/csrf-in-2025-solved-but-still-bypassable-942ca382ab77 |
| 15 | CSRF Token Validation Bypass — 7 Categories | PortSwigger | Method-dependent, token absent = bypass, cross-user reuse, non-session cookie binding, double-submit cookie, cookie tossing, format-only validation | https://portswigger.net/web-security/csrf/bypassing-token-validation |
| 16 | CSRF Bypass via JSON — €XXX Bounty | Kirollos Botros | `enctype="text/plain"` trick: `<input name='{"email":"victim@mail.com","key":"' value='"}'>` produces JSON body. `fetch()` with JSON triggers CORS preflight → fails. `text/plain` bypasses preflight | https://medium.com/@kirollosbotros16/bypass-csrf-attack-in-json-and-get-xxx-d8dea24d7c0b |
| 17 | JSON CSRF with Content-Type Confusion | Omar Essam | Server accepts `text/plain; application/json` — browser treats as safe, server parses substring. Also: empty JSON body `{}` with data in query params works | https://infosecwriteups.com/busting-csrf-the-hidden-dangers-of-json-exploited-fd4aeb4cf47e |
| 18 | Bypassing Referer-Based CSRF Defenses | Nile Okomo | `<meta name="referrer" content="no-referrer">`, domain in path (`https://attacker.com/victim.com`), query string trick, subdomain bypass for `startsWith()` checks | https://medium.com/@beingnile/day-19-even-more-and-more-csrf-39656ed27a46 |

## SameSite Cookie Bypass

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 19 | Bypassing SameSite Restrictions — Official | PortSwigger | GET bypass for Lax, client-side redirect gadgets, sibling domain compromise, 2-minute Lax grace window, OAuth cookie refresh exploitation | https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions |
| 20 | SameSite=Lax Bypass via Method Override | Bash Overflow | `_method=POST` in GET request: browser sends GET (Lax allows with cookies), server treats as POST. Laravel/Symfony/Express all support `_method` | https://osintteam.blog/samesite-lax-bypass-via-method-override-3daa328b0ee6 |
| 21 | SameSite=Lax Bypass via Cookie Refresh | PRiTi.EX | Force OAuth login → new session cookie → 2-minute window → CSRF fires. Popup: `window.onclick = () => { window.open('/social-login'); setTimeout(changeEmail, 5000); }` | https://medium.com/@pritiranjanmohanta6371/csrf-bypass-via-cookie-refresh-samesite-lax-practical-walkthrough-053b4ad77c18 |
| 22 | SameSite=Strict Bypass via Client-Side Redirect | L4V4NY4 AGR3 | `/post/comment/confirmation?postId=1/../../my-account` → path traversal in redirect destination. Client-side redirect is same-site by definition → Strict cookies included | https://osintteam.blog/lab-samesite-strict-bypass-via-client-side-redirect-fa778dd5d285 |
| 23 | SameSite Bypass via Popup Technique | ReFang | Odoo-based platform: no CSRF token, SameSite=Lax, popup form submission treated as top-level navigation → cookies sent | https://medium.com/@refang/how-i-found-a-csrf-vulnerability-that-could-take-over-student-accounts-on-an-educational-platform-6e65bc70816f |

## CSRF + OAuth Chains

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 24 | OAuth CSRF: Authorization Code Flow ATO | cyberpro151 | Facebook OAuth linking missing `state` validation, attacker's Facebook binds to victim's account, 2-click ATO | https://medium.com/@cyberpro151/oauth-csrf-exploiting-the-authorization-code-flow-for-account-takeover-f67cee914d39 |
| 25 | CSRF in Google OAuth Binding — Target.com | Rakesh Mali | `/account/social-connect/google` missing `state` validation, attacker's Google account binds to victim, one-click ATO via "Continue with Google" | https://rocky1696.medium.com/account-takeover-via-csrf-in-google-oauth-binding-target-com-6ccc40403ce0 |

## Advanced CSRF Techniques

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 26 | CSPT2CSRF — Client-Side Path Traversal to CSRF | Doyensec (Maxence Schmitt) | CSPT reroutes API requests through front-end routing, GET→POST chaining, found in Mattermost/Rocket.Chat, Burp extension released | https://blog.doyensec.com/2024/07/02/cspt2csrf.html |
| 27 | One-Click ATO via CSRF Chain with Cloudflare Email Routing | n0body | Fake CSRF tokens (`_ADVcgi`, `_sessionmarker`) present but never validated, Cloudflare Email Worker parses leaked email, JS polls for victim email, full automation | https://medium.com/@n0body1337/a-unique-case-of-one-click-account-takeover-via-csrf-chain-93ad43ceed8b |
| 28 | CSRF + Self-XSS via Akamai WAF Bypass | Kariiem Gamal | WAF bypass payload: `<details ontoggle=print() open>0dYSs3y.?</details>`, pre-auth flows have weaker WAF rules, CSRF cookie pre-fetch via popup | https://kariiem.medium.com/bypassing-an-akamai-waf-to-achieve-self-xss-via-csrf-chain-0fd28fcb9250 |
| 29 | CSRF Leading to Account Deletion | one33se7en | Multi-step flow protects first step but leaves final confirmation (`/u/user_settings?confirmation=delete-user`) unprotected. Time-limited: 15 minutes after login | https://one33se7en.medium.com/interesting-case-of-csrf-in-redacted-981dc2ba5f10 |
| 30 | Referrer Policy Bypass via HTTP Link Header Injection | Mahmoud Fawzy | CVE-2025-4664 ("Slonser Trick"): `Link` header with `referrerpolicy="unsafe-url"` forces full URL leakage via Referer, leaks admin auth token from URL | https://medium.com/@Zero-Ray/referrer-policy-bypass-via-http-link-header-injection-e9e9025bf221 |

## Key Numbers to Remember

| Metric | Value | Source |
|--------|-------|--------|
| CSRF + IDOR bounty | $4,500 | Prabhakar |
| Email change CSRF bounty | $1,300 | Gautam Sharma |
| CSRF to ATO bounty | $600 | Levi Ackerman |
| Django crypto exchange CSRF bounty | $50 (downgraded, controversial) | Hacker MD |
| JSON CSRF bounty | €XXX | Kirollos Botros |
| SameSite Lax grace window | 2 minutes | PortSwigger |
| CSRF token bypass categories | 7 per endpoint | PortSwigger |
| Top CSRF chain impact | Email change → password reset → ATO | 15+ writeups |

## CSRF Testing Checklist

```
TOKEN TESTING (all 7 on each endpoint):
□ Remove token parameter entirely
□ Submit empty token value
□ Use your own valid token (cross-user)
□ Submit random same-length string
□ Submit token = "0", "true", "null"
□ Put token in URL instead of body
□ Reuse previously valid token

METHOD TESTING:
□ POST → GET
□ _method=POST in GET request
□ X-HTTP-Method-Override header

CONTENT-TYPE TESTING:
□ application/x-www-form-urlencoded
□ multipart/form-data
□ text/plain
□ text/plain; application/json
□ enctype="text/plain" form trick for JSON

SAMESITE TESTING:
□ SameSite=None → classic CSRF works
□ SameSite=Lax → test GET state changes
□ SameSite=Lax → test method override
□ SameSite=Lax → test 2-minute grace window
□ SameSite=Strict → look for redirect gadgets
□ Check sibling subdomains

ATO CHAIN TESTING:
□ Email change endpoint exists and CSRF-vulnerable?
□ Password reset accepts changed email?
□ Backup contact method (phone/SMS)?
□ OAuth linking lacks state validation?
```

## Top Techniques Ranked by Frequency

| Rank | Technique | Frequency | Impact |
|------|-----------|-----------|--------|
| 1 | Email change CSRF → password reset → ATO | 15+ writeups | Critical |
| 2 | Token removal (parameter deleted, not validated) | 12+ writeups | High |
| 3 | SameSite=Lax bypass via GET requests | 10+ writeups | High |
| 4 | JSON CSRF via `text/plain` enctype | 8+ writeups | High |
| 5 | Cross-user token reuse | 8+ writeups | High |
| 6 | Method override (`_method=POST`) | 7+ writeups | High |
| 7 | OAuth missing `state` parameter | 5+ writeups | Critical |
| 8 | SameSite=Lax bypass via cookie refresh | 5+ writeups | High |
| 9 | SameSite=Strict bypass via redirect gadgets | 4+ writeups | High |
| 10 | CSRF + IDOR chaining | 4+ writeups | Critical |
