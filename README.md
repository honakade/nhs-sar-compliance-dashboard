# NHS SAR & IG Compliance Dashboard

![Dashboard Screenshot](dashboard_screenshot)

## Overview

A Power BI dashboard built to monitor Subject Access Request (SAR) compliance across NHS clinical teams, tracking SLA performance against the ICO-mandated 30-day response requirement.

Built to mirror real-world NHS Information Governance analyst work, using a 800-row synthetic dataset modelled on BHRUT IG processes.

---

## Background

Subject Access Requests are a legal right under UK GDPR and the Data Protection Act 2018. NHS trusts are required to respond within 30 days. Failure to comply risks ICO enforcement action and reputational damage to the trust.

This dashboard replicates the kind of compliance monitoring tool an IG analyst would build and maintain in a real NHS setting — tracking request volumes, SLA performance, team-level compliance, and overdue case detail.

---

## What it shows

- Weekly SAR request volumes and trend across 22 weeks
- Overall SLA compliance % against the ICO 30-day requirement (currently 69.8%)
- Average turnaround days by request type — Deceased requests flagged as slowest at ~28 days
- SLA compliance by team — IG Team identified as below the 80% trust threshold
- RAG status summary (Red / Amber / Green) across all 800 requests
- Overdue request detail log with days-open count and conditional formatting
- Quarter slicer for period filtering

---

## Key findings (Week 22, 2024)

| Metric | Value |
|---|---|
| Total requests YTD | 800 |
| SLA compliance | 69.8% |
| Avg turnaround | 24 days |
| Currently overdue | 23 |
| Worst performing team | IG Team |
| Slowest request type | Deceased (~28 days) |

---

## Data model

Star schema with 3 tables:

```
fact_requests (800 rows)
    ├── dim_type (5 rows)  — TypeID → RequestType, SLA_Days
    └── dim_team (5 rows)  — TeamID → TeamName, Department
```

A calculated DateTable was added in Power BI for time intelligence (WeekNum, Quarter, Month).

---

## Tools used

| Tool | Purpose |
|---|---|
| Excel | Synthetic dataset creation with formulas |
| Power Query | Data type corrections and transformations |
| Power BI Desktop | Data modelling, DAX measures, visuals |
| DAX | Calculated columns and measures |

---

## Key DAX measures

```dax
Total Requests = COUNTROWS(fact_requests)

Completed Requests = 
CALCULATE(COUNTROWS(fact_requests), fact_requests[Status] = "Completed")

On Time Requests = 
CALCULATE(
    COUNTROWS(fact_requests),
    fact_requests[Status] = "Completed",
    fact_requests[OnTimeSLA_Fixed] = "Yes"
)

SLA Compliance % = DIVIDE([On Time Requests], [Completed Requests], 0)

Overdue Requests = 
CALCULATE(COUNTROWS(fact_requests), fact_requests[Status] = "Overdue")

Avg Turnaround Days = 
AVERAGEX(
    FILTER(fact_requests, fact_requests[Status] = "Completed"),
    fact_requests[DaysOpen]
)
```

## Key DAX calculated column

```dax
RAG_Status = 
SWITCH(TRUE(),
    fact_requests[Status] = "Overdue", "Red",
    fact_requests[DaysOpen] >= 21 && 
    fact_requests[Status] = "Open",    "Amber",
    "Green"
)
```

---

## Dashboard design

- NHS blue colour scheme (`#003087`, `#0072CE`)
- RAG traffic light conditional formatting on Status and Days Open columns
- IG Team highlighted in red on SLA compliance chart
- 80% reference line on SLA by team chart
- 30 day reference line on avg turnaround chart
- `#F0F4F9` canvas background with white KPI card tiles

---

## Files

| File | Description |
|---|---|
| `SAR_Data.xlsx` | Synthetic dataset (800 rows) |
| `SAR_Dashboard.pbix` | Power BI report file |
| `SAR_Findings_Note.pdf` | Director-level compliance summary |
| `dashboard_screenshot.png` | Dashboard preview |
| `README.md` | This file |

---

## About

Built by Hannah Onakade — Data & BI Analyst with NHS Information Governance experience at BHRUT.

This project demonstrates end-to-end BI skills: data modelling, DAX, Power Query, conditional formatting, and the ability to translate compliance data into actionable insights for non-technical stakeholders.


