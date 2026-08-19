<!-- markdownlint-disable -->

# Hardening Report: Azure--k8s-deploy/v5.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Azure--k8s-deploy/v5.0.3** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two workflow files reference external actions/workflows using mutable tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks:
- `.github/workflows/release-pr.yml`: `uses: Azure/action-release-workflows/.github/workflows/release_js_project.yaml@v1` (tag `v1`)
- `.github/workflows/run-integration-tests-private.yml`: `uses: azure/login@v2.3.0` (tag `v2.3.0`)

Locations:

- `.github/workflows/release-pr.yml:14`
- `.github/workflows/run-integration-tests-private.yml:27`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, workflows inherit the default repository token permissions (which may be broad). Each of these files should declare minimal required permissions:
- `defaultLabels.yml`
- `prettify-code.yml`
- `run-integration-tests-basic.yml`
- `run-integration-tests-bluegreen-ingress.yml`
- `run-integration-tests-bluegreen-service.yml`
- `run-integration-tests-bluegreen-smi.yml`
- `run-integration-tests-canary-pod.yml`
- `run-integration-tests-canary-smi.yml`
- `run-integration-tests-resource-annotation.yml`
- `unit-tests.yml`

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

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks across integration test workflow files directly interpolate `${{ env.NAMESPACE }}` inside shell commands. Although `env.NAMESPACE` is set at the job level, it is itself derived from `${{ github.run_id }}` — a GitHub context value that flows through YAML template substitution before the shell processes it. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk. Examples of offending lines:
- `run: kubectl create ns ${{ env.NAMESPACE }}`
- `python test/integration/k8s-deploy-delete.py 'Service' 'all' ${{ env.NAMESPACE }}`
- `az group create --location eastus2 --name ${{ env.NAMESPACE }}`
These appear in all integration test workflow files. The value should be passed via an environment variable and referenced as `$NAMESPACE` (without `${{ }}`) inside the `run:` block.

Locations:

- `.github/workflows/run-integration-tests-basic.yml:26`
- `.github/workflows/run-integration-tests-bluegreen-ingress.yml:26`
- `.github/workflows/run-integration-tests-bluegreen-service.yml:26`
- `.github/workflows/run-integration-tests-bluegreen-smi.yml:34`
- `.github/workflows/run-integration-tests-canary-pod.yml:26`
- `.github/workflows/run-integration-tests-canary-smi.yml:33`
- `.github/workflows/run-integration-tests-private.yml:33`
- `.github/workflows/run-integration-tests-resource-annotation.yml:26`

### unsafe-shell (severity: high)

Two workflow files pipe remote content directly to a shell interpreter using `curl ... | sh`, which executes arbitrary remote code without any integrity verification. Offending lines:
- `curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install-edge | sh`
- `curl -sL https://linkerd.github.io/linkerd-smi/install | sh`
The script should be downloaded to a file first, its integrity verified (e.g. via checksum), and then executed separately.

Locations:

- `.github/workflows/run-integration-tests-bluegreen-smi.yml:38`
- `.github/workflows/run-integration-tests-bluegreen-smi.yml:40`
- `.github/workflows/run-integration-tests-canary-smi.yml:37`
- `.github/workflows/run-integration-tests-canary-smi.yml:39`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, unsafe-shell

**Notes:**

Fixed all four finding types across 12 workflow files:

1. unpinned-uses: Pinned Azure/action-release-workflows to SHA 3c677ba5ab58f5c5c1a6f0cfb176b333b1f27405 in release-pr.yml, and azure/login to SHA a457da9ea143d694b1b9c7c869ebb04ebe844ef5 in run-integration-tests-private.yml.

2. missing-permissions: Added top-level permissions blocks to all 10 workflow files that lacked them. defaultLabels.yml gets issues:write and pull-requests:write (required by the stale action). All integration test workflows get contents:read. The private cluster workflow retains its existing id-token:write.

3. script-injection: In all 8 integration test workflow files, replaced every ${{ env.NAMESPACE }} inside run: shell blocks with $NAMESPACE shell variable references. Each affected step now has an env: block setting NAMESPACE: ${{ env.NAMESPACE }}, and the shell script uses "$NAMESPACE" instead.

4. unsafe-shell: In run-integration-tests-bluegreen-smi.yml and run-integration-tests-canary-smi.yml, replaced both curl|sh pipe patterns (linkerd install-edge and linkerd-smi install) with a download-then-execute pattern using mktemp. The canary-smi file also had a YAML structural issue where the curl commands were incorrectly embedded inside the setup-minikube step's with: block; this was fixed by creating a proper separate step named 'Install linkerd and add controlplane to cluster'.

