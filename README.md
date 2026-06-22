<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=700&size=28&pause=1000&color=2E75B6&center=true&vCenter=true&width=700&lines=HR+Workforce+Analytics+Dashboard;Power+BI+%7C+DAX;50%2C000%2B+Employee+Records" alt="Typing SVG" />

<br/>

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![CSV](https://img.shields.io/badge/Dataset-5%20CSV%20Files-217346?style=for-the-badge&logo=microsoftexcel&logoColor=white)
![Video Record](https://img.shields.io/badge/Records-50%2C000%2B-FF6B6B?style=for-the-badge&logo=databricks&logoColor=white)

</div>

---

## 📊 Dashboard Preview

![HR Workforce Analytics Dashboard](dashboard_preview.png)

> 🎬 **A full video walkthrough explaining this project — the data model, DAX logic, and dashboard design — is included in this repository.**

---

## 📁 Project Structure

```
pr4/
├── 📂 Datasets/
│   ├── employee_data.csv                   (13,164 KB)
│   ├── employee_engagement_survey_data.csv  (1,113 KB)
│   ├── performance_data.csv                 (1,610 KB)
│   ├── recruitment_data.csv                 (9,718 KB)
│   └── training_and_development_data.csv    (4,355 KB)
├── 📂 Assets/
├── HR_Workforce_Analytics_Dashboard.pbix
└── HR_Workforce_Analytics_Report.pdf
```

---

## 🔢 Key Metrics

<div align="center">

| Metric | Value |
|--------|-------|
| 👥 Total Employees | 50,000 |
| ✅ Active Employees | 41,054 |
| 📉 Attrition Rate | 17.9% |
| ⚧ Gender Ratio | 55.3% |
| 📅 Avg Tenure | 4.69 Years |

</div>

---

## 🗂️ Data Model — Star Schema

The data model was built around **EmployeeMaster** as the central fact-dimension table, with four related tables and one standalone pipeline table.

```
DimDate ──────────────────────────────────────────────────────────┐
  [Date] ──1:Many──► EmployeeMaster[DateOfJoining]  (active)       │
  [Date] ──1:Many──► EmployeeMaster[DateOfTermination] (inactive)  │
                                                                    │
EmployeeMaster ◄──── Central Hub ─────────────────────────────────┘
    │ [Employee ID] ──1:Many──► Training[Employee ID]
    │ [Employee ID] ──1:Many──► EngagementSurvey[Employee ID]
    └─ [Employee ID] ──1:Many──► Performance[Employee ID]

Recruitment ── Standalone (no Employee ID — pre-hire applicants)
```

### Tables Loaded & Renamed

| Original File | Renamed Table | Role |
|---|---|---|
| `employee_data.csv` | `EmployeeMaster` | Core fact-dimension hub |
| `training_and_development_data.csv` | `Training` | Training records |
| `employee_engagement_survey_data.csv` | `EngagementSurvey` | Survey scores |
| `performance_data.csv` | `Performance` | Performance ratings |
| `recruitment_data.csv` | `Recruitment` | Standalone hiring pipeline |
| *(Blank Query)* | `DimDate` | Date dimension (2018–2024) |

---

## ⚙️ Steps Performed in Power BI Desktop

### Step 1 — Load Data & Power Query Transformations

#### 📦 Dataset

**Employee / HR Dataset All in One** — Published on Kaggle by Ravindra Singh Rana.  
License: Open Data Commons (free for education & research).

- Loaded all 5 CSV files via **Home → Get Data → Text/CSV**
- Renamed each table to its logical name (see table above)
- On `EmployeeMaster` in Power Query Editor:
  - Changed `DateOfJoining` → **Date** type
  - Changed `DateOfTermination` → **Date** type
  - Changed `Salary` → **Decimal Number**
  - Changed `Current Employee Rating` → **Whole Number**
  - Applied **Trim** on `DepartmentType` column to remove trailing spaces (fixes "Production " gap in slicers)
- Clicked **Close & Apply**

---

### Step 2 — Build DimDate via Blank Query

Created a custom date dimension spanning **2018–2024** using **Advanced Editor** with M code. The table includes 7 columns: `Date`, `Year`, `Quarter`, `Month_Num`, `Month_Name`, `Weekday`, `Year_Quarter`.

- Marked `DimDate` as the **Date Table** in Power BI
- Created **active** relationship: `DimDate[Date] → EmployeeMaster[DateOfJoining]`
- Created **inactive** relationship: `DimDate[Date] → EmployeeMaster[DateOfTermination]`

---

### Step 3 — Relationships in Model View

Established relationships using `Employee ID` as the bridge key:

| From | To | Cardinality | Direction | Status |
|------|----|-------------|-----------|--------|
| `EmployeeMaster[Employee ID]` | `Training[Employee ID]` | 1:Many | Single | Active |
| `EmployeeMaster[Employee ID]` | `EngagementSurvey[Employee ID]` | 1:Many | Single | Active |
| `EmployeeMaster[Employee ID]` | `Performance[Employee ID]` | 1:Many | Single | Active |

> `Recruitment` has **no relationship** — it tracks pre-hire applicants and contains no `Employee ID`.

---

### Step 4 — Dedicated Measures Table

Created a blank table named **`_Measures`** via **Home → Enter Data** to house all DAX measures in one organised location.

---

### Step 5 — DAX Calculated Columns (Row Context)

All calculated columns were added to `EmployeeMaster` via right-click → **New Column**:

| Column | Purpose |
|--------|---------|
| `Tenure_Years` | Years worked; uses `TODAY()` for Active employees, `DateOfTermination` for others — based on `EmployeeStatus` text, not blank-date logic |
| `Career_Level_Band` | Classifies tenure into bands (Senior / Mid / Leadership) using `SWITCH(TRUE())` |
| `Full_Name` | Concatenates `FirstName` & `LastName` using `&` operator |
| `Salary_Band` | Groups salary into 4 bands (A–D) for compensation distribution |
| `Is_Active` | Binary flag — `1` if `EmployeeStatus = "Active"`, else `0`; based on status text, not date blanks |

---

### Step 6 — DAX Measures (Filter Context)

All measures were created inside `_Measures` via right-click → **New Measure**:

| Measure | Description |
|---------|-------------|
| `Total_Headcount` | Total employee records — baseline KPI |
| `Active_Headcount` | Count of employees where `Is_Active = 1` |
| `Attrition_Rate %` | Percentage of employees who left |
| `Avg_Tenure` | Average years across active workforce |
| `Gender_Ratio %` | Female percentage of workforce |
| `Avg_Salary` | Mean salary (used in Compensation page) |
| `Avg_Engagement_Score` | Average from EngagementSurvey |
| `Avg_Satisfaction_Score` | Average satisfaction from survey |
| `Avg_WorkLife_Balance` | Average work-life balance score |
| `Training_Cost_Total` | Sum of training costs |
| `Avg_Training_Duration` | Average training days per employee |
| `Training_Completion_Rate %` | Percentage of completed trainings |
| `Total_Applicants` | Total recruitment pipeline count |
| `Offer_Acceptance_Rate %` | Offered ÷ Total Applicants |

---

### Step 7 — Dashboard Pages Built

#### 🟦 Page 1 — Workforce Overview
- **KPI Cards**: Total Employees · Active Employees · Attrition Rate % · Gender Ratio % · Avg Tenure
- **Performance Distribution by Department** — matrix visual
- **Hiring Trend** — line chart over time
- **Workforce by Career Level Band** — pie chart (Senior / Mid / Leadership)
- **Annual Hiring vs Same Period Last Year** — clustered bar chart
- **Active Employees by Department** — horizontal bar chart
- **Slicers**: Year · Department · Gender · EmployeeClass · Career Level Band

#### 🟦 Page 2 — Attrition & Compensation Analysis


**KPI Cards**: Terminated Employees · Avg Salary · Attrition Rate % · Training Investment · Avg Desired Salary (from Recruitment)
**Attrition Rate by** Department — horizontal bar chart showing attrition % ranked by department
**Employees by Gender** — donut chart of Total Headcount split by GenderCode
**Salary by Department** — clustered column chart of Total Salary Cost per department
**Salary Cost by Business Unit** — treemap visualising total salary spend across business units
**Headcount vs Avg Salary** — scatter chart plotting Active Headcount (X) against Avg Salary (Y) by department
**Department Salary Ranking** — table visual with columns: Department · Avg Salary · Active Headcount · Salary Rank (using Salary_Rank_Dept measure)
**Slicers**: Year · Department · Gender · Career Level Band · Salary Band


#### 🟦 Page 3 — Training & Performance


**KPI Cards: Avg Performance Rating** · Senior Headcount · Total Salary Cost · Avg Training Cost · High Performer %
**Employees by Training Outcome** — clustered column chart of Active Headcount by Training Outcome
**Training Type Distribution** — donut chart of employee count by Training Type
**Training Cost by Department** — clustered bar chart of Total Training Cost per department
**Training Cost vs Performance** — scatter chart plotting Total Training Cost (X) against Avg Performance Rating (Y) by department
**Performance Rating Distribution** — pie chart of Total Headcount by Performance Label
**High Performers by Department** — funnel chart of High Performers Count ranked by department
**Slicers**: Year · Department · Gender · Performance Label · Training Type


All visuals are cross-filtered — selecting any slicer or chart segment updates the entire page dynamically.

---

## 🛠️ Tools & Technologies

<div align="center">

![Power BI Desktop](https://img.shields.io/badge/Power%20BI%20Desktop-F2C811?style=flat-square&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX%20Language-0078D4?style=flat-square&logo=microsoft&logoColor=white)
![Power Query](https://img.shields.io/badge/Power%20Query%20(M)-217346?style=flat-square&logo=microsoftexcel&logoColor=white)
![Star Schema](https://img.shields.io/badge/Data%20Model-Star%20Schema-8B5CF6?style=flat-square)
![Kaggle Dataset](https://img.shields.io/badge/Dataset-Kaggle-20BEFF?style=flat-square&logo=kaggle&logoColor=white)

</div>

---

<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=16&pause=2000&color=2E75B6&center=true&vCenter=true&width=500&lines=Built+with+%E2%9D%A4%EF%B8%8F+by+Rensee+Gajipara+%7C+B.Tech+AI+%26+Data+Science+%7C+2027" alt="Footer" />

[![Author](https://img.shields.io/badge/Author-RENSEE%20GAJIPARA-2E75B6?style=for-the-badge&logo=github&logoColor=white)](https://github.com/RENSEE-GAJIPARA)
[![GitHub](https://img.shields.io/badge/GitHub-RENSEE--GAJIPARA-181717?style=for-the-badge&logo=github)](https://github.com/RENSEE-GAJIPARA)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-rensee--gajipara-0A66C2?style=for-the-badge&logo=linkedin)](https://linkedin.com/in/rensee-gajipara)

</div>
