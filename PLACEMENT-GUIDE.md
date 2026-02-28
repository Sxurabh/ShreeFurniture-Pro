# 📁 New Files — Placement Guide
## Shree Furniture | Where Each New File Goes

---

## Files Generated in This Update

```
shreefurniture-pro/                   ← monorepo root
│
├── CLAUDE.md                         ← 🆕 REPLACE existing (was in NewDocs only)
│                                        Auto-loaded by Claude Code, Cursor, Antigravity
│
├── STATUS.md                         ← 🆕 NEW — update before every AI session
│
├── NewDocs/
│   ├── README.md                     (existing — no change)
│   ├── 01-product-requirements.md    (existing — no change)
│   ├── 02-user-stories-and-acceptance-criteria.md  (existing)
│   ├── 03-information-architecture.md              (existing)
│   ├── 04-system-architecture.md     ← 🆕 REPLACE binary file with this Mermaid version
│   ├── 05-database-schema.md         (existing — no change)
│   ├── 06-api-contracts.md           (existing — no change)
│   ├── 07-monorepo-structure.md      (existing — no change)
│   ├── 08-cart-pricing-engine-spec.md (existing — no change)
│   ├── 09-engineering-scope-definition.md (existing — no change)
│   ├── 10-development-phases.md      (existing — no change)
│   ├── 11-environment-and-devops.md  (existing — no change)
│   ├── 12-testing-strategy.md        (existing — no change)
│   ├── CLAUDE.md                     (existing — keep as backup/reference copy)
│   ├── CONVENTIONS.md                (existing — no change)
│   ├── DECISIONS.md                  (existing — no change)
│   ├── MEDUSA-V2-PATTERNS.md         ← 🆕 NEW
│   └── KNOWN-ISSUES.md               ← 🆕 NEW (update as you encounter bugs)
│
└── .agent/
    ├── ARCHITECTURE.md               (existing — no change)
    ├── mcp_config.json               (existing — no change)
    ├── agents/                       (existing — no change)
    ├── rules/
    │   ├── GEMINI.md                 (existing — no change, keep as base rules)
    │   └── SHREE-FURNITURE.md        ← 🆕 NEW (project-specific Antigravity rules)
    ├── scripts/                      (existing — no change)
    └── skills/                       (existing — no change)
```

---

## Action Checklist

**Step 1 — Root `CLAUDE.md`**
Copy the new `CLAUDE.md` to the monorepo root (same level as `package.json`).
Keep the existing `NewDocs/CLAUDE.md` as-is — it's fine to have both.
IDEs load the root one automatically; the NewDocs one is a backup reference.

**Step 2 — `STATUS.md`**
Place at the monorepo root (same level as `package.json` and `CLAUDE.md`).
Open it and fill in which phase you are actually in right now.
Update the ✅/🚧/⬜ status for anything already built.

**Step 3 — Replace `04-system-architecture.md`**
The original file is a binary (image/PDF) and invisible to AI agents.
Replace it entirely with the new Mermaid text version.
The Mermaid diagrams render in GitHub, Notion, Cursor, and most IDEs.

**Step 4 — Add `MEDUSA-V2-PATTERNS.md`**
Drop into `NewDocs/`. No other config needed.
Add a reference to it in `NewDocs/README.md` under the doc index.

**Step 5 — Add `KNOWN-ISSUES.md`**
Drop into `NewDocs/`. Start adding entries as you hit issues during development.

**Step 6 — `.agent/rules/SHREE-FURNITURE.md`**
Drop into `.agent/rules/` alongside the existing `GEMINI.md`.
Antigravity will load all files in `.agent/rules/` automatically.
No changes needed to `GEMINI.md` — `SHREE-FURNITURE.md` extends it.

---

## One-Line Description of Each New File

| File | One-Line Purpose |
|---|---|
| `CLAUDE.md` (root) | Hard rules + stack + what NOT to build — auto-loaded by every AI IDE |
| `STATUS.md` | What's already built — prevents AI from regenerating existing code |
| `NewDocs/04-system-architecture.md` | Mermaid diagrams replacing the unreadable binary — now AI-readable |
| `NewDocs/MEDUSA-V2-PATTERNS.md` | v1 vs v2 patterns side-by-side — prevents silent breakage from stale training data |
| `NewDocs/KNOWN-ISSUES.md` | Living gotchas log — stops the same bug from being debugged twice |
| `.agent/rules/SHREE-FURNITURE.md` | Antigravity project rules — which agents, which skills, anti-patterns |
