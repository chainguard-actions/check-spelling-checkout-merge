<!-- markdownlint-disable -->

# Hardening Report: check-spelling--checkout-merge/v0.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--checkout-merge/v0.0.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` field in action.yml directly interpolates `${{ github.action_path }}` into the shell command string. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it. Offending line: `run: ${{ github.action_path }}/merge`

Locations:

- `action.yml:47`

### github-env-injection (severity: high)

The `report_failure()` function in the `merge` script writes unsanitized, user-controlled values to `$GITHUB_OUTPUT` and `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The argument `$1` passed to `report_failure` is constructed from `$INPUT_BASE_REF`, `$INPUT_HEAD_REF`, and `$INPUT_PATH`, which are all set from `inputs.*` in action.yml. A newline-containing input value could inject additional key=value pairs. Offending lines: `echo "MERGE_FAILED=1" >> "$GITHUB_ENV"` (line 38), `echo "message=$1" >> "$GITHUB_OUTPUT"` (line 40).

Locations:

- `merge:38`
- `merge:40`

### unpinned-uses (severity: high)

The workflow file uses `actions/checkout@v3`, which is pinned to a mutable tag rather than an immutable 40-character SHA commit hash. This means the action could be silently updated to a different (potentially malicious) version without any change to the workflow file. It should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`.

Locations:

- `.github/workflows/checkout.yml:14`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

1. script-injection (action.yml line 47): Moved `${{ github.action_path }}` from the `run:` field into the `env:` block as `ACTION_PATH: ${{ github.action_path }}`, then changed `run:` to `"$ACTION_PATH/merge"` to avoid template interpolation in the shell command string. 2. github-env-injection (merge lines 38, 40): Added sanitization in `report_failure()` using `safe_message=$(printf '%s' "$1" | tr -d '\n\r')` before writing to `$GITHUB_OUTPUT`, preventing newline injection from user-controlled input values. The `MERGE_FAILED=1` write is a hardcoded constant and safe as-is. 3. unpinned-uses (checkout.yml line 14): Pinned `actions/checkout@v3` to the full immutable SHA `actions/checkout@a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3`.

