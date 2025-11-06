# MODULE 3: TRANSFORMATIONS - DEEP DIVE

## Overview: Transformation Essentials

**Recap:** Transformations are the "processing engine" where business logic is applied to data flowing from source to target.

**Module 3 Structure:**
- **Topic 9:** Passive Transformations (Expression, Filter, Router, Sequence Generator)
- **Topic 10:** Active Transformations (Aggregator, Sorter, Rank)
- **Topic 11:** Lookup Transformation
- **Topic 12:** Joiner Transformation
- **Topic 13:** Union and Normalizer
- **Topic 14:** Update Strategy Transformation
- **Topic 15:** Stored Procedure and SQL Transformation

---

## Topic 9: Passive Transformations

### EXPRESSION TRANSFORMATION 🔥 (MOST IMPORTANT)

**Type:** Passive, Connected  
**Purpose:** Calculate values, perform data conversions, string manipulations, date operations  
**Usage:** 80% of mappings use Expression transformation

---

#### Port Types in Expression

1. **Input Port (I):** Receives data from previous transformation
2. **Output Port (O):** Sends data to next transformation
3. **Variable Port (V):** Intermediate calculations (not passed forward)

**Port Flow:**
```
Input Ports → Variable Ports → Output Ports
   (I)            (V)              (O)
```

**Example Structure:**
```
Expression: EXP_CALCULATE_BONUS
┌─────────────────────────────────┐
│ I  EMP_NAME                     │ ← Input
│ I  SALARY                       │ ← Input
│ V  v_BonusPercent               │ ← Variable (intermediate)
│ V  v_BonusAmount                │ ← Variable (intermediate)
│ O  O_EMP_NAME_UPPER             │ ← Output
│ O  O_TOTAL_COMPENSATION         │ ← Output
└─────────────────────────────────┘
```

**Why Variable Ports?**
- Break complex calculations into steps
- Reuse values in multiple outputs
- Improve readability

**Example:**
```
Variable v_BonusPercent = IIF(SALARY > 80000, 0.15, 0.10)
Variable v_BonusAmount = SALARY * v_BonusPercent
Output O_TOTAL = SALARY + v_BonusAmount
```

---

#### Expression Functions - Key Categories

**1. String Functions**

| Function | Purpose | Example |
|----------|---------|---------|
| `UPPER(string)` | Convert to uppercase | `UPPER('john')` → 'JOHN' |
| `LOWER(string)` | Convert to lowercase | `LOWER('JOHN')` → 'john' |
| `INITCAP(string)` | Capitalize first letter | `INITCAP('john smith')` → 'John Smith' |
| `LTRIM(string)` | Remove leading spaces | `LTRIM('  abc')` → 'abc' |
| `RTRIM(string)` | Remove trailing spaces | `RTRIM('abc  ')` → 'abc' |
| `SUBSTR(string, start, length)` | Extract substring | `SUBSTR('Informatica', 1, 6)` → 'Inform' |
| `LENGTH(string)` | Get string length | `LENGTH('Hello')` → 5 |
| `INSTR(string, search)` | Find position | `INSTR('Informatica', 'for')` → 3 |
| `CONCAT(str1, str2)` | Concatenate | `CONCAT('Hello', ' World')` → 'Hello World' |
| `REPLACE(string, old, new)` | Replace text | `REPLACE('Hi Tom', 'Tom', 'John')` → 'Hi John' |

**2. Numeric Functions**

| Function | Purpose | Example |
|----------|---------|---------|
| `ROUND(number, precision)` | Round number | `ROUND(123.456, 2)` → 123.46 |
| `TRUNC(number, precision)` | Truncate | `TRUNC(123.456, 2)` → 123.45 |
| `CEIL(number)` | Round up | `CEIL(12.3)` → 13 |
| `FLOOR(number)` | Round down | `FLOOR(12.8)` → 12 |
| `ABS(number)` | Absolute value | `ABS(-10)` → 10 |
| `POWER(base, exp)` | Exponentiation | `POWER(2, 3)` → 8 |
| `SQRT(number)` | Square root | `SQRT(16)` → 4 |
| `MOD(dividend, divisor)` | Modulus | `MOD(10, 3)` → 1 |

**3. Date Functions**

| Function | Purpose | Example |
|----------|---------|---------|
| `SYSDATE` | Current system date | `SYSDATE` → 2025-11-06 |
| `ADD_TO_DATE(date, unit, value)` | Add to date | `ADD_TO_DATE(SYSDATE, 'DD', 7)` → +7 days |
| `TO_DATE(string, format)` | Convert to date | `TO_DATE('2025-01-15', 'YYYY-MM-DD')` |
| `TO_CHAR(date, format)` | Convert to string | `TO_CHAR(SYSDATE, 'DD-MON-YYYY')` → '06-NOV-2025' |
| `LAST_DAY(date)` | Last day of month | `LAST_DAY('2025-02-10')` → '2025-02-28' |
| `TRUNC(date, unit)` | Truncate date | `TRUNC(SYSDATE, 'MM')` → First of month |

**Date Format Codes:**
- `YYYY` - 4-digit year
- `MM` - Month (01-12)
- `DD` - Day (01-31)
- `HH24` - Hour (00-23)
- `MI` - Minute (00-59)
- `SS` - Second (00-59)

**4. Conditional Functions**

**IIF (Immediate IF)** - Most Important!
```
Syntax: IIF(condition, value_if_true, value_if_false)

Example 1: IIF(SALARY > 50000, 'High', 'Low')
Example 2: IIF(ISNULL(COMMISSION), 0, COMMISSION)
```

**DECODE** - Multi-condition switch
```
Syntax: DECODE(expression, value1, result1, value2, result2, ..., default)

Example:
DECODE(DEPT_CODE,
  'IT', 'Information Technology',
  'HR', 'Human Resources',
  'FIN', 'Finance',
  'Unknown Department')
```

**Nested IIF:**
```
IIF(SALARY > 80000, 'Senior',
  IIF(SALARY > 50000, 'Mid',
    IIF(SALARY > 30000, 'Junior', 'Entry')))
```

**5. NULL Handling Functions**

| Function | Purpose | Example |
|----------|---------|---------|
| `ISNULL(value)` | Check if NULL | `ISNULL(COMMISSION)` → TRUE/FALSE |
| `IIF(ISNULL(value), default, value)` | Replace NULL | `IIF(ISNULL(SALARY), 0, SALARY)` |
| `NULLVALUE(value, default)` | Replace NULL (shorthand) | `NULLVALUE(COMMISSION, 0)` |

**6. Conversion Functions**

| Function | Purpose | Example |
|----------|---------|---------|
| `TO_INTEGER(value)` | Convert to integer | `TO_INTEGER('123')` → 123 |
| `TO_DECIMAL(value, scale)` | Convert to decimal | `TO_DECIMAL('12.34', 2)` |
| `TO_CHAR(value)` | Convert to string | `TO_CHAR(123)` → '123' |

---

#### Real-World Expression Scenarios

**Scenario 1: Data Cleansing**
```
Clean employee names and handle NULL salaries:

I  EMP_NAME (input)
V  v_CleanName = LTRIM(RTRIM(UPPER(EMP_NAME)))
I  SALARY (input)
O  O_EMP_NAME = v_CleanName
O  O_SALARY = IIF(ISNULL(SALARY), 0, SALARY)
```

**Scenario 2: Calculated Columns**
```
Calculate total compensation with bonus:

I  SALARY
I  COMMISSION
V  v_BonusPercent = IIF(SALARY > 80000, 0.15, IIF(SALARY > 50000, 0.10, 0.05))
V  v_Bonus = SALARY * v_BonusPercent
V  v_TotalComm = IIF(ISNULL(COMMISSION), 0, COMMISSION)
O  O_TOTAL_COMPENSATION = SALARY + v_Bonus + v_TotalComm
```

**Scenario 3: Date Manipulations**
```
Calculate age and seniority:

I  BIRTH_DATE
I  HIRE_DATE
V  v_CurrentDate = SYSDATE
O  O_AGE = TRUNC((v_CurrentDate - BIRTH_DATE) / 365.25)
O  O_YEARS_OF_SERVICE = TRUNC((v_CurrentDate - HIRE_DATE) / 365.25)
O  O_LOAD_DATE = TO_CHAR(v_CurrentDate, 'YYYY-MM-DD HH24:MI:SS')
```

**Scenario 4: Categorization**
```
Categorize employees by salary band:

I  SALARY
O  O_SALARY_BAND = 
    IIF(SALARY > 100000, 'Executive',
      IIF(SALARY > 80000, 'Senior',
        IIF(SALARY > 50000, 'Mid-Level',
          IIF(SALARY > 30000, 'Junior', 'Entry-Level'))))
```

---

#### Expression Transformation Properties

**General Tab:**
- Transformation Name
- Description
- Type: Passive

**Ports Tab:**
- Add/Remove/Edit ports
- Set port types (I, O, V)
- Define expressions

**Properties Tab:**
- **Tracing Level:** Normal, Terse, Verbose (for debugging)

---

### FILTER TRANSFORMATION 🔥

**Type:** Active, Connected  
**Purpose:** Filter rows based on condition (like SQL WHERE clause)  
**Row Behavior:** Removes rows that don't meet condition

---

#### How Filter Works

**Input:** 100 rows  
**Filter Condition:** `SALARY > 50000`  
**Output:** Only rows where SALARY > 50000 (maybe 30 rows)

**Visual:**
```
Source (100 rows)
    ↓
Filter: SALARY > 50000
    ↓
Target (30 rows) ← Only matching rows pass through
```

---

#### Filter Condition Syntax

**Simple Conditions:**
```
SALARY > 50000
STATUS = 'A'
DEPT_ID = 10
HIRE_DATE > TO_DATE('2020-01-01', 'YYYY-MM-DD')
```

**Complex Conditions (AND/OR):**
```
SALARY > 50000 AND DEPT_ID = 10
STATUS = 'A' OR STATUS = 'P'
SALARY > 80000 AND (DEPT = 'IT' OR DEPT = 'Sales')
```

**Using Functions:**
```
ISNULL(COMMISSION)  ← Keep only NULL commission rows
LENGTH(EMP_NAME) > 10
SUBSTR(EMP_CODE, 1, 2) = 'IT'
```

---

#### Filter vs Source Qualifier Filter

| Aspect | Filter Transformation | Source Qualifier Filter |
|--------|----------------------|------------------------|
| Location | In Informatica (after extraction) | At source database |
| Performance | Slower (filters after extraction) | Faster (filters at source) |
| When to Use | Complex conditions, multiple sources | Simple SQL WHERE conditions |
| Flexibility | Can use any transformation functions | Limited to SQL syntax |

**Best Practice:** Use Source Qualifier filter when possible (better performance).

---

#### Real-World Filter Scenarios

**Scenario 1: Active Employees Only**
```
Filter Condition: STATUS = 'A'
Use Case: Load only currently active employees
```

**Scenario 2: Recent Hires**
```
Filter Condition: HIRE_DATE >= ADD_TO_DATE(SYSDATE, 'DD', -90)
Use Case: Employees hired in last 90 days
```

**Scenario 3: High Earners in Specific Department**
```
Filter Condition: SALARY > 80000 AND DEPT_ID IN (10, 20, 30)
Use Case: Senior employees in IT, Sales, Finance
```

**Scenario 4: Exclude Invalid Records**
```
Filter Condition: NOT ISNULL(EMP_ID) AND LENGTH(EMP_NAME) > 0
Use Case: Data quality - remove rows with missing critical data
```

---

### ROUTER TRANSFORMATION 🔥

**Type:** Active, Connected  
**Purpose:** Route data to multiple targets/paths based on conditions  
**Think of it as:** Multiple filters in one transformation

---

#### Router vs Filter

| Feature | Filter | Router |
|---------|--------|--------|
| Output Groups | 1 (pass/fail) | Multiple (many paths) |
| Use Case | Simple filtering | Multi-path routing |
| Default Group | No | Yes (catches unmatched rows) |

**When to Use Router:**
- Need to send data to different targets based on conditions
- Multiple categories/segments
- Want to capture unmatched rows

---

#### Router Structure

**Components:**
1. **Input Group:** Receives all incoming rows
2. **Output Groups:** Multiple groups with conditions (like CASE WHEN)
3. **Default Group:** Catches rows that don't match any condition

**Visual:**
```
Source (100 rows)
    ↓
Router
    ├─→ Group 1: SALARY > 80000  (20 rows) → Senior_Target
    ├─→ Group 2: SALARY 50K-80K  (40 rows) → Mid_Target
    ├─→ Group 3: SALARY < 50000  (30 rows) → Junior_Target
    └─→ Default Group: (10 rows) → Error_Target
```

---

#### Creating Router Groups

**Example: Salary-Based Routing**

**Group 1: HIGH_SALARY**
- Condition: `SALARY > 80000`
- Target: SENIOR_EMP

**Group 2: MID_SALARY**
- Condition: `SALARY >= 50000 AND SALARY <= 80000`
- Target: MID_EMP

**Group 3: LOW_SALARY**
- Condition: `SALARY < 50000`
- Target: JUNIOR_EMP

**Default Group:**
- No condition (catches NULL salaries, invalid data)
- Target: ERROR_LOG

---

#### Router Properties

**Group Priority:**
- Conditions evaluated in order
- First matching condition wins
- Row goes to ONLY ONE group (not multiple)

**Important:** Make conditions mutually exclusive!

**Bad Example:**
```
Group 1: SALARY > 50000  ← Matches 80K
Group 2: SALARY > 80000  ← Will never get 80K+ (Group 1 already caught them)
```

**Good Example:**
```
Group 1: SALARY > 80000
Group 2: SALARY >= 50000 AND SALARY <= 80000
Group 3: SALARY < 50000
```

---

#### Real-World Router Scenarios

**Scenario 1: Customer Segmentation**
```
Router: RTR_CUSTOMER_SEGMENT

Group PREMIUM: ANNUAL_SPEND > 100000 → Premium_Customers
Group GOLD: ANNUAL_SPEND >= 50000 AND < 100000 → Gold_Customers
Group SILVER: ANNUAL_SPEND >= 20000 AND < 50000 → Silver_Customers
Group BRONZE: ANNUAL_SPEND < 20000 → Bronze_Customers
Default: → Unclassified_Customers
```

**Scenario 2: Geographic Routing**
```
Router: RTR_REGION

Group NORTH: REGION = 'N' → North_Sales
Group SOUTH: REGION = 'S' → South_Sales
Group EAST: REGION = 'E' → East_Sales
Group WEST: REGION = 'W' → West_Sales
Default: → Invalid_Region
```

**Scenario 3: Transaction Type Routing**
```
Router: RTR_TRANSACTION_TYPE

Group SALES: TXN_TYPE = 'SALE' → Sales_Fact
Group RETURNS: TXN_TYPE = 'RETURN' → Returns_Fact
Group EXCHANGES: TXN_TYPE = 'EXCHANGE' → Exchange_Fact
Default: → Unknown_Transactions
```

---

### SEQUENCE GENERATOR TRANSFORMATION

**Type:** Passive, Connected  
**Purpose:** Generate unique numeric values (surrogate keys)  
**Common Use:** Create primary keys for data warehouse dimension/fact tables

---

#### Why Sequence Generator?

**Business Need:**
- Source systems might not have proper keys
- Need to generate unique IDs for target tables
- Create surrogate keys (business-independent keys)

**Example:**
```
Source Data (no unique key):
Name: John Smith, Dept: IT
Name: John Smith, Dept: Sales  ← Same name, different person!

After Sequence Generator:
EMP_SK: 1, Name: John Smith, Dept: IT
EMP_SK: 2, Name: John Smith, Dept: Sales
```

---

#### Sequence Generator Properties

**Key Properties:**

1. **Start Value:** Initial number (default: 0)
2. **Increment By:** Step size (default: 1)
3. **End Value:** Maximum value (optional)
4. **Current Value:** Tracking field
5. **Cycle:** Restart after reaching end value? (Yes/No)
6. **Number of Cached Values:** Performance optimization

**Example Configuration:**
```
Start Value: 1
Increment By: 1
End Value: 999999999
Cycle: No
Cache: 1000
```

**Generated Sequence:**
```
Row 1: NEXTVAL = 1
Row 2: NEXTVAL = 2
Row 3: NEXTVAL = 3
...
```

---

#### Sequence Generator Ports

**Output Ports (2):**

1. **NEXTVAL:** Returns next value in sequence (most used)
2. **CURRVAL:** Returns current value (last generated)

**Usage:**
```
Mapping Flow:
Source → Sequence Generator → Expression → Target
            ↓
         NEXTVAL (1, 2, 3, ...)
            ↓
         Maps to Target.EMP_SK
```

---

#### Real-World Scenarios

**Scenario 1: Dimension Table Surrogate Key**
```
Source: Customer data from CRM (no reliable key)
Sequence Generator: SQ_CUSTOMER_SK
  Start: 1, Increment: 1
Output: Maps to DIM_CUSTOMER.CUSTOMER_SK

Result:
CUSTOMER_SK | CUSTOMER_NAME | PHONE
1           | John Smith    | 555-1234
2           | Jane Doe      | 555-5678
```

**Scenario 2: Fact Table Load**
```
Generate unique transaction IDs:
Sequence Generator: SQ_SALES_ID
Output: SALES_ID (maps to FACT_SALES.SALES_ID)
```

---

#### Sequence Generator Best Practices

**When to Use:**
- Generating surrogate keys for dimensions
- Creating unique identifiers when source lacks them
- Fact table sequence numbers

**Performance:**
- Set appropriate cache size (1000-10000)
- Higher cache = fewer repository hits = better performance
- But cache lost on session failure (gaps in sequence)

**Multiple Sessions:**
- Each session gets its own range from cache
- Session 1: 1-1000
- Session 2: 1001-2000
- No duplicates across sessions

---

## Topic 10: Active Transformations

### AGGREGATOR TRANSFORMATION 🔥 (HIGH PRIORITY)

**Type:** Active, Connected  
**Purpose:** Perform aggregate calculations (SUM, AVG, COUNT, MIN, MAX)  
**Row Behavior:** Groups rows and outputs one row per group

---

#### How Aggregator Works

**Concept:** Like SQL GROUP BY

**Example:**
```
Input (5 rows):
DEPT    | SALARY
IT      | 50000
IT      | 60000
HR      | 45000
HR      | 48000
Sales   | 55000

Aggregator: Group by DEPT, SUM(SALARY)

Output (3 rows):
DEPT    | TOTAL_SALARY
IT      | 110000
HR      | 93000
Sales   | 55000
```

---

#### Aggregate Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `SUM(port)` | Total of values | `SUM(SALARY)` |
| `AVG(port)` | Average | `AVG(SALARY)` |
| `COUNT(*)` | Count all rows | `COUNT(*)` |
| `COUNT(port)` | Count non-null | `COUNT(COMMISSION)` |
| `MIN(port)` | Minimum value | `MIN(SALARY)` |
| `MAX(port)` | Maximum value | `MAX(SALARY)` |
| `FIRST(port)` | First value in group | `FIRST(EMP_NAME)` |
| `LAST(port)` | Last value in group | `LAST(EMP_NAME)` |
| `STDDEV(port)` | Standard deviation | `STDDEV(SALARY)` |
| `VARIANCE(port)` | Variance | `VARIANCE(SALARY)` |
| `MEDIAN(port)` | Median value | `MEDIAN(SALARY)` |
| `PERCENTILE(port, n)` | Nth percentile | `PERCENTILE(SALARY, 90)` |

---

#### Aggregator Port Types

**Port Configuration:**

1. **Group By Ports:** Define grouping columns
   - Check "Group By" property for the port
   - Like SQL GROUP BY columns

2. **Aggregate Expression Ports:** Calculate aggregates
   - Use aggregate functions
   - Output ports with expressions

**Example Structure:**
```
Aggregator: AGG_DEPT_SALARY

Input/Output Ports:
├─ DEPT_ID (Group By = TRUE) ← Grouping column
├─ SALARY (Group By = FALSE) ← Will aggregate this
└─ TOTAL_SALARY = SUM(SALARY) ← Output with aggregate
└─ AVG_SALARY = AVG(SALARY) ← Output
└─ EMP_COUNT = COUNT(*) ← Output
```

---

#### Creating Aggregator Step-by-Step

**Scenario:** Calculate total and average salary per department

**Steps:**

1. **Create Aggregator Transformation**
   - Name: `AGG_DEPT_STATS`

2. **Add Ports:**
   - Input: `DEPT_ID` → Set Group By = TRUE
   - Input: `SALARY` → Set Group By = FALSE
   - Output: `TOTAL_SALARY` → Expression: `SUM(SALARY)`
   - Output: `AVG_SALARY` → Expression: `AVG(SALARY)`
   - Output: `EMP_COUNT` → Expression: `COUNT(*)`

3. **Connect in Mapping:**
   ```
   Source → Aggregator → Target
   ```

4. **Result:**
   ```
   DEPT_ID | TOTAL_SALARY | AVG_SALARY | EMP_COUNT
   10      | 250000       | 62500      | 4
   20      | 180000       | 60000      | 3
   30      | 320000       | 64000      | 5
   ```

---

#### Aggregator Cache

**How It Works:**
- Aggregator stores data in cache during processing
- Groups and calculates in memory
- Writes results at end

**Cache Properties:**

1. **Cache Directory:** Where temp cache files stored
2. **Cache Size:** Memory allocated for caching
3. **Sorted Input:** Optimization (explained below)

---

#### Sorted Input Optimization 🔥 (EXAM IMPORTANT)

**Concept:** If data is pre-sorted by group columns, Aggregator can use less memory.

**Without Sorted Input:**
```
Input (Random Order):
DEPT: IT, HR, IT, Sales, HR, IT
↓
Aggregator holds ALL groups in memory simultaneously
```

**With Sorted Input:**
```
Input (Sorted by DEPT):
DEPT: HR, HR, IT, IT, IT, Sales
↓
Aggregator processes one group at a time, releases memory
↓
Better performance, less memory
```

**How to Enable:**
1. Add **Sorter Transformation** before Aggregator (sort by group columns)
2. Set **Sorted Input = TRUE** in Aggregator properties

**Performance Impact:**
- Reduces memory usage by 70-90%
- Essential for large data volumes
- Critical for exam scenarios!

---

#### Real-World Aggregator Scenarios

**Scenario 1: Sales Summary by Region**
```
Input: Daily sales transactions
Aggregator: AGG_REGION_SALES
  Group By: REGION, YEAR, MONTH
  Aggregates:
    - TOTAL_SALES = SUM(SALES_AMOUNT)
    - TOTAL_QTY = SUM(QUANTITY)
    - AVG_SALE = AVG(SALES_AMOUNT)
    - TRANSACTION_COUNT = COUNT(*)
Output: Monthly regional sales summary
```

**Scenario 2: Employee Statistics by Department**
```
Aggregator: AGG_DEPT_STATS
  Group By: DEPT_ID
  Aggregates:
    - HEADCOUNT = COUNT(*)
    - TOTAL_PAYROLL = SUM(SALARY)
    - AVG_SALARY = AVG(SALARY)
    - MIN_SALARY = MIN(SALARY)
    - MAX_SALARY = MAX(SALARY)
```

**Scenario 3: Product Performance**
```
Aggregator: AGG_PRODUCT_PERF
  Group By: PRODUCT_ID, QUARTER
  Aggregates:
    - TOTAL_REVENUE = SUM(REVENUE)
    - TOTAL_COST = SUM(COST)
    - PROFIT = SUM(REVENUE) - SUM(COST)
    - MARGIN_PCT = (SUM(REVENUE) - SUM(COST)) / SUM(REVENUE) * 100
```

---

### SORTER TRANSFORMATION

**Type:** Active, Connected  
**Purpose:** Sort data in ascending or descending order  
**Use Cases:** Prepare data for Aggregator, remove duplicates, sorting requirements

---

#### Sorter Properties

**Sort Keys:**
- Select which columns to sort by
- Specify direction (Ascending/Descending)
- Define sort order (Primary, Secondary, Tertiary)

**Example:**
```
Sort by DEPT_ID (Ascending), then SALARY (Descending)

Result:
DEPT_ID | SALARY
10      | 80000 ← Highest in Dept 10
10      | 65000
10      | 50000
20      | 90000 ← Highest in Dept 20
20      | 75000
```

---

#### Sorter Options

**1. Distinct Output Rows:**
- Removes duplicate rows
- Like SQL DISTINCT
- Keeps only unique combinations

**Example:**
```
Input:
IT, Manager
IT, Manager  ← Duplicate
HR, Manager

Output (Distinct = TRUE):
IT, Manager
HR, Manager
```

**2. Case Sensitive:**
- String comparison case-sensitive
- Default: Case insensitive

**3. Sort Order:**
- **ASCII:** Standard character order
- **EBCDIC:** IBM mainframe order

---

#### Sorter Cache

**Work Directory:**
- Temporary storage for sorting large datasets
- Configure in session properties

**Performance Tuning:**
- Adequate disk space in work directory
- Fast disk I/O improves sort performance

---

#### Real-World Sorter Scenarios

**Scenario 1: Pre-sort for Aggregator**
```
Source → Sorter (Sort by DEPT_ID) → Aggregator (Sorted Input = TRUE)
Benefit: Reduced memory usage in Aggregator
```

**Scenario 2: Remove Duplicates**
```
Sorter Transformation:
  Sort Keys: EMP_ID, DEPT_ID
  Distinct: TRUE
  
Use Case: Eliminate duplicate employee records from source
```

**Scenario 3: Top N Analysis**
```
Sorter: Sort by SALARY (Descending)
↓
Rank: Rank by SALARY
↓
Filter: Keep TOP 10
Result: Top 10 highest-paid employees
```

---

### RANK TRANSFORMATION

**Type:** Active, Connected  
**Purpose:** Rank records based on criteria, filter top/bottom N records  
**Common Use:** Top 10 customers, bottom 5 performers, quartile analysis

---

#### How Rank Works

**Concept:** Assigns rank to each row based on specified port values

**Example:**
```
Input:
EMP_NAME    | SALES
John        | 100000
Sarah       | 150000
Mike        | 80000
Lisa        | 120000

Rank by SALES (Descending):

EMP_NAME    | SALES   | RANK
Sarah       | 150000  | 1 ← Highest
Lisa        | 120000  | 2
John        | 100000  | 3
Mike        | 80000   | 4 ← Lowest
```

---

#### Rank Properties

**1. Rank Port:**
- Output port containing rank number
- Automatically generated

**2. Top/Bottom:**
- **Top N:** Select highest values
- **Bottom N:** Select lowest values

**3. Number of Ranks:**
- How many to return (e.g., Top 10)

**4. Rank By Port:**
- Column to rank on
- Direction: Ascending/Descending

**5. Group By Ports:**
- Rank within groups
- Like partitioning

---

#### Rank Configuration Example

**Top 5 Salespeople:**
```
Rank Transformation: RNK_TOP_SALES

Properties:
- Rank Port: o_Rank
- Top/Bottom: Top
- Number of Ranks: 5
- Rank By: SALES_AMOUNT (Descending)

Result: Only top 5 salespeople pass through
```

**Top 3 per Department:**
```
Rank Transformation: RNK_TOP_PER_DEPT

Properties:
- Rank Port: o_Rank
- Top/Bottom: Top
- Number of Ranks: 3
- Group By: DEPT_ID
- Rank By: SALARY (Descending)

Result: Top 3 earners in each department
```

---

#### Real-World Rank Scenarios

**Scenario 1: Customer Loyalty Program**
```
Identify top 100 customers by purchase amount:

Rank: RNK_TOP_CUSTOMERS
  - Top: 100
  - Rank By: TOTAL_PURCHASES (Desc)
  - Output: VIP customer list
```

**Scenario 2: Product Performance**
```
Find bottom 20 products by sales:

Rank: RNK_BOTTOM_PRODUCTS
  - Bottom: 20
  - Rank By: SALES_QTY (Asc)
  - Output: Products to discontinue
```

**Scenario 3: Regional Analysis**
```
Top 5 stores per region:

Rank: RNK_TOP_STORES
  - Group By: REGION
  - Top: 5
  - Rank By: REVENUE (Desc)
  - Output: Best performers in each region
```

---

## Topic 11: Lookup Transformation 🔥 (CRITICAL FOR EXAM)

**Type:** Passive OR Active (configurable), Connected OR Unconnected  
**Purpose:** Look up and retrieve data from relational tables, flat files, or views  
**Common Use:** Validate data, enrich data with reference information, get codes/descriptions

---

### Understanding Lookup - The Concept

**Think of Lookup as:** Searching a dictionary to find additional information

**Business Example:**
```
Input Data:
EMP_ID | DEPT_CODE
101    | D10
102    | D20

Lookup Table (DEPARTMENT):
DEPT_CODE | DEPT_NAME
D10       | Information Technology
D20       | Human Resources

After Lookup:
EMP_ID | DEPT_CODE | DEPT_NAME
101    | D10       | Information Technology
102    | D20       | Human Resources
```

---

### Lookup Types: Connected vs Unconnected

#### **Connected Lookup** (Most Common)

**Characteristics:**
- Part of mapping pipeline (connected with links)
- Receives input from pipeline
- Returns multiple columns
- Can be configured as passive or active
- Processes all rows

**Visual Flow:**
```
Source → Expression → Lookup (Connected) → Target
                         ↓
                   Lookup Table
```

**Port Types in Connected Lookup:**
- **Input (I):** Lookup condition columns
- **Output (O):** Return columns from lookup table
- **Lookup (L):** Lookup table columns
- **Return (R):** Values returned from lookup

**Example Ports:**
```
Connected Lookup: LKP_DEPARTMENT

Input Ports:
├─ I  DEPT_CODE (from pipeline)

Lookup Ports:
├─ L  LKP_DEPT_CODE (lookup table key)
├─ L  LKP_DEPT_NAME
├─ L  LKP_LOCATION

Output Ports:
├─ O  DEPT_CODE (pass through)
├─ O  DEPT_NAME (returned from lookup)
├─ O  LOCATION (returned from lookup)

Lookup Condition:
DEPT_CODE = LKP_DEPT_CODE
```

---

#### **Unconnected Lookup**

**Characteristics:**
- NOT in pipeline (called like a function)
- Called from Expression transformation
- Returns ONLY ONE value (column)
- Always passive
- Called only when needed

**Visual Flow:**
```
Source → Expression ----calls---→ [Lookup: Unconnected]
              ↓                         ↓
           Target                  Lookup Table
```

**How to Call:**
```
In Expression Transformation:
:LKP.LKP_GET_DEPT_NAME(DEPT_CODE)

Syntax:
:LKP.LookupTransformationName(InputValue1, InputValue2, ...)
```

**Example:**
```
Unconnected Lookup: LKP_GET_DEPT_NAME
  - Input Port: IN_DEPT_CODE
  - Lookup Port: LKP_DEPT_CODE
  - Return Port: DEPT_NAME (only one return port allowed)
  - Condition: IN_DEPT_CODE = LKP_DEPT_CODE

Called from Expression:
O_DEPT_NAME = :LKP.LKP_GET_DEPT_NAME(DEPT_CODE)
```

---

### When to Use Connected vs Unconnected

| Scenario | Use Connected | Use Unconnected |
|----------|--------------|-----------------|
| Need multiple return values | ✓ | ✗ (only 1 return value) |
| Lookup for every row | ✓ | ✗ (performance issue) |
| Conditional lookup | ✗ | ✓ (call only if needed) |
| Reuse across multiple mappings | ✗ | ✓ (more reusable) |
| Simpler to understand | ✓ | ✗ (more complex syntax) |

---

### Lookup Condition

**Purpose:** Define how to match input data with lookup table

**Syntax:**
```
Input_Port = Lookup_Port
```

**Examples:**

**Single Column Match:**
```
DEPT_CODE = LKP_DEPT_CODE
```

**Multiple Column Match:**
```
COUNTRY_CODE = LKP_COUNTRY_CODE
AND STATE_CODE = LKP_STATE_CODE
```

**Function in Condition:**
```
UPPER(CUSTOMER_NAME) = UPPER(LKP_CUSTOMER_NAME)
```

---

### Lookup Cache 🔥 (VERY IMPORTANT FOR EXAM)

**What is Lookup Cache?**
- Informatica loads lookup table data into memory (cache)
- Searches happen in memory (fast)
- Without cache, queries database for each row (slow)

---

#### Types of Lookup Cache

#### **1. Static Cache (Default)**

**Characteristics:**
- Cache built at session start
- Remains unchanged during session
- Data is READ-ONLY

**When to Use:**
- Lookup table doesn't change during session
- Reference/dimension tables
- Most common scenario

**Example:**
```
Lookup: LKP_COUNTRY
Source: COUNTRY_MASTER (5000 rows)
Cache: Built once, used for all lookups
```

---

#### **2. Dynamic Cache** 🔥

**Characteristics:**
- Cache updated during session
- New rows inserted into cache
- Used for Slowly Changing Dimensions (SCD)

**When to Use:**
- Need to track new/changed records
- SCD Type 1 or Type 2 implementations
- Avoid duplicate inserts

**Properties:**
- **Dynamic Lookup Cache:** Enabled
- **NewLookupRow:** Port indicating if row is new (0=exists, 1=new)

**Example Flow:**
```
Input Row: CUST_ID = 501

Lookup in Cache:
  - If Found → Return existing values, NewLookupRow = 0
  - If Not Found → Insert to cache, NewLookupRow = 1
```

**Usage in SCD:**
```
If NewLookupRow = 1:
  Insert new dimension record
Else:
  Update existing record (Type 1) OR
  Insert new version (Type 2)
```

---

#### **3. Persistent Cache**

**Characteristics:**
- Cache saved to disk after session
- Reused in subsequent sessions
- Faster session startup (no rebuild)

**When to Use:**
- Large lookup tables
- Lookup data rarely changes
- Frequent session runs

**Configuration:**
- **Persistent Cache:** Enabled
- **Cache File Name:** Specify location

**Benefit:**
- Session 1: Builds cache (10 min)
- Session 2+: Reuses cache (30 sec)

---

#### **4. Shared Cache**

**Characteristics:**
- Multiple transformations share same cache
- Built once, used by many
- Saves memory

**When to Use:**
- Same lookup table used multiple times in mapping
- Multiple mappings use same lookup

**Example:**
```
Mapping has 3 lookups on PRODUCT table:
  - LKP_PRODUCT_1 (Shared Cache = TRUE)
  - LKP_PRODUCT_2 (Shared Cache = TRUE)
  - LKP_PRODUCT_3 (Shared Cache = TRUE)

Result: Cache built once, shared by all 3
```

---

### Lookup Performance Tuning 🔥

#### **1. Use Source Qualifier Join Instead of Lookup**

**When Possible:** If lookup table and source in same database

**Instead of This:**
```
Source (Oracle) → Lookup: Join with Lookup Table (Oracle) → Target
```

**Do This:**
```
Source Qualifier:
  SQL Override: SELECT s.*, l.DEPT_NAME
                FROM SOURCE s
                JOIN LOOKUP_TABLE l ON s.DEPT_CODE = l.DEPT_CODE
↓
Target
```

**Benefit:** Database join faster than Informatica lookup

---

#### **2. Index Lookup Columns**

**Best Practice:** Create index on lookup condition columns in source table

**Example:**
```
Lookup Condition: DEPT_CODE = LKP_DEPT_CODE

Database:
CREATE INDEX IDX_DEPT_CODE ON DEPARTMENT(DEPT_CODE);
```

---

#### **3. Limit Lookup Columns**

**Problem:** Loading unnecessary columns wastes memory

**Bad:**
```
SELECT * FROM DEPARTMENT (50 columns loaded)
```

**Good:**
```
SELECT DEPT_CODE, DEPT_NAME FROM DEPARTMENT (only 2 columns)
```

**How to Configure:**
In Lookup properties → SQL Override → Select only needed columns

---

#### **4. Filter Lookup Table**

**Reduce rows loaded into cache**

**Example:**
```
Lookup SQL Override:
SELECT DEPT_CODE, DEPT_NAME
FROM DEPARTMENT
WHERE ACTIVE_FLAG = 'Y'
  AND COUNTRY = 'USA'
```

**Benefit:** Smaller cache, faster lookups

---

### Lookup SQL Override

**Purpose:** Customize SQL query to fetch lookup data

**When to Use:**
- Filter lookup table
- Join multiple tables
- Optimize query performance
- Select specific columns

**Example 1: Filter Lookup**
```
SELECT COUNTRY_CODE, COUNTRY_NAME
FROM COUNTRY_MASTER
WHERE REGION = 'ASIA'
```

**Example 2: Join Multiple Tables**
```
SELECT d.DEPT_CODE, d.DEPT_NAME, l.LOCATION_NAME
FROM DEPARTMENT d
JOIN LOCATION l ON d.LOCATION_ID = l.LOCATION_ID
WHERE d.ACTIVE_FLAG = 'Y'
```

---

### Return Values on Lookup Failure

**What Happens When Lookup Doesn't Find Match?**

**Options:**

**1. Return NULL** (Default)
```
Input: DEPT_CODE = 'D99' (doesn't exist)
Output: DEPT_NAME = NULL
```

**2. Set Default Value**
```
In Expression after Lookup:
O_DEPT_NAME = IIF(ISNULL(DEPT_NAME), 'Unknown', DEPT_NAME)
```

**3. Error Handling**
```
Use Router after Lookup:
  - Group 1: ISNULL(DEPT_NAME) → Error_Target
  - Group 2: NOT ISNULL(DEPT_NAME) → Main_Target
```

---

### Real-World Lookup Scenarios

**Scenario 1: Enrich Customer Data**
```
Source: Transaction data (CUST_ID, TXN_AMOUNT)
Lookup: CUSTOMER table (CUST_ID, NAME, SEGMENT, REGION)

Connected Lookup: LKP_CUSTOMER
  Condition: CUST_ID = LKP_CUST_ID
  Returns: NAME, SEGMENT, REGION

Output: Transactions with customer details
```

**Scenario 2: Validate Product Codes**
```
Unconnected Lookup: LKP_VALID_PRODUCT

Expression:
  O_PRODUCT_STATUS = 
    IIF(ISNULL(:LKP.LKP_VALID_PRODUCT(PRODUCT_CODE)),
        'Invalid', 
        'Valid')

Router:
  - Valid products → Main_Target
  - Invalid products → Error_Log
```

**Scenario 3: Get Exchange Rates**
```
Source: International sales (AMOUNT, CURRENCY, DATE)
Lookup: EXCHANGE_RATE table (DATE, CURRENCY, RATE)

Connected Lookup: LKP_EXCHANGE_RATE
  Condition: 
    TXN_DATE = LKP_DATE
    AND CURRENCY = LKP_CURRENCY
  Returns: EXCHANGE_RATE

Expression:
  O_USD_AMOUNT = AMOUNT * EXCHANGE_RATE
```

**Scenario 4: Slowly Changing Dimension Type 2**
```
Dynamic Lookup: LKP_CUSTOMER_DIM
  Cache: Dynamic
  Condition: CUST_ID = LKP_CUST_ID

Router based on NewLookupRow:
  - NewLookupRow = 1 → Insert new record
  - NewLookupRow = 0 AND attributes changed → 
      Expire old record, Insert new version
  - NewLookupRow = 0 AND no change → Update last_modified_date
```

---

## Topic 12: Joiner Transformation 🔥

**Type:** Active, Connected  
**Purpose:** Join two sources (like SQL JOIN)  
**Common Use:** Combine data from different tables, sources, or databases

---

### Understanding Joiner

**Concept:** Merge two data streams based on matching column(s)

**Business Example:**
```
Source 1: EMPLOYEES          Source 2: DEPARTMENTS
EMP_ID | NAME | DEPT_ID      DEPT_ID | DEPT_NAME
101    | John | 10           10      | IT
102    | Jane | 20           20      | HR
103    | Mike | 10           30      | Sales

After Join (EMP.DEPT_ID = DEPT.DEPT_ID):
EMP_ID | NAME | DEPT_ID | DEPT_NAME
101    | John | 10      | IT
102    | Jane | 20      | HR
103    | Mike | 10      | IT
```

---

### Joiner Components

**Two Input Groups:**
1. **Master Source** (Driving table)
2. **Detail Source** (Lookup table)

**Join Condition:**
- Specifies how to match rows
- Can be on multiple columns

**Visual:**
```
Source 1 (Master) ──┐
                    ├──→ Joiner ──→ Output
Source 2 (Detail) ──┘
```

---

### Types of Joins

#### **1. Normal Join (Inner Join)**

**Behavior:** Returns only matching rows from both sources

**Example:**
```
EMPLOYEES (Master):          DEPARTMENTS (Detail):
EMP_ID | DEPT_ID             DEPT_ID | DEPT_NAME
101    | 10                  10      | IT
102    | 20                  20      | HR
103    | 99 (no match)       30      | Sales (no match)

Result (Normal Join):
EMP_ID | DEPT_ID | DEPT_NAME
101    | 10      | IT
102    | 20      | HR
```

---

#### **2. Master Outer Join (Left Outer Join)**

**Behavior:** All rows from Master + matching rows from Detail

**Example:**
```
EMPLOYEES (Master):          DEPARTMENTS (Detail):
EMP_ID | DEPT_ID             DEPT_ID | DEPT_NAME
101    | 10                  10      | IT
102    | 20                  20      | HR
103    | 99                  

Result (Master Outer):
EMP_ID | DEPT_ID | DEPT_NAME
101    | 10      | IT
102    | 20      | HR
103    | 99      | NULL ← No match, but employee included
```

---

#### **3. Detail Outer Join (Right Outer Join)**

**Behavior:** All rows from Detail + matching rows from Master

**Example:**
```
EMPLOYEES (Master):          DEPARTMENTS (Detail):
EMP_ID | DEPT_ID             DEPT_ID | DEPT_NAME
101    | 10                  10      | IT
102    | 20                  20      | HR
                              30      | Sales

Result (Detail Outer):
EMP_ID | DEPT_ID | DEPT_NAME
101    | 10      | IT
102    | 20      | HR
NULL   | 30      | Sales ← No employees, but dept included
```

---

#### **4. Full Outer Join**

**Behavior:** All rows from both sources (matched + unmatched)

**Example:**
```
EMPLOYEES (Master):          DEPARTMENTS (Detail):
EMP_ID | DEPT_ID             DEPT_ID | DEPT_NAME
101    | 10                  10      | IT
102    | 20                  20      | HR
103    | 99                  30      | Sales

Result (Full Outer):
EMP_ID | DEPT_ID | DEPT_NAME
101    | 10      | IT
102    | 20      | HR
103    | 99      | NULL ← Unmatched from Master
NULL   | 30      | Sales ← Unmatched from Detail
```

---

### Join Condition

**Syntax:** Specify matching columns

**Single Column:**
```
EMP.DEPT_ID = DEPT.DEPT_ID
```

**Multiple Columns:**
```
EMP.COUNTRY_CODE = DEPT.COUNTRY_CODE
AND EMP.REGION_CODE = DEPT.REGION_CODE
```

**Best Practice:**
- Use equality conditions (=) for best performance
- Avoid functions in join condition if possible

---

### Joiner vs Lookup vs Source Qualifier Join

| Feature | Joiner | Lookup | SQ Join |
|---------|--------|--------|---------|
| Join Type | All types (inner, outer) | Only inner | All types |
| Sources | Any (different DBs, files) | Any | Same database only |
| Performance | Slowest | Medium | Fastest |
| Sorted Input | Optional (improves perf) | N/A | Automatic |
| When to Use | Different sources/DBs | Reference data | Same DB tables |

**Decision Tree:**
```
Both sources in same database?
├─ YES → Use Source Qualifier Join (best performance)
└─ NO → Need reference data?
         ├─ YES → Use Lookup
         └─ NO → Use Joiner
```

---

### Joiner Performance Optimization

#### **1. Sorted Input** 🔥 (Important!)

**Concept:** Pre-sort both sources by join columns

**Without Sorted Input:**
- Joiner loads entire Detail source into memory
- Memory-intensive for large datasets

**With Sorted Input:**
- Joiner processes streams together
- Much less memory required

**How to Configure:**
```
1. Add Sorter before Joiner for both sources:
   - Sort Master by join columns
   - Sort Detail by join columns

2. In Joiner properties:
   - Sorted Input = TRUE
   - Master Sort Order = (specify columns)
   - Detail Sort Order = (specify columns)
```

**Performance Impact:**
- 50-80% memory reduction
- Critical for large datasets
- **Exam Favorite Question!**

---

#### **2. Designate Master and Detail Wisely**

**Rule:** Smaller dataset should be Detail source

**Why?**
- Detail source loaded into memory (cache)
- Master source drives the join
- Smaller detail = less memory

**Example:**
```
Source 1: 1,000,000 employees (Master)
Source 2: 100 departments (Detail)

Joiner loads 100 departments in memory
Streams through 1M employees
```

---

#### **3. Join Condition on Indexed Columns**

**Best Practice:** Use indexed columns in join condition

**Note:** Applies when using database joins, not as critical for Joiner transformation

---

### Joiner Limitations

**Cannot Join:**
1. Sorted input with Normal or Full Outer joins (use Master/Detail Outer only)
2. CLOB/BLOB data types
3. More than two sources (nest Joiners for multiple joins)

---

### Real-World Joiner Scenarios

**Scenario 1: Combine Employee and Department Data**
```
Source 1: EMPLOYEES (Oracle)
Source 2: DEPARTMENTS (SQL Server)

Joiner: JOIN_EMP_DEPT
  Type: Normal Join
  Condition: EMP.DEPT_ID = DEPT.DEPT_ID
  Master: EMPLOYEES (larger)
  Detail: DEPARTMENTS (smaller)

Output: Employee details with department names
```

**Scenario 2: Identify Unmatched Records**
```
Source 1: CURRENT_CUSTOMERS (Master)
Source 2: ORDERS (Detail)

Joiner: JOIN_CUST_ORDERS
  Type: Master Outer Join
  Condition: CUST.CUST_ID = ORD.CUST_ID

Filter after Joiner:
  Condition: ISNULL(ORD.ORDER_ID)
  
Output: Customers with no orders (prospect for campaigns)
```

**Scenario 3: Multi-Column Join**
```
Source 1: SALES_DATA (Master)
Source 2: EXCHANGE_RATES (Detail)

Joiner: JOIN_SALES_RATES
  Condition: 
    SALES.TXN_DATE = RATES.RATE_DATE
    AND SALES.CURRENCY = RATES.CURRENCY
  Type: Normal Join

Output: Sales with correct exchange rate
```

---

## Topic 13: Union and Normalizer Transformations

### UNION TRANSFORMATION

**Type:** Active, Connected  
**Purpose:** Combine multiple data streams vertically (like SQL UNION ALL)  
**Use Case:** Merge similar data from different sources

---

#### How Union Works

**Concept:** Stack datasets on top of each other

**Example:**
```
Source 1: USA_SALES          Source 2: CANADA_SALES
REGION | AMOUNT              REGION | AMOUNT
East   | 1000                Ontario| 800
West   | 1500                Quebec | 900

Union Output:
REGION  | AMOUNT
East    | 1000
West    | 1500
Ontario | 800
Quebec  | 900
```

---

#### Union vs SQL UNION

| Feature | Union Transformation | SQL UNION |
|---------|---------------------|-----------|
| Duplicates | Keeps all (UNION ALL) | Removes (UNION) |
| Performance | Faster (no distinct) | Slower (distinct check) |
| Sources | Any (files, DBs) | Same database only |

**Note:** Union transformation does NOT remove duplicates. To remove duplicates, add Sorter with Distinct option after Union.

---

#### Union Configuration

**Input Groups:**
- Can have 2 or more input groups
- All must have same structure (column count, compatible types)

**Example:**
```
Union: UNION_ALL_REGIONS

Input Group 1 (USA_SALES):
  - REGION (String)
  - AMOUNT (Decimal)

Input Group 2 (CANADA_SALES):
  - REGION (String)
  - AMOUNT (Decimal)

Input Group 3 (MEXICO_SALES):
  - REGION (String)
  - AMOUNT (Decimal)

Output: Combined dataset from all 3 regions
```

---

#### Real-World Union Scenarios

**Scenario 1: Consolidate Regional Data**
```
Sources: North_Sales, South_Sales, East_Sales, West_Sales
Union: UNION_ALL_SALES
Output: National_Sales_Summary
```

**Scenario 2: Merge Historical and Current Data**
```
Source 1: SALES_ARCHIVE (2020-2024)
Source 2: SALES_CURRENT (2025)
Union: UNION_HISTORICAL_CURRENT
Output: SALES_COMPLETE
```

---

### NORMALIZER TRANSFORMATION

**Type:** Active, Connected  
**Purpose:** Convert columns to rows (pivot/unpivot operation)  
**Common Use:** Process COBOL sources, denormalize data

---

#### How Normalizer Works

**Concept:** Transform horizontal data to vertical

**Example:**
```
Input (Columns):
PRODUCT | JAN_SALES | FEB_SALES | MAR_SALES
ProductA| 1000      | 1100      | 1200

Normalizer Output (Rows):
PRODUCT  | MONTH | SALES
ProductA | JAN   | 1000
ProductA | FEB   | 1100
ProductA | MAR   | 1200
```

---

#### Normalizer Configuration

**Normalizer Columns (GK_):**
- Generated Key columns
- Indicates which occurrence

**Normalizer Ports:**
- Define which columns to normalize
- Specify number of occurrences

**Example Setup:**
```
Input Ports:
- PRODUCT (pass through)
- Q1_SALES
- Q2_SALES
- Q3_SALES
- Q4_SALES

Normalizer Configuration:
- Occurs: 4 (for 4 quarters)
- GK Column: GK_QUARTER (1, 2, 3, 4)

Output:
PRODUCT | GK_QUARTER | SALES
ProdA   | 1          | 1000 (Q1)
ProdA   | 2          | 1200 (Q2)
ProdA   | 3          | 1100 (Q3)
ProdA   | 4          | 1300 (Q4)
```

---

## Topic 14: Update Strategy Transformation 🔥 (CRITICAL)

**Type:** Active, Connected  
**Purpose:** Determine which rows to INSERT, UPDATE, DELETE, or REJECT  
**Use Case:** Slowly Changing Dimensions, incremental loads, conditional updates

---

### Understanding Update Strategy

**Problem:** By default, Informatica inserts ALL rows into target

**Solution:** Update Strategy flags rows for specific operations

**Operations:**
- **DD_INSERT (0):** Insert row
- **DD_UPDATE (1):** Update existing row
- **DD_DELETE (2):** Delete row
- **DD_REJECT (3):** Reject row (don't load)

---

### How Update Strategy Works

**Expression in Update Strategy:**
```
IIF(condition, DD_INSERT,
  IIF(condition, DD_UPDATE,
    IIF(condition, DD_DELETE, DD_REJECT)))
```

**Example:**
```
Update Strategy: UPD_EMPLOYEE_STRATEGY

Expression:
IIF(ISNULL(LKP_EMP_ID), DD_INSERT,  ← New record
  IIF(LKP_SALARY != SALARY, DD_UPDATE,  ← Changed record
    DD_REJECT))  ← No change, skip
```

---

### Update Strategy Configuration

**Single Expression:**
- One formula determines operation for all rows

**Example 1: Insert New, Update Existing**
```
Lookup: LKP_TARGET (check if exists)

Update Strategy Expression:
IIF(ISNULL(LKP_EMP_ID), DD_INSERT, DD_UPDATE)

Logic:
- If lookup returns NULL → Record doesn't exist → INSERT
- If lookup returns value → Record exists → UPDATE
```

**Example 2: Delete Inactive, Insert/Update Active**
```
Update Strategy Expression:
IIF(STATUS = 'D', DD_DELETE,
  IIF(ISNULL(LKP_EMP_ID), DD_INSERT, DD_UPDATE))

Logic:
- STATUS = 'D' → DELETE
- New record → INSERT
- Existing record → UPDATE
```

---

### Target Properties for Update Strategy

**Target must be configured to accept update operations:**

**Session Properties → Mapping Tab → Target:**
- **Treat source rows as:** Data Driven (not Insert)

**Target Table Properties:**
- Update/Insert/Delete as Update allowed

**Key Columns:**
- Define primary key for UPDATE/DELETE operations
- Informatica uses key to identify which row to update/delete

---

### Real-World Update Strategy Scenarios

**Scenario 1: Incremental Load (SCD Type 1)**
```
Source: Daily employee changes
Lookup: Check if employee exists in target
Update Strategy:
  - New employee (LKP returns NULL) → DD_INSERT
  - Changed data → DD_UPDATE
  - No change → DD_REJECT

Result: Only new/changed records processed
```

**Scenario 2: Soft Delete**
```
Source: Employees with STATUS column
Update Strategy:
  IIF(STATUS = 'TERMINATED', DD_UPDATE,
    IIF(ISNULL(LKP_EMP_ID), DD_INSERT, DD_UPDATE))

Expression Transformation (before Update Strategy):
  O_IS_ACTIVE = IIF(STATUS = 'TERMINATED', 'N', 'Y')

Result: Terminated employees marked inactive, not deleted
```

**Scenario 3: Multi-Condition Strategy**
```
Update Strategy for Customer Dimension:

Expression:
IIF(LKP_NEWLOOKUPROW = 1, DD_INSERT,  ← New customer
  IIF(CUSTOMER_STATUS = 'DELETED', DD_DELETE,  ← Removed
    IIF(LKP_EMAIL != EMAIL OR 
        LKP_PHONE != PHONE OR 
        LKP_ADDRESS != ADDRESS, DD_UPDATE,  ← Changed
      DD_REJECT)))  ← No change

Result: Handles inserts, deletes, updates, and skips unchanged
```

---

### Update Strategy Best Practices

1. **Always use DD_ constants** (not numeric values)
2. **Test with small dataset** before production
3. **Configure target properties** correctly (Data Driven)
4. **Define primary keys** in target for UPDATE/DELETE
5. **Use Lookup** to determine if record exists
6. **Log rejected rows** for troubleshooting

---

## Topic 15: Stored Procedure and SQL Transformation

### STORED PROCEDURE TRANSFORMATION

**Type:** Passive OR Active, Connected OR Unconnected  
**Purpose:** Execute database stored procedures  
**Use Cases:** Pre/post session SQL, complex database logic, call external procedures

---

#### When to Use Stored Procedure

**Scenarios:**
- Execute pre-session tasks (truncate tables, disable indexes)
- Call post-session cleanup (archive, email notifications)
- Perform complex calculations in database
- Execute security/audit procedures

---

#### Stored Procedure Configuration

**Connection Type:**
1. **Unconnected (Most Common):**
   - Called from pre/post session properties
   - Not in mapping flow

2. **Connected:**
   - Part of mapping pipeline
   - Can pass/receive values

**Parameters:**
- **Input:** Pass values TO procedure
- **Output:** Receive values FROM procedure
- **Input/Output:** Both

**Example:**
```
Stored Procedure: SP_ARCHIVE_OLD_DATA

Input Parameter: p_days_old (Number)
Output Parameter: p_rows_archived (Number)

Database Procedure:
CREATE PROCEDURE sp_archive_old_data(p_days_old NUMBER, p_rows_archived OUT NUMBER)
AS
BEGIN
  INSERT INTO ARCHIVE SELECT * FROM SALES WHERE TXN_DATE < SYSDATE - p_days_old;
  p_rows_archived := SQL%ROWCOUNT;
  DELETE FROM SALES WHERE TXN_DATE < SYSDATE - p_days_old;
  COMMIT;
END;
```

---

### SQL TRANSFORMATION

**Type:** Active, Connected  
**Purpose:** Execute SQL queries during data flow  
**Use Cases:** Insert/update/delete mid-pipeline, complex queries, dynamic SQL

---

#### SQL Transformation Features

**Database Operations:**
- **Insert:** Add rows to tables
- **Update:** Modify existing rows
- **Delete:** Remove rows
- **Select:** Query data (rare, use Lookup instead)

**Dynamic SQL:**
- SQL can include port values
- Constructed at runtime

**Example:**
```
SQL Transformation: SQL_UPDATE_AUDIT

SQL Query:
INSERT INTO AUDIT_LOG (EMP_ID, ACTION, ACTION_DATE)
VALUES (:EMP_ID, 'PROCESSED', SYSDATE)

Input Ports:
- EMP_ID (used in SQL)

Result: Logs each processed employee to audit table
```

---

#### SQL Transformation vs Stored Procedure

| Feature | SQL Transformation | Stored Procedure |
|---------|-------------------|------------------|
| Location | In mapping | Pre/post session or in mapping |
| Flexibility | Dynamic SQL | Fixed procedure |
| Performance | Slower (row-by-row) | Faster (batch) |
| Maintenance | Easy (change in mapping) | Needs DBA (change in DB) |

---

## SUMMARY: MODULE 3 KEY CONCEPTS

### Expression Transformation
✅ Passive, used for calculations, string/date operations
✅ IIF and DECODE for conditional logic
✅ Variable ports for intermediate calculations

### Filter & Router
✅ Filter: Remove rows (single output)
✅ Router: Multiple conditional outputs (like CASE WHEN)
✅ Router has default group for unmatched rows

### Aggregator
✅ Active transformation, reduces rows
✅ SUM, AVG, COUNT, MIN, MAX functions
✅ Sorted Input optimization reduces memory

### Sorter & Rank
✅ Sorter: Sort data, remove duplicates (Distinct option)
✅ Rank: Top N / Bottom N analysis
✅ Rank can group by columns (top N per group)

### Lookup
✅ Connected: Multiple return values, in pipeline
✅ Unconnected: Called like function, single return value
✅ Static Cache (read-only) vs Dynamic Cache (updatable)
✅ Persistent Cache (saved to disk) vs Shared Cache (multiple transformations)
✅ Use Source Qualifier join when both sources in same DB (best performance)

### Joiner
✅ Combines two sources (Normal, Master Outer, Detail Outer, Full Outer)
✅ Smaller dataset should be Detail source
✅ Sorted Input optimization reduces memory usage
✅ Use Source Qualifier join if sources in same database

### Update Strategy
✅ Determines INSERT/UPDATE/DELETE/REJECT operations
✅ DD_INSERT (0), DD_UPDATE (1), DD_DELETE (2), DD_REJECT (3)
✅ Target must be set to "Data Driven" mode
✅ Critical for incremental loads and SCD implementations

---

## PRACTICE SCENARIOS FOR MODULE 3 MCQs

**Scenario 1:** You need to calculate bonus as 10% of salary if salary > 50000, else 5%. Which transformation and function?
- **Answer:** Expression transformation with `IIF(SALARY > 50000, SALARY * 0.10, SALARY * 0.05)`

**Scenario 2:** Load only employees from IT and HR departments. Which transformation?
- **Answer:** Filter with condition `DEPT IN ('IT', 'HR')`

**Scenario 3:** Send high earners (>80K) to one target, mid earners (50K-80K) to another, low earners (<50K) to third target. Which transformation?
- **Answer:** Router with 3 output groups + default group

**Scenario 4:** Calculate total sales per region. Input: 1000 rows, Output: 5 rows (5 regions). Active or Passive?
- **Answer:** Active (Aggregator transformation, row count changes)

**Scenario 5:** Your Aggregator is running out of memory with 10 million rows. What optimization?
- **Answer:** Add Sorter before Aggregator, enable Sorted Input in Aggregator properties

**Scenario 6:** Need to get department name for each employee. Department table has 50 rows, employee table has 1 million rows. Which transformation?
- **Answer:** Lookup transformation (reference data) OR Source Qualifier join if same database

**Scenario 7:** Lookup transformation returns NULL for some employees. What does this indicate?
- **Answer:** No matching record found in lookup table (department doesn't exist)

**Scenario 8:** You're loading customer dimension. New customers should INSERT, changed customers should UPDATE. Which transformation?
- **Answer:** Update Strategy with Dynamic Lookup to detect new/changed records

**Scenario 9:** Combine sales data from 4 regional files into one target. Which transformation?
- **Answer:** Union transformation (4 input groups)

**Scenario 10:** In Joiner, master source has 1M rows, detail has 100 rows. Current performance is slow. What's the issue?
- **Answer:** Master/Detail designation is reversed. Smaller dataset (100 rows) should be Detail.

---

## HANDS-ON EXERCISES

### Exercise 1: Expression Transformation
**Objective:** Clean and transform employee data
```
Create mapping:
- Source: EMPLOYEES (EMP_ID, FIRST_NAME, LAST_NAME, SALARY, HIRE_DATE)
- Expression:
  - O_FULL_NAME = FIRST_NAME || ' ' || LAST_NAME
  - O_ANNUAL_COMPENSATION = SALARY * 12
  - O_YEARS_SERVICE = TRUNC((SYSDATE - HIRE_DATE) / 365.25)
  - O_CATEGORY = IIF(SALARY > 80000, 'Senior', IIF(SALARY > 50000, 'Mid', 'Junior'))
- Target: EMP_TRANSFORMED
```

### Exercise 2: Router Transformation
**Objective:** Route employees to different targets by salary band
```
Create mapping:
- Source: EMPLOYEES
- Router:
  - Group SENIOR: SALARY > 80000 → SENIOR_EMP
  - Group MID: SALARY 50K-80K → MID_EMP  
  - Group JUNIOR: SALARY < 50000 → JUNIOR_EMP
  - Default → ERROR_LOG
```

### Exercise 3: Aggregator Transformation
**Objective:** Calculate department statistics
```
Create mapping:
- Source: EMPLOYEES
- Aggregator:
  - Group By: DEPT_ID
  - O_HEADCOUNT = COUNT(*)
  - O_TOTAL_SALARY = SUM(SALARY)
  - O_AVG_SALARY = AVG(SALARY)
  - O_MAX_SALARY = MAX(SALARY)
- Target: DEPT_STATS
```

### Exercise 4: Lookup Transformation
**Objective:** Enrich employee data with department information
```
Create mapping:
- Source: EMPLOYEES
- Lookup: LKP_DEPARTMENT
  - Lookup Table: DEPARTMENT
  - Condition: DEPT_ID = LKP_DEPT_ID
  - Return: DEPT_NAME, LOCATION
- Target: EMP_WITH_DEPT
```

### Exercise 5: Update Strategy
**Objective:** Implement incremental load
```
Create mapping:
- Source: EMPLOYEES_DAILY
- Lookup: LKP_EMPLOYEE_TARGET (check if exists)
- Update Strategy:
  - Expression: IIF(ISNULL(LKP_EMP_ID), DD_INSERT, DD_UPDATE)
- Target: EMPLOYEE_DWH (set to Data Driven mode)
```

---

**🎯 MODULE 3 COMPLETE!**

You've now covered the most critical transformations for TCS Wings exam. Practice these scenarios in your Informatica environment before moving to Module 4.

**Ready for Module 4: Advanced Mapping Techniques?**