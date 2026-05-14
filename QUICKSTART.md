# Quickstart · AI Agent Setup Guide

> **Audience**: AI coding agent (Claude Code / Cursor / Cline / Codex / 其他)
> **Goal**: 把 phd-scout-flow 装好并跑 `/phd-scout-init`
> **Approach**: 4 个 Task 按序执行，只在 checkpoint 问用户。

---

## Task 1 · 收集输入

询问：

> "用来抽关键词的资料路径？建议二选一：
> A. capability-network vault 绝对路径（推荐）
> B. fallback：CV / transcript / SoP 等任意组合的绝对路径
> 也可两个都给。"

存：

```
CAPABILITY_VAULT  # 可能为空
FALLBACK_PATHS    # 可能为空
```

至少一个非空。两个都空 → 让用户先备齐再回来。

---

## Task 2 · 准备本地环境

```bash
mkdir -p "$HOME/.phd-scout/logs" "$HOME/.phd-scout/templates"
cp "$REPO_PATH"/templates/* "$HOME/.phd-scout/templates/"
```

验证：`ls ~/.phd-scout/templates/` 应见 4+ 文件（profile / keywords / eval / notion-schema-inbox）。

---

## Task 3 · 装 skill

按你所在 AI 工具：

### Claude Code

```bash
mkdir -p "$HOME/.claude/skills"
cp -r "$REPO_PATH"/skills/* "$HOME/.claude/skills/"
```

重启 Claude Code 若 slash command 未识别。

### Cursor / Cline / Codex / 其他

把 `skills/phd-scout-init/SKILL.md` 和 `skills/phd-scout/SKILL.md` 内容加入你工具的 rules / instructions 文件（如 `.cursor/rules/`, `.clinerules`, `AGENTS.md`）。Codex 用户：直接 `Read skills/phd-scout-init/SKILL.md and run it`。

---

## Task 4 · 配 Notion + 跑 init

1. 按 [docs/notion-mcp-setup.md](./docs/notion-mcp-setup.md) 装 Notion MCP（约 5 分钟）
2. 跑 `/phd-scout-init`，它会：
   - 抽关键词 + 让你审
   - 跑 6 项配置问卷
   - 半自动建 Notion Inbox DB
   - 写 `profile.yaml` 和 `keywords.md`

### 自检

```bash
[ -s "$HOME/.phd-scout/profile.yaml" ] && echo "profile OK"
[ -s "$HOME/.phd-scout/keywords.md" ] && echo "keywords OK"
grep -q "^## Keywords" "$HOME/.phd-scout/keywords.md"
```

---

## 完成

```
下一步：
  /phd-scout                # 跑第一次搜索（手动）
  /phd-keyword-optimize     # 攒够 Notion 反馈后跑（≥3 条要/不要）
```

---

## Troubleshooting

| 卡点 | 原因 / 修法 |
|---|---|
| Skill not recognized | 重启 AI 工具 / 检查 skill 路径 |
| No keyword source | 提供 CV / transcript / SoP 或 capability vault |
| Notion 写入失败 | OAuth 未完成 / database id 错 / 字段被删 — 报错时检查 |
| 太多 / 太少结果 | 反馈循环慢慢校准 |
