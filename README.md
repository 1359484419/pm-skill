# pm-skill

Claude Code skill for [projectmanager](https://github.com/1359484419/projectmanager) — manage tasks, sprints, and generate daily/weekly reports via MCP.

## Quick Start

### 1. Get a PAT (Personal Access Token)

Log in to your projectmanager instance → **Settings** (top-right) → **API Tokens** → enter a name and select tenant → **Generate**.

Copy the token (starts with `pmt_`) immediately — it is shown only once.

### 2. Register MCP Server

**Claude Code (CLI / Desktop / Web):**

```bash
claude mcp add --transport http pm https://<your-domain>/mcp \
  --header "Authorization: Bearer pmt_<your-token>"
```

**Other MCP clients** (Cursor, Windsurf, etc.): copy `mcp-config.example.json` into your client's MCP config and replace the URL and token.

### 3. Install Skill

```bash
claude skill add --url https://github.com/1359484419/pm-skill
```

Or manually: clone this repo and copy `SKILL.md` to your project's `.claude/skills/` directory.

### 4. Verify

Start a Claude Code session and say:

> list my projects

The `list_projects` tool should return your project list.

## What's Included

| File | Purpose |
|------|---------|
| `SKILL.md` | Skill definition — tells Claude when and how to use the PM tools |
| `mcp-config.example.json` | MCP client config template |

## Available Tools

| Tool | Description |
|------|-------------|
| `list_projects` | List all projects (key, name) |
| `list_sprints(projectKey)` | Active / next / recent sprints |
| `list_epics(projectKey)` | Epics for a project |
| `list_my_tasks(projectKey, sprint)` | My tasks in current / previous sprint |
| `create_tasks(projectKey, target, tasks)` | Batch create tasks (up to 20) |
| `update_task_status(taskSeq, status)` | Update task status (TODO → IN_PROGRESS → COMPLETED → DONE) |

## Usage Examples

- "把今天做的事整理成任务挂到当前 sprint"
- "写日报" / "写周报"
- "PM-42 做完了"
- "列出我这个 sprint 的任务"

## License

MIT
