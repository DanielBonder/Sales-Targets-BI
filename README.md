# Sales Targets BI Dashboard

A **Power BI** dashboard to monitor and analyze employees’ monthly sales performance against targets.  
Includes real-time KPIs, relative attainment based on working days, period-over-period comparisons, and annual pace forecasting.

<p align="left">
  <a href="https://github.com/DanielBonder/Sales-Targets-BI/raw/main/PowerBI/PROJECT.pbix">
    <img alt="Download PBIX" src="https://img.shields.io/badge/Download-PBIX-blue?logo=power-bi&logoColor=white" />
  </a>
  <img alt="Made with Power BI" src="https://img.shields.io/badge/Made%20with-Power%20BI-yellow?logo=power-bi&logoColor=white" />
  <img alt="Data Sources: Excel" src="https://img.shields.io/badge/Data%20Sources-Excel-green?logo=microsoft-excel&logoColor=white" />
  <img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-lightgrey" />
  <img alt="Last Commit" src="https://img.shields.io/github/last-commit/DanielBonder/Sales-Targets-BI" />
</p>

---

## 📊 Dashboard Overview
The dashboard provides:
- KPI showing the number of working days completed in the current month
- Monthly sales target attainment percentage adjusted to the month’s progress
- Comparison of performance against previous periods
- Annual sales pace forecast

![Dashboard Overview](images/model-Deshborad.png)

---

## 🧭 Features
- Relative target attainment that accounts for elapsed **working days**
- Clear separation between **actuals vs. targets**
- **Period comparisons** (month-over-month / year-over-year)
- **Clean data model** with employee and date dimensions
- Built entirely on **Excel** sources via **Power Query**

---

## 🗂 Power BI Data Model

### Data Fields
![Data Fields](images/model-data-fields.png)

### Table Relationships
Model relationships:
- `dimemployee[EmployeeKey]` ↔ `factsalesVStargets[EmployeeNum]`
- `dim_date[Date_Key]` ↔ `factsalesVStargets[start_of_month_22_23]`

![Relationships Diagram](images/model-relationships.png)

---

## 📂 Data Sources
All data sources are provided as **Excel files** located in `data/raw/` and loaded into Power BI via **Power Query**:

- **FactDummySale.xlsx** – Actual sales transactions (shifted to 2025 using `Date.AddYears`)
- **DimEmployee.xlsx** – Employee dimension table
- **Dim_Date.xlsx** – Daily date dimension with working day and holiday indicators
- **Targets.xlsx** – Monthly sales targets per employee

> If you cannot share real data, provide a synthetic/demo version with the same schema.

---

## 🔄 Power Query ETL (Summary)
1. Import Excel files: `FactDummySale.xlsx`, `DimEmployee.xlsx`, `Targets.xlsx`
2. Mark working days by merging `NOT_WORKING_DAY` and `HOLIDAY` columns
3. Adjust transaction dates to 2025 with `Date.AddYears`
4. Filter only working days and limit data to the current date (`DateTime.LocalNow`)
5. Convert monthly targets to first-of-month dates (`start_of_month`)
6. Create helper table `stg_target_actual_sales` for monthly aggregation of sales and distinct working days
7. Join targets with actual sales in `factsalesVStargets`
8. Replace `NULL` values with `0` for numeric fields

---

## 🧮 Key DAX Measures
```DAX
Total Sales :=
SUM(FactDummySale[SalesAmount])

Target Amount (Month) :=
VAR _start = MIN(Dim_Date[StartOfMonth])
RETURN CALCULATE(
    SUM(Targets[TargetAmount]),
    Dim_Date[StartOfMonth] = _start
)

Working Days in Month :=
CALCULATE(
    COUNTROWS(Dim_Date),
    Dim_Date[IsWorkingDay] = TRUE(),
    ALLEXCEPT(Dim_Date, Dim_Date[Year], Dim_Date[MonthNumber])
)

Elapsed Working Days (to Today) :=
CALCULATE(
    COUNTROWS(Dim_Date),
    Dim_Date[IsWorkingDay] = TRUE(),
    Dim_Date[Date] <= TODAY(),
    ALLEXCEPT(Dim_Date, Dim_Date[Year], Dim_Date[MonthNumber])
)

Relative Target Attainment :=
DIVIDE(
  [Total Sales],
  [Target Amount (Month)] *
  DIVIDE([Elapsed Working Days (to Today)], [Working Days in Month])
)

YoY Sales :=
CALCULATE([Total Sales], DATEADD(Dim_Date[Date], -1, YEAR))
```

---

## 🚀 Getting Started
1. Download the PBIX file using the button at the top.
2. Open the file with Power BI Desktop (latest version recommended).
3. Place Excel source files in the `data/raw/` directory (or update paths in Power Query).
4. Click Refresh to load the latest data.

---

## 📂 Project Structure
```
Sales-Targets-BI/
├─ PowerBI/
│  └─ PROJECT.pbix
├─ images/
│  ├─ model-Deshborad.png
│  ├─ model-data-fields.png
│  └─ model-relationships.png
├─ data/
│     ├─ FactDummySale.xlsx
│     ├─ DimEmployee.xlsx
│     ├─ Dim_Date.xlsx
│     └─ Targets.xlsx
└─ README.md
```

---

## 📄 License
This project is licensed under the MIT License – you are free to use, modify, and distribute with attribution.

---

## 👤 Author
Daniel Bonder  
📧 Email: your.email@example.com  
🔗 LinkedIn  

---

## 🏷 Tags
Power BI · DAX · Business Intelligence · Sales Analytics · Data Modeling
