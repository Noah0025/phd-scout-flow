---
name: phd-scout
description: 常规 PhD 搜索与初评。读取 ~/.phd-scout/profile.yaml 和 keywords.md，按 6 类形态搜索，执行 60% 匹配过滤，验链，生成评估，写入 Notion Inbox 与本地日志。用法：/phd-scout
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
   - enabled opportunity types
   - match threshold
   - funding rule
   - region rule
   - language rule
   - must keywords
   - nice keywords
   - exclude keywords

缺失 `profile.yaml` 或 `keywords.md` 时，停止并提示先运行 `/phd-scout-init`。

---

## Step 1 — 读取 Notion Inbox 去重基准

从 Notion Inbox 读取未过期、未明确拒绝的条目。

去重键：

```
<title_normalized> + <institution_normalized>
```

若 Notion MCP 不可用：

- 跳过远端去重。
- 使用本地日志中的历史 URL 和标题做弱去重。
- 在最终报告中标记 Notion 未写入。

---

## Step 2 — 生成搜索队列

按 profile 中启用的形态循环。

| 形态 | 典型来源 | 搜索词模板 |
|---|---|---|
| project_position | EURAXESS / FindAPhD / jobs.ac.uk | `PhD [must_keyword] [nice_keyword] funded` |
| pi_open_call | 院系页 / PI 主页 | `[must_keyword] PhD position research group` |
| cdt_dtp | CDT / DTP 页面 | `doctoral training [must_keyword] PhD studentship` |
| msca_dn | EURAXESS MSCA filter | `MSCA Doctoral Network [must_keyword] PhD` |
| outbound_scholarship | 奖学金 + 院系页 | `[scholarship_name] PhD [must_keyword] supervisor` |
| industrial_phd | 企业 / 大学联合岗位 | `industrial PhD [must_keyword] funded` |

规则：

- 只使用 `keywords.md` 中的词。
- 不根据搜索结果临时发明新关键词。
- 每类形态最多保留候选若干条，后续过滤后只推荐最佳 1 条。

---

## Step 3 — 搜索与候选提取

每类形态至少跑一轮搜索。若无结果：

1. 换用同一 must 关键词的 L1 变体。
2. 换用另一个 must 关键词。
3. 仍无结果时，用 nice 关键词补搜，但不能把 nice 当作硬性命中。

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

## Step 4 — 60% 匹配过滤

过滤规则：

1. 至少命中 1 个 `must` 关键词。
2. 支撑关键词命中率 `matched_nice / total_relevant_nice >= match_threshold`。
3. 不命中 `exclude` 中的硬排除词。
4. 满足 funding rule。
5. 满足 region rule 和 language rule；若信息缺失，降级为待确认，不直接通过 A 级。

匹配分格式：

```
硬性 X/N · 支撑 Y/M · 总 Z%
```

---

## Step 5 — 验链

对通过过滤的候选逐一打开原链接验证：

- 页面存在
- 岗位仍开放或信息未过期
- title / institution 与候选一致
- funding / deadline 没有被搜索摘要误读

若普通抓取失败：

- 尝试用可用浏览器工具打开。
- 仍失败则标记为 `link_unverified`，优先级最高为 B。

404、closed、filled、deadline 已过且无 rolling 标记 → 排除。

---

## Step 6 — 评估与选择

读取：

- `templates/phd-eval-template.md`
- `templates/phd-eval-criteria.md`

每个启用形态最多选择 1 条最佳候选。若某形态全部为 C 或被排除，记录“无合格新候选”。

评估必须包含：

- 形态
- 匹配分
- 为何推荐（一句因果链）
- A/B/C 优先级
- 不确定信息
- 下一步人工检查点

不预测录取概率。

---

## Step 7 — 写入 Notion Inbox

若 Notion MCP 可用且 profile 中有 database id：

1. 按 `templates/notion-schema-inbox.md` 字段写入。
2. 页面正文使用 `templates/phd-eval-template.md`。
3. 写入失败重试 1 次。
4. 仍失败则只写本地日志，并在终端报告中列出失败项。

字段名以模板为准，不临时新增字段。

---

## Step 8 — 写入本地日志

写入：

```
~/.phd-scout/logs/scout-YYYY-MM-DD.yaml
```

日志 schema 可由 AI 调整，但必须能支持 `/phd-keyword-optimize` 归因：

- 本次使用的 must / nice / exclude
- 搜索 query
- 候选 URL
- 命中关键词
- 未命中关键词
- 通过 / 排除原因
- 是否写入 Notion
- 用户后续 feedback 可关联的去重键

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
