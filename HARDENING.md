<!-- markdownlint-disable -->

# Hardening Report: github-community-projects--stale-repos/v9.0.17

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **github-community-projects--stale-repos/v9.0.17** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image referenced by a mutable tag (:v9) rather than an immutable SHA digest. This means the image could be silently replaced with a different version, enabling supply-chain attacks. The image reference `docker://ghcr.io/github-community-projects/stale_repos:v9` should be replaced with a SHA-pinned form such as `docker://ghcr.io/github-community-projects/stale_repos@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:10`

### script-injection (severity: high)

Sub-rule (a): The `${{ matrix.python-version }}` expression is interpolated directly inside `run:` shell command strings in two steps. Although `matrix.*` values are typically developer-controlled, any `${{ ... }}` expression inside a `run:` block undergoes YAML template substitution before the shell sees it, making it a script-injection risk. Offending lines: `run: uv python install ${{ matrix.python-version }}` (line 40) and `run: uv sync --frozen --python ${{ matrix.python-version }}` (line 42). The fix is to move the value into an `env:` variable and reference it as a quoted shell variable, e.g. `"$PYTHON_VERSION"`.

Locations:

- `.github/workflows/python-package.yml:40`
- `.github/workflows/python-package.yml:42`

### github-env-injection (severity: high)

In the `version` step, the env var `TAG_NAME` (set at the workflow level from `${{ github.event.inputs.TAG_NAME || github.ref }}`, an attacker-controllable `workflow_dispatch` input) is read inside a `run:` block and its derived values (`tag`, `version`, `major`) are written directly to `$GITHUB_OUTPUT` without sanitization. A malicious value containing newlines could inject arbitrary key-value pairs into the output context. The fix is to sanitize each value with `printf '%s' "$VAR" | tr -d '\n\r'` before writing to `$GITHUB_OUTPUT`.

Locations:

- `.github/workflows/major-version-updater.yml:38`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

1. action.yml: Pinned the Docker image 'docker://ghcr.io/github-community-projects/stale_repos:v9' to its immutable SHA256 digest 'sha256:3a0bb3fbeedce10eedae5eadbeb5cb0b1c159a30b1c435cef6e3491c09395d3c', preserving the 'docker://' scheme and ':v9' tag inline.
2. .github/workflows/python-package.yml: Moved '${{ matrix.python-version }}' expressions out of both 'run:' blocks into 'env:' blocks as PYTHON_VERSION, then referenced as quoted shell variable "$PYTHON_VERSION" in the run commands.
3. .github/workflows/major-version-updater.yml: Added sanitization of the tag, version, and major derived values using 'printf '%s' | tr -d '\n\r'' before writing them to $GITHUB_OUTPUT, preventing newline injection attacks from the attacker-controllable TAG_NAME workflow_dispatch input.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansions in the 'force update major tag' step of .github/workflows/major-version-updater.yml. Changed `git tag -f v${STEPS_VERSION_OUTPUTS_MAJOR} ${STEPS_VERSION_OUTPUTS_TAG}` and `git push -f origin v${STEPS_VERSION_OUTPUTS_MAJOR}` to use double-quoted expansions: `git tag -f "v${STEPS_VERSION_OUTPUTS_MAJOR}" "${STEPS_VERSION_OUTPUTS_TAG}"` and `git push -f origin "v${STEPS_VERSION_OUTPUTS_MAJOR}"`. This prevents shell metacharacters in the env var values from being interpreted as shell commands.

