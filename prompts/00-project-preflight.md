# Prompt 00 — Project preflight, scope and Git safety

> Run this first in the repository root. It establishes safe context; it is not permission to rewrite code.

```text
Act as a Principal .NET Solution Architect and repository modernization lead.

We are beginning a controlled modernization of an existing production .NET Web API. All in-scope APIs have already been migrated to .NET 10 with minimum changes. This initiative is NOT a framework upgrade. Keep TargetFramework at net10.0. Do not upgrade to .NET 11 or change the deployment runtime as part of this work.

Your task in this phase is project identification, scope definition and safe Git preflight only.

1. Identify the repository root, solution file(s), API project(s), test project(s), shared libraries and the likely runtime entry point(s).
2. Inspect each relevant .csproj, Directory.Build.props/targets, Directory.Packages.props, global.json, NuGet configuration and CI workflow enough to establish target framework and SDK alignment.
3. Confirm the selected API project targets net10.0. Report any mismatch; do not change it.
4. Inspect `git status --short --branch`, current branch, current commit and available main branch references. Do not assume the local `main` is current.
5. Check whether the working tree is clean. Include tracked, untracked and ignored files where relevant to avoid overwriting user work.
6. Propose a unique branch name: `feature/<api-short-name>-modernization-<work-item-or-date>`.
7. If and only if the working tree is safe, the intended base `main` reference is known, and the branch name does not already exist, create the feature branch from that base. Do not overwrite any branch.
8. If there are uncommitted changes, an uncertain base, a branch collision or any risk of losing work, do not switch branches or modify files. Report the exact blocker and safe next step.
9. Do not run `git reset`, `git clean`, force checkout, stash, force push, merge or rebase. Do not discard user changes.
10. Do not edit application source, tests, package references, configuration or deployment files. Do not push, create a PR or deploy.

Produce `docs/modernization/00-scope-and-baseline.md` only if the repository is safe to modify. Otherwise provide the same information in your response without creating or changing files.

Report:
- Repository and solution.
- API project(s) included and excluded.
- Shared project dependencies.
- Target framework and SDK evidence.
- Current branch, base commit and worktree status.
- Proposed/created feature branch.
- Risks or blockers.
- Exact commands executed and their outcomes.

Never claim that a branch was created unless the Git command succeeded and you verified the current branch afterward.
```
