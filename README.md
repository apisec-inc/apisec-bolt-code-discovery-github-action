# Code Discovery GitHub Action

Scans your repository with the Surface CLI (`apisec-code-bolt`) and uploads the result to APIsec. Existing Code Discovery apps are updated in place.

Ship this cutover as **`v1.0.0`** / **`v1`**. Keep **`v0.1.8`** pinned if you need to roll back to Code Discovery.

## Usage

```yaml
name: API Discovery

on:
  push:
    branches: [main]

jobs:
  discover:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: apisec-inc/apisec-bolt-code-discovery-github-action@v1
        with:
          api-endpoint: ${{ secrets.API_DISCOVERY_ENDPOINT }}
          api-token: ${{ secrets.API_DISCOVERY_TOKEN }}
```

`api-endpoint` is the applicationsservice base URL (the same value Surface uses as `--api-url`).

### Dry run

```yaml
- uses: apisec-inc/apisec-bolt-code-discovery-github-action@v1
  with:
    api-endpoint: ${{ secrets.API_DISCOVERY_ENDPOINT }}
    api-token: ${{ secrets.API_DISCOVERY_TOKEN }}
    dry-run: true
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `api-endpoint` | Yes | | APIsec applicationsservice base URL |
| `api-token` | Yes | | APIsec personal access token |
| `repo-path` | No | `.` | Repository root to analyze |
| `dry-run` | No | `false` | Analyse only; do not upload |
| `config-path` | No | `.codediscovery.yml` | Ignored |
| `host-url` | No | | Ignored — set the instance host in the APIsec console |
| `pr-title` | No | | Ignored — no spec PR is opened |
| `pr-body` | No | | Ignored — no spec PR is opened |

## Outputs

| Output | Description |
|--------|-------------|
| `success` | `true` only when the CLI exited 0 |
| `application-id` | APIsec application id after a successful upload |
| `endpoints-count` | Number of API routes found |
| `frameworks-detected` | Comma-separated frameworks |
| `spec-path` | Empty (spec is published by the engine) |
| `instance-id` | Empty |
| `pr-url` | Empty |

The Action never fails the workflow. Check `success` if a later step should stop.

## How it works

1. Installs Python 3.11 and Surface CLI `0.1.11`
2. Registers or reuses one APIsec application per GitHub repo (`github:<repository_id>`)
3. Uploads the Surface manifest; the reasoning engine publishes the OpenAPI spec

Set the instance host URL in the APIsec console after the first run if the default `/` is not your API base.

## Requirements

- Python 3.11 (installed by the Action)
- An APIsec PAT with `app:create`

## Rollback

Pin `@v0.1.8` to run Code Discovery again.
