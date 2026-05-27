# Hardening Report: helm--chart-testing-action/v2.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **helm--chart-testing-action/v2.8.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates attacker-controlled `inputs.*` expressions into the shell command string without first assigning them to environment variables. Specifically, `${{ inputs.version }}`, `${{ inputs.yamllint_version }}`, and `${{ inputs.yamale_version }}` are embedded directly in the shell command passed to bash. A malicious input value (e.g., containing shell metacharacters or newlines) could alter the command being executed. These values should be assigned to `env:` variables and referenced as `$VERSION`, `$YAMLLINT_VERSION`, etc.

Locations:

- `action.yml:27`

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

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Moved all three ${{ inputs.* }} expressions (${{ inputs.version }}, ${{ inputs.yamllint_version }}, ${{ inputs.yamale_version }}) from the run: block into an env: block as VERSION, YAMLLINT_VERSION, and YAMALE_VERSION respectively. The shell script now references these safe environment variables with double-quotes ("$VERSION", "$YAMLLINT_VERSION", "$YAMALE_VERSION") instead of directly interpolating GitHub Actions expressions, eliminating the script injection risk.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed ct.sh to sanitize the `version`, `yamllint_version`, and `yamale_version` variables at the start of `install_chart_testing()` using `printf '%s' ... | tr -d '\n\r'` before they are used to construct `cache_dir` and `venv_dir`. Also replaced direct `echo` writes to `$GITHUB_PATH` and `$GITHUB_ENV` with `printf`-based writes that sanitize values inline as defense-in-depth. This prevents newline injection attacks via the `inputs.version` (or other version inputs) value.

