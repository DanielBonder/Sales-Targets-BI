# פרויקט BI: עמידה ביעדי מכירות חודשיים לעובדים

פרויקט BI מקצה לקצה (Power BI) שמדמה טעינה יומית של מכירות, חיבור ליעדים חודשיים, חישוב אחוז עמידה יחסי לפי התקדמות החודש, והתחשבות בימי אי-עבודה (סופ"ש/חגים).

## דמו ותמונות
צילומי מסך יופיעו בתיקיית `docs/images`. מומלץ להוסיף GIF קצר של ניווט בדשבורד.

## קבצי עזר (מהמרצה)
- `Example_External_Data.xlsx` — טבלת יעדים חודשיים/נתוני חוץ לדוגמה.
- `timedimension.xlsx` / `dim_date.xlsx` — עזר לבניית מימד זמן.
- `קבצי עזר ליצירת הפרוייקט.xlsx` — אוגד קבצי עזר.

> שימו אותם תחת `data/raw/` או `docs/demo/` לפי הצורך. אל תעלו נתונים רגישים.

## מבנה נתונים ומודל
- **FactDummySale** (מכירות בפועל) — מסונכרן לשנת 2025 ע״י הסטה בשנים.
- **DimEmployee** — מימד עובדים.
- **Dim_Date** — מימד זמן יומי עם סימון ימי עבודה/חגים.
- **Targets** (Example_External_Data) — יעדים חודשיים לפי עובד.

יחסים: `Dim_Date[Date] ↔ FactDummySale[NewOrderDate]`, `DimEmployee ↔ FactDummySale`, ו-`Targets(start_of_month) ↔ Dim_Date[StartOfMonth]` (או באמצעות טבלת גישור `factSalesVsTargets`).

## שלבי Power Query (תקציר לביצוע/שחזור)
1. **פרמטרים**
   - `P_FileName` (טקסט) — שם קובץ חלק מהפרויקט.
   - `P_Calendar_Start` (תאריך) — תאריך התחלה לבניית מימד הזמן.

2. **ייבוא טבלאות**: `FactDummySale`, `DimEmployee`, `Example_External_Data`.

3. **סימון ימי עבודה** (`full_working_days`):
   - שרשור `NOT_WORKING_DAY` ו-`HOLIDAY` לעמודה `merge_not_working_days` (ללא מפריד).
   - החלפת `XX` ב-`X` (אות גדולה).
   - הבטחת טיפוס `date` לשדה `date`.

4. **ניקוי/התאמת מכירות** (`FactDummySale`):
   - יצירת `NewOrderDate = Date.AddYears([OrderDate], 12)` ואז טיפוס `date`.
   - Merge עם `FULL_WORKING_DAYS` על `NewOrderDate`↔`date` ו-Semi-Join להשארת ימי עבודה בלבד (סינון `blank`).
   - סינון דינמי עד **היום הנוכחי**: `DateTime.Date(DateTime.LocalNow())` ב־Advanced Editor.

5. **Targets**:
   - `choose columns` להסרת עמודות מיותרות.
   - יצירת `start_of_month = #date([Year],[Month],1)` וטיפוס `date`.
   - מחיקת `Year`/`Month` המקוריים אם לא נדרשים.

6. **שלב עזר** `stg_target_actual_sales` (Reference מ-`FactDummySale`):
   - `Start of Month` ל-`NewOrderDate`.
   - `Group By` לפי `EmployeeKey` ו-`StartOfMonth` לסכום `SalesAmount`.
   - `All Rows` כדי להפיק `empDistinctDaysCount` (ספירת ימי עבודה ייחודיים בפועל).

7. **`factSalesVsTargets`**:
   - Reference מ-`Targets`, Join ל-`stg_target_actual_sales`.
   - Expand לשדות `SalesAmount`, `empDistinctDaysCount`.
   - החלפת `null` ב-`0` לשדות אלה.

8. **טעינה למודל**:
   - ביטול `Enable Load` לטבלאות ביניים: `full_working_days`, `stg_target_actual_sales`, `FactDummySale` (מקורית אם יש עותק מעובד), `Targets` (אם נשענים על טבלה משולבת).
   - טעינת מודל ובדיקת יחסים.

## מדדים (DAX) מוצעים
> צרפו את המדדים ישירות בקובץ PBIX.
- `Total Sales := SUM(FactDummySale[SalesAmount])`
- `Working Days in Month := COUNTROWS(FILTER(Dim_Date, Dim_Date[IsWorkingDay] = TRUE() && SAMEPERIODMONTH(Dim_Date[Date], SELECTEDVALUE(Dim_Date[Date]))))`
- `Elapsed Working Days := COUNTROWS(FILTER(Dim_Date, Dim_Date[IsWorkingDay] = TRUE() && Dim_Date[Date] <= TODAY() && MONTH(Dim_Date[Date]) = MONTH(TODAY()) && YEAR(Dim_Date[Date]) = YEAR(TODAY())))`
- `Relative Target Attainment := DIVIDE([Total Sales], [Target for Period] * DIVIDE([Elapsed Working Days], [Working Days in Month]))`
- `YoY Sales := CALCULATE([Total Sales], DATEADD(Dim_Date[Date], -1, YEAR))`

התאימו את שמות העמודות/טבלאות למה שיש בקובץ שלכם בפועל.

## איך לשחזר/להריץ
1. הניחו את קבצי הדמו תחת `data/raw/` ופתחו את `powerbi/report.pbix` (לא מצורף כאן).
2. עדכנו נתיבי מקורות ב-Power Query לפי המיקום המקומי.
3. רעננו את המודל ובדקו את דפי הדשבורד (KPI חודשי, עמידה יחסית, פילוחים לעובד/חודש).

## מבנה הרפו
ראו את עץ התיקיות ואת קבצי ההגדרה (`.gitignore`, `.gitattributes`, CI).

## רישיון
MIT (ראו `LICENSE`).

## יוצר/ת קשר
דניאל בונדר — הוסיפו LinkedIn/Email לפי הצורך.
