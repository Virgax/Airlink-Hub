# Airlink Hub — Whole Parts v3.4.0 State Documentation
## Sprint 2 COMPLETE — End of Day 2026-05-01

> **Purpose:** Master state document. Replaces older fragmented project files.
> Load this into the project as primary memory after deleting redundant files.
> **Maintained by:** Jaime Estevez (DR0002), Director of Software & IT

---

# 📋 EXECUTIVE SUMMARY

The Whole Parts module (internal parts request system replacing Shopify+Vercel)
has reached **end-to-end production-ready state** for the backend layer.
Sprint 2 (5 sub-sprints) is **fully validated** as of 2026-05-01.

**What works end-to-end right now:**
- Employee submits parts request → SQL stores order → Fishbowl SO auto-created
- Failed/legacy orders can be retried in bulk or individually
- "My orders" view returns full history with photos and Fishbowl sync status
- Catalog browsable by brand → category → model with live Fishbowl inventory

**What's pending:** Frontend (Sprint 3) and tab activation (Sprint 2D).

---

# 🏗️ ARCHITECTURE OVERVIEW

```
┌──────────────────────────────────────────────────────────────┐
│                      AIRLINK HUB (Electron)                  │
│  Single HTML file, contextIsolation=false, mssql via IPC     │
│                                                              │
│  Whole Parts UI (Sprint 3 — pending)                         │
│  ├─ Brand selector                                           │
│  ├─ Category grid                                            │
│  ├─ Model selector                                           │
│  ├─ Parts grid (with 5 inventory counters)                   │
│  ├─ Cart + Submit                                            │
│  └─ My Orders + Retry button                                 │
└────────────────────┬─────────────────────────────────────────┘
                     │
                     ↓ HTTPS calls
┌──────────────────────────────────────────────────────────────┐
│                   POWER AUTOMATE FLOWS                       │
│                                                              │
│  PA_Hub_Parts_GetBrandsWithCategories  → SP_..._GetBrands    │
│  PA_Hub_Parts_GetModels                → SP_..._GetModels    │
│  PA_Hub_Parts_GetCatalog               → SP_..._GetCatalog   │
│  PA_Hub_Parts_SubmitOrder        ┐     → SP_..._SubmitOrder  │
│  PA_Hub_Parts_RetryPending       ├──→ PA_Hub_FB_CreateSO ──┐ │
│  PA_Hub_Parts_GetMyOrders        ┘     → SP_..._GetMyOrders│ │
└────────────────────┬─────────────────────────────────────────┘
                     │                                       │
       ┌─────────────┴─────────────┐                         │
       ↓                           ↓                         │
┌──────────────┐         ┌──────────────────┐                │
│  SQL Server  │         │ airlink-fb-svc   │←───────────────┘
│ 192.168.181  │         │ 192.168.181.246  │
│  .248:13999  │         │  Node + Express  │
│              │         │  PM2 managed     │
│  AIRLINK DB  │         │  ngrok tunnel    │
│  AirlinkDR   │         │                  │
│  SPN         │         │  Atomic session  │
│              │         │  withSession()   │
│  alappuser   │         └────────┬─────────┘
└──────────────┘                  │
                                  ↓
                       ┌──────────────────────┐
                       │  Fishbowl Advanced   │
                       │  REST API            │
                       │  192.168.181.247     │
                       │  :2456               │
                       │                      │
                       │  shopify@FB          │
                       │  customerId=4        │
                       │  locationGroupId=43  │
                       └──────────────────────┘
```

---

# 🎯 SPRINT STATUS

| Sprint | Status | Validated | Notes |
|---|---|---|---|
| Phase 1A — Schema | ✅ DONE | Yes | Brand+Photo+RequestedBy* columns, SEQ_WholeParts_OrderNumber |
| Phase 1B — Taxonomy + 5 SPs | ✅ DONE | Yes | 4 views, 5 SPs in AIRLINK |
| Phase 1C — Photos | ✅ DONE | Yes | 345/394 unique parts uploaded |
| Phase 1C-B — PA Flows 1-4 | ✅ DONE | Yes | Brands/Categories/Models/Catalog |
| Phase 1D — Fishbowl Integration | ✅ DONE | Yes | SPC_FishbowlInventory cache, sync working |
| **Sprint 2A — Submit + FB Sync** | ✅ **DONE** | ✅ E2E | WP-2026-05-01-000009 → FB SO 301548 |
| **Sprint 2B — GetMyOrders SP** | ✅ **DONE** | ✅ E2E | Includes photos, FB status |
| **Sprint 2B-bis — GetMyOrders flow** | ✅ **DONE** | ✅ E2E | PA wrapper validated |
| **Sprint 2C — RetryPending SP** | ✅ **DONE** | ✅ E2E | Filters WP-% only |
| **Sprint 2C-bis — RetryPending flow** | ✅ **DONE** | ✅ E2E | WP-2026-04-29-000005 → FB SO 301542 |
| Sprint 2D — Tab activation | ⏳ Pending | — | Run LX_Menus_Setup.sql |
| Sprint 3 — Frontend Hub v3.4.0 | ⏳ Pending | — | 4 screens + cart + my orders |
| Sprint 4 — Build + deploy zip | ⏳ Pending | — | After Sprint 3 |

---

# 🗄️ DATABASE — SQL SERVER

**Connection:** `192.168.181.248:13999`
**User:** `alappuser`
**Databases used by Whole Parts:** `AIRLINK` (canonical)

## Tables

### `AIRLINK.dbo.SPC_ShopifyOrder` (orders + lines, denormalized)

Key columns relevant to Whole Parts:

| Column | Type | Notes |
|---|---|---|
| OrderNumber | varchar(50) | `WP-YYYY-MM-DD-NNNNNN` for Whole Parts; numeric for legacy Shopify |
| PartNumber | varchar(50) | Reference to SPC_PartNumber |
| QTY | int | Quantity requested |
| Location | varchar(100) | Warehouse location of the part |
| IsOpen | bit | 1 = open, 0 = closed |
| CreatedAtDate | date | |
| CreatedAtTime | time | |
| RequestedByCode | varchar(20) | Employee code (DR0002 etc) — NULL for legacy Shopify |
| RequestedByName | varchar(100) | Employee full name |
| RequestedByEmail | varchar(100) | Employee email |
| Notes | varchar(500) | Optional notes from submitter |
| **FishbowlSONum** | varchar(20) | Fishbowl SO Number once synced |
| **FishbowlSOId** | int | Fishbowl SO internal ID |

### `AIRLINK.dbo.SPC_PartNumber` (parts catalog)

Standard catalog with `Description`, `Color`, `Capacity`, `Model`, `Brand`,
`Photo` (varbinary, base64-encoded JPEG).

### `AIRLINK.dbo.SPC_FishbowlInventory` (inventory cache)

Populated by `airlink-fb-svc` cron every 10 min. Used by `GetCatalog` to
display fishbowlQty + inProcessQty per part.

### `AIRLINK.dbo.SPC_PartsInTransit` (deferred — ERP sub-section)

Schema exists, not yet wired up. Pending alignment with ERP team.

## Sequences

- `AIRLINK.dbo.SEQ_WholeParts_OrderNumber` — int sequence used to generate
  the unique part of `WP-YYYY-MM-DD-NNNNNN`.

## Stored Procedures (all in `AIRLINK.dbo`, GRANT EXECUTE TO alappuser)

| SP | Purpose | Status |
|---|---|---|
| `SP_Hub_Parts_GetBrandsWithCategories` | Catalog tree level 1+2 | ✅ |
| `SP_Hub_Parts_GetModels` | Models by brand+category | ✅ |
| `SP_Hub_Parts_GetCatalog` | Parts list w/ photos + FB inventory | ✅ |
| `SP_Hub_Parts_SubmitOrder` | Insert order + lines, returns OrderNumber | ✅ |
| `SP_Hub_Parts_GetMyOrders` | List employee's orders w/ photos + FB status | ✅ |
| `SP_Hub_Parts_RetryPending` | List pending WP- orders for retry | ✅ |
| `SP_Hub_SetFishbowlSO` | Update FishbowlSONum/SOId after sync | ✅ |

---

# 🔌 NODE SERVICE — `airlink-fb-svc`

**Location:** `D:\airlink-fb-svc\` on `192.168.181.246`
**Version:** v1.0.4
**Process manager:** PM2
**Public URL:** `https://airlink-fb-svc.ngrok.app` (fixed domain)
**Auth:** Bearer token in `SERVICE_API_KEY` env var

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/health` | Service status |
| POST | `/api/sales-order` | Create FB SO from Hub order. **Requires `hubOrderNumber`** |
| GET | `/api/inventory/:partNumber` | Single part inventory query |
| POST | `/api/sync-inventory` | Manual trigger of full sync (cron does it every 10 min) |
| GET | `/api/sync-status` | Last sync metadata |

## Architecture decisions

- **Atomic `withSession()` pattern** for SO creation: login → op → logout
  per request. Prevents token cache races.
- **Batch session pattern** for inventory sync: 1 login → 8408 ops → 1 logout.
- **Empty-response workaround for FB:** FB's `/api/import/SalesOrder` returns
  HTTP 200 with empty body `""` on success. Workaround: query `so` table
  by unique `note` (`hubOrderNumber`) to retrieve created SO. 300ms commit
  delay before query.

## Source files (latest in repo)

```
D:\airlink-fb-svc\
├── src/
│   ├── server.js              ← shutdown handler closes FB batch
│   ├── fishbowl.js            ← withSession() + beginBatch/endBatch
│   ├── salesOrderService.js   ← FB workaround + 300ms delay
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

## PM2 management

```bash
# Status
pm2 status

# Logs
pm2 logs airlink-fb-svc --lines 50

# Restart (NOTE: restart doesn't always reload modules)
pm2 stop airlink-fb-svc
pm2 delete airlink-fb-svc
pm2 start ecosystem.config.js
```

---

# ⚡ POWER AUTOMATE FLOWS

All flows live in `default-776b130e-a1ea-446a-a1f6-793c70bd6f82` environment.

## Active flows (Whole Parts)

| Flow | Workflow ID | Purpose |
|---|---|---|
| PA_Hub_Parts_GetBrandsWithCategories | `5eca8ef551454c9daaf5623a20296fad` | Catalog tree |
| PA_Hub_Parts_GetModels | `9cf61cf72f504d8386d97f723a420220` | Models filter |
| PA_Hub_Parts_GetCatalog | `169be9401d2a40caa2be1a1b83836032` | Parts grid |
| **PA_Hub_Parts_SubmitOrder** | `576387728650402ca9304698fb7525a1` | Submit + FB sync |
| **PA_Hub_FB_CreateSalesOrder** | `e98ffbfd8f884adf84d5f169ca10c272` | Internal FB SO creator |
| **PA_Hub_Parts_GetMyOrders** | `4cca1056db3c4c04830bde2768f972f3` | My orders w/ photos |
| **PA_Hub_Parts_RetryPending** | `294bd4661c344b50b7070055eb762e31` | Retry pending orders |

## Key contract — `PA_Hub_FB_CreateSalesOrder`

**Trigger schema (required):**
```json
{
  "hubOrderNumber": "string",
  "lines": [
    {"partNumber": "string", "quantity": int, "description": "string"}
  ]
}
```

⚠️ **Field is `hubOrderNumber` NOT `orderNumber`.** This is the canonical
contract; any caller (Submit, RetryPending, future flows) must respect it.

## Sample payloads

### SubmitOrder (E2E validated)

```json
POST <PA_Hub_Parts_SubmitOrder URL>
{
  "employeeCode": "DR0002",
  "employeeName": "Jaime Estevez",
  "employeeEmail": "jestevez@airlinkdr.com",
  "notes": "...",
  "lines": [
    {
      "partNumber": "L5015087",
      "quantity": 2,
      "location": "GENERAL WAREHOUSE 10|6|B"
    }
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
  "message": "",
  "fishbowlSONum": "301548",
  "fishbowlSyncStatus": "synced",
  "fishbowlError": ""
}
```

### RetryPending (bulk mode)

```json
POST <PA_Hub_Parts_RetryPending URL>
{}
```

### RetryPending (single order)

```json
POST <PA_Hub_Parts_RetryPending URL>
{
  "orderNumber": "WP-2026-04-29-000005"
}
```

### GetMyOrders

```json
POST <PA_Hub_Parts_GetMyOrders URL>
{
  "employeeCode": "DR0002",
  "onlyOpen": false,
  "limitDays": 30
}
```

---

# 🐛 KEY GOTCHAS & LESSONS LEARNED

## Power Automate

### 1. Schema inference is treacherous

When `Use sample payload to generate schema` runs on a single-element array,
PA infers `object` not `array`. **Always manually inspect the generated schema.**

Example bug: `lines` field with one element → PA generates `"type": "object"`,
which silently unpacks arrays at runtime, breaking Parse_JSON downstream.

### 2. PA does NOT respect SP DEFAULT values

When a parameter is missing or `null` in the trigger body, PA passes
**explicit NULL** to the SP — it does NOT activate the SP's `DEFAULT` value.

**Defense pattern in SP:**
```sql
IF @MaxAge IS NULL OR @MaxAge <= 0 SET @MaxAge = 7;
IF @OrderNumber = '' OR @OrderNumber = 'null'
    OR LTRIM(RTRIM(ISNULL(@OrderNumber,''))) = ''
    SET @OrderNumber = NULL;
```

### 3. Array → string conversion is NOT automatic

When trigger receives `lines: [...]` as native array, PA cannot pass it
directly as `nvarchar(max)` to a SP. Need a `Compose` step:

```
Compose_LinesJson = string(triggerBody()?['lines'])
```

Then pass `outputs('Compose_LinesJson')` to the SP parameter.

### 4. `Parse_JSON` Content must be a string

If you point `Parse_JSON.Content` to an array variable directly, it errors
with "content expects a value but got null". Always feed it a string —
either from a `Compose` that runs `string(...)` or from a SP output that
returns JSON-as-string.

### 5. `Append to array variable` is type-strict

```javascript
// ❌ FAILS — interpreted as Object, not array element
json(concat('{"foo":"bar"}'))

// ✅ WORKS — native object literal in code view
{
  "foo": "@{variables('something')}",
  "bar": "@{items('Apply_to_each')?['baz']}"
}
```

Use the `</>` (code view) toggle to enter object literals natively.

### 6. PA HTTP timeout is short

Default ~2 minutes. For child flow calls, add explicit timeout:
**Settings → Timeout → `PT5M`**

### 7. `Run after` settings matter for error capture

To capture errors from upstream HTTP calls, set Parse_JSON to:
- ✅ is successful
- ✅ has failed
- ✅ is skipped
- ✅ has timed out

Otherwise the flow short-circuits on failure and never appends to results.

## Fishbowl

### 1. Empty body on success

`/api/import/SalesOrder` returns `HTTP 200` with empty string body `""` on
successful imports. SOs ARE created. Workaround: query `so` table by unique
`note` field (= hubOrderNumber) after a 300ms commit delay.

### 2. MySQL syntax

FB uses MySQL internally. `LIMIT N` not `TOP N`. `customerContact` not
`customerName`.

### 3. Required SO fields

```javascript
customerId        = 4   // "Shopify Customer"
locationGroupId   = 43  // "001 Wind Tunnel"
salesman          = "shopify"
status            = 10  // Estimate
currency          = "Dominican Peso"
note              = hubOrderNumber  // ← used for workaround query
```

## SQL Server

### 1. JSON_VALUE is case-sensitive

`JSON_VALUE(value, '$.qty')` will NOT match a JSON field named `quantity`.
Defensive pattern:

```sql
COALESCE(
    JSON_VALUE(value, '$.quantity'),
    JSON_VALUE(value, '$.qty')
)
```

### 2. Date arithmetic with NULL

`DATEADD(day, -NULL, GETDATE())` returns NULL, which silently breaks all
downstream comparisons (`WHERE date >= NULL` is always false).

### 3. `SPC_ShopifyOrder` table is shared with legacy Movement/Nexus

Numeric `OrderNumber` (e.g. `6861201441008`) = legacy Shopify orders from
the Movement era. They have `RequestedByCode = NULL` and lack `FishbowlSONum`.

**Always filter Whole Parts queries with:**
```sql
WHERE OrderNumber LIKE 'WP-%'
  AND RequestedByCode IS NOT NULL
```

---

# 📊 PRODUCTION DATA SAMPLES

## Validated orders end-of-day 2026-05-01

| OrderNumber | RequestedBy | Status | FishbowlSONum | FishbowlSOId |
|---|---|---|---|---|
| WP-2026-04-29-000005 | DR0002 | synced (via retry) | 301542 | 2319 |
| WP-2026-04-29-000006 | DR0002 | synced (via submit) | 301541 | 2318 |
| WP-2026-05-01-000009 | DR0002 | synced (via submit) | 301548 | — |

## Inventory sync (cron every 10 min)

- ~8408 parts populated from Fishbowl per cycle
- Avg sync duration: ~170s
- Uses `beginBatch()` → 1 FB session for all parts

---

# 🔮 NEXT STEPS

## Immediate (next session)

### Sprint 2D — Activate Tab_WholeParts in LX_Menus

Run `LX_Menus_Setup.sql` in SPN database:

```sql
USE SPN;
INSERT INTO LX_Menus (EmployeeCode, MenuKey, IsActive)
VALUES ('DR0002', 'Tab_WholeParts', 1);
```

(Adjust to actual schema of LX_Menus.)

### Sprint 3 — Frontend Hub v3.4.0

4-screen flow:
1. **Brand selector** — calls `PA_Hub_Parts_GetBrandsWithCategories`
2. **Category grid** — derived from brand selection
3. **Model selector** — calls `PA_Hub_Parts_GetModels`
4. **Parts grid** — calls `PA_Hub_Parts_GetCatalog`, displays:
   - Photo, PartNumber, Description, Color, Capacity
   - 5 inventory counters: IN PO / FedEx / F (Fishbowl) / IN PROC / M
   - Add to cart button
5. **Cart drawer** — submit calls `PA_Hub_Parts_SubmitOrder`
6. **My Orders view** — calls `PA_Hub_Parts_GetMyOrders`
   - Show status badge: synced / pending / closed
   - Retry button on pending → calls `PA_Hub_Parts_RetryPending` with single
     orderNumber

### Sprint 4 — Build & deploy

1. Run `node --check` on all modified JS in main script block
2. Validate IPC: `sql-exec`, `sql-ping`, `app-restart`
3. Package as `airlink-hub_3.4.0.zip`
4. Deploy via existing distribution channels

## Deferred (parking lot)

- ERP sub-section in Hub Supply Chain tab (pending Fishbowl PO status alignment)
- PA_Weather_Report flow for Santo Domingo, DR (HTML template stage)
- Print service investigation (warehouse computer)
- LX Agent bilingual support (separate task)
- DSM1 / BH / AQC2 modules to follow IQC pattern

---

# 📁 PROJECT FILE INVENTORY

## What this document REPLACES (safe to delete after loading this)

These files are now consolidated here:

- ❌ `Hub_Project_Status_v33.docx` — superseded by this MD
- ❌ `Airlink_AI_Project_State.md` — superseded by this MD
- ❌ Any partial sprint state docs from earlier sessions

## What to KEEP in the project

- ✅ `DB_Schema_Reference.txt` — canonical column references
- ✅ `Hub_SQL_Schema.docx` — formal schema documentation
- ✅ `LX_Agent_vFinal.html` — LX Agent template
- ✅ `Nexus_Integration_Documentation_v2.docx` — sister system reference
- ✅ `Movement_V1_5.xlsm` — legacy logic reference (IQC migration source)
- ✅ `PA_Flow_Checklists.docx` — flow build patterns

## To upload next

- ⬆️ `airlink-hub_3.3.7.zip` — current production state for context
- ⬆️ `airlink-hub_3.4.0.zip` — once Sprint 3+4 complete

---

# 🔑 CREDENTIALS & SECRETS REFERENCE

> ⚠️ **Do NOT paste actual values in this MD.** Reference only.
> Values stored securely in `D:\airlink-fb-svc\.env` and SQL Server.

| Service | Credential location |
|---|---|
| SQL Server `alappuser` password | Environment / connection strings |
| Fishbowl `shopify` user | `D:\airlink-fb-svc\.env` |
| Fishbowl appId=1013 | `D:\airlink-fb-svc\.env` |
| `SERVICE_API_KEY` (Bearer token) | `D:\airlink-fb-svc\.env` |
| ngrok auth | ngrok config on 192.168.181.246 |
| OpenWeatherMap API | Memory note (deferred PA flow) |

> ⚠️ **Action item:** Rotate Shopify API credentials previously exposed:
> `${SHOPIFY_ADMIN_TOKEN_LEGACY}`, `${SHOPIFY_API_SECRET_LEGACY}`

---

# 📚 GLOSSARY

| Term | Meaning |
|---|---|
| **Hub** | Airlink Hub desktop application (Electron) |
| **Whole Parts** | Internal parts request module (replaces Shopify+Vercel) |
| **WP-** | Order number prefix for Whole Parts orders |
| **SO** | Sales Order (in Fishbowl context) |
| **FB** | Fishbowl |
| **SP** | Stored Procedure |
| **PA** | Power Automate |
| **DR0002** | Jaime's employee code |
| **LX Agent** | Copilot Studio agent embedded in Hub |
| **alappuser** | SQL Server service account for Hub |
| **DSM / BH / AQC2** | Future Hub modules following IQC pattern |
| **IQC** | Incoming Quality Control (existing Hub module) |

---

# 📅 CHANGELOG

| Date | Sprint | Changes |
|---|---|---|
| 2026-04-28 | Phase 1D | Fishbowl integration deployed, airlink-fb-svc v1.0.4 |
| 2026-04-29 | Sprint 2A | Submit + FB sync — initial implementation |
| 2026-04-29 | Sprint 2B/2B-bis | GetMyOrders SP + flow with photos |
| 2026-04-29 | Sprint 2C/2C-bis | RetryPending SP + flow |
| 2026-05-01 | Sprint 2A re-validation | Bug discovered in Compose_LinesJson, fixed end-to-end |
| 2026-05-01 | **Sprint 2 COMPLETE** | All 5 sub-sprints validated, ready for Sprint 3 |

---

*Document maintained by Jaime Estevez, Director of Software & IT.*
*Last updated: 2026-05-01 end-of-day.*
*Next update: After Sprint 3 completion.*
