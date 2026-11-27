# 08 — Deliverables and recommended workspace

## Recommended documentation workspace

Create this structure in the repository after the scope and branch are confirmed:

```text
 docs/
   modernization/
     README.md
     00-scope-and-baseline.md
     01-architecture-assessment.md
     02-application-responsibility-matrix.md
     03-api-contract-inventory.md
     04-upstream-downstream-dependencies.md
     05-data-flow-and-side-effects.md
     06-existing-test-baseline.md
     07-test-gap-analysis.md
     08-performance-baseline.md
     09-strategy-decision.md
     10-target-architecture.md
     11-modernization-plans.md
     12-implementation-order.md
     13-parity-testing-strategy.md
     14-production-readiness-criteria.md
     decision-log.md
     progress.md
     risks-and-unknowns.md
     plans/
       PLAN-001-<name>.md
       PLAN-002-<name>.md
     parity/
       endpoint-parity-matrix.md
       functional-parity-matrix.md
       test-evidence-index.md
     reports/
       baseline-test-report.md
       pilot-slice-report.md
       final-validation-report.md
       dev-readiness-report.md
```

The list is a recommended set. Do not create empty files merely to match a template. Create a document when relevant information exists and mark any not-applicable item with a reason.

## Core deliverables and contents

| Deliverable | Must contain |
|---|---|
| Scope and baseline | Repository, solution, API project, branch, base commit, SDK, target framework, environments, out-of-scope items |
| Architecture assessment | Current components, key code paths, layering, hotspots, known constraints |
| Responsibility matrix | Every endpoint/capability, business purpose, rules, data owner, dependencies, implementation path |
| API contract inventory | Routes, methods, versions, DTOs, status codes, errors, auth, serialization, pagination |
| Dependency map | Upstream/downstream system, protocol, operation, criticality, failure behaviour, owner |
| Data flow and side effects | Reads/writes, transaction, messages, cache, external calls, idempotency |
| Test baseline | Exact commands, environment, pass/fail/skip counts, existing failures, artifacts |
| Test gap analysis | Untested risk, impact, scenario, test layer and priority |
| Performance baseline | Workload, environment, p50/p95/p99, throughput, CPU, allocations, GC, query/dependency timing |
| Strategy decision | Option comparison and selected approach with rationale |
| Target architecture | Structure, dependency direction, behavior placement, transition path |
| Modernization plans | Independent plan descriptions with dependencies and acceptance criteria |
| Implementation order | Dependency-aware order, priority, gates and reason |
| Parity strategy | Scenario source, normalization, side-effect safety, comparator and acceptance |
| Readiness criteria | Test, security, configuration, data, deployment, monitoring and rollback gates |
| Decision log | Date, context, options, decision, rationale, consequences, approver |
| Progress | Plan state, tests executed, evidence links, blockers and next work |

## Plan status model

Use a small, explicit vocabulary:

- `NOT_STARTED`
- `ASSESSMENT_COMPLETE`
- `APPROVED`
- `IN_PROGRESS`
- `BLOCKED`
- `IMPLEMENTED_AWAITING_VALIDATION`
- `VALIDATION_FAILED`
- `PARITY_FAILED`
- `COMPLETE`
- `DEFERRED`
- `REJECTED`

Do not use `COMPLETE` when only source code has changed. A plan is complete only after validation and evidence.

## Traceability model

Every required responsibility should be traceable across:

`Responsibility ID → API/trigger → existing code path → target use case → dependency/side effect → test scenarios → parity evidence → plan ID → status`.

This prevents endpoints or non-HTTP work (for example hosted services and message consumers) from disappearing during a rewrite.

## Recommended AI execution rhythm

- Assessment: one broad read-only pass, then targeted follow-up for unknowns.
- Planning: propose options and plans; human review of material choices.
- Implementation: one plan/slice at a time; short feedback loop.
- Validation: run affected tests early, then the broader suite at gate boundaries.
- Review: run an independent adversarial review prompt after the final report.

## What to put in every plan

Use `templates/plan-template.md`. At minimum it must specify:
- Goal and business rationale.
- Existing responsibility and behaviour to preserve.
- Scope and non-goals.
- Dependencies and impacted files/components.
- Target design and implementation steps.
- API/data/security/resilience implications.
- Tests and parity scenarios.
- Acceptance criteria.
- Risks and rollback.
- Validation commands and evidence expected.

## Evidence storage

Keep test result files, coverage, logs, parity outputs and performance artifacts in the repository's approved artifact location. Do not commit large or sensitive outputs by default. Link to CI artifacts or sanitized reports. Never commit secrets, raw production data or sensitive customer payloads.
