# MultiTool CLI GitHub Action

A GitHub Action that installs the [MultiTool CLI](https://github.com/wack/multitool) and runs `multi check` to validate your repository's requirements with AI-agent checks.

`multi check` recursively scans your repository for `CHECKS.md` files and runs every check it finds. Each check is a natural-language requirement that an AI agent verifies against your codebase — the kind of non-functional ("-ility") requirement that has no direct unit test (e.g. "every public function is documented", "no image is larger than 5 MB"). The command exits `0` when every requirement is satisfied and `1` when any is not, so it works directly as a CI gate. See the [`multi check` guide](https://github.com/wack/multitool/blob/trunk/guides/checks.md) for details on authoring `CHECKS.md` files.

## Prerequisites

- An API key for your chosen provider, supplied via the provider's native environment variable (see [Credentials](#credentials)).
- A **Linux or macOS runner** (e.g. `ubuntu-latest` or `macos-latest`). `multi check` sandboxes each check in a copy-on-write clone of the working tree — APFS `clonefile` on macOS and reflinks on Linux, with an automatic fallback to a plain copy on Linux filesystems that lack reflink support (e.g. ext4), so any Linux runner works, including GitHub-hosted `ubuntu-latest`. Windows is not yet supported for running checks. (Installation via `install-only` works on any runner.)

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version` | Version string (e.g. `v0.5.0`). If none provided, defaults to the latest release. | No | `latest` |
| `install-only` | When `true`, install the CLI and then exit without running `multi check`. | No | `false` |
| `cache-binary` | When enabled (default), the `multi` binary is added to the Actions cache and reused between runs. | No | `true` |
| `directory` | The directory to recursively scan for `CHECKS.md` files. | No | `.` |
| `provider` | The model provider: `anthropic`, `openai`, or `gemini`. When unset, resolved from env/config (defaults to `anthropic`). | No |  |
| `model` | The concrete model ID (e.g. `claude-sonnet-4-6`). Must be a known model for the selected provider. When unset, resolved from env/config. | No |  |
| `effort` | The agent effort level: `low`, `medium`, or `high`. When unset, resolved from env/config (defaults to `low`). | No |  |
| `concurrency` | Maximum number of checks to run concurrently (a positive integer). When unset, defaults to the runner's CPU core count. | No |  |
| `trace-archive` | When set to a path, capture every agent check session and bundle the traces into this `.tar.gz` archive. | No |  |
| `enable-colors` | Whether to color the output: `always`, `never`, or `auto`. | No | `always` |
| `log-level` | The maximum log level: `trace`, `debug`, `info`, `warn`, `error`, or `off`. When unset, uses the CLI default (`info`). | No |  |

## Credentials

`multi check` reads provider API keys **directly from each provider's native environment variable** — never from an action input or config file. Set the appropriate secret as an environment variable on the job or step:

| Provider  | API key environment variable            |
| --------- | --------------------------------------- |
| Anthropic | `ANTHROPIC_API_KEY`                     |
| OpenAI    | `OPENAI_API_KEY`                        |
| Gemini    | `GOOGLE_API_KEY` (or `GEMINI_API_KEY`)  |

## Examples

### Run checks

```yaml
name: Requirements

on:
  push:
    branches: [main]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Validate requirements
        uses: wack/multitool-action@v1
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

The step fails the job if any requirement is unsatisfied.

### Install only

Install the CLI without running any checks — useful when you want to invoke `multi` yourself in later steps:

```yaml
- name: Install MultiTool CLI
  uses: wack/multitool-action@v1
  with:
    install-only: "true"
    version: "v0.5.0"
```

### Select a provider, model, and effort

```yaml
- name: Validate requirements
  uses: wack/multitool-action@v1
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
  with:
    provider: "anthropic"
    model: "claude-opus-4-8"
    effort: "high"
    concurrency: "8"
```

### Scan a specific directory and capture traces

```yaml
- name: Validate requirements
  uses: wack/multitool-action@v1
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
  with:
    directory: "./services/api"
    trace-archive: "check-traces.tar.gz"

- name: Upload check traces
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: check-traces
    path: check-traces.tar.gz
```
