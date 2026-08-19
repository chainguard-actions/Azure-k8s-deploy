<!-- markdownlint-disable -->

# Hardening Report: Azure--k8s-deploy/v6.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Azure--k8s-deploy/v6.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple workflow run: blocks directly interpolate ${{ env.NAMESPACE }}, ${{ env.NAMESPACE1 }}, and ${{ env.NAMESPACE2 }} expressions inside shell commands (rule a). Any ${{ ... }} expression inside a run: block flows through YAML template substitution before the shell processes it, enabling script injection. Examples include: `run: kubectl create ns ${{ env.NAMESPACE }}`, `run: kubectl delete ns ${{ env.NAMESPACE }}`, `python test/integration/k8s-deploy-delete.py 'Service' 'all' ${{ env.NAMESPACE }}`, `az group create --location eastus2 --name ${{ env.NAMESPACE }}`, and `for ns in ${{ env.NAMESPACE1 }} ${{ env.NAMESPACE2 }}`. These should be replaced with safe env-var references like `$NAMESPACE` (set via the env: block) and double-quoted.

Locations:

- `.github/workflows/run-integration-tests-basic.yml:27`
- `.github/workflows/run-integration-tests-bluegreen-ingress.yml:27`
- `.github/workflows/run-integration-tests-bluegreen-service.yml:27`
- `.github/workflows/run-integration-tests-bluegreen-smi.yml:28`
- `.github/workflows/run-integration-tests-canary-pod.yml:27`
- `.github/workflows/run-integration-tests-canary-smi.yml:28`
- `.github/workflows/run-integration-tests-namespace-optional.yml:31`
- `.github/workflows/run-integration-tests-private.yml:39`
- `.github/workflows/run-integration-tests-resource-annotation.yml:27`

### unpinned-uses (severity: high)

Two workflow files reference external actions using mutable tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks: (1) `uses: azure/login@v3.0.0` in run-integration-tests-private.yml — the tag `v3.0.0` can be moved to point to a different commit; (2) `uses: Azure/action-release-workflows/.github/workflows/release_js_project.yaml@v1` in release-pr.yml — the tag `v1` is mutable. Both should be pinned to a full SHA digest.

Locations:

- `.github/workflows/run-integration-tests-private.yml:33`
- `.github/workflows/release-pr.yml:14`

### missing-permissions (severity: medium)

Multiple workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions (which may be read/write for contents), violating the principle of least privilege. Affected files: defaultLabels.yml, prettify-code.yml, run-integration-tests-basic.yml, run-integration-tests-bluegreen-ingress.yml, run-integration-tests-bluegreen-service.yml, run-integration-tests-bluegreen-smi.yml, run-integration-tests-canary-pod.yml, run-integration-tests-canary-smi.yml, run-integration-tests-namespace-optional.yml, run-integration-tests-resource-annotation.yml, unit-tests.yml.

Locations:

- `.github/workflows/defaultLabels.yml:1`
- `.github/workflows/prettify-code.yml:1`
- `.github/workflows/run-integration-tests-basic.yml:1`
- `.github/workflows/run-integration-tests-bluegreen-ingress.yml:1`
- `.github/workflows/run-integration-tests-bluegreen-service.yml:1`
- `.github/workflows/run-integration-tests-bluegreen-smi.yml:1`
- `.github/workflows/run-integration-tests-canary-pod.yml:1`
- `.github/workflows/run-integration-tests-canary-smi.yml:1`
- `.github/workflows/run-integration-tests-namespace-optional.yml:1`
- `.github/workflows/run-integration-tests-resource-annotation.yml:1`
- `.github/workflows/unit-tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across 14 workflow files:

1. script-injection: Replaced all ${{ env.NAMESPACE }}, ${{ env.NAMESPACE1 }}, and ${{ env.NAMESPACE2 }} expressions inside run: shell blocks with safe double-quoted env var references ($NAMESPACE, $NAMESPACE1, $NAMESPACE2). Remaining ${{ env.NAMESPACE* }} expressions are only in with: action input blocks, which are not shell-executed.

2. unpinned-uses: Pinned azure/login@v3.0.0 to SHA 532459ea530d8321f2fb9bb10d1e0bcf23869a43 in run-integration-tests-private.yml, and pinned Azure/action-release-workflows/.github/workflows/release_js_project.yaml@v1 to SHA 3c677ba5ab58f5c5c1a6f0cfb176b333b1f27405 in release-pr.yml.

3. missing-permissions: Added top-level `permissions: {}` to all 11 affected workflow files: defaultLabels.yml, prettify-code.yml, run-integration-tests-basic.yml, run-integration-tests-bluegreen-ingress.yml, run-integration-tests-bluegreen-service.yml, run-integration-tests-bluegreen-smi.yml, run-integration-tests-canary-pod.yml, run-integration-tests-canary-smi.yml, run-integration-tests-namespace-optional.yml, run-integration-tests-resource-annotation.yml, and unit-tests.yml. run-integration-tests-private.yml already had job-level permissions and now also has a top-level permissions: {} block.

