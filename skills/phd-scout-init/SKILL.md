---
name: phd-scout-init
description: 首次配置 phd-scout-flow。从 capability-network vault 或 fallback 资料抽取 L0 seed，扩展 L1/L2 关键词，跑配置问卷（9 项），写入 ~/.phd-scout/profile.yaml 与 keywords.md，并引导创建 Notion Inbox。用法：/phd-scout-init
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

2. **检查并自动复制模板**到 `~/.phd-scout/templates/`（若用户直接跑 init 没跑 QUICKSTART Task 3，这一步兜底）：

```bash
# 找到 phd-scout-flow 仓库路径（用户可能在不同位置 clone）
REPO=$(dirname "$(dirname "$(realpath "${BASH_SOURCE[0]:-$0}")")")
# 若 ~/.phd-scout/templates/ 为空 → 从 repo 复制
if [ -z "$(ls -A ~/.phd-scout/templates/ 2>/dev/null)" ]; then
  cp "$REPO"/templates/*.template "$REPO"/templates/*.md "$REPO"/templates/*.yaml ~/.phd-scout/templates/ 2>/dev/null
fi
```

若复制失败（如 `$REPO` 解析失败 / 文件不存在）→ 提示用户先按 QUICKSTART Task 3 手动复制，并停止。

3. 检查必需模板：
   - `~/.phd-scout/templates/phd-profile.yaml.template`
   - `~/.phd-scout/templates/phd-keywords.md.template`
   - `~/.phd-scout/templates/notion-schema-inbox.md`

4. 检查是否已有配置：
   - `profile.yaml` 已存在 → 询问用户是否覆盖、补充或退出
   - `keywords.md` 已存在 → 询问用户是否覆盖、补充或退出

---

## Step 1 — 读取输入资料

优先读取 capability-network vault。

### 1a. capability-network 路径

若用户提供 `CAPABILITY_VAULT`，**不要硬编码目录名**——capability-network 标准结构是 `Fields/Concepts/Sources/Overviews/`，但很多用户用了自定义命名（如 `99_Fields/01_Concept/07_Articles/` 这类带序号前缀）。

**读取顺序**：

1. 必读：`$CAPABILITY_VAULT/index.md` — capability-network 强制有的入口文件，包含 TOC 和 wikilinks 指向所有 Field / Summary / Concept 卡。AI 先读这个建立全局视图。
2. 按 index 中的 wikilinks 解析实际文件路径（vault 内不同子目录都可能）。
3. **兜底 glob**：若 index 缺失或链接不全，跑 `find $CAPABILITY_VAULT -name "*.md" -not -path "*/node_modules/*" -not -path "*/.obsidian/*"`，按 frontmatter `tags:` 字段识别：
   - `tags: [Field]` → Field 卡
   - `tags: [Concept/A]` / `[Concept/B]` / `[Concept/C]` → 概念卡
   - `tags: [Study/Course]` / `[Study/Paper]` / `[Study/Thesis]` / `[Study/Internship]` 等 → 来源总结
   - `tags: [Overview]` → 全局视图
   - `tags: [Meta]` → index 自身或元数据

重点抽取：

- Field 名称（领域级映射）
- Concept/A 与 Concept/B 标题（方法 + 视角）
- Summary 中反复出现的方法、对象、数据、问题域
- 核心能力 checklist

不要从子目录名推断分类——以 frontmatter tag 为准。

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

## Step 2 — 生成 L0 Seed（批量呈现 + 用户标异议项）

输出一张候选表，**AI 默认全标 A**，让用户只标异议项（而不是逐项确认）。

| seed | 来源证据 | AI 草案 |
|---|---|---|
| [your_core_method] | [[source_or_file]] | A |
| [your_problem_domain] | [[source_or_file]] | A |
| [your_supporting_tool] | [[source_or_file]] | A |
| ... | ... | A |

规则：

- L0 必须来自输入资料，不凭空添加。
- 每个 seed 要有证据来源。
- **AI 默认全标 A**（优先级最高），用户只标异议项。
- 用户回复格式：
  - "OK" 或 "确认" → 接受默认（全 A）
  - "改 3,7 为 B" → 把 seed 3 和 7 改 B（支撑分）
  - "删 5" → seed 5 不进入关键词树
  - "5 改 C" → seed 5 标 C 级（明确排除）
  - "加 [keyword] 到 A" → 用户补充 seed

不接受"逐项问"模式——用户标完异议后整体进入 Step 3。

⚠️ 注意：A 级是**优先级**不是 hard hit 门槛（详见 README "命名契约"段）。

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

## Step 4 — 配置问卷（9 项）

必须逐项展示给用户确认，不直接套默认值。

| # | 配置项 | 默认草案 | 用户动作 |
|---|---|---|---|
| 1 | 方向（A/B/C 标签） | L0 seed 全部标为 A | 改为 A / B / C / 删除 |
| 2 | 地区 | open | 选择 Europe / North America / Oceania / Asia / open（multi） |
| 3 | 开始时间窗 | open | 填季度或 open |
| 4 | 形态偏好 | project_position + pi_open_call | 可多选 7 类形态（含 pi_cold_email：先找 PI 后陶瓷） |
| 5 | 资金硬门槛 | funded_only | funded_only / accept_self_funded / already_funded |
| 6 | 语言 | english_only | english_only / accepts_other_languages |
| 7 | 偏好源（可选） | 留空 | 填自己想到的机构 / 站点 domain / 区域 |
| 8 | 匹配阈值 | 0.6 | 0.4-0.8 内调整 |
| 9 | 每次写入条数 | 1 | 1-5；1 = 精准反馈，3-5 = 多看，≥5 反馈疲劳 |

**偏好源（「偏好源」项）问法**：

> "有没有特别想盯紧的机构、站点或区域？例如 'UFZ', 'ETH Zurich', 'Germany'。
> 留空也行——下一步 (Step 4.5) AI 会再推一批候选让你勾。"

收集为三类（用户可任填一项或全填）：

- `institutions`: 机构名列表
- `sites`: 站点 domain 列表（用于 `site:` 操作符）
- `regions_focus`: 区域名列表（强化国家信号）

**和 Step 4.5 的关系**：

- **「偏好源」项**：你**自己想到的**特殊机构 / 网站 / 区域（即使 AI 没推也想跟）
- **Step 4.5**：AI 推荐候选 + 你勾选（默认勾 top-N）

两者最终都写到 `preferred_sources` 同一字段，合并去重。**「偏好源」项留空不会跳过 Step 4.5**——Step 4.5 总是跑，AI 兜底推荐。

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

### 展示给用户（默认勾选 top-N，用户"取消"而非"全勾"）

按三类列出，**AI 默认勾选 top-N**，让用户**取消**不想要的（而不是从头勾全部）：

- 机构 institutions：默认勾选 **top-5**（按推荐理由相关度排序），其余 7-10 个放折叠区"显示更多"
- 学科聚合站 sites：默认勾选 **top-3**，其余放"显示更多"
- 区域聚合站 regions_focus：默认勾选 **top-2**，其余放"显示更多"

→ 默认 10 个勾选项（不是 27 个全勾），用户只需"取消 / 加更多"。

格式示例：

```
机构（默认勾选 top-5）：
[x] 1. UFZ — [理由]
[x] 2. ETH Zurich — [理由]
[x] 3. TU Delft — [理由]
[x] 4. EAWAG — [理由]
[x] 5. KIT — [理由]
[ ] 6. ... (折叠，回复"显示更多"查看)
```

让用户：

- 直接回复"OK"或"确认" → 接受默认 10 项
- "取消 1, 3" / "加 6, 9" → 调整默认
- "全选" → 全部 27 项都加（如本次 dogfood）
- "全部不选" → preferred_sources 留空，fallback 到默认主源

**AI 不要替用户最终决定**，但**给一个合理的默认起点**，减少决策疲劳。

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
- 配置问卷结果（含可选「偏好源」项 `preferred_sources`）
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
