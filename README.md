# APIsec Surface GitHub Action

Scans your repository with the Surface CLI ([`apisec-surface`](https://pypi.org/project/apisec-surface/)) and uploads the result to APIsec. Surface replaces Code Discovery / Code Bolt; applications created by Code Discovery are updated in place.

## Quick start

1. Add two repository secrets (Settings → Secrets and variables → Actions):
   - `API_DISCOVERY_ENDPOINT`: your APIsec API URL, for example `https://api.apisecapps.com`
   - `API_DISCOVERY_TOKEN`: an APIsec personal access token with the `app:create` permission
2. Add `.github/workflows/api-discovery.yml`:

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

Existing Code Discovery workflows only need the tag changed to `@v1`. The `contents: write` and `pull-requests: write` permissions are no longer needed.

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
| `api-endpoint` | Yes | | APIsec API URL |
| `api-token` | Yes | | APIsec personal access token |
| `repo-path` | No | `.` | Repository root to analyze |
| `dry-run` | No | `false` | Analyze only; do not upload |
| `config-path` | No | `.codediscovery.yml` | Ignored |
| `host-url` | No | | Ignored; set the instance host in the APIsec console |
| `pr-title` | No | | Ignored; no spec PR is opened |
| `pr-body` | No | | Ignored; no spec PR is opened |

## Outputs

| Output | Description |
|--------|-------------|
| `success` | `true` only when the scan completed |
| `application-id` | APIsec application id after a successful upload |
| `endpoints-count` | Number of API routes found |
| `frameworks-detected` | Comma-separated frameworks |
| `spec-path` | Empty (the spec is published by APIsec) |
| `instance-id` | Empty |
| `pr-url` | Empty |

The Action never fails the workflow. Check `success` if a later step should stop.

## How it works

1. Installs Python 3.11 and Surface CLI `1.0.0`
2. Registers or reuses one APIsec application per GitHub repository
3. Uploads the Surface analysis; APIsec publishes the OpenAPI spec

After the first run, set the instance host URL in the APIsec console if the default `/` is not your API base.

## Changes from Code Discovery (`v0.1.x`)

- No pull request is opened and no files are committed to your repository.
- `host-url`, `config-path`, `pr-title` and `pr-body` are accepted but ignored.
- `spec-path`, `instance-id` and `pr-url` outputs are empty.

## Supported frameworks

- **Java**: Spring Boot, Micronaut, JAX-RS (Jersey, Quarkus, RESTEasy), Servlet
- **Python**: FastAPI, Flask, Django
- **JavaScript / TypeScript**: Express, NestJS, Fastify, Koa
- **.NET**: ASP.NET Core, ASP.NET MVC / Web API, Web Forms, gRPC, WCF
- **Ruby**: Rails, Sinatra, Grape, Hanami, Roda, Padrino
- **GraphQL**

Go (Gin) and Argos are not yet supported in Surface. Keep those repositories on `@v0.1.9` until they are.

## Rollback

Pin `@v0.1.9` to run Code Discovery again.
