# Hardening Report: helm--chart-testing-action/v2.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **helm--chart-testing-action/v2.8.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates attacker-controlled `inputs.*` expressions into the shell command string without first assigning them to environment variables. Specifically, `${{ inputs.version }}`, `${{ inputs.yamllint_version }}`, and `${{ inputs.yamale_version }}` are embedded directly in the shell command passed to `./ct.sh`. A caller supplying a malicious value (e.g., containing shell metacharacters or command substitution) could achieve arbitrary code execution on the runner. The fix is to assign each input to an `env:` variable and reference those variables in the `run:` block instead.

Locations:

- `action.yml:28`
- `action.yml:29`
- `action.yml:30`

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

Fixed script injection in actions/hardened/helm--chart-testing-action/v2.8.0/action.yml by moving ${{ inputs.version }}, ${{ inputs.yamllint_version }}, and ${{ inputs.yamale_version }} from the run: block into an env: block as CT_VERSION, CT_YAMLLINT_VERSION, and CT_YAMALE_VERSION respectively. The shell script now references these as plain environment variables with proper quoting, eliminating the risk of shell metacharacter injection.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed ct.sh to sanitize attacker-controlled version inputs (version, yamllint_version, yamale_version) before they are used in paths written to $GITHUB_PATH and $GITHUB_ENV. Applied two layers of defense: (1) early sanitization of all three version variables at the start of install_chart_testing() using `printf '%s' "${var}" | tr -d '\n\r'`, and (2) write-time sanitization of the derived cache_dir and venv_dir path strings at all four GITHUB_PATH/GITHUB_ENV write sites (lines 100, 103, 106, 107) using the same pattern. This prevents newline injection attacks where a malicious version string could inject arbitrary environment variables or PATH entries into subsequent workflow steps.

