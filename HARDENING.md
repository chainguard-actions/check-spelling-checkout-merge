<!-- markdownlint-disable -->

# Hardening Report: check-spelling--checkout-merge/v0.0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--checkout-merge/v0.0.6** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` field in action.yml directly interpolates the expression `${{ github.action_path }}` inside the shell command string. Any `${{ ... }}` expression interpolated directly in a `run:` block is a script-injection risk because the value is substituted by the Actions template engine before the shell ever sees it. Offending line: `run: ${{ github.action_path }}/merge`

Locations:

- `action.yml:44`

### github-env-injection (severity: high)

The `report_failure()` function in the `merge` script writes `echo "message=$1" >> "$GITHUB_OUTPUT"` without sanitization. The argument `$1` is constructed from `$INPUT_BASE_REF` and `$INPUT_HEAD_REF`, which are set from `inputs.base_ref` and `inputs.head_ref` (attacker-controllable via pull request events). An attacker can inject newlines into these values to smuggle arbitrary key=value pairs into `$GITHUB_OUTPUT`. The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent before every write. Affected call sites pass strings like `"Can't get history for base_ref ($INPUT_BASE_REF)..."` and `"Can't get head_ref ($INPUT_HEAD_REF)..."` directly to `report_failure`, which then writes them unsanitized to $GITHUB_OUTPUT.

Locations:

- `merge:37`
- `merge:75`
- `merge:80`
- `merge:84`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two high-severity findings:

1. script-injection (action.yml line 44): Moved `${{ github.action_path }}` from the `run:` field into the `env:` block as `ACTION_PATH: ${{ github.action_path }}`, then referenced it as `"$ACTION_PATH/merge"` in the run command. This prevents template engine substitution directly in the shell command string.

2. github-env-injection (merge script): In `report_failure()`, added sanitization `safe_message=$(printf '%s' "$1" | tr -d '\n\r')` before writing to `$GITHUB_OUTPUT`. The sanitized variable is used for all output writes (GITHUB_OUTPUT, ::error:: annotation, and GITHUB_STEP_SUMMARY), preventing newline injection from attacker-controlled `$INPUT_BASE_REF` and `$INPUT_HEAD_REF` values.

