---
name: phd-scout-init
description: 首次配置 phd-scout-flow。从 capability-network vault 或 fallback 资料抽取 L0 seed，扩展 L1/L2 关键词，跑 6 维问卷，写入 ~/.phd-scout/profile.yaml 与 keywords.md，并引导创建 Notion Inbox。用法：/phd-scout-init
---

# PhD Scout Init

首次运行配置。目标是生成两份本地文件：

- `~/.phd-scout/profile.yaml`
- `~/.phd-scout/keywords.md`

关键词扩展只在本 skill 执行。后续 `/phd-scout` 运行时必须严格读取 `keywords.md`，不得自行扩展。

---

## CONFIG

```
SCOUT_HOME = ~/.phd-scout
PROFILE    = $SCOUT_HOME/profile.yaml
KEYWORDS   = $SCOUT_HOME/keywords.md
LOG_DIR    = $SCOUT_HOME/logs
TEMPLATES  = $SCOUT_HOME/templates
```

可选输入：

```
CAPABILITY_VAULT = /path/to/capability-network-vault
FALLBACK_PATHS   = /path/to/cv_or_transcript_or_sop
```

---

## Step 0 — 环境检查

1. 创建本地目录：

```bash
mkdir -p "$HOME/.phd-scout/logs" "$HOME/.phd-scout/templates"
```

2. 检查模板是否存在：
   - `templates/phd-profile.yaml.template`
   - `templates/phd-keywords.md.template`
   - `templates/notion-schema-inbox.md`

3. 检查是否已有配置：
   - `profile.yaml` 已存在 → 询问用户是否覆盖、补充或退出
   - `keywords.md` 已存在 → 询问用户是否覆盖、补充或退出

---

## Step 1 — 读取输入资料

优先读取 capability-network vault。

### 1a. capability-network 路径

若用户提供 `CAPABILITY_VAULT`，读取：

- `$CAPABILITY_VAULT/index.md`
- `$CAPABILITY_VAULT/Fields/*.md`
- `$CAPABILITY_VAULT/Concepts/**/*.md` 的标题、frontmatter、一级/二级标题
- `$CAPABILITY_VAULT/Sources/**/*.md` 的 TL;DR 和核心能力段落

重点抽取：

- Field 名称
- Concept/A 与 Concept/B 标题
- Summary 中反复出现的方法、对象、数据、问题域
- 核心能力 checklist

### 1b. fallback 资料

若没有 capability-network vault，询问用户提供任一资料路径：

- transcript
- CV
- SoP / Personal Statement
- 论文清单与摘要
- 课程项目描述
- 个人主页或公开简介文本

只抽取与“做过什么 / 会什么 / 想研究什么”相关的信息。不要从单个课程名过度推断研究方向。

---

## Step 2 — 生成 L0 Seed

输出一张候选表，让用户确认。

| seed | 来源证据 | 初始标记 |
|---|---|---|
| [your_core_method] | [[source_or_file]] | must |
| [your_problem_domain] | [[source_or_file]] | must |
| [your_supporting_tool] | [[source_or_file]] | nice |

规则：

- L0 必须来自输入资料，不凭空添加。
- 每个 seed 要有证据来源。
- 默认标记只是草案，必须展示给用户逐项确认。
- 用户可把 seed 标为 `must` / `nice` / `no`。

---

## Step 3 — 扩展 L1/L2 关键词

对用户确认后的 L0 seed 执行扩展。

### L1：同义 / 缩写 / 语言变体

每个 L0 seed 补充：

- 常见英文表达
- 常见缩写
- 拼写变体
- 必要时补本地语言变体

### L2：上位 + 下位

每个 L0 seed 补充：

- 上位领域词
- 下位方法词
- 子问题词
- 常见应用对象词

不要把 L2 扩到泛化过强的词。若一个词任何候选人都可能使用，默认不收。

---

## Step 4 — 6 维问卷

必须逐项展示给用户确认，不直接套默认值。

| 维度 | 默认草案 | 用户动作 |
|---|---|---|
| 方向 | L0 seed 全部标为 must | 改为 must / nice / no |
| 地区 | open | 选择 Europe / North America / Oceania / Asia / open |
| 开始时间窗 | open | 填季度或 open |
| 形态偏好 | project_position, pi_open_call | 可多选 6 类形态 |
| 资金硬门槛 | funded_only | funded_only / accept_self_funded / already_funded |
| 语言 | english_only | english_only / accepts_other_languages |
| 偏好源（可选） | 留空 | 填心仪的机构名 / 站点 domain / 国家区域 |

匹配阈值默认 `0.6`，可在 `0.4-0.8` 内调整。

**偏好源（第 7 维）问法**：

> "有没有特别想盯紧的机构、站点或区域？例如 'UFZ', 'ETH Zurich', 'Germany'。
> 留空也行——只用默认 6 类主源搜（EURAXESS / FindAPhD / jobs.ac.uk 等）。"

收集为三类（用户可任填一项或全填）：

- `institutions`: 机构名列表
- `sites`: 站点 domain 列表（用于 `site:` 操作符）
- `regions_focus`: 区域名列表（强化国家信号）

填了的话，`/phd-scout` 每次会在默认主源之外**追加搜索**这些偏好源，扩大覆盖。

---

## Step 5 — 过滤关键词树

根据问卷结果处理关键词：

- `must`：硬性关键词池，搜索结果至少命中 1 个。
- `nice`：支撑关键词池，用于 60% 匹配。
- `no`：排除词或降权词，不进入正向搜索。

若用户把某个 L0 标为 `no`，其 L1/L2 默认也排除，除非用户明确保留其中某个词。

---

## Step 6 — 写入文件

按模板生成：

- `$PROFILE`
- `$KEYWORDS`

`profile.yaml` 必须包含：

- profile version
- Notion Inbox database id 占位
- 6 维问卷结果（含可选第 7 维 `preferred_sources`）
- match threshold
- enabled opportunity types
- funding rule
- language rule
- keyword source

`keywords.md` 必须包含：

- L0/L1/L2 树
- must / nice / exclude 分区
- 每个 L0 的证据来源
- 最后更新时间

---

## Step 7 — Notion Inbox 引导

读取 `docs/notion-mcp-setup.md` 和 `templates/notion-schema-inbox.md`，引导用户完成：

1. 创建 Notion Inbox database。
2. 按 schema 添加字段。
3. 把 Notion database id 写入 `profile.yaml`。
4. 验证 AI 工具能 search / create page。

若用户暂时没有 Notion MCP，仍可完成本地配置，但 `/phd-scout` 只能输出终端报告和日志，不能写 Inbox。

---

## Step 8 — 输出报告

```
# PhD Scout Init Report
日期：YYYY-MM-DD

## 输入资料
- capability-network vault: <path or none>
- fallback paths: <paths or none>

## 关键词
- L0 seed: N
- L1 variants: N
- L2 related terms: N
- exclude: N

## Profile
- 地区：...
- 时间窗：...
- 形态：...
- 资金：...
- 语言：...
- 匹配阈值：...

## Notion
- Inbox database: configured / pending

## 下一步
- /phd-scout
```
