# Scope and rules of engagement

- Target provided by user: `https://www.cynbit.in/`
- Assessment perspective: external, unauthenticated user.
- Authorization status: user initially limited work to passive and authorized testing and prohibited exploitation, disruption, and overload. No broad active-test authorization, test accounts, or scope beyond the provided website was established. The user subsequently authorized exactly one metadata-only GET each to three specified paths, as documented below.
- Allowed here: ordinary, low-volume reads of publicly accessible pages and standard public metadata/resources (HTTP GET only).
- Excluded: form submission, account creation/login/password reset, authenticated activity, API mutation/replay/tampering, IDOR/RBAC tests, injection payloads, brute force, scanning, fuzzing, rate-limit tests, load tests, destructive actions, or requests to non-public/inferred hosts.
- Consequence: authentication, session, authorization, input-validation and rate-limit properties are untested and must not be represented as findings.
- Primary methodology: OWASP Web Security Testing Guide (WSTG) v4.2; mapping and exclusions are recorded in `methodology.md`.
- Host boundary: only `www.cynbit.in` was treated as authorized. Although robots.txt and the sitemap on `www` name the apex host `cynbit.in`, those apex URLs were not requested. External providers and the portal URL seen in client code were not navigated to.
- Robots boundary: `/api/`, `/admin/`, and `/portal/` are listed as disallowed in public robots.txt. After the explicit limited authorization noted below, each exact trailing-slash URL received one GET; the slashless redirect targets were not requested. Their listing or redirect is not evidence those paths serve an application.
- Limited exception subsequently authorized by the user: exactly one unauthenticated GET each to `https://www.cynbit.in/api/`, `/admin/`, and `/portal/`, with response status/redirect metadata only. Each returned 308 to its slashless counterpart. Redirects were not followed, bodies were not read, and no further requests were sent. These observations do not establish an API/admin/portal service exists.
- Data handling: no credentials, form submissions, user records, or payment tokens were collected. Dynamic credential variable names in public client code were not populated or invoked.

Started: 2026-11-02T18:31:39.4145725+05:30 (workspace local time).
