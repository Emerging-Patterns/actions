# actions

Reusable GitHub Actions and workflows for Emerging-Patterns Bend repositories.

## Install

Reference workflows from this repository with `uses:` (pin a tag or commit SHA).

```yaml
jobs:
  example:
    uses: Emerging-Patterns/actions/.github/workflows/<workflow>.yml@<tag>
```

## Usage

`nix-flake-check.yml`:

```yaml
jobs:
  check:
    uses: Emerging-Patterns/actions/.github/workflows/nix-flake-check.yml@main
```

`github-release.yml` (caller triggers on tags):

```yaml
on:
  push:
    tags: ["v*"]
jobs:
  release:
    uses: Emerging-Patterns/actions/.github/workflows/github-release.yml@main
    secrets: inherit
```

