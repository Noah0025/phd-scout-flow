# phd-scout-flow

> 从能力网络 / 简历 出发，自动搜 PhD、初步评估、反馈优化——把精力留给真正值得调研的项目。

---

## 这是什么

一套 AI workflow：

```
你的资料 → 关键词库 → 搜索 + 初评 → Notion 反馈 → 关键词库慢慢校准
```

核心论点：**搜索 PhD 不是关键词搜索，是匹配度过滤**。
算法负责广度，你负责品味——反馈环节给"不喜欢"留出口，哪怕匹配度 100%，你说不要就降权。

---

## 输入资料

关键词抽取的准度 = 资料的丰富度 + 资料对"你是谁 / 你能做什么"的描述清晰度。

**推荐**：先跑 [obsidian-capability-network](https://github.com/Noah0025/obsidian-capability-network)（同系列上一步），把学习/科研轨迹结构化后再来。

**最低门槛**：CV / transcript / SoP / 论文清单 / 课程项目描述 / 学术主页文本 任意组合。

---

## 三条命令

```
/phd-scout-init          # 一次性：抽关键词 + 配置问卷 + 建 Notion Inbox
/phd-scout               # 常规搜索 + 评估 + 写 Notion（手动 or cron）
/phd-scout --review-feedback   # 攒够反馈后跑，调整关键词库
```

---

## 关键词模型（一维 role）

每个关键词只有一个 `role`：

- **`search`** — 进搜索 query 的核心方向词（问题域 / 应用领域，PhD 招聘描述高频出现）
- **`context`** — 评估时背景信号（方法 / 技能 / 邻域词，不进 query 避免跑偏）
- **`exclude`** — 不要的方向，命中即排除候选

---

## 7 类 PhD 形态

| 形态 | 典型来源 |
|---|---|
| project_position | EURAXESS / FindAPhD / jobs.ac.uk |
| pi_open_call | 院系页 / Scholar PI 主页 |
| pi_cold_email | Google Scholar → 实验室主页（输出 PI 卡片）|
| cdt_dtp | UKRI / 各 CDT 站 |
| msca_dn | EURAXESS MSCA filter |
| outbound_scholarship | CSC / DAAD / Chevening |
| industrial_phd | ANRT (CIFRE) / Industriepromotion |

默认 enabled 仅 `project_position + pi_open_call`，init 问卷可加其他。

---

## 数据位置

```
~/.phd-scout/
├── profile.yaml        # 配置 + 偏好（~15 行精简）
├── keywords.md         # 关键词库（一张表，role 维度）
└── logs/               # 每次搜索的归因日志
```

Notion Inbox 是默认评估写入目标。想写到别处（Obsidian / Markdown / 邮件）→ 改 `skills/phd-scout/SKILL.md` 的 Step 5 即可。

---

## 设计原则

1. **能力网络是起跑线，不是终点** —— AI 抽完关键词必须经用户审
2. **反馈胜过算法** —— 高匹配但你说不要 = 降权；低匹配但你说想要 = 升权
3. **报告写到 Notion** —— 这是唯一给用户看的地方
4. **不替你投递** —— 投递是高情境决策，AI 不替你按"提交"

---

## 安装

**给 AI agent**：把 [QUICKSTART.md](./QUICKSTART.md) 整份喂给你的 AI 代理。

**给人类**：跟 QUICKSTART 一步步来，约 10 分钟。

---

## License

MIT
