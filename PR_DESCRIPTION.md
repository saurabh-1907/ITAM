## IT Asset Lifecycle Manager – SAP CAP + Fiori + HANA Cloud + XSUAA

Implements the FRD in this repo (`IT_Asset_Lifecycle_Manager_Functional_Requirements_v1_1`) with the same stack and conventions as [saurabh-1907/project1-sprint](https://github.com/saurabh-1907/project1-sprint/tree/feature/psa-cap-backend): CAP Node.js, OData V4, SAP HANA Cloud (HDI), XSUAA + approuter, Fiori Elements apps in a sandbox Fiori launchpad, MTA for Cloud Foundry, `node --test` + `cds.test`.

### What's included

**Data model**: Employees, AssetCategories, Vendors, Assets, AssetAssignments, OwnershipLedgers (immutable), MaintenanceRecords, ArchiveRecords, ReportFiles, ReportLogs, plus sample data.

**Services (FRD Service Layer)**
| Service | Role | Content |
|---|---|---|
| `AssetAdminService` `/odata/v4/admin` | Admin | asset + master-data maintenance (draft), lifecycle actions, history, maintenance, archive, report audit log |
| `AssetManagerService` `/odata/v4/manager` | Manager, Admin | KPI dashboard, report preview, PDF export / download / e-mail, read-only operations |
| `AssetUserService` `/odata/v4/user` | User | inventory (no costs, no archived), own assignments, own history |

**Lifecycle rules (FR-001 … FR-012)**
- unique asset tag / serial; category and procurement date mandatory; initial status AVAILABLE; inactive categories and vendors can't be selected
- assign → transfer → return close and open possession periods; **distribution date ≥ procurement date**; only active employees
- maintenance start / complete; retire with a reason; **retire and archive are rejected while assigned**; archive only for retired assets
- **no physical delete**; every event writes an ownership-ledger row (actor, date, from/to employee and status); ledger and archive rows reject UPDATE / DELETE at the database-service level
- offboarding: an employee who still holds assets can't be set to INACTIVE

**Dashboard and reports (FR-021 … FR-026)**
- KPI tiles (total, utilisation, available, overdue returns, open maintenance, end-of-life soon, asset value), donut and bar charts with labels and values, plus a table with the same numbers (NFR-011); drill-down to the asset list
- 4 reports (inventory, assignments, maintenance, lifecycle history), filtered the same way as the dashboard (status, category, location, date)
- PDF with title, generation time, requester, filters, a labelled chart and all rows (dependency-free PDF writer); limit 5,000 rows
- download link only works for the requester and expires after 15 min (NFR-010)
- e-mail: validated recipients; idempotent `requestId`, so a retry or double-click never sends twice; failures come back as `FAILED`, are logged, and can be retried (NFR-009)
- audit log of export / download / e-mail without report contents (NFR-012)

**Fiori**: role-based launchpad (Admin / Manager / User tiles with live KPI numbers); Manage Assets (List Report + Object Page with General, Procurement, Current State, Assignments, Maintenance, History and Retirement/Archive facets and the 7 lifecycle actions); Employees, Categories, Vendors, Report Audit Log, Asset Overview, Hardware Inventory, My Assignments and My Asset History apps; SAPUI5 Dashboard and Reports & Export apps; all labels in `_i18n/i18n.properties`.

**Security / deployment**: `xs-security.json` with Admin / Manager / User roles and the role collections ITAM_Admin / ITAM_Manager / ITAM_User; `mta.yaml` (HDI container, XSUAA, CAP server, approuter).

### How it was tested
- `npm test`: **40 tests pass** (roles and record-level access, every lifecycle rule, ledger immutability via OData and on the database, offboarding, dashboard KPIs and filters, report preview, PDF content, download ownership and expiry, e-mail validation / idempotency / failure and retry, audit log)
- `cds build --production` succeeds (HANA artifacts generated)
- Loaded the launchpad and apps in Chromium against SAPUI5 1.153.0 as Admin, Manager and User: launchpad, dashboard, reports (preview → export → download → e-mail), Manage Assets list / object page / action dialog, and the user apps

### Open points / not in scope
- **INT-01 … INT-04** and the **S4HANA_PROD / QAS** destinations (purchase orders, goods receipts, invoice PDFs to DIM, vendor notifications) look like they come from an invoice process and don't match the asset scope. Please confirm whether they're needed.
- E-mail delivery uses a `mail-api` service behind a BTP Destination when configured; locally and in tests the mail is only logged.
- Generated PDFs are stored in HANA instead of SAP Document Information Management.
- CI/CD (NFR-006), performance and availability targets, and the `ind1` region are deployment settings.

### Run in SAP Business Application Studio / deploy
Step-by-step guide: [`docs/BAS_RUN_AND_DEPLOY.md`](docs/BAS_RUN_AND_DEPLOY.md) – entitlements, HANA Cloud instance, BAS dev space, local run, hybrid run (HANA + XSUAA + approuter), `mbt build` + `cf deploy`, role collections, troubleshooting.

### Run locally
```bash
npm install
npx cds watch   # http://localhost:4004/index.html  – admin@itam-demo.com/admin, manager@itam-demo.com/manager, ravi.kumar@itam-demo.com/user
```

🤖 Generated with [Claude Code](https://claude.com/claude-code)

https://claude.ai/code/session_01UwQH9KmKvkFSBeNoM86n8k
