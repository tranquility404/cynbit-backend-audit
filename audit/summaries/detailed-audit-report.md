# Cynbit Technologies — Detailed External Security & Backend Audit Report

**Target:** `https://www.cynbit.in/`  
**Assessment date:** 2026-11-02 (+05:30)  
**Assessment type:** unauthenticated external black-box review; predominantly passive  
**Primary methodology:** OWASP Web Security Testing Guide (WSTG) v4.2  
**Evidence base:** files under `../evidence/`, route/endpoint inventories, findings ledger, architecture notes, and performance notes in this audit dataset.

> **Scope statement:** This was not a full penetration test or a source-code/backend review. No source code, infrastructure access, test accounts, or broad active-testing authorization were supplied. The user explicitly permitted only one metadata-only GET each to `https://www.cynbit.in/api/`, `/admin/`, and `/portal/`; each returned a redirect, and none of those redirects was followed. No forms, payment actions, authentication flows, or authorization boundaries were tested.

## 1. Executive summary

The review collected public-page, response-header, metadata, DOM, and selected JavaScript bundle evidence for `www.cynbit.in`. The public surface is strongly consistent with a Next.js frontend served through Vercel. Public client code outlines a BitElevate payment-to-credential flow, but its server-side enforcement is unknown. No REST, GraphQL, tRPC, or application API behavior was confirmed.

The evidence supports **two confirmed informational hardening observations**:

1. Both checked `www` security contact paths returned 404: `/.well-known/security.txt` and `/security.txt` (SEC-001).
2. One captured homepage HTML response lacked five common browser-hardening headers: CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, and Permissions-Policy (SEC-002).

These are scoped hygiene/configuration observations, not demonstrated exploits. **No exploitable vulnerability was confirmed.** This is not assurance that untested behavior is secure.

| Ledger severity | Count |
|---|---:|
| Critical | 0 |
| High | 0 |
| Medium | 0 |
| Low | 0 |
| Informational | 2 |

The highest-value follow-up is an owner-authorized review of payment verification and credential issuance, followed by properly scoped tests of authentication, authorization, session handling, server-side input validation, and API behavior. The payment flow is recorded as an unverified review priority—not a vulnerability finding.

## 2. Scope, authorization, and limitations

### In scope

- The user-provided origin `https://www.cynbit.in/`.
- Low-volume ordinary GETs for publicly available pages, metadata, and public assets; browser DOM inspection and text-only inspection of public JavaScript assets.
- Following the user's explicit later instruction, exactly one unauthenticated GET to each of `/api/`, `/admin/`, and `/portal/` on the `www` host, recording response metadata only.

### Not in scope / not performed

- Apex host `cynbit.in`, other subdomains, Cashfree, GitHub, Unsplash, or other external services. The sitemap and client bundle mention these, but the apex-host pages and provider endpoints were not navigated to or tested.
- The Next.js server actions, payment checkout, portal login, or any route requiring authentication.
- Form submission, account creation, login, password reset, request tampering/replay, IDOR/RBAC tests, injection payloads, enumeration, brute force, rate-limit testing, scanning, fuzzing, or load testing.
- Broad crawling of sitemap paths or robots-disallowed routes. The three authorized trailing-slash paths were not followed to their slashless redirect targets.

The public `robots.txt` lists `/api/`, `/admin/`, and `/portal/` as disallowed. This directive is not evidence that an application exists at any path. The exact trailing-slash GETs only exposed 308 redirects to slashless locations; the destinations were not requested.

## 3. Method and WSTG coverage

The audit uses the OWASP WSTG v4.2 family structure. The project page identified v4.2 as the available stable release and v5.0 as in development at assessment time. This mapping describes the evidence collected and tests omitted; it does not claim active execution of all WSTG cases.

| WSTG family | Coverage | Assessment result |
|---|---|---|
| WSTG-INFO — Information Gathering | Partial | Selected public routes, robots.txt, sitemap, response headers, public assets, browser resource list, and the three authorized redirects recorded. No broader crawling, DNS/subdomain enumeration, or scanning. |
| WSTG-CONF — Configuration and Deployment | Partial | One root response header set, public metadata, security.txt status, and the three redirects observed. No TLS/cipher, origin-bypass, cloud storage, HTTP-method, or broad route-header testing. |
| WSTG-IDNT — Identity Management | Not tested | No identity creation/proofing/role lifecycle was tested. |
| WSTG-ATHN — Authentication | Not tested | No login, logout, reset, MFA, or credential policy tests. The portal login URL in client text is on the out-of-scope apex host. |
| WSTG-ATHZ — Authorization | Not tested | No role, object, IDOR, entitlement, or payment-bypass checks. |
| WSTG-SESS — Session Management | Not tested | No authenticated cookie/session lifecycle was assessed. |
| WSTG-INPV — Input Validation | Surface inventory only | Visible controls and required attributes recorded; no input was submitted or manipulated. |
| WSTG-ERRH — Error Handling | Not tested | No invalid input or failure condition was generated. |
| WSTG-CRYP — Cryptography | Not assessed | Ordinary HTTPS navigation only; no TLS/cipher or application crypto review. |
| WSTG-BUSL — Business Logic | Static observation only | Client-side BitElevate order, checkout, verification, and credential display path mapped; no action or payment invoked. |
| WSTG-CLNT — Client-side Testing | Partial | DOM/assets and selected bundles inspected; no active XSS, CSP execution, storage, postMessage, or state-manipulation tests. |
| WSTG-APIT — API Testing | Inventory only | No application API response observed. Three trailing-slash paths returned 308; two server-action names were found in public client code, but no action contract or behavior was tested. |

Full test boundaries and category details are in `../reconnaissance/methodology.md` and `../reconnaissance/scope.md`.

## 4. Public attack surface and route inventory

### Browser-rendered public routes

The following 15 routes were rendered read-only in the browser: `/`, `/contact`, `/careers`, `/products`, `/bitelevate`, `/bitelevate/paths`, `/navigation`, `/blog`, `/services`, `/case-studies`, `/diaries`, `/blog/ai-agents-business-operations`, `/privacy`, `/terms`, and `/cookies`. HTTP status codes were not recorded for most browser navigations; do not infer a 200 response code from renderability alone. The homepage separately has a raw GET response record.

The homepage navigation also exposed `/story`, `/team`, and `/impact`; those routes were not independently requested. The detailed route inventory is `../reconnaissance/routes.csv`.

### Sitemap

The sitemap requested from `www.cynbit.in` returned 200 XML (14,264 characters) and contained 82 `<loc>` entries, all using `https://cynbit.in` rather than the requested `www` hostname:

| Sitemap route family | Count |
|---|---:|
| Main pages (including home, policies, hubs) | 16 |
| Blog article routes (excluding `/blog` listing) | 25 |
| Case studies | 13 |
| Careers | 20 |
| Diaries | 8 |
| **Total** | **82** |

These are inventory entries from the sitemap, not 82 visited pages. The apex-host URLs were not requested under the stated host boundary. `lastmod` values span 2024-08-15 through 2026-11-02; the dates are site-provided sitemap metadata and were not independently verified. Full rows are in `../reconnaissance/sitemap-routes.csv` and captured XML is `../evidence/20261102T183937+0530-sitemap.xml`.

### Public metadata and redirect checks

- `GET https://www.cynbit.in/robots.txt`: 200, `text/plain`, 200 characters. It allows `/`, specifies `Crawl-delay: 1`, points to `https://cynbit.in/sitemap.xml`, and disallows `/api/`, `/admin/`, and `/portal/`.
- `GET https://www.cynbit.in/sitemap.xml`: 200 XML; canonical loc host is apex `cynbit.in`.
- `GET https://www.cynbit.in/.well-known/security.txt`: 404.
- `GET https://www.cynbit.in/security.txt`: 404.
- Exactly one GET each to `/api/`, `/admin/`, and `/portal/`: 308 Permanent Redirect to relative locations `/api`, `/admin`, `/portal`, respectively. `Server: Vercel` was present. Redirects were not followed; bodies were not read; no retries were sent. This proves only that the trailing-slash form redirected.

Detailed redirect evidence is `../evidence/20261102T190546+0530-robots-disallowed-path-get-check.txt`; public metadata evidence is `../evidence/20261102T183937+0530-public-metadata-resources.txt`.

## 5. Observed architecture and backend indicators

### Directly observed

- Public assets use `/_next/static/...`; selected bundles use App Router-like route/chunk paths. This strongly indicates a Next.js frontend, but is not a source-code confirmation of the whole deployment.
- The sampled homepage response included `Server: Vercel`, `X-Nextjs-Prerender: 1`, `X-Nextjs-Stale-Time: 300`, and `X-Vercel-Cache: HIT`. Vercel appears to serve/cache the sampled response; the backend implementation and hosting of server actions remain unknown.
- The homepage includes Google tag identifier `G-LK6CQWCWX0`, Google Fonts/Material Symbols references, and same-origin assets.
- Four public form-related JavaScript chunks were inspected as text after their URLs appeared in public-page resources. Each returned 200: contact (24,530 text characters), products (43,981), BitElevate (90,393), and blog (7,523). A simple search found no `/api/`, `/graphql`, or `/trpc` URL literals in these sampled chunks; this does not prove those APIs do not exist.

### BitElevate payment-to-credential client flow (unverified server behavior)

The public BitElevate bundle names `createBitelevateCashfreeOrder` and `verifyBitelevatePayment` as Next.js server references. The inspected client sequence indicates it:

1. Requests order/session creation from the first server action.
2. Reads a returned `paymentSessionId` and environment indicator.
3. Loads/reuses the Cashfree v3 browser SDK and opens checkout using that session ID.
4. After checkout, calls the verification action with `token` and `orderId`.
5. On a success result, can render returned `bitElevateCredentials` values (email/password) and copy an individual value or combined login text to the clipboard.

These action names and references are client-visible by design in the observed bundle and are not a finding by themselves. The code excerpts contained dynamic credential variables, not static credentials; credential-like sample text was redacted in the evidence record. No order, payment, server action, or portal login was invoked. Whether server code authenticates provider state, verifies settled status/amount/currency, binds the order to the applicant, enforces expiry/replay rules, or makes credential issuance idempotent is unknown.

The client text references `https://cynbit.in/intern-portal/login`, which is outside the host scope and was not visited. The evidence artifacts are `../evidence/20261102T1842+0530-bitelevate-server-action-context.txt` and `../evidence/20261102T1842+0530-bitelevate-client-config-review.txt`.

### Backend conclusions

No backend framework, database, API gateway, data model, source implementation, or application security control can be identified reliably from this evidence. In particular, there is no evidence of a Spring Boot backend. Any Java/Spring examples are conditional implementation guidance only.

## 6. Findings

The authoritative JSONL entries are in `../findings/findings.jsonl`. “Confirmed” indicates the exact observation was directly supported by the cited response; it does not mean exploitation was demonstrated.

### SEC-001 — No public security.txt at the checked `www` locations

- **Severity/status/confidence:** Informational / Confirmed / High.
- **Evidence:** Both conventional paths, `/.well-known/security.txt` and `/security.txt`, returned 404 on `www.cynbit.in` (metadata capture at `../evidence/20261102T183937+0530-public-metadata-resources.txt`).
- **Impact:** A researcher may have difficulty identifying a preferred coordinated-disclosure contact. No direct compromise or exploitable weakness is implied.
- **Limitations:** The finding applies only to the two checked paths on `www`; apex and other hosts were not checked.
- **Recommendation:** Publish a monitored RFC 9116 `/.well-known/security.txt` with a verified contact and valid `Expires` value on intended production hosts; optionally redirect `/security.txt` to it.

### SEC-002 — Common browser-hardening headers absent from one homepage response

- **Severity/status/confidence:** Informational / Confirmed / High for the sampled response only.
- **Evidence:** The raw homepage GET returned 200 HTML and the captured response header list did not contain CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, or Permissions-Policy (`../evidence/20261102T184041+0530-homepage-response-headers.txt`). No `meta[http-equiv]` element was observed in the root DOM summary.
- **Positive context:** `Strict-Transport-Security: max-age=63072000` was present.
- **Impact:** This is a defense-in-depth gap if another weakness is present; no XSS, clickjacking, MIME-sniffing exploit, or data disclosure was demonstrated. Other routes or response layers may differ.
- **Recommendation:** Review actual frontend and API response paths; deploy a narrowly tailored CSP in report-only mode before enforcement, add appropriate `frame-ancestors`/legacy framing policy, `X-Content-Type-Options: nosniff`, Referrer-Policy, and a least-privilege Permissions-Policy. Keep HSTS; do not add `includeSubDomains` before validating all subdomains.

### Observations intentionally not classified as findings

- `Access-Control-Allow-Origin: *` was seen only on public homepage HTML and no `Access-Control-Allow-Credentials` was observed. This alone is not reported as a vulnerability; API CORS was not tested.
- Vercel/Next.js response headers and framework indicators are technology disclosure, not a demonstrated security defect.
- A public server-action name/reference is not itself a vulnerability. No server-side payment flaw was observed because the server action was not called or reviewed.
- The absence of API literals in the homepage resource list and four selected bundles does not prove API absence.
- `robots.txt` disallow entries and the resulting 308 redirects do not prove hidden applications exist.

## 7. Authentication, authorization, sessions, and input validation

### Authentication and identity

No authentication workflow was tested. The `/intern-portal/login` URL is merely a text reference in the BitElevate bundle and is on the out-of-scope apex hostname. No password policy, MFA, account recovery, enumeration, or identity-verification conclusion is possible.

### Authorization and sessions

No authenticated state, roles, object identifiers, tenant boundary, or session lifecycle was tested. The homepage response had no `Set-Cookie` header, but that single anonymous response says nothing about authenticated routes. No conclusion about IDOR, privilege escalation, CSRF, cookie flags, token storage, expiration, or logout is warranted.

### Visible data-entry surfaces

The browser DOM inspection recorded controls only; it did not submit, mutate, or validate data:

| Route | Observed controls |
|---|---|
| `/contact` | Required `fullName` (text), `email`, `message`; optional `company`, `industry` |
| `/products` | Required `fullName`, `email`, `company`, `message` |
| `/bitelevate` | Required `fullName`, `university`, `email`, `phone`, `track` (select), `duration` (select), `ambitions`; optional `referralName` |
| `/blog` | Optional `q` search control; required newsletter email |
| `/blog/ai-agents-business-operations` | Required newsletter email |
| `/careers` listing | No form observed on the listing page |

The inspected DOM lacked explicit form `action` and `method` attributes. That does not identify where or how Next.js handlers submit data. Browser `required` flags do not establish server-side validation. No data-validation, injection, persistence, privacy-retention, or error-handling claim can be made from these observations.

## 8. API and hidden-route assessment

The endpoint inventory (`../endpoints/endpoints.csv`) includes 29 rows spanning public page navigations, public metadata, selected JS chunks, the three authorized redirects, two client-side action names, and one out-of-scope portal reference.

| Candidate/surface | Evidence | Conclusion |
|---|---|---|
| `/api/` | One authorized GET returned 308 `Location: /api` | Slash normalization only; slashless target and API behavior not tested |
| `/admin/` | One authorized GET returned 308 `Location: /admin` | Slash normalization only; admin behavior not tested |
| `/portal/` | One authorized GET returned 308 `Location: /portal` | Slash normalization only; portal behavior not tested |
| `createBitelevateCashfreeOrder` | Name/reference in public BitElevate bundle | Server action candidate; method, URL, inputs, response, auth, and behavior unknown; not invoked |
| `verifyBitelevatePayment` | Name/reference in public BitElevate bundle | Server action candidate; method, URL, inputs, response, auth, and behavior unknown; not invoked |
| `https://cynbit.in/intern-portal/login` | String in public client bundle | Out-of-scope apex-host URL; not visited |
| REST / GraphQL / tRPC | No request or application response confirmed in available browser/resource evidence | No endpoint contract inferred; absence not established |

Do not treat route names, robots entries, client references, or redirects as evidence of route existence, authorization flaws, or exploitable behavior.

## 9. Performance and availability observations

- One homepage browser sample at 1046×686 recorded approximately 596.4 ms total navigation duration, 192.3 ms DOMContentLoaded, and 596.4 ms load. This was a single local browser observation with uncontrolled cache/network conditions—not a benchmark or user-experience distribution.
- The same browser sample reported `transferSize=300`, `encodedBodySize=15,613`, and `decodedBodySize=100,660`. A separate raw GET reported `Content-Length: 100660`. These captures differ in cache/compression context and should not be compared as equivalent wire-byte measurements.
- The raw response had `Age: 3735` and `X-Vercel-Cache: HIT`; it represents a cached response, not an origin/backend latency measurement.
- On `/bitelevate/paths`, image URLs used `/_next/image` with `w=1920&q=75` for source URLs containing `i.pravatar.cc/100?img=21/22/23`. This merits an optimization review only; actual transfer dimensions, bytes, rendering impact, and user impact were not measured.
- No repeated timings, Core Web Vitals, uncached/origin comparison, API latency, query analysis, throughput, concurrency, resource saturation, or load test was performed. **No performance finding is confirmed.**

Detailed caveats are in `../performance/observations.md` and `../evidence/public-dom-forms-performance.txt`.

## 10. Recommendations and illustrative implementation guidance

Priorities below are based on the limited evidence; they do not imply that the target uses Java/Spring Boot. Full notes and snippets are in `../findings/recommendations.md`.

### Priority 1 — Publish a coordinated-disclosure contact

Create an RFC 9116 `/.well-known/security.txt` with a verified, monitored contact and current expiration date. Publish only on hosts the owner intends to support. Do not deploy a placeholder contact.

### Priority 1 — Roll out browser security headers

Configure response headers at the actual frontend/edge layer (likely Next.js/Vercel for rendered pages), and separately on any API responses. Start CSP in report-only mode, inspect violations/dependencies, then enforce a restrictive policy. Confirm embedding needs before choosing `frame-ancestors` or `X-Frame-Options`.

Illustrative Java servlet-filter pattern for a Spring backend only:

```java
@Component
final class BrowserSecurityHeadersFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain)
            throws ServletException, IOException {
        response.setHeader("X-Content-Type-Options", "nosniff");
        response.setHeader("Referrer-Policy", "strict-origin-when-cross-origin");
        response.setHeader("Permissions-Policy",
                "camera=(), microphone=(), geolocation=()");
        response.setHeader("X-Frame-Options", "DENY");
        response.setHeader("Content-Security-Policy-Report-Only",
                "default-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'");
        chain.doFilter(request, response);
    }
}
```

This sample omits imports and must be adapted to application behavior. It is not evidence that a Spring component controls the target's frontend headers.

### Priority 1 — Verify payment and credential issuance server-side

Before any authorized black-box test, code-review the server actions and provider adapter. Trust only server-owned order/applicant state and authoritative provider verification; validate settled status, amount, currency, applicant/order binding, expiry, replay resistance, and idempotency. Store issuance with a unique order/application constraint; avoid logging or placing credentials in URLs. Make issued access short-lived and revocable. Use Cashfree's current official server SDK/signature flow; the client must not decide payment success.

Illustrative Java logic (all types/adapters are placeholders):

```java
@Transactional
public IssuedAccess issueAfterPayment(UUID applicantId, String orderId) {
    PaymentOrder expected = orders.findForApplicant(applicantId, orderId)
        .orElseThrow(AccessDeniedException::new);
    VerifiedPayment payment = gateway.fetchAndVerify(orderId);
    if (!payment.isSettled()
            || !payment.amount().equals(expected.amount())
            || !payment.currency().equals(expected.currency())) {
        throw new PaymentNotSettledException();
    }
    // Implement with a database uniqueness constraint for safe retries.
    return accessIssuer.issueOnce(applicantId, expected.id());
}
```

### Priority 2 — Validate public form data on the server

Use allowlisted fields and enum values, enforce max lengths and documented formats, normalize conservatively, reject unexpected content types/fields, encode output by context, and minimize PII in logs. Add CSRF protection where cookie-authenticated state changes exist, and proportionate abuse controls. Test invalid values and error handling only in an authorized environment.

### Priority 2/3 — Complete authorized API and security review

Obtain written authorization for exact hosts/routes, role accounts, test environment, request limits, and allowed test classes. Then verify authentication, authorization, session controls, API CORS, schema validation, error responses, and business invariants without production payment or personal data. The three trailing-slash redirects are not a substitute for this work.

### Priority 3 — Establish performance baselines

Use repeated owner-approved synthetic/RUM and backend telemetry to measure percentiles, cache behavior, Core Web Vitals, payloads, and provider dependencies. Verify actual image variants/delivery. Do not load-test without a written safe plan.

## 11. Evidence and artifact register

| Artifact | Use in this report |
|---|---|
| `../findings/findings.jsonl` | Authoritative two-item informational finding ledger |
| `../findings/category-assessment.md` | Separates findings from untested and non-finding observations |
| `../findings/recommendations.md` | Expanded remediations and illustrative Java examples |
| `../reconnaissance/scope.md` | Authorization, host boundary, and excluded tests |
| `../reconnaissance/methodology.md` | WSTG v4.2 category mapping and test limitations |
| `../reconnaissance/routes.csv` | Selected rendered/discovered routes and route status |
| `../reconnaissance/sitemap-routes.csv` | 82 sitemap entries; all canonical apex-host URLs marked unrequested |
| `../endpoints/endpoints.csv` | Public GETs, redirect results, static chunks, action candidates, out-of-scope portal reference |
| `../evidence/20261102T184041+0530-homepage-response-headers.txt` | Homepage status, content size, response headers; primary SEC-002 evidence |
| `../evidence/20261102T183937+0530-public-metadata-resources.txt` | Robots, sitemap response excerpt, security.txt 404 evidence |
| `../evidence/20261102T183937+0530-sitemap.xml` | Captured 82-location sitemap XML |
| `../evidence/20261102T190546+0530-robots-disallowed-path-get-check.txt` | Exactly three authorized GETs and 308 redirects; no redirect following |
| `../evidence/20261102T184119+0530-public-form-client-bundles.txt` | Four public bundle status/size and URL-literal observations |
| `../evidence/20261102T1842+0530-bitelevate-server-action-context.txt` | Payment-action/client-credential flow excerpts; dynamic values redacted |
| `../evidence/20261102T1842+0530-bitelevate-client-config-review.txt` | Read-only string checks on the BitElevate bundle |
| `../evidence/public-dom-forms-performance.txt` | Route, form-control, DOM, and single timing sample notes |
| `../evidence/20261102T183937+0530-homepage-response-headers.txt` | Earlier incomplete header record; superseded by the complete 18:40 capture |
| `../architecture/observations.md` | Observed versus inferred architecture notes |
| `../performance/observations.md` | Single-sample performance evidence and limitations |

No screenshot, full homepage HTML capture, authenticated response, application API response, payment-provider response, or error trace is present. The report intentionally does not invent evidence for those areas.

## 12. Overall conclusion

The available evidence supports a limited external assessment of public web configuration and client-side surface discovery. Two low-impact informational hardening observations are recorded. The observed Next.js/Vercel indicators and BitElevate client flow provide useful leads for a properly authorized backend review, but do not establish backend implementation or vulnerability. Authentication, authorization, input handling, payment enforcement, API contracts, and performance remain materially unassessed. Treat this report as a scoped evidence handoff, not a clean bill of health or a full penetration-test attestation.