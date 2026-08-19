<!-- markdownlint-disable -->

# Hardening Report: github-community-projects--stale-repos/v9.0.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **github-community-projects--stale-repos/v9.0.13** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses a Docker image referenced by a mutable tag (`v9`) instead of an immutable SHA digest. This means the action could silently pull a different (potentially malicious) image on future runs. The `image:` field should use a SHA digest, e.g. `docker://ghcr.io/github-community-projects/stale_repos@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:10`

### script-injection (severity: high)

Sub-rule (a): `${{ matrix.python-version }}` is interpolated directly inside `run:` shell command strings. Although the matrix values are hardcoded in this workflow, any `${{ ... }}` expression in a `run:` block is subject to YAML template substitution before the shell sees it, making it a script-injection risk. The value should be passed via an `env:` variable and then double-quoted in the shell. Offending lines: `run: uv python install ${{ matrix.python-version }}` and `run: uv sync --frozen --python ${{ matrix.python-version }}`.

Locations:

- `.github/workflows/python-package.yml:37`
- `.github/workflows/python-package.yml:39`

### github-env-injection (severity: high)

In the `version` step, the env var `TAG_NAME` is sourced from `${{ github.event.inputs.TAG_NAME || github.ref }}` (user-controlled via `workflow_dispatch` input). The step then writes derived values (`tag=`, `version=`, `major=`) to `$GITHUB_OUTPUT` using `echo` without first sanitizing with `printf '%s' ... | tr -d '\n\r'`. A newline embedded in the input could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`.

Locations:

- `.github/workflows/major-version-updater.yml:33`

### script-injection (severity: high)

Sub-rule (b): In the `force update major tag` step, the shell variables `${STEPS_VERSION_OUTPUTS_MAJOR}` and `${STEPS_VERSION_OUTPUTS_TAG}` (sourced from `steps.version.outputs.major` and `steps.version.outputs.tag`, which are themselves derived from the user-controlled `TAG_NAME` input) are expanded unquoted in shell commands: `git tag -f v${STEPS_VERSION_OUTPUTS_MAJOR} ${STEPS_VERSION_OUTPUTS_TAG}` and `git push -f origin v${STEPS_VERSION_OUTPUTS_MAJOR}`. Unquoted expansion allows shell metacharacters in the values to be interpreted by the shell.

Locations:

- `.github/workflows/major-version-updater.yml:37`
- `.github/workflows/major-version-updater.yml:38`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

1. action.yml: Pinned docker://ghcr.io/github-community-projects/stale_repos:v9 with immutable SHA256 digest (sha256:652c0ff12aead3ffee90bf233e7272d48eae7f9f8c45587d1ec12d41f0ba71d3), preserving the docker:// scheme and :v9 tag inline.
2. .github/workflows/python-package.yml: Moved both ${{ matrix.python-version }} expressions into env: blocks (PYTHON_VERSION) and referenced them as double-quoted "$PYTHON_VERSION" in the run: shell commands.
3. .github/workflows/major-version-updater.yml (github-env-injection): Added printf '%s' ... | tr -d '\n\r' sanitization for tag, version, and major values before writing to $GITHUB_OUTPUT.
4. .github/workflows/major-version-updater.yml (script-injection): Added double-quotes around ${STEPS_VERSION_OUTPUTS_MAJOR} and ${STEPS_VERSION_OUTPUTS_TAG} in the git tag and git push commands to prevent shell metacharacter interpretation.

