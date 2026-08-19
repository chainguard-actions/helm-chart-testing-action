<!-- markdownlint-disable -->

# Hardening Report: helm--chart-testing-action/v2.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **helm--chart-testing-action/v2.8.0** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation of untrusted inputs inside a run: block. In action.yml, the composite action run: step passes `${{ inputs.version }}`, `${{ inputs.yamllint_version }}`, and `${{ inputs.yamale_version }}` directly as shell arguments. These values are controlled by the calling workflow and are interpolated by the Actions template engine before the shell ever sees them, allowing an attacker to inject arbitrary shell commands (e.g. a version value of `3.14.0; malicious-command`).

Locations:

- `action.yml:24`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation of `${{ github.event.repository.default_branch }}` inside run: blocks in the workflow. This GitHub context value flows through YAML template substitution before the shell parses it, enabling script injection. Affected steps: 'Run chart-testing (list-changed)', 'Run chart-testing (lint)', and 'Run chart-testing (install)'.

Locations:

- `.github/workflows/test-action.yml:72`
- `.github/workflows/test-action.yml:79`
- `.github/workflows/test-action.yml:86`

### github-env-injection (severity: high)

In ct.sh, the variables `cache_dir` and `venv_dir` are derived from the user-controlled `version` input (passed via `inputs.version` in action.yml). These values are written directly to `$GITHUB_PATH` and `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline embedded in the version string could inject arbitrary environment variables or PATH entries into subsequent workflow steps.

Locations:

- `ct.sh:89`
- `ct.sh:92`
- `ct.sh:95`
- `ct.sh:96`

### unpinned-uses (severity: high)

The workflow references `helm/kind-action@v1.13.0`, which is pinned to a mutable version tag rather than an immutable 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved. All other `uses:` references in the workflow and action.yml are correctly pinned to full SHAs.

Locations:

- `.github/workflows/test-action.yml:83`

### missing-permissions (severity: medium)

The workflow file `.github/workflows/test-action.yml` has no top-level `permissions:` key and none of its three jobs (`test_ct_action`, `test_ct_action_custom`, `test_ct_action_with_helm`) define a `permissions:` block. Without explicit permissions, the workflow inherits the repository default (typically `write` for all scopes on private repos), granting unnecessarily broad access to the GITHUB_TOKEN.

Locations:

- `.github/workflows/test-action.yml:1`

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

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 8 findings across 3 files:

1. action.yml (script-injection / static-inline-injection): Moved ${{ inputs.version }}, ${{ inputs.yamllint_version }}, and ${{ inputs.yamale_version }} from the run: block into an env: block as CT_VERSION, CT_YAMLLINT_VERSION, CT_YAMALE_VERSION. The run: block now references these as shell environment variables.

2. ct.sh (github-env-injection): Added sanitization for cache_dir and venv_dir before writing to $GITHUB_PATH and $GITHUB_ENV. Used `printf '%s' "${var}" | tr -d '\n\r'` to strip newlines, storing results in safe_cache_dir and safe_venv_dir, then used printf for all writes to GITHUB_PATH/GITHUB_ENV.

3. .github/workflows/test-action.yml (script-injection): Moved ${{ github.event.repository.default_branch }} to env: blocks (as DEFAULT_BRANCH) in all three affected steps: 'Run chart-testing (list-changed)', 'Run chart-testing (lint)', and 'Run chart-testing (install)'.

4. .github/workflows/test-action.yml (unpinned-uses): Pinned helm/kind-action@v1.13.0 to full commit SHA 92086f6be054225fa813e0a4b13787fc9088faab with the tag preserved as a comment.

5. .github/workflows/test-action.yml (missing-permissions): Added top-level `permissions: {}` to deny all permissions by default.

