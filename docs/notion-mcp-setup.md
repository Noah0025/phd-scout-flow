# Notion MCP Setup

Notion Inbox 是 `/phd-scout` 的结果入口。默认用 Notion 官方 hosted MCP：

```
https://mcp.notion.com/mcp
```

官方文档：

- https://developers.notion.com/guides/mcp/overview
- https://developers.notion.com/guides/mcp/get-started-with-mcp

---

## 1. 创建 Inbox Database

1. 在 Notion 新建一个 page。
2. 添加 database，命名为 `PhD Inbox`。
3. 按 `templates/notion-schema-inbox.md` 创建字段。
4. 复制 database id，后面写入 `~/.phd-scout/profile.yaml`。

database id 通常在 URL 中，形如：

```
https://www.notion.so/workspace/<database_id>?v=<view_id>
```

---

## 2. 配置 MCP

Notion MCP 使用 OAuth。配置后，第一次调用 Notion 工具时按提示登录并授权。

### Claude Code

```bash
claude mcp add --transport http notion https://mcp.notion.com/mcp
```

然后在 Claude Code 中运行 `/mcp`，完成 OAuth。

### Cursor

Cursor Settings -> MCP -> Add new global MCP server，填入：

```json
{
  "mcpServers": {
    "notion": {
      "url": "https://mcp.notion.com/mcp"
    }
  }
}
```

保存并重启 Cursor，首次使用 Notion 工具时完成 OAuth。

项目级配置可放在 `.cursor/mcp.json`。

### Cline

如果 Cline 所在环境支持 remote MCP，添加：

```json
{
  "mcpServers": {
    "notion": {
      "url": "https://mcp.notion.com/mcp"
    }
  }
}
```

如果只支持 stdio，可用 `mcp-remote`：

```json
{
  "mcpServers": {
    "notion": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.notion.com/mcp"]
    }
  }
}
```

### Codex

在 `~/.codex/config.toml` 添加：

```toml
[mcp_servers.notion]
url = "https://mcp.notion.com/mcp"
```

然后运行：

```bash
codex mcp login notion
```

完成 OAuth。

---

## 3. 验证

让 AI 工具执行一次 Notion search：

```
Search Notion for "PhD Inbox"
```

通过标准：

- 能找到刚创建的 database 或 page。
- 能读取 database 字段。
- 能创建一条测试 page。
- 测试完成后删除该 page。

---

## 4. 写入 profile.yaml

打开：

```
~/.phd-scout/profile.yaml
```

填入：

```yaml
notion:
  inbox_database_id: "[your_database_id]"
```

---

## 5. Optional Token Fallback

默认不用 token。Notion 官方 hosted MCP 使用 OAuth。

只有在需要无人值守、且你的 AI 工具无法完成 OAuth 时，才考虑 integration token + open-source MCP server。官方文档说明该 open-source server 不再活跃维护，所以这里作为 fallback，不作为默认安装路径。

申请入口：

```
https://www.notion.so/my-integrations
```

步骤：

1. New integration。
2. 选择 workspace。
3. 给 integration 命名。
4. Capabilities 至少开启 read content、insert content、update content。
5. 复制 internal integration token。
6. 打开 `PhD Inbox` 所在 page，添加该 integration 的访问权限。
7. 按所用 MCP server 的文档把 token 写入本地配置或环境变量。

边界：

- 不把 token 写进本仓库。
- 不把 token 写进 `profile.yaml`。
- 若只是在本机交互式使用，优先用 OAuth。

---

## Notes

- 官方 hosted MCP 需要人工 OAuth，不支持 bearer token。
- 需要无人值守时，可以研究 open-source Notion MCP server + integration token；官方文档说明该方案不再活跃维护。
- `/phd-scout` 写入失败时，不中断搜索；它会保留本地日志并在终端报告里标记。
