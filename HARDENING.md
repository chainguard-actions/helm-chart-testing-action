<!-- markdownlint-disable -->

# Hardening Report: helm--chart-testing-action/v2.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **helm--chart-testing-action/v2.8.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates `${{ inputs.version }}`, `${{ inputs.yamllint_version }}`, and `${{ inputs.yamale_version }}` as unquoted arguments in the shell command string passed to `./ct.sh`. This violates rule (a) — any `${{ ... }}` expression inside a `run:` block is a script-injection risk — and rule (b) — the values are passed unquoted, allowing an attacker-controlled input containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) to execute arbitrary commands. The offending lines are:
  `--version ${{ inputs.version }}`
  `--yamllint-version ${{ inputs.yamllint_version }}`
  `--yamale-version ${{ inputs.yamale_version }}`
Fix: route each input through an `env:` variable and double-quote the shell expansion, e.g. `"$VERSION"`.

Locations:

- `action.yml:25`
- `action.yml:26`
- `action.yml:27`

### github-env-injection (severity: high)

In `ct.sh`, four writes to `$GITHUB_PATH` and `$GITHUB_ENV` use values (`${cache_dir}` and `${venv_dir}`) that are derived from the `version`, `yamllint_version`, and `yamale_version` shell variables. These variables are populated from CLI arguments supplied directly by the `${{ inputs.* }}` interpolations in `action.yml`, making them attacker-controlled. None of the writes are preceded by the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`), so a newline embedded in an input value could inject arbitrary environment variables or PATH entries into subsequent workflow steps.
  Line ~88: `echo "${cache_dir}" >> "${GITHUB_PATH}"`
  Line ~91: `echo "CT_CONFIG_DIR=${cache_dir}/etc" >> "${GITHUB_ENV}"`
  Line ~94: `echo "VIRTUAL_ENV=${venv_dir}" >> "${GITHUB_ENV}"`
  Line ~95: `echo "${venv_dir}/bin" >> "${GITHUB_PATH}"`
Fix: sanitize each value before writing, e.g. `safe=$(printf '%s' "${cache_dir}" | tr -d '\n\r'); echo "${safe}" >> "${GITHUB_PATH}"`.

Locations:

- `ct.sh:88`
- `ct.sh:91`
- `ct.sh:94`
- `ct.sh:95`

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

Fixed two categories of findings:

1. action.yml (script-injection / static-inline-injection): Moved `${{ inputs.version }}`, `${{ inputs.yamllint_version }}`, and `${{ inputs.yamale_version }}` from the `run:` block into an `env:` block as `CT_VERSION`, `CT_YAMLLINT_VERSION`, and `CT_YAMALE_VERSION`. The shell script now references them as double-quoted variables (`"$CT_VERSION"`, etc.), preventing shell metacharacter injection.

2. ct.sh (github-env-injection): Added sanitization before all four writes to `$GITHUB_PATH` and `$GITHUB_ENV`. `cache_dir` is sanitized into `safe_cache_dir` and `venv_dir` into `safe_venv_dir` using `printf '%s' "${var}" | tr -d '\n\r'` before being written, preventing newline injection attacks.

