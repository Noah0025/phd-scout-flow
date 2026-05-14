# Notion Schema · Keywords Mirror

这是可选数据库。默认 source of truth 是 `~/.phd-scout/keywords.md`。

| 字段 | 类型 | 选项 | 用途 |
|---|---|---|---|
| Keyword | Title | - | keyword text |
| Level | Select | L0, L1, L2 | keyword level |
| Status | Select | A, B, C | matching role（与 keywords.md 章节标题一致；面向人类） |
| Parent | Relation / Text | - | parent L0 or L1 |
| Evidence | Text | - | input source or feedback evidence |
| Source | Select | init, optimize, manual | where the term came from |
| Last Updated | Date | - | last edit date |
| Notes | Text | - | context and caveats |

---

## Sync Rule

No automatic two-way sync by default.

If this database is used:

- User may edit it as a human-friendly mirror.
- AI must ask before treating it as source of truth.
- Confirmed changes should still be written back to `~/.phd-scout/keywords.md`.
