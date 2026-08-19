<!-- markdownlint-disable -->

# Hardening Report: catchpoint--workflow-telemetry-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **catchpoint--workflow-telemetry-action/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of pinned 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks if the tagged version is updated or compromised.

build.yml: actions/checkout@v4 (line 12), actions/setup-node@v4 (line 14)
release.yml: actions/checkout@v4 (line 18), actions/setup-node@v4 (line 26), actions/github-script@v6 (line 54)
tag-v2.yml: actions/checkout@v4 (line 14)

Locations:

- `.github/workflows/build.yml:12`
- `.github/workflows/build.yml:14`
- `.github/workflows/release.yml:18`
- `.github/workflows/release.yml:26`
- `.github/workflows/release.yml:54`
- `.github/workflows/tag-v2.yml:14`

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is interpolated directly inside a `run:` shell command string. In release.yml line 30, `${{ github.event.inputs.version_scale }}` is embedded directly in the shell command `npm version ${{ github.event.inputs.version_scale }}`. Even though the input is declared as a `choice` type, the expression is substituted into the shell command string before the shell processes it, allowing potential injection if the value contains shell metacharacters. The value must be passed via an `env:` variable and double-quoted in the shell command instead.

Locations:

- `.github/workflows/release.yml:30`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and no individual job within any of these files defines a `permissions:` block. Without explicit permissions, workflows run with the default repository permissions (which may be read/write), violating the principle of least privilege. Each workflow should declare the minimal permissions required.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/tag-v2.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across build.yml, release.yml, and tag-v2.yml:

1. unpinned-uses: Pinned all 6 action references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020 # v4
   - actions/github-script@v6 → @d7906e4ad0b1822421a7e6a35d5ca353c962f410 # v6

2. script-injection: In release.yml, moved `${{ github.event.inputs.version_scale }}` out of the `run:` shell string into a step-level `env:` block as `VERSION_SCALE`, then referenced it as `"$VERSION_SCALE"` in the shell command.

3. missing-permissions: Added `permissions: {}` top-level block to all three workflow files to enforce least privilege.

