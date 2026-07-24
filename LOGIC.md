# Business Logic Writeups — The Ones That Matter

> Business logic flaws are the bugs no scanner finds. They require understanding the business, mapping every field, and thinking like someone who wants to steal money or abuse workflows.

## The Must-Reads

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 1 | Comprehensive Business Logic Flaws Guide (2026) | chs.us | 339 insights across 13 categories: workflow bypass, race conditions, payment logic, privilege escalation, state manipulation, attack chains — with CVEs and methodology | https://chs.us/guides/business-logic-flaws/ |
| 2 | Exploiting Business Logic Error Vulnerabilities | Intigriti (Ayoub) | Exploitable vs non-exploitable distinction, currency confusion (USD→JPY), inconsistent input validation → injection, secondary portal weak validation, JWT none algorithm in business context | https://www.intigriti.com/researchers/blog/hacking-tools/exploiting-business-logic-error-vulnerabilities |
| 3 | OWASP Business Logic Testing Methodology (WSTG v4.2) | OWASP | 8-step methodology: data validation, request forging, integrity checks, process timing, function limits, workflow circumvention, defenses against misuse, unexpected uploads | https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/10-Business_Logic_Testing/00-Introduction_to_Business_Logic |
| 4 | OWASP Business Logic Security Cheat Sheet | OWASP | Server-side value derivation rules, state machine enforcement, common flaw categories, testing guidance — "No scanner will find these bugs for you" | https://cheatsheetseries.owasp.org/cheatsheets/Business_Logic_Security_Cheat_Sheet.html |
| 5 | Business Logic Vulnerability Patterns (2026 Methodology) | su6osec | Structured testing checklist per category, 7 mindset questions to ask at every feature, high-value areas prioritization | https://github.com/su6osec/Bug-Bounty-Hunting-Methodology-2026/blob/main/vulnerabilities/business_logic.md |

## Price & Quantity Manipulation

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 6 | Price Manipulation — Complete Bug Bounty Guide | BugBounty.info | Client-side price trust, negative values, integer overflow, discount stacking, currency conversion abuse, systematic flow mapping — with HackerOne #1162788, #1408498, #1300748 | https://bugbounty.info/Attack-Surface/Web/Business-Logic/Price-Manipulation |
| 7 | Price Manipulation Guide (10 Modules) | Rishu Rana | Structured guide from basics to expert, attack mindmap, severity reference table with bounty ranges ($50–$10,000+), mobile testing with Frida/Objection | https://github.com/Rishurana2867/price-manipulation-guide |
| 8 | How I Found Price Manipulation Twice | HackMoN Ai | Buying for free via null/0/-1/0.1 quantity, bypassing shipping fees, interception methodology with Burp | https://undercodetesting.com/how-i-found-price-manipulation-twice-detailed-write-up-approach/ |
| 9 | Exploiting Logic Flaw via Fractional Quantities | Hamzadzworm | Quantity: 0.1 or 0.01 purchases high-value items at fraction of price, improper integer validation | https://hamzadzworm.medium.com/exploiting-logic-flaw-to-bypass-payment-using-fractional-quantities-e2f62aa512fb |
| 10 | From Quantity Manipulation to Negative Shipping Costs | Hangga Aji Sayekti | Manipulating quantity triggers negative shipping calculation, shipping cost logic separate from price logic | https://cybersecuritywriteups.com/from-quantity-manipulation-to-negative-shipping-costs-a-business-logic-flaw-in-an-e-commerce-73f6c80b9b17 |
| 11 | Quantity Manipulation to Zero Total | Bryan Zheng | Negative quantity = deduction, mathematical puzzle of balancing multiple items to $0 total via quantity combinations | https://medium.com/@bry4nzheng/idor-insecure-direct-object-reference-price-manipulation-on-private-bug-bounty-program-2527431ffcfd |
| 12 | Uncovering Business Logic: Real-World Case Study | Nishanthan | PortSwigger lab technique in real world: modified product price from $1337 to $0.13 via Burp interception, server-side validation absent | https://medium.com/@nishanthannisha008/uncovering-business-logic-vulnerabilities-a-real-world-case-study-18bfbdae46b3 |

## Race Conditions

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 13 | Race Conditions — Complete Bug Bounty Reference | BugBounty.info | Single-packet attack, Burp parallel send, Turbo Intruder setup, coupon reuse race, double spend, rate limit bypass, feature limit bypass — with HackerOne #454949, #604534, #413759, #165570 | https://bugbounty.info/Attack-Surface/Web/Business-Logic/Race-Conditions |
| 14 | Race Condition + Business Logic: Decimal Point Error | Lekssik (Oleksandr Opanasiuk) | Rounding exploitation (0.009→0.01), rising minimum bypass via race, thread isolation exploitation, multi-step scripting, chaining two low-severity bugs into critical | https://medium.com/@04sabsas/bugbounty-writeup-creative-thinking-is-our-everything-race-condition-business-logic-error-2f3e82b9aa17 |
| 15 | Exploiting Race Conditions for Infinite Discounts | Aditya Bhatt | BurpSuite Repeater parallel execution, scenario table (coupon/payment reset/inventory/privilege), mitigation-aware testing | https://infosecwriteups.com/bug-bounty-race-exploiting-race-conditions-for-infinite-discounts-a2cb2f233804 |
| 16 | Race Condition + Business Logic Error Writeup | Lekssik (GitHub) | Script architecture: Script 1 creates payments + collects tokens, Script 2 spawns concurrent sub-scripts with `&` operator, thread-parallel execution | https://github.com/Lekssik/writeup/blob/master/1.%20Race%20Condition%2BBusiness%20Logic%20Error |

## Workflow & State Machine Bugs

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 17 | State Machine Bugs | BugBounty.info | Step skipping, state replay, out-of-order execution — every multi-step flow is a target | https://bugbounty.info/Attack-Surface/Web/Business-Logic/State-Machine |
| 18 | $1,500 Bounty: Business Logic in Project Secrets Management | Abhi Sharma | UI vs API authorization mismatch, direct API crafting bypassing frontend restrictions, role-based policy mismatch, free-tier limitation bypass — CVSS 8.3 | https://infosecwriteups.com/1-500-bounty-business-logic-flaw-in-project-secrets-management-4b985175071e |

## Real-World Breaches & Financial Impact

| # | Title | Author | What You Learn | URL |
|---|-------|--------|----------------|-----|
| 19 | 5 Real Business Logic Failures That Led to Major Breaches | APIsec (Dan Barahona) | USPS (broken access in Informed Delivery API), Symantec (23K SSL certs revoked), Venmo (unauthenticated developer API), Coinbase ($250K bounty — crypto balance validation) | https://www.apisec.ai/blog/5-real-world-examples-of-business-logic-vulnerabilities-that-resulted-in-data-breaches |
| 20 | API Pentest: Business Logic in Fantasy Pools | rickyma18 | Unauthenticated Prometheus `/metrics` → leaked IDs → BOLA on transaction endpoints → financial data + payment tokens, OTP flow abuse, referral abuse | https://github.com/rickyma18/api-pentest-otp-business-logic |

## Key Numbers to Remember

| Metric | Value | Source |
|--------|-------|--------|
| Coinbase business logic bounty | $250,000 | APIsec |
| Symantec certs revoked from logic flaw | 23,000 | APIsec |
| Price manipulation bounty range (P1) | $1,000–$10,000+ | Rishu Rana |
| Discount stacking bounty range (P2) | $300–$2,000 | Rishu Rana |
| Currency manipulation bounty range | $500–$3,000 | Rishu Rana |
| $1,500 bounty (CVSS 8.3) | UI vs API mismatch | Abhi Sharma |
| OWASP business logic test steps | 8 | OWASP WSTG v4.2 |
| "No scanner will find these" | — | OWASP Cheat Sheet |

## The 7 Questions to Ask at Every Feature

```
"What would happen if I...
 ...did this out of order?"
 ...sent this twice?"
 ...used this for a different purpose?"
 ...modified this client-side value?"
 ...did this very fast / concurrently?"
 ...did this as a different user?"
 ...did this after I shouldn't be able to?"
```

## Severity Reference

| Vulnerability | Severity | Bounty Range |
|---|---|---|
| Buy item for free | P1 Critical | $1,000–$10,000+ |
| Pay fraction of price | P1-P2 High | $500–$5,000 |
| Infinite coupon abuse | P2 High | $300–$2,000 |
| Currency manipulation | P2 High | $500–$3,000 |
| Discount amplification | P2-P3 Medium | $150–$1,000 |
| 500 error on price field | P3-P4 Low | $50–$300 |
