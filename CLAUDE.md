# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a GitHub Action that installs and optionally runs the MultiTool CLI. It's a composite action that wraps the MultiTool CLI installation and authentication process.

## Architecture

The action is implemented as a **composite action** (action.yml) with three main steps:

1. **Install**: Uses `jaxxstorm/action-install-gh-release@v2.1.0` to install the `multi` binary from the `wack/multitool` repository
2. **Login**: Authenticates with MultiTool using email/password credentials (conditional on `install-only` not being true)
3. **Run**: Executes `multi run` in the specified working directory (conditional on both `install-only` and `dry-run` not being true)

## Key Files

- `action.yml`: The composite action definition containing all inputs, outputs, and execution steps
- `.github/workflows/on-push.yml`: CI workflow that tests the action on non-trunk branches
- `README.md`: User-facing documentation with usage examples

## Testing

The action is tested via `.github/workflows/on-push.yml` which:
- Triggers on push to any branch except trunk
- Tests installation by installing v0.3.0 of the CLI
- Verifies the binary is in PATH and the version matches
- Tests authentication using secrets (with dry-run enabled)

To test changes locally before pushing:
1. Create a test workflow in a consuming repository
2. Reference your branch: `uses: wack/multitool-action@your-branch-name`
3. Use `install-only: "true"` for quick installation tests
4. Use `dry-run: "true"` to test login without running commands

## Input Handling

All inputs are strings (GitHub Actions limitation). Boolean-like inputs use string values:
- `"true"` / `"false"` for booleans
- Conditional logic in steps uses `${{ inputs.name != 'true' }}`

The `cache-binary` input uses a special syntax: `${{ inputs.cache-binary && 'enable' }}` to convert the string to the cache action's expected value.

## Working Directory

The `working-directory` input (defaults to `.`) is applied only to the "Run" step, not to the "Login" step. This is intentional as login is a global operation while `multi run` should execute in the user's specified directory.

## Credentials

Email and password are passed directly to the `multi login` command via command-line arguments. Both are wrapped in single quotes to handle special characters: `--email='${{ inputs.email }}'`
