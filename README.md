# Change Champion Action

GitHub Action for automating releases with [change-champion/champ](https://github.com/change-champion/champ).

Handles changeset checking, release PR creation, git tagging, and GitHub Release publishing.

## Usage

```yaml
name: Change Champion

on:
  pull_request:
    branches: [ main ]
    types: [ opened, synchronize, reopened, closed ]
  push:
    branches: [ main ]
    paths:
      - '.changes/*.md'
      - '!.changes/README.md'
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write
  issues: write

jobs:
  change-champion:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
        with:
          ref: ${{ github.event.pull_request.merge_commit_sha || github.sha }}
          fetch-depth: 0
          fetch-tags: true
          persist-credentials: false

      - uses: change-champion/action@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Input            | Description                                                   | Default               |
|------------------|---------------------------------------------------------------|-----------------------|
| `token`          | GitHub token for creating PRs and releases                    | `${{ github.token }}` |
| `php-version`    | PHP version to use                                            | `8.2`                 |
| `version`        | Version of change-champion CLI to install                     | `latest`              |
| `check`          | Whether to check for changesets on PRs and comment if missing | `true`                |
| `create-tag`     | Whether to create and push a git tag on release PR merge      | `true`                |
| `create-release` | Whether to create a GitHub Release on release PR merge        | `true`                |
| `publish`        | Deprecated: use `create-tag` and `create-release` instead     | `true`                |

## Outputs

| Output              | Description                                |
|---------------------|--------------------------------------------|
| `published`         | Whether a release was published            |
| `version`           | The released or next version               |
| `hasChangesets`     | Whether there are pending changesets       |
| `pullRequestNumber` | The pull request number if one was created |

## What It Does

The action detects the GitHub event type and runs the appropriate workflow:

- **PR opened/updated** — Checks for changeset files and comments if missing
- **Push to main** — Runs `champ version` to process changesets, then creates a release PR
- **Release PR merged** — Creates a git tag and GitHub Release

## Setup

1. Run `champ init` to set up the `.changes/` directory (requires
   the [change-champion CLI](https://github.com/change-champion/champ))
2. Add the workflow above to `.github/workflows/change-champion.yml`
3. Go to **Settings > Actions > General > Workflow permissions** and enable:
    - "Read and write permissions"
    - "Allow GitHub Actions to create and approve pull requests"

## Configuration

The action reads from your project's `.changes/config.json` to control its behavior:

| Key                   | Default      | Description                                              |
|-----------------------|--------------|----------------------------------------------------------|
| `releaseBranchPrefix` | `"release/"` | Branch prefix for release PRs (e.g., `release/v1.2.0`)   |
| `versionPrefix`       | `"v"`        | Prefix for git tags and version headers (e.g., `v1.2.0`) |
| `draftRelease`        | `false`      | Create GitHub Releases as drafts for manual publishing   |

This file is created by `champ init`. See the [champ CLI docs](https://github.com/change-champion/champ) for
all
configuration options.

## Sub-actions

Individual sub-actions are available for more granular control:

- `change-champion/action/actions/setup` — Setup PHP and install the CLI
- `change-champion/action/actions/check` — Check for changesets on PRs
- `change-champion/action/actions/version` — Process changesets and create release PR
- `change-champion/action/actions/publish` — Create git tag and GitHub Release

## License

The MIT License (MIT). Please see [License File](LICENSE) for more information.
