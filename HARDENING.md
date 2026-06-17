<!-- markdownlint-disable -->

# Hardening Report: github-community-projects--stale-repos/v9.0.15

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **github-community-projects--stale-repos/v9.0.15** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image reference with a mutable tag (`v9`) instead of an immutable SHA digest. This means the action could silently pull a different (potentially malicious) image if the tag is moved. The failing reference is: `image: "docker://ghcr.io/github-community-projects/stale_repos:v9"`. It should be pinned to a full SHA256 digest, e.g. `image: "docker://ghcr.io/github-community-projects/stale_repos@sha256:<64-hex-char-digest>"`

Locations:

- `action.yml:10`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yml from the mutable tag `ghcr.io/github-community-projects/stale_repos:v9` to the immutable digest `ghcr.io/github-community-projects/stale_repos@sha256:c8e36cc33e5fce9bb7c9b7eaf8b8273868c27b89a248e025037df59d28179d63 # v9`. The original tag is preserved as a comment for readability.

