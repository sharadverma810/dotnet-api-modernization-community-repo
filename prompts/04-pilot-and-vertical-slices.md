# Prompt 04 — Pilot implementation and sequential vertical slices

> Run after the skeleton/parity harness gate passes. Use the same prompt for the pilot and each subsequent capability, updating the plan ID and slice scope.

```text
Act as the Principal .NET Solution Architect, Senior Application Engineer and Test Engineer responsible for migrating one approved business capability at a time.

Read:
- Approved target architecture and strategy ADR.
- Application responsibility matrix.
- API contract inventory.
- Upstream/downstream and data-flow maps.
- Test baseline and gap analysis.
- Parity strategy.
- The selected vertical-slice plan and dependency order.

The API is already on .NET 10. Keep `net10.0`. Do not widen the task to unrelated framework or package upgrades.

CURRENT SLICE
Plan ID: <PLAN-ID>
Capability / use case: <CAPABILITY>
In-scope operations/endpoints/triggers: <LIST OR REFERENCE TO PLAN>
Explicit non-goals: <LIST>

PRE-FLIGHT
1. Confirm current feature branch and clean/understood worktree.
2. Confirm prerequisites and previous slice gates passed.
3. Re-read the existing implementation path and the actual target path.
4. Confirm baseline test commands and known pre-existing failures.
5. Confirm test data and external side effects are isolated.
6. Identify any contract or business decision requiring human approval. Do not infer it.

IMPLEMENT A COMPLETE VERTICAL SLICE
The capability must be complete from its entry point to its real business outcome:
- Endpoint/route/trigger and DTO contract.
- Authentication, authorization and resource-level checks.
- Request validation at the correct boundary.
- Application use case and business rules.
- Domain invariants where applicable.
- Real persistence/data access.
- Real or controlled integration with required downstream dependencies.
- Transactions, concurrency and idempotency where required.
- Events/messages/cache side effects where required.
- Error mapping, logging, metrics, tracing and cancellation.
- Appropriate unit, component, integration, contract, acceptance and parity tests.

Do not stop at a mock-backed use case. Test fakes are appropriate for isolated unit/component scenarios; integration and acceptance scenarios must exercise the real provider/infrastructure behaviour where that behaviour matters.

IMPLEMENTATION RULES
- Preserve approved API contracts and intended business behaviour.
- Keep changes scoped to this capability and required shared infrastructure.
- Use direct application service/use-case calls unless the approved ADR states otherwise.
- Do not create a mediator framework or generic pipeline.
- Do not introduce unnecessary interfaces, generic repositories or layers.
- Do not make unrelated cleanup changes.
- Do not silently change validation response, error codes, transaction boundaries, retry rules or message contracts.
- Preserve CancellationToken propagation.
- Never add real credentials or production data to code/tests.

VALIDATION CYCLE
After each logical change:
1. Build the affected projects.
2. Run targeted unit/component tests.
3. Run integration tests against real/approved infrastructure or faithful provider tests.
4. Run contract tests.
5. Run acceptance scenarios.
6. Run parity scenarios against the reference implementation using identical controlled inputs.
7. Compare response/status/headers plus data state, transactions, outbound calls, messages, cache and audit effects as relevant.
8. Run security/authorization, concurrency/idempotency and failure tests relevant to this capability.
9. Run performance comparison if the slice changes a hot path or performance-sensitive behaviour.
10. Run the relevant regression suite.

PARITY SAFETY
- Do not invoke real non-idempotent downstream operations twice.
- Use isolated test data, fake/recording outbound adapters, controlled broker/database or safe test environment.
- Normalize only approved nondeterministic fields.
- Do not mask real mismatches with broad JSON ignore rules.
- Classify every difference as regression, intentional approved change, known legacy defect, nondeterminism or unresolved.
- Unexplained behaviour differences block completion.

SLICE ACCEPTANCE
Mark the slice complete only when:
- All in-scope responsibilities are implemented with real required behaviour.
- API contract is compatible or approved changes are recorded.
- Data, transaction, security and side-effect semantics are correct.
- Required tests pass.
- Parity passes or differences are explicitly approved.
- No unexplained regression exists.
- Logs/traces/metrics and cancellation are appropriate.
- Performance claims are supported by evidence.
- Plan, traceability, risk and progress documents are updated.

AFTER THE PILOT
If this is the pilot, explicitly assess whether the target architecture, test harness, abstractions, DI, observability and implementation cadence should be adjusted before scaling. Do not assume the pilot pattern should be copied blindly.

STOP CONDITIONS
Stop and report if tests cannot isolate side effects, a business rule is ambiguous, a schema/message/auth change is required, a parity gap cannot be explained, the target architecture proves inadequate, or a dependent plan prerequisite is incomplete.

FINAL RESPONSE
Report plan ID, slice, files changed, behaviour preserved, new behaviour implemented, tests and exact commands/results, parity comparison, performance evidence, risks, documentation updates and COMPLETE/BLOCKED/FAILED status. Do not begin the next slice unless the current slice's gate is passed.
```
