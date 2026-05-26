**AIRLINK DISTRIBUTION**

**Unified Daily Report Agent**

**Power Automate — Flow Checklists**

PA-INIT-01  |  PA-MARK-01  |  PA-SAVE-01  |  PA-CLOSE-01

*v1.0  |  2026  |  Internal Use Only*

**Resumen de Flujos**

Los 4 flujos de Power Automate trabajan en conjunto con el agente de Copilot Studio para gestionar el ciclo completo del reporte diario: inicio de sesión, seguimiento de prioridades, cierre y distribución.

| **Flujo** | **Cuándo** | **Qué hace** | **Disparado por** |
| --- | --- | --- | --- |
| **PA-INIT-01** | Al iniciar sesión | Consulta SQL, prioridades, preguntas aleatorias. Inyecta variables al agente. | Copilot Studio — topic Session_Start |
| **PA-MARK-01** | Tras completar Q1 | Marca prioridades de ayer como usadas en BD. | Copilot Studio — topic Q1_Followup |
| **PA-SAVE-01** | Tras completar Q7 | Guarda prioridades de mañana en BD. | Copilot Studio — topic Q7_Priorities |
| **PA-CLOSE-01** | Al finalizar sesión | Llama Claude API, corre Office Script, genera PDF, envía emails. | Copilot Studio — topic Session_End |

*ℹ Todos los flujos son disparados vía HTTP Request desde Copilot Studio (Power Automate Connector). Ninguno tiene trigger de Schedule ni de evento de SharePoint.*

| **PA-INIT-01 — Inicio de Sesión** Trigger: HTTP POST desde Copilot Studio — topic: Session_Start |
| --- |

**Propósito**

Recibe el código del empleado, consulta sus datos en SQL, obtiene las prioridades del día anterior, selecciona 2 preguntas aleatorias del banco y devuelve todo en un JSON que Copilot Studio inyecta como variables en la conversación.

**Input requerido**

| **Variable** | **Tipo** | **Valor / Fuente** |
| --- | --- | --- |
| **EmployeeCode** | String | Único dato que el empleado escribe al iniciar. Ej: 'DR0308' |

**Pasos del flujo**

| **#** | **Acción** | **Detalle** | **Parámetros / Notas** |
| --- | --- | --- | --- |
| **1** | **Trigger HTTP Request** | Configurar como 'When an HTTP request is received'. | Método: POST Body schema: { "EmployeeCode": "string" } Guardar URL generada — se usa en Copilot Studio topic Session_Start. |
| **2** | **Inicializar variables** | Crear variables para almacenar los resultados de cada SP antes de usarlos. | Crear: varEmployee (objeto), varYesterday (objeto), varQ1 (objeto), varQ2 (objeto) |
| **3** | **Ejecutar SP — GetEmployee** | Llamar sp_GetEmployeeAndSupervisor_ByCode con el EmployeeCode recibido. | Connector: SQL Server → Execute stored procedure SP: sp_GetEmployeeAndSupervisor_ByCode Parámetro: @EmployeeCode = triggerBody()?['EmployeeCode'] |
| **4** | **Condición — Empleado existe** | Verificar que el SP devolvió al menos 1 fila. | Condición: length(body('Ejecutar_SP_GetEmployee')?['ResultSets']?['Table1']) greater than 0 SI NO → ir al paso 5 (error) SI SÍ → continuar paso 6 |
| **5** | **Response — Error empleado** | Devolver error si el código no existe en BD. TERMINA el flujo. | Status: 404 Body: { "Error": "EMPLOYEE_NOT_FOUND", "Message": "Código no encontrado. Verifica tu código e intenta de nuevo." } |
| **6** | **Asignar varEmployee** | Extraer campos del primer resultado del SP. | varEmployee = first(body('Ejecutar_SP_GetEmployee')?['ResultSets']?['Table1']) |
| **7** | **Condición — Nivel soportado** | Verificar que ReportLevel = 1 (expandir en versiones futuras). | Condición: variables('varEmployee')?['ReportLevel'] equals 1 SI NO → Response 400: { "Error": "LEVEL_NOT_SUPPORTED" } SI SÍ → continuar |
| **8** | **Ejecutar SP — GetYesterdayPriorities** | Obtener prioridades del día anterior para el empleado. | SP: sp_GetYesterdayPriorities Parámetro: @EmployeeCode = triggerBody()?['EmployeeCode'] |
| **9** | **Asignar varYesterday** | Extraer el único registro devuelto por el SP. | varYesterday = first(body('Ejecutar_SP_Yesterday')?['ResultSets']?['Table1']) |
| **10** | **Ejecutar SP — GetRandomQuestions** | Seleccionar 2 preguntas aleatorias del banco para este empleado. | SP: sp_GetRandomQuestions_ByEmployee @EmployeeCode, @Department, @Position → desde varEmployee @Level = 1 @NumberOfQuestions = 2 |
| **11** | **Asignar varQ1 y varQ2** | Extraer fila 0 y fila 1 del resultado. | varQ1 = body('SP_Questions')?['ResultSets']?['Table1']?[0] varQ2 = body('SP_Questions')?['ResultSets']?['Table1']?[1] |
| **12** | **Condición — Preguntas disponibles** | Verificar que el banco devolvió al menos 2 preguntas. | Condición: length(body('SP_Questions')?['ResultSets']?['Table1']) greater than or equal to 2 SI NO → usar preguntas GENERAL de fallback (definir 2 hardcoded como respaldo) |
| **13** | **Calcular ReportDate** | Obtener la fecha de hoy en formato YYYY-MM-DD. | Expression: formatDateTime(utcNow(), 'yyyy-MM-dd') Guardar en variable varReportDate |
| **14** | **Response — 200 OK** | Devolver JSON completo con todas las variables para Copilot Studio. | Status: 200 Content-Type: application/json Body: ver tabla 'Output JSON' abajo |

**Output JSON — Body del Response (paso 14)**

*ℹ Copilot Studio mapea cada campo de este JSON a una variable de sesión. Los nombres deben coincidir exactamente.*

| **Variable** | **Tipo** | **Valor / Fuente** |
| --- | --- | --- |
| **EmployeeName** | String | varEmployee?['EmployeeName'] |
| **EmployeeCode** | String | triggerBody()?['EmployeeCode'] |
| **DepartmentName** | String | varEmployee?['DepartmentName'] |
| **PositionName** | String | varEmployee?['PositionName'] |
| **SupervisorName** | String | varEmployee?['SupervisorName'] |
| **SupervisorEmail** | String | varEmployee?['SupervisorEmail'] |
| **EmployeeEmail** | String | varEmployee?['EmployeeEmail'] |
| **ReportLevel** | Integer | varEmployee?['ReportLevel'] |
| **ReportDate** | String | varReportDate |
| **YesterdayStatusFlag** | String | varYesterday?['StatusFlag']  → 'FIRST_REPORT' │ 'NO_PRIORITIES' │ 'HAS_PRIORITIES' |
| **YesterdayPriorityID** | Integer | varYesterday?['PriorityID']  → NULL si no hay |
| **YesterdayReportDate** | String | varYesterday?['ReportDate']  → NULL si no hay |
| **YesterdayPriority1** | String | varYesterday?['Priority1']   → NULL si no hay |
| **YesterdayPriority2** | String | varYesterday?['Priority2']   → NULL si no hay |
| **YesterdayPriority3** | String | varYesterday?['Priority3']   → NULL si no hay |
| **RandomQ1_ID** | Integer | varQ1?['QuestionID'] |
| **RandomQ1_ES** | String | varQ1?['Question_ES'] |
| **RandomQ1_EN** | String | varQ1?['Question_EN'] |
| **RandomQ1_Category** | String | varQ1?['Category'] |
| **RandomQ1_GateCriteria** | String | varQ1?['GateCriteria'] |
| **RandomQ1_ScoringCriteria** | String | varQ1?['ScoringCriteria'] |
| **RandomQ2_ID** | Integer | varQ2?['QuestionID'] |
| **RandomQ2_ES** | String | varQ2?['Question_ES'] |
| **RandomQ2_EN** | String | varQ2?['Question_EN'] |
| **RandomQ2_Category** | String | varQ2?['Category'] |
| **RandomQ2_GateCriteria** | String | varQ2?['GateCriteria'] |
| **RandomQ2_ScoringCriteria** | String | varQ2?['ScoringCriteria'] |

**Escenarios de prueba**

|  | **Escenario de prueba** | **Input** | **Resultado esperado** |
| --- | --- | --- | --- |
| **T1** | Código válido con prioridades de ayer | DR0308 | StatusFlag = HAS_PRIORITIES, Priority1/2/3 pobladas, 2 preguntas distintas |
| **T2** | Código válido, primer reporte | DR0826 | StatusFlag = FIRST_REPORT, Priority1/2/3 = null |
| **T3** | Código válido, ayer sin prioridades | DR0234 | StatusFlag = NO_PRIORITIES, PriorityID devuelto para marcar |
| **T4** | Código inválido | DR9999 | Response 404, mensaje de error claro |
| **T5** | Empleado nivel no soportado (L4+) | DR0156 | Response 400, LEVEL_NOT_SUPPORTED |
| **T6** | Departamento sin preguntas específicas | Cualquier L1 | SP devuelve preguntas GENERAL, 2 categorías distintas |

| **PA-MARK-01 — Marcar Prioridades Usadas** Trigger: HTTP POST desde Copilot Studio — topic: Q1_Followup |
| --- |

**Propósito**

Flujo ligero. Se dispara inmediatamente después de que el empleado completa Q1 (seguimiento de ayer). Marca las prioridades del día anterior como IsUsed = 1 en la tabla DailyReport_YesterdayPriorities para cerrar el ciclo de seguimiento.

*ℹ Si YesterdayStatusFlag = **'**FIRST_REPORT**'** o **'**NO_PRIORITIES**'**, Copilot Studio NO dispara este flujo — no hay nada que marcar.*

**Input requerido**

| **Variable** | **Tipo** | **Valor / Fuente** |
| --- | --- | --- |
| **EmployeeCode** | String | Código del empleado. Disponible en el contexto de la sesión de Copilot Studio. |
| **PriorityID** | Integer | ID del registro de prioridades a marcar. Proviene de YesterdayPriorityID inyectado en PA-INIT-01. |

**Pasos del flujo**

| **#** | **Acción** | **Detalle** | **Parámetros / Notas** |
| --- | --- | --- | --- |
| **1** | **Trigger HTTP Request** | When an HTTP request is received. | Método: POST Body schema: { "EmployeeCode": "string", "PriorityID": 0 } |
| **2** | **Validar input** | Verificar que PriorityID no sea null ni 0. | Condición: empty(triggerBody()?['PriorityID']) equals false AND triggerBody()?['PriorityID'] greater than 0 SI NO → Response 400: { "Error": "INVALID_PRIORITY_ID" } |
| **3** | **Ejecutar SP — MarkPrioritiesAsUsed** | Llamar el SP para marcar el registro. | SP: sp_MarkPrioritiesAsUsed @EmployeeCode = triggerBody()?['EmployeeCode'] @PriorityID   = triggerBody()?['PriorityID'] |
| **4** | **Verificar resultado** | Confirmar que el SP actualizó al menos 1 fila. | Valor a revisar: body('SP_Mark')?['ResultSets']?['Table1']?[0]?['RowsUpdated'] SI = 0 → loguear warning (no bloquear el flujo) |
| **5** | **Response — 200 OK** | Confirmar éxito a Copilot Studio. | Status: 200 Body: { "Result": "OK", "RowsUpdated": [valor del SP] } |

**Escenarios de prueba**

|  | **Escenario de prueba** | **Input** | **Resultado esperado** |
| --- | --- | --- | --- |
| **T1** | PriorityID válido existente | { EmployeeCode: DR0308, PriorityID: 42 } | RowsUpdated = 1, Response 200 OK |
| **T2** | PriorityID que no existe en BD | { EmployeeCode: DR0308, PriorityID: 999 } | RowsUpdated = 0, Response 200 con warning logueado |
| **T3** | PriorityID = null o 0 | { EmployeeCode: DR0308, PriorityID: null } | Response 400, INVALID_PRIORITY_ID |
| **T4** | Doble llamada mismo PriorityID | Llamar 2 veces con ID = 42 | Segunda llamada: RowsUpdated = 0 (ya marcado), sin error |

| **PA-SAVE-01 — Guardar Prioridades de Mañana** Trigger: HTTP POST desde Copilot Studio — topic: Q7_Priorities |
| --- |

**Propósito**

Guarda las prioridades que el empleado estableció para el día siguiente. Se dispara SIEMPRE al terminar Q7 — tanto si el empleado dio prioridades como si no (HasPriorities = 0). Expira automáticamente cualquier prioridad anterior no usada del mismo empleado.

*ℹ Este flujo siempre responde 200 OK. Nunca debe bloquear el avance de la conversación.*

**Input requerido**

| **Variable** | **Tipo** | **Valor / Fuente** |
| --- | --- | --- |
| **EmployeeCode** | String | Código del empleado. |
| **ReportDate** | String | Fecha del reporte actual (YYYY-MM-DD). Viene de varReportDate inyectado en PA-INIT-01. |
| **Priority1** | String | Texto de prioridad 1. NULL si no aplica. |
| **Priority2** | String | Texto de prioridad 2. NULL si no aplica. |
| **Priority3** | String | Texto de prioridad 3. NULL si no aplica. |
| **Level** | Integer | Nivel del reporte (1 para L1). Viene de ReportLevel de la sesión. |
| **HasPriorities** | Integer | 1 si el empleado dio prioridades. 0 si respondió que no tiene. |

**Pasos del flujo**

| **#** | **Acción** | **Detalle** | **Parámetros / Notas** |
| --- | --- | --- | --- |
| **1** | **Trigger HTTP Request** | When an HTTP request is received. | Método: POST Body schema: { EmployeeCode, ReportDate, Priority1, Priority2, Priority3, Level, HasPriorities } |
| **2** | **Validar input mínimo** | Verificar que EmployeeCode y ReportDate estén presentes. | Condición: not(empty(EmployeeCode)) AND not(empty(ReportDate)) SI NO → Response 400: { "Error": "MISSING_REQUIRED_FIELDS" } |
| **3** | **Ejecutar SP — SaveTomorrowPriorities** | Llamar el SP que expira registros anteriores e inserta el nuevo. | SP: sp_SaveTomorrowPriorities @EmployeeCode  = triggerBody()?['EmployeeCode'] @ReportDate    = triggerBody()?['ReportDate'] @Priority1     = triggerBody()?['Priority1'] @Priority2     = triggerBody()?['Priority2'] @Priority3     = triggerBody()?['Priority3'] @Level         = triggerBody()?['Level'] @HasPriorities = triggerBody()?['HasPriorities'] |
| **4** | **Extraer NewPriorityID** | Guardar el ID del nuevo registro creado. | varNewPriorityID = body('SP_Save')?['ResultSets']?['Table1']?[0]?['NewPriorityID'] |
| **5** | **Response — 200 OK** | Confirmar éxito siempre. | Status: 200 Body: { "Result": "OK", "NewPriorityID": varNewPriorityID, "HasPriorities": triggerBody()?['HasPriorities'] } |

**Escenarios de prueba**

|  | **Escenario de prueba** | **Input** | **Resultado esperado** |
| --- | --- | --- | --- |
| **T1** | Empleado da 3 prioridades | HasPriorities=1, Priority1/2/3 con texto | NewPriorityID > 0, HasPriorities = 1 |
| **T2** | Empleado da 1 prioridad | HasPriorities=1, Priority1 con texto, 2/3 null | NewPriorityID > 0, Priority2/3 = null en BD |
| **T3** | Empleado no da prioridades | HasPriorities=0, Priority1/2/3 = null | NewPriorityID > 0, HasPriorities = 0 en BD |
| **T4** | Segundo reporte mismo día | Mismo EmployeeCode, misma ReportDate | Registro anterior expirado, nuevo insertado |
| **T5** | ReportDate o EmployeeCode vacíos | Campos requeridos en null | Response 400, MISSING_REQUIRED_FIELDS |

| **PA-CLOSE-01 — Cierre y Reporte** Trigger: HTTP POST desde Copilot Studio — topic: Session_End |
| --- |

**Propósito**

Es el flujo más complejo. Recibe todas las respuestas y flags de la sesión, llama a Claude API para calificar el reporte, ejecuta el Office Script para poblar el Excel, convierte la hoja del empleado a PDF vía Graph API, y envía los emails al supervisor y al empleado.

**Input requerido**

*ℹ Copilot Studio envía un JSON con TODOS los campos acumulados durante la conversación. Ver sección Output del agente en el Copilot Studio Prompt.*

| **Variable** | **Tipo** | **Valor / Fuente** |
| --- | --- | --- |
| **EmployeeName / Code / Dept / Position / Supervisor / Date / Level** | Strings | Datos del empleado desde sesión |
| **EmployeeEmail / SupervisorEmail** | Strings | Emails para envío. Vienen de PA-INIT-01 via varEmployee. |
| **Answer_Q1 … Answer_Q5** | Strings | Respuestas de preguntas fijas |
| **Answer_QR1 / QR2** | Strings | Respuestas de preguntas aleatorias |
| **QuestionText_QR1 / QR2** | Strings | Texto de la pregunta aleatoria en español |
| **Category_QR1 / QR2** | Strings | Categoría de cada pregunta aleatoria |
| **ScoringCriteria_QR1 / QR2** | Strings | Criterios de scoring por categoría |
| **TomPriority1 / 2 / 3** | Strings | Prioridades de mañana (o null) |
| **YesterdayPriority1 / 2 / 3** | Strings | Prioridades de ayer (contexto para Claude) |
| **Flag_NoPreviousPriorities** | Bool/String | 'true' o 'false' |
| **Flag_NoTomorrowPriorities** | Bool/String | 'true' o 'false' |
| **Flag_HonestyLow** | Bool/String | 'true' o 'false' |
| **Flag_MaxRetryQ2 … QR2** | Bool/String | 'true' o 'false' por pregunta |

**Pasos del flujo**

| **#** | **Acción** | **Detalle** | **Parámetros / Notas** |
| --- | --- | --- | --- |
| **1** | **Trigger HTTP Request** | When an HTTP request is received. | Método: POST Body: JSON completo de la sesión No definir schema fijo — usar dynamic content |
| **2** | **Inicializar variables** | Crear variables para resultados intermedios. | varClaudeResponse (string) varGradingJSON (object) varExcelPath (string) varPDFPath (string) |
| **3** | **Construir prompt Claude** | Armar el body para la API de Claude usando el Grading Prompt y los valores del trigger. | Usar una acción 'Compose' para construir el JSON con el grading prompt. Reemplazar todos los {{placeholders}} con los valores del triggerBody(). Model: claude-sonnet-4-6 max_tokens: 4000 |
| **4** | **HTTP POST — Claude API** | Llamar a la API de Claude para calificar el reporte. | URL: https://api.anthropic.com/v1/messages Headers:   x-api-key: [variable de entorno en PA]   anthropic-version: 2023-06-01   Content-Type: application/json Body: output del paso 3 |
| **5** | **Verificar respuesta Claude** | Confirmar que la API respondió con status 200 y content. | Condición: outputs('HTTP_Claude')?['statusCode'] equals 200 SI NO → loguear error + enviar email de alerta a admin + terminar |
| **6** | **Extraer JSON de Claude** | Parsear el JSON del content[0].text de la respuesta. | Expression: json(body('HTTP_Claude')?['content']?[0]?['text']) Guardar en varGradingJSON |
| **7** | **Construir body Office Script** | Serializar varGradingJSON a string para pasarlo al script. | Expression: string(varGradingJSON) Guardar en varScriptInput |
| **8** | **Obtener plantilla Excel de SharePoint** | Leer el archivo template desde SharePoint para el script. | Connector: SharePoint → Get file content Ruta: /Reportes/Templates/Daily_Report_Template.xlsx |
| **9** | **Ejecutar Office Script** | Poblar las 3 hojas del Excel con los datos del reporte. | Connector: Excel Online (Business) → Run script File: Daily_Report_Template.xlsx (from SharePoint) Script: Daily_Report_OfficeScript Parameter jsonData: varScriptInput |
| **10** | **Guardar Excel con nombre dinámico** | Copiar el archivo poblado a la carpeta del mes correspondiente. | Nombre: @{triggerBody()?['EmployeeCode']}_@{triggerBody()?['ReportDate']}.xlsx Ruta destino: /Reportes/@{formatDateTime(utcNow(),'yyyy')}/@{formatDateTime(utcNow(),'MM')}/ Conector: SharePoint → Create file |
| **11** | **Convertir hoja empleado a PDF** | Usar Graph API para exportar solo la hoja 'Employee Report ES' como PDF. | HTTP GET: https://graph.microsoft.com/v1.0/sites/{siteId}/drives/{driveId}/items/{fileId}/workbook/worksheets/Employee%20Report%20ES/range/usedRange Alternativa más simple: Convert file via Graph: POST /drives/{id}/items/{id}/content?format=pdf |
| **12** | **Guardar PDF en SharePoint** | Almacenar el PDF del empleado junto al Excel. | Nombre: @{EmployeeCode}_@{ReportDate}_Employee.pdf Misma carpeta del paso 10 |
| **13** | **Construir email supervisor** | Reemplazar todos los placeholders del template Email_Supervisor.html. | Usar múltiples acciones 'Replace' o una Compose con concat(). Clases dinámicas: ScoreBannerClass, Alert_X_Class, Alert_X_Icon Splitear ActionItems_EN en 5 items por '; ' |
| **14** | **Enviar email supervisor** | Enviar email con Excel adjunto. | Connector: Office 365 Outlook → Send an email (V2) To: SupervisorEmail Subject: if(FailedReport, '⚠ REPORT FAILED — {Name} │ {Date}', 'Daily Report — {Name} │ {Score}% │ {Date}') Body: HTML del paso 13 Attachment: Excel del paso 10 |
| **15** | **Construir email empleado** | Reemplazar placeholders del template Email_Employee.html. | Mismo patrón que paso 13 pero con variables _ES Clases dinámicas: ConfirmBannerClass, ConfirmIcon, ConfirmMessage Conditional blocks para notices de prioridades |
| **16** | **Enviar email empleado** | Enviar email con PDF adjunto. | Connector: Office 365 Outlook → Send an email (V2) To: EmployeeEmail Subject: según Score_Status (ver guía en Email_Employee.html) Body: HTML del paso 15 Attachment: PDF del paso 12 |
| **17** | **Response — 200 OK a Copilot Studio** | Confirmar que el flujo completó exitosamente. | Status: 200 Body: { "Result": "OK", "Score_Final": varGradingJSON?['Score_Final'], "Score_Status": varGradingJSON?['Score_Status'], "ExcelPath": varExcelPath } |
| **18** | **Manejo de errores (scope)** | Agregar un scope de error global con 'Configure run after: has failed'. | En el scope de error: 1. Loguear el error en una tabla SQL o lista SharePoint 2. Enviar email de alerta al admin con el error 3. Notificar a Copilot Studio con Response 500 |

**Notas de implementación**

- Paso 4 — API Key de Claude: NO hardcodear en el flujo. Guardar en un Azure Key Vault o como variable de entorno en Power Automate (Environment Variables).

- Paso 9 — Office Script: El script debe estar publicado en el mismo tenant de Excel Online donde vive el template. No es posible usar scripts de otro tenant.

- Paso 11 — PDF via Graph API: Requiere permiso Files.ReadWrite en el app registration de Power Automate. Alternativa más simple: usar el conector de Excel Online para descargar como PDF si está disponible en el tenant.

- Paso 13/15 — Replace de placeholders: Dado que son muchos (55 en supervisor, 31 en empleado), se recomienda una Compose con una función concat() anidada o usar el patrón: replace(replace(replace(template, '{{var1}}', val1), '{{var2}}', val2), ...)

- Timeout: Claude API puede tardar 10-30 segundos. Configurar el timeout del HTTP action en 60 segundos mínimo.

**Escenarios de prueba**

|  | **Escenario de prueba** | **Input** | **Resultado esperado** |
| --- | --- | --- | --- |
| **T1** | Sesión completa — reporte aprobado | Score_Final ≥ 70, todos los flags = false | Excel 3 hojas, PDF, email supervisor normal, email empleado verde |
| **T2** | Reporte fallido Score < 70 | Respuestas pobres, Score_Final < 70 | Email supervisor con asunto ⚠ FAILED, banner rojo empleado |
| **T3** | Flag_NoTomorrowPriorities = true | Empleado no dio prioridades en Q7 | Alerta activa (roja) en Excel supervisor, notice en email empleado |
| **T4** | Flag_NoPreviousPriorities = true | Primer reporte del sistema | Alerta info en email empleado, sin Q1 en Excel |
| **T5** | Flag_LowHonesty = true | Q4 negado / sin problemas reportados | Alerta baja honestidad en Excel supervisor, S3 score ≤ 6 |
| **T6** | Claude API falla (503) | API no responde o error 5xx | Email de alerta a admin, Response 500 a Copilot Studio, empleado recibe mensaje de error |
| **T7** | Excel con nombre duplicado | Mismo EmployeeCode + misma fecha | SharePoint sobrescribe o crea con sufijo _v2 (según config del tenant) |

*Airlink Distribution  |  Internal Use Only  |  v1.0 — 2026*