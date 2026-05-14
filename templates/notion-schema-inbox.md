# Notion Schema · PhD Inbox

Notion Inbox 是搜索结果的主界面。字段名建议保持不变，方便 skill 写入。

| 字段 | 类型 | 选项 | 用途 |
|---|---|---|---|
| Title | Title | - | 岗位或机会标题 |
| 机构 | Text | - | institution full name |
| 城市国家 | Text | - | city + country |
| 形态 | Select | project_position, pi_open_call, pi_cold_email, cdt_dtp, msca_dn, outbound_scholarship, industrial_phd | 机会类型 |
| 链接 | URL | - | 原始页面（PI cold email 时填 PI 主页或实验室主页） |
| 截止 | Date | - | deadline；PI cold email / 未知则留空 |
| PI | Text | - | PI / group / program contact（PI cold email 必填） |
| 资助 | Select | Funded (Stipend), Funded (Salary), Self-funded, Industrial, 未明 | funding signal；PI cold email 默认填"未明" |
| 命中关键词 | Multi-select | 由 AI 写入 | matched keywords |
| 匹配分 | Text | - | `A 级 X/N · B 级 Y/M · 总分 Z%` |
| 优先级 | Select | A, B, C | 初筛等级 |
| Feedback | Select | 要, 不要, 观望 | 用户反馈，供 optimize 使用 |
| 备注 | Text | - | 用户自由记录原因 |
| 创建日期 | Date | - | 写入日期 |

页面正文使用 `templates/phd-eval-template.md`。

---

## 使用约定

- 用户只需要维护 `Feedback` 和 `备注`。
- `/phd-keyword-optimize` 只读取 Feedback 非空条目。
- `观望` 不计入正负样本，但备注会显示在优化报告里。
- **重命名字段安全**：scout 用 LLM 语义映射，把 "机构" 改成 "Institution"、把 "要" 改成 "Want" 都能识别。
- **加新字段安全**：scout 不会动你自加的字段（如个人标签、备注列等）。
- **不要删除核心字段或改字段类型**（如 Text → Number）：会导致写入失败；若发生，scout 会尝试自动补建该字段。
- 调 select 选项颜色 / 顺序：无影响。

---

## PI Cold Email 形态的字段处理

`pi_cold_email` 候选没有 funding / deadline / 岗位标题等字段。在同一 Inbox DB 内异质填法：

| 字段 | PI cold email 怎么填 |
|---|---|
| Title | 用 `[PI 姓名] · [研究方向关键词]` 形式，例 `Dr. Foo · spatial epidemiology` |
| 机构 | PI 所属大学 / 研究所 |
| 链接 | PI 主页 / 实验室主页 |
| 截止 | 留空（没有固定 deadline） |
| PI | **必填**（其他形态可空） |
| 资助 | 默认填 `未明`；若已知用户自带奖学金，可填 `Self-funded` |
| 命中关键词 | 从 PI 近 3 年论文标题中提取 |
| 匹配分 | A 级 / B 级按 PI 论文与关键词命中算分 |

页面正文用 `templates/phd-eval-template.md` 的 **PI Cold Email 备用模板**段，而不是默认正文。
