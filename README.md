 Amanat

*Every deposit, every deadline, on record.*

A security-deposit and Fixed Deposit Receipt (FDR) management portal for a
regulation bureau: register every deposit an institution holds, watch
maturities before they lapse, move each instrument through approval, renewal
and release, and keep a tamper-evident audit trail of every action.

## Features

- **Dashboard** — totals, active deposits, pending approvals, what matures in
  the next 30/90 days, a 12-month maturity timeline, bank and status
  breakdowns, and recent activity.
- **Deposits register** — search, filter (status, bank, institution, maturity
  window), sort and paginate every FDR; create, edit and view full details;
  export any filtered view to CSV.
- **Approval workflow** — Draft → Pending approval → Approved / Rejected;
  release requests; renewal that links the replacement FDR to the old one.
- **Alerts** — configurable reminder days (default 90/60/30/7 before
  maturity), overdue alerts, and a review-SLA alert; acknowledge or re-run
  from the alerts centre.
- **Documents** — attach PDF/JPG/PNG scans (up to 10 MB) to each FDR,
  download or remove them.
- **Audit history** — every action recorded with actor, time and status
  change; SHA-256 hash-chained with a one-click *Verify integrity* check.
- **Integrity Suite** — per-FDR health score with explainable findings and
  fixes, submitted vs officer-verified provenance, and a 12-month Maturity
  Radar with projected payouts.
- **Spreadsheet bulk ingestion** — upload CSV deposit schedules up to 500 rows; pre-import validation with Health Check findings, fuzzy bank matching, and atomic draft creation.
- **Release readiness checklist** — 5-point checklist (affidavit received, no pending complaints, closure certificate if closing, matured or lock-in complete, bank discharge noted) recorded on the release request and enforced before the Competent Authority sanctions it.
- **Notices and candidate queues** — generate and issue numbered official letters (maturity reminders, overdue notices, shortfall notices, release letters) with standard bureau references (`FDRP/NOT/2026/NNNN`).
- **Alert escalation** — configurable aging thresholds (`escalate_after_days`, default 3) that flag unacknowledged alerts with an Escalated badge and elevated sort priority.
- **Bureau performance analytics** — executive operating metrics (approval and release turnaround, verification rate, document coverage, on-time renewals, data quality) with 6-month historical trends.
- **Document categorization & instrument lineage** — typed document classification (`fdr_receipt`, `lien_letter`, etc.) with missing lien letter detection; complete multi-generation renewal lineage chains with bidirectional navigation.

## Tech stack

- **Backend:** Python 3.11+, FastAPI, SQLAlchemy 2.0, SQLite (single file),
  Pydantic v2, JWT (PyJWT) + bcrypt, pytest
- **Frontend:** React 19, TypeScript, Vite, Tailwind CSS v4, shadcn/ui,
  TanStack Query, Recharts, Playwright (smoke test)

## Prerequisites

- Python 3.11 or newer — `python --version`
- Node.js 20 or newer — `node --version`

## Setup (one time)

**Windows:**

```bat
scripts\setup.bat
```

**macOS / Linux:**

```sh
bash scripts/setup.sh
```

This creates a virtual environment in `backend\.venv`, installs the backend
requirements and the frontend packages. Optional: copy `.env.example` to
`backend/.env` to change the secret key, database path or other settings —
the defaults work out of the box.

## Run

**Windows:**

```bat
scripts\run.bat
```

**macOS / Linux:**

```sh
bash scripts/run.sh
```

Then open **http://localhost:5173**. The API and its interactive docs are at
**http://localhost:8000/docs**. On first start the app creates its database
and seeds demo data automatically (14 institutions, 44 FDRs with 18 months of history,
documents and alerts).

### Demo logins

| Role | Role Title | Email | Password | Access Scope |
|---|---|---|---|---|
| Administrator | Section Officer | `admin@fdrportal.demo` | `Admin@2026` | Bureau-wide operations, imports, settings, notices |
| Reviewer | Scrutiny Officer | `reviewer@fdrportal.demo` | `Review@2026` | Verifications, observations, submissions approval |
| Competent Authority | Approver | `approver@fdrportal.demo` | `Approve@2026` | Maker-checker release sanctions & returns |
| Auditor | Audit Officer | `auditor@fdrportal.demo` | `Audit@2026` | Reads everything, runs the audit integrity check, exports; changes nothing |
| Institution | Deccan Crest Institute | `deccan.crest@fdrportal.demo` | `Inst@2026` | Scoped deposit submission, replies, release requests |

## Demo flow (about 2-3 minutes)

1. Sign in as **Institution** (`Institution: Deccan Crest`). Land on the institution home page, click **Submit an FDR**, enter deposit receipt details with the fixed institution scope, upload the receipt PDF, and **Submit for approval**.
2. Sign out and sign in as **Reviewer** (`Reviewer`). Open the submitted FDR, click **Observations**, and **Raise an observation** requesting branch lien confirmation.
3. Sign in as **Institution**, open the observation, and post an official reply with clarifications.
4. Sign in as **Reviewer**, close the observation, click **Mark Verified** (or attach a confirmation letter to **Bank-confirm**), and **Approve** the deposit with an audit remark.
5. Sign in as **Administrator** (`Admin`), open the approved FDR, and click **Request Release** with the 5-point checklist complete.
6. Switch to **Reviewer** to verify that release sanction is refused (approver role required).
7. Sign in as **Competent Authority** (`Authority`), review the completed checklist, and click **Approve Release** to sanction release.
8. Notice issuance: official release letter `FDRP/NOT/2026/NNNN` is issued with an HMAC-SHA256 verification code and QR mark. Open the public route `/verify` to confirm authenticity.
9. Sign in as **Auditor** (`Auditor`): open the Audit Log, run **Verify integrity** (*Intact*), and observe that all operational action buttons are hidden.

## Project structure

```
backend/
  app/
    main.py            # app factory, CORS, static hosting, startup
    config.py          # settings from environment / .env
    db.py              # engine and session
    models.py          # SQLAlchemy models
    schemas.py         # Pydantic request/response models
    security.py        # password hashing, JWT, roles
    audit.py           # hash-chained audit log
    rules.py           # maturity state, projected value, findings, health
    alerts.py          # alert engine
    seed.py            # demo data
    routers/           # one module per API area
  tests/               # pytest suite (temp seeded SQLite per session, ~10 s)
  data/                # SQLite file (created on first run, gitignored)
  uploads/             # uploaded documents (gitignored)
frontend/
  src/
    lib/               # typed API client, types, formatting helpers
    components/        # ui/ primitives, layout shell, shared widgets
    pages/             # one folder per screen
e2e/                   # Playwright smoke test
scripts/               # setup / run / reset-demo for Windows and macOS/Linux
```

## Testing

Backend tests (fast, temp SQLite, no network):

```sh
cd backend
.venv\Scripts\python -m pytest        # Windows
.venv/bin/python -m pytest            # macOS / Linux
```

Browser smoke test: a real Chromium run of the whole client flow (admin
creates and submits an FDR, reviewer approves it, audit check, alerts, CSV
export), plus screenshots of every screen in `e2e/screenshots/`. It builds
the frontend if no build exists yet (run `npm run build` in `frontend` to
refresh an old one) and starts its own backend on port 8010 with a
temporary database, so your demo data is never touched and the app does not
need to be running. Takes about 20 seconds.

Windows:

```bat
backend\.venv\Scripts\python -m pip install playwright
backend\.venv\Scripts\python -m playwright install chromium
backend\.venv\Scripts\python e2e\smoke_test.py
```

macOS / Linux:

```sh
backend/.venv/bin/python -m pip install playwright
backend/.venv/bin/python -m playwright install chromium
backend/.venv/bin/python e2e/smoke_test.py
```

The first two lines are needed once per machine.

## Reset the demo

Deletes the SQLite file and all uploads so the next start re-seeds fresh
demo data:

```bat
scripts\reset-demo.bat      :: Windows
bash scripts/reset-demo.sh  :: macOS / Linux
```

## Troubleshooting

- **Port 8000 or 5173 already in use** — another process holds the port.
  Find and stop it (Windows: `netstat -ano | findstr :8000` then
  `taskkill /PID <pid> /F`; macOS/Linux: `lsof -ti :8000 | xargs kill`), then
  run again.
- **`Python 3.11 or newer is required`** — the backend uses Python 3.11
  features. Install a current Python and make sure `python` (Windows) or
  `python3` (macOS/Linux) points to it.
- **`npm install` fails or the frontend misbehaves** — delete
  `frontend/node_modules` and run `scripts\setup.bat` / `scripts/setup.sh`
  again.
- **macOS/Linux: "Permission denied" running a script** — run it with
  `bash scripts/…` instead of executing it directly.
- **`npm install` reports 1 high severity vulnerability** — this is an
  advisory for `zod`, the form-validation library: very large inputs can use a
  lot of memory while the browser validates them. It only affects the
  officer's own browser and is not reachable by other users. The fix needs a
  major-version upgrade, planned for the next phase; do not run
  `npm audit fix --force`, it would break the forms.
- **The demo data looks wrong or the database is locked** — stop the app,
  run the reset script above, start again.
