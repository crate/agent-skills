# CrateDB Agent Skills

This is a collection of official CrateDB agent skills which can be used in agentic workflows. 
The collection consists of

* `/cratedb:core` - basic skills about CrateDB
* `/cratedb:mcp-setup` - setting up the [CrateDB MCP server](https://github.com/crate/cratedb-mcp) to access your CrateDB instance
* `/cratedb:datamodel` - assistance to data modeling with CrateDB
* `/cratedb:query` - help to query optimarization

## Installation

### Claude Code

The CrateDB plugin are available on [Anthropic's community marketplace](https://github.com/anthropics/claude-plugins-community).
To get started, you will need to enable the marketplace. The following two commands will enable the marketplace and install the
CrateDB plugin:

```shell
claude plugin marketplace add anthropics/claude-plugins-community
claude plugin install cratedb@claude-community
```

## Configuration

To connect to your CrateDB instance, you can use the [CrateDB MCP server](https://github.com/crate/cratedb-mcp). To set it up, you use the `/cratedb:mcp-setup` skill. 
