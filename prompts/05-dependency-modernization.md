# Prompt 05 — Safe dependency and pattern modernization

> Run one technology workstream at a time. Replace `<WORKSTREAM>` with MediatR, FluentValidation, FluentAssertions or direct Polly integration. Do not combine multiple package removals into one opaque change.

```text
Act as a Principal .NET Architect and senior engineer specializing in safe dependency modernization.

WORKSTREAM: <WORKSTREAM>
PLAN ID: <PLAN-ID>

Read the approved plan, architecture decision, source usage inventory, responsibility matrix, test baseline, parity strategy and dependency order. The API is already on .NET 10. Keep `net10.0` and avoid unrelated package upgrades.

GENERAL RULES
1. Verify branch/worktree and plan approval.
2. Inspect actual direct and transitive package references and all usages.
3. Map the behaviour currently provided by the dependency before removing it.
4. Identify hidden behaviours, DI registration, reflection scanning, configuration, error handling, lifetimes and telemetry.
5. Implement the smallest safe change for this workstream only.
6. Preserve public API and business behaviour.
7. Run focused tests after each meaningful change, then relevant regression and parity tests.
8. Do not weaken assertions, tests or validation to achieve a green result.
9. Do not claim a package has been removed until source, configuration, DI, project and dependency graph checks confirm it.
10. Record any remaining transitive dependencies accurately.

WORKSTREAM-SPECIFIC GUIDANCE

IF WORKSTREAM = MEDIATR:
- Inventory all requests, commands, queries, handlers, notifications, pipeline behaviours, pre/post processors, exception handlers, decorators and registrations.
- Trace validation, transactions, authorization, logging, caching, idempotency, domain events and exception mapping currently executed through the pipeline.
- Extract each responsibility to its correct boundary and add tests before deleting dispatch code.
- Replace request dispatch with direct constructor-injected application use-case/service calls.
- Keep CQRS naming only where it clarifies the use case; do not require a mediator.
- Preserve notification timing/delivery and all transaction semantics.
- Do not build a custom mediator, generic `Send` dispatcher, reflection dispatch or generic pipeline to recreate MediatR.
- Remove the package only after all production references, behaviours and registrations are eliminated or deliberately retained elsewhere.

IF WORKSTREAM = FLUENTVALIDATION:
- Inventory every validator, rule, rule set, child/nested/collection rule, async rule, condition, cascade mode, custom validator, localization rule, error code and DI dependency.
- Classify each rule: request shape, cross-field application rule, async data check, authorization, or domain invariant.
- Consider native ASP.NET Core/.NET 10 validation, DataAnnotations, IValidatableObject, Microsoft.Extensions.Validation where supported, and small explicit asynchronous validators.
- Verify actual host style: Minimal API and MVC/Razor validation support are not interchangeable assumptions.
- Preserve timing, rule order, property path, error code/message and response envelope when contractual.
- Keep DB-dependent checks in an appropriate application boundary and retain database constraints for integrity.
- Do not move domain invariants only into API DTO annotations.
- If replacement is more complex or risky, document why FluentValidation is retained rather than removing it for package-count reasons.

IF WORKSTREAM = FLUENTASSERTIONS:
- Keep the existing test framework.
- Prefer its native assertion API (xUnit, NUnit or MSTest as applicable).
- Consider Shouldly only when the framework cannot provide sufficient clarity/diagnostics and approval exists.
- Migrate by test category and preserve assertion intent, equivalency/deep comparison, ordering, exception and async semantics.
- Do not delete or weaken tests. Do not change production code in this workstream.
- Compare test inventory and execution results before and after.

IF WORKSTREAM = POLLY:
- Inventory every policy/pipeline: retry predicates/count/delay/jitter, circuit breaker thresholds, timeout budgets, fallback, rate limit, hedging, strategy order and cancellation.
- Evaluate `Microsoft.Extensions.Resilience` and `Microsoft.Extensions.Http.Resilience` for DI/HttpClient integration.
- These packages are built on Polly. State accurately that direct Polly integration may be replaced while Polly remains transitive.
- Preserve end-to-end timeout budgets, strategy ordering, telemetry and failure semantics.
- Retry only safe/idempotent operations or those protected by idempotency/deduplication.
- Avoid stacked retries across client/service/gateway layers.
- Test transient/permanent failures, cancellation, exhaustion and circuit state using controlled failures.

VALIDATION
- Build affected projects.
- Run relevant unit, integration, contract, resilience and parity tests.
- Verify DI composition and application startup.
- Inspect package references and dependency graph.
- Search code/config/tests for stale references and old registrations.
- Verify no test-only dependency leaks into production.

DOCUMENTATION
Update the plan, package inventory, architecture ADR, dependency map, parity evidence, decision log and progress. Record what was removed, what remains transitively, what behaviour moved and why.

STOP CONDITIONS
Stop if a hidden behaviour cannot be preserved, replacement semantics are unclear, validation response changes, retry safety is unknown, a material package conflict exists, or tests cannot prove the migration.

FINAL RESPONSE
Provide a before/after dependency usage table, files changed, behaviours moved/preserved, tests and exact commands/results, dependency graph findings, risks and plan status. Do not start another dependency workstream automatically.
```
