<!-- markdownlint-disable -->

# Hardening Report: check-spelling--checkout-merge/v0.0.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--checkout-merge/v0.0.8** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block in action.yml directly interpolates GitHub Actions expressions `${{ github.action_path }}` and `${{ !endsWith(github.action_path, '/') && '/' || ''}}` inside the shell command string. Any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection risk — the YAML template substitution happens before the shell ever sees the value, bypassing shell quoting. Offending line: `run: ${{ github.action_path }}${{ !endsWith(github.action_path, '/') && '/' || ''}}merge`

Locations:

- `action.yml:45`

### github-env-injection (severity: high)

The `report_failure()` function in the `merge` script writes `echo "message=$1" >> "$GITHUB_OUTPUT"` where `$1` is a string embedding `$INPUT_BASE_REF`, `$INPUT_HEAD_REF`, and `$INPUT_PATH`. These variables are set from `inputs.base_ref`, `inputs.head_ref`, and `inputs.path` respectively (untrusted caller-controlled values). No sanitization step (`printf '%s' ... | tr -d '\n\r'`) is applied before the write, allowing a newline character in any of those inputs to inject arbitrary key=value pairs into GITHUB_OUTPUT. The same `report_failure()` function also writes `echo "MERGE_FAILED=1" >> "$GITHUB_ENV"` — that specific write is a literal and safe — but the `message=` write on the next line is not.

Locations:

- `merge:65`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

1. action.yml: Moved `${{ github.action_path }}` out of the `run:` block into an `ACTION_PATH` env var. Changed `run:` to `"$ACTION_PATH/merge"` (plain shell variable). The double-slash-avoidance logic was dropped since Linux paths handle double slashes gracefully. 2. merge script: In `report_failure()`, added `safe_message=$(printf '%s' "$1" | tr -d '\n\r')` before writing to GITHUB_OUTPUT, stripping any newline/carriage-return characters that could inject additional key=value pairs from untrusted inputs (base_ref, head_ref, path).

