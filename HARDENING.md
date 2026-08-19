<!-- markdownlint-disable -->

# Hardening Report: github-community-projects--stale-repos/v9.0.14

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **github-community-projects--stale-repos/v9.0.14** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses a Docker image referenced by a mutable tag (:v9) instead of an immutable SHA digest. This means the action could silently pull a different (potentially malicious) image on future runs. The image reference `docker://ghcr.io/github-community-projects/stale_repos:v9` must be replaced with a SHA-digest reference such as `docker://ghcr.io/github-community-projects/stale_repos@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:10`

### script-injection (severity: high)

Sub-rule (a): `${{ matrix.python-version }}` is interpolated directly inside `run:` shell command strings in python-package.yml. Although `matrix.*` values are typically developer-controlled, any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted into the shell command string before the shell parses it, allowing metacharacters to be interpreted. Offending lines:
  - `run: uv python install ${{ matrix.python-version }}`
  - `run: uv sync --frozen --python ${{ matrix.python-version }}`
Fix: move the value into an `env:` variable and reference it as a quoted shell variable, e.g. `"$PYTHON_VERSION"`.

Locations:

- `.github/workflows/python-package.yml:33`
- `.github/workflows/python-package.yml:35`

### github-env-injection (severity: high)

In major-version-updater.yml, the workflow-level env var `TAG_NAME` is set to `${{ github.event.inputs.TAG_NAME || github.ref }}`. On `workflow_dispatch` events, `github.event.inputs.TAG_NAME` is attacker-controllable. The `version` step then derives `tag`, `version`, and `major` from `$TAG_NAME` and writes them to `$GITHUB_OUTPUT` via `{ echo "tag=${tag}"; echo "version=${version}"; echo "major=${major}"; } >> "$GITHUB_OUTPUT"` without first sanitizing the values with `printf '%s' ... | tr -d '\n\r'`. A newline embedded in the input could inject arbitrary key=value pairs into GITHUB_OUTPUT.

Locations:

- `.github/workflows/major-version-updater.yml:35`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

1. action.yml: Pinned Docker image `ghcr.io/github-community-projects/stale_repos:v9` to immutable digest `sha256:652c0ff12aead3ffee90bf233e7272d48eae7f9f8c45587d1ec12d41f0ba71d3`, preserving the `docker://` scheme and `:v9` tag inline.
2. python-package.yml: Moved `${{ matrix.python-version }}` out of both `run:` blocks into `env: PYTHON_VERSION:` variables, referencing them as `"$PYTHON_VERSION"` in the shell commands.
3. major-version-updater.yml: Added `printf '%s' ... | tr -d '\n\r'` sanitization for `tag`, `version`, and `major` values before writing them to `$GITHUB_OUTPUT`, preventing newline injection from the attacker-controllable `TAG_NAME` workflow_dispatch input.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted shell variable expansions in .github/workflows/major-version-updater.yml:
1. In the 'version' step (line 35): Quoted the TAG_NAME parameter expansion: `tag="${TAG_NAME/refs\/tags\//}"` and also quoted the derived `version` and `major` variables.
2. In the 'force update major tag' step (lines 44-45): Double-quoted both STEPS_VERSION_OUTPUTS_MAJOR and STEPS_VERSION_OUTPUTS_TAG in the git tag and git push commands to prevent word-splitting and glob expansion of step output values.

