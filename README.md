# pm-skill

[projectmanager](https://github.com/1359484419/projectmanager) 的 Claude Code skill —— 通过 MCP 协议管理任务、Sprint，生成日报/周报。

## 前置条件

- 一个运行中的 **projectmanager** 实例（自托管）
- **Claude Code**（CLI / 桌面版 / 网页版）或任意支持 MCP 的客户端（Cursor、Windsurf 等）

## 安装

### 第一步：生成 PAT（个人访问令牌）

1. 登录 projectmanager 网页端
2. 点击右上角头像 → **设置**
3. 找到 **API 令牌** 区域
4. 输入令牌名称，选择所属租户 → 点击 **生成**
5. 立即复制令牌（以 `pmt_` 开头，仅显示一次）

### 第二步：注册 MCP Server

**Claude Code：**

```bash
claude mcp add --transport http pm https://<你的域名>/mcp \
  --header "Authorization: Bearer pmt_<你的令牌>"
```

**其他 MCP 客户端**（Cursor、Windsurf 等）：将以下内容添加到客户端的 MCP 配置中（参考 `mcp-config.example.json`）：

```json
{
  "mcpServers": {
    "pm": {
      "type": "http",
      "url": "https://<你的域名>/mcp",
      "headers": {
        "Authorization": "Bearer pmt_<你的令牌>"
      }
    }
  }
}
```

### 第三步：安装 Skill

```bash
claude skill add --url https://github.com/1359484419/pm-skill
```

或手动安装：克隆本仓库，将 `SKILL.md` 复制到你项目的 `.claude/skills/` 目录下。

### 第四步：验证

启动一个新的 Claude Code 会话，输入：

> 列出项目

Claude 会调用 `list_projects` 并返回你的项目列表。如果看到结果，说明安装成功。

## 可用工具

| 工具 | 参数 | 说明 |
|------|------|------|
| `list_projects` | — | 列出租户下所有项目（key、名称） |
| `list_sprints` | `projectKey` | 查看 active / next / 最近关闭的 Sprint |
| `list_epics` | `projectKey` | 列出 Epic（id、名称、季度、状态） |
| `list_my_tasks` | `projectKey`, `sprint?`（current/previous） | 我在某个 Sprint 的任务 |
| `create_tasks` | `projectKey`, `target`, `tasks[]` | 批量创建任务（每次最多 20 条） |
| `update_task_status` | `taskSeq`, `status` | 更新任务状态 |

### 任务挂载目标

| Target | 含义 |
|--------|------|
| `current_sprint` | 挂到当前活跃 Sprint |
| `next_sprint` | 挂到下个 Sprint（不存在时自动创建） |
| `backlog` | 放入待办（不挂 Sprint） |

### 任务状态流转

`TODO` → `IN_PROGRESS` → `COMPLETED` → `DONE`

## 使用指南

### 把工作整理成任务

告诉 Claude 你做了什么，让它帮你整理成任务：

> 我今天修了登录页跳转的 bug，给注册表单加了密码校验，还重构了 auth 中间件。挂到当前 sprint。

Claude 会：
1. 把你的描述拆分成结构化任务（类型、标题、故事点）
2. 展示任务清单表格，等你确认
3. 确认后调用 `create_tasks`，返回任务编号（如 `DEV-1`、`DEV-2`）

### 更新任务状态

> DEV-1 做完了

Claude 会调用 `update_task_status("DEV-1", "COMPLETED")`。

### 生成日报

> 写日报

Claude 会通过 `list_my_tasks` 拉取当前 Sprint 任务，按状态分组输出：

```
## 日报 · 2026-07-07 · 你的名字

**今日完成**
- DEV-1 修复登录跳转 bug（1pt）

**进行中**
- DEV-2 注册表单密码校验 —— 完成 80%

**待办 / 明日计划**
- DEV-3 重构 auth 中间件

**风险 / 阻塞**
- 无
```

### 生成周报

> 写周报

Claude 会拉取当前和上个 Sprint 的任务，汇总本周完成、结转、下周计划。

### 查看我的任务

> 我这个 sprint 有什么任务？

Claude 会调用 `list_my_tasks` 并以表格展示。

### 更多用法

| 你说的话 | Claude 的动作 |
|----------|--------------|
| "列出项目" | `list_projects` → 项目列表 |
| "看看 DEV 的 sprint" | `list_sprints("DEV")` → active/next/recent |
| "放到下个 sprint" | `create_tasks(target=next_sprint)` |
| "先放 backlog" | `create_tasks(target=backlog)` |
| "DEV-3 开始做了" | `update_task_status("DEV-3", "IN_PROGRESS")` |

## 文件结构

```
pm-skill/
├── README.md                  # 本文件
├── SKILL.md                   # Skill 定义（Claude 读取此文件）
├── mcp-config.example.json    # MCP 客户端配置模板
└── LICENSE                    # MIT 协议
```

## 常见问题

| 问题 | 解决方案 |
|------|----------|
| "Tool not found" | 确认 MCP server 已注册：`claude mcp list` 应显示 `pm` |
| 连接被拒绝 | 检查 projectmanager 实例是否运行中，URL 是否正确 |
| 401 未授权 | PAT 可能已过期或被撤销，重新生成一个 |
| 返回空项目列表 | 确认 PAT 绑定了正确的租户 |

## License

MIT
