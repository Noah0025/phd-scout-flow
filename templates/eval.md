# PhD Evaluation — Template + Criteria

由 `/phd-scout` 在评估候选时使用：写到 Notion 页面正文 + 决定 A/B/C 评级。

---

## Notion Page Body 模板

```
机构：[institution]
城市/国家：[city] · [country]
形态：[project_position / pi_open_call / pi_cold_email / cdt_dtp / msca_dn / outbound_scholarship / industrial_phd]
链接：[url]
截止日期：[date or unknown]
PI/课题组：[pi_or_group or unknown]
资助类型：[Funded (Stipend / Salary) / Self-funded / Industrial / 未明]

命中关键词（search）：[kw_1, kw_2]
命中关键词（context）：[kw_3]

项目能力点（job_signals，LLM 从描述抽取）：[signal_1, signal_2, ...]
项目覆盖率：[covered]/[total] = [match_rate]%

优先级：[A / B / C]
为何推荐：[一句因果链]
链接验证：[verified / link_unverified]
不确定信息：[列出需人工核实的项；若无写"无"]
```

---

## 评级规则（A / B / C）

**评估视角**：项目角度 — "项目需要什么 / 用户能覆盖多少"。LLM 综合判，不死靠数学阈值。

核心度量：

```
job_signals = LLM 从 PhD 完整描述抽取的关键能力点 (3-8 个)
covered_signals = 用户关键词库（含 search + context）覆盖的 signal 数
match_rate = covered_signals / |job_signals|
```

### A — 优先调研

- `match_rate ≥ 0.6`（参考线）
- `search` 类关键词至少 1 个真实命中（方向真对上）
- 资金清楚、deadline 可行、形态符合
- 链接已验证，cross-check 通过

A = 值得人工深调研。

### B — 可以关注

- `match_rate` 在 [0.3, 0.6)
- `search` 命中弱，但 `context` 方法/技能命中多（方向相邻 + 专业匹配深）
- funding / deadline 信息缺失需人工确认
- 链接验证失败但其他信号强（`link_unverified`）

B = 进 Inbox，但不排在 A 前。

### C — 暂不投入

- `match_rate < 0.3`
- 命中 `exclude` 关键词
- funding 与 profile 硬门槛冲突
- deadline 已过且无 rolling
- 页面 closed / filled
- 信息缺失太多无法判

C 默认不写入 Notion；若写入，必须标明原因。

---

## 判断边界

- 不给录取概率。
- 不因机构名 / 排名单独升 A。
- 数学阈值是参考线，不是死过滤——LLM 综合所有信号判最终评级。

---

## PI Cold Email 形态特殊规则

`pi_cold_email` 候选没有 funding / deadline 字段。Notion 正文改用：

```
PI 姓名：[pi_full_name]
机构 / 课题组：[institution] · [group]
主页：[lab_url]
邮箱：[pi_email or "需在主页找"]
近期方向（≤3 年）：[3-5 篇代表论文标题或一句话研究主题]
方向匹配：[强 / 中 / 弱] — [reason]
形态：pi_cold_email
是否找到 "openings" 段：[Yes / No / Unknown]
为何推荐：[一句因果链]
```

评级规则：

- A：PI 近 3 年论文与 search 关键词强匹配 ≥2 篇 + 主页明示"accepting PhD students"
- B：≥1 篇近 3 年相关，但主页未明示招生（可尝试 cold email）
- C：论文匹配度弱 / 多年无相关产出 / 主页明确"not accepting"

"链接验证" 指 PI 主页可访问 + 邮箱可读。两者都验不了 → 最高 B。
