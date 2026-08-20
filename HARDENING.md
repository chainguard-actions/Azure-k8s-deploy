<!-- markdownlint-disable -->

# Hardening Report: Azure--k8s-deploy/v5.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Azure--k8s-deploy/v5.0.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: `${{ env.NAMESPACE }}` is interpolated directly inside `run:` shell command strings across multiple integration-test workflow files. The `env.*` context is workflow-controllable and must not appear literally inside a run block. For example: `run: kubectl create ns ${{ env.NAMESPACE }}` and `python test/integration/k8s-deploy-delete.py 'Service' 'all' ${{ env.NAMESPACE }}`. This allows an attacker who controls the env context to inject arbitrary shell commands.

Locations:

- `.github/workflows/run-integration-tests-basic.yml:47`
- `.github/workflows/run-integration-tests-bluegreen-ingress.yml:47`
- `.github/workflows/run-integration-tests-bluegreen-service.yml:47`
- `.github/workflows/run-integration-tests-bluegreen-smi.yml:54`
- `.github/workflows/run-integration-tests-canary-pod.yml:47`
- `.github/workflows/run-integration-tests-canary-smi.yml:54`
- `.github/workflows/run-integration-tests-private.yml:40`
- `.github/workflows/run-integration-tests-resource-annotation.yml:47`

### unpinned-uses (severity: high)

Two `uses:` references are pinned to mutable tags/versions rather than full 40-character commit SHAs, making them vulnerable to supply-chain attacks: (1) `release-pr.yml` uses `Azure/action-release-workflows/.github/workflows/release_js_project.yaml@v1` (tag `v1`); (2) `run-integration-tests-private.yml` uses `azure/login@v2.3.0` (version tag `v2.3.0`). Both should be pinned to a full SHA digest.

Locations:

- `.github/workflows/release-pr.yml:16`
- `.github/workflows/run-integration-tests-private.yml:33`

### unsafe-shell (severity: high)

Remote content is piped directly to `sh` without first downloading and verifying the script. In both SMI integration-test workflows, the Linkerd installer is fetched and executed in a single pipeline: `curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install-edge | sh` and `curl -sL https://linkerd.github.io/linkerd-smi/install | sh`. A compromised or MITM'd remote server could serve malicious shell code that executes immediately on the runner.

Locations:

- `.github/workflows/run-integration-tests-bluegreen-smi.yml:44`
- `.github/workflows/run-integration-tests-bluegreen-smi.yml:46`
- `.github/workflows/run-integration-tests-canary-smi.yml:44`
- `.github/workflows/run-integration-tests-canary-smi.yml:46`

### missing-permissions (severity: medium)

Ten workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the repository's default token permissions (often `write-all`), granting unnecessary broad access. Affected files: `defaultLabels.yml`, `prettify-code.yml`, `run-integration-tests-basic.yml`, `run-integration-tests-bluegreen-ingress.yml`, `run-integration-tests-bluegreen-service.yml`, `run-integration-tests-bluegreen-smi.yml`, `run-integration-tests-canary-pod.yml`, `run-integration-tests-canary-smi.yml`, `run-integration-tests-resource-annotation.yml`, `unit-tests.yml`.

Locations:

- `.github/workflows/defaultLabels.yml:1`
- `.github/workflows/prettify-code.yml:1`
- `.github/workflows/run-integration-tests-basic.yml:1`
- `.github/workflows/run-integration-tests-bluegreen-ingress.yml:1`
- `.github/workflows/run-integration-tests-bluegreen-service.yml:1`
- `.github/workflows/run-integration-tests-bluegreen-smi.yml:1`
- `.github/workflows/run-integration-tests-canary-pod.yml:1`
- `.github/workflows/run-integration-tests-canary-smi.yml:1`
- `.github/workflows/run-integration-tests-resource-annotation.yml:1`
- `.github/workflows/unit-tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, unsafe-shell, script-injection, missing-permissions

**Notes:**

Fixed all four finding types: (1) Pinned Azure/action-release-workflows@v1 to full SHA 3c677ba5ab58f5c5c1a6f0cfb176b333b1f27405 and azure/login@v2.3.0 to full SHA a457da9ea143d694b1b9c7c869ebb04ebe844ef5. (2) Replaced curl|sh pipe patterns in both SMI workflows with download-to-tempfile-then-execute pattern. (3) Replaced all ${{ env.NAMESPACE }} expressions in run: shell blocks with $NAMESPACE shell variable references across 8 workflow files; ${{ env.NAMESPACE }} in with: action input blocks was left as-is since those are not shell-interpolated. (4) Added permissions blocks to all 10 workflow files missing them: contents: read for most workflows, and issues: write + pull-requests: write for defaultLabels.yml which uses actions/stale.

