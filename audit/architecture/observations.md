# Backend / architecture observations

## Observed
- Public-facing pages describe Cynbit Technologies as providing AI automation, custom software, business platforms, cloud systems, and the BitElevate ecosystem.
- Rendered routes and public bundle names include `/_next/static/...`; the site is strongly consistent with a Next.js App Router frontend. Homepage response headers include `Server: Vercel`, `X-Nextjs-Prerender: 1`, and `X-Vercel-Cache: HIT` for the sampled response.
- The public root response contains a Google tag with ID `G-LK6CQWCWX0`, Google Fonts/Material Symbols references, and same-origin assets.
- The public BitElevate page bundle references Next.js server actions named `createBitelevateCashfreeOrder` and `verifyBitelevatePayment`, loads Cashfree's v3 SDK as part of a client flow, and handles a `paymentSessionId`, then `token`/`orderId`, followed by a success result that can include dynamic credentials. Client code shows a portal URL under the apex host. No server action or payment was invoked; the only portal-path request was the separately authorized GET to `www.cynbit.in/portal/`, which returned a redirect and was not followed. The apex-host login URL was not visited.
- No REST, GraphQL, or tRPC endpoint was confirmed in observed browser requests or the inspected small set of page bundles. Absence from these samples does not establish absence from the application.

## Inferred / not verified
- The public frontend appears content/marketing-oriented, with contact/newsletter/application surfaces and an apparent payment-to-credential issuance flow. These observations do not identify the server framework, database, hosting of backend actions, business rules, or security controls.
- Next.js and Vercel are technology indicators, not a confirmed complete backend architecture. No Spring Boot backend is evidenced; Spring examples in recommendations are illustrative only.
- The payment workflow is a high-priority authorization-dependent review item. The client-visible action names are not themselves a vulnerability. Whether server-side code validates provider authenticity, payment status/amount/currency, order-to-applicant binding, expiry/replay, or idempotency is unknown.
- Exact action URLs/methods/payloads/responses are not known because server actions were not invoked. `https://cynbit.in/intern-portal/login` is an apex-host URL discovered in source and was not visited because it is outside the provided `www` host scope.
