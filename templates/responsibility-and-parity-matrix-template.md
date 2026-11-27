# Responsibility and parity matrix

## A. Responsibility inventory

| Responsibility ID | Capability / trigger | Business purpose | Existing code path | Input | Output | Criticality | Target use case | Plan ID | Status |
|---|---|---|---|---|---|---|---|---|---|
| RESP-001 | | | | | | | | | |

## B. API contract matrix

| Contract ID | Method | Route/version | Request schema | Response schema | Status/error contract | Auth policy | Pagination/filter semantics | Consumer(s) | Contract test |
|---|---|---|---|---|---|---|---|---|---|
| API-001 | | | | | | | | | |

## C. Functional parity matrix

| Scenario ID | Responsibility | Scenario/input | Expected business outcome | Data/state change | Transaction result | Message/event side effect | Downstream call | Security expectation | Old result | New result | Difference classification | Evidence/status |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| PAR-001 | | | | | | | | | | | | |

Difference classification must be one of:

- MATCH
- REGRESSION
- APPROVED_CHANGE
- KNOWN_LEGACY_DEFECT
- NONDETERMINISTIC_WITH_APPROVED_NORMALIZATION
- UNRESOLVED

`UNRESOLVED` blocks plan completion. `APPROVED_CHANGE` must link to a decision/approval. A known defect must include an explicit disposition (preserve for compatibility, fix in this initiative, or defer).

## D. Test traceability matrix

| Responsibility ID | Unit | Component | Integration | Contract | Acceptance/E2E | Security | Resilience | Concurrency/idempotency | Parity evidence | Coverage gap/owner |
|---|---|---|---|---|---|---|---|---|---|---|
| RESP-001 | | | | | | | | | | |
