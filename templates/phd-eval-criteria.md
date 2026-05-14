# PhD Evaluation Criteria

用于 `/phd-scout` 给候选机会打 A/B/C 评级。规则服务于初筛，不代表录取概率。

⚠️ 关键词分级（A 级 / B 级 / C 级）是**优先级 + 加权**，不是 hard hit 门槛。评级分以下三档基于加权分 + 其他条件综合判断。

---

## A — 优先调研

同时满足：

- 加权匹配分 ≥ 0.75（A 级命中多 + B 级覆盖好）。
- 资金信息清楚，符合 profile 的 funding rule。
- 开始时间窗和截止日期可行。
- 形态符合用户启用的 opportunity type。
- 原链接已验证，核心信息不依赖搜索摘要。

A 的含义：值得进入人工深调研。

---

## B — 可以关注

满足大部分条件，但存在一条明显不确定或轻微不符：

- 加权匹配分在 [threshold, 0.75) 之间（够通过过滤，但不够强）。
- funding 写法不清楚，需要人工确认。
- deadline / start window 信息缺失。
- A 级命中弱但 B 级覆盖好（方向相邻而非正中）。
- 链接可访问但页面信息分散。
- 地区或语言规则需要进一步确认。

B 的含义：可以进 Inbox，但不要排在 A 前面。

---

## C — 暂不投入

出现任一情况：

- A 级 + B 级命中都很少（加权分接近或低于 threshold）。
- funding 与 profile 硬门槛冲突。
- deadline 已过，且没有 rolling / open call 信息。
- 页面显示 closed / filled。
- 信息缺失太多，无法判断。
- 命中 C 级关键词（profile.c_level_is_hard_exclude=true 时直接排除，否则降级到 C）。
- 与负向反馈特征高度重合。

C 的含义：通常不写入 Notion；若写入，必须标明原因。

---

## 判断边界

- 不给录取概率。
- 不因为机构名或排名单独升为 A。
- 不因为关键词数量多就自动升为 A；关键词必须和岗位主题发生真实关系。
- 不能验证链接时，最高为 B。
- **A 级零命中也不自动判 C**：只要加权分通过 threshold（B 级覆盖好），仍可能是 B 级评级。最终评级看综合分 + 信息完整度。

---

## PI Cold Email 形态的特殊规则

`pi_cold_email` 候选没有 funding / deadline 字段，不能套用上面 A/B/C 规则。改用：

- **A**：PI 近期论文与 A 级关键词 强匹配（≥2 篇近 3 年相关）+ 主页明确写"accepting PhD students" / 有 openings 段
- **B**：PI 近期论文与 A 级关键词 相关（≥1 篇近 3 年）但**主页未明示招生**——可以尝试 cold email
- **C**：论文匹配度弱 / 多年无相关产出 / 课题组明确写"not accepting"

cold email 形态的"链接验证"指的是 **PI 主页可访问 + 邮箱可读取**。两者都验不了 → 最高 B。
