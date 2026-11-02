# Cynbit Technologies — Detailed Backend Audit Report

**Prepared by:** Aman Verma  
**Role:** Backend Developer Intern  
**Target:** https://www.cynbit.in/  
**Audit date:** 2 November 2026, 21:00 (+05:30)  
**Scope:** External, unauthenticated review of the www host

## 1. Executive summary

This audit reviewed Cynbit’s publicly accessible website from a backend security perspective. It recorded public routes and application surfaces, inspected selected client-side bundles and response headers, and observed the BitElevate payment-to-credential flow in client code.

Two confirmed **Informational** security-hardening findings were identified:

1. The two checked security.txt locations on the www host returned 404.
2. Five common browser-security headers were absent from the single captured homepage response.

No exploitable vulnerability was confirmed within the permitted scope. This does not mean the backend is secure: authentication, authorization, server-side input validation, payment verification, and credential issuance were not exercised.

The highest-value follow-up is the BitElevate workflow. The public client code references Cashfree order creation and payment verification, and can display dynamic credentials after a success result. The server-side implementation and its security controls were not tested.

| Audit measure | Result |
|---|---:|
| Public routes rendered | 15 |
| Sitemap entries inventoried | 82 |
| Public JavaScript bundles inspected | 4 |
| Endpoint and application-surface records | 29 |
| Confirmed Informational findings | 2 |
| Confirmed exploitable vulnerabilities | 0 |

## 2. Scope, coverage, and limits

### 2.1 Audit scope

The review was limited to the public, unauthenticated surface of **www.cynbit.in**. The 15 rendered routes were:

- Home
- Contact
- Careers
- Products
- BitElevate
- BitElevate paths
- Navigation
- Blog
- Services
- Case studies
- Diaries
- One blog article
- Privacy policy
- Terms and conditions
- Cookie policy

The public sitemap contained 82 URL entries. Its URLs used the apex host, **cynbit.in**; those pages were not requested because the authorized host was **www.cynbit.in**. The sitemap entries are an inventory, not 82 visited routes.

Exactly one metadata-only GET was authorized for each of **/api/**, **/admin/**, and **/portal/** on the www host. Each returned a 308 redirect to its slashless path. The redirects were not followed, and they do not establish that an API, admin interface, or portal exists at the destination.

### 2.2 Methodology

The primary investigation techniques included:

- Browser-based application reconnaissance.
- HTTP request and response inspection.
- Public resource and metadata analysis.
- Client-side bundle inspection and security-header analysis.
- Route and endpoint inventory; static examination of publicly available application logic.

No source code, internal infrastructure, test accounts, or broad active-testing authorization was supplied.

No brute-force attacks, destructive payloads, authenticated testing, payment transactions, privilege-escalation attempts, or unauthorized data access were performed.

### 2.3 Coverage

| **Audit Area** | **Coverage** |
|---|---|
| Information gathering | Partial |
| Configuration and deployment | Partial |
| AuthN / AuthZ / Sessions | Not tested |
| Input validation and logic | Surface and static analysis |
| Errors, cryptography, and performance | Not assessed / Observational |
| Client-side and API surface | Partial / Inventory only |

## 3. Public application and backend indicators

### Observed frontend indicators

Public assets and response headers strongly indicate a Next.js frontend served through Vercel. This does not identify the backend framework, database, internal APIs, or where server actions execute.

The sampled homepage response included:

| Header | Observed value |
|---|---|
| Server | Vercel |
| X-Nextjs-Prerender | 1 |
| X-Nextjs-Stale-Time | 300 |
| X-Vercel-Cache | HIT |
| Strict-Transport-Security | max-age=63072000 |

The homepage also referenced Google Analytics, Google Fonts, Material Symbols, and same-origin assets. These are technology and configuration observations, not findings by themselves.

### Backend limits

No REST, GraphQL, or tRPC application response was confirmed. Four selected public JavaScript bundles were inspected; limited searches did not identify literal URLs for those API styles. This does not prove that such APIs do not exist.

The client-visible Next.js server-action references do not reveal their request contracts, server-side controls, storage model, or internal service boundaries. No reliable conclusion can be made about Cynbit’s internal backend architecture from the available public evidence.

## 4. Confirmed security findings

### SEC-001 — No public security.txt at the checked www locations

**Severity:** Informational  
**Status:** Confirmed  
**Confidence:** High

The following paths returned HTTP 404 on **www.cynbit.in**:

- /.well-known/security.txt
- /security.txt

**Potential impact:** Security researchers may not have a clearly identifiable channel for coordinated vulnerability disclosure. This is a security-hygiene concern; it does not demonstrate compromise or an exploitable vulnerability.

**Recommendation:** Publish an RFC 9116 security.txt file at /.well-known/security.txt on each intended public production host. Include a monitored contact, a valid expiration date, and relevant reporting instructions. Optionally redirect /security.txt to the well-known path.

### SEC-002 — Common browser-security headers absent from the sampled homepage response

**Severity:** Informational  
**Status:** Confirmed  
**Confidence:** High for the captured response only

The single captured homepage response did not include:

| Header not observed | Typical purpose |
|---|---|
| Content-Security-Policy | Restricts resource loading and can reduce impact from some injection flaws |
| X-Frame-Options | Legacy framing protection |
| X-Content-Type-Options | Prevents MIME-type sniffing |
| Referrer-Policy | Controls referrer information disclosure |
| Permissions-Policy | Restricts browser capabilities |

The response did include **Strict-Transport-Security: max-age=63072000**, which is a positive observation.

**Potential impact:** The missing headers reduce defense in depth if a separate weakness exists. No XSS, clickjacking, MIME-sniffing exploit, or data disclosure was demonstrated. This finding applies to the captured homepage response; other routes or response layers may differ.

**Recommendation:** Review actual frontend and API responses. Introduce Content-Security-Policy in report-only mode first, validate the required application dependencies and framing behavior, and then move to enforcement. Do not apply restrictive defaults without checking product requirements.

Illustrative response-header configuration:
```http
Content-Security-Policy-Report-Only: default-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

This example is not production-ready as written. Confirm required third-party origins and framing needs before adopting these values.

## 5. BitElevate payment-to-credential workflow

### What the public client code showed

The BitElevate bundle references two Next.js server actions:

- **createBitelevateCashfreeOrder**
- **verifyBitelevatePayment**

The observed client-side sequence is:

1. The frontend requests payment-order or session creation.
2. The client receives a paymentSessionId and an environment indicator.
3. The Cashfree browser SDK is loaded or reused, and checkout is opened.
4. After checkout, the client invokes the verification action with token and orderId.
5. A successful result can provide BitElevate credentials for display and copying in the browser.
6. Credential values are dynamic email/password values; no static credentials were identified in the inspected evidence.

Neither server action was invoked. No payment was attempted, and no credential issuance was triggered. The action names and client references are not vulnerabilities by themselves.

### What remains unverified

The server-side implementation was not available to inspect. It is unknown whether the backend:

- Independently obtains the authoritative payment status from Cashfree.
- Checks the expected order, applicant, amount, currency, and settlement state.
- Enforces expiry and replay protection.
- Makes repeated verification and credential issuance idempotent.
- Protects issued credentials from unnecessary exposure in logs, URLs, analytics, or client persistence.

These are high-priority review questions, not confirmed defects.

### Recommended payment and credential controls

For an authorized code review or safe test environment, verify that the server creates orders from trusted applicant and pricing records; confirms payment through the provider’s server-side interface; checks ownership, amount, currency, and final status; prevents replay; and issues credentials only after successful verification. Persist issuance state with constraints that make retries safe. Protect credentials in storage and logs.

The [separate inferred architecture note](INFERRED_ARCHITECTURE.md) presents one possible design for these requirements. It is not a description of the deployed backend.

## 6. API and data-handling observations

### Authorized path checks

| Requested path | Result |
|---|---|
| /api/ | 308 redirect to /api |
| /admin/ | 308 redirect to /admin |
| /portal/ | 308 redirect to /portal |

The redirects were not followed. They demonstrate trailing-slash normalization only.

Four public JavaScript bundles associated with Contact, Products, BitElevate, and Blog functionality were inspected. No literal REST, GraphQL, or tRPC endpoint URLs were identified in the limited searches. This does not establish that the application has no backend APIs.

The client bundle referenced **https://cynbit.in/intern-portal/login**. That apex-host URL was not visited because it was outside the authorized www host boundary.

### Visible forms and controls

| Route | Publicly visible form fields or controls |
|---|---|
| Contact | Required name, email, and message; optional company and industry |
| Products | Required name, email, company, and message |
| BitElevate | Required name, university, email, phone, track, duration, and ambitions; optional referral name |
| Blog | Search control and newsletter email |
| Blog article | Newsletter email |

The inspected forms did not expose explicit HTML action and method attributes. No form was submitted. The observations do not establish where requests are processed or whether server-side validation is present.

## 7. Performance and reliability observations

One browser navigation sample for the homepage at a 1046 × 686 viewport recorded:

| Metric | Sample |
|---|---:|
| Navigation duration | ~596 ms |
| DOMContentLoaded | ~192 ms |
| Load event | ~596 ms |

The sampled response was cached by Vercel. This is one uncontrolled browser sample, not a benchmark or a measurement of uncached origin or backend latency.

A BitElevate paths page included Next.js image URLs with **w=1920&q=75** for source URLs containing **i.pravatar.cc/100?img=21/22/23**. This may be worth an optimization review, but actual transfer dimensions, bytes, and user impact were not measured.

No repeatable timing study, Core Web Vitals capture, backend latency measurement, concurrency test, or load test was performed. No performance vulnerability or confirmed performance finding was identified.

## 8. Recommended follow-up

| Priority | Recommendation | Rationale |
|---|---|---|
| P1 | Publish a monitored security.txt and review browser-security headers | Improves disclosure readiness and browser defense in depth |
| P1 | Review the BitElevate payment-verification and credential-issuance boundary | Checks payment integrity, order ownership, and credential protection |
| P2 | Review server-side validation, authentication, sessions, and authorization | These controls were not exercised |
| P3 | Establish repeatable performance baselines and review image delivery | The available performance sample is not conclusive |

A deeper authorized review could cover authentication and authorization, role enforcement, resource ownership and IDOR, session and token security, API validation, controlled request manipulation, error handling, payment/order integrity, replay resistance, and credential issuance. It should use written authorization that defines the permitted hosts, routes, accounts, test environment, and test types.

I’d be happy to conduct that deeper review with advance notice and explicit written scope. Contact: **[work.with.aman.verma@gmail.com](mailto:work.with.aman.verma@gmail.com)**.

## 9. Evidence and conclusion

The audit confirmed two Informational hardening findings and identified the BitElevate payment-to-credential flow as an important area for backend review. No exploitable vulnerability was confirmed. Authentication, authorization, input validation, payment enforcement, API behavior, and repeatable performance remain untested; they should not be read as either secure or vulnerable based on this report.

The detailed evidence is available in the [finding ledger](audit/findings/findings.jsonl), [endpoint inventory](audit/endpoints/endpoints.csv), [rendered-route inventory](audit/reconnaissance/routes.csv), and [evidence index](audit/evidence/README.md).

The factual audit should be read separately from [Inferred Backend Architecture](INFERRED_ARCHITECTURE.md), which contains a clearly labeled proposed design.

**Prepared by:** Aman Verma  
**Backend Developer | AI Engineer**  
**Contact:** work.with.aman.verma@gmail.com