# 📒 Mi Bitácora · Airlink Distribution DR

**Owner:** Jaime Estevez · DR0002 · Director Software & IT
**Iniciada:** 2026-05-06
**Última actualización:** 2026-05-06

> **Propósito:** Documento vivo para llevar continuidad de mi día a día en Airlink. Sirve como fuente de verdad para mis daily reports, contexto entre conversaciones con Claude, y memoria persistente de iniciativas, aprendizajes y compromisos. Cada día se agrega una entrada nueva al tope de la sección de Bitácora Diaria.

---

## 🧭 Cómo usar este documento

**Al inicio del día (o cuando llegue input nuevo):**
- Transcripts de reuniones → van a la entrada del día actual, sub-sección **Reuniones**
- Conversaciones técnicas con el equipo → van a **Colaboración**
- Errores, bugs, bloqueos → van a **Problemas y dificultades**
- Aprendizajes técnicos nuevos → al final del día se promueven a la sección **Aprendizajes y Patrones**
- Decisiones arquitectónicas importantes → a **Decisiones técnicas registradas**
- Ideas para mejorar procesos → a **Sugerencias / Backlog**

**Al final del día:**
- Cerrar la entrada del día con el estatus de cada prioridad (✅ ❌ 🔄)
- Definir las prioridades para mañana al final de la entrada de hoy
- Si algo no se completó, automáticamente se hereda a mañana o se mueve al backlog

**Al inicio del día siguiente:**
- Las "Prioridades para mañana" de ayer se convierten en las prioridades a revisar en Q1 del daily report
- Se crea una entrada nueva al tope con la fecha del día

---

## 🪪 Sobre Mí

### Identidad y rol
- **Nombre:** Jaime Joan Estevez
- **Código de empleado:** DR0002
- **Posición:** Director of Software & IT
- **Departamento:** IT / Software
- **Supervisor directo:** Victor Abuharoon (DR0668, Presidente)
- **Empresa:** Airlink Distribution DR (Free Trade Zone, República Dominicana)

### Background
- Mechatronics Engineer (INTEC)
- ~10 años en mechatronics, machining, manufacturing engineering, electrical/electronics, control systems, automation industrial
- Director del stack tecnológico AI-driven de Airlink

### Misión actual
- **Misión primaria:** Construir y escalar el stack tecnológico AI-driven de Airlink, con target de 70% de sistemas AI-driven y volume scaling significativo.
- **Misión paralela:** Posicionar a Airlink favorablemente en la evaluación formal de adquisición por Assurant (Biju Nair, Brandon Johnson).

### Estilo de trabajo
- Iterativo con capacidad explícita de rollback ("rollback" es instrucción real, se honra inmediatamente)
- Análisis antes de documentación — completar análisis primero, generar artefactos después
- Comunicación en español; artefactos técnicos en inglés
- Preferencia por evaluaciones técnicas honestas sobre hedging diplomático
- Validación de SQL/IPC contra stored procedures y schemas reales antes de wirear UI
- Entrega por fases: SQL schema → PA flow spec → frontend implementation
- MDs de referencia al final de sesiones para preservar continuidad

---

## 👥 Equipo y Stakeholders

### Reportes directos
| Persona | Código | Rol | Área |
|---|---|---|---|
| Jorge Ozuna | DR0005 | Desarrollador Semi Senior | Software |
| Wilson Hilario | DR0143 | Desarrollador | Software |
| Adonis Diaz | DR0155 | Supervisor de IT | IT |
| Massiel Gonzalez | DR0287 | Analista de Ciberseguridad | IT |

### Cross-functional partners
- **Irvin De La Rosa** (DR0941) — Software/Movement
- **Joel Fermín** — R2 ISO compliance, auditoría
- **Emilio Alfonso** — Engineering
- **Haxel** — ERP / Fishbowl
- **Osterman** — Producción
- **Anthony Pasiminio** — Network infrastructure

### Liderazgo / Owners
- **Gabriel y Junior** — Owners
- **Victor Abuharoon** — Presidente (DR0668)
- **Michael Petiton** — Director de Finanzas

### Assurant (acquisition)
- **Biju Nair** — EVP & President Global Connected Living
- **Brandon Johnson** — SVP Global Supply Chain & Engineering

---

## 🛠 Stack Técnico Core

### Plataformas principales
- **Fishbowl ERP** (servicio: `airlink-hub`)
- **Movement / Nexus** (PMS / WMS)
- **AirlinkHub** (Electron desktop app + agentes DINO)
- **Phonecheck** (genera PDFs de sanitización auto)
- **SQL Server** principal: `192.168.181.248:13999` (DBs: SPN + AirlinkDR; AirlinkDR es target de migración)
- **SQL Server dev:** `192.168.181.246:8989` (ARLQA)
- **AirlinkUSA:** AFS2SQL1
- **Microsoft 365 + Power Automate**
- **Azure Custom Vision**
- **Node.js dashboards**
- **Fortinet** (upgrade próximo mes)
- **Zebra/ZPL printers**
- **FedEx API**
- **Assurant:** Oracle ERP

### Tunnels / endpoints
- **ngrok producción TV:** `airlink-production-tv.ngrok.app`
- **Vercel:** `airlink-shopify-be.vercel.app` (logs vía dashboard manual)
- **OpenWeatherMap API:** `${OPENWEATHERMAP_API_KEY}` (PA_Weather_Report, Santo Domingo)

### Conexiones SQL
- Usuario: `alappuser` (también conocido como `ALAppUser`)
- DB principal: `AirlinkDR`
- DB legacy activa: `SPN`
- Inventario: `AIRLINK.dbo.SPC_PartInventory` (M-ring)

---

## 🚀 Iniciativas Activas

### 1. Hub v3.4.0 — Whole Parts module
**Estado:** Sprint 2D completado (2026-05-02), Sprint 3 pendiente
**Resumen:** Sistema interno de requisición de partes para empleados, reemplazando Shopify + Vercel backend. Empleados navegan/solicitan; Nexus/Movement `ShopifyOrder` cumple vía scan-to-fulfill.
**Próximos pasos:**
- Sprint 3: implementación de catálogo + cart + submit
- Phase 4: Warehouse Node.js print service
- Phase 5: Frontend completo
- Verificar TBDs Fishbowl REST API (PO endpoint path/filter, SO POST body schema)

### 2. DINO Ecosystem (11 agentes, 5-tier access pyramid)
**Estado:** Tier 1 (Nexus, KPI, Bark) en pruebas activas; resto planificado
**Cobertura de tiers:**
- 🟢 **Todos los empleados:** Nexus, KPI, Bark
- 🟡 **Supervisores+:** Data
- 🟠 **Role-specific:** ISO-R2, Bata, HR, Money
- 🔴 **Engineering leadership con NDA:** IP (executives explícitamente excluidos)
- 🔴 **Executives only:** Leadership
- 🔴 **Solo Victor Abuharoon:** My DINO

**Despliegue por fases:**
- **v3.4.x:** Tier 1 Dinos
- **v3.5.0:** Operacionales (Data, Bata)
- **v3.6.0:** Sensitivos (IP, Leadership, My DINO)

**Pruebas en curso (mayo 2026):** DINO Bark + DINO Nexus operatividad para auditoría R2 ISO

### 3. Auditoría R2 ISO (Stage 1 = 18-21 mayo · Stage 2 = 22-30 junio)
**Estado:** 70% de evidencias completadas tras hallazgos de Trasenda
**Conducido por:** Joel Fermín
**Stage 1 (remoto, 4 días):** Revisión de políticas, SOPs, Records, Training Documentation, evidencias
**Stage 2 (presencial, 6 días, 2 auditores):** Receiving, Producción, Data Processing, Warehouse, Shipping, E-Waste Storage

**Puntos pendientes críticos:**
- Documentación de Recicla (manejo de baterías, cert ISO del cliente)
- Formularios de calibración de equipos (matriz lista, falta certificado de Phoenix Calibration)
- 60 días consecutivos de evidencia CCTV (enero/febrero/marzo)
- Formalización del comité de seguridad
- Drill / simulacro de evacuación
- Cuarto de monitoreo CCTV (estructura)

**Mi participación directa:**
- Pruebas de DINO Bark + Nexus para que la auditoría tenga evidencia operativa
- Diseño de DINO ISO con visibilidad cruzada (propuesta arquitectónica)
- Pre-auditoría interna planificada antes del 14 de mayo

### 4. Acquisition Assurant
**Estado:** En espera de respuesta de Biju Nair tras reunión presencial
**Materiales:** HTML v7, PPTX 21 slides + 10 compact + 5 poster show-room, Word charter, presenter cheat sheet
**Co-presenter:** Vic (estratégico/competitivo)
**Standing rule:** Nunca referenciar a Biju en voz narrativa en copy; solo como co-inventor en citas USPTO

### 5. Carrier Lock + MDM Reporting
**Estado:** Estructura original validada (ver entrada 2026-05-06 sobre rollback)
**Componentes:**
- Vista `VW_CarrierLocked_withStatus` — 8 reglas de status
- `sp_CL_WeeklyPendingToSubmit_Report` (preview semanal)
- `sp_CL_WeeklyPendingToSubmit_Apply` (commit a SM_Master)
- Filtro: `LIKE '%GALAXY%'` (Samsung)

**Universo dual confirmado:**
- `VW_FULLDEVICEREPORT` = inventario en casa
- `SM_Master` = inventario en casa + inventario del cliente

### 6. LX Agent — Daily Report system
**Estado:** Operativo con limitación de tokens/timeout pendiente de fix
**Componentes:** Frontend Electron + 5 PA flows + Excel + PDF generation
**Issue activo:** Limitación de tokens por turno + timeout de Power Automate causa caídas con respuestas largas (afectó a Adonis)

### 7. Movement-API producción
**Estado:** Pendiente de exponer URL de producción
**Plan:** ngrok segundo dominio fijo OR PA Custom Connector con swagger.json
**Activación secuencial:** Receiving + IQC → QC Grading → Box Building → Shipping
**Endpoint pendiente:** Packing List (ShippingLabel controller actualmente vacío)

---

## 📅 Bitácora Diaria

### 📍 2026-05-07 (mañana — pendiente)

**Prioridades planificadas:**
1. **P1 (mañana, antes del mediodía):** Aplicar fix concreto a la limitación de tokens y timeout del LX Agent. Decidir entre: truncar input en frontend, ajustar límites de tokens en API, optimizar system prompt del validador, o reestructurar flow de PA. Validar contra el caso de Adonis antes de promover.
2. **P2 (tarde, después de P1):** Documentar el caso del LX Agent en formato Gotchas + cerrar el ciclo del problema heredado.
3. **P3 (tarde, paralelo a P2):** Continuar pruebas de DINO Bark + DINO Nexus + iniciar diseño de arquitectura de visibilidad cruzada para DINO ISO. Boceto técnico listo para discutir con Joel Fermín antes del 14 de mayo (pre-auditoría interna).

---

### 📍 2026-05-06 — Miércoles

#### Prioridades de hoy (estatus al cierre)

| # | Prioridad heredada de ayer (2026-05-05) | Estatus | Notas |
|---|---|---|---|
| 1 | Investigar y resolver el problema de envío del SLX que afectó a Adonis | 🔄 Parcial | Causa raíz identificada (tokens + timeout PA). Fix pendiente. |
| 2 | Identificar error específico y aplicar fix | ❌ No completada | Diagnóstico hecho, fix concreto no aplicado. Hereda a 2026-05-07. |
| 3 | Documentar en Gotchas o escalar | ❌ No completada | Sin documentación formal. Hereda a 2026-05-07. |

**Razón honesta del avance parcial:** Día redirigido a auditoría R2 ISO + pruebas DINO Bark/Nexus + ajuste de Carrier Lock SQL al final del día.

#### Tareas principales

**Tarea 1 — Reunión ejecutiva R2 ISO con Joel Fermín + pruebas DINO Bark/Nexus**
- Joel reportó 70% de cumplimiento de evidencias tras hallazgos del asesor de Trasenda
- Stage 1: 18-21 mayo (remoto, 4 días)
- Stage 2: 22-30 junio (presencial, 6 días, 2 auditores)
- Puntos pendientes: Recicla, calibración equipos, CCTV 60 días, comité de seguridad
- Pre-auditoría interna planificada antes del 14 de mayo
- Pruebas funcionales sobre DINO Bark (Training) y DINO Nexus (Movement/PMS + Nexus/WMS) para asegurar operatividad ante auditoría

**Tarea 2 — Iteración sobre Carrier Lock SQL (vista + 2 procedures)**
- Objetivo inicial: añadir nuevo status `Resubmit` para IMEIs sometidos sin respuesta del carrier en últimos 6 meses
- Despliegue: ALTER VIEW + 2 ALTER PROCEDURE
- Diagnóstico sobre 3,481 IMEIs → solo 503 visibles
- **Hallazgo clave:** universo dual no contemplado en diseño inicial
  - `VW_FULLDEVICEREPORT` = inventario en casa
  - `SM_Master` = inventario casa + cliente
- Decisión: rollback completo de los 3 objetos
- Validación: regla `Unlock Didn't Work - Resubmit` ya existente cubre correctamente el caso de uso real (resubmit lo dispara feedback del cliente, no regla automática de fechas)

#### Reuniones del día

**Reunión ejecutiva R2 ISO (mañana, conducida por Joel Fermín)**
- Asistentes: ejecutivos (yo incluido) + Joel
- Joel reportó:
  - Plan de auditoría iniciado hace ~1 mes con asesor de Trasenda
  - Evidencias por área completadas: producción, registro, certificaciones, calibración (parcial), manejo de productos, manejo de desechos
  - Hallazgos del asesor llevaron a corrección al 70% de cumplimiento
  - Logística y sistema fueron los puntos más críticos (Haxel + yo dimos soporte)
  - Data sanitization es uno de los puntos más importantes para auditoría (datos personales en equipos)
  - Control de documentos no está 100% optimizado, falta recursos
  - DINO formalmente integrado al sistema de management de documentación
  - 7 procedimientos listos para manejo de desechos
  - Disposition de baterías: pendiente comprar contenedor + área de almacenamiento
  - Downstream due diligence: iPhone (cliente comprador, no procesador) ✅ completo, Recicla ❌ pendiente
  - Stage 1 originalmente programado para ayer, pero asesor cambió por viaje
  - Junior tocó punto crítico: el equipo no se está integrando al 100% con la auditoría — pidió "martillazo" para meterse de lleno

**Notas mías:**
- DINO ISO en uso real sería diferenciador único: "primera auditoría con agente virtual"
- En lugar de tablet con compendio físico, auditor pregunta a DINO y DINO responde con info entrenada
- Validación previa hecha hace una semana con Aime + Irvin sobre estructura de control de documentos

#### Colaboración

| Persona | Interacción |
|---|---|
| **Joel Fermín** | Reunión ejecutiva R2 ISO |
| **Adonis Diaz** | Recibí daily report (score 92% — excelente, diagnóstico de bottleneck en core switch 1 GB) |
| **Massiel Gonzalez** | Recibí daily report (score 73% — 8 tickets resueltos 100% + 2 laptops configuradas) |
| **Jorge Ozuna** | Recibí daily report (score 88% — flow Movement/Spartan-Loki 20 pasos validado, deploy producción exitoso, acceso Prolog para Mioselis) |
| **Wilson Hilario** | Reporte pendiente — terminando asignación |

#### Problemas y dificultades

**Problema 1: Modelo de datos de dos mundos descubierto en producción durante deploy del Carrier Lock**
- **Síntoma:** 3,481 IMEIs esperados vs 503 visibles tras deploy del nuevo status `Resubmit`
- **Causa raíz:** Asunción incorrecta sobre universo de datos. La vista solo cubre inventario en casa (`VW_FULLDEVICEREPORT`); el resto pertenece al inventario del cliente (`SM_Master`).
- **Manejo:** Rollback completo de los 3 objetos (vista + 2 procedures) sin necesidad de escalación.
- **Lección:** Validar el universo de datos antes de modificar lógica de filtros en sistemas con fuentes paralelas.

**Problema 2: Limitación de tokens y timeout de PA en LX Agent (heredado)**
- **Causa raíz:** identificada (tokens/turno + PA timeout)
- **Estatus:** fix concreto no aplicado, hereda a 2026-05-07 como P1

#### Sugerencias generadas hoy

1. **Arquitectura DINO ISO con visibilidad cruzada sobre los demás Dinos** — evitar duplicidad de documentación, una sola fuente de verdad por especialista
2. **Proceso formal de validación de assumptions de datos antes de modificar lógica en sistemas con múltiples fuentes paralelas** — checklist de pre-deploy
3. **Definir SLA de tokens y timeout para LX Agent y otros flujos Claude+PA** — formalizar límites operacionales antes de aplicar fix técnico

#### Decisiones técnicas registradas

- **Carrier Lock — universo de datos dual confirmado:** `VW_FULLDEVICEREPORT` (en casa) ≠ `SM_Master` (casa + cliente). Cualquier reporte que opere sobre ambos universos debe contemplar este split explícitamente. La regla `Unlock Didn't Work - Resubmit` ya existente es suficiente para el flujo real (resubmit lo dispara cliente, no regla automática).
- **Rollback como decisión consciente, no error:** descubrir que la lógica original ya cubría el caso real es resultado válido de iteración. Documentar el aprendizaje vale más que mantener cambio innecesario.

---

## 🧠 Aprendizajes y Patrones Acumulados

### Arquitectura
- **Hub = capa de management/supervisión; Nexus = capa de ejecución operativa para floor workers.** Ambos sobre el mismo SQL Server backend.
- **Direct SQL sobre PA para workflows speed-critical:** producción scan-heavy usa direct `mssql` IPC, no PA flows.
- **Un PA flow por DINO:** isolation requirements de NDA, role-gating, no se pueden manejar con un single flow + switch.
- **`VW_FULLDEVICEREPORT` ≠ `SM_Master`:** dos universos distintos. Inventario en casa vs total (casa + cliente).

### Datos / SQL
- **`varbinary` fields en SPN** requieren explicit `CONVERT(VARCHAR, ...)` wrapping
- **Supervisor field** almacena solo la parte numérica del employee code → requiere self-join via `TRY_CAST`
- **Submit_Date / Unlock_Date / Last_Validation_Date son varchar** en `SM_Master` → siempre usar `TRY_CONVERT(date, ...)` para comparaciones
- **`ROW_NUMBER() OVER (PARTITION BY ... ORDER BY ...)`** ordena por `Last_Validation_Date` primero en `SM_Latest` CTE

### Phonecheck / R2 / ISO
- **Phonecheck genera PDFs de sanitización auto** — no confundir con workflows de evidencia R2/ISO de Joel Fermín
- **Parts traceability** (Receiving/Disassembly → Qualification con Emilio) es separado del workstream R2/ISO de Joel

### Power Automate
- **Shopify webhook delays** siguen exponential backoff (2, 5, 10, 40 min) — root cause es dispatch delay en development/quickstart stores
- **PA `Find files in folder`** retorna array directo (no `{value: []}`) — acceder via `body(...)?[0]?['Id']`
- **SP param via Expression tab:** `triggerBody()?['employeeCode']` (string, no object)
- **Response normalization:** `ResultSets.Table1[0]` → `Table1[0]` → `raw`
- **`Compose_Meta` antes de Condition blocks**
- **Claude JSON parse:** `json(body('HTTP_Claude')?['content']?[0]?['text'])`

### Frontend / Build
- **PPTX debe usar `LAYOUT_WIDE`** (13.333×7.5 in), NO `LAYOUT_16x9` (causa overflow significativo)
- **DOCX border order sigue OOXML schema:** `top, left, bottom, right`
- **LibreOffice renderiza Georgia más ancho que PowerPoint** — apparent overflow en QA renders se resuelve en PowerPoint real
- **CSS variable inheritance falla dentro de iframes** — usar explicit per-element `!important`
- **`node --check /tmp/cs.js` antes de cada ZIP package** — validación de syntax JS
- **Extracción de script block:** `html[html.find('<script>', 35000)+8 : html.rfind('</script>')]`
- **Electron source files:** siempre trabajar desde fuentes reales, no HTML standalone. Extraer de `.exe` via NSIS → 7z → `@electron/asar` cuando no se da source zip.

### Hub specifics (v3.3.7+)
- `contextIsolation: false` → preload usa `window.electronAPI = {}` directamente, NO `contextBridge`
- Panels attach a `.hc` NO `#hub`
- Logout: `app.relaunch() + app.exit(0)` para reset completo
- API key Anthropic hardcoded client-side via base64 obfuscation (línea 668 index.html) → fix recomendado: rutear via PA proxy
- `HUB.claude` es dead code; calls van directo a Anthropic API con system prompt hardcoded

---

## 📋 Backlog / Pending

### Hub
- `npm install` en `airlink-desktop` folder (mssql)
- `SP_Hub_IQC_GetUnit` SP → wire `iqcLoadUnit()` a direct `sql()`
- `SP_Hub_IQC_Save` SP → wire `iqcSubmit()` a direct `sql()`
- PA flows: `PA_UNIT_LOOKUP_URL`, `PA_FG_SUMMARY_URL`, `PA_RECEIVED_URL`
- DSM1/BH/AQC2 modules siguen mismo patrón IQC
- Ejecutar `LX_Menus_Setup.sql` en SPN para activar tabs por empleado

### Carrier Lock + LX Agent
- ⏰ Aplicar fix LX Agent (P1 mañana)
- ⏰ Documentar Gotchas LX Agent (P2 mañana)
- Migrar API key Claude del cliente Electron a PA proxy server-side

### DINO Ecosystem
- ⏰ Continuar pruebas Bark + Nexus (P3 mañana)
- ⏰ Diseño arquitectura DINO ISO con visibilidad cruzada (boceto antes 14 mayo)
- Engineering leadership list — exact employees con acceso DINO IP
- Executive list — confirma `Tab_Executive` users
- Sales leadership list — para `Dino_Money_Sales`
- Bata team list — para `Dino_Bata`
- Quality/Compliance team list — para `Dino_ISO_R2`
- HR team list — para `Dino_HR`
- DinoBot_Data: ¿connect a SharePoint, o llamar SQL via PA dentro del bot? (recomendación: SQL via PA, como tool call)

### Whole Parts
- Sprint 3: catálogo + cart + submit
- Phase 4: Warehouse Node.js print service
- Phase 5: Frontend completo
- Verificar TBDs Fishbowl REST API (PO endpoint, SO POST schema)

### ERP / Movement
- Sub-section ERP en Hub Supply Chain tab (incoming shipments + Fishbowl PO status)
- Movement-API producción URL (ngrok segundo dominio O PA Custom Connector)
- Activación: Receiving + IQC → QC Grading → Box Building → Shipping
- Packing List endpoint (ShippingLabel controller vacío)

### Infraestructura
- Fortinet upgrade próximo mes
- Migración SPN → AirlinkDR (cleaner target)

---

## 📚 Referencias Clave

### Documentos canonical
- `Hub_v337_Source_Reference.md` (908 líneas) — file tree, IPC handlers, HUB endpoints, TAB_MAP, security findings
- `Dino_Ecosystem_Reference.md` (825 líneas) — los 11 DINOs, tiers, system prompts
- `Airlink_AI_Project_State.md` — presentación Assurant, contexto Biju/Brandon
- `Hub_WholeParts_State_2026-05-01.md` — estado Whole Parts
- `DB_Schema_Reference.txt` — referencias canónicas de columnas
- `Hub_SQL_Schema.docx` — schema formal
- `LX_Agent_vFinal.html` — template LX Agent
- `Nexus_Integration_Documentation_v2.docx` — sistema hermano
- `Movement_V1_5.xlsm` — legacy logic (IQC migration source)
- `PA_Flow_Checklists.docx` — patrones de build de flows

### Asset clave
- `IMG_1544.png` — Airlink Strategic Thinking Flywheel (5-color palette)

### Fechas críticas
- **2026-05-14 (jueves):** Pre-auditoría interna R2 ISO
- **2026-05-18 al 21:** Stage 1 R2 ISO (remoto)
- **2026-06-22 al 30:** Stage 2 R2 ISO (presencial)
- **Próximo mes:** Fortinet upgrade

---

## 🔚 Cierre del documento

Este documento se actualiza al cierre de cada día (idealmente como parte del flujo del daily report) y al inicio del siguiente día (cuando llegan los inputs nuevos del día anterior).

**Última entrada:** 2026-05-06
**Próxima entrada esperada:** 2026-05-07 (al iniciar el día con Q1 del daily report)
