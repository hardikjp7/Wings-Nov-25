# MODULE 6: SCENARIO-BASED CONCEPTS

## Overview

This module covers real-world ETL scenarios that frequently appear in TCS Wings exam - both MCQs and hands-on. Master these patterns and you'll excel in the exam!

**Topics Covered:**
- **Topic 25:** Slowly Changing Dimensions (SCD) - Types 1, 2, 3
- **Topic 26:** Incremental Loading Strategies
- **Topic 27:** Data Quality and Validation Scenarios
- **Topic 28:** Common Business Scenarios & Solutions

---

## Topic 25: Slowly Changing Dimensions (SCD) 🔥🔥🔥

### What is a Slowly Changing Dimension?

**Dimension:** Descriptive information about business entities (Customer, Product, Location, Employee)

**Slowly Changing:** Attributes change over time, but infrequently

**Problem:** How do we handle changes in dimension data?

**Example:**
```
Customer Dimension:
CUST_ID | NAME        | ADDRESS           | CITY      | STATUS
1001    | John Smith  | 123 Main St       | New York  | Gold

Change occurs:
- John moves to 456 Oak Ave, Boston
- Status upgraded to Platinum

Question: How do we track this change?
```

**Three Standard Approaches:**
1. **Type 1:** Overwrite (no history)
2. **Type 2:** Add new version (full history)
3. **Type 3:** Add new column (limited history)

---

### SCD Type 1: Overwrite 🔥

**Concept:** Replace old value with new value (no history kept)

**When to Use:**
- History not important
- Corrections to data errors
- Changes that don't require tracking

**Example:**

**Before Change:**
```
CUST_SK | CUST_ID | NAME        | ADDRESS         | CITY      | STATUS
1       | 1001    | John Smith  | 123 Main St     | New York  | Gold
```

**After Change (Type 1):**
```
CUST_SK | CUST_ID | NAME        | ADDRESS         | CITY      | STATUS
1       | 1001    | John Smith  | 456 Oak Ave     | Boston    | Platinum
                                 ↑ Overwritten    ↑ Overwritten ↑ Overwritten
```

**Result:** Old values lost, no history preserved

---

#### Implementing SCD Type 1

**Mapping Design:**

**Step 1: Lookup existing record**
```
Source (Daily Customer Changes)
  ↓
Lookup: LKP_CUSTOMER_DIM
  - Lookup Table: DIM_CUSTOMER
  - Condition: CUST_ID = LKP_CUST_ID
  - Cache: Dynamic
  - Returns: CUST_SK, NAME, ADDRESS, CITY, STATUS
```

**Step 2: Detect changes**
```
Expression: EXP_DETECT_CHANGE
  - Input: Source columns + Lookup return columns
  - Logic:
    V_IsNewRecord = IIF(ISNULL(LKP_CUST_SK), 1, 0)
    V_HasChanged = IIF(
                     SRC_ADDRESS != LKP_ADDRESS OR
                     SRC_CITY != LKP_CITY OR
                     SRC_STATUS != LKP_STATUS, 1, 0)
    O_ChangeFlag = IIF(V_IsNewRecord = 1, 'INSERT',
                     IIF(V_HasChanged = 1, 'UPDATE', 'NOCHANGE'))
```

**Step 3: Route based on change**
```
Router: RTR_CHANGE_TYPE
  - Group INSERT: O_ChangeFlag = 'INSERT'
  - Group UPDATE: O_ChangeFlag = 'UPDATE'
  - Default: O_ChangeFlag = 'NOCHANGE' (no action)
```

**Step 4: Apply changes**
```
Update Strategy: UPD_TYPE1
  - For INSERT group: DD_INSERT
  - For UPDATE group: DD_UPDATE
  ↓
Target: DIM_CUSTOMER (Data Driven mode)
```

**Complete Flow:**
```
Source → Lookup (Dynamic) → Expression (Detect) → Router → Update Strategy → Target
                                                    ├─INSERT→ DD_INSERT
                                                    ├─UPDATE→ DD_UPDATE
                                                    └─NOCHANGE→ (skip)
```

---

### SCD Type 2: Add New Version 🔥🔥

**Concept:** Insert new row for changed record, maintain full history

**When to Use:**
- History is critical (regulatory, audit)
- Analytics need historical trends
- Most common SCD type in data warehouses

**Key Columns:**
- **Effective Date (START_DATE):** When version became active
- **End Date (END_DATE):** When version became inactive
- **Current Flag (IS_CURRENT):** Flag for current version (Y/N)
- **Version Number (VERSION):** Optional version counter

**Example:**

**Before Change:**
```
CUST_SK | CUST_ID | NAME        | ADDRESS      | STATUS | START_DATE | END_DATE   | IS_CURRENT
1       | 1001    | John Smith  | 123 Main St  | Gold   | 2024-01-01 | 9999-12-31 | Y
```

**After Change (Type 2):**
```
CUST_SK | CUST_ID | NAME        | ADDRESS      | STATUS   | START_DATE | END_DATE   | IS_CURRENT
1       | 1001    | John Smith  | 123 Main St  | Gold     | 2024-01-01 | 2025-11-06 | N ← Expired
2       | 1001    | John Smith  | 456 Oak Ave  | Platinum | 2025-11-06 | 9999-12-31 | Y ← New version
```

**Result:** Both versions preserved, can analyze historical data

---

#### Implementing SCD Type 2

**Mapping Design (Complex but Important!):**

**Step 1: Lookup existing current record**
```
Lookup: LKP_CUSTOMER_DIM
  - Dynamic Cache: YES
  - Lookup Table: DIM_CUSTOMER
  - Condition: CUST_ID = LKP_CUST_ID AND IS_CURRENT = 'Y'
  - Returns: CUST_SK, all attributes, NewLookupRow
```

**Step 2: Detect new vs changed**
```
Expression: EXP_DETECT_CHANGE_TYPE2

  V_IsNewRecord = LKP_NEWLOOKUPROW
  
  V_HasChanged = IIF(V_IsNewRecord = 0,
                   IIF(SRC_ADDRESS != LKP_ADDRESS OR
                       SRC_CITY != LKP_CITY OR
                       SRC_STATUS != LKP_STATUS, 1, 0),
                   0)
  
  O_ChangeType = IIF(V_IsNewRecord = 1, 'INSERT',
                   IIF(V_HasChanged = 1, 'TYPE2', 'NOCHANGE'))
```

**Step 3: Route to two streams**
```
Router: RTR_TYPE2
  - Group INSERT: O_ChangeType = 'INSERT'
  - Group TYPE2_NEW: O_ChangeType = 'TYPE2'
  - Group TYPE2_EXPIRE: O_ChangeType = 'TYPE2' (duplicate group)
  - Default: NOCHANGE
```

**Step 4A: For TYPE2 - Expire old record**
```
Expression: EXP_EXPIRE_OLD
  - O_CUST_SK = LKP_CUST_SK (existing key)
  - O_END_DATE = SYSDATE
  - O_IS_CURRENT = 'N'
  
Update Strategy: UPD_EXPIRE
  - DD_UPDATE (update existing record)
```

**Step 4B: For TYPE2 - Insert new version**
```
Expression: EXP_INSERT_NEW
  - O_CUST_SK = NEXTVAL (from Sequence Generator)
  - O_CUST_ID = SRC_CUST_ID
  - O_NAME = SRC_NAME
  - O_ADDRESS = SRC_ADDRESS (new value)
  - O_CITY = SRC_CITY (new value)
  - O_STATUS = SRC_STATUS (new value)
  - O_START_DATE = SYSDATE
  - O_END_DATE = TO_DATE('9999-12-31', 'YYYY-MM-DD')
  - O_IS_CURRENT = 'Y'
  - O_VERSION = LKP_VERSION + 1
  
Update Strategy: UPD_INSERT_NEW
  - DD_INSERT
```

**Step 4C: For INSERT - New customer**
```
Expression: EXP_NEW_CUSTOMER
  - O_CUST_SK = NEXTVAL
  - O_START_DATE = SYSDATE
  - O_END_DATE = TO_DATE('9999-12-31', 'YYYY-MM-DD')
  - O_IS_CURRENT = 'Y'
  - O_VERSION = 1
  
Update Strategy: UPD_NEW
  - DD_INSERT
```

**Complete Flow (Type 2):**
```
Source 
  ↓
Lookup (Dynamic Cache)
  ↓
Expression (Detect Change)
  ↓
Router
  ├─ INSERT → Expression → Update Strategy (DD_INSERT) ──┐
  │                                                       │
  ├─ TYPE2_EXPIRE → Expression → Update Strategy (DD_UPDATE) ─→ Target (DIM_CUSTOMER)
  │                                                       │
  └─ TYPE2_NEW → Sequence Generator → Expression → Update Strategy (DD_INSERT) ─┘
```

---

#### SCD Type 2 Variations

**Variation 1: Using Transaction Control**
```
Instead of separate Update Strategy transformations:
Use Transaction Control to commit after expiring old + inserting new
Ensures atomicity (both or neither)
```

**Variation 2: Effective Date from Source**
```
Instead of SYSDATE:
Use effective date from source (if available)
More accurate for backdated changes
```

**Variation 3: Version Number vs Surrogate Key**
```
Some designs use:
CUST_ID + VERSION as composite key
Instead of surrogate CUST_SK
```

---

### SCD Type 3: Add New Column 🔥

**Concept:** Add column(s) to store previous value (limited history - usually 1 previous)

**When to Use:**
- Need current and previous value only
- Limited history sufficient
- Less common than Type 1 or Type 2

**Key Columns:**
- **CURRENT_VALUE:** Current value
- **PREVIOUS_VALUE:** Previous value
- **EFFECTIVE_DATE:** When current became effective

**Example:**

**Before Change:**
```
CUST_SK | CUST_ID | NAME        | CURRENT_STATUS | PREVIOUS_STATUS | STATUS_CHANGE_DATE
1       | 1001    | John Smith  | Gold           | Silver          | 2024-06-01
```

**After Change (Type 3):**
```
CUST_SK | CUST_ID | NAME        | CURRENT_STATUS | PREVIOUS_STATUS | STATUS_CHANGE_DATE
1       | 1001    | John Smith  | Platinum       | Gold            | 2025-11-06
                                 ↑ New current   ↑ Old current    ↑ Updated
```

**Result:** Can compare current to previous, but only 1 level of history

---

#### Implementing SCD Type 3

**Mapping Design:**

**Step 1: Lookup existing record**
```
Lookup: LKP_CUSTOMER_DIM
  - Lookup Table: DIM_CUSTOMER
  - Condition: CUST_ID = LKP_CUST_ID
  - Returns: CUST_SK, CURRENT_STATUS, PREVIOUS_STATUS
```

**Step 2: Detect change and shift values**
```
Expression: EXP_TYPE3
  
  V_IsNewRecord = IIF(ISNULL(LKP_CUST_SK), 1, 0)
  V_HasChanged = IIF(SRC_STATUS != LKP_CURRENT_STATUS, 1, 0)
  
  O_CURRENT_STATUS = SRC_STATUS
  O_PREVIOUS_STATUS = IIF(V_HasChanged = 1, 
                          LKP_CURRENT_STATUS,  ← Shift current to previous
                          LKP_PREVIOUS_STATUS) ← Keep previous
  O_STATUS_CHANGE_DATE = IIF(V_HasChanged = 1, SYSDATE, LKP_STATUS_CHANGE_DATE)
  
  O_ChangeFlag = IIF(V_IsNewRecord = 1, 'INSERT',
                   IIF(V_HasChanged = 1, 'UPDATE', 'NOCHANGE'))
```

**Step 3: Update Strategy**
```
Router → Update Strategy
  - INSERT: DD_INSERT
  - UPDATE: DD_UPDATE
```

**Complete Flow:**
```
Source → Lookup → Expression (Shift Values) → Router → Update Strategy → Target
```

---

### SCD Type Comparison 🔥

| Aspect | Type 1 | Type 2 | Type 3 |
|--------|--------|--------|--------|
| **History** | None | Full | Limited (previous only) |
| **Storage** | Low | High (new row per change) | Low |
| **Complexity** | Simple | Complex | Medium |
| **Common Usage** | 40% | 50% | 10% |
| **Best For** | Corrections, unimportant changes | Audit, compliance, trending | Simple before/after |
| **Performance** | Fast (update) | Slower (insert+update) | Fast (update) |

---

### SCD Hybrid Approaches

**Combination SCD (Type 1 + Type 2):**
- Some attributes Type 1 (overwrite)
- Other attributes Type 2 (history)

**Example:**
```
Customer Dimension:
- Name: Type 1 (corrections, no history needed)
- Address: Type 2 (track moves for shipping analysis)
- Phone: Type 1 (corrections)
- Credit Score: Type 2 (track over time for risk)
```

**Implementation:**
```
In detect change logic:
V_Type1_Changed = IIF(SRC_NAME != LKP_NAME OR SRC_PHONE != LKP_PHONE, 1, 0)
V_Type2_Changed = IIF(SRC_ADDRESS != LKP_ADDRESS OR SRC_SCORE != LKP_SCORE, 1, 0)

O_Action = IIF(V_Type2_Changed = 1, 'TYPE2',
             IIF(V_Type1_Changed = 1, 'TYPE1', 'NOCHANGE'))
```

---

### Real-World SCD Scenarios

**Scenario 1: Employee Dimension**
```
Business Requirement:
- Track salary history (Type 2)
- Update phone number without history (Type 1)
- Track previous and current department (Type 3)

Implementation:
Hybrid SCD:
- SALARY: Type 2 (new row on change)
- PHONE: Type 1 (overwrite)
- DEPT_ID: Type 3 (CURRENT_DEPT, PREVIOUS_DEPT)
```

**Scenario 2: Product Dimension**
```
Business Requirement:
- Track price changes over time
- Needed for historical sales analysis

Implementation:
Type 2 SCD:
- PRODUCT_SK, PRODUCT_ID, PRODUCT_NAME, PRICE
- START_DATE, END_DATE, IS_CURRENT

Query for historical sales:
SELECT s.SALE_DATE, p.PRODUCT_NAME, p.PRICE, s.QUANTITY
FROM FACT_SALES s
JOIN DIM_PRODUCT p 
  ON s.PRODUCT_SK = p.PRODUCT_SK
  AND s.SALE_DATE BETWEEN p.START_DATE AND p.END_DATE
```

**Scenario 3: Customer Status Tracking**
```
Business Requirement:
- Need current and previous status only
- Used for upgrade/downgrade analysis

Implementation:
Type 3 SCD:
- CURRENT_STATUS, PREVIOUS_STATUS, STATUS_CHANGE_DATE

Analysis Query:
SELECT COUNT(*)
FROM DIM_CUSTOMER
WHERE CURRENT_STATUS = 'Gold' AND PREVIOUS_STATUS = 'Silver'
(Find customers upgraded from Silver to Gold)
```

---

## Topic 26: Incremental Loading Strategies 🔥

### What is Incremental Loading?

**Full Load:** Extract and load entire dataset every time
- Simple but inefficient
- Suitable for small dimensions

**Incremental Load:** Extract and load only changed/new records
- Efficient for large datasets
- Reduces processing time and resources

**Example:**
```
Full Load (Daily):
- Load 10M customer records every day
- Time: 2 hours
- Even if only 1000 changed

Incremental Load (Daily):
- Load only 1000 changed records
- Time: 2 minutes
- 60x faster!
```

---

### Change Data Capture (CDC) Methods

#### **1. Timestamp-Based CDC** 🔥 (Most Common)

**Concept:** Use timestamp column to identify changes

**Source Table Requirement:**
```
CUSTOMERS table must have:
- LAST_MODIFIED_DATE (timestamp)
- Updated whenever row changes
```

**Implementation:**

**Mapping Variable:**
```
$$LastLoadDate
- Type: Date/Time
- Initial Value: 01/01/2020 (or earliest date)
- Persistent: YES (saves value between runs)
```

**Source Qualifier SQL Override:**
```sql
SELECT *
FROM CUSTOMERS
WHERE LAST_MODIFIED_DATE > TO_DATE('$$LastLoadDate', 'MM/DD/YYYY HH24:MI:SS')
```

**Update Variable (Expression at end of mapping):**
```
$$LastLoadDate = MAX(LAST_MODIFIED_DATE)
```

**How It Works:**
```
Run 1 (Initial Load):
- $$LastLoadDate = 01/01/2020
- Extracts all records > 01/01/2020 (all records)
- Updates $$LastLoadDate = 11/05/2025 23:59:59
- Loads to target

Run 2 (Incremental):
- $$LastLoadDate = 11/05/2025 23:59:59 (persisted)
- Extracts only records > 11/05/2025 23:59:59
- Updates $$LastLoadDate = 11/06/2025 23:59:59
- Loads only new/changed records
```

---

#### **2. Flag-Based CDC**

**Concept:** Source system sets flag for changed records

**Source Table Requirement:**
```
CUSTOMERS table:
- CHANGE_FLAG (char, values: 'N' = New, 'U' = Updated, ' ' = No change)
- Source system sets flag when record changes
```

**Implementation:**

**Source Qualifier Filter:**
```sql
SELECT *
FROM CUSTOMERS
WHERE CHANGE_FLAG IN ('N', 'U')
```

**Post-Load Process:**
```
After successful load:
Post-SQL on source:
UPDATE CUSTOMERS 
SET CHANGE_FLAG = ' '
WHERE CHANGE_FLAG IN ('N', 'U')
```

---

#### **3. Version Number CDC**

**Concept:** Version number incremented on changes

**Source Table:**
```
CUSTOMERS table:
- VERSION_NUM (integer)
- Incremented whenever record changes
```

**Implementation:**

**Mapping Variable:**
```
$$LastVersion
- Type: Integer
- Persistent: YES
```

**Source Qualifier:**
```sql
SELECT *
FROM CUSTOMERS
WHERE VERSION_NUM > $$LastVersion
```

**Update Variable:**
```
$$LastVersion = MAX(VERSION_NUM)
```

---

#### **4. Database CDC (Advanced)**

**Concept:** Database tracks changes automatically

**Technologies:**
- Oracle: Oracle Streams, GoldenGate
- SQL Server: Change Data Capture (CDC) feature
- Database maintains change tables

**Implementation:**
```
Change Table: CUSTOMERS_CT
- Contains: INSERT, UPDATE, DELETE operations
- Timestamp of change
- Before/After values

Mapping reads from change table instead of source
```

---

### Delta Detection Patterns

#### **Pattern 1: Simple Delta (New Records Only)**

**Use Case:** Append-only tables (transactions, logs)

**Logic:**
```
Source Qualifier:
SELECT * FROM TRANSACTIONS
WHERE TXN_DATE = $SYSDATE - 1

Target: Simple insert (no update needed)
```

---

#### **Pattern 2: Insert/Update Delta** 🔥

**Use Case:** Dimensions with inserts and updates

**Logic:**
```
1. Extract changed records (timestamp-based)
2. Lookup existing records in target
3. If exists → UPDATE
4. If not exists → INSERT
```

**Mapping:**
```
Source (changed records)
  ↓
Lookup Target (check if exists)
  ↓
Expression: 
  O_UpdateFlag = IIF(ISNULL(LKP_CUST_SK), 'INSERT', 'UPDATE')
  ↓
Router:
  - INSERT Group → Update Strategy (DD_INSERT)
  - UPDATE Group → Update Strategy (DD_UPDATE)
  ↓
Target (Data Driven mode)
```

---

#### **Pattern 3: Insert/Update/Delete Delta**

**Use Case:** Handle deletions from source

**Two Approaches:**

**A. Soft Delete (Logical):**
```
Expression:
O_IS_ACTIVE = IIF(SOURCE_DELETE_FLAG = 'Y', 'N', 'Y')

Update Strategy: DD_UPDATE
(Mark as inactive, don't physically delete)
```

**B. Hard Delete (Physical):**
```
Separate workflow:
1. Extract current target keys
2. Compare with source keys
3. Identify deleted keys (in target but not in source)
4. Delete from target

Update Strategy: DD_DELETE
```

---

### Incremental Load with SCD Type 2

**Complexity:** Incremental + SCD Type 2 combined

**Logic:**
```
1. Extract changed records (incremental)
2. Lookup current version in target (Dynamic Cache)
3. If changed:
   a. Expire old version (UPDATE)
   b. Insert new version (INSERT)
4. If new:
   a. Insert first version (INSERT)
```

**This is the most common real-world pattern!**

---

### Late-Arriving Dimensions

**Problem:** Fact record arrives before dimension record

**Example:**
```
Day 1:
- Sales transaction for Customer ID 9999
- Customer 9999 not yet in dimension!
- What to do?

Day 2:
- Customer 9999 arrives in dimension feed
```

**Solutions:**

**Solution 1: Default Dimension Record**
```
Create default record in dimension:
CUST_SK = -1
CUST_ID = -1
CUST_NAME = 'Unknown'

Fact table uses -1 for missing customers
Later, can update fact to correct CUST_SK (optional)
```

**Solution 2: Hold and Retry**
```
Workflow:
1. Load dimension first
2. Load facts
3. If dimension key missing:
   - Write to error/hold table
   - Retry next run after dimension updated
```

**Solution 3: Temporal Fix**
```
Insert placeholder dimension record immediately
Update with actual data when arrives
```

---

## Topic 27: Data Quality and Validation Scenarios

### Data Quality Dimensions

**Six Dimensions of Data Quality:**
1. **Accuracy:** Correct values
2. **Completeness:** No missing critical data
3. **Consistency:** Same across systems
4. **Validity:** Conforms to business rules
5. **Uniqueness:** No duplicates
6. **Timeliness:** Available when needed

---

### Common Validation Scenarios

#### **Scenario 1: Null/Missing Value Handling**

**Business Rule:** Critical fields cannot be null

**Implementation:**
```
Expression: EXP_VALIDATE
  V_IsValid = IIF(
    ISNULL(EMP_ID) OR 
    ISNULL(EMP_NAME) OR 
    ISNULL(HIRE_DATE), 0, 1)
  
  O_ERROR_DESC = IIF(V_IsValid = 0,
    'Missing critical fields: ' ||
    IIF(ISNULL(EMP_ID), 'EMP_ID ', '') ||
    IIF(ISNULL(EMP_NAME), 'EMP_NAME ', '') ||
    IIF(ISNULL(HIRE_DATE), 'HIRE_DATE', ''),
    '')

Router:
  - Valid: V_IsValid = 1 → Main Target
  - Invalid: V_IsValid = 0 → Error Target
```

---

#### **Scenario 2: Format Validation**

**Business Rules:**
- Email must contain @
- Phone must be 10 digits
- SSN must be XXX-XX-XXXX format

**Implementation:**
```
Expression: EXP_FORMAT_VALIDATE

  V_EmailValid = IIF(INSTR(EMAIL, '@') > 0, 1, 0)
  V_PhoneValid = IIF(LENGTH(REGEXP_REPLACE(PHONE, '[^0-9]', '')) = 10, 1, 0)
  V_SSNValid = IIF(LENGTH(SSN) = 11 AND 
                   SUBSTR(SSN, 4, 1) = '-' AND 
                   SUBSTR(SSN, 7, 1) = '-', 1, 0)
  
  V_IsValid = IIF(V_EmailValid = 1 AND V_PhoneValid = 1 AND V_SSNValid = 1, 1, 0)
  
  O_ERROR_DESC = IIF(V_IsValid = 0,
    IIF(V_EmailValid = 0, 'Invalid Email; ', '') ||
    IIF(V_PhoneValid = 0, 'Invalid Phone; ', '') ||
    IIF(V_SSNValid = 0, 'Invalid SSN; ', ''),
    '')
```

---

#### **Scenario 3: Range Validation**

**Business Rules:**
- Age must be 18-65
- Salary must be > 0 and < 1,000,000
- Date must be within last 100 years

**Implementation:**
```
Expression: EXP_RANGE_VALIDATE

  V_Age = TRUNC((SYSDATE - BIRTH_DATE) / 365.25)
  V_AgeValid = IIF(V_Age >= 18 AND V_Age <= 65, 1, 0)
  
  V_SalaryValid = IIF(SALARY > 0 AND SALARY < 1000000, 1, 0)
  
  V_DateValid = IIF(HIRE_DATE >= ADD_TO_DATE(SYSDATE, 'YYYY', -100) AND
                    HIRE_DATE <= SYSDATE, 1, 0)
  
  V_IsValid = IIF(V_AgeValid = 1 AND V_SalaryValid = 1 AND V_DateValid = 1, 1, 0)
```

---

#### **Scenario 4: Reference Data Validation**

**Business Rule:** Foreign keys must exist in reference tables

**Implementation:**
```
Lookup: LKP_VALID_DEPT
  - Lookup Table: DIM_DEPARTMENT
  - Condition: DEPT_ID = LKP_DEPT_ID
  - Return: LKP_DEPT_NAME

Expression:
  V_DeptValid = IIF(ISNULL(LKP_DEPT_NAME), 0, 1)
  O_ERROR_DESC = IIF(V_DeptValid = 0, 'Invalid DEPT_ID', '')

Router:
  - Valid department → Main Target
  - Invalid department → Error Target
```

---

#### **Scenario 5: Duplicate Detection**

**Business Rule:** No duplicate records based on business key

**Implementation Approach 1: Sorter + Expression**
```
Source
  ↓
Sorter: Sort by CUST_ID, START_DATE DESC
  - Distinct: No (keep all)
  ↓
Expression: Track Previous Row
  - Variable: v_PrevCustID = CUST_ID (save current for next row)
  - Variable: v_RowNum = v_RowNum + 1
  - Output: O_IsDuplicate = IIF(CUST_ID = v_PrevCustID, 1, 0)
  ↓
Router:
  - Unique: O_IsDuplicate = 0 → Main Target
  - Duplicate: O_IsDuplicate = 1 → Error Target
```

**Implementation Approach 2: Aggregator**
```
Source
  ↓
Aggregator:
  - Group By: CUST_ID
  - O_RecordCount = COUNT(*)
  ↓
Filter: O_RecordCount > 1
  ↓
  (Identifies duplicates for investigation)
```

---

### Data Cleansing Patterns

#### **Pattern 1: Standardization**

**Clean and standardize text data**

```
Expression: EXP_STANDARDIZE

  # Remove extra spaces, standardize case
  O_NAME = INITCAP(LTRIM(RTRIM(REPLACECHR(0, NAME, CHR(9), ' '))))
  
  # Standardize phone format
  O_PHONE = SUBSTR(PHONE, 1, 3) || '-' || 
            SUBSTR(PHONE, 4, 3) || '-' || 
            SUBSTR(PHONE, 7, 4)
  
  # Standardize state codes
  O_STATE = UPPER(LTRIM(RTRIM(STATE)))
```

---

#### **Pattern 2: Substitution**

**Replace invalid values with defaults**

```
Expression: EXP_SUBSTITUTE

  # Replace NULL with default
  O_COMMISSION = IIF(ISNULL(COMMISSION), 0, COMMISSION)
  
  # Replace negative salary with 0
  O_SALARY = IIF(SALARY < 0, 0, SALARY)
  
  # Replace invalid status with default
  O_STATUS = DECODE(STATUS,
    'A', 'Active',
    'I', 'Inactive',
    'T', 'Terminated',
    'Unknown')  ← Default
```

---

#### **Pattern 3: Enrichment**

**Add derived/calculated data**

```
Expression: EXP_ENRICH

  # Calculate age
  O_AGE = TRUNC((SYSDATE - BIRTH_DATE) / 365.25)
  
  # Categorize customers
  O_CUSTOMER_SEGMENT = IIF(ANNUAL_REVENUE > 100000, 'Enterprise',
                          IIF(ANNUAL_REVENUE > 50000, 'Corporate',
                            IIF(ANNUAL_REVENUE > 10000, 'SMB', 'Individual')))
  
  # Add audit columns
  O_LOAD_DATE = SYSDATE
  O_SOURCE_SYSTEM = 'CRM'
```

---

## Topic 28: Common Business Scenarios & Solutions

### Scenario 1: Daily Sales Load

**Business Requirement:**
- Load previous day's sales transactions
- Enrich with customer and product details
- Handle late-arriving dimensions

**Solution Design:**
```
Workflow: wf_Daily_Sales_Load

Step 1: Load Dimensions (if not already loaded)
  - Session: s_Load_Customers
  - Session: s_Load_Products

Step 2: Load Facts
  Source: SALES_TRANSACTIONS (filter: TXN_DATE = $SYSDATE - 1)
    ↓
  Lookup: Customer Dimension (get CUST_SK)
    ↓
  Lookup: Product Dimension (get PROD_SK)
    