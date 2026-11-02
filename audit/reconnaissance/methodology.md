# OWASP WSTG methodology and coverage

## Method
This review maps observations to WSTG category families; it does not claim that individual active test cases were executed.
- Assessment mode: external, unauthenticated review of public pages/resources on `www.cynbit.in` only; predominantly passive, with one explicitly authorized GET each to three exact trailing-slash paths.
- Allowed actions used: ordinary low-volume GETs for public pages/metadata/assets and read-only DOM/static-bundle inspection.
- Authorization boundary: no broad active-test authorization, test accounts, or approval for other hosts was established. The user later authorized only one metadata-only GET each to `/api/`, `/admin/`, and `/portal/`; no redirect was followed. WSTG tests requiring interaction, payloads, authenticated roles, request tampering, or load were not run.

Coverage statuses mean:

- **PARTIAL / OBSERVED** — limited evidence was collected through passive inspection and the single, route-limited GET exception; this is not complete WSTG test coverage.
- **NOT TESTED** — omitted due to scope/authorization or unavailable evidence.
- **NOT ASSESSED** — no reliable conclusion can be drawn from the passive sample.

## Category matrix

| WSTG family | Coverage | Passive evidence | Not performed / limitation |
|---|---|---|---|
| WSTG-INFO — Information Gathering | PARTIAL | Public navigation/pages, robots.txt, sitemap, public response headers, selected JS assets, browser resource list, and one request each to the three explicitly authorized robots-disallowed trailing-slash paths | No crawling/following of apex-host sitemap entries or redirected paths, DNS/subdomain enumeration, scanning, or service fingerprinting beyond observed headers/assets |
| WSTG-CONF — Configuration and Deployment Management Testing | PARTIAL | One homepage response header set; public robots/sitemap/security.txt status; exactly one authorized GET each to the three robots-disallowed trailing-slash paths, returning redirects | No redirect targets followed, no other routes/hosts/HTTP methods, TLS configuration testing, origin/CDN bypass, cloud storage, admin-interface interaction, or broad header survey |
| WSTG-IDNT — Identity Management Testing | NOT TESTED | No identity workflow within authorized passive-only scope | No account creation, identity proofing, enumeration, or role lifecycle tests |
| WSTG-ATHN — Authentication Testing | NOT TESTED | A portal URL is only named in client code and is on the out-of-scope apex host | No login, logout, reset, MFA, credential policy, or auth response testing |
| WSTG-ATHZ — Authorization Testing | NOT TESTED | BitElevate payment-to-credential flow mapped statically as a review item | No roles, object IDs, horizontal/vertical access checks, or payment entitlement bypass tests |
| WSTG-SESS — Session Management Testing | NOT TESTED | No authenticated cookie/session behavior assessed | No cookie attributes, fixation, rotation, timeout, CSRF, or token lifecycle testing |
| WSTG-INPV — Input Validation Testing | PARTIAL / OBSERVED ONLY | Names, types, and required/optional attributes of visible form controls recorded | No submission, malformed data, boundary values, injection, encoding, or server-side validation testing |
| WSTG-ERRH — Error Handling | NOT TESTED | No invalid request or input was generated | No error pages, stack traces, exception handling, or failure-state behavior tested |
| WSTG-CRYP — Weak Cryptography | NOT ASSESSED | Ordinary HTTPS public navigation only | No TLS/certificate/cipher review, crypto implementation, token entropy, or key-management review |
| WSTG-BUSL — Business Logic Testing | PARTIAL / STATIC OBSERVATION | Client-side outline of BitElevate order creation, checkout, verification, and credential display | No checkout or action invocation; payment authorization, order binding, replay, idempotency, and issuance behavior unknown |
| WSTG-CLNT — Client-side Testing | PARTIAL | Public DOM, page assets, selected static bundles, one browser timing sample | No active DOM-XSS, CSP execution, postMessage, storage, service-worker, or client-side state manipulation tests |
| WSTG-APIT — API Testing | NOT TESTED / INVENTORY ONLY | No REST/GraphQL/tRPC application response observed; exactly one GET each to `/api/`, `/admin/`, and `/portal/` returned 308 slash-normalization redirects; two Next.js server-action names found in public client bundle | Redirects not followed; server action method/URL/payload/response and any API auth/schema/rate limiting were not tested |

## Explicit exclusions

No active vulnerability scan, fuzzing, request replay/tampering, login or registration, form submission, payment attempt, access-control check, brute-force/rate-limit check, or load test was conducted. The three trailing-slash GETs were the sole limited active exception and were authorized explicitly; no redirect was followed. Do not interpret “not observed” as “not present” or “not vulnerable.” Expand testing only after written authorization that names the domains, routes, accounts, allowed test classes, and safe rate limits.