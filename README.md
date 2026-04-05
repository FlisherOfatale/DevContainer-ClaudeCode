# Claude Code Devcontainer Template

A ready-to-use VS Code devcontainer template for working with Claude Code. Provides a sandboxed Docker environment with authentication persistence, the Claude Code extension pre-installed, and sensible defaults.

---

## What's Included

```
.devcontainer/
  devcontainer.json    # Container config, mounts, extensions
  Dockerfile           # Node 20 + Claude Code + dev tools
  post-create.sh       # Runs once after container creation
CLAUDE.md              # Project context file (read by Claude automatically)
.claudeignore          # Files Claude should skip
.gitignore
```

---

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (macOS/Windows) or Docker Engine (Linux)
- [VS Code](https://code.visualstudio.com/)
- [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) for VS Code
- An Anthropic account ([claude.ai](https://claude.ai) — Pro/Max) or an API key

---

## Quick Start

1. **Copy this template into your project:**
   ```bash
   cp -r /path/to/this-template/.devcontainer your-project/
   cp /path/to/this-template/CLAUDE.md your-project/
   cp /path/to/this-template/.claudeignore your-project/
   ```

2. **Open the project in VS Code:**
   ```bash
   code your-project/
   ```

3. **Reopen in Container:**
   - Click "Reopen in Container" when prompted, **or**
   - `Cmd/Ctrl+Shift+P` → `Dev Containers: Reopen in Container`

4. **Authenticate** (first time only — see Authentication section below).

5. **Start Claude Code:**
   ```bash
   claude
   # or, inside the container sandbox:
   claude --dangerously-skip-permissions
   ```

---

## Authentication

Authentication credentials live in `~/.claude` and `~/.claude.json`. The devcontainer bind-mounts these from your host so you never have to re-authenticate after a container rebuild.

### Option A — OAuth (Pro / Max subscribers, recommended)

1. **Authenticate on your host machine once:**
   ```bash
   # On your host (outside the container):
   claude
   # Follow the browser OAuth flow to log in.
   ```
   This writes credentials to `~/.claude` on your host.

2. Open the devcontainer — credentials are automatically available via the bind-mount.

### Option B — API Key

Set your key in your host shell before opening the container:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
```

Or uncomment the `remoteEnv` block in `devcontainer.json`:
```json
"remoteEnv": {
  "ANTHROPIC_API_KEY": "${localEnv:ANTHROPIC_API_KEY}"
}
```

> ⚠️ Never commit API keys to version control. Use environment variables or a secrets manager.

### Verifying Auth

The `post-create.sh` script prints your auth status in the terminal when the container starts. You can also check manually:
```bash
cat ~/.claude.json | jq 'keys'
```

---

## Using `--dangerously-skip-permissions`

Inside a devcontainer, Claude is already sandboxed — it can only access what you've mounted. This makes it safe to bypass per-action permission prompts:

```bash
claude --dangerously-skip-permissions
```

> **Note:** While the container isolates your host filesystem, if you mount `~/.gitconfig` or SSH keys, Claude could still perform git operations. See the security notes below.

---

## Customization

### Add VS Code Extensions

Edit the `extensions` array in `devcontainer.json`:
```json
"extensions": [
  "Anthropic.claude-code",
  "ms-python.python",
  "dbaeumer.vscode-eslint"
]
```

### Change the Base Language

The Dockerfile uses Node 20. To switch to Python:
```dockerfile
FROM mcr.microsoft.com/devcontainers/python:3.12
```
Then add Node via the `features` block in `devcontainer.json` (Claude Code requires Node 18+):
```json
"features": {
  "ghcr.io/devcontainers/features/node:1": { "version": "20" }
}
```

### Add Services (e.g. a Database)

Create a `docker-compose.yml` in `.devcontainer/`, then update `devcontainer.json`:
```json
"dockerComposeFile": "docker-compose.yml",
"service": "app",
"workspaceFolder": "/workspace"
```

### Project Context for Claude

Edit `CLAUDE.md` to describe your project. Claude reads this file automatically at the start of every session. Good things to include:
- What the project does
- Key commands (`npm test`, `make build`, etc.)
- Architecture notes and file conventions
- Things Claude should avoid touching

---

## Security Notes

- The container **cannot** access your host filesystem beyond what you explicitly mount.
- Git credentials are mounted **read-only** by default.
- If you need to prevent Claude from making git pushes, remove the `~/.gitconfig` mount and set `GIT_TERMINAL_PROMPT=0` in `remoteEnv`.
- For untrusted repositories, use a named Docker volume for Claude credentials (Option B) instead of the host bind-mount, so session tokens stay isolated.
- The `--dangerously-skip-permissions` flag is safe inside the container but should never be used on a bare host machine.

---

## Troubleshooting

**"Authentication required" every container rebuild**
The bind-mount path may not exist on your host. Run `claude` on your host once to create `~/.claude`, then rebuild.

**Container fails to build**
Check Docker is running: `docker info`. On Windows, ensure WSL 2 is enabled.

**OAuth browser window doesn't open inside the container**
This is expected. Authenticate on your host first (Option A), or use an API key (Option B).

**Claude Code not found after rebuild**
The `npm install -g` in the Dockerfile runs at build time. If you change the Dockerfile, rebuild the container: `Cmd/Ctrl+Shift+P` → `Dev Containers: Rebuild Container`.
