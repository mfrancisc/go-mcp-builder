# Go MCP Builder Skill

An opinionated skill for building production-ready Model Context Protocol (MCP) servers in Go.

## What This Skill Provides

- Complete guide for building new MCP servers
- Step-by-step instructions for adding tools to existing servers
- Project structure and organization patterns
- Tool design best practices
- Observability patterns (logging, metrics, middleware)

## Usage

The skill is automatically available when building MCP servers. It activates when you:

- Mention "MCP server", "Model Context Protocol", or "Go server"
- Ask to create a new MCP server
- Request to add tools to an existing MCP server
- Need help with MCP server patterns or best practices

## Technologies Used

- **Go SDK**: `github.com/modelcontextprotocol/go-sdk/mcp`
- **Logging**: `log/slog` (Go 1.21+ standard library)
- **Metrics**: `github.com/prometheus/client_golang/prometheus`
- **HTTP Client**: Standard library with connection pooling
- **Testing**: Standard library with table-driven tests

## Installation

### Option 1: Global Installation (Recommended)

Install the skill globally to use it across all your Cursor workspaces:

```bash
git clone https://github.com/codeready-toolchain/go-mcp-builder \
    ~/.cursor/skills/go-mcp-builder
```

### Option 2: Workspace-Local Installation

Install the skill in a specific workspace:

```bash
cd your-mcp-project/
git clone https://github.com/codeready-toolchain/go-mcp-builder \
    .cursor/skills/go-mcp-builder
```

## TODO
- [] expand the skill to support adding new tools in existing servers, or refactoring existing servers.
- [] add `/references` with best practices, examples, more testing patterns ( unit/integration testing )
- [] add `/scripts` for project initialization, adding tool to an existing server, creating containerfiles, generating golangci config file
