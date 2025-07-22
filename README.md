# MultiTool CLI Github Actions
A GitHub Action to install and run the MultiTool CLI.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version` | Version string (e.g. v0.3.0). If none provided, defaults to latest | No | `latest` |
| `install-only` | Whether to only install the CLI and then exit | No | `false` |
| `dry-run` | Skips running `multi run`. Useful for testing login credentials. | No | `false` |
| `cache-binary` | When enabled (default), the `multi` binary will be added to the Actions cache and reused between runs | No | `true` |
| `email` | An email address to use when logging in. | No | - |
| `password` | The password to use when logging in. It is recommended to pass this as an environment secret. | No | - |

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