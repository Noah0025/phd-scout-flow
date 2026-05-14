---
name: phd-keyword-optimize
description: 根据 Notion Inbox feedback 和 ~/.phd-scout/logs/scout-*.yaml 做关键词闭环。输出优先级、新增、降级三张表，必须人工确认后才写回 keywords.md。用法：/phd-keyword-optimize
---

# PhD Keyword Optimize

反馈闭环。只根据日志与 Notion Feedback 调整关键词，不凭感觉改。

---

## CONFIG

```
SCOUT_HOME = ~/.phd-scout
PROFILE    = $SCOUT_HOME/profile.yaml
KEYWORDS   = $SCOUT_HOME/keywords.md
LOG_DIR    = $SCOUT_HOME/logs
```

---

## Step 0 — 读取运行日志

扫描：

```bash
ls "$HOME/.phd-scout/logs"/scout-*.yaml 2>/dev/null
```

若无日志：

- 仍可读取 Notion Feedback。
- 无法做关键词命中归因时，报告“日志不足”，不写回关键词。

日志中至少提取：

- query
- A 级 / B 级 / C 级
- matched keywords
- missing keywords
- candidate title
- institution
- URL
- decision reason
- notion page id 或去重键

---

## Step 1 — 读取 Notion Feedback

从 PhD Inbox 读取 Feedback 非空条目。

有效值：

- `要`
- `不要`
- `观望`

处理规则：

- `要`：正向样本。
- `不要`：负向样本。
- `观望`：不计入优化，只保留备注供人工参考。
- 有效样本少于 3 条 → 停止，不写回。

同时读取备注字段。备注可以解释“为什么要 / 不要”，但不能单独作为写回依据。

---

## Step 2 — 归因分析

把 feedback 与 scout 日志按 URL、Notion page id 或标题+机构匹配。

### 正向样本

统计：

- 高频 A 级 / B 级 命中词
- 常出现的形态
- 常出现的国家或地区
- 常出现的 funding 信号
- 常出现的机构 / 站点 domain（用于建议加进 `preferred_sources`）
- 用户备注中反复出现的偏好

### 负向样本

统计：

- 重复出现且导致”不要”的关键词
- 重复出现的地区 / 形态 / funding 问题
- 搜索命中但实际不相关的词
- 出现 ≥3 次”不要”的机构（用于建议从 `preferred_sources` 移除或加排除）

同一负向特征出现 ≥3 次，才列入降级或排除候选。

---

## Step 3 — 输出三张表

只输出三张表，不直接写文件。

### 表 1：建议调整优先级

| 项目 | 当前 | 建议 | 证据 | 影响 |
|---|---|---|---|---|
| [keyword] | B | A | 正向样本 N 次 | 提高硬筛权重 |

### 表 2：建议新增关键词

| 新词 | 层级 | 归属 L0 | 证据 | 建议分区 |
|---|---|---|---|---|
| [new_keyword] | L1/L2 | [seed] | 正向样本 N 次 | B |

### 表 3：建议降级 / 排除

| 项目 | 当前 | 建议 | 证据 | 风险 |
|---|---|---|---|---|
| [keyword_or_feature] | B | C | 负向样本 N 次 | 可能误杀相邻方向 |

### 表 4（仅当源信号显著时）：建议调整 preferred_sources

输入来自三个地方：

1. Notion Feedback "要"/"不要" 关联到的机构（来自候选评估字段）
2. Scout 日志的 `discovered_sources` 字段（探测搜索发现的新源）
3. 用户备注里反复提到的机构 / 站点

| 类型 | 项目 | 当前 | 建议 | 证据 |
|---|---|---|---|---|
| institutions | [institution_name] | 不在 list | 加进 institutions | 正向样本 ≥3 次 |
| institutions | [institution_name] | 在 institutions | 移除 | 负向样本 ≥3 次 |
| sites | [example.org] | 不在 list（来自 discovered_sources） | 加进 sites | 跨 ≥2 次 scout 出现且正向样本 ≥3 次 |
| sites | [example.org] | 在 sites | 移除 | 负向样本 ≥3 次 |
| regions_focus | [country / region] | 不在 list | 加进 regions_focus | 正向样本 ≥3 次 |
| regions_focus | [country / region] | 在 regions_focus | 移除 | 负向样本 ≥3 次 |

判断原则：

- `discovered_sources` 中单次 scout 出现的 domain 不进入此表（要跨次累积才算信号）
- 用户 "要" 的候选所在 domain → 该 domain 累计正向 +1
- 用户 "不要" 的候选所在 domain → 该 domain 累计负向 +1
- 同一 domain 既正又负 → 优先看绝对值差，差 < 2 时归为"不动"

若三类都无显著信号，跳过这张表。

---

## Step 4 — 人工确认

输出后等待用户明确回复。

接受格式：

- `yes`：全部接受
- `no`：全部放弃
- `accept 1,3`：只接受指定行
- `edit ...`：用户手动给出修改方案

没有明确确认，不写 `keywords.md`。

---

## Step 5 — 写回 keywords.md

确认后：

1. 备份当前文件：

```bash
cp "$HOME/.phd-scout/keywords.md" "$HOME/.phd-scout/keywords.md.bak-YYYY-MM-DD"
```

2. 按确认结果更新：
   - A 级 / B 级 / C 级 分区
   - L0/L1/L2 树
   - changelog

3. 删除本轮已消费日志：

```bash
rm "$HOME/.phd-scout/logs"/scout-*.yaml
```

只删除本轮使用过的 scout 日志。若某日志未能匹配 Notion feedback，保留。

---

## Step 6 — 输出报告

```
# Keyword Optimize Report
日期：YYYY-MM-DD

## 输入
- 日志：N 个
- Feedback：N 条（要 N / 不要 N / 观望 N）

## 结果
- 优先级调整：N
- 新增关键词：N
- 降级 / 排除：N

## 写回
- keywords.md: updated / unchanged
- backup: <path>
- consumed logs: N
```
