<!-- markdownlint-disable -->

# Hardening Report: git-for-windows--setup-git-for-windows-sdk/v1.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **git-for-windows--setup-git-for-windows-sdk/v1.11.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags instead of pinned full-length SHA commits, making them vulnerable to supply-chain attacks.

- matrix.yml: `actions/checkout@v4` (line 18), `actions/upload-artifact@v4` (line 32)
- npm-run-package.yml: `actions/checkout@v4` (line 19)
- release-tag.yml: `actions/checkout@v4` (line 14), `actions/github-script@v7` (line 17), `actions/github-script@v7` (line 57)
- test.yml: `actions/checkout@v4` (line 13), `actions/checkout@v4` (line 28), `actions/upload-artifact@v4` (line 43)

All of these should be replaced with their full 40-character hex commit SHA.

Locations:

- `.github/workflows/matrix.yml:18`
- `.github/workflows/matrix.yml:32`
- `.github/workflows/npm-run-package.yml:19`
- `.github/workflows/release-tag.yml:14`
- `.github/workflows/release-tag.yml:17`
- `.github/workflows/release-tag.yml:57`
- `.github/workflows/test.yml:13`
- `.github/workflows/test.yml:28`
- `.github/workflows/test.yml:43`

### script-injection (severity: high)

GitHub Actions expressions are interpolated directly inside `run:` shell command strings, allowing injection of arbitrary shell commands.

(a) matrix.yml, line 30: `${{ matrix.architecture }}` is interpolated directly into a shell command: `./build-extra/installer/release.sh --output=$PWD/installer-${{ matrix.architecture }} 0-test`. The `matrix.*` context is workflow-controllable and flows through YAML template substitution before the shell sees it.

(a) npm-run-package.yml, lines 34–35: `${{github.actor}}` is interpolated directly into shell commands: `git config user.name "${{github.actor}}"` and `git config user.email "${{github.actor}}@users.noreply.github.com"`. An attacker-controlled username containing shell metacharacters could inject commands.

Locations:

- `.github/workflows/matrix.yml:30`
- `.github/workflows/npm-run-package.yml:34`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, workflows run with the default repository permissions, which may be overly broad (e.g. `write` for contents on some repository configurations).

- matrix.yml: no permissions defined at top-level or job level
- npm-run-package.yml: no permissions defined at top-level or job level
- test.yml: no permissions defined at top-level or job level

Locations:

- `.github/workflows/matrix.yml:1`
- `.github/workflows/npm-run-package.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three finding types across four workflow files:

1. **unpinned-uses**: Pinned all action references to full SHAs:
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4
   - actions/upload-artifact@v4 → @ea165f8d65b6e75b540449e92b4886f43607fa02 # v4
   - actions/github-script@v7 → @f28e40c7f34bde8b3046d885e986cb6290c5673b # v7

2. **script-injection**: Moved GitHub context expressions out of run: shell strings into env: blocks:
   - matrix.yml: `${{ matrix.architecture }}` → env var `ARCHITECTURE`
   - npm-run-package.yml: `${{github.actor}}` → env var `ACTOR`

3. **missing-permissions**: Added `permissions: contents: read` to matrix.yml, npm-run-package.yml, and test.yml. release-tag.yml already had `permissions: contents: write` which was preserved as it needs write access to create releases and push branches.

