# FAQ

## Is this an official Microsoft modernization framework?

No. It is community-maintained guidance. Microsoft documentation is linked where useful, but this repository is not endorsed by Microsoft.

## Does this playbook upgrade projects to .NET 10?

No. It assumes the in-scope APIs are already on .NET 10 (`net10.0`). Framework migration is outside the playbook's scope. The project should use its approved .NET 10 SDK and runtime servicing level.

## Should I create a completely new API project?

Not automatically. First assess whether the existing host, routes, middleware, configuration and deployment boundary are fit for purpose. Prefer in-place or hybrid modernization when it is safer. Create a new host only when the benefits outweigh added migration, integration and deployment risks.

## Does contract parity prove that the new API is correct?

No. Contract parity validates externally observable HTTP behaviour. Functional parity also validates business decisions, state changes, transaction boundaries, authorization, messages/events, downstream calls, idempotency and failure behaviour.

## Should every endpoint initially return dummy data?

No. Use fakes and test adapters to validate the skeleton and contract harness. Then implement complete vertical slices using real business and infrastructure behaviour. Never allow test-only data sources to be registered in production.

## Should I always remove MediatR, FluentValidation, FluentAssertions and Polly?

No. Inventory how each is used and what responsibilities it provides. Remove or replace only when a simpler alternative preserves the required semantics and is better for the application. The project may deliberately retain a dependency when that is the safest choice.

## Does Microsoft.Extensions.Resilience eliminate Polly?

No. Microsoft's resilience integration is built on Polly. It can replace direct Polly usage with a Microsoft.Extensions integration model, but Polly may remain in the dependency graph.

## Are Span<T>, records, pooling or newer C# features automatically faster?

No. Choose them based on semantics, ownership/lifetime constraints and measurement. Readability, correctness and maintainability remain important. Do not make performance claims without comparable measurements.

## Can I paste my company's code into an AI prompt from this repository?

Only when your employer's policies, data classification rules and the AI tool's approved configuration explicitly allow it. This repository does not grant permission to disclose source code or confidential information. Never put proprietary project details into this public repository.

## Can I use the playbook with an agent other than GitHub Copilot?

Yes. The prompts are intended to be tool-agnostic. Adapt tool commands to your agent while preserving the same scope, stop conditions, approval gates and evidence requirements.
