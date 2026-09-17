<!-- markdownlint-disable -->

# Hardening Report: github-community-projects--stale-repos/v9.0.18

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **github-community-projects--stale-repos/v9.0.18** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image referenced by a mutable tag (`v9`) instead of an immutable SHA digest. This means the action could silently pull a different (potentially malicious) image if the tag is moved. The failing reference is: `image: "docker://ghcr.io/github-community-projects/stale_repos:v9"`. It should be replaced with a SHA-pinned reference such as `image: "ghcr.io/github-community-projects/stale_repos@sha256:<64-hex-digest>"`.

Locations:

- `action.yml:10`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yml from the mutable tag `docker://ghcr.io/github-community-projects/stale_repos:v9` to the immutable digest `docker://ghcr.io/github-community-projects/stale_repos:v9@sha256:d339771f144c452b12485738a3e406a4b6e1ee00567554c295ad133214ddcc23`. The `docker://` scheme and `:v9` tag are preserved inline as required.

