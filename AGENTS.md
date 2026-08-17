# app-sources

Version manifests for every application the Nexcess platform installs.

## Architecture in a paragraph

**This repo is the only place application versioning is controlled.** Adding a line to a `versions`
file is what makes a version installable; changing `latest-version` is what new installs get. There
is no code, no build, no test suite, and no deploy step — merging to the branch a fleet reads *is*
the deploy, and `nxcli` refetches these files over HTTPS at install time, so nothing here is ever
cloned or cached. The content is three plain-text control files per application major version, laid
out as `<app>/<major>/`. Two long-lived branches make it a two-stage promotion pipeline: `master`
for dev and QE, `production` for the production fleet, deliberately holding different values.
**Tarballs no longer belong here.** New applications are served from **pubfiles**
(`pubfiles.nexcess.net`), the internal mirror hosting application tarballs ready for install;
Magento 2 and Shopware 6 already source from it exclusively, and the committed `*.tar.gz` files are
legacy.

## File map

```
akeneo/2/                 orphaned — no application consumes this manifest
craft/{3,4}/
drupal/{7,8,9}/
expressionengine/{5,6,7}/
magento/1/
magento/2/
  README.md               di.xml MariaDB patch required before adding a Magento 2 tarball
  patches/                legacy Magento patches + .md5 sidecars; no live consumer found
shopware/6/               pubfiles-only: manifests here, tarballs on pubfiles
sylius/1/
wordpress/{4,5,6}/
wordpress/4/plugins/      orphaned — nothing fetches this directory
wordpress/5/plugins/      BigCommerce OAuth connector zips; live, see Conventions
.gitattributes            routes *.tar.gz through git-LFS
```

## The three control files

Every `<app>/<major>/` directory holds the same three files.

| File | Format | Role |
|---|---|---|
| `versions` | one `version\|tarball` pair per line | the full set of installable versions |
| `latest-version` | a single version string | the version installs actually get |
| `latest-tarball` | a tarball filename, or empty | legacy; no live consumer |

Format rules that matter, because nothing validates them on merge:

- **Both fields on every line.** A line with no `|` maps that version to nothing and the install
  fails later at the checksum step, with a misleading error. Use the literal `none` when there is no
  tarball, as every WordPress line does.
- **A trailing blank line is harmless** — the file is trimmed before parsing. `magento/2/versions`
  has one today. An interior blank line is not harmless.
- **Ordering is not parsed**, but keep entries ascending so a reviewer can see what was added.
- **`latest-version` must name a version `versions` lists on the same branch.**
- **Directory names are a contract.** `nxcli` computes the path from the application name and major
  version, so never rename one. Creating a *new* major-version directory (`craft/5/`,
  `wordpress/7/`) does nothing on its own — `nxcli` only reaches majors it explicitly supports, and
  falls back to its newest supported major otherwise.

## Where tarballs come from

The manifests are uniform; tarball sourcing is not. Three modes coexist:

| Mode | Applications | Tarball source |
|---|---|---|
| **pubfiles** (current) | Magento 2, Shopware 6 | `pubfiles.nexcess.net`, filename derived from the version |
| GitHub raw (legacy) | Craft, Drupal, ExpressionEngine, Magento 1, Sylius | committed here, next to the manifest |
| upstream | WordPress | wordpress.org, via `wp core download` |

New applications use pubfiles; do not commit new tarballs. For a pubfiles application the filename
is derived from the version rather than read from the manifest, so adding a version needs two
independent things to be true: the line exists here, **and** pubfiles already serves the tarball.
Fill the tarball column in truthfully regardless — it is what a reviewer scans.

**Every tarball needs an adjacent `.md5`.** The installer fetches `<tarball-url>.md5` and compares
it on every install, pubfiles included, even when the tarball is already cached on the server. A
missing or stale sidecar fails the install. This applies to tarballs published on pubfiles too.

## Checks before opening a PR

There is nothing to build or test. Run these.

```bash
# Every versions line must be `version|tarball` with both fields populated.
# Expected: one "blank line" hit for magento/2/versions — its harmless trailing newline.
git ls-files '*/versions' | while read -r f; do
  awk -F'|' 'NF==0 {print FILENAME": "FNR": blank line"}
             NF!=0 && (NF!=2 || $1=="" || $2=="") {print FILENAME": "FNR": "$0}' "$f"
done

# latest-version must name a version that versions actually lists.
git ls-files '*/latest-version' | while read -r f; do
  d=$(dirname "$f"); v=$(tr -d '\n' < "$f")
  cut -d'|' -f1 "$d/versions" | grep -qxF "$v" || echo "$d: latest-version '$v' absent from versions"
done

# Referenced tarballs with no committed file.
# Expected: only Magento 2 and Shopware 6 entries, which pubfiles serves.
git ls-files '*/versions' | while read -r f; do
  d=$(dirname "$f")
  cut -d'|' -f2 "$f" | grep -vx -e none -e '' | while read -r t; do
    [ -f "$d/$t" ] || echo "$d/$t"
  done
done

# Every committed tarball needs an .md5 beside it.
# Expected: craft/3/craft-3.7.39.tar.gz.md5 — a pre-existing gap.
git ls-files '*.tar.gz' | while read -r t; do
  [ -f "$t.md5" ] || echo "missing sidecar: $t.md5"
done

# Committed tarballs stored as plain blobs instead of git-LFS pointers.
# Expected: seven pre-existing hits under craft/, drupal/9, expressionengine/.
git ls-files '*.tar.gz' | while read -r t; do
  head -c 40 "$t" | grep -q 'git-lfs.github.com' || echo "not LFS: $t"
done

# What is staged on master but not yet promoted to production.
git fetch upstream
git diff upstream/production upstream/master -- '*/versions' '*/latest-version' '*/latest-tarball'
```

## Conventions

- **`master` is not production.** Landing a version on `master` exposes it to dev and QE;
  production reads the `production` branch, promoted by its own PR. Two paths pin `master`
  regardless, so treat it as potentially customer-reachable rather than assumed safe: the
  `wordpress/5/plugins/` mirror below, and the EL6-era `nxcli-legacy` build, which has no branch
  option at all.
- **A merge is a live change.** `nxcli` refetches on every install, so a bad `latest-version`
  affects the next install on that branch's fleet. There is no deploy gate behind this repo.
- **`latest-version` is what platform-driven installs get**, not merely a default. A provision from
  the platform never names a specific version — it always takes `latest-version`. The other
  `versions` entries are reachable only by invoking `nxcli` directly with an explicit version.
- **Before adding any Magento 2 version, read [magento/2/README.md](magento/2/README.md)** — the
  `app/etc/di.xml` MariaDB version pattern must be patched first.
- **`*.tar.gz` is git-LFS tracked** (`.gitattributes`, since 2018), but seven committed tarballs are
  plain git blobs — added by commits that bypassed the LFS filter, such as GitHub web-UI uploads. If
  you must add a tarball, commit it from a clone with `git-lfs` installed and confirm with the LFS
  check above. Prefer pubfiles.
- **`wordpress/5/plugins/*.zip` is live and bypasses promotion.** A cron pulls those zips from the
  `master` branch every 30 minutes and stages them on customer-facing servers. The `wordpress/5`
  source path is used regardless of the WordPress major being installed, which is why
  `wordpress/4/plugins/` is orphaned — and because it pins `master`, editing a zip reaches customers
  without a `production` PR.
- **One version bump per commit**, with the `issue:` trailer — see
  [CONTRIBUTING.md](CONTRIBUTING.md).

## See also

- [README.md](README.md) — what this repo is and how to add a version
- [magento/2/README.md](magento/2/README.md) — mandatory Magento 2 pre-upload patch
- [CONTRIBUTING.md](CONTRIBUTING.md) — branch, commit, and PR conventions
