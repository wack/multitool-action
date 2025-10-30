# MultiTool CLI Github Actions
A GitHub Action to install and run the MultiTool CLI.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version` | Version string (e.g. v0.3.0). If none provided, defaults to latest | No | `latest` |
| `force` | Boolean. Whether to apply the `--force` flag, which skips canarying and immediately deploys the app with 100% traffic. | No | `false` |
| `install-only` | Whether to only install the CLI and then exit | No | `false` |
| `dry-run` | Skips running `multi run`. Useful for testing login credentials. | No | `false` |
| `cache-binary` | When enabled (default), the `multi` binary will be added to the Actions cache and reused between runs | No | `true` |
| `email` | An email address to use when logging in. | No |  |
| `password` | The password to use when logging in. It is recommended to pass this as an environment secret. | No |  |
|  `working-directory` | The working directory to run the `multi run` command in. Defaults to the root of the repository. | No | $PWD

## Examples

Install Only 
``` yaml
- name: Install MultiTool CLI
    uses: wack/multitool-action@v0.0.1
    with:
        install-only: true
```

Install and login:
``` yaml
- name: Install MultiTool CLI and Login
    uses: wack/multitool-action@v0.0.1
    with:
        email: ${{ secrets.MULTITOOL_EMAIL }}
        password: ${{ secrets.MULTITOOL_PASSWORD }}
```

### Using `force` with Different Environments

MultiTool's canary mode gradually rolls out deployments with incremental traffic shifts, which provides safer deployments but takes longer to complete. However, staging and other non-production environments often don't receive enough traffic for canary mode to be effective, making the extended deployment time unnecessary.

The `force` flag allows you to skip canarying and immediately deploy with 100% traffic. This is ideal for non-production environments where you want faster deployments, while still maintaining the safety of canary deployments in production.

Here's an example using GitHub's [environments feature](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment) to deploy to both staging and production, with `force` enabled only for staging:

``` yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [staging, production]
    environment: ${{ matrix.environment }}
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to ${{ matrix.environment }}
        uses: wack/multitool-action@v0.0.1
        with:
          email: ${{ secrets.MULTITOOL_EMAIL }}
          password: ${{ secrets.MULTITOOL_PASSWORD }}
          # Use force for staging (fast deployment), but canary for production (safe deployment)
          force: ${{ matrix.environment != 'production' }}
```

This workflow will:
- Deploy to staging with `force: true` - skipping canary for faster deployments
- Deploy to production with `force: false` - using canary mode for safer, gradual rollouts
