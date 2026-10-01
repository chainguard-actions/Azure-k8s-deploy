<!-- markdownlint-disable -->

# Hardening Report: Azure--k8s-deploy/v7.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Azure--k8s-deploy/v7.0.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unsafe-shell (severity: high)

Two `run:` steps in the composite action pipe remote content directly to a shell interpreter without first downloading and verifying the script. Line 38: `curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install-edge | sh` and line 40: `curl -sL https://linkerd.github.io/linkerd-smi/install | sh`. If either remote URL is compromised or redirected, arbitrary code will execute on the runner. The scripts should be downloaded to a file first, inspected/verified, and then executed separately.

Locations:

- `.github/actions/minikube-setup/action.yml:38`
- `.github/actions/minikube-setup/action.yml:40`

## Iteration Notes

### Iteration 1

**Fixes applied:** unsafe-shell

**Notes:**

Fixed two unsafe `curl | sh` patterns in `.github/actions/minikube-setup/action.yml`:
1. Line 38: `curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install-edge | sh` → downloads to a temp file via `mktemp`, then executes with `sh "$LINKERD_INSTALL_SCRIPT"`.
2. Line 40: `curl -sL https://linkerd.github.io/linkerd-smi/install | sh` → downloads to a temp file via `mktemp`, then executes with `sh "$SMI_INSTALL_SCRIPT"`.
No `--` separator was added to the `sh` invocations since those were shell stdin-reading options, not script arguments.

