---
name: phd-keyword-optimize
description: 反馈闭环。读 Notion Inbox 的 Feedback + 本地 scout 日志做归因，输出三张表（关键词调整 / 新增 / 调整 preferred_sources）让用户确认，强制人工审批后才写回 keywords.md / profile.yaml。用法：/phd-keyword-optimize
---

# PhD Keyword Optimize

反馈闭环。只根据日志 + Notion Feedback 调整关键词，不凭感觉改。

---

## CONFIG

```
SCOUT_HOME = ~/.phd-scout
PROFILE    = $SCOUT_HOME/profile.yaml
KEYWORDS   = $SCOUT_HOME/keywords.md
LOG_DIR    = $SCOUT_HOME/logs
```

---

## Step 1 — 读取输入

### 本地 scout 日志

扫描 `$LOG_DIR/scout-*.yaml`。无日志 → 仅读 Notion Feedback，无法做关键词命中归因时报"日志不足"，不写回。

### Notion Feedback

读 PhD Inbox 中 Feedback 非空条目：

- `要` → 正向样本
- `不要` → 负向样本
- `观望` → 不计入优化，仅备注供人工参考

**有效样本 < 3 条** → 停止，不写回（防小样本噪音）。

---

## Step 2 — 归因分析

把 Notion feedback 与 scout 日志按 `feedback_key`（title + institution 归一化）匹配。

### 正向样本统计

- 高频 `role: search` / `context` 命中词
- 常出现的形态、机构、站点
- 常出现的国家 / 地区
- 常出现的 funding 信号
- 备注里反复提到的偏好

### 负向样本统计

- 重复出现且导致"不要"的关键词
- 重复出现的地区 / 形态 / funding 问题
- 搜索命中但实际不相关的词
- 出现 ≥ 3 次"不要"的机构 / 站点

**同一负向特征出现 ≥ 3 次** 才列入降级或排除候选。

---

## Step 3 — 输出三张表 → 等用户确认

```
表 1：调整关键词 role
| 关键词 | 当前 role | 建议 role | 证据 | 风险 |
| [kw]   | context  | search    | 正向 N 次 | 提高搜索权重 |
| [kw]   | search   | exclude   | 负向 N 次 | 可能误杀相邻方向 |

表 2：建议新增关键词
| 新词 | role | parent | 证据 |
| [kw] | context | [seed] | 正向样本 N 次 |

表 3：调整 preferred_sources
| 类型 | 项 | 当前 | 建议 | 证据 |
| institutions | [Inst A] | 不在 list | 加进 | 正向 ≥3 次 |
| sites | [example.com] | 在 list | 移除 | 负向 ≥3 次 |
| institutions | [Inst B] | 在 list | 移除 | 负向 ≥3 次 |
```

判断原则：

- 单次反馈不入表，必须**累计 ≥3 次**同一特征才算信号
- 同一对象既正又负 → 看绝对值差，差 < 2 时归为"不动"
- 三类信号都不显著 → 跳过对应表

---

## Step 4 — 人工确认（强制）

输出三张表后等用户明确回复：

- `yes` / `确认` → 全部接受
- `no` / `放弃` → 全部不写
- `accept 1,3` → 只接受指定行
- `edit ...` → 用户手动给修改方案

**没有明确确认，不写任何文件**。

---

## Step 5 — 写回（确认后才执行）

### 5a. 写回 keywords.md（表 1 + 表 2）

```bash
cp "$KEYWORDS" "$KEYWORDS.bak-YYYY-MM-DD"
```

按确认结果更新关键词表：role 改动 / 新增条目 / 移除条目。追加 changelog 行。

### 5b. 写回 profile.yaml（仅当表 3 非空）

```bash
cp "$PROFILE" "$PROFILE.bak-YYYY-MM-DD"
```

更新 `preferred_sources.institutions` / `sites` 列表（追加 / 删除）。不动其他字段。

### 5c. 删已消费的 scout 日志

按文件粒度，明确列举 `consumed_log_paths`，只删全部候选都已成功归因且写回的日志：

```bash
for log_path in "${consumed_log_paths[@]}"; do rm "$log_path"; done
```

未匹配 feedback 的日志保留（备后续 feedback）。

---

## 输出报告

```
# Keyword Optimize Report — YYYY-MM-DD

## 输入
- 日志：N 个 scout-*.yaml
- Feedback：N 条（要 N / 不要 N / 观望 N）

## 结果
- role 调整：N 条
- 新增关键词：N 条
- preferred_sources 调整：N 条

## 写回
- keywords.md: updated / unchanged (backup: <path>)
- profile.yaml: updated / unchanged (backup: <path>)
- 消费日志：N 个删除
```

---

## 设计原则

- 所有建议必须有数据依据（≥3 次累计），不凭主观
- 只输出三张表，没有额外解读
- 写入前**强制人工确认**——AI 不替用户决定关键词改向
- 保留备份 + changelog，万一改错可回滚
