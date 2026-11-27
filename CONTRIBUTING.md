# Contributing

Thank you for helping improve the .NET API Modernization Playbook. The goal is practical, evidence-based guidance that teams can safely adapt to real production APIs.

## Before opening an issue or pull request

1. Read the [README](README.md), [engineering rules](guides/07-engineering-rules-and-ai-guardrails.md) and [feedback guidelines](docs/FEEDBACK-GUIDELINES.md).
2. Search existing Issues and Discussions to avoid duplicates.
3. For a substantial change, open an issue or discussion first so the intended problem and scope can be agreed.
4. Do not include confidential or proprietary information. This repository is public.

## What contributions are useful?

- Corrections to technical guidance, especially when backed by current official documentation.
- Improvements to stage sequencing, approval gates or safety rules.
- Better AI prompts with clear inputs, outputs, stop conditions and validation requirements.
- Synthetic examples that illustrate an implementation or testing technique.
- Improvements to parity testing, API contracts, integration testing, resilience or performance measurement.
- Lessons learned from applying the playbook, provided the description is sanitized.
- Fixes for broken links, unclear language, inconsistent terminology or repository automation.

## What not to contribute

- Employer/client source code, proprietary designs, internal API contracts or unredacted architecture diagrams.
- Production logs, payloads, personal data, credentials, tokens, keys, connection strings or internal hostnames.
- Unverified claims that a package or language feature is always faster, safer or better.
- Prompts that grant agents unrestricted permission to push, merge, deploy, delete data or modify production systems.
- Changes that silently weaken validation, tests, parity requirements or security safeguards.

## Pull request process

1. Fork the repository and create a focused branch, for example `docs/improve-parity-guidance`.
2. Keep one logical concern per pull request.
3. Explain the problem, proposed change and expected benefit.
4. Link related issues or discussions.
5. Update cross-links and the changelog when appropriate.
6. Run the Markdown quality check locally or ensure the GitHub Actions check passes.
7. Confirm that examples are synthetic and contain no secrets or confidential information.
8. Describe any source or official documentation used for technical claims.

## Writing guidance

- Prefer clear, direct language and explain acronyms on first use.
- Distinguish mandatory rules from recommendations and options.
- Make prerequisites, assumptions, risks and stop conditions explicit.
- Use relative links for files within this repository.
- Use fenced code blocks with a language identifier.
- Keep prompts tool-agnostic unless a tool-specific instruction is necessary.
- Avoid claiming that tests passed unless they were actually executed.
- Date time-sensitive guidance and link to authoritative sources.

## Technical claims and sources

Prefer official Microsoft, .NET, GitHub or package-maintainer documentation for product behaviour. Link the relevant page and, where possible, specify the framework or package version to which the claim applies. If a recommendation is based on experience rather than a verified guarantee, label it as a recommendation or observation.

## Review and acceptance

Maintainers may request changes, narrow scope, defer work to the roadmap or close contributions that are unsafe, unsupported or outside the project's purpose. Contributions are reviewed for accuracy, clarity, safety, maintainability and consistency with the playbook principles.

By submitting a contribution, you agree that it may be distributed under the repository's MIT License. Please do not submit content that you do not have the right to license.
