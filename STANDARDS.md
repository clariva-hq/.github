# Engineering standards (clariva-hq)

These rules apply to **every repository in the clariva-hq organisation**, new and existing. Adopted 2026-11-18.

- A new repo isn't ready until it meets all of them.
- An existing repo that is missing one gets it on its next feature branch.
- Each repo implements the rules in its own files, because toolchains differ; the table under each rule shows how.

## The rules

### 1. PR template
Every repo has `.github/pull_request_template.md`, and every PR fills it in. Sections:
- summary;
- type of change;
- related items;
- what changed;
- how it was tested, **and what wasn't**;
- risk and impact;
- deploy and rollback;
- checklist.

Repos add sections for their stack (e.g. API/SSE risk for the backend, licensing for content). The template in this
repo is the fallback for repos that don't have their own.

### 2. Branch rules
- A GitHub ruleset on `main`:
  - changes only through a pull request;
  - the required CI checks must pass;
  - no force-push;
  - no deletion.
- Locally, the pre-commit `no-commit-to-branch` hook blocks commits on `main`.
- Work happens on feature branches (`feat/…`, `fix/…`, `chore/…`, `docs/…`).
- Keep the ruleset in the repo as `.github/rulesets/protect-main.json`, so it is reviewable and easy to re-apply.

### 3. `main` is the default branch

### 4. Claude reviews every PR
- **Workflow:** `.github/workflows/claude-review.yml` (`anthropics/claude-code-action`).
- **When it runs:**
  - when a PR is opened, reopened or marked ready;
  - again when the `claude-review` label is added;
  - never on draft or bot PRs.
- **Setup:** the Claude GitHub App is installed on the repo, and the `CLAUDE_CODE_OAUTH_TOKEN` org secret is available to it.
  The repo also needs the `claude-review` label, which GitHub doesn't create on its own:
  `gh label create claude-review -R clariva-hq/<repo> --color 25076d --description "Re-run the Claude review"`.
- A human still reviews and merges. Claude's review is advice, not an approval.

### 5. CI/CD
GitHub Actions on every pull request and every push to `main`:
- lint and format check;
- unit tests with coverage (rule 7);
- secret scan with gitleaks, over the full history;
- Dependabot for dependencies and for GitHub Actions: weekly, grouped into one PR per ecosystem.

Deploy and release (CD) is a workflow once the repo ships something. Anything that creates cloud cost is run by the owner.

### 6. Latest linters
Use the current release, pinned in pre-commit and CI, and kept current by Dependabot and `pre-commit autoupdate`.

| Stack | Lint and format |
|---|---|
| Python | ruff (`ruff check`, `ruff format`) |
| TypeScript / web | biome |
| React Native | ESLint + Prettier, or biome |
| Swift | swift-format (with Xcode) or SwiftLint |

### 7. Unit tests, at least 90% coverage
- Unit tests for all code, with at least **90% coverage, enforced in CI**: the job fails below 90%.
- A repo below 90% today:
  - must cover at least 90% of the **changed lines** in every PR (e.g. `diff-cover`);
  - has a total floor that only goes up, tracked in its backlog until it reaches 90%.
- UI views may be excluded when the repo documents that decision. Everything else counts.

| Stack | Coverage tool |
|---|---|
| Python | pytest-cov (`--cov-fail-under`), diff-cover |
| Swift | Xcode coverage (`xccov`) with a gate script |
| TypeScript / React Native | Jest or Vitest coverage thresholds |

### 8. Mobile app rules
Repos with a mobile app also follow [MOBILE.md](MOBILE.md): no magic numbers in UI code, Dynamic Type, every
screen size on iOS and Android, accessibility basics. Each app repo links MOBILE.md from its `AGENTS.md`.

## Who changes what
- **Files in a repo** (workflows, configs, templates, docs) change through a PR, like any code.
- **GitHub settings** are changed by an org owner: rulesets, default branch, app installs, secrets and repo visibility.

## Checklist for a new repo
- [ ] `main` is the default branch, and the ruleset from `.github/rulesets/protect-main.json` is applied
- [ ] `.github/pull_request_template.md`
- [ ] `.github/workflows/ci.yml` (lint, tests with the 90% gate, gitleaks) and `.github/dependabot.yml` (grouped)
- [ ] `.github/workflows/claude-review.yml`, the Claude GitHub App given access, and the token secret available
- [ ] `.pre-commit-config.yaml` (no commits on `main`, gitleaks, the stack's linter)
- [ ] A README with setup, test and lint commands
- [ ] The ruleset's required checks match the CI job names
- [ ] Mobile apps: `AGENTS.md` links [MOBILE.md](MOBILE.md)

Repos with no code (like this one) need rules 1–5, with gitleaks as the only CI check.

## Changing these rules
Open a PR here. Say what changes and why, and which repos need updating.
