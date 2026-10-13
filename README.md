# Employee Attendance Dashboard (Power BI)

An interactive Power BI dashboard that turns raw employee login and logout records into a one-page view of attendance, lateness and working hours.

![Dashboard overview](images/dashboard.png)

> **Note:** The dataset is synthetic and created for practice. It does not represent real employees.

## Objective

Turn about 49,500 attendance records (250 employees, Jan to Oct 2026) into a single page that an HR team can read quickly and act on.

## Dashboard contents

- **KPI cards:** total employees, present today, absent, late arrivals, average working hours
- **Daily attendance %** trend with a 95% target line
- **Department attendance %** with green / amber / red thresholds
- **Login time distribution** by time slot
- **Top 5 late comers** table
- **Slicers:** date range, employee, department, shift

## Key findings (1 Jan to 12 Oct 2026)

- Overall attendance was **95.1%**, just above the 95% target
- **HR** had the lowest attendance (91.3%) and the highest late rate (23.8% of present days)
- The top 25 employees (10% of staff) accounted for about **26%** of all late logins
- Lateness rose in **July (23.4%)** and **August (22.1%)** versus about 18.5% in a typical month
- Weekday made little difference: late rates stayed between 18.8% and 19.7%
- 721 records had a missing logout punch

## Data model

Star schema with three tables:

| Table | Purpose |
|---|---|
| `Employees` | EmpID, Name, Department, Designation, Manager, Location, Shift, JoinDate, LateAfter |
| `Attendance` | One row per employee per working day (fact table) |
| `Calendar` | Date table, marked as the date table |

Relationships: `Employees[EmpID]` 1 to many `Attendance[EmpID]`, and `Calendar[Date]` 1 to many `Attendance[Date]`.

A `Shifts` sheet (start time, end time, grace minutes) is merged into `Employees` in Power Query. A `Holidays` sheet is included for reference.

## Business rules

| Rule | Definition |
|---|---|
| Present | LoginTime is not blank |
| Leave | LoginTime blank and Remarks = "Approved Leave" |
| Absent | LoginTime blank and no approved leave |
| Late | LoginTime is after shift start plus 30 minutes grace |
| Early leave | LogoutTime is before shift end |
| Shifts | General 09:00 to 18:00, Early 08:00 to 17:00 |

## Power Query steps

- Added `Status` (Present / Leave / Absent)
- Added `Working Hours` = logout minus login, in hours
- Merged shift timings into `Employees` and added `LateAfter` = shift start plus grace minutes

## Main DAX

```dax
Late Count = SUM(Attendance[Is Late])

Attendance % =
VAR total = COUNTROWS(Attendance)
VAR pres = CALCULATE(COUNTROWS(Attendance), Attendance[Status] = "Present")
RETURN IF(total = 0, BLANK(), DIVIDE(pres, total))

Present Today =
VAR d = MAX(Attendance[Date])
RETURN CALCULATE(COUNTROWS(Attendance), Attendance[Status] = "Present", Attendance[Date] = d)

Is Late (column) =
IF(Attendance[LoginTime] > RELATED(Employees[LateAfter]), 1, 0)

Day Status (column) =
SWITCH(TRUE(),
    Attendance[Status] = "Leave", "Leave",
    Attendance[Status] = "Absent", "Absent",
    Attendance[Is Late] = 1, "Late",
    "On Time")
```

## Repository structure

```
├── Employee_Attendance_Dashboard.pbix
├── data/
│   └── Employee_Attendance_Data.xlsx
├── images/
│   └── dashboard.png
└── README.md
```

## How to open

1. Download the `.pbix` file and open it in Power BI Desktop.
2. If the data does not load, go to **Home → Transform data → Data source settings → Change Source**, and point it to `data/Employee_Attendance_Data.xlsx` on your computer.
3. Click **Refresh**.
4. Set the date slicer to 1 Jan to 12 Oct 2026 to see the numbers above.

## Tools

Power BI Desktop, Power Query, DAX, Excel

## What I learned

Data modelling with a star schema, writing DAX measures that respond to slicers, and designing a clean one-page dashboard with consistent fonts, colours and layout.

## Limitations

- Synthetic data only
- "Today" cards show the latest date in the selected slicer range
- Attendance % counts approved leave days in the denominator
