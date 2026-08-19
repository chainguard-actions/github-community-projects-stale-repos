<!-- markdownlint-disable -->

# Hardening Report: github-community-projects--stale-repos/v9.0.15

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **github-community-projects--stale-repos/v9.0.15** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The Docker image reference in action.yml uses a mutable version tag (':v9') instead of an immutable SHA digest. This means the action could silently pull a different (potentially malicious) image if the tag is moved. The reference 'docker://ghcr.io/github-community-projects/stale_repos:v9' should be replaced with a pinned digest such as 'ghcr.io/github-community-projects/stale_repos@sha256:<64-hex-char-digest>'.

Locations:

- `action.yml:10`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yml from the mutable tag 'docker://ghcr.io/github-community-projects/stale_repos:v9' to the immutable digest 'docker://ghcr.io/github-community-projects/stale_repos:v9@sha256:652c0ff12aead3ffee90bf233e7272d48eae7f9f8c45587d1ec12d41f0ba71d3'. The docker:// scheme and :v9 tag are preserved inline alongside the digest.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three findings across two workflow files:

1. **python-package.yml (script-injection)**: Moved `${{ matrix.python-version }}` out of both `run:` commands into `env:` blocks as `PYTHON_VERSION`, then referenced as `"$PYTHON_VERSION"` in the shell commands (`uv python install` and `uv sync --frozen --python`).

2. **major-version-updater.yml (script-injection)**: Added double-quotes around `${STEPS_VERSION_OUTPUTS_MAJOR}` and `${STEPS_VERSION_OUTPUTS_TAG}` in the `git tag` and `git push` commands to prevent shell metacharacter interpretation.

3. **major-version-updater.yml (github-env-injection)**: Added sanitization of `tag`, `version`, and `major` variables using `printf '%s' "$var" | tr -d '\n\r'` before writing them to `$GITHUB_OUTPUT`, preventing newline injection that could allow an attacker to inject additional output variables.

