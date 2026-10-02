<!-- markdownlint-disable -->

# Hardening Report: check-spelling--checkout-merge/v0.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--checkout-merge/v0.0.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside the run: shell command string. The line `run: ${{ github.action_path }}/merge` causes GitHub Actions to substitute the expression value into the shell command before the shell ever sees it. Any ${{ ... }} in a run: block is a script-injection risk regardless of which context it reads from.

Locations:

- `action.yml:44`

### github-env-injection (severity: high)

The report_failure() function in the merge script writes `echo "message=$1" >> "$GITHUB_OUTPUT"` without sanitization. The argument $1 is constructed from $INPUT_BASE_REF and $INPUT_HEAD_REF (e.g. `report_failure "Can't get history for base_ref ($INPUT_BASE_REF)..."`) which are set from inputs.base_ref and inputs.head_ref — untrusted caller-controlled inputs. A newline embedded in these values could inject arbitrary key=value pairs into $GITHUB_OUTPUT. The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent before every write.

Locations:

- `merge:41`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

1. script-injection (action.yml line 44): Moved `${{ github.action_path }}` out of the `run:` shell command and into the `env:` block as `ACTION_PATH: ${{ github.action_path }}`. The run command now uses `"$ACTION_PATH/merge"` — a plain environment variable reference with no expression interpolation in the shell string. 2. github-env-injection (merge line 41): In the `report_failure()` function, added `safe_message=$(printf '%s' "$1" | tr -d '\n\r')` before writing to `$GITHUB_OUTPUT`, and updated both the `echo "message=..."` and the `::error ::` annotation to use `$safe_message`. This prevents newline-embedded values in `INPUT_BASE_REF` or `INPUT_HEAD_REF` from injecting arbitrary key=value pairs into `$GITHUB_OUTPUT`.

