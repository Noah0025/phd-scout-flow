# phd-scout-flow

> 从能力网络出发，自动搜 PhD、初步评估、闭环修正——把精力留给真正值得调研的项目。

---

## 这是什么

一套 AI workflow，帮你把"找 PhD"从**一次次手动 Google** 变成**可累积、可反馈的系统**：

```
能力网络  →  关键词库  →  配置问卷校准  →  搜索 + 初评  →  Notion 反馈  →  关键词库自动调整
                                                                              ↑
                                                                          闭环
```

核心论点：**搜索 PhD 不是关键词搜索，是匹配度过滤**。
反馈环节给"不喜欢"留出口——哪怕匹配度 100%，你说不要就降权。算法负责广度，你负责品味。

---

## 输入资料：越丰富，越像你，找得越准

关键词抽取的准度 = 资料的丰富度 + 资料对"你是谁 / 你能做什么"的描述清晰度。

**推荐方案**：
先跑 [obsidian-capability-network](https://github.com/Noah0025/obsidian-capability-network)（同系列上一步）。它不是"另一个工具"，是**把你的学习/科研轨迹结构化成 Field + Concept 卡**。这套 workflow 读结构化后的 vault，关键词从"已梳理过的能力网络"里提取，比从原始 PDF 里"猜"要准。

**最低门槛**：
没装 capability-network 也能跑，提供以下任一或组合：

- 成绩单（transcript）
- 简历 / CV
- 已写过的 SoP / Personal Statement
- 发表论文清单 + 摘要
- 课程论文 / final project 描述
- 学术主页 / LinkedIn about 段
- 任何能描述你"做过什么 + 能做什么"的文字

**预期效果**（核心规律：资料越丰富 + 越能反映"你具体做过什么 + 能做什么"，关键词越像你）：

| 资料丰富度 | 找出来的方向 |
|---|---|
| 结构化能力网络 + 简历 + SoP + 论文清单 | 接近你的研究轨迹 |
| 简历 + 成绩单 + SoP | 大致方向对得上 |
| 仅简历 | 偏泛，容易和其他人共用词 |
| 一两段描述 | 看运气，靠反馈循环慢慢修 |

→ 资料少也能开始，但首跑结果只是起点——靠反馈循环（`/phd-keyword-optimize`）逐步收敛。
→ 想用得稳，建议补一遍 capability-network，先把"你具备什么"梳清楚。

---

## 为什么这套比手动搜更值得

| 维度 | 手动搜 | 这套 workflow |
|---|---|---|
| **搜索广度** | 1-2 个平台、3-5 个关键词 | 7 类形态 × 多源 × 关键词树（L0/L1/L2） |
| **频率** | 想起来才搜一次 | 可挂 cron 定期跑 |
| **过滤成本** | 每条都点开自己读 | 60% 匹配规则前置过滤 + AI 初评 |
| **沉淀** | 一次性，下次重来 | 反馈 → 关键词库迭代，越用越准 |

→ 你的时间不该花在初筛上，应该花在**真正值得投的项目的深度调研**。

---

## 7 类 PhD 形态（多数人只搜了 1-2 种）

| 形态 | 典型来源 | 适合谁 |
|---|---|---|
| 项目制岗位 | EURAXESS / FindAPhD / jobs.ac.uk | 项目+资金已 listed，省心 |
| PI Open Call | 院系页 / Scholar PI 主页 | 自带方向，看招聘公告 |
| PI Cold Email | Google Scholar / 实验室主页 | 先找 PI 后陶瓷，无现成岗位也试 |
| CDT / DTP | UKRI / 各 CDT 站 | 英国 cohort 制 |
| MSCA Doctoral Network | EURAXESS MSCA filter | 跨国 + 工业合作 |
| 国家奖学金 outbound | CSC / DAAD / Chevening | 自带钱找 PI |
| Industrial PhD | ANRT (CIFRE) / Industriepromotion | 企业资助 |

→ skill 默认全收，profile 里能关掉不要的。

---

## 安装

**给 AI agent**：把 [QUICKSTART.md](./QUICKSTART.md) 整份喂给你的 AI 代理（Claude Code / Cursor / Cline / Codex），它会自动跑完 8 步配置。

**给人类**：跟着 QUICKSTART 一步步来，约 15 分钟。

---

## 仓库结构

```
phd-scout-flow/
├── skills/
│   ├── phd-scout-init/        首启动：抽关键词 + 问卷 + 建 Notion DB
│   ├── phd-scout/             常规搜索 + 评估 + 写 Notion
│   └── phd-keyword-optimize/  反馈闭环：归因 → 三张表 → 关键词库调整
├── templates/                 keywords / profile / eval / log / Notion schema
└── docs/                      Notion MCP 配置教程 + 7 类形态详解
```

---

## 三条命令

```
/phd-scout-init          # 一次性，做完不用再跑
/phd-scout               # 常规搜索，手动或 cron 触发
/phd-keyword-optimize    # 攒够反馈后跑一次，调整关键词库
```

---

## 数据存哪儿

默认本地 markdown，与 skill 同目录附近：

```
~/.phd-scout/
├── profile.yaml        # 配置问卷结果
├── keywords.md         # 关键词库
└── logs/               # 每次搜索的归因日志
```

也可以把 `keywords.md` 显式放到 Notion 数据库里作为 source of truth——
适合在手机上随时改的人。两种都行，自己挑。

Notion Inbox（评估报告）是默认写入目标。如果你想写到别处（Obsidian / Logseq / 邮件 / Markdown 文件），自己改 `skills/phd-scout/SKILL.md` 的 Step 7 即可，其余流程不变。

---

## 设计原则

1. **能力网络是起跑线，不是终点** —— AI 抽关键词后必须经问卷过滤 + 用户审
2. **60% 匹配是推荐值** —— profile 里可调（0.4-0.8）
3. **反馈胜过算法** —— 高匹配但你说不要 = 降权；低匹配但你说想要 = 升权
4. **报告写到 Notion** —— 这是唯一给用户看的地方，其他日志 AI 自己留底
5. **不强绑 Telegram / 邮件** —— 通知渠道用户自配，skill 不假设

---

## 命名契约（ABC 关键词分级）

### 语义

- **A 级** = 优先级最高（搜索优先用 + 评估加权大，默认 weight = 2）。**不是** hard hit 门槛。
- **B 级** = 优先级次之（A 失败时补搜 + 评估支撑分，默认 weight = 1）。
- **C 级** = 硬排除（命中 C 级整体排除候选，profile 里可关）。

评估时用**加权匹配分**：`(2 × A_hits + 1 × B_hits) / (2 × A_total + 1 × B_total) ≥ threshold` 通过过滤。
A 级零命中也不自动判 C 级评级——只要加权分够，仍可能是 B 级。

### 格式（不同文件用不同形式，是有意的）

| 文件 / 场合 | 形式 | 例子 |
|---|---|---|
| `keywords.md` 章节标题 | `## A` / `## B` / `## C` | 用户最常看的视图，简洁 |
| YAML 字段名（profile / log） | `a_level` / `b_level` / `c_level` | 编程友好，无歧义 |
| profile.yaml 问卷 priority 值 | `"A"` / `"B"` / `"C"` | 字符串简短 |
| 文档正文 / 注释 | "A 级关键词" / "B 级关键词" / "C 级关键词" | 中文表达 |

不要在 YAML 字段里写 `## A` 或在 markdown 章节里写 `a_level`——格式跨界会让 grep / 解析错乱。

---

## 不做什么

- 不替你投递（投递是高情境决策，AI 不该替你按"提交"）
- 不深度调研 PI（初评够过滤就行，深调研留给你决定后做）
- 不预测录取概率（不可靠的数字会污染你的判断）
- 不爬付费墙后的内容

---

## License

MIT
