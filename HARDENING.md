<!-- markdownlint-disable -->

# Hardening Report: helm--chart-testing-action/v2.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **helm--chart-testing-action/v2.8.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Three `inputs.*` expressions are interpolated directly inside a `run:` shell command string in action.yml. The values `${{ inputs.version }}`, `${{ inputs.yamllint_version }}`, and `${{ inputs.yamale_version }}` are passed as unquoted CLI arguments to `./ct.sh`. An attacker who controls these inputs (e.g. via `workflow_dispatch` or a calling workflow) can inject arbitrary shell commands. The offending lines are:
  `--version ${{ inputs.version }} \`
  `--yamllint-version ${{ inputs.yamllint_version }} \`
  `--yamale-version ${{ inputs.yamale_version }}`
Fix: route each input through an `env:` variable and double-quote the shell expansion, e.g. `env: { VERSION: "${{ inputs.version }}" }` and then `--version "$VERSION"`.

Locations:

- `action.yml:27`
- `action.yml:28`
- `action.yml:29`

### github-env-injection (severity: high)

In ct.sh, the variables `cache_dir` and `venv_dir` are derived from the user-controlled `version` input (passed via CLI argument from `${{ inputs.version }}` in action.yml) and are written unsanitized to `$GITHUB_PATH` and `$GITHUB_ENV`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any of the four writes. A newline embedded in the `version` input could inject arbitrary environment variable assignments or PATH entries into subsequent workflow steps.

Offending lines in ct.sh:
  `echo "${cache_dir}" >> "${GITHUB_PATH}"`
  `echo "CT_CONFIG_DIR=${cache_dir}/etc" >> "${GITHUB_ENV}"`
  `echo "VIRTUAL_ENV=${venv_dir}" >> "${GITHUB_ENV}"`
  `echo "${venv_dir}/bin" >> "${GITHUB_PATH}"`

Fix: sanitize each value before writing, e.g.:
  `safe_cache_dir=$(printf '%s' "${cache_dir}" | tr -d '\n\r')`
  `echo "${safe_cache_dir}" >> "${GITHUB_PATH}"`

Locations:

- `ct.sh:80`
- `ct.sh:83`
- `ct.sh:86`
- `ct.sh:87`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:28`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.yamllint_version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:29`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.yamale_version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

Fixed two files:

1. action.yml: Moved all three `${{ inputs.version }}`, `${{ inputs.yamllint_version }}`, and `${{ inputs.yamale_version }}` expressions out of the `run:` shell string into an `env:` block (as CT_VERSION, CT_YAMLLINT_VERSION, CT_YAMALE_VERSION). The shell script now uses double-quoted `"$CT_VERSION"`, `"$CT_YAMLLINT_VERSION"`, `"$CT_YAMALE_VERSION"` references, eliminating script injection risk.

2. ct.sh: Added sanitization of `cache_dir` and `venv_dir` using `printf '%s' ... | tr -d '\n\r'` before all four writes to $GITHUB_PATH and $GITHUB_ENV. The sanitized variables `safe_cache_dir` and `safe_venv_dir` are used for all writes, preventing newline-based environment injection attacks.

