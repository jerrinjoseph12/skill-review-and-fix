# CI/CD Pipeline Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

Covers GitHub Actions (`.github/workflows/*.yml`), Jenkins (`Jenkinsfile`), Bitbucket Pipelines (`bitbucket-pipelines.yml`), GitLab CI (`.gitlab-ci.yml`) and Azure Pipelines (`azure-pipelines.yml`).

## Authorities
- GitHub *Security hardening for GitHub Actions*
- OWASP Top 10 CI/CD Security Risks
- SLSA framework
- Jenkins pipeline best practices and the credentials binding docs
- Bitbucket, GitLab and Azure DevOps security docs

## Tooling to look for and run
- `actionlint`, `zizmor` (GitHub Actions)
- `checkov -d .github` or the relevant scanner for the platform
- Shell steps: `shellcheck` (see [shell-scripts.md](shell-scripts.md))

## Security
- `[C]` GitHub `pull_request_target` or `workflow_run` workflows that check out and build the PR's head code. Untrusted code then runs with secrets and a write token.
- `[C]` Script injection: untrusted context values placed directly in `run:` (`${{ github.event.issue.title }}`, `github.head_ref`, PR body or branch names). Pass them through `env:` and quote them.
- `[C]` Secrets printed with `echo`, or written to artifacts, caches or logs; `set -x` in steps that handle secrets.
- `[C]` Jenkins Groovy string interpolation of credentials, such as `sh "curl -u ${PASSWORD} ..."`. Use single quotes so the shell expands the variable, inside `withCredentials`.
- `[H]` Third-party actions, pipes or orbs referenced by a mutable tag or branch (`@v3`, `@main`). Pin them to a full commit SHA, as the project's policy requires.
- `[H]` `permissions:` missing or set to `write-all`. Declare least-privilege permissions per workflow or job.
- `[H]` Long-lived cloud keys stored as secrets where OIDC federation is available (`id-token: write` plus a role assumption).
- `[H]` Self-hosted runners used by workflows triggered from forks or public repos.
- `[M]` Deployment secrets available to every branch. Use protected or environment-scoped secrets, such as GitLab protected variables or Bitbucket deployment variables.

## Correctness and reliability
- `[H]` `continue-on-error: true`, `allow_failure: true`, `|| true` or `catchError` hiding failing tests or scans.
- `[H]` Production deploys triggered from any branch or on PRs, without environment protection, approvals or a branch filter.
- `[H]` Required tests or security scans removed, skipped or made conditional, so they no longer run on the default branch.
- `[M]` Cache keys that don't include the lockfile hash, so stale dependencies are restored; caches shared between untrusted and trusted workflows (cache poisoning).
- `[M]` Multi-line `run:` steps without fail-fast behavior. Bash is fail-fast by default on GitHub but not in custom shells, Jenkins `sh` or Bitbucket scripts, so add `set -euo pipefail`.
- `[M]` No `timeout-minutes` or `timeout`, so a hung job uses up the runner; no `concurrency` group for deploy jobs.

## Maintainability
- `[L]` Duplicated job definitions that should use reusable workflows, templates or YAML anchors; tool versions not pinned (`node-version: latest`); unclear job and step names.
