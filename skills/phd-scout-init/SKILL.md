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
| [your_core_method] | [[source_or_file]] | A |
| [your_problem_domain] | [[source_or_file]] | A |
| [your_supporting_tool] | [[source_or_file]] | B |

规则：

- L0 必须来自输入资料，不凭空添加。
- 每个 seed 要有证据来源。
- 默认标记只是草案，必须展示给用户逐项确认。
- 用户可把 seed 标为 `A` / `B` / `C`（C 表示明确排除，init 允许主动标）。

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
| 方向 | L0 seed 全部标为 A | 改为 A / B / C |
| 地区 | open | 选择 Europe / North America / Oceania / Asia / open |
| 开始时间窗 | open | 填季度或 open |
| 形态偏好 | project_position, pi_open_call | 可多选 7 类形态（含 pi_cold_email：先找 PI 后陶瓷） |
| 资金硬门槛 | funded_only | funded_only / accept_self_funded / already_funded |
| 语言 | english_only | english_only / accepts_other_languages |
| 偏好源（可选） | 留空 | 填心仪的机构名 / 站点 domain / 国家区域 |

匹配阈值默认 `0.6`，可在 `0.4-0.8` 内调整。

**偏好源（第 7 维）问法**：

> "有没有特别想盯紧的机构、站点或区域？例如 'UFZ', 'ETH Zurich', 'Germany'。
> 留空也行——只用默认 7 类形态主源搜（EURAXESS / FindAPhD / jobs.ac.uk 等）。"

收集为三类（用户可任填一项或全填）：

- `institutions`: 机构名列表
- `sites`: 站点 domain 列表（用于 `site:` 操作符）
- `regions_focus`: 区域名列表（强化国家信号）

填了的话，`/phd-scout` 每次会在默认主源之外**追加搜索**这些偏好源，扩大覆盖。

---

## Step 4.5 — 机构 / 学科平台 / 区域聚合站预筛（AI 推荐 + 用户勾选）

目的：扩大搜索覆盖，不只代替用户搜他已知的源。

基于已确认的 (方向 A 级 / 地区 / 形态)，AI 实时推三类候选（不读静态文件，靠当下知识 + 必要时 web 检索）：

### 候选 1 — 机构（institutions）

8-15 个该方向 + 地区里**有持续招博士传统**的机构。每条带：

- 机构名
- 所在国家
- 推荐理由（一句话：例 "在 [your_problem_domain] 方向常发 funded PhD"）

### 候选 2 — 学科聚合站（sites）

5-10 个该方向特有的招聘聚合平台（不是 EURAXESS / FindAPhD 这种通用主源）。例：

- 环境工程 → 学科性 job board / 学会招聘页
- 计算机 → 学科 mailing list mirror / 实验室聚合站
- 生命科学 → field-specific careers

每条带：domain + 推荐理由。

### 候选 3 — 区域聚合站（regions_focus）

3-8 个区域性聚合站，覆盖用户问卷里 `region.include` 的范围（例德国 → Helmholtz portal / GerWin / Academics.de）。

### 展示给用户

按三类列出，每条前加 `[ ]`。让用户：

- 勾选 → 写入 `preferred_sources.institutions / sites / regions_focus`
- 全部不选 → `preferred_sources` 留空（fallback 到默认主源）
- 自己补充 → 用户手填的也加进对应列表

**重要**：AI 不要替用户决定，不要默认全选。

---

## Step 5 — 过滤关键词树

根据问卷结果处理关键词：

- `A 级`：硬性关键词池，搜索结果至少命中 1 个。
- `B 级`：支撑关键词池，用于 60% 匹配。
- `C 级`：排除词或降权词，不进入正向搜索（init 允许主动标，也可由 optimize 反馈循环产生）。

若用户把某个 L0 标为 `C`，其 L1/L2 默认也排除，除非用户明确保留其中某个词。

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
- enabled opportunity types（默认仅 focus 2 类，用户在问卷里勾选其他）
- opportunity_types.priority（focus / normal / low 三层）
- funding rule
- language rule
- keyword source
- search.max_total_written_per_scout（默认 1，单独问用户："默认每次写入 Notion 1 条最优候选。要看更多可调 3-5；超过 5 反馈会疲劳。"）

`keywords.md` 必须包含：

- L0/L1/L2 树
- A 级 / B 级 / C 级 分区
- 每个 L0 的证据来源
- 最后更新时间

---

## Step 7 — Notion Inbox 配置

### 7.1 检查 Notion MCP 可用性

调一次 Notion MCP search 测试连通性。

- ✅ 可用 → 继续 7.2
- ❌ 不可用 → **引导装 MCP**（不降级，不进入手动模式）：
  > "phd-scout-flow 默认把评估报告写到 Notion，需要先装 Notion MCP。
  > 请按 `docs/notion-mcp-setup.md` 完成配置，完成后回来重跑 `/phd-scout-init`。
  > 如果你想把结果写到别处（Obsidian / 邮件 / 别的地方），后续可以自己改
  > `skills/phd-scout/SKILL.md` 的 Step 7。默认写入 Notion。"

### 7.2 询问用户：已有 DB 还是新建？

> "你已经有 PhD Inbox database 了吗？
> A. 已有 → 给我 database ID
> B. 没有 → 我帮你建一个（半自动）
> C. 自己建 → 我给你 schema 文档，你照着建"

### 7.3 若选 B（半自动建 DB）

1. 询问父 page：
   > "PhD Inbox 想放到哪个 page 下？请给我 page URL 或 page ID。
   > （workspace 根作为 parent 在多数 MCP 客户端不支持，建议先在 Notion 里建一个空 page 用作 parent。）"

2. 调用当前客户端暴露的 Notion MCP **create-database** 工具（不同客户端命名略不同，常见名：`notion-create-database` / `notion-create-pages` 系列；OpenAI 客户端可能省略 `notion-` 前缀）：
   - `parent` = 用户给的 page (`{ "type": "page_id", "page_id": "..." }`)
   - `title` = `PhD Inbox`（默认；用户可改）
   - `properties` = 按 `templates/notion-schema-inbox.md` 的 14 字段全套（Title / Text / URL / Date / Select / Multi-select 类型；select 选项含 7 类形态 / 4 类资助 / A/B/C 优先级 / 要-不要-观望）

   property shape 示例：
   ```json
   {
     "形态": {
       "select": {
         "options": [
           {"name": "project_position"},
           {"name": "pi_open_call"},
           {"name": "pi_cold_email"},
           {"name": "cdt_dtp"},
           {"name": "msca_dn"},
           {"name": "outbound_scholarship"},
           {"name": "industrial_phd"}
         ]
       }
     }
   }
   ```

3. 输出 DB URL，让用户审：
   > "已建好 PhD Inbox database：[URL]
   >
   > 你可以现在去 Notion 里：
   > - 改字段名（例如'机构'→'Institution'）—— skill 用语义匹配，不影响写入
   > - 加新字段（如自己的标签）—— skill 不会动这些字段
   > - 调 select 颜色 / 顺序 —— 无影响
   > - **不要删除 14 个核心字段**，否则写入会失败
   >
   > 改完确认回复'好'。"

4. 等用户确认。

5. 把 DB ID 写入 `profile.yaml`。

### 7.4 若选 A / C

- A：用户给 DB ID → 写入 profile
- C：输出 `templates/notion-schema-inbox.md` 内容 + `docs/notion-mcp-setup.md` 链接 → 等用户建好后给 ID

### 7.5 字段映射机制说明（顺便告诉用户）

> "scout 写入时会用 LLM 做语义映射——你把'机构'改成'Institution'、'Funded (Stipend)'改成'有奖学金'都能识别。
> 但**删除字段**或**改字段类型**（如 Text → Number）会导致写入失败。
> 增加字段是安全的，skill 不会动。"

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
- C 级: N

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
