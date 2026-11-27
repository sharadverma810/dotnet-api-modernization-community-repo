# 09 — Applying the playbook across multiple Web APIs

## Recommendation

Apply the method **one API modernization boundary at a time**, not as one portfolio-wide refactor across every API. You can standardize the discovery template, test approach, architecture principles and quality gates, but each API needs its own responsibility map, dependency graph, risk profile and parity suite.

## Portfolio-level inventory

Before selecting the first API, create a portfolio register:

| Field | Description |
|---|---|
| API ID/name | Stable internal name |
| Business capability | What business area it supports |
| Criticality | Business / customer / financial / operational impact |
| Traffic profile | Typical and peak request rates if known |
| Dependency count | Databases, queues, topics, caches, external services |
| Contract consumers | Internal/external APIs and client applications |
| Data ownership | Data stores and domain responsibility |
| Current architecture health | Coupling, complexity, testability, known risks |
| Test maturity | Coverage quality, integration and contract test availability |
| Operational health | Incidents, latency, error rates, observability |
| Shared-library impact | Common packages/components affected |
| Modernization status | Not assessed / assessed / approved / in progress / complete |

## Choosing the first API

Do not automatically choose the smallest API or the most complex API.

Choose a pilot API that is:

- Representative of the architecture patterns used by other APIs.
- Important enough to validate the playbook against real engineering constraints.
- Bounded enough that the team can complete assessment and one or two full slices.
- Supported by accessible test data and testable dependencies.
- Not so operationally risky that early architecture learning threatens a critical delivery.

A good pilot helps identify reusable standards (for example parity harness conventions, logging, validation approach and CI reports) without assuming every API has the same business architecture.

## Rollout sequence

1. Inventory all APIs and their shared dependencies.
2. Select one pilot API and apply the complete playbook.
3. Review the pilot's architecture, cost, defects, cycle time and test evidence.
4. Convert proven, genuinely reusable elements into shared templates/libraries only where appropriate.
5. Select the next API based on risk reduction and business priority.
6. Repeat per API; do not copy the pilot's folder structure or abstractions blindly.
7. Migrate shared libraries only with a separate compatibility plan and consumer inventory.

## Shared libraries

A shared library can affect many services. Before changing it:

- List all consumers and versions.
- Determine which APIs deploy together and which deploy independently.
- Preserve binary/source compatibility where required.
- Avoid changing all consumers in a single broad feature branch unless explicitly planned.
- Consider additive API evolution and versioning.
- Add consumer-level contract tests where practical.
- Coordinate package versions and release sequence.
- Keep shared library modernization separate from API capability migration when this improves isolation.

Do not introduce a new shared framework for application patterns solely to make all APIs look identical. Standardize cross-cutting infrastructure only when there is a real shared concern and clear ownership.

## Portfolio metrics

Track outcomes across APIs:

- Number of responsibilities mapped and migrated.
- Number of API/functional parity scenarios passed.
- New regression defects and escaped defects.
- Critical test gaps closed.
- Build/test pipeline duration and reliability.
- Package references removed or deliberately retained.
- Performance deltas on representative endpoints.
- Deployment/rollback issues.
- Maintenance complexity and team feedback.

Do not compare APIs solely by lines changed or package count. The strongest outcome is reduced operational and change risk with preserved business behaviour.
