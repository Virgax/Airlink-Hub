**NEXUS Integration Documentation v2.0**  ·  Airlink Distribution DR  ·  Confidential

**NEXUS**
Integration Architecture & Development Guide
Airlink Hub  ×  Movement 1.5  ×  Movement-API  ×  Movement-Fend
 

| **Field** | **Value** |
| --- | --- |
| Document Version | v2.0 — Nexus.7z Source |
| Status | Draft — For Development Team |
| Prepared by | Jaime Joan Estevez M. — Director, Software & IT |
| Organization | Airlink Distribution DR |
| Target Project | Nexus — Unified Operations Platform |
| Date | April 20, 2026 |

# **1. Executive Summary**

Nexus unifies four systems — Airlink Hub (Electron desktop), Movement 1.5 (Excel VBA), Movement-API (.NET 8 REST backend), and Movement-Fend (Next.js 14 web frontend) — into a single integrated platform backed by a shared SQL Server data layer.

*Mission: Replace fragmented Excel-based workflows with a real-time, role-aware operations platform. All modules share a single source of truth across desktop (Hub) and web (Movement-Fend).*

## **1.1 Systems Being Integrated**

| **System** | **Type** | **Current State** | **Role in Nexus** |
| --- | --- | --- | --- |
| Airlink Hub | Electron Desktop | v3.3.4 — Active Production | Desktop shell, AI layer, nav/auth host |
| Movement 1.5 | Excel VBA (.xlsm) | Active Production | Source of truth for ~15 operational processes |
| Movement-API | .NET 8 REST API | Active Development | Backend for all operations — 25 controllers |
| Movement-Fend | Next.js 14 Web App | Active Development | Browser frontend — 63 pages/routes |
| SQL Server | DB 192.168.181.248:13999 | Active Production | 4 DBs: AIRLINK, AirlinkDR, SPN, AirlinkUSA |
| Power Automate | PA Flows (HTTP) | Active | Auth, AI flows, logistics sync, menus |
| Claude API | anthropic claude-sonnet-4-6 | Active | LX Agent, Claude Chat, Dino |

# **2. Database Architecture**

Movement-API connects to 4 SQL Server databases. All production DBs run on 192.168.181.248:13999 (ALAppUser). A dev environment runs on 192.168.181.246:8989 (ARLQA).

| **DB Name** | **EF Context** | **Server** | **Role** |
| --- | --- | --- | --- |
| AIRLINK | AirlinkContext | 192.168.181.248:13999 | Primary operations — SPC_*, PRD_*, G_* tables. Movement-API main DB. |
| AirlinkDR | AirlinkDrContext | 192.168.181.248:13999 | Original Hub DB — Box, Receiving, Unit, etc. (legacy, still active). |
| SPN | SpnContext | 192.168.181.248:13999 | HR & Permissions — Empleados, LX_Menus, auth. |
| AirlinkUSA | AirlinkUsaContext | AFS2SQL1 (separate) | US Operations — Receiving, Unit for US warehouse. |

## **2.1 AIRLINK DB — Complete Table Inventory**

| **Table** | **Group** | **Key Columns / Purpose** |
| --- | --- | --- |
| SPC_Label | Labels | ID, Color, Size — label templates |
| SPC_ProjectsCode | Projects | Code (PK), Nick, GCColor, Flowtype, SM, Mix, Battery, Saleable, AuthBy, CreatedBy, UpdatedAt |
| SPC_ProjectsLogos | Projects | Nick, LogoIMG, LogoSize1/2 — project logo images |
| SPC_FlowType | Catalog | ID, Description — flow type reference |
| SPC_DeviceType | Catalog | ID, Nick, Device — device type reference |
| SPC_Receiving | Receiving | ID, Tracking, GCN (unique), BoxNumber, Invoice, LotNumber, Project, IMEI, FBIMEI, Model, DeviceType, ReceivedBy, UnlockCode, BModel |
| SPC_Unit | Units | ID, Serial (unique), IMEI (unique), Model, Brand, Color, Capacity, Carrier, Battery, DeviceType, GCN, Project, Grade, Condition, IsFMi, IsGGLock, IsKNOX, IsMDM, RAM, Processor + more |
| SPC_PPBoxLink | Labels | ID, GCN, QR1, Createdby, Date, Time |
| SPC_Box / SpcBoxLog | Supply Chain | Box management and movement audit log |
| SPC_BoxShipping | Supply Chain | Box shipping records |
| SPC_PalletShipping | Supply Chain | Pallet shipping records |
| SPC_Shipped | Supply Chain | Shipped units tracking |
| SPC_Inventory / Log | Warehouse | Kanban inventory + movement log |
| SPC_PartInventory / Historical | Warehouse | Parts inventory with full history |
| SPC_PartNumber | Warehouse | Part number catalog |
| SPC_PartsToReceive | Warehouse | Parts pending receipt |
| SPC_Po | Supply Chain | Purchase Orders |
| SPC_ShopifyOrder | E-Commerce | Shopify orders integration |
| SPC_TranferLocation | Warehouse | Location transfers |
| SPC_RackArea | Warehouse | Rack/area definitions (Level, Name, Position) |
| SPC_BhConfiguration | Print City | Back Housing configuration |
| SPC_AuxCondition | Catalog | Auxiliary conditions reference |
| SPC_BatterySpecial | Catalog | Special battery specifications |
| SPC_LabelAid | Labels | Label helper/reference data |
| PRD_Asn | Production | ASN records for production |
| PRD_CometicGrading / History | Production | Cosmetic grading records + history |
| PRD_GradeMatrix | Production | Grade calculation matrix |
| GCondition | Global Cat. | Global conditions catalog |
| GGrade | Global Cat. | Global grades catalog |
| GModel | Global Cat. | Global models catalog |
| GDeviceModelNumber | Global Cat. | Device model numbers |
| GStep | Global Cat. | Process steps catalog |
| GMovementErrorLog / ErrorLog | System | Error logging tables |
| ScpBoxType | Catalog | Box types reference |

## **2.2 AirlinkDR DB — Legacy Hub Tables**

*AirlinkDR is the original Hub database. Many tables have been replicated to AIRLINK as the migration progresses. Both remain active.*

| **Table** | **Purpose** |
| --- | --- |
| Tracking_Master | Logistics shipment tracking — SC Dashboard, MovementStatus column |
| PRD_UPH | Production metrics — Back Housing TV dashboard (Date, Time, Project, Process, Model, QTY, Line) |
| Hub_ChatSessions | AI chat sessions (ChatID, EmployeeCode, Agent, Title, CreatedAt, IsDeleted) |
| Hub_ChatMessages | AI messages (MsgID, ChatID, Seq, Role, Content, CreatedAt) |
| Receiving / Box / BoxHistory | Legacy receiving and box management (also in AIRLINK) |
| BoxShipping / PalletShipping / Shipped | Legacy shipping tables (also in AIRLINK) |
| Inventory / InventoryLog | Legacy warehouse inventory (also in AIRLINK) |
| PartInventory / Historical | Legacy parts inventory (also in AIRLINK) |
| Po / PpboxLink / ProjectsCode | Legacy PO, PPBox, projects (also in AIRLINK) |
| CometicGrading / History | Legacy grading records (also in AIRLINK as PRD_) |
| Unit / SpcBoxLog | Legacy unit and box log records (also in AIRLINK) |

# **3. Movement-API — .NET 8 Backend**

Movement-API is a .NET 8 / ASP.NET Core REST API using Clean Architecture (Domain / Application / Infrastructure). It is the backend consumed by Movement-Fend (browser) and available to Hub for specific operations requiring business logic.

Source: Nexus.7z / Movement-Api branch  |  Current URL: https://localhost:7256 (dev). Production URL: TBD.

Universal response shape: { StatusCode, Message, Data, IsSuccess }

## **3.1 Complete Endpoint Reference (25 Controllers)**

**Login**  

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| POST | /api/login | Auth: {CodeUser, Password} → {NameEmployee, CodeEmployee} |

**Project**  /api/Project/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/Project/names | List all project names |
| POST | /api/Project/project | Create new project (ProjectDTO body) |
| GET | /api/Project/info-project-nick/{nick} | Project info by nick: {Sm, Saleable, Mix} |
| GET | /api/Project/info-project-code/{proyectCode} | Project info by code |
| GET | /api/Project/codes-projects | List all project codes |

**Receiving**  /api/Receiving/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| POST | /api/Receiving/add-data-receiving | Bulk insert receiving records (List<CaptureDataReceiving>) |
| GET | /api/Receiving/gcn-validate/{gcn} | Validate GCN exists in SPC_Receiving |
| GET | /api/Receiving/validate-imei/{imei} | Validate IMEI exists in SPC_Unit |
| GET | /api/Receiving/gift-card/{gnc} | Get PPBox info by GCN → PPBoxConsultDTO |
| GET | /api/Receiving/imei-by-gcn/{gcn} | Get IMEI by GCN from SPC_Receiving |

**PPBoxLink**  /api/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/data-phone/{gcn} | Phone data: {IMEI, Model, Color} from SPC_Unit |
| POST | /api/add-pp-box-link | Save PPBox link: {GCN, QR1, CreatedBy} |
| GET | /api/exist-qr/{qr} | Check if QR exists in SPC_PPBoxLink |

**ASN**  /api/ASN/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/ASN/template-asn | Download ASN Excel template |
| POST | /api/ASN/upload-asn/{createBy} | Upload ASN from Excel file (IFormFile) |
| POST | /api/ASN/save-asn/{createBy} | Save ASN record by quantity (ASN_dto body) |

**IQC**  /api/IQC/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/IQC/info-device/{gcn} | Get device info for IQC by GCN |

**QCGrading**  /api/QCGrading/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| POST | /api/QCGrading/cal-grade-device | Calculate cosmetic grade (GradeDevice body) |
| GET | /api/QCGrading/get-info/{gcn} | Get existing QC grading info by GCN |
| POST | /api/QCGrading/save | Save cosmetic grading result (CometicGradingRequest) |

**QuickFG**  /api/QuickFG/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/QuickFG/info-device/{gcn}/{createBy} | Quick FG device info by GCN + creator |

**BoxBuilding**  /api/BoxBuilding/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/BoxBuilding/data-initial-box-building | Initial data for Box Building module |
| GET | /api/BoxBuilding/consult-bin/{numberBin} | Get counter/content of bin |
| POST | /api/BoxBuilding/save-box-building | Save list of box building entries |
| GET | /api/BoxBuilding/validate-box-by-gcn/{gnc} | Validate box by GCN |

**FGBoxBuilding**  /api/FGBoxBuilding/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/FGBoxBuilding/data-phone/{gcn} | Phone data by GCN for FG box building |
| GET | /api/FGBoxBuilding/count-item-box/{box} | Count items in FG box |
| POST | /api/FGBoxBuilding/save | Save FG box building record |

**BoxShipping**  /api/BoxShipping/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/BoxShipping/options | Select options for Box Shipping |
| GET | /api/BoxShipping/new-box-number | Generate new box number |
| POST | /api/BoxShipping/add-box-shipping/{boxNumber} | Save units to a box shipping |
| GET | /api/BoxShipping/box-by-numberBox-for-transfer/{numberBox} | Box contents for transfer |
| GET | /api/BoxShipping/validate-unit-by/{gcn}/{imei} | Validate unit by GCN + IMEI before save |
| GET | /api/BoxShipping/box-by-numberBox/{numberBox} | List units by box number |
| GET | /api/BoxShipping/box-shipping-by-numberBox/{numberBox} | Box shipping records by number |
| POST | /api/BoxShipping/devices-by-location | Get devices by location (PoInfoDTO) |

**GCPrinting**  /api/GCPrinting/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/GCPrinting/gcprinting-data-initial | Initial data: {Codes[], Devices[]} |
| GET | /api/GCPrinting/print-labels/{projectCode}/{deviceType}/{amountLabel}/{labelSize}/{isRMA}/{isFB} | Generate ZPL labels for Zebra printer |

**Battery**  /api/Battery/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/Battery/info-label/{labelType} | Battery label info by LabelType enum |

**Inventory**  /api/Inventory/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/Inventory/info-kanban | Kanban inventory overview |
| POST | /api/Inventory/validate-kanban | Validate kanban entry (Kanban body) |
| GET | /api/Inventory/validate/{box} | Validate box exists in inventory |
| POST | /api/Inventory/save-box-kanban | Save box to kanban (BoxInKanbanReq) |
| POST | /api/Inventory/save-part | Save part inventory record (SavePartInventoryReq) |

**PO (Purchase Orders)**  /api/PO/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/PO/po-search/{poNumber} | Search POs by number |
| POST | /api/PO/po-add | Add new PO (List<PODTO>) |
| GET | /api/PO/select-options | Dropdown options for PO form |
| PUT | /api/PO/po-update | Update existing PO (PODTO body) |
| GET | /api/PO/model-number-search/{modelNumber} | Get device by model number |
| GET | /api/PO/template-po | Download PO Excel template |
| POST | /api/PO/upload-excel-po/{createBy} | Upload PO from Excel (IFormFile) |

**PalletShipping**  /api/PalletShipping/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/PalletShipping/pallet-no-and-box | Available pallet numbers and boxes |
| POST | /api/PalletShipping/{palletNo}/save | Save pallet shipping record |
| GET | /api/PalletShipping/Box/{palletNo} | Get boxes by pallet number |

**Shipped**  /api/Shipped/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/Shipped/pallet-no | Pallet numbers for shipped |
| GET | /api/Shipped/box-number | Box numbers for shipped |
| POST | /api/Shipped/save | Save shipped record (ShippedRequest) |
| GET | /api/Shipped/exist-tracking/{tracking} | Check if tracking number exists |

**ShopifyOrder**  /api/ShopifyOrder/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/ShopifyOrder/order-numbers | Open Shopify order numbers |
| GET | /api/ShopifyOrder/info-order/{partNumber} | Order info by part number |
| POST | /api/ShopifyOrder/validate-material | Validate material for order (MaterialReq) |
| POST | /api/ShopifyOrder/save-order | Save Shopify order (ShopifyOrderReq) |

**TransferBinABin**  /api/TransferBinABin/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/TransferBinABin/{boxNumber}/gift-cards | Get GCNs in a box/bin |
| PUT | /api/TransferBinABin/{fromBoxNumber}/{toBoxNumber} | Transfer GCNs between bins |

**TransferBox**  /api/TransferBox/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/TransferBox/info-device/{box} | Device info in box |
| GET | /api/TransferBox/destination | Valid transfer destinations |
| POST | /api/TransferBox/transfer-without-delete | Transfer box keeping source |
| POST | /api/TransferBox/transfer-and-delete | Transfer box and delete from source |

**TransferUnits**  /api/TransferUnits/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/TransferUnits/info-device/{gcn} | Unit info by GCN |
| POST | /api/TransferUnits/transfer-and-delete | Transfer unit and remove from source |

**UpdateCondition**  /api/UpdateCondition/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/UpdateCondition/info-device/{gcn} | Current device condition by GCN |
| PUT | /api/UpdateCondition/device/{gcn} | Update device condition |

**Reporteria**  /api/Reporteria/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| GET | /api/Reporteria/reporte/{number_reporte} | Get report by report number |

**PrintCity**  /api/PrintCity/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| POST | /api/PrintCity/battery-label | Print battery label (PrintBatteryLabel) |
| GET | /api/PrintCity/{qty}/bin-inventory | Print bin inventory labels |
| GET | /api/PrintCity/info-inventory-label/page/{page}/{count} | Inventory labels paginated |
| POST | /api/PrintCity/info-inventory-label/page/{page}/{count}/pagination | Filter inventory labels |
| POST | /api/PrintCity/inventory-label | Print inventory label (PrintInventoryLabel) |
| GET | /api/PrintCity/info-mro-label/page/{page}/{count} | MRO labels paginated |
| POST | /api/PrintCity/mro-label | Print MRO label (PrintMROLabel) |
| GET | /api/PrintCity/info-part-gc | Part GC info for printing |
| GET | /api/PrintCity/part-gc/{project}/{partType}/{qty} | Print part GC labels |
| POST | /api/PrintCity/mri-and-mrb-print | Print MRI and MRB labels |
| GET | /api/PrintCity/mri-and-mrb/{page}/{count} | MRI/MRB data paginated |
| POST | /api/PrintCity/mri-and-mrb/page/{page}/{count}/pagination | Filter MRI/MRB data |
| GET | /api/PrintCity/bh-model-condition | Back Housing model conditions |
| POST | /api/PrintCity/bh-model-condition-print | Print BH model condition label |
| GET | /api/PrintCity/bh-fg | BH Finished Goods data |
| POST | /api/PrintCity/bh-fg-print | Print BH FG label |

**ShippingLabel**  /api/ShippingLabel/

| **Method** | **Endpoint** | **Description** |
| --- | --- | --- |
| *—* | *Controller registered — no endpoints implemented yet* |  |

# **4. Movement-Fend — Next.js 14 Frontend**

Movement-Fend is the browser-accessible web frontend for all operational modules. It communicates exclusively with Movement-API. 63 pages/routes implemented across 5 functional areas.

Stack: Next.js 14.1.4 · React 18 · NextUI · ApexCharts · Axios · react-data-table · xlsx · SweetAlert2

Auth: sessionStorage.getItem("dataUser") — session-based flag. No JWT yet.

Git branch: LC-Movement-frontend / MS-Movement-frontend (merged)

## **4.1 Supply Chain Pages (22 routes)**

| **Route** | **Module Name** | **Notes** |
| --- | --- | --- |
| /supply_chain/receiving | Receiving | New shipment receiving |
| /supply_chain/pp_box_link | PP BOX LINK | Link PP box by QR/GCN |
| /supply_chain/pp_box_consult | PP Box Consult | Consult PP box contents |
| /supply_chain/shopify_order | Shopify Order | E-commerce order processing |
| /supply_chain/gc_printing | GC Printing | Gift card label printing (Zebra ZPL) |
| /supply_chain/fg_box_building | FG Box Building | Build Finished Goods boxes |
| /supply_chain/box_building | Box Building | Build bins/boxes from GCNs |
| /supply_chain/box_shipping | Box Shipping | Ship boxes out |
| /supply_chain/pallet_shipping | Pallet Shipping | Build and ship pallets |
| /supply_chain/shipped | Shipped | Confirm shipped records |
| /supply_chain/shipping_label | Shipping Label | Generate shipping labels (in development) |
| /supply_chain/box | Transfer Box | Transfer box to destination |
| /supply_chain/units | Transfer Units | Transfer individual units |
| /supply_chain/bin-a-bin | Transfer Bin A Bin | Move GCNs between bins |
| /supply_chain/transfer | Transfer Bin A Bin | Alternate transfer route |
| /supply_chain/update_condition | Update Condition | Update device condition by GCN |
| /supply_chain/devices | Devices Inventory | Device inventory view |
| /supply_chain/part | Part Inventory | Parts inventory management |
| /supply_chain/po_creator | PO Creator | Create purchase orders |
| /supply_chain/create_project | Create Project | New project/lot creation |
| /supply_chain/asn/qty | ASN QTY | ASN by quantity |
| /supply_chain/asn/upload_list | ASN Upload | Upload ASN from file |

## **4.2 Production — Phone Process Pages (18 routes)**

| **Route** | **Module Name** | **Notes** |
| --- | --- | --- |
| /iqc | IQC | Incoming Quality Control — 24 tests + cosmetic grade |
| /dsm1 | DSM1 | Disassembly Stage 1 |
| /bh_replacement | BH Replacement | Back Housing replacement |
| /bh_removal | BH Removal | Back Housing removal |
| /bh_assembly | BH Assembly | Back Housing assembly |
| /closing | Closing | Final closing step |
| /ic | IC | Internal Check |
| /aqc | AQC | Appearance Quality Control |
| /aqc2 | AQC 2 | Second AQC step |
| /itriage | ITriage | Internal triage |
| /rewok | Rework | Device rework |
| /l3_triage | L3 Triage | Level 3 triage |
| /overlay | OverLay | Overlay process step |
| /jigs | Jigs | Jigs process step |
| /production/phone_process/unlocked_validator | Unlocked Validator | Validate unlocked device |
| /production/phone_process/wpt | WPT | Wireless Power Test |
| /production/phone_process/pcb_qc | PCB QC | PCB quality control |
| /production/phone_process/l3_Repair | L3 Repair | Level 3 repair |
| /production/phone_process/activation | Activation | Device activation |

## **4.3 Print City / Labels Pages (10 routes)**

| **Route** | **Module Name** | **Notes** |
| --- | --- | --- |
| /print_city | Print City Hub | PrintCity navigation hub |
| /fg_label | FG Label | FG label generation |
| /fg_laptop | FG Laptop | FG laptop label |
| /fg_watch | FG Watch | FG watch label |
| /camera_fg | CameraFG | Camera FG label |
| /free_l_fg | FreeLFG | Free location FG label |
| /mro_label | MRO Label | MRO label printing |
| /mri&mrb | MRI & MRB | MRI and MRB labels paginated |
| /battery | Battery | Battery label printing |
| /inventory_label | INVENTORY LABEL | Inventory label printing |
| /custom_label | Custom Label | Custom label generator |
| /bh_fg_with_gc | BH FG WITH GC | Back Housing FG label with GiftCard |
| /bh_fg_without_gc | BH FG WITHOUT GC | Back Housing FG label without GiftCard |

## **4.4 Production Tools ****&**** General Pages**

| **Route** | **Module Name** | **Notes** |
| --- | --- | --- |
| /qc_grading | QCGrading | Cosmetic/functional grading |
| /quick_fg | QUICK FG | Fast FG device lookup and processing |
| /laptop_validator | Laptop Validator | Laptop validation + part number creation |
| /match_unit | Match Unit | Unit matching tool |
| /units_received | Units Received | Units received tracking view |
| /reporteria | Reporteria | Reports viewer by report number |
| /production | Production Hub | Production navigation hub |
| /profile | Profile | User profile |
| /calendar | Calendar | Calendar view |
| /login | Login | Authentication page |

## **4.5 Route Definitions (routesPage.ts)**

Supply Chain: receiving, PPBoxLink, ShopifyOrder, GCPrinting, FGBoxBuilding, CreateProject,

  UpdateCondition, PoCreator, Shipped, ShippingLabel, BoxBuilding, BoxShipping,

  PalletShipping, PPBoxConsult, Transfer, Devices, Part, Box, Units, BinABin,

  QTY_ASN, UploadListASN

Phone Process: IQC, DSM1, BH_Replacement, BHRemoval, BHAssembly, Closing, IC, Aqc,

  Aqc2, ITriage, Rework, l3Triage, OverLay, UnlockedValidator, WPT, PCB_QC,

  L3Repair, Activation

FG Label: CameraFG, FreeLFG

Print City: MROLabel, Jigs, LaptopValidator, QCGrading, MatchUnit, QuickFG,

  FGWatch, FGLaptop, MriAndMrb, Battery, InventoryLabel, CustomLabel

# **5. Airlink Hub v3.3.x — Electron Desktop**

## **5.1 Technology Stack**

| **Layer** | **Technology** | **Details** |
| --- | --- | --- |
| Runtime | Electron (Windows) | main.js + preload.js + src/index.html |
| UI | Single-file HTML/JS | src/index.html (~3,500 lines, no bundler) |
| Database | SQL Server mssql | 192.168.181.248:13999 — SPN + AirlinkDR |
| Auth | Power Automate HTTP | SP_Hub_ValidateLogin → PA_Hub_Login |
| Permissions | LX_Menus (SPN) | SP_Hub_GetMenus — per-employee tab visibility |
| AI — Claude Chat | Anthropic API direct | claude-sonnet-4-6, streaming, thinking mode |
| AI — LX Agent | PA + Claude API | Daily report, Copilot Studio integration |
| AI — Dino | PA + SharePoint | Documentation search assistant |
| Chat History | SQL + 4 PA Flows | Hub_ChatSessions, Hub_ChatMessages in AirlinkDR |
| Logistics | Direct SQL (sqlExec) | SP_Hub_Logistics_GetDashboard, Tracking_Master |
| Production TV | Node.js + ngrok | airlink-production-tv.ngrok.app, PRD_UPH table |

## **5.2 IPC Channels**

contextIsolation: false — preload.js exposes window.electronAPI directly (NOT via contextBridge).

| **IPC Channel** | **Direction** | **Purpose** |
| --- | --- | --- |
| sql-exec | Renderer → Main | Execute SP directly against SQL Server |
| sql-ping | Renderer → Main | Test database connectivity |
| app-restart | Renderer → Main | app.relaunch() + app.exit(0) |

## **5.3 Tab Permissions (LX_Menus / TAB_MAP)**

| **Key** | **LX_Menus Column** | **Label** | **Content** |
| --- | --- | --- | --- |
| db | Tab_Dashboards | Dashboards | Logistics Dashboard + Back Housing TV |
| prod | Tab_Production | Production | Unit Lookup, Processes, IQC Status, Reports |
| sc | Tab_SupplyChain | Supply Chain | Logistics Dashboard, Register Shipment |
| rep | Tab_Reports | Reports | Reports (in development) |
| bi | Tab_AirlinkBI | Airlink BI | BI (coming soon) |
| oc | Tab_OrgChart | Org Chart | iframe orgchart.html |
| wh | Tab_Warehouse | Warehouse | Inventory Control (in development) |
| pc | Tab_PrintCity | Print City | Label printing (in development) |

# **6. Movement 1.5 — Excel VBA Workbook**

Movement_V1_5.xlsm — 20 sheets. Primary user: DR0005 (Jorge Antonio Ozuna Cabral). These processes are the migration targets for Nexus modules.

## **6.1 Sheet Inventory**

| **Sheet** | **Type** | **Key Columns / Purpose** | **Rows** |
| --- | --- | --- | --- |
| FGReport_v2 | Pivot | FG Report by condition: Without/With Condition, Shadow, Broken, Carrier Locked, CG, Watch FMI | ~3,019 |
| FGReport_v22 | Pivot | Same as v2 + NO SPEN column | ~3,018 |
| RAW | Staging | LOCATION, BOXN, AGING, AGING RANGE, WAREHOUSE, PROGRAM, GIFTCARD, GCN, DEVICE TYPE, BRAND, SKU, MODEL NO, MODEL, COLOR + more | ~2 |
| ReceivingData | Data Entry | RECEIVING DATE, TRACKING, GIFTCARD, Invoice, LPN, Model Number, Reg Model Number, Make, Model, Memory, Color, Initial Carrier, Grade, DeviceLock, IMEI, Serial, Battery Health %, 100% Working, FCCID, Added Date, COSMETIC GRADE, Inspection Date, USER | Template |
| VW_Fun&Cosm | SQL View | 71 real columns: identity, erasure, grade/cosmetic, 46 functional tests. See §6.2. | Header only |
| Fun&Cosm View All | Report | FUNCTIONALITY & COSMETIC RESULT REPORT — pivot by Model × PhoneCheckProfile × Route | ~32 |
| PN Uploader | Upload | Description, Model, Color, Capacity, Status, PartType | Template |
| FB Cost | Data Entry | FB Partnumber, Description, Serial, Location, WHS, Class, Facility, SMB Number, PO Number, Brand | Template |
| ASN | ASN Mgmt | IMEI, Serial Number, Tracking Number, Project Code, Model, Color, Capacity, SKU, Device Type, Ship From Location | Template |
| Main | Navigation | DR0005 launcher buttons for macros | 5 |
| Receiving Upload | Upload Tool | Button/macro to upload receiving data | 3 |
| PL / PL2 / PL3 | Packing Lists | BOX PACKING LIST — PO, Content, Description, Customer SKU, IMEI, GiftCard, BARCODE | 129/109/103 |
| LCD_CONDITION | Reference | LCD condition grading reference | Empty |
| sample_tab series | Templates | 5 pivot-based report templates | 14–498 |

## **6.2 VW_Fun****&****Cosm — 71 Real Columns**

| **Group** | **Columns** |
| --- | --- |
| Identity (8) | CreatedAtDate, CreatedAtTime, Project, Nick, GCN, Imei, IQC ROUTE, PhoneCheckProfile |
| Erasure (3) | Erased, EraseProtocol, ErasureLink |
| Grade / Cosmetic (14) | Model, Color, Conditions, Grade, SA, SA1, SB, SC, BH Grade, LCD1Condition, LCD2Condition, BatterySOH, BATTERY HEALTH %, Fully Functional |
| Functional Tests (46) | Accelerometer, Bluetooth, Bottom Mic Quality, Brightness, Cosmetics, Digitizer N Pattern, Earpiece Quality, Face ID, Flashlight, Flip Switch, Front Camera, Front Camera Quality, Front Mic Quality, Front Video Camera, GPS, Glass Condition, Gyroscope, LCD, LiDAR, Light Sensor, Loudspeaker Quality, Multi Touch, OEM Parts, Power Button, Proximity Sensor, Rear Cam to Gallery, Main Camera, Rear Camera, Rear Camera Quality, Rear Mic Quality, Rear Video Camera, Screen Rotation, Telephoto Camera, Telephoto Camera Quality, True Tone, UltraWide Camera, UltraWide Camera Quality, Vibration, Volume Down Button, Volume Up Button, WiFi, Wireless Charging, Battery Warning, Beta OS, MDM, Prueba, FAIL |

# **7. Nexus Migration Map**

| **Process** | **M1.5 Sheet** | **API Available** | **Nexus Module** | **Hub Section** |
| --- | --- | --- | --- | --- |
| Receiving | ReceivingData | ✓ Receiving | Receiving module | SC > Processes |
| ASN Upload | ASN | ✓ ASN | ASN module | SC > Processes |
| FG Report | FGReport + RAW | ✓ Reporteria | FG Report viewer | Prod > Reports |
| QC Grading / Fun&Cosm | VW_Fun&Cosm | ✓ QCGrading | QC Grading | Prod > Processes |
| IQC | — | ✓ IQC | IQC module | Prod > Processes |
| Quick FG | — | ✓ QuickFG | Quick FG | Prod > Processes |
| Box Building | — | ✓ BoxBuilding | Box Building | Supply Chain |
| FG Box Building | — | ✓ FGBoxBuilding | FG Box Building | Production |
| Box Shipping | — | ✓ BoxShipping | Box Shipping | Supply Chain |
| Pallet Shipping | — | ✓ PalletShipping | Pallet module | Supply Chain |
| Inventory (Kanban) | — | ✓ Inventory | Kanban inventory | Warehouse |
| Purchase Orders | FB Cost (partial) | ✓ PO | PO module | Supply Chain |
| Transfer Bin-to-Bin | — | ✓ TransferBinABin | Transfer | Warehouse |
| Transfer Box | — | ✓ TransferBox | Transfer | Warehouse |
| Transfer Units | — | ✓ TransferUnits | Transfer | Warehouse |
| Update Condition | — | ✓ UpdateCondition | Condition updater | Production |
| GC Label Printing | — | ✓ GCPrinting | GC Print | Print City |
| Battery Labels | — | ✓ Battery + PrintCity | Battery labels | Print City |
| Inventory Labels | — | ✓ PrintCity | Inventory labels | Print City |
| MRO / MRI / MRB Labels | — | ✓ PrintCity | MRO/MRI/MRB | Print City |
| BH Labels | — | ✓ PrintCity | BH labels | Print City |
| PP Box Kits | — | ✓ PPBoxLink | PP Box module | Print City |
| Shopify Orders | — | ✓ ShopifyOrder | Shopify module | Supply Chain |
| Laptop Validation | — | pending merge | Laptop module | Production |
| Packing List | PL / PL2 / PL3 | ✗ No endpoint yet | PL generator | SC > Processes |
| Shipping Label | — | controller empty | Shipping Label | Supply Chain |
| Logistics Dashboard | N/A | ✓ Hub sqlExec | SC Dashboard (done) | SC > Dashboards |

# **8. Technical Reference**

## **8.1 Connection Strings**

AIRLINK (Production): Server=192.168.181.248,13999; Database=AIRLINK; User Id=ALAppUser; Password=Airlink*000856

AirlinkDR (Prod):     Server=192.168.181.248,13999; Database=AirlinkDR; User Id=ALAppUser; Password=Airlink*000856

SPN (Production):     Server=192.168.181.248,13999; Database=SPN; User Id=ALAppUser; Password=Airlink*000856

AirlinkUSA:           Server=AFS2SQL1; Database=AirlinkUSA; User Id=airlink; Password=airlink

AIRLINK (Dev):        Server=192.168.181.246,8989; Database=AIRLINK; User Id=ARLQA; Password=ARLQA123

AirlinkDR (Dev):      Server=192.168.181.246,8989; Database=AirlinkDR; User Id=ARLQA; Password=ARLQA123

## **8.2 AI Configuration**

| **Parameter** | **Value** |
| --- | --- |
| Claude model (Hub) | claude-sonnet-4-6 |
| Max tokens (Hub chat) | 16,000 |
| Max tokens (PA grading) | 12,000 |
| Thinking budget | 8,000 tokens (interleaved-thinking-2025-05-14) |
| OpenWeatherMap API key | ${OPENWEATHERMAP_API_KEY} (Santo Domingo, DO) |

## **8.3 External Services**

| **Service** | **URL / Config** |
| --- | --- |
| Back Housing TV | https://airlink-production-tv.ngrok.app (fixed ngrok domain) |
| TV update interval | 5 minutes (300,000ms) |
| Movement-API (dev) | https://localhost:7256 |
| Movement-API (prod) | TBD — needs ngrok fixed domain or internal deployment |
| Zebra Browser Print | localhost — requires Zebra Browser Print app running on client |

## **8.4 Key Patterns ****&**** Conventions**

| **Pattern** | **Rule** |
| --- | --- |
| Employee codes | DR + 4 digits zero-padded — DR0002, DR0020, DR0200 |
| Cédula validation | RIGHT(REPLACE(Cedula,'-',''),4) — last 4 digits after stripping hyphens |
| Supervisor field (SPN) | TRY_CAST(CONVERT(VARCHAR,e.Supervisor) AS INT) = sup.Numero |
| varbinary columns (SPN.Empleados) | Always CONVERT(VARCHAR,...) — implicit conversion errors otherwise |
| SP result normalization (PA) | Check ResultSets.Table1[0] → Table1[0] → raw (in that order) |
| PA empty result guard | coalesce(...?['Table1'],json('[]')) — Table1 key missing when 0 rows |
| PA string params | Expression tab only: triggerBody()?['field'] — NOT Dynamic Content |
| PA function names | toUpper() lowercase t — Upper() throws template function error |
| Panel z-index | Section panels use z-index:10 — sidebar (.sb) must stay visible |
| Panel container | Panels append to .hc NOT #hub — sidebar never covered |
| DOM generation | Always createElement/appendChild — never string concatenation with nested quotes |
| Hub version format | airlink-hub_X.X.X.zip — increment 0.0.1 per release |
| Movement-API response | { StatusCode, Message, Data, IsSuccess } — universal ResponseDTO |
| GCN (GiftCard Number) | Primary unit identifier throughout Movement (SPC_Receiving.GCN unique) |

## **8.5 MovementStatus Values (Tracking_Master)**

| **Value** | **Meaning** |
| --- | --- |
| Inbound_PreTransit | Shipment created, not yet picked up by carrier |
| Inbound_Transit | In transit to DR |
| Inbound_Delivery | Out for delivery |
| Inbound_Delivered | Received — closed this week |
| Outbound_PreTransit | Pick UP — FedEx scheduled to collect |
| Outbound_Transit | In transit to destination |
| Outbound_Delivery | Out for delivery |
| Outbound_Delivered | Delivered — closed this week |
| Clearance | On hold — customs or documentation clearance required |
| Unknown | Status not yet determined by tracking API |

## **8.6 FedEx Service Type Codes**

| **Code** | **Full Service Name** | **Type** |
| --- | --- | --- |
| IE | FedEx International Economy | Express parcel |
| IEF | FedEx International Economy Freight | Express freight |
| IPFS | FedEx International Priority Freight Service | Express freight |
| IPED | FedEx International Priority Express Door | Express parcel |
| FG | FedEx Ground | Ground |
| P-2 | FedEx 2Day (Priority 2) | Express parcel |

## **8.7 Hub Section Tree**

Dashboards  > Operations > Logistics Dashboard | Back Housing TV

            > Production > Back Housing TV  |  Warehouse > (soon)

Production  > Dashboards > Back Housing TV

            > Reports    > Devices: Inventory / Unlocked / Processed

                        > Laptops / Housing-Enclosure / LCD

            > Processes  > NPI (17 steps: IQC → Packing)

                        > Regular Process (IQC → DSM1 → Triage → DSM2)

                        > Unit Lookup | IQC Status

Supply Chain> Dashboards > Logistics Dashboard

            > Reports    > Inbound/Outbound ASNs, Units Rcvd/Shipped, Parts

            > Processes  > Register Shipment / ASN Upload / Receiving

Warehouse   > Reports    > Parts Inventory / Devices / FG

            > Inventory Control > Devices / Parts

Print City  > FG Labels  > Phones / Laptops / Watches / Cameras / Housing

            > PP Box Kits / Battery / Parts GiftCard / Inventory Label

            > MRO / MRI&MRB / BH Model Condition / BIN / GiftCard Reprint

Airlink Distribution DR  ·  Software & IT  ·  Nexus Integration Documentation v2.0

CONFIDENTIAL — Internal Use Only

Page  of