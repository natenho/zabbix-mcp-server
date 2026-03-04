# UVX Support Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Enable execution via `uvx --from git+https://github.com/mpeirone/zabbix-mcp-server zabbix-mcp` without cloning the repository.

**Architecture:** Rename the `src/` package to `zabbix_mcp/`, update the entry point in `pyproject.toml`, and update all references to the old path in docs, scripts, and Dockerfile.

**Tech Stack:** Python packaging (setuptools), uv/uvx

---

### Task 1: Create working branch

**Files:**
- No files modified

**Step 1: Create and checkout the branch**

```bash
git checkout -b feature/uvx-support
```

Expected: branch created and active.

**Step 2: Confirm active branch**

```bash
git branch --show-current
```

Expected: `feature/uvx-support`

---

### Task 2: Rename package src → zabbix_mcp

**Files:**
- Rename: `src/` → `zabbix_mcp/`

**Step 1: Rename the directory**

```bash
git mv src zabbix_mcp
```

Expected: directory renamed, git tracks the rename.

**Step 2: Verify structure**

```bash
ls zabbix_mcp/
```

Expected: `__init__.py  zabbix_mcp_server.py`

---

### Task 3: Update pyproject.toml

**Files:**
- Modify: `pyproject.toml`

**Step 1: Update entry point and add package discovery**

Update the `[project.scripts]` block:
```toml
[project.scripts]
zabbix-mcp = "zabbix_mcp.zabbix_mcp_server:main"
```

Add package discovery configuration just before `[build-system]`:
```toml
[tool.setuptools.packages.find]
where = ["."]
include = ["zabbix_mcp*"]
```

**Step 2: Verify local installation**

```bash
uv pip install -e . --quiet
```

Expected: installation without errors.

**Step 3: Verify entry point**

```bash
uv run zabbix-mcp --help 2>&1 || true
```

Expected: runs without `ModuleNotFoundError`.

---

### Task 4: Update Dockerfile

**Files:**
- Modify: `Dockerfile`

**Step 1: Update COPY and CMD**

Line 15 — change `COPY src/ ./src/` to:
```dockerfile
COPY zabbix_mcp/ ./zabbix_mcp/
```

Line 27 (CMD) — change to use the installed entry point:
```dockerfile
CMD ["python", "-c", "import sys; sys.path.insert(0, '.'); from zabbix_mcp.zabbix_mcp_server import mcp; mcp.run(transport='sse', host='0.0.0.0', port=8000)"]
```

---

### Task 5: Update scripts/start_server.py

**Files:**
- Modify: `scripts/start_server.py:23`

**Step 1: Fix sys.path.insert**

Line 23 — change:
```python
sys.path.insert(0, str(Path(__file__).parent.parent / "src"))
```
To:
```python
sys.path.insert(0, str(Path(__file__).parent.parent))
```

Line 177 — the import `from zabbix_mcp_server import main` must be:
```python
from zabbix_mcp.zabbix_mcp_server import main as server_main
```

---

### Task 6: Update scripts/test_server.py

**Files:**
- Modify: `scripts/test_server.py:264`

**Step 1: Fix path reference**

Find the line with `src/zabbix_mcp_server.py` and update to:
```python
print("2. Start the server: uvx --from git+https://github.com/mpeirone/zabbix-mcp-server zabbix-mcp")
```

---

### Task 7: Update config/mcp.json

**Files:**
- Modify: `config/mcp.json`

**Step 1: Replace configuration with uvx**

Replace the content with the uvx format:
```json
{
  "mcpServers": {
    "zabbix": {
      "type": "stdio",
      "command": "uvx",
      "args": [
        "--from",
        "git+https://github.com/mpeirone/zabbix-mcp-server",
        "zabbix-mcp"
      ],
      "env": {
        "ZABBIX_URL": "https://zabbix.example.com",
        "ZABBIX_TOKEN": "<your_api_token>",
        "READ_ONLY": "true"
      }
    }
  }
}
```

---

### Task 8: Update MCP_SETUP.md

**Files:**
- Modify: `MCP_SETUP.md`

**Step 1: Rewrite with uvx as primary method**

New file content:

```markdown
# MCP Client Setup

## Recommended Setup (uvx)

The simplest way — no need to clone the repository:

```json
{
  "mcpServers": {
    "zabbix": {
      "type": "stdio",
      "command": "uvx",
      "args": [
        "--from",
        "git+https://github.com/mpeirone/zabbix-mcp-server",
        "zabbix-mcp"
      ],
      "env": {
        "ZABBIX_URL": "https://zabbix.example.com",
        "ZABBIX_TOKEN": "<your_api_token>"
      }
    }
  }
}
```

Replace `ZABBIX_URL` and `ZABBIX_TOKEN` with your environment values.

### Alternative Authentication

```json
"env": {
  "ZABBIX_URL": "https://zabbix.example.com",
  "ZABBIX_USER": "<username>",
  "ZABBIX_PASSWORD": "<password>"
}
```

### Optional Variables

- `READ_ONLY` — `true` to block write operations
- `VERIFY_SSL` — `false` to disable SSL verification
- `DEBUG` — `1` for detailed logging

---

## Alternative Setup (local repository)

For those who cloned the repository:

```json
{
  "mcpServers": {
    "zabbix": {
      "command": "uv",
      "args": [
        "run",
        "--directory",
        "/path/to/zabbix-mcp-server",
        "python",
        "zabbix_mcp/zabbix_mcp_server.py"
      ],
      "env": {
        "ZABBIX_URL": "https://zabbix.example.com",
        "ZABBIX_TOKEN": "<your_api_token>"
      }
    }
  }
}
```

Update `--directory` with the actual project path.

---

## Troubleshooting

**Server does not start:**
- Check if `uvx` is installed: `uvx --version`
- Verify `ZABBIX_URL` is correct and reachable

**Authentication error:**
- Confirm the token or credentials in Zabbix
- Check if the Zabbix API is enabled

**Debug:**
```json
"env": {
  "DEBUG": "1"
}
```

**Manual test:**
```bash
ZABBIX_URL=https://zabbix.example.com ZABBIX_TOKEN=xxx uvx --from git+https://github.com/mpeirone/zabbix-mcp-server zabbix-mcp
```
```

---

### Task 9: Update README.md

**Files:**
- Modify: `README.md`

**Step 1: Add uvx section as main Quick Start**

Replace the `## Installation` section with:

```markdown
## Installation

### Recommended method: uvx (no cloning required)

Requires only [uv](https://docs.astral.sh/uv/) installed:

```bash
uvx --from git+https://github.com/mpeirone/zabbix-mcp-server zabbix-mcp
```

Configure via environment variables before running, or use directly in the MCP configuration (see [MCP_SETUP.md](MCP_SETUP.md)).

### Alternative method: clone the repository

<details>
<summary>Expand instructions</summary>

1. **Clone the repository:**
   ```bash
   git clone https://github.com/mpeirone/zabbix-mcp-server.git
   cd zabbix-mcp-server
   ```

2. **Install dependencies:**
   ```bash
   uv sync
   ```

3. **Configure environment variables:**
   ```bash
   cp config/.env.example .env
   # Edit .env with your Zabbix settings
   ```

4. **Test the installation:**
   ```bash
   uv run python scripts/test_server.py
   ```

</details>

### Prerequisites

- [uv](https://docs.astral.sh/uv/) installed
- Access to a Zabbix server with API enabled
```

**Step 2: Update the "Running the Server" section**

Replace the usage block with:

```markdown
### Running the Server

**Via uvx (recommended):**
```bash
ZABBIX_URL=https://zabbix.example.com ZABBIX_TOKEN=xxx uvx --from git+https://github.com/mpeirone/zabbix-mcp-server zabbix-mcp
```

**With local repository:**
```bash
uv run python zabbix_mcp/zabbix_mcp_server.py
```
```

**Step 3: Update "Project Structure" section**

Update the block:
```
├── src/
│   └── zabbix_mcp_server.py    # Main server implementation
```
To:
```
├── zabbix_mcp/
│   └── zabbix_mcp_server.py    # Main server implementation
```

---

### Task 10: Final commit

**Step 1: Check modified files**

```bash
git status
```

**Step 2: Stage and commit**

```bash
git add -A
git commit -m "feat: add uvx support by renaming src to zabbix_mcp package"
```

Expected: commit created on branch `feature/uvx-support`.
