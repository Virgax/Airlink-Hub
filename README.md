# Airlink Platform

Internal monorepo for Airlink Distribution DR's AI-driven operations platform.

> 🤖 **If you are Claude Code:** read `CLAUDE.md` first. It is the project's master context.

---

## What lives here

This is the working repo for Jaime Estevez (Director of Software & IT) and the Software team at Airlink. It coordinates work across multiple deployed systems:

- **Airlink Hub** — Electron desktop app (`airlink-desktop/`)
- **airlink-fb-svc** — Node.js Fishbowl bridge (`D:\airlink-fb-svc\` on `192.168.181.246`)
- **Nexus** — Movement-API (.NET 8) + Movement-Fend (Next.js 14)
- **Power Automate** flows — `default-776b130e-a1ea-446a-a1f6-793c70bd6f82` environment
- **SQL Server** — `192.168.181.248:13999` (AIRLINK, AirlinkDR, SPN); dev on `192.168.181.246:8989`

---

## Repo layout

```
.
├── CLAUDE.md                                    ← Master context (read first)
├── README.md                                    ← This file
├── docs/
│   ├── Mi_Bitacora_Airlink.md                   ← Daily continuity log
│   ├── Hub_WholeParts_State_2026-05-01.md       ← Active sprint state
│   ├── Dino_Ecosystem_Reference.md              ← 11 DINO agents reference
│   ├── DB_Schema_Reference.txt                  ← Schema canonical
│   ├── Hub_SQL_Schema.md                        ← Formal SQL schema
│   ├── Nexus_Integration_Documentation_v2.md    ← Sister system
│   ├── PA_Flow_Checklists.md                    ← PA flow build patterns
│   └── LX_Agent_vFinal.html                     ← LX Agent template
└── legacy/
    └── Movement_V1_5.xlsm                        ← Excel VBA system being migrated
```

---

## Quick links

| If you need to | Read |
|---|---|
| Get oriented | `CLAUDE.md` |
| Know what sprint is active | `docs/Hub_WholeParts_State_2026-05-01.md` |
| Plan a DINO agent | `docs/Dino_Ecosystem_Reference.md` |
| Look up a SQL column | `docs/DB_Schema_Reference.txt` |
| Build a PA flow | `docs/PA_Flow_Checklists.md` + `CLAUDE.md` §7 |
| Understand Nexus internals | `docs/Nexus_Integration_Documentation_v2.md` |
| See yesterday's status | `docs/Mi_Bitacora_Airlink.md` (latest entry) |

---

## Current active sprint

**Hub v3.4.0 — Whole Parts Sprint 3** (frontend). Backend Sprint 2 complete and validated end-to-end as of 2026-05-01. See `CLAUDE.md` §4.1 for the full table.

---

## Critical security debt

⚠️ Anthropic API key currently hardcoded (base64 obfuscation) in Hub client at `index.html` line 668. SQL password hardcoded in `main.js`. **Fix:** route Claude calls through a PA flow proxy. See `CLAUDE.md` §8.

---

## Ownership

**Director:** Jaime Estevez (DR0002) — `jestevez@airlinkdr.com`
**Reports to:** Victor Abuharoon (DR0668), President — Airlink Distribution DR
