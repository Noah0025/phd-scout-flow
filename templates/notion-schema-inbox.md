# Notion Schema · PhD Inbox

Notion Inbox 是搜索结果的主界面。字段名建议保持不变，方便 skill 写入。

| 字段 | 类型 | 选项 | 用途 |
|---|---|---|---|
| Title | Title | - | 岗位或机会标题 |
| 机构 | Text | - | institution full name |
| 城市国家 | Text | - | city + country |
| 形态 | Select | project_position, pi_open_call, cdt_dtp, msca_dn, outbound_scholarship, industrial_phd | 机会类型 |
| 链接 | URL | - | 原始页面 |
| 截止 | Date | - | deadline；未知则留空 |
| PI | Text | - | PI / group / program contact |
| 资助 | Select | Funded (Stipend), Funded (Salary), Self-funded, Industrial, 未明 | funding signal |
| 命中关键词 | Multi-select | 由 AI 写入 | matched keywords |
| 匹配分 | Text | - | `硬性 X/N · 支撑 Y/M · 总分 Z%` |
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
- 字段可以增加，但删除或重命名上述字段会影响自动写入。
