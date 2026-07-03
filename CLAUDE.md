# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a GitHub Action that installs and optionally runs the MultiTool CLI. It's a composite action that wraps the MultiTool CLI installation and the `multi check` command.

`multi` was previously a canary-deployment tool (`multi login` / `multi run`). It has been repurposed to run an **agentic test suite**: `multi check` recursively scans a repository for `CHECKS.md` files and runs AI agents that attest to non-functional ("-ility") requirements that can't be unit-tested. This action wraps that new behavior. The legacy `login`/`run` subcommands are soft-deprecated in the CLI and are no longer surfaced by this action.

## Architecture

The action is implemented as a **composite action** (action.yml) with two steps:

1. **Install**: Uses `jaxxstorm/action-install-gh-release@v2.1.0` to install the `multi` binary from the `wack/multitool` repository
2. **Run checks**: Executes `multi check` against the specified `directory` (conditional on `install-only` not being `true`), passing through the provider/model/effort/concurrency/trace-archive flags. `multi check` exits `0` when all requirements pass and `1` when any fails, so the step's success/failure is the CI gate.

## Key Files

- `action.yml`: The composite action definition containing all inputs and execution steps
- `.github/workflows/on-push.yml`: CI workflow that tests the action on non-trunk branches
- `README.md`: User-facing documentation with usage examples

## Testing

The action is tested via `.github/workflows/on-push.yml` which:
- Triggers on push to any branch except trunk
- Tests installation by installing a pinned version of the CLI (must be a version that includes `multi check`, e.g. v0.5.0)
- Verifies the binary is in PATH and the version matches
- Verifies the `check` subcommand is available via `multi check --help`

The CI job runs on `ubuntu-latest` and only exercises installation and help/version smoke tests. It does **not** run a real `multi check`: doing so requires a provider API key in the environment. (`multi check` itself runs on both Linux and macOS runners — the check sandbox uses APFS `clonefile` on macOS and reflinks on Linux, with a plain-copy fallback on filesystems without reflink support, so `ubuntu-latest` works. Windows is not yet supported for running checks.)

To test changes locally before pushing:
1. Create a test workflow in a consuming repository (on a `ubuntu-latest` or `macos-latest` runner)
2. Reference your branch: `uses: wack/multitool-action@your-branch-name`
3. Set a provider API key (e.g. `ANTHROPIC_API_KEY`) as an `env` value
4. Use `install-only: "true"` for quick installation-only tests

## Input Handling

All inputs are strings (GitHub Actions limitation). Boolean-like inputs use string values:
- `"true"` / `"false"` for booleans
- Conditional logic in steps uses `${{ inputs.name != 'true' }}`

The `cache-binary` input is translated to the install action's `cache` value with `${{ inputs.cache-binary == 'true' && 'enable' || '' }}`. The comparison against the string `'true'` is required: `inputs.cache-binary` is always a non-empty string (`"true"` or `"false"`), and non-empty strings are truthy in Actions expressions, so the older `inputs.cache-binary && 'enable'` always emitted `'enable'` and caching could never be disabled. The install action caches only when `cache` is exactly `'enable'`; any other value (including empty string) disables it.

Optional `multi check` flags (`provider`, `model`, `effort`, `concurrency`, `trace-archive`, `log-level`) default to an empty string and are only appended to the command when non-empty, using `${{ inputs.x != '' && format('--flag={0}', inputs.x) || '' }}`. This mirrors the CLI's config precedence (flag > env > file): an unset input contributes no flag, so the CLI's env/config resolution is preserved.

## Directory

The `directory` input (defaults to `.`) is passed as the positional argument to `multi check`, which recursively scans it for `CHECKS.md` files. It is relative to the runner's working directory (the repository root after checkout).

## Credentials

`multi check` reads provider API keys **directly from each provider's native environment variable** — never from an action input, command-line argument, or config file. Users must set the appropriate secret as an `env` value on the job or step:

- Anthropic: `ANTHROPIC_API_KEY`
- OpenAI: `OPENAI_API_KEY`
- Gemini: `GOOGLE_API_KEY` (or `GEMINI_API_KEY`)

Composite-action run steps inherit environment variables from the workflow/job/step, so `env:` set by the consumer is visible to the `multi check` step without any input plumbing.
