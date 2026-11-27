# Feedback guidelines

This playbook should improve through real-world use. Positive results, failed attempts, edge cases and disagreements are all useful when shared safely.

## What feedback helps most?

- Which stage or prompt you tried and what you expected.
- Whether the agent found all endpoints, triggers, business rules and integrations.
- Which responsibilities were difficult to discover.
- Whether the selected architecture was appropriate and why.
- Whether contract tests and functional parity tests caught meaningful differences.
- Whether a vertical slice was small enough to validate independently.
- Which package migrations were more complicated than expected.
- Which AI instructions were ambiguous, over-prescriptive or unsafe.
- Which acceptance criteria were missing.

## Suggested feedback format

1. **Context:** broad API style, framework version, hosting model and test framework. Keep the project anonymous.
2. **Stage:** identify the prompt or guide used.
3. **Expected result:** what you expected the agent or process to produce.
4. **Actual result:** what happened, including what was missed.
5. **Impact:** confusion, rework, risk or time saved, described qualitatively unless a measurement is reproducible and non-sensitive.
6. **Suggestion:** what should change in this public playbook?

## Never share these items

- Employer/client names when not authorized for public disclosure.
- Source code, private repository links, proprietary API contracts or internal architecture diagrams.
- Real customer/user records, payloads, identifiers or personal data.
- Tokens, passwords, API keys, certificates, private keys, secrets or connection strings.
- Internal hostnames, network topology, tenant IDs or security configuration that could expose an environment.
- Production logs, stack traces or incident details that reveal confidential information.

Use fake names, synthetic payloads, invented identifiers and generic descriptions. Review screenshots and AI-generated reports before posting; they can contain secrets or proprietary information.

## Where to provide feedback

- Use the [Experience report issue form](../.github/ISSUE_TEMPLATE/field_feedback.yml) for actionable feedback.
- Use the [Improvement proposal form](../.github/ISSUE_TEMPLATE/improvement.yml) for a specific change.
- Use GitHub Discussions for open-ended questions and discussion, if enabled.
- Use a private channel for security and conduct reports; never post sensitive information in public.
