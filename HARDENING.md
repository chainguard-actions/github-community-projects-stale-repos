<!-- markdownlint-disable -->

# Hardening Report: github-community-projects--stale-repos/v9.0.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **github-community-projects--stale-repos/v9.0.11** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml Docker image reference uses a mutable tag (`v9`) instead of a SHA digest. This means the image can be silently replaced with a different version, enabling supply-chain attacks. The reference `docker://ghcr.io/github-community-projects/stale_repos:v9` should be replaced with a SHA-pinned form such as `docker://ghcr.io/github-community-projects/stale_repos@sha256:<64-hex-char-digest>`.

Note: The use-action.yml workflow correctly pins the same image with a SHA digest (`docker://ghcr.io/github-community-projects/stale_repos:v9@sha256:374277cb9863651aeebd9a45f09083aaec013e72982d217bf4b058baa60bbc4a`), but the action.yml itself does not.

Locations:

- `action.yml:10`

### script-injection (severity: high)

Rule (a): `${{ matrix.python-version }}` is interpolated directly inside `run:` shell command strings. Although `matrix` values are typically developer-controlled, any `${{ ... }}` expression inside a `run:` block is subject to YAML template substitution before the shell sees it, making it a script-injection risk. The value should be passed via an `env:` variable and then referenced as a quoted shell variable.

Offending lines:
- `run: uv python install ${{ matrix.python-version }}`
- `run: uv sync --frozen --python ${{ matrix.python-version }}`

Locations:

- `.github/workflows/python-package.yml:40`
- `.github/workflows/python-package.yml:42`

### github-env-injection (severity: high)

The `version` step in major-version-updater.yml writes values derived from the untrusted input `TAG_NAME` (set from `${{ github.event.inputs.TAG_NAME || github.ref }}`) to `$GITHUB_OUTPUT` without sanitization. An attacker-controlled `TAG_NAME` containing newlines could inject arbitrary key-value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent step outputs.

The required sanitization (`printf '%s' "$TAG_NAME" | tr -d '\n\r'`) is absent before the write:
```
{ echo "tag=${tag}"; echo "version=${version}"; echo "major=${major}"; } >> "$GITHUB_OUTPUT"
```

Locations:

- `.github/workflows/major-version-updater.yml:38`

### script-injection (severity: high)

Rule (b): The `force update major tag` step uses unquoted shell variable expansions `${STEPS_VERSION_OUTPUTS_MAJOR}` and `${STEPS_VERSION_OUTPUTS_TAG}` in shell commands. These variables hold values from `steps.version.outputs.major` and `steps.version.outputs.tag`, which are themselves derived from the untrusted `TAG_NAME` input (`github.event.inputs.TAG_NAME || github.ref`). Unquoted expansions allow shell metacharacters in the values to be interpreted by the shell.

Offending lines:
- `git tag -f v${STEPS_VERSION_OUTPUTS_MAJOR} ${STEPS_VERSION_OUTPUTS_TAG}`
- `git push -f origin v${STEPS_VERSION_OUTPUTS_MAJOR}`

These should be double-quoted: `"${STEPS_VERSION_OUTPUTS_MAJOR}"` and `"${STEPS_VERSION_OUTPUTS_TAG}"`.

Locations:

- `.github/workflows/major-version-updater.yml:41`
- `.github/workflows/major-version-updater.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed 4 findings across 3 files:
1. action.yml: Pinned Docker image `ghcr.io/github-community-projects/stale_repos:v9` with SHA256 digest `sha256:652c0ff12aead3ffee90bf233e7272d48eae7f9f8c45587d1ec12d41f0ba71d3`, preserving the `docker://` scheme and `:v9` tag.
2. .github/workflows/python-package.yml: Moved both `${{ matrix.python-version }}` expressions into `env:` blocks as `PYTHON_VERSION` and referenced them as `"$PYTHON_VERSION"` in the shell commands.
3. .github/workflows/major-version-updater.yml: (a) Sanitized all values derived from `TAG_NAME` using `printf '%s' ... | tr -d '\n\r'` before writing to `$GITHUB_OUTPUT` to prevent newline injection. (b) Double-quoted `${STEPS_VERSION_OUTPUTS_MAJOR}` and `${STEPS_VERSION_OUTPUTS_TAG}` in the `force update major tag` step to prevent shell metacharacter interpretation.

