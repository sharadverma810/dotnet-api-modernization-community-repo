# Prompt 02 — Architecture strategy decision and implementation planning

> Run after the read-only assessment has sufficient evidence. This phase may create or update planning documents but must not modify application source or tests.

```text
Act as a Principal .NET Solution Architect and modernization programme lead.

Read all existing documents under `docs/modernization/`, verify important claims against source code, and use the actual repository as the source of truth. The API already targets .NET 10. Do not add a framework upgrade to this initiative.

Your objective is to recommend the lowest-risk architecture and a dependency-aware set of independently testable implementation plans.

PART 1 — Validate the assessment
- Check that every exposed endpoint and non-HTTP trigger is represented.
- Check that MediatR handlers and pipeline behaviours are mapped.
- Check validators, transactions, caching, authorization, logging, notifications, idempotency and exception handling.
- Check direct/transitive package usage.
- Check that tests and baseline failures are correctly represented.
- Correct documentation errors only when source evidence is clear; record material corrections.
- Add unresolved items to risks-and-unknowns.md.

PART 2 — Compare three modernization strategies
A. Incremental modernization within existing API implementation.
B. Hybrid modernization: preserve existing host/public boundary, build a cleaner internal application structure and migrate by vertical slice.
C. New API host/project with contract-first skeleton and controlled coexistence.

Compare each for: API compatibility, coupling, ability to isolate dependencies, testability, business-risk discovery, performance, deployment complexity, observability, rollback, data/schema compatibility, team/operational cost and expected maintainability.

Do not choose C simply because a new skeleton looks cleaner. Do not choose A simply because it changes fewer files. Recommend the approach supported by repository evidence. The default starting hypothesis is B unless the assessment proves otherwise.

PART 3 — Define the target architecture
- Decide whether the host stays, whether new projects are warranted, and why.
- Define feature/use-case organization and dependency direction.
- Define endpoint-to-use-case interaction.
- Define validation, domain invariants, authorization, transactions, caching, messaging, resilience, logging and error mapping boundaries.
- Identify which abstractions isolate real infrastructure and which should not exist.
- Define temporary old/new coexistence and capability routing, if needed.
- Define data/schema and event/message compatibility constraints.
- Avoid a custom mediator/framework that recreates removed dependencies.

PART 4 — Decide what “skeleton” means
If appropriate, design a minimal contract-first target skeleton. It may exist inside the current host/solution; a second deployable host requires justification. Define only the routes, DTOs, auth, DI, configuration, observability, error handling and test seams needed to prove the boundary.

Fakes and dummy data must be test-only. Do not build all endpoints as successful dummy implementations and postpone real logic. A skeleton is not production-ready and cannot prove functional parity.

PART 5 — Create modernization plans
Create one plan per coherent workstream. At minimum evaluate:
- Test/parity harness and risk-based test gap closure.
- Minimal target architecture/skeleton if justified.
- Representative pilot vertical slice.
- Remaining business capability slices in dependency order.
- FluentAssertions replacement (test-only; use existing assertion framework first, Shouldly only if justified).
- MediatR behaviour extraction and direct use-case invocation.
- FluentValidation replacement or deliberate retention decision.
- Direct Polly usage migration to Microsoft.Extensions resilience integration where appropriate.
- Performance/memory optimization based on measured hot paths.
- Package/dead-code cleanup and final verification.

Keep package migrations distinct. Where validation is part of a MediatR behaviour, plan the safe extraction order; do not automatically combine MediatR and FluentValidation into one opaque change.

Each plan must contain:
1. ID/title/status/priority.
2. Business/technical rationale.
3. Existing behaviour and responsibilities to preserve.
4. Goal and measurable expected outcome.
5. Explicit Clean Architecture and vertical-slice alignment: feature boundary, use case, domain rules, application orchestration, infrastructure adapters, dependency direction and complete-slice boundary.
6. Explicit AI-native delivery approach: repository evidence, deterministic context, permitted automation, human approvals, traceability and safeguards against fake or incomplete production behavior.
7. Scope and non-goals.
8. Affected projects/files/components.
9. Prerequisites/dependencies and dependency-aware sequencing.
10. Proposed design and independently executable implementation steps.
11. API/data/security/resilience implications.
12. Unit/component/integration/contract/acceptance/parity tests.
13. Concrete parity validation for contract, business outcomes, data/transactions, side effects, failures, authorization and performance where relevant.
14. Measurable success criteria, acceptance criteria and definition of done.
15. Risks, owners, mitigations, rollback triggers and rollback/recovery path.
16. Validation commands and evidence expected.

Plans must be concrete rather than a list of themes: identify the capability or concern being changed, the responsibilities it preserves, the files/components likely affected, prerequisites, ordered implementation steps, outputs, and evidence needed for approval. Create separate plans for each applicable workstream below; do not merge independent package migrations into one opaque plan:

- Test/parity harness and risk-based test gap closure.
- Minimal target architecture/skeleton, if justified.
- Representative pilot vertical slice.
- Remaining business capability vertical slices in dependency order.
- FluentAssertions replacement or documented retention decision.
- MediatR behavior extraction and direct use-case invocation.
- FluentValidation replacement or deliberate retention decision.
- Direct Polly usage migration to Microsoft.Extensions resilience integration where appropriate.
- Measured performance and memory optimization.
- Package/dead-code cleanup and final verification.

PLAN QUALITY CHECKS
- Reject a plan that does not name the responsibilities, endpoints/triggers, files or components it affects or preserves.
- Reject a plan with no explicit prerequisites, dependency order, implementation outputs or independently verifiable steps.
- Reject a plan whose success criteria are subjective or lack commands, measurements, fixtures or other evidence.
- Reject a plan that describes layers without mapping a real feature through a complete vertical slice and stating dependency direction.
- Reject a plan that treats AI generation as a substitute for repository inspection, human approval or test evidence.
- Reject a plan that uses production fakes, placeholder success responses or unbounded autonomous changes.
- Reject a parity plan that compares only status codes or JSON shape; require business outcomes, data/transactions, side effects and failure/security behavior where applicable.
- Reject a risk section without named owners, rollback triggers and a recovery path for code, configuration, data, schema, messages and external side effects as applicable.
- Reject a plan without focused validation commands, expected evidence and a definition of done.

PART 6 — Prioritize and sequence
Build a dependency-aware order. Prioritize prerequisites before refactors. Explain which plans may run in parallel and which must be sequential. A single API modernization should not become a multi-project big-bang change. State how shared libraries will be handled.

PART 7 — Approval gates
Identify decisions that require human approval, including API/business contract changes, schema/data migration, auth changes, new host/deployment topology, message contract changes, materially different retry/timeout semantics and DEV deployment.
Do not create project-specific implementation details without repository evidence. Keep independent dependency migrations in separate plans. Treat the following as approval blockers rather than AI-autonomous decisions: contract or business behavior changes, schema/data changes, authorization changes, new hosting topology, message changes, materially different resilience semantics and deployment.

DELIVERABLES
Create/update:
- 09-strategy-decision.md
- 10-target-architecture.md
- 11-modernization-plans.md
- 12-implementation-order.md
- 13-parity-testing-strategy.md
- 14-production-readiness-criteria.md
- `plans/PLAN-###-<name>.md` for each real plan
- decision-log.md
- risks-and-unknowns.md

FINAL RESPONSE
Provide the selected strategy and why the other options were not selected, target architecture, plan summary, sequence, risks, required approvals and measurable gates. Explicitly state whether a separate API host/project is recommended. Do not implement application code or modify tests in this phase.
```
