---
name: phd-scout
description: PhD 搜索 + 初评 + 写 Notion / 反馈优化（可选子模式）。读 ~/.phd-scout/profile.yaml 和 keywords.md，按启用形态搜索，验证并评估，写入 Notion Inbox 或本地日志。用法：/phd-scout 或 /phd-scout --review-feedback
---

# PhD Scout

核心搜索流程 5 步：读配置 → 搜索 → 验证过滤 → 评估排序 → 写入。

输入来自本地配置，运行时**不扩展关键词**。

---

## CONFIG

```
SCOUT_HOME = ~/.phd-scout
PROFILE    = $SCOUT_HOME/profile.yaml
KEYWORDS   = $SCOUT_HOME/keywords.md
LOG_DIR    = $SCOUT_HOME/logs
TEMPLATES  = $SCOUT_HOME/templates
```

SKILL 内置默认（profile 未配则用）：

```
queries_per_type: 3
results_per_query: 20
max_written_per_scout: 1
```

---

## Step 1 — 读配置

读 `$PROFILE` + `$KEYWORDS`。检查必需字段：

- `notion.inbox_database_id`
- `opportunity_types.enabled`
- `regions` / `funding` / `language`
- 关键词表（含 `role: search | context | exclude`）

缺失则停止并提示先跑 `/phd-scout-init`。

Notion 不可用时（MCP 未配置 / database_id 占位 / 401-403）→ 跳过远端去重和写入，本地日志兜底；终端报告标"Notion 未连接"。

---

## Step 2 — 搜索

按 `opportunity_types.enabled` 顺序循环。每形态跑约 3 条 query（profile.search.queries_per_type）。

### Query 构造规则

- 只用 `role: search` 的关键词进 query（`context` / `exclude` 不进 query）
- 每条 query 末尾追加 `[current_year] OR [next_year]`，例外：`pi_cold_email` 的 Google Scholar 搜索不加年份

### 形态搜索词模板（占位符 `[search]` = 一个 search 角色关键词）

| 形态 | 默认主源 | 搜索词模板 |
|---|---|---|
| project_position | euraxess.ec.europa.eu / findaphd.com / jobs.ac.uk | `site:euraxess.ec.europa.eu PhD [search] funded` |
| pi_open_call | 院系页 / PI 主页 | `PhD position [search] research group accepting students` |
| pi_cold_email | Google Scholar / 实验室主页 | `[search] author:` (Scholar) → followup: `[pi_name] lab openings` |
| cdt_dtp | UKRI / cdt 站 | `site:ukri.org OR site:jobs.ac.uk doctoral training [search] PhD studentship` |
| msca_dn | EURAXESS MSCA | `site:euraxess.ec.europa.eu MSCA Doctoral Network [search]` |
| outbound_scholarship | CSC / DAAD / Chevening | `site:daad.de OR site:csc.edu.cn PhD [search] supervisor` |
| industrial_phd | ANRT (CIFRE) / industry | `industrial PhD [search] funded` |

### 缩写处理

若 search 关键词是缩写（如 `[abbr]`），query 拼成 `"[full_form]" OR [abbr]`（防 Google 忽略缩写）。

### 偏好源追加搜索

若 `preferred_sources.institutions / sites` 非空：每形态主搜完后追加 1-3 条 query：

- institutions: `[机构名] PhD [search]`
- sites: `site:[domain] PhD [search]`

### pi_cold_email 特殊流程

1. Google Scholar 搜 `[search]`，过滤近 3 年高发文者 → 提取 PI 名 + 机构 + 近期论文
2. 对每个 PI 找其实验室主页 / 院系主页
3. 主页找 "openings" / "join us" / "PhD positions" 段
4. 输出 PI 卡片（不是岗位卡片）—— 用 `templates/eval.md` 的 PI 备用段

### 失败回退

某形态主搜 0 候选 → 用同一关键词的 L1 变体（parent 字段下的同族词）重跑 1 轮。仍 0 → 本形态记 `no_eligible_candidate`，跳过。不无限补搜。

---

## Step 3 — 验证过滤

对每个候选打开原链接验证（WebFetch，失败则浏览器工具兜底）。

### 必须检查

1. **页面存在**且未 closed / filled
2. **deadline** 未过（无 rolling 标记）
3. **Cross-check**：fetched 页面 title / institution 是否与搜索摘要对应 —— 不一致 → 重搜一次精确 query；仍不一致 → 标 `link_unverified`（最高 B 级）
4. **funding rule**：profile 要求 `funded_only` 但候选无 funding 信息 → 标 `待确认`
5. **region / language rule**：硬性不符 → 排除

### 排除条件

- 404 / closed / filled / deadline 已过
- 命中 `role: exclude` 的关键词 → 直接排除
- A 级 + B 级 keyword hits 都为 0（毫无相关度）

---

## Step 4 — 评估排序

读 `templates/eval.md`。LLM 综合判 A/B/C：

### 项目角度覆盖率（核心度量）

```
1. 从 PhD 描述抽 job_signals（3-8 个能力点）
2. 用户 keywords (含 search + context) 覆盖了多少 signal → match_rate
3. LLM 综合判 A/B/C：
   - A: match_rate ≥ 0.6 + search 命中真实 + funding/deadline 通过 + 链接验证通过
   - B: match_rate ∈ [0.3, 0.6) 或链接 unverified 或 funding/deadline 部分缺失
   - C: match_rate < 0.3 / 信息缺失太多 / 命中 exclude
```

### 跨形态去重 + 选 top-N

```
1. 同 URL 在多形态命中 → 合并 1 条候选，记 matched_types
2. 按评级 + match_rate 排序
3. 取前 N 条（N = profile.search.max_written_per_scout，默认 1）
4. 全部为 C → 报"无合格新候选"，不强行写入
```

---

## Step 5 — 写入 Notion + 本地日志

### 写 Notion Inbox

若 Notion MCP 可用且 inbox_database_id 有效：

1. fetch DB schema（`notion-fetch` / `notion-retrieve-database`）
2. LLM 把 `templates/notion-schema-inbox.md` 的 standard 字段名映射到 DB 实际字段（处理用户改名 / 加字段）
3. 按映射写入：核心字段 + 页面正文（`templates/eval.md` 格式）
4. 写入失败 → 重试 1 次；二次失败 → 本地日志 + 终端报告

**不主动改用户 DB schema**——若必需字段缺失，报错并写本地 markdown，让用户自己修。

### 写本地日志

```
~/.phd-scout/logs/scout-YYYY-MM-DD.yaml
```

最小 schema：

```yaml
date: 2026-05-14
queries: [...]
candidates:
  - title: ...
    url: ...
    institution: ...
    type: project_position
    matched_keywords: { search: [...], context: [...] }
    match_rate: 0.57
    priority: B
    decision: written | excluded | needs_check
    feedback_key: "[title_normalized]__[inst_normalized]"  # 关联 Notion feedback 用
```

### 终端报告

```
# PhD Scout Report — YYYY-MM-DD
形态: enabled X 类
候选: 共 N 条，通过过滤 M 条
写入 Notion: K 条（top-N 选最佳）
未通过原因: [closed: 2, deadline_passed: 3, link_unverified: 1, ...]
```

---

## --review-feedback 子模式（可选 - 合并旧 /phd-keyword-optimize）

跑 `/phd-scout --review-feedback`：基于 Notion Feedback + 本地日志做关键词调整。

### 流程

1. 读 Notion Inbox 中 Feedback 非空条目（要 / 不要 / 观望）
2. 与本地 `~/.phd-scout/logs/scout-*.yaml` 关联（按 `feedback_key`）
3. 有效样本 < 3 条 → 停止，不写回
4. 归因分析：
   - 正向样本（要）→ 高频 search / context 命中词 + 高频机构 / 站点
   - 负向样本（不要）→ 重复 ≥ 3 次的特征列入降级 / 排除候选

### 输出三张表 → 等用户确认

```
表 1：调整 role（如 search → context，或 context → exclude）
表 2：建议新增关键词（含 parent 关系）
表 3：调整 preferred_sources（加机构 / 站点 / 移除负向源）
```

接受格式：`yes` / `no` / `accept 1,3` / `edit ...`

### 写回（强制人工确认）

- 备份 `keywords.md` 和 `profile.yaml`
- 按确认结果更新
- 删除已消费的 scout-*.yaml 日志（按文件粒度，全部候选都成功归因才删）

不调用 `--review-feedback` 时这段流程不执行。
