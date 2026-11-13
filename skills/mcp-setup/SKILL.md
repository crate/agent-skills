# CrateDB MCP Server Setup

## Prerequisites

- Python 3.10 or newer
- [uv](https://docs.astral.sh/uv/) package manager (`brew install uv` / `pip install uv`)
- A running CrateDB instance (local or cloud)
- `cratedb-mcp` 0.2.0 or newer

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
  ghcr.io/crate/cratedb-mcp:0.2.0
```

The credentials `crate:crate` should not be used in real-world deployment and are shown here for illustration. Do not put credentials in URLs committed to config files.

## Configuration

| Environment variable | Default | Description |
|---|---|---|
| `CRATEDB_CLUSTER_URL` | `http://localhost:4200` | CrateDB REST endpoint with credentials |
| `CRATEDB_MCP_TRANSPORT` | `stdio` | Transport: `stdio`, `sse`, `http`, `streamable-http` |
| `CRATEDB_MCP_HOST` | `127.0.0.1` | Listening host (HTTP transports only) |
| `CRATEDB_MCP_PORT` | `8000` | Listening port (HTTP transports only) |
| `CRATEDB_MCP_PATH` | — | URL path override (HTTP transports only) |
| `CRATEDB_MCP_AUTH_TOKEN` | — | Bearer token that HTTP clients must send; required over HTTP unless auth is disabled |
| `CRATEDB_MCP_AUTH_DISABLED` | `false` | Set to `true` to serve over HTTP without a token (local work only) |
| `CRATEDB_MCP_ALLOWED_HOSTS` | — | Space-separated host names or IP addresses, besides localhost, that clients use to reach the server (HTTP transports only); other hosts get 421 |
| `CRATEDB_MCP_ALLOWED_ORIGINS` | — | Space-separated browser origins that may call the server (HTTP transports only); other origins get 403 |
| `CRATEDB_MCP_MAX_ROWS` | `100` | Default row limit of `query_sql`; a call can ask for up to 1,000 |
| `CRATEDB_MCP_HTTP_TIMEOUT` | `30.0` | HTTP request timeout in seconds |
| `CRATEDB_MCP_DOCS_CACHE_TTL` | `3600` | Documentation cache lifetime in seconds |
| `CRATEDB_MCP_PERMIT_ALL_STATEMENTS` | `false` | Allow any SQL statement (write operations) |
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
export CRATEDB_MCP_AUTH_TOKEN="$(openssl rand -hex 32)"
cratedb-mcp serve --transport=http --host=127.0.0.1 --port=8000

# View the built-in system prompt
cratedb-mcp show-prompt
```

Over HTTP the server refuses to start without `CRATEDB_MCP_AUTH_TOKEN`. For local
work only, set `CRATEDB_MCP_AUTH_DISABLED=true` instead. Clients connect to
`http://127.0.0.1:8000/mcp` and send the token as `Authorization: Bearer <token>`.

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
        "list_schemas",
        "list_tables",
        "describe_table",
        "analyze_table_layout",
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

### Open WebUI

Open WebUI 0.6.31 and later connect to the standard image. Run it on the Docker network
that Open WebUI and CrateDB share (here `demo`, with CrateDB as `cratedb`):

```shell
export CRATEDB_MCP_AUTH_TOKEN="$(openssl rand -hex 32)"
docker run --rm --name=cratedb-mcp --network=demo \
  -e CRATEDB_MCP_TRANSPORT=streamable-http \
  -e CRATEDB_MCP_HOST=0.0.0.0 \
  -e CRATEDB_MCP_ALLOWED_HOSTS=cratedb-mcp \
  -e CRATEDB_MCP_AUTH_TOKEN \
  -e CRATEDB_CLUSTER_URL=http://crate:crate@cratedb:4200/ \
  ghcr.io/crate/cratedb-mcp:0.2.0
```

In Open WebUI, add a connection of type "MCP (Streamable HTTP)" with the URL
`http://cratedb-mcp:8000/mcp`. Choose bearer authentication and enter the token
(`echo "$CRATEDB_MCP_AUTH_TOKEN"` prints it). `CRATEDB_MCP_ALLOWED_HOSTS` lists the
container name because Open WebUI reaches the server by that name.

The credentials `crate:crate` should not be used in real-world deployment and are shown here for illustration. Do not put credentials in URLs committed to config files.

## Available tools

| Tool | Description |
|---|---|
| `query_sql` | Execute a read query on CrateDB; returns at most 100 rows unless `max_rows` asks for more (up to 1,000), and marks a cut result with `truncated` |
| `explain_query` | Show how CrateDB plans or runs a SELECT query |
| `list_schemas` | List the schemas that hold tables or views |
| `list_tables` | List the tables and views of one schema |
| `describe_table` | Describe one table or view: columns, types, keys, sharding and partitioning |
| `analyze_table_layout` | Report shards, shard sizes and partitions of tables, with findings against sharding guidance |
| `get_cratedb_documentation_index` | Return the CrateDB documentation table of contents |
| `fetch_cratedb_docs` | Fetch a specific CrateDB documentation page |
| `get_cluster_health` | Return cluster health from `sys.health` |
| `diagnose_workload` | List slow, failed or running statements, or the latencies per statement type |

## Security

By default only one read statement per call is permitted: `SELECT`, `WITH`, `VALUES`, `SHOW`, or an `EXPLAIN` of a `SELECT`, `WITH` or `VALUES` statement. To enforce read-only access
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
