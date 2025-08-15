# Sales Targets BI Dashboard

דשבורד Power BI לניטור עמידה ביעדי מכירות חודשיים לעובדים, כולל חישוב עמידה יחסית לפי ימי עבודה, השוואה בין תקופות, והצגת KPI בזמן אמת.

---

## מבט כללי על הדשבורד
הדשבורד מציג:
- KPI של ימי עבודה שעברו מתוך החודש
- אחוז עמידה ביעד מכירות חודשי בהתחשב בהתקדמות החודש
- השוואת ביצועים לתקופות קודמות
- חיזוי קצב מכירות שנתי

![מבט כללי](docs/images/dashboard-overview.png)

---

## מבנה המודל ב-Power BI

### שדות הנתונים
![שדות הנתונים](docs/images/model-data-fields.png)

### קשרים בין הטבלאות
הקשרים בין הטבלאות במודל:
- `dimemployee[EmployeeKey]` ↔ `factsalesVStargets[EmployeeNum]`
- `dim_date[Date_Key]` ↔ `factsalesVStargets[start_of_month_22_23]`

![תרשים קשרים](docs/images/model-relationships.png)

---

## מקורות נתונים
- **FactDummySale** – טבלת מכירות בפועל (מותאמת לשנת 2025 ע"י הסטת שנים)
- **DimEmployee** – מימד עובדים
- **Dim_Date** – מימד זמן יומי עם סימון ימי עבודה וחגים
- **Targets** – יעדי מכירות חודשיים לכל עובד

---

## שלבי Power Query (תקציר)
1. ייבוא טבלאות: FactDummySale, DimEmployee, Example_External_Data
2. סימון ימי עבודה באמצעות שילוב עמודות NOT_WORKING_DAY ו-HOLIDAY
3. התאמת תאריכים ל-2025 בעזרת Date.AddYears
4. סינון לפי ימי עבודה בלבד והגבלה עד לתאריך הנוכחי (DateTime.LocalNow)
5. המרת יעדים חודשיים לתחילת החודש (start_of_month)
6. יצירת טבלאות עזר לקיבוץ מכירות חודשיות ומספר ימי עבודה בפועל
7. חיבור יעדים מול מכירות בפועל בטבלת factsalesVStargets
8. טיפול בערכי NULL והמרתם ל-0

---

## מדדי DAX עיקריים
```DAX
Total Sales := SUM(FactDummySale[SalesAmount])

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
  [Target Amount (Month)] * DIVIDE([Elapsed Working Days (to Today)], [Working Days in Month])
)

YoY Sales := CALCULATE([Total Sales], DATEADD(Dim_Date[Date], -1, YEAR))
