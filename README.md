# .NET API Modernization Playbook

**A community-driven, AI-assisted approach to modernizing existing production .NET Web APIs.**

> **Status:** Community playbook · v1.0.0 · First published 27 November 2026
>
> **Scope:** Existing .NET 10 Web APIs. Framework migration is intentionally out of scope.

This repository provides a repeatable engineering method, reusable AI prompts, decision gates, checklists and templates for modernizing existing .NET Web APIs without losing required business responsibilities or externally relied-upon behaviour.

It is intended for developers, architects, test engineers and teams using GitHub Copilot Agent or another repository-aware coding agent. The guidance is tool-agnostic: adapt it to your repository, coding standards, delivery process and risk profile.

## The core recommendation

Use **contract-first + test-first + evidence-based architecture decisions + vertical-slice implementation**.

1. Understand the existing API before changing it.
2. Catalogue its responsibilities, contracts, data behaviour, security rules and integrations.
3. Establish an executable baseline and identify test gaps.
4. Choose the least risky target: modernize in place, use a hybrid approach, or create a separate API host only when justified.
5. Build a contract/parity harness before major refactoring.
6. Prove the target design with one representative, end-to-end business capability.
7. Implement remaining capabilities one complete vertical slice at a time.
8. Modernize dependencies separately, preserving all behaviour they currently provide.
9. Measure performance instead of assuming newer syntax or fewer packages are faster.
10. Finish with full regression, integration, security, parity and deployment-readiness validation.

**Do not treat a successful dummy response as functional parity.** Contract parity covers externally visible API behaviour. Functional parity also covers business outcomes, state changes, transaction semantics, authorization, events/messages, downstream calls, idempotency and failure behaviour.

## Who should use this?

- Teams maintaining existing production .NET Web APIs.
- Architects evaluating a rewrite versus incremental modernization.
- Engineers simplifying MediatR, FluentValidation, FluentAssertions or direct Polly integrations.
- Teams improving test confidence before increasing AI-assisted coding.
- Contributors who want to share practical improvements to this playbook.

## Start here

| Read / run | Why |
|---|---|
| [Recommended strategy](guides/01-recommended-strategy.md) | Understand the recommended approach and trade-offs. |
| [Stages and decision gates](guides/02-stages-and-gates.md) | Follow the lifecycle and approval gates. |
| [Implementation order](guides/03-implementation-order.md) | Sequence work by risk and dependencies. |
| [Target architecture](guides/04-target-architecture.md) | Decide what to keep, simplify or replace. |
| [Parity and testing](guides/05-parity-and-testing.md) | Build confidence through executable evidence. |
| [Technical modernization decisions](guides/06-technical-modernization-decisions.md) | Evaluate packages and .NET 10/C# 14 options. |
| [Engineering rules and AI guardrails](guides/07-engineering-rules-and-ai-guardrails.md) | Apply non-negotiable safety and quality rules. |
| [Deliverables and workspace](guides/08-deliverables-and-project-workspace.md) | Keep findings and decisions traceable. |
| [Multi-API rollout](guides/09-multi-api-rollout.md) | Pilot once, then scale across a portfolio. |

## Reusable AI prompts

Run prompts in order, with human review at the defined gates. They are not instructions to give an agent unrestricted permission to refactor or deploy.

| Prompt | Purpose | Expected change |
|---|---|---|
| [00 — Project preflight](prompts/00-project-preflight.md) | Confirm scope, Git state and branch safety. | Branch only after safety checks. |
| [01 — Read-only assessment](prompts/01-read-only-assessment.md) | Discover responsibilities, contracts, dependencies and tests. | Documentation only. |
| [02 — Strategy and plan](prompts/02-strategy-and-plan.md) | Compare modernization options and produce ordered plans. | Documentation only. |
| [03 — Contract-first skeleton](prompts/03-contract-first-skeleton.md) | Establish contracts and a test harness; build a skeleton only if justified. | Limited, approval-gated. |
| [04 — Pilot and vertical slices](prompts/04-pilot-and-vertical-slices.md) | Implement complete capabilities one at a time. | Application and tests. |
| [05 — Dependency modernization](prompts/05-dependency-modernization.md) | Migrate dependencies in isolated work packages. | Application and tests. |
| [06 — Final validation and DEV readiness](prompts/06-final-validation-and-dev-readiness.md) | Run full suites and assemble readiness evidence. | Fixes/docs only; no deployment. |
| [07 — Independent review](prompts/07-independent-review.md) | Challenge assumptions, parity and readiness claims. | Review first; fixes only when authorized. |

## Suggested adoption path

**For your first API:** choose a representative but manageable service, run Prompts 00–02, review the evidence and approve an approach. Do not start by creating a second API automatically. Use Prompt 03 only when the architecture decision calls for a skeleton or when a contract harness is needed. Then use Prompt 04 for a pilot capability and the remaining vertical slices. Use Prompt 05 to manage package migrations without bundling unrelated changes. Finish with Prompts 06 and 07.

**For multiple APIs:** pilot the process on one service. Refine shared guidance based on what worked, but assess each API independently. Do not assume every API needs the same architecture or package replacements.

## Key principles

- Preserve intended business responsibilities, not accidental defects. Record ambiguous or defective behaviour for an owner decision; never silently change it.
- Existing API consumers and operational dependencies are part of the contract.
- Keep the existing host and deployment boundary when they are fit for purpose. A separate new API host needs an evidence-based justification.
- Fakes are test tools, not production implementations. Never allow dummy data to be registered in production.
- Prefer explicit application services/use cases over replacing one mediator with another mediator by default.
- Do not remove FluentValidation or any dependency solely to reduce package count. Map its actual semantics first.
- Microsoft.Extensions.Resilience is a Microsoft integration layer built on Polly; it can replace direct Polly integration, not necessarily Polly as a transitive dependency.
- New C# syntax, `Span<T>`, records, pooling and source generation are not automatic performance wins. Measure and use them only where they fit.
- No production deployment, destructive Git operation, schema change or externally visible contract change without explicit authorization.

## Documentation map

- `guides/` — strategy, architecture, stages, tests, technical choices and rules.
- `prompts/` — reusable AI-agent instructions.
- `templates/` — assessment, plan, parity, risk and progress templates.
- `docs/` — contribution, feedback, roadmap, FAQ, governance and repository setup.
- `.github/` — issue forms, pull request template, Dependabot and Markdown quality workflow.

For an all-in-one copy, see [Complete playbook](docs/COMPLETE-PLAYBOOK.md).

## Contribute and give feedback

Feedback from real projects is welcome. Start with [Feedback guidelines](docs/FEEDBACK-GUIDELINES.md) and [CONTRIBUTING.md](CONTRIBUTING.md). You can propose improvements to prompts, decision gates, examples, test strategies or technical guidance.

**Never submit employer/client source code, proprietary architecture diagrams, production payloads, personal data, credentials, tokens, connection strings, internal URLs or confidential incident details.** Use synthetic examples and redact identifying details. This repository is public.

Use GitHub Issues for actionable defects and proposals. Use Discussions for questions and experience reports if repository Discussions are enabled.

## Disclaimer

This is community-maintained engineering guidance, not an official Microsoft recommendation, certification, security review or guarantee of production readiness. Every recommendation must be validated against the target application's requirements, runtime, dependencies, hosting model, threat model and operational environment. Teams remain responsible for engineering decisions, testing and release approval.

## License

The contents of this repository are licensed under the [MIT License](LICENSE), unless otherwise stated. Third-party names and product marks belong to their respective owners.
