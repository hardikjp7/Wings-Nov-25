# MODULE 4: ADVANCED MAPPING TECHNIQUES

## Overview

Module 4 covers reusable objects, workflow management, session configuration, and parameterization - essential for building maintainable, production-ready ETL solutions.

**Topics Covered:**
- **Topic 16:** Mapplets - Reusable Mapping Objects
- **Topic 17:** Worklets and Workflows
- **Topic 18:** Sessions and Session Configuration
- **Topic 19:** Parameter Files and Variables
- **Topic 20:** Error Handling and Recovery

---

## Topic 16: Mapplets - Reusable Mapping Objects

### What is a Mapplet?

**Mapplet:** A reusable object containing a set of transformations that can be used in multiple mappings.

**Think of it as:** A "function" or "subroutine" for your ETL logic - create once, use many times.

---

### Why Use Mapplets?

**Benefits:**

1. **Reusability:** Create once, use in multiple mappings
2. **Maintainability:** Update in one place, changes reflect everywhere
3. **Standardization:** Enforce common business logic
4. **Complexity Management:** Encapsulate complex logic
5. **Team Collaboration:** Share standard components across team

**Example Use Cases:**
- Standard address formatting logic
- Common date transformations
- Reusable data quality rules
- Standard surrogate key generation
- Common lookup patterns

---

### Mapplet Structure

**Components:**

1. **Input Transformation:** Receives data from mapping (required)
2. **Transformations:** Any transformations except certain types (see restrictions)
3. **Output Transformation:** Sends data back to mapping (required)

**Visual:**
```
Mapping → [Input] → [Transformations] → [Output] → Mapping
          └──────────── Mapplet ────────────┘
```

**Example Mapplet:**
```
MAPPLET: MPL_FORMAT_NAME

Input Transformation
  ↓
Expression: Clean and format name
  - LTRIM(RTRIM(name))
  - INITCAP(name)
  ↓
Output Transformation
```

---

### Creating a Mapplet

**Step-by-Step:**

1. **Designer → Mapplet Designer (tab)**

2. **Create New Mapplet:**
   - Mapplets → Create
   - Name: `mpl_StandardNameFormat`

3. **Add Input Transformation:**
   - Automatically created
   - Define input ports (columns coming in)

4. **Add Transformations:**
   - Drag/drop transformations
   - Configure logic
   - Example: Expression for name formatting

5. **Add Output Transformation:**
   - Automatically created
   - Define output ports (columns going out)

6. **Connect Objects:**
   - Input → Transformations → Output

7. **Validate and Save**

---

### Mapplet Example: Standard Name Formatting

**Business Requirement:** Standardize employee names across all mappings

**Mapplet: mpl_FormatName**

```
Input Transformation:
├─ I_FIRST_NAME
├─ I_LAST_NAME
└─ I_MIDDLE_NAME

Expression: EXP_FORMAT
├─ V_CleanFirst = LTRIM(RTRIM(UPPER(I_FIRST_NAME)))
├─ V_CleanLast = LTRIM(RTRIM(UPPER(I_LAST_NAME)))
├─ V_CleanMiddle = LTRIM(RTRIM(UPPER(I_MIDDLE_NAME)))
├─ O_FORMATTED_NAME = V_CleanLast || ', ' || V_CleanFirst || 
                      IIF(ISNULL(V_CleanMiddle), '', ' ' || SUBSTR(V_CleanMiddle, 1, 1) || '.')

Output Transformation:
└─ O_FORMATTED_NAME
```

**Usage in Mapping:**
```
Source → Mapplet (mpl_FormatName) → Target
         └─ Input: FIRST_NAME, LAST_NAME, MIDDLE_NAME
         └─ Output: FORMATTED_NAME
```

**Result:**
```
Input: first='john', last='smith', middle='michael'
Output: 'SMITH, JOHN M.'
```

---

### Mapplet Restrictions

**Cannot Include:**
- **Source Qualifier** (sources must be in main mapping)
- **Target definitions** (targets in main mapping)
- **Normalizer** transformation
- **COBOL sources**
- **XML sources**
- **Joiner** transformation (with some exceptions)
- **Pre/Post SQL**

**Can Include:**
- Expression, Filter, Router
- Lookup (connected/unconnected)
- Aggregator, Rank, Sorter
- Update Strategy
- Sequence Generator
- Stored Procedure, SQL transformation
- Other mapplets (nested mapplets)

---

### Mapplet Properties

**Input/Output Transformation Properties:**

**Active vs Passive:**
- If mapplet contains active transformation → Mapplet is active
- If only passive transformations → Mapplet is passive

**Important:** Once created, input/output ports cannot be changed easily (breaks existing mappings)

---

### Real-World Mapplet Scenarios

**Scenario 1: Address Standardization**
```
Mapplet: mpl_StandardizeAddress

Input:
- ADDRESS_LINE1, ADDRESS_LINE2, CITY, STATE, ZIP

Transformations:
- Expression: Clean, format, validate
- Lookup: Validate state codes
- Expression: Concatenate standardized address

Output:
- STD_ADDRESS, VALIDATED_STATE, FORMATTED_ZIP
```

**Scenario 2: Date Dimension Key Generation**
```
Mapplet: mpl_GetDateKey

Input:
- TRANSACTION_DATE

Transformations:
- Expression: Format date as YYYYMMDD
- Lookup: Get date dimension key from DATE_DIM table

Output:
- DATE_KEY (surrogate key)
```

**Scenario 3: Data Quality Checks**
```
Mapplet: mpl_DataQualityCheck

Input:
- EMP_ID, EMAIL, PHONE, SSN

Transformations:
- Expression: Validate formats
  - Email contains @
  - Phone is 10 digits
  - SSN format valid
- Expression: Data Quality Score (0-100)

Output:
- DQ_SCORE, IS_VALID, ERROR_DESC
```

---

### Mapplet Best Practices

1. **Naming Convention:** `mpl_` prefix (e.g., `mpl_FormatName`)
2. **Single Purpose:** One reusable function per mapplet
3. **Documentation:** Add clear description of inputs/outputs
4. **Version Control:** Track changes, test before deployment
5. **Performance:** Keep mapplets simple, avoid heavy transformations
6. **Testing:** Test mapplet independently before using in mappings

---

## Topic 17: Worklets and Workflows

### Understanding Workflows

**Workflow:** A set of instructions that tells Integration Service how to execute tasks.

**Think of it as:** The "execution plan" - defines WHEN and HOW mappings run.

**Components:**
- Sessions (run mappings)
- Tasks (email, command, decision)
- Links (define execution order)
- Scheduling (when to run)

---

### Workflow Manager Tool

**Purpose:** Create, configure, and manage workflows

**Key Objects:**
1. **Session:** Runs a mapping
2. **Task:** Performs specific action
3. **Worklet:** Reusable workflow component
4. **Workflow:** Container for sessions and tasks

---

### Creating a Workflow

**Step-by-Step:**

1. **Open Workflow Manager**

2. **Create Workflow:**
   - Workflows → Create
   - Name: `wf_LoadEmployees`

3. **Add Session Task:**
   - Tasks → Create → Session
   - Select mapping: `m_LoadEmployees`
   - Name session: `s_LoadEmployees`

4. **Configure Session Properties:**
   - Connections (source, target)
   - Session properties

5. **Add Other Tasks** (optional):
   - Email task, Command task, Decision task

6. **Link Tasks:**
   - Define execution order
   - Start → Session → End

7. **Validate and Save**

---

### Session Task 🔥 (Most Important)

**Session:** An instance of a mapping with runtime parameters.

**Key Configuration Areas:**

#### **1. General Properties**

**Session Name:**
- Unique identifier
- Naming convention: `s_MappingName`

**Run Mode:**
- **Normal:** Regular execution
- **Recovery:** Restart from failure point

**Enable High Precision:**
- For decimal calculations requiring high accuracy

---

#### **2. Mapping Tab**

**Source Connections:**
- Select ODBC/native connection for each source
- Flat file directory paths

**Target Connections:**
- Select connection for each target
- Target load type (normal, bulk)

**Target Load Type:**
- **Normal:** Standard insert/update
- **Bulk:** Fast load (database-specific, e.g., SQL*Loader for Oracle)

---

#### **3. Config Object Tab**

**Connection Object:**
- Database connections
- Application connections (SAP, Salesforce)

---

#### **4. Properties Tab** 🔥

**Critical Settings:**

**A. Source Properties:**
- **Source File Directory:** Location of flat files
- **Source File Type:** Direct/Indirect (file list)
- **Success File Directory:** Mark successful files

**B. Target Properties:**
- **Target File Directory:** Output file location
- **Target Load Type:** Normal/Bulk
- **Insert/Update/Delete:** Enable operations
- **Truncate Target Table:** Clear before load (Yes/No)
- **Commit Type:** Target/Source (affects transactions)
- **Commit Interval:** Number of rows before commit (default: 10000)

**C. Performance:**
- **DTM Buffer Pool Size:** Memory for data processing (default: Auto)
- **Buffer Block Size:** Size of each buffer block
- **Commit Interval:** Rows before commit (balance: smaller = safer, larger = faster)

**D. Error Handling:**
- **On Pre-Post SQL Error:** Stop/Continue
- **On Stored Procedure Error:** Stop/Continue
- **Error Threshold:** Maximum errors before session fails

**E. Partitioning:**
- **Partition Type:** Pass-through, Round-robin, Key range
- **Number of Partitions:** For parallel processing

---

### Session Configuration Examples

**Example 1: Basic Database Load**
```
Session: s_LoadEmployees

Source Connection: Oracle_HR (ODBC)
Target Connection: Oracle_DWH (ODBC)

Properties:
- Commit Type: Target
- Commit Interval: 10000
- Truncate Target: No
- Target Load Type: Normal
```

**Example 2: Flat File to Database**
```
Session: s_LoadSalesFile

Source:
- Connection: N/A (flat file)
- Source File Directory: /data/inbound/
- Source Filename: sales_$SYSDATE.csv

Target:
- Connection: SQL_Server_DW
- Truncate Target: No
- Commit Interval: 5000
```

**Example 3: Bulk Load (High Performance)**
```
Session: s_BulkLoadTransactions

Source Connection: Oracle_OLTP
Target Connection: Oracle_DWH

Properties:
- Target Load Type: Bulk
- Truncate Target: Yes
- Commit Type: Source
- DTM Buffer Size: 64000000 (64 MB)
```

---

### Workflow Tasks

#### **1. Start Task**
- Every workflow starts with Start task
- Automatically created

#### **2. Session Task**
- Runs a mapping
- Primary execution component

#### **3. Decision Task**
- Branch workflow based on conditions
- Like IF-THEN-ELSE logic

**Example:**
```
Decision: $s_LoadEmployees.Status = SUCCEEDED
  True → Email_Success
  False → Email_Failure
```

#### **4. Email Task**
- Send email notifications
- Success/failure alerts

**Configuration:**
```
Email Task: EMAIL_SUCCESS
- To: admin@company.com
- Subject: Load Succeeded
- Body: Employee load completed successfully at $SYSDATE
```

#### **5. Command Task**
- Execute shell scripts, batch files
- OS-level operations

**Examples:**
```
Command: Archive files
  Command: mv /data/inbound/*.csv /data/archive/

Command: FTP transfer
  Command: ftp -n < ftp_script.txt
```

#### **6. Assignment Task**
- Set workflow variable values
- Used with Decision tasks

**Example:**
```
Assignment: SET_ERROR_FLAG
  Variable: $$ERROR_FLAG = 1
```

#### **7. Timer Task**
- Wait for specified time/condition
- Pause workflow execution

**Example:**
```
Timer: WAIT_10_MIN
  Wait: 10 minutes
```

---

### Workflow Links

**Link Types:**

**1. Sequential (Default):**
```
Task1 → Task2 → Task3
(Execute in order)
```

**2. Conditional:**
```
Task1 → Decision
         ├─ True → Task2
         └─ False → Task3
```

**3. Parallel:**
```
       ┌→ Task2
Task1 ─┼→ Task3
       └→ Task4
(All run simultaneously)
```

**Link Conditions:**
- **On Success:** Run if previous task succeeded
- **On Failure:** Run if previous task failed
- **Unconditional:** Always run

---

### Worklet 🔥

**Worklet:** Reusable workflow component (like mapplet for workflows)

**Benefits:**
- Reuse common workflow logic
- Standardize processes
- Simplify complex workflows

**Example Worklet:**
```
Worklet: wklt_StandardErrorHandling

Components:
- Decision: Check session status
- Email_Success (if succeeded)
- Email_Failure (if failed)
- Command: Archive files

Usage: Add to any workflow for standard error handling
```

**Creating Worklet:**
1. Workflow Manager → Worklets → Create
2. Add tasks (sessions, decisions, emails)
3. Link tasks
4. Save worklet

**Using Worklet:**
1. In workflow, drag worklet from worklet designer
2. Connect to workflow tasks
3. Worklet executes as single unit

---

### Workflow Properties

**General:**
- Workflow name
- Description
- Enable recovery (Yes/No)

**Scheduler:**
- Run mode: Manual/Scheduled
- Frequency: Daily, Weekly, Monthly
- Start time
- End date

**Recovery:**
- **Enable Recovery:** Save state for restart
- **Suspend on Error:** Pause (not fail) on error
- **Recovery Strategy:** Restart task/Resume from task

---

### Real-World Workflow Scenarios

**Scenario 1: Daily Employee Load**
```
Workflow: wf_DailyEmployeeLoad

Start
  ↓
Session: s_LoadEmployees (Run mapping)
  ↓
Decision: Check Status
  ├─ Success → Email_Admin (Success notification)
  └─ Failure → Email_Admin (Failure notification)
  ↓
Command: Archive source files
  ↓
End

Schedule: Daily at 2:00 AM
```

**Scenario 2: Multi-Stage Load**
```
Workflow: wf_DataWarehouseLoad

Start
  ↓
Session: s_LoadDimensions (Load dimension tables)
  ↓
Decision: Dimensions Loaded?
  ├─ Yes → Session: s_LoadFacts (Load fact tables)
  │          ↓
  │        Decision: Facts Loaded?
  │          ├─ Yes → Session: s_BuildAggregates
  │          └─ No → Email_Failure
  └─ No → Email_Failure
  ↓
Email_Success
  ↓
End
```

**Scenario 3: Parallel Processing**
```
Workflow: wf_ParallelRegionLoad

Start
  ↓
         ┌→ Session: s_LoadNorth
         │
Start  ──┼→ Session: s_LoadSouth
         │
         ├→ Session: s_LoadEast
         │
         └→ Session: s_LoadWest
           ↓
   (All regions load simultaneously)
           ↓
      Session: s_ConsolidateAll
           ↓
         End
```

---

## Topic 18: Sessions and Session Configuration

### Session Deep Dive

Session is where "the rubber meets the road" - it's the runtime execution of your mapping with all configurations.

---

### Session Log Files 🔥 (Critical for Troubleshooting)

**Log Types:**

**1. Session Log:**
- Overall session execution details
- Start/end times
- Row counts (read, written, errors)
- Transformation statistics

**Location:** `$PMStorageDir/sesslogs/`

**Example Log:**
```
Session: s_LoadEmployees
Start Time: 2025-11-06 02:00:00
End Time: 2025-11-06 02:15:30
Status: SUCCEEDED

Source Rows: 10000
Target Rows: 9500
Rejected Rows: 500

Transformation Statistics:
- Filter: Filtered 500 rows
- Aggregator: Output 100 rows
```

**2. Workflow Log:**
- Workflow-level execution
- Task execution order
- Overall status

**3. Error Log:**
- Detailed error messages
- Failed rows
- Constraint violations

---

### Session Performance Tuning

**Key Performance Settings:**

#### **1. Commit Interval**

**What it does:** Number of rows processed before committing to target

**Trade-off:**
- **Larger (50000+):** Faster load, more rollback risk
- **Smaller (1000):** Safer, slower performance

**Recommendation:**
```
Small datasets (<100K rows): 10000
Medium datasets (100K-1M): 50000
Large datasets (>1M): 100000+
```

---

#### **2. DTM Buffer Pool Size**

**What it does:** Memory allocated for data transformation

**Setting:**
- **Auto (default):** Informatica calculates
- **Manual:** Specify in bytes

**Recommendation:**
```
Default: Auto
For large volumes: 64000000 (64 MB) to 128000000 (128 MB)
```

**Warning:** Don't set too high (can cause memory issues)

---

#### **3. Buffer Block Size**

**What it does:** Size of each data block in buffer

**Default:** 64000 bytes

**Tuning:**
- Large row size: Increase buffer block size
- Many small rows: Keep default

---

#### **4. Partitioning** 🔥

**Purpose:** Parallel processing to improve performance

**Partition Types:**

**A. Pass-Through:**
- No repartitioning
- Data flows through as-is

**B. Round-Robin:**
- Distributes rows evenly across partitions
- Good for balanced processing

**C. Hash:**
- Partition by hash value of key
- Keeps related rows together

**D. Key Range:**
- Partition by value ranges
- Example: A-M in partition 1, N-Z in partition 2

**Configuration:**
```
Session Properties → Partition Tab:
- Number of Partitions: 4
- Partition Type: Round-Robin
```

**Example:**
```
Source: 1,000,000 rows
Partitions: 4
Result: 250,000 rows per partition (processed in parallel)
Performance: 3-4x faster
```

**Note:** Requires sufficient server resources (CPU, memory)

---

### Pre-SQL and Post-SQL

**Purpose:** Execute SQL before/after session

**Common Uses:**

**Pre-SQL:**
- Truncate target table
- Disable indexes/constraints
- Archive old data
- Set session parameters

**Post-SQL:**
- Rebuild indexes
- Enable constraints
- Update statistics
- Audit logging

**Configuration:**
```
Session Properties → Mapping Tab → Target Properties:
- Pre-SQL: TRUNCATE TABLE EMP_STG
- Post-SQL: 
    ALTER INDEX IDX_EMP REBUILD;
    ANALYZE TABLE EMP_STG COMPUTE STATISTICS;
```

---

### Target Load Options

**Target Properties:**

**1. Insert:**
- Load new rows
- Default operation

**2. Update:**
- Modify existing rows
- Requires key columns defined
- Used with Update Strategy

**3. Delete:**
- Remove rows
- Requires key columns
- Used with Update Strategy

**4. Truncate:**
- Clear entire table before load
- Fast operation
- Cannot rollback

**5. Upsert (Update/Insert):**
- Update if exists, insert if new
- Database-specific feature

---

### Session Recovery 🔥

**Purpose:** Restart failed sessions from failure point (not from beginning)

**How it Works:**
1. Session fails midway
2. Informatica saves state
3. Restart session in recovery mode
4. Resumes from last commit point

**Configuration:**
```
Session Properties → General:
- Enable Recovery: Yes

Workflow Properties:
- Enable Recovery: Yes
```

**Important:**
- Only works if commit intervals set properly
- Source data must be available
- Target must support recovery

**Recovery Process:**
```
Original Session:
- Processed 500,000 rows
- Committed at 400,000
- Failed at 500,000

Recovery Session:
- Starts from 400,001
- Processes remaining rows
- Completes load
```

---

## Topic 19: Parameter Files and Variables

### Why Parameters?

**Problem:** Hard-coded values make mappings inflexible

**Solution:** Use parameters and variables for dynamic values

**Benefits:**
- Environment portability (Dev, Test, Prod)
- Dynamic file paths
- Reusable sessions
- Runtime configuration

---

### Types of Parameters

**1. Mapping Parameters:**
- Defined in mapping
- Used in transformations
- Set in parameter file

**2. Mapping Variables:**
- Defined in mapping
- Can change value during session
- Persist across sessions

**3. Session Parameters:**
- Defined in session
- Override mapping parameters
- Database connections, file paths

**4. Workflow Variables:**
- Defined in workflow
- Used across tasks
- Decision logic, conditional execution

---

### Mapping Parameters

**Definition:** Constants defined in mapping, values set at runtime

**Syntax:** `$$ParameterName` or `$ParameterName`

**Example:**

**In Mapping (Expression Transformation):**
```
Expression: EXP_FILTER_SALARY

Filter Condition:
SALARY > $$MinSalary

Mapping Parameter:
Name: $$MinSalary
Type: Integer
Default: 50000
```

**In Parameter File:**
```
[Folder1.Workflow1.Session1]
$$MinSalary=60000
```

**Result:** Filter uses 60000 instead of default 50000

---

### Mapping Variables

**Definition:** Values that can change during and persist after session

**Use Cases:**
- Track last processed date for incremental loads
- Count records
- Store maximum values

**Example: Incremental Load**

**Mapping Variable:**
```
Name: $$LastLoadDate
Type: Date/Time
Initial Value: 01/01/2020
Is Persistent: Yes
```

**In Expression:**
```
Expression: EXP_INCREMENTAL

Filter Condition:
TRANSACTION_DATE > $$LastLoadDate

Update Variable:
$$LastLoadDate = MAX(TRANSACTION_DATE)
```

**How it Works:**
```
Session 1:
- $$LastLoadDate = 01/01/2020
- Loads all data > 01/01/2020
- Updates $$LastLoadDate = 11/01/2025

Session 2:
- $$LastLoadDate = 11/01/2025 (persisted from Session 1)
- Loads only data > 11/01/2025
- Updates $$LastLoadDate = 11/06/2025
```

---

### Session Parameters

**Built-in Session Parameters:**

**Source/Target:**
- `$PMSourceFileDir` - Source file directory
- `$PMTargetFileDir` - Target file directory
- `$PMBadFileDir` - Rejected rows directory
- `$InputFile` - Source filename

**Dates:**
- `$PMSessionStartTime` - Session start timestamp
- `$PMSessionEndTime` - Session end timestamp
- `$SYSDATE` - Current date

**Example Usage:**
```
Source File Directory: /data/inbound/$MMM$DD$YYYY/
(Expands to: /data/inbound/Nov062025/)

Target Filename: Sales_$SYSDATE.txt
(Expands to: Sales_20251106.txt)
```

---

### Parameter File

**Purpose:** Store parameter values outside of repository

**Location:** Server directory (specified in session properties)

**Format:** INI file structure

**Structure:**
```
[Folder.Workflow.Session]
ParameterName=Value

[Folder.Workflow.Session.Instance]
ParameterName=Value
```

**Example Parameter File: `params.txt`**
```
[HR_Project.wf_LoadEmployees.s_LoadEmployees]
$$MinSalary=60000
$$DeptFilter='IT','HR','Finance'
$PMSourceFileDir=/data/hr/inbound/
$PMTargetFileDir=/data/hr/outbound/
$InputFile=employees_$SYSDATE.csv

[GLOBAL]
$$LoadDate=11/06/2025
$$Environment=PRODUCTION
```

**Configuring Parameter File:**
```
Session Properties → General Tab:
- Parameter Filename: $PMRootDir/params/params.txt
```

---

### Workflow Variables

**Purpose:** Share values across workflow tasks

**Syntax:** `$$WorkflowVariableName`

**Example:**

**Assignment Task:**
```
Assignment: SET_LOAD_DATE
$$LoadDate = $SYSDATE
```

**Decision Task:**
```
Decision: CHECK_SESSION_STATUS
Condition: $s_LoadEmployees.Status = SUCCEEDED AND $$LoadDate = $SYSDATE
```

**Email Task:**
```
Email: SUCCESS_NOTIFICATION
Subject: Load Completed for $$LoadDate
Body: Employee load completed successfully on $$LoadDate
```

---

### Built-in Variables

**Session Variables:**
- `$SessionName` - Current session name
- `$MappingName` - Mapping name
- `$SourceName` - Source name
- `$TargetName` - Target name

**Task Variables:**
- `$TaskName.Status` - Task completion status (SUCCEEDED/FAILED)
- `$TaskName.StartTime` - Task start time
- `$TaskName.EndTime` - Task end time
- `$TaskName.ErrorMsg` - Error message if failed

**Workflow Variables:**
- `$WorkflowName` - Current workflow name
- `$WorkflowRunId` - Unique run identifier
- `$WorkflowRunStartTime` - Workflow start time

---

### Real-World Parameter Scenarios

**Scenario 1: Environment-Specific Configuration**
```
Parameter File (DEV):
[GLOBAL]
$$SourceDB=DEV_ORACLE
$$TargetDB=DEV_SQLSERVER
$$LoadDate=11/06/2025

Parameter File (PROD):
[GLOBAL]
$$SourceDB=PROD_ORACLE
$$TargetDB=PROD_SQLSERVER
$$LoadDate=11/06/2025

Same mapping/workflow, different parameter files per environment
```

**Scenario 2: Dynamic File Processing**
```
Session: s_LoadSalesFiles
Source File Directory: /data/sales/$YYYY/$MM/
Source Filename: sales_*.csv (wildcard)

Each month automatically reads from correct directory:
- January 2025: /data/sales/2025/01/
- February 2025: /data/sales/2025/02/
```

**Scenario 3: Incremental Load with Variable**
```
Mapping Variable: $$LastExtractDate
Initial Value: 01/01/2020
Persistent: Yes

Source Qualifier Filter:
SELECT * FROM TRANSACTIONS
WHERE TXN_DATE > TO_DATE('$$LastExtractDate', 'MM/DD/YYYY')

Expression (end of mapping):
$$LastExtractDate = MAX(TXN_DATE)

Result: Each run processes only new data since last run
```

---

## Topic 20: Error Handling and Recovery

### Error Handling Strategies

**Types of Errors:**
1. **Fatal Errors:** Session stops immediately
2. **Non-Fatal Errors:** Session continues, logs errors
3. **Row-Level Errors:** Specific rows rejected

---

### Session Error Handling Configuration

**Properties → Error Handling:**

**1. Stop on Errors:**
- Number of errors before session fails
- Default: 0 (no limit)

**Example:**
```
Stop on Errors: 100
Result: Session fails if more than 100 row errors
```

**2. Override Tracing:**
- **Terse:** Minimal logging
- **Normal:** Standard logging (default)
- **Verbose Initialization:** Detailed startup info
- **Verbose Data:** Row-by-row details (slow!)

**3. Error Log:**
- **Error Log Type:** Relational/File
- **Error Log Table:** Database table for errors
- **Max Error Rows:** Limit rejected rows logged

---

### Rejected Rows Handling

**Bad Files:**
- Informatica writes rejected rows to bad files
- Location: `$PMBadFileDir`
- Format: Same as source

**Example:**
```
Source: employees.csv
Bad File: employees.bad
Contains: Rows that failed transformation/load

Common Reasons:
- Data type mismatch
- Constraint violation
- Null in NOT NULL column
- Invalid date format
```

---

### Row Error Handling in Transformations

**Expression Transformation:**
```
Error Handling Expression:
O_STATUS = IIF(ERROR(), 'ERROR', 'SUCCESS')

ERROR() function returns:
- 0: No error
- Non-zero: Error occurred
```

**Filter Transformation:**
```
Filter Condition:
NOT ERROR()

Result: Filters out error rows
```

---

### Session Recovery

**(Covered in Topic 18, recap here)**

**Enable Recovery:**
1. Session Properties → Enable Recovery: Yes
2. Workflow Properties → Enable Recovery: Yes

**Recovery Command:**
```
pmcmd startworkflow -recover -folder FolderName -workflow WorkflowName
```

**Recovery Best Practices:**
- Set appropriate commit intervals
- Enable recovery for long-running sessions
- Test recovery in non-prod environment
- Monitor disk space for recovery files

---

### Workflow Error Handling

**Decision Task:**
```
Check session status:
Decision: $s_LoadEmployees.Status = SUCCEEDED
  True → Continue workflow
  False → Send error notification, stop workflow
```

**Email Notifications:**
```
Email on Failure:
- To: dba@company.com, admin@company.com
- Subject: FAILED: $WorkflowName
- Body: 
    Workflow: $WorkflowName
    Session: $s_LoadEmployees.Name
    Error: $s_LoadEmployees.ErrorMsg
    Time: $PMSessionStartTime
```

---

### Monitoring and Alerting

**Workflow Monitor:**
- Real-time session status
- Row counts
- Error messages
- Performance metrics

**Key Metrics to Monitor:**
- **Rows Read:** Source extraction count
- **Rows Written:** Target load count
- **Rows Rejected:** Failed rows
- **Throughput:** Rows/second
- **Elapsed Time:** Total execution time

**Setting Alerts:**
```
Alert Condition:
- If session fails
- If rows rejected > threshold
- If execution time > expected

Action:
- Send email
- Write to log
- Execute command
```

---

### Troubleshooting Common Errors 🔥

**Error 1: Session Initialization Failed**
```
Cause: Connection issues, repository unavailable
Solution:
- Check database connectivity
- Verify repository service running
- Check network connectivity
```

**Error 2: ORA-01722: Invalid Number**
```
Cause: Converting non-numeric string to number
Solution:
- Add TO_INTEGER validation in Expression
- Use ERROR() function to catch conversion errors
- Filter invalid data before conversion
```

**Error 3: Out of Memory**
```
Cause: Insufficient memory for Aggregator/Joiner
Solution:
- Enable Sorted Input
- Increase DTM buffer size
- Use Source Qualifier join instead of Joiner
- Partition the session
```

**Error 4: Unique Constraint Violation**
```
Cause: Duplicate key in target table
Solution:
- Add Filter to remove duplicates
- Use Update Strategy instead of insert-only
- Add Sorter with Distinct option
- Check source data for duplicates
```

**Error 5: Lock Timeout**
```
Cause: Target table locked by another process
Solution:
- Schedule workflow when table is free
- Increase timeout in database
- Use staging table instead of direct load
```

---

### Best Practices for Error Handling

**1. Layered Error Handling:**
```
Level 1: Transformation Level
- Validate in Expression
- Filter invalid rows early

Level 2: Session Level  
- Configure error thresholds
- Enable recovery
- Set appropriate commit intervals

Level 3: Workflow Level
- Decision tasks check status
- Email notifications
- Retry logic

Level 4: Monitoring
- Workflow Monitor alerts
- Log analysis
- Automated health checks
```

**2. Proactive Data Quality:**
```
Source → Data Quality Checks → Filter Invalid → Process Valid
           (Expression)           (Router)        (Main Flow)
                                     ↓
                                Error_Target
```

**3. Graceful Degradation:**
```
Instead of failing entire load:
- Route bad records to error table
- Continue processing good records
- Investigate errors post-load
```

---

## SUMMARY: MODULE 4 KEY CONCEPTS

### Mapplets
✅ Reusable mapping components (like functions)
✅ Contains transformations between Input and Output
✅ Cannot contain Source/Target definitions
✅ Use for standardization and maintainability

### Workflows and Sessions
✅ Workflow = execution plan (WHEN to run)
✅ Session = runtime instance of mapping (HOW to run)
✅ Tasks: Session, Decision, Email, Command, Timer
✅ Links define execution order and conditions

### Session Configuration
✅ Connections: Source/Target database connections
✅ Commit Interval: Balance between speed and safety
✅ DTM Buffer Size: Memory for data processing
✅ Partitioning: Parallel processing for performance
✅ Pre/Post SQL: Execute SQL before/after load

### Parameters and Variables
✅ Mapping Parameters: Constants, set in parameter file
✅ Mapping Variables: Change during session, persist
✅ Session Parameters: $PMSourceFileDir, $SYSDATE
✅ Workflow Variables: Share values across tasks
✅ Parameter File: INI format, environment-specific config

### Error Handling
✅ Bad files: Rejected rows written to .bad files
✅ Error threshold: Stop session after N errors
✅ Recovery: Restart from failure point
✅ Monitoring: Workflow Monitor for real-time status
✅ Alerting: Email notifications on failure

---

## PRACTICE SCENARIOS FOR MODULE 4 MCQs

**Scenario 1:** You have name formatting logic used in 10 mappings. When requirements change, you update all 10 mappings. What's the better approach?
- **Answer:** Create a Mapplet for name formatting, use it in all 10 mappings. Update once, changes reflect everywhere.

**Scenario 2:** Your workflow must load dimensions before facts. How do you ensure this?
- **Answer:** Link tasks sequentially: s_LoadDimensions → (on success) → s_LoadFacts

**Scenario 3:** Session loads 5 million rows and fails at 4.5M due to network issue. You don't want to reload all 5M. What should you configure?
- **Answer:** Enable Recovery in session and workflow. Set appropriate Commit Interval (e.g., 100000). Restart in recovery mode.

**Scenario 4:** You need different source directories for Dev, Test, and Prod environments without changing the session. How?
- **Answer:** Use parameter $PMSourceFileDir in session, define different values in environment-specific parameter files.

**Scenario 5:** Your daily load should process only records added since last run. How do you track the last processed date?
- **Answer:** Use Mapping Variable $LastLoadDate (Persistent=Yes). Filter: TXN_DATE > $LastLoadDate. Update variable with MAX(TXN_DATE).

**Scenario 6:** Session writes 100K rows but only 95K appear in target. Where are the missing 5K rows?
- **Answer:** Check bad file ($PMBadFileDir) for rejected rows. Review session log for error details.

**Scenario 7:** Your Aggregator transformation runs out of memory with 10M rows. What's the best optimization?
- **Answer:** Add Sorter before Aggregator (sort by group-by columns), enable Sorted Input in Aggregator properties.

**Scenario 8:** Workflow should send different emails based on session success/failure. How?
- **Answer:** Use Decision task: Check $s_SessionName.Status = SUCCEEDED → Email_Success; else → Email_Failure

**Scenario 9:** Your target table must be truncated before load, and indexes rebuilt after load. Where do you configure this?
- **Answer:** Session → Target Properties → Pre-SQL: TRUNCATE TABLE; Post-SQL: REBUILD INDEX

**Scenario 10:** You want to pass today's date to a Filter condition without hardcoding. What do you use?
- **Answer:** Use $SYSDATE parameter or mapping parameter $LoadDate set dynamically in parameter file

---

## HANDS-ON EXERCISES

### Exercise 1: Create a Mapplet
**Objective:** Build reusable address formatting logic
```
Steps:
1. Create Mapplet: mpl_FormatAddress
2. Input Transformation:
   - I_ADDRESS_LINE1
   - I_ADDRESS_LINE2
   - I_CITY
   - I_STATE
   - I_ZIP
3. Add Expression:
   - V_CleanAddr1 = LTRIM(RTRIM(UPPER(I_ADDRESS_LINE1)))
   - V_CleanCity = INITCAP(LTRIM(RTRIM(I_CITY)))
   - O_FORMATTED_ADDRESS = V_CleanAddr1 || ', ' || V_CleanCity || ', ' || I_STATE || ' ' || I_ZIP
4. Output Transformation:
   - O_FORMATTED_ADDRESS
5. Use in mapping: Source → mpl_FormatAddress → Target
```

### Exercise 2: Create a Complete Workflow
**Objective:** Build end-to-end workflow with error handling
```
Steps:
1. Create Workflow: wf_EmployeeLoad
2. Add Session: s_LoadEmployees (uses m_LoadEmployees mapping)
3. Configure Session:
   - Source Connection: Oracle_HR
   - Target Connection: Oracle_DWH
   - Commit Interval: 10000
4. Add Decision Task: Check session status
   - Condition: $s_LoadEmployees.Status = SUCCEEDED
5. Add Email Tasks:
   - Email_Success (if succeeded)
   - Email_Failure (if failed)
6. Link tasks:
   Start → s_LoadEmployees → Decision → Email_Success/Email_Failure → End
7. Test workflow
```

### Exercise 3: Configure Parameters
**Objective:** Use parameters for flexible configuration
```
Steps:
1. In Mapping: m_LoadEmployees
   - Add Mapping Parameter: $MinSalary (Integer, default 50000)
   - In Filter: SALARY > $MinSalary

2. Create Parameter File: params.txt
   [HR_Folder.wf_EmployeeLoad.s_LoadEmployees]
   $MinSalary=60000
   $PMSourceFileDir=/data/hr/inbound/
   $PMTargetFileDir=/data/hr/outbound/

3. Configure Session:
   - General Tab → Parameter Filename: /params/params.txt

4. Run session with different parameter values
```

### Exercise 4: Implement Incremental Load
**Objective:** Load only new/changed records
```
Steps:
1. Create Mapping Variable: $LastLoadDate
   - Type: Date/Time
   - Initial Value: 01/01/2020
   - Persistent: Yes

2. In Source Qualifier:
   - SQL Override: 
     SELECT * FROM EMPLOYEES 
     WHERE LAST_MODIFIED_DATE > TO_DATE('$LastLoadDate', 'MM/DD/YYYY')

3. In Expression (end of mapping):
   - Variable: $LastLoadDate = MAX(LAST_MODIFIED_DATE)

4. Run session multiple times, verify incremental behavior
```

### Exercise 5: Error Handling Setup
**Objective:** Configure comprehensive error handling
```
Steps:
1. Session Configuration:
   - Properties → Stop on Errors: 100
   - Error Log Type: File
   - Bad File Directory: /data/errors/

2. In Mapping:
   - Add Expression after transformations:
     O_HAS_ERROR = ERROR()
   - Add Router:
     Group VALID: ERROR() = 0 → Main_Target
     Group INVALID: ERROR() > 0 → Error_Target

3. Workflow:
   - Add Decision: Check session status
   - Add Email on failure with error details

4. Test with intentional errors (e.g., invalid data types)
5. Review bad files and error logs
```

---

## TCS WINGS EXAM - MODULE 4 FOCUS AREAS

**High Priority Topics:**

1. **Mapplet Usage:**
   - When to use mapplets vs regular transformations
   - Mapplet restrictions (no sources/targets)
   - Benefits of reusability

2. **Workflow Components:**
   - Session vs Workflow distinction
   - Task types (Session, Decision, Email, Command)
   - Link conditions and execution order

3. **Session Performance:**
   - Commit Interval impact
   - DTM Buffer Size tuning
   - Partitioning benefits
   - Sorted Input optimization

4. **Parameters:**
   - Difference between Parameters and Variables
   - When to use mapping parameters vs session parameters
   - Parameter file structure
   - Built-in parameters ($SYSDATE, $PMSourceFileDir)

5. **Error Handling:**
   - Bad file purpose and location
   - Recovery mechanism
   - Error thresholds
   - Session log interpretation

**Scenario-Based Questions to Expect:**

1. **Performance Issues:**
   - "Session running slow with 10M rows in Aggregator"
   - Answer: Sorted Input optimization

2. **Error Recovery:**
   - "Session failed at 4M out of 5M rows, how to resume?"
   - Answer: Enable recovery, restart in recovery mode

3. **Environment Migration:**
   - "Move from Dev to Prod without changing session"
   - Answer: Use parameter files for environment-specific values

4. **Incremental Loading:**
   - "Load only today's transactions"
   - Answer: Use persistent mapping variable to track last load date

5. **Reusability:**
   - "Same logic in 15 mappings, difficult to maintain"
   - Answer: Create mapplet with common logic

---

**🎯 MODULE 4 COMPLETE!**

You now understand:
- Reusable objects (Mapplets, Worklets)
- Workflow orchestration and session configuration
- Performance tuning techniques
- Parameterization for flexibility
- Comprehensive error handling

**Ready for Module 5: Performance & Optimization?**

This module will deep-dive into:
- Advanced performance tuning strategies
- Partitioning techniques in detail
- Pushdown optimization
- Caching strategies
- Bottleneck identification and resolution

Type **"Module 5"** to continue!