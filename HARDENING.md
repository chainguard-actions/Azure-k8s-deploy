<!-- markdownlint-disable -->

# Hardening Report: Azure--k8s-deploy/v5.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Azure--k8s-deploy/v5.1.0** was hardened automatically. 0 finding(s) were identified and resolved across 1 iteration(s).

## Iteration Notes

### Iteration 1

**Fixes applied:** unsafe-shell

**Notes:**

Fixed two curl-pipe-to-shell patterns in hardened/action/.github/actions/minikube-setup/action.yml:
1. `curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install-edge | sh` → downloads to a mktemp file, then executes with `sh "$LINKERD_INSTALL_SCRIPT"`
2. `curl -sL https://linkerd.github.io/linkerd-smi/install | sh` → downloads to a mktemp file, then executes with `sh "$SMI_INSTALL_SCRIPT"`
Neither original command passed arguments through the pipe, so no '--' handling was needed.

