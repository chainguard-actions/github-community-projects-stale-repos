<!-- markdownlint-disable -->

# Hardening Report: github-community-projects--stale-repos/v9.0.19

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **github-community-projects--stale-repos/v9.0.19** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image referenced by a mutable version tag (`v9`) rather than an immutable SHA digest. The image `docker://ghcr.io/github-community-projects/stale_repos:v9` can be silently replaced at any time, enabling a supply-chain attack. It should be pinned to a full SHA256 digest, e.g. `image: ghcr.io/github-community-projects/stale_repos@sha256:<64-hex-char-digest> # v9`.

Locations:

- `action.yml:10`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yml from `docker://ghcr.io/github-community-projects/stale_repos:v9` to `docker://ghcr.io/github-community-projects/stale_repos:v9@sha256:d1b04c72bf19a22a2dec012420b05bb4e666fdba88ad69d8853a255cdc5089ca`, preserving the docker:// scheme and the :v9 tag inline for readability.

