# Deno Update Action

Like Dependabot, but for Deno dependencies. Automatically checks for outdated dependencies and creates individual PRs for each update.

## Features

- 🔍 Scans for outdated Deno dependencies using `deno outdated`
- 📦 Creates individual PRs per dependency (Dependabot-style)
- 🔄 Updates existing PRs when newer versions are available
- 🎯 Configurable version update policy (latest, compatible, patch-only)
- 🏷️ Customizable PR labels

## Usage

```yaml
name: Deno Dependency Updates
on:
  schedule:
    - cron: '0 0 * * 1' # Weekly on Mondays
  workflow_dispatch: # Manual trigger

jobs:
  update-dependencies:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - name: Checkout repository
        uses: actions/checkout@v6

      - name: Setup Deno
        uses: denoland/setup-deno@v2
        with:
          deno-version: 2.6.9

      - uses: johngeorgewright/workflows/actions/deno-update@main
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          version-policy: compatible # or "latest" or "patch"
          pr-labels: dependencies,deno
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `github-token` | GitHub token with PR creation permissions | Yes | - |
| `version-policy` | Update policy: `compatible`, `latest`, or `patch` | No | `compatible` |
| `working-directory` | Directory to run commands in | No | `.` |
| `pr-labels` | Comma-separated PR labels | No | `dependencies,deno` |
| `dry-run` | If `true`, only check for updates without creating PRs | No | `false` |
| `recursive` | If `true`, checks workspace members recursively | No | `false` |

## Version Policies

- **`compatible`**: Updates minor and patch versions only (respects semver)
- **`latest`**: Updates all versions including major breaking changes
- **`patch`**: Updates patch versions only

## How It Works

1. Runs `deno outdated` to find outdated dependencies
2. For each outdated dependency:
   - Checks if a PR already exists (branch: `dependencies/deno/<package-name>`)
   - Creates a new branch or updates existing one
   - Updates `deno.json` or `deno.jsonc`
   - Commits and pushes changes
   - Creates a new PR or updates the existing one

## Requirements

- Repository must have a `deno.json` or `deno.jsonc` file
- **Repository must have a `deno.lock` file** (run `deno install` to generate)
- GitHub token must have permissions to create branches and PRs
- Deno dependencies must follow standard version format (e.g., `jsr:@std/fs@1.0.0`)

> **Important:** The action requires a `deno.lock` file to detect outdated dependencies. If you don't have one, run `deno install` in your repository and commit the generated file.


## Example PR

The action creates PRs with titles like:
```
Bump jsr:@std/fs from 0.200.0 to 0.201.0
```

Each PR includes:
- Current and new version
- Update type (major/minor/patch)
- Automatic labels
