<p align="center">
  <img src=".github/assets/logo.svg" alt="gh-actions-templates" width="560">
</p>

# gh-actions-templates

Reusable GitHub Actions workflows shared across m13tLabs repos, called via `uses:`.

| Workflow | Purpose |
| --- | --- |
| [`ansible-role-ci.yml`](.github/workflows/ansible-role-ci.yml) | `ansible-lint`, `molecule test` and an end-to-end install for an Ansible role |
| [`ansible-role-release.yml`](.github/workflows/ansible-role-release.yml) | Version bump, docs regeneration, changelog, tag and GitHub Release |
| [`ansible-role-galaxy.yml`](.github/workflows/ansible-role-galaxy.yml) | Ansible Galaxy import on tag push |
| [`docker-ci.yml`](.github/workflows/docker-ci.yml) | `hadolint`, build and smoke test per Dockerfile, multi-arch build check |
| [`docker-release.yml`](.github/workflows/docker-release.yml) | Version bump, changelog, multi-arch build and push, provenance attestation, GitHub Release |
| [`maven-release.yml`](.github/workflows/maven-release.yml) | Version bump, changelog, signed deploy to Maven Central, next `-SNAPSHOT`, GitHub Release |

## Usage

```yaml
jobs:
  ci:
    uses: m13tLabs/gh-actions-templates/.github/workflows/docker-ci.yml@main
    with:
      dockerfiles: '["Dockerfile","Dockerfile.gpu"]'
      smoke_test: docker run --rm "$IMAGE" --version
```

Pin `@main` to track the latest, or a tag/SHA for stability. There is no compatibility
guarantee across changes to `main`. Release workflows authenticate as a GitHub App so they can
push to a protected default branch, and need `secrets: inherit`:

| Secret | Used by |
| --- | --- |
| `CI_PRIVATE_KEY` | Release workflows (GitHub App private key) |
| `ANSIBLE_GALAXY_API_TOKEN` | `ansible-role-galaxy.yml` |
| `DOCKER_HUB_USER` / `DOCKER_HUB_TOKEN` | Docker workflows with `dockerhub: true` |
| `OSS_SONATYPE_USER` / `OSS_SONATYPE_PASS` / `OSS_SONATYPE_GPG_PRIVATE_KEY` / `OSS_SONATYPE_GPG_PASSPHRASE` | `maven-release.yml` |

Caller examples, inputs and outputs for every workflow are in
[.github/DEVELOPER.md](.github/DEVELOPER.md).

## Contributing

See [.github/CONTRIBUTING.md](.github/CONTRIBUTING.md).
