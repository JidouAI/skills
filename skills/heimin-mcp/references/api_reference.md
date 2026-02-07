# Heimin MCP API Reference

Complete tool-by-tool documentation for all 34 Heimin MCP tools.

## Table of Contents

1. [User & Team Management](#user--team-management) (6 tools)
2. [Project Management](#project-management) (6 tools)
3. [Task Management](#task-management) (4 tools)
4. [Page Management](#page-management) (2 tools)
5. [CRM - Customer Management](#crm---customer-management) (4 tools)
6. [CRM - Contact Management](#crm---contact-management) (2 tools)
7. [CRM - Deal Management](#crm---deal-management) (6 tools)
8. [CRM - Notes & Search](#crm---notes--search) (4 tools)
9. [ID Chain Diagram](#id-chain-diagram)
10. [Parameter Quick Reference](#parameter-quick-reference)

---

## User & Team Management

### heimin_user_list_teams

List all teams the current user belongs to. **Starting point for all workflows.**

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| *(none)* | — | — | No parameters needed |

**Returns**: Array of teams with `teamId`, `name`, metadata.

---

### heimin_team_get_info

Get basic information about a team.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `teamId` | Yes | string | From `heimin_user_list_teams` |

**Returns**: Team name, description, member count, project count, settings.

---

### heimin_team_list_members

List all members in a team. Use to get `userId` values for task/deal assignment.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `teamId` | Yes | string | From `heimin_user_list_teams` |

**Returns**: Array of members with `userId`, `name`, `email`, role.

---

### heimin_team_list_projects

List all projects in a team.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `teamId` | Yes | string | From `heimin_user_list_teams` |

**Returns**: Array of projects with `projectId`, `name`, `description`, status.

---

### heimin_team_list_pages

List team-level pages (NOT project pages).

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `teamId` | Yes | string | From `heimin_user_list_teams` |
| `status` | No | string | Filter by page status |

**Returns**: Array of pages with `pageId`, `title`, `status`.

---

### heimin_team_create_page

Create a team-level page (not tied to any project).

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `teamId` | Yes | string | From `heimin_user_list_teams` |
| `title` | Yes | string | Page title |
| `content` | Yes | string | Markdown content |
| `status` | No | string | e.g., "draft", "published" |

**Returns**: Created page with `pageId`.

---

## Project Management

### heimin_project_get_info

Get basic information about a project.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `projectId` | Yes | string | From `heimin_team_list_projects` |

**Returns**: Project name, description, owner, dates, metadata.

---

### heimin_project_get_statistics

Get project statistics (task counts, completion rates).

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `projectId` | Yes | string | From `heimin_team_list_projects` |

**Returns**: Total tasks, completed tasks, completion %, status breakdown.

---

### heimin_project_list_tasks

List all tasks in a project with optional filtering.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `projectId` | Yes | string | From `heimin_team_list_projects` |
| `status` | No | string | Filter: "open", "completed", etc. |
| `assigneeId` | No | string | Filter by assignee (`userId`) |

**Returns**: Array of tasks with `taskId`, `title`, `status`, `assigneeId`.

---

### heimin_project_list_pages

List all pages in a project (project-level pages only).

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `projectId` | Yes | string | From `heimin_team_list_projects` |
| `status` | No | string | Filter by page status |

**Returns**: Array of pages with `pageId`, `title`, `status`.

---

### heimin_project_create_task

Create a new task in a project.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `projectId` | Yes | string | From `heimin_team_list_projects` |
| `title` | Yes | string | Task title |
| `content` | Yes | string | Markdown description |
| `assigneeId` | No | string | `userId` from `heimin_team_list_members` |
| `dueDate` | No | string | ISO 8601: "YYYY-MM-DD" |
| `priority` | No | string | "low", "medium", "high" |

**Returns**: Created task with `taskId`.

---

### heimin_project_create_page

Create a new page in a project.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `projectId` | Yes | string | From `heimin_team_list_projects` |
| `title` | Yes | string | Page title |
| `content` | Yes | string | Markdown content |
| `status` | No | string | e.g., "draft", "published" |

**Returns**: Created page with `pageId`.

---

## Task Management

### heimin_task_get_details

Get full details of a task. Accepts task ID or Heimin URL.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `taskId` | Yes | string | Task ID or `https://heimin.app/zh/task/{id}` |

**Returns**: Full task with `taskId`, `title`, `content`, `status`, `assigneeId`, `priority`, `dueDate`, comment count.

---

### heimin_task_update

Update a task. Only pass fields you want to change.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `taskId` | Yes | string | Task ID or URL |
| `title` | No | string | New title |
| `content` | No | string | New Markdown content |
| `status` | No | string | "open", "in_progress", "completed", etc. |
| `assigneeId` | No | string | New assignee (`userId`) |
| `priority` | No | string | New priority |

**Returns**: Updated task object.

---

### heimin_task_add_comment

Add a comment to a task.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `taskId` | Yes | string | Task ID or URL |
| `content` | Yes | string | Markdown comment |

**Returns**: Comment with `commentId`, author, timestamp.

---

### heimin_task_list_comments

List all comments for a task.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `taskId` | Yes | string | Task ID or URL |

**Returns**: Array of comments with `commentId`, `content`, author, timestamp.

---

## Page Management

### heimin_page_get_details

Get full details of a page. Accepts page ID or Heimin URL.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `pageId` | Yes | string | Page ID or `https://heimin.app/zh/page/{id}` |

**Returns**: Full page with `pageId`, `title`, `content`, `status`, dates, author.

**Note**: Works for both team-level and project-level pages.

---

### heimin_page_update

Update a page. Only pass fields you want to change.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `pageId` | Yes | string | Page ID or URL |
| `title` | No | string | New title |
| `content` | No | string | New Markdown content |
| `status` | No | string | New status (creator/admin only) |

**Returns**: Updated page object.

**Note**: Status changes require creator or project admin permissions.

---

## CRM - Customer Management

### heimin_crm_create_customer

Create a new customer.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `name` | Yes | string | Customer name |
| `teamId` | No | string | Team to associate |
| `ownerId` | No | string | Owner (`userId`) |
| `status` | No | string | "active", "inactive", "prospect", etc. |
| `source` | No | string | Acquisition source |
| `tags` | No | array | String tags, e.g., `["VIP", "enterprise"]` |
| `profile` | No | object | Custom fields, e.g., `{"industry": "tech"}` |

**Returns**: Created customer with `customerId`.

---

### heimin_crm_get_customer

Get detailed customer info including contacts and note count.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `customerId` | Yes | string | From list/create operations |

**Returns**: Customer with `customerId`, `name`, `status`, contact count, note count, profile.

---

### heimin_crm_update_customer

Update customer info. Only pass fields you want to change.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `customerId` | Yes | string | From list/get operations |
| `name` | No | string | New name |
| `status` | No | string | New status |
| `source` | No | string | Updated source |
| `tags` | No | array | Updated tags |
| `profile` | No | object | Updated profile |
| `ownerId` | No | string | Reassign owner |

**Returns**: Updated customer object.

---

### heimin_crm_list_customers

List all customers in a team with optional filters.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `teamId` | Yes | string | From `heimin_user_list_teams` |
| `search` | No | string | Text search on name/email |
| `status` | No | string | Filter by status |
| `ownerId` | No | string | Filter by owner |
| `tags` | No | array | Filter by tags |

**Returns**: Array of customers with `customerId`, `name`, `status`, `ownerId`, tags.

---

## CRM - Contact Management

### heimin_crm_add_contact

Add a contact person to a customer.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `customerId` | Yes | string | From customer operations |
| `name` | Yes | string | Contact person name |
| `email` | No | string | Email address |
| `phone` | No | string | Phone number |
| `title` | No | string | Job title |
| `lineId` | No | string | LINE messenger ID |
| `isPrimary` | No | boolean | Mark as primary contact |

**Returns**: Contact with `contactId`.

---

### heimin_crm_get_contacts

Get all contacts for a customer.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `customerId` | Yes | string | From customer operations |

**Returns**: Array of contacts with `contactId`, `name`, `email`, `phone`, `title`, `isPrimary`.

---

## CRM - Deal Management

### heimin_crm_create_deal

Create a new deal/opportunity for a customer.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `customerId` | Yes | string | From customer operations |
| `name` | Yes | string | Deal name |
| `ownerId` | No | string | Deal owner (`userId`) |
| `value` | No | number | Monetary value |
| `currency` | No | string | "USD", "TWD", "EUR", etc. |
| `stage` | No | string | "prospect", "negotiation", "won", etc. |
| `probability` | No | number | 0-100 win probability |
| `expectedCloseDate` | No | string | ISO 8601: "YYYY-MM-DD" |
| `notes` | No | string | Deal notes |

**Returns**: Created deal with `dealId`.

---

### heimin_crm_get_deal

Get detailed deal information.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `dealId` | Yes | string | From deal list/create operations |

**Returns**: Full deal with all fields including `actualCloseDate`.

---

### heimin_crm_update_deal

Update a deal. Only pass fields you want to change. **Changing `stage` auto-recalculates `probability`.**

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `dealId` | Yes | string | From deal operations |
| `name` | No | string | New name |
| `value` | No | number | New value |
| `currency` | No | string | New currency |
| `stage` | No | string | New stage (triggers probability recalc) |
| `probability` | No | number | Manual probability override |
| `expectedCloseDate` | No | string | New expected close |
| `actualCloseDate` | No | string | Set when deal closes |
| `notes` | No | string | Updated notes |
| `ownerId` | No | string | Reassign owner |

**Returns**: Updated deal object.

---

### heimin_crm_delete_deal

**Permanently** delete a deal. Cannot be undone.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `dealId` | Yes | string | From deal operations |

**Returns**: Deletion confirmation.

---

### heimin_crm_list_deals

List all deals for a specific customer.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `customerId` | Yes | string | From customer operations |
| `stage` | No | string | Filter by stage |
| `ownerId` | No | string | Filter by owner |

**Returns**: Array of deals with `dealId`, `name`, `value`, `stage`, `probability`.

---

### heimin_crm_list_team_deals

List all deals across the entire team.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `teamId` | Yes | string | From `heimin_user_list_teams` |
| `stage` | No | string | Filter by stage |
| `ownerId` | No | string | Filter by owner |
| `customerId` | No | string | Filter by customer |

**Returns**: Array of all team deals with `dealId`, `customerId`, `name`, `value`, `stage`.

---

## CRM - Notes & Search

### heimin_crm_add_note

Add an interaction note. Auto-indexed for semantic search.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `customerId` | Yes | string | From customer operations |
| `content` | Yes | string | Markdown note content |
| `noteType` | No | string | "call", "email", "meeting", "note", etc. |
| `metadata` | No | object | Custom key-value data |

**Returns**: Note with `noteId`.

---

### heimin_crm_get_notes

Get interaction notes for a customer.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `customerId` | Yes | string | From customer operations |
| `noteType` | No | string | Filter by type |
| `limit` | No | number | Max results |

**Returns**: Array of notes with `noteId`, `content`, `noteType`, timestamp.

---

### heimin_crm_update_note

Update a note. Content changes regenerate the search embedding.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `noteId` | Yes | string | From note add/get operations |
| `content` | No | string | New Markdown content |
| `noteType` | No | string | New note type |

**Returns**: Updated note object.

---

### heimin_crm_search_notes

AI-powered semantic search across notes. Understands meaning, not just keywords.

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `query` | Yes | string | Natural language search query |
| `teamId` | No | string | Scope to team |
| `customerId` | No | string | Scope to customer |
| `noteType` | No | string | Filter by type |
| `limit` | No | number | Max results |

**Returns**: Array of notes ranked by relevance with `noteId`, `customerId`, `content`, relevance score.

---

## ID Chain Diagram

```
heimin_user_list_teams (no params)
  │
  ├─ teamId ──┬── heimin_team_get_info
  │           ├── heimin_team_list_members ──── userId/ownerId/assigneeId
  │           ├── heimin_team_list_projects ─── projectId
  │           │     ├── heimin_project_get_info
  │           │     ├── heimin_project_get_statistics
  │           │     ├── heimin_project_list_tasks ── taskId
  │           │     │     ├── heimin_task_get_details
  │           │     │     ├── heimin_task_update
  │           │     │     ├── heimin_task_add_comment
  │           │     │     └── heimin_task_list_comments
  │           │     ├── heimin_project_list_pages ── pageId
  │           │     │     ├── heimin_page_get_details
  │           │     │     └── heimin_page_update
  │           │     ├── heimin_project_create_task
  │           │     └── heimin_project_create_page
  │           ├── heimin_team_list_pages ── pageId
  │           ├── heimin_team_create_page
  │           ├── heimin_crm_list_customers ── customerId
  │           │     ├── heimin_crm_get_customer
  │           │     ├── heimin_crm_update_customer
  │           │     ├── heimin_crm_add_contact ──── contactId
  │           │     ├── heimin_crm_get_contacts
  │           │     ├── heimin_crm_create_deal ──── dealId
  │           │     ├── heimin_crm_list_deals ───── dealId
  │           │     │     ├── heimin_crm_get_deal
  │           │     │     ├── heimin_crm_update_deal
  │           │     │     └── heimin_crm_delete_deal
  │           │     ├── heimin_crm_add_note ─────── noteId
  │           │     ├── heimin_crm_get_notes ────── noteId
  │           │     │     └── heimin_crm_update_note
  │           │     └── heimin_crm_search_notes
  │           ├── heimin_crm_list_team_deals
  │           └── heimin_crm_search_notes (team-wide)
  │
  └── heimin_crm_create_customer ── customerId
```

---

## Parameter Quick Reference

### Date Format
All dates: `YYYY-MM-DD` (ISO 8601), e.g., `"2026-03-15"`

### Common Enums

| Field | Known Values |
|-------|-------------|
| Task `status` | "open", "in_progress", "completed" |
| Task `priority` | "low", "medium", "high" |
| Customer `status` | "active", "inactive", "prospect" |
| Deal `stage` | "prospect", "negotiation", "won", "lost" |
| Note `noteType` | "call", "email", "meeting", "note" |
| Page `status` | "draft", "published" |

### URL-Accepting Tools

These tools accept either an ID or a full Heimin URL as the ID parameter:

- `heimin_task_get_details` — `taskId` or `https://heimin.app/zh/task/{id}`
- `heimin_task_update` — same
- `heimin_task_add_comment` — same
- `heimin_task_list_comments` — same
- `heimin_page_get_details` — `pageId` or `https://heimin.app/zh/page/{id}`
- `heimin_page_update` — same
