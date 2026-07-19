# devin-template

A Dev Container with **Devin CLI**, **Claude Code**, Node 22, Python 3, and `gh` — plus automatic handling of corporate TLS-interception proxies.

<https://github.com/ravikiranvs/devin-template>

## Use it

**New project:** click **Use this template** on GitHub, open in VS Code, and accept *Reopen in Container*.

**Existing repo:**

```bash
npm install -g @devcontainers/cli
```
```bash
$env:NODE_OPTIONS = "--use-system-ca"
devcontainer templates apply --template-id ghcr.io/ravikiranvs/devin-template/devin-workspace:latest --workspace-folder .
```

Then open the folder in VS Code and reopen in the container.

## First run

```bash
devin setup     # authenticate the Devin CLI
```

Then in a Devin session, once per repo: `/setup-matt-pocock-skills`

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
| Probe a different host | `"containerEnv": { "CORP_PROBE_HOST": "host:443" }` |

## Troubleshooting

| Symptom | Fix |
|---|---|
| `postCreateCommand failed with exit code 127` | Wrong script path, or CRLF endings — keep `.gitattributes` |
| `SSL certificate problem: self-signed certificate` | Run `sudo sh .devcontainer/scripts/install-corp-ca.sh` to see why |
| `npm` fails TLS but `git` works | `NODE_EXTRA_CA_CERTS` must be set — check the `ENV` block |
| `COPY failed: file not found` | `"context": ".."` is required under `build` |
| Create hangs with no output | An interactive prompt with no TTY — add `--yes` |

Check the CA state inside the container:

```bash
openssl s_client -connect github.com:443 </dev/null 2>/dev/null | grep 'Verify return code'
```
