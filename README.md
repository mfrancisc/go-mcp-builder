# Go MCP Builder Skill

An opinionated skill for building production-ready Model Context Protocol (MCP) servers in Go.

## What This Skill Provides

- Complete guide for building new MCP servers
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
git clone https://github.com/mfrancisc/go-mcp-builder \
    ~/.cursor/skills/go-mcp-builder
```

### Option 2: Workspace-Local Installation

Install the skill in a specific workspace:

```bash
cd your-mcp-project/
git clone https://github.com/mfrancisc/go-mcp-builder \
    .cursor/skills/go-mcp-builder
```

## Roadmap

### Coming Soon

- [ ] **Tool Management** - Support for adding new tools to existing servers and refactoring existing servers
- [ ] **Reference Documentation** - Comprehensive guides in `/references`:
  - [ ] Best practices for production MCP servers
  - [ ] Complete working examples
  - [ ] Testing patterns (unit, integration, benchmarks)
  - [ ] Quick reference guide
- [ ] **Automation Scripts** - Helper scripts in `/scripts`:
  - [ ] Project initialization (`init-project.sh`)
  - [ ] Tool generation (`add-tool-to-server.sh`)
  - [ ] Containerfile generation
  - [ ] Complete GH action configuration generation
