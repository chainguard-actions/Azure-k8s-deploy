<!-- markdownlint-disable -->

# Hardening Report: Azure--k8s-deploy/v7.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Azure--k8s-deploy/v7.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses `azure/login@v3.0.0` — a mutable tag reference rather than a pinned 40-character commit SHA. This means the action could be silently updated to a different (potentially malicious) version without any change to the workflow file.

Locations:

- `.github/workflows/run-integration-tests-private.yml:31`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job, meaning they run with the default (potentially broad) token permissions: prettify-code.yml, unit-tests.yml, defaultLabels.yml, run-integration-tests-basic.yml, run-integration-tests-bluegreen-ingress.yml, run-integration-tests-bluegreen-service.yml, run-integration-tests-bluegreen-smi.yml, run-integration-tests-canary-pod.yml, run-integration-tests-canary-smi.yml, run-integration-tests-namespace-optional.yml, run-integration-tests-resource-annotation.yml.

Locations:

- `.github/workflows/prettify-code.yml:1`
- `.github/workflows/unit-tests.yml:1`
- `.github/workflows/defaultLabels.yml:1`
- `.github/workflows/run-integration-tests-basic.yml:1`
- `.github/workflows/run-integration-tests-bluegreen-ingress.yml:1`
- `.github/workflows/run-integration-tests-bluegreen-service.yml:1`
- `.github/workflows/run-integration-tests-bluegreen-smi.yml:1`
- `.github/workflows/run-integration-tests-canary-pod.yml:1`
- `.github/workflows/run-integration-tests-canary-smi.yml:1`
- `.github/workflows/run-integration-tests-namespace-optional.yml:1`
- `.github/workflows/run-integration-tests-resource-annotation.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ env.NAMESPACE }}` (and similar `${{ ... }}` expressions) into shell command strings. Per rule (a), any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the template engine before the shell ever sees it, bypassing shell quoting. The `env.NAMESPACE` variable is itself derived from `${{ github.run_id }}`. Affected steps include `run: kubectl create ns ${{ env.NAMESPACE }}` and multiline run blocks passing `${{ env.NAMESPACE }}` as arguments to Python scripts. All values should be passed via env vars and referenced as `"$NAMESPACE"` in the shell.

Locations:

- `.github/workflows/run-integration-tests-basic.yml:28`
- `.github/workflows/run-integration-tests-bluegreen-ingress.yml:22`
- `.github/workflows/run-integration-tests-bluegreen-service.yml:22`
- `.github/workflows/run-integration-tests-bluegreen-smi.yml:24`
- `.github/workflows/run-integration-tests-canary-pod.yml:22`
- `.github/workflows/run-integration-tests-canary-smi.yml:24`
- `.github/workflows/run-integration-tests-resource-annotation.yml:22`
- `.github/workflows/run-integration-tests-namespace-optional.yml:30`
- `.github/workflows/run-integration-tests-private.yml:38`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

1. unpinned-uses: Pinned azure/login@v3.0.0 to azure/login@532459ea530d8321f2fb9bb10d1e0bcf23869a43 # v3.0.0 in run-integration-tests-private.yml.

2. missing-permissions: Added top-level permissions blocks to 11 workflow files. Used permissions: {} for CI/test workflows (prettify-code.yml, unit-tests.yml, and all integration test workflows). Used permissions: {issues: write, pull-requests: write} for defaultLabels.yml since the stale action requires those permissions. run-integration-tests-private.yml already had job-level permissions so was not modified for this finding.

3. script-injection: In all 9 affected workflow files, moved every ${{ env.NAMESPACE }}, ${{ env.NAMESPACE1 }}, and ${{ env.NAMESPACE2 }} expression that appeared inside run: blocks into the step's env: block (as NS, NS1, NS2), then referenced them as "$NS", "$NS1", "$NS2" in the shell scripts. References in with: blocks (action inputs) were left unchanged as they are not shell-interpolated.

