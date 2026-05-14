# Quickstart · AI Agent Setup Guide

> **Audience**: AI coding agent (Claude Code / Cursor / Cline / Codex / other).
> **Goal**: Install phd-scout-flow and run first configuration.
> **Approach**: Read this entire file first. Execute tasks in order. Ask user only at marked checkpoints.

---

## Pre-flight · Detect environment + locate repo

### 0a. Detect AI tool

- Claude Code -> `tool = claude_code`
- Cursor -> `tool = cursor`
- Cline -> `tool = cline`
- Codex -> `tool = codex`
- Otherwise -> `tool = other`

If unclear, ask:

> Which AI tool are you using to set this up? (Claude Code / Cursor / Cline / Codex / other)

### 0b. Locate this repo

Set `REPO_PATH` to the absolute path containing this `QUICKSTART.md`.

Verify:

```bash
[ -f "$REPO_PATH/QUICKSTART.md" ] || echo ERROR
```

---

## Task 1 · Collect inputs

Ask:

> Do you have an obsidian-capability-network vault to use as keyword source? If yes, paste its absolute path. If no, paste one or more fallback files/folders such as CV, transcript, SoP, paper list, or project descriptions.

Save:

```bash
export CAPABILITY_VAULT="<path-or-empty>"
export FALLBACK_PATHS="<paths-or-empty>"
```

At least one source must exist.

---

## Task 2 · Create local data directory

```bash
mkdir -p "$HOME/.phd-scout/logs" "$HOME/.phd-scout/templates"
```

Verify:

```bash
[ -d "$HOME/.phd-scout" ] && [ -d "$HOME/.phd-scout/logs" ] && [ -d "$HOME/.phd-scout/templates" ]
```

---

## Task 3 · Install templates

```bash
cp "$REPO_PATH"/templates/* "$HOME/.phd-scout/templates/"
```

Verify:

```bash
ls "$HOME/.phd-scout/templates/"
```

Expected files:

- `phd-profile.yaml.template`
- `phd-keywords.md.template`
- `phd-eval-template.md`
- `phd-eval-criteria.md`
- `notion-schema-inbox.md`
- `notion-schema-keywords.md`
- `scout-log-example.yaml`

---

## Task 4 · Install skills

Branch by `tool`.

### tool == claude_code

```bash
mkdir -p "$HOME/.claude/skills"
cp -r "$REPO_PATH"/skills/* "$HOME/.claude/skills/"
```

Tell user to restart Claude Code if slash commands are not visible.

### tool == cursor

```bash
mkdir -p "$REPO_PATH/.cursor/rules"
for skill_dir in "$REPO_PATH"/skills/*/; do
  skill_name=$(basename "$skill_dir")
  out="$REPO_PATH/.cursor/rules/${skill_name}.mdc"
  {
    echo "---"
    echo "description: ${skill_name}"
    echo "alwaysApply: true"
    echo "---"
    echo
    cat "$skill_dir/SKILL.md"
  } > "$out"
done
```

Tell user to open this repo or their working folder in Cursor.

### tool == cline

```bash
out="$REPO_PATH/.clinerules"
echo "# phd-scout-flow · merged skills" > "$out"
for skill_dir in "$REPO_PATH"/skills/*/; do
  skill_name=$(basename "$skill_dir")
  echo >> "$out"
  echo "---" >> "$out"
  echo >> "$out"
  echo "# === SKILL: ${skill_name} ===" >> "$out"
  echo >> "$out"
  cat "$skill_dir/SKILL.md" >> "$out"
done
```

### tool == codex

Codex can read repo instructions from `AGENTS.md`, but this repo does not ship one. For setup, tell the user to invoke the skill by naming the file, for example:

```
Read skills/phd-scout-init/SKILL.md and run /phd-scout-init
```

If the user wants persistent Codex instructions, create them outside this repo or in their own workspace.

### tool == other

Tell user to load the relevant `skills/*/SKILL.md` content into their AI tool when invoking that command.

---

## Task 5 · Configure Notion MCP

Read:

```bash
cat "$REPO_PATH/docs/notion-mcp-setup.md"
```

Ask:

> Do you already have a Notion Inbox database for PhD results? If yes, paste the database id. If no, create one using templates/notion-schema-inbox.md, then paste the database id.

If Notion MCP is not ready, continue with local setup and mark Notion as pending.

---

## Task 6 · Run first configuration

Invoke:

```
/phd-scout-init
```

The skill must:

1. Read capability-network vault or fallback files.
2. Extract L0 seed.
3. Expand L1/L2.
4. Run the 6 dimensional questionnaire.
5. Write `~/.phd-scout/profile.yaml`.
6. Write `~/.phd-scout/keywords.md`.
7. Insert Notion Inbox database id if available.

Verify:

```bash
[ -f "$HOME/.phd-scout/profile.yaml" ] || echo "missing profile"
[ -f "$HOME/.phd-scout/keywords.md" ] || echo "missing keywords"
```

---

## Task 7 · Self-check

Check local files:

```bash
test -s "$HOME/.phd-scout/profile.yaml"
test -s "$HOME/.phd-scout/keywords.md"
grep -q "threshold:" "$HOME/.phd-scout/profile.yaml"
grep -q "^## A" "$HOME/.phd-scout/keywords.md" && grep -q "^## B" "$HOME/.phd-scout/keywords.md" && grep -q "^## C" "$HOME/.phd-scout/keywords.md"
```

Optional smoke test:

```
/phd-scout
```

If Notion is pending, `/phd-scout` should still write a local log and print a terminal report.

---

## Task 8 · Report completion

Output:

```
phd-scout-flow installed.

Repo: <REPO_PATH>
Tool: <tool>
Local home: ~/.phd-scout/
Profile: ~/.phd-scout/profile.yaml
Keywords: ~/.phd-scout/keywords.md
Notion Inbox: configured / pending

Next:
- /phd-scout
- Fill Feedback in Notion Inbox
- /phd-keyword-optimize
```

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Skill not recognized | Tool did not load local skill files | Restart tool or paste SKILL.md directly |
| No keyword source | Missing vault/fallback path | Provide CV, transcript, SoP, or capability vault |
| Notion write fails | MCP not authenticated or database id wrong | Re-run OAuth and verify database id |
| Too many weak results | Keywords too broad | Add negative feedback and run optimize |
| No results | Keywords too narrow or shape disabled | Review profile and enabled opportunity types |
