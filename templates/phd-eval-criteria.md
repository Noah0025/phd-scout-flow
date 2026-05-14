# PhD Evaluation Criteria

用于 `/phd-scout` 给候选机会打 A/B/C 评级。规则服务于初筛，不代表录取概率。

⚠️ **评估视角**：项目角度 — 看"项目需要的能力 / 用户能覆盖多少"，不是"用户库 / 项目命中"。

LLM 综合判，不靠硬数学阈值。

---

## 核心度量：match_rate（项目角度覆盖率）

```
job_signals = LLM 从 PhD 完整描述抽取的关键能力点 (3-8 个)
covered_signals = 用户关键词库（含 A/B 级 + search_anchor/weight_only）覆盖的 signal 数
match_rate = covered_signals / |job_signals|
```

---

## A — 优先调研

LLM 综合判（不是死阈值）：

- `match_rate ≥ 0.6` 是参考线（覆盖 ≥ 60% job_signals）
- A 级 search_anchor 至少 1 个真实命中（方向对上，不是边缘擦边）
- 资金信息清楚，符合 profile 的 funding rule
- 开始时间窗和截止日期可行
- 形态符合用户启用的 opportunity type
- 原链接已验证，cross-check 通过

A 的含义：值得进入人工深调研。

---

## B — 可以关注

满足大部分条件，但存在一条明显不确定或轻微不符：

- `match_rate` 在 [0.3, 0.6) 之间（部分覆盖，但有方向相邻信号）
- A 级 search_anchor 命中弱，但 weight_only 方法类命中多（方向相邻 + 专业匹配深）
- funding 写法不清楚，需要人工确认
- deadline / start window 信息缺失
- 链接可访问但页面信息分散
- 地区或语言规则需要进一步确认
- 链接验证失败但其他信号强（Step 5 标 `link_unverified`）

B 的含义：可以进 Inbox，但不要排在 A 前面。

---

## C — 暂不投入

出现任一情况：

- `match_rate < 0.3`（覆盖太少）
- A 级 + B 级命中都为 0（毫无相关度，Step 4 已排除）
- funding 与 profile 硬门槛冲突
- deadline 已过，且没有 rolling / open call 信息
- 页面显示 closed / filled
- 信息缺失太多，无法判断
- 显著命中 C 级关键词（profile.matching.c_level_action=downrank 时降到 C 评级；=exclude 时整体排除）
- 与负向反馈特征高度重合

C 的含义：通常不写入 Notion；若写入，必须标明原因。

---

## 判断边界

- 不给录取概率。
- 不因为机构名或排名单独升为 A。
- 不因为关键词数量多就自动升为 A；关键词必须和岗位主题发生真实关系。
- 不能验证链接时，最高为 B。
- **数学阈值（0.3 / 0.6）是参考线，不是死过滤**——LLM 综合所有信号（match_rate + A_hits 真实性 + funding 完整度 + 形态对齐）判最终评级。

---

## PI Cold Email 形态的特殊规则

`pi_cold_email` 候选没有 funding / deadline 字段，不能套用上面 A/B/C 规则。改用：

- **A**：PI 近期论文与 A 级关键词 强匹配（≥2 篇近 3 年相关）+ 主页明确写"accepting PhD students" / 有 openings 段
- **B**：PI 近期论文与 A 级关键词 相关（≥1 篇近 3 年）但**主页未明示招生**——可以尝试 cold email
- **C**：论文匹配度弱 / 多年无相关产出 / 课题组明确写"not accepting"

cold email 形态的"链接验证"指的是 **PI 主页可访问 + 邮箱可读取**。两者都验不了 → 最高 B。
