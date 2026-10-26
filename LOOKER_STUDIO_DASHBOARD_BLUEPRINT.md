# Sales KPI Dashboard — Looker Studio Build Guide

A complete blueprint for building a custom Looker Studio (formerly Data
Studio) dashboard on top of the **On Demand Wellness** master tracking
sheet. Follow this guide top-to-bottom and you'll have a 4-page,
auto-refreshing dashboard wired to your Google Sheet.

---

## 0. Before you start

**Source sheet:** Master tracking sheet with one row per month per
business/offer/platform.

**Schema (34 columns):**

| # | Column | Type | Notes |
|---|--------|------|-------|
| 1 | Date | Date | First-of-month |
| 2 | Business Name | Text | Dimension |
| 3 | Setter Name | Text | Dimension |
| 4 | Closer Name | Text | Dimension |
| 5 | Offer Name | Text | Dimension |
| 6 | Platform | Text | Dimension |
| 7 | Ad Spend | Currency | Metric |
| 8 | Reach | Number | Metric |
| 9 | Landing Page Views (GA) | Number | Metric |
| 10 | New Opt ins (Email) | Number | Metric |
| 11 | New Applications | Number | Metric |
| 12 | Emails Sent | Number | Metric |
| 13 | Emails Opened | Number | Metric |
| 14 | Emails Replied | Number | Metric |
| 15 | Dials | Number | Metric |
| 16 | DMs | Number | Metric |
| 17 | Follow Ups | Number | Metric |
| 18 | Meaningful Conversations | Number | Metric |
| 19 | Triage Calls Booked | Number | Metric |
| 20 | Triage Calls Due | Number | Metric |
| 21 | Triage Calls Showed | Number | Metric |
| 22 | Sales Calls Booked | Number | Metric |
| 23 | Sales Calls Due | Number | Metric |
| 24 | Sales Calls Showed | Number | Metric |
| 25 | New Customers Closed | Number | Metric |
| 26 | PIF | Currency | Paid In Full revenue |
| 27 | Revenue | Currency | Metric |
| 28 | Cash Collected | Currency | Metric |
| 29 | Recurring Cash | Currency | Metric |
| 30 | Recurring Cash Collected | Currency | Metric |
| 31 | Refund Amount | Currency | Metric |
| 32 | Total Expenses | Currency | Metric |
| 33 | Customers Cancelled | Number | Metric |
| 34 | Chargebacks | Currency | Metric |

**Sheet hygiene checklist (do this once):**

1. Row 1 contains the headers above — exactly.
2. `Date` column is formatted as Date (Format → Number → Date).
3. All currency columns formatted as Currency.
4. No merged cells, no totals row, no blank rows between data.
5. Keep one tab as the canonical source (e.g. `Master`). Looker Studio
   will pick this tab.

---

## 1. Connect the sheet to Looker Studio

1. Go to <https://lookerstudio.google.com> → **Create → Report**.
2. Pick the **Google Sheets** connector.
3. Select your spreadsheet → the `Master` tab.
4. Check **Use first row as headers**.
5. Click **Add**.
6. Rename the data source to `Sales Master` (top-left).

---

## 2. Create calculated fields (one-time setup)

In the data source editor (**Resource → Manage added data sources →
Edit**), click **Add a Field** and create each of these. Paste the
formula exactly.

### Efficiency / ratio fields

| Field name | Formula | Format |
|---|---|---|
| ROAS | `Revenue / Ad Spend` | Number (2 decimals) |
| CAC | `Ad Spend / New Customers Closed` | Currency |
| Cost per Opt-in | `Ad Spend / New Opt ins (Email)` | Currency |
| Cost per Application | `Ad Spend / New Applications` | Currency |
| Opt-in Rate | `New Opt ins (Email) / Landing Page Views (GA)` | Percent |
| Application Rate | `New Applications / New Opt ins (Email)` | Percent |
| Email Open Rate | `Emails Opened / Emails Sent` | Percent |
| Email Reply Rate | `Emails Replied / Emails Sent` | Percent |
| Triage Show Rate | `Triage Calls Showed / Triage Calls Due` | Percent |
| Sales Show Rate | `Sales Calls Showed / Sales Calls Due` | Percent |
| Close Rate | `New Customers Closed / Sales Calls Showed` | Percent |
| Avg Customer Value | `Revenue / New Customers Closed` | Currency |
| Avg Cash per Customer | `Cash Collected / New Customers Closed` | Currency |

### Profitability fields

| Field name | Formula | Format |
|---|---|---|
| Net Cash | `Cash Collected - Total Expenses - Refund Amount - Chargebacks` | Currency |
| Profit Margin | `(Cash Collected - Total Expenses - Refund Amount - Chargebacks) / Cash Collected` | Percent |
| Recurring Share | `Recurring Cash / Revenue` | Percent |

### Date helpers

| Field name | Formula | Format |
|---|---|---|
| Month | `MONTH(Date)` | Number |
| Year | `YEAR(Date)` | Number |
| Year-Month | `FORMAT_DATETIME("%Y-%m", Date)` | Text |

> **Tip:** if a ratio comes out as `Infinity` or `NaN` for empty months,
> wrap the numerator in `IF(... > 0, ..., NULL)` to suppress.

---

## 3. Report-level controls (header bar)

Add these once at the top of page 1 — they'll filter every page if you
set them as **Report-level**.

1. **Date range control** → default: This year.
2. **Drop-down filter:** `Business Name`.
3. **Drop-down filter:** `Offer Name`.
4. **Drop-down filter:** `Platform`.
5. **Drop-down filter:** `Closer Name`.
6. **Drop-down filter:** `Setter Name`.

Select all six → right-click → **Make report-level**.

---

## 4. Page layout

### Page 1 — Executive Overview

**Top row (6 scorecards):**

| Scorecard | Metric | Compare to |
|---|---|---|
| Revenue | `Revenue` (SUM) | Previous period |
| Cash Collected | `Cash Collected` (SUM) | Previous period |
| New Customers | `New Customers Closed` (SUM) | Previous period |
| Net Cash | `Net Cash` (SUM) | Previous period |
| ROAS | `ROAS` (AVG) | Previous period |
| CAC | `CAC` (AVG) | Previous period |

**Middle row:**

- **Time series chart** — X: `Date`, Y: `Revenue`, `Cash Collected`
  (two lines). Title: *Revenue vs Cash Collected*.
- **Bar chart** — Dimension: `Offer Name`, Metric: `Revenue`. Sort
  desc. Title: *Revenue by Offer*.

**Bottom row:**

- **Table** — Dimensions: `Year-Month`. Metrics: `Revenue`,
  `Cash Collected`, `New Customers Closed`, `Avg Customer Value`,
  `Net Cash`. Sort by Year-Month desc. Title: *Monthly Summary*.

### Page 2 — Marketing & Lead Gen

**Top row (5 scorecards):** `Ad Spend`, `Reach`, `Landing Page Views
(GA)`, `New Opt ins (Email)`, `New Applications`.

**Funnel section:**

- Use scorecards side-by-side with conversion % between them:
  - LP Views → scorecard
  - Opt-in Rate → scorecard (%)
  - Opt-ins → scorecard
  - Application Rate → scorecard (%)
  - Applications → scorecard

**Cost efficiency row (4 scorecards):**

- `Cost per Opt-in`
- `Cost per Application`
- `CAC`
- `ROAS`

**Platform breakdown:**

- **Bar chart** — Dim: `Platform`, Metrics: `Ad Spend`, `Revenue`,
  `ROAS`. Title: *Performance by Platform*.

**Trend:**

- **Time series** — X: `Date`, Y: `Ad Spend`, `New Opt ins (Email)`,
  `New Applications`. Title: *Lead-Gen Trend*.

### Page 3 — Sales Activity & Funnel

**Outreach scorecards row:** `Dials`, `DMs`, `Emails Sent`,
`Follow Ups`, `Meaningful Conversations`.

**Email funnel (3 scorecards + 2 rates):**

- `Emails Sent` → `Email Open Rate` → `Emails Opened` →
  `Email Reply Rate` → `Emails Replied`.

**Call funnel (table):**

Build a table with these metrics as columns (single row):

| Triage Booked | Triage Due | Triage Showed | Triage Show Rate | Sales Booked | Sales Due | Sales Showed | Sales Show Rate | Closed | Close Rate |
|---|---|---|---|---|---|---|---|---|---|

**Setter leaderboard:**

- **Table** — Dim: `Setter Name`. Metrics: `Triage Calls Booked`,
  `Triage Calls Showed`, `Triage Show Rate`. Sort by Booked desc.

**Closer leaderboard:**

- **Table** — Dim: `Closer Name`. Metrics: `Sales Calls Showed`,
  `New Customers Closed`, `Close Rate`, `Revenue`. Sort by Revenue
  desc.

### Page 4 — Revenue & Retention

**Revenue mix scorecards:** `PIF`, `Revenue`, `Recurring Cash`,
`Recurring Cash Collected`, `Recurring Share` (%).

**Stacked bar** — `PIF` and `Recurring Cash` as two metrics over
`Year-Month`.

**Risk scorecards row:** `Customers Cancelled`, `Refund Amount`,
`Chargebacks`, `Total Expenses`.

**Profitability time series:** X: `Date`, Y: `Cash Collected`,
`Total Expenses`, `Net Cash`.

**Revenue by Offer table:** Dim: `Offer Name`. Metrics: `Revenue`,
`PIF`, `Recurring Cash`, `New Customers Closed`,
`Avg Customer Value`. Sort by Revenue desc.

---

## 5. Styling

- **Theme:** Report → Theme → Simple Dark or Constellation.
- **Brand colors:** Theme → Customize → set primary to your brand
  accent.
- **Scorecard style:** show comparison arrows, hide axis lines.
- **Title every chart** — top-left, bold, 14pt.
- **Page nav:** Insert → Page navigation (left sidebar style) so all 4
  pages are one click away.

---

## 6. Auto-refresh

Looker Studio caches Sheets data for ~15 minutes by default. To force
fresher updates:

1. Data source → **Data freshness** → set to **15 minutes** (minimum
   for Sheets).
2. Top right → **View** → users can hit **Refresh data** anytime.

The dashboard will pick up new rows automatically as long as you keep
adding them to the same tab with no schema changes.

---

## 7. Sharing

- **View-only link:** Share → "Anyone with the link can view".
- **Schedule emails:** File → Schedule email delivery → weekly PDF.

---

## 8. Common pitfalls

| Symptom | Fix |
|---|---|
| Scorecard shows `Infinity` | Numerator/denominator has a zero — wrap in `IF(... > 0, ..., NULL)`. |
| Dates not grouping monthly | Time series → Date dimension → granularity → Month. |
| New rows not appearing | Confirm the sheet's data range extends. Data source → Edit Connection → re-pick tab to expand range. |
| Filter shows blanks | Add a filter on the chart: `Business Name` IS NOT NULL. |
| Numbers look wrong after filter | A filter is hiding rows — check report-level filters at top. |

---

## 9. Estimated build time

Working through this guide: **30–45 minutes** the first time.

---

## 10. Next steps after the first build

- Add **annotations** (Insert → Text) on the Revenue trend for big
  events (launches, ad changes).
- Add a **goal line** on the revenue chart (Style → Reference line →
  constant) once you set a monthly target.
- Once a quarter, audit the calculated fields against actuals to catch
  formula drift.
