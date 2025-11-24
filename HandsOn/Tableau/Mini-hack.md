Great question! Let me teach you a \*\*systematic approach\*\* to break down complex Tableau tasks. These questions can seem overwhelming, but with the right strategy, they become manageable.



---



\## 🎯 \*\*Step-by-Step Breakdown Strategy\*\*



\### \*\*Step 1: Identify the Big Picture\*\*

Read through everything once quickly to understand:

\- \*\*What's the goal?\*\* (Create 3 visualization sheets)

\- \*\*What data do I have?\*\* (food\_cleaned\_data.csv)

\- \*\*What's the output?\*\* (3 sheets + 3 CSV exports)



---



\### \*\*Step 2: Break Into Chunks\*\*

Divide the task into \*\*independent sections\*\*. Each sheet is a separate mini-project.



```

Task = Sheet 1 + Sheet 2 + Sheet 3

```



Work on \*\*ONE sheet at a time\*\*. Don't try to do everything at once!



---



\### \*\*Step 3: Use the "Question Template" Method\*\*



For each sheet, extract information using this template:



\#### 📊 \*\*Chart Template\*\*



| Element | What to Look For | Example from Sheet 1 |

|---------|------------------|----------------------|

| \*\*Chart Title\*\* | What's the visualization called? | "City Delivery Outcome" |

| \*\*Chart Type\*\* | Bar, Line, Area, Pie, etc.? | Bar |

| \*\*Rows\*\* | What goes on Y-axis? | Order Count |

| \*\*Columns\*\* | What goes on X-axis? | City |

| \*\*Filters\*\* | What data to include/exclude? | (none in Sheet 1) |

| \*\*Colors\*\* | What creates different colors? | Delivery Status |

| \*\*Labels\*\* | What text appears on chart? | AOV (Per City) |

| \*\*Calculated Fields\*\* | What new metrics to create? | Order Count, Order Value, AOV |



---



\## 🔍 \*\*Let's Break Down Sheet 1 Together\*\*



\### \*\*Original Question:\*\*

```

Sheet 1: Delivery\_Outcome

Chart Title: City Delivery Outcome

Rows: Order Count

Columns: City

Mark: Bar



1\. Create Calculated Fields:

&nbsp;  a. Order Count = Find the distinct count of orders using Order Id.

&nbsp;  b. Order Value = Find the product of Quantity and Price.

&nbsp;  c. AOV (Per City) = Total Order Value / Order Count



2\. Use Quick Table Calculation for Order Count in Rows as "Percent of Total"...

3\. Add a "%" symbol...

4\. Differentiate the chart using Delivery Status in color...

```



---



\### \*\*My Breakdown Process:\*\*



\#### \*\*Round 1: Extract Basic Structure\*\*

I create a simple checklist:



```

✅ SHEET NAME: Delivery\_Outcome

✅ TITLE: City Delivery Outcome

✅ CHART TYPE: Bar Chart



AXES:

\- X-axis (Columns): City

\- Y-axis (Rows): Order Count



TO CREATE:

□ 3 Calculated Fields

□ Apply percentage calculation

□ Format numbers

□ Add colors

□ Add labels

□ Export CSV

```



---



\#### \*\*Round 2: Decode Calculated Fields\*\*



Let me translate the business language into Tableau language:



| Business Requirement | Translation | Tableau Formula |

|---------------------|-------------|-----------------|

| "Distinct count of orders" | Count unique Order IDs | `COUNTD(\[Order Id])` |

| "Product of Quantity and Price" | Multiply two fields | `\[Quantity] \* \[Price]` |

| "Total Order Value / Order Count" | Division of aggregates | `SUM(\[Order Value]) / COUNTD(\[Order Id])` |



\*\*💡 Key Insight:\*\* 

\- Words like "distinct count" → `COUNTD()`

\- "Product" → multiply `\*`

\- "Total" → `SUM()`

\- "Average" → `AVG()`



---



\#### \*\*Round 3: Decode Complex Instructions\*\*



\*\*Original:\*\*

> "Use Quick Table Calculation for Order Count in Rows as 'Percent of Total' and compute using 'Specific Dimensions' which should be 'Delivery Status'."



\*\*My Translation Process:\*\*



1\. \*\*What field?\*\* → Order Count

2\. \*\*What calculation?\*\* → Percent of Total

3\. \*\*Compute using what?\*\* → Delivery Status



\*\*What this means:\*\*

\- Show each bar as a % of total orders within each delivery status group

\- This creates percentages that compare cities within "Delivered" vs "Not Delivered" groups



---



\#### \*\*Round 4: Decode Formatting Requirements\*\*



\*\*Original:\*\*

> "Add a '%' symbol at the end of the Order Count values, formatted to one decimal place."



\*\*Translation:\*\*

```

Current display: 34.210

Required display: 34.2%



Action needed:

1\. Format as percentage

2\. Set to 1 decimal place

```



---



\## 🧩 \*\*Common Tricky Patterns \& How to Handle Them\*\*



\### \*\*Pattern 1: Nested Requirements\*\*



\*\*Example:\*\*

> "Differentiate the chart using Delivery Status in color and use AOV (Per City) in Label with a suffix as ' - AOV'."



\*\*Break it down:\*\*

```

This sentence has 2 actions:



Action 1: "Differentiate using Delivery Status in color"

&nbsp;  → Drag Delivery Status to Color shelf



Action 2: "Use AOV (Per City) in Label with suffix ' - AOV'"

&nbsp;  → Drag AOV to Label shelf

&nbsp;  → Format label to add " - AOV" text

```



---



\### \*\*Pattern 2: Conditional Logic\*\*



\*\*Example from Sheet 2:\*\*

> "Customer Type = Set flag 'New Customer' when minimum of Order Date for each Customer ID equals Order Date, otherwise set flag as 'Returning Customer'"



\*\*My Breakdown:\*\*



```

IF condition:

&nbsp;  If \[this customer's first order date] = \[current order date]

&nbsp;  

THEN:

&nbsp;  "New Customer"

&nbsp;  

ELSE:

&nbsp;  "Returning Customer"

```



\*\*Tableau Formula:\*\*

```

IF {FIXED \[Customer ID]: MIN(\[Order Date])} = \[Order Date]

THEN "New Customer"

ELSE "Returning Customer"

END

```



\*\*💡 Translation Key:\*\*

\- "When" / "If" → `IF`

\- "Otherwise" → `ELSE`

\- "Each Customer ID" → `{FIXED \[Customer ID]: ...}` (LOD expression)



---



\### \*\*Pattern 3: Multi-Step Transformations\*\*



\*\*Example from Sheet 3:\*\*

> "Order the Restaurant Name using average of Rating in descending order."



\*\*Step-by-step:\*\*

```

1\. WHAT to sort? → Restaurant Name (on Rows)

2\. HOW to sort? → By another field

3\. WHICH field? → Rating

4\. HOW to aggregate? → Average

5\. WHAT order? → Descending (high to low)

```



---



\## 📋 \*\*My Universal Checklist for Any Tableau Question\*\*



Copy this template for every sheet:



```

SHEET NAME: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_



1\. BASIC STRUCTURE

&nbsp;  □ Chart title: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

&nbsp;  □ Chart type: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

&nbsp;  □ Rows (Y-axis): \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

&nbsp;  □ Columns (X-axis): \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_



2\. CALCULATED FIELDS NEEDED

&nbsp;  □ Field 1: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ = \[Formula]

&nbsp;  □ Field 2: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ = \[Formula]

&nbsp;  □ Field 3: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ = \[Formula]



3\. FILTERS

&nbsp;  □ Filter 1: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ (Show filter? Yes/No)

&nbsp;  □ Filter 2: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ (Type: Dropdown/Slider/etc)



4\. VISUAL PROPERTIES

&nbsp;  □ Color: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

&nbsp;  □ Label: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

&nbsp;  □ Detail: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

&nbsp;  □ Tooltip: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_



5\. SPECIAL CALCULATIONS

&nbsp;  □ Table calculations: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

&nbsp;  □ LOD expressions: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

&nbsp;  □ Parameters: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_



6\. FORMATTING

&nbsp;  □ Number formats: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

&nbsp;  □ Date formats: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

&nbsp;  □ Custom text: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_



7\. SORTING/ORDERING

&nbsp;  □ Sort by: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

&nbsp;  □ Order: Ascending/Descending



8\. OUTPUT

&nbsp;  □ Export location: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

&nbsp;  □ Export filename: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

```



---



\## 🎓 \*\*Pro Tips for Understanding Tricky Requirements\*\*



\### \*\*Tip 1: Highlight Action Words\*\*



Read through and highlight verbs:



> "\*\*Create\*\* Calculated Fields, \*\*Use\*\* Quick Table Calculation, \*\*Add\*\* a % symbol, \*\*Differentiate\*\* using color, \*\*Export\*\* data"



Each verb = one action you need to do.



---



\### \*\*Tip 2: Number Everything\*\*



Even if the question doesn't number steps clearly, you number them:



```

Original: "Filter the chart using year of Order Date as 2024 and show filter in dropdown"



My version:

Step 1: Filter Order Date by year = 2024

Step 2: Show the filter

Step 3: Change filter type to dropdown

```



---



\### \*\*Tip 3: Draw It Out\*\*



For complex visualizations, I sketch:



```

Sheet 1: Bar Chart



&nbsp;    City

&nbsp;    ↓

\[Bar]\[Bar]\[Bar] ← Order Count (%)

&nbsp; A    B    C

&nbsp; 

Colors = Delivery Status

Labels = AOV value

```



This visual helps me understand what goes where.



---



\### \*\*Tip 4: Translate Tableau Jargon\*\*



| Tableau Term | What It Really Means |

|--------------|---------------------|

| "Rows shelf" | Y-axis (vertical) |

| "Columns shelf" | X-axis (horizontal) |

| "Marks card" | Properties of chart elements |

| "Quick Table Calculation" | Percentage/running total calculations |

| "LOD Expression" | Calculation at specific grouping level |

| "COUNTD" | Count unique values |

| "DATETRUNC" | Round date to year/month/day |



---



\### \*\*Tip 5: Work Backwards from Output\*\*



Look at what they want as final result, then figure out what's needed:



\*\*They want:\*\* 34.2%

\*\*Work backwards:\*\*

\- Need % symbol → Format as percentage

\- Need 1 decimal → Set decimal places to 1

\- Need percentage value → Apply "Percent of Total" calculation



---



\## 🚀 \*\*Practice Exercise\*\*



Let me give you a mini-challenge using Sheet 2:



\*\*Question:\*\*

> "Filter the chart using the year of Order Date as '2024' and show the filter in dropdown."



\*\*Your turn! Break it down using my method:\*\*



<details>

<summary>Click to see my breakdown</summary>



```

Action Items:

1\. Identify filter field: Order Date

2\. Extract filter level: Year

3\. Set filter value: 2024

4\. Make filter visible: Show Filter

5\. Change display type: Dropdown



Tableau Steps:

1\. Drag \[Order Date] to Filters shelf

2\. Select "Years" in the dialog

3\. Check only "2024"

4\. Right-click filter → Show Filter

5\. Click dropdown arrow on filter card → Single Value (dropdown)

```



</details>



---



\## 📚 \*\*Summary: Your New Approach\*\*



When you see a complex Tableau question:



1\. \*\*Don't panic\*\* - Read through once for overview

2\. \*\*Break into sheets\*\* - One sheet at a time

3\. \*\*Use the template\*\* - Fill in: Title, Type, Rows, Columns, etc.

4\. \*\*Highlight action words\*\* - Create, Add, Filter, Sort, Export

5\. \*\*Translate business → Tableau\*\* - "Distinct count" = COUNTD()

6\. \*\*Number the steps\*\* - Even if they don't

7\. \*\*Sketch if needed\*\* - Visual understanding helps

8\. \*\*Work systematically\*\* - Check off each requirement



---



\*\*Want me to walk through Sheet 2 or Sheet 3 using this breakdown method so you can see it in action again?\*\* 



Or do you have a specific part that's still confusing? Let me know!







Perfect! Let me walk you through \*\*Sheet 2\*\* and \*\*Sheet 3\*\* using my systematic breakdown method. Watch how I transform confusing instructions into clear, actionable steps.



---



\# 🔍 \*\*Sheet 2: Revenue\_vs\_Experience - Complete Breakdown\*\*



\## \*\*Step 1: Read \& Highlight Action Words\*\*



Let me read the original requirement and highlight key actions:



> Sheet 2: Revenue\_vs\_Experience

> 

> Chart Title: Monthly Revenue vs Customer Experience by City

> 

> Rows: Order Value

> Columns: Month (Order Month)

> Mark: Area

> 

> 1. \*\*Create\*\* a calculated field:

>    a. Order Month = \*\*Truncate\*\* the month from Order Date.

>    b. Customer Type = \*\*Set flag\*\* "New Customer" when minimum of Order Date for each Customer ID equals Order Date, otherwise \*\*set flag\*\* as "Returning Customer"

> 

> 2. \*\*Filter\*\* the chart using the year of Order Data as "2024" and \*\*show\*\* the filter in dropdown.

> 

> 3. \*\*Differentiate\*\* the chart using City in color and in Label to \*\*analyze\*\* the order value of each city.

> 

> 4. \*\*Use\*\* Customer Type in detail and average of Rating in label for further analysis.

> 

> Finally, \*\*export\*\* the data into "csv"...



---



\## \*\*Step 2: Fill Out My Template\*\*



```

SHEET NAME: Revenue\_vs\_Experience



1\. BASIC STRUCTURE

&nbsp;  ✓ Chart title: Monthly Revenue vs Customer Experience by City

&nbsp;  ✓ Chart type: Area Chart

&nbsp;  ✓ Rows (Y-axis): Order Value (SUM)

&nbsp;  ✓ Columns (X-axis): Order Month (as Month)



2\. CALCULATED FIELDS NEEDED

&nbsp;  ✓ Field 1: Order Month = DATETRUNC('month', \[Order Date])

&nbsp;  ✓ Field 2: Customer Type = IF...THEN logic (see below)

&nbsp;  ✓ Field 3: Order Value = Already created in Sheet 1 ✓



3\. FILTERS

&nbsp;  ✓ Filter 1: Order Date (Year = 2024) 

&nbsp;             Show filter? YES

&nbsp;             Type: Dropdown



4\. VISUAL PROPERTIES

&nbsp;  ✓ Color: City

&nbsp;  ✓ Label: City + Average Rating

&nbsp;  ✓ Detail: Customer Type

&nbsp;  ✓ Tooltip: (default)



5\. SPECIAL CALCULATIONS

&nbsp;  ✓ LOD expression for Customer Type

&nbsp;  ✓ Average of Rating (1 decimal)



6\. FORMATTING

&nbsp;  ✓ Rating: 1 decimal place format



7\. OUTPUT

&nbsp;  ✓ Export: Revenue\_vs\_Experience.csv

```



---



\## \*\*Step 3: Decode Each Requirement\*\*



\### \*\*Requirement 1a: "Order Month = Truncate the month from Order Date"\*\*



\*\*Translation Process:\*\*



```

Business Language → Tableau Language



"Truncate the month" = Cut off everything except month

"from Order Date" = Use the Order Date field



Example:

If Order Date = 2024-03-15 14:30:25

Then Order Month = 2024-03-01 (first day of month)

```



\*\*Tableau Formula:\*\*

```

DATETRUNC('month', \[Order Date])

```



\*\*Why?\*\*

\- `DATETRUNC` = Truncate (cut off) parts of a date

\- `'month'` = Keep year and month, remove day/time

\- This groups all orders in March 2024 together



---



\### \*\*Requirement 1b: Customer Type Logic - The Tricky One!\*\*



\*\*Original:\*\*

> "Customer Type = Set flag 'New Customer' when minimum of Order Date for each Customer ID equals Order Date, otherwise set flag as 'Returning Customer'"



\*\*Let me break this down word by word:\*\*



\#### \*\*Analysis:\*\*



```

"Set flag" = Create a label/category

"New Customer when..." = IF condition is true

"minimum of Order Date" = First/earliest order date

"for each Customer ID" = Calculate separately per customer

"equals Order Date" = Matches the current row's order date

"otherwise" = ELSE

"Returning Customer" = Alternative label

```



\#### \*\*Visual Example:\*\*



Let's say we have this data:



| Customer ID | Order Date | Order Value |

|-------------|------------|-------------|

| C001 | 2024-01-05 | $50 |

| C001 | 2024-02-10 | $75 |

| C002 | 2024-01-15 | $100 |

| C001 | 2024-03-20 | $60 |



\*\*Step-by-step logic:\*\*



For \*\*Customer C001\*\*:

\- Minimum Order Date = 2024-01-05 (first order ever)



Row 1: Order Date = 2024-01-05

\- Does 2024-01-05 = 2024-01-05? \*\*YES\*\* → "New Customer" ✓



Row 2: Order Date = 2024-02-10

\- Does 2024-01-05 = 2024-02-10? \*\*NO\*\* → "Returning Customer"



Row 4: Order Date = 2024-03-20

\- Does 2024-01-05 = 2024-03-20? \*\*NO\*\* → "Returning Customer"



For \*\*Customer C002\*\*:

\- Minimum Order Date = 2024-01-15



Row 3: Order Date = 2024-01-15

\- Does 2024-01-15 = 2024-01-15? \*\*YES\*\* → "New Customer" ✓



\#### \*\*The Formula:\*\*



```tableau

IF {FIXED \[Customer ID]: MIN(\[Order Date])} = \[Order Date]

THEN "New Customer"

ELSE "Returning Customer"

END

```



\*\*Let me explain each part:\*\*



```

{FIXED \[Customer ID]: MIN(\[Order Date])}

&nbsp;└─┬─┘ └──────┬──────┘  └────┬────┘

&nbsp;  │          │              │

&nbsp;  │          │              └─ Find the earliest date

&nbsp;  │          └─ For each unique customer

&nbsp;  └─ LOD (Level of Detail) expression



= \[Order Date]

&nbsp; └─ Compare to current row's date



IF \[match]

THEN "New Customer"    ← First-time purchaser

ELSE "Returning Customer" ← Has ordered before

END

```



\*\*Why LOD?\*\*

\- We need to know "What's this customer's FIRST order?"

\- But we're looking at ALL their orders

\- LOD lets us calculate at CUSTOMER level while displaying at ORDER level



---



\### \*\*Requirement 2: Filter Logic\*\*



\*\*Original:\*\*

> "Filter the chart using the year of Order Data as '2024' and show the filter in dropdown."



\*\*Breaking it down:\*\*



```

Task Analysis:

┌─────────────────────────────────────┐

│ What to filter: Order Date          │

│ Filter level: Year (not day/month)  │

│ Filter value: 2024                  │

│ Visibility: Show on dashboard       │

│ Display type: Dropdown (not slider) │

└─────────────────────────────────────┘

```



\*\*Numbered Steps:\*\*

1\. Drag \[Order Date] to Filters shelf

2\. When dialog opens, select \*\*Years\*\*

3\. Check \*\*2024\*\* only (uncheck others)

4\. Click OK

5\. Right-click on the filter pill → \*\*Show Filter\*\*

6\. Click dropdown arrow on filter card → \*\*Single Value (dropdown)\*\*



\*\*Visual Guide:\*\*

```

Before filter:

Data from 2020, 2021, 2022, 2023, 2024, 2025...



After filter:

Data from 2024 only ✓



Filter card appearance:

┌──────────────┐

│ Order Date   │

│ 2024       ▼ │  ← Dropdown

└──────────────┘

```



---



\### \*\*Requirement 3: Color \& Label\*\*



\*\*Original:\*\*

> "Differentiate the chart using City in color and in Label to analyze the order value of each city."



\*\*Translation:\*\*



This is actually \*\*TWO separate actions\*\* in one sentence:



```

Sentence breakdown:



"Differentiate the chart using City in color"

&nbsp;  ↓

&nbsp;  Action 1: Drag \[City] to Color shelf

&nbsp;  Result: Each city gets a different color



"and in Label"

&nbsp;  ↓

&nbsp;  Action 2: Drag \[City] to Label shelf

&nbsp;  Result: City name appears on each area



"to analyze the order value of each city"

&nbsp;  ↓

&nbsp;  Purpose: User can see which colored area = which city

```



\*\*Why both?\*\*

\- \*\*Color alone:\*\* User sees different areas but doesn't know which is which

\- \*\*Label alone:\*\* No visual differentiation

\- \*\*Both together:\*\* Color + name = clear identification ✓



---



\### \*\*Requirement 4: Detail \& Additional Label\*\*



\*\*Original:\*\*

> "Use Customer Type in detail and average of Rating in label for further analysis."



\*\*Breaking it down:\*\*



```

This sentence has 2 parts:



Part 1: "Customer Type in detail"

&nbsp;  └─ Drag \[Customer Type] to Detail shelf

&nbsp;  

Part 2: "average of Rating in label"

&nbsp;  └─ Drag \[Rating] to Label shelf

&nbsp;  └─ Change aggregation to AVG

&nbsp;  └─ Format to 1 decimal place

```



\*\*What does "Detail" do?\*\*

\- It affects the granularity (level of detail) of the visualization

\- Without Customer Type in Detail: One line per City per Month

\- With Customer Type in Detail: Data split by New vs Returning customers

\- But it doesn't show visually (unlike Color or Label)



\*\*Visual Example:\*\*



```

Without Customer Type in Detail:

Month: Jan | Order Value: $10,000 (all customers combined)



With Customer Type in Detail:

Month: Jan | Order Value: $10,000

&nbsp;           ├─ New Customers: $4,000

&nbsp;           └─ Returning Customers: $6,000

```



---



\## \*\*Step 4: The Complete Build Sequence\*\*



Here's the exact order I would build this in Tableau:



```

BUILD ORDER (Follow this sequence):



□ 1. Create calculated field: Order Month

□ 2. Create calculated field: Customer Type

□ 3. Drag \[Order Month] to Columns

□ 4. Drag \[Order Value] to Rows

□ 5. Change mark type to Area

□ 6. Drag \[Order Date] to Filters → Year → 2024

□ 7. Show filter → Change to dropdown

□ 8. Drag \[City] to Color

□ 9. Drag \[City] to Label

□ 10. Drag \[Customer Type] to Detail

□ 11. Drag \[Rating] to Label (as AVG)

□ 12. Format AVG(Rating) to 1 decimal

□ 13. Set chart title

□ 14. Export data



DONE! ✓

```



---



\# 🔍 \*\*Sheet 3: Restaurant\_Performance - Complete Breakdown\*\*



Now let's tackle the most complex sheet!



\## \*\*Step 1: Read \& Highlight\*\*



> Sheet 3: Restaurant\_Performance

> 

> Chart Title: Restaurant Month Wise Performance

> 

> Rows: Restaurant Name

> Columns: Month(Order Month)

> Mark: Square

> 

> 1. \*\*Create\*\* Parameter:

>    a. Select Metric: \*\*Create\*\* a list of string values "Average Rating", "Total Revenue" and "Total Orders"

> 

> 2. \*\*Create\*\* a calculated field named as "Selected Measure" with the following conditions: \[table with conditions]

> 

> 3. \*\*Filter\*\* the chart using Category as "Fast Food" and \*\*show\*\* the filter in dropdown.

> 

> 4. \*\*Differentiate\*\* the chart using Selected Measure in color and in Label.

> 

> 5. \*\*Order\*\* the Restaurant Name using average of Rating in descending order.

> 

> 6. \*\*Change\*\* the color for Selected Measure to Temperature Diverging and \*\*add\*\* borders with black color.



---



\## \*\*Step 2: Fill Out Template\*\*



```

SHEET NAME: Restaurant\_Performance



1\. BASIC STRUCTURE

&nbsp;  ✓ Chart title: Restaurant Month Wise Performance

&nbsp;  ✓ Chart type: Square (heat map style)

&nbsp;  ✓ Rows (Y-axis): Restaurant Name

&nbsp;  ✓ Columns (X-axis): Month (Order Month)



2\. CALCULATED FIELDS NEEDED

&nbsp;  ✓ Field 1: Order Month = (already created in Sheet 2)

&nbsp;  ✓ Field 2: Selected Measure = CASE statement (see below)



3\. PARAMETERS NEEDED

&nbsp;  ✓ Parameter 1: Select Metric (String list)

&nbsp;                Values: "Average Rating", "Total Revenue", "Total Orders"



4\. FILTERS

&nbsp;  ✓ Filter 1: Category = "Fast Food"

&nbsp;             Show filter? YES

&nbsp;             Type: Dropdown



5\. VISUAL PROPERTIES

&nbsp;  ✓ Color: Selected Measure (Temperature Diverging palette)

&nbsp;  ✓ Label: Selected Measure

&nbsp;  ✓ Borders: Black



6\. SORTING

&nbsp;  ✓ Restaurant Name: Sort by AVG(Rating), Descending



7\. OUTPUT

&nbsp;  ✓ Export: Restaurant\_Performance.csv

```



---



\## \*\*Step 3: Decode Complex Requirements\*\*



\### \*\*Requirement 1: Create Parameter\*\*



\*\*Original:\*\*

> "Select Metric: Create a list of string values 'Average Rating', 'Total Revenue' and 'Total Orders'"



\*\*What's a Parameter?\*\*



Think of it as a \*\*dropdown menu that users can interact with\*\* to change what the chart shows.



\*\*Visual concept:\*\*



```

User sees this control:

┌─────────────────────┐

│ Select Metric     ▼ │

├─────────────────────┤

│ ○ Average Rating    │

│ ○ Total Revenue     │

│ ○ Total Orders      │

└─────────────────────┘



When user selects "Total Revenue":

→ Chart updates to show revenue values

→ Colors change based on revenue

```



\*\*Step-by-step creation:\*\*



```

1\. Right-click in Data pane

2\. Select "Create Parameter"

3\. Fill in dialog:

&nbsp;  

&nbsp;  Name: Select Metric

&nbsp;  

&nbsp;  Data type: String

&nbsp;  

&nbsp;  Allowable values: List

&nbsp;  

&nbsp;  List of values:

&nbsp;  ┌──────────────────┐

&nbsp;  │ Average Rating   │

&nbsp;  │ Total Revenue    │

&nbsp;  │ Total Orders     │

&nbsp;  └──────────────────┘

&nbsp;  

&nbsp;  Current value: Average Rating (default)



4\. Click OK

5\. Right-click parameter → Show Parameter Control

```



---



\### \*\*Requirement 2: Dynamic Calculated Field - The Brain of This Sheet!\*\*



\*\*Original:\*\*

> "Create a calculated field named as 'Selected Measure' with the following conditions:"



| Select Metric | Flag |

|--------------|------|

| Average Rating | AVG(Rating) |

| Total Revenue | SUM(Order Value) |

| Total Orders | COUNTD(Order Id) |



\*\*What does this mean?\*\*



This creates a \*\*dynamic field that changes based on what the user selects\*\* in the parameter.



\*\*Concept:\*\*



```

IF user chooses "Average Rating"

&nbsp;  THEN show AVG(Rating)



IF user chooses "Total Revenue"

&nbsp;  THEN show SUM(Order Value)



IF user chooses "Total Orders"

&nbsp;  THEN show COUNTD(Order Id)

```



\*\*The Formula:\*\*



```tableau

CASE \[Select Metric]

&nbsp;   WHEN "Average Rating" THEN AVG(\[Rating])

&nbsp;   WHEN "Total Revenue" THEN SUM(\[Order Value])

&nbsp;   WHEN "Total Orders" THEN COUNTD(\[Order Id])

END

```



\*\*Let me explain CASE statement:\*\*



```

CASE \[Parameter Name]

&nbsp;    └─ Check what value the parameter has



WHEN "Value 1" THEN \[Do This]

&nbsp;    └─ If parameter = "Value 1", return this calculation



WHEN "Value 2" THEN \[Do That]

&nbsp;    └─ If parameter = "Value 2", return this calculation



END

└─ Close the statement

```



\*\*Real Example:\*\*



```

Restaurant: Pizza Hut

Month: January

Parameter selected: "Total Revenue"



CASE \[Select Metric]                      ← Check parameter

&nbsp;   WHEN "Average Rating" THEN AVG(\[Rating])  ← Skip (not selected)

&nbsp;   WHEN "Total Revenue" THEN SUM(\[Order Value]) ← USE THIS! ✓

&nbsp;   WHEN "Total Orders" THEN COUNTD(\[Order Id])  ← Skip (not selected)

END



Result: Returns $15,450 (sum of all orders for Pizza Hut in Jan)

```



---



\### \*\*Requirement 5: Sorting Logic\*\*



\*\*Original:\*\*

> "Order the Restaurant Name using average of Rating in descending order."



\*\*Translation:\*\*



```

What to sort: Restaurant Name (on Rows)

How to sort: By a different field (not alphabetically)

Which field: Rating

How to aggregate: Average

What order: Descending (high to low)



Result: 

\- Restaurant with highest avg rating appears first

\- Restaurant with lowest avg rating appears last

```



\*\*Step-by-step:\*\*



```

1\. Right-click on "Restaurant Name" in Rows shelf

2\. Select "Sort..."

3\. Dialog opens - fill it:



&nbsp;  Sort By: Field

&nbsp;  

&nbsp;  Sort Order: Descending

&nbsp;  

&nbsp;  Field Name: Rating

&nbsp;  

&nbsp;  Aggregation: Average



4\. Click OK

```



\*\*Visual Result:\*\*



```

Before sorting (alphabetical):

\- Burger King (3.5★)

\- Dominos (4.2★)

\- KFC (4.5★)

\- McDonald's (3.8★)



After sorting (by rating, descending):

\- KFC (4.5★)         ← Highest rated on top

\- Dominos (4.2★)

\- McDonald's (3.8★)

\- Burger King (3.5★) ← Lowest rated at bottom

```



---



\### \*\*Requirement 6: Visual Formatting\*\*



\*\*Original:\*\*

> "Change the color for Selected Measure to Temperature Diverging and add borders with black color."



\*\*This has TWO formatting actions:\*\*



\#### \*\*Action 1: Change Color Palette\*\*



```

1\. Click on Color in Marks card

2\. Click "Edit Colors..."

3\. Palette dropdown → Select "Temperature Diverging"

4\. Click OK



What is Temperature Diverging?

\- Blue (cold) = Low values

\- White = Middle values

\- Red (hot) = High values



Visual:

Low ■■■■■■■■■■■■■ High

&nbsp;   Blue → Red

```



\#### \*\*Action 2: Add Borders\*\*



```

1\. Click on Color in Marks card

2\. Click "Border"

3\. Select black color

4\. Borders now appear around each square



Visual:

Without borders:    With borders:

■■■■               ┌─┬─┬─┐

■■■■               ├─┼─┼─┤

■■■■               └─┴─┴─┘

```



---



\## \*\*Step 4: Complete Build Sequence for Sheet 3\*\*



```

BUILD ORDER:



□ 1. Create Parameter: Select Metric

□ 2. Show Parameter Control

□ 3. Create calculated field: Selected Measure

□ 4. Drag \[Restaurant Name] to Rows

□ 5. Drag \[Order Month] to Columns (as Month)

□ 6. Change mark type to Square

□ 7. Drag \[Selected Measure] to Color

□ 8. Drag \[Selected Measure] to Label

□ 9. Drag \[Category] to Filters → "Fast Food"

□ 10. Show filter → Change to dropdown

□ 11. Sort Restaurant Name by AVG(Rating), Descending

□ 12. Edit Colors → Temperature Diverging

□ 13. Add black borders

□ 14. Set chart title

□ 15. Export data



DONE! ✓

```



---



\# 🎯 \*\*Key Patterns You Should Memorize\*\*



\## \*\*Pattern 1: Date Truncation\*\*



```

Requirement phrase → Tableau function



"Truncate to month" → DATETRUNC('month', \[Date])

"Truncate to year"  → DATETRUNC('year', \[Date])

"Truncate to day"   → DATETRUNC('day', \[Date])

```



\## \*\*Pattern 2: Conditional Logic\*\*



```

"When X, otherwise Y" → IF-THEN-ELSE



IF \[condition]

THEN \[value if true]

ELSE \[value if false]

END

```



\## \*\*Pattern 3: LOD (Level of Detail)\*\*



```

"For each \[Group]" → {FIXED \[Group]: ...}



Example:

"For each Customer" → {FIXED \[Customer ID]: MIN(\[Order Date])}

"For each City"     → {FIXED \[City]: SUM(\[Sales])}

```



\## \*\*Pattern 4: Dynamic Calculations\*\*



```

"Based on selection" → Parameter + CASE statement



CASE \[Parameter]

&nbsp;   WHEN "Option 1" THEN \[Calc 1]

&nbsp;   WHEN "Option 2" THEN \[Calc 2]

END

```



---



\# 📝 \*\*Practice Challenge for You\*\*



Try breaking down this requirement using my method:



> "Create a calculated field named 'Sales Category' that flags 'High' when Order Value is greater than the average Order Value for that City, otherwise flags 'Low'."



\*\*Your task:\*\*

1\. Identify the IF-THEN-ELSE structure

2\. Identify if LOD is needed

3\. Write the formula



<details>

<summary>Click to see the answer</summary>



\*\*Breakdown:\*\*

```

Condition: Order Value > Average Order Value per City

True: "High"

False: "Low"



Need LOD? YES - "for that City" means calculate average at City level

```



\*\*Formula:\*\*

```tableau

IF \[Order Value] > {FIXED \[City]: AVG(\[Order Value])}

THEN "High"

ELSE "Low"

END

```



</details>



---



\*\*Do you want me to:\*\*

1\. Create a practice worksheet with more examples?

2\. Explain any specific part in more detail?

3\. Show you how to debug when things don't work?



Let me know what would help you most! 🚀

