# CrateDB MCP Server Setup

## Prerequisites

- Python 3.10 or newer
- [uv](https://docs.astral.sh/uv/) package manager (`brew install uv` / `pip install uv`)
- A running CrateDB instance (local or cloud)

## Installation

### Ephemeral (recommended for AI assistant config)

No install needed — `uvx` downloads and runs on first use:

```shell
uvx cratedb-mcp serve
```

### Persistent

```shell
uv tool install --upgrade cratedb-mcp
cratedb-mcp serve
```

### Docker / Podman

```shell
docker run --rm -i \
  -e CRATEDB_CLUSTER_URL=http://crate:crate@localhost:4200/ \
  ghcr.io/crate/cratedb-mcp:0.1.1
```

The credentials `crate:crate` should not be used in real-world deployment and is showed here for illustration. Do not put credentials in URLs committed to config files.

## Configuration

| Environment variable | Default | Description |
|---|---|---|
| `CRATEDB_CLUSTER_URL` | `http://localhost:4200` | CrateDB REST endpoint with credentials |
| `CRATEDB_MCP_TRANSPORT` | `stdio` | Transport: `stdio`, `sse`, `http`, `streamable-http` |
| `CRATEDB_MCP_HOST` | `127.0.0.1` | Listening host (HTTP transports only) |
| `CRATEDB_MCP_PORT` | `8000` | Listening port (HTTP transports only) |
| `CRATEDB_MCP_PATH` | — | URL path override (HTTP transports only) |
| `CRATEDB_MCP_HTTP_TIMEOUT` | `30.0` | HTTP request timeout in seconds |
| `CRATEDB_MCP_DOCS_CACHE_TTL` | `3600` | Documentation cache lifetime in seconds |
| `CRATEDB_MCP_PERMIT_ALL_STATEMENTS` | `false` | Allow non-SELECT SQL (write operations) |
| `CRATEDB_MCP_INSTRUCTIONS` | — | Path/URL to replace the default system prompt |
| `CRATEDB_MCP_CONVENTIONS` | — | Path/URL to append custom conventions to the system prompt |

### CrateDB Cloud connection

```shell
export CRATEDB_CLUSTER_URL="https://<user>:<password>@<host>.cratedb.net:4200/"
```

## Starting the server

```shell
# stdio (default — for AI assistant clients)
cratedb-mcp serve

# HTTP transport (for network-accessible deployment)
cratedb-mcp serve --transport=http --host=127.0.0.1 --port=8000

# View the built-in system prompt
cratedb-mcp show-prompt
```

## Integrating with AI assistants

### Claude Desktop / Cline / Cursor / Roo Code / Windsurf

Add to your MCP config file:

```json
{
  "mcpServers": {
    "cratedb-mcp": {
      "command": "uvx",
      "args": ["cratedb-mcp", "serve"],
      "env": {
        "CRATEDB_CLUSTER_URL": "http://localhost:4200/",
        "CRATEDB_MCP_TRANSPORT": "stdio"
      },
      "alwaysAllow": [
        "get_cluster_health",
        "get_table_metadata",
        "get_cratedb_documentation_index",
        "fetch_cratedb_docs"
      ]
    }
  }
}
```

Config file locations:
- **Claude Desktop**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Cline**: `cline_mcp_settings.json`
- **Cursor**: `~/.cursor/mcp.json` or `.cursor/mcp.json`
- **Roo Code**: `mcp_settings.json` or `.roo/mcp.json`
- **Windsurf**: `~/.codeium/windsurf/mcp_config.json`

### VS Code

In `settings.json` (user-wide):

```json
{
  "mcp": {
    "servers": {
      "cratedb-mcp": {
        "command": "uvx",
        "args": ["cratedb-mcp", "serve"],
        "env": {
          "CRATEDB_CLUSTER_URL": "http://localhost:4200/",
          "CRATEDB_MCP_TRANSPORT": "stdio"
        }
      }
    }
  },
  "chat.mcp.enabled": true
}
```

### Goose

In `~/.config/goose/config.yaml`:

```yaml
extensions:
  cratedb-mcp:
    name: CrateDB MCP
    type: stdio
    cmd: uvx
    args: [cratedb-mcp, serve]
    enabled: true
    envs:
      CRATEDB_CLUSTER_URL: "http://localhost:4200/"
      CRATEDB_MCP_TRANSPORT: "stdio"
    timeout: 300
```

### Open WebUI (via mcpo)

```shell
export CRATEDB_CLUSTER_URL=http://crate:crate@localhost:4200/
docker run --rm --name=cratedb-mcpo -p 8000:8000 \
  -e CRATEDB_CLUSTER_URL \
  ghcr.io/crate/cratedb-mcpo:latest
```

The credentials `crate:crate` should not be used in real-world deployment and is showed here for illustration. Do not put credentials in URLs committed to config files.

## Available tools

| Tool | Description |
|---|---|
| `query_sql` | Execute a SELECT query on CrateDB |
| `get_table_columns` | Return column schema for all tables |
| `get_table_metadata` | Return table metadata (shards, replicas, record counts) |
| `get_cratedb_documentation_index` | Return the CrateDB documentation table of contents |
| `fetch_cratedb_docs` | Fetch a specific CrateDB documentation page |
| `get_cluster_health` | Return cluster health from `sys.health` |

## Security

By default only `SELECT` statements are permitted. To enforce read-only access
at the database level, create a dedicated user:

```sql
CREATE USER "read-only" WITH (password = 'YOUR_PASSWORD');
GRANT DQL TO "read-only";
```

Then set `CRATEDB_CLUSTER_URL` to use those credentials. To allow write
operations, set `CRATEDB_MCP_PERMIT_ALL_STATEMENTS=true` (use with caution).

## Custom system prompt

```shell
# Export the built-in prompt, edit it, then reload
cratedb-mcp show-prompt > my-instructions.md
cratedb-mcp serve --instructions=my-instructions.md

# Or extend without replacing
cratedb-mcp serve --conventions=my-conventions.md
```

Both `--instructions` and `--conventions` accept file paths, HTTP(S) URLs, or `-` for stdin.
