# DINO Ecosystem — Master Reference for v3.4.x+

**Generated:** 2026-05-01
**Source:** `DINO_ECOSYSTEM_MASTER.txt` + 11 individual `DINO_*.txt` system prompts
**Purpose:** Canonical reference for Airlink's 11-agent Dino ecosystem. Read this BEFORE planning any AI Agents Expansion sprint.
**Status:** **Planning phase — NOT yet implemented in Hub.** v3.3.7 has only 1 generic Dino.

---

## 1 · TL;DR

Airlink is moving from **1 generic Dino** to a **specialized 11-agent ecosystem**, each with:

- A unique **system prompt / persona** (the .txt file uploaded to chat)
- A unique **SharePoint knowledge base** scoped to the Dino's domain
- A unique **access control** (from "everyone" to "Victor only")
- A dedicated **Copilot Studio bot**
- A dedicated **Power Automate flow** (1 per Dino, replicated from `PA_Hub_Dino`)

The 11 Dinos are organized by access tier and deployment phase. **Hub UI must be redesigned** to host this catalog with proper access-gated visibility.

---

## 2 · The 11 Dinos — Roster

| # | Name | Emoji | Access | Domain | File |
|---|---|---|---|---|---|
| 1 | **DINO Nexus** | 🦖🏢 | 🟢 Everyone | Central operations hub (policies, SOPs, Fishbowl, glossary) | `DINO_NEXUS.txt` (6,543 chars) |
| 2 | **DINO KPI** | 🦖📊 | 🟢 Everyone | Performance metrics (operational only, no financial) | `DINO_KPI.txt` (6,073 chars) |
| 3 | **DINO Bark** | 🦖🎓 | 🟢 Everyone | Training, onboarding, 30/60/90 day milestones | `DINO_BARK.txt` (6,968 chars) |
| 4 | **DINO Data** | 🦖📈 | 🟡 Supervisors+ | Traceability, IMEI lookup, batch history, inventory | `DINO_DATA.txt` (6,986 chars) |
| 5 | **DINO ISO-R2** | 🦖📋♻️ | 🟡 Quality/Compliance/Exec/Auditors | ISO 9001/14001 + R2:2013 compliance | `DINO_ISO_R2.txt` (6,410 chars) |
| 6 | **DINO HR** | 🦖👥 | 🟠 HR + Executives | HR docs, policies, training records (SPN), compensation | `DINO_HR.txt` (6,367 chars) |
| 7 | **DINO Money** | 🦖💰 | 🟠 Finance + Exec + Sales(limited) | Pricing, costing, margins, bidding, market intel | `DINO_MONEY.txt` (7,751 chars) |
| 8 | **DINO IP** | 🦖🔐 | 🔴 Engineering leadership + NDA (NO Executives) | Engineering drawings, BOMs, trade secrets, supplier data | `DINO_IP.txt` (6,375 chars) |
| 9 | **DINO Leadership** | 🦖👔 | 🔴 Executive team only | Strategic plans, M&A, board materials, transitions | `DINO_LEADERSHIP.txt` (6,670 chars) |
| 10 | **DINO Bata** | 🦖🧪 | 🟡 Dev/Testing/Training | Sandbox mirroring Nexus for safe experimentation | `DINO_BETA.txt` (6,099 chars) |
| 11 | **My DINO** | 🦖💭 | 🔴 Victor only | Private thought partner, strategic reflection, brainstorm | `MY_DINO.txt` (6,055 chars) |

**All 11 system prompts are under the 8,000 character limit for Copilot Studio** ✓

---

## 3 · Access Matrix

```
                            ┌──────────────────────────────────────────────────┐
                            │                  EMPLOYEES                       │
                            ├────────────┬───────────┬──────────┬──────────────┤
                            │  General   │ Supervisor│ Manager  │  Executive   │
┌───────────────────────────┼────────────┼───────────┼──────────┼──────────────┤
│ 🟢 Nexus                  │     ✓      │     ✓     │    ✓     │      ✓       │
│ 🟢 KPI                    │     ✓      │     ✓     │    ✓     │      ✓       │
│ 🟢 Bark                   │     ✓      │     ✓     │    ✓     │      ✓       │
├───────────────────────────┼────────────┼───────────┼──────────┼──────────────┤
│ 🟡 Data                   │            │     ✓     │    ✓     │      ✓       │
│ 🟡 ISO-R2 (Q/Comp only)   │            │   *Q/C    │   *Q/C   │      ✓       │
│ 🟡 Bata (Dev/Test only)   │            │   *Dev    │   *Dev   │              │
├───────────────────────────┼────────────┼───────────┼──────────┼──────────────┤
│ 🟠 HR (HR team only)      │            │           │   *HR    │      ✓       │
│ 🟠 Money (Fin/Sales/Exec) │            │           │  *Fin/S  │      ✓       │
├───────────────────────────┼────────────┼───────────┼──────────┼──────────────┤
│ 🔴 IP (Eng + NDA)         │            │           │  *Eng+NDA│   ❌ DENIED  │
│ 🔴 Leadership             │            │           │          │      ✓       │
│ 🔴 My DINO                │            │           │          │  *Victor only│
└───────────────────────────┴────────────┴───────────┴──────────┴──────────────┘
```

**Notable rules:**
- **DINO IP excludes executives** — even C-suite can't access (per IP protection policy). Only Engineering leadership with active NDA.
- **My DINO is single-user** (Victor / president) — completely private.
- **DINO Money has tiered access** — Sales sees pricing only, Finance/Exec see costs + margins.
- **DINO Bata is environment-restricted** — only Dev/Test/Training team.

---

## 4 · Architectural Pattern

### 4.1 Current state (v3.3.7)

```
Hub UI (sidebar)         PA Flow              Copilot Studio
─────────────────        ──────────           ──────────────
[Dino tab]      ───────▶ HUB.dino   ────────▶  [Single Dino bot]
                         (one flow)              ├── KB: SharePoint general
                                                  └── Persona: generic
```

### 4.2 Target state (v3.4.x or v3.5.0)

```
Hub UI (sidebar)                    PA Flows                     Copilot Studio
─────────────────────────           ────────────────             ───────────────────
[Asistentes] (group)
├── LX Agent (existing)             (no change — iframe)
├── Claude (existing)               (no change — direct API)
└── [Dinos] (new sub-group)
    ├── 🦖🏢 Nexus       ────────▶ PA_Hub_Dino_Nexus      ────▶ DinoBot_Nexus       (KB: SP Operations)
    ├── 🦖📊 KPI         ────────▶ PA_Hub_Dino_KPI        ────▶ DinoBot_KPI         (KB: SP Dashboards)
    ├── 🦖🎓 Bark        ────────▶ PA_Hub_Dino_Bark       ────▶ DinoBot_Bark        (KB: SP Training+SPN)
    ├── 🦖📈 Data        ────────▶ PA_Hub_Dino_Data       ────▶ DinoBot_Data        (KB: SP Reports)
    ├── 🦖📋♻️ ISO-R2    ────────▶ PA_Hub_Dino_ISO_R2     ────▶ DinoBot_ISO_R2      (KB: SP Compliance)
    ├── 🦖👥 HR          ────────▶ PA_Hub_Dino_HR         ────▶ DinoBot_HR          (KB: SP HR)
    ├── 🦖💰 Money       ────────▶ PA_Hub_Dino_Money      ────▶ DinoBot_Money       (KB: SP Finance)
    ├── 🦖🔐 IP          ────────▶ PA_Hub_Dino_IP         ────▶ DinoBot_IP          (KB: SP Engineering)
    ├── 🦖👔 Leadership  ────────▶ PA_Hub_Dino_Leadership ────▶ DinoBot_Leadership  (KB: SP Strategic)
    ├── 🦖🧪 Bata        ────────▶ PA_Hub_Dino_Bata       ────▶ DinoBot_Bata        (KB: SP Sandbox)
    └── 🦖💭 My DINO     ────────▶ PA_Hub_MyDino          ────▶ DinoBot_MyDino      (KB: Victor private SP)

Shared infrastructure (no change):
├── HUB.hist.list
├── HUB.hist.save     ← used by Claude + ALL Dinos for chat history
├── HUB.hist.load
├── HUB.hist.del
└── HUB.sess          ← session creation
```

### 4.3 Why "1 flow per Dino" wins over "1 flow with switch"

| Concern | 1 flow per Dino (chosen) | 1 flow with switch |
|---|---|---|
| Access control | ✅ Per-flow SQL check (cleanest) | ⚠️ All checks in one switch (complex) |
| Audit logging | ✅ Per-Dino granularity | ⚠️ One log stream, must filter |
| Failure isolation | ✅ Dino down doesn't affect others | ❌ Single point of failure |
| Maintenance | ⚠️ 11 flows to maintain | ✅ 1 flow |
| Copy-paste cost | ⚠️ 11 flows (mostly identical) | ✅ Minimal |
| Add new Dino | ⚠️ Clone flow + update Hub | ✅ Add case to switch |

**Verdict:** For an ecosystem with NDAs, executive exclusions, and single-user privacy (My DINO), the isolation benefit outweighs maintenance cost.

---

## 5 · PA Flow Contract (Standard for All 11)

All Dino flows share the same input/output schema:

### Input
```json
{
  "employeeCode": "DR0002",
  "employeeName": "Jaime Estevez",
  "sessionID": "abc123-...",
  "question": "user's question text"
}
```

### Output
```json
{
  "sessionID": "abc123-...",
  "response": "Dino's reply text",
  "accessGranted": true,
  "errorCode": null
}
```

### Access denied response
```json
{
  "sessionID": null,
  "response": "🚫 No tienes acceso a este Dino. Contacta a tu supervisor.",
  "accessGranted": false,
  "errorCode": "ACCESS_DENIED"
}
```

### Flow steps (template per Dino)

```
1. HTTP Request Trigger (POST)
   - Schema: {employeeCode, employeeName, sessionID, question}

2. Compose_Meta
   - dinoId: "Nexus" (or whichever Dino)
   - timestamp: utcNow()
   - source: "Hub_v3.4.x"

3. SQL: Verify Access
   - SP_Hub_Dino_VerifyAccess
   - Params: @EmployeeCode, @DinoId
   - Returns: {accessGranted: bit, reason: varchar}

4. Condition: accessGranted = 1
   - YES: continue to step 5
   - NO:  return access denied response (status 200, accessGranted=false)

5. Condition: sessionID is empty
   - YES: SP_Hub_NewSession (@EmployeeCode, @AgentType='DINO_NEXUS')
   - NO:  use provided sessionID

6. HTTP: Call Copilot Studio Direct Line
   - URL: bot-specific Direct Line endpoint
   - Headers: Authorization: Bearer {bot-secret}
   - Body: {message, sessionID, userContext}

7. Parse response

8. SQL: Log access (audit trail)
   - SP_Hub_Dino_LogAccess (@EmployeeCode, @DinoId, @SessionID, @QuestionLength, @ResponseLength)

9. Return Response (status 200)
```

---

## 6 · SQL Schema Extensions Needed

### 6.1 New SP: `SP_Hub_Dino_VerifyAccess`

```sql
CREATE OR ALTER PROCEDURE SP_Hub_Dino_VerifyAccess
    @EmployeeCode VARCHAR(20),
    @DinoId VARCHAR(50)  -- 'Nexus', 'KPI', 'Bark', 'Data', 'ISO_R2', 'HR',
                          -- 'Money', 'IP', 'Leadership', 'Bata', 'MyDino'
AS
BEGIN
    SET NOCOUNT ON;
    DECLARE @AccessGranted BIT = 0;
    DECLARE @Reason VARCHAR(200) = '';

    -- Tier 1: Everyone (Nexus, KPI, Bark)
    IF @DinoId IN ('Nexus', 'KPI', 'Bark')
    BEGIN
        SET @AccessGranted = 1;
        SET @Reason = 'Tier 1 - All employees';
    END
    -- Tier 2: Supervisor and above (Data)
    ELSE IF @DinoId = 'Data'
    BEGIN
        IF EXISTS (
            SELECT 1 FROM SPN.dbo.LX_Menus
            WHERE EmployeeCode = @EmployeeCode
              AND ReportLevel >= 2  -- supervisor or higher
        )
        BEGIN
            SET @AccessGranted = 1;
            SET @Reason = 'Supervisor+';
        END
        ELSE SET @Reason = 'Requires supervisor level or above';
    END
    -- Tier 3: Specific role-based (HR, Money, ISO_R2, Bata)
    ELSE IF @DinoId = 'HR'
    BEGIN
        IF EXISTS (
            SELECT 1 FROM SPN.dbo.LX_Menus
            WHERE EmployeeCode = @EmployeeCode
              AND (Dino_HR = 1 OR Tab_Executive = 1)
        )
        BEGIN
            SET @AccessGranted = 1;
            SET @Reason = 'HR team or Executive';
        END
        ELSE SET @Reason = 'Requires HR team or Executive role';
    END
    -- ... similar logic for Money, ISO_R2, Bata, Leadership ...

    -- IP: Engineering leadership + NDA, EXCLUDES executives
    ELSE IF @DinoId = 'IP'
    BEGIN
        IF EXISTS (
            SELECT 1 FROM SPN.dbo.LX_Menus
            WHERE EmployeeCode = @EmployeeCode
              AND Dino_IP_NDA_Signed = 1
              AND Dino_IP_EngLeadership = 1
              AND (Tab_Executive IS NULL OR Tab_Executive = 0)  -- exclude execs
        )
        BEGIN
            SET @AccessGranted = 1;
            SET @Reason = 'Engineering leadership with active NDA';
        END
        ELSE SET @Reason = 'Requires Eng leadership + active NDA, excludes executives';
    END
    -- My DINO: Victor only (DR0001 or whatever Victor's code is)
    ELSE IF @DinoId = 'MyDino'
    BEGIN
        IF @EmployeeCode = 'DR0001'  -- Victor's code (CONFIRM)
        BEGIN
            SET @AccessGranted = 1;
            SET @Reason = 'Victor only';
        END
        ELSE SET @Reason = 'My DINO is restricted to Victor only';
    END

    SELECT @AccessGranted AS AccessGranted, @Reason AS Reason;
END
GO

GRANT EXECUTE ON SP_Hub_Dino_VerifyAccess TO alappuser;
```

### 6.2 New SP: `SP_Hub_Dino_LogAccess` (audit trail)

```sql
CREATE TABLE Hub_DinoAccessLog (
    LogID         INT IDENTITY PRIMARY KEY,
    EmployeeCode  VARCHAR(20),
    DinoId        VARCHAR(50),
    SessionID     UNIQUEIDENTIFIER,
    QuestionLen   INT,
    ResponseLen   INT,
    AccessedAt    DATETIME DEFAULT GETDATE()
);

CREATE OR ALTER PROCEDURE SP_Hub_Dino_LogAccess
    @EmployeeCode VARCHAR(20),
    @DinoId VARCHAR(50),
    @SessionID UNIQUEIDENTIFIER,
    @QuestionLength INT,
    @ResponseLength INT
AS
BEGIN
    SET NOCOUNT ON;
    INSERT INTO Hub_DinoAccessLog (EmployeeCode, DinoId, SessionID, QuestionLen, ResponseLen)
    VALUES (@EmployeeCode, @DinoId, @SessionID, @QuestionLength, @ResponseLength);
END
GO

GRANT EXECUTE ON SP_Hub_Dino_LogAccess TO alappuser;
```

### 6.3 LX_Menus extensions

Add columns to `SPN.dbo.LX_Menus`:

```sql
ALTER TABLE SPN.dbo.LX_Menus ADD
    Dino_HR              BIT NOT NULL DEFAULT 0,
    Dino_Money_Sales     BIT NOT NULL DEFAULT 0,  -- pricing only
    Dino_Money_Finance   BIT NOT NULL DEFAULT 0,  -- full access
    Dino_ISO_R2          BIT NOT NULL DEFAULT 0,
    Dino_Bata            BIT NOT NULL DEFAULT 0,
    Dino_IP_EngLeadership BIT NOT NULL DEFAULT 0,
    Dino_IP_NDA_Signed   BIT NOT NULL DEFAULT 0,
    Dino_Leadership      BIT NOT NULL DEFAULT 0,
    Tab_Executive        BIT NOT NULL DEFAULT 0;  -- generic Executive flag

-- Note: Tier 1 Dinos (Nexus, KPI, Bark) need NO column — always granted
-- Note: Data uses ReportLevel >= 2 (supervisor)
-- Note: My DINO check is hardcoded (single user)
```

### 6.4 Update `SP_Hub_GetMenus`

The current SP returns `Tab_*` columns. Extend to also return `Dino_*` columns so the Hub knows which Dinos to show in the sidebar.

---

## 7 · Hub UI Integration Plan

### 7.1 Sidebar restructure

Current chat sidebar (3 agents flat):
```
[Asistentes]
├── 🧠 LX Agent    (always visible)
├── 🤖 Claude      (always visible)
└── 🐶 Dino        (always visible)
```

New chat sidebar (Dinos as collapsible group):
```
[Asistentes]
├── 🧠 LX Agent
├── 🤖 Claude
└── [🦖 Dinos] ▼
    ├── 🦖🏢 Nexus       (visible if Dino_Nexus access)
    ├── 🦖📊 KPI         (always)
    ├── 🦖🎓 Bark        (always)
    ├── 🦖📈 Data        (visible if supervisor+)
    ├── 🦖📋♻️ ISO-R2    (visible if Dino_ISO_R2)
    ├── 🦖👥 HR          (visible if Dino_HR)
    ├── 🦖💰 Money       (visible if Dino_Money_*)
    ├── 🦖🔐 IP          (visible if Dino_IP_EngLeadership AND Dino_IP_NDA_Signed)
    ├── 🦖👔 Leadership  (visible if Tab_Executive)
    ├── 🦖🧪 Bata        (visible if Dino_Bata)
    └── 🦖💭 My DINO     (visible if EmployeeCode = Victor's)
```

### 7.2 Code changes in `index.html`

#### `AGCFG` (line 691) — add 11 entries

```javascript
var AGCFG = {
  lx: { icon:'🧠', name:'LX Agent', sub:'Reporte Diario · Airlink Distribution', hist:false, img:null },
  cl: { icon:'🤖', name:'Claude',   sub:'Asistente IA · claude-sonnet-4-6',     hist:true,  img:'...' },

  // NEW Dino entries — replace single 'dn' entry
  'dn-nexus':      { icon:'🦖🏢',  name:'DINO Nexus',      sub:'Operaciones · Airlink',           hist:true, dinoId:'Nexus',      flowKey:'dinoNexus' },
  'dn-kpi':        { icon:'🦖📊',  name:'DINO KPI',         sub:'Métricas operacionales',          hist:true, dinoId:'KPI',         flowKey:'dinoKpi' },
  'dn-bark':       { icon:'🦖🎓',  name:'DINO Bark',        sub:'Training & onboarding',           hist:true, dinoId:'Bark',        flowKey:'dinoBark' },
  'dn-data':       { icon:'🦖📈',  name:'DINO Data',        sub:'Trazabilidad operacional',        hist:true, dinoId:'Data',        flowKey:'dinoData' },
  'dn-iso-r2':     { icon:'🦖📋',  name:'DINO ISO-R2',      sub:'Compliance ISO + R2',             hist:true, dinoId:'ISO_R2',      flowKey:'dinoIsoR2' },
  'dn-hr':         { icon:'🦖👥',  name:'DINO HR',          sub:'Recursos Humanos',                hist:true, dinoId:'HR',          flowKey:'dinoHr' },
  'dn-money':      { icon:'🦖💰',  name:'DINO Money',       sub:'Pricing & Finanzas',              hist:true, dinoId:'Money',       flowKey:'dinoMoney' },
  'dn-ip':         { icon:'🦖🔐',  name:'DINO IP',          sub:'Engineering · Trade Secrets',     hist:true, dinoId:'IP',          flowKey:'dinoIp' },
  'dn-leadership': { icon:'🦖👔',  name:'DINO Leadership',  sub:'Strategic Vault',                 hist:true, dinoId:'Leadership',  flowKey:'dinoLeadership' },
  'dn-bata':       { icon:'🦖🧪',  name:'DINO Bata',        sub:'Sandbox & Testing',               hist:true, dinoId:'Bata',        flowKey:'dinoBata' },
  'dn-my':         { icon:'🦖💭',  name:'My DINO',          sub:'Private Thought Partner',         hist:true, dinoId:'MyDino',      flowKey:'myDino' }
};
```

#### `HUB` object (line 645) — add 11 flow URLs

```javascript
var HUB = {
  // ... existing ...
  dino:           _c('...'),  // legacy generic — keep for compat then deprecate
  dinoNexus:      _c('...'),
  dinoKpi:        _c('...'),
  dinoBark:       _c('...'),
  dinoData:       _c('...'),
  dinoIsoR2:      _c('...'),
  dinoHr:         _c('...'),
  dinoMoney:      _c('...'),
  dinoIp:         _c('...'),
  dinoLeadership: _c('...'),
  dinoBata:       _c('...'),
  myDino:         _c('...')
};
```

#### Refactor `sendD()` (line 2627) → generic `sendDino(agentKey)`

```javascript
function sendDino(agentKey) {
  var cfg = AGCFG[agentKey];
  if (!cfg || !cfg.dinoId) return;

  var inp = document.getElementById('dinp'),
      t = inp.value.trim();
  if (!t) return;

  inp.value = ''; inp.style.height = 'auto';
  document.getElementById('dsnd').disabled = true;

  if (!_chatId) _chatId = mkUUID();
  var isFirst = _seqD === 0;
  var title = isFirst ? mkTitle(t) : '';
  _seqD++;
  var uSeq = _seqD;

  addM('d','u',t); DC++;
  showT('d', cfg.icon, true);
  paSave('dn-' + cfg.dinoId.toLowerCase(), _chatId, isFirst ? title : 'update', uSeq, 'user', t);

  var go = function() {
    fetch(HUB[cfg.flowKey], {  // ← uses per-Dino flow URL
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        employeeCode: CU.code,
        employeeName: CU.name,
        sessionID: DS,
        question: t
      })
    })
    .then(function(r) { if (!r.ok) throw new Error('HTTP ' + r.status); return r.json(); })
    .then(function(d) {
      if (d.accessGranted === false) {
        rmT('d');
        addM('d', 'b', '🚫 ' + (d.response || 'Acceso denegado.'), true);
        return;
      }
      if (d.sessionID) DS = d.sessionID;
      var rep = d.response || '⚠️ Sin respuesta.';
      rmT('d'); addM('d', 'b', rep, true);
      document.getElementById('dcnt').textContent = DC + ' mensajes';
      _seqD++;
      paSave('dn-' + cfg.dinoId.toLowerCase(), _chatId, isFirst ? title : 'update', _seqD, 'assistant', rep);
      renderHist('dn-' + cfg.dinoId.toLowerCase());
    })
    .catch(function(err) { rmT('d'); addM('d','b','⚠️ Error: ' + err.message, true); });

    document.getElementById('dsnd').disabled = false;
    document.getElementById('dinp').focus();
  };

  if (!DS) {
    fetch(HUB.sess, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ employeeCode: CU.code, agentType: 'DINO_' + cfg.dinoId.toUpperCase() })
    })
    .then(function(r) { return r.json(); })
    .then(function(d) { DS = d.sessionID || 'local-' + Date.now(); go(); })
    .catch(function() { DS = 'local-' + Date.now(); go(); });
  } else { go(); }
}
```

#### Sidebar rendering — collapsible Dino group

Update sidebar render to group all `dn-*` entries under a collapsible "Dinos" header. Reuse the existing `makeSbGroup()` pattern that's already used for tree navs.

#### Visibility filter

After login, `applyTabVisibility(menus)` already iterates over `TAB_MAP`. Extend with a `DINO_MAP`:

```javascript
var DINO_MAP = [
  { agentKey:'dn-nexus',      col:null },                                   // always
  { agentKey:'dn-kpi',        col:null },                                   // always
  { agentKey:'dn-bark',       col:null },                                   // always
  { agentKey:'dn-data',       col:null,    minLevel:2 },                    // supervisor+
  { agentKey:'dn-iso-r2',     col:'Dino_ISO_R2' },
  { agentKey:'dn-hr',         col:'Dino_HR' },
  { agentKey:'dn-money',      col:'Dino_Money_Sales',
                              colAny:['Dino_Money_Sales', 'Dino_Money_Finance', 'Tab_Executive'] },
  { agentKey:'dn-ip',         col:null,
                              colAll:['Dino_IP_EngLeadership', 'Dino_IP_NDA_Signed'],
                              excludeIf:'Tab_Executive' },                  // execs blocked
  { agentKey:'dn-leadership', col:'Tab_Executive' },
  { agentKey:'dn-bata',       col:'Dino_Bata' },
  { agentKey:'dn-my',         col:null,    onlyEmployeeCode:'DR0001' }      // Victor
];

function applyDinoVisibility(menus, employeeCode) {
  DINO_MAP.forEach(function(d) {
    var btn = document.querySelector('[data-agent-key="' + d.agentKey + '"]');
    if (!btn) return;

    var allowed = false;

    if (d.onlyEmployeeCode) {
      allowed = (employeeCode === d.onlyEmployeeCode);
    } else if (d.minLevel) {
      allowed = (CU.lvl >= d.minLevel);
    } else if (d.colAll) {
      allowed = d.colAll.every(function(c) { return menus[c] == 1; });
      if (allowed && d.excludeIf && menus[d.excludeIf] == 1) allowed = false;
    } else if (d.colAny) {
      allowed = d.colAny.some(function(c) { return menus[c] == 1; });
    } else if (d.col) {
      allowed = (menus[d.col] == 1);
    } else {
      allowed = true;  // Tier 1
    }

    btn.style.display = allowed ? '' : 'none';
  });
}
```

---

## 8 · Copilot Studio Bot Configuration (Per Dino)

### 8.1 Naming convention

| Bot Name in Studio | Direct Line ID | Hub flow that calls it |
|---|---|---|
| `Airlink-Dino-Nexus` | (auto-generated) | `PA_Hub_Dino_Nexus` |
| `Airlink-Dino-KPI` | (auto-generated) | `PA_Hub_Dino_KPI` |
| `Airlink-Dino-Bark` | (auto-generated) | `PA_Hub_Dino_Bark` |
| `Airlink-Dino-Data` | (auto-generated) | `PA_Hub_Dino_Data` |
| `Airlink-Dino-ISO-R2` | (auto-generated) | `PA_Hub_Dino_ISO_R2` |
| `Airlink-Dino-HR` | (auto-generated) | `PA_Hub_Dino_HR` |
| `Airlink-Dino-Money` | (auto-generated) | `PA_Hub_Dino_Money` |
| `Airlink-Dino-IP` | (auto-generated) | `PA_Hub_Dino_IP` |
| `Airlink-Dino-Leadership` | (auto-generated) | `PA_Hub_Dino_Leadership` |
| `Airlink-Dino-Bata` | (auto-generated) | `PA_Hub_Dino_Bata` |
| `Airlink-MyDino` | (auto-generated) | `PA_Hub_MyDino` |

### 8.2 Per-bot configuration steps

For each Dino:

1. **Create new bot** in Copilot Studio
2. **Set instructions** = paste contents of corresponding `DINO_*.txt` (under 8K char limit ✓)
3. **Connect knowledge sources** (SharePoint sites — see SharePoint mapping below)
4. **Configure Direct Line** channel
5. **Test** in Copilot Studio canvas before connecting to Hub

### 8.3 SharePoint KB mapping (TO BE CONFIRMED with SharePoint admin)

| Dino | Suggested SharePoint location |
|---|---|
| Nexus | `/sites/AirlinkOps` (operational docs, glossary, policies) |
| KPI | `/sites/AirlinkDashboards` (Power BI reports, dashboard exports) |
| Bark | `/sites/AirlinkTraining` (onboarding, SPN tracking exports) |
| Data | (none — direct SQL via PA flow) |
| ISO-R2 | `/sites/AirlinkCompliance/ISO`, `/sites/AirlinkCompliance/R2` |
| HR | `/sites/AirlinkHR` (restricted by SP permissions) |
| Money | `/sites/AirlinkFinance/Pricing`, `/sites/AirlinkFinance/Costing` |
| IP | `/sites/AirlinkEngineering/IP` (highly restricted SP permissions) |
| Leadership | `/sites/AirlinkExecutive` (C-suite SP) |
| Bata | (mirror of Operations site, `/sites/AirlinkOpsTest`) |
| My DINO | Victor's personal OneDrive folder, scoped only to him |

⚠️ **Critical:** SharePoint permissions must align with Dino access tiers. Even if PA flow grants access, SP itself enforces document-level security. **Both layers must be configured.**

---

## 9 · Migration Plan (v3.3.7 → Dino Ecosystem)

### Phase 0: Foundation (NOT shipped to users yet)

1. ☐ Create `SP_Hub_Dino_VerifyAccess` and `SP_Hub_Dino_LogAccess` SQL
2. ☐ Add `Hub_DinoAccessLog` table
3. ☐ Extend `LX_Menus` with new `Dino_*` columns
4. ☐ Update `SP_Hub_GetMenus` to return new columns
5. ☐ Configure SharePoint sites/permissions per Dino (with SP admin)

### Phase 1: Foundation Dinos (Tier 1 - Everyone)

6. ☐ Build `DinoBot_Nexus` in Copilot Studio + connect KB + test
7. ☐ Build `DinoBot_KPI` + KB + test
8. ☐ Build `DinoBot_Bark` + KB + test
9. ☐ Create `PA_Hub_Dino_Nexus`, `PA_Hub_Dino_KPI`, `PA_Hub_Dino_Bark` flows
10. ☐ Refactor Hub `sendD()` → `sendDino(agentKey)` (generic)
11. ☐ Hub UI: add 3 new entries to `AGCFG` + collapsible group
12. ☐ Test end-to-end in dev
13. ☐ Ship as **v3.4.x** patch (after Whole Parts ships)

### Phase 2: Operational Dinos (Tier 2 - Supervisors+)

14. ☐ Build DinoBot_Data + KB
15. ☐ Build DinoBot_ISO_R2 + KB
16. ☐ Build DinoBot_Bata + KB
17. ☐ Create 3 PA flows
18. ☐ Hub UI: add 3 entries
19. ☐ Test
20. ☐ Ship as **v3.5.x**

### Phase 3: Sensitive Dinos (Tiers 3-4)

21. ☐ Build DinoBot_HR, Money, IP, Leadership, MyDino
22. ☐ Special access checks in PA flows (NDA verification, single-user)
23. ☐ Audit logging on Eng/Exec/Victor flows
24. ☐ Create 5 PA flows
25. ☐ Hub UI: add 5 entries
26. ☐ Test with stakeholders (HR Director, CFO, Victor)
27. ☐ Ship as **v3.6.0**

---

## 10 · Critical Rules All Dinos Share

From the system prompt files, ALL Dinos enforce these rules:

### 10.1 Language matching (MANDATORY)
- Spanish query → respond entirely in Spanish
- English query → respond entirely in English
- NEVER mix languages within a response

### 10.2 Document creation forbidden
- Dinos CANNOT create downloadable files (PDF, Word, Excel)
- Dinos CAN create visual summaries, diagrams, tables IN responses
- NEVER provide download links to non-existent documents (causes 404)
- Only link to EXISTING documents in SharePoint

### 10.3 Dual access options for documents

```
Spanish: "📄 [Documento] 👁️ [Ver en línea](webUrl) | 📥 [Descargar](downloadUrl)"
English: "📄 [Document] 👁️ [View online](webUrl) | 📥 [Download](downloadUrl)"
```

### 10.4 Cross-Dino redirects

Each Dino knows about the others and redirects appropriately:

```
DINO Nexus → "Para compliance, consulta DINO ISO-R2"
DINO KPI   → "Para análisis profundo, consulta DINO Data"
DINO Bark  → "Para procedimientos completos, consulta DINO Nexus"
DINO Money → "Para datos operativos, consulta DINO Data"
DINO HR    → "Para training, consulta DINO Bark"
... (etc.)
```

This pattern means **users get routed to the right Dino** instead of getting wrong answers from the wrong specialist.

---

## 11 · Security & Compliance Considerations

### 11.1 Audit requirements

| Dino | Audit log requirement |
|---|---|
| Nexus, KPI, Bark | Optional (low sensitivity) |
| Data, ISO-R2, Bata | Standard logging |
| HR, Money | Enhanced logging (who accessed what when) |
| **IP** | **Maximum logging — every access logged with reason** |
| **Leadership** | **Maximum logging — board-relevant material** |
| **My DINO** | **Audit-exempt — privacy paramount** (Victor's choice) |

### 11.2 Pre-decisional information (Leadership)

DINO Leadership specifically warns:
> "⚠️ This document is pre-decisional. No compartir hasta aprobación final."

Hub UI should reinforce this — maybe a banner when DINO Leadership is selected.

### 11.3 NDA verification (IP)

Before EVERY session with DINO IP, the bot prompts:
> "Before proceeding, I must verify: 1) You are Engineering leadership, 2) You have an active NDA on file, 3) You understand all access is logged."

The PA flow's access check (SQL) handles 1 and 3. The bot itself prompts for 2 (NDA confirmation). **This is intentional double-verification.**

### 11.4 My DINO privacy

> "No other users, no sharing, no logging to company systems."

Implication: The standard `Hub_DinoAccessLog` should NOT log My DINO usage. The PA flow for My DINO should skip the audit step. Document this clearly to avoid accidental logging.

---

## 12 · Implementation Cost Estimate (Sprint 4+)

| Component | Effort estimate |
|---|---|
| SQL: SP_Hub_Dino_VerifyAccess + LogAccess + table + LX_Menus extension | 2-3 hours |
| Copilot Studio: 11 bots × ~30 min config each | 6-8 hours |
| Copilot Studio: 11 KB connections + tests | 4-6 hours |
| PA Flows: 11 flows × ~30 min (template-based) | 6-8 hours |
| Hub UI: refactor sendD → sendDino | 2 hours |
| Hub UI: AGCFG + sidebar group + visibility filter | 3-4 hours |
| Hub UI: testing + integration | 4 hours |
| Documentation: per-Dino runbook | 2 hours |
| Security review | 2 hours |
| **TOTAL** | **31-39 hours** (roughly 1 work-week of focused effort) |

⚠️ **This DOES NOT include**:
- SharePoint site creation/permissions (depends on SP admin)
- KB content curation (need to identify which docs to feed each Dino)
- Bot prompt tuning per real-world testing
- User training rollout

Realistic timeline: **~2-3 weeks** end-to-end if Sprint 4 is dedicated to this.

---

## 13 · Dino-Specific Special Behaviors

### DINO Bark — Quiz mode
When user says "quiz me" or "pruébame", Bark generates 3-5 multiple choice questions with feedback.

### DINO Data — IMEI lookup
Specifically responds to IMEI/serial numbers with full device timeline. Hub could potentially deep-link to this from the existing Production → Unit Lookup feature.

### DINO Money — Tiered responses
Sales sees pricing only; Finance/Exec see costs + margins. The bot itself enforces this based on context passed by PA flow (employeeCode → role lookup).

### DINO IP — NDA prompt
First message of every session prompts for NDA confirmation before any content.

### DINO Bata — Sandbox markers
All responses prefixed with 🧪 and `[TEST]` to prevent confusion with production.

### My DINO — 5 modes
Victor can invoke specific modes:
1. Reflection (passive listening)
2. Brainstorm (active idea generation)
3. Challenge (devil's advocate)
4. Organize (framework building)
5. Journal (capture and affirm)

---

## 14 · Open Questions Before Implementation

Before starting Sprint 4, these need answers:

1. ☐ **Victor's employee code?** — `MY_DINO` access check needs the exact value (currently assumed `DR0001`)
2. ☐ **SharePoint admin engagement** — who configures the SP sites and permissions?
3. ☐ **NDA tracking** — where is NDA signed status currently maintained? (Need source for `Dino_IP_NDA_Signed` column)
4. ☐ **Engineering leadership list** — exact employees who get IP access (Director Eng + above? Specific roles?)
5. ☐ **Executive list** — confirms `Tab_Executive` users (CEO, CFO, COO, CTO, Victor as President, others?)
6. ☐ **Sales leadership** — exact list for `Dino_Money_Sales` access
7. ☐ **Bata team** — exact list for `Dino_Bata` (Software & IT? Anyone else?)
8. ☐ **Quality / Compliance team** — exact list for `Dino_ISO_R2`
9. ☐ **HR team** — exact list for `Dino_HR`
10. ☐ **DinoBot_Data** — should this connect to SharePoint, or directly call SQL via PA inside the bot? (recommendation: SQL via PA, like a "tool call")

---

## 15 · Glossary

| Term | Meaning |
|---|---|
| **Dino** | One of the 11 specialized AI agents in this ecosystem |
| **Tier** | Access tier: 🟢 Everyone, 🟡 Restricted, 🟠 Sensitive, 🔴 Maximum |
| **DINO_ID** | Internal identifier for the Dino (e.g., 'Nexus', 'IP', 'MyDino') |
| **agentKey** | Hub UI identifier (e.g., `dn-nexus`, `dn-ip`, `dn-my`) |
| **flowKey** | Key in HUB object pointing to PA flow URL (e.g., `dinoNexus`) |
| **Direct Line** | Microsoft channel API for Copilot Studio bot integration |
| **NDA** | Non-Disclosure Agreement — required for DINO IP access |
| **Pre-decisional** | Information not yet finalized/announced (DINO Leadership) |
| **CoC** | Chain of Custody (R2 compliance term, DINO ISO-R2) |
| **CAPA** | Corrective/Preventive Action (ISO 9001 term, DINO ISO-R2) |
| **MRI** | Material Receiving Inspection |
| **SPN** | Training tracking system (DINO HR + Bark integration) |

---

## 16 · References to Source Files

This MD synthesizes information from:

- `DINO_ECOSYSTEM_MASTER.txt` — Master catalog
- `DINO_NEXUS.txt` — Nexus system prompt (6,543 chars)
- `DINO_KPI.txt` — KPI system prompt (6,073 chars)
- `DINO_BARK.txt` — Bark system prompt (6,968 chars)
- `DINO_DATA.txt` — Data system prompt (6,986 chars)
- `DINO_ISO_R2.txt` — ISO-R2 system prompt (6,410 chars)
- `DINO_HR.txt` — HR system prompt (6,367 chars)
- `DINO_MONEY.txt` — Money system prompt (7,751 chars)
- `DINO_IP.txt` — IP system prompt (6,375 chars)
- `DINO_LEADERSHIP.txt` — Leadership system prompt (6,670 chars)
- `DINO_BATA.txt` — Bata system prompt (6,099 chars) — Note: file is `DINO_BETA.txt` but content is "Bata"
- `MY_DINO.txt` — My DINO system prompt (6,055 chars)

**Total: ~92K chars across 12 files**

When implementing, copy the contents of each `DINO_*.txt` directly into the corresponding Copilot Studio bot's instructions field.

---

**End of Dino Ecosystem Reference.**

Use this as the canonical planning document for Sprint 4+ (AI Agents Expansion). Update when implementation begins to track actual flow URLs, bot IDs, and SharePoint sites.
