# Cynbit Technologies — Backend Audit

**Aman Verma · Backend Developer Intern · 2 November 2026**

> **The BitElevate page can display credentials after checkout. What did this audit actually verify about the payment behind them?**

**Find out what was observed, where the evidence stops, and what to review next in the [Detailed Audit Report](DETAILED_AUDIT.md).**

## Audit snapshot

| | |
|---|---:|
| Confirmed findings | 2 Informational |
| Exploitable vulnerabilities confirmed | 0 |
| Public routes rendered | 15 |
| Public JavaScript bundles inspected | 4 |
| Sitemap entries inventoried | 82, all on the out-of-scope apex host |
| Endpoint and application-surface records | 29; not 29 confirmed APIs |

### What I found

- **SEC-001:** Both checked security.txt paths on www.cynbit.in returned 404.
- **SEC-002:** One homepage response lacked five common browser-hardening headers; HSTS was present. No exploit was demonstrated.
- The public BitElevate bundle references Cashfree order creation, payment verification, checkout, and dynamic credential display. The server-side checks were not tested.

### What I inferred

The client code suggests a payment-to-credential flow. It does not prove how the server verifies payment, binds orders to applicants, prevents replay, or issues credentials. Those are follow-up questions, not confirmed vulnerabilities.

### Why the scope matters

The brief asked me to:

> “Identify potential issues or opportunities related to API behaviour, data handling, security, performance, authentication, error handling, or backend architecture.”

It felt a little like being asked to reverse-engineer the entire engine platform with the hood still latched. With REA, I could reverse-engineer and rebuild the platform end to end, then use my backend skills to compare optimization techniques and track latency and performance in Grafana. That is a broader engagement requiring access and explicit authorization; this audit stays with evidence the permitted scope could support.

### Suggested next steps

Publish a monitored security.txt, roll out the missing browser headers carefully, and review payment verification and credential issuance in an authorized environment.

I’d be happy to conduct a deeper penetration test with advance notice and written scope, covering authentication, authorization, sessions, input handling, API security, payment logic, and related controls. [Contact me at work.with.aman.verma@gmail.com](mailto:work.with.aman.verma@gmail.com).

## Explore the inferred architecture

The [Inferred Backend Architecture](INFERRED_ARCHITECTURE.md) is a separate design proposal. It covers multi-module hexagonal design, shared ports with feature-owned adapters, event-driven consumers, SOLID rationale, rate limits, job queues, Mermaid diagrams, and illustrative Java examples. It is clearly labeled as inferred or proposed—not an audit finding.