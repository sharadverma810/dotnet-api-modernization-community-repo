# Project principles

These principles guide decisions about the playbook itself.

1. **Preserve responsibilities, not old implementation details.** Identify defects and undocumented behaviour; require an owner decision before changing externally relied-upon outcomes.
2. **Evidence over assumptions.** Trace recommendations to code, tests, official documentation or measurements.
3. **Contract before implementation.** Make externally visible behaviour explicit before refactoring internals.
4. **Functional parity is broader than response equality.** Include side effects, state transitions, security and failure behaviour.
5. **Small, complete vertical slices.** Prefer end-to-end capability increments over broad rewrites and dummy endpoints.
6. **Architecture follows context.** Do not force every API into the same project structure or pattern.
7. **Simpler is better when semantics are preserved.** Avoid recreating removed libraries as custom frameworks.
8. **Performance is measured.** Avoid blanket use of language features and micro-optimizations.
9. **AI operates within guardrails.** Require explicit scope, evidence, tests, stop conditions and human review for risk decisions.
10. **No unsafe automation by default.** Prompts must not authorize production deployment, destructive Git operations or real external side effects without approval.
11. **Public feedback must be sanitized.** The community must not receive confidential code, data or operational details.
12. **The playbook evolves.** Treat technical guidance as versioned and revisit time-sensitive claims.
