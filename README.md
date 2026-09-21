# actions

## Install

Call a workflow from this repository with `uses:`. Pin the ref to a release tag or commit SHA.

```yaml
jobs:
  check:
    uses: Emerging-Patterns/actions/.github/workflows/nix-flake-check.yml@<tag-or-sha>
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
