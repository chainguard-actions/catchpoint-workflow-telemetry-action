<!-- markdownlint-disable -->

# Hardening Report: catchpoint--workflow-telemetry-action/v1.8.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **catchpoint--workflow-telemetry-action/v1.8.5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): A workflow_dispatch user-controlled input is interpolated directly inside a run: shell command. In release.yml, the step `run: npm version ${{ github.event.inputs.version_scale }}` embeds the raw expression into the shell command string. Although this input is constrained to a choice list (patch/minor/major), any ${{ }} expression in a run: block is a script-injection finding because YAML template substitution occurs before the shell ever sees the value.

Locations:

- `.github/workflows/release.yml:31`

### unpinned-uses (severity: high)

Multiple uses: references across all three workflow files use mutable tag refs instead of pinned 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if the referenced tags are moved or compromised. Failing references: build.yml — actions/checkout@v3 (line 13), actions/setup-node@v3 (line 15); release.yml — actions/checkout@v3 (line 19), actions/setup-node@v3 (line 27), actions/checkout@v3 (line 38), actions/github-script@v6 (line 56); tag.yml — actions/checkout@v3 (line 14).

Locations:

- `.github/workflows/build.yml:13`
- `.github/workflows/build.yml:15`
- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:27`
- `.github/workflows/release.yml:38`
- `.github/workflows/release.yml:56`
- `.github/workflows/tag.yml:14`

### missing-permissions (severity: medium)

None of the three workflow files declare a top-level permissions: key, and no job within them declares job-level permissions either. Without explicit permissions, workflows inherit the default repository token permissions (which may be write-all depending on repository settings), granting broader access than necessary.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/tag.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across build.yml, release.yml, and tag.yml:

1. script-injection (release.yml): Moved `${{ github.event.inputs.version_scale }}` into a step env var `VERSION_SCALE` and referenced it as `"$VERSION_SCALE"` in the run command.

2. unpinned-uses: Pinned all 7 mutable tag references to full 40-char SHAs — actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26, actions/setup-node@v3 → @3235b876344d2a9aa001b8d1453c930bba69e610, actions/github-script@v6 → @d7906e4ad0b1822421a7e6a35d5ca353c962f410. Original tags preserved as inline comments.

3. missing-permissions: Added `permissions: {}` top-level block to all three workflow files to explicitly deny all default token permissions.

