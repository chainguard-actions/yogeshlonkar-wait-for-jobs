<!-- markdownlint-disable -->

# Hardening Report: yogeshlonkar--wait-for-jobs/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **yogeshlonkar--wait-for-jobs/v1.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of immutable 40-character SHA digests, making them vulnerable to supply-chain attacks if the tag is moved.

check-dist.yaml: actions/checkout@v6, actions/setup-node@v6, actions/upload-artifact@v7
codeql-analysis.yaml: actions/checkout@v6, github/codeql-action/init@v4, github/codeql-action/autobuild@v4, github/codeql-action/analyze@v4
dependabot-pr-merge.yaml: dependabot/fetch-metadata@v2
on-push.yaml: actions/checkout@v6, actions/setup-node@v6, actions/upload-artifact@v7
on-tag.yaml: actions/checkout@v6
with-wait-for-jobs.yaml: actions/upload-artifact@v7, actions/checkout@v6

Locations:

- `.github/workflows/check-dist.yaml:24`
- `.github/workflows/check-dist.yaml:27`
- `.github/workflows/check-dist.yaml:44`
- `.github/workflows/codeql-analysis.yaml:36`
- `.github/workflows/codeql-analysis.yaml:41`
- `.github/workflows/codeql-analysis.yaml:51`
- `.github/workflows/codeql-analysis.yaml:57`
- `.github/workflows/dependabot-pr-merge.yaml:13`
- `.github/workflows/on-push.yaml:17`
- `.github/workflows/on-push.yaml:19`
- `.github/workflows/on-push.yaml:43`
- `.github/workflows/on-tag.yaml:12`
- `.github/workflows/with-wait-for-jobs.yaml:43`
- `.github/workflows/with-wait-for-jobs.yaml:50`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ ... }} expressions into shell commands (sub-rule a), allowing an attacker to inject arbitrary shell commands.

on-tag.yaml: ${{ github.ref_name }} is interpolated directly into a bash conditional and command substitution inside a run: block. A tag name containing shell metacharacters could alter the script's behavior.
  Line 15: if [[ "${{ github.ref_name }}" =~ ${semver} ]]; then
  Line 16: if [[ "${{ github.ref_name }}" == *"-rc"* ]]; then
  Line 20: major=$(echo "${{ github.ref_name }}" | cut -d '.' -f 1)

on-push.yaml: ${{ needs.job1.outputs.out1 }} and ${{ fromJSON(steps.wait-for-jobs.outputs.outputs).out2 }} are interpolated directly into echo commands inside run: blocks.
  Line 50: echo "Some step that can run without job 2:: ${{ needs.job1.outputs.out1 }}"
  Line 55: echo "Some step that needs job 2:: ${{ fromJSON(steps.wait-for-jobs.outputs.outputs).out2 }}"

with-wait-for-jobs.yaml: ${{ needs.job1.outputs.out1 }}, ${{ needs.job2.outputs.out2 }}, and ${{ fromJSON(steps.wait-for-jobs.outputs.outputs).out3 }} are interpolated directly into run: blocks.
  Line 51: echo "Some step that can run without job 3:: ${{ needs.job1.outputs.out1 }} :: ${{ needs.job2.outputs.out2 }}"
  Line 56: echo "Some step that needs job 3:: ${{ fromJSON(steps.wait-for-jobs.outputs.outputs).out3 }}"

without-wait-for-jobs.yaml: ${{ needs.job1.outputs.out1 }}, ${{ needs.job2.outputs.out2 }}, and ${{ needs.job3.outputs.out3 }} are interpolated directly into run: blocks.
  Line 43: echo "Some step that can run without job 3:: ${{ needs.job1.outputs.out1 }} :: ${{ needs.job2.outputs.out2 }}"
  Line 45: echo "Some step that needs job 3:: ${{ needs.job3.outputs.out3 }}"

Locations:

- `.github/workflows/on-tag.yaml:15`
- `.github/workflows/on-tag.yaml:16`
- `.github/workflows/on-tag.yaml:20`
- `.github/workflows/on-push.yaml:50`
- `.github/workflows/on-push.yaml:55`
- `.github/workflows/with-wait-for-jobs.yaml:51`
- `.github/workflows/with-wait-for-jobs.yaml:56`
- `.github/workflows/without-wait-for-jobs.yaml:43`
- `.github/workflows/without-wait-for-jobs.yaml:45`

### missing-permissions (severity: medium)

Five workflow files have no top-level `permissions:` block and no job-level `permissions:` block on any of their jobs. Without explicit permissions, workflows run with the default token permissions (which may be write-all depending on repository settings), violating the principle of least privilege.

- check-dist.yaml: no permissions declared at any level
- on-push.yaml: no permissions declared at any level
- on-tag.yaml: no permissions declared at any level
- with-wait-for-jobs.yaml: no permissions declared at any level
- without-wait-for-jobs.yaml: no permissions declared at any level

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

Fixed all three finding types across 7 workflow files:

1. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments: actions/checkout@v6→d23441a4, actions/setup-node@v6→249970729, actions/upload-artifact@v7→043fb46d, github/codeql-action/{init,autobuild,analyze}@v4→e4fba868, dependabot/fetch-metadata@v2→21025c70.

2. script-injection: Moved all ${{ }} expressions from run: blocks into step env: blocks and referenced them as plain shell variables in: on-tag.yaml (github.ref_name→REF_NAME), on-push.yaml (needs.job1.outputs.out1→JOB1_OUT1, fromJSON(...)→WAIT_OUT2), with-wait-for-jobs.yaml (job1/job2 outputs→JOB1_OUT1/JOB2_OUT2, fromJSON(...)→WAIT_OUT3), without-wait-for-jobs.yaml (job1/job2/job3 outputs→JOB1_OUT1/JOB2_OUT2/JOB3_OUT3).

3. missing-permissions: Added top-level permissions blocks to check-dist.yaml (contents: read), on-push.yaml (contents: read), on-tag.yaml (contents: write, needed for git tag/push), with-wait-for-jobs.yaml (contents: read), without-wait-for-jobs.yaml (contents: read). The codeql-analysis.yaml and dependabot-pr-merge.yaml already had permissions blocks.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerability in .github/workflows/on-tag.yaml by double-quoting `${major}` in both shell commands: changed `git tag -f ${major}` to `git tag -f "${major}"` and `git push -f origin ${major}` to `git push -f origin "${major}"`. This prevents shell metacharacter interpretation when the variable (derived from the attacker-controllable `github.ref_name`) is expanded.

