**Ticket:** <!-- Hyperlink the issue, e.g. [MAD-123](https://liquidweb.atlassian.net/browse/MAD-123) — required, or state why there isn't one -->

**Target branch:** <!-- `master` (dev + QE) or `production` (customer-facing). Say which and why. -->

## Why

<!-- What problem does this solve, or what requirement does it fulfill? One sentence is usually
enough if there's a ticket — it should contain the details. -->

## What

<!-- Bullet points: which app, which major version, which versions added or removed, and whether
`latest-version` moved. -->

### Blast radius

<!-- Which fleet does this reach on merge? `production` is customer-facing; `master` is dev/QE, with
the caveats in AGENTS.md. A `latest-version` change is the highest-impact edit: platform-driven
installs always take it. -->

## Test plan

**Tarball availability:**
- [ ] pubfiles application (Magento 2 / Shopware 6) — the tarball **and** its `.md5` are served:
      `curl -sfI https://pubfiles.nexcess.net/<path>/<name>-<version>.tar.gz`
      then the same URL with `.md5` appended
- [ ] legacy application — tarball and `.md5` committed alongside the manifest, as LFS pointers
- [ ] WordPress — the tarball column is the literal `none`

**Manifest checks** (see [AGENTS.md](../AGENTS.md#checks-before-opening-a-pr)):
- [ ] every `versions` line is `version|tarball` with both fields populated
- [ ] `latest-version` names a version `versions` lists on this branch
- [ ] the missing-tarball check reports only Magento 2 / Shopware 6 entries
- [ ] the sidecar and LFS checks report only their documented pre-existing hits

**Manual verification:**
1. <!-- e.g. installed <app> <version> on a dev cloud account and confirmed the reported version -->

## Deploy notes

<!-- Delete this section if not needed. -->

**Rollout order:** <!-- promoting to `production` after a dev/QE soak? link the master-side PR -->
**Rollback:** <!-- revert the commit on this branch; nxcli refetches on the next install -->

## Checklist

- [ ] One application version bump per commit, with an `issue:` trailer
- [ ] Branched from the branch being targeted; not a `master` → `production` merge
- [ ] For Magento 2: the `di.xml` patch in [magento/2/README.md](../magento/2/README.md) is applied
- [ ] Docs (`README.md`/`AGENTS.md`) updated if this changes how versions are sourced or promoted
- [ ] No secrets, tokens, or real customer/financial data included
