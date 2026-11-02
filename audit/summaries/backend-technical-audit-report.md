# Backend Technical Audit Report — structured handoff

**Target:** `https://www.cynbit.in/`  
**Assessment mode:** external, unauthenticated, predominantly passive; one explicitly authorized GET each to three exact trailing-slash paths  
**Methodology:** OWASP WSTG v4.2 category mapping (`../reconnaissance/methodology.md`)  
**Evidence date:** 2026-11-02 (+05:30)

This is a dataset-oriented technical handoff, not a claim of a full penetration test. No source code, infrastructure, authenticated account, or test account was supplied. Broad active-testing authorization was not supplied; the only specific active allowance was one unauthenticated metadata-only GET to each of `/api/`, `/admin/`, and `/portal/` on `www.cynbit.in`. Static public asset inspection was also performed.

## Executive disposition

- **Confirmed ledger items:** 2 informational hardening observations; no confirmed exploitable vulnerability was established.
- **High-priority unverified review item:** public BitElevate client code indicates payment order creation, checkout, payment verification, and credential issuance. Server-side authorization, provider verification, applicant/order binding, replay handling, and idempotency remain unknown because the actions were not invoked.
- **No confirmed API endpoint behavior:** no REST, GraphQL, or tRPC response was captured. One GET each to the robots-disallowed trailing-slash paths `/api/`, `/admin/`, and `/portal/` returned a 308 redirect to the slashless path; no redirect was followed. Two Next.js server-action names are visible in a public bundle; their request contracts and server behavior are unknown.
- **Authentication, authorization, session security, and server-side data validation:** not tested. The visible forms provide surface inventory only.
- **Performance:** not benchmarked. One uncontrolled browser navigation sample was served from a cache; no backend latency or scalability conclusion is possible.

## Surface and architecture observations

Observed public paths rendered read-only are listed in `../reconnaissance/routes.csv`; 82 additional sitemap loc entries are in `../reconnaissance/sitemap-routes.csv`. The sitemap served from `www` advertises apex-host (`cynbit.in`) URLs; those URLs were not requested because they were outside the explicitly provided host scope. `robots.txt` names `/api/`, `/admin/`, and `/portal/` as disallowed; each exact trailing-slash path received its one authorized GET, returned a 308, and was not followed to the slashless path.

Public `/_next/static/...` assets and response headers are strongly consistent with a Next.js frontend deployed on Vercel (`Server: Vercel`, `X-Nextjs-Prerender`, `X-Vercel-Cache`). This does not identify backend services, databases, or the implementation of server actions. A Google tag ID `G-LK6CQWCWX0` was present on the homepage.

The public form surfaces recorded in `../evidence/public-dom-forms-performance.txt` are `/contact`, `/products`, `/bitelevate`, `/blog`, and one blog article. Required/optional fields were noted, but no action/method attributes or validation behavior were established. The separate `/intern-portal/login` address is only a source-code reference to the apex host and was not visited.

## Findings by requested category

| Category | Result |
|---|---|
| Security configuration | Two informational observations: no `security.txt` at either checked `www` path (SEC-001); five common browser hardening headers absent on the sampled root response (SEC-002). HSTS was present. |
| Authentication | Not assessed. No login, reset, registration, MFA, or credential policy tests. |
| Authorization | Not assessed. No IDOR/RBAC/payment-entitlement test. BitElevate flow is an unverified review item, not a confirmed defect. |
| Session management | Not assessed. No authenticated session or cookie lifecycle review. No `Set-Cookie` on the one public homepage response is not evidence about authenticated routes. |
| Input/data validation | Form controls and required flags recorded; no inputs sent. Server-side validation, injection resistance, error handling, and persistence behavior are unknown. |
| API/backend | No API behavior confirmed; two named Next.js server actions are only static client observations. |
| Performance | No confirmed issue. One browser timing sample and a cached Vercel response are insufficient for a finding. The `w=1920&q=75` avatar URL pattern is a possible optimization review only. |

See `../findings/category-assessment.md`, `../findings/findings.jsonl`, and `../findings/recommendations.md` for evidence, statuses, and actions.

## Evidence-backed response observations

- Raw `GET https://www.cynbit.in/`: status 200, `Content-Type: text/html`, `Content-Length: 100660`; `X-Vercel-Cache: HIT`, `Age: 3735`, `Strict-Transport-Security: max-age=63072000`, and `Access-Control-Allow-Origin: *`.
- The sampled HTML response did not include `Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`, or `Permissions-Policy`. This is a per-response observation; no exploitation or XSS/clickjacking condition was demonstrated.
- Wildcard CORS on public homepage HTML, with no `Access-Control-Allow-Credentials` observed, is not reported as a vulnerability. API CORS behavior was not tested.
- `GET /robots.txt` and `/sitemap.xml` on the `www` host returned 200. Both `GET /.well-known/security.txt` and `GET /security.txt` on `www` returned 404. One explicitly authorized GET each to `/api/`, `/admin/`, and `/portal/` returned 308 with a relative `Location` to its slashless form. Those targets were not requested; no backend behavior or route existence is inferred from the redirect alone.

## Recommended next steps

1. Publish a monitored RFC 9116 `security.txt` on in-scope production hosts.
2. Roll out missing browser headers safely, beginning with CSP report-only and verifying real frontend dependencies/embedding requirements.
3. Have the system owner review the BitElevate server-action implementation and payment provider integration against the invariants in `../findings/recommendations.md`; conduct black-box verification only under written, route-specific authorization using test accounts/orders.
4. With authorization, test form validation, auth/session behavior, access control, API contracts, errors, rate controls, and client-side controls in a defined non-production environment.
5. Establish repeatable RUM/synthetic/backend performance measurements before optimization or load testing.

## Limitations and interpretation

All unseen behavior is unknown, not presumed secure or insecure. In particular, public server-action names do not prove vulnerable server handling, required browser form attributes do not prove server validation, a robots disallow entry does not prove a route exists, and a single cached browser sample does not establish performance. Do not expand the target to apex, subdomains, Cashfree, or the intern portal without written authorization.
