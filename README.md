# Student Journey CRM & BI System — Ireland Education & Visa Consultancy

An end-to-end CRM and business-intelligence project for an Ireland-based education and
visa consultancy that recruits international students into Irish universities. It models
the full recruitment lifecycle — from first enquiry to arrival and university commission —
and surfaces it through a Power BI dashboard suite.

> **Note:** All data is **synthetic** (generated with Mockaroo and processed in Python).
> Names, emails, phone numbers, tuition figures and commission rates are fictional and
> carry no real personal or contractual information.

---

## Business problem

The agency earns revenue from **university commissions**, not student fees, and its core
problem is **student-journey leakage**: many prospects enquire or attend a consultation but
never progress through application, offer, visa and arrival. This project makes that leakage
measurable — how many leads are lost, where, and what the surviving pipeline is worth.

## Architecture

```
Mockaroo (synthetic data)
  → Python / pandas  — cleaning, business validation, reconciliation
  → Clean CSVs
  → PostgreSQL       — relational model
  → Power BI + DAX   — six-dashboard suite
  → Business report
```

## Repository structure

```
.
├── README.md
├── requirements.txt
├── data/
│   ├── raw/                 # synthetic Mockaroo exports (inputs)
│   └── cleaned/             # the 7 validated *_clean.csv files (pipeline output)
├── notebooks/
│   ├── 01_data_cleaning.ipynb           # clean + validate the six source tables
│   ├── 02_relationship_consistency.ipynb # enforce journey/application/visa consistency
│   └── 03_finance.ipynb                 # derive the commission (finance) table
├── sql/                     # PostgreSQL schema + load scripts
├── dashboard/
│   └── Edu_Agency_Dashboard.pbix
└── docs/
    ├── project_context_pack.md          # full project specification
    └── business_report.pdf              # written analysis & findings
```

## Data model

Seven tables form a star-like model around the student journey (the spine).

| Table | Rows | Grain |
| --- | --- | --- |
| students | 1,000 | one per student |
| consultants | 5 | one per consultant |
| marketing_campaigns | 30 | one per campaign |
| student_journeys | 1,000 | one per lead |
| university_applications | 1,118 | one per application |
| visa_applications | 289 | one per visa case |
| university_commissions | 135 | one per approved-visa journey |

Relationships are single-direction. `student_journeys` is 1-to-many with
`university_applications`, and **1-to-0..1** with `visa_applications` and
`university_commissions` (not every journey reaches a visa or earns a commission).

## How to reproduce

```bash
pip install -r requirements.txt
```

Run the notebooks in order — `01` → `02` → `03`. Each reads from `data/raw/`,
validates with assertions, and writes the cleaned tables to `data/cleaned/`. Then load
the cleaned CSVs into PostgreSQL (`sql/`) and open the Power BI file in `dashboard/`.

Paths are relative to the repository root; set the base directory once at the top of
each notebook.

## Key findings

- **6% lead-to-arrival conversion.** The funnel runs 1,000 leads → 717 applicant
  journeys → 478 with an offer → 289 visa cases → 135 approved → 60 arrived. Leakage is
  spread across the funnel, not concentrated at the visa stage.
- **Admissions.** 52% application-level offer rate; 67% of applicants secure at least one
  offer (≈1.56 applications each).
- **Visa.** 90% approval rate among final decisions (135 approved / 15 refused).
- **Finance.** €174k earned commission, €203k forecast, with a **50.7% collection rate** —
  half of earned commission is still outstanding. The business effectively runs two
  pipelines: student conversion and commission collection.

## Data quality & validation

The cleaning pipeline uses reusable helpers (`standardize_columns`, `clean_text`,
`clean_id`, `parse_mixed_date`) and ends with assertion-based checks that block a bad save:
primary-key null/duplicate checks, foreign-key checks on every relationship, a composite
visa↔application↔journey check, and business-rule checks (no future dates, no
pre-application student holding an application).

## Limitations

Synthetic dataset with invented commission rates and tuition; no case-event/contact
history; no application or visa processing dates; no full operating-cost model (so
contribution after marketing is **not** profit); partial campaign attribution; a single
snapshot rather than a time series.

## Tech stack

Python (pandas, NumPy) · Jupyter · PostgreSQL · Power BI (DAX) · Mockaroo
