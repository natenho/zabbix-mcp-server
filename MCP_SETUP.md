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
