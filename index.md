# FIMSPkgdownTemplate

A GitHub Template Repository providing shared
[pkgdown](https://pkgdown.r-lib.org/) theme files for NOAA-FIMS R
packages.

## What this template provides

- **`pkgdown/extra.css`** — shared NOAA-FIMS CSS stylesheet (fonts,
  colours, navbar, footer)
- **`pkgdown/favicon/`** — FIMS favicon set (`.ico`, `.svg`, `.png`,
  web-app manifests)
- **`pkgdown/FIMS_PKGDOWN_TEMPLATE_VERSION`** — version marker used by
  the sync workflow
- **`.github/workflows/sync-fims-pkgdown-template.yml`** — workflow to
  keep the above files up to date in downstream packages (see [Sync
  workflow](#sync-workflow) below)
- **`pkgdown/_pkgdown.yml.example`** — starter config you can copy and
  customise

## What downstream packages must maintain themselves

Each downstream package is responsible for:

- **`pkgdown/_pkgdown.yml`** — site URL, navbar structure, reference
  index, article list, authors block, etc. The sync workflow will
  **never** overwrite this file.
- Any package-specific files and vignettes.

Use `pkgdown/_pkgdown.yml.example` in this repo as a starting point.

## Adopting the template in an existing package

### Option A — Use GitHub’s “Use this template” button

1.  Click **Use this template → Create a new repository** on the
    [template repo
    page](https://github.com/NOAA-FIMS/fims-pkgdown-template).  
    This copies all files into your new repo automatically.
2.  Copy `pkgdown/_pkgdown.yml.example` to `pkgdown/_pkgdown.yml` and
    customise it for your package.
3.  Delete `pkgdown/_pkgdown.yml.example` if you no longer need it.

### Option B — Add pkgdown files to an existing package manually

Run the following from the root of your package repository:

``` bash
TEMPLATE_REPO="https://raw.githubusercontent.com/NOAA-FIMS/fims-pkgdown-template/main"

mkdir -p pkgdown/favicon

# CSS
curl -sSL "${TEMPLATE_REPO}/pkgdown/extra.css" -o pkgdown/extra.css

# Favicons
for f in apple-touch-icon.png favicon-96x96.png favicon.ico favicon.svg \
          site.webmanifest web-app-manifest-192x192.png web-app-manifest-512x512.png; do
  curl -sSL "${TEMPLATE_REPO}/pkgdown/favicon/${f}" -o "pkgdown/favicon/${f}"
done

# Version marker
curl -sSL "${TEMPLATE_REPO}/pkgdown/FIMS_PKGDOWN_TEMPLATE_VERSION" \
     -o pkgdown/FIMS_PKGDOWN_TEMPLATE_VERSION

# Sync workflow
mkdir -p .github/workflows
curl -sSL "${TEMPLATE_REPO}/.github/workflows/sync-fims-pkgdown-template.yml" \
     -o .github/workflows/sync-fims-pkgdown-template.yml
```

Then copy and customise `pkgdown/_pkgdown.yml.example` as your
`pkgdown/_pkgdown.yml`.

## Sync workflow

File: `.github/workflows/sync-fims-pkgdown-template.yml`

This workflow is intended to run in **downstream repos**, not in the
template repo itself.

### Triggers

| Trigger             | Schedule                             |
|---------------------|--------------------------------------|
| `schedule`          | Every Monday at 06:00 UTC            |
| `workflow_dispatch` | Manual trigger via GitHub Actions UI |

### Behaviour

1.  Checks out the downstream repo.
2.  Checks out `NOAA-FIMS/fims-pkgdown-template` at its default branch
    into a temporary directory.
3.  Copies/updates only the template-managed files:
    - `pkgdown/extra.css`
    - `pkgdown/favicon/**`
    - `pkgdown/FIMS_PKGDOWN_TEMPLATE_VERSION`
4.  **Does not touch** `pkgdown/_pkgdown.yml`.
5.  If there are changes, opens a Pull Request to the default branch
    with a clear title and description.
6.  If there are no changes, exits silently.

### Required permissions and secrets

The workflow requests `contents: write` and `pull-requests: write` via
the `permissions` block, which is satisfied by the automatic
`GITHUB_TOKEN` in most repositories.

### Files the workflow will overwrite

| File                                    | Overwritten? |
|-----------------------------------------|--------------|
| `pkgdown/extra.css`                     | ✅ Yes       |
| `pkgdown/favicon/*`                     | ✅ Yes       |
| `pkgdown/FIMS_PKGDOWN_TEMPLATE_VERSION` | ✅ Yes       |
| `pkgdown/_pkgdown.yml`                  | ❌ Never     |

## Opting out or disabling the sync workflow

To disable automatic sync in a downstream repo:

- **Disable the workflow** via the GitHub Actions UI
  (`Actions → sync-fims-pkgdown-template → ⋯ → Disable workflow`), or
- **Delete the workflow file**
  `.github/workflows/sync-fims-pkgdown-template.yml` from your repo.

You can still use the template files without the automatic sync workflow
by simply keeping the files without the workflow.

## Create a gh-pages branch to deploy pkgdown site

After you initialize the template repository or add the files to an
existing repository you will need to copy and paste the following code
into your terminal/git bash to create an empty `gh-pages` branch. Github
should automatically detect that a `gh-pages` branch was created and in
Settings \> Pages switch to publishing Pages from `gh-pages`.

    # from your repo working directory
    git checkout --orphan gh-pages
    git rm -rf . 2>/dev/null || true

    # add something so the branch isn’t totally empty
    touch .nojekyll
    git add .nojekyll
    git commit -m "Initialize gh-pages branch"

    git push -u origin gh-pages

After doing this step, you will want to go to Actions \>
call-update-pkgdown (side nav) and then look for the `Run workflow`
button towards the top right of the screen and click that to re-run your
pkgdown site build and deploy.

## FIMS Website admin

If you have just created a new FIMS package, eventually there will be a
PR in the \[FIMS website repo\] that you will need to review and add the
following: - Appropriate tags - Hex logo (if available)

## If pkgdown build fails

If your pkgdown build fails, please see the [ghactions4r
page](https://nmfs-ost.github.io/ghactions4r/) for more details on
adding ubuntu libraries and other additional arguments to the
`.github/workflow/call-update-pkgdown.yml` file

## Template versioning

`pkgdown/FIMS_PKGDOWN_TEMPLATE_VERSION` contains the current template
version (e.g. `0.1.0`). When the sync workflow opens a PR, the version
will be visible in the PR title and branch name, making it easy to track
which version of the template your package is using.
