# PLAN-XXX — <Plan title>

## Status

NOT_STARTED | ASSESSMENT_COMPLETE | APPROVED | IN_PROGRESS | BLOCKED | IMPLEMENTED_AWAITING_VALIDATION | VALIDATION_FAILED | PARITY_FAILED | COMPLETE | DEFERRED | REJECTED

## Priority and owner

- Priority: P0 / P1 / P2 / P3
- Owner/reviewer:
- Target API/project:
- Related decision/ADR:

## 1. Goal

What measurable outcome should this plan achieve?

## 1a. Success criteria

- Functional outcome and measurable target:
- Contract compatibility target:
- Quality, reliability or maintainability target:
- Performance/resource target, when applicable:
- Evidence required to demonstrate success:

## 2. Business and technical rationale

Why is the change needed? What risk, cost or limitation does it address?

## 3. Existing responsibilities and behaviour to preserve

List the responsibility IDs, endpoints/triggers, business rules, public contracts, data behaviour, auth and side effects.

## 4. Current state and evidence

- Relevant project/files/types/methods:
- Existing tests:
- Baseline results:
- Known limitations:

## 5. Scope

### In scope

-

### Out of scope

-

## 6. Dependencies and prerequisites

- Upstream plans:
- Downstream plans:
- Required infrastructure/test data:
- Required approvals:

## 7. Target design

Describe the intended implementation, dependency direction and why it is simpler/safer.

## 7a. Clean Architecture and vertical slice alignment

- Feature/capability boundary and responsibility IDs:
- Entry point and use case/command/query:
- Domain rules and invariants:
- Application orchestration:
- Infrastructure adapters and outbound ports:
- Dependency direction and prohibited dependencies:
- Complete vertical-slice boundary:
- Why this structure improves cohesion and independent delivery:

## 8. Implementation steps

1.
2.
3.

## 8a. AI-native delivery approach

- Repository evidence and context the agent must inspect:
- Prompt, task decomposition and deterministic inputs:
- Agent-automated work permitted:
- Human review and approval gates:
- Evidence, traceability and generated-artifact handling:
- Safety boundaries preventing fake, placeholder or incomplete production behavior:

## 9. Impact assessment

- API contract:
- Business behaviour:
- Data/schema/transactions:
- Messaging/events:
- External dependencies:
- Authentication/authorization:
- Resilience/timeouts:
- Observability:
- Performance/memory:
- CI/CD/deployment:

## 10. Test and parity plan

- Unit/component:
- Integration:
- Contract:
- Acceptance/E2E:
- Security/concurrency/resilience:
- Parity scenarios:
- Business outcomes and decision results to compare:
- Data, transaction and concurrency effects to compare:
- Messages, external calls and other side effects to compare:
- Failure, authorization and boundary cases to compare:
- Normalization rules and intentional differences:
- Parity comparator, fixtures and evidence location:
- Side-effect isolation:
- Performance validation:

## 11. Acceptance criteria

- [ ]
- [ ]
- [ ]

## 12. Risks and mitigations

| Risk | Impact | Likelihood | Mitigation | Owner |
|---|---|---|---|---|

Required risk coverage:

- Implementation and dependency risks:
- API/behavioral compatibility risks:
- Data, schema, transaction and message risks:
- Configuration, authorization and deployment risks:
- Observability and performance risks:
- Rollback trigger and decision owner:

## 13. Rollback

- Code and configuration rollback:
- Data/schema rollback or forward-fix strategy:
- Message/event and side-effect recovery:
- Traffic/capability routing or feature-flag rollback:
- Validation required before resuming forward delivery:
- What is the safe rollback path? Do not assume data, schema, messages or external side effects are reversible.

## 14. Validation commands and evidence

| Command/suite | Environment | Expected evidence | Actual result |
|---|---|---|---|

## 15. Definition of done

State objective conditions required before status can become COMPLETE.

## 16. Implementation notes and decision history

Record material deviations and approvals.
