# MODULE 2: CORE COMPONENTS

## Topic 5: Sources and Targets (Relational, Flat Files)

### Understanding Sources

**Source:** Any system or file from which Informatica extracts data during ETL process.

**Think of it as:** The "starting point" of your data journey.

---

### Types of Sources

#### 1. **Relational Database Sources**

**Most Common:**
- Oracle
- SQL Server
- DB2
- MySQL
- PostgreSQL
- Teradata

**How Informatica Connects:**
- Uses ODBC (Open Database Connectivity) or native drivers
- Reads table structures and data
- Can read from views, synonyms, and tables

**What Informatica Imports:**
- Table name
- Column names
- Data types (VARCHAR, NUMBER, DATE, etc.)
- Primary keys
- Foreign keys (for reference, not enforced in ETL)

**Example:**
```
Source Table: EMPLOYEES (in Oracle)
┌─────────────┬──────────────┬─────────┐
│ EMP_ID      │ EMP_NAME     │ SALARY  │
│ NUMBER(10)  │ VARCHAR2(50) │ NUMBER  │
│ PK          │              │         │
└─────────────┴──────────────┴─────────┘
```

When you import this, Informatica creates a **source definition** that mirrors this structure.

---

#### 2. **Flat File Sources**

**Types:**
- **Fixed Width:** Each column has fixed position and length
- **Delimited:** Columns separated by delimiter (comma, pipe, tab)

**Common Flat Files:**
- CSV (Comma-Separated Values)
- TXT (Text files)
- DAT (Data files)
- Custom delimited files

**Example - Delimited File (CSV):**
```
EMP_ID,EMP_NAME,SALARY
101,John Smith,50000
102,Jane Doe,60000
103,Bob Johnson,55000
```

**Example - Fixed Width File:**
```
101       John Smith                50000
102       Jane Doe                  60000
103       Bob Johnson               55000
│←10→│   │←25→│                    │←10→│
EMP_ID    EMP_NAME                  SALARY
```

**Flat File Properties You Define:**
- Delimiter (for delimited files): comma, pipe |, tab, semicolon
- Text qualifier: " (double quotes) for text fields
- Header rows: Skip first row if it contains column names
- Code page: Character encoding (UTF-8, ASCII)

---

#### 3. **Other Sources** (Brief Overview)

- **XML Sources:** Structured XML files
- **COBOL Sources:** Legacy mainframe files
- **Application Sources:** SAP, Salesforce (through connectors)
- **Web Services:** REST/SOAP APIs

**For TCS Wings:** Focus on Relational and Flat Files - most common in exam scenarios.

---

### Understanding Targets

**Target:** The destination where transformed data is loaded.

**Think of it as:** The "endpoint" of your data journey.

---

### Types of Targets

#### 1. **Relational Database Targets**

Same databases as sources:
- Oracle, SQL Server, DB2, MySQL, etc.

**Target Operations:**
- **Insert:** Add new rows
- **Update:** Modify existing rows
- **Delete:** Remove rows
- **Truncate:** Delete all rows before loading
- **Update/Insert (Upsert):** Update if exists, insert if new

**Target Table Example:**
```
Target Table: EMP_DWH (Data Warehouse)
┌─────────────┬──────────────┬─────────┬────────────┐
│ EMP_ID      │ EMP_NAME     │ SALARY  │ LOAD_DATE  │
│ NUMBER(10)  │ VARCHAR2(50) │ NUMBER  │ DATE       │
│ PK          │              │         │            │
└─────────────┴──────────────┴─────────┴────────────┘
```

---

#### 2. **Flat File Targets**

- Write data to CSV, TXT, or other flat files
- Specify delimiter, text qualifier
- Can generate files dynamically

**When to Use:**
- Export data for external systems
- Generate reports in file format
- Archive data
- Share data with non-database systems

---

### Source vs Target: Key Differences

| Aspect | Source | Target |
|--------|--------|--------|
| Purpose | Read data FROM | Write data TO |
| Operations | Read-only | Insert/Update/Delete |
| Modifications | Cannot modify source data | Can modify target structure |
| Multiple | Can have multiple sources in one mapping | Usually one target per mapping |
| Connection | Read connection | Write connection |

---

## Topic 6: Source Analyzer and Target Designer

### Source Analyzer (Designer Sub-Tool)

**Purpose:** Import and manage source definitions.

---

### How to Import a Relational Source

**Step-by-Step Process:**

1. **Open Designer** → Click on **Source Analyzer** tab

2. **Menu:** Sources → Import from Database

3. **Select ODBC/Native Connection:**
   - Choose existing connection OR create new
   - Provide: Username, Password, Connection String

4. **Select Tables:**
   - Browse available tables
   - Select one or multiple tables
   - Click OK

5. **Result:**
   - Source definition appears in Source Analyzer window
   - Shows all columns with data types
   - Displays keys and indexes

**What You See:**
```
Source Definition: EMPLOYEES
├── EMP_ID (NUMBER) [PK]
├── EMP_NAME (VARCHAR2)
├── DEPT_ID (NUMBER) [FK]
├── SALARY (NUMBER)
└── HIRE_DATE (DATE)
```

---

### How to Import a Flat File Source

**Step-by-Step Process:**

1. **Source Analyzer** → Sources → Import from File

2. **Browse to File Location:**
   - Select your CSV/TXT file
   - Example: `C:\Data\Employees.csv`

3. **Configure File Properties:**

   **For Delimited File:**
   - Delimiter: Comma (or pipe, tab)
   - Text Qualifier: " (if text is in quotes)
   - Import first row as column names: Yes/No
   
   **For Fixed Width:**
   - Define column positions manually
   - Specify width for each field

4. **Define Column Data Types:**
   - Informatica tries to auto-detect
   - You can manually adjust:
     - String → String(length)
     - Number → Integer, Decimal, etc.
     - Date → Date/Time

5. **Click OK:**
   - Source definition created
   - File definition stored in repository

**What You See:**
```
Source Definition: Employees_FF (Flat File)
├── EMP_ID (String, 10)
├── EMP_NAME (String, 50)
├── SALARY (Decimal)
└── HIRE_DATE (Date/Time)
```

**Important Notes:**
- Flat file source definition is just metadata (structure)
- Actual file path specified later in session properties
- Can use same definition for multiple files with same structure

---

### Source Analyzer - Key Features

**Edit Source:**
- Right-click source → Edit
- Modify column properties
- Add/remove columns (metadata only)

**Source Qualifier:**
- Automatically created when you use a source
- We'll cover this in transformations
- Allows filtering, sorting at source level

**Multiple Sources:**
- Import multiple sources for complex mappings
- Join data from different sources

---

### Target Designer (Designer Sub-Tool)

**Purpose:** Create and manage target definitions.

---

### Creating a Target - Two Ways

#### Method 1: Import Existing Target Table

**Process:** (Same as importing source)

1. **Target Designer** → Targets → Import from Database

2. Select connection and table

3. Target definition created

**When to Use:**
- Target table already exists in database
- Loading to existing data warehouse tables

---

#### Method 2: Create New Target in Informatica

**Process:**

1. **Target Designer** → Targets → Create

2. **Define Target Properties:**
   - Name: EMP_TARGET
   - Database Type: Oracle, SQL Server, etc.
   - Table Type: Table (or View)

3. **Add Columns:**
   - Click "Columns" tab
   - Add column by column:
     - Column Name: EMP_ID
     - Data Type: NUMBER(10)
     - Key Type: Primary Key
     - Nullable: No
   
4. **Define Constraints:**
   - Primary Key
   - Not Null constraints

5. **Save Target Definition**

**Result:**
```
Target Definition: EMP_TARGET
├── EMP_ID (NUMBER, 10) [PK, NOT NULL]
├── EMP_NAME (VARCHAR2, 50)
├── SALARY (NUMBER, 10, 2)
└── LOAD_DATE (DATE)
```

**Important:**
- This only creates metadata in repository
- To physically create table in database:
  - Targets → Generate/Execute SQL
  - Review SQL statement
  - Execute to create table

---

### Target Designer - Key Features

**Target Load Plan:**
- Normal (default): Standard insert/update operations
- Bulk mode: Faster loading for large volumes

**Target Properties:**
- Truncate target table: Clear table before loading
- Update override: Custom SQL for updates
- Merge: Combination of insert and update

**Generate SQL:**
- Automatically generate CREATE TABLE statement
- Execute DDL directly from Informatica
- Export SQL for DBA review

---

### Source vs Target Designer: Quick Comparison

| Feature | Source Analyzer | Target Designer |
|---------|-----------------|-----------------|
| Primary Use | Import source structures | Create/import target structures |
| Common Action | Import existing tables/files | Create new or import existing |
| Modification | Metadata only | Can generate actual DB objects |
| File Support | Read file definitions | Write file definitions |
| SQL Generation | No | Yes (CREATE TABLE) |

---

## Topic 7: Transformations - Overview and Categories

### What is a Transformation?

**Transformation:** A repository object that generates, modifies, or passes data.

**Think of it as:** The "processing" step in ETL - this is where business logic happens.

**Example:**
```
Source Data:        Transformation:         Target Data:
John smith    →     Convert to Upper   →    JOHN SMITH
50000         →     Add 10% bonus      →    55000
NULL          →     Replace with 0     →    0
```

---

### Why Transformations?

Business requirements need data manipulation:
- **Cleansing:** Fix data quality issues (nulls, duplicates, formatting)
- **Validation:** Ensure data meets business rules
- **Calculations:** Derive new values (totals, percentages)
- **Filtering:** Include only relevant records
- **Aggregation:** Sum, count, average across groups
- **Joining:** Combine data from multiple sources
- **Lookups:** Reference data from other tables

---

### Transformation Categories

#### 1. **Active vs Passive Transformations**

**Passive Transformation:**
- Does NOT change number of rows
- Row in = row out
- Examples: Expression, Sequence Generator, Lookup (certain modes)

**Example - Expression (Passive):**
```
Input:         Transformation:              Output:
3 rows    →    (Upper case name)       →    3 rows
              (Calculate bonus)
```

**Active Transformation:**
- CAN change number of rows
- May add, remove, or combine rows
- Examples: Filter, Aggregator, Router, Joiner

**Example - Filter (Active):**
```
Input:         Transformation:              Output:
100 rows  →    (Keep only SALARY > 50000) → 30 rows
```

**Example - Aggregator (Active):**
```
Input:         Transformation:              Output:
100 rows  →    (Group by DEPT,            → 10 rows
               Sum SALARY)                  (one per dept)
```

---

#### 2. **Connected vs Unconnected Transformations**

**Connected Transformation:**
- Physically connected in mapping dataflow
- Part of the pipeline (receives and passes data)
- Most transformations are connected

**Visual:**
```
Source → Transformation → Transformation → Target
         (Connected)      (Connected)
```

**Unconnected Transformation:**
- Not in main dataflow
- Called from other transformations (like a function)
- Used for reusable lookups

**Visual:**
```
Source → Expression ----calls---→ [Lookup]
              ↓                    (Unconnected)
           Target
```

**Example Use Case:**
Expression transformation calls unconnected Lookup to validate a code:
```
If LOOKUP_VALUE(COUNTRY_CODE) = 'VALID' then COUNTRY_CODE else 'UNKNOWN'
```

---

### Standard Transformations in PowerCenter

**Here's the complete list** (we'll deep-dive in Module 3):

#### **Passive Transformations:**

1. **Expression** 🔥 (Most Used)
   - Calculate values, string operations, date functions
   - Row-by-row processing

2. **Sequence Generator**
   - Generate unique numeric keys (surrogate keys)
   - Auto-increment values

3. **Lookup** (Can be active or passive)
   - Search reference data from tables/files
   - Validate, enrich data

4. **Stored Procedure**
   - Call database stored procedures
   - Pre/post session SQL

5. **SQL Transformation**
   - Execute SQL queries mid-pipeline
   - Insert/update/delete during transformation

---

#### **Active Transformations:**

6. **Filter** 🔥 (High Priority)
   - Remove unwanted rows based on conditions
   - Example: Keep only active employees

7. **Router** 🔥 (High Priority)
   - Split data into multiple streams based on conditions
   - Example: Route high/medium/low salary to different targets

8. **Aggregator** 🔥 (High Priority)
   - Perform calculations on groups
   - SUM, AVG, COUNT, MIN, MAX

9. **Sorter**
   - Sort data ascending/descending
   - Required before certain transformations (Aggregator, Joiner)

10. **Joiner** 🔥 (High Priority)
    - Combine two sources (like SQL JOIN)
    - Inner, outer, full outer joins

11. **Rank**
    - Rank records based on criteria
    - Top N, Bottom N analysis

12. **Union**
    - Combine multiple data streams (vertical merge)
    - Like SQL UNION

13. **Update Strategy** 🔥 (Critical for loads)
    - Determine insert/update/delete operations
    - Flag rows for specific operations

14. **Normalizer**
    - Convert columns to rows
    - Denormalize data from COBOL sources

---

### Transformation Selection Guide

**Common Scenarios:**

| Requirement | Transformation |
|-------------|----------------|
| Calculate derived columns | Expression |
| Generate surrogate keys | Sequence Generator |
| Remove rows based on condition | Filter |
| Split data to multiple paths | Router |
| Get reference data | Lookup |
| Sum/count by groups | Aggregator |
| Combine two sources | Joiner |
| Determine insert vs update | Update Strategy |
| Find top 10 customers | Rank |
| Merge two similar sources | Union |

---

## Topic 8: Mappings - Creating and Configuring

### What is a Mapping?

**Mapping:** A visual representation of data flow from sources through transformations to targets.

**Think of it as:** The "blueprint" of your ETL process - it shows WHAT to do with data (not WHEN to run it).

**Anatomy of a Mapping:**
```
┌────────┐     ┌──────────────┐     ┌──────────┐     ┌────────┐
│ Source │ --> │ Source       │ --> │ Transfor │ --> │ Target │
│        │     │ Qualifier    │     │ mations  │     │        │
└────────┘     └──────────────┘     └──────────┘     └────────┘
  (Read)       (Extract logic)       (Transform)      (Load)
```

---

### Mapping Components

#### 1. **Source Definition**
- Imported from Source Analyzer
- Represents data to be extracted

#### 2. **Source Qualifier** (Auto-created)
- Special transformation between source and rest of mapping
- Functions:
  - Apply source filter (WHERE clause)
  - Join multiple tables at source
  - Sort data at source
  - Select specific columns
  - Custom SQL override

**Why Source Qualifier?**
- Push processing to database (better performance)
- Database can filter/sort faster than Informatica server

#### 3. **Transformations**
- Processing logic
- Can have multiple transformations in sequence
- Data flows through them

#### 4. **Target Definition**
- From Target Designer
- Receives transformed data

---

### Creating Your First Mapping

**Scenario:** Load employee data from EMPLOYEES table to EMP_DWH table, converting names to uppercase.

**Step-by-Step:**

1. **Open Designer → Mappings → Create**
   - Name: m_Load_Employees
   - Folder: Select project folder
   - Click OK

2. **Mapping Workspace Opens (blank canvas)**

3. **Add Source:**
   - Drag EMPLOYEES from Source Analyzer to workspace
   - Source appears with its columns
   - Source Qualifier automatically created

4. **Add Transformation:**
   - Transformations menu → Create → Expression
   - Name: exp_Transform_Names
   - Add ports (columns):
     - Input: EMP_NAME (from source)
     - Output: EMP_NAME_UPPER (new column)
     - Expression: `UPPER(EMP_NAME)`

5. **Add Target:**
   - Drag EMP_DWH from Target Designer to workspace

6. **Connect Objects:**
   - Click port in Source Qualifier, drag to Expression
   - Click port in Expression, drag to Target
   - Map all columns:
     - Source.EMP_ID → Target.EMP_ID
     - Expression.EMP_NAME_UPPER → Target.EMP_NAME
     - Source.SALARY → Target.SALARY

7. **Validate Mapping:**
   - Mappings → Validate
   - Check for errors
   - Green checkmark = valid

8. **Save Mapping:**
   - Repository → Save

**Visual Result:**
```
[EMPLOYEES] → [SQ_EMPLOYEES] → [EXP_TRANSFORM] → [EMP_DWH]
              (Source         (UPPER function)   (Target)
               Qualifier)
```

---

### Mapping Designer Interface

**Key Elements:**

**Workspace:**
- Canvas where you design
- Drag and drop objects
- Draw connections (links)

**Toolbar:**
- Add transformations
- Link objects
- Validate, save

**Navigator:**
- Shows all objects in mapping
- Quick navigation

**Output Window:**
- Validation messages
- Errors and warnings

---

### Port Concepts (Important!)

**Port:** A column in a transformation.

**Port Types:**

1. **Input Port (I):**
   - Receives data from previous transformation
   - Example: EMP_NAME coming from source

2. **Output Port (O):**
   - Sends data to next transformation
   - Example: Transformed name going to target

3. **Variable Port (V):**
   - Store intermediate calculations
   - Not passed to next transformation
   - Used within the transformation

**Port Representation:**
```
Expression Transformation
┌────────────────────────────┐
│ I  EMP_NAME                │ ← Input from source
│ V  v_FirstPart             │ ← Variable for calculation
│ O  O_EMP_NAME_UPPER        │ ← Output to target
└────────────────────────────┘
```

---

### Connecting Transformations

**Link/Connection:** Line between transformations showing data flow.

**Rules:**
- Output port → Input port
- Data types should match (or be convertible)
- Cannot create circular dependencies

**Connection Methods:**

1. **Drag and Drop:**
   - Click source port, drag to target port

2. **Autolink:**
   - Select both transformations
   - Mappings → Autolink (connects matching column names)

3. **Link by Position:**
   - Connects in order (first to first, second to second)

---

### Mapping Properties

**Access:** Mappings → Edit (right-click mapping)

**Key Properties:**

**General:**
- Mapping name
- Description
- Folder location

**Properties Tab:**
- Tracing Level: Data, normal, verbose (for debugging)
- Parameter: Can use mapping parameters/variables

**Advanced:**
- Pushdown Optimization: Push transformations to database
- Partitioning: Parallel processing

---

### Mapping Validation

**Why Validate?**
- Check for errors before execution
- Ensure all ports connected
- Verify data type compatibility
- Identify missing links

**Validation Results:**

**Valid Mapping:**
```
✓ Mapping is valid and can be saved
✓ All required ports connected
✓ No data type mismatches
```

**Invalid Mapping:**
```
✗ Target EMP_ID not connected
✗ Data type mismatch: String to Number
✗ Duplicate port names in transformation
```

**Best Practice:** Always validate before saving!

---

### Mapping Best Practices

1. **Naming Conventions:**
   - Mappings: `m_LoadEmployees` or `mp_EmployeeLoad`
   - Transformations: `exp_`, `lkp_`, `agr_`, `flt_` prefixes

2. **Organization:**
   - Group related objects
   - Use consistent layout (left to right flow)
   - Add descriptions/comments

3. **Performance:**
   - Filter early (remove unwanted rows ASAP)
   - Use Source Qualifier filtering when possible
   - Sort at source instead of using Sorter transformation

4. **Maintainability:**
   - Use mapplets for reusable logic
   - Document complex expressions
   - Keep mappings focused (single purpose)

---

## SUMMARY: MODULE 2 KEY CONCEPTS

### Sources & Targets
✅ Source = where data comes FROM (databases, flat files)  
✅ Target = where data goes TO (databases, flat files)  
✅ Source Analyzer = import source definitions  
✅ Target Designer = create/import target definitions  

### Transformations
✅ Active = can change row count (Filter, Aggregator, Router)  
✅ Passive = does NOT change row count (Expression, Lookup)  
✅ Connected = in data flow pipeline  
✅ Unconnected = called like a function  

### Mappings
✅ Mapping = visual data flow from source to target  
✅ Source Qualifier = auto-created, can filter/join at source  
✅ Ports = columns (Input, Output, Variable)  
✅ Always validate mapping before saving  

---

## PRACTICE SCENARIOS FOR MCQs

**Scenario 1:** You need to load data from a CSV file. Which tool do you use to import the file structure?
- **Answer:** Source Analyzer → Import from File

**Scenario 2:** Your mapping removes rows where SALARY < 30000. Is this an active or passive transformation?
- **Answer:** Active (Filter transformation changes row count)

**Scenario 3:** You want to convert EMP_NAME to uppercase. Which transformation?
- **Answer:** Expression (passive transformation for calculations)

**Scenario 4:** Where can you apply a WHERE clause to filter data at the source database?
- **Answer:** Source Qualifier properties (SQL override or filter condition)

**Scenario 5:** In your mapping, you see validation error "Target port not connected." What's the issue?
- **Answer:** Not all target columns are mapped from transformations

**Scenario 6:** You need to generate unique sequence numbers for new records. Which transformation?
- **Answer:** Sequence Generator

---

## HANDS-ON EXERCISE (Try This!)

**Exercise: Create a Simple Mapping**

**Objective:** Load employee data, filter active employees only, convert name to uppercase.

**Steps:**
1. Import source: EMPLOYEES table (or create flat file)
2. Create target: EMP_ACTIVE table
3. Create mapping: m_Load_Active_Employees
4. Add Filter transformation: FILTER_ACTIVE
   - Condition: STATUS = 'A'
5. Add Expression transformation: EXP_TRANSFORM
   - Expression: UPPER(EMP_NAME)
6. Connect: Source → Filter → Expression → Target
7. Validate and save

**Expected Flow:**
```
EMPLOYEES → SQ → FILTER → EXPRESSION → EMP_ACTIVE
                (STATUS='A') (UPPER)
```

Try this, and let me know:
- Did you face any issues?
- Which part was confusing?
- Ready for deeper transformation details?

---

## NEXT MODULE PREVIEW

**Module 3: Deep Dive into Transformations** 🔥

This is THE MOST IMPORTANT module for TCS Wings exam!

We'll cover:
- Expression (calculations, string functions, date functions)
- Filter & Router (conditional processing)
- Lookup (connected & unconnected)
- Aggregator (grouping, calculations)
- Joiner (combining sources)
- Update Strategy (insert/update/delete logic)

These 6 transformations appear in 70% of scenario-based MCQs!

**Ready for Module 3?**