# IDOR & Broken Access Control Mastery Reference — Foundations + Stories + Skills Extracted

> **Same honesty as the SSRF version:** no document makes anyone "100%" done with a bug class — access control failures are the most common category in real-world breaches precisely because every application implements authorization differently, by hand, with no framework that gets it right by default. What this document gives you: the living reference layer (Part A), 13 real stories told properly — not just the payload, but the whole hunt (Part B), and a consolidated cheat sheet (Part C). Read Part A first.

---

# Part A — Foundational References (bookmark these, not just this file)

| # | Resource | Why it matters |
|---|----------|-----------------|
| A1 | **PortSwigger Web Security Academy — Access Control topic** ([link](https://portswigger.net/web-security/access-control)) | The clearest conceptual split between horizontal privilege escalation (same-level user reading/modifying another user's data) and vertical privilege escalation (a low-privilege user reaching admin-only functionality) — free labs for every variant. Read this before anything else in this document. |
| A2 | **OWASP API Security Top 10 — API1:2023 BOLA & API5:2023 BFLA** ([link](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)) | BOLA (Broken Object Level Authorization) is IDOR's formal name inside API-specific threat modeling; BFLA (Broken Function Level Authorization) is the vertical-escalation cousin — admin-only endpoints reachable by a normal user because the *function*, not just the *object*, was never access-checked. Most modern targets are API-first, so this is now more relevant than the classic OWASP Top 10 write-up. |
| A3 | **HackTricks — 403 & 401 Bypasses** ([link](https://hacktricks.wiki/en/network-services-pentesting/pentesting-web/403-and-401-bypasses.html)) | The definitive access-control-bypass catalog: HTTP verb tampering, path manipulation (`/./path`, double URL-encoding, Unicode slash tricks), header spoofing (`X-Forwarded-For`, `X-Original-URL`), protocol downgrades. This is what you run *after* finding a 403/401 you want past. |
| A4 | **PayloadsAllTheThings — Insecure Direct Object References** ([github.com/swisskyrepo/PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Insecure%20Direct%20Object%20References)) | Raw technique reference — ID-encoding variations, common parameter names to grep for, IDOR-in-headers examples. Grep-able, not meant to be read start to finish. |
| A5 | **"Book of Bug Bounty Tips" — IDOR page** ([gowsundar.gitbook.io](https://gowsundar.gitbook.io/book-of-bugbounty-tips/idor)) | Raw tip-density dump from working hunters — the page to check when you're stuck on a specific object type (files, sequential IDs, UUIDs, mobile-API duplicates of a web endpoint). |
| A6 | **YesWeHack — Hacking GraphQL Endpoints** ([link](https://www.yeswehack.com/learn-bug-bounty/hacking-graphql-endpoints)) | GraphQL access control fails in ways REST doesn't — per-resolver authorization is easy to forget, mass assignment via extra mutation fields is trivial to test for, and introspection often just hands you the whole schema. Essential once your target has moved past plain REST. |
| A7 | **Burp Suite extensions — Autorize & AuthMatrix** ([Autorize](https://portswigger.net/bappstore/9cbefe2e5b174a179869da84824))([AuthMatrix](https://portswigger.net/bappstore/1968797897a349f28e9713f65a49f8b3)) | Autorize automatically replays every request you make as a lower-privileged session and flags anything that *shouldn't* have succeeded but did — this automates the single most time-consuming manual step in access control testing. AuthMatrix does the same across a defined matrix of roles/endpoints, useful for BFLA specifically. |
| A8 | **OWASP Testing Guide — Authorization Testing chapter** ([cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Testing_Automation_Cheat_Sheet.html)) | The formal testing methodology — useful for structuring a *complete* access control test pass on a target rather than testing opportunistically and hoping you covered everything. |
| A9 | **HackerOne — "How an IDOR Vulnerability Led to User Profile Modification"** ([hackerone.com/blog](https://www.hackerone.com/blog/how-idor-vulnerability-led-user-profile-modification)) | Not a single incident — HackerOne's own data: IDOR makes up roughly 7% of all vulnerabilities reported on the platform, rising to 15% for government programs and 11% for automotive. Useful for knowing which verticals to prioritize if you're optimizing for this bug class specifically. |

---

# Part B — 13 Stories, Every Skill Extracted

Told properly — the actual hunt, not just the payload — because *how* someone got there is what transfers to your next target. Each story ends with the same five-part skill breakdown as before: recon/discovery, fuzzing/testing, tools, the access-control-specific trick, and the mindset lesson.

---

## 1. The Signup Page That Didn't Ask for a CaC Card
**Author:** Mahin VM · [zhackx.in](https://www.zhackx.in/DoD_IDOR/) · [HackerOne #969223](https://hackerone.com/reports/969223)

It was another ordinary day scoping the U.S. Department of Defense's public bug bounty program — a huge scope, a huge tech-stack sprawl, and, as the researcher put it, an easy program to get lost in. Poking around, something felt off: a signup page that *didn't* demand the usual CaC (Common Access Card) military ID the rest of the DoD's login flows required. That mismatch alone was worth pulling on. After creating an account and navigating to the settings page, the URL itself gave up nothing — no obvious `id` parameter, nothing that screamed IDOR. But clicking "Update" and catching the request in Burp changed everything: buried in the POST body was the researcher's own numeric user ID, submitted right alongside the profile fields being changed. The immediate question — *why does the server need me to tell it who I am, when it already has my session cookie?* — turned out to be exactly the right one. Swapping that ID for another account's and resending the request updated a stranger's profile without so much as a permission check. From there, changing the *email* field on someone else's account and walking straight to "forgot password" completed a full takeover — no clicks, no victim interaction, nothing.

- **Recon/discovery skill:** Noticed an inconsistency in the *authentication requirements* between two parts of the same site (one flow demanded CaC, another didn't) — a signal that this corner of the app was built differently, and differently-built corners are where authorization gaps live.
- **Fuzzing/testing skill:** None needed — the win here was reading a POST body carefully rather than only skimming the URL. A huge share of IDOR gets missed because hunters only check for `id=` in the address bar and never open the request body.
- **Tools used:** Burp Suite (Proxy + Repeater) only — no automation at all.
- **Access-control trick:** Client-supplied identity in the request body, redundant with (and untrusted against) the session — the server trusted whichever ID showed up in the POST, not the one tied to the authenticated session.
- **Mindset lesson:** Ask "why does the client need to tell the server this?" every time you see a value the server should already know (its own session owner's ID). That question alone is often the entire vulnerability.

---

## 2. Three Digits Apart — MTN Group
**Author:** theranger · [HackerOne #1272478](https://hackerone.com/reports/1272478)

The setup was almost comically simple: two throwaway accounts, one browser each. In Browser A, logged in as the "attacker" account, the researcher opened the profile update page and tried changing the address — capturing the request in Burp Repeater along the way. In Browser B, logged in as a "victim" account, the same update flow revealed the victim's numeric account ID sitting right there in the page, no extraction trickery required. Back in Repeater with the attacker's captured request, swapping in the victim's ID and changing the email field instead sent clean — no error, no rejection. From there it was a straight walk to "forgot password," using the now-attacker-controlled email, and the account was taken over without the victim ever seeing a notification or clicking anything. What made the researcher pause and actually write it up as *critical* rather than just "another IDOR" was the ID scheme itself: their two test accounts landed at 572 and 573 — sequential, three-digit, meaning the entire user base was enumerable in a tight, guessable range. A single script hitting every ID nearby could silently hijack hundreds of accounts.

- **Recon/discovery skill:** Ran the exact same flow as two separate users side-by-side specifically to compare what each request looked like — the two-browser, two-account technique is the fastest way to spot an IDOR that requires *comparison*, not just single-account probing.
- **Fuzzing/testing skill:** Noted that the account IDs were small, sequential, three-digit numbers — immediately reasoned out the entire feasible ID range (roughly 1–999 at the time) rather than treating the one victim ID as an isolated data point.
- **Tools used:** Burp Suite Repeater — no automation; this is a case where the *severity* came from reasoning about the ID space, not from actually running a script against it.
- **Access-control trick:** Sequential, low-entropy numeric IDs turning a single-victim IDOR into a mass-account-takeover primitive.
- **Mindset lesson:** When you find an IDOR, always ask what the *ID space* looks like — a UUID-based IDOR affecting one account you can prove is a very different severity conversation than a three-digit sequential ID affecting the entire user base, and the report should say so explicitly.

---

## 3. The `id` Field Nobody Meant to Leave In
**Author:** bugra · [HackerOne #950881 — Automattic/Atavist](https://hackerone.com/reports/950881)

Signing up for a fresh account on Atavist and poking around the account settings page, one request stood out immediately when captured in a proxy: changing the account's email address sent a POST request carrying an `id` parameter — set, naturally, to the researcher's own user ID. It's the kind of parameter that's easy to walk past if you're not deliberately checking every field in every body, since nothing about the *response* hinted that anything was wrong. Swapping that single value for a guessed nearby ID and resending confirmed it: the email of a completely different account changed, with no ownership check performed server-side at all. The report itself was almost apologetically short — because the bug was. And in the same spirit as the MTN find, the researcher flagged that these IDs were sequential too, meaning the exposure wasn't limited to one lucky guess; it scaled to every account on the platform.

- **Recon/discovery skill:** Treated the *settings/profile update* flow as priority-one territory — profile editing endpoints are consistently one of the highest-yield places to look for IDOR because they inherently take "whose data am I changing" as an implicit or explicit parameter.
- **Fuzzing/testing skill:** None beyond manual ID substitution — the lesson here isn't technique depth, it's discipline: check *every* parameter in *every* request against *every* feature, especially the boring, easily-overlooked "update my profile" ones.
- **Tools used:** Browser + proxy only.
- **Access-control trick:** Same class as story #1 and #2 — client-supplied ID in a body parameter, unchecked against session.
- **Mindset lesson:** The report follow-up matters too: when Automattic shipped a patch, the researcher didn't just accept "fixed" — they retested and only then confirmed closure. Never treat "we deployed a fix" as the end of your job.

---

## 4. Stripping Parameters Until the Server Told the Truth
**Author:** bugra · [HackerOne #1695454 — Automattic/Pressable](https://hackerone.com/reports/1695454)

This one started as a low-impact IDOR and became critical purely through refusing to stop at "confirmed." Pressable's API-application management page let users create their own API apps; intercepting the update request revealed an `application[id]` parameter. Swapping it for a second test account's application ID and resending moved that API app into the attacker's account — a real bug, but a shallow one: at worst it just yanked an app away from its owner, annoying but not devastating. Rather than reporting that and moving on, the researcher kept pulling at it: what if the request were stripped down further? Removing every parameter *except* `application[id]` and the CSRF token, then resending, triggered a validation error — "Name must be provided." But the error page, in the process of complaining, rendered and printed the *entire* application page for that ID anyway, Client ID and Client Secret included. That single escalation step turned "I can steal your API app" into "I can extract every credential associated with your API integration and act as you everywhere that integration is trusted."

- **Recon/discovery skill:** Located the bug in an "API application management" feature — a category of settings page many hunters skip because it doesn't look consumer-facing or high-value at first glance.
- **Fuzzing/testing skill:** Deliberately fuzzed the request by *removing* parameters rather than adding or changing them — an underused escalation technique: stripped-down, intentionally-broken requests often produce verbose error pages that leak more than the "successful" request path ever would.
- **Tools used:** Burp Repeater — the escalation was pure manual creativity, not automation.
- **Access-control trick:** Verbose error-handling as an information-disclosure amplifier on top of an otherwise low-severity IDOR.
- **Mindset lesson:** Confirming a bug is the *start* of the investigation, not the end. The instinct to ask "can I make this worse" — including via ugly, malformed, "this should never happen" requests — is what separates a $50 low-severity duplicate from a real payout.

---

## 5. The Session That Didn't Check Itself
**Author:** (independent researcher) · [HackerOne #3154983 — Mozilla](https://hackerone.com/reports/3154983)

Firefox Accounts' `/v1/account/destroy` endpoint existed to let a signed-in user delete their own account — a legitimately dangerous action, which usually means it gets *more* scrutiny during development, not less. And yet: the endpoint accepted an email address as the target of deletion, authenticated only by "are you logged in via SSO at all," never checking whether the *specific* session making the request actually belonged to the account named in that email field. For any attacker who also happened to be an SSO user themselves, that gap meant naming any other SSO-registered account's email was sufficient to delete it — a fully authenticated, fully "legitimate-looking" request destroying someone else's account entirely.

- **Recon/discovery skill:** Specifically targeted a *destructive* action (account deletion) rather than a read/data-exposure endpoint — destructive endpoints are worth disproportionate testing time because their impact ceiling is inherently higher (deletion and financial actions almost always outrank data-leak IDORs in severity).
- **Fuzzing/testing skill:** Tested the *identity binding* of the session specifically — asked "does this endpoint check that I am who I claim to be, or just that I'm logged in as *someone*?" — a distinct and often-forgotten authorization check separate from plain authentication.
- **Tools used:** Not detailed publicly, but conceptually requires nothing beyond a proxy and two SSO-registered test accounts.
- **Access-control trick:** Authentication-authorization conflation — the endpoint checked "is this request authenticated" and stopped there, never checking "is this authenticated session authorized to act on *this specific* target object."
- **Mindset lesson:** Every time an endpoint takes a target identifier (email, ID, username) as a parameter *and* you're independently authenticated, explicitly test whether those two facts are ever cross-checked against each other. Very often they aren't.

---

## 6. A Typical Tuesday Night That Turned Into $12,500
**Author:** Krishna Kumar · [InfoSec Write-ups](https://infosecwriteups.com/graphql-security-how-i-found-and-exploited-critical-idor-and-authorization-bypass-in-a-42ab78e13642)

Coffee brewing, scrolling HackerOne for a next target on a slow Tuesday night — the kind of unremarkable start most real bugs actually have. A private fintech program, "FinanceFlow," caught the eye specifically because its scope explicitly listed a GraphQL API endpoint — and most hunters, as the researcher notes, skip straight past GraphQL toward familiar REST endpoints, which is exactly why it was worth the extra attention. Introspection turned out to be fully enabled, handing over the complete schema — every query, mutation, and type the API supported, effectively free documentation of the entire attack surface with zero guessing required. From there, two separate weaknesses stacked together: first, an IDOR in a transaction-lookup query where the object ID wasn't validated against the requesting user's ownership, letting one account read another's financial transaction records outright. Second — and this is what pushed severity from "notable" to "critical" — GraphQL's batching feature let dozens of these unauthorized lookups be packed into a *single* HTTP request, meaning the exposure wasn't a slow one-ID-at-a-time leak but a request that could exfiltrate large swaths of other users' financial data in one shot. The combination scored a 9.1 CVSS and a $12,500 payout.

- **Recon/discovery skill:** Deliberately chose a target *because* it listed GraphQL in scope, specifically reasoning that GraphQL gets under-tested relative to REST — picking targets based on where competition is thin, not just where the program pays well.
- **Fuzzing/testing skill:** Used GraphQL introspection as a discovery step — effectively "fuzzing for free" since introspection, when enabled, hands over the full schema rather than requiring blind endpoint guessing.
- **Tools used:** GraphQL introspection queries (via a GraphQL client or Burp's GraphQL-aware extensions), manual query crafting to test object-level authorization on the transaction resolver.
- **Access-control trick:** Classic BOLA in a GraphQL resolver, escalated by **batch query abuse** — the same underlying flaw, but GraphQL's request-batching feature turned single-record exposure into bulk exfiltration.
- **Mindset lesson:** When a program lists a GraphQL endpoint in scope, that's a direct signal worth acting on — most hunters' unfamiliarity with GraphQL is exactly the gap to exploit as a hunter, not a reason to avoid the target.

---

## 7. The Base64 String That Decoded Into a Bypass
**Author:** Omar ElSayed · [Medium](https://medium.com/@bxrowski0x/how-to-find-idors-like-a-pro-158cf23baf23)

The target was a collaboration platform — organization admins could create groups and invite users as collaborators. Adding their own account as a collaborator to a group they had no business being in was the opening move, followed by the obvious next step: capturing that invite request in Repeater and swapping the invited user's ID for someone who'd never been invited at all. The response came back `204 No Content` — technically "nothing to show you," but suspiciously identical to what a *successful* invite also returned, which was enough to keep pulling on the thread rather than write it off as a dead end. That instinct paid off across the rest of the app: three more IDORs surfaced in short order across posts (read/write) and the approvers/groups features. But the sharpest find was a separate profile-update endpoint, where changing the target `user_id` alone returned a clean "User not authenticated" error — a real, working block, unlike the collaborator flow. Instead of stopping there, the researcher went looking for *why* the check existed at all, and found a strange authentication header riding along in the request. Decoding it — plain Base64 — revealed the researcher's own account ID sitting inside, formatted as `id:id`. Re-encoding the *victim's* ID in that same `victim_id:victim_id` shape and swapping it into the header (leaving the URL's `user_id` alone) sailed straight past the check that had just blocked the naive attempt. From there: change the victim's email, walk to password reset, full account takeover.

- **Recon/discovery skill:** Treated a `204 No Content` response with suspicion rather than dismissal — an ambiguous or "empty" response that matches what a legitimate success case would also look like is a signal to dig deeper, not evidence of failure.
- **Fuzzing/testing skill:** Noticed a header that looked like meaningless noise and manually decoded it instead of ignoring it — a habit worth having on *every* unfamiliar header value: try Base64-decoding anything that looks like it might be encoded (padding characters, character-set patterns) before assuming it's opaque.
- **Tools used:** Burp Repeater plus a Base64 decoder (built into Burp's own inspector panel, or CyberChef) — no automation, pure manual curiosity.
- **Access-control trick:** Authorization check enforced against a *header-encoded* identity value instead of (or in addition to) the URL parameter — meaning a naive tester who only ever touches the obvious `user_id` in the URL hits a dead end, while the real bypass sits one layer deeper in a header most people don't think to inspect, let alone decode.
- **Mindset lesson:** A blocked first attempt at IDOR doesn't mean the endpoint is safe — it might mean the *real* identity check lives somewhere less obvious than the parameter you happened to try first. Map every place an identity value could plausibly be hiding (URL, body, header, cookie) before concluding a feature is properly protected.

---

## 8. Two Bugs, One Chain, City-Mobil
**Source:** Deteact · [blog.deteact.com](https://blog.deteact.com/account-takeover-via-idor/)

Not every account takeover comes from a single clean bug — sometimes it takes noticing that two separate, individually-modest flaws sit right next to each other. On a ride-hailing platform's Enterprise-tier product, organizations could group multiple projects together, and connecting a project to an Enterprise-tier group upgraded that project's own subscription status. The first crack: broken access control let one organization view *other organizations'* projects and managers entirely — a real bug on its own, enough to leak confidential business data, but not yet a takeover of anything. The second crack showed up while poking at an unrelated feature: updating a user's profile allowed attaching a "manager" (a lower-privileged role tied to a project) purely by its numeric identifier — no check that the manager being attached actually belonged to the requesting account's own organization at all. Individually, bug two is a fairly ordinary "assign-by-ID with no ownership check" IDOR. But chained with bug one — which had just handed over the IDs of *other organizations'* managers on a platter — the combination let an attacker enumerate a target manager's ID via the access-control flaw, then attach that manager to their own account via the IDOR, inheriting that manager's project-level privileges and access wholesale.

- **Recon/discovery skill:** Kept a running mental inventory of "interesting IDs I've seen leaked, even from bugs that don't seem exploitable on their own" — the access-control flaw wasn't valuable *by itself* for takeover purposes; it became valuable the moment a second, unrelated bug needed exactly the kind of ID it had been leaking.
- **Fuzzing/testing skill:** Tested the "attach an existing manager by ID" feature specifically for ownership validation — a pattern worth generalizing: any feature that lets you *attach* an existing object (a manager, a device, a payment method, an integration) by referencing its ID deserves an ownership check, and frequently doesn't get one.
- **Tools used:** Standard proxy-based manual testing across two separate features, cross-referenced by hand.
- **Access-control trick:** Chaining a data-exposure access-control bug with a separate attach-by-ID IDOR — neither bug alone reaches "account takeover" severity, but combined they do.
- **Mindset lesson:** Don't evaluate every finding in total isolation. A "medium" information-disclosure bug that leaks IDs is worth revisiting the moment you find *any* other feature that trusts an ID without checking ownership — the two may combine into something much bigger than either alone.

---

## 9. The Password That Was Never Set
**Source:** Payatu · [payatu.com/blog](https://payatu.com/blog/a-journey-idor-to-account-takeover/)

Most IDOR hunts start at the profile-update page, and this one nearly ended there too — the researcher notes explicitly that the update-profile section is usually hardened precisely *because* it's the first place everyone looks, given it handles PII directly. The actual opening came from an unusual signup flow instead: this application let users create an account without setting a password up front, deferring that step to a later "finish setup" stage after first login. That gap in the normal login-first assumption was the whole story. The change-email feature sent a confirmation link containing a `UserID` parameter — swap that value for a victim's ID, and the confirmation link, when clicked by *anyone*, silently changed the victim's email server-side with no visible change on the victim's own screen. Because the victim account had never had a password set, there was no "enter your current password to confirm" friction standing in the way at all — the researcher simply logged in as the victim (no password needed yet, by design) once the email swap landed, and walked straight through to full account control.

- **Recon/discovery skill:** Explicitly avoided fixating only on the "obvious" profile-update page and instead mapped the *entire* signup and account-lifecycle flow, including edge cases like "what if a user never finishes account setup." Unusual account states (no password set, unverified email, partial registration) are disproportionately likely to have gaps, because developers test the *happy path* far more than the incomplete one.
- **Fuzzing/testing skill:** Tried converting a blocked `POST` into a `GET` on a separate endpoint elsewhere in the same hunt — a simple, always-worth-trying move: a 403 on `POST /api/user/1234` doesn't mean `GET` is equally protected.
- **Tools used:** Proxy-based manual testing; notes checking for parallel mobile-API endpoints (appending `.json` to a path, or hitting the same object via a documented mobile API) as an alternate route to the same data when the web UI is well-defended.
- **Access-control trick:** IDOR in an email-change confirmation link, escalated to full account takeover specifically because the target account had no password set yet — removing the one friction point that would normally block "login as victim" even after email takeover.
- **Mindset lesson:** "Take your time to understand the application, and eventually you will find the IDOR" — patience with the *full* application flow, not just the standard checklist of "obvious" endpoints, is explicitly what this write-up credits the find to.

---

## 10. Bypassing 403s Like a Pro
**Author:** Shrirang Diwakar · [Medium](https://shrirangdiwakar.medium.com/bypassing-403s-like-a-pro-2-100-broken-access-control-66beef4afa8c)

An IoT service-delivery platform, and a routine directory-enumeration pass with standard dirbuster wordlists — the kind of recon step that's easy to run on autopilot and stop paying attention to. One result stood out from the noise: `/console`, returning a clean 403 Forbidden. Rather than treating that as a locked door and moving on, the researcher treated it as a lead. Manual path-manipulation attempts followed a familiar checklist: case variations, trailing slashes, double slashes, `/./` and `/..` traversal-style tricks, semicolon injection into the path, alternate extensions. Most of it, run by hand, was slow going — so the researcher switched to the **403 Bypasser** Burp extension to automate the whole battery of known tricks against the captured request in one pass. The extension came back clean: prefixing the path with `/./` — turning `/console` into `/./console` — sailed straight through and landed on a full internal system console manager, no authentication required at all. From there, every subsequent request to that manager got the same `/./` prefix bolted on by hand, turning a one-off bypass into a fully browsable internal admin panel.

- **Recon/discovery skill:** Ran ordinary directory brute-forcing (dirbuster wordlists) as a matter of habit — and paid close attention to 403 responses specifically, rather than filtering them out as "not interesting" the way many hunters configure their tools to do by default.
- **Fuzzing/testing skill:** Systematically tried the *entire* known category of path-manipulation bypass tricks — case changes, slashes, traversal characters, semicolons, extension swaps — rather than trying one or two and giving up, then automated that same battery with a purpose-built extension once manual testing got repetitive.
- **Tools used:** Standard directory-brute-force wordlists, Burp Suite, the **403 Bypasser** extension (by Gil Nothmann) specifically to automate the bypass battery instead of hand-testing each variant.
- **Access-control trick:** Path-normalization mismatch between the access-control layer and the actual request router — `/./console` is functionally identical to `/console` to the backend router, but the security check in front of it was pattern-matching the literal string and missed the equivalent form.
- **Mindset lesson:** A 403 is a *lead*, not a dead end — access control implemented as a simple string-match check in front of a path is extremely common and extremely fragile, and the fix (Burp match/replace rules prefixing every request with the working bypass) is worth setting up the moment you confirm one, so you don't have to remember to add it by hand on every subsequent request.

---

## 11. Recon First, Bypass Second — A Fortune 100 Financial Institution
**Author:** Damaidec · [Medium](https://medium.com/@damaidec/403-bypass-on-a-fortune-100-financial-institution-p3-156d33bc6ed)

A wide `*.target.com` wildcard scope is an invitation, and the researcher treated it as one — starting, as always, with `subfinder` to enumerate the full subdomain footprint before touching anything else, then `httpx` to sort out which of those subdomains were actually alive and worth spending time on. That ordinary recon pass surfaced one subdomain that looked distinctly more interesting than the rest of the pile — unusual enough in naming and structure to be worth a closer look ahead of the others. Testing there specifically for access control gaps, the researcher pulled from a learned technique picked up earlier at a CTF (the "Ambassador World Cup" event, explicitly credited in the write-up as the origin of the method) and combined it with the standard 403-bypass toolbox: HTTP verb tampering across GET/POST/PUT/DELETE/TRACE, alternate directory paths, and header fuzzing, cross-referencing against known automation like `byp4xx`, `bypass-403`, and `forbiddenpass` for anything manual testing missed. The combination landed the researcher's first P3-severity bug — found, notably, within just a few hours of starting, specifically because the recon phase had already done the work of narrowing a huge wildcard scope down to the one subdomain worth focusing on.

- **Recon/discovery skill:** Ran subdomain enumeration (`subfinder`) and live-host probing (`httpx`) as the mandatory first step on a wildcard scope, before any vulnerability-specific testing at all — a wide scope without recon-driven prioritization is a huge time sink; narrowing it first is what made "a few hours" possible.
- **Fuzzing/testing skill:** Systematically ran HTTP verb tampering (GET/POST/PUT/DELETE/TRACE and beyond) against the identified 403 endpoint, alongside path and header fuzzing — treating "which HTTP method" as its own fuzzable dimension, not just URLs and parameters.
- **Tools used:** `subfinder`, `httpx` for recon; `byp4xx`, `bypass-403`, `forbiddenpass` for automated bypass-technique coverage.
- **Access-control trick:** A verb-tampering or path-based bypass (specific technique redacted by the researcher per disclosure policy) — the meta-lesson is the toolbox and process, not one specific payload.
- **Mindset lesson:** A technique learned once, in an unrelated context (a CTF), became a repeatable real-world skill — treat CTF/lab techniques as a permanent addition to your live-hunting toolkit, not a one-time academic exercise.

---

## 12. A DELETE Request That Reached Across Accounts
**Author:** Medusa · [InfoSec Write-ups](https://infosecwriteups.com/idor-leads-to-unauthorized-deletion-how-i-earned-500-in-bug-bounty-335bd6a2c75d)

A smaller bounty than most of the stories above, and worth including specifically because most real IDOR write-ups skip past *deletion*-class bugs in favor of the flashier read/write-data or account-takeover stories. During a routine hunt, a `DELETE` request targeting a specific resource ID surfaced during normal proxy interception. Testing whether that resource ID could be swapped for one belonging to a completely different account — and, notably, a different tenant within the same multi-tenant system — confirmed the flaw: role-based access control was actively enforced on the *frontend* (the UI simply didn't expose a delete button for resources you didn't own), but the backend endpoint itself performed no ownership check at all, trusting whatever resource ID showed up in the request regardless of who sent it.

- **Recon/discovery skill:** Paid attention to `DELETE` requests specifically during proxy interception, rather than only cataloguing `GET`/`POST` traffic — destructive HTTP verbs are frequently tested less rigorously by developers than read/write ones, on the (often false) assumption that the UI alone prevents users from ever triggering them against the wrong target.
- **Fuzzing/testing skill:** Cross-tenant ID substitution specifically — testing not just "another user's ID" but "another *tenant's* ID entirely" in a multi-tenant system, a distinction that matters because tenant isolation is sometimes checked at a different layer (or not at all) compared to same-tenant user isolation.
- **Tools used:** Standard proxy interception and manual request replay.
- **Access-control trick:** Frontend-only authorization — the UI hid the delete option, but the backend endpoint enforced nothing, meaning the "access control" that existed was purely cosmetic.
- **Mindset lesson:** Never assume a backend check exists just because the frontend UI doesn't expose the action — the only way to know is to construct the request directly and send it yourself, bypassing the UI entirely.

---

## 13. Top 25 IDOR Reports — Pattern Compilation
**Author:** Cristian Cornea · [Medium](https://link.medium.com/vsGpIQS3Bbb)

Not a single hunt — a curated round-up of 25 disclosed IDOR reports, organized to teach *shape recognition* rather than any one incident. The compilation draws a distinction worth internalizing on its own: **Generic IDOR** (the exploited result is directly visible in the response — you change an ID, you see someone else's data right there) versus **Blind IDOR** (the action succeeds — a record gets modified or deleted — but the response gives you nothing to look at directly, so you have to confirm impact indirectly, the same way you would with blind SSRF). It also catalogs IDOR by *object type* rather than by parameter location alone: reference-to-objects (a bank account or order ID swapped in a URL), and reference-to-files (a sequentially-numbered log or document, like `example.com/1.log`, `example.com/2.log`, enumerable purely by incrementing a filename with no ID parameter involved at all).

- **Recon/discovery skill:** Frames "gather every API request the application makes" as the mandatory first step before any IDOR testing begins — you cannot test what you haven't first catalogued.
- **Fuzzing/testing skill:** N/A directly (aggregator), but implicitly recommends brute-forcing sequential file-style references (`1.log`, `2.log`, `3.log`) the same way you would numeric ID parameters — file-reference IDOR is easy to forget because it doesn't look like a "parameter" at all.
- **Tools used:** N/A (reference piece).
- **Access-control trick:** The Generic vs. Blind IDOR distinction, and the file-reference IDOR category specifically — a shape many hunters never think to check because it doesn't present as a typical `id=` parameter.
- **Mindset lesson:** Read compilation pieces like this one to build a mental catalog of *shapes* — 25 cases at once teaches pattern recognition in a way that no single deep story can.

---

# Part C — Master Cheat Sheet (Everything Above, Consolidated)

### Discovery — where to look
- Profile/settings update endpoints — the single highest-yield category across nearly every story above (#1, #3, #9).
- Any feature that lets you *attach* an existing object by ID (a manager, a device, an integration, a payment method) — check for ownership validation specifically (#8).
- Destructive HTTP verbs (`DELETE`, and state-changing `PUT`/`PATCH`) — frequently under-tested by developers relative to `GET`/`POST` (#12).
- Unusual account states — signup flows that defer password-setting, unverified accounts, partial registration — edge cases get tested far less than the happy path (#9).
- GraphQL endpoints specifically, if listed in scope — most hunters skip them, which is precisely the opportunity (#6).
- Sequentially-numbered file references (`1.log`, `2.log`) — a category of IDOR that doesn't look like a parameter at all (#13).
- Any endpoint returning an ambiguous "success-shaped" response (`204 No Content`, empty `200`) to a request you're not sure worked — don't dismiss it as failure without testing further (#7).

### Testing/fuzzing — concrete techniques
- **Two-account, two-browser comparison** — log in as two separate test accounts simultaneously and diff every request/response between them; this is the fastest way to spot an IDOR that only shows up in comparison (#2).
- **Parameter stripping, not just substitution** — remove parameters (down to the bare minimum required) rather than only changing values; verbose error responses from malformed requests frequently leak more than the "successful" path (#4).
- **HTTP verb tampering** — test every relevant endpoint with GET/POST/PUT/DELETE/PATCH/TRACE; a 403 on one verb says nothing about the others (#9, #11).
- **Decode anything that looks encoded** — Base64, hex, URL-encoding in headers and cookies, not just URL/body parameters; authorization checks sometimes key off a value hidden in a header nobody thinks to inspect (#7).
- **Cross-tenant ID substitution in multi-tenant systems** — test not just "another user" but "another tenant entirely"; isolation logic is sometimes implemented at a different layer than same-tenant user isolation (#12).
- **Check for parallel API surfaces** — a well-defended web UI often has a less-scrutinized mobile API exposing the same objects (appending `.json`, hitting `/api/v1/...` directly) (#9).
- **Reason about the ID space itself, not just one instance** — sequential/low-entropy IDs turn a single-account IDOR into a mass-enumeration primitive; say so explicitly in your severity assessment (#2, #3).
- **GraphQL-specific:** run an introspection query first (often enabled by default) to get the full schema for free; test mutations for **mass assignment** by adding fields (like a `role` field) the client shouldn't be allowed to set; test whether authorization propagates into nested resolvers, not just the root query; check whether **batching** lets you multiply a single-object IDOR into bulk exfiltration (#6, A6).
- **403/401-specific:** run the full path-manipulation battery — case changes, trailing/double slashes, `/./` and `/..`, semicolon injection, alternate extensions, Unicode slash tricks — either by hand or via the **403 Bypasser** Burp extension or standalone tools (`byp4xx`, `bypass-403`, `forbiddenpass`) (#10, #11, A3).

### Tools worth having ready
- **Autorize** (Burp extension) — automatically replays your traffic as a lower-privileged session and flags anything that shouldn't have succeeded. The single biggest time-saver for systematic access control testing (A7).
- **AuthMatrix** (Burp extension) — role × endpoint authorization matrix testing, useful specifically for BFLA (vertical privilege escalation across defined roles) (A7).
- **403 Bypasser**, **byp4xx**, **bypass-403**, **forbiddenpass** — automate the standard 403/401 bypass battery instead of testing each variant by hand (#10, #11, A3).
- **ffuf** — for ID-space fuzzing (`ffuf -w ids.txt -u https://target/api/user/FUZZ`) and for finding parallel API endpoints via path fuzzing.
- **CyberChef** or Burp's built-in decoder — for decoding suspicious header/cookie values on sight (#7).
- **A GraphQL client (or Burp's GraphQL-aware tooling)** — for introspection queries and crafting batched/aliased requests (#6, A6).

### The two-tier severity framing worth using in every report
1. **Is it Generic or Blind?** Can you see the exploited data directly in the response, or only infer success indirectly? (#13)
2. **What does the ID space actually look like?** Sequential/low-entropy IDs affecting the whole user base is a fundamentally different severity than a single hard-to-guess UUID affecting one account — always characterize this explicitly rather than leaving it implicit (#2, #3).

### The one meta-lesson tying everything together
Access control is the one category where the *application's own business logic* has to be understood correctly for the bug to even be visible — unlike XSS or SQLi, there's no universal payload that reveals an IDOR. Every story above starts with genuinely understanding what a feature is *supposed* to do (who should be able to see this, attach that, delete this) before the "just change the ID" step becomes meaningful at all. The technical trick is almost always trivial once you've correctly identified where an ownership check *should* exist but doesn't — which is why patient, full-application mapping (story #9's explicit lesson) consistently outperforms rushing straight to parameter-tampering on the first endpoint you see.

---

# Part D — Keeping This Current

- Re-run the **Autorize** extension against every new feature you test, as a standing habit, not a one-off tool you reach for occasionally — it catches the boring, easy-to-miss authorization gaps automatically while you focus manual attention on business-logic-specific chains like story #8.
- Watch HackerOne Hacktivity filtered to `IDOR` and separately to `Broken Access Control`/`BOLA` weekly — the specific *shape* of the bug (sequential IDs, header-hidden identity, GraphQL batching) evolves as applications adopt new frameworks.
- As more targets move to GraphQL and gRPC-style APIs, expect the *shape* of access control bugs to shift away from classic URL-parameter IDOR toward resolver-level and per-field authorization gaps — A6 and story #6 are the leading edge of that shift, worth revisiting periodically.
- Re-read OWASP's API Security Top 10 (A2) when a new edition ships — BOLA and BFLA rankings and examples get refreshed as the working group tracks how real-world API authorization failures are evolving.

---
*Compiled from PortSwigger, OWASP, HackTricks, PayloadsAllTheThings, Book of Bug Bounty Tips, YesWeHack, Medium, InfoSec Write-ups, Payatu, Deteact, and HackerOne Hacktivity — 4 July 2026.*
