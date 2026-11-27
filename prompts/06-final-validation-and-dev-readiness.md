# Prompt 06 — Full regression, parity and DEV deployment readiness

> Run after every approved slice/workstream is complete. This prompt prepares readiness evidence; it does not authorize deployment.

```text
Act as a Principal .NET Architect, QA/Test Lead, DevSecOps reviewer and release-readiness assessor.

Read all modernization plans, current progress, responsibility matrix, API contract inventory, architecture decision, test baseline, parity matrices, performance reports, risk register and decision log.

The API remains on .NET 10. Do not upgrade the framework or broaden scope.

OBJECTIVE
Determine whether the modernized API is a credible candidate for DEV deployment and QA validation. This is an evidence review, not a declaration of guaranteed production safety.

PHASE 1 — Completeness and traceability
- Verify every in-scope responsibility and non-HTTP trigger is represented.
- Map each responsibility to the target code path/use case.
- Map every responsibility to tests and parity evidence.
- Verify every approved plan status and acceptance criterion.
- Identify any missing endpoint, background job, message handler, data side effect or downstream dependency.
- Confirm obsolete implementation paths are retired only when no longer required.

PHASE 2 — Build and test execution
Run the repository's actual validation matrix, not an assumed generic command:
1. Restore.
2. Build all relevant projects/configurations.
3. Unit tests.
4. Component/application tests.
5. Integration tests.
6. Contract tests.
7. Acceptance tests.
8. End-to-end tests.
9. Parity tests.
10. Security/authorization/tenant isolation tests.
11. Concurrency/idempotency tests.
12. Resilience/fault-injection tests.
13. Database migration/data compatibility tests, if applicable.
14. Performance/load/memory tests, where planned.
15. Static analysis, formatting and dependency/security checks used by CI.

For every command, record exact invocation, commit, SDK, environment, duration, passed/failed/skipped counts and artifact path. Distinguish pre-existing failures from new failures. Do not suppress failures or mark unexecuted tests as passing.

PHASE 3 — API and functional parity
For every endpoint/capability compare:
- Method/route/version/schema/status/headers/content type.
- Serialization, null, date/time, numeric precision and pagination semantics.
- AuthN/AuthZ and resource/tenant access.
- Validation errors and exception mapping.
- Business outcomes and resulting state.
- Database state, transactions, concurrency and idempotency.
- Events/messages and delivery-relevant metadata.
- Downstream calls, retries, timeouts and failure handling.
- Cache, audit and other relevant side effects.

Classify each difference. Unexplained differences block readiness. Confirm the parity harness did not cause duplicate real side effects.

PHASE 4 — Package and architecture review
- Search direct and transitive references.
- Confirm MediatR behaviours have been handled and no hidden pipeline remains.
- Confirm FluentValidation is removed or deliberately retained with reason.
- Confirm assertion replacement preserved test intent.
- Confirm direct Polly references and Microsoft.Extensions resilience use are understood; do not claim transitive Polly is absent if it remains.
- Review unnecessary abstractions, project cycles, duplicate code, DI scopes and test-only code.

PHASE 5 — Performance and resource review
Compare equivalent workload/environment against baseline. Include p50/p95/p99, throughput, CPU, memory/allocations, GC, error rate and database/downstream timings as available. Highlight regressions and measurement limitations. Do not infer gains from code style.

PHASE 6 — Security and operations review
Check:
- Authentication, claims/scopes, authorization and tenant isolation.
- Secrets/configuration references and environment differences.
- Logs/metrics/traces/correlation and sensitive-data redaction.
- Health/readiness and dependency checks.
- Timeouts, retries, circuit breakers and idempotency.
- Database schema/data compatibility and migration sequence.
- Message schema and producer/consumer rollout compatibility.
- Capacity/connection pool/thread pool limits where relevant.
- CI/CD artifact, runtime and app configuration compatibility.

PHASE 7 — DEV readiness pack
Create/update:
- `reports/final-validation-report.md`
- `reports/dev-readiness-report.md`
- `reports/test-evidence-index.md` (or equivalent)
- deployment checklist
- rollback plan
- QA scenario handover

The DEV readiness report must state:
- Candidate commit/build artifact.
- Implemented scope and remaining exclusions.
- Test matrix with exact results.
- API and functional parity status.
- Performance comparison and limitations.
- Package/dependency status.
- Database/message/configuration changes.
- Security and observability review.
- Known defects, residual risks and owners.
- DEV deployment prerequisites.
- Smoke tests and QA scenarios.
- Rollback trigger and procedure.
- Explicit go/no-go recommendation.

APPROVAL AND DEPLOYMENT RULE
Do not deploy, push, merge, create a release or modify environments. Only prepare readiness evidence. Human release owner approval is required for DEV deployment. If a deployment tool or environment is accessible, do not invoke it without explicit authorization.

FINAL RESPONSE
Give an evidence-based GO / CONDITIONAL GO / NO-GO recommendation for DEV. A GO means readiness criteria have been met for DEV deployment and QA testing; it does not mean production release is approved or guaranteed safe. List every unmet criterion and every unexecuted test.
```
