# Repository setup and publishing guide

This folder is prepared to become a public GitHub repository. The recommended repository name is `dotnet-api-modernization-playbook`.

## Suggested GitHub metadata

- **Name:** `dotnet-api-modernization-playbook`
- **Description:** `A contract-first, test-first, AI-assisted playbook for modernizing existing production .NET Web APIs.`
- **Visibility:** Public
- **Default branch:** `main`
- **License:** MIT
- **Topics:** `dotnet`, `dotnet10`, `csharp`, `aspnetcore`, `webapi`, `software-architecture`, `modernization`, `refactoring`, `testing`, `api-contracts`, `parity-testing`, `github-copilot`, `ai-assisted-development`

Before publishing, replace `OWNER/REPO` in the Markdown workflow badge and `.github/ISSUE_TEMPLATE/config.yml` with the actual GitHub owner and repository name.

## Option A — Create using GitHub CLI

Install Git and GitHub CLI, authenticate, then run these commands from the repository root:

```bash
gh auth login
git init --initial-branch=main
git add .
git commit -m "Initial public release: .NET API modernization playbook"
gh repo create dotnet-api-modernization-playbook --public --source . --remote origin --push
```

If you want the repository under an organization or a specific account, use the qualified name:

```bash
gh repo create YOUR-OWNER/dotnet-api-modernization-playbook --public --source . --remote origin --push
```

Do not publish until you have reviewed the complete file list and confirmed that no confidential information, personal details or unintended metadata is present.

## Option B — Create through the GitHub website

1. Sign in to GitHub.
2. Create a new repository named `dotnet-api-modernization-playbook`.
3. Set visibility to **Public**.
4. Do not initialize with another README, `.gitignore` or license because this package already includes them.
5. Download and extract the repository ZIP, then follow the Git commands below.

```bash
git init --initial-branch=main
git add .
git commit -m "Initial public release: .NET API modernization playbook"
git remote add origin https://github.com/YOUR-OWNER/dotnet-api-modernization-playbook.git
git push -u origin main
```

## Create the issue-form labels

The issue forms reference these labels. Create them once from the repository root after publishing (or create equivalent labels in GitHub):

```bash
gh label create "type: defect" --description "Documentation or prompt defect"
gh label create "type: enhancement" --description "Suggested playbook improvement"
gh label create "type: feedback" --description "Sanitized experience report"
```

If a label already exists, keep it and verify its meaning rather than creating a duplicate.

## Recommended repository features

After creation:

- Enable **Issues** and **Discussions**.
- Enable **Private vulnerability reporting** if available for the repository.
- Confirm the Markdown quality workflow succeeds on the first push.
- Protect `main`: require pull requests, prevent force pushes and branch deletion, and require conversation resolution. Add required status checks after confirming their exact check names.
- Consider requiring one approval if the repository has multiple maintainers. For a personal repository, use a review process appropriate to your capacity.
- Add a repository social preview image if desired; it is optional.
- Add maintainers to `CODEOWNERS` only when actual GitHub usernames are known.
- Confirm the README badge and issue-form contact links use the real repository URL.

## First public release checklist

- [ ] Repository name, description, topics and visibility confirmed.
- [ ] No client/employer code, sensitive information or private links included.
- [ ] `OWNER/REPO` placeholders replaced.
- [ ] MIT License and attribution reviewed by the repository owner.
- [ ] Code of Conduct and private reporting contact route reviewed.
- [ ] Issues and Discussions enabled as intended.
- [ ] Private vulnerability reporting enabled if available.
- [ ] Markdown quality workflow passes.
- [ ] Default branch is `main` and branch protections are configured.
- [ ] First release/tag and changelog entry published.
