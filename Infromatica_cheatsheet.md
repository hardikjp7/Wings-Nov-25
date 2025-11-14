# INFORMATICA POWERCENTER - QUICK EXAM CHEAT SHEET

## 🔥 CRITICAL CONCEPTS (MEMORIZE FIRST!)

### SCD Type 2 (HIGHEST EXAM PRIORITY!)
```
Required Components:
✓ Dynamic Lookup Cache (MUST enable!)
✓ Sequence Generator (for surrogate keys)
✓ Router (3 groups: NEW, EXPIRE_OLD, INSERT_NEW)
✓ Update Strategy (DD_INSERT, DD_UPDATE, DD_INSERT)
✓ Target columns: START_DATE, END_DATE, IS_CURRENT, VERSION

Key Settings:
✓ Lookup: Dynamic Cache = YES, Condition: ID = LKP_ID AND IS_CURRENT = 'Y'
✓ Session: Target mode = Data Driven (CRITICAL!)
✓ NEWLOOKUPROW = 1 → New record (INSERT)
✓ NEWLOOKUPROW = 0 → Existing record (check for changes)

Logic Flow:
1. If NEWLOOKUPROW = 1 → INSERT with IS_CURRENT='Y', VERSION=1
2. If Changed → UPDATE old (IS_CURRENT='N', END_DATE=SYSDATE)
              + INSERT new (IS_CURRENT='Y', START_DATE=SYSDATE, VERSION+1)
3. If No Change → Skip (NOCHANGE group)
```

### Update Strategy Codes
```
DD_INSERT = 0   → Insert new row
DD_UPDATE = 1   → Update existing row
DD_DELETE = 2   → Delete row
DD_REJECT = 3   → Reject/skip row
```

### Active vs Passive Transformations
```
ACTIVE (Changes Row Count):
✓ Filter, Router, Aggregator, Joiner, Rank
✓ Update Strategy, Normalizer, Union, SQL Transform

PASSIVE (No Row Count Change):
✓ Expression, Sequence Generator
✓ Lookup (can be active), Stored Procedure
```

---

## MODULE 1: FUNDAMENTALS

### Architecture Flow
```
Domain (Admin framework)
  ↓
Repository Service (Manages metadata)
  ↓
Repository Database (Stores metadata)
  ↓
Integration Service (Executes workflows)
```

### Client Tools
```
Designer → Create mappings (4 windows)
  ├─ Source Analyzer: Import sources
  ├─ Target Designer: Create targets
  ├─ Transformation Developer: Build transformations
  └─ Mapplet Designer: Reusable components

Workflow Manager → Create workflows/sessions
Workflow Monitor → Monitor execution, view logs
Repository Manager → Admin, users, folders
```

### Key Concepts
```
✓ Repository stores METADATA (not actual data)
✓ Metadata = definitions, mappings, workflows, connections
✓ Integration Service = execution engine
✓ Session = runtime instance of mapping
✓ Workflow = orchestration of sessions/tasks
```

---

## MODULE 2: CORE COMPONENTS

### Sources & Targets
```
Sources: Where data comes FROM
- Relational: Oracle, SQL Server, DB2
- Flat Files: CSV, TXT (delimited/fixed-width)

Targets: Where data goes TO
- Same as sources
- Can INSERT, UPDATE, DELETE, TRUNCATE
```

### Source Qualifier
```
Auto-created with relational sources
Functions:
✓ Filter at source (WHERE clause)
✓ Join tables at source (best performance)
✓ Sort at source
✓ SQL Override for custom queries
```

### Mapping Basics
```
Flow: Source → Source Qualifier → Transformations → Target

Port Types:
I = Input (receives data)
O = Output (sends data)
V = Variable (intermediate calculations)
L = Lookup (for lookup transformations)
```

---

## MODULE 3: TRANSFORMATIONS

### Expression Transformation ⭐
```
Type: Passive, Connected
Use: Calculations, string/date operations

Key Functions:
IIF(condition, true, false) → Conditional
DECODE(val, v1, r1, v2, r2, default) → Multi-condition
UPPER/LOWER/INITCAP → String case
SUBSTR(str, start, len) → Substring
CONCAT(str1, str2) → Concatenate
LTRIM/RTRIM → Remove spaces
LENGTH(str) → String length
INSTR(str, search) → Find position
ROUND/TRUNC → Numeric operations
SYSDATE → Current date/time
TO_DATE(str, format) → String to date
TO_CHAR(date, format) → Date to string
ADD_TO_DATE(date, unit, val) → Date arithmetic
ISNULL(val) → Check NULL
```

### Filter Transformation
```
Type: Active, Connected
Use: Remove rows based on condition
Condition Examples:
- STATUS = 'A'
- SALARY > 50000
- HIRE_DATE >= ADD_TO_DATE(SYSDATE, 'DD', -90)
- NOT ISNULL(COMMISSION)

Performance: Filter early in pipeline!
```

### Router Transformation
```
Type: Active, Connected
Use: Route data to multiple paths

Components:
- Multiple Output Groups (with conditions)
- Default Group (catches unmatched)

Example Groups:
- HIGH: SALARY > 80000
- MID: SALARY 50000-80000
- LOW: SALARY < 50000
- DEFAULT: All others

Note: Row goes to FIRST matching group only!
```

### Aggregator Transformation ⭐
```
Type: Active, Connected
Use: Group and aggregate data

Functions:
SUM, AVG, COUNT, MIN, MAX, FIRST, LAST
STDDEV, VARIANCE, MEDIAN, PERCENTILE

Ports:
- Group By = TRUE → Grouping columns
- Group By = FALSE → Columns to aggregate

CRITICAL Optimization:
✓ Add Sorter before Aggregator (sort by group-by columns)
✓ Enable "Sorted Input" in Aggregator
✓ Reduces memory by 70-90%!
```

### Lookup Transformation ⭐⭐
```
Type: Passive or Active, Connected or Unconnected
Use: Retrieve reference data

Connected Lookup:
- Part of pipeline
- Returns multiple columns
- Used for: Enrichment, validation

Unconnected Lookup:
- Called like function: :LKP.LKP_NAME(input)
- Returns single value
- Used for: Conditional lookups

Cache Types:
✓ Static: Read-only (reference data)
✓ Dynamic: Updatable during session (SCD!)
✓ Persistent: Saved to disk, reused across runs
✓ Shared: Multiple lookups share same cache

Performance:
✓ SQL Override to filter/limit columns
✓ Index lookup columns in database
✓ Use Source Qualifier join if same DB
```

### Joiner Transformation
```
Type: Active, Connected
Use: Combine two sources (like SQL JOIN)

Join Types:
- Normal (Inner): Only matching rows
- Master Outer (Left): All Master + matching Detail
- Detail Outer (Right): All Detail + matching Master
- Full Outer: All rows from both

Master vs Detail:
✓ SMALLER source = Detail (loaded in memory)
✓ LARGER source = Master (drives join)
✓ Wrong designation = poor performance!

Optimization:
✓ Sort both sources by join columns
✓ Enable "Sorted Input" in Joiner
✓ Or use Source Qualifier join (same DB)
```

### Update Strategy Transformation ⭐⭐
```
Type: Active, Connected
Use: Determine INSERT/UPDATE/DELETE operations

Expression:
IIF(ISNULL(LKP_KEY), DD_INSERT,
  IIF(HAS_CHANGED, DD_UPDATE, DD_REJECT))

CRITICAL Session Settings:
✓ Target: "Treat source rows as" = Data Driven
✓ Update as Update: Enabled
✓ Define Key Columns in target properties
```

### Sequence Generator
```
Type: Passive, Connected
Use: Generate unique numeric values (surrogate keys)

Properties:
- Start Value: 1
- Increment By: 1
- End Value: 999999999
- Cycle: No
- Cache: 1000

Ports: NEXTVAL, CURRVAL
```

### Sorter Transformation
```
Type: Active, Connected
Use: Sort data

Options:
✓ Sort Keys: Define columns and direction (ASC/DESC)
✓ Distinct: Remove duplicates
✓ Case Sensitive: For string sorting

Use Before: Aggregator, Joiner (for Sorted Input optimization)
```

### Rank Transformation
```
Type: Active, Connected
Use: Top N / Bottom N analysis

Properties:
- Top/Bottom: Select which to keep
- Number of Ranks: How many (e.g., 10)
- Rank By: Column to rank on
- Group By: Partition for ranking

Example: Top 5 salespeople per region
```

### Union Transformation
```
Type: Active, Connected
Use: Combine multiple sources vertically (like UNION ALL)

Note: Does NOT remove duplicates (unlike SQL UNION)
All input groups must have compatible structures
```

---

## MODULE 4: ADVANCED TECHNIQUES

### Mapplets
```
Purpose: Reusable transformation logic

Components:
- Input Transformation (required)
- Regular transformations
- Output Transformation (required)

Cannot Include:
✗ Source Qualifier
✗ Target definitions
✗ Normalizer, Joiner (some restrictions)

Benefits:
✓ Reusability across mappings
✓ Standardization
✓ Easier maintenance
```

### Workflows & Sessions
```
Workflow = Execution plan (WHEN to run)
Session = Runtime instance of mapping (HOW to run)

Workflow Tasks:
- Session: Run mapping
- Decision: Branch based on condition
- Email: Send notifications
- Command: Execute shell scripts
- Timer: Wait/delay
- Assignment: Set variables

Session Configuration:
✓ Source/Target Connections
✓ Commit Interval (10000-100000)
✓ DTM Buffer Size (64-128 MB)
✓ Pre-SQL / Post-SQL
```

### Parameters & Variables
```
Mapping Parameters:
- Constants defined in mapping
- Values set in parameter file
- Example: $$MinSalary

Mapping Variables:
- Can change during session
- Persistent = saved between runs
- Example: $$LastLoadDate (for incremental)

Session Parameters:
- $PMSourceFileDir, $PMTargetFileDir
- $SYSDATE, $PMSessionStartTime
- $InputFile, $PMBadFileDir

Parameter File Format:
[Folder.Workflow.Session]
$$MinSalary=60000
$PMSourceFileDir=/data/input/
```

### Error Handling
```
Session Properties:
- Stop on Errors: Max errors before fail (default 0 = no limit)
- Tracing Level: Normal, Verbose (for debugging)
- Error Log: File or table

Bad Files:
- Location: $PMBadFileDir
- Contains: Rejected rows
- Causes: Constraint violations, type mismatches

Recovery:
- Enable Recovery: YES (session and workflow)
- Allows restart from failure point
- Requires appropriate commit intervals
```

---

## MODULE 5: PERFORMANCE & OPTIMIZATION

### Performance Tuning Priority
```
1. Source Optimization (Filter/Join at source)
2. Sorted Input (Aggregator/Joiner)
3. Partitioning (Parallel processing)
4. Pushdown Optimization (Database processing)
5. Bulk Load (Fast target loading)
6. Cache Optimization (Lookup)
```

### Sorted Input Optimization ⭐⭐⭐
```
For Aggregator:
1. Add Sorter before Aggregator
2. Sort by all group-by columns
3. Aggregator Properties → Sorted Input = TRUE
Result: 70-90% memory reduction

For Joiner:
1. Sort Master by join columns
2. Sort Detail by join columns
3. Joiner Properties → Sorted Input = TRUE
Result: No cache needed, much less memory
```

### Partitioning Types
```
Pass-Through: No repartitioning
Round-Robin: Even distribution (general use)
Hash: By key value (for Aggregator/Joiner on group/join columns)
Key Range: By value ranges

Best Practice:
✓ Number of Partitions = Number of CPUs
✓ Hash for Aggregator/Joiner (on group-by/join columns)
✓ Round-Robin for balanced general processing
✓ Check partition balance in session log
```

### Pushdown Optimization
```
When: Source and target in SAME database
What: Transformations executed in database (not Informatica)
How: Session Properties → Pushdown = Source/Target/Full

Benefits:
✓ Reduced network traffic
✓ Leverages database power
✓ Faster for large datasets

Supported: Filter, Expression, Aggregator, Joiner, Sorter
Not Supported: Complex expressions, mapplets, unconnected lookups
```

### Session Performance Settings
```
DTM Buffer Pool Size:
- Auto (default) or Manual
- Large sessions: 64000000 - 128000000 (64-128 MB)

Buffer Block Size:
- Default: 64000 (64 KB)
- Adjust for row size

Commit Interval:
- Small (5000-10000): Safer, slower
- Large (50000-100000): Faster, more risk
- Balance based on requirements

Target Load Type:
- Normal: Standard
- Bulk: 5-10x faster (disable constraints/indexes)
```

### Lookup Optimization
```
1. SQL Override: Filter rows, select only needed columns
   SELECT DEPT_ID, DEPT_NAME FROM DEPT WHERE ACTIVE='Y'

2. Index lookup columns in database
   CREATE INDEX idx_dept ON DEPT(DEPT_ID);

3. Persistent Cache: Reuse across sessions

4. Shared Cache: Multiple lookups share cache

5. Replace with Source Qualifier join (same DB)
```

---

## MODULE 6: SCENARIO-BASED CONCEPTS

### SCD Type 1 (Overwrite)
```
Use: No history needed, simple updates

Logic:
IF new record → INSERT
IF changed → UPDATE
IF no change → SKIP

Transformations:
- Lookup (Dynamic): Check existence
- Expression: Detect change
- Router: INSERT/UPDATE/SKIP
- Update Strategy: DD_INSERT/DD_UPDATE
```

### SCD Type 2 (Full History) ⭐⭐⭐
```
Use: Track all changes with full history

Required Columns:
- CUST_SK (Surrogate key)
- CUST_ID (Business key)
- Attributes (tracked columns)
- START_DATE, END_DATE
- IS_CURRENT ('Y' or 'N')
- VERSION (optional)

Logic:
IF NEWLOOKUPROW = 1 → INSERT (new customer)
  START_DATE = SYSDATE
  END_DATE = '9999-12-31'
  IS_CURRENT = 'Y'
  VERSION = 1

IF Changed → TWO operations:
  A) UPDATE old version:
     END_DATE = SYSDATE
     IS_CURRENT = 'N'
  
  B) INSERT new version:
     New CUST_SK (NEXTVAL)
     START_DATE = SYSDATE
     END_DATE = '9999-12-31'
     IS_CURRENT = 'Y'
     VERSION = OLD_VERSION + 1

IF No Change → SKIP

Transformations Flow:
Source → Lookup (Dynamic, IS_CURRENT='Y') → Expression (Detect) →
Router (NEW/EXPIRE/INSERT_NEW) → Update Strategy → Target

Critical Settings:
✓ Lookup: Dynamic Cache = YES
✓ Session: Data Driven mode
✓ Three Update Strategy transformations or paths
```

### SCD Type 3 (Limited History)
```
Use: Need current and previous value only

Columns:
- CURRENT_STATUS
- PREVIOUS_STATUS
- STATUS_CHANGE_DATE

Logic:
IF changed:
  PREVIOUS = CURRENT (shift values)
  CURRENT = NEW
  CHANGE_DATE = SYSDATE

Simple UPDATE operation
```

### Incremental Loading ⭐⭐
```
Purpose: Load only changed/new records since last run

Method 1: Timestamp-Based (Most Common)
1. Create Mapping Variable: $$LastLoadDate
   - Type: Date/Time
   - Persistent: YES

2. Source Qualifier SQL Override:
   WHERE LAST_MODIFIED_DATE > '$$LastLoadDate'

3. Update Variable (Expression):
   $$LastLoadDate = MAX(LAST_MODIFIED_DATE)

Method 2: Flag-Based
- Source system sets change flag
- Filter: WHERE CHANGE_FLAG IN ('N', 'U')
- Post-SQL: Reset flags after load

Method 3: Version Number
- Similar to timestamp, use version number
```

### Data Quality Validation
```
Common Validations:

1. Null Check:
   V_HasNulls = IIF(ISNULL(EMP_ID) OR ISNULL(NAME), 1, 0)

2. Format Validation:
   V_EmailValid = IIF(INSTR(EMAIL, '@') > 0, 1, 0)
   V_PhoneValid = IIF(LENGTH(PHONE) = 10, 1, 0)

3. Range Validation:
   V_SalaryValid = IIF(SALARY >= 10000 AND SALARY <= 500000, 1, 0)
   V_AgeValid = IIF(AGE >= 18 AND AGE <= 65, 1, 0)

4. Reference Data Validation:
   Lookup: Check if foreign key exists
   V_IsValid = IIF(ISNULL(LKP_KEY), 0, 1)

Pattern:
Source → Expression (Validate) → Router → Valid/Invalid Targets
```

### Duplicate Detection
```
Method 1: Sorter + Expression
- Sorter: Sort by business key
- Expression: Compare with previous row
  V_PrevID = ID (save for next row)
  O_IsDup = IIF(ID = V_PrevID, 1, 0)

Method 2: Aggregator
- Group By: Business key
- COUNT(*) > 1 = Duplicates
```

---

## MODULE 7: EXAM STRATEGY

### Time Management
```
MCQ Section (60 min, 30-40 questions):
- Pass 1 (30 min): Easy questions
- Pass 2 (20 min): Difficult questions
- Pass 3 (10 min): Review and guess

Hands-On (120 min, 2-4 tasks):
- Analysis: 5 min
- Implementation: 90 min
- Testing: 20 min
- Buffer: 5 min
```

### Common Exam Mistakes ⚠️
```
❌ 1. Dynamic Lookup not enabled (SCD Type 2)
❌ 2. "Data Driven" mode not set (Update Strategy)
❌ 3. Variable not marked "Persistent" (Incremental)
❌ 4. Key Columns not defined (UPDATE operations)
❌ 5. Mapping not validated before session creation
❌ 6. Not testing before submission
```

### Hands-On Checklist
```
For SCD Type 2:
☐ Target has START_DATE, END_DATE, IS_CURRENT, VERSION
☐ Sequence Generator added
☐ Lookup: Dynamic Cache = YES
☐ Lookup Condition includes IS_CURRENT = 'Y'
☐ Router has 3 groups (NEW, EXPIRE, INSERT_NEW)
☐ Update Strategy correctly configured
☐ Session: Data Driven mode
☐ Target: Key columns defined
☐ Validated with green checkmark
☐ Tested with sample data

For Incremental Load:
☐ Variable created with Persistent = YES
☐ Source Qualifier has SQL Override with variable
☐ Expression updates variable with MAX value
☐ Tested: Run 1 (full), Run 2 (incremental)
```

### Quick Debugging
```
Session Fails:
1. Check session log (first error)
2. Common causes:
   - Connection issue
   - Table doesn't exist
   - Data Driven not enabled
   - SQL syntax error

Wrong Row Counts:
1. Check transformation statistics in log
2. Identify where rows lost/added
3. Verify filter conditions
4. Check bad file for rejected rows

Performance Issues:
1. Workflow Monitor → Performance Details
2. Identify bottleneck transformation
3. Apply Sorted Input if Aggregator/Joiner
4. Check partitioning balance
```

---

## 🎯 MUST-KNOW FOR EXAM SUCCESS

### Top 10 MCQ Topics
```
1. SCD Type 2 (Dynamic Lookup, IS_CURRENT, NEWLOOKUPROW)
2. Active vs Passive transformations
3. Sorted Input optimization (Aggregator/Joiner)
4. Hash Partitioning (for group-by/join columns)
5. Update Strategy codes (0,1,2,3) + Data Driven mode
6. Persistent Variables (incremental loading)
7. Lookup cache types (Dynamic, Persistent, Shared)
8. Master/Detail in Joiner (smaller = Detail)
9. Pushdown Optimization (same database)
10. Session configuration (Commit Interval, DTM Buffer)
```

### Top 3 Hands-On Scenarios
```
1. SCD Type 2 (80% probability)
   - Practice until you can build in 30-40 min
   - Know all critical settings by heart

2. Incremental Load (60% probability)
   - Persistent variable setup
   - SQL Override pattern
   - Variable update logic

3. Data Validation (50% probability)
   - Expression with multiple rules
   - Router to separate valid/invalid
   - Error logging
```

### Critical Formula Quick Reference
```
IIF(condition, true_value, false_value)

DECODE(expression,
  value1, result1,
  value2, result2,
  default_result)

Null Handling:
IIF(ISNULL(column), default, column)

Date Difference:
TRUNC((SYSDATE - BIRTH_DATE) / 365.25) → Age in years

String Operations:
UPPER(LTRIM(RTRIM(column))) → Clean and uppercase
SUBSTR(string, start, length) → Extract substring
CONCAT(str1, str2) or str1 || str2 → Concatenate
```

---

## 🔥 FINAL EXAM DAY REMINDERS

**30 Minutes Before Exam:**
```
✓ Skim this cheat sheet (5 min)
✓ Review SCD Type 2 steps (3 min)
✓ Review Update Strategy codes (1 min)
✓ Deep breath, positive mindset (1 min)
✓ Ready to excel! 💪
```

**During Exam:**
```
✓ Read questions carefully (keywords!)
✓ For MCQs: Eliminate wrong answers first
✓ For hands-on: Validate before creating session
✓ Test mappings at least once
✓ Check session logs for errors
✓ Save work frequently
✓ Don't panic if stuck - move to next question
✓ Trust your preparation!
```

---

## 🌟 YOU'VE GOT THIS!

You've studied comprehensively. You know the concepts. You've practiced scenarios.

**Trust your preparation. Stay calm. Execute confidently.**

**GOOD LUCK! 🚀🎯🏆**