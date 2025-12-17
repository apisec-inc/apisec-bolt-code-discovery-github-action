# Code Discovery GitHub Action

Automatically discover API endpoints in your repository and upload them to your API security platform. This action uses the Code Discovery CLI to scan your codebase, generate OpenAPI specifications, and create pull requests with the results.

## Features

- 🔍 **Automatic Framework Detection** - Detects Spring Boot, Micronaut, FastAPI, Flask, ASP.NET Core, and more
- 📝 **OpenAPI Generation** - Generates OpenAPI 3.0 specifications from your code
- 🔄 **State Management** - Tracks application and instance IDs across runs
- 🔀 **Pull Request Creation** - Automatically creates PRs with discovered endpoints
- ⚠️ **Non-Blocking** - Warnings don't fail your workflow

## Usage

### Basic Example

```yaml
name: API Discovery

on:
  push:
    branches: [main]

jobs:
  discover:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
      - uses: apisec-bolt/code-discovery-action@v1
        with:
          api-endpoint: ${{ secrets.API_DISCOVERY_ENDPOINT }}
          api-token: ${{ secrets.API_DISCOVERY_TOKEN }}
```

### Scheduled Runs

```yaml
name: API Discovery

on:
  schedule:
    - cron: '0 2 * * *' # Daily at 2 AM

jobs:
  discover:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
      - uses: apisec-bolt/code-discovery-action@v1
        with:
          api-endpoint: ${{ secrets.API_DISCOVERY_ENDPOINT }}
          api-token: ${{ secrets.API_DISCOVERY_TOKEN }}
          pr-title: 'chore: update OpenAPI specification [skip ci]'
          pr-body: 'Automatically generated OpenAPI specification from code discovery'
```

### Custom Configuration

```yaml
- uses: apisec-bolt/code-discovery-action@v1
  with:
    api-endpoint: ${{ secrets.API_DISCOVERY_ENDPOINT }}
    api-token: ${{ secrets.API_DISCOVERY_TOKEN }}
    repo-path: './backend'
    config-path: './backend/.codediscovery.yml'
```

### Dry Run (Testing)

```yaml
- uses: apisec-bolt/code-discovery-action@v1
  with:
    api-endpoint: ${{ secrets.API_DISCOVERY_ENDPOINT }}
    api-token: ${{ secrets.API_DISCOVERY_TOKEN }}
    dry-run: true  # Skip API upload, still generates spec
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `api-endpoint` | ✅ Yes | - | External API endpoint URL |
| `api-token` | ✅ Yes | - | Bearer token for API authentication |
| `repo-path` | ❌ No | `.` | Path to repository root to analyze |
| `config-path` | ❌ No | `.codediscovery.yml` | Path to .codediscovery.yml config file |
| `pr-title` | ❌ No | `chore: update OpenAPI specification` | Pull request title |
| `pr-body` | ❌ No | `Automatically generated OpenAPI specification` | Pull request body |
| `dry-run` | ❌ No | `false` | Skip API upload (for testing) |

## Outputs

| Output | Description |
|--------|-------------|
| `spec-path` | Path to generated OpenAPI specification (`apisec-bolt-code-discovery/openapi_spec.yaml`) |
| `application-id` | Application ID from API (if uploaded) |
| `instance-id` | Instance ID from API (if uploaded) |
| `pr-url` | URL of created pull request |
| `success` | Whether discovery succeeded |

## Required Permissions

This action requires the following GitHub permissions:

- `contents: write` - To commit generated files
- `pull-requests: write` - To create pull requests

Add these permissions to your workflow:

```yaml
permissions:
  contents: write
  pull-requests: write
```

## How It Works

1. **Install CLI** - Installs Code Discovery CLI (version 0.2.0)
2. **Configure Credentials** - Sets up API credentials from inputs
3. **Run Discovery** - Scans repository and generates OpenAPI spec
4. **Create PR** - Creates a new branch, commits files, and opens a pull request

### Generated Files

The action generates files in the `apisec-bolt-code-discovery/` directory:

- `openapi_spec.yaml` - OpenAPI 3.0 specification
- `state.yaml` - State file tracking application and instance IDs

Both files are automatically committed to the pull request.

## State Management

The action uses state management to track your application across runs:

- **First Run**: Creates a new application and stores IDs in `state.yaml`
- **Subsequent Runs**: Updates the existing application using stored IDs

The state file (`apisec-bolt-code-discovery/state.yaml`) is safe to commit and contains no sensitive data.

## Error Handling

The action is designed to be non-blocking:

- ⚠️ Discovery warnings don't fail the workflow
- ⚠️ API errors are logged but don't stop execution
- ✅ Workflow continues even if discovery encounters issues

## Supported Frameworks

- **Java**: Spring Boot, Micronaut
- **Python**: FastAPI, Flask
- **.NET**: ASP.NET Core

More frameworks coming soon!

## Requirements

- Python 3.9+ (available on GitHub Actions runners)
- Git (for PR creation)
- GitHub CLI (`gh`) - pre-installed on GitHub Actions runners

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Support

For issues and questions, please open an issue in the [repository](https://github.com/apisec-bolt/code-discovery-action).

