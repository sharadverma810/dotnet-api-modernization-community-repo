# 07 — Engineering rules and AI guardrails

These rules apply to every AI prompt and every implementation plan. Treat them as non-negotiable unless a named human owner approves a documented exception.

## A. Repository and Git safety

1. Work on one API modernization scope per feature branch by default. Shared-library changes must be included only when their impact is understood and approved.
2. Inspect `git status`, current branch and current commit before any operation.
3. Create the feature branch from the intended `main` reference only when the working tree is safe.
4. Never overwrite a branch with the same name. Choose a unique name or stop.
5. Never run destructive Git operations (`reset --hard`, `clean -fd`, force checkout, force push, history rewrite) unless the user explicitly requests the exact operation and understands the effect.
6. Never discard, overwrite or stash existing user work without explicit approval.
7. Do not push, merge, create a PR or deploy unless explicitly requested/authorized.
8. Keep changes scoped. Do not perform unrelated formatting or cleanup that obscures the modernization diff.
9. Record the base commit and branch in the assessment.

## B. Evidence and truthfulness

1. Inspect actual source code, configuration and tests. Do not infer behaviour solely from class names or README files.
2. Cite concrete file paths, types and methods in analysis documents.
3. Distinguish facts, assumptions, hypotheses and unknowns.
4. Never claim a test, build, benchmark, parity comparison or deployment succeeded unless it actually ran and produced evidence.
5. Record the exact command, commit, environment and results.
6. Preserve pre-existing test failures in the baseline register; do not silently attribute them to new code or claim they were fixed.
7. Do not invent missing requirements. Create an unknown/question with an owner.
8. Do not claim “production safe” or “zero risk”. State verified evidence and residual risks.

## C. Behaviour and compatibility

1. Existing intended business responsibilities and approved external contracts are the reference.
2. Distinguish required compatibility from accidental behaviour and defects.
3. Do not change route, HTTP verb, API version, DTO schema, status code, header, authorization, serialization, error envelope or pagination semantics without approval.
4. Do not normalize away a meaningful parity difference.
5. Do not remove hidden behaviour from middleware, filters, MediatR behaviours, validators, decorators, repositories or hosted services without replacing and testing it.
6. Preserve data ownership, transaction boundaries, concurrency, idempotency and message delivery semantics.
7. Preserve cancellation and timeout budgets.
8. Do not perform schema/data migrations as incidental refactoring.

## D. Test integrity

1. Characterize important existing behaviour before changing it.
2. Add or update tests in the same plan/slice that changes behaviour.
3. Never delete, skip, weaken or loosen an assertion merely to obtain a green build.
4. Do not mark a flaky test as passed; follow a documented quarantine policy with owner and expiry if unavoidable.
5. Run focused tests after each change and the relevant full regression suite before declaring a plan complete.
6. Contract tests alone do not prove functional parity.
7. Unit tests with mocked repositories do not prove real database semantics.
8. Use isolated test infrastructure and deterministic data.
9. No duplicate production writes, messages or non-idempotent calls for shadow/parity testing.
10. Record intentional differences and obtain approval before updating expected results.

## E. Architecture and code quality

1. Prefer explicit direct use-case/application-service calls over a new mediator abstraction.
2. Do not build a custom framework merely to replace a removed package.
3. Do not add a project, layer, interface, generic repository or abstraction without a demonstrated reason.
4. Keep business decisions out of controllers/endpoints and infrastructure adapters.
5. Keep domain invariants enforceable outside the HTTP path.
6. Use dependency inversion for real external dependencies, not every internal method call.
7. Avoid service locator, hidden global state and static mutable state.
8. Avoid synchronous blocking over asynchronous I/O.
9. Propagate cancellation to I/O operations.
10. Keep logs useful, structured and free of secrets/credentials/sensitive payloads.

## F. Dependency modernization

1. Inventory every package usage and transitive dependency before removal.
2. Replace one concern at a time.
3. Map and preserve MediatR pipeline behaviours.
4. Do not assume native validation is equivalent to FluentValidation.
5. Prefer native assertions from the existing test framework; add Shouldly only with justification.
6. Microsoft.Extensions resilience packages use Polly internally; distinguish direct integration removal from transitive package removal.
7. Review retry safety and idempotency before changing resilience.
8. Remove unused package references only after source, configuration, DI and runtime paths are checked.
9. Avoid unrelated package upgrades and target-framework changes.

## G. Performance and modern language features

1. Capture a baseline before optimization.
2. Optimize measured hot paths and algorithmic bottlenecks first.
3. Do not use Span, Memory, records, structs, pooling, ValueTask or unsafe code simply because they are available.
4. Assess lifetime, ownership, copying, boxing, thread safety and readability.
5. Benchmark the same workload under comparable conditions.
6. Report both improvements and regressions; never cherry-pick a single metric.
7. Performance changes must not weaken correctness, validation, observability or security.

## H. Security and data protection

1. Do not print, copy or commit secrets, tokens, certificates, personal data or production payloads.
2. Use environment variables, secret stores and approved test secrets.
3. Never connect test automation to production data without explicit approval and a safe data-handling plan.
4. Maintain authentication, authorization, tenant isolation and resource-level access controls.
5. Treat logs, snapshots and parity artifacts as potentially sensitive; redact and retain them according to policy.
6. Avoid test adapters that can be activated accidentally in production.
7. Check dependency security/advisories as part of final validation.

## I. Plan and approval discipline

1. Assessment is read-only.
2. Architecture and plan documents require review before skeleton/production-code work.
3. Each slice has objective, scope, non-goals, acceptance criteria, tests, risks and rollback.
4. A failing gate blocks dependent work.
5. Human approval is required for API/business contract changes, database/schema changes, auth changes, new deployment topology, data migrations, material resilience changes and DEV deployment.
6. The agent may proceed automatically only within an approved plan and its non-controversial implementation steps.
7. Update progress and traceability as work is completed.
8. Stop and report when a necessary decision cannot be inferred safely.

## J. Definition of done

A plan is done only when its acceptance criteria are evidenced, relevant tests pass, behavioural differences are understood and approved, documentation is updated, and the reviewer can reproduce the validation. “Code compiled” is not a definition of done.
