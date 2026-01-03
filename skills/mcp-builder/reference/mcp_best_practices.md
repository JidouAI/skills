# MCP Server Best Practices

## Quick Reference

### Server Naming
- **Python**: `{service}_mcp` (e.g., `slack_mcp`)
- **Node/TypeScript**: `{service}-mcp-server` (e.g., `slack-mcp-server`)

### Tool Naming
- Use snake_case with service prefix
- Format: `{service}_{action}_{resource}`
- Example: `slack_send_message`, `github_create_issue`

### Response Formats
- Support both JSON and Markdown formats
- JSON for programmatic processing
- Markdown for human readability

### Pagination
- Always respect `limit` parameter
- Return `has_more`, `next_offset`, `total_count`
- Default to 20-50 items

### Transport
- **Streamable HTTP**: For remote servers, multi-client scenarios
- **stdio**: For local integrations, command-line tools
- Avoid SSE (deprecated in favor of streamable HTTP)

---

## Server Naming Conventions

Follow these standardized naming patterns:

**Python**: Use format `{service}_mcp` (lowercase with underscores)
- Examples: `slack_mcp`, `github_mcp`, `jira_mcp`

**Node/TypeScript**: Use format `{service}-mcp-server` (lowercase with hyphens)
- Examples: `slack-mcp-server`, `github-mcp-server`, `jira-mcp-server`

The name should be general, descriptive of the service being integrated, easy to infer from the task description, and without version numbers.

---

## Tool Naming and Design

### Tool Naming

1. **Use snake_case**: `search_users`, `create_project`, `get_channel_info`
2. **Include service prefix**: Anticipate that your MCP server may be used alongside other MCP servers
   - Use `slack_send_message` instead of just `send_message`
   - Use `github_create_issue` instead of just `create_issue`
3. **Be action-oriented**: Start with verbs (get, list, search, create, etc.)
4. **Be specific**: Avoid generic names that could conflict with other servers

### Tool Design

Create tool schemas that minimize MALFORMED_FUNCTION_CALL errors and maximize LLM comprehension.

**Core Principles**:
- **Explicit types**: Always declare types explicitly—never rely on inference
- **Required fields**: Use `.optional()` only for truly optional params; required by default
- **Flatten structures**: Avoid deep nesting; prefer flat parameter lists
- **Use enums**: Constrain allowed values with `z.enum()` instead of free-form strings
- **Rich descriptions**: Include purpose, constraints, format, and examples via `.describe()`

**General Guidelines**:
- Tool descriptions must narrowly and unambiguously describe functionality
- Descriptions must precisely match actual functionality
- Provide tool annotations (readOnlyHint, destructiveHint, idempotentHint, openWorldHint)
- Keep tool operations focused and atomic

#### Parameter Description Template

Every parameter description should include:

```
[What it is] + [Constraints] + [Format/Example]
```

Example:
```typescript
const schema = z.object({
  userId: z.string().describe(
    "Unique user identifier in UUID v4 format. Must be 36 characters. Example: 550e8400-e29b-41d4-a716-446655440000"
  ),
});
```

#### Quick Reference by Type

| Type | Description Pattern |
|------|---------------------|
| String ID | "Unique [entity] identifier in [format] format. Example: [value]" |
| Date | "Date in YYYY-MM-DD format. Example: 2025-01-15" |
| DateTime | "Timestamp in ISO 8601 format with timezone. Example: 2025-01-15T14:30:00Z" |
| Email | "Valid email address. Example: user@example.com" |
| Integer | "[Description]. Must be an integer between [min] and [max]." |
| Boolean | "[True behavior] vs [false behavior]. Default: [value]." |
| Array | "List of [type]. [Item constraints]. Min [n], max [m] items. Example: [value]" |
| Enum | Use `z.enum()`—description explains purpose only |

#### Common Anti-Patterns

Avoid these patterns that cause tool-calling failures:

```typescript
// ❌ Vague description
z.object({
  userId: z.string().describe("User ID"),
})

// ✅ Specific with format and example
z.object({
  userId: z.string().describe(
    "User identifier in UUID v4 format. Example: 550e8400-e29b-41d4-a716-446655440000"
  ),
})


// ❌ Missing format specification
z.object({
  date: z.string().describe("The date"),
})

// ✅ Explicit format with example
z.object({
  date: z.string().describe("Date in ISO 8601 format (YYYY-MM-DD). Example: 2025-01-15"),
})


// ❌ Implicit constraints
z.object({
  pageSize: z.number().describe("Number of items per page"),
})

// ✅ Explicit bounds
z.object({
  pageSize: z.number().int().min(1).max(100).default(20).describe(
    "Items per page. Range: 1-100."
  ),
})


// ❌ Free-form when values are constrained
z.object({
  status: z.string().describe("The order status"),
})

// ✅ Use enum
z.object({
  status: z.enum(["pending", "confirmed", "shipped", "delivered"]).describe(
    "Current order status."
  ),
})
```

#### Tool-Level Description

The main tool description should include:

- **What**: Primary purpose
- **When**: Triggering conditions
- **Returns**: Output format summary
- **Side effects**: Any state changes (if applicable)

Example:
```typescript
server.tool(
  "create_calendar_event",
  `Create a new calendar event and send invitations.

Use when the user wants to schedule meetings, appointments, or block time.

Returns the created event with confirmation details and event ID.

Side effects: Creates event in primary calendar, sends email invitations.`,
  schema,
  async (params) => { /* handler */ }
);
```

#### Complete Example

```typescript
import { z } from "zod";

const CreateTaskSchema = z.object({
  title: z.string()
    .min(1)
    .max(200)
    .describe("Task title. 1-200 characters."),

  description: z.string()
    .max(2000)
    .optional()
    .describe("Task details. Plain text, max 2000 characters."),

  priority: z.enum(["low", "medium", "high", "urgent"])
    .default("medium")
    .describe("Task urgency level."),

  dueDate: z.string()
    .optional()
    .describe("Due date in YYYY-MM-DD format. Example: 2025-01-15"),

  assigneeId: z.string()
    .optional()
    .describe("Assignee user ID in UUID format. Leave empty for unassigned."),
});

server.tool(
  "create_task",
  "Create a new task. Use when adding tasks, to-dos, or action items.",
  CreateTaskSchema,
  async (params) => { /* handler */ }
);
```

---

## Response Formats

All tools that return data should support multiple formats:

### JSON Format (`response_format="json"`)
- Machine-readable structured data
- Include all available fields and metadata
- Consistent field names and types
- Use for programmatic processing

### Markdown Format (`response_format="markdown"`, typically default)
- Human-readable formatted text
- Use headers, lists, and formatting for clarity
- Convert timestamps to human-readable format
- Show display names with IDs in parentheses
- Omit verbose metadata

---

## Pagination

For tools that list resources:

- **Always respect the `limit` parameter**
- **Implement pagination**: Use `offset` or cursor-based pagination
- **Return pagination metadata**: Include `has_more`, `next_offset`/`next_cursor`, `total_count`
- **Never load all results into memory**: Especially important for large datasets
- **Default to reasonable limits**: 20-50 items is typical

Example pagination response:
```json
{
  "total": 150,
  "count": 20,
  "offset": 0,
  "items": [...],
  "has_more": true,
  "next_offset": 20
}
```

---

## Transport Options

### Streamable HTTP

**Best for**: Remote servers, web services, multi-client scenarios

**Characteristics**:
- Bidirectional communication over HTTP
- Supports multiple simultaneous clients
- Can be deployed as a web service
- Enables server-to-client notifications

**Use when**:
- Serving multiple clients simultaneously
- Deploying as a cloud service
- Integration with web applications

### stdio

**Best for**: Local integrations, command-line tools

**Characteristics**:
- Standard input/output stream communication
- Simple setup, no network configuration needed
- Runs as a subprocess of the client

**Use when**:
- Building tools for local development environments
- Integrating with desktop applications
- Single-user, single-session scenarios

**Note**: stdio servers should NOT log to stdout (use stderr for logging)

### Transport Selection

| Criterion | stdio | Streamable HTTP |
|-----------|-------|-----------------|
| **Deployment** | Local | Remote |
| **Clients** | Single | Multiple |
| **Complexity** | Low | Medium |
| **Real-time** | No | Yes |

---

## Security Best Practices

### Authentication and Authorization

**OAuth 2.1**:
- Use secure OAuth 2.1 with certificates from recognized authorities
- Validate access tokens before processing requests
- Only accept tokens specifically intended for your server

**API Keys**:
- Store API keys in environment variables, never in code
- Validate keys on server startup
- Provide clear error messages when authentication fails

### Input Validation

- Sanitize file paths to prevent directory traversal
- Validate URLs and external identifiers
- Check parameter sizes and ranges
- Prevent command injection in system calls
- Use schema validation (Pydantic/Zod) for all inputs

### Error Handling

- Don't expose internal errors to clients
- Log security-relevant errors server-side
- Provide helpful but not revealing error messages
- Clean up resources after errors

### DNS Rebinding Protection

For streamable HTTP servers running locally:
- Enable DNS rebinding protection
- Validate the `Origin` header on all incoming connections
- Bind to `127.0.0.1` rather than `0.0.0.0`

---

## Tool Annotations

Provide annotations to help clients understand tool behavior:

| Annotation | Type | Default | Description |
|-----------|------|---------|-------------|
| `readOnlyHint` | boolean | false | Tool does not modify its environment |
| `destructiveHint` | boolean | true | Tool may perform destructive updates |
| `idempotentHint` | boolean | false | Repeated calls with same args have no additional effect |
| `openWorldHint` | boolean | true | Tool interacts with external entities |

**Important**: Annotations are hints, not security guarantees. Clients should not make security-critical decisions based solely on annotations.

---

## Error Handling

- Use standard JSON-RPC error codes
- Report tool errors within result objects (not protocol-level errors)
- Provide helpful, specific error messages with suggested next steps
- Don't expose internal implementation details
- Clean up resources properly on errors

Example error handling:
```typescript
try {
  const result = performOperation();
  return { content: [{ type: "text", text: result }] };
} catch (error) {
  return {
    isError: true,
    content: [{
      type: "text",
      text: `Error: ${error.message}. Try using filter='active_only' to reduce results.`
    }]
  };
}
```

---

## Testing Requirements

Comprehensive testing should cover:

- **Functional testing**: Verify correct execution with valid/invalid inputs
- **Integration testing**: Test interaction with external systems
- **Security testing**: Validate auth, input sanitization, rate limiting
- **Performance testing**: Check behavior under load, timeouts
- **Error handling**: Ensure proper error reporting and cleanup

---

## Documentation Requirements

- Provide clear documentation of all tools and capabilities
- Include working examples (at least 3 per major feature)
- Document security considerations
- Specify required permissions and access levels
- Document rate limits and performance characteristics
