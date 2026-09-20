# Homebrew tap starter

Create a public Homebrew tap using **Use this template → Create a new repository**.
Choose your account or organization, name the repository `homebrew-tap`, and copy
only the default branch. Replace `OWNER`, `PRODUCT`, and `FORMULA` in the examples
with your own values.

This starter provides an empty tap, shared validation and deletion workflows, and
Dependabot updates. Package registration and updates happen from each product
repository; you do not edit the tap to add a package.

## Check the new tap

Allow GitHub Actions and public reusable workflows in your repository settings.
Run **Actions → test → Run workflow**. An empty tap passes.
Workflows discover the caller repository; no account names need changing in them.

## Connect a product

In your product repository, add `.github/homebrew/formula.yml`:

```yaml
name: example
desc: Example command-line tool
license: MIT
dependencies:
  build:
    - go
install: |
  system "go", "build", *std_go_args, "./cmd/example"
test: |
  system bin/"example", "--version"
```

Adapt installation, dependencies and tests to your product. See
[Homebrew automation](https://github.com/releaseway/homebrew-actions) for prebuilt
GitHub Release assets and the complete workflow contract.

Add this job to your product's pull-request workflow:

```yaml
jobs:
  homebrew:
    permissions:
      contents: read
    uses: releaseway/homebrew-actions/.github/workflows/check.yml@c035d14050bca1e2028a6368823797d36c9cdbc5 # v0.1.0
    with:
      tap-repository: OWNER/homebrew-tap
```

After your product release completes, call the publishing workflow. Your `release`
job must expose the exact source `commit` and `version` as job outputs:

```yaml
jobs:
  homebrew:
    needs: release
    permissions:
      contents: read
    uses: releaseway/homebrew-actions/.github/workflows/publish.yml@c035d14050bca1e2028a6368823797d36c9cdbc5 # v0.1.0
    with:
      tap-repository: OWNER/homebrew-tap
      commit: ${{ needs.release.outputs.commit }}
      version: ${{ needs.release.outputs.version }}
    secrets:
      tap_deploy_key: ${{ secrets.HOMEBREW_TAP_DEPLOY_KEY }}
```

Your existing release process can remain in place.
[releaseway/actions](https://github.com/releaseway/actions) is available if you need
an action to publish immutable GitHub Releases. Release-asset Formulae require a
published immutable release.

## Configure tap write access

The product workflow needs a credential with write access to this tap. One option
is a dedicated SSH deploy key: add its public key to this tap with write access,
and store its private key as `HOMEBREW_TAP_DEPLOY_KEY` in the product's Actions secrets.
The organization must allow deploy keys.

The setup helper can configure that pair without printing the private key:

```sh
git clone https://github.com/releaseway/homebrew-actions.git
git -C homebrew-actions checkout c035d14050bca1e2028a6368823797d36c9cdbc5
TAP_REPO=OWNER/homebrew-tap SOURCE_REPO=OWNER/PRODUCT \
  bash homebrew-actions/scripts/setup-deploy-key.sh
```

Alternatively pass a suitably scoped token as the `tap_token` secret instead of
`tap_deploy_key`. Supply exactly one. Direct pushes by that credential must be
allowed by the tap's branch policy. A normal product `GITHUB_TOKEN` does not have
write access to a separate tap.

Repository secrets and permissions require setup; they are not provided by this
starter. Credentials stay in your repositories.

## Install and update

Once the product workflow publishes its Formula:

```sh
brew install OWNER/tap/FORMULA
brew update
brew upgrade FORMULA
```

The first publish creates the Formula. Later releases update it through the same
workflow. Add another product through that product's spec and workflow without
changing this tap's files or workflows. Change installation behavior in the
product spec; generated Formula edits are replaced by the next publish.

## Remove a Formula

Run **Actions → delete formula** with the Formula name. Select `dry-run` to preview.
A normal run deletes the Formula and pushes to the default branch using the tap's
workflow token. A missing Formula fails. Existing installations remain installed.

## Maintain this tap

Dependabot proposes updates to the pinned shared workflows. Review and merge those
updates to receive automation improvements. Files added to the starter later are
not automatically copied into existing taps.

Public workflows and implementation live in
[homebrew-actions](https://github.com/releaseway/homebrew-actions). This repository
contains only the initial tap configuration.

## License

MIT. Package licenses are declared independently in their product Formula specs.
