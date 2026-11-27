# Prompt 07 — Independent adversarial architecture and parity review

> Run as a separate review pass. Ideally use a fresh agent/session that did not implement the changes.

```text
Act as an independent Principal Architect, senior QA auditor and skeptical production-readiness reviewer. Your job is to find reasons this modernization may be incomplete or unsafe. Do not assume the implementation agent's report is correct.

Review the complete diff and all documentation under `docs/modernization/`. Verify claims against source code, tests, package references, configuration and CI outputs where accessible.

Do not make code changes during the first review pass.

REVIEW 1 — Scope completeness
- Is every API endpoint, background process, message handler and scheduled task accounted for?
- Are any upstream/downstream dependencies, data flows or side effects missing?
- Are responsibilities mapped to target implementations and tests?
- Are there undocumented behavioural assumptions?

REVIEW 2 — Contract and behaviour
- Did routes, verbs, versions, DTOs, nullability, serialization, status codes, errors, headers, pagination or auth change?
- Are changes approved and documented?
- Are there differences hidden by normalizers, broad JSON ignores or snapshot updates?
- Is functional parity proved beyond response JSON?

REVIEW 3 — Cross-cutting semantics
- Were all MediatR behaviours found and preserved or deliberately removed?
- Did validation order, async checks, error codes or response shape change?
- Were transaction boundaries, exception handling, caching, idempotency, logging, authorization and cancellation preserved?
- Are there retries that can duplicate writes or messages?
- Are timeouts nested or inconsistent?

REVIEW 4 — Architecture quality
- Does the target genuinely reduce coupling and complexity?
- Are there unnecessary projects, interfaces, generic repositories, factories or custom pipelines?
- Is any custom mediator merely recreating MediatR?
- Are responsibilities placed at appropriate boundaries?
- Are there cyclic dependencies, service locator patterns or incorrect DI lifetimes?
- Are test fakes impossible to select in production composition?

REVIEW 5 — Test strength
- Did tests get deleted, skipped, weakened or made less precise?
- Do tests use realistic provider semantics where needed?
- Are write/transaction/message side effects verified safely?
- Are concurrency, idempotency, auth, dependency failure and cancellation tested where required?
- Are tests deterministic and reproducible?
- Are failures accurately distinguished from baseline failures?

REVIEW 6 — Package and performance claims
- Are package removals real, including DI scanning/configuration and direct references?
- Are transitive dependencies described accurately?
- Are FluentValidation alternatives semantically equivalent?
- Is Microsoft.Extensions resilience correctly described as built on Polly?
- Are Span/Memory/records/pooling/ValueTask and other optimizations justified?
- Are performance claims backed by comparable measurements?

REVIEW 7 — Deployment readiness
- Can the artifact run with DEV configuration and supported runtime?
- Are schema changes backward-compatible and sequenced?
- Are message contracts compatible across rolling deployment?
- Are logs/metrics/traces/health checks useful?
- Is rollback technically possible with the data and message changes?
- Are there manual steps missing from the release checklist?

OUTPUT
Create `reports/independent-review.md` containing:
- Overall assessment and recommendation.
- Findings grouped as BLOCKER, HIGH, MEDIUM, LOW or OBSERVATION.
- For each finding: evidence (file/class/test/command), impact, likelihood, recommendation and required owner/approval.
- Missing or weak evidence.
- Tests that should be added or rerun.
- Exact changes required before DEV.
- Items that may safely be deferred with rationale.

Do not fix issues during this pass. First report findings. Do not declare GO if there is any unresolved blocker or unexplained parity difference.
```
