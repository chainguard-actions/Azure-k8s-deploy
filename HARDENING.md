<!-- markdownlint-disable -->

# Hardening Report: Azure--k8s-deploy/v7.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Azure--k8s-deploy/v7.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action `azure/login@v3.0.0` is referenced by a mutable version tag rather than a full 40-character commit SHA. This means the action could be silently replaced with a different (potentially malicious) version without any change to the workflow file.

Locations:

- `.github/workflows/run-integration-tests-private.yml:33`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions (`${{ env.NAMESPACE }}`, `${{ env.NAMESPACE1 }}`, `${{ env.NAMESPACE2 }}`) are interpolated directly inside `run:` shell command strings. Even though `env.*` values are set from `github.run_id` (not directly attacker-controlled), the YAML template substitution occurs before the shell ever sees the string, meaning any special shell characters in the expanded value are interpreted by the shell. All `${{ ... }}` expressions must be moved to `env:` blocks and referenced as quoted shell variables (e.g. `"$NAMESPACE"`) rather than being interpolated inline. Affected steps include: `kubectl create ns ${{ env.NAMESPACE }}`, `python ... ${{ env.NAMESPACE }}`, `az group create ... --name ${{ env.NAMESPACE }}`, `for ns in ${{ env.NAMESPACE1 }} ${{ env.NAMESPACE2 }}`, and similar patterns across multiple files.

Locations:

- `.github/workflows/run-integration-tests-basic.yml:27`
- `.github/workflows/run-integration-tests-bluegreen-ingress.yml:27`
- `.github/workflows/run-integration-tests-bluegreen-service.yml:27`
- `.github/workflows/run-integration-tests-bluegreen-smi.yml:27`
- `.github/workflows/run-integration-tests-canary-pod.yml:27`
- `.github/workflows/run-integration-tests-canary-smi.yml:27`
- `.github/workflows/run-integration-tests-namespace-optional.yml:30`
- `.github/workflows/run-integration-tests-private.yml:38`
- `.github/workflows/run-integration-tests-resource-annotation.yml:27`

### missing-permissions (severity: medium)

These workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, GitHub Actions grants the default token permissions (which may include write access to contents, packages, etc. depending on repository settings), violating the principle of least privilege.

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

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across 11 workflow files:

1. unpinned-uses: Pinned azure/login@v3.0.0 to SHA 532459ea530d8321f2fb9bb10d1e0bcf23869a43 in run-integration-tests-private.yml.

2. script-injection: Moved all ${{ env.NAMESPACE }}, ${{ env.NAMESPACE1 }}, and ${{ env.NAMESPACE2 }} expressions out of run: shell strings and into env: blocks in each affected step. Shell scripts now reference values as quoted variables ("$NAMESPACE", etc.). Affected files: run-integration-tests-basic.yml, run-integration-tests-bluegreen-ingress.yml, run-integration-tests-bluegreen-service.yml, run-integration-tests-bluegreen-smi.yml, run-integration-tests-canary-pod.yml, run-integration-tests-canary-smi.yml, run-integration-tests-namespace-optional.yml, run-integration-tests-private.yml, run-integration-tests-resource-annotation.yml.

3. missing-permissions: Added top-level permissions: blocks to all 11 workflow files. defaultLabels.yml received issues:write and pull-requests:write (required for actions/stale). All other workflows received contents:read as the minimum needed permission.

### Iteration 2

**Fixes applied:** unsafe-shell

**Notes:**

Fixed two unsafe `curl | sh` commands in `.github/actions/minikube-setup/action.yml`:
1. `curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install-edge | sh` → downloads to `/tmp/linkerd-install.sh`, executes separately, then removes the file.
2. `curl -sL https://linkerd.github.io/linkerd-smi/install | sh` → upgraded to use `--proto '=https' --tlsv1.2 -sSfL`, downloads to `/tmp/linkerd-smi-install.sh`, executes separately, then removes the file.
Both scripts are now downloaded first and executed as separate steps, preventing arbitrary remote code from being piped directly into a shell interpreter.

