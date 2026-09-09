<!-- markdownlint-disable -->

# Hardening Report: check-spelling--checkout-merge/v0.0.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--checkout-merge/v0.0.8** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` field in action.yml directly interpolates `${{ github.action_path }}` and `${{ !endsWith(github.action_path, '/') && '/' || ''}}` expressions inside the shell command string (sub-rule a). Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it, bypassing shell quoting. The entire command to execute is constructed from these expressions: `run: ${{ github.action_path }}${{ !endsWith(github.action_path, '/') && '/' || ''}}merge`.

Locations:

- `action.yml:46`

### github-env-injection (severity: high)

In the `merge` script, the `report_failure` function writes `echo "message=$1" >> "$GITHUB_OUTPUT"` (line 65) where the argument `$1` is a string that embeds `$INPUT_BASE_REF` and/or `$INPUT_HEAD_REF`. These env vars are set directly from `inputs.base_ref` and `inputs.head_ref` (attacker-controlled inputs) in action.yml. The values are written to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`), allowing a newline-injection attack to forge additional output variables. Affected call sites pass strings like `"Can't get history for base_ref ($INPUT_BASE_REF). ..."` and `"Couldn't check out base_ref ($INPUT_BASE_REF); ..."` and `"Can't get head_ref ($INPUT_HEAD_REF). ..."`.

Locations:

- `merge:65`
- `merge:100`
- `merge:104`
- `merge:108`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

1. script-injection (action.yml line 46): Moved `github.action_path` out of the `run:` shell string into the `env:` block as `ACTION_PATH`. The shell script now uses `"${ACTION_PATH%/}/merge"` to construct the script path safely, replacing the `${{ github.action_path }}${{ !endsWith(...) && '/' || ''}}merge` template expression. 2. github-env-injection (merge lines 65, 100, 104, 108): Added `safe_message=$(printf '%s' "$1" | tr -d '\n\r')` at the top of `report_failure()` and replaced all uses of `$1` within the function body with `$safe_message`. This sanitizes attacker-controlled values (from `$INPUT_BASE_REF` and `$INPUT_HEAD_REF`) before they are written to `$GITHUB_OUTPUT`, preventing newline-injection attacks.

