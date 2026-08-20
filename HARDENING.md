<!-- markdownlint-disable -->

# Hardening Report: catchpoint--workflow-telemetry-action/v1.8.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **catchpoint--workflow-telemetry-action/v1.8.6** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files reference GitHub Actions using mutable version tags (@v3, @v6) instead of pinned full 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit. Affected references: build.yml — actions/checkout@v3 (line 13), actions/setup-node@v3 (line 15); release.yml — actions/checkout@v3 (line 17), actions/setup-node@v3 (line 26), actions/checkout@v3 (line 37), actions/github-script@v6 (line 52); tag.yml — actions/checkout@v3 (line 13).

Locations:

- `.github/workflows/build.yml:13`
- `.github/workflows/build.yml:15`
- `.github/workflows/release.yml:17`
- `.github/workflows/release.yml:26`
- `.github/workflows/release.yml:37`
- `.github/workflows/release.yml:52`
- `.github/workflows/tag.yml:13`

### missing-permissions (severity: medium)

None of the three workflow files declare a top-level or job-level `permissions:` block. Without explicit permissions, workflows run with the default (often write-all) token permissions, violating the principle of least privilege. All three files — build.yml, release.yml, and tag.yml — are affected.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/tag.yml:1`

### script-injection (severity: high)

Sub-rule (a): A `run:` block in release.yml directly interpolates the user-controlled expression `${{ github.event.inputs.version_scale }}` into a shell command: `run: npm version ${{ github.event.inputs.version_scale }}`. Before the shell executes the command, GitHub Actions substitutes the raw expression value into the command string, allowing an attacker to inject arbitrary shell metacharacters via the `version_scale` workflow_dispatch input. The value should be passed via an `env:` variable and double-quoted in the shell instead.

Locations:

- `.github/workflows/release.yml:34`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across build.yml, release.yml, and tag.yml:

1. unpinned-uses: Pinned all 7 action references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3
   - actions/setup-node@v3 → @3235b876344d2a9aa001b8d1453c930bba69e610 # v3
   - actions/github-script@v6 → @d7906e4ad0b1822421a7e6a35d5ca353c962f410 # v6

2. missing-permissions: Added top-level permissions blocks to all three files:
   - build.yml: contents: read
   - release.yml: contents: read (custom token handles write ops)
   - tag.yml: contents: write (needs to push tags)

3. script-injection: Moved `${{ github.event.inputs.version_scale }}` out of the run: shell string into an env: block as VERSION_SCALE, then referenced it as "$VERSION_SCALE" in the npm version command.

