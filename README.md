# app-sources

Version manifests for every application the Nexcess platform installs.

## Why this repo exists

**Application versioning is controlled here and nowhere else.** When a customer provisions
WordPress, Magento, Shopware, Drupal, Craft, ExpressionEngine, or Sylius, `nxcli` asks this repo
which versions exist and which one to install. Adding a line here makes a version installable;
changing one line changes what every new install gets.

There is no code, no build, and no release artifact. Merging to the right branch is the deploy.

Tarballs are a separate concern and are moving out. New applications are served from **pubfiles**
(`pubfiles.nexcess.net`), the internal mirror that hosts application tarballs ready for use and
installation. Magento 2 and Shopware 6 already source from it exclusively; the `*.tar.gz` files
still committed here are legacy.

## Layout

One directory per application, then one per major version, each holding three plain-text files:

```
<app>/<major>/versions          every installable version, one `version|tarball` per line
<app>/<major>/latest-version    the version installs get unless one is named explicitly
<app>/<major>/latest-tarball    legacy; no live consumer
```

[AGENTS.md](AGENTS.md) has the annotated file map, the format rules, the tarball sourcing table, and
the pre-merge checks.

## Branches: master is not production

| Branch | Read by |
|---|---|
| `master` | dev and QE |
| `production` | the production fleet |

The two branches deliberately hold different values — a version is exercised on `master` first, then
promoted to `production` in its own PR. They have diverged since commit `fda2418`; promotion is a
separate commit on a branch cut from `production`, not a merge of `master`.

Two paths pin `master` regardless, so it is not provably customer-free — see
[AGENTS.md](AGENTS.md#conventions).

## Adding a version

1. Branch from the branch you are targeting (`master` for dev/QE, `production` to promote).
2. Add the version to `<app>/<major>/versions`, keeping ascending order.
3. Make sure the tarball is reachable:
   - **pubfiles application** (Magento 2, Shopware 6) — confirm the mirror already serves the
     tarball and its `.md5`. The filename is derived from the version, so the manifest's tarball
     column is not what gets fetched; fill it in truthfully anyway.
   - **legacy application** — the tarball and its `.md5` must be committed alongside the manifest,
     from a clone with `git-lfs` installed.
   - **WordPress** — use the literal `none`; core comes from wordpress.org.
4. Update `latest-version` when the new version should be what installs get. Platform-driven
   provisions always take `latest-version`; they cannot request anything else.
5. Run the checks in [AGENTS.md](AGENTS.md#checks-before-opening-a-pr).
6. Open a PR against `upstream`.

Magento 2 has an extra mandatory step before any tarball is added — see
[magento/2/README.md](magento/2/README.md).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).
