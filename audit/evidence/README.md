# Evidence index

All listed evidence comes from public, read-only inspection on 2026-11-02 (+05:30); no form, payment, or server action was invoked. Evidence files retain only observations needed for this audit and avoid applicant data, credentials, or payment tokens.

- `20261102T184041+0530-homepage-response-headers.txt` — raw GET response metadata for `https://www.cynbit.in/`, status 200, including sampled response headers and content length.
- `20261102T183937+0530-homepage-response-headers.txt` — earlier incomplete header capture; superseded by the later complete file above.
- `20261102T183937+0530-public-metadata-resources.txt` — `www` robots.txt/sitemap response, plus 404 observations for `www` security.txt locations. Contains the sitemap XML response excerpt; canonical URLs point to apex host.
- `20261102T183937+0530-sitemap.xml` — captured public sitemap XML; 82 loc entries all use `https://cynbit.in`, and those canonical-host URLs were not requested.
- `20261102T184119+0530-public-form-client-bundles.txt` — response status/size and simple URL-literal observations from four public page bundles fetched from observed asset URLs.
- `20261102T1842+0530-bitelevate-server-action-context.txt` — redacted snippets showing public client references to payment actions and dynamic credential rendering; no action invoked.
- `20261102T1842+0530-bitelevate-client-config-review.txt` — static text-search notes from the same public bundle; credential-like output is redacted.
- `20261102T190546+0530-robots-disallowed-path-get-check.txt` — response status/redirect metadata for the three exact trailing-slash paths the user explicitly authorized for one GET each. Redirects were not followed; response bodies were not read.
- `public-dom-forms-performance.txt` — summarized browser observations of inspected pages, form control names, and one limited timing sample; no form data was entered.

No screenshot, application API response, authenticated response, application error response, or payment provider response was captured. The three redirect responses are transport-level path-normalization results only. Findings must be interpreted within these limitations.
