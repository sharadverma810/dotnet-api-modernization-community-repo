# Prompt 03 — Contract-first skeleton and parity harness

> Run only after Prompt 02 has been reviewed and the architecture/skeleton plan has been approved.

```text
Act as a Principal .NET Architect and senior implementation engineer.

Read the approved architecture decision, target architecture, plan, API contract inventory, responsibility matrix and parity strategy. The project is already on .NET 10; keep `TargetFramework` at `net10.0`.

Implement only the approved skeleton/parity-harness plan. Do not begin business capability migrations beyond the approved pilot setup. Do not create a new deployable API host unless the approved ADR explicitly selected it.

PRE-CHANGE SAFETY
1. Verify branch, baseline commit and working tree.
2. Confirm this plan is approved and prerequisites passed.
3. Run or verify the baseline build/test status.
4. List planned file changes and explain why each is needed.
5. Stop if there are unexpected local changes or a scope conflict.

SKELETON REQUIREMENTS
Preserve existing external API behaviour at the contract boundary:
- Routes, verbs and versions.
- Request/response DTO shape and serialization settings.
- Status codes, headers, content types and error contract.
- AuthN/AuthZ policy and claims expectations.
- Pagination/filter/sort/null/default semantics.
- Exception mapping, cancellation and correlation behaviour.

Use the approved target architecture. Prefer the current host and existing middleware/configuration if those are already correct. Create only the minimum project/folder boundaries required. Do not create generic abstractions or a project per layer without a reason.

TEST COMPOSITION
- Create or adapt contract tests that can run against the existing implementation and target skeleton.
- Use the existing test framework and test runner.
- Use `WebApplicationFactory<TEntryPoint>`/`TestServer` if appropriate for this ASP.NET Core host.
- Use deterministic test fixtures.
- Fake repositories, downstream clients and message publishers only in explicit test configuration.
- Ensure production DI/configuration cannot accidentally select fake/test adapters.
- Add a guard test or registration check if the repository's composition pattern makes fake activation a realistic risk.

IMPORTANT LIMITATION
A skeleton may prove route registration, serialization, DI composition, auth wiring, status/error mapping and contract compatibility. It does NOT prove business functionality, data correctness, transaction semantics, downstream integration or functional parity.

Do not create successful dummy implementations for every endpoint and call them production-complete. For an unfinished capability, use a test-only fake in its contract harness or leave its production path on the old implementation until a complete slice is ready. Do not expose placeholder successful responses in DEV/PROD.

PARITY HARNESS
- Use the same scenario definitions for old and new paths where technically possible.
- Capture request, response, status, headers and structured body.
- Define allowed normalization for truly nondeterministic fields only.
- Add controlled adapters for database state, outbound calls and message capture.
- Keep write scenarios in isolated test infrastructure.
- Never duplicate real production side effects.

VALIDATION
1. Build the affected projects.
2. Run the pre-existing baseline tests relevant to the change.
3. Run contract tests against both old implementation and target skeleton.
4. Run auth, error, serialization, cancellation and route tests.
5. Verify test-only fake registrations cannot leak into production.
6. Record exact commands and outcomes.
7. Review package changes; do not perform unrelated package upgrades.

DOCUMENTATION
Update skeleton plan status, target architecture, parity strategy, progress and decision log. Record limitations honestly.

STOP CONDITIONS
Stop if the contract inventory is incomplete, auth behaviour is unknown, fake wiring could reach production, contract differences are unexplained, or tests cannot isolate side effects safely.

FINAL RESPONSE
List changed files, implemented skeleton elements, contract-test coverage, exact test results, known gaps, fake adapter safeguards, deviations and whether the skeleton gate passed. Do not claim functional parity.
```
