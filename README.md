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

`publish` runs `ez publish` only from `workflow_dispatch`. The hub `0x` changes only on publish, so a tag can lead it. The README `0x` stays at the last published package.

```yaml
name: lock-upgrade

on:
  schedule:
    - cron: "0 6 * * 1"
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

`lock-upgrade` runs `ez lock --upgrade` and opens a pull request when the tree changes. `package` is forwarded as `--package` when set. `branch` defaults to `chore/ez-lock-upgrade`. An empty `title` is `Upgrade ez lock`, or `Upgrade ez lock for <package>` when `package` is set. An empty `body` is that command and the diff stat.

The workflow uses `github.token`. The caller needs `contents: write` and `pull-requests: write`. Pull request CI may stay idle under `github.token`; that is accepted. `secrets.token` is an optional escape hatch for checkout, push, and the pull request.
