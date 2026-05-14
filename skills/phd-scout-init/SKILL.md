---
name: phd-scout-init
description: 首次配置 phd-scout-flow。从 capability-network vault 或 fallback 资料抽关键词，跑配置问卷，写入 ~/.phd-scout/profile.yaml 和 keywords.md，并引导建 Notion Inbox。用法：/phd-scout-init
---

# PhD Scout Init

首次配置 4 步：环境检查 → 抽关键词 → 配置问卷 → 建 Notion Inbox。

输出：

- `~/.phd-scout/profile.yaml`
- `~/.phd-scout/keywords.md`

---

## CONFIG

```
SCOUT_HOME = ~/.phd-scout
PROFILE    = $SCOUT_HOME/profile.yaml
KEYWORDS   = $SCOUT_HOME/keywords.md
LOG_DIR    = $SCOUT_HOME/logs
TEMPLATES  = $SCOUT_HOME/templates

CAPABILITY_VAULT = /path/to/capability-network-vault   # 可选
FALLBACK_PATHS   = /path/to/cv_or_transcript_or_sop    # 可选
```

---

## Step 1 — 环境检查

1. 创建本地目录：

```bash
mkdir -p "$HOME/.phd-scout/logs" "$HOME/.phd-scout/templates"
```

2. 若 `~/.phd-scout/templates/` 为空，自动从 repo 复制（兜底 QUICKSTART Task 3）：

```bash
REPO=$(dirname "$(dirname "$(realpath "${BASH_SOURCE[0]:-$0}")")")
if [ -z "$(ls -A ~/.phd-scout/templates/ 2>/dev/null)" ]; then
  cp "$REPO"/templates/* ~/.phd-scout/templates/ 2>/dev/null
fi
```

复制失败 → 提示用户手动跑 QUICKSTART Task 3 并停止。

3. 检查已有配置：`profile.yaml` / `keywords.md` 存在 → 询问覆盖 / 补充 / 退出。

---

## Step 2 — 抽关键词（一次性输出 + 用户改异议项）

读取输入资料：

### 资料来源

- **优先**：capability-network vault（`$CAPABILITY_VAULT/index.md` + frontmatter `tags:` 识别 Field/Concept/Summary）—— 不要硬编码目录名（用户可能自定义）
- **fallback**：CV / transcript / SoP / paper list / project descriptions 等

### 输出关键词表（AI 默认全 role=search + 自动识别异常项）

直接生成最终 keywords.md 草稿表，让用户**只标异议项**（不接受逐项确认）：

| keyword | role | parent | evidence | notes |
|---|---|---|---|---|
| [your_problem_domain] | search | — | [source] | 问题域，进搜索 query |
| [your_method_name] | context | — | [source] | 方法，仅评估加权 |
| [your_method_abbr] | context | [your_method_name] | init expansion | abbr |

### AI 自动判 role 规则

- `search`：问题域 / 应用领域 / 学科方向（PhD 招聘描述高频出现）
- `context`：方法 / 模型 / 工具 / 技能（PhD 招聘描述少有，但你专业相关）
- `exclude`：仅当用户主动标 / 资料中明示不想做的方向

### L1/L2 扩展（自动）

每个 L0 关键词自动扩 L1（同义 / 缩写 / 语言变体），记 `parent` 字段。L2（上位 / 下位）只在用户接受 L1 后扩。

### 用户回复格式

- `OK` / `确认` → 接受默认
- `改 3,7 为 context` → 改 role
- `5 改 exclude` → 标排除
- `删 5` → 移除
- `加 [keyword] (role)` → 用户补充

---

## Step 3 — 配置问卷（精简 6 项）

| # | 项 | 默认 | 用户动作 |
|---|---|---|---|
| 1 | regions | open | Europe / North America / Oceania / Asia / open（multi） |
| 2 | start_window | open | 填季度或 open |
| 3 | opportunity_types.enabled | project_position + pi_open_call | 可加 pi_cold_email / cdt_dtp / msca_dn / outbound_scholarship / industrial_phd |
| 4 | funding | funded_only | funded_only / accept_self_funded / already_funded |
| 5 | language | English | 多选 |
| 6 | max_written_per_scout | 1 | 1-5（推荐 1，反馈精准；调高反馈疲劳）|

**偏好源（可选）**：

> "想盯紧的机构 / 站点 / 区域？例：'[Institution A]', '[example.edu]', '[Country]'。
> 留空也行——默认主源已覆盖通用平台。"

填入 `preferred_sources.institutions` / `sites`。AI 可根据用户方向 + 地区**主动建议** 3-5 个候选机构 + 2-3 个学科聚合站，用户取消不要的即可（不是从零勾全部）。

---

## Step 4 — 建 Notion Inbox

1. 检查 Notion MCP 可用性 —— 不可用则引导装 MCP（见 `docs/notion-mcp-setup.md`），不降级
2. 询问用户：
   - A. 已有 Inbox DB → 给 database ID
   - B. 半自动建 → 给 parent page ID，AI 调 `notion-create-database` 按 `templates/notion-schema-inbox.md` 创建
   - C. 手动建 → 给 schema 文档让用户照建
3. 写 DB ID 到 `profile.yaml.notion.inbox_database_id`

### 字段映射说明（顺便告知）

> scout 写入时 LLM 做语义映射——"机构" 改成 "Institution" 都能识别。
> 删字段或改字段类型会导致写入失败（scout 不会自动修改你的 DB，会报错让你自己修）。

---

## 输出报告

```
# PhD Scout Init Report — YYYY-MM-DD

## 输入
- capability-network vault: <path or none>
- fallback paths: <paths>

## 关键词
- search: N · context: M · exclude: K

## 配置
- regions / start_window / types / funding / language / max_written_per_scout

## Notion
- Inbox DB: configured / pending

## 下一步
- /phd-scout
```
