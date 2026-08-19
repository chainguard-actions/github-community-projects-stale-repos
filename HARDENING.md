<!-- markdownlint-disable -->

# Hardening Report: github-community-projects--stale-repos/v9.0.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **github-community-projects--stale-repos/v9.0.12** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image referenced by a mutable tag (`v9`) rather than an immutable SHA digest. This means the action could silently pull a different (potentially malicious) image on future runs. The failing reference is: `image: "docker://ghcr.io/github-community-projects/stale_repos:v9"`. It should be pinned to a SHA digest, e.g. `image: "ghcr.io/github-community-projects/stale_repos@sha256:<64-hex-char-digest>"`

Locations:

- `action.yml:10`

### script-injection (severity: high)

Rule (a): Two `run:` steps in the `build` job directly interpolate `${{ matrix.python-version }}` inside shell command strings. Although `matrix` values are typically developer-controlled, any `${{ ... }}` expression directly inside a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it, bypassing shell quoting. Offending lines:
- `run: uv python install ${{ matrix.python-version }}` (line 36)
- `run: uv sync --frozen --python ${{ matrix.python-version }}` (line 38)
Fix: move the value into an `env:` variable and reference it as a quoted shell variable, e.g. `"$PYTHON_VERSION"`.

Locations:

- `.github/workflows/python-package.yml:36`
- `.github/workflows/python-package.yml:38`

### github-env-injection (severity: high)

The `version` step in major-version-updater.yml reads the `$TAG_NAME` environment variable (set at the workflow level from `${{ github.event.inputs.TAG_NAME || github.ref }}`, an attacker-controllable value via `workflow_dispatch`) and writes derived values (`${tag}`, `${version}`, `${major}`) to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A crafted tag name containing newlines could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, poisoning subsequent steps. The offending write is: `{ echo "tag=${tag}"; echo "version=${version}"; echo "major=${major}"; } >> "$GITHUB_OUTPUT"`

Locations:

- `.github/workflows/major-version-updater.yml:32`

### script-injection (severity: high)

Rule (b): The `force update major tag` step uses unquoted shell variable expansions `${STEPS_VERSION_OUTPUTS_MAJOR}` and `${STEPS_VERSION_OUTPUTS_TAG}` in shell commands. These env vars hold values from `steps.version.outputs.major` and `steps.version.outputs.tag`, which are derived from the workflow-controllable `$TAG_NAME` input. Unquoted expansions allow shell metacharacters (spaces, semicolons, etc.) in the values to be interpreted by the shell. Offending lines:
- `git tag -f v${STEPS_VERSION_OUTPUTS_MAJOR} ${STEPS_VERSION_OUTPUTS_TAG}`
- `git push -f origin v${STEPS_VERSION_OUTPUTS_MAJOR}`
Fix: quote all expansions: `"${STEPS_VERSION_OUTPUTS_MAJOR}"` and `"${STEPS_VERSION_OUTPUTS_TAG}"`.

Locations:

- `.github/workflows/major-version-updater.yml:36`
- `.github/workflows/major-version-updater.yml:37`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed 4 findings across 3 files:
1. action.yml: Pinned Docker image from mutable tag 'v9' to immutable digest 'docker://ghcr.io/github-community-projects/stale_repos:v9@sha256:652c0ff12aead3ffee90bf233e7272d48eae7f9f8c45587d1ec12d41f0ba71d3'.
2. .github/workflows/python-package.yml: Moved both ${{ matrix.python-version }} interpolations out of run: shell strings into env: blocks (PYTHON_VERSION), referencing them as quoted shell variables "$PYTHON_VERSION".
3. .github/workflows/major-version-updater.yml: (a) Sanitized tag/version/major values with printf+tr before writing to $GITHUB_OUTPUT to prevent newline injection. (b) Quoted all shell variable expansions in git tag and git push commands to prevent shell metacharacter interpretation.

