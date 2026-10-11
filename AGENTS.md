# gh-actions-templates

Reusable GitHub Actions workflows (`workflow_call`) shared across m13tLabs repos: Ansible
role repos and Docker image repos.

- [README.md](README.md): what each workflow is for, minimal caller usage.
- [.github/DEVELOPER.md](.github/DEVELOPER.md): per-template behavior, inputs/outputs,
  implementation notes, repo design and **known gotchas**. Read it before changing a workflow.
- [.github/CONTRIBUTING.md](.github/CONTRIBUTING.md): branching, commit style and the local
  validation commands. Run them before declaring a workflow change done.

## Repo layout

- `.github/workflows/ansible-role-{ci,release,galaxy}.yml`, `docker-{ci,release}.yml`: the
  templates (the product).
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
