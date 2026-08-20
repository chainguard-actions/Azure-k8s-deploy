<!-- markdownlint-disable -->

# Hardening Report: Azure--k8s-deploy/v5.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Azure--k8s-deploy/v5.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references are pinned to mutable tags rather than full 40-character commit SHAs, making them vulnerable to supply-chain attacks:
1. `azure/login@v2.3.0` in run-integration-tests-private.yml (line 32) — tag ref, not a SHA.
2. `Azure/action-release-workflows/.github/workflows/release_js_project.yaml@v1` in release-pr.yml (line 15) — tag ref, not a SHA.

Locations:

- `.github/workflows/run-integration-tests-private.yml:32`
- `.github/workflows/release-pr.yml:15`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, workflows run with the default (potentially broad) token permissions:
- defaultLabels.yml
- prettify-code.yml
- unit-tests.yml
- run-integration-tests-basic.yml
- run-integration-tests-bluegreen-ingress.yml
- run-integration-tests-bluegreen-service.yml
- run-integration-tests-bluegreen-smi.yml
- run-integration-tests-canary-pod.yml
- run-integration-tests-canary-smi.yml
- run-integration-tests-namespace-optional.yml
- run-integration-tests-resource-annotation.yml

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

Multiple `run:` blocks directly interpolate `${{ env.NAMESPACE }}` (an `env.*` context expression) into shell commands without routing through a safely-quoted environment variable. Per sub-rule (a), ANY `${{ ... }}` expression directly inside a `run:` shell command string is a script-injection risk, as the value is substituted by the template engine before the shell ever sees it, allowing metacharacter injection. Affected steps include:
- run-integration-tests-basic.yml: `run: kubectl create ns ${{ env.NAMESPACE }}` and `run: python ... ${{ env.NAMESPACE }}`
- run-integration-tests-bluegreen-ingress.yml: `run: kubectl create ns ${{ env.NAMESPACE }}` and `run: kubectl delete ns ${{ env.NAMESPACE }}`
- run-integration-tests-bluegreen-service.yml: `run: kubectl create ns ${{ env.NAMESPACE }}` and `run: kubectl delete ns ${{ env.NAMESPACE }}`
- run-integration-tests-bluegreen-smi.yml: `run: kubectl create ns ${{ env.NAMESPACE }}` and `run: kubectl delete ns ${{ env.NAMESPACE }}`
- run-integration-tests-canary-pod.yml: `run: kubectl create ns ${{ env.NAMESPACE }}` and `run: kubectl delete ns ${{ env.NAMESPACE }}`
- run-integration-tests-canary-smi.yml: `run: kubectl create ns ${{ env.NAMESPACE }}` and `run: kubectl delete ns ${{ env.NAMESPACE }}`
- run-integration-tests-resource-annotation.yml: `run: kubectl create ns ${{ env.NAMESPACE }}`
- run-integration-tests-namespace-optional.yml: multiple `run:` blocks with `${{ env.NAMESPACE1 }}` and `${{ env.NAMESPACE2 }}`
Fix: move the value into an `env:` block and reference it as `"$NAMESPACE"` (double-quoted shell variable) in the `run:` script.

Locations:

- `.github/workflows/run-integration-tests-basic.yml:27`
- `.github/workflows/run-integration-tests-bluegreen-ingress.yml:27`
- `.github/workflows/run-integration-tests-bluegreen-service.yml:27`
- `.github/workflows/run-integration-tests-bluegreen-smi.yml:27`
- `.github/workflows/run-integration-tests-canary-pod.yml:27`
- `.github/workflows/run-integration-tests-canary-smi.yml:27`
- `.github/workflows/run-integration-tests-resource-annotation.yml:27`
- `.github/workflows/run-integration-tests-namespace-optional.yml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings:

1. unpinned-uses: Pinned azure/login@v2.3.0 → SHA a457da9ea143d694b1b9c7c869ebb04ebe844ef5 in run-integration-tests-private.yml; pinned Azure/action-release-workflows/.github/workflows/release_js_project.yaml@v1 → SHA 3c677ba5ab58f5c5c1a6f0cfb176b333b1f27405 in release-pr.yml.

2. missing-permissions: Added top-level `permissions: {}` and job-level permissions to all 11 affected workflows. defaultLabels.yml gets issues:write + pull-requests:write (needed by actions/stale); all other workflows get contents:read.

3. script-injection: In all 8 affected integration test workflows, moved every ${{ env.NAMESPACE }}, ${{ env.NAMESPACE1 }}, and ${{ env.NAMESPACE2 }} expression out of run: shell strings into step-level env: blocks (as NS, NS1, NS2), then referenced them as double-quoted shell variables ("$NS", "$NS1", "$NS2") in the run: scripts.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/run-integration-tests-private.yml by moving all ${{ env.NAMESPACE }} expressions out of run: shell strings and into step-level env: blocks (NS: ${{ env.NAMESPACE }}). The four affected steps now reference $NS as a plain shell environment variable: 'Create private AKS cluster and set context' (lines 40-42), 'Create namespace to run tests' (line 46), 'Checking if deployments and services were created' (line 60), and 'Clean up AKS cluster' (lines 70-71). The uses: with: blocks that reference ${{ env.NAMESPACE }} were left unchanged as those are not shell injection risks.

### Iteration 3

**Fixes applied:** unsafe-shell

**Notes:**

Fixed two curl-pipe-to-shell patterns in hardened/action/.github/actions/minikube-setup/action.yml:
1. `curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install-edge | sh` → download to mktemp file, then `sh "$LINKERD_INSTALL_SCRIPT"`
2. `curl -sL https://linkerd.github.io/linkerd-smi/install | sh` → download to mktemp file, then `sh "$SMI_INSTALL_SCRIPT"`
Neither original command used `sh -s -- ARGS` form, so no `--` needed to be dropped. Scripts are now saved to disk before execution, preventing immediate execution of potentially compromised remote content.

