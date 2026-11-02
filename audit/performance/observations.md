# Performance observations

## Available passive evidence

- One browser navigation sample for the homepage at viewport 1046x686 recorded duration about 596.4 ms, DOMContentLoaded about 192.3 ms, and load about 596.4 ms. The local browser profile/cache and network conditions were uncontrolled; this is not a benchmark or representative timing distribution.
- That browser sample reported `transferSize=300`, `encodedBodySize=15,613`, and `decodedBodySize=100,660` bytes. Separately, a raw public GET response reported `Content-Length: 100660`. These are different capture paths/cache/compression observations and must not be compared as equivalent wire-size measurements.
- The raw homepage response had `Age: 3735` and `X-Vercel-Cache: HIT`, so the captured response was served from a cache. It does not demonstrate uncached origin performance or backend latency.
- A BitElevate paths page included `_next/image` avatar URLs with `w=1920&q=75` for sources whose URLs contain `i.pravatar.cc/100?img=21/22/23`. This is a potential image-sizing review item only; actual downloaded dimensions/bytes and user impact were not measured.

## Assessment

No repeatable timing study, Core Web Vitals capture, payload budget, origin comparison, query analysis, concurrency/load test, or backend latency assessment was performed. No performance finding is confirmed. Use representative synthetic monitoring and application telemetry with owner authorization before drawing performance conclusions; do not load test this target without explicit approval and a safe test plan.
