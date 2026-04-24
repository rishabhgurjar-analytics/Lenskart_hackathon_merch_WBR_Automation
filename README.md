# Lenskart_hackathon_merch_WBR_Automation
This project is created to Automate the end to end data flow, visualisation and insighting for the Merchandising WBR


# Merch WBR Automation
### Lenskart Merchandising — Weekly Business Review powered by Claude AI + Power BI MCP

---

## What This Is

This project automates the end-to-end generation of the **Lenskart Merch Weekly Business Review (WBR)** — a report that previously took several hours of manual data pulling, formatting, and analysis.

With this setup, a single conversation with Claude produces:
- A **3-tab interactive HTML dashboard** with auto-generated insights, KPI cards, and charts
- A **2-tab Excel report** (Monthly + Weekly) with MoM%/WoW% colour-coded throughout
- All data pulled **live** from Power BI semantic models via MCP — no copy-paste, no manual exports

---

## Architecture

```
User (Claude.ai chat)
        │
        ▼
   Claude AI (Sonnet)
        │
        ├── Power BI MCP Server (XMLA/MCP)
        │       ├── Lenskart_DSR          → LK Business workspace
        │       ├── Cluade Sematic for Inventory → Digital Analytics workspace
        │       ├── Core Perf & Inv Dashboard   → Merch Reports workspace
        │       └── ContA Perf & Inv Dashboard  → Merch Reports workspace
        │
        └── Python / Node.js (file generation)
                ├── WBR_Dashboard.html    → 3-tab interactive HTML
                └── WBR_DD-Mon-YYYY.xlsx → 2-tab Excel report
```

**No ETL pipeline. No scheduled jobs. No manual data pulls.**  
Claude connects directly to live Power BI models via XMLA, runs DAX queries, processes results in Python, and writes output files — all within a single conversation.

---

## Data Models Connected

| Model | Workspace | Used For |
|---|---|---|
| `Lenskart_DSR` | LK Business | Revenue · Orders · ATV · NPS · Returns · New/Repeat · BOGO |
| `Cluade Sematic for Inventory` | Digital Analytics | WH stock · Store stock (Apr-2025 → present) |
| `Core Performance & Inventory Dashboard - Updated` | Merch Reports | Core PID availability · WOH · Spread @85% |
| `Continuity A Performance & Inventory Dashboard - Updated` | Merch Reports | Cont A PID availability · WOH · Spread @50% |

---

## Outputs

### 1. HTML Dashboard — `WBR_Dashboard.html`
Three tabs, dark-navy theme, no server required — opens in any browser.

**Tab 1 — Monthly View**
- 13 months (Apr-25 → current month★)
- Channel Level: Overall → Offline (Vintage/FY26) → Online (Web/Assisted) → HTO → Marketplace
- Merch Brand Category: 18 brands
- Inventory: WH + Store stock (EOMONTH snapshots from `Cluade Sematic for Inventory`)
- Auto-generated insights panel · KPI cards · Net Revenue trend chart · Brand share bars

**Tab 2 — Weekly View**
- 6 weeks ending current week★
- Same channel / brand / inventory structure
- Inventory uses Sunday (last-day-of-week) snapshots
- WoW% colour-coded green/red

**Tab 3 — Store Availability**
- Continuity A @50% spread (measure: `FAST - Test _new`)
- Core @85% spread (measure: `FAST - Test`)
- Monthly + Weekly tables per category
- Colour-coded: green ≥ threshold · amber within −15pp · red below
- Data: Cont A from Feb-2026, Core from Oct-2025

### 2. Excel Report — `WBR_DD-Mon-YYYY.xlsx`
- **Tab 1 — Monthly:** 13 months · Gross (₹L) · Net (₹L) · Net Qty · MoM% Gross · MoM% Net
- **Tab 2 — Weekly:** 6 weeks · same columns with WoW%
- Sections: Channel Level · Merch Brand Category · Inventory
- MoM%/WoW% colour-coded (green = growth, red = decline)

---

## Key Technical Rules

| Area | Rule |
|---|---|
| Inventory source | Always `Cluade Sematic for Inventory` for stock — never Cont A/Core models |
| WH stock column | `warehouse_quantity` — `website_quantity` = OFFLINE reserved stock (not online) |
| Inventory aggregation | Never SUM across days — use EOMONTH (monthly) or Sunday snapshot (weekly) |
| Revenue | Always show both Gross (₹L) and Net (₹L) |
| Date field | `Dim_Date_TimeLine[Date]` — never `Dim_Master_Data[create_date]` |
| Country filter | Do NOT apply `country = "IN"` at overall level — understates online revenue |
| All India | Always query directly from model — never average from regions |
| Assisted channel | `lead_team` "Chat" + "Pre_Sales" — always sum together |
| customer_tag_product | NUMERIC: 1 = New, 2 = Repeat |
| merch_brand_cat | In `Dim_Product_Master`, NOT `Dim_Brand_Category` |
| NPS date | Always add `USERELATIONSHIP(Dim_NPS_Master[nps_date], Dim_Date_TimeLine[Date])` |
| Availability measures | Cont A = `FAST - Test _new` · Core = `FAST - Test` — do not swap |
| Workspace names | Case-sensitive: "LK Business", "Digital Analytics", "Merch Reports" |

---

## Files in This Repository

| File | Purpose |
|---|---|
| `WBR_Dashboard.html` | Live interactive dashboard — updated in-place every WBR run |
| `WBR_DD-Mon-YYYY.xlsx` | Excel report generated fresh each run |
| `wbr_memory_dsr.md` | 21-section reference: all field mappings, DAX patterns, rules, edge cases |
| `wbr_prompt_dsr.md` | Steps 0–8 to rebuild WBR from scratch — paste at start of any new session |
| `Merch_WBR_README.docx` | One-page project read me (Word format) |

---

## How to Run

1. Open a new conversation in **Claude.ai** (Sonnet model)
2. Paste the contents of `wbr_prompt_dsr.md` at the start
3. Say: **"Update the Merch WBR"**
4. Claude will:
   - Reconnect to all 4 Power BI models
   - Pull fresh data (revenue, inventory, availability)
   - Update `WBR_Dashboard.html` in-place
   - Generate a new `WBR_DD-Mon-YYYY.xlsx`
   - Present both files

> **Note:** Sessions expire between conversations — reconnection is automatic when `wbr_prompt_dsr.md` is pasted.

---

## Pre-Share Checklist

Before sharing any output file, verify:
- [ ] All rows in both Excel tabs scanned for blanks
- [ ] Weekly sub-rows (Vintage, FY26, Web, Assisted, brands) populated via separate DAX queries
- [ ] Inventory numbers vary across months (not identical — that means data wasn't fetched)
- [ ] Weekly inventory section populated (Sunday snapshots)
- [ ] All partial months/weeks labelled with ★

---

## Stack

- **Claude AI** (Anthropic Sonnet) — orchestration, DAX generation, data processing, file generation
- **Power BI MCP Server** — XMLA connection to Fabric semantic models
- **Python** (`openpyxl`, `python-docx`) — Excel and Word file generation
- **Node.js** (`docx`) — Word document generation
- **HTML/CSS/JS** — Interactive dashboard (vanilla, no framework)

---

*Last updated: 24-Apr-2026*


