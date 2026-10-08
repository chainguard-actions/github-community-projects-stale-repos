<!-- markdownlint-disable -->

# Hardening Report: github-community-projects--stale-repos/v9.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **github-community-projects--stale-repos/v9.1.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml Docker image reference uses a mutable version tag (`v9`) instead of an immutable SHA digest. This means the action could silently pull a different (potentially malicious) image if the tag is moved. The failing reference is: `image: "docker://ghcr.io/github-community-projects/stale_repos:v9"`. It should be pinned to a full SHA256 digest, e.g. `image: "ghcr.io/github-community-projects/stale_repos@sha256:<64-hex-char-digest> # v9"`.

Locations:

- `action.yml:10`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yml from the mutable tag `v9` to an immutable SHA256 digest. Updated `docker://ghcr.io/github-community-projects/stale_repos:v9` to `docker://ghcr.io/github-community-projects/stale_repos:v9@sha256:b259f6c3d90e6e29c7d9a0644f4bf7b4004f51e08184cb3f834e98dc56de5745`, preserving the `docker://` scheme and the tag inline as required.

