# Deno Update Action

Like Dependabot, but for Deno dependencies. Automatically checks for outdated dependencies and creates individual PRs for each update.

## Features

- 🔍 Scans for outdated Deno dependencies using `deno outdated`
- 📦 Creates individual PRs per dependency (Dependabot-style)
- 🔄 Updates existing PRs when newer versions are available
- 🎯 Configurable version update policy (latest or compatible)
- 🏷️ Customizable PR labels
- 🔒 Automatically preserves semver prefixes (`^`, `~`) in `deno.json`

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
          version-policy: latest # or "compatible"
          pr-labels: dependencies,deno
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|------|
| `github-token` | GitHub token with PR creation permissions | Yes | - |
| `version-policy` | Update policy: `latest` or `compatible` | No | `latest` |
| `working-directory` | Directory to run commands in | No | `.` |
| `pr-labels` | Comma-separated PR labels | No | `dependencies,deno` |
| `dry-run` | If `true`, only check for updates without creating PRs | No | `false` |
| `recursive` | If `true`, checks workspace members recursively | No | `false` |

## Version Policies

- **`latest`** (default): Updates to the absolute latest version available, including major breaking changes. Uses `deno update --latest`.
- **`compatible`**: Updates to the newest version that respects semver constraints in your `deno.json`. Uses `deno update --compatible`. For example:
  - `~1.0.0` → updates to latest `1.0.x` (patch updates only)
  - `^1.0.0` → updates to latest `1.x.x` (minor and patch updates)
  - `1.0.0` → updates to latest `1.x.x` (minor and patch updates)

Semver prefixes (`^`, `~`) in `deno.json` are automatically preserved by Deno's update command.

## How It Works

1. Runs `deno outdated` to find outdated dependencies
2. For each outdated dependency:
   - Checks if a PR already exists (branch: `dependencies/deno/<package-name>`)
   - Creates a new branch or updates existing one
   - Runs `deno update --latest` or `deno update --compatible` based on policy
   - Commits and pushes changes (semver prefixes are preserved automatically)
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
