<!-- markdownlint-disable -->

# Hardening Report: git-for-windows--setup-git-for-windows-sdk/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **git-for-windows--setup-git-for-windows-sdk/v2.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files use mutable tag-based refs instead of pinned 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced action tags are moved or compromised. Unpinned refs found:
- actions/checkout@v6
- actions/setup-node@v6
- actions/upload-artifact@v7
- actions/github-script@v9

Locations:

- `.github/workflows/matrix.yml:19`
- `.github/workflows/matrix.yml:31`
- `.github/workflows/npm-run-package.yml:20`
- `.github/workflows/npm-run-package.yml:23`
- `.github/workflows/release-tag.yml:13`
- `.github/workflows/release-tag.yml:17`
- `.github/workflows/release-tag.yml:82`
- `.github/workflows/test.yml:14`
- `.github/workflows/test.yml:15`
- `.github/workflows/test.yml:33`
- `.github/workflows/test.yml:55`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings.

1. matrix.yml: `${{ matrix.architecture }}` is embedded directly in a shell command: `./build-extra/installer/release.sh --output=$PWD/installer-${{ matrix.architecture }} 0-test`. The matrix context is workflow-controllable and flows through YAML template substitution before the shell sees it.

2. npm-run-package.yml: `${{github.actor}}` is embedded directly in shell commands: `git config user.name "${{github.actor}}"` and `git config user.email "${{github.actor}}@users.noreply.github.com"`. The github.actor value is attacker-controllable (e.g. via a fork PR) and could contain shell metacharacters.

Locations:

- `.github/workflows/matrix.yml:30`
- `.github/workflows/npm-run-package.yml:40`
- `.github/workflows/npm-run-package.yml:41`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the repository's default token permissions (which may be broad). Each workflow should declare minimal required permissions.

- matrix.yml: no permissions declared
- npm-run-package.yml: no permissions declared
- test.yml: no permissions declared

(release-tag.yml correctly declares `permissions: contents: write` at the top level.)

Locations:

- `.github/workflows/matrix.yml:1`
- `.github/workflows/npm-run-package.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across four workflow files:

1. unpinned-uses: Pinned all four actions to their full 40-character SHA commits with tag comments preserved:
   - actions/checkout@v6 → df4cb1c069e1874edd31b4311f1884172cec0e10
   - actions/setup-node@v6 → 249970729cb0ef3589644e2896645e5dc5ba9c38
   - actions/upload-artifact@v7 → 043fb46d1a93c77aae656e7c1c64a875d1fc6a0a
   - actions/github-script@v9 → 3a2844b7e9c422d3c10d287c895573f7108da1b3

2. script-injection: Moved GitHub Actions expressions out of run: shell strings into env: blocks:
   - matrix.yml: ${{ matrix.architecture }} → env var ARCHITECTURE
   - npm-run-package.yml: ${{github.actor}} → env var ACTOR (used in both git config commands)

3. missing-permissions: Added `permissions: contents: read` at the top level of matrix.yml, npm-run-package.yml, and test.yml. release-tag.yml already had `permissions: contents: write` declared.

