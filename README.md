Process Mining — BPI Challenge 2012
End-to-end process analysis of a Dutch bank loan application process
![Python](https://img.shields.io/badge/Python-3.x-blue) ![SQL](https://img.shields.io/badge/SQL-SQLite-lightgrey) ![Celonis](https://img.shields.io/badge/Celonis-EMS-purple) ![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
---
Project Summary
This project performs a full end-to-end Process Mining analysis of the BPI Challenge 2012 dataset — a real loan application process from a Dutch financial institution. The analysis connects three tools in a coherent analytical narrative: Python for data preparation, SQL for quantitative KPI calculation, and Celonis EMS for visual process mining and PQL-based metrics.
Key findings at a glance:
26.2% of cases follow a fast-rejection path (happy path = decline in 0 days)
53.6% rework rate concentrated exclusively in manual work items
O_SENT_BACK identified as primary bottleneck with 3.78 days avg waiting time
4,336 unique variants indicating low process standardization
Cross-tool validation confirms consistency across all three analytical layers
---
Dataset
Source: BPI Challenge 2012 — 4TU Research Data  
Format: XES (converted to CSV for analysis)  
Size: 262,200 events · 13,087 cases · 23 activities  
Process: Loan application lifecycle at a Dutch bank  
Sub-processes:
`A_` Application lifecycle (submission, acceptance, decline)
`O_` Offer management (creation, sending, cancellation)
`W_` Manual work items (follow-up, validation, completion)
---
Project Structure
```
├── Proyecto_PM.ipynb          # Main notebook — Phases 1 and 2
│
├── data/
│   ├── event_log_clean.csv    # Cleaned event log (Phase 1 output)
│   └── cycle_times.csv        # Cycle time per case (Phase 1 output)
│
├── sql_outputs/
│   ├── sql_overview.csv
│   ├── sql_frecuencia_actividades.csv
│   ├── sql_actividades_iniciales.csv
│   ├── sql_actividades_finales.csv
│   ├── sql_cycle_time_por_estado.csv
│   ├── sql_distribucion_duracion.csv
│   ├── sql_bottleneck.csv
│   ├── sql_rework.csv
│   ├── sql_rework_rate.csv
│   └── sql_variantes.csv
│
└── docs/
    └── BPI2012_RCA.md         # Root Cause Analysis document
```
---
Tech Stack
Tool	Purpose
Python · pandas · pm4py	XES parsing, data cleaning, cycle time calculation
SQLite · pandas	KPI queries, bottleneck detection, variant analysis
Celonis EMS	Process Explorer, Variant Explorer, PQL custom KPIs
Matplotlib	Cycle time distribution, activity frequency charts
---
Phase 1 — Data Preparation (Python)
Notebook: `Proyecto_PM.ipynb`
Loaded BPI 2012 XES file using pm4py (262,200 raw events)
Filtered lifecycle transitions — kept only `COMPLETE` events (164,506 events)
Renamed and typed columns for downstream compatibility
Calculated cycle time per case (min/max timestamp delta)
Exported clean tables for SQL and Celonis ingestion
Key output:
Metric	Value
Events after filtering	164,506
Cases	13,087
Median cycle time	0.81 days
Mean cycle time	8.61 days
Max cycle time	91.4 days
---
Phase 2 — SQL Analysis (SQLite)
Notebook: `Proyecto_PM.ipynb` (Phase 2 section)
Queries built on the cleaned event log loaded into SQLite:
Query	Finding
Process overview	13,087 cases · 164,506 events · 23 activities
Activity frequency	W_Completeren aanvraag leads with 23,967 events
Initial/final activities	100% start at A_SUBMITTED · 7,635 end at A_DECLINED
Cycle time by final state	Declined cases avg 0d · Complex cases up to 91d
Duration distribution	51.9% resolve < 1 day · 10.2% exceed 30 days
Bottleneck detection	O_SENT_BACK: 3.78 days avg waiting time
Rework analysis	W_Nabellen incomplete dossiers: 7.6x avg repetitions
Variant analysis	4,336 unique variants · Top 10 cover 53% of cases
---
Phase 3 — Celonis EMS
Environment: Celonis Training EMS (`training.celonis.cloud`)  
Data Model: `BPI2012_MODEL` — single activity table, event log configured
Process Explorer
Full process map rendered with 100% of activities and 70.5% of connections visualized. Confirms the bimodal process structure identified in Python.
Variant Explorer
2,312 variants identified
Variant #1 (happy path): `A_SUBMITTED → A_PARTLYSUBMITTED → A_DECLINED` — 26% of cases, 0.0 days
Process Overview KPIs (auto-generated)
Cases per day: 79
Events per day: 991
Throughput time: 7 days
Custom PQL KPIs
KPI	Formula approach	Result
Avg Days To Decline	`CALC_THROUGHPUT` (A_SUBMITTED → A_DECLINED)	2.07 days
Rework Rate %	`PU_COUNT` + `DOMAIN_TABLE` + `CASE WHEN`	53.6%
Avg Amount Declined	`AVG` + `CASE WHEN` activity filter	€12,191
Cases Over 30 Days %	`CALC_THROUGHPUT` + `CASE WHEN` threshold	10.2%
Avg Events Per Case	`AVG` + `PU_COUNT` + `DOMAIN_TABLE`	12.57
---
Phase 4 — Root Cause Analysis
Full RCA document available at `docs/BPI2012_RCA.md`
Top findings
1. 26% early rejections — missing pre-qualification filter  
One in four applications is rejected before full evaluation. A pre-screening step at intake could reduce process volume by ~26%.
2. 53.6% rework rate — documentation quality at intake  
All rework concentrates in manual W_ activities. Root cause: incomplete documentation submitted by applicants triggers repeated follow-up cycles.
3. O_SENT_BACK bottleneck — manual routing after offer rejection  
3.78 days average stall when an offer is returned. Automated routing would eliminate this queue.
4. 4,336 variants — no enforced process rules  
High variant count reflects case routing driven by individual judgment rather than system rules.
Cross-tool validation
KPI	Python	SQL	Celonis
Total cases	13,087	13,087	13,087
Happy path %	26%	26%	26.2%
Cases > 30 days	10.3%	10.3%	10.2%
Avg events/case	~12.6	—	12.57
---
How to Run
```bash
# 1. Clone the repository
git clone https://github.com/YOUR_USERNAME/bpi2012-process-mining

# 2. Install dependencies
pip install pandas pm4py matplotlib

# 3. Download the dataset
# https://data.4tu.nl/articles/dataset/BPI_Challenge_2012/12689204
# Place BPI_Challenge_2012.xes in the root folder

# 4. Open and run the notebook
jupyter notebook Proyecto_PM.ipynb
```
---
About
Built as a portfolio project demonstrating Process Mining competencies across Python, SQL, and Celonis EMS.  
Dataset: BPI Challenge 2012 (open access, 4TU Research Data).
