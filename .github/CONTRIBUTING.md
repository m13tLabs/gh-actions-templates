# Contributing

- Branch from and open pull requests against **develop**.
- Use [Conventional Commits](https://www.conventionalcommits.org/) (`feat(Docker): …`,
  `fix(Release): …`) — the release templates build changelogs from them via `git-cliff`.
- Document new or changed inputs in the workflow's `description:` and in the matching table
  in [DEVELOPER.md](DEVELOPER.md).
- Report bugs and feature requests as
  [issues](https://github.com/m13tLabs/gh-actions-templates/issues).

## Repo layout

Besides the templates in `.github/workflows/`, the repo root holds a minimal, throwaway
Ansible role (`tasks/`, `meta/`, `molecule/default/`, `tests/e2e/`, `.ansible-lint`) that
`pipeline.yml`'s `self-test` job runs `ansible-role-ci.yml` against. Why it lives at the root,
how the templates are built and known pitfalls are in [DEVELOPER.md](DEVELOPER.md).

## Validating changes

`pipeline.yml` runs on every push/PR: [actionlint](https://github.com/rhysd/actionlint) over
all workflows, and `ansible-role-ci.yml` against the fixture role. Locally:

```sh
# Workflow syntax / expressions / action inputs / shellcheck on run: blocks.
# The -ignore flags mask actionlint's stale metadata for actions/create-github-app-token@v3.
actionlint \
  -ignore 'input "client-id" is not defined in action "actions/create-github-app-token' \
  -ignore 'missing input "app-id" which is required by action "actions/create-github-app-token' \
  .github/workflows/*.yml

# Fixture role lint (profile: min)
ansible-lint

# e2e playbook, as ansible-role-ci.yml's end2end job runs it
cd tests/e2e
ansible-playbook playbook.yml -i "localhost," --connection=local
ansible-playbook verify.yml -i "localhost," --connection=local
cd ../..

# Molecule (needs Docker)
python3 -m molecule test
```

On macOS with Docker Desktop, molecule may report `Unable to contact the Docker daemon` even
though `docker info` works: `docker-py` ignores the CLI context. Export
`DOCKER_HOST=$(docker context inspect desktop-linux --format '{{.Endpoints.docker.Host}}')`
first.

`.ansible/` is created by these runs and is gitignored.

### Templates without a self-test

`ansible-role-release.yml`, `ansible-role-galaxy.yml`, `docker-release.yml` push commits/tags
and create real releases, and `docker-ci.yml` needs a real `Dockerfile` at the caller's root.
Validate changes to them by pinning a real consumer repo's caller workflow (e.g.
`openjarvis`) to your branch or PR SHA before merging.
