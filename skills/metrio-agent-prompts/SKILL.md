---
name: metrio-agent-prompts
description: |
  Design Metrio AI agent prompts including orchestrator, sub-agents, and memory service prompts.
  Use when: (1) creating new Metrio agent, (2) designing orchestrator prompt, (3) creating sub-agent prompts,
  (4) setting up memory extraction/merger prompts, (5) user mentions "metrio prompt", "agent prompt",
  "orchestrator", "sub-agent", "memory extraction", "memory merger".
---

# Metrio Agent Prompts Designer

Design efficient, complete prompt sets for Metrio AI agents.

## Architecture

```
┌───────────────────────────────────────────────┐
│               Orchestrator                     │
│  - Receives system messages + conversation    │
│  - Coordinates sub-agents via YAML            │
│  - Returns JSON {status, responseType,        │
│    content, structContent?, reason?}          │
└──────────────┬──────────────┬─────────────────┘
               │              │
    ┌──────────▼────────┐ ┌──▼────────────────┐
    │   Sub-Agent(s)    │ │  Status Monitor   │
    │  (domain tasks)   │ │  (error handling) │
    └───────────────────┘ └───────────────────┘

┌───────────────────────────────────────────────┐
│             Memory Service                     │
│  Extraction → Merger (async, post-response)   │
└───────────────────────────────────────────────┘
```

## Workflow

### 1. Clarify Requirements

Before designing, ask if unclear:

- **Agent purpose**: Primary function? Domain/industry?
- **Sub-agents**: What capabilities? What MCP tools?
- **Memory**: What to remember? Conflict resolution strategy?

### 2. Design Prompts

Design in this order, using the reference templates:

1. **Orchestrator** - [orchestrator-template.md](references/orchestrator-template.md)
2. **Sub-Agents** - [sub-agent-template.md](references/sub-agent-template.md)
3. **Status Monitor** - Always include (template in sub-agent-template.md)
4. **Memory Extraction & Merger** - [memory-template.md](references/memory-template.md)

### 3. Deliver

Output each prompt as a separate markdown section:

```markdown
## Orchestrator Prompt
[prompt content]

## Sub-Agent: [name]
[prompt content]

## Memory Extraction
[prompt content]

## Memory Merger
[prompt content]
```

## Design Principles

- **Orchestrator gets context automatically**: User metadata, memory context, and conversation history are injected as system messages by the backend. Don't redefine input formats — instruct the orchestrator how to use them.
- **Sub-agents are stateless**: Pass ALL needed context via YAML. They know nothing about prior conversation.
- **JSON-only output**: Orchestrator must return pure JSON, no markdown or extra text.
- **Memory extracts only actionable info**: Define clear memoryTypes and merge strategies per domain.

## Quick Reference

### Orchestrator Output (JSON only)

```json
{
  "status": "completed | failed",
  "responseType": "text | struct",
  "content": "string (required)",
  "structContent": {},
  "reason": "string (only when failed)"
}
```

### Sub-Agent Communication (YAML)

```yaml
# Orchestrator → Sub-Agent
task: task_type
data:
  field1: value1

# Sub-Agent → Orchestrator
status: success | failed
result: { ... }
error: "if failed"
recoverable: true | false
```
