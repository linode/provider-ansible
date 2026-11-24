# provider-ansible (linode fork)

Builds of [crossplane-contrib/provider-ansible](https://github.com/crossplane-contrib/provider-ansible)
with fixes that aren't in an upstream release yet, published to
`ghcr.io/linode/provider-ansible`.

This is the fork's default branch. It only holds this README and a copy of
`.github/workflows/ci.yml`, because GitHub only offers manual runs (`gh
workflow run`) for workflows on the default branch. A manual run uses the copy
of `ci.yml` on the branch it runs on, so this copy only needs to keep
`workflow_dispatch`. The code is on the branches below.

## Release branches

| Branch | Based on | Package versions |
|---|---|---|
| `release-latest-fork` | upstream `main` | `v<next upstream version>-linode.N` |
| `release-0.6-fork` | upstream `v0.6.0` | `v0.6.0-linode.N` |

Each release branch is a merge of the topic branches below, and contains no
commits of its own. Every topic branch is, or will be, an upstream PR. The
topic branches are opened from `glennpratt/provider-ansible`, so changes to
them go through glennpratt's branches first.

| Topic branch | Change | In |
|---|---|---|
| `feat/git-ssh-auth` | Git SSH key auth for roles and collections ([#197](https://github.com/crossplane-contrib/provider-ansible/issues/197)), with `GIT_CRED_DIR` set per run | both |
| `fix/galaxy-install-on-run` | Run `ansible-galaxy install` only before ansible actually runs, not on every reconcile | both |
| `build/pin-python-deps` | Pin the image's Python dependencies (ansible, ansible-runner, ...) with hashes | latest |
| `build/pin-python-deps-v0.6.0` | The same, pinned to the versions shipped in v0.6.0 | 0.6 |
| `ci/publish-dispatch` | Let CI publish to a fork's registry from tags and manual runs | latest |
| `ci/publish-dispatch-v0.6.0` | The same, for the v0.6.0 `ci.yml`, plus the newer `up` CLI that current GitHub runners need to build packages | 0.6 |
| `backport/v0.6.0-git-improvements` | The git SSH and galaxy fixes cherry-picked onto `v0.6.0` | 0.6 |

The 0.6 line has the two fixes cherry-picked onto `v0.6.0`. Its image has
the same Python, ansible and CRDs as upstream v0.6.0. Alpine's git has moved
to a later security release.

## Publishing

One-time setup:

1. Set the registry: `gh variable set XPKG_REG_ORGS -R linode/provider-ansible -b ghcr.io/linode`.
2. After the first publish, make the package public in its settings
   (`https://github.com/orgs/linode/packages/container/provider-ansible/settings`).
   If the package was first pushed some other way, also give this repository
   write access under "Manage Actions access".

To release, push a tag on a release branch. CI publishes the package with the
tag name as its version:

```sh
git tag v0.6.0-linode.1 origin/release-0.6-fork
git push origin v0.6.0-linode.1
```

To publish from any other branch, start CI by hand:

```sh
gh workflow run ci.yml --ref <branch> -f version=v0.0.0-test.1   # registry from XPKG_REG_ORGS
gh workflow run ci.yml --ref <branch> -f xpkg_reg_orgs=ghcr.io/someone-else
```

The package is pushed whether or not the other CI jobs (lint, tests, e2e) in
the same run pass. Check them before using a release.

## Installing

```yaml
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-ansible
spec:
  package: ghcr.io/linode/provider-ansible:v0.6.0-linode.1
```

## Maintenance

- **Adding a fix:** branch from upstream `main` and open an upstream PR. Merge
  the branch into `release-latest-fork`. For the 0.6 line, cherry-pick it
  onto a `-v0.6.0` variant branch and merge that into `release-0.6-fork`.
  Add it to the table above.
- **Following upstream:** merge upstream `main` into `release-latest-fork`.
  When upstream merges or releases a topic branch, drop it from the table.
  When an upstream release contains all of them, retire the release branch.
- **Updating ansible:** edit `cluster/images/provider-ansible/requirements.in`
  and run `./update-requirements.sh` next to it (needs [uv](https://docs.astral.sh/uv/)),
  or `./update-requirements.sh --upgrade-package ansible-core`. Keep its
  `--python-version` in step with the Python base image in the `Dockerfile`.
  Test your playbooks against the new ansible before releasing.
- **Versions:** use the upstream version the branch is based on, or the next
  one for `release-latest-fork`, plus `-linode.N`. Never reuse a tag.
