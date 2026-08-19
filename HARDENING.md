<!-- markdownlint-disable -->

# Hardening Report: yogeshlonkar--wait-for-jobs/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yogeshlonkar--wait-for-jobs/v1.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tag-based refs instead of pinned 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the tag is moved. Failing references include: check-dist.yaml: actions/checkout@v6, actions/setup-node@v6, actions/upload-artifact@v7; codeql-analysis.yaml: actions/checkout@v6, github/codeql-action/init@v4, github/codeql-action/autobuild@v4, github/codeql-action/analyze@v4; dependabot-pr-merge.yaml: dependabot/fetch-metadata@v3; on-push.yaml: actions/checkout@v6, actions/setup-node@v6, actions/upload-artifact@v7, actions/checkout@v6; on-tag.yaml: actions/checkout@v6; with-wait-for-jobs.yaml: actions/upload-artifact@v7, actions/checkout@v6.

Locations:

- `.github/workflows/check-dist.yaml:24`
- `.github/workflows/check-dist.yaml:27`
- `.github/workflows/check-dist.yaml:47`
- `.github/workflows/codeql-analysis.yaml:36`
- `.github/workflows/codeql-analysis.yaml:40`
- `.github/workflows/codeql-analysis.yaml:47`
- `.github/workflows/codeql-analysis.yaml:51`
- `.github/workflows/dependabot-pr-merge.yaml:13`
- `.github/workflows/on-push.yaml:18`
- `.github/workflows/on-push.yaml:20`
- `.github/workflows/on-push.yaml:49`
- `.github/workflows/on-push.yaml:61`
- `.github/workflows/on-tag.yaml:12`
- `.github/workflows/with-wait-for-jobs.yaml:44`
- `.github/workflows/with-wait-for-jobs.yaml:57`

### script-injection (severity: high)

Multiple run: blocks interpolate ${{ }} expressions directly into shell command strings (sub-rule a). This allows expression values to be interpreted as shell code before the shell ever sees them. Specific violations: on-tag.yaml lines 16, 17, 21 use ${{ github.ref_name }} directly in shell conditionals and command substitution; on-push.yaml line 60 uses ${{ needs.job1.outputs.out1 }} and line 69 uses ${{ fromJSON(steps.wait-for-jobs.outputs.outputs).out2 }} in echo commands; with-wait-for-jobs.yaml line 55 uses ${{ needs.job1.outputs.out1 }} and ${{ needs.job2.outputs.out2 }}, line 65 uses ${{ fromJSON(steps.wait-for-jobs.outputs.outputs).out3 }}; without-wait-for-jobs.yaml line 48 uses ${{ needs.job1.outputs.out1 }} and ${{ needs.job2.outputs.out2 }}, line 51 uses ${{ needs.job3.outputs.out3 }}.

Locations:

- `.github/workflows/on-tag.yaml:16`
- `.github/workflows/on-tag.yaml:17`
- `.github/workflows/on-tag.yaml:21`
- `.github/workflows/on-push.yaml:60`
- `.github/workflows/on-push.yaml:69`
- `.github/workflows/with-wait-for-jobs.yaml:55`
- `.github/workflows/with-wait-for-jobs.yaml:65`
- `.github/workflows/without-wait-for-jobs.yaml:48`
- `.github/workflows/without-wait-for-jobs.yaml:51`

### missing-permissions (severity: medium)

Five workflow files have no top-level permissions: key and no per-job permissions: blocks. Without explicit permissions, workflows inherit the default repository token permissions (which may be read/write), violating the principle of least privilege. Affected files: check-dist.yaml, on-push.yaml, on-tag.yaml, with-wait-for-jobs.yaml, without-wait-for-jobs.yaml.

Locations:

- `.github/workflows/check-dist.yaml:1`
- `.github/workflows/on-push.yaml:1`
- `.github/workflows/on-tag.yaml:1`
- `.github/workflows/with-wait-for-jobs.yaml:1`
- `.github/workflows/without-wait-for-jobs.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across 7 workflow files:

1. unpinned-uses: Pinned all action references to full 40-char commit SHAs with tag comments preserved: actions/checkout@v6→d23441a4, actions/setup-node@v6→249970729, actions/upload-artifact@v7→043fb46d, github/codeql-action/{init,autobuild,analyze}@v4→e4fba868, dependabot/fetch-metadata@v3→25dd0e34.

2. script-injection: Moved all ${{ }} expressions out of run: blocks into step-level env: blocks, referencing them as plain shell variables ($VAR_NAME). Affected: on-tag.yaml (github.ref_name→REF_NAME), on-push.yaml (needs outputs and fromJSON expressions), with-wait-for-jobs.yaml (needs outputs and fromJSON expressions), without-wait-for-jobs.yaml (needs outputs).

3. missing-permissions: Added top-level permissions blocks to check-dist.yaml (contents: read), on-push.yaml (contents: read), on-tag.yaml (contents: write — needed to push tags), with-wait-for-jobs.yaml (contents: read), without-wait-for-jobs.yaml (contents: read). The codeql-analysis.yaml and dependabot-pr-merge.yaml already had permissions blocks.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansion in .github/workflows/on-tag.yaml. Changed `git tag -f ${major}` to `git tag -f "${major}"` and `git push -f origin ${major}` to `git push -f origin "${major}"` on lines 27-28. The `major` variable is derived from `$REF_NAME` which comes from `github.ref_name` (a workflow-controllable value), so double-quoting prevents shell metacharacter injection attacks.

