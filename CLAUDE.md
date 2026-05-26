# CLAUDE.md — Airlink Platform Project Context

> **Read this first.** This file is the canonical context for Claude Code working on the Airlink platform. It consolidates everything previously held in Claude.ai project memory + reference docs into one document the agent loads automatically.
>
> **Owner:** Jaime Joan Estevez (DR0002) — Director of Software & IT, Airlink Distribution DR
> **Migrated from Claude.ai:** 2026-05-26
> **Last canonical state:** 2026-05-06 (Mi Bitácora) + 2026-05-01 (Whole Parts Sprint 2 complete)

---

## 0 · Table of Contents

1. [Identity & Mission](#1--identity--mission)
2. [Team & Stakeholders](#2--team--stakeholders)
3. [Technology Stack](#3--technology-stack)
4. [Active Initiatives & Status](#4--active-initiatives--status)
5. [Architecture Decisions](#5--architecture-decisions)
6. [Database Architecture](#6--database-architecture)
7. [Power Automate Flow Catalog](#7--power-automate-flow-catalog)
8. [Code Layout & Repos](#8--code-layout--repos)
9. [Working Style & Conventions](#9--working-style--conventions)
10. [Lessons Learned & Gotchas](#10--lessons-learned--gotchas)
11. [Backlog](#11--backlog)
12. [Glossary](#12--glossary)
13. [Critical Dates](#13--critical-dates)
14. [Reference Documents](#14--reference-documents)

---

## 1 · Identity & Mission

### Who

- **Name:** Jaime Joan Estevez M.
- **Employee Code:** DR0002
- **Role:** Director of Software & IT
- **Reports to:** Victor Abuharoon (DR0668, President) — exclusively
- **Company:** Airlink Distribution DR, SRL — Free Trade Zone, Dominican Republic
- **Background:** Mechatronics Engineer (INTEC, DR). ~10 years across machining, mechanical design, manufacturing engineering, electrical/electronics, control systems, and industrial automation, plus current technology leadership.

### Primary mission

**Build and scale an AI-driven operations platform.**

Operating philosophy: *"AI is the opposite of labor arbitrage — use AI to increase labor value, not reduce labor cost."* Target: ~70 % of operational systems AI-driven with significant volume scaling.

### Parallel mission

Position Airlink favorably for the formal acquisition evaluation by **Assurant** (Biju Nair — EVP & President, Global Connected Living; Brandon Johnson — SVP, Global Supply Chain & Engineering). Assurant runs Oracle ERP; Airlink runs Fishbowl + custom stack.

### Honest framing (locked in for all Assurant comms)

*"Today the engine runs on Airlink data alone; the integration is the work we are proposing to do together."*

Never overclaim. Never reference Biju in narrative voice in copy; only as co-inventor in USPTO citations.

---

## 2 · Team & Stakeholders

### Direct reports

| Person | Code | Role | Area |
|---|---|---|---|
| Jorge Ozuna | DR0005 | Semi-Senior Developer | Software |
| Wilson Hilario | DR0143 | Developer | Software |
| Adonis Diaz | DR0155 | IT Supervisor | IT |
| Massiel Gonzalez | DR0287 | Cybersecurity Analyst | IT |

### Cross-functional partners

- **Irvin De La Rosa** (DR0941) — Software / Movement
- **Joel Fermín** — R2 ISO compliance, audit
- **Emilio Alfonso** — Engineering (parts traceability: Receiving/Disassembly → Qualification → Production)
- **Haxel** — ERP / Fishbowl
- **Osterman** — Production
- **Anthony Pasiminio** — Network infrastructure
- **Michael Petiton** — Finance Director

### Leadership / Owners

- **Gabriel and Junior** — Owners
- **Victor Abuharoon** (DR0668) — President

### Assurant (acquisition)

- **Biju Nair** — EVP & President, Global Connected Living
- **Brandon Johnson** — SVP, Global Supply Chain & Engineering

---

## 3 · Technology Stack

### Core platforms

- **Fishbowl ERP** — Advanced REST API; shared service account `airlink-hub` (Jaime to register). MySQL underneath; `LIMIT N` not `TOP N`.
- **Movement / Nexus** — PMS / WMS unified platform replacing fragmented Excel workflows. .NET 8 REST API (Movement-API, 25 controllers) + Next.js 14 frontend (Movement-Fend, 63 pages).
- **Movement 1.5** — Legacy Excel VBA workbook (`Movement_V1_5.xlsm`) — source of truth for ~15 operational processes being migrated to Nexus. See `legacy/Movement_V1_5.xlsm`.
- **AirlinkHub** — Electron desktop app (single HTML, `contextIsolation: false`, mssql via IPC). Hosts the chat agents (Claude, LX, DINO).
- **Phonecheck** — Generates sanitization PDFs automatically (separate from R2/ISO evidence workstream).
- **Microsoft 365 + Power Automate** — Primary integration layer for external APIs, Fishbowl, print orchestration, menu access control.
- **Azure Custom Vision** — Image classification.
- **Node.js dashboards** — Production TV dashboards.
- **Fortinet** — Network security (upgrade scheduled for next month).
- **Zebra / ZPL printers** — Warehouse label printing.
- **FedEx API** — Shipping integration.
- **NoviSign** — Digital signage platform for production TV dashboards.
- **Assurant** — Oracle ERP (target integration counterpart).

### SQL Server endpoints

| Purpose | Host:Port | Databases | User |
|---|---|---|---|
| **Production** | `192.168.181.248:13999` | `AIRLINK`, `AirlinkDR`, `SPN` | `alappuser` (a.k.a. `ALAppUser`) |
| **Dev / QA** | `192.168.181.246:8989` (ARLQA) | mirrors of prod | `alappuser` |
| **AirlinkUSA** | `AFS2SQL1` | `AirlinkUSA` | separate creds |

- **Inventory canonical (M-ring):** `AIRLINK.dbo.SPC_PartInventory`
- **Migration target (cleaner):** SPN → AirlinkDR

### Tunnels & external endpoints

- **ngrok production TV** (fixed): `airlink-production-tv.ngrok.app`
- **ngrok airlink-fb-svc** (fixed): `airlink-fb-svc.ngrok.app`
- **Second fixed ngrok domain** planned for Movement-API production exposure
- **Vercel:** `airlink-shopify-be.vercel.app` (corporate account, **not** linked to MCP workspace — access logs via Vercel dashboard manually)
- **Movement-API dev:** `localhost:7256` (merged version on Airlink local server)

### External APIs

- **OpenWeatherMap API key:** `${OPENWEATHERMAP_API_KEY}` — used in `PA_Weather_Report` flow for Santo Domingo, DR.
- **Anthropic API:** Direct call from Hub client. ⚠️ Currently hardcoded with base64 obfuscation at `index.html` line 668. **Security fix pending:** route through PA proxy.
- **Shopify Custom App token** (planned for `airlink-shopify-sync`): scopes `read_orders`, `read_products`.

### Credentials reference (NEVER paste actual values in any committed file)

All secrets live in `.env` files (gitignored) and are referenced in docs and code by their variable name only — e.g. `${OPENWEATHERMAP_API_KEY}`, not the literal value. See `SECURITY.md` for the full policy and `.env.example` for the canonical list of variables.

| Variable | Where the real value lives |
|---|---|
| `SQL_PASSWORD` (alappuser) | `airlink-desktop/.env`, `airlink-fb-svc/.env` |
| `FISHBOWL_USER`, `FISHBOWL_PASSWORD` | `airlink-fb-svc/.env` |
| `FISHBOWL_APP_ID` (= 1013) | `airlink-fb-svc/.env` |
| `SERVICE_API_KEY` (Bearer) | `airlink-fb-svc/.env` |
| `NGROK_AUTHTOKEN` | ngrok config on `192.168.181.246` |
| `OPENWEATHERMAP_API_KEY` | PA flow connection (also `.env.local` for dev) |
| `ANTHROPIC_API_KEY` | PA proxy env (target — see security debt §8) |
| `SHOPIFY_ADMIN_TOKEN` | `airlink-shopify-sync/.env` (when service is built) |

⚠️ **Historical note:** Two Shopify credentials and one OpenWeatherMap key were committed in earlier iterations of this repo and have been rotated. Tracking IDs in `SECURITY.md`.

---

## 4 · Active Initiatives & Status

### 4.1 Hub v3.4.0 — Whole Parts module ⭐ active

**State:** Sprint 2 complete (2026-05-01). Sprint 2D complete (2026-05-02). **Sprint 3 next.**

**What it is:** Internal employee parts request system replacing the Shopify + Vercel backend. Employees browse and request; Nexus/Movement `ShopifyOrder` module fulfills via scan-to-fulfill.

**Sprint table:**

| Sprint | Status | Notes |
|---|---|---|
| Phase 1A — Schema | ✅ | Brand+Photo+RequestedBy* columns, SEQ_WholeParts_OrderNumber |
| Phase 1B — Taxonomy + 5 SPs | ✅ | 4 views, 5 SPs in `AIRLINK` |
| Phase 1C — Photos | ✅ | 345/394 unique parts uploaded |
| Phase 1C-B — PA Flows 1-4 | ✅ | Brands/Categories/Models/Catalog |
| Phase 1D — Fishbowl Integration | ✅ | `SPC_FishbowlInventory` cache, sync working |
| **Sprint 2A — Submit + FB Sync** | ✅ E2E | WP-2026-05-01-000009 → FB SO 301548 |
| **Sprint 2B — GetMyOrders SP** | ✅ E2E | Photos + FB status |
| **Sprint 2B-bis — PA flow** | ✅ E2E | Wrapper validated |
| **Sprint 2C — RetryPending SP** | ✅ E2E | Filters WP-% only |
| **Sprint 2C-bis — PA flow** | ✅ E2E | WP-2026-04-29-000005 → FB SO 301542 |
| Sprint 2D — Tab activation | ✅ | `Tab_WholeParts` live in `LX_Menus` for 4 testers (DR0002, DR0005, DR0143, DR0941) |
| **Sprint 3 — Frontend Hub v3.4.0** | ⏳ Next | 4 screens + cart + my orders |
| Sprint 4 — Build & deploy zip | ⏳ | After Sprint 3 |
| Phase 4 — Warehouse Node.js print svc | ⏳ | After Sprint 4 |

**Pending TBDs:**
- Fishbowl REST PO endpoint path
- Fishbowl Sales Orders POST body schema

**M-ring inventory canonical:** `AIRLINK.dbo.SPC_PartInventory`. IT-ring skipped initially (shows 0).

**Production-validated orders:**

| OrderNumber | RequestedBy | Path | FishbowlSONum |
|---|---|---|---|
| WP-2026-04-29-000005 | DR0002 | via retry | 301542 |
| WP-2026-04-29-000006 | DR0002 | via submit | 301541 |
| WP-2026-05-01-000009 | DR0002 | via submit | 301548 |

**Inventory sync:** Cron every 10 min, ~8408 parts, ~170 s, single FB session via `beginBatch()`.

### 4.2 DINO Ecosystem — 11 specialized agents

**State:** Tier 1 (Nexus, KPI, Bark) in active testing. Rest planned.

**Architecture decision (locked):** One Power Automate flow per Dino, **not** a single flow with switch. Required for NDA enforcement, executive exclusions, and My DINO privacy.

**Access tier pyramid:**

| Tier | Dinos | Who |
|---|---|---|
| 🟢 All employees | Nexus, KPI, Bark | Everyone |
| 🟡 Supervisor+ | Data | ReportLevel ≥ 2 |
| 🟠 Role-specific | ISO-R2, Bata, HR, Money | Per role flags |
| 🔴 Engineering leadership + NDA | IP | **Executives explicitly excluded** |
| 🔴 Executive only | Leadership | C-suite |
| 🔴 Single-user | My DINO | DR0668 (Victor) only |

**Phased deployment:**
- **v3.4.x:** Tier 1 (Nexus, KPI, Bark)
- **v3.5.0:** Operational (Data, Bata)
- **v3.6.0:** Sensitive (IP, Leadership, My DINO)

**Detail reference:** `docs/Dino_Ecosystem_Reference.md` (825 lines — full system prompts, SPs, KB mappings, sprint plan).

**Shared infrastructure (no change):** `HUB.hist.list`, `HUB.hist.save`, `HUB.hist.load`, `HUB.hist.del`, `HUB.sess`. Claude and Dino do **not** share a base flow; they share these 5.

### 4.3 R2 ISO Audit — Stage 1 May 18-21 · Stage 2 June 22-30

**State:** ~70 % of evidence completed after Trasenda findings (Joel Fermín conducting).

**Stage 1:** Remote, 4 days, policies/SOPs/Records/Training/evidence review.
**Stage 2:** On-site, 6 days, 2 auditors. Covers Receiving, Production, Data Processing, Warehouse, Shipping, E-Waste Storage.

**Critical pending items:**
- Recicla documentation (battery handling, client ISO cert)
- Equipment calibration forms (matrix ready, missing Phoenix Calibration cert)
- 60 consecutive days of CCTV evidence (Jan/Feb/Mar)
- Safety committee formalization
- Evacuation drill
- CCTV monitoring room (structure)

**Jaime's direct involvement:**
- DINO Bark + Nexus testing for audit operational evidence
- DINO ISO architecture design with cross-Dino visibility
- Internal pre-audit before May 14

### 4.4 Assurant Acquisition

**State:** Awaiting Biju Nair response after in-person meeting.

**Materials produced:** HTML presentation v7 (21 slides), PPTX 21 + 10 compact + 5 poster showroom, Word charter, presenter cheat sheet, AI mission/vision Word doc.

**Co-presenter:** Vic (strategic / competitive narrative).

**VDR:** AirlinkHub's OD Agent (on OneDrive/SharePoint, shown to Assurant ~2 months ago).

**Standing rule:** Never reference Biju in narrative voice. Only as co-inventor in USPTO citations.

Biju's recorded statement on AI as a questioning system was woven into presentation materials without direct attribution.

### 4.5 Carrier Lock + MDM Reporting

**State:** Original structure validated. Rollback executed 2026-05-06 after universe-of-data discovery.

**Components:**
- View `VW_CarrierLocked_withStatus` — 8 status rules
- `sp_CL_WeeklyPendingToSubmit_Report` — preview
- `sp_CL_WeeklyPendingToSubmit_Apply` — commit to `SM_Master`
- Filter: `LIKE '%GALAXY%'` (Samsung)

**Dual-universe rule (CRITICAL):**
- `VW_FULLDEVICEREPORT` = **inventory in house only**
- `SM_Master` = **inventory in house + client inventory**

Any report operating across both universes must explicitly contemplate this split. The `Unlock Didn't Work - Resubmit` rule already covers the real-world case (resubmit is client-triggered, not date-driven).

**Lesson:** Validate the data universe before modifying filter logic in systems with parallel sources.

### 4.6 LX Agent — Daily Report system

**State:** Operational. Token + PA timeout limitation pending fix (P1 inherited).

**Components:** Electron frontend + 5 PA flows + Excel generation + PDF generation.

**Active issue:** Token-per-turn + PA timeout causes failures on long responses (affected Adonis's daily report).

**PA flows:**
- `PA-INIT-01` — session start (HTTP POST from Copilot Studio, topic `Session_Start`)
- `PA-MARK-01` — after Q1, marks yesterday's priorities used
- `PA-SAVE-01` — after Q7, saves tomorrow's priorities
- `PA-CLOSE-01` — at session end, calls Claude API, runs Office Script, generates PDF, sends emails

Detail: `docs/PA_Flow_Checklists.md`.

### 4.7 Movement-API production exposure

**State:** Pending production URL exposure.

**Plan:** ngrok second fixed domain OR PA Custom Connector via swagger.json.

**Sequential activation:** Receiving + IQC → QC Grading → Box Building → Shipping.

**Missing endpoint:** Packing List (`ShippingLabel` controller currently empty).

### 4.8 Shopify migration

**Plan:** Standalone Node.js polling service `airlink-shopify-sync` to replace Vercel webhook approach (60-second poll of Shopify Admin API, upsert via `SP_Hub_ShopifyOrder_Upsert`).

**Requires:**
- `SPC_ShopifyOrder` DDL schema
- Shopify Custom App token (`shpat_xxx`, `read_orders` + `read_products` scopes)

**Background:** Quickstart store webhook delays follow exponential backoff (2, 5, 10, 40 min) — dispatch delay, not retry failure.

---

## 5 · Architecture Decisions

### Two-layer pattern

- **Hub** = management / supervision layer.
- **Nexus** = operational execution layer for floor workers.
- Both sit on the same SQL Server backend.

### Direct SQL vs. Power Automate

- **Direct `mssql` IPC** for speed-critical / scan-heavy workflows (production scanning).
- **PA flows** reserved for Fishbowl integration, print orchestration, external APIs, menu access control.

### One PA flow per DINO

Required for NDA enforcement, role-gating, executive exclusions. Cannot be handled with a single flow + switch.

### Sequential delivery within each module

`SQL schema → PA flow spec → frontend implementation`. Analysis and architecture locked **before** documentation is generated.

### Storage of long sessions

Comprehensive reference MDs are built at the end of large sessions to enable context restoration in future runs. This file is one of them.

### Atomic Fishbowl session pattern

`airlink-fb-svc` uses `withSession()` per request (login → op → logout) for SO creation to prevent token cache races. Inventory sync uses **batch** pattern (1 login → 8408 ops → 1 logout).

---

## 6 · Database Architecture

### Production server: `192.168.181.248:13999`

#### `AIRLINK` DB (`AirlinkContext`)
Primary operations — `SPC_*`, `PRD_*`, `G_*` tables. Movement-API main DB.

Key Whole Parts tables:
- `SPC_PartInventory` — inventory canonical (M-ring)
- `SPC_PartNumber` — parts catalog (Description, Color, Capacity, Model, Brand, Photo varbinary base64 JPEG)
- `SPC_ShopifyOrder` — orders + lines, denormalized; **shared with legacy Movement/Nexus**
- `SPC_FishbowlInventory` — populated every 10 min by `airlink-fb-svc` cron
- `SPC_PartsInTransit` — deferred (pending ERP team alignment)
- `SPC_WholeParts_Brand` / `_Category` / `_Collection` / `_Part` — taxonomy

Sequences:
- `SEQ_WholeParts_OrderNumber` — int seq for `WP-YYYY-MM-DD-NNNNNN`

#### `AirlinkDR` DB (`AirlinkDrContext`)
Original Hub DB — Box, Receiving, Unit, etc. Hub Sessions live here:

```
Hub_ChatSessions  (ChatID uniqueidentifier PK, EmployeeCode, Agent, Title, CreatedAt, UpdatedAt, IsDeleted)
Hub_ChatMessages  (MsgID bigint PK, ChatID FK, Seq, Role, Content, CreatedAt)
Hub_Sessions      (SessionID uniqueidentifier PK, EmployeeCode, AgentType, StartedAt, LastActivity)
Hub_Messages      (MessageID int PK, SessionID FK, Role, Content, CreatedAt)
```

SPs in `AirlinkDR`:
- `SP_Hub_NewSession(@EmployeeCode, @AgentType)` → returns SessionID
- `SP_Hub_SaveMessage(@SessionID, @Role, @Content)` → returns MessageID
- `SP_Hub_LoadHistory(@SessionID, @Limit=40)` → returns ASC ordered messages

#### `SPN` DB (`SpnContext`)
HR & permissions — `Empleados`, `Posiciones`, `Departamentos`, `LX_Menus`, auth.

`Empleados` key columns: `Numero` (PK int), `Codigo` (varchar e.g. `DR0002`), `Nombre`, `Apellido1/2/3`, `Cedula`, `E_Mail`, `Posicion`, `ID_Departamento`, `Nivel`, `Supervisor`, `Estatus`, `Foto_imagen`, `foto` (varbinary), `Apodo`, `Celular`.

Joins:
- `Empleados.Posicion = Posiciones.Codigo`
- `Empleados.ID_Departamento = Departamentos.ID`

SPs in `SPN`:
- `SP_Hub_ValidateLogin(@EmployeeCode, @LastFour)` → returns `IsValid`, `EmployeeCode`, `EmployeeName`, `EmployeeEmail`, `PositionName`, `DepartmentName`, `ReportLevel`. Returns empty strings if LastFour mismatches (security).

Cedula format in DB: `XXX-XXXXXXX-X`. LastFour: `RIGHT(REPLACE(Cedula,'-',''), 4)`.
Photo sync: `\\ADRDB1\Fotos Empleados\{Numero}.jpg` → `OneDrive\EmployeePics\{Codigo}.jpg`.

### Separate server: `AFS2SQL1`

`AirlinkUSA` DB (`AirlinkUsaContext`) — US operations Receiving, Unit.

### SQL Server gotchas

- **`varbinary` fields in SPN** require explicit `CONVERT(VARCHAR, ...)` wrapping.
- **`Supervisor` field** stores numeric portion only → self-join via `TRY_CAST`.
- **`Submit_Date` / `Unlock_Date` / `Last_Validation_Date`** are `varchar` in `SM_Master` → always `TRY_CONVERT(date, ...)` for comparisons.
- **`ROW_NUMBER() OVER (PARTITION BY ... ORDER BY ...)`** orders by `Last_Validation_Date` first in `SM_Latest` CTE.
- **`JSON_VALUE` is case-sensitive** — use `COALESCE(JSON_VALUE(value,'$.quantity'), JSON_VALUE(value,'$.qty'))`.
- **`DATEADD(day, -NULL, GETDATE())`** returns NULL → silently breaks downstream comparisons.
- **`SPC_ShopifyOrder` is shared with legacy.** Numeric `OrderNumber` (e.g. `6861201441008`) = legacy Shopify; have `RequestedByCode = NULL` and no `FishbowlSONum`. Always filter Whole Parts queries with `WHERE OrderNumber LIKE 'WP-%' AND RequestedByCode IS NOT NULL`.

### All SPs

`GRANT EXECUTE ON [SP_Name] TO alappuser` for every SP used by Hub.

### Detail reference

`docs/DB_Schema_Reference.txt` — canonical column references.
`docs/Hub_SQL_Schema.md` — formal schema documentation with CREATE scripts.

---

## 7 · Power Automate Flow Catalog

All flows live in environment `default-776b130e-a1ea-446a-a1f6-793c70bd6f82`.

### Whole Parts flows

| Flow | Workflow ID | Purpose |
|---|---|---|
| `PA_Hub_Parts_GetBrandsWithCategories` | `5eca8ef551454c9daaf5623a20296fad` | Catalog tree level 1+2 |
| `PA_Hub_Parts_GetModels` | `9cf61cf72f504d8386d97f723a420220` | Models filter |
| `PA_Hub_Parts_GetCatalog` | `169be9401d2a40caa2be1a1b83836032` | Parts grid (photos + FB inventory) |
| `PA_Hub_Parts_SubmitOrder` | `576387728650402ca9304698fb7525a1` | Submit + FB sync |
| `PA_Hub_FB_CreateSalesOrder` | `e98ffbfd8f884adf84d5f169ca10c272` | Internal FB SO creator |
| `PA_Hub_Parts_GetMyOrders` | `4cca1056db3c4c04830bde2768f972f3` | My orders w/ photos |
| `PA_Hub_Parts_RetryPending` | `294bd4661c344b50b7070055eb762e31` | Retry pending orders |

### Canonical contract — `PA_Hub_FB_CreateSalesOrder`

```json
{
  "hubOrderNumber": "string",
  "lines": [
    { "partNumber": "string", "quantity": 0, "description": "string" }
  ]
}
```

⚠️ **Field is `hubOrderNumber` NOT `orderNumber`.** Any caller must respect this.

### Sample payload — Submit

```json
POST <PA_Hub_Parts_SubmitOrder URL>
{
  "employeeCode": "DR0002",
  "employeeName": "Jaime Estevez",
  "employeeEmail": "jestevez@airlinkdr.com",
  "notes": "...",
  "lines": [
    { "partNumber": "L5015087", "quantity": 2, "location": "GENERAL WAREHOUSE 10|6|B" }
  ]
}
```

Response:

```json
{
  "ok": true,
  "orderNumber": "WP-2026-05-01-000009",
  "linesInserted": 1,
  "status": "ok",
  "fishbowlSONum": "301548",
  "fishbowlSyncStatus": "synced"
}
```

### Daily Report flows (LX Agent)

| Flow | Trigger | Purpose |
|---|---|---|
| `PA-INIT-01` | Copilot Studio topic `Session_Start` | Query SQL, priorities, random questions; inject into agent |
| `PA-MARK-01` | After Q1 | Mark yesterday's priorities as used in DB |
| `PA-SAVE-01` | After Q7 | Save tomorrow's priorities in DB |
| `PA-CLOSE-01` | Session end | Call Claude API, run Office Script, generate PDF, send emails |

Detail: `docs/PA_Flow_Checklists.md`.

### Other planned flows

- `PA_Hub_IQC_GetUnit` — wire to `SP_Hub_IQC_GetUnit`
- `PA_Hub_IQC_Save` — wire to `SP_Hub_IQC_Save`
- `PA_UNIT_LOOKUP_URL`, `PA_FG_SUMMARY_URL`, `PA_RECEIVED_URL`
- `PA_Hub_Parts_FishbowlInventory` (spec delivered Phase 3)
- `PA_Hub_Parts_Submit` (spec delivered Phase 3)
- `PA_Weather_Report` — Santo Domingo, DR (OpenWeatherMap)
- 11 × `PA_Hub_Dino_*` flows (one per DINO)

---

## 8 · Code Layout & Repos

### `airlink-desktop` (Hub Electron source)

```
airlink-desktop/
├── main.js              ← DB config + IPC handlers (mssql, sql-exec, sql-ping, app-restart)
├── preload.js           ← window.electronAPI = {} direct assignment (NO contextBridge — silently fails)
├── index.html           ← Single-file UI; HUB endpoint object, TAB_MAP, _SECTION_TREES,
│                          switchSection, handleNavAction, initWPPanel
├── package.json
└── ...
```

**Build state:** `airlink-desktop_3.4.0.zip` ready with full UI scaffolding (tab button, TAB_MAP, _SECTION_TREES, switchSection, handleNavAction stubs, initWPPanel with 4 stubs).

**Run `npm install` (mssql dependency) in `airlink-desktop/` folder.**

⚠️ **Critical security findings:**
- Anthropic API key hardcoded in client via base64 obfuscation — `index.html` line 668
- SQL password hardcoded in `main.js`
- `HUB.claude` is **dead code** — calls go directly to Anthropic API
- **Fix:** Route Claude calls through a PA flow proxy

### `airlink-fb-svc` (Node.js Fishbowl bridge)

```
D:\airlink-fb-svc\          ← on 192.168.181.246, PM2-managed
├── src/
│   ├── server.js              ← shutdown handler closes FB batch
│   ├── fishbowl.js            ← withSession() + beginBatch/endBatch
│   ├── salesOrderService.js   ← FB workaround + 300ms commit delay
│   ├── csvBuilder.js
│   ├── inventorySync.js       ← uses beginBatch in finally
│   ├── db.js
│   └── logger.js
├── sql/
│   └── Phase_1D_Fishbowl_Setup.sql
├── ecosystem.config.js
├── package.json (v1.0.4)
├── .env                       ← SERVICE_API_KEY, FB creds, SQL creds
└── logs/
```

**Endpoints:** `/api/health`, `POST /api/sales-order` (requires `hubOrderNumber`), `GET /api/inventory/:partNumber`, `POST /api/sync-inventory`, `GET /api/sync-status`.

**Public URL:** `https://airlink-fb-svc.ngrok.app` (fixed domain). **Auth:** Bearer token in `SERVICE_API_KEY` env var.

**FB required fields for SO:** `customerId=4` ("Shopify Customer"), `locationGroupId=43` ("001 Wind Tunnel"), `salesman="shopify"`, `status=10` (Estimate), `currency="Dominican Peso"`, `note=hubOrderNumber`.

**PM2 cycle when modules need reload:**
```bash
pm2 stop airlink-fb-svc
pm2 delete airlink-fb-svc
pm2 start ecosystem.config.js
```

### `nexus` (Movement-API + Movement-Fend)

- **Movement-API:** .NET 8 REST, 25 controllers. Dev on `localhost:7256`.
- **Movement-Fend:** Next.js 14, 63 pages/routes.
- **Status:** Active development. Production URL exposure pending.

### `airlink-shopify-sync` (planned)

Standalone Node.js polling service. 60-second poll of Shopify Admin API → upsert via `SP_Hub_ShopifyOrder_Upsert`.

---

## 9 · Working Style & Conventions

### Communication

- **Bilingual:** Spanish for conversation; English for technical artifacts.
- **Direct, iterative.** Prefer evaluative honesty over diplomatic hedging.
- **Rollback is a real instruction** — when Jaime says "rollback", honor it immediately. Rollback is not failure; it's iteration.
- **One thing at a time.** Don't bundle fixes.

### Code delivery

- **Complete, copy-paste-ready code** preferred over snippets.
- **Screenshots** for visual feedback are normal.
- **Validation:** SQL/IPC validated against actual stored procedures and schemas before wiring UI.

### Sequencing

- Analysis & architecture locked → then documentation → then implementation.
- **SQL schema → PA flow spec → frontend implementation.**
- Comprehensive reference MDs at end of large sessions for context restoration.

### Electron specifics

- **Always work from actual Electron source files**, not standalone HTML.
- Extraction path when no source zip is provided: `.exe` → NSIS → 7z → `@electron/asar` unpack.
- `contextIsolation: false` → preload uses `window.electronAPI = {}` direct assignment; `contextBridge.exposeInMainWorld()` silently fails in this config.
- Panels attach to `.hc` **NOT** `#hub`.
- Agent bar (`.ab`) hidden outside Chat section.
- Logout: `app.relaunch() + app.exit(0)` for complete state reset.
- **Pre-package validation:** `node --check /tmp/cs.js` before every zip — catches silent syntax errors (unbalanced backticks, missing braces).

### Reporting

- Structured daily report prompts via LX Agent system.
- Bitácora (`docs/Mi_Bitacora_Airlink.md`) is the living continuity document.

### Stakeholder / presentation work

- Context-giving over question-heavy approaches.
- Anchor stories over interview style.

---

## 10 · Lessons Learned & Gotchas

### Power Automate

1. **Schema inference is treacherous.** Single-element array → PA infers `object`, silently unpacks at runtime, breaks Parse_JSON downstream. Manually inspect generated schemas.
2. **PA does NOT respect SP DEFAULT values.** Missing/null parameter → explicit NULL to SP. Defensive pattern in SP:
   ```sql
   IF @MaxAge IS NULL OR @MaxAge <= 0 SET @MaxAge = 7;
   IF @OrderNumber = '' OR @OrderNumber = 'null'
       OR LTRIM(RTRIM(ISNULL(@OrderNumber,''))) = ''
       SET @OrderNumber = NULL;
   ```
3. **Array → string is NOT automatic.** Use `Compose_LinesJson = string(triggerBody()?['lines'])` then pass `outputs('Compose_LinesJson')` to SP.
4. **`Parse_JSON.Content` must be a string.** Pointing to an array variable errors with "content expects a value but got null".
5. **`Append to array variable` is type-strict.** Use the `</>` code-view toggle for native object literals.
6. **PA HTTP timeout is short (~2 min default).** For child flow calls: Settings → Timeout → `PT5M`.
7. **`Run after` matters for error capture.** Set Parse_JSON to: is successful + has failed + is skipped + has timed out. Otherwise short-circuits on failure.
8. **SP parameter via Expression tab:** `triggerBody()?['employeeCode']` as bare string, not object mapping. Diagnostic: INPUTS log showing `{"employeeCode":"DR0002"}` (wrapped) vs `"DR0002"` (bare).
9. **PA response normalization:** `ResultSets.Table1[0] ?? Table1[0] ?? raw`.
10. **`Find files in folder`** returns direct array (not `{value: []}` wrapper): `body(...)?[0]?['Id']`; empty check via `empty(body(...))`.
11. **Always `Compose_Meta` before `Condition` blocks** when meta values are needed downstream.
12. **Claude JSON parse:** `json(body('HTTP_Claude')?['content']?[0]?['text'])`.

### Fishbowl

1. **Empty body on success.** `/api/import/SalesOrder` returns HTTP 200 with `""` body even though SOs are created. Workaround: query `so` table by unique `note` (= hubOrderNumber) after a 300 ms commit delay.
2. **MySQL underneath.** Use `LIMIT N` not `TOP N`. `customerContact` not `customerName`.

### Frontend / Build

- **PPTX must use `LAYOUT_WIDE` (13.333 × 7.5 in)**, NOT `LAYOUT_16x9` (causes significant overflow).
- **DOCX border order follows OOXML schema:** `top, left, bottom, right`.
- **LibreOffice renders Georgia wider than PowerPoint** — apparent QA-render overflow resolves in real PowerPoint.
- **CSS variable inheritance fails inside iframes** — use explicit per-element `!important` rules, not cascade.
- **Depth-counting pattern** for JS function boundary detection in large HTML files is reliable; simple string replace requires exact whitespace matching.
- **Extract script block from HTML:** `html[html.find('<script>', 35000)+8 : html.rfind('</script>')]`.

### Shopify

- Webhook delays on quickstart stores match exponential backoff (2, 5, 10, 40 min). Root cause is dispatch delay, not retry failure.

### Phonecheck vs R2/ISO (don't confuse)

- Phonecheck generates sanitization PDFs **automatically**.
- R2/ISO evidence workstream is **separate** (Joel Fermín).
- Parts traceability (Receiving/Disassembly → Qualification, Emilio) is also separate.

---

## 11 · Backlog

### Hub

- [ ] `npm install` in `airlink-desktop/` (mssql)
- [ ] `SP_Hub_IQC_GetUnit` SP → wire `iqcLoadUnit()` to direct `sql()`
- [ ] `SP_Hub_IQC_Save` SP → wire `iqcSubmit()` to direct `sql()`
- [ ] PA flows: `PA_UNIT_LOOKUP_URL`, `PA_FG_SUMMARY_URL`, `PA_RECEIVED_URL`
- [ ] DSM1 / BH / AQC2 modules follow same IQC pattern
- [ ] Execute `LX_Menus_Setup.sql` in SPN to activate tabs per employee
- [ ] Migrate Anthropic API key from Electron client → server-side PA proxy

### Whole Parts

- [ ] **Sprint 3:** catalog + cart + submit frontend
- [ ] **Phase 4:** Warehouse Node.js print service (Production TV pattern)
- [ ] **Phase 5:** Complete frontend
- [ ] Verify Fishbowl REST API TBDs (PO endpoint path/filter, SO POST body schema)

### LX Agent

- [ ] **P1:** Apply concrete fix to token + PA timeout limitation. Decide between: truncate input frontend-side, adjust API token limits, optimize validator system prompt, or restructure PA flow. Validate against Adonis case before promoting.
- [ ] **P2:** Document case as Gotchas + close problem cycle

### DINO Ecosystem

- [ ] Continue Bark + Nexus testing
- [ ] Design DINO ISO architecture with cross-Dino visibility (sketch before May 14)
- [ ] Engineering leadership list (exact employees with DINO IP access)
- [ ] Executive list (confirm `Tab_Executive` users)
- [ ] Sales leadership list (for `Dino_Money_Sales`)
- [ ] Bata team list (for `Dino_Bata`)
- [ ] Quality / Compliance team list (for `Dino_ISO_R2`)
- [ ] HR team list (for `Dino_HR`)
- [ ] DinoBot_Data: connect to SharePoint OR call SQL via PA inside bot? **Recommendation:** SQL via PA, as tool call

### ERP / Movement

- [ ] ERP sub-section in Hub Supply Chain tab (incoming shipments + Fishbowl PO status, populates `SPC_PartsInTransit`) — deferred pending ERP team alignment
- [ ] Movement-API production URL (ngrok 2nd domain OR PA Custom Connector via swagger.json)
- [ ] Sequential activation: Receiving + IQC → QC Grading → Box Building → Shipping
- [ ] Packing List endpoint (`ShippingLabel` controller currently empty)

### Infrastructure

- [ ] Fortinet upgrade (next month)
- [ ] Migration SPN → AirlinkDR (cleaner target)

---

## 12 · Glossary

| Term | Meaning |
|---|---|
| **Hub** | Airlink Hub Electron desktop application |
| **Nexus** | Unified Operations Platform (Movement-API + Movement-Fend) |
| **Movement 1.5** | Legacy Excel VBA workbook being replaced by Nexus |
| **Whole Parts** | Internal parts request module (replaces Shopify + Vercel) |
| **WP-** | Order number prefix for Whole Parts orders |
| **SO** | Sales Order (Fishbowl) |
| **FB** | Fishbowl |
| **SP** | Stored Procedure |
| **PA** | Power Automate |
| **LX Agent** | Daily Report Copilot Studio agent embedded in Hub |
| **DINO** | Specialized AI agent (11 in ecosystem) |
| **alappuser** | SQL Server service account for Hub |
| **IQC** | Incoming Quality Control (Hub module) |
| **DSM / BH / AQC2** | Future Hub modules following IQC pattern |
| **M-ring** | `AIRLINK.dbo.SPC_PartInventory` (canonical inventory) |
| **IT-ring** | Inventory layer skipped initially (Whole Parts) |
| **DR0002** | Jaime's employee code |
| **DR0668** | Victor's employee code |
| **VDR** | Virtual Data Room (= AirlinkHub's OD Agent for Assurant) |
| **R2** | Responsible Recycling certification |

---

## 13 · Critical Dates

| Date | Event |
|---|---|
| **2026-05-14 (Thu)** | Internal R2 ISO pre-audit |
| **2026-05-18 to 21** | R2 ISO Stage 1 (remote) |
| **2026-06-22 to 30** | R2 ISO Stage 2 (on-site) |
| **Next month** | Fortinet upgrade |

---

## 14 · Reference Documents

All living in `docs/` in this repo:

| File | Purpose |
|---|---|
| `Mi_Bitacora_Airlink.md` | Daily log / continuity (419 lines, last entry 2026-05-06) |
| `Hub_WholeParts_State_2026-05-01.md` | Whole Parts master state (599 lines) |
| `Dino_Ecosystem_Reference.md` | 11 DINOs full reference (825 lines) — system prompts, SPs, KB mappings |
| `DB_Schema_Reference.txt` | Canonical column references |
| `Hub_SQL_Schema.md` | Formal schema with CREATE scripts |
| `Nexus_Integration_Documentation_v2.md` | Sister system architecture |
| `PA_Flow_Checklists.md` | PA flow build patterns (Daily Report flows in detail) |
| `LX_Agent_vFinal.html` | LX Agent template |

Legacy:

| File | Purpose |
|---|---|
| `legacy/Movement_V1_5.xlsm` | Excel VBA system being migrated to Nexus (20 sheets, IQC migration source) |

### Documents that existed but were retired

- `Hub_v337_Source_Reference.md` (908 lines) — file tree, IPC handlers, HUB endpoints, TAB_MAP, security findings. Information consolidated into [Section 8](#8--code-layout--repos) and [Section 9](#9--working-style--conventions) of this file.
- `Hub_Project_Status_v33.docx` — superseded by Hub_WholeParts_State_2026-05-01.md
- `Airlink_AI_Project_State.md` — superseded by Hub_WholeParts_State_2026-05-01.md

---

## 15 · How to use this document with Claude Code

1. **First action when opening this repo:** read this file end-to-end. It contains everything Claude.ai previously held in project memory.
2. **For any task involving the Hub frontend:** also read `docs/Hub_WholeParts_State_2026-05-01.md` for current sprint state.
3. **For DINO work:** also read `docs/Dino_Ecosystem_Reference.md` — full system prompts and access logic.
4. **For database work:** start with [Section 6](#6--database-architecture), then `docs/DB_Schema_Reference.txt` and `docs/Hub_SQL_Schema.md`.
5. **For PA flow work:** start with [Section 7](#7--power-automate-flow-catalog), then `docs/PA_Flow_Checklists.md`.
6. **For Movement-API / Nexus work:** read `docs/Nexus_Integration_Documentation_v2.md`.
7. **When session ends:** if substantial work was done, generate or update a state doc in `docs/` so the next session starts hot.

### Update protocol for this file

- **Add to Section 4** when a new initiative starts or a sprint completes.
- **Add to Section 10** when a new gotcha is discovered.
- **Add to Section 11** when items are completed (move to a "Done" log if useful).
- **Touch Section 13** when audit dates shift.
- **Touch Section 14** when reference docs are added or retired.

---

*Document maintained by Jaime Estevez, Director of Software & IT.*
*Migrated from Claude.ai project memory: 2026-05-26.*
