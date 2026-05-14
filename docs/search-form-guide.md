# Search Form Guide

`/phd-scout` 覆盖 7 类 PhD 形态。profile 里可以开关，不需要每次都搜全部。

> 设计前提：搜得比用户快、广，第三视角，消除信息差。
> 默认主源 + 用户偏好源 + 探测搜索，三层覆盖。

---

## 1. 项目制岗位

典型来源：

- EURAXESS
- FindAPhD
- jobs.ac.uk
- 大学 job board

适合：

- 想看清楚项目、资金、deadline 后再决定的人。
- 方向能接受一定范围内匹配的人。

关键过滤信号：

- funding 是否写明
- deadline
- start date
- project description 是否命中 A 级关键词
- eligibility

搜索词模板：

```
PhD [a_level_keyword] [b_level_keyword] funded
[a_level_keyword] PhD studentship [country_or_region]
site:euraxess.ec.europa.eu [a_level_keyword] PhD
```

---

## 2. PI Open Call

典型来源：

- 院系 faculty page
- PI / group homepage
- lab news
- 招生页面

适合：

- 方向较明确，愿意主动联系 PI 的人。
- 可以接受信息不如岗位广告完整的人。

关键过滤信号：

- PI 是否明确欢迎 PhD inquiry
- group 近期主题是否匹配
- funding pathway 是否清楚
- 是否需要自带资金

搜索词模板：

```
[a_level_keyword] research group PhD openings
[a_level_keyword] prospective PhD students supervisor
[a_level_keyword] lab PhD opportunity
```

---

## 3. CDT / DTP

典型来源：

- doctoral training centre 页面
- cohort program 页面
- funder listing

适合：

- 接受 cohort 制和课程/轮转结构的人。
- 想在较宽方向里选 project 的人。

关键过滤信号：

- cohort topic 是否覆盖 A 级关键词
- 是否有明确 project list
- funding eligibility
- training structure

搜索词模板：

```
doctoral training [a_level_keyword] PhD studentship
CDT [a_level_keyword] PhD
DTP [a_level_keyword] funded PhD
```

---

## 4. MSCA Doctoral Network

典型来源：

- EURAXESS
- consortium page
- partner university pages

适合：

- 接受跨国流动、联合培养或工业合作的人。
- 想要结构化项目和明确 funding 的人。

关键过滤信号：

- mobility rule
- host country
- secondment
- research package 是否匹配
- application deadline

搜索词模板：

```
MSCA Doctoral Network [a_level_keyword] PhD
site:euraxess.ec.europa.eu MSCA [a_level_keyword] doctoral candidate
```

---

## 5. 国家奖学金 Outbound

典型来源：

- scholarship page
- destination university supervisor page
- graduate school admission page

适合：

- 已有或计划申请外部 funding 的人。
- 想先锁定 PI / group，再处理资金路径的人。

关键过滤信号：

- scholarship eligibility
- host acceptance requirement
- supervisor fit
- deadline 是否能和 admission 对齐

搜索词模板：

```
[scholarship_name] PhD [a_level_keyword] supervisor
[a_level_keyword] PhD supervisor accepting students
external funding PhD [a_level_keyword]
```

---

## 6. PI Cold Email

跟 PI Open Call 相邻但不一样：那个是**找招聘公告**（被动），这个是**先找 PI 后陶瓷**（主动）。

典型来源：

- Google Scholar（近 3 年活跃发文者）
- 实验室 / 课题组主页
- 院系 faculty page（找方向匹配的 PI）
- 个人学术主页 / ORCID

适合：

- 方向明确，知道自己要解决什么问题的人
- 愿意先识别 PI 再主动联系，能接受"没有现成岗位"的人
- 自带或能申请奖学金的人（CSC / DAAD / Chevening 等）

关键过滤信号：

- PI 近 3 年是否在 A 级关键词 方向有真实产出（看论文，不看主页措辞）
- 课题组主页是否有 "openings" / "join us" / "PhD positions" 段
- 找不到 openings 段也可以发，但要做好"不一定有回音"的预期

输出**不是岗位卡片**，是 PI 卡片（见 `templates/phd-eval-template.md` 备用模板）。

搜索词模板：

```
[a_level_keyword] author:                    # Google Scholar 找活跃发文者
[pi_name] research group homepage
[pi_name] lab openings
site:[university_domain] [pi_name] PhD positions
```

---

## 7. Industrial PhD

典型来源：

- company career page
- university-industry program page
- public-private doctoral program

适合：

- 接受应用导向、企业合作和交付约束的人。
- 希望研究问题靠近真实场景的人。

关键过滤信号：

- employment status
- IP / publication constraint
- university supervisor 是否明确
- funding and contract
- company problem 是否真实匹配方向

搜索词模板：

```
industrial PhD [a_level_keyword] funded
industry doctoral [a_level_keyword]
company PhD position [a_level_keyword]
```

---

## 使用原则

- 形态不是优劣排序，只是入口不同。
- profile 可以关闭不想看的形态。
- 运行结果靠 Feedback 收敛，不靠第一次搜索定型。

---

## 搜索策略：广度优先 + 预算控 + 质量收敛

`/phd-scout` 每次跑都按下面四条规则执行，避免"无限制全网搜"。

### 1. 资源预算（硬上限）

| 参数 | 默认 | 含义 |
|---|---|---|
| `total_queries_per_scout` | 30 | 一次 scout 总查询数上限 |
| `queries_per_type` | 3 | 每形态主查询数（3 轮组合） |
| `preferred_source_queries` | 3 | 每形态偏好源追加查询数 |
| `probe_queries` | 1 | 探测搜索固定 1 次 |
| `results_per_query` | 20 | 每查询取前 N 条 |

超出总预算时，从 priority `low` 层开始砍。`focus` 层永远保留。`probe_queries` 不受挤压。

### 2. 形态优先级

profile 把 7 类形态分成 focus / normal / low。预算先分给 focus，依次到 low。

```yaml
opportunity_types:
  priority:
    focus: [project_position, pi_open_call]
    normal: [cdt_dtp, msca_dn]
    low: [pi_cold_email, outbound_scholarship, industrial_phd]
```

→ 默认 focus 是最常见两类，低预算时仍保覆盖。

### 3. 关键词组合（每形态 3 轮）

```
轮 1: 主 A 级 × 形态词模板          (最广覆盖)
轮 2: 主 A 级 × top-2 B 级          (适中聚焦)
轮 3: 第二 A 级 × 形态词模板        (覆盖另一方向)
```

不做 A × A（太窄）。B 级只配 A 级。

### 4. 失败定义与回退

- 失败 = 3 轮累计候选 < 2 条 或 全被过滤
- 回退：换 L1 → 用 B 级补 1 轮（不算 A 级）→ 仍失败就放弃此形态
- 不无限补搜——保护预算

### 5. 跨形态去重

同 URL 在多形态命中 → 合并为 1 候选，记 `matched_types: [...]`，Notion 写 1 条。

### 6. 时效性

`append_year_filter: true` 时，所有查询词追加 `[current_year] OR [next_year]`。
例外：`pi_cold_email` 的 Scholar 查询不加（影响 author filter）。

### 7. 跨次稳定性

同一 query 字符串在 `query_repeat_cooldown_days`（默认 14 天）内不重跑——避免 cron 调度时浪费预算。

### 8. 写入数量上限

`max_total_written_per_scout`（默认 1）决定一次 scout 最多向 Notion 写多少条。

不是"每形态各写 1 条"——是**跨所有形态选总 N 条最佳**。

| N | 适合什么场景 | 风险 |
|---|---|---|
| **1（推荐）** | 每天/每周看 1 条最优候选，认真读、给精准反馈 | 长尾候选看不到 |
| 2-3 | 想多看几个选项做对比 | 反馈成本上升 |
| ≥5 | 信息丰富 | Inbox 快速淹没，反馈疲劳，optimize 信号变弱 |

→ 形态扩展（开 7 类）和写入数量（默认 1）是**正交**的：搜得广是为了覆盖，写得少是为了反馈质量。两者不冲突。

---

## 广度策略：三层源池

PhD 项目不是都发布在 EURAXESS / FindAPhD 这些主源上——区域性聚合站、机构官网、学科平台都可能漏掉。`/phd-scout` 用三层覆盖：

### 一级：默认主源（写死，每次都搜）

7 类形态各自绑定的典型来源（见上文每类的"典型来源"段）。

### 二级：用户偏好源（init 阶段勾选 + optimize 阶段沉淀）

`profile.yaml` 的 `preferred_sources`：

- `institutions`：心仪机构
- `sites`：学科 / 区域性聚合站 domain
- `regions_focus`：强化区域信号

来源：

1. `/phd-scout-init` Step 4.5：AI 根据问卷推荐候选（机构 / 学科平台 / 区域站），用户勾选
2. `/phd-keyword-optimize` 表 4：用户反馈 ≥3 次"要"的同一源，建议加入

### 三级：探测搜索（每次 1 次，发现新源）

`/phd-scout` Step 3.6：每次跑一次通用 Google 查询（不带 site filter），看是否有新平台 / 新机构出现 ≥2 次。

发现物：

- 写入日志 `discovered_sources` 字段
- 不自动加进 preferred_sources（保持可预测）
- 用户在 Notion 标"要" ≥3 次该源候选 → optimize 表 4 建议沉淀

### 为什么这样设计

- 写死主源 → 漏长尾平台
- 全网无限制搜 → 噪音爆炸 + 成本高
- 用户自己维护源清单 → 重新创造中介成本

三层 = 主源稳 + 用户偏好可控 + 探测发现长尾，闭环自然收敛。
