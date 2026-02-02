---
name: go-mcp-builder
description: Build Model Context Protocol (MCP) servers in Go, or add tools to existing Go MCP servers. Use when creating new MCP servers, adding tools/resources/prompts to existing servers, or when the user mentions MCP or Model Context Protocol.
---

# Go MCP Builder

Build production-ready Model Context Protocol (MCP) servers in Go following best practices, or enhance existing servers with new tools.

## When to Use

Use this skill when:
- Creating a new MCP server in Go from scratch
- Adding tools, resources, or prompts to an existing Go MCP server
- User mentions "MCP server", "Model Context Protocol"
- Integrating external APIs or services as MCP tools

## Creating a New MCP Server

### 1. Understand the Requirements

**Ask the user these questions if not clear:**
- What service/API will this MCP server integrate with?
- What tools/capabilities should it provide?
- Will it run locally (stdio) or as a remote service (HTTP)?
- Are there authentication requirements (API keys, OAuth)?
- What output format should the tool/server return (JSON/Markdown) ?

### 2. Server Naming Convention

Follow the Go MCP naming pattern:
- Format: `{service}-mcp-server` (lowercase with hyphens)
- Examples: `github-mcp-server`, `slack-mcp-server`, `argocd-mcp-server`
- Must be general, descriptive, and easy to infer from the task

### 3. Project Structure

Create a well-organized project:

```
{service}-mcp-server/
├── main.go                    # Entry point
├── go.mod                     # Go module definition
├── go.sum                     # Dependency checksums
├── README.md                  # Documentation
├── LICENSE                    # Apache 2.0 license
├── Containerfile              # Container image definition
├── .gitignore                 # Git ignore patterns
├── .golangci.yml              # Linter configuration
├── cmd/
│   └── server/
│       └── main.go            # CLI command implementation
├── internal/
│   ├── server/
│   │   └── init.go            # MCP server setup
│   ├── tools/
│   │   ├── tool1.go           # Individual tool implementations
│   │   └── tool2.go
│   ├── client/
│   │   └── api_client.go      # External API client
│   ├── middleware/
│   │   ├── logging.go         # Request/response logging
│   │   ├── metrics.go         # Prometheus metrics
│   │   └── recovery.go        # Panic recovery
│   └── config/
│       └── config.go          # Configuration management
└── test/
    ├── integration/
    │   └── server_test.go     # Integration tests
    └── fixtures/
        └── testdata.json      # Test data
```

### 4. Core Implementation Steps

**Step 1: Initialize the project**

```bash
go mod init github.com/{org}/{service}-mcp-server
go get github.com/modelcontextprotocol/go-sdk/mcp
go get github.com/prometheus/client_golang/prometheus  # Metrics
```

**Step 2: Create the server foundation**

Implement in `internal/server/server.go`:
- Server struct with configuration
- MCP server initialization
- Tool registration
- Middleware setup
- Transport configuration (stdio/HTTP)

**Step 3: Implement tools**

Each tool in `internal/tools/` should:
- Have a clear, action-oriented name with service prefix
- Use struct-based input parameters with validation
- Return properly formatted responses (support JSON or Markdown)
- Handle errors gracefully with helpful messages
- Include comprehensive godoc comments

**Step 4: Add observability**

- Structured logging with `slog` (log to stderr, never stdout for stdio)
- Prometheus metrics for tool calls, errors, latency
- Request/response logging middleware
- Panic recovery middleware

**Step 5: Create CLI and main entry point**

Support command-line flags using the standard `flag` package:
- `--transport` (stdio or http)
- `--listen` (host:port for HTTP)
- `--api-key` or `--token` (authentication)
- `--api-url` (external service URL)
- `--debug` (enable debug logging with source file/line numbers)

### 5. Tool Design Best Practices

**Tool Naming:**
- Use snake_case: `github_create_issue`, `slack_send_message`
- Include service prefix to avoid conflicts
- Be action-oriented: get, list, search, create, update, delete, start, stop

**Tool Structure:**

```go
package tools

import (
    "context"
    "log/slog"
    
    "github.com/google/jsonschema-go/jsonschema"
    "github.com/modelcontextprotocol/go-sdk/mcp"
)

// 1. Define tool with explicit schemas at package level
var SearchTool = &mcp.Tool{
    Name:         "search",
    Description:  "Search for resources",
    InputSchema:  SearchInputSchema,
    OutputSchema: SearchOutputSchema,
}

// 2. Define bare input/output structs (no jsonschema tags)
type SearchInput struct {
    ResourceID string `json:"resource_id"`
    Limit      int    `json:"limit,omitempty"`
    Format     string `json:"format,omitempty"`
}

type SearchOutput struct {
    Results []Resource `json:"results"`
    Total   int        `json:"total"`
}

// 3. Define schemas manually at package level for consistency
//    This approach works for both simple and complex cases
var SearchInputSchema = &jsonschema.Schema{
    Type: "object",
    Properties: map[string]*jsonschema.Schema{
        "resource_id": {
            Type:        "string",
            Description: "Unique identifier for the resource",
        },
        "limit": {
            Type:        "integer",
            Description: "Maximum results to return",
            Minimum:     jsonschema.Ptr(1.0),
            Maximum:     jsonschema.Ptr(100.0),
            Default:     20,
        },
        "format": {
            Type:        "string",
            Description: "Response format",
            Enum:        []any{"json", "markdown"},
            Default:     "markdown",
        },
    },
    Required: []string{"resource_id"},
}

var SearchOutputSchema = &jsonschema.Schema{
    Type: "object",
    Properties: map[string]*jsonschema.Schema{
        "results": {
            Type:        "array",
            Description: "List of matching resources",
            Items: &jsonschema.Schema{
                Type: "object",
            },
        },
        "total": {
            Type:        "integer",
            Description: "Total number of results",
        },
    },
}

// 4. Use `ToolHandlerFor[Input, Output]`, and always return `nil` for `CallToolResult` and let the SDK handle serialization.
// SDK automatically:
// - serializes output to JSON
// - adds to CallToolResult.Content
// - sets CallToolResult.StructuredContent
// Note: Only populate CallToolResult manually if you need custom content types
// (images, markdown, multiple content blocks). 
func SearchToolHandle(logger *slog.Logger, client *Client) mcp.ToolHandlerFor[SearchInput, SearchOutput] {
    return func(ctx context.Context, req *mcp.CallToolRequest, input SearchInput) (*mcp.CallToolResult, SearchOutput, error) {
        
        // Perform operation
        results, err := client.Search(ctx, input.Query)
        if err != nil {
            // SDK automatically formats error message
            return nil, SearchOutput{}, fmt.Errorf("search failed: %w", err)
        }
        
        // Build output
        output := SearchOutput{
            Results: results,
            Total:   len(results),
        }
        
 
        return nil, output, nil
    }
}
```

**Registering Tools:**

```go
package server

import (
    "github.com/modelcontextprotocol/go-sdk/mcp"
    "yourproject/internal/tools"
)

func New(logger *slog.Logger, client *Client) *mcp.Server {
    server := mcp.NewServer(
        &mcp.Implementation{
            Name:    "yourservice-mcp-server",
            Version: "0.1.0",
        },
        &mcp.ServerOptions{
            Logger: logger,
        },
    )
    
    // Register tools using the exported definitions and handler factories
    mcp.AddTool(server, tools.SearchTool, tools.SearchToolHandle(logger, client))
    mcp.AddTool(server, tools.ListTool, tools.ListToolHandle(logger, client))
    
    return server
}
```

### 6. Observability Patterns

**Structured Logging with Debug Flag:**

```go
import (
    "flag"
    "log/slog"
    "os"
)

func main() {
    // Parse flags
    debug := flag.Bool("debug", false, "enable debug logging")
    flag.Parse()
    
    // Initialize logger based on debug flag
    logger := setupLogger(*debug)
    
    // Use logger throughout the application
    logger.Info("server starting", slog.Bool("debug", *debug))
}

func setupLogger(debug bool) *slog.Logger {
    opts := &slog.HandlerOptions{
        Level: slog.LevelInfo,  // Default to Info
    }
    
    if debug {
        opts.Level = slog.LevelDebug
        opts.AddSource = true  // Add source file and line number in debug mode
    }
    
    // Always log to stderr (never stdout for stdio transport!)
    handler := slog.NewJSONHandler(os.Stderr, opts)
    return slog.New(handler)
}

// Log with context
logger.Info("tool called",
    slog.String("tool", "search_items"),
    slog.String("query", input.Query),
    slog.Int("limit", input.Limit),
)

// Debug logs only appear when --debug is set
logger.Debug("detailed operation info",
    slog.Any("params", params),
    slog.String("state", "processing"),
)

// Log errors
logger.Error("API request failed",
    slog.Any("error", err),
    slog.String("endpoint", "/api/v1/items"),
)
```

**Metrics:**

Use middleware to collect metrics centrally for all tools:

```go
package middleware

import (
    "context"
    "strconv"
    "time"
    
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
    "github.com/modelcontextprotocol/go-sdk/mcp"
)

var (
    mcpCallsTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Name: "mcp_calls_total",
            Help: "Total number of MCP calls",
        },
        []string{"server", "method", "tool", "success"},
    )
    
    mcpCallDuration = promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Name: "mcp_call_duration_seconds",
            Help: "Duration of MCP calls in seconds",
            Buckets: prometheus.DefBuckets,
        },
        []string{"server", "method", "tool", "success"},
    )
)

// NewMetricsMiddleware creates middleware that collects metrics for all MCP calls
func NewMetricsMiddleware(serverName string) mcp.Middleware {
    return func(next mcp.MethodHandler) mcp.MethodHandler {
        return func(ctx context.Context, method string, req mcp.Request) (mcp.Result, error) {
            // Start timing
            start := time.Now()
            
            // Call next middleware/handler
            result, err := next(ctx, method, req)
            
            // Calculate duration
            duration := time.Since(start)
            
            // Extract tool name if this is a tool call
            var tool string
            if p, ok := req.GetParams().(*mcp.CallToolParamsRaw); ok {
                tool = p.Name
            }
            
            // Determine success
            success := err == nil
            if r, ok := result.(*mcp.CallToolResult); ok {
                success = success && !r.IsError
            }
            
            // Record metrics
            labels := []string{serverName, method, tool, strconv.FormatBool(success)}
            mcpCallsTotal.WithLabelValues(labels...).Inc()
            mcpCallDuration.WithLabelValues(labels...).Observe(duration.Seconds())
            
            return result, err
        }
    }
}

// Register middleware in server initialization
func New(logger *slog.Logger, client *Client) *mcp.Server {
    server := mcp.NewServer(
        &mcp.Implementation{
            Name:    "myservice-mcp-server",
            Version: "0.1.0",
        },
        &mcp.ServerOptions{
            Logger: logger,
        },
    )
    
    // Add metrics middleware (applies to all tools automatically)
    server.AddReceivingMiddleware(NewMetricsMiddleware("myservice"))
    
    // Register tools...
    mcp.AddTool(server, myservice.SearchTool, myservice.SearchToolHandle(logger, client))
    
    return server
}
```

**HTTP Transport and Metrics:**

For HTTP transport, expose both MCP and metrics on the same server:

```go
package main

import (
    "context"
    "encoding/json"
    "flag"
    "fmt"
    "log/slog"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"
    
    "github.com/modelcontextprotocol/go-sdk/mcp"
    "github.com/prometheus/client_golang/prometheus/promhttp"
    "yourproject/internal/server"
)

func main() {
    // Parse flags
    var (
        transport = flag.String("transport", "http", "Transport mode: 'http' or 'stdio'")
        listen    = flag.String("listen", "127.0.0.1:8080", "Listen address for HTTP transport")
        debug     = flag.Bool("debug", false, "Enable debug logging")
    )
    flag.Parse()
    
    // Validate transport
    if *transport != "http" && *transport != "stdio" {
        fmt.Fprintf(os.Stderr, "Error: invalid transport %q, must be 'http' or 'stdio'\n", *transport)
        os.Exit(1)
    }
    
    // Setup logger
    logger := setupLogger(*debug)
    logger.Info("starting MCP server",
        slog.String("transport", *transport),
        slog.String("listen", *listen),
        slog.Bool("debug", *debug),
    )
    
    // Setup graceful shutdown
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    
    sigChan := make(chan os.Signal, 1)
    signal.Notify(sigChan, os.Interrupt, syscall.SIGTERM)
    
    go func() {
        sig := <-sigChan
        logger.Info("shutdown signal received", slog.String("signal", sig.String()))
        cancel()
    }()
    
    // Create MCP server
    srv := server.New(logger, client)
    
    // Run based on transport mode
    switch *transport {
    case "stdio":
        // Stdio transport - no metrics endpoint
        if err := srv.Run(ctx, &mcp.StdioTransport{}); err != nil {
            logger.Error("server failed", slog.Any("error", err))
            os.Exit(1)
        }
        logger.Info("server stopped gracefully")
        
    case "http":
        // HTTP transport - serve MCP and metrics on same server
        mux := http.NewServeMux()
        
        // MCP endpoint at /mcp
        mux.Handle("/mcp", mcp.NewStreamableHTTPHandler(
            func(*http.Request) *mcp.Server { return srv },
            &mcp.StreamableHTTPOptions{},
        ))
        
        // Metrics endpoint at /metrics (only for HTTP transport)
        mux.Handle("/metrics", promhttp.Handler())
        
        // Health check endpoint
        mux.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
            w.Header().Set("Content-Type", "application/json")
            json.NewEncoder(w).Encode(map[string]string{
                "status": "healthy",
                "time":   time.Now().Format(time.RFC3339),
            })
        })
        
        // Configure HTTP server
        httpServer := &http.Server{
            Addr:         *listen,
            Handler:      mux,
            ReadTimeout:  15 * time.Second,
            WriteTimeout: 15 * time.Second,
            IdleTimeout:  60 * time.Second,
        }
        
        logger.Info("HTTP server starting",
            slog.String("mcp_endpoint", "http://"+*listen+"/mcp"),
            slog.String("metrics_endpoint", "http://"+*listen+"/metrics"),
            slog.String("health_endpoint", "http://"+*listen+"/health"),
        )
        
        // Run HTTP server with graceful shutdown
        errChan := make(chan error, 1)
        go func() {
            if err := httpServer.ListenAndServe(); err != nil && err != http.ErrServerClosed {
                errChan <- err
            }
        }()
        
        // Wait for shutdown signal or error
        select {
        case err := <-errChan:
            logger.Error("HTTP server failed", slog.Any("error", err))
            os.Exit(1)
            
        case <-ctx.Done():
            logger.Info("shutting down HTTP server")
            
            // Create shutdown context with timeout
            shutdownCtx, shutdownCancel := context.WithTimeout(context.Background(), 5*time.Second)
            defer shutdownCancel()
            
            // Attempt graceful shutdown
            if err := httpServer.Shutdown(shutdownCtx); err != nil {
                logger.Error("shutdown failed", slog.Any("error", err))
                os.Exit(1)
            }
            
            logger.Info("server stopped gracefully")
        }
    }
}

func setupLogger(debug bool) *slog.Logger {
    opts := &slog.HandlerOptions{
        Level: slog.LevelInfo,
    }
    if debug {
        opts.Level = slog.LevelDebug
        opts.AddSource = true
    }
    
    handler := slog.NewJSONHandler(os.Stderr, opts)
    return slog.New(handler)
}
```

**Usage:**

```bash
# HTTP mode with metrics
./myservice-mcp-server --transport http --listen 127.0.0.1:8080

# Stdio mode (no metrics endpoint)
./myservice-mcp-server --transport stdio

# HTTP with debug logging
./myservice-mcp-server --transport http --listen :8080 --debug
```

**Endpoints (HTTP mode only):**
- `http://localhost:8080/mcp` - MCP server endpoint
- `http://localhost:8080/metrics` - Prometheus metrics

### 7. Testing

**Unit Tests:**

```go
func TestToolHandler(t *testing.T) {
    tests := []struct {
        name    string
        input   ToolInput
        want    string
        wantErr bool
    }{
        {
            name:  "valid input",
            input: ToolInput{Query: "test", Limit: 10},
            want:  "expected result",
        },
        {
            name:    "invalid input",
            input:   ToolInput{},
            wantErr: true,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result, err := HandleTool(context.Background(), tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("HandleTool() error = %v, wantErr %v", err, tt.wantErr)
            }
            // Additional assertions...
        })
    }
}
```

**Integration Tests:**

Test with a real MCP client connecting to your server.

### 8. Documentation

**README.md should include:**
- Overview and features
- Installation instructions
- Configuration options
- Usage examples (stdio and HTTP transports)
- Authentication setup
- Development guide
- Testing instructions
- License

**Tool Documentation:**

Each tool must have comprehensive godoc:

```go
// SearchItems searches for items matching the given query.
//
// This tool searches across all items in the system, supporting partial matches
// and various filters.
//
// Input Parameters:
//   - query (string, required): Search string to match against items
//   - format (string, optional): Response format - "json" or "markdown" (default: "markdown")
//
// Returns:
//   - JSON format: Structured data 
//   - Markdown format: Human-readable formatted list with headers and details
//
// Errors:
//   - Returns error result if authentication fails
//   - Returns error result if rate limit exceeded
//   - Returns empty results if no matches found
func SearchItems(ctx context.Context, req *mcp.CallToolRequest) (*mcp.CallToolResult, error) {
    // Implementation...
}
```