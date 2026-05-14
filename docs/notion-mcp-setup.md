# Notion MCP Setup

Notion Inbox 是 `/phd-scout` 默认评估输出。装 Notion 官方 hosted MCP：

```
https://mcp.notion.com/mcp
```

官方文档：https://developers.notion.com/guides/mcp/overview

---

## 1. 配 MCP

OAuth 授权（首次调用 Notion 工具时按提示登录）。

### Claude Code

```bash
claude mcp add --transport http notion https://mcp.notion.com/mcp
```

然后运行 `/mcp` 完成 OAuth。

### Cursor

Settings → MCP → Add：

```json
{ "mcpServers": { "notion": { "url": "https://mcp.notion.com/mcp" }}}
```

### Cline

支持 remote MCP 时同上；只支持 stdio 时用 `mcp-remote`：

```json
{ "mcpServers": { "notion": {
  "command": "npx",
  "args": ["-y", "mcp-remote", "https://mcp.notion.com/mcp"]
}}}
```

### Codex

`~/.codex/config.toml`:

```toml
[mcp_servers.notion]
url = "https://mcp.notion.com/mcp"
```

然后 `codex mcp login notion`。

---

## 2. 建 Inbox Database

3 种方式：

- **A. 半自动**（推荐）：跑 `/phd-scout-init` 时让 AI 调 `notion-create-database` 建好让你审改
- **B. 手动**：按 `templates/notion-schema-inbox.md` 字段表自建
- **C. 已有**：直接给现有 DB ID 给 init

DB ID 在 URL：`https://www.notion.so/workspace/<DATABASE_ID>?v=...`

---

## 3. 验证

让 AI 工具试调一次 Notion search：找得到你刚建的 DB 即通过。

---

## 字段映射机制（顺便了解）

`/phd-scout` 每次写入前会 fetch DB schema，用 LLM 做语义映射：

- 改字段名安全（"机构" → "Institution"）
- 加新字段安全（skill 不动你自加的）
- **不要删核心字段或改字段类型** — scout 不会自动改你的 DB schema，遇到结构错会报错并写本地 markdown

核心字段：见 `templates/notion-schema-inbox.md`。
