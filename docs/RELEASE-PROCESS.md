# Release process

This repository is versioned as a documentation and prompt playbook. Changes to prompts, gate requirements or strategy can affect how teams perform production work, so releases should summarize behaviour changes clearly.

## Versioning guidance

Use semantic versioning as a communication convention:

- **Patch:** typo fixes, link corrections and clarifications that do not change the recommended process.
- **Minor:** new prompts, templates, examples or backwards-compatible guidance improvements.
- **Major:** changes to core stages, non-negotiable rules, prompt execution order or recommendations that materially alter adoption.

This is a playbook versioning policy, not an API compatibility guarantee.

## Before a release

1. Review merged changes and open issues.
2. Check relative links and Markdown quality.
3. Review all technical links and version-sensitive statements.
4. Confirm examples remain synthetic and contain no secrets or confidential information.
5. Update `CHANGELOG.md`, `CITATION.cff` and version references where appropriate.
6. Summarize migration notes for changed prompts or templates.
7. Create a GitHub release from the intended tag after the default branch is stable.

## Prompt compatibility

When a prompt changes materially, document the reason, expected output change and any required review by teams already using an earlier version. Avoid silently changing a prompt's authority, permissions or stop conditions.
