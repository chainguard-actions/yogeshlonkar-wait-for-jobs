<!-- markdownlint-disable -->

# Hardening Report: yogeshlonkar--wait-for-jobs/v1.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yogeshlonkar--wait-for-jobs/v1.0.3** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tag refs instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks if the referenced tag is moved or overwritten.

check-dist.yaml: actions/checkout@v7 (line 24), actions/setup-node@v6 (line 26), actions/upload-artifact@v7 (line 46)
codeql-analysis.yaml: actions/checkout@v7 (line 43), github/codeql-action/init@v4 (line 47), github/codeql-action/autobuild@v4 (line 57), github/codeql-action/analyze@v4 (line 66)
dependabot-pr-merge.yaml: dependabot/fetch-metadata@v3 (line 15)
on-push.yaml: actions/checkout@v7 (line 17), actions/setup-node@v6 (line 19), actions/upload-artifact@v7 (line 40), actions/checkout@v7 (line 47)
on-tag.yaml: actions/checkout@v7 (line 10)
with-wait-for-jobs.yaml: actions/upload-artifact@v7 (line 47), actions/checkout@v7 (line 53)

Locations:

- `.github/workflows/check-dist.yaml:24`
- `.github/workflows/codeql-analysis.yaml:43`
- `.github/workflows/dependabot-pr-merge.yaml:15`
- `.github/workflows/on-push.yaml:17`
- `.github/workflows/on-tag.yaml:10`
- `.github/workflows/with-wait-for-jobs.yaml:47`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any of their jobs. Without explicit permissions, workflows inherit the default (often write-all) token permissions, granting unnecessary access.

- .github/workflows/check-dist.yaml
- .github/workflows/on-push.yaml
- .github/workflows/on-tag.yaml
- .github/workflows/with-wait-for-jobs.yaml
- .github/workflows/without-wait-for-jobs.yaml

Locations:

- `.github/workflows/check-dist.yaml:1`
- `.github/workflows/on-push.yaml:1`
- `.github/workflows/on-tag.yaml:1`
- `.github/workflows/with-wait-for-jobs.yaml:1`
- `.github/workflows/without-wait-for-jobs.yaml:1`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions into shell command strings (sub-rule a). Before the shell executes the command, GitHub Actions substitutes the expression value as raw text, allowing an attacker to inject shell metacharacters.

1. on-tag.yaml (lines 14–17): `${{ github.ref_name }}` is interpolated directly into bash conditionals and a command substitution: `if [[ "${{ github.ref_name }}" =~ ${semver} ]]`, `major=$(echo "${{ github.ref_name }}" | cut -d '.' -f 1)`. A tag name containing shell metacharacters (e.g. `v1.0; malicious-command`) would execute arbitrary code.

2. on-push.yaml (line 50): `echo "Some step that can run without job 2:: ${{ needs.job1.outputs.out1 }}"`
   on-push.yaml (line 57): `echo "Some step that needs job 2:: ${{ fromJSON(steps.wait-for-jobs.outputs.outputs).out2 }}"`

3. with-wait-for-jobs.yaml (line 48): `echo "Some step that can run without job 3:: ${{ needs.job1.outputs.out1 }} :: ${{ needs.job2.outputs.out2 }}"`
   with-wait-for-jobs.yaml (line 57): `echo "Some step that needs job 3:: ${{ fromJSON(steps.wait-for-jobs.outputs.outputs).out3 }}"`

4. without-wait-for-jobs.yaml (line 41): `echo "Some step that can run without job 3:: ${{ needs.job1.outputs.out1 }} :: ${{ needs.job2.outputs.out2 }}"`
   without-wait-for-jobs.yaml (line 44): `echo "Some step that needs job 3:: ${{ needs.job3.outputs.out3 }}"`

Fix: move the values into `env:` variables and reference them as `"$ENV_VAR"` in the shell script.

Locations:

- `.github/workflows/on-tag.yaml:14`
- `.github/workflows/on-push.yaml:50`
- `.github/workflows/on-push.yaml:57`
- `.github/workflows/with-wait-for-jobs.yaml:48`
- `.github/workflows/with-wait-for-jobs.yaml:57`
- `.github/workflows/without-wait-for-jobs.yaml:41`
- `.github/workflows/without-wait-for-jobs.yaml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across 7 workflow files:

1. unpinned-uses: Pinned all 7 action references to full 40-char SHAs with tag comments: actions/checkout@v7→3d3c42e5, actions/setup-node@v6→249970729, actions/upload-artifact@v7→043fb46d, github/codeql-action/{init,autobuild,analyze}@v4→e4fba868, dependabot/fetch-metadata@v3→25dd0e34.

2. missing-permissions: Added top-level `permissions:` blocks to check-dist.yaml (contents: read), on-push.yaml (contents: read), on-tag.yaml (contents: write — needed to push tags), with-wait-for-jobs.yaml (contents: read), and without-wait-for-jobs.yaml (contents: read).

3. script-injection: Moved all ${{ }} expressions out of run: blocks into step-level env: blocks and referenced them as plain $ENV_VAR shell variables in all affected files: on-tag.yaml (github.ref_name→REF_NAME), on-push.yaml (two steps), with-wait-for-jobs.yaml (two steps), without-wait-for-jobs.yaml (two steps).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansion in .github/workflows/on-tag.yaml. The `major` variable (derived from `github.ref_name` via `REF_NAME` env var) was used unquoted in `git tag -f ${major}` and `git push -f origin ${major}`. Both references have been double-quoted to `"${major}"` to prevent shell metacharacter injection from attacker-controlled tag names.

