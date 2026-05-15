# Mythos CI Verification

`mythos verify --ci` brings Mythos verification into GitHub CI without calling a model.

It is read-only. It does not require an Anthropic key, does not modify files, and does not write to `MEMORY.md`.

## What it checks

`verify --ci` reviews the current PR/diff for high-impact repository changes:

- `package.json` script changes and npm lifecycle hooks
- GitHub Actions workflow changes
- shell, deploy, Docker, and package-manager surfaces
- `.env`, `.npmrc`, private-key-like files, and high-confidence secrets
- changed Mythos receipts under `.mythos/receipts/`

If no Mythos receipt is present, the command still runs in generic PR-review mode. If a receipt is changed in the PR, CI also checks receipt integrity and whether changed files are covered by the receipt.

## GitHub Actions setup

```yaml
name: Mythos Verify

on:
  pull_request:
  push:

jobs:
  mythos-verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-node@v4
        with:
          node-version: 22

      - run: npx mythos-router verify --ci
```

## Exit behavior

Default mode is intentionally review-friendly:

- `INFO` and `WARN` findings pass CI.
- `HIGH` findings fail CI.
- Runtime errors, such as running outside a git repository, exit with code `2`.

Use strict mode if you want warnings to fail CI:

```bash
npx mythos-router verify --ci --strict
```

Use JSON output for downstream tooling:

```bash
npx mythos-router verify --ci --json
```

Compare against a specific base ref:

```bash
npx mythos-router verify --ci --base origin/main
```

## Example finding

```text
WARN package-json-scripts-changed package.json
  package.json scripts changed
  Evidence:
    - scripts.test changed
  Why: Package scripts can execute commands during test, build, install, publish, or CI workflows.
  Recommendation: Review script changes before merge and make sure they match the PR intent.
```
