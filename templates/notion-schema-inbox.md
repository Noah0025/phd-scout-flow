# Notion Schema · PhD Inbox

Notion Inbox 是搜索结果的主界面。最小字段集（其他细节放页面正文，见 `eval.md`）。

| 字段 | 类型 | 选项 | 用途 |
|---|---|---|---|
| Title | Title | - | 岗位或机会标题（PI cold email 用 "PI 姓名 · 方向"）|
| 机构 | Text | - | institution full name |
| 链接 | URL | - | 原始页面 URL（PI cold email 用主页/实验室页）|
| 形态 | Select | project_position / pi_open_call / pi_cold_email / cdt_dtp / msca_dn / outbound_scholarship / industrial_phd | 机会类型 |
| 截止 | Date | - | deadline（PI cold email 留空）|
| 优先级 | Select | A / B / C | LLM 评级（见 `eval.md`）|
| Feedback | Select | 要 / 不要 / 观望 | 用户反馈（供反馈优化用）|
| 备注 | Text | - | 用户自由记录 |

页面正文按 `templates/eval.md` 写入（含 PI / 资助 / 匹配度 / 为何推荐 / 链接验证等详细信息）。

---

## 字段映射规则

`/phd-scout` 写入前 fetch DB schema，用 LLM 做字段语义映射：

- **改字段名安全**（"机构" → "Institution" 之类，skill 自动识别）
- **加字段安全**（skill 不动你自加的字段）
- **不要删核心字段** / **不要改字段类型**（会导致写入失败；scout 报错并写本地 markdown，不会自动改你的 DB schema）

---

## 半自动建 DB（init Step 7 用）

调 `notion-create-database` 工具，properties JSON：

```json
{
  "Title":   { "title": {} },
  "机构":    { "rich_text": {} },
  "链接":    { "url": {} },
  "形态":    { "select": { "options": [
    {"name": "project_position"}, {"name": "pi_open_call"}, {"name": "pi_cold_email"},
    {"name": "cdt_dtp"}, {"name": "msca_dn"}, {"name": "outbound_scholarship"}, {"name": "industrial_phd"}
  ]}},
  "截止":    { "date": {} },
  "优先级":  { "select": { "options": [{"name": "A"}, {"name": "B"}, {"name": "C"}]}},
  "Feedback":{ "select": { "options": [{"name": "要"}, {"name": "不要"}, {"name": "观望"}]}},
  "备注":    { "rich_text": {} }
}
```

→ MCP 工具名常见为 `notion-create-database`（OpenAI 客户端可能省略 `notion-` 前缀）。
