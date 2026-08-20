<!-- markdownlint-disable -->

# Hardening Report: Azure--k8s-deploy/v5.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Azure--k8s-deploy/v5.0.4** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two workflow files reference external actions/workflows by mutable tags rather than full 40-character commit SHAs, making them vulnerable to supply-chain attacks: (1) release-pr.yml uses `Azure/action-release-workflows/.github/workflows/release_js_project.yaml@v1` (tag `v1`); (2) run-integration-tests-private.yml uses `azure/login@v2.3.0` (tag `v2.3.0`).

Locations:

- `.github/workflows/release-pr.yml:15`
- `.github/workflows/run-integration-tests-private.yml:30`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs, meaning they run with the default (potentially broad) token permissions: defaultLabels.yml, prettify-code.yml, unit-tests.yml, run-integration-tests-basic.yml, run-integration-tests-bluegreen-ingress.yml, run-integration-tests-bluegreen-service.yml, run-integration-tests-bluegreen-smi.yml, run-integration-tests-canary-pod.yml, run-integration-tests-canary-smi.yml, run-integration-tests-namespace-optional.yml, run-integration-tests-resource-annotation.yml.

Locations:

- `.github/workflows/defaultLabels.yml:1`
- `.github/workflows/prettify-code.yml:1`
- `.github/workflows/unit-tests.yml:1`
- `.github/workflows/run-integration-tests-basic.yml:1`
- `.github/workflows/run-integration-tests-bluegreen-ingress.yml:1`
- `.github/workflows/run-integration-tests-bluegreen-service.yml:1`
- `.github/workflows/run-integration-tests-bluegreen-smi.yml:1`
- `.github/workflows/run-integration-tests-canary-pod.yml:1`
- `.github/workflows/run-integration-tests-canary-smi.yml:1`
- `.github/workflows/run-integration-tests-namespace-optional.yml:1`
- `.github/workflows/run-integration-tests-resource-annotation.yml:1`

### script-injection (severity: high)

Multiple workflow files interpolate `${{ env.NAMESPACE }}` (and `${{ env.NAMESPACE1 }}` / `${{ env.NAMESPACE2 }}`) directly inside `run:` shell command strings (rule a). The env var is set to `test-${{ github.run_id }}` which flows through YAML template substitution before the shell sees it. Affected files and representative steps: run-integration-tests-basic.yml (`run: kubectl create ns ${{ env.NAMESPACE }}`), run-integration-tests-bluegreen-ingress.yml, run-integration-tests-bluegreen-service.yml, run-integration-tests-bluegreen-smi.yml, run-integration-tests-canary-pod.yml, run-integration-tests-canary-smi.yml, run-integration-tests-namespace-optional.yml, run-integration-tests-private.yml, and run-integration-tests-resource-annotation.yml. The value should be passed via an environment variable and referenced as `$NAMESPACE` in the shell instead.

Locations:

- `.github/workflows/run-integration-tests-basic.yml:28`
- `.github/workflows/run-integration-tests-bluegreen-ingress.yml:29`
- `.github/workflows/run-integration-tests-bluegreen-service.yml:29`
- `.github/workflows/run-integration-tests-bluegreen-smi.yml:36`
- `.github/workflows/run-integration-tests-canary-pod.yml:28`
- `.github/workflows/run-integration-tests-canary-smi.yml:35`
- `.github/workflows/run-integration-tests-namespace-optional.yml:32`
- `.github/workflows/run-integration-tests-private.yml:40`
- `.github/workflows/run-integration-tests-resource-annotation.yml:29`

### unsafe-shell (severity: high)

Two workflow files pipe remote content directly to a shell interpreter without first downloading and verifying the script. In run-integration-tests-bluegreen-smi.yml and run-integration-tests-canary-smi.yml, the Linkerd installer is fetched and executed with `curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install-edge | sh` and `curl -sL https://linkerd.github.io/linkerd-smi/install | sh`. A compromised or MITM'd remote server could serve malicious shell code that executes immediately on the runner.

Locations:

- `.github/workflows/run-integration-tests-bluegreen-smi.yml:34`
- `.github/workflows/run-integration-tests-canary-smi.yml:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, unsafe-shell

**Notes:**

Fixed all four findings across 13 workflow files:

1. unpinned-uses: Pinned Azure/action-release-workflows@v1 to SHA 3c677ba5ab58f5c5c1a6f0cfb176b333b1f27405 in release-pr.yml, and azure/login@v2.3.0 to SHA a457da9ea143d694b1b9c7c869ebb04ebe844ef5 in run-integration-tests-private.yml.

2. missing-permissions: Added top-level permissions blocks to defaultLabels.yml (issues:write + pull-requests:write for stale action), prettify-code.yml (contents:read), unit-tests.yml (contents:read), and all 9 integration test workflow files (contents:read).

3. script-injection: Replaced all ${{ env.NAMESPACE }}, ${{ env.NAMESPACE1 }}, ${{ env.NAMESPACE2 }} expressions in run: shell steps with $NAMESPACE, $NAMESPACE1, $NAMESPACE2 shell variable references across all 9 affected integration test files. Occurrences in with: YAML blocks were left unchanged as they are not shell injection risks.

4. unsafe-shell: Fixed curl-pipe-to-shell in run-integration-tests-bluegreen-smi.yml and run-integration-tests-canary-smi.yml by downloading the linkerd install scripts to /tmp files first, then executing them with sh separately. No -- argument was present in the original pipe forms so none needed to be dropped.

