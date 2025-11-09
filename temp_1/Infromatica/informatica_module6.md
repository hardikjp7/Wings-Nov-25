  Lookup: Customer Dimension (get CUST_SK)
    ↓
  Lookup: Product Dimension (get PROD_SK)
    ↓
  Expression: Handle Missing Dimensions
    O_CUST_SK = IIF(ISNULL(LKP_CUST_SK), -1, LKP_CUST_SK)  ← Default key
    O_PROD_SK = IIF(ISNULL(LKP_PROD_SK), -1, LKP_PROD_SK)
    ↓
  Router:
    - Valid: CUST_SK != -1 AND PROD_SK != -1 → FACT_SALES
    - Missing Dims: CUST_SK = -1 OR PROD_SK = -1 → ERROR_LOG
    ↓
  Target: FACT_SALES

Step 3: Notification
  - Email success/failure status
  - Include row counts

Schedule: Daily at 2:00 AM
```

---

### Scenario 2: Customer 360 View

**Business Requirement:**
- Consolidate customer data from multiple sources
- CRM, ERP, Support System, Web Analytics
- Create single customer master

**Solution Design:**
```
Source 1: CRM_CUSTOMERS (CUST_ID, NAME, EMAIL, PHONE)
Source 2: ERP_CUSTOMERS (CUST_CODE, COMPANY, ADDRESS)
Source 3: WEB_CUSTOMERS (WEB_ID, LOGIN, PREFERENCES)

Mapping: m_Customer_360

Step 1: Standardize IDs
  Expression: Map different ID formats to common CUST_ID
  
Step 2: Union All Sources
  Union Transformation (3 input groups)
  
Step 3: Deduplicate
  Sorter: Sort by CUST_ID
  Aggregator: Group by CUST_ID
    - FIRST(NAME), FIRST(EMAIL), FIRST(PHONE)
    - Logic: First non-null value wins
  
Step 4: Merge Attributes
  Expression: Combine attributes from all sources
    O_MASTER_NAME = IIF(ISNULL(CRM_NAME), ERP_NAME, CRM_NAME)
    O_MASTER_EMAIL = IIF(ISNULL(CRM_EMAIL), WEB_EMAIL, CRM_EMAIL)
  
Step 5: Data Quality Score
  Expression:
    V_Completeness = (IIF(ISNULL(NAME),0,1) + 
                      IIF(ISNULL(EMAIL),0,1) + 
                      IIF(ISNULL(PHONE),0,1)) / 3 * 100
    O_DQ_SCORE = V_Completeness
  
Step 6: Load Master
  Target: DIM_CUSTOMER_MASTER
```

---

### Scenario 3: Reconciliation Report

**Business Requirement:**
- Compare source count vs target count
- Generate reconciliation report
- Alert on mismatches

**Solution Design:**
```
Workflow: wf_Reconciliation

Step 1: Count Source Records
  Source Qualifier SQL Override:
    SELECT COUNT(*) as SOURCE_COUNT
    FROM EMPLOYEES
    WHERE LOAD_DATE = $SYSDATE - 1
  → Store in workflow variable $SourceCount

Step 2: Count Target Records
  Source Qualifier SQL Override:
    SELECT COUNT(*) as TARGET_COUNT
    FROM EMP_DWH
    WHERE LOAD_DATE = $SYSDATE - 1
  → Store in workflow variable $TargetCount

Step 3: Decision Task
  Condition: $SourceCount = $TargetCount
  
  If TRUE:
    → Email: "Reconciliation Passed"
    
  If FALSE:
    → Email: "Reconciliation FAILED"
       Subject: ALERT - Record Count Mismatch
       Body: 
         Source Count: $SourceCount
         Target Count: $TargetCount
         Difference: $SourceCount - $TargetCount
         
Step 4: Log to Reconciliation Table
  Insert into RECON_LOG:
    LOAD_DATE, SOURCE_COUNT, TARGET_COUNT, STATUS, DIFFERENCE
```

---

### Scenario 4: Slowly Growing Fact Table Optimization

**Business Requirement:**
- Fact table has 5 billion rows
- Daily load adds 50 million rows
- Queries becoming slow

**Solution Design:**
```
Optimization Strategy:

1. Table Partitioning (Database Level)
   CREATE TABLE FACT_SALES (
     SALE_DATE DATE,
     ...
   )
   PARTITION BY RANGE (SALE_DATE) (
     PARTITION P_2024 VALUES LESS THAN (TO_DATE('2025-01-01')),
     PARTITION P_2025_Q1 VALUES LESS THAN (TO_DATE('2025-04-01')),
     PARTITION P_2025_Q2 VALUES LESS THAN (TO_DATE('2025-07-01')),
     ...
   );

2. Informatica Session Optimization
   - Bulk Load Mode
   - Commit Interval: 100000
   - Partitioning: 4 partitions (Round-Robin)
   
3. Index Management
   Pre-SQL:
     ALTER INDEX idx_sale_date UNUSABLE;
   
   Post-SQL:
     ALTER INDEX idx_sale_date REBUILD ONLINE;
   
4. Incremental Aggregates
   Instead of re-aggregating 5B rows:
   - Maintain pre-aggregated summary tables
   - Update only affected aggregates daily
```

---

### Scenario 5: Multi-Source Product Dimension

**Business Requirement:**
- Product data from 3 sources with overlaps
- Need to merge with priority: Source A > Source B > Source C
- Track source of each attribute

**Solution Design:**
```
Mapping: m_Product_Master

Source A: Primary system (highest priority)
Source B: Secondary system
Source C: Legacy system

Step 1: Full Outer Join
  Joiner: Join all 3 sources on PRODUCT_CODE
  - Type: Full Outer Join
  - Ensures all products from all sources included

Step 2: Attribute Selection with Priority
  Expression: EXP_MERGE
    # Name - Priority A > B > C
    O_PRODUCT_NAME = IIF(NOT ISNULL(A.NAME), A.NAME,
                        IIF(NOT ISNULL(B.NAME), B.NAME,
                          IIF(NOT ISNULL(C.NAME), C.NAME, 'Unknown')))
    
    # Price - Priority A > B > C
    O_PRICE = IIF(NOT ISNULL(A.PRICE), A.PRICE,
                 IIF(NOT ISNULL(B.PRICE), B.PRICE,
                   IIF(NOT ISNULL(C.PRICE), C.PRICE, 0)))
    
    # Track source
    O_NAME_SOURCE = IIF(NOT ISNULL(A.NAME), 'A',
                       IIF(NOT ISNULL(B.NAME), 'B', 'C'))
    
    # Data quality indicator
    O_SOURCE_COUNT = IIF(NOT ISNULL(A.PRODUCT_CODE),1,0) +
                     IIF(NOT ISNULL(B.PRODUCT_CODE),1,0) +
                     IIF(NOT ISNULL(C.PRODUCT_CODE),1,0)
    
    O_CONFIDENCE = DECODE(O_SOURCE_COUNT,
                    3, 'High',      ← All 3 sources agree
                    2, 'Medium',    ← 2 sources
                    1, 'Low')       ← Only 1 source

Step 3: SCD Type 2 on Product Master
  - Track changes to master product attributes
  - Dynamic Lookup on existing product master
  - Type 2 logic for changed attributes
```

---

### Scenario 6: Hierarchical Data Processing

**Business Requirement:**
- Organization hierarchy (Manager-Employee)
- Need to flatten hierarchy for reporting
- Calculate levels, paths

**Solution Design:**
```
Input:
EMP_ID | EMP_NAME      | MANAGER_ID
1      | CEO           | NULL
2      | VP Sales      | 1
3      | Dir Sales     | 2
4      | Sales Rep 1   | 3
5      | Sales Rep 2   | 3

Desired Output:
EMP_ID | EMP_NAME    | LEVEL | PATH
1      | CEO         | 1     | /CEO
2      | VP Sales    | 2     | /CEO/VP Sales
3      | Dir Sales   | 3     | /CEO/VP Sales/Dir Sales
4      | Sales Rep 1 | 4     | /CEO/VP Sales/Dir Sales/Sales Rep 1

Solution:
Use recursive SQL in Source Qualifier or Stored Procedure:

WITH RECURSIVE emp_hierarchy AS (
  -- Base case: top level (no manager)
  SELECT EMP_ID, EMP_NAME, MANAGER_ID, 
         1 as LEVEL,
         '/' || EMP_NAME as PATH
  FROM EMPLOYEES
  WHERE MANAGER_ID IS NULL
  
  UNION ALL
  
  -- Recursive case: employees with managers
  SELECT e.EMP_ID, e.EMP_NAME, e.MANAGER_ID,
         h.LEVEL + 1,
         h.PATH || '/' || e.EMP_NAME
  FROM EMPLOYEES e
  JOIN emp_hierarchy h ON e.MANAGER_ID = h.EMP_ID
)
SELECT * FROM emp_hierarchy;

Informatica Implementation:
- Use SQL Transformation or Source Qualifier override
- Or use Stored Procedure transformation
```

---

### Scenario 7: Real-Time CDC Integration

**Business Requirement:**
- Near real-time data updates (< 5 min latency)
- Source: Oracle with Golden Gate CDC
- Target: Data Warehouse

**Solution Design:**
```
Architecture:
Source DB → Golden Gate → Change Table → Informatica → Target DW

Workflow: wf_Realtime_CDC (runs every 5 minutes)

Mapping: m_Process_Changes

Source: CDC_CHANGES table
  Columns: OPERATION (I/U/D), TABLE_NAME, OLD_VALUES, NEW_VALUES, 
           CHANGE_TIMESTAMP, COMMIT_SCN

Step 1: Parse Operation Type
  Expression:
    O_OperationType = OPERATION
    O_IsInsert = IIF(OPERATION = 'I', 1, 0)
    O_IsUpdate = IIF(OPERATION = 'U', 1, 0)
    O_IsDelete = IIF(OPERATION = 'D', 1, 0)

Step 2: Route by Operation
  Router:
    - INSERT_GROUP: O_IsInsert = 1
    - UPDATE_GROUP: O_IsUpdate = 1
    - DELETE_GROUP: O_IsDelete = 1

Step 3: Apply Changes
  Update Strategy:
    - INSERT_GROUP → DD_INSERT
    - UPDATE_GROUP → DD_UPDATE
    - DELETE_GROUP → DD_DELETE
  
  Target: DIM_CUSTOMER (Data Driven mode)

Step 4: Update High Water Mark
  Expression:
    $LastSCN = MAX(COMMIT_SCN)
  
  Next run extracts:
    WHERE COMMIT_SCN > $LastSCN

Monitoring:
- Track lag between source change and target update
- Alert if lag > 10 minutes
```

---

### Scenario 8: Data Archival Strategy

**Business Requirement:**
- Archive data older than 7 years
- Keep online data for recent 7 years
- Historical data to archive database

**Solution Design:**
```
Workflow: wf_Annual_Archive (runs yearly)

Step 1: Identify Archive Candidates
  Source Qualifier:
    SELECT * FROM FACT_SALES
    WHERE SALE_DATE < ADD_TO_DATE(SYSDATE, 'YYYY', -7)

Step 2: Load to Archive
  Mapping: m_Archive_Sales
    Source: FACT_SALES (old data)
    Target: ARCHIVE_DB.FACT_SALES
    
  Session Configuration:
    - Bulk Load mode
    - High commit interval (100000)
    - Disable indexes on archive table

Step 3: Verify Archive Success
  Decision Task:
    Check: $s_Archive_Sales.Status = SUCCEEDED
    
Step 4: Purge from Online
  Post-SQL on Target:
    DELETE FROM FACT_SALES
    WHERE SALE_DATE < ADD_TO_DATE(SYSDATE, 'YYYY', -7)
  
  OR separate session with Update Strategy (DD_DELETE)

Step 5: Rebuild Indexes
  Post-SQL:
    ANALYZE TABLE FACT_SALES COMPUTE STATISTICS;
    ALTER INDEX idx_sale_date REBUILD;

Step 6: Notification
  Email: Archive Summary
    - Rows Archived: $ArchiveRowCount
    - Date Range: $MinDate to $MaxDate
    - Archive Location: ARCHIVE_DB
```

---

### Scenario 9: Data Masking for Privacy

**Business Requirement:**
- Production data copied to Dev/Test
- Mask PII (email, phone, SSN, credit card)
- Maintain referential integrity

**Solution Design:**
```
Mapping: m_Mask_PII

Source: PROD_CUSTOMERS
  
Step 1: Mask Email
  Expression:
    V_EmailPrefix = SUBSTR(EMAIL, 1, INSTR(EMAIL, '@') - 1)
    V_EmailDomain = SUBSTR(EMAIL, INSTR(EMAIL, '@'))
    V_MaskedPrefix = SUBSTR(V_EmailPrefix, 1, 2) || '****'
    O_EMAIL = V_MaskedPrefix || V_EmailDomain
    
    Example: john.smith@company.com → jo****@company.com

Step 2: Mask Phone
  Expression:
    O_PHONE = 'XXX-XXX-' || SUBSTR(PHONE, -4)
    
    Example: 123-456-7890 → XXX-XXX-7890

Step 3: Mask SSN
  Expression:
    O_SSN = 'XXX-XX-' || SUBSTR(SSN, -4)
    
    Example: 123-45-6789 → XXX-XX-6789

Step 4: Mask Credit Card
  Expression:
    O_CC_NUMBER = 'XXXX-XXXX-XXXX-' || SUBSTR(CC_NUMBER, -4)
    
    Example: 1234-5678-9012-3456 → XXXX-XXXX-XXXX-3456

Step 5: Preserve Keys
  Expression:
    O_CUST_ID = CUST_ID  ← Keep original for referential integrity
    O_NAME = INITCAP(SUBSTR(CUST_ID, 1, 4) || '_' || CUST_ID)
    
    Example: CUST_1001 → Cust_1001 (consistent, not real name)

Target: DEV_CUSTOMERS
```

---

### Scenario 10: Cross-Reference Table Maintenance

**Business Requirement:**
- Multiple source systems use different IDs for same entity
- Need cross-reference mapping
- System A: CUST_A_ID, System B: CUST_B_ID, System C: CUST_C_ID

**Solution Design:**
```
Cross-Reference Table: XREF_CUSTOMER
MASTER_CUST_ID | SOURCE_SYSTEM | SOURCE_CUST_ID | EFFECTIVE_DATE | END_DATE
1              | A             | A1001          | 2024-01-01     | 9999-12-31
1              | B             | B2050          | 2024-01-01     | 9999-12-31
1              | C             | C9876          | 2024-01-01     | 9999-12-31

Mapping: m_Build_XREF

Step 1: Match Records from Multiple Sources
  Source A: Customers from System A
  Source B: Customers from System B
  
  Joiner: Match on NAME + ADDRESS (fuzzy match)
    OR use external match service

Step 2: Assign Master ID
  Sequence Generator: Generate MASTER_CUST_ID
  
Step 3: Create Cross-Reference Records
  Normalizer or Union:
    For each source, create XREF record
  
Step 4: Maintain History (SCD Type 2)
  If ID mapping changes:
    - Expire old mapping
    - Insert new mapping

Usage in Fact Load:
  Lookup XREF table to convert source ID to master ID
  
  Lookup: LKP_XREF_CUSTOMER
    Input: SOURCE_SYSTEM, SOURCE_CUST_ID
    Return: MASTER_CUST_ID
  
  Use MASTER_CUST_ID in fact table
```

---

## SUMMARY: MODULE 6 KEY CONCEPTS

### SCD (Slowly Changing Dimensions)
✅ **Type 1:** Overwrite (no history) - simplest, fastest
✅ **Type 2:** Add new version (full history) - most common, uses START_DATE, END_DATE, IS_CURRENT
✅ **Type 3:** Add previous column (limited history) - rarely used
✅ **Hybrid:** Combine Type 1 + Type 2 for different attributes
✅ **Implementation:** Dynamic Lookup + Router + Update Strategy

### Incremental Loading
✅ **Timestamp-Based:** Use LAST_MODIFIED_DATE with persistent variable
✅ **Flag-Based:** Source sets change flag
✅ **Version-Based:** Track version numbers
✅ **CDC:** Database change data capture
✅ **Delta Logic:** Extract only changed records, Insert/Update/Delete

### Data Quality
✅ **Validation:** Null checks, format checks, range checks, reference checks
✅ **Cleansing:** Standardization, substitution, enrichment
✅ **Duplicate Detection:** Sorter + Expression or Aggregator
✅ **Error Handling:** Router to separate valid/invalid, error logging

### Business Scenarios
✅ **Daily Loads:** Incremental with dimension lookup
✅ **Customer 360:** Multi-source consolidation with deduplication
✅ **Reconciliation:** Source vs target count verification
✅ **Archival:** Move old data to archive, purge from online
✅ **Data Masking:** PII protection for non-prod environments
✅ **Cross-Reference:** Maintain ID mappings across systems

---

## PRACTICE SCENARIOS FOR MODULE 6 MCQs

**Scenario 1:** Customer address changed. You need full history for shipping analysis. Which SCD type?
- **Answer:** Type 2 (maintains complete history with START_DATE, END_DATE, IS_CURRENT)

**Scenario 2:** Employee phone number updated (typo correction). History not needed. Which SCD type?
- **Answer:** Type 1 (simple overwrite, no history)

**Scenario 3:** Need to track current and previous department only. Which SCD type?
- **Answer:** Type 3 (CURRENT_DEPT, PREVIOUS_DEPT columns)

**Scenario 4:** In SCD Type 2, what does IS_CURRENT = 'Y' indicate?
- **Answer:** This is the current/active version of the record

**Scenario 5:** Your incremental load uses $LastLoadDate. Where is this value stored between sessions?
- **Answer:** Repository (because it's a persistent mapping variable)

**Scenario 6:** Source has 1M records. Only 1000 changed yesterday. Best loading strategy?
- **Answer:** Incremental load using timestamp-based CDC (extract only changed 1000 records)

**Scenario 7:** Email validation fails. Customer has email without @. What should mapping do?
- **Answer:** Router to send invalid records to error table, valid records to main target

**Scenario 8:** You need to detect duplicate CUST_ID records in source. Which transformation?
- **Answer:** Sorter (sort by CUST_ID) + Expression (compare current with previous) or Aggregator (GROUP BY CUST_ID, COUNT(*) > 1)

**Scenario 9:** Fact record arrives with CUST_ID that doesn't exist in dimension. What to do?
- **Answer:** Use default dimension key (e.g., -1 for "Unknown Customer") or write to error/hold table for retry

**Scenario 10:** SCD Type 2 mapping. Customer status changed from Gold to Platinum. What happens?
- **Answer:** Old version: END_DATE updated to today, IS_CURRENT = 'N'; New version: inserted with IS_CURRENT = 'Y', START_DATE = today

**Scenario 11:** Dynamic Lookup returns NEWLOOKUPROW = 1. What does this mean?
- **Answer:** Record not found in lookup cache (new record, needs INSERT)

**Scenario 12:** You're loading 10M daily transactions to 5B row fact table. What optimizations?
- **Answer:** Bulk load mode, high commit interval (100K), partitioning, disable indexes during load

---

## HANDS-ON EXERCISES

### Exercise 1: Implement SCD Type 1
**Objective:** Build complete Type 1 implementation
```
Requirements:
- Source: CUSTOMER_DAILY (changes feed)
- Target: DIM_CUSTOMER (dimension)
- Type 1 attributes: PHONE, EMAIL (overwrite, no history)

Steps:
1. Create mapping: m_SCD_Type1_Customer

2. Add Lookup (Dynamic Cache):
   - Table: DIM_CUSTOMER
   - Condition: CUST_ID = LKP_CUST_ID
   - Returns: CUST_SK, PHONE, EMAIL

3. Add Expression:
   V_IsNew = NEWLOOKUPROW
   V_PhoneChanged = IIF(SRC_PHONE != LKP_PHONE, 1, 0)
   V_EmailChanged = IIF(SRC_EMAIL != LKP_EMAIL, 1, 0)
   V_HasChange = IIF(V_PhoneChanged = 1 OR V_EmailChanged = 1, 1, 0)
   O_Flag = IIF(V_IsNew = 1, 'INSERT',
              IIF(V_HasChange = 1, 'UPDATE', 'SKIP'))

4. Add Router:
   - INSERT: O_Flag = 'INSERT'
   - UPDATE: O_Flag = 'UPDATE'

5. Add Update Strategy:
   - INSERT group: DD_INSERT
   - UPDATE group: DD_UPDATE

6. Configure session:
   - Target: Data Driven mode

7. Test with sample data:
   - Insert new customer
   - Update existing customer phone
   - Verify overwrite behavior
```

### Exercise 2: Implement SCD Type 2
**Objective:** Build complete Type 2 with version tracking
```
Requirements:
- Source: CUSTOMER_DAILY
- Target: DIM_CUSTOMER (with START_DATE, END_DATE, IS_CURRENT, VERSION)
- Type 2 attributes: ADDRESS, CITY, STATUS

Steps:
1. Create mapping: m_SCD_Type2_Customer

2. Add Sequence Generator:
   - Start: 1
   - For new CUST_SK generation

3. Add Lookup (Dynamic Cache):
   - Condition: CUST_ID = LKP_CUST_ID AND IS_CURRENT = 'Y'
   - Returns: CUST_SK, ADDRESS, CITY, STATUS, VERSION

4. Add Expression (Detect Change):
   V_IsNew = NEWLOOKUPROW
   V_AddressChanged = IIF(SRC_ADDRESS != LKP_ADDRESS, 1, 0)
   V_CityChanged = IIF(SRC_CITY != LKP_CITY, 1, 0)
   V_StatusChanged = IIF(SRC_STATUS != LKP_STATUS, 1, 0)
   V_HasType2Change = IIF(V_AddressChanged=1 OR V_CityChanged=1 OR V_StatusChanged=1, 1, 0)
   O_ChangeType = IIF(V_IsNew=1, 'INSERT',
                    IIF(V_HasType2Change=1, 'TYPE2', 'SKIP'))

5. Add Router (3 outputs):
   - NEW: O_ChangeType = 'INSERT'
   - TYPE2_EXPIRE: O_ChangeType = 'TYPE2'
   - TYPE2_NEW: O_ChangeType = 'TYPE2'

6. Expire Old (TYPE2_EXPIRE path):
   Expression:
     O_CUST_SK = LKP_CUST_SK
     O_END_DATE = SYSDATE
     O_IS_CURRENT = 'N'
   Update Strategy: DD_UPDATE

7. Insert New Version (TYPE2_NEW path):
   Expression:
     O_CUST_SK = NEXTVAL
     O_START_DATE = SYSDATE
     O_END_DATE = TO_DATE('9999-12-31')
     O_IS_CURRENT = 'Y'
     O_VERSION = LKP_VERSION + 1
   Update Strategy: DD_INSERT

8. Insert New Customer (NEW path):
   Expression:
     O_CUST_SK = NEXTVAL
     O_START_DATE = SYSDATE
     O_END_DATE = TO_DATE('9999-12-31')
     O_IS_CURRENT = 'Y'
     O_VERSION = 1
   Update Strategy: DD_INSERT

9. Test scenarios:
   - New customer insert
   - Existing customer address change (Type 2)
   - Verify old version expired, new version active
```

### Exercise 3: Implement Incremental Load
**Objective:** Build timestamp-based incremental load
```
Requirements:
- Source: EMPLOYEES table (has LAST_MODIFIED_DATE)
- Target: EMP_DWH
- Load only changed records since last run

Steps:
1. Create Mapping Variable:
   Name: $LastLoadDate
   Type: Date/Time
   Initial Value: 01/01/2020 00:00:00
   Is Persistent: YES

2. Create mapping: m_Incremental_Employee_Load

3. Source Qualifier SQL Override:
   SELECT *
   FROM EMPLOYEES
   WHERE LAST_MODIFIED_DATE > TO_DATE('$LastLoadDate', 'MM/DD/YYYY HH24:MI:SS')

4. Add transformations (as needed):
   - Expression for transformations
   - Lookup to check if exists
   - Update Strategy for Insert/Update logic

5. At end of mapping, update variable:
   Expression:
     $LastLoadDate = MAX(LAST_MODIFIED_DATE)

6. Test:
   Run 1: Loads all records (from 01/01/2020)
   - Note $LastLoadDate value after run
   
   Run 2: Only loads records modified since Run 1
   - Verify incremental behavior
   
   Check Repository:
   - Verify $LastLoadDate persisted between runs
```

### Exercise 4: Data Quality Validation
**Objective:** Build comprehensive validation logic
```
Requirements:
- Validate employee data
- Rules:
  * EMP_ID, EMP_NAME, HIRE_DATE cannot be null
  * EMAIL must contain @
  * SALARY must be > 0 and < 1000000
  * AGE must be 18-65
- Route valid to main target, invalid to error table

Steps:
1. Create mapping: m_Employee_Validation

2. Add Expression: EXP_VALIDATE

   # Null checks
   V_HasNulls = IIF(ISNULL(EMP_ID) OR ISNULL(EMP_NAME) OR ISNULL(HIRE_DATE), 1, 0)
   
   # Format checks
   V_EmailValid = IIF(INSTR(EMAIL, '@') > 0, 1, 0)
   
   # Range checks
   V_Age = TRUNC((SYSDATE - BIRTH_DATE) / 365.25)
   V_AgeValid = IIF(V_Age >= 18 AND V_Age <= 65, 1, 0)
   V_SalaryValid = IIF(SALARY > 0 AND SALARY < 1000000, 1, 0)
   
   # Overall validation
   V_IsValid = IIF(V_HasNulls=0 AND V_EmailValid=1 AND V_AgeValid=1 AND V_SalaryValid=1, 1, 0)
   
   # Error description
   O_ERROR_DESC = IIF(V_IsValid = 0,
     IIF(V_HasNulls=1, 'Missing required fields; ', '') ||
     IIF(V_EmailValid=0, 'Invalid email; ', '') ||
     IIF(V_AgeValid=0, 'Age out of range; ', '') ||
     IIF(V_SalaryValid=0, 'Salary out of range; ', ''),
     '')
   
   O_IS_VALID = V_IsValid

3. Add Router:
   - VALID: O_IS_VALID = 1 → EMP_TARGET
   - INVALID: O_IS_VALID = 0 → ERROR_LOG

4. Configure targets:
   - EMP_TARGET: Main employee table
   - ERROR_LOG: Include ERROR_DESC, all source columns, LOAD_DATE

5. Test with intentional errors:
   - Record with null EMP_ID
   - Record with invalid email
   - Record with age 70
   - Record with salary 2000000
   - Verify routing to ERROR_LOG with descriptions
```

### Exercise 5: Reconciliation Process
**Objective:** Build source-to-target reconciliation
```
Requirements:
- Compare source vs target row counts
- Generate reconciliation report
- Send alert if mismatch

Steps:
1. Create Workflow: wf_Reconciliation

2. Create Mapping 1: m_Count_Source
   Source Qualifier:
     SELECT 'SOURCE' as SOURCE_TYPE,
            COUNT(*) as RECORD_COUNT,
            SUM(SALARY) as TOTAL_AMOUNT,
            SYSDATE as CHECK_DATE
     FROM EMPLOYEES
     WHERE LOAD_DATE = $SYSDATE - 1
   Target: TEMP_RECON_COUNTS

3. Create Mapping 2: m_Count_Target
   Source Qualifier:
     SELECT 'TARGET' as SOURCE_TYPE,
            COUNT(*) as RECORD_COUNT,
            SUM(SALARY) as TOTAL_AMOUNT,
            SYSDATE as CHECK_DATE
     FROM EMP_DWH
     WHERE LOAD_DATE = $SYSDATE - 1
   Target: TEMP_RECON_COUNTS (append)

4. Create Mapping 3: m_Compare_Counts
   Source: TEMP_RECON_COUNTS
   
   Aggregator:
     FIRST(RECORD_COUNT) WHERE SOURCE_TYPE='SOURCE' as SRC_COUNT
     FIRST(RECORD_COUNT) WHERE SOURCE_TYPE='TARGET' as TGT_COUNT
     FIRST(TOTAL_AMOUNT) WHERE SOURCE_TYPE='SOURCE' as SRC_AMOUNT
     FIRST(TOTAL_AMOUNT) WHERE SOURCE_TYPE='TARGET' as TGT_AMOUNT
   
   Expression:
     V_CountMatch = IIF(SRC_COUNT = TGT_COUNT, 1, 0)
     V_AmountMatch = IIF(SRC_AMOUNT = TGT_AMOUNT, 1, 0)
     O_STATUS = IIF(V_CountMatch=1 AND V_AmountMatch=1, 'PASS', 'FAIL')
     O_COUNT_DIFF = SRC_COUNT - TGT_COUNT
     O_AMOUNT_DIFF = SRC_AMOUNT - TGT_AMOUNT
   
   Target: RECON_LOG

5. Add to workflow:
   Start
     ↓
   s_Count_Source
     ↓
   s_Count_Target
     ↓
   s_Compare_Counts
     ↓
   Decision: Check Status
     ├─ PASS → Email_Success
     └─ FAIL → Email_Alert
     ↓
   End

6. Test:
   - Run with matching counts (success case)
   - Manually modify target (create mismatch)
   - Run again, verify alert sent
```

---

**🎯 MODULE 6 COMPLETE!**

You've now mastered the most exam-critical scenarios:
- SCD Types 1, 2, and 3 (implementation details)
- Incremental loading strategies with CDC
- Data quality validation and cleansing patterns
- Common business scenarios and solutions

**Completion Status:**
✅ Module 1: Fundamentals (Architecture, Tools, Repository)
✅ Module 2: Core Components (Sources, Targets, Mappings)
✅ Module 3: Transformations (Expression, Filter, Router, Lookup, Aggregator, Joiner, Update Strategy)
✅ Module 4: Advanced Techniques (Mapplets, Workflows, Sessions, Parameters, Error Handling)
✅ Module 5: Performance & Optimization (Tuning, Partitioning, Pushdown, Caching)
✅ Module 6: Scenario-Based Concepts (SCD, Incremental Loads, Data Quality)

**📊 Overall Progress: 95% Complete!**

**Remaining:**
📍 **Module 7: Hands-On Preparation** (Final module!)

---

## TCS WINGS EXAM - MODULE 6 CRITICAL POINTS

### For MCQ Section:

**SCD Questions (Very High Probability):**
1. **Scenario:** "Customer address changed, need full history" → Answer: Type 2
2. **Scenario:** "Phone number typo fixed, no history needed" → Answer: Type 1
3. **Scenario:** "Need current and previous status only" → Answer: Type 3
4. **Technical:** "In Type 2, what does IS_CURRENT='Y' mean?" → Answer: Current/active version
5. **Technical:** "Type 2 uses which transformation?" → Answer: Dynamic Lookup + Router + Update Strategy
6. **Technical:** "NEWLOOKUPROW=1 means?" → Answer: New record (not found in cache)

**Incremental Load Questions:**
1. **Scenario:** "Load only yesterday's transactions" → Answer: Filter on date, or use incremental with timestamp
2. **Technical:** "Persistent mapping variable stores where?" → Answer: Repository
3. **Technical:** "Best CDC method for large tables?" → Answer: Timestamp-based with LAST_MODIFIED_DATE
4. **Performance:** "1M rows source, 1K changed. Best strategy?" → Answer: Incremental (extract only 1K)

**Data Quality Questions:**
1. **Scenario:** "Email missing @, what to do?" → Answer: Route to error table using Router
2. **Scenario:** "Detect duplicate CUST_ID" → Answer: Sorter + Expression or Aggregator with COUNT
3. **Technical:** "Validate foreign key exists" → Answer: Lookup transformation

### For Hands-On Section:

**Most Likely Hands-On Tasks:**
1. **Implement SCD Type 2** (80% probability)
   - Create mapping with Dynamic Lookup, Router, Update Strategy
   - Handle expire old + insert new version
   - Include IS_CURRENT, START_DATE, END_DATE columns

2. **Incremental Load** (70% probability)
   - Use mapping variable $LastLoadDate
   - Filter source by timestamp
   - Update variable at end

3. **Data Validation** (60% probability)
   - Expression for validation rules
   - Router to separate valid/invalid
   - Load valid to target, invalid to error table

4. **Lookup Enrichment** (50% probability)
   - Source with foreign key
   - Lookup to get dimension data
   - Handle missing lookups (default key or error)

**Common Mistakes to Avoid:**
❌ Forgetting to set Dynamic Lookup cache for SCD
❌ Not setting target to "Data Driven" mode for Update Strategy
❌ In Type 2, forgetting to expire old version before inserting new
❌ Not making mapping variable Persistent
❌ Wrong Master/Detail designation in Joiner
❌ Forgetting to validate mapping before saving

---

## FINAL PREPARATION CHECKLIST FOR MODULES 1-6

**Core Concepts You Must Know:**

**Architecture & Tools:**
☑ Domain → Repository Service → Integration Service flow
☑ Designer (4 sub-tools), Workflow Manager, Workflow Monitor
☑ Repository stores metadata, not data

**Transformations:**
☑ Active vs Passive (row count change)
☑ Connected vs Unconnected (in pipeline vs called)
☑ Expression: IIF, DECODE, string/date functions
☑ Filter vs Router (single vs multiple outputs)
☑ Aggregator: Sorted Input optimization
☑ Lookup: Dynamic, Persistent, Shared cache
☑ Joiner: Master/Detail, Sorted Input
☑ Update Strategy: DD_INSERT(0), DD_UPDATE(1), DD_DELETE(2)

**Performance:**
☑ Sorted Input reduces memory 70-90%
☑ Hash partitioning for Aggregator/Joiner (on group-by/join columns)
☑ Number of partitions = Number of CPUs
☑ Pushdown Optimization for same database
☑ Bulk load + high commit interval for large loads

**SCD (Most Important!):**
☑ Type 1: Overwrite, no history
☑ Type 2: New version, full history, IS_CURRENT/START_DATE/END_DATE
☑ Type 3: Previous column, limited history
☑ Type 2 uses: Dynamic Lookup + Router + Update Strategy
☑ NEWLOOKUPROW=1 means new record

**Incremental Load:**
☑ Timestamp-based: LAST_MODIFIED_DATE > $LastLoadDate
☑ Persistent mapping variable stores in repository
☑ Update variable: $LastLoadDate = MAX(LAST_MODIFIED_DATE)

**Session Configuration:**
☑ Commit Interval: Balance speed vs safety
☑ DTM Buffer Size: Auto or manual (64-128 MB)
☑ Target mode: Data Driven for Update Strategy
☑ Pre-SQL: Disable indexes, Post-SQL: Rebuild indexes

---

## MOCK MCQ QUIZ - Test Your Knowledge

**Question 1:** Your Aggregator groups 10M rows by DEPT_ID. Memory usage is very high. What's the BEST optimization?
A) Increase DTM Buffer Size
B) Add Sorter before Aggregator, enable Sorted Input
C) Use more partitions
D) Reduce commit interval

**Answer:** B (Sorted Input reduces Aggregator memory by 70-90%)

---

**Question 2:** Customer status changed from Gold to Platinum. Business needs full history for trend analysis. Which SCD type?
A) Type 1
B) Type 2
C) Type 3
D) No SCD needed

**Answer:** B (Type 2 maintains full history with versions)

---

**Question 3:** In SCD Type 2 mapping, Dynamic Lookup returns NEWLOOKUPROW=1. What does this indicate?
A) Record found in cache
B) Record not found in cache (new record)
C) Record updated in cache
D) Record deleted from cache

**Answer:** B (NEWLOOKUPROW=1 means record doesn't exist, needs INSERT)

---

**Question 4:** You have 4 CPU cores. How many partitions should you configure for optimal performance?
A) 2
B) 4
C) 8
D) 16

**Answer:** B (Match number of partitions to CPU cores)

---

**Question 5:** Aggregator groups by DEPT_ID. Which partition type should you use?
A) Pass-Through
B) Round-Robin
C) Hash on DEPT_ID
D) Key Range

**Answer:** C (Hash on group-by columns ensures same DEPT_ID goes to same partition)

---

**Question 6:** Both source and target are Oracle database. What optimization can dramatically improve performance?
A) Use more memory
B) Enable Pushdown Optimization
C) Add more transformations
D) Reduce commit interval

**Answer:** B (Pushdown executes transformations in database)

---

**Question 7:** Mapping variable $LastLoadDate is marked as Persistent. Where is this value stored between sessions?
A) Session cache
B) Target table
C) Repository database
D) Parameter file

**Answer:** C (Persistent variables stored in repository)

---

**Question 8:** Email validation fails (no @ symbol). What should your mapping do?
A) Stop session with error
B) Replace with default email
C) Route to error table using Router
D) Skip the record

**Answer:** C (Router separates valid/invalid, sends invalid to error table)

---

**Question 9:** Joiner has Master source (1M rows) and Detail source (50K rows). Performance is slow. What's likely the issue?
A) Need more partitions
B) Master/Detail designation is reversed
C) Need Sorted Input
D) Commit interval too low

**Answer:** B (Smaller dataset should be Detail, larger should be Master)

---

**Question 10:** In SCD Type 1, what happens to old attribute values when data changes?
A) Stored in history table
B) Moved to previous column
C) Overwritten (lost)
D) Archived with timestamp

**Answer:** C (Type 1 overwrites, no history kept)

---

## YOUR SCORE INTERPRETATION

**9-10 correct:** Excellent! You're ready for the exam
**7-8 correct:** Good! Review missed concepts
**5-6 correct:** Need more practice on key topics
**Below 5:** Review modules again, focus on transformations and SCD

---

**Ready for Module 7: Hands-On Preparation?**

Module 7 will cover:
- Step-by-step real-world mapping exercises
- Debugging techniques and common errors
- Session troubleshooting guide
- End-to-end implementation scenarios
- Exam day tips and strategies

This is the FINAL module that brings everything together with practical exercises you can implement in your Informatica environment!

Type **"Module 7"** to continue to the final module! 🎯# MODULE 6: SCENARIO-BASED CONCEPTS

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
    