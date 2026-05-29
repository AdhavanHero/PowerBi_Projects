# 📊 Investment Intelligence: Integrated Investment Readiness Dashboard

> **A Power BI Capstone Project** — Data-driven mapping of high-potential MSME zones across India using multi-source financial and digital payment datasets.

---

## 🗂️ Table of Contents

- [Project Overview](#-project-overview)
- [Problem Statement](#-problem-statement)
- [Solution Architecture](#-solution-architecture)
- [Dashboard Walkthrough](#-dashboard-walkthrough)
- [Datasets](#-datasets)
- [Data Model & DAX Measures](#-data-model--dax-measures)
- [Investment Score — Methodology](#-investment-score--methodology)
- [How to Use This Report](#-how-to-use-this-report)
- [Repository Structure](#-repository-structure)
- [How to Share / Reproduce](#-how-to-share--reproduce)
- [Key Insights](#-key-insights)
- [Use Cases](#-use-cases)
- [Tech Stack](#-tech-stack)
- [Author](#-author)

---

## 🎯 Project Overview

India's MSME sector contributes over **30% of GDP** and employs more than 110 million people — yet investors, banks, and policymakers routinely struggle to identify *where* to deploy capital most effectively. Data on MSME registrations, bank credit, and digital payment activity exists, but it lives in **silos**.

This dashboard bridges that gap by integrating three independent government datasets into a single **Investment Readiness Score** — a weighted index that ranks every state and district across India by their economic maturity and investment potential.

---

## ❗ Problem Statement

| Challenge | Description |
|-----------|-------------|
| **Fragmented Data** | MSME registrations, RBI credit data, and UPI/NFS payment volumes exist independently with no unified view |
| **No Unified Metric** | Investors lack a single score to distinguish "investment-ready" zones from underdeveloped ones |
| **Credit Gap Blindspot** | Credit disbursement patterns don't visibly align (or misalign) with enterprise density |
| **Information Asymmetry** | Banks can't easily spot underserved districts with high MSME density but low credit penetration |

---

## 🏗️ Solution Architecture

```
Raw Data Sources
      │
      ├── MSME Registrations (UDYAM Portal)   → 788 districts, 36 states
      ├── RBI Credit Data (SCB, March 2025)   → District + Population Group
      ├── UPI Transactions (NPCI)             → State-wise, FY 2023–26
      └── NFS Transactions (NPCI)             → District-wise, FY 2020–26
                    │
                    ▼
           Power BI Data Model
        (Star Schema, Relationships)
                    │
                    ▼
         5-Page Interactive Dashboard
    ┌─────────────────────────────────────────┐
    │  Page 1: MSME Registrations Overview    │
    │  Page 2: RBI Credit Analysis            │
    │  Page 3: UPI Pulse (Digital Payments)   │
    │  Page 4: NFS District-Level View        │
    │  Page 5: Investment Score / Maturity    │
    └─────────────────────────────────────────┘
```

---

## 🖥️ Dashboard Walkthrough

### Page 1 — MSME Registrations

**Purpose:** Understand the geographic distribution and composition of registered enterprises.

**Key Visuals:**
- Choropleth map of total MSMEs by state/district
- Stacked bar: Micro / Small / Medium breakdown by state
- KPI cards: Total MSMEs, Micro %, Small %, Medium %

**Key DAX Measures:**
```dax
Total MSMEs = SUM(MSME_Registrations[Total])

Micro % = DIVIDE(SUM(MSME_Registrations[Micro]), [Total MSMEs])
```

> 💡 **What to look for:** States with a very high Micro % (>90%) flag regions where formalisation drives are most needed and where micro-lending products can be targeted.

---

### Page 2 — RBI Credit Analysis

**Purpose:** Evaluate how institutional credit aligns with enterprise density.

**Key Visuals:**
- District-level map: Credit-per-MSME ratio (hot/cold spots)
- Bar chart: Total Credit (₹ Cr) by state
- Scatter plot: MSME count vs. Credit disbursed (reveals under-served outliers)
- Rural vs. Urban credit split

**Key DAX Measures:**
```dax
Total Credit (Cr) = SUM(RBI_Credit[Total_Credit_Cr])

Credit per MSME (Lakh) = [Total Credit (Cr)] / [Total MSMEs] * 100

Rural Credit =
    CALCULATE([Total Credit (Cr)], Dim_PopGroup[Group] = "RURAL")
```

> 💡 **What to look for:** Districts in the **top-right quadrant** of the scatter (high MSMEs, high credit) are thriving. Districts in the **bottom-right** (high MSMEs, low credit) are your highest-priority cold spots — ripe for bank expansion or NBFC products.

---

### Page 3 — UPI Pulse

**Purpose:** Measure digital payment adoption as a proxy for economic formalisation.

**Key Visuals:**
- Line chart: UPI volume trend over time (FY 2023–24 → 2025–26) by state
- Ranked bar: Top states by UPI transaction value (₹ Cr)
- State slicer for drill-down

**Dataset coverage:** 37 states × 3 fiscal years × monthly granularity (1,258 rows)

**Key DAX Measures:**
```dax
UPI Volume (Mn) = SUM(UPI_Pulse[Volume_Mn])

UPI Value (Cr) = SUM(UPI_Pulse[Value_Cr])
```

> 💡 **What to look for:** States showing steep UPI growth curves signal a rapid informal-to-formal economic transition — an early-mover signal for fintech investors.

---

### Page 4 — NFS District-Level View

**Purpose:** Analyse ATM/POS network transaction volumes at district granularity.

**Key Visuals:**
- District-level map: NFS transaction volume heatmap
- YoY growth chart by state
- Top 20 districts ranked by NFS volume

**Dataset coverage:** 38 states, 260 districts, FY 2020–21 → 2025–26 (11,280 rows)

**Key DAX Measures:**
```dax
NFS Volume (Mn) = SUM(NFS_Districts[Volume_Mn])

NFS YoY Growth =
    VAR Current  = [NFS Volume]
    VAR Previous = CALCULATE([NFS Volume], SAMEPERIODLASTYEAR('Date'[Date]))
    RETURN DIVIDE(Current - Previous, Previous)
```

> 💡 **What to look for:** Districts with positive NFS YoY growth but low UPI adoption indicate regions still reliant on card/ATM infrastructure — a gap where digital wallet products can expand.

---

### Page 5 — Investment Score (Maturity Index)

**Purpose:** Synthesise all three data dimensions into a single, comparable ranking.

**Scoring Formula:**

```
Investment Score = (MSME Score × 0.40) + (Credit Score × 0.40) + (Digital Score × 0.20)
```

Where each component is min-max normalised to a 0–1 scale.

**Tier Classification:**

| Tier | Score Range | Interpretation |
|------|-------------|----------------|
| **Tier 1** | > 0.75 | Investment-Ready — strong across all dimensions |
| **Tier 2** | 0.50 – 0.75 | Emerging — strong in 1–2 dimensions, gaps remain |
| **Tier 3** | 0.25 – 0.50 | Developing — nascent ecosystem, selective opportunity |
| **Tier 4** | < 0.25 | Early Stage — needs infrastructure/policy intervention |

**Key DAX Measures:**
```dax
MSME Score   = NORMALIZE(SUM(MSME_Registrations[Total]))
Credit Score = NORMALIZE([Credit per MSME])
Digital Score = NORMALIZE([UPI Volume] + [NFS Volume])

Investment Score =
    ([MSME Score] * 0.4) + ([Credit Score] * 0.4) + ([Digital Score] * 0.2)

Investment Tier =
    IF([Investment Score] > 0.75, "Tier 1",
    IF([Investment Score] > 0.50, "Tier 2",
    IF([Investment Score] > 0.25, "Tier 3", "Tier 4")))
```

> 💡 **What to look for:** Sort states by Investment Score. Tier 4 states are candidates for government subsidy and digital literacy programs. Tier 1 states are targets for private equity and NBFC expansion.

---

## 📁 Datasets

| File | Source | Rows | Coverage | Key Columns |
|------|--------|------|----------|-------------|
| `MSME_District_Wise_MSME_UDYAM.csv` | UDYAM / MoMSME | 788 | 36 states, 783 districts | `state_name`, `district_name`, `micro`, `small`, `medium`, `total` |
| `DISTRICT_AND_POPULATION_GROUP-WISE_CREDIT_OF_SCB_s_MARCH_2025.xlsx` | RBI | — | Districts + Population Groups (Rural/Urban/Metro) | Credit amount outstanding (₹ Crore) |
| `year-month-and-state-wise-volume-and-value-of-upi-transaction-statistics.csv` | NPCI / data.gov.in | 1,258 | 37 states, FY 2023–26 | `fiscal_year`, `month`, `state`, `volume`, `value` |
| `digital-payments-and-transactions-...-nfs-system.csv` | NPCI / data.gov.in | 11,280 | 38 states, 260 districts, FY 2020–26 | `fiscal_year`, `month`, `state`, `district_as_per_lgd`, `volume` |

All datasets are sourced from **open government data portals** (data.gov.in, RBI, NPCI) and are freely redistributable for academic/analytical purposes.

---

## 🔗 Data Model & DAX Measures

The Power BI model uses a **star schema**:

```
Fact Tables:
  ├── MSME_Registrations   (district grain)
  ├── RBI_Credit           (district + population group grain)
  ├── UPI_Pulse            (state + month grain)
  └── NFS_Districts        (district + month grain)

Dimension Tables:
  ├── Dim_Geography        (State → District hierarchy)
  ├── Dim_PopGroup         (Rural / Semi-Urban / Urban / Metro)
  └── Date                 (fiscal year calendar)
```

Relationships are set via `lg_dt_code` (LGD district code) as the common geographic key across MSME, NFS, and Credit tables.

---

## 🚀 How to Use This Report

### Prerequisites
- **Power BI Desktop** (free) — [download here](https://powerbi.microsoft.com/desktop/)
- Windows OS (Power BI Desktop is Windows-only; Mac users can use a VM or Power BI Service)

### Steps

```bash
# 1. Clone / download this repository
git clone https://github.com/<your-username>/investment-intelligence-dashboard.git
cd investment-intelligence-dashboard

# 2. Place data files in the /data folder (see Repository Structure below)

# 3. Open the report
#    Double-click Capstone_project.pbix  OR
#    Open Power BI Desktop → File → Open → Capstone_project.pbix

# 4. If prompted for data source paths, click Transform Data →
#    update file paths to point to your local /data folder

# 5. Click Refresh to reload all datasets
```

### Refreshing Data
If you update any CSV/XLSX file, click **Home → Refresh** in Power BI Desktop to reload. Alternatively, publish to Power BI Service to enable scheduled refresh.

---

## 📂 Repository Structure

```
investment-intelligence-dashboard/
│
├── README.md                          ← You are here
│
├── Capstone_project.pbix              ← Main Power BI report file
│
├── docs/
│   └── Power_Bi_problem_statement.docx  ← Original problem statement & DAX reference
│
└── data/
    ├── MSME_District_Wise_MSME_UDYAM.csv
    ├── DISTRICT_AND_POPULATION_GROUP-WISE_CREDIT_OF_SCB_s_MARCH_2025.xlsx
    ├── year-month-and-state-wise-volume-and-value-of-upi-transaction-statistics.csv
    └── digital-payments-and-transactions-...-nfs-system.csv
```

> ⚠️ **Note on `.pbix` files:** Power BI files embed the data model and visuals together. If you modify the source CSVs, re-open the `.pbix` and refresh. Do **not** rename or move source files without updating the connection strings inside Power BI.

---

## 🌐 How to Share / Reproduce

### Option A — Share the `.pbix` directly (simplest)
Upload `Capstone_project.pbix` to GitHub. Anyone with Power BI Desktop can open it.
The data is embedded inside the `.pbix` by default, so no separate files are needed for viewing.

### Option B — Publish to Power BI Service (for live sharing)
1. Open the `.pbix` in Power BI Desktop
2. Click **Home → Publish → Select workspace**
3. Share the report link with stakeholders (view-only, no Power BI account needed with public sharing)

### Option C — Export to PDF (for presentations)
In Power BI Desktop: **File → Export → Export to PDF**
This creates a static snapshot of all dashboard pages.

### Option D — Embed in a website or portfolio
After publishing to Power BI Service, use **File → Embed Report → Website or portal** to get an iframe embed code.

### Sharing the Datasets on GitHub
Since all source data files are open government data, they can be committed directly to the repo:

```bash
git add data/
git commit -m "Add source datasets (open govt data — UDYAM, RBI, NPCI)"
git push origin main
```

If any file exceeds GitHub's 100MB limit (the NFS CSV at ~11K rows is safe), use [Git LFS](https://git-lfs.github.com/):
```bash
git lfs track "*.csv"
git add .gitattributes
```

---

## 💡 Key Insights

- **Micro enterprises dominate:** Over 95% of all MSME registrations nationwide are Micro-category, highlighting the scale of informal-to-formal transition still needed.
- **Credit cold spots exist:** Several districts in Central and North-East India have significant MSME density but credit-per-MSME ratios well below the national average.
- **UPI is growing fastest in Tier-2 cities:** State-level UPI trends show accelerating adoption in states like Rajasthan, MP, and Odisha — not just the metros.
- **NFS growth signals infrastructure readiness:** Districts with strong NFS YoY growth but low UPI uptake are transition zones — attractive for digital payment infrastructure investment.

---

## 🏦 Use Cases

### For Banks & NBFCs
- Use the **RBI Credit** page to find districts with high MSME density but low credit penetration — prime territory for new branch openings or MSME loan products
- Use the **Digital Score** as a creditworthiness proxy for informal businesses entering the formal sector

### For Government & Policymakers
- Target **Tier 4 states** in the Investment Score for subsidies, digital literacy drives, and UDYAM registration campaigns
- Track **Micro %** over time as a KPI for formalisation policy success

### For Private Equity & Venture Capital
- Identify **Tier 1 districts** for launching B2B fintech, supply chain finance, or embedded lending products
- Use **UPI volume trends** to forecast which states are fastest-growing for tech-enabled financial services

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| **Power BI Desktop** | Data modelling, DAX measures, interactive visualisations |
| **Power Query (M)** | Data cleaning, transformation, column standardisation |
| **DAX** | Custom measures: normalisation, weighted scoring, tier classification |
| **Excel / CSV** | Source data formats |
| **Python (optional)** | EDA and data validation prior to Power BI ingestion |

---

## 👤 Author

**[Adhavan Ponram]**
- GitHub: [@AdhavanHero](https://github.com/AdhavanHero)
- LinkedIn: [Adhavan_Ponram](https://www.linkedin.com/in/adhavan-ponram/)

*This project was built as a capstone to demonstrate integrated business intelligence using real Indian government datasets. All source data is publicly available.*

---

## 📄 Data Sources & Attribution

| Dataset | Publisher | Licence |
|---------|-----------|---------|
| MSME District-wise UDYAM Registrations | Ministry of MSME, Govt. of India | Open Government Data (OGD) |
| District & Population Group-wise Credit of SCBs, March 2025 | Reserve Bank of India (RBI) | RBI Open Data |
| State-wise UPI Transaction Statistics | NPCI via data.gov.in | Open Government Data (OGD) |
| NFS District-wise Transaction Volumes | NPCI via data.gov.in | Open Government Data (OGD) |

---

*Made with ❤️ and Power BI*

