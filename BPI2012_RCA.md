# Root Cause Analysis — BPI Challenge 2012
## Loan Application Process | Process Mining Project

**Analyst:** mspessot  
**Tools:** Python (pandas, pm4py) · SQL (SQLite) · Celonis EMS  
**Dataset:** BPI Challenge 2012 — Dutch financial institution  
**Date:** May 2026

---

## 1. Process Overview

The BPI Challenge 2012 dataset contains the loan application process of a Dutch bank, covering **13,087 cases** and **164,506 events** across **23 activities** organized in three sub-processes:

- **A_ (Application):** Core application lifecycle — submission, pre-acceptance, acceptance, decline, cancellation
- **O_ (Offer):** Offer creation, selection, sending, and cancellation
- **W_ (Work items):** Manual tasks performed by bank employees — completing applications, following up on incomplete dossiers, validating offers

The process was analyzed end-to-end using Python for data preparation, SQL for quantitative KPI calculation, and Celonis for visual process mining and PQL-based metrics.

---

## 2. Key Findings

### 2.1 Process operates at two distinct speeds

| Segment | Cases | Duration |
|---|---|---|
| Fast track (rejection) | 51.9% | < 1 day |
| Standard evaluation | 30.5% | 7–30 days |
| Problematic cases | 10.2% | > 30 days |
| Critical cases | ~3% | > 60 days |

The median cycle time is **0.81 days**, while the mean is **8.61 days** — a 10x gap that reveals two fundamentally different process populations rather than a single cohesive flow. The **Celonis Process Overview confirmed a throughput time of 7 days**, consistent with Python calculations.

### 2.2 The "happy path" is a rejection, not an approval

The most frequent process variant — followed by **26.2% of all cases (3,429 cases)** — is:

```
A_SUBMITTED → A_PARTLYSUBMITTED → A_DECLINED → Process End
```

This variant completes in **0 days on average** (confirmed by both SQL and Celonis PQL: `Avg Days To Decline = 2.07 days` for all declined cases). One in four loan applications is rejected in the first two steps, before reaching full evaluation.

**Root cause hypothesis:** A significant share of applicants do not meet basic eligibility criteria (credit score, income threshold, documentation completeness). The absence of a pre-qualification filter at the entry point forces the process to handle high volumes of non-viable applications through the full intake flow before rejecting them.

**Business impact:** Operational cost of processing 3,429 cases that could potentially be filtered earlier. Estimated at ~26% of total process volume absorbed by early rejections.

### 2.3 Rework is concentrated in manual work items

**Rework Rate: 53.6%** of cases have at least one repeated activity (confirmed by PQL `REWORK_RATE_PCT` KPI).

Top rework activities identified in SQL analysis:

| Activity | Cases with rework | Avg repetitions |
|---|---|---|
| W_Nabellen incomplete dossiers | High | 7.6x |
| W_Nabellen offertes | 4,274 | 5.2x |
| W_Completeren aanvraag | 4,420 | 4.8x |

All top rework activities are W_ (manual work items). Zero rework was detected in automated A_ or O_ activities.

**Root cause:** Manual follow-up tasks repeat because the underlying trigger — incomplete or incorrect documentation submitted by applicants — is not resolved at the source. Each iteration of W_Nabellen incomplete dossiers represents a back-and-forth cycle between the bank employee and the applicant that could be reduced through:
- Structured digital intake forms with mandatory field validation
- Automated document completeness checks at submission
- Clear applicant-facing checklists before A_SUBMITTED

### 2.4 O_SENT_BACK is the primary bottleneck

The waiting time analysis (SQL `LAG()` function + Celonis Process Explorer) identified **O_SENT_BACK** as the activity with the highest average waiting time: **3.78 days**.

When an offer is returned, the process stalls for nearly 4 days before the next action is taken. This represents a queue bottleneck — likely cases waiting for manual reassignment or review after an offer is rejected by the applicant.

**Root cause:** Lack of automated routing after offer rejection. Cases entering O_SENT_BACK state require manual intervention to determine next steps, creating an unmanaged queue.

### 2.5 Process fragmentation indicates low standardization

**4,336 unique variants** for 13,087 cases — an average of 3 cases per variant. A well-standardized process would concentrate 70–80% of cases in 5–10 variants.

The top 10 variants cover only **53% of cases** (Celonis Variant Explorer), meaning nearly half of all loan applications follow a unique or near-unique path through the process.

**Root cause:** Absence of enforced process rules and decision gates. The high variant count suggests that case routing depends heavily on individual employee judgment rather than system-enforced business rules, leading to inconsistent handling and unpredictable outcomes.

---

## 3. Root Cause Summary

| Finding | Root Cause | Recommended Action |
|---|---|---|
| 26% early rejections | No pre-qualification filter | Implement eligibility screening before A_SUBMITTED |
| 53.6% rework rate | Incomplete documentation at intake | Mandatory digital intake forms with validation |
| O_SENT_BACK bottleneck (3.78d) | Manual routing after offer rejection | Automated case reassignment workflow |
| 4,336 variants | No enforced decision gates | Define and implement mandatory process checkpoints |
| 10.2% cases > 30 days | Unresolved rework loops | SLA alerts and escalation rules for stalled cases |

---

## 4. Metrics Consistency Across Tools

A key strength of this analysis is the **cross-tool validation** of findings:

| KPI | Python | SQL | Celonis PQL |
|---|---|---|---|
| Total cases | 13,087 | 13,087 | 13,087 |
| Avg cycle time | 8.61 days | — | 7 days (throughput) |
| Happy path % | 26% | 26% | 26.2% |
| Cases > 30 days | 10.3% | 10.3% | 10.2% |
| Avg events/case | ~12.6 | — | 12.57 |

The consistency across independent tools confirms the reliability of the findings and validates the data preparation pipeline built in Phase 1.

---

## 5. Conclusions

The BPI Challenge 2012 loan application process presents three systemic issues that compound each other:

1. **High intake volume of non-viable applications** drives unnecessary processing cost
2. **Documentation quality issues** create rework loops that extend cycle times for viable applications
3. **Manual routing and low process standardization** produce unpredictable outcomes and bottlenecks

Addressing the pre-qualification filter alone could reduce process volume by ~26%, directly impacting resource allocation for the remaining cases. Combined with structured intake validation, the rework rate could be significantly reduced, freeing capacity currently absorbed by W_ follow-up activities.

---

*Analysis performed as part of a Process Mining portfolio project using BPI Challenge 2012 open dataset.*  
*Tools: Python 3 · pandas · pm4py · SQLite · Celonis EMS Training Environment*
