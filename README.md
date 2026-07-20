# Devin Template

A Podman-based Dev Container with **Devin CLI**, Node 22, Python 3, and gh — plus GPU passthrough and automatic handling of corporate TLS-interception proxies.

<https://github.com/ravikiranvs/devin-template>

> **Podman only.** The container config uses Podman-specific args that Docker will reject.

## Prerequisites

- Podman with a running machine
- [GPU container access](https://podman-desktop.io/docs/podman/gpu) — follow this set up guide
- VS Code with the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
- Node.js (for the `devcontainer` CLI)

Add to VS Code User Settings `settings.json`:

```jsonc
"dev.containers.dockerPath": "podman",
"dev.containers.dockerComposePath": "podman-compose",
"dev.containers.mountWaylandSocket": false
```

## Usage

**1. Install the devcontainers CLI**

```powershell
npm install -g @devcontainers/cli
```

**2. Apply the template**

```powershell
# --use-system-ca lets Node trust a corporate proxy root already in the Windows store.
$env:NODE_OPTIONS = "--use-system-ca"
devcontainer templates apply --template-id ghcr.io/ravikiranvs/devin-template/devin-workspace:latest --workspace-folder .
```

> `apply` is a one-time copy and won't overwrite an existing `.devcontainer/`. To pick up template updates, delete the folder and re-apply.

**3. Open it in the container**

1. Open the folder in VS Code (`code .`)
2. `F1` → **Dev Containers: Reopen in Container**
3. Wait for the build and `postCreate` to finish — watch the log via **Dev Containers: Show Container Log**

## First run

```bash
devin setup     # authenticate the Devin CLI
```

## What happens on container create

`postCreateCommand.sh` runs two things:

1. **Corporate CA** — if TLS is being intercepted, extracts the proxy root and installs it into the trust store. Does nothing when the chain already verifies, so it's safe off-network.
2. **Skills** — installs [mattpocock/skills](https://github.com/mattpocock/skills) into `.agents/skills/`.

## Customising

| Change | Where |
|---|---|
| apt packages | first `RUN` in `Dockerfile` |
| Node version | NodeSource `setup_22.x` line |
| Extra setup | bottom of `scripts/postCreateCommand.sh` |
| Skill selection | `--skill` flags in the same script |
| Disable GPU | drop `"--device", "nvidia.com/gpu=all"` from `runArgs` |