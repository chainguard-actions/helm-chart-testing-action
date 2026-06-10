<!-- markdownlint-disable -->

# Hardening Report: helm--chart-testing-action/v2.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **helm--chart-testing-action/v2.8.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block in action.yml directly interpolates three user-controlled input expressions into the shell command string without routing them through env vars: `--version ${{ inputs.version }}`, `--yamllint-version ${{ inputs.yamllint_version }}`, and `--yamale-version ${{ inputs.yamale_version }}`. These expressions are expanded by the GitHub Actions template engine before the shell ever sees the command, allowing an attacker who controls these inputs to inject arbitrary shell commands (e.g., a version value of `3.14.0 && malicious-command`).

Locations:

- `action.yml:26`
- `action.yml:27`
- `action.yml:28`

### github-env-injection (severity: high)

In ct.sh, the variable `cache_dir` is constructed as `"${RUNNER_TOOL_CACHE}/ct/${version}/${arch}"` where `version` is derived from the `inputs.version` action input (passed as a CLI argument from action.yml). `venv_dir` is derived from `cache_dir`. All four writes to the special environment files — `echo "${cache_dir}" >> "${GITHUB_PATH}"`, `echo "CT_CONFIG_DIR=${cache_dir}/etc" >> "${GITHUB_ENV}"`, `echo "VIRTUAL_ENV=${venv_dir}" >> "${GITHUB_ENV}"`, and `echo "${venv_dir}/bin" >> "${GITHUB_PATH}"` — use values that trace back to the user-controlled `inputs.version` without the required `printf '%s' ... | tr -d '\n\r'` sanitization. A newline character embedded in `inputs.version` would allow injection of arbitrary key=value pairs into `$GITHUB_ENV` or arbitrary paths into `$GITHUB_PATH`.

Locations:

- `ct.sh:97`
- `ct.sh:100`
- `ct.sh:103`
- `ct.sh:104`

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

Fixed script-injection in action.yml by moving all three ${{ inputs.version }}, ${{ inputs.yamllint_version }}, and ${{ inputs.yamale_version }} expressions into an env: block (as CT_VERSION, CT_YAMLLINT_VERSION, CT_YAMALE_VERSION) and referencing them as plain shell environment variables in the run: block. Fixed github-env-injection in ct.sh by sanitizing cache_dir and venv_dir with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_PATH and $GITHUB_ENV, preventing newline injection attacks.

