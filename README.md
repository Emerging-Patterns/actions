# actions

## Install

Call a workflow from this repository with `uses:`. Pin the ref to a release tag or commit SHA.

```yaml
jobs:
  check:
    uses: Emerging-Patterns/actions/.github/workflows/nix-flake-check.yml@<tag-or-sha>
  release:
    permissions:
      contents: write
    uses: Emerging-Patterns/actions/.github/workflows/github-release.yml@<tag-or-sha>
  tag:
    permissions:
      contents: write
    uses: Emerging-Patterns/actions/.github/workflows/tag.yml@<tag-or-sha>
    with:
      version: ${{ inputs.version }}
      bump: ${{ inputs.bump }}
  publish:
    uses: Emerging-Patterns/actions/.github/workflows/publish.yml@<tag-or-sha>
    with:
      tag: ${{ inputs.tag }}
  lock-upgrade:
    permissions:
      contents: write
      pull-requests: write
    uses: Emerging-Patterns/actions/.github/workflows/lock-upgrade.yml@<tag-or-sha>
    with:
      package: ${{ inputs.package }}
  release-please:
    permissions:
      contents: write
      pull-requests: write
      issues: write
    uses: Emerging-Patterns/actions/.github/workflows/release-please.yml@<tag-or-sha>
```

## Usage

```yaml
name: ci

on:
  pull_request:
  push:

jobs:
  check:
    uses: Emerging-Patterns/actions/.github/workflows/nix-flake-check.yml@main
```

```yaml
name: release

on:
  push:
    tags:
      - "v*"

permissions:
  contents: write

jobs:
  release:
    uses: Emerging-Patterns/actions/.github/workflows/github-release.yml@main
```

`github-release` creates a GitHub release for an existing `v*` tag. `release-please` already creates that release, so callers of `release-please` skip this workflow.

```yaml
name: release-please

on:
  push:
    branches: [main] # or master

permissions:
  contents: write
  pull-requests: write
  issues: write

jobs:
  release-please:
    uses: Emerging-Patterns/actions/.github/workflows/release-please.yml@main
```

That caller is thin: it has no `with:` block. The workflow defaults `config-file` to `.github/release-please-config.json` and `manifest-file` to `.github/release-please-manifest.json`. Pass `config-file` and `manifest-file` only when a caller overrides those paths.

**Never squash-merge release-please pull requests.** Use a regular merge (or rebase-merge if you must) so release-please can create the tag/release from the merge. Squash breaks the release.

`release-please` is the automatic path. A push to the default branch opens or updates a release pull request from conventional commits. A breaking change (`BREAKING CHANGE` or `type!:`) bumps major, `feat` bumps minor, and other changelog commits (`fix`, `perf`, `deps`, `revert`) bump patch. `chore`, `docs`, `refactor`, `test`, `ci`, `build`, and `style` stay out of the changelog, so they do not open a release. With nothing to release, the run succeeds and does not tag. A regular merge of the release pull request tags `vX.Y.Z` and creates the GitHub release. The release commit is a `chore`, so that merge does not open another release. release-please writes the notes from `CHANGELOG.md`. Callers should not also run `github-release.yml` on that tag.

Pin `@main` while this workflow is moving, or pin a release tag or commit SHA. `issues: write` is required so release-please can label the pull request.

Leave the `release-type` input empty and commit `.github/release-please-config.json` and `.github/release-please-manifest.json` in the caller. `googleapis/release-please-action` v4 reads extra files from that config. A non-empty `release-type` input ignores `config-file` and `manifest-file`.

`go` maintains `CHANGELOG.md` and versions `flake.nix` through `extra-files`. `simple` rewrites `version.txt`. `node` updates `package.json`, and it is what release-please uses when the config omits `release-type`. Set `include-component-in-tag` to false so the tag is `vX.Y.Z`.

`.github/release-please-config.json`:

```json
{
  "$schema": "https://raw.githubusercontent.com/googleapis/release-please/main/schemas/config.json",
  "packages": {
    ".": {
      "release-type": "go",
      "include-component-in-tag": false,
      "extra-files": [
        {
          "type": "generic",
          "path": "flake.nix"
        }
      ]
    }
  }
}
```

`.github/release-please-manifest.json`:

```json
{
  ".": "0.1.0"
}
```

The manifest entry is the version already released. Mark the version line in `flake.nix`:

```nix
version = "0.1.0"; # x-release-please-version
```

Repos with more version files put them in `extra-files`. Generic files (`version.bend`, READMEs) need `x-release-please-version` on the version line. A VS Code `package.json` uses the JSON updater at that repo's path:

```json
"extra-files": [
  { "type": "generic", "path": "flake.nix" },
  { "type": "generic", "path": "version.bend" },
  { "type": "json", "path": "package.json", "jsonpath": "$.version" },
  { "type": "generic", "path": "README.md" }
]
```

`config-file` and `manifest-file` override `.github/release-please-config.json` and `.github/release-please-manifest.json`. `target-branch` overrides the default branch. Set `bootstrap-sha` in the config to the adoption commit so older history does not open the first release pull request. `secrets.token` is an optional PAT; with the default `github.token`, the tag push does not start another workflow. Outputs include `tag_name` when a root release is created. Hub publish stays a separate `workflow_dispatch` of `publish.yml`.

```yaml
name: tag

on:
  workflow_dispatch:
    inputs:
      version:
        description: "X.Y.Z to tag (empty = version in flake.nix)"
        required: false
        type: string
        default: ""
      bump:
        description: "none, patch, minor, or major"
        required: false
        type: choice
        options:
          - none
          - patch
          - minor
          - major
        default: none

permissions:
  contents: write

jobs:
  tag:
    uses: Emerging-Patterns/actions/.github/workflows/tag.yml@main
    with:
      version: ${{ inputs.version }}
      bump: ${{ inputs.bump }}
```

`tag` is the manual escape hatch. Automatic releases use `release-please`.

```yaml
name: publish

on:
  workflow_dispatch:
    inputs:
      tag:
        description: "Existing tag to publish, e.g. v0.4.0"
        required: true
        type: string

jobs:
  publish:
    uses: Emerging-Patterns/actions/.github/workflows/publish.yml@main
    with:
      tag: ${{ inputs.tag }}
```

`publish` runs `ez publish` only from `workflow_dispatch`. `release-please` does not call it. The hub `0x` changes only on publish, so a tag can lead it. The README `0x` stays at the last published package.

```yaml
name: lock-upgrade

on:
  workflow_dispatch:
    inputs:
      package:
        description: "Package to upgrade"
        required: false
        type: string
        default: ""

permissions:
  contents: write
  pull-requests: write

jobs:
  upgrade:
    uses: Emerging-Patterns/actions/.github/workflows/lock-upgrade.yml@main
    with:
      package: ${{ inputs.package }}
```

`lock-upgrade` is `workflow_call` only. Callers are `workflow_dispatch`. Directors and humans trigger an ordered pass across repos, following the dependency graph. The workflow runs `ez lock --upgrade` and opens a pull request when the tree changes. `package` is forwarded as `--package` when set. `branch` defaults to `chore/ez-lock-upgrade`. An empty `title` is `Upgrade ez lock`, or `Upgrade ez lock for <package>` when `package` is set. An empty `body` is that command and the diff stat.

The workflow uses `github.token`. The caller needs `contents: write` and `pull-requests: write`. Pull request CI may stay idle under `github.token`; that is accepted. `secrets.token` is an optional escape hatch for checkout, push, and the pull request.
