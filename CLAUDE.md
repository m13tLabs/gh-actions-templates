# gh-actions-templates

Reusable GitHub Actions workflows (`workflow_call`) shared across m13tLabs Ansible role repos.
See [README.md](README.md) for what each workflow does and how callers use it.

## Repo layout

- `.github/workflows/ansible-role-ci.yml` — lint/molecule/e2e CI template.
- `.github/workflows/ansible-role-release.yml` — version bump, changelog, GitHub Release.
- `.github/workflows/ansible-role-galaxy.yml` — Ansible Galaxy publish on tag push.
- `.github/workflows/lint.yml` — actionlint, runs on every push/PR to this repo.
- `.github/workflows/self-test.yml` — calls `ansible-role-ci.yml` against the fixture role
  below, on every push/PR to this repo.
- `tasks/`, `meta/`, `molecule/default/`, `tests/e2e/`, `.ansible-lint` — a minimal, throwaway
  Ansible role checked into this repo's *root*, whose only purpose is giving `self-test.yml`
  something real to lint/converge/verify. It is not a publishable role. This repo is
  intentionally dual-purpose (templates + fixture); don't mistake the fixture files for the
  actual product.

## Why the fixture lives at repo root, not a subdirectory

A reusable workflow's own `actions/checkout` step always checks out the *caller's* repo at
the caller's ref, at `$GITHUB_WORKSPACE` root — there is no way to make `ansible-role-ci.yml`
check out or operate against a subdirectory like `tests/fixture-role/` instead. So the only
way to self-test this repo's own CI template against real Ansible tooling (without standing up
a second repo) is to make this repo's root itself look like a role. That's the tradeoff behind
the current design; a separate dedicated fixture repo pinned via `@<branch-or-sha>` was the
alternative considered and rejected for now (more infra, needs cross-repo dispatch/token).

`ansible-role-release.yml` and `ansible-role-galaxy.yml` are **not** self-tested this way — they
push commits/tags and create real GitHub Releases / trigger real Galaxy imports, so running them
against this repo automatically would be destructive. Validate changes to those by pointing a
real consumer repo's caller workflow at your branch/PR SHA before merging.

## Validating changes

```sh
# Workflow syntax / expressions / action-input schema / shellcheck on run: blocks
actionlint .github/workflows/*.yml

# Fixture role lint (profile: min, see .ansible-lint)
ansible-lint

# e2e playbook exactly as ansible-role-ci.yml's end2end job invokes it
cd tests/e2e
ansible-playbook playbook.yml -i "localhost," --connection=local
ansible-playbook verify.yml -i "localhost," --connection=local
cd ../..

# Molecule (needs Docker running)
python3 -m molecule test
```

On macOS with Docker Desktop, `python3 -m molecule test` may fail with
`Unable to contact the Docker daemon` even though `docker info` works fine from the shell —
`docker-py`'s `from_env()` doesn't pick up the Docker Desktop CLI context automatically. Fix by
exporting `DOCKER_HOST` to match `docker context inspect desktop-linux --format
'{{.Endpoints.docker.Host}}'` before running molecule.

`.ansible/` is created locally by `ansible-galaxy`/`ansible-lint`/`molecule` runs — it's
gitignored, don't check it in.

## Known gotchas in these workflow files (caught by actionlint, worth remembering)

- `workflow_call` inputs only support `type: boolean | number | string`. `type: choice` and
  `options:` are `workflow_dispatch`-only — using them on a `workflow_call` input is an invalid
  schema and the workflow won't run at all when called.
- An input can't be both `required: true` and carry a `default:` — the default is unreachable
  and actionlint flags it.
- `actions/create-github-app-token@v3` takes `app-id` (required) + `private-key`. There is no
  `client-id` input — that's a different action's convention, not this one's.
- `mikepenz/action-junit-report@v6`'s `require_tests` input defaults to `false`, so the
  "Publish test report" step in `ansible-role-ci.yml` won't fail the job just because the
  fixture role produces no JUnit XML — no need to wire up a junit callback for the fixture.
- GitHub Actions `run:` blocks execute under `bash -e` (`set -e`) by default. A line of the
  form `[ -f some/optional/file ] && pip install -r some/optional/file` as the **last** line of
  a `run:` block will abort the whole step with exit 1 whenever the file doesn't exist — even
  though nothing actually went wrong, since the compound `test && cmd` statement's own exit
  status is non-zero and it's the final statement in the script. This broke the `molecule` job's
  "Install dependencies" step for any consumer repo without a `molecule/default/requirements.txt`
  (caught live via `self-test.yml`, see git history around 2026-08-22). Fixed by appending a
  trailing `true` after the conditional line. The same conditional pattern in the `lint` job's
  "Install dependencies" step is safe today only because a non-conditional `pip install
  ansible-lint` line follows it — don't reorder that step without keeping something
  unconditional last.
- The molecule job's "Publish test report" step (`mikepenz/action-junit-report@v6`) logs
  `Failed to create checks using the provided token — Resource not accessible by integration`
  when the calling workflow's `GITHUB_TOKEN` lacks `checks: write` (true for `self-test.yml`,
  which doesn't declare a `permissions:` block). This is non-fatal — `fail_on_failure` and
  `fail_on_parse_error` both default to `false` — but real consumer repos wanting the check
  annotation to actually post will need `permissions: checks: write` on their caller job.
