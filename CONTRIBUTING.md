# Contributing

Version manifests for every application the Nexcess platform installs.

A change here is a live change to what customers can install. There is no CI, no validator, and no
deploy gate — review is the only check, so make the diff easy to verify.

## Development setup

Nothing to install or build. See [AGENTS.md](AGENTS.md#checks-before-opening-a-pr) for the pre-merge
checks — not repeated here.

## Pick the right branch

| Target | Branch |
|---|---|
| Expose a version to dev and QE | `master` |
| Promote a version to customers | `production` |

Branch from the branch you are targeting. Do not merge `master` into `production` — promotion has
been done as its own commit on a branch cut from `production`, carrying only the entries being
promoted. The two branches have diverged since `fda2418`.

## Commits

Conventional Commits plus an `issue:` trailer:

```
chore: set WP latest version to 6.9.5

issue: MAD-XXXXX
```

One version bump per commit. A commit that bumps two applications is doing too much.

## Opening a PR

Use this repo's PR template. Target the `nexcess/app-sources` remote (`upstream` if you are working
from a fork), not your own.

## Before you open a PR

- [ ] The `versions` line is `version|tarball` with both fields populated, in ascending order
- [ ] `latest-version` names a version that `versions` lists on the same branch
- [ ] The tarball and its `.md5` are reachable — on pubfiles for Magento 2 and Shopware 6, committed
      here for the legacy applications, `none` for WordPress
- [ ] Any tarball you committed is a git-LFS pointer, not a plain blob
- [ ] The checks in [AGENTS.md](AGENTS.md#checks-before-opening-a-pr) report only their documented
      pre-existing hits
- [ ] For Magento 2: the `di.xml` patch in [magento/2/README.md](magento/2/README.md) is applied
- [ ] Docs (`README.md`/`AGENTS.md`) updated if this changes how versions are sourced or promoted
