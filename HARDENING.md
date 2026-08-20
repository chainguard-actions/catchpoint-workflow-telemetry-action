<!-- markdownlint-disable -->

# Hardening Report: catchpoint--workflow-telemetry-action/v1.8.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **catchpoint--workflow-telemetry-action/v1.8.7** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is directly interpolated inside a `run:` shell command. In release.yml, the line `run: npm version ${{ github.event.inputs.version_scale }}` injects a user-controlled `workflow_dispatch` input directly into the shell command without going through an env var. An attacker with dispatch access could supply a value containing shell metacharacters to achieve command injection. The value should be passed via an `env:` variable and that variable should be double-quoted in the shell command.

Locations:

- `.github/workflows/release.yml:34`

### unpinned-uses (severity: high)

All `uses:` references across the three workflow files use mutable tag-based refs (@v3, @v6) instead of immutable 40-character commit SHA digests. If the referenced action's tag is moved (e.g. by a supply-chain compromise), the workflow will silently execute different code. Affected references: build.yml — `actions/checkout@v3` (line 13), `actions/setup-node@v3` (line 15); release.yml — `actions/checkout@v3` (line 22), `actions/setup-node@v3` (line 29), `actions/checkout@v3` (line 44), `actions/github-script@v6` (line 58); tag.yml — `actions/checkout@v3` (line 14). Each should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`.

Locations:

- `.github/workflows/build.yml:13`
- `.github/workflows/build.yml:15`
- `.github/workflows/release.yml:22`
- `.github/workflows/release.yml:29`
- `.github/workflows/release.yml:44`
- `.github/workflows/release.yml:58`
- `.github/workflows/tag.yml:14`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` block, and none of the individual jobs define job-level `permissions:` blocks. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (which may be `write-all` for older repositories). Each workflow should declare the minimal permissions required, e.g. `permissions: contents: read` at the top level, with job-level overrides where write access is needed.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/tag.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across build.yml, release.yml, and tag.yml:

1. script-injection (release.yml line 34): Moved `${{ github.event.inputs.version_scale }}` out of the `run:` shell command into an `env:` block as `VERSION_SCALE`, then referenced it as `"$VERSION_SCALE"` in the shell.

2. unpinned-uses: Pinned all 7 action references to full commit SHAs — actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26, actions/setup-node@v3 → @3235b876344d2a9aa001b8d1453c930bba69e610, actions/github-script@v6 → @d7906e4ad0b1822421a7e6a35d5ca353c962f410. Original tags preserved as inline comments.

3. missing-permissions: Added top-level `permissions: contents: read` to all three workflow files. Jobs requiring write access (update-version, create-release in release.yml; build in tag.yml) have job-level `permissions: contents: write` overrides following least-privilege principle.

