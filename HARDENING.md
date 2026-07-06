<!-- markdownlint-disable -->

# Hardening Report: github-community-projects--stale-repos--/v9.0.16

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **github-community-projects--stale-repos--/v9.0.16** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses a Docker image with a mutable tag instead of a SHA digest: `image: "docker://ghcr.io/github-community-projects/stale_repos:v9"`. This is vulnerable to supply-chain attacks as the tag can be moved to point to a different image.

Locations:

- `action.yml:10`

### script-injection (severity: high)

Sub-rule (a): ${{ matrix.python-version }} is interpolated directly inside run: shell commands. This allows any value in the matrix to be injected as shell code before the shell ever sees it. Offending lines: `run: uv python install ${{ matrix.python-version }}` and `run: uv sync --frozen --python ${{ matrix.python-version }}`

Locations:

- `.github/workflows/python-package.yml:34`
- `.github/workflows/python-package.yml:36`

### script-injection (severity: high)

Sub-rule (b): In the 'force update major tag' step, the env vars ${STEPS_VERSION_OUTPUTS_MAJOR} and ${STEPS_VERSION_OUTPUTS_TAG} (sourced from steps.version.outputs.*, which are derived from the user-controlled github.event.inputs.TAG_NAME) are used unquoted in git commands: `git tag -f v${STEPS_VERSION_OUTPUTS_MAJOR} ${STEPS_VERSION_OUTPUTS_TAG}` and `git push -f origin v${STEPS_VERSION_OUTPUTS_MAJOR}`. Unquoted shell variable expansion allows shell metacharacter injection.

Locations:

- `.github/workflows/major-version-updater.yml:40`

### github-env-injection (severity: high)

In the 'version' step of major-version-updater.yml, the env var TAG_NAME (set from ${{ github.event.inputs.TAG_NAME || github.ref }}, which is user-controlled via workflow_dispatch) is used to derive values written to $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). An attacker can inject newlines to poison GITHUB_OUTPUT entries.

Locations:

- `.github/workflows/major-version-updater.yml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed 4 findings across 3 files: (1) action.yml: Pinned Docker image ghcr.io/github-community-projects/stale_repos:v9 to SHA digest sha256:652c0ff12aead3ffee90bf233e7272d48eae7f9f8c45587d1ec12d41f0ba71d3 with v9 tag comment. (2) python-package.yml: Moved ${{ matrix.python-version }} into env: blocks as PYTHON_VERSION and referenced as "$PYTHON_VERSION" in both Install Python and Install dependencies steps. (3) major-version-updater.yml: Added printf/tr sanitization to strip newlines from tag/version/major values before writing to GITHUB_OUTPUT (github-env-injection fix); also quoted all env var expansions in git tag and git push commands to prevent shell metacharacter injection (script-injection fix).

### Iteration 2

**Fixes applied:** unpinned-uses

**Notes:**

Fixed the Docker image reference in hardened/action/.github/workflows/use-action.yml. The original reference `docker://ghcr.io/github-community-projects/stale_repos:v9@sha256:374277cb9863651aeebd9a45f09083aaec013e72982d217bf4b058baa60bbc4a` had an invalid 63-character hex digest and a mutable :v9 tag. Replaced with `docker://ghcr.io/github-community-projects/stale_repos@sha256:652c0ff12aead3ffee90bf233e7272d48eae7f9f8c45587d1ec12d41f0ba71d3 # v9` using the correct 64-character sha256 digest from the registry, with the mutable tag moved to a comment.

