<!-- markdownlint-disable -->

# Hardening Report: helm--chart-testing-action/v2.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **helm--chart-testing-action/v2.8.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Three `${{ inputs.* }}` expressions are directly interpolated inside a `run:` shell command string in action.yml. The offending lines are:
  `--version ${{ inputs.version }} \`
  `--yamllint-version ${{ inputs.yamllint_version }} \`
  `--yamale-version ${{ inputs.yamale_version }}`
An attacker controlling these inputs (e.g. via `workflow_dispatch` or a calling workflow) can inject arbitrary shell commands. The values must be passed via an `env:` block and then referenced as quoted shell variables (e.g. `"$VERSION"`) instead of being interpolated directly.

Locations:

- `action.yml:28`
- `action.yml:29`
- `action.yml:30`

### github-env-injection (severity: high)

In ct.sh, the `version`, `yamllint_version`, and `yamale_version` variables are populated from CLI arguments that originate directly from `inputs.*` values in action.yml. These user-controlled values flow into `cache_dir` and `venv_dir`, which are then written unsanitized to `$GITHUB_PATH` and `$GITHUB_ENV`:
  `echo "${cache_dir}" >> "${GITHUB_PATH}"`
  `echo "CT_CONFIG_DIR=${cache_dir}/etc" >> "${GITHUB_ENV}"`
  `echo "VIRTUAL_ENV=${venv_dir}" >> "${GITHUB_ENV}"`
  `echo "${venv_dir}/bin" >> "${GITHUB_PATH}"`
Without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) before each write, a newline embedded in an input value can inject arbitrary environment variables or PATH entries into subsequent workflow steps.

Locations:

- `ct.sh:103`
- `ct.sh:106`
- `ct.sh:109`
- `ct.sh:110`

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

Fixed two categories of issues: (1) In action.yml, moved all three ${{ inputs.* }} expressions (version, yamllint_version, yamale_version) out of the run: shell string and into an env: block as CT_VERSION, CT_YAMLLINT_VERSION, CT_YAMALE_VERSION; the shell script now references them as quoted variables. (2) In ct.sh, added sanitization of cache_dir and venv_dir using printf '%s' | tr -d '\n\r' before all four writes to $GITHUB_PATH and $GITHUB_ENV, preventing newline injection attacks.

