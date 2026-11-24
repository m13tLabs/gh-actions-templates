# gh-actions-templates

Reusable GitHub Actions workflows (`workflow_call`) shared across m13tLabs repos (Ansible
roles, Docker images, Maven/Gradle libraries) and some `ContinuousSecurityTooling` repos
(`zap-java`, `keycloak-auditor`, `keycloak-reporter`).

- [README.md](README.md): what each workflow is for, minimal caller usage.
- [.github/DEVELOPER.md](.github/DEVELOPER.md): per-template behavior, inputs/outputs,
  implementation notes, repo design and **known gotchas**. Read it before changing a workflow.
- [.github/CONTRIBUTING.md](.github/CONTRIBUTING.md): branching, commit style and the local
  validation commands. Run them before declaring a workflow change done.

## Repo layout

- `.github/workflows/ansible-role-{ci,release,galaxy}.yml`, `docker-{ci,release}.yml`,
  `maven-release.yml`, `gradle-release.yml`: the templates (the product).
- `.github/workflows/npm-release.yml`: work in progress, still a `workflow_dispatch`
  workflow, not callable yet and not documented in DEVELOPER.md.
- `.github/workflows/pipeline.yml`: actionlint + `self-test` (`ansible-role-ci.yml` against
  the fixture role), on every push/PR.
- `tasks/`, `meta/`, `molecule/default/`, `tests/e2e/`, `.ansible-lint`: a throwaway fixture
  role at the repo root, only there for `self-test`. Not a publishable role, and it must stay
  at the root (see DEVELOPER.md, "Repo design").

## Rules

- Only `ansible-role-ci.yml` is self-tested. Never run the release/Galaxy templates against
  this repo; they push commits/tags and create real releases. Validate them and the Docker
  templates by pinning a consumer repo (e.g. `openjarvis`) to the branch/PR SHA.
- When adding or changing an input, update its `description:` and the matching table in
  DEVELOPER.md.
- Keep the QEMU setup of `docker-ci.yml` and `docker-release.yml` in sync.
- Never splice `${{ inputs.* }}` into a `run:` script; pass it through `env:`.
- Release templates with a `bump_command` input (`docker-release.yml`, `maven-release.yml`)
  must run it via `env` + `bash -euo pipefail -c` followed by `git add -u`. Keep them alike.
- Callers outside `m13tLabs` can't use `secrets: inherit`; document every secret a template
  needs, since those callers map each one explicitly.
