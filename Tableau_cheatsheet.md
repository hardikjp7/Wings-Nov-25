# TABLEAU EXAM CHEAT SHEET 📝

---

## **MODULE 1 & 2: FOUNDATIONS & BASIC VISUALIZATIONS**

### **Core Concepts**
```
DIMENSIONS (Blue Pills)
- Categorical data (Text, Dates, Boolean)
- Create headers/labels
- Discrete (separate categories)

MEASURES (Green Pills)  
- Numeric data that can be aggregated
- Create axes
- Continuous (ranges)

Blue Pill = Discrete = Headers
Green Pill = Continuous = Axis
```

### **Marks Card Hierarchy**
```
1. Position (Rows/Columns) - Most important
2. Color - Secondary insight
3. Size - Magnitude
4. Label - Exact values
5. Detail - Add granularity without visual change
6. Tooltip - Hover information
```

### **Chart Selection Quick Guide**
```
Compare categories → BAR CHART
Trend over time → LINE CHART
Part-to-whole (few) → PIE/TREE MAP
Correlation → SCATTER PLOT
Distribution → BOX PLOT/HISTOGRAM
Actual vs Target → BULLET GRAPH
Time comparison → GANTT CHART
Geographic → MAP
```

---

## **MODULE 3: WORKING WITH DATA**

### **Aggregation Types**
```
SUM     - Total (most common)
AVG     - Mean value
MEDIAN  - Middle value (less affected by outliers)
COUNT   - Number of records
COUNTD  - Unique count (CRITICAL for customers, orders)
MIN/MAX - Lowest/highest
```

### **Granularity Rule**
```
Granularity = ALL dimensions in view
More dimensions = MORE detail = DIFFERENT aggregations

Example:
View 1: Sales by Year → 4 values
View 2: Sales by Year, Month → 48 values
View 3: Sales by Year, Month, Product → 1000s of values

Same data, different granularity = different results!
```

### **Sorting**
```
Alphabetic - A to Z
Manual - Drag to reorder
By Field - Sort by measure (common)
  → Right-click axis → Sort → Field (Aggregated measure)
```

### **Groups vs Sets vs Hierarchies**
```
GROUPS
- Combine members manually
- Static
- Multiple categories
- Use: Simplify, combine small categories

SETS
- Dynamic (conditions) or Manual
- Binary: In/Out only
- Updates automatically
- Use: Top N, comparisons, highlighting

HIERARCHIES
- Drill-down structure
- Broad → Specific (Country → State → City)
- Use: Navigation, organized structure
```

---

## **MODULE 4: CALCULATIONS** ⭐⭐⭐

### **Row-Level vs Aggregate** (MOST TESTED!)
```
ROW-LEVEL
[Profit] / [Sales]
- Calculates per row FIRST
- Then aggregated in view (SUM, AVG, etc.)
- No AGG functions in formula

AGGREGATE  
SUM([Profit]) / SUM([Sales])
- Aggregates FIRST
- Then calculates
- Has AGG functions (SUM, AVG, COUNT)

CRITICAL: They give DIFFERENT results!
Row-level averaged ≠ Aggregate division
```

### **Key Functions**

**STRING FUNCTIONS**
```
UPPER([Text])           - Uppercase
LOWER([Text])           - Lowercase
LEFT([Text], 3)         - First 3 characters
RIGHT([Text], 4)        - Last 4 characters
MID([Text], 5, 3)       - Starting position 5, take 3
TRIM([Text])            - Remove whitespace
SPLIT([Text], " ", 1)   - Split by space, 1st part
CONTAINS([Text], "Pro") - Returns TRUE/FALSE
REPLACE([Text], "-", "") - Replace - with nothing
```

**NUMBER FUNCTIONS**
```
ROUND([Sales], 2)       - 2 decimals
ROUND([Sales], -2)      - Nearest 100
ABS([Profit])           - Absolute value (no negatives)
CEILING([Price])        - Round up
FLOOR([Price])          - Round down
POWER([Sales], 2)       - Square
SQRT([Area])            - Square root
```

**DATE FUNCTIONS** ⭐
```
TODAY()                 - Current date
YEAR([Date])            - Extract year
MONTH([Date])           - Extract month (1-12)
DATENAME('month', [Date]) - "January"
QUARTER([Date])         - Quarter (1-4)

DATEDIFF('day', [Start], [End])     - Days between
DATEDIFF('month', [Start], [End])   - Months between
DATEDIFF('year', [Birth], TODAY())  - Age calculation

DATEADD('day', 30, [Date])    - Add 30 days
DATEADD('month', -3, TODAY()) - 3 months ago

DATETRUNC('month', [Date])    - First day of month
DATEPART('weekday', [Date])   - Day of week (1-7)
```

### **Logical Functions**
```
IF [Sales] > 1000 THEN "High"
ELSEIF [Sales] > 500 THEN "Medium"
ELSE "Low"
END

CASE [Region]
  WHEN "West" THEN "Pacific"
  WHEN "East" THEN "Atlantic"
  ELSE "Other"
END

CASE
  WHEN [Sales] > 10000 THEN "Tier 1"
  WHEN [Sales] > 5000 THEN "Tier 2"
  ELSE "Tier 3"
END

IIF([Profit] > 0, "Profitable", "Loss")  - Shortcut

[Region] IN ("West", "East", "South")    - Multiple OR
```

### **NULL Handling**
```
IFNULL([Field], 0)         - If NULL, use 0
ZN([Field])                - Zero if Null (shortcut)
ISNULL([Field])            - Returns TRUE/FALSE
[Field] IS NOT NULL        - Filter condition
```

---

## **MODULE 5: ADVANCED VISUALIZATIONS**

### **Dual Axis**
```
HOW TO CREATE:
1. Drag first measure to Rows
2. Drag second measure to Rows
3. Right-click second → Dual Axis

SYNCHRONIZE AXES:
Right-click axis → Synchronize Axis
- Use when: Same units, comparable scale
- Don't use: Different units ($ vs %)

TWO MARKS CARDS:
Top = First measure
Bottom = Second measure
All = Both measures
```

### **Reference Lines**
```
SCOPE:
Table  - One line across entire view
Pane   - One line per partition (e.g., per category)
Cell   - One line per cell (detailed)

COMMON USES:
Average       - WINDOW_AVG or AVG
Target        - Constant value
Zero Line     - Constant (0)
Median        - Median
Percentiles   - 25th, 75th, etc.

Reference Band = Shaded area between two values
Distribution   = Statistical spread (±1 SD, ±2 SD)
```

### **Trend Lines**
```
Add: Analytics pane → Drag Trend Line

MODELS:
Linear      - Straight line (most common)
Exponential - Exponential growth/decay
Polynomial  - Curved line

R-SQUARED (R²):
1.0 = Perfect fit (100% variance explained)
0.8 = Good fit
0.5 = Moderate
0.2 = Weak (don't trust trend!)

P-VALUE:
< 0.05 = Statistically significant
> 0.05 = May be random

OPTIONS:
☑ Allow trend line per color - Separate trend per category
☑ Show confidence bands - Prediction interval
```

### **Parameters** ⭐
```
WHAT: User-controlled value (not from data)

CREATE:
Right-click Data pane → Create Parameter

SETTINGS:
- Data type (String, Integer, Float, Date, Boolean)
- Current value (default)
- Allowable values: All, List, Range

USE IN CALCULATIONS:
IF [Parameters.Metric] = "Sales" THEN SUM([Sales])
ELSEIF [Parameters.Metric] = "Profit" THEN SUM([Profit])
END

SHOW CONTROL:
Right-click parameter → Show Parameter Control
```

---

## **MODULE 6: DASHBOARDS**

### **Tiled vs Floating**
```
TILED (Default)
✓ Snap to grid
✓ Resize together
✓ No overlapping
✓ Responsive
Use: Standard layouts

FLOATING
✓ Can overlap
✓ Fixed position (x, y)
✓ Fixed size
✓ Layer on top
Use: Logos, popups, watermarks
```

### **Containers**
```
HORIZONTAL - Items left-to-right ┌─┬─┬─┐
VERTICAL   - Items top-to-bottom  ├─┤

BEST PRACTICE:
Main structure: Vertical container
Rows: Horizontal containers inside vertical
Nest max 3 levels deep

DISTRIBUTION:
Uniform = All items equal size
```

### **Dashboard Size**
```
Fixed Size    - Exact pixels (1200 x 800)
Automatic     - Fills browser window
Range         - Min/Max bounds (800-1400)

COMMON SIZES:
Desktop:  1366 x 768, 1920 x 1080
Tablet:   1024 x 768
Phone:    375 x 667, 360 x 640

DEVICE DESIGNER:
Bottom toolbar → Device layouts
Different layout per device type
```

### **Dashboard Actions**
```
FILTER ACTION
User clicks → Filters other sheets
Dashboard → Actions → Add Action → Filter

HIGHLIGHT ACTION  
User hovers/clicks → Highlights across sheets
Less disruptive than filter

URL ACTION
Click mark → Opens URL
Can pass parameters: ?region=<Region>

Options:
- Run on: Hover, Select, Menu
- Clearing: Show all values, Keep filtered values
- Target sheets: Which sheets affected
```

### **Show/Hide Buttons**
```
Click object dropdown → Add Show/Hide Button
Creates collapsible sections
Use for: Filters, help text, advanced options
```

---

## **MODULE 7: TABLE CALCULATIONS & LOD** ⭐⭐⭐

### **Table Calculations - Quick Reference**
```
RUNNING_SUM(SUM([Sales]))           - Cumulative total
WINDOW_AVG(SUM([Sales]), -2, 0)     - 3-period moving avg
LOOKUP(SUM([Sales]), -1)            - Previous value
LOOKUP(SUM([Sales]), FIRST())       - First value
RANK(SUM([Sales]))                  - Position (1, 2, 3...)
INDEX()                             - Row number (1, 2, 3...)
SIZE()                              - Total rows in partition
TOTAL(SUM([Sales]))                 - Grand total

COMMON CALCULATIONS:
% Change: (SUM([Sales]) - LOOKUP(SUM([Sales]), -1)) / LOOKUP(SUM([Sales]), -1)
% of Total: SUM([Sales]) / TOTAL(SUM([Sales]))
Running %: RUNNING_SUM(SUM([Sales])) / TOTAL(SUM([Sales]))
```

### **Compute Using** ⭐⭐⭐ (CRITICAL!)
```
DIRECTION table calc operates:

Table (across)       - Left to right, entire table
Table (down)         - Top to bottom, entire table
Table (across then down) - Read like book
Pane (down)          - Down within each partition
Pane (across)        - Across within each partition
Specific Dimensions  - Along chosen dimension(s)

ADDRESSING vs PARTITIONING:
Addressing   = Direction of calculation (checked boxes)
Partitioning = Restarts calculation (unchecked boxes)

EXAMPLE:
Running Total by Month, restarting each Year
Compute Using: Month (addressing)
Partition by: Year (check "Specific Dimensions", uncheck Year)
```

### **LOD Expressions** ⭐⭐⭐ (CRITICAL!)

**SYNTAX & USE CASES**
```
FIXED - Independent of view dimensions
{ FIXED [Customer] : SUM([Sales]) }
- Customer total sales
- Use: Different granularity than view

INCLUDE - View dimensions PLUS specified
{ INCLUDE [Product] : SUM([Sales]) }
- Adds Product detail, then aggregates back
- Use: "Average per X" calculations

EXCLUDE - View dimensions MINUS specified  
{ EXCLUDE [Product] : SUM([Sales]) }
- Removes Product, shows category total
- Use: % of subtotal calculations

NO DIMENSIONS - Grand total
{ FIXED : SUM([Sales]) }
- Total across all data
- Use: % of grand total
```

**COMMON LOD PATTERNS**
```
Customer Lifetime Value:
{ FIXED [Customer] : SUM([Profit]) }

First Purchase Date:
{ FIXED [Customer] : MIN([Order Date]) }

Orders per Customer:
{ FIXED [Customer] : COUNTD([Order ID]) }

% of Category Total:
SUM([Sales]) / { EXCLUDE [Product] : SUM([Sales]) }

% of Grand Total (LOD way):
SUM([Sales]) / { FIXED : SUM([Sales]) }

New vs Returning:
IF [Order Date] = { FIXED [Customer] : MIN([Order Date]) }
THEN "New" ELSE "Returning" END
```

### **LOD vs Table Calc**
```
Use LOD when:
✓ Need different granularity than view
✓ Customer-level in product view
✓ Independent of filters (FIXED)
✓ Part of larger calculation

Use Table Calc when:
✓ Running totals, ranks
✓ Comparing to previous/other values
✓ Moving averages
✓ Working with values in table
```

### **Filter Order of Operations** ⭐⭐⭐
```
1. Extract Filters
2. Data Source Filters
3. Context Filters ← Add filters here to affect LOD
4. FIXED LOD        ← Calculates here (ignores dimension filters!)
5. Dimension Filters (Top N, conditional, general)
6. INCLUDE/EXCLUDE LOD ← Calculates here
7. Measure Filters
8. Table Calc Filters

KEY INSIGHTS:
- FIXED ignores dimension filters
- Make filter Context to have FIXED respect it
- Table calcs calculated last (affected by all filters)
- Context filters for Top N to work correctly
```

### **Context Filters**
```
WHY USE:
1. Make Top N work correctly with other filters
2. LOD expressions need to respect filter
3. Performance (pre-filter large data)

HOW TO CREATE:
Right-click filter → Add to Context
Filter turns gray

EXAMPLE:
Problem: Filter Region = West, then Top 10 Products
Without context: Top 10 calculated first (all regions), 
                 then filtered to West → might show <10
With context: Filter to West first, 
              then calculate Top 10 within West → exactly 10
```

---

## **MODULE 8: ADVANCED TOPICS**

### **Joins**
```
INNER JOIN - Only matches (most restrictive)
LEFT JOIN  - All from left + matches from right (most common)
RIGHT JOIN - All from right + matches from left (rare)
FULL OUTER - Everything from both (rare)

COMMON ISSUE:
Many-to-many join → Duplicates → Inflated sums
Solution: Check granularity, use LOD, or blend instead

CROSS-DATABASE JOIN:
Join tables from different databases
Requires Tableau Desktop
Use extracts for performance
```

### **Blending**
```
WHEN TO USE:
✓ Different data sources (SQL + Excel)
✓ Different granularities
✓ Quick comparisons
✓ Small secondary data

vs JOINS:
- Happens at worksheet level (not data source)
- Secondary data pre-aggregated (*)
- Slower than joins
- Can't access row-level from secondary

INDICATORS:
Blue checkmark  = Primary data source
Orange checkmark = Secondary data source
Orange chain link = Active linking field

Edit: Data → Edit Relationships
```

### **Data Preparation**
```
DATA INTERPRETER
Auto-cleans messy Excel (titles, notes, blanks)
Check box on Data Source connection

PIVOT
Wide → Long format
Select columns → Right-click → Pivot
Creates: Pivot Field Names, Pivot Field Values

SPLIT
"Last, First" → "Last" + "First"
Right-click column → Split or Custom Split

UNION
Stacks rows (append)
Drag table BELOW previous (not beside)
Wildcard union for pattern matching
```

### **Performance Optimization** ⭐
```
EXTRACTS
When: Large data, slow DB, complex joins
Options: Filter, aggregate, hide fields

QUERY OPTIMIZATION
✓ Data source filters (earliest!)
✓ Context filters (create subset)
✓ Limit dimensions (only needed fields)
✓ Simple joins (fewer, inner when possible)

DASHBOARD OPTIMIZATION
✓ 5-8 sheets maximum
✓ Filter actions > filter controls
✓ Hide unused sheets
✓ <10,000 marks per sheet

CALCULATION OPTIMIZATION
✓ Boolean > string comparisons
✓ Break complex calcs into steps
✓ Avoid nested LODs
✓ Simple table calcs > complex

TOOL:
Help → Settings and Performance → Start Performance Recording
Shows: Query time, calculation time, rendering time
```

---

## **EXAM QUICK TIPS** 🎯

### **High-Frequency Topics**
```
1. Row-level vs Aggregate calculations ⭐⭐⭐
2. Table calc "Compute Using" direction ⭐⭐⭐
3. FIXED vs INCLUDE vs EXCLUDE ⭐⭐⭐
4. Filter order of operations ⭐⭐⭐
5. Chart type selection ⭐⭐
6. Date functions (DATEDIFF, DATEADD) ⭐⭐
7. Context filters for Top N ⭐⭐
8. Aggregation and granularity ⭐⭐
9. Joins vs Blending ⭐⭐
10. Dashboard actions ⭐⭐
```

### **Common Errors & Fixes**
```
"Cannot mix aggregate and non-aggregate"
Fix: Make all row-level OR all aggregate
✓ SUM(IF [Category]="Furniture" THEN [Sales] ELSE 0 END)
✓ IF [Category]="Furniture" THEN [Sales] ELSE 0 END (then aggregate)

"Table calc returns NULL"
Fix: LOOKUP offset goes out of range
✓ IFNULL(LOOKUP(SUM([Sales]), -1), 0)

"LOD doesn't work as expected"
Fix: FIXED ignores dimension filters
✓ Make filter a Context Filter

"Top N shows wrong count"
Fix: Applied after other filters
✓ Make dimension filter a Context Filter

"Wrong results from calculation"
Fix: Check granularity - different dimensions = different results
✓ Verify what's in view (Detail card counts!)
```

### **Decision Trees**

**Calculation Choice:**
```
Different granularity than view? → LOD
Compare to other values in table? → Table Calc
Simple math at view level? → Regular Calc
```

**LOD Type:**
```
Independent of view? → FIXED
Add detail? → INCLUDE  
Remove detail? → EXCLUDE
```

**Chart Type:**
```
Compare categories? → Bar
Trend over time? → Line
Part of whole? → Tree Map (>5 cats) or Pie (<5)
Correlation? → Scatter
Distribution? → Box Plot
```

---

## **LAST-MINUTE REMINDERS** ✅

```
✓ Read questions CAREFULLY - subtle details matter
✓ Watch for "NOT", "EXCEPT", "ALWAYS" in questions
✓ Eliminate obviously wrong answers first
✓ Don't spend >2 minutes on one MCQ
✓ Build basic versions first in hands-on, refine later
✓ Test as you build - don't wait till end
✓ Save frequently!
✓ Manage time: 50% build, 30% refine, 20% verify
✓ If stuck, move on - come back if time permits
✓ Format calculations (currency, %, dates)
✓ Name everything descriptively
✓ Check all requirements before submitting

STAY CALM → TRUST YOUR PREP → YOU'VE GOT THIS! 💪
```

---