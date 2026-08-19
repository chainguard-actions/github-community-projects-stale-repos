<!-- markdownlint-disable -->

# Hardening Report: github-community-projects--stale-repos/v9.0.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **github-community-projects--stale-repos/v9.0.10** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses a Docker image referenced by a mutable tag (`v9`) rather than an immutable SHA digest. This means the action could silently pull a different (potentially malicious) image on future runs. The reference `docker://ghcr.io/github-community-projects/stale_repos:v9` should be replaced with a SHA-pinned form such as `ghcr.io/github-community-projects/stale_repos@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:10`

### script-injection (severity: high)

Sub-rule (a): `${{ matrix.python-version }}` is interpolated directly inside two `run:` shell command strings in python-package.yml. GitHub Actions performs YAML template substitution before the shell ever sees the value, so a crafted matrix value could inject arbitrary shell commands. Offending lines: `run: uv python install ${{ matrix.python-version }}` and `run: uv sync --frozen --python ${{ matrix.python-version }}`. Fix: set an env var (e.g. `PYTHON_VERSION: ${{ matrix.python-version }}`) and reference `"$PYTHON_VERSION"` (double-quoted) in the run block.

Locations:

- `.github/workflows/python-package.yml:35`
- `.github/workflows/python-package.yml:37`

### script-injection (severity: high)

Sub-rule (b): In major-version-updater.yml, the `force update major tag` step expands `${STEPS_VERSION_OUTPUTS_MAJOR}` and `${STEPS_VERSION_OUTPUTS_TAG}` without double-quoting inside the run: block: `git tag -f v${STEPS_VERSION_OUTPUTS_MAJOR} ${STEPS_VERSION_OUTPUTS_TAG}` and `git push -f origin v${STEPS_VERSION_OUTPUTS_MAJOR}`. These env vars hold values derived from `steps.version.outputs.*`, which are themselves derived from the user-controlled `github.event.inputs.TAG_NAME`. Unquoted expansions allow shell metacharacter injection. Fix: quote all expansions as `"${STEPS_VERSION_OUTPUTS_MAJOR}"` and `"${STEPS_VERSION_OUTPUTS_TAG}"`.

Locations:

- `.github/workflows/major-version-updater.yml:38`
- `.github/workflows/major-version-updater.yml:39`

### github-env-injection (severity: high)

In major-version-updater.yml, the `version` step reads the `TAG_NAME` env var (set at workflow level from `${{ github.event.inputs.TAG_NAME || github.ref }}` — a user-controlled `workflow_dispatch` input) and writes derived values directly to `$GITHUB_OUTPUT` via `echo "tag=${tag}"; echo "version=${version}"; echo "major=${major}"` without first sanitizing with `printf '%s' ... | tr -d '\n\r'`. A newline embedded in the input could inject additional key=value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by later steps.

Locations:

- `.github/workflows/major-version-updater.yml:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

1. action.yml: Pinned Docker image from `docker://ghcr.io/github-community-projects/stale_repos:v9` to `docker://ghcr.io/github-community-projects/stale_repos:v9@sha256:652c0ff12aead3ffee90bf233e7272d48eae7f9f8c45587d1ec12d41f0ba71d3`, preserving the docker:// scheme and tag inline.
2. python-package.yml: Moved both `${{ matrix.python-version }}` expressions from `run:` shell strings into `env:` blocks as `PYTHON_VERSION`, and referenced them as `"$PYTHON_VERSION"` (double-quoted) in the run scripts.
3. major-version-updater.yml (script-injection): Double-quoted all variable expansions in the `force update major tag` step: `"v${STEPS_VERSION_OUTPUTS_MAJOR}"` and `"${STEPS_VERSION_OUTPUTS_TAG}"`.
4. major-version-updater.yml (github-env-injection): Added `printf '%s' ... | tr -d '\n\r'` sanitization for tag, version, and major values before writing them to $GITHUB_OUTPUT, preventing newline injection attacks.

