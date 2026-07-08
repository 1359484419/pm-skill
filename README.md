# pm-skill

Claude Code skill for [projectmanager](https://github.com/1359484419/projectmanager) — manage tasks, sprints, and generate daily/weekly reports via MCP.

## Prerequisites

- A running **projectmanager** instance (self-hosted)
- **Claude Code** (CLI / Desktop / Web) or any MCP-compatible client (Cursor, Windsurf, etc.)

## Installation

### Step 1: Generate PAT (Personal Access Token)

1. Log in to your projectmanager Web UI
2. Click your avatar (top-right) → **Settings**
3. Go to **API Tokens** section
4. Enter a token name, select the tenant → click **Generate**
5. Copy the token immediately (starts with `pmt_`, shown only once)

### Step 2: Register MCP Server

**Claude Code:**

```bash
claude mcp add --transport http pm https://<your-domain>/mcp \
  --header "Authorization: Bearer pmt_<your-token>"
```

**Other MCP clients** (Cursor, Windsurf, etc.): add the following to your MCP config (refer to `mcp-config.example.json`):

```json
{
  "mcpServers": {
    "pm": {
      "type": "http",
      "url": "https://<your-domain>/mcp",
      "headers": {
        "Authorization": "Bearer pmt_<your-token>"
      }
    }
  }
}
```

### Step 3: Install Skill

```bash
claude skill add --url https://github.com/1359484419/pm-skill
```

Or manually: clone this repo and copy `SKILL.md` into your project's `.claude/skills/` directory.

### Step 4: Verify

Start a new Claude Code session and say:

> list my projects

Claude should call `list_projects` and return your project list. If it works, you're all set.

## Available Tools

| Tool | Parameters | Description |
|------|-----------|-------------|
| `list_projects` | — | List all projects (key, name) in your tenant |
| `list_sprints` | `projectKey` | Show active / next / recent (closed) sprints |
| `list_epics` | `projectKey` | List epics (id, name, quarter, status) |
| `list_my_tasks` | `projectKey`, `sprint?` (current/previous) | My tasks in a sprint |
| `create_tasks` | `projectKey`, `target`, `tasks[]` | Batch create up to 20 tasks |
| `update_task_status` | `taskSeq`, `status` | Update task status |

### Task Targets

| Target | Meaning |
|--------|---------|
| `current_sprint` | Add to the active sprint |
| `next_sprint` | Add to the next planned sprint (auto-creates if needed) |
| `backlog` | Add to backlog (no sprint) |

### Task Statuses

`TODO` → `IN_PROGRESS` → `COMPLETED` → `DONE`

## Usage Guide

### Create Tasks from Work Description

Tell Claude what you did, and ask it to organize into tasks:

> I fixed the login page redirect bug, added password validation to the signup form, and refactored the auth middleware. Put these in the current sprint.

Claude will:
1. Parse your description into structured tasks (type, title, points)
2. Show you a summary table for confirmation
3. After you confirm, call `create_tasks` and return the task IDs (e.g. `DEV-1`, `DEV-2`)

### Update Task Status

> DEV-1 is done

Claude will call `update_task_status("DEV-1", "COMPLETED")`.

### Generate Daily Report

> Write my daily report

Claude will fetch your current sprint tasks via `list_my_tasks`, group by status, and output:

```
## Daily Report · 2026-07-07 · Your Name

**Completed Today**
- DEV-1 Fix login redirect bug (1pt)

**In Progress**
- DEV-2 Add password validation — 80% done

**Tomorrow**
- DEV-3 Refactor auth middleware

**Blockers**
- None
```

### Generate Weekly Report

> Write my weekly report

Claude will pull tasks from both current and previous sprints and summarize completed work, carryovers, and next week's plan.

### List My Tasks

> What tasks do I have in this sprint?

Claude will call `list_my_tasks` and display a formatted table.

### Other Examples

| What you say | What happens |
|---|---|
| "List my projects" | `list_projects` → project table |
| "Show sprints for DEV" | `list_sprints("DEV")` → active/next/recent |
| "Put these in the next sprint" | `create_tasks(target=next_sprint)` |
| "Move to backlog" | `create_tasks(target=backlog)` |
| "DEV-3 is in progress" | `update_task_status("DEV-3", "IN_PROGRESS")` |

## File Structure

```
pm-skill/
├── README.md                  # This file
├── SKILL.md                   # Skill definition (Claude reads this)
├── mcp-config.example.json    # MCP client config template
└── LICENSE                    # MIT
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Tool not found" | Make sure MCP server is registered: `claude mcp list` should show `pm` |
| Connection refused | Check your projectmanager instance is running and the URL is correct |
| 401 Unauthorized | Your PAT may be expired or revoked — generate a new one |
| No projects returned | Verify the PAT is bound to the correct tenant |

## License

MIT
