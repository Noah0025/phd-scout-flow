---
name: phd-scout
description: 常规 PhD 搜索与初评。读取 ~/.phd-scout/profile.yaml 和 keywords.md，按 7 类形态搜索，执行 60% 匹配过滤，验链，生成评估，写入 Notion Inbox 与本地日志。用法：/phd-scout
---

# PhD Scout

常规搜索流程。输入来自本地配置，不在运行时扩展关键词。

---

## CONFIG

```
SCOUT_HOME = ~/.phd-scout
PROFILE    = $SCOUT_HOME/profile.yaml
KEYWORDS   = $SCOUT_HOME/keywords.md
LOG_DIR    = $SCOUT_HOME/logs
TEMPLATES  = $SCOUT_HOME/templates
```

---

## Step 0 — 读取配置

1. 读取 `$PROFILE`。
2. 读取 `$KEYWORDS`。
3. 检查必需字段：
   - Notion Inbox database id
   - enabled opportunity types + priority (focus / normal / low)
   - matching.threshold_hint
   - funding rule
   - region rule
   - language rule
   - A 级关键词
   - B 级关键词
   - C 级关键词
4. 读取可选字段 `preferred_sources`：
   - `institutions` / `sites` / `regions_focus` 任一非空 → 启用追加搜索（Step 3.5）
   - 全部为空 → 跳过追加搜索，只走默认主源
5. 读取 `search` 预算字段：
   - `total_queries_per_scout`、`queries_per_type`、`preferred_source_total_per_type`、`probe_queries`、`results_per_query`、`query_repeat_cooldown_days`、`append_year_filter`
   - 不存在则退回 SKILL 内置默认（见下）

缺失 `profile.yaml` 或 `keywords.md` 时，停止并提示先运行 `/phd-scout-init`。

---

## Step 1 — 读取 Notion Inbox 去重基准

从 Notion Inbox 读取未过期、未明确拒绝的条目。

去重键：

```
<title_normalized> + <institution_normalized>
```

### Notion 不可用的三种情况（区分处理）

- **(a) MCP 完全不可用**（连不上 / 未配置）→ 跳过远端去重 + 用本地日志弱去重 + 终端报告标"Notion 未连接"
- **(b) inbox_database_id 缺失或为占位符**（如 `[PENDING]`）→ 提示用户先完成 init Step 7，并跳过本步骤；scout 仍可跑（只写本地日志）
- **(c) DB ID 有但权限不够**（401 / 403）→ 同 (a) 处理 + 报告里建议用户检查 integration access

---

## Step 2 — 生成搜索队列

按 profile 中启用的形态循环。

### 搜索 query 构造规则（重要）

- **主搜 query 只用 `seed_type: search_anchor` 的关键词**——方法类（`weight_only`）不进搜索 query，只在 Step 4 评估时加权
- **所有主搜 query 必须带 `site:` 过滤**到默认主源（通用 Google 找学术职位质量低）
- **缩写关键词用 `"full name" OR ABBR` 组合**（防 Google 忽略缩写如 [method_abbreviation]）

### 形态搜索词模板（占位符 `[anchor]` = 一个 A 级 search_anchor 关键词）

| 形态 | 默认主源 site | 搜索词模板 |
|---|---|---|
| project_position | euraxess.ec.europa.eu / findaphd.com / jobs.ac.uk | `site:euraxess.ec.europa.eu PhD [anchor] funded` |
| pi_open_call | 院系页 / PI 主页 | `PhD position [anchor] research group accepting students` |
| pi_cold_email | scholar.google.com / 实验室主页 | `site:scholar.google.com [anchor] author:` → followup: `[pi_name] lab openings` |
| cdt_dtp | UKRI / cdt 站 | `site:ukri.org OR site:jobs.ac.uk doctoral training [anchor] PhD studentship` |
| msca_dn | EURAXESS MSCA | `site:euraxess.ec.europa.eu MSCA Doctoral Network [anchor]` |
| outbound_scholarship | CSC / DAAD / Chevening | `site:daad.de OR site:csc.edu.cn PhD [anchor] supervisor` |
| industrial_phd | ANRT (CIFRE) / industry | `site:anrt.asso.fr industrial PhD [anchor]` 或 `[anchor] industrial PhD funded` |

**pi_cold_email 特殊流程**（与其他形态不同）：

1. Google Scholar 搜 `[a_level_keyword]`，过滤近 3 年高发文者 → 提取 PI 名 + 机构 + 近期论文
2. 对每个 PI 找其实验室主页 / 院系主页
3. 主页找 "openings" / "join us" / "PhD positions" 段
4. 输出 **PI 卡片**（不是岗位卡片）：见 `templates/phd-eval-template.md` 备用模板
5. 评级按 `templates/phd-eval-criteria.md` 的"PI Cold Email 特殊规则"

规则：

- 只使用 `keywords.md` 中的词。
- 不根据搜索结果临时发明新关键词。
- 每类形态最多保留候选若干条，后续过滤后只推荐最佳 1 条。

---

## Step 2.5 — 预算分配与形态优先级

读 `search.total_queries_per_scout`（默认 30）、`opportunity_types.priority`（focus / normal / low）、`min_reserved_for_normal`（默认 6）、`min_reserved_for_low`（默认 0）。

### 预算分配公式

```
单形态总开销 = queries_per_type + min(preferred_source_total_per_type, preferred_sources 非空时 1，否则 0)

预算保留：
  reserved_for_normal = min_reserved_for_normal
  reserved_for_low    = min_reserved_for_low
  probe 预算固定保留：probe_queries
  focus 可用预算 = total_queries_per_scout - probe_queries - reserved_for_normal - reserved_for_low

分配顺序：
  1. focus 层按形态循环分配，每形态扣"单形态总开销"，直到 focus 可用预算耗尽
     若 focus 层某形态已分到 queries_per_type 但偏好源 query 仍超预算
       → 该形态偏好源 query 数按剩余预算 prorate（最少 1，最多 preferred_source_total_per_type）
  2. normal 层用 reserved_for_normal 预算，同样按形态循环分配
  3. low 层用 reserved_for_low 预算（默认 0 → low 形态不主动跑，除非用户调高）
  4. probe_queries 始终独立保留，不能被其他层挤占
```

**示例**（默认值 + 2 focus 形态 + preferred_sources 非空）：

```
total=30, probe=1, normal_reserve=6, low_reserve=0
focus 可用 = 30 - 1 - 6 - 0 = 23
focus 形态 A: queries_per_type(3) + preferred_source_total_per_type(3) = 6
focus 形态 B: 6
→ focus 用掉 12，剩 11 → normal 形态分到 11+6 = 17（但 normal_reserve 只承诺 6）
注意：reserved 是下限，超出 reserved 的部分可被更高层用完后顺延
```

总预算上限优先于完整覆盖。**不要为了搜完所有形态而无限增查询数**。

### 单形态内的查询模板（3 轮）

**只用 `seed_type: search_anchor` 的 A 级关键词进搜索 query**。"主 A" / "第二 A" 按 keywords.md 中 search_anchor seed 的出现顺序（相当于用户认为最重要的在前）。

```
轮 1: 主 A anchor × 形态词模板（最广覆盖）
轮 2: 主 A anchor × 1 个 B anchor（适中聚焦，B 必须也是 search_anchor 类型）
轮 3: 第二 A anchor × 形态词模板（覆盖另一方向）
```

规则：

- 不做 `A anchor × A anchor` 组合（太窄，覆盖面缩小）
- `weight_only` 关键词（方法类）**绝不进搜索 query**（避免噪音）
- B 级搜索补搜也只用 search_anchor 类
- 若只有 1 个 A anchor → 跳过轮 3，节省预算
- **缩写处理**：若 anchor 是缩写（如 `[method_abbreviation]`），query 自动改为 `"全称" OR 缩写` 组合，例 `"[full_form]" OR [abbreviation]`

### 时效性

若 `append_year_filter: true`（默认）：

每条 query 末尾追加 `[current_year] OR [next_year]`。例：

```
PhD [a_level_keyword] [b_level_keyword] funded 2026 OR 2027
```

例外：`pi_cold_email` 的 Google Scholar 搜索不加年份（影响 author filter）。

### 跨次稳定性

每条 query 字符串与本地 `~/.phd-scout/logs/scout-*.yaml` 历史 query 对比。
若同一 query 在 `query_repeat_cooldown_days`（默认 14）内已跑过 → 跳过本次，节省预算。

---

## Step 3 — 搜索与候选提取

按 Step 2.5 分配的预算执行 3 轮主查询。

每查询取前 `results_per_query` 条（默认 20）。

### "失败"定义与回退

一个形态被视为 **失败** 当且仅当：

- 3 轮主查询累计候选 < 2 条 **或**
- 候选全部通过过滤后（Step 4）剩 0 条

失败时回退顺序：

1. 换 A 级关键词的 L1 变体重跑一轮（仍计入预算）
2. 改用 B 级关键词补搜 1 轮，**但 B 级命中不算 A 级**
3. 仍失败 → 在日志记 `no_eligible_candidate`，本形态结束（不无限补搜）

成功定义：≥1 条通过 Step 4-5 过滤的候选。

对每条候选提取：

- title
- institution
- city / country
- opportunity type
- URL
- deadline
- PI / group
- funding
- language signal
- matched keywords
- missing keywords

---

## Step 3.5 — 偏好源追加搜索（仅当 `preferred_sources` 非空）

每次 scout 跑都要做这一步，确保你勾选过的偏好源都覆盖到。

对每个启用形态，主搜完后追加一轮：

- `institutions` 非空 → 对每个机构拼接 `[机构名] PhD [a_level_keyword]` 跑一次
- `sites` 非空 → 对每个 site 跑 `site:[domain] PhD [a_level_keyword]`
- `regions_focus` 非空 → 对每个区域拼接 `[country/region] PhD [a_level_keyword] funded`

追加搜索的候选合并进主搜候选池，统一进入 Step 4 过滤。匹配规则不变。

不要把"机构名"或"site"本身当作关键词命中——它们只用于扩大搜索覆盖，不计入 60% 匹配分。

---

## Step 3.6 — 探测搜索（每次 scout 跑 1 次）

目的：发现新源 / 新机构 / 新聚合站，不让搜索范围被默认主源 + preferred_sources 写死。

执行 1 次通用 Google 搜索（**不带任何 site 过滤**）：

```
[a_level_keyword] PhD position [current_year_or_next_year]
```

**探测 query 显式豁免 cooldown 检查**——和 `probe_queries` 预算保留同理由：探测的价值在长尾发现，跨次重复跑同一查询并不浪费（每次结果可能不同）。

为减少重复噪音，探测 query 可在多个 A 级关键词之间轮换：每次 scout 选用上次未用的 A 级关键词；若所有 A 级都用过 → 重新轮换。

从结果前 30 条提取 domain 分布。规则：

- 同一 domain 出现 ≥ **2 次** → 记入 `discovered_sources`（见 `templates/scout-log-example.yaml`）
- 已在 `preferred_sources.sites` 或默认主源里的 domain → 不重复记录
- 单次出现的 domain → 忽略（噪音）

**不自动加进 preferred_sources**——保持 scout 行为可预测。新源的沉淀路径：

```
discovered_sources（日志） → 用户在 Notion 标"要" ≥3 次该源候选
  → /phd-keyword-optimize 表 4 建议加进 preferred_sources.sites
  → 用户确认 → 写入 profile
```

候选物本身（不是 domain）该被验链 / 过滤的还是走 Step 4-6。探测的产物只有 domain 进入 `discovered_sources` 字段。

---

## Step 4 — 硬性筛选（只过滤死亡条件，不做数学打分）

**不做加权分硬过滤**——评估视角应该是"项目需要什么 / 用户覆盖多少"，**用关键词库做分母会过度稀释**（库越大分越低）。匹配度判断挪到 Step 6 评估时由 LLM 综合判。

Step 4 只过滤死亡条件：

1. **C 级硬排除**：命中任何 C 级关键词 → 整体排除（除非 profile.matching.c_level_is_hard_exclude = false）
2. **funding 不符** → 排除（profile.questionnaire.funding.rule）
3. **deadline 已过且无 rolling 标记** → 排除
4. **明显过期年份信号**（防 EURAXESS 等排序不按 deadline 的坑）：候选描述/标题/URL 中出现 ≥3 年前的年份（如 2022/2023 当前 2026）→ 标 `likely_expired`，最高 B 级，必须 Step 5 验链确认
5. **region rule / language rule**：硬性不符 → 排除；信息缺失 → 标"待确认"进入 Step 5
6. **A 级 + B 级 keyword hits 都为 0** → 排除（毫无相关度）

通过 Step 4 → 进入 Step 4.5 跨形态去重 → Step 5 验链。

匹配度的"绝对计数"仍要记录（供 Step 6 评估 + 日志归因用）：

```
A_hits: 候选描述里命中的 A 级关键词列表（含 search_anchor + weight_only）
B_hits: 候选描述里命中的 B 级关键词列表
```

**不**生成 weighted_score——那是过度简化的伪指标（详见 Step 6 的"项目角度"评估）。

---

## Step 4.5 — 跨形态去重与候选合并

同一候选可能在多个形态查询里出现（如同一岗位既被 project_position 抓到，也被 msca_dn 抓到）。

合并规则：

```
去重键：normalize(url) + normalize(title + institution)
命中多形态时：
  - 保留为 1 条候选
  - matched_types: [project_position, msca_dn, ...]
  - 取最高匹配分
  - 优先级取最严形态规则（如 msca_dn 有 mobility 检查未通过 → 整体降级）
  - 不重复写入 Notion
```

跨形态去重在 Step 4 过滤之后、Step 5 验链之前。验链对合并后的 1 条候选做一次即可。

---

## Step 5 — 验链 + Cross-check

对通过过滤的候选逐一打开原链接验证：

- 页面存在
- 岗位仍开放或信息未过期
- title / institution 与候选一致
- funding / deadline 没有被搜索摘要误读

### Cross-check（防 URL ID 不稳定的坑）

某些平台（如 EURAXESS）的 URL/ID 可能复用或失效，验链 fetched 的页面**可能不是搜索结果显示的那条**。验链时必须比对：

1. fetched 页面的 title 关键词 是否包含搜索摘要的 title 关键词（核心专业词，如 [problem_domain_example] / [applied_domain_example]）
2. fetched 页面的 institution 是否与搜索摘要的 institution 一致

不一致 → **重搜本条候选**（用更精确的 query：`"title 关键词" institution`）；重搜失败 → 标 `link_unverified` 并降级到 B 级。

### 抓取失败

- 普通抓取失败 → 尝试浏览器工具
- 仍失败 → 标记 `link_unverified`，最高 B 级

### 排除条件

- 404 / closed / filled
- deadline 已过且无 rolling 标记
- Cross-check 失败 + 重搜失败
- 候选描述里的年份明显过旧（Step 4 已标 `likely_expired` 的候选要二次确认 deadline）

---

## Step 6 — 评估与选择（项目角度打分 + LLM 综合判 A/B/C）

读取：

- `templates/phd-eval-template.md`
- `templates/phd-eval-criteria.md`

### 项目角度匹配度（核心）

**不用"用户关键词库 / total"做分母**——库越大分越被稀释。视角颠倒：

```
1. LLM 从 PhD 完整描述（Step 5 fetched 全文）抽取 job_signals (3-8 个)：
   - 研究方向 / 主题
   - 关键方法 / 工具
   - 申请条件中的硬技能要求
   - PI 课题组的近期主题（如能识别）

2. 看用户的关键词库（A 级 + B 级，含 weight_only 方法类）覆盖了多少个 signal：
   match_rate = covered_signals / |job_signals|

3. 通用示例（数字仅作语义说明）：
   job_signals = [signal_1, signal_2, signal_3, signal_4, signal_5, signal_6, signal_7]  (7 个)
   用户命中 = [signal_1] ✓ / [signal_2] ✓ / [signal_4] ≈ / [signal_6] ≈  (4 个 hits)
   match_rate = 4/7 = 57%   ← 用户"能贡献什么"的实际度量
```

### LLM 综合判 A/B/C（按 phd-eval-criteria.md）

不要把 match_rate 当硬阈值。LLM 综合看：

- match_rate（项目角度覆盖率）
- A 级 search_anchor 是否命中（方向是否真实对上）
- weight_only 方法类是否命中（专业匹配深度）
- funding 完整度
- deadline 可行性
- 形态符合度

→ 综合判 A（优先调研）/ B（关注）/ C（不投入）。具体阈值参考 `phd-eval-criteria.md`。

### 写入决策（按 max_total_written_per_scout）

不是"每形态各写 1 条"——按 `search.max_total_written_per_scout`（默认 1）跨形态选总 N 条最佳：

```
1. 把所有形态通过 Step 4-5 的候选合并到一个池
2. 按 LLM 综合评级排序（A > B > C），同级按 match_rate
3. 取前 N 条（N = max_total_written_per_scout）
4. 约束：单形态 ≤ max_written_per_type（默认 1，防一类占满）
5. 若某形态被分配 0 条 → 日志记"该形态有候选但未入选"，不报"无合格新候选"
```

写入数量与噪音的权衡（用户在 profile 中调）：

| N | 体验 | 反馈难度 |
|---|---|---|
| 1（推荐） | 每次最优 1 条，认真读、给精准反馈 | 低 |
| 2-3 | 多看几个选项 | 中 |
| ≥5 | 信息丰富但 Inbox 淹没，反馈疲劳，optimize 信号变弱 | 高 |

若全部候选都为 C 或被排除 → 报”无合格新候选”，不强行写入。

评估必须包含：

- 形态
- 匹配分
- 为何推荐（一句因果链）
- A/B/C 优先级
- 不确定信息
- 下一步人工检查点

不预测录取概率。

---

## Step 7 — 写入 Notion Inbox（语义映射）

若 Notion MCP 可用且 profile 中有 database id：

### 7.1 Fetch DB schema（每次写入前）

调用当前客户端暴露的 Notion MCP **fetch / retrieve-database** 工具（常见名：`notion-fetch` / `notion-retrieve-database`；OpenAI 客户端可能省略 `notion-` 前缀）。

input：`database_id` = profile 中的 inbox_database_id
output：当前实际字段列表（含名称、类型、select 选项值）

→ 这一步是为了支持用户自己改字段名 / 加新字段。每次写入都 fetch 是廉价操作。

### 7.2 做字段语义映射

把 `templates/notion-schema-inbox.md` 的 standard 字段名（如"机构"/"截止"/"匹配分"）映射到 DB 实际字段名。

LLM 用语义识别处理常见情况：

| Standard | DB 实际 | 怎么映射 |
|---|---|---|
| "机构" | "Institution" | 直接同义匹配 |
| "截止" | "Deadline" / "Due" / "Application by" | 语义识别（日期字段 + 截止相关词） |
| "形态" | "Type" / "Category" / "Opportunity Type" | 看 select 选项是否含 `project_position` 等关键值 |
| "匹配分" | "Match Score" / "Score" | Text 字段 + 含 % 或 X/N 格式 |
| "Feedback" | "Status" / "Want?" / "想要" | Select 字段 + 选项含"要/不要"或近义 |

无法语义识别的 standard 字段 → 视为缺失，按 7.4 处理。

### 7.3 写入

按映射好的字段名 + `templates/phd-eval-template.md` 正文写一条 Notion page。

- 用户自加的字段（不在 standard 列表里）→ skill 不动，留空
- 用户改 select 选项值（如把"要"→"Want"）→ 写入时同步映射

### 7.4 异常处理

**字段缺失（用户改字段名 + LLM 映射不上 / 用户删了字段）**：

1. 非必需字段（机构 / 城市国家 / PI / 资助 / 命中关键词 / 匹配分 / 备注 等）找不到对应 DB 字段 → 跳过该字段写入，警告但不中止
2. **必需字段缺失**（Title / 链接 / 形态 / 优先级 / Feedback）→ **不要直接中止**，先尝试自动补建：
   - 调当前客户端暴露的 Notion MCP **update-data-source** 工具（常见名 `notion-update-data-source`；注意：补字段是更新 data source schema，不是 database metadata），按 `templates/notion-schema-inbox.md` 的 spec 加回这个 property
   - 补建成功 → 继续写入
   - 补建失败（权限不足 / API 错误）→ 这条候选写本地日志 + 终端报告标红，提示用户去 DB 加回字段

**API 失败重试**：

- 写入失败 → 重试 1 次
- 二次失败 → 重新 fetch schema（用户可能在 scout 跑期间改了 DB）→ 重做映射 → 再试 1 次
- 三次失败 → 只写本地日志，终端报告列出失败项 + DB URL，让用户人工处理

**绝对不要**因为 1 条候选写入失败就中止整次 scout——把失败项标记后，继续处理其他候选。

### 7.5 关于"写到别处"

默认写 Notion。如果用户想写到别处（Obsidian / Logseq / Markdown 文件 / 邮件），**自己改本 Step 的实现**。其余 Step（搜索 / 评估 / 日志）不变。

---

## Step 8 — 写入本地日志

写入：

```
~/.phd-scout/logs/scout-YYYY-MM-DD.yaml
```

日志 schema 可由 AI 调整，但必须能支持 `/phd-keyword-optimize` 归因：

- 本次使用的 A 级 / B 级 / C 级
- 本次使用的 preferred_sources（institutions / sites / regions_focus）
- 搜索 query（含探测搜索那一条）
- 候选 URL
- 命中关键词
- 未命中关键词
- 通过 / 排除原因
- 是否写入 Notion
- 用户后续 feedback 可关联的去重键
- `discovered_sources`：探测搜索发现的新 domain（≥2 次出现的）

---

## Step 9 — 输出报告

```
# PhD Scout Report
日期：YYYY-MM-DD

## 配置
- 形态：...
- 匹配阈值：...
- Notion：可写 / 不可写

## 结果
| 形态 | 候选 | 通过过滤 | 写入 Notion | 最佳 |
|---|---:|---:|---:|---|

## 已写入
- [[title]] — A/B/C · 匹配分 · URL

## 无结果或排除
- <形态>：原因

## 下一步
- 在 Notion Inbox 填 Feedback
- 攒够反馈后运行 /phd-keyword-optimize
```
