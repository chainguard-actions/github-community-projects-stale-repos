<!-- markdownlint-disable -->

# Hardening Report: github-community-projects--stale-repos/v9.0.14

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **github-community-projects--stale-repos/v9.0.14** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image reference with a mutable tag instead of a SHA digest. The image `docker://ghcr.io/github-community-projects/stale_repos:v9` uses the tag `:v9`, which can be silently updated to point to a different (potentially malicious) image. It should be pinned to a specific SHA digest, e.g. `docker://ghcr.io/github-community-projects/stale_repos@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:10`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image in action.yml from the mutable tag `ghcr.io/github-community-projects/stale_repos:v9` to the immutable digest `ghcr.io/github-community-projects/stale_repos@sha256:f2ca313765e6ac6d54c2d17cb669c0b40a9bcbdf385c8ad91b0eb773bb2da0fd # v9`. The comment is placed outside the YAML string quotes to preserve readability while ensuring the image reference is immutable.

