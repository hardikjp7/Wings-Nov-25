# TCS Wings Tableau Hands-On Exam Preparation Guide

Based on the questions you've shared, here's a comprehensive preparation guide for the hands-on exam:

---

## Common Exam Pattern Analysis

### Typical Structure:
1. **3-4 Sheets** with specific visualizations
2. **Calculated fields** creation (3-5 per exam)
3. **Filters** with specific display requirements
4. **Data export** to specific paths (CSV format)
5. **Formatting requirements** (decimals, colors, labels)
6. **File saving** as HackBook.twb

---

## Essential Skills Checklist

### 1. **Data Connection & Setup**
```
✓ Connect to CSV file from specific path
✓ Create new workbook named "HackBook.twb"
✓ Save workbook in correct location
✓ Navigate file system paths
```

**Practice Steps:**
```
1. File → Open → Navigate to ~/Desktop/Project/[folder-name]/inputfile/
2. Connect to .csv file
3. File → Save As → Name: HackBook.twb
4. Save location: ~/Desktop/Project/[folder-name]/
```

---

### 2. **Creating Calculated Fields** (CRITICAL)

You'll need to create 5-10 calculated fields per exam. Master these patterns:

#### A. **Count Calculations**
```
Order Count = COUNTD([Order Id])
Customer Count = COUNTD([Customer ID])
Product Count = COUNTD([Product ID])
```

#### B. **Mathematical Operations**
```
Order Value = [Quantity] * [Price]
Total Cost = [Quantity] * [Unit Price]
Profit = [Sales] - [Cost]
Discount Amount = [Sales] * [Discount]
```

#### C. **Average/Aggregate Calculations**
```
AOV (Average Order Value) = SUM([Order Value]) / COUNTD([Order ID])
Avg Rating = AVG([Rating])
Total Revenue = SUM([Sales])
```

#### D. **Date Calculations** (VERY COMMON)
```
Order Month = DATE(DATETRUNC('month', [Order Date]))
Order Year = YEAR([Order Date])
Order Quarter = DATEPART('quarter', [Order Date])
```

#### E. **Conditional Logic** (FREQUENT)
```
Customer Type = 
IF {FIXED [Customer ID] : MIN([Order Date])} = [Order Date] 
THEN "New Customer" 
ELSE "Returning Customer" 
END

Delivery Status Flag = 
IF [Delivery Status] = "Delivered" THEN "Success" 
ELSE "Failed" 
END
```

#### F. **Parameter-Based Calculations** (IMPORTANT)
```
Selected Measure = 
CASE [Select Metric]
WHEN "Average Rating" THEN AVG([Rating])
WHEN "Total Revenue" THEN SUM([Order Value])
WHEN "Total Orders" THEN COUNTD([Order Id])
END
```

---

### 3. **Chart Types You MUST Know**

#### A. **Bar Chart** (Most Common)
```
Configuration:
- Rows: Measure (e.g., SUM(Sales))
- Columns: Dimension (e.g., Category)
- Color: Sub-dimension (e.g., Sub-Category)
- Mark Type: Bar (Automatic)

Steps:
1. Drag dimension to Columns
2. Drag measure to Rows
3. Drag sub-dimension to Color
4. Add title
```

#### B. **Line Chart** (Common)
```
Configuration:
- Rows: Measure (e.g., AVG(Discount))
- Columns: Dimension (e.g., Sub-Category) or Date
- Color: Category for multiple lines
- Mark Type: Line

Steps:
1. Drag dimension to Columns
2. Drag measure to Rows
3. Change mark type to Line
4. Add Color for differentiation
```

#### C. **Pie Chart** (Frequent)
```
Configuration:
- Columns: Category (creates multiple pies)
- Angle: Measure (e.g., SUM(Quantity))
- Color: Sub-Category
- Mark Type: Pie

Steps:
1. Drag Category to Columns (for multiple pies)
2. Change mark type to Pie
3. Drag measure to Angle
4. Drag dimension to Color
5. Add Labels
```

#### D. **Heat Map** (Geography-based)
```
Configuration:
- Rows: Latitude (generated)
- Columns: Longitude (generated)
- Color: Measure (e.g., MAX(Total Cost))
- Mark Type: Map (Automatic)

Steps:
1. Change State field: Geographic Role → State/Province
2. Drag State to view (creates map)
3. Drag measure to Color
4. Change color palette (e.g., Temperature Diverging)
```

#### E. **Area Chart**
```
Configuration:
- Rows: Measure (e.g., Order Value)
- Columns: Date (Month)
- Color: Dimension (e.g., City)
- Mark Type: Area

Steps:
1. Drag date to Columns
2. Drag measure to Rows
3. Change mark type to Area
4. Add dimension to Color
```

#### F. **Square/Heat Map (Matrix)**
```
Configuration:
- Rows: Dimension 1 (e.g., Restaurant Name)
- Columns: Dimension 2 (e.g., Month)
- Color: Measure
- Mark Type: Square

Steps:
1. Drag dimensions to Rows and Columns
2. Change mark type to Square
3. Drag measure to Color and Label
4. Format colors and borders
```

---

### 4. **Table Calculations** (CRITICAL)

#### Quick Table Calculations:
```
Common requirement: "Percent of Total"

Steps:
1. Right-click measure pill → Quick Table Calculation
2. Select "Percent of Total"
3. Right-click again → Compute Using
4. Select "Specific Dimensions"
5. Choose dimension (e.g., Delivery Status)
```

**Other Common Table Calcs:**
- Running Total
- Difference
- Percent Difference
- Rank

---

### 5. **Formatting Requirements** (ALWAYS TESTED)

#### A. **Number Formatting**
```
Requirement: "1 decimal place"

Steps:
1. Right-click measure → Format
2. Numbers → Custom
3. Set decimals to 1
4. Or use Format pane (right-side)
```

#### B. **Percentage Formatting**
```
Requirement: "Add % symbol, 1 decimal"

Method 1 (for actual percentages):
1. Right-click measure → Format
2. Numbers → Percentage
3. Decimal places: 1

Method 2 (for table calc percentages):
1. Format as above
2. Ensure shows as 34.2% (not 0.342)
```

#### C. **Custom Label Formatting**
```
Requirement: "Add suffix ' - AOV'"

Steps:
1. Click Label on Marks card
2. Click Text button
3. Edit text: <AGG(Field)> - AOV
4. Format number within: <AGG(Field):1> for 1 decimal
```

#### D. **Color Formatting**
```
Common requirements:
- "Temperature Diverging" palette
- "Add borders with black color"

Steps:
1. Click Color → Edit Colors
2. Select palette: Temperature Diverging
3. Click Color → Border → Black
4. Border width: As needed
```

---

### 6. **Filters** (ALWAYS PRESENT)

#### A. **Creating Filters**
```
Steps:
1. Drag dimension/measure to Filters shelf
2. Select values
3. Right-click filter → Show Filter
4. Change display type (dropdown, list, slider)
```

#### B. **Year Filter** (Common)
```
Requirement: "Filter by year 2024, show as dropdown"

Steps:
1. Drag Order Date to Filters
2. Select "Years"
3. Check "2024"
4. Right-click filter → Show Filter
5. Filter card dropdown → Single Value (dropdown)
```

#### C. **Category Filter** (Common)
```
Requirement: "Filter Category = 'Fast Food', show dropdown"

Steps:
1. Drag Category to Filters
2. Select "Fast Food" only
3. Show Filter
4. Change to dropdown format
```

---

### 7. **Sorting** (Frequent Requirement)

```
Requirement: "Order by Rating descending"

Steps:
1. Right-click dimension (e.g., Restaurant Name)
2. Sort
3. Sort by: Field
4. Field: Rating
5. Aggregation: Average
6. Order: Descending
7. OK
```

---

### 8. **Parameters** (Advanced but Common)

#### Creating Parameter:
```
Requirement: "Create parameter with metrics"

Steps:
1. Right-click in Data pane → Create Parameter
2. Name: Select Metric
3. Data type: String
4. Allowable values: List
5. Add values:
   - Average Rating
   - Total Revenue
   - Total Orders
6. Current value: (select one)
7. OK
8. Right-click parameter → Show Parameter Control
```

#### Using Parameter in Calculation:
```
Create calc: Selected Measure

Formula:
CASE [Select Metric]
WHEN "Average Rating" THEN AVG([Rating])
WHEN "Total Revenue" THEN SUM([Order Value])
WHEN "Total Orders" THEN COUNTD([Order Id])
END

Use this calc in visualization
```

---

### 9. **Data Export** (MANDATORY - Don't Forget!)

#### Export Steps (CRITICAL):
```
Method 1: Export All (Full data)
1. View worksheet
2. Analysis → View Data
3. Full Data tab
4. Export All button
5. Save as: [specified name].csv
6. Path: ~/Desktop/Project/[folder]/Output_Data/[name].csv

Method 2: Download (View data only)
1. Analysis → View Data
2. Download button (bottom)
3. Save to specified path

IMPORTANT: 
- File name MUST match requirement exactly
- Path MUST be correct
- Include .csv extension
```

#### Common Export Names:
- Delivery_Outcome.csv
- Revenue_vs_Experience.csv
- Restaurant_Performance.csv
- TotalSales.csv
- AvgDiscount.csv
- HighestOrder.csv

---

### 10. **Chart Titles** (Always Required)

```
Adding Title:

Steps:
1. Worksheet → Show Title (checked)
2. Double-click title area
3. Enter exact text as specified
4. Format as needed
5. OK

Examples:
- "City Delivery Outcome"
- "Monthly Revenue vs Customer Experience by City"
- "Total Sales of Each Sub-Category"
```

---

## Complete Workflow for Each Sheet

### Standard Process:
```
1. CREATE NEW SHEET
   - Rename to match requirement

2. CREATE CALCULATED FIELDS
   - All fields needed for that sheet
   - Test each calculation

3. BUILD VISUALIZATION
   - Drag dimensions/measures as specified
   - Set mark type
   - Add colors, labels

4. APPLY TABLE CALCULATIONS
   - If required (e.g., Percent of Total)
   - Set compute using correctly

5. ADD FILTERS
   - Create filters as specified
   - Show filter controls
   - Set display type

6. FORMAT
   - Numbers (decimals)
   - Colors (palettes)
   - Labels (suffixes/prefixes)
   - Borders
   - Sorting

7. ADD TITLE
   - Exact text as specified

8. EXPORT DATA
   - Analysis → View Data → Export All
   - Correct filename
   - Correct path

9. VERIFY
   - Visual matches requirements
   - All formatting correct
   - Data exported
```

---

## Common Mistakes to Avoid

### ❌ CRITICAL ERRORS (Will Lose Marks):
```
1. Wrong file name (HackBook.twb vs other name)
2. Export to wrong path
3. Wrong CSV export name
4. Forgot to export data
5. Wrong calculated field formula
6. Wrong aggregation (SUM vs AVG vs COUNT vs COUNTD)
7. Didn't save workbook
8. Wrong mark type
9. Forgot to show filter
10. Wrong decimal places
```

### ⚠️ Common Errors:
```
1. Incorrect table calculation compute using
2. Wrong color palette
3. Missing labels
4. Wrong sort order
5. Filter not shown as dropdown
6. Geographic role not set for maps
7. Date not truncated correctly
8. Parameter not shown
```

---

## Time Management Strategy

### For 3-4 Sheet Exam (Typical Duration: 90-120 minutes):

```
Per Sheet (20-30 minutes):
- 5 min: Create calculated fields
- 10 min: Build visualization
- 5 min: Format and configure
- 5 min: Export data
- 5 min: Verify and fix issues

Buffer: 20-30 minutes for issues/review
```

---

## Practice Checklist

### Before Exam:
```
✓ Practice connecting to CSV from specific paths
✓ Create 20+ calculated fields (all types)
✓ Build each chart type 5+ times
✓ Practice table calculations
✓ Practice formatting (decimals, %, colors)
✓ Practice data export to specific paths
✓ Practice parameter creation and use
✓ Practice geographic role assignment
✓ Practice filter display types
✓ Complete 3-5 full mock problems
```

---

## Quick Reference Card

### Most Common Calculations:
```
1. COUNTD([Order ID]) - Distinct count
2. [Field1] * [Field2] - Multiplication
3. SUM([Field]) / COUNTD([ID]) - Average per entity
4. DATETRUNC('month', [Date]) - Month extraction
5. {FIXED [ID] : MIN([Date])} - First occurrence
6. CASE [Parameter] WHEN... - Parameter switch
7. AVG([Rating]) - Average
8. IF condition THEN... ELSE... END - Conditional
```

### Most Common Formats:
```
1. Right-click measure → Format → Number → Decimal: 1
2. Click Color → Edit Colors → Temperature Diverging
3. Click Color → Border → Black
4. Label → Text → Add suffix/prefix
5. Filter → Show Filter → Dropdown format
```

### Data Export Command:
```
Analysis → View Data → Export All → Save as [name].csv
Path: ~/Desktop/Project/[folder]/Output_Data/[name].csv
```

---

## Final Checklist Before Submission

```
Before Closing Environment:

□ All sheets created and named correctly
□ All calculated fields working
□ All visualizations match requirements
□ All formatting applied (decimals, colors, labels)
□ All filters shown and configured
□ All titles added
□ All data exported to correct paths
□ Workbook saved as HackBook.twb
□ Run test.exe (if available) to check score
□ Verify all CSV files exist in Output_Data folder
```

---

## Mock Problem for Practice

Try this complete problem to test your readiness:

### Mock Exam: Sales Analysis

**Files:** sales_data.csv (Customer ID, Order ID, Order Date, Product, Category, Quantity, Price, Rating)

**Sheet 1: Category Performance**
- Bar chart: Category vs Total Revenue
- Color by Product
- Filter: Order Year = 2024 (dropdown)
- Table calc: Percent of Total
- Format: 1 decimal, % symbol
- Export: CategoryPerformance.csv

**Sheet 2: Monthly Trend**
- Line chart: Month vs Average Rating
- Color by Category
- Filter: Category (dropdown)
- Add labels with 2 decimals
- Export: MonthlyTrend.csv

**Sheet 3: Customer Analysis**
- Create calc: Customer Type (New vs Returning)
- Create calc: Order Value = Quantity * Price
- Area chart: Month vs Order Value
- Color by Customer Type
- Detail: Rating
- Export: CustomerAnalysis.csv

Complete this in 60 minutes!

---

**You're now ready for the TCS Wings Tableau hands-on exam!** Practice these patterns, and you'll handle any variation they throw at you. Good luck! 🎯
