# HR Analytics & Employee Attrition Dashboard (Power BI)

## 📌 Project Overview
Employee turnover represents a significant cost to modern enterprises. This project involves the development of an interactive HR Analytics dashboard for Atlas Labs to analyze a 16.1% attrition rate across multiple variables. The solution provides a comprehensive view of workforce demographics, performance tracking, and flight risks, enabling HR leadership to make data-driven retention decisions.

## 💡 Business Impact & Key Takeaways
* **Targeted Retention:** Identified specific flight risks by analyzing attrition rates against tenure, travel frequency, and overtime requirements. 
* **Performance Visibility:** Built a performance tracking matrix that compares self-ratings against manager ratings alongside job, environment, and relationship satisfaction metrics.
* **Automated Reporting:** Engineered a scalable relational data model and dynamic DAX measures that automatically update core KPIs (Total Employees, Active vs. Inactive, Attrition Rate) as new data flows in.

## ⚙️ Technical Implementation & Data Modeling
The backend of this dashboard relies on a structured, relational data model rather than a flat file.
* **Schema Design:** Constructed a star schema connecting dimension tables (`DimEmployee`, `DimDate`, `DimEducationLevel`, `DimRatingLevel`) to the central fact table (`FactPerformanceRating`).
* **Custom Measures:** Utilized `COUNTAX` and conditional `IF` statements to calculate core business metrics, such as isolating inactive employees (`COUNTAX(DimEmployee, IF(DimEmployee[Attrition] = "Yes",1))`).

## 📊 Dashboard Workspaces
1. **Overview:** Executive summary displaying hiring trends over time, current attrition rates, and active employee distribution across the Technology, Sales, and Human Resources departments.
2. **Demographics:** Granular breakdown of the workforce by Age, Gender, Marital Status, and Ethnicity, mapped against average salary bands.
3. **Performance Tracker:** Interactive employee-level evaluation tool tracking historical review dates, satisfaction levels, and work-life balance.
4. **Attrition:** Deep-dive diagnostic view isolating attrition drivers, revealing spikes in turnover related to frequent business travel and mandatory overtime.

## 💻 Advanced DAX Showcase
To support advanced time-intelligence and dynamic forecasting, I scripted custom DAX tables and conditional columns.

**Dynamic Date Table Generation:**
```dax
DimDate = 
VAR _minYear = YEAR(MIN(DimEmployee[HireDate]))
VAR _maxYear = YEAR(MAX(DimEmployee[HireDate]))
VAR _fiscalStart = 4 

RETURN
ADDCOLUMNS(
    CALENDAR(
                DATE(_minYear,1,1),
                DATE(_maxYear,12,31)
),
"Year",YEAR([Date]),
"Year Start",DATE( YEAR([Date]),1,1),
"YearEnd",DATE( YEAR([Date]),12,31),
"MonthNumber",MONTH([Date]),
"MonthStart",DATE( YEAR([Date]), MONTH([Date]), 1),
"MonthEnd",EOMONTH([Date],0),
"DaysInMonth",DATEDIFF(DATE( YEAR([Date]), MONTH([Date]), 1),EOMONTH([Date],0),DAY)+1,
"YearMonthNumber",INT(FORMAT([Date],"YYYYMM")),
"YearMonthName",FORMAT([Date],"YYYY-MMM"),
"DayNumber",DAY([Date]),
"DayName",FORMAT([Date],"DDDD"),
"DayNameShort",FORMAT([Date],"DDD"),
"DayOfWeek",WEEKDAY([Date]),
"MonthName",FORMAT([Date],"MMMM"),
"MonthNameShort",FORMAT([Date],"MMM"),
"Quarter",QUARTER([Date]),
"QuarterName","Q"&FORMAT([Date],"Q"),
"YearQuarterNumber",INT(FORMAT([Date],"YYYYQ")),
"YearQuarterName",FORMAT([Date],"YYYY")&" Q"&FORMAT([Date],"Q"),
"QuarterStart",DATE( YEAR([Date]), (QUARTER([Date])*3)-2, 1),
"QuarterEnd",EOMONTH(DATE( YEAR([Date]), QUARTER([Date])*3, 1),0),
"WeekNumber",WEEKNUM([Date]),
"WeekStart", [Date]-WEEKDAY([Date])+1,
"WeekEnd",[Date]+7-WEEKDAY([Date]),
"FiscalYear",if(_fiscalStart=1,YEAR([Date]),YEAR([Date])+ QUOTIENT(MONTH([Date])+ (13-_fiscalStart),13)),
"FiscalQuarter",QUARTER( DATE( YEAR([Date]),MOD( MONTH([Date])+ (13-_fiscalStart) -1 ,12) +1,1) ),
"FiscalMonth",MOD( MONTH([Date])+ (13-_fiscalStart) -1 ,12) +1
)

```
**Automated Next Review Date Calculation:**
```dax

NextReviewDate = 
FORMAT (
    IF (
        MAX ( FactPerformanceRating[ReviewDate] ) = BLANK (),
        MAX ( DimEmployee[HireDate] ) + 365,
        MAX ( FactPerformanceRating[ReviewDate] ) + 365
    ),
    "mm/dd/yyyy"
)
