# Recommendations and illustrative implementation patterns

Priorities below address observed hardening gaps and unverified trust boundaries. They are recommendations, not claims about Cynbit's current backend stack. The public evidence indicates a Next.js/Vercel-served frontend; no Spring Boot backend was identified. Apply controls at the actual response-serving layer (edge/Next.js for frontend documents; backend for API responses).

## P1 — Publish a monitored security.txt

Publish `/.well-known/security.txt` on each in-scope production host per RFC 9116, with a monitored disclosure contact and a valid `Expires` date. Optionally redirect `/security.txt` to the well-known path. Do not publish a placeholder contact or include private credentials. Retest only the explicitly authorized hosts.

Illustrative file (replace all placeholders with verified values):

```text
Contact: mailto:security@example.invalid
Expires: 2027-11-02T00:00:00.000Z
Preferred-Languages: en
```

The `.invalid` domain is intentionally non-routable and is not a usable production contact.

## P1 — Add and safely roll out browser defense headers

The sampled public homepage response omitted several defense-in-depth headers. Configure them consistently at the frontend/edge and API layers as applicable. Start CSP in report-only mode, inventory required scripts/styles/images/frames, resolve violations, then enforce a narrowly scoped policy. Avoid adding `unsafe-inline`/`unsafe-eval` reflexively. Preserve HSTS; decide `includeSubDomains` only after confirming all subdomains are HTTPS-capable. `frame-ancestors` should reflect legitimate embedding requirements; `X-Frame-Options` may be retained as a legacy fallback.

Illustrative Spring servlet filter only (not evidence the target uses Spring; adapt the policy to the real application and test routes):

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

Imports omitted for brevity: Spring `OncePerRequestFilter` and Jakarta Servlet types. On a Next.js/Vercel frontend, prefer its deployment/header configuration or middleware for document responses rather than assuming an unrelated Spring filter controls those headers.

## P1 — Enforce payment and credential issuance invariants on the server

Before enabling an authorized payment test, code-review or verify that server-side logic:

1. Creates an order from server-owned applicant/product/price data; never trusts client-supplied amount, currency, environment, or paid state.
2. Verifies provider authenticity using the official Cashfree SDK/signature mechanism, and/or fetches authoritative order/payment state server-to-server.
3. Requires final successful settlement, expected amount and currency, and an order bound to the intended applicant before granting access.
4. Makes issuance idempotent with a unique order/application constraint; treats duplicate callbacks/verification requests safely.
5. Rejects expired, cancelled, mismatched, or already-consumed order tokens and avoids credentials in URLs, logs, analytics, or clipboard telemetry.
6. Issues least-privilege, time-bounded credentials through a protected channel and provides rotation/revocation.

Illustrative Java service logic (provider and repository APIs are placeholders; use Cashfree's current official server SDK and verify the exact signed fields):

```java
@Transactional
public IssuedAccess issueAfterPayment(UUID applicantId, String orderId) {
    PaymentOrder expected = orders.findForApplicant(applicantId, orderId)
        .orElseThrow(AccessDeniedException::new);

    // Provider adapter must verify authentic provider state server-to-server.
    VerifiedPayment payment = gateway.fetchAndVerify(orderId);
    if (!payment.isSettled()
            || !payment.amount().equals(expected.amount())
            || !payment.currency().equals(expected.currency())) {
        throw new PaymentNotSettledException();
    }

    // Backed by a unique constraint on payment_order_id; safe on retries.
    return accessIssuer.issueOnce(applicantId, expected.id());
}
```

Do not expose a paid/verified boolean to the browser as authority. Prefer opaque, short-lived references; the client-visible Next server-action identifiers are not, by themselves, a flaw.

## P2 — Define server-side validation for public forms

For contact/product/application/newsletter inputs, validate on the server regardless of browser attributes: allowlist known fields, normalize conservatively, enforce documented max lengths and enum choices, validate email/phone formats without brittle over-rejection, reject unexpected content types/fields, encode output for its context, and avoid logging full personal data. Use CSRF protections for cookie-authenticated state changes, abuse controls sized to normal use, and clear privacy/retention rules. Verify errors do not expose stack traces. These controls were not tested here.

## P2 — Establish measured performance baselines

Use owner-approved production RUM/synthetic monitoring and backend traces to collect repeated p50/p95/p99 latency, cache-hit behavior, Core Web Vitals, payload sizes, and payment/API dependencies. Check image `srcset`/actual delivered dimensions for the observed avatar URL pattern. Do not infer origin or scalability from the single cached sample; load testing requires a separately approved safe plan.

## P3 — Reassess CORS and headers across actual API responses

The only wildcard CORS evidence is a public HTML response without credentialed CORS. Inventory actual APIs in an authorized environment and set explicit allowed origins/methods/headers; avoid reflecting arbitrary origins. Reassess security headers on HTML and API error/success paths.
