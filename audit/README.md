# Cynbit external audit dataset

Target: `https://www.cynbit.in/`

This dataset records an external, unauthenticated, predominantly passive review using OWASP WSTG v4.2 as the primary methodology. Broad active-testing authorization and controlled test accounts were not supplied; the user did explicitly authorize one metadata-only GET each to `/api/`, `/admin/`, and `/portal/` on the `www` host. Work otherwise remains limited to public pages/resources. No forms are submitted, no requests are altered or replayed, no authentication or authorization boundaries are tested, and no load/security scanners are run. See `reconnaissance/methodology.md` for the coverage matrix.

`findings/findings.jsonl` is the authoritative finding ledger. It contains one JSON object per evidence-backed finding using fields `id`, `title`, `category`, `severity`, `status`, `confidence`, `evidence`, `impact`, `remediation`, and `references`. Severity values are `Critical`, `High`, `Medium`, `Low`, and `Informational`; status values are `Confirmed` and `Needs Validation`. Notes distinguish observed facts from inferences and untested hypotheses. Items that are only review gaps are kept outside the finding ledger.

Timestamps use ISO 8601 with the machine's local offset (+05:30). Evidence must omit or redact credentials, session tokens, and personal data.

## map
- `summaries/backend-technical-audit-report.md`: concise structured backend/security/performance handoff.
- `summaries/detailed-audit-report.md`: consolidated report analyzing the full audit dataset and its evidence.
- `summaries/coverage.md`: test coverage and explicit not-tested areas.
- `findings/recommendations.md`: prioritized actions and illustrative Java/Spring examples (not a claim that the target uses Spring Boot).
- `endpoints/endpoints.csv`: observed public GET surfaces, the three specifically authorized redirect checks, and server-action candidates discovered passively.
- `reconnaissance/sitemap-routes.csv`: 82 sitemap entries; apex-host URLs were not requested because only `www.cynbit.in` was in scope.
- `evidence/`: evidence records and their limitations.