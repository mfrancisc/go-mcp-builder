# Go MCP Builder Skill

An opinionated skill for building production-ready Model Context Protocol (MCP) servers in Go.

## What This Skill Provides

### Core Skill (`SKILL.md`)
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

## TODO
- [] expand the skill to support adding new tools in existing servers, or refactoring existing servers.
- [] add `/references` with best practices, examples, more testing patterns ( unit/integration testing )
- [] add `/scripts` for project initialization, adding tool to an existing server, creating containerfiles, generating golangci config file