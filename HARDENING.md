<!-- markdownlint-disable -->

# Hardening Report: github-community-projects--stale-repos/v9.0.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **github-community-projects--stale-repos/v9.0.10** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The Docker action image reference uses a mutable tag (`v9`) instead of an immutable SHA digest. This means the action could silently pull a different (potentially malicious) image if the tag is moved. The reference `docker://ghcr.io/github-community-projects/stale_repos:v9` should be replaced with a pinned digest such as `docker://ghcr.io/github-community-projects/stale_repos@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:10`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag `ghcr.io/github-community-projects/stale_repos:v9` with the immutable SHA256 digest `ghcr.io/github-community-projects/stale_repos@sha256:f2ca313765e6ac6d54c2d17cb669c0b40a9bcbdf385c8ad91b0eb773bb2da0fd` in action.yml line 10. The original tag is preserved as a comment (`# v9`) for readability.

