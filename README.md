# 📋 HR Ops Automation Suite — Google Sheets

> Real-time master data sync + automated two-week HR performance reports, built entirely on Google Apps Script.

![Google Apps Script](https://img.shields.io/badge/Google_Apps_Script-4285F4?style=flat-square&logo=google&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Google_Sheets-34A853?style=flat-square&logo=google-sheets&logoColor=white)
![Gmail](https://img.shields.io/badge/Gmail-EA4335?style=flat-square&logo=gmail&logoColor=white)
![Trigger](https://img.shields.io/badge/Trigger-onEdit_%2B_Weekly_Cron-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Production_Ready-brightgreen?style=flat-square)

---

## Overview

Two-module automation layer over a multi-sheet Google Sheets HR database:

| Module | Trigger | What It Does |
|--------|---------|--------------|
| **Sheet Sync** | `onEdit` (real-time) | Upserts edits from individual HR sheets → `MASTER DATA`; flags duplicate phone numbers in red |
| **Weekly Report** | Weekly time-based cron | Computes HR KPIs for the last two Mon–Sun windows and emails a formatted comparison table |

**No server. No external APIs. No cost.**

---

## Architecture

```
┌──────────────┐    onEdit     ┌─────────────┐
│  Preetika    │ ─────────────▶│             │
│  Anadi       │  phone match/ │ MASTER DATA │
│  Charu Mam   │  upsert row   │             │
└──────────────┘               └─────────────┘
       │
       │ duplicate?
       ▼
  🔴 Red highlight + Alert

┌──────────────┐  Weekly Cron  ┌─────────────┐
│  Preetika    │ ─────────────▶│  Gmail HTML │
│  Anadi       │  KPI compute  │   Report    │
└──────────────┘               └─────────────┘
```

---

## Module 1 — Real-Time Sheet Sync (`onEdit`)

### Watched Sheets
`Preetika` · `Anadi` · `Charu Mam`

### Logic
1. Fires on any cell edit in a watched sheet
2. Reads phone number from Column C of the edited row
3. Scans `MASTER DATA` Column C for a match
   - **Match found** → overwrite entire row (upsert)
   - **No match** → `appendRow()` (insert)
4. Clears all backgrounds in phone column, re-scans for duplicates
   - Duplicates → red highlight + `UI.alert()`
5. Refreshes pivot table by toggling `A1` value

### Sheet Schema (all watched sheets + MASTER DATA)

| Column | Field |
|--------|-------|
| A | Name |
| C (col 3) | Contact No. *(dedup key)* |
| … | All other fields synced as-is |

---

## Module 2 — Weekly HR Report (`mainHRReport_TwoWeekComparison`)

### Date Windows (auto-computed, Mon–Sun)
```
Last Week : Most recent completed Monday → Sunday
Prev Week : The Monday → Sunday before that
```

### Column Mapping (HR sheets)

| Column Index | Field |
|---|---|
| 0 (A) | Date |
| 4 (E) | Position / Job Role |
| 14 (O) | Contacted (Yes / No) |
| 15 (P) | Scheduled |
| 16 (Q) | Interview Done |

### KPIs Computed

| Metric | Formula |
|--------|---------|
| Total Calls | Rows where Contacted = Yes or No |
| Contacted | Rows where Contacted = Yes |
| Calls / Day | Total Calls ÷ 6 |
| Scheduled | Rows where Scheduled = "Scheduled" |
| Interview Done | Rows where Done = "Yes" |
| Scheduling Ratio | Scheduled ÷ Contacted × 100 |
| Top 2 Jobs | Top 2 positions by row frequency |

> **Note:** One row is subtracted from `totalCalls` and `contactedYes` as a data hygiene offset for header/test rows.

### Email Output

Single HTML email with a 5-column comparison table:

```
Metric | Preetika (Last) | Anadi (Last) | Preetika (Prev) | Anadi (Prev)
```

---

## Project Structure

```
hr-ops-automation/
│
├── Code.gs                    # All script logic (both modules)
├── README.md                  # This file
└── docs/
    └── sheet_schema.md        # Column reference for all sheets
```

---

## How to Run

### 1. Deploy

```
Google Sheet → Extensions → Apps Script → Paste Code.gs → Save
```

### 2. Configure Recipients (Weekly Report)

```javascript
// Code.gs — Line 3
const recipientEmails = [
  "hr@yourcompany.com",
  "manager@yourcompany.com"
];
```

### 3. Set Triggers

```
Apps Script → Triggers → Add Trigger

── Trigger 1: Sheet Sync ──────────────────────
  Function   : onEdit
  Event type : From spreadsheet → On edit
  (Auto-registered; no manual setup needed)

── Trigger 2: Weekly Report ───────────────────
  Function   : mainHRReport_TwoWeekComparison
  Event type : Time-driven → Week timer → Monday 9:00 AM
```

### 4. Authorize Permissions

On first run, Google will prompt:
- `spreadsheets` — read/write sheet data
- `gmail.send` — send weekly report via MailApp
- `ui` — show duplicate alerts

---

## Key Takeaways

- **Upsert pattern over append-only** — phone number dedup key ensures `MASTER DATA` stays normalized without manual reconciliation across 3 HR sheets
- **Real-time + scheduled in one codebase** — `onEdit` handles live data integrity while the weekly cron handles async reporting, covering both operational and analytical needs
- **Zero-infrastructure analytics** — full KPI pipeline (filter → aggregate → format → deliver) runs inside Google's runtime; no warehouse, no BI tool required

---

## Author

**Nitesh Duhan** — Data Scientist & ML Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-niteshduhan--carp112-0A66C2?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/niteshduhan-carp112)
[![Email](https://img.shields.io/badge/Email-niteshduhan686@gmail.com-EA4335?style=flat-square&logo=gmail)](mailto:niteshduhan686@gmail.com)
[![Instagram](https://img.shields.io/badge/Instagram-@nitesh._duhan-E4405F?style=flat-square&logo=instagram)](https://www.instagram.com/nitesh._duhan)
