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

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                   Orchestrator                       │
│  - Entry point, receives OrchestratorInput          │
│  - Coordinates sub-agents via YAML                  │
│  - Returns {status, content} JSON                   │
└──────────────┬───────────────────────┬──────────────┘
               │                       │
    ┌──────────▼──────────┐ ┌─────────▼─────────┐
    │    Sub-Agent A      │ │   Status Monitor   │
    │  (domain-specific)  │ │  (error handling)  │
    └─────────────────────┘ └────────────────────┘

┌─────────────────────────────────────────────────────┐
│               Memory Service                         │
│  ┌─────────────────┐    ┌─────────────────┐        │
│  │   Extraction    │ -> │     Merger      │        │
│  └─────────────────┘    └─────────────────┘        │
└─────────────────────────────────────────────────────┘
```

## Workflow

### 1. Clarify Requirements

Before designing, ask these questions if unclear:

**Agent Purpose**
- What is the agent's primary function?
- What domain/industry is it for?

**Sub-Agents Needed**
- What distinct capabilities are required?
- What MCP tools will each sub-agent use?

**Memory Requirements**
- What information should be remembered?
- How should conflicts be resolved?

### 2. Design Prompts

After requirements are clear, design in this order:

1. **Orchestrator** - See [orchestrator-template.md](references/orchestrator-template.md)
2. **Sub-Agents** - See [sub-agent-template.md](references/sub-agent-template.md)
3. **Status Monitor** - Always include for error handling
4. **Memory Extraction** - See [memory-template.md](references/memory-template.md)
5. **Memory Merger** - Based on extraction categories

## Design Principles

### Prompt Efficiency
- Be concise - every token costs
- Use structured formats (YAML for inter-agent, JSON for output)
- Avoid redundant instructions

### Context Completeness
- Orchestrator must pass ALL needed context to sub-agents
- Sub-agents are stateless - assume no prior knowledge
- Include relevant IDs, user info, and task specifics

### Error Resilience
- Always include status-monitor
- Define recoverable vs non-recoverable errors
- Set reasonable retry limits (typically 3)

### Memory Design
- Extract only actionable information
- Define clear merge strategies
- Protect critical fields from accidental overwrite

## Output Format

Deliver prompts as separate markdown code blocks:

```markdown
## Orchestrator Prompt
[prompt content]

## Sub-Agent: [name]
[prompt content]

## Status Monitor
[prompt content]

## Memory Extraction
[prompt content]

## Memory Merger
[prompt content]
```

## Quick Reference

### Orchestrator Input/Output

```typescript
// Input
interface OrchestratorInput {
  userId: string;
  userMessage: string;
  context?: Record<string, any>;
}

// Output
{ "status": "completed" | "failed", "content": "回覆訊息" }
```

### Sub-Agent Communication (YAML)

```yaml
# Orchestrator -> Sub-Agent
task: task_type
data:
  field1: value1
  context: "necessary context"

# Sub-Agent -> Orchestrator
status: success | failed
result: { ... }
error: "if failed"
recoverable: true | false
```

### Status Monitor Decision

```yaml
decision: retry | abort | alternative
reason: "決策原因"
suggestion: "替代方案"
message_to_user: "若停止，給使用者的訊息"
```
