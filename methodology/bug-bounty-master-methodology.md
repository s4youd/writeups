# The Complete Bug Bounty Methodology — Recon, Testing, Reporting, and AI

> This is a working methodology, not a reading list. Everything a senior hunter would actually pull from Haddix's recon methodology, zseano's application-analysis discipline, WAHH's authorization/session models, HackTricks' bypass catalogs, PayloadsAllTheThings, OWASP's API risk model, and 2026's field data on AI-assisted hunting has been extracted and rewritten directly into the steps below. Use it as a checklist per target, top to bottom.

---

# 1. Before You Touch a Tool: Picking and Scoping a Target

Most wasted weeks in bug bounty happen here, before recon even starts.

- Read the policy page twice and write down, verbatim, what's in scope, out of scope, and any testing restrictions (rate limits, no-automated-scanning clauses, forbidden test accounts). Every tool you run afterward should read from this list — never test from memory.
- Favor programs with recently expanded scope (a new wildcard domain, a freshly announced API, a recent acquisition folded in) — that surface hasn't been picked over yet. A 5+ year old program with a huge, static scope requires either deep specialization or a lot of patience for a first bounty; a newly-launched or recently-expanded program rewards generalists faster.
- Private invitations are worth prioritizing over hopping into a brand-new public program: less competition, faster triage turnaround, and companies that invite researchers directly tend to have more mature security teams that respond well to good reports.
- Vulnerability Disclosure Programs (no cash reward) are a legitimate training ground, not a waste of time — lower competition, often less-hardened targets, and a track record of clean, high-quality reports there is what gets you invited to the private programs that pay.
- Before writing a single request, search the platform's disclosed-report archive for the same program. This tells you the company's actual recurring bug patterns, what triagers there tolerate or reject, and what's already been found so you don't duplicate.
- Pick a program sized to where you are. A mid-size SaaS company with a bounded but real scope teaches the full methodology in weeks; a scope the size of a hyperscaler will eat months before you've even mapped it, let alone tested it.

---

# 2. Reconnaissance — Passive and Wide, Then Active and Narrow

The core discipline underneath every serious recon methodology: **each phase's output filters the next phase's input.** Brute-forcing directories against 5,000 unfiltered subdomains just buries the signal you actually need. Go in this order.

### 2.1 Organizational footprint, before DNS
- Read the company's "About / Investors / Acquisitions" pages and Crunchbase/Wikipedia history. Acquired companies routinely keep running on the acquirer's legacy, less-hardened infrastructure for years — this is one of the highest-value, lowest-effort recon steps and most hunters skip it entirely.
- Pull the org's ASN (`asnmap -d target.com`, or WHOIS `origin`/`route` lookups) and enumerate every IP block registered to that ASN — this surfaces infrastructure that never shows up in a DNS-based subdomain scan at all.
- Do a reverse WHOIS lookup by organization name and by any registrant email tied to the primary domain — sibling domains under the same registrant frequently share infrastructure and get much less security attention than the flagship brand.
- Check reverse DMARC records (services that spider DMARC records and index which other domains share the same policy) and reverse CSP lookups (matching Content-Security-Policy headers across domains) — both surface entirely separate, unlinked domains belonging to the same org that certificate-transparency-based enumeration will never find, because they share infrastructure signals rather than certificates.
- Sweep alternate TLDs. A company that hardens `target.com` for years routinely leaves `target.io`, `target.net`, `target.xyz`, `target.dev` running old code with weaker or no monitoring. Loop the full IANA TLD list against the base domain name and probe each candidate with an HTTP client for a response.
- Search Shodan/Censys by the org's TLS certificate "org" field, by favicon hash, and by JARM/JA3 fingerprint — this finds infrastructure at the network level regardless of whether it was ever linked in DNS.

### 2.2 Subdomain enumeration
Run every passive source before any brute-force:
```
subfinder -d target.com -all -o subfinder.txt
amass enum -passive -d target.com -o amass.txt
assetfinder --subs-only target.com >> subs.txt
curl -s "https://crt.sh/?q=%.target.com&output=json" | jq -r '.[].name_value' | sort -u >> ct.txt
chaos -d target.com -key $PDCP_API_KEY -o chaos.txt   # aggregated bug-bounty-program datasets
sort -u subfinder.txt amass.txt subs.txt ct.txt chaos.txt -o all_subs.txt
```
Only after that's exhausted, brute-force the remaining name space with a curated wordlist against a fast resolver (`puredns`/`shuffledns` + a vetted public resolver list) — brute-forcing before passive enumeration wastes queries chasing names passive sources would have handed you for free.

On repeat targets, diff every run against the previous one and alert on anything new. Newly-appeared subdomains are disproportionately rewarding: they're freshly deployed, often still in a "works on my machine" state, and haven't had a security review pass yet.

### 2.3 Resolving, then prioritizing — the step most hunters skip or do badly
```
cat all_subs.txt | httpx -silent -status-code -title -tech-detect \
  -content-length -follow-redirects -ip -cname -threads 50 -o httpx_full.txt
```
A large scope routinely produces thousands of live hosts. Testing all of them is not realistic — at even 10 minutes per host, 5,000 hosts is a full month of nothing but triage. Prioritization is the actual skill here, not an afterthought:

1. **Filter to scope first, every time.** Anything outside the documented scope gets dropped immediately regardless of how interesting it looks — testing out-of-scope assets is both wasted effort and, on many programs, a rules violation.
2. **Naming signal ranks highest.** Hosts containing `dev`, `staging`, `stage`, `test`, `uat`, `qa`, `internal`, `admin`, `beta`, `old`, `legacy`, `v1` consistently correlate with weaker access control and looser deployment hygiene than the polished production frontend — every methodology that tracks real findings converges on this same pattern.
3. **Tech-fingerprint signal tells you which bug class to expect.** A Next.js app suggests SSRF via its image-optimization pipeline. A raw exposed Swagger/OpenAPI file suggests a large, likely under-tested API surface. A live GraphQL endpoint with introspection on suggests missing per-resolver authorization. A recognizable admin tool (Jenkins, Kibana, Grafana, phpMyAdmin) appearing where the app UI was expected is an internal tool accidentally exposed to the internet — treat any hit like this as immediate priority.
4. **Newness beats familiarity.** If you have historical scan data, anything that's brand new this week outranks anything that's been sitting there for a year.
5. **Absence of a WAF is a green flag.** Hosts without Cloudflare/Akamai/Imperva in front of them (check via response header fingerprinting or a WAF-detection pass) are lower-hanging fruit; the heavily-protected production frontend is usually also the most competitively-tested, least-rewarding surface.
6. **Outliers in title/content-length are worth a manual look even without other signal** — a handful of odd responses in a sea of identical templated pages is exactly where something unusual is running.
7. Screenshot every host that survives the filters above for fast visual triage — a login page, an internal dashboard, and a default server page all need very different next steps, and screenshots let you sort hundreds of hosts in minutes instead of opening each one.

The realistic goal: take a raw list of thousands down to a shortlist of 50–150 using scope + naming + tech signal alone, then manually pick 5–10 to go deep on first, favoring hosts that look bespoke and custom-built over anything running an off-the-shelf CMS or SaaS product — bespoke code is where logic bugs live; templated software is where already-known, already-patched, already-found CVEs live.

### 2.4 Finding the origin IP behind a CDN/WAF
Most modern targets sit behind Cloudflare, Akamai, or similar. Reaching the origin server directly bypasses the WAF, rate limiting, and geofencing entirely.
- Query historical DNS records for A-record entries that predate a CDN migration.
- Search Shodan/Censys by TLS certificate hash or by favicon hash — organizations frequently reuse the exact same favicon on the CDN-fronted host and the unprotected origin.
- Check whether any subdomains found during enumeration (mail servers, dev/staging hosts, misconfigured DNS entries) route directly to the origin instead of through the CDN — they often share hosting with the real backend.
- Inspect TLS certificate Subject Alternative Name entries; occasionally the direct origin hostname leaks there even though the CDN-fronted name is the one publicly advertised.

### 2.5 Ports and services
```
naabu -iL alive_hosts.txt -p - -silent -o open_ports.txt
nmap -sV -sC -p <found_ports> <target> -oA nmap_detail
```
Any non-standard open port — a database, an admin panel, a message queue, a CI/CD tool like Jenkins — deserves manual follow-up. Most hunters only ever look at 80/443, so anything found on an unusual port is consistently high-severity and low-competition.

### 2.6 Content, endpoints, parameters, and secrets
```
gau target.com | tee gau_urls.txt
waybackurls target.com | tee wayback_urls.txt
katana -u https://target.com -jc -kf all -d 3 -o katana_urls.txt   # JS-aware active crawl

ffuf -u https://target.com/FUZZ -w wordlist.txt -mc 200,301,302,403 -recursion   # only on shortlisted hosts

katana -u https://target.com -jc | grep '\.js$' > js_files.txt
python3 linkfinder.py -i js_files.txt -o results.html   # endpoints/params buried in JS bundles
```
- Scan every discovered JavaScript bundle for secrets (API keys, internal hostnames, hardcoded credentials) with a secret-scanning tool, and separately grep the same bundles for endpoint paths the UI never links to directly — client-side JS routinely exposes more of the API surface than the rendered site does.
- Dork GitHub/GitLab for the company's org and any employee accounts: exposed `.env` files, committed and later "deleted" credentials still recoverable from commit history, internal hostnames referenced in config files. Leaked secrets found this way are consistently among the fastest, highest-severity findings in bug bounty because they require no exploitation at all, just discovery.
- Google/Bing dork for indexed-but-unlinked content: administrative paths, exposed JSON files, directory-listing pages, and pages containing strings like API keys or internal error output that got indexed by accident.
- Pull every historical URL (`gau` + `waybackurls` + the active crawl) into one file and bucket it by likely vulnerability class using parameter-name pattern matching — turns an unreviewable wall of tens of thousands of URLs into a short, prioritized candidate list per vulnerability type worth actual manual attention.
- Check for exposed API schemas — `/swagger.json`, `/openapi.json`, a live `/graphql` endpoint with introspection enabled. An exposed schema is close to a complete map of the backend handed to you for free; always check whether older API versions (`/v1/` when the app has moved on to `/v2/`) are still live, since older versions routinely miss authorization fixes that shipped in the newer one.

---

# 3. Application Analysis — Use It Like a User First, Then Like an Attacker

Recon maps the surface; this phase maps the actual logic, and it corrects the trap that a recon-only approach falls into: a giant asset list with no understanding of how any of it actually works.

- Register at least two accounts at every privilege tier the app offers before testing anything — you need two same-tier accounts to compare access control (does account A see account B's data?) and one account per role if roles exist (does a low-privilege account reach admin-only functionality?).
- With an intercepting proxy running the whole time, walk every flow as an ordinary user before trying to break anything: signup, login, password reset, profile edit, checkout, settings, delete-account, every onboarding step. Read the *request bodies*, not just the URLs — a large share of real-world access-control bugs are client-supplied identity fields sitting quietly in a POST body that the server trusts instead of using its own session.
- For every parameter that looks redundant with something the server should already know from the session (a user ID in the body when a session cookie already identifies the user, a price sent from the client, a role field the UI never exposes but the API still accepts), ask explicitly: *why does the client need to tell the server this at all?* That single question is very often the entire vulnerability.
- Read the API docs, the OpenAPI/Swagger schema, and — critically — decompile or proxy the mobile app if one exists. Mobile clients frequently call a larger, less-curated slice of the backend API than the web client does, including old internal endpoints nobody thought to remove.
- Note every place client trust and server trust seem to blur: file upload, "import from a URL," webhook registration, PDF/report/screenshot generation, "share" or "export" features, anything that makes the server fetch or render content on the user's behalf — these are the shapes SSRF hides inside, and they rarely have an obviously-named "url" parameter.
- Look for functionality the UI hides but the backend still accepts — disabled buttons whose underlying request still fires, admin-only UI fragments visible in the JS bundle even though the visible interface gates them client-side.

---

# 4. Testing by Vulnerability Class

Match what recon and application analysis already told you to the class most likely present, rather than spraying every payload at every endpoint.

## 4.1 Broken access control (IDOR / BOLA / BFLA) — the highest-yield category on most platforms
This is formally split by OWASP's API risk model into distinct failure modes worth testing separately, because they have different root causes:
- **Broken Object Level Authorization (BOLA/IDOR)** — the API accepts an object ID as input and never verifies the requesting user actually owns or has permission to access that specific object. This is consistently the single most common API vulnerability class across every published dataset, because it requires a developer to remember to add an ownership check on every single object-returning endpoint, and it only takes one missed endpoint.
- **Broken Function Level Authorization (BFLA)** — distinct from BOLA: instead of reaching someone *else's* data, a low-privileged user reaches a privileged *function* (an admin-only delete/promote/refund endpoint) because the authorization check exists only in the UI (the button is hidden) and not on the backend route itself. Always test whether admin-only actions succeed when called directly by a non-admin session, even if the UI never shows the option.
- **Broken Object Property Level Authorization** — a merge of two older categories: excessive data exposure (the API returns more fields than the UI displays, including internal/sensitive ones) and mass assignment (the API accepts more fields in a write request than the UI ever submits, letting you set fields like `role`, `isAdmin`, or `balance` directly if you simply add them to the JSON body). Always try adding extra fields to a write request that mirror fields you've seen returned in a read response.

Practical technique: for every endpoint that takes an ID, run the same authenticated request twice — once with your own session and target ID, once with your own session but a *different* account's ID — and check whether the response returns that other account's data or lets you modify it. Automating this specific replay across an entire recorded session (every request you made as user A replayed as user B) is the single highest-leverage manual-testing time-saver in this whole document; it turns hours of one-by-one comparison into an automated pass you only need to review the flagged results of.

Also reason explicitly about the ID space itself once you find a hit: sequential, low-entropy numeric IDs (account #572, #573) mean the entire user base is enumerable in a tight, guessable range — a very different, much higher severity than the same bug against random UUIDs where you can only prove impact against accounts you already know. State this explicitly in the report; it's the difference between a low bounty and a critical one for the identical underlying bug.

## 4.2 Server-Side Request Forgery (SSRF)
Look for it in features that make the *server* fetch or render something: import-from-URL, webhook registration and delivery, PDF/report generation, "fetch a link preview," audio/video streaming from a source URL, avatar-from-URL upload.

Order of techniques to try, roughly fastest-to-slowest:
1. Try the raw internal address first — no bypass, no cleverness — before assuming a filter is even in place.
2. If blocked, test every redirect status code separately (301, 302, 303, 307, 308) — filters routinely check only 301/302 and miss the rest, since 303 in particular is an unusual code most validation logic never accounts for.
3. Try alternate IP encodings for `127.0.0.1` / internal ranges: short-form (`127.1`), octal (`0177.0.0.1`), decimal-integer form, hex (`0x7F000001`), IPv6 loopback (`::1`), and `0.0.0.0`.
4. Chain through an open redirect — your own hosted redirect page, or a public one — if the filter validates the initial URL but not where it ultimately lands.
5. If a filter regex-matches on a full URL pattern, try dropping the scheme entirely or using scheme-relative syntax.
6. Try protocol downgrade tricks (HTTP/0.9 requests, dropping the Host header) and mixed slash-encoding on `file://` scheme payloads.
7. Once basic SSRF is confirmed and internal hosts are reachable, the `gopher://` protocol can be used to speak to internal services directly — Redis, Memcached, MySQL, FastCGI, SMTP — for a path toward RCE, not just information disclosure.
8. Watch for DNS rebinding opportunities: if a filter resolves and validates a domain once, then the actual HTTP client resolves it again separately when fetching, a domain that returns a public IP on the first check and an internal IP on the second slips straight through.

Once SSRF is confirmed, always escalate past a bare "it made a request" proof — go for the cloud metadata endpoint:

| Provider | Endpoint | Notes |
|---|---|---|
| AWS EC2 (IMDSv1) | `http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>` | Plain GET, no special headers — works via basic SSRF directly |
| AWS EC2 (IMDSv2) | Requires a `PUT` request plus a token header first | Much harder to trigger through basic SSRF; check which version is in use before assuming success |
| AWS ECS task role | `http://169.254.170.2/v2/credentials/` | Needs the container's task UUID, frequently readable from `/proc/self/environ` if you have any local file read |
| GCP | `http://169.254.169.254/computeMetadata/v1/` | Normally requires a `Metadata-Flavor: Google` header — test without it anyway, since some proxies leak the response regardless; grab an identity token via the `?audience=` parameter for IAP-protected backends |
| Azure | `http://169.254.169.254/metadata/instance?api-version=2021-02-01` | Normally requires a `Metadata: true` header |

The recurring root cause behind almost every SSRF bypass: the filter validates one representation of the input (a resolved IP, a decoded URL, a normalized hostname) while the actual HTTP client fetches a *different* representation of the same input (the raw string, the post-redirect target, a re-encoded value). Every technique above is a variation on exploiting that specific mismatch.

## 4.3 Bypassing 401/403 access-control gates
Once you hit a 403/401 on a resource you believe you shouldn't be blocked from, work through this matrix — path tricks first, they have the highest success rate, then method and header tricks:

- **Path manipulation:** trailing slash (`/admin/`), case variation (`/Admin`, `/AdMiN`), path insertion (`/./admin`, `/admin/.`, double slashes `//admin`), URL-encoding the path (single and double encoding, Unicode variants), and — specifically against Nginx front-ends — the `/..;/` path-normalization bug, where appending `/..;/admin` to a blocked path gets normalized differently by the proxy than by the backend application it forwards to, creating a disconnect the access rule doesn't account for.
- **HTTP method tampering:** most access-control rules are only ever tested against GET. Try POST, PUT, PATCH, DELETE, HEAD, OPTIONS, TRACE, and even made-up verbs against the same blocked path — and separately, try the `X-HTTP-Method-Override` header to make a GET request behave as a different verb server-side, since some frameworks honor this header without re-checking authorization for the effective method.
- **Header spoofing:** `X-Forwarded-For: 127.0.0.1`, `X-Real-IP: localhost`, `X-Originating-IP`, `Client-IP`, `X-Remote-IP` — many applications and load balancers trust client-supplied IP headers for access decisions, especially for anything gated as "internal only."
- **Header/URL bypass headers used by some proxies and app frameworks:** `X-Original-URL` and `X-Rewrite-URL`, which some setups use to determine the effective path separately from the one the reverse proxy actually enforced rules against.
- **Parameter pollution:** appending parameters like `?admin=1`, `?debug=true`, or duplicate parameters (`?role=user&role=admin`) — some stacks process the first occurrence of a duplicated parameter while the access-control layer checks the last, or vice versa, creating an exploitable gap between the two.
- **Removing or altering the Host header entirely** can bypass protections that key off a specific expected host value.
- **HTTP request smuggling** is the deeper version of this same class of bug: when a front-end proxy and a back-end server disagree about where one request ends and the next begins, you can smuggle a request past authorization logic the proxy believes it's enforcing.

## 4.4 Injection-class bugs (XSS, SQLi, SSTI, command injection)
- **XSS:** prioritize any field whose value gets reflected somewhere else in the app later — comments, display names, search terms echoed on the results page, error messages that include user input verbatim. Stored XSS (persisted and shown to other users) is worth dramatically more than reflected; always check whether a reflected-looking sink is actually stored somewhere first.
- **SQLi:** prioritize any DB-backed filter, search, or sort parameter. Validate candidates one at a time with a targeted, scoped tool run rather than a blind mass scan across every parameter — mass, unscoped scanning against a live target both wastes time on false positives and risks tripping rate limits or WAF bans that cost you the rest of the engagement.
- **SSTI:** look for any input concatenated near "template," "render," or included verbatim in generated output like emails, PDFs, or error pages — a classic tell is an input field where basic math-like syntax gets evaluated rather than displayed literally.
- **Command injection:** anywhere user input reaches a shell-out (image processing, file conversion, "ping this host" utilities, export tools) is worth testing for shell metacharacter injection.

## 4.5 GraphQL-specific issues
GraphQL fails access control differently from REST specifically because authorization has to be enforced per-resolver rather than per-route, and it's easy for a team to forget one:
- Always attempt introspection first (`__schema` query) — if enabled in production, it hands you the entire object/field/mutation map for free.
- Test whether the same object-level authorization gaps exist per-field/per-resolver, not just per-query — a query might be gated correctly while a nested resolver returning related data is not.
- Test for mass assignment via extra mutation fields the client UI never submits but the schema accepts.
- Check for missing query-depth/complexity limits, which allow a single deeply-nested query to cause excessive resource consumption — a denial-of-service vector unique to GraphQL's flexible query shape.
- Check whether batching is allowed unrestricted, which can be abused to bypass rate limiting on sensitive mutations (password reset attempts, coupon code guessing) by sending many attempts inside a single HTTP request.

## 4.6 File upload
- Confirm whether extension filtering happens client-side only (trivially bypassed) or server-side.
- Test polyglot/double-extension files, content-type header spoofing independent of actual file content, and SVG files carrying embedded script for stored XSS via image rendering.
- Check where the uploaded file is served from — an upload path served from the same origin as the app, with executable permissions, is a direct path to remote code execution.

## 4.7 Business logic and workflow abuse
This is a distinct OWASP-recognized risk category in its own right precisely because the API often works exactly as designed — the vulnerability is that the design didn't anticipate abuse at scale or out of sequence, so automated scanners structurally cannot find it.
- **Race conditions:** any action that should be limited to "once" (redeem a coupon, use a promo balance, submit a vote, claim a referral bonus) is worth firing many times in rapid, near-simultaneous parallel requests — timing-sensitive check-then-act logic frequently has no locking around the check, so several requests can all pass the check before any of them completes the act.
- **Workflow-order bypass:** try skipping steps in a multi-step flow (checkout, KYC, onboarding) by calling a later step's endpoint directly without completing the earlier ones — many flows enforce order only in the frontend's navigation, not in the backend's state machine.
- **Negative values and boundary abuse:** quantities, prices, and discount fields accepting negative numbers, zero, or values beyond expected bounds can produce logic that developers never considered (a negative quantity that *adds* money instead of charging it, for instance).
- **Resource consumption limits:** unrestricted repeated actions with real-world cost (email/SMS sending, PDF generation, expensive search queries) worth testing for missing rate limits, since these translate directly into a spend/DoS vector against the company, not just an inconvenience.
- **Unsafe consumption of third-party APIs:** if the target itself calls out to another API or LLM provider and trusts the response without validation, that's now its own recognized risk category — check whether responses from those upstream calls are sanitized before being rendered or acted on.

## 4.8 OAuth/SSO and open redirect chaining
- Test whether the `redirect_uri` parameter is validated against an exact allow-list or just a loose prefix/substring match — a loose match is bypassable and turns into token/code leakage to an attacker-controlled host.
- An open redirect by itself is usually low severity; chained with a loosely-validated OAuth flow that trusts the redirect target, it becomes full account takeover — always check whether a "low" finding chains into something higher before finalizing severity.
- Check account-linking flows specifically for confusion between which account a linked identity actually attaches to, especially when email verification isn't re-checked at link time.

---

# 5. Confirming, Escalating, and Chaining Before You Report

- Never submit anything a scanner flagged without personally reproducing it end to end, exactly as a triager would need to. Automated "critical" flags are hypotheses, not conclusions — submitting unvalidated scanner output is the fastest way to burn trust with a program and get your future submissions deprioritized.
- Always push one step further before finalizing severity: a blind SSRF that only proves outbound DNS resolution is a fundamentally different report than one that reaches cloud metadata and extracts live IAM credentials — don't stop at the first proof point if a next step is available and safe to test.
- Actively look for chains: an open redirect plus a loosely-validated OAuth flow becomes account takeover; a self-XSS plus a CSRF gap that lets you deliver the payload to someone else's session becomes stored XSS against arbitrary users. Individually these are "low." Combined, they're critical — programs consistently reward the chain, not the isolated primitive.
- Reason about scale, not just the one instance you can prove: a sequential, low-entropy ID space turns a single-account IDOR into a mass-account-takeover primitive, and that reasoning belongs explicitly in the report, not left for the triager to infer.

---

# 6. Writing the Report

A technically strong finding with a weak report gets under-paid or bounced. A modest finding with a clear one gets triaged fast and paid fairly. Structure every time:
1. **Title** — vulnerability class + location + impact in one line. Not "XSS found" — "Stored XSS in profile bio field, executes against any user viewing the profile."
2. **Summary** — two or three sentences a non-technical reader could still follow.
3. **Steps to reproduce** — numbered, unambiguous, exact requests and responses, screenshots or a short recording. If a developer couldn't fix the bug from the report alone, the report isn't finished.
4. **Impact** — the worst realistic outcome stated plainly: "this allows retrieval of IAM credentials, compromising the entire AWS account," not "this seems bad."
5. **Remediation suggestion** — showing you understand the correct fix (allow-lists over deny-lists, ownership checks on every object route, parameterized queries, disabling redirects on the fetch client) builds triager trust over successive reports, not just this one.
6. **Severity justification** — cite the relevant CVSS vector and any applicable multiplier explicitly; for example, unauthorized access to another user's personal data on an EU-scoped target is a GDPR exposure and legitimately raises severity — state that reasoning rather than leaving it implicit.

Before submitting, ask: *if I were the developer, could I fix this from only what I've written?* If the answer is no, keep writing.

If a report gets downgraded and you believe the severity reasoning is wrong, respond professionally with evidence and ask the triager to justify the rating — triage is a dialogue, not a final verdict, and a calm, factual pushback with clear reasoning has corrected unfair downgrades in documented cases. Demanding a specific payout or treating the disagreement as hostile does the opposite.

---

# 7. Where AI Actually Helps in 2026, and Where It Actively Hurts You

By mid-2026 the large majority of active hunters use AI somewhere in their workflow, and reported AI-related vulnerability submissions have grown sharply year over year. But the field data from triage teams is consistent on one point: **your edge disappears the moment every other hunter on the same target runs the same model against it the same way.** What still separates top hunters is judgment about which AI output to trust, which to validate, and which to discard entirely — not access to AI itself.

### Genuinely good uses, by phase
- **Recon synthesis:** feeding a large `httpx`/`nuclei` output dump to an LLM to produce a prioritized shortlist, or having it write a one-off glue script for your pipeline. Mechanical synthesis over large, repetitive text is exactly what LLMs are reliable at, and a mistake here just means re-checking a shortlist, not submitting a false finding.
- **JavaScript analysis:** minified or obfuscated JS bundles are tedious to read by hand; an LLM identifying likely API endpoints, auth logic, or hardcoded secrets inside a large bundle is fast and generally reliable, because it's pattern extraction, not judgment.
- **Narrowly scoped code review:** asking "does user input on this specific parameter reach this specific SQL query unsanitized?" against a bounded snippet works well. Asking an LLM to review an entire unfamiliar codebase for "any vulnerabilities" produces confident, plausible-sounding hallucinations — the difference is scope, not capability.
- **Report drafting:** turning your own notes and captured request/response pairs into a clean, structured report (title, summary, repro steps, impact) — you already did the finding; the model just improves the write-up's clarity and speed. Every technical claim it writes still needs your own verification before it goes out.
- **Schema review:** pointing an LLM at a large OpenAPI or GraphQL schema to flag endpoints whose authorization pattern looks inconsistent with the rest of the API is a good use — comparison across hundreds of endpoints is tedious for a human and cheap to verify once flagged, so a wrong flag costs you almost nothing.
- **Learning an unfamiliar stack quickly** — "explain this framework's server-side data-fetching model" before you start testing it speeds up application analysis without replacing the testing itself.

### Where it actively hurts you
- **Autonomous "point-and-scan-and-submit" agents.** Running a model against a target with no human validation step produces confident false positives at scale — triage teams now explicitly describe this pattern as flooding queues with low-quality noise, and it damages your standing with programs, sometimes leading to a restricted submission queue.
- **Unscoped whole-codebase review.** Models reliably misread custom framework semantics, overstate whether a "vulnerable-looking" code path is actually reachable, and confuse user-controlled versus server-controlled data once a call chain gets long — the failure mode is confident and specific-sounding, which makes it more dangerous than an obviously wrong guess.
- **Business logic and workflow-abuse bugs remain squarely a human-judgment problem.** Race conditions, order-of-operations bypasses, and anything that depends on understanding *this specific company's* business rules is where AI currently adds the least value — this is exactly where top hunters still separate from the pack.
- **Rule of thumb: never submit anything a model calls "critical" until you've personally walked through the exact repro steps and confirmed the impact yourself,** the same way a triager will. If you can't reproduce it manually end to end, it doesn't go out.

### A fast-growing target category worth knowing explicitly: AI features themselves
Programs increasingly ship LLM-backed chatbots, coding assistants, document summarizers, and internal search tools, and this surface has its own emerging bug class — prompt injection, insecure handling of model output, and excessive agency granted to tool-calling agents. If a target has an AI feature in scope, treat it as its own attack surface:
- Test prompt injection through any content the model ingests that a user (possibly an attacker) controls — uploaded documents, fetched web pages, tool outputs the model reads back.
- Check whether the model's own tool-calling ability can be abused to read data or take actions outside what the feature was meant to allow — this is functionally a BFLA problem wearing an AI costume: the model is acting as an intermediary with more effective privilege than the user should have.
- Check how the model's output gets rendered downstream — an LLM response inserted unsanitized into HTML is XSS with extra steps, and it's frequently missed because the injection point (a chat message) doesn't look like a traditional form field to test.
- Watch for SSRF-via-prompt-injection specifically: a crafted prompt that causes the backend model or its agent framework to make an outbound request to an internal service on the attacker's behalf, effectively turning a chat interface into a request-forgery primitive.

---

# 8. Habits That Compound (the part most compilations skip)

- Spend fifteen to twenty minutes a week scanning newly disclosed reports across platforms, filtered to one or two vulnerability classes — not to read everything, but to catch new bypass techniques as they surface. The exact validation/execution mismatch behind an SSRF bypass disclosed on an unrelated target today is very likely to reappear on your target next month.
- Keep notes per program, not per bug — features already tested, parameters already seen, dead ends already ruled out. Revisiting a target months later without notes means re-doing work you've already done once.
- Follow individual researchers directly rather than only aggregator sites — one-off tips and freshly discovered techniques surface in individual posts months before they get compiled into any wiki or cheat sheet.
- Re-test previously patched bugs on active programs periodically. Regressions happen routinely after refactors, framework upgrades, or infrastructure migrations, and a regression of a bug you already understand is a very fast re-find.
- Set up continuous monitoring on your best-performing programs — a diffed subdomain/endpoint watch that alerts on anything new surfaces attack surface the moment it's deployed, which is disproportionately unpatched and untested compared to established production endpoints that have already absorbed a year of scrutiny.
- Practice continuously on deliberately vulnerable applications, not only live targets — building muscle memory without burning goodwill or engagement time on a real program compounds faster than trial-and-error on live targets alone.
- Periodically revisit the defensive side of each vulnerability class — knowing exactly what the *correct* fix looks like (allow-lists instead of deny-lists, ownership checks on every object route, disabling redirects on outbound fetch clients, per-resolver GraphQL authorization) tells you precisely which half-implemented mitigations to expect and target on real, imperfectly-patched applications.

---

# The One Meta-Lesson Underneath All of This

Every vulnerability class in this document — access control, SSRF, injection, business logic, AI-feature abuse — comes down to the same underlying pattern: **something in the system disagrees with itself about the truth.** The WAF's view of a redirect status code disagrees with the app's. The UI's exposed role disagrees with the API's accepted role. The validated-and-decoded form of an input disagrees with the actually-fetched form. A documented cloud metadata protection disagrees with what's actually enforced server-side. Methodology, tooling, and now AI are all just ways of finding that disagreement faster and more completely than the last person who tested the same target — and that disagreement keeps reappearing in completely unrelated codebases, which is why reading how someone found it on a target that isn't yours still pays off on the one that is.
