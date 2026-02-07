---
name: heimin-mcp
description: |
  **Heimin MCP Integration**: Complete guide for interacting with Heimin's project management and CRM platform via MCP tools. Covers team/project/task/page management plus full CRM capabilities (customers, contacts, deals, notes with semantic search).
  - MANDATORY TRIGGERS: Heimin, heimin, task management, project management, CRM, customer management, deal pipeline, team collaboration, create task, list projects, customer notes, interaction tracking, deal tracking
  - Use this skill whenever the user wants to interact with Heimin in any way — listing teams, creating tasks, managing projects, tracking customers, managing deals, adding notes, or searching interaction history. Also trigger when the user pastes a Heimin URL (heimin.app) or mentions any Heimin entity by name.
---

# Heimin MCP

## Overview

Heimin is a project management and CRM platform. This skill teaches you how to use all 34 Heimin MCP tools effectively. The tools are organized into two domains:

- **Project Management**: Teams → Projects → Tasks / Pages
- **CRM**: Customers → Contacts / Deals / Notes

Every workflow starts with `heimin_user_list_teams` to get a `teamId`. From there, you chain operations using returned IDs.

## Golden Rule: The ID Chain

You cannot skip steps in the ID chain. Almost every tool requires an ID obtained from a previous call. The two main chains are:

```
Project Management:
  heimin_user_list_teams → teamId
    → heimin_team_list_projects → projectId
      → heimin_project_list_tasks → taskId
      → heimin_project_list_pages → pageId

CRM:
  heimin_user_list_teams → teamId
    → heimin_crm_list_customers → customerId
      → heimin_crm_list_deals → dealId
      → heimin_crm_get_contacts → contactId
      → heimin_crm_get_notes → noteId
```

When assigning tasks or deals to people, you also need `heimin_team_list_members` to get valid `userId` values for the `assigneeId` or `ownerId` parameters.

**Shortcut**: Task and Page tools accept either an ID or a full Heimin URL (e.g., `https://heimin.app/zh/task/{taskId}`). If the user gives you a URL, pass it directly as the `taskId` or `pageId` parameter — no need to resolve the ID first.

## Key Patterns

### Content is Always Markdown

All content fields (task descriptions, page content, comments, notes) support full Markdown formatting. Use headings, bold, lists, links, code blocks — whatever makes the content clear and useful.

### Partial Updates

All update tools (`heimin_task_update`, `heimin_crm_update_customer`, `heimin_crm_update_deal`, `heimin_page_update`, `heimin_crm_update_note`) support partial updates. Only pass the fields you want to change — unchanged fields are preserved.

### Auto-Calculated Fields

- Changing a deal's `stage` automatically recalculates its `probability`
- Updating a note's `content` regenerates its semantic search embedding
- Timestamps are set automatically on creation

### Filtering Lists

Most list tools accept optional filters that combine together:

```
heimin_project_list_tasks(projectId, status: "open", assigneeId: userId)
heimin_crm_list_customers(teamId, search: "Acme", status: "active", tags: ["VIP"])
heimin_crm_list_team_deals(teamId, stage: "negotiation", ownerId: userId)
```

### Semantic Search

`heimin_crm_search_notes` uses AI-powered semantic search — it understands meaning, not just keywords. Use natural language queries like "client mentioned budget constraints" or "pricing discussion for enterprise plan". Scope searches with `teamId`, `customerId`, or `noteType`.

## Common Workflows

### 1. Explore Team Resources

```
heimin_user_list_teams()                    → pick teamId
heimin_team_get_info(teamId)                → team details
heimin_team_list_members(teamId)            → all members (userId values)
heimin_team_list_projects(teamId)           → all projects (projectId values)
```

### 2. Create and Manage a Task

```
heimin_project_create_task(projectId, title, content, assigneeId?, dueDate?, priority?)
  → returns taskId
heimin_task_add_comment(taskId, "Starting work on this")
heimin_task_update(taskId, status: "in_progress")
heimin_task_update(taskId, status: "completed")
```

### 3. CRM: New Customer with Deal

```
heimin_crm_create_customer(name, teamId, ownerId?, status?, tags?, profile?)
  → returns customerId
heimin_crm_add_contact(customerId, name, email?, phone?, title?, isPrimary?)
heimin_crm_create_deal(customerId, name, value?, currency?, stage?, ownerId?)
  → returns dealId
heimin_crm_add_note(customerId, content, noteType: "meeting")
heimin_crm_update_deal(dealId, stage: "negotiation")
```

### 4. Search Interaction History

```
heimin_crm_search_notes(query: "discussed pricing", teamId?, customerId?, limit?)
  → ranked results with customerId
heimin_crm_get_customer(customerId)         → drill into matched customer
heimin_crm_list_deals(customerId)           → see their deals
```

### 5. Project Documentation

```
heimin_project_create_page(projectId, title, content, status?)
  → returns pageId
heimin_page_update(pageId, content: "updated content", status: "published")
```

Team-level pages (not tied to any project) use:
```
heimin_team_create_page(teamId, title, content, status?)
heimin_team_list_pages(teamId, status?)
```

### 6. Project Health Check

```
heimin_project_get_statistics(projectId)    → task counts, completion rate
heimin_project_list_tasks(projectId, status: "open")  → open tasks
```

## Two Scopes for Pages

Heimin has pages at two levels — don't confuse them:

- **Team pages** (`heimin_team_create_page`, `heimin_team_list_pages`): Shared across the whole team, not tied to a project. Good for team wikis, policies, announcements.
- **Project pages** (`heimin_project_create_page`, `heimin_project_list_pages`): Belong to a specific project. Good for project docs, specs, meeting notes.

Both types are read/updated with the same tools: `heimin_page_get_details` and `heimin_page_update`.

## Two Scopes for Deals

Similarly, deals can be listed at two levels:

- **Customer deals** (`heimin_crm_list_deals`): All deals for one specific customer. Requires `customerId`.
- **Team deals** (`heimin_crm_list_team_deals`): All deals across the entire team. Requires `teamId`. Supports filtering by `customerId`, `ownerId`, or `stage`.

Use team-level deals for pipeline overviews and reporting. Use customer-level deals when focused on a single customer.

## CRM Notes Best Practices

Notes are the backbone of customer interaction tracking. They support:

- **noteType**: Categorize as "call", "email", "meeting", "note", or custom types
- **metadata**: Attach arbitrary key-value data (e.g., `{"duration": "30min", "attendees": 3}`)
- **Semantic search**: Notes are auto-indexed. Use `heimin_crm_search_notes` for AI-powered retrieval.

When adding notes, write them as if you're recording what happened for a colleague who wasn't there. Include context, decisions made, action items, and next steps.

## Permission Notes

- Only page creators or project admins can change page `status`
- Deal deletion (`heimin_crm_delete_deal`) is permanent and cannot be undone
- Task/deal assignment requires a valid `userId` from `heimin_team_list_members`

## Quick Reference

For detailed API parameters, return types, and the full tool catalog, read:
- `references/api_reference.md` — Complete tool-by-tool documentation with all parameters

## Date Format

All date parameters use ISO 8601 format: `YYYY-MM-DD` (e.g., "2026-03-15").

## Contact Fields

When adding contacts to customers, the available fields are: `name` (required), `email`, `phone`, `title` (job title), `lineId` (LINE messenger ID), and `isPrimary` (boolean, marks as primary contact).
