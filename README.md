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

Add reusable workflows under `.github/workflows/` and composite actions under `.github/actions/<name>/`. Consumer repos call them by path and ref; keep inputs and outputs documented in each workflow file.
