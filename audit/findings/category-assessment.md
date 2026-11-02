# Finding and category audit

This document separates evidence-backed observations from untested areas. The authoritative finding records are in `findings.jsonl`.

| Category | Evidence-backed result | Classification |
|---|---|---|
| Security configuration / disclosure | `www` security.txt paths returned 404; see SEC-001 | Confirmed informational hygiene finding |
| Browser security headers | Sampled homepage response lacks CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, and Permissions-Policy; see SEC-002 | Confirmed informational hardening observation limited to one response |
| CORS | Wildcard ACAO observed on public homepage HTML, without ACA-Credentials | Not a finding on this evidence; public HTML only, API behavior unknown |
| TLS/HSTS | HSTS present on sampled homepage response (`max-age=63072000`) | Positive observation only; no TLS audit |
| Authentication | No auth workflow tested; portal route on apex host was not visited | Not assessed, not a finding |
| Authorization / IDOR | No object/role/payment authorization test performed | Not assessed, not a finding |
| Session security | No authenticated session/cookie lifecycle assessed; no Set-Cookie on sampled public homepage response | Not assessed, not a finding |
| Input/data validation | Form controls and required attributes observed; no submission or invalid input | Not assessed, not a finding |
| Business logic / payment | Public client code maps order/checkout/verification/credential flow | NEEDS AUTHORIZED REVIEW; no exploit or validation defect observed |
| API security | No API request confirmed; two Next.js action references only | Not tested; no endpoint behavior inferred |
| Performance | One uncontrolled cached browser sample; no repeatable study | No confirmed performance finding |

The payment-flow review gap is deliberately not entered as a vulnerability in the ledger: backend enforcement is unknown, and static client references alone do not establish a defect.
