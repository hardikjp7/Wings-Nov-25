# MODULE 5: PERFORMANCE & OPTIMIZATION

## Overview

Performance optimization is critical for production ETL systems. This module covers techniques to identify bottlenecks and improve session execution speed.

**Topics Covered:**
- **Topic 21:** Performance Tuning Basics
- **Topic 22:** Partitioning Techniques
- **Topic 23:** Pushdown Optimization
- **Topic 24:** Caching Strategies

---

## Topic 21: Performance Tuning Basics

### Understanding Performance Metrics

**Key Performance Indicators (KPIs):**

1. **Throughput:** Rows processed per second
2. **Elapsed Time:** Total session execution time
3. **Memory Usage:** RAM consumed during processing
4. **CPU Utilization:** Processor usage
5. **I/O Operations:** Disk read/write operations

**Target Performance:**
```
Good Performance: 10,000+ rows/second
Average Performance: 5,000-10,000 rows/second
Poor Performance: <5,000 rows/second

Note: Varies based on transformation complexity and row size
```

---

### Performance Tuning Methodology

**Step-by-Step Approach:**

**Step 1: Identify the Bottleneck**
- Use Workflow Monitor
- Analyze session logs
- Check performance details

**Step 2: Determine Root Cause**
- Source extraction slow?
- Transformation processing slow?
- Target loading slow?
- Network latency?

**Step 3: Apply Optimization**
- Specific techniques based on bottleneck
- Test and measure improvement

**Step 4: Iterate**
- Continue optimizing until acceptable performance

---

### Using Workflow Monitor for Performance Analysis 🔥

**Accessing Performance Details:**

1. **Open Workflow Monitor**
2. **Select completed/running session**
3. **Right-click → Get Session Performance Details**

**What You'll See:**

```
Session Performance Details:

Source Statistics:
├─ Source: EMPLOYEES
├─ Rows Read: 1,000,000
├─ Read Throughput: 8,500 rows/sec
└─ Time: 117 seconds

Transformation Statistics:
├─ Filter: FLT_ACTIVE
│   ├─ Input Rows: 1,000,000
│   ├─ Output Rows: 850,000
│   └─ Time: 12 seconds
│
├─ Aggregator: AGG_DEPT
│   ├─ Input Rows: 850,000
│   ├─ Output Rows: 50
│   └─ Time: 425 seconds ⚠️ BOTTLENECK!
│
└─ Expression: EXP_CALC
    ├─ Input Rows: 50
    ├─ Output Rows: 50
    └─ Time: 0.5 seconds

Target Statistics:
├─ Target: DEPT_SUMMARY
├─ Rows Written: 50
├─ Write Throughput: 1,000 rows/sec
└─ Time: 0.05 seconds

Total Session Time: 554 seconds
```

**Analysis:**
- Aggregator taking 425 out of 554 seconds (77%)
- This is the bottleneck!
- Solution: Enable Sorted Input for Aggregator

---

### Common Performance Bottlenecks

#### **1. Source Bottleneck**

**Symptoms:**
- Long source read time
- Low throughput at source
- Network latency

**Solutions:**

**A. Source Qualifier Filter:**
```
Instead of reading 10M rows and filtering in Informatica:
Source Qualifier → SQL Override:
  SELECT * FROM EMPLOYEES 
  WHERE HIRE_DATE >= '2024-01-01'
  AND STATUS = 'A'

Result: Read only 1M rows from source
```

**B. Source Qualifier Join:**
```
Instead of reading 2 tables separately and joining in Informatica:
Source Qualifier → SQL Override:
  SELECT e.*, d.DEPT_NAME 
  FROM EMPLOYEES e
  JOIN DEPARTMENTS d ON e.DEPT_ID = d.DEPT_ID

Result: Database performs join (faster)
```

**C. Index Source Tables:**
```
Ensure indexes on:
- Join columns
- Filter columns
- Sort columns

CREATE INDEX idx_emp_hire_date ON EMPLOYEES(HIRE_DATE);
```

**D. Parallel Source Extraction:**
```
Use source-based partitioning:
- Partition by key range
- Multiple queries extract simultaneously
```

---

#### **2. Transformation Bottleneck**

**Symptoms:**
- Specific transformation takes long time
- High memory usage
- Slow throughput through transformation

**Solutions by Transformation:**

**A. Aggregator:**
```
Problem: Loading entire dataset in memory

Solution 1: Sorted Input
- Add Sorter before Aggregator (sort by group-by columns)
- Enable Sorted Input in Aggregator
- Memory reduction: 70-90%

Solution 2: Aggregator Cache Size
- Increase cache directory space
- Set cache size appropriately

Solution 3: Push to Database
- Use Source Qualifier for aggregation when possible
```

**B. Joiner:**
```
Problem: Loading detail source in memory

Solution 1: Designate Master/Detail Correctly
- Smaller dataset = Detail
- Larger dataset = Master

Solution 2: Sorted Input
- Sort both sources by join columns
- Enable Sorted Input in Joiner

Solution 3: Use Source Qualifier Join
- If both sources in same database
- Database join is faster
```

**C. Lookup:**
```
Problem: Large lookup table, slow cache build

Solution 1: Filter Lookup Table
- SQL Override to reduce rows
- SELECT only needed columns

Solution 2: Index Lookup Columns
- Index on lookup condition columns

Solution 3: Persistent Cache
- Build once, reuse in subsequent runs

Solution 4: Source Qualifier Join
- Replace lookup with SQ join if possible
```

**D. Expression:**
```
Problem: Complex calculations slow

Solution 1: Simplify Expressions
- Break complex logic into multiple transformations
- Use variable ports efficiently

Solution 2: Move to Database
- Use SQL transformation for complex logic
- Database may execute faster

Solution 3: Avoid Unnecessary Conversions
- Minimize TO_CHAR, TO_DATE conversions
```

**E. Filter:**
```
Problem: Filter after expensive transformations

Solution: Filter Early
- Move Filter close to source
- Remove unwanted rows before expensive operations

Bad:
Source → Aggregator → Lookup → Filter → Target

Good:
Source → Filter → Aggregator → Lookup → Target
```

---

#### **3. Target Bottleneck**

**Symptoms:**
- Slow write operations
- Long commit times
- Target database slow

**Solutions:**

**A. Bulk Load Mode:**
```
Session Properties → Target Properties:
- Target Load Type: Bulk

Benefits:
- Uses database-specific bulk loaders
- Oracle: SQL*Loader
- SQL Server: Bulk Insert
- 5-10x faster than normal mode

Limitations:
- No constraints checked during load
- Limited error handling
- Indexes/triggers disabled
```

**B. Disable Indexes/Constraints:**
```
Pre-SQL:
  ALTER INDEX idx_emp UNUSABLE;
  ALTER TABLE EMPLOYEES DISABLE CONSTRAINT fk_dept;

Post-SQL:
  ALTER INDEX idx_emp REBUILD;
  ALTER TABLE EMPLOYEES ENABLE CONSTRAINT fk_dept;

Benefit: Faster loads, rebuild after
```

**C. Increase Commit Interval:**
```
Session Properties:
- Commit Interval: 50000 (instead of 10000)

Trade-off:
- Faster: Fewer commits, less overhead
- Risk: More data lost on failure
```

**D. External Loading:**
```
Instead of Informatica loading:
1. Informatica writes to flat file
2. Database bulk loader loads file
3. Much faster for very large volumes
```

**E. Partition Target:**
```
Database Table Partitioning:
- Range partitioning (by date, ID range)
- Hash partitioning
- Parallel inserts to different partitions
```

---

#### **4. Network Bottleneck**

**Symptoms:**
- High latency between Informatica server and databases
- Slow data transfer
- Network timeouts

**Solutions:**

**A. Pushdown Optimization:**
```
Execute transformations in database instead of Informatica
(Covered in Topic 23)
```

**B. Local Staging:**
```
Instead of:
Remote DB → Informatica → Remote DB

Do:
Remote DB → Local staging → Informatica → Local target → Remote DB
```

**C. Compression:**
```
Enable network compression in database connections
```

---

### Session-Level Performance Settings 🔥

#### **1. DTM (Data Transformation Manager) Buffer Pool**

**What It Is:** Memory allocated for data processing

**Configuration:**
```
Session Properties → Config Object:
- $PMBufferMemPoolSize: Auto (default) or Manual value

Recommendations:
- Small sessions: Auto
- Large sessions: 64000000 (64 MB) to 128000000 (128 MB)
- Very large: 256000000 (256 MB)
```

**Formula:**
```
Buffer Pool Size = Buffer Block Size × Number of Buffers

Example:
64KB × 1000 buffers = 64 MB
```

**Warning:** Setting too high can cause out-of-memory errors!

---

#### **2. Buffer Block Size**

**What It Is:** Size of each data block in buffer

**Configuration:**
```
Session Properties:
- $PMDefaultBufferBlockSize: 64000 (default, 64KB)

Adjust based on row size:
- Small rows (<100 bytes): 64000
- Medium rows (100-500 bytes): 128000
- Large rows (>500 bytes): 256000
```

**Formula:**
```
Rows per Block = Buffer Block Size / Avg Row Size

Example:
64KB block / 200 bytes row = 320 rows per block
```

---

#### **3. Commit Interval**

**What It Is:** Number of rows before committing transaction

**Configuration:**
```
Session Properties → Target Properties:
- Commit Interval: 10000 (default)

Tuning:
- High volume, high reliability: 5000-10000
- High volume, acceptable risk: 50000-100000
- Very large batch loads: 100000+
```

**Trade-offs:**
```
Small Commit Interval (1000):
✓ Less data lost on failure
✓ Better for concurrent users
✗ Slower (more commits)
✗ More redo log overhead

Large Commit Interval (100000):
✓ Faster (fewer commits)
✓ Less overhead
✗ More data lost on failure
✗ Longer rollback time
```

---

#### **4. Commit Type**

**Options:**

**A. Target-Based Commit (Default):**
```
Commits after specified interval of rows written to target
Best for: Most scenarios
```

**B. Source-Based Commit:**
```
Commits after specified interval of rows read from source
Best for: Recovery scenarios, maintaining consistency
```

**C. User-Defined Commit:**
```
Use Transaction Control Transformation to define commit points
Best for: Complex business logic requiring specific commit points
```

---

### Mapping-Level Optimization

#### **1. Reduce Transformation Count**

**Problem:** Too many transformations slow processing

**Solution:**
```
Bad Design:
Source → Expression1 → Expression2 → Expression3 → Filter → Target
(Multiple passes through data)

Good Design:
Source → Expression (combine all logic) → Filter → Target
(Single pass)
```

**Combining Transformations:**
```
Instead of 3 separate Expressions:
EXP1: Calculate field A
EXP2: Calculate field B
EXP3: Calculate field C

Use 1 Expression:
Calculate A, B, C together
```

---

#### **2. Filter Early**

**Principle:** Remove unwanted rows as early as possible

**Example:**
```
Bad:
Source (1M rows) → Lookup → Aggregator → Filter (keep 100K) → Target

Good:
Source (1M rows) → Filter (keep 100K) → Lookup → Aggregator → Target

Benefit: Lookup and Aggregator process only 100K instead of 1M
```

---

#### **3. Avoid Unnecessary Lookups**

**Problem:** Every lookup adds overhead

**Solution:**
```
If source and lookup in same database:
Replace Lookup with Source Qualifier join

If lookup table small and static:
Consider hardcoding values (DECODE) for very small lists
```

---

#### **4. Use Mapplets for Complex Logic**

**Benefits:**
- Pre-tested, optimized logic
- Consistent performance across mappings
- Easier maintenance

---

## Topic 22: Partitioning Techniques 🔥

### What is Partitioning?

**Partitioning:** Dividing data into multiple streams for parallel processing

**Concept:**
```
Without Partitioning:
Source (1M rows) → [Single Thread] → Target
Time: 100 seconds

With Partitioning (4 partitions):
Source (1M rows) → [Thread 1: 250K] → Target
                 → [Thread 2: 250K] → Target
                 → [Thread 3: 250K] → Target
                 → [Thread 4: 250K] → Target
Time: 30 seconds (3.3x faster)
```

**Benefits:**
- Parallel processing
- Better resource utilization (multiple CPUs)
- Significantly faster for large datasets

---

### Partition Types

#### **1. Pass-Through Partitioning**

**How It Works:** Data passes through without repartitioning

**Use Case:** Maintain existing partitions from previous transformation

**Example:**
```
Source → Sorter (4 partitions) → Aggregator (Pass-through) → Target
         └─ Creates 4 partitions    └─ Keeps 4 partitions
```

**Configuration:** Automatically applied, no special config needed

---

#### **2. Round-Robin Partitioning**

**How It Works:** Distributes rows evenly across partitions in circular fashion

**Distribution:**
```
Rows: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
4 Partitions:

Partition 1: 1, 5, 9
Partition 2: 2, 6, 10
Partition 3: 3, 7
Partition 4: 4, 8
```

**Use Case:**
- Balanced data distribution
- No grouping requirements
- General parallel processing

**Configuration:**
```
Transformation Properties → Partition Tab:
- Partition Type: Round-Robin
- Number of Partitions: 4
```

---

#### **3. Hash Partitioning**

**How It Works:** Partitions based on hash value of specified column(s)

**Distribution:**
```
Hash on DEPT_ID:
Rows with DEPT_ID 10 → Partition 1
Rows with DEPT_ID 20 → Partition 2
Rows with DEPT_ID 30 → Partition 3
Rows with DEPT_ID 40 → Partition 4

Key: Same DEPT_ID always goes to same partition
```

**Use Case:**
- Aggregator (group by DEPT_ID)
- Joiner (join on DEPT_ID)
- Rank (rank within DEPT_ID)
- Ensures related rows in same partition

**Configuration:**
```
Transformation Properties → Partition Tab:
- Partition Type: Hash
- Number of Partitions: 4
- Hash Keys: DEPT_ID
```

**Critical Rule:** Hash on the same columns you're grouping/joining by!

---

#### **4. Key Range Partitioning**

**How It Works:** Partitions based on column value ranges

**Distribution:**
```
Key Range on EMP_ID (1M rows, 4 partitions):

Partition 1: EMP_ID 1 - 250000
Partition 2: EMP_ID 250001 - 500000
Partition 3: EMP_ID 500001 - 750000
Partition 4: EMP_ID 750001 - 1000000
```

**Use Case:**
- Evenly distributed numeric keys
- Known data distribution
- Specific range-based processing

**Configuration:**
```
Transformation Properties → Partition Tab:
- Partition Type: Key Range
- Number of Partitions: 4
- Partition Key: EMP_ID
- Ranges: Define manually or let Informatica calculate
```

---

### Partition Points 🔥

**Partition Point:** Transformation where partitioning is created or changed

**Types:**

**1. Partition Point (Creates Partitions):**
```
Source → Filter (Partition Point: 4 partitions) → Target
         └─ Data divided into 4 streams here
```

**2. Pass-Through (Maintains Partitions):**
```
Source → Filter (4 partitions) → Expression (Pass-through) → Target
                                  └─ Keeps 4 partitions
```

**3. Merge Point (Combines Partitions):**
```
Source → Filter (4 partitions) → Aggregator → Target
                                  └─ Merges to 1 stream
```

---

### Optimal Partitioning Strategies

#### **Strategy 1: Source-Based Partitioning**

**Concept:** Partition at source extraction

**Example:**
```
Source Qualifier Properties:
- SQL Override with UNION ALL

Partition 1:
SELECT * FROM EMPLOYEES WHERE DEPT_ID BETWEEN 1 AND 25

Partition 2:
SELECT * FROM EMPLOYEES WHERE DEPT_ID BETWEEN 26 AND 50

Partition 3:
SELECT * FROM EMPLOYEES WHERE DEPT_ID BETWEEN 51 AND 75

Partition 4:
SELECT * FROM EMPLOYEES WHERE DEPT_ID BETWEEN 76 AND 100

Result: 4 parallel extractions from source
```

**Benefits:**
- Fastest extraction
- Distributes database load
- Parallel from start to finish

---

#### **Strategy 2: Pipeline Partitioning**

**Concept:** Partition in transformation pipeline

**Example:**
```
Source → Filter (Partition Point: Round-Robin, 4) → Expression → Target
         └─ Partitioning starts here
```

**Best for:** When source partitioning not possible

---

#### **Strategy 3: Aggregator/Joiner Partitioning**

**Critical Rule:** Must use Hash partitioning on group-by/join columns!

**Example: Aggregator**
```
Source → Sorter → Aggregator (Hash on DEPT_ID, 4 partitions) → Target
                  └─ Group By: DEPT_ID

Why Hash?
- All rows with DEPT_ID=10 go to same partition
- Aggregation performed independently in each partition
- Correct results guaranteed
```

**Example: Joiner**
```
Master → Joiner (Hash on EMP_ID, 4 partitions) → Target
Detail ──┘
         └─ Join Condition: EMP_ID

Why Hash?
- Matching EMP_IDs from both sources go to same partition
- Join performed independently in each partition
```

---

### Partitioning Best Practices

**1. Number of Partitions:**
```
Rule of Thumb:
Number of Partitions = Number of CPUs

Example:
4 CPU cores → 4 partitions
8 CPU cores → 8 partitions

Don't exceed CPU count (overhead increases)
```

**2. Data Distribution:**
```
Ensure balanced distribution:
- Round-Robin: Automatic balance
- Hash: Good balance if data distributed evenly
- Key Range: Manual tuning may be needed

Check partition statistics in session log
```

**3. Memory Considerations:**
```
Each partition uses memory:
Total Memory = Memory per Partition × Number of Partitions

Example:
Aggregator cache: 100 MB per partition
4 partitions = 400 MB total

Ensure sufficient memory available!
```

**4. Transformation Support:**
```
Not all transformations support partitioning:

Supported:
✓ Filter, Router, Expression
✓ Aggregator, Sorter, Rank
✓ Joiner, Lookup

Not Supported:
✗ Sequence Generator (must be before partition point)
✗ Normalizer
✗ Some update strategies
```

---

### Partition Troubleshooting

**Problem 1: Unbalanced Partitions**
```
Partition 1: 500K rows (100 sec)
Partition 2: 100K rows (20 sec)
Partition 3: 50K rows (10 sec)
Partition 4: 350K rows (70 sec)

Session waits for slowest (Partition 1: 100 sec)

Solution:
- Use Round-Robin for better balance
- Adjust key ranges if using Key Range
- Check data skew in hash keys
```

**Problem 2: Out of Memory with Partitions**
```
4 partitions × Aggregator cache = Exceeds memory

Solution:
- Reduce number of partitions
- Enable Sorted Input (reduces cache)
- Increase server memory
```

**Problem 3: No Performance Gain**
```
Cause: Bottleneck elsewhere (source, target, network)

Solution:
- Identify actual bottleneck (use Workflow Monitor)
- Partition entire pipeline, not just one transformation
```

---

## Topic 23: Pushdown Optimization 🔥

### What is Pushdown Optimization?

**Concept:** Execute transformations in the source/target database instead of Informatica server

**Traditional Processing:**
```
Source DB → Extract to Informatica → Transform in Informatica → Load to Target DB
            (Network transfer)        (Informatica CPU/Memory)    (Network transfer)
```

**Pushdown Processing:**
```
Source DB → Transform in Database (SQL) → Extract results → Load to Target DB
            └─ Database CPU/Memory         (Minimal transfer)
```

**Benefits:**
- Reduced network traffic
- Leverages database processing power
- Faster for database-optimized operations
- Reduced Informatica server load

---

### Types of Pushdown Optimization

#### **1. Source-Side Pushdown**

**Concept:** Push transformations to source database

**How It Works:**
```
Informatica generates SQL and sends to source database
Database executes transformations
Results sent back to Informatica
```

**Supported Transformations:**
- Filter
- Expression (simple calculations)
- Aggregator
- Joiner (if both sources in same DB)
- Sorter

**Example:**
```
Without Pushdown:
1. SELECT * FROM EMPLOYEES (1M rows extracted)
2. Filter in Informatica: STATUS = 'A'
3. Aggregate in Informatica: SUM(SALARY) by DEPT

With Pushdown:
1. Database executes:
   SELECT DEPT_ID, SUM(SALARY) 
   FROM EMPLOYEES 
   WHERE STATUS = 'A' 
   GROUP BY DEPT_ID
2. Only 50 result rows sent to Informatica
```

---

#### **2. Target-Side Pushdown**

**Concept:** Push transformations to target database before loading

**Use Case:** When source and target are same database

**Example:**
```
Oracle source → Transformations → Oracle target (same database)

Pushdown:
Oracle database performs transformations internally
Results written directly to target table
```

---

#### **3. Full Pushdown**

**Concept:** Entire mapping executed in database

**Requirements:**
- Source and target in same database
- All transformations supported by pushdown

**Result:** Informatica just orchestrates; database does all work

---

### Configuring Pushdown Optimization

**Session Level:**
```
Session Properties → Config Object Tab:
- Pushdown Optimization: Full/Source/Target/None

Options:
- None: Traditional Informatica processing
- Source: Push to source database
- Target: Push to target database
- Full: Push entire mapping to database
```

**Mapping Level:**
```
Individual Transformation Properties:
- Some transformations have pushdown settings
```

---

### Pushdown Optimization Rules

**When Pushdown Works:**
✓ Source and target in same database (for Full)
✓ Supported transformations only
✓ Simple expressions (database-compatible functions)
✓ No unconnected lookups
✓ No sequence generators in flow

**When Pushdown Doesn't Work:**
✗ Complex Informatica-specific functions
✗ Unsupported transformations (Normalizer, etc.)
✗ Mixed sources (file + database)
✗ Mapplets with unsupported logic

---

### Pushdown Example Scenario

**Scenario:** Load daily sales summary from Oracle source to Oracle DW

**Without Pushdown:**
```
Mapping:
Oracle Source (SALES table, 10M rows)
  ↓
Filter: SALE_DATE = $SYSDATE - 1
  ↓
Aggregator: SUM(AMOUNT) GROUP BY PRODUCT_ID
  ↓
Oracle Target (SALES_SUMMARY, 1000 rows)

Processing:
- Extract 10M rows from source (slow network transfer)
- Filter in Informatica (9.9M rows removed)
- Aggregate in Informatica (memory intensive)
- Load 1000 rows to target

Time: 30 minutes
```

**With Source Pushdown:**
```
Session Config: Pushdown = Source

Informatica generates SQL:
SELECT PRODUCT_ID, SUM(AMOUNT) as TOTAL_AMOUNT
FROM SALES
WHERE SALE_DATE = SYSDATE - 1
GROUP BY PRODUCT_ID

Processing:
- Database filters and aggregates (fast, internal)
- Extract only 1000 result rows (minimal transfer)
- Load 1000 rows to target

Time: 3 minutes (10x faster!)
```

---

### Pushdown Best Practices

**1. Use for Large Datasets:**
```
Benefit increases with data volume:
- Small datasets (<10K rows): Minimal benefit
- Medium datasets (100K-1M): Moderate benefit
- Large datasets (>1M rows): Significant benefit
```

**2. Check Query Performance:**
```
Informatica-generated SQL may not be optimal
Review execution plans
Add indexes if needed
```

**3. Test Before Production:**
```
Pushdown doesn't always improve performance
Compare pushdown vs traditional for your specific scenario
```

**4. Monitor Database Load:**
```
Pushdown transfers load to database
Ensure database has capacity
May impact other database users
```

---

## Topic 24: Caching Strategies

### Lookup Caching (Revisited with Advanced Concepts)

#### **Cache Types Summary**

**1. Static Cache:**
- Read-only, built once at session start
- Best for: Reference tables that don't change during session

**2. Dynamic Cache:**
- Updated during session (inserts/updates)
- Best for: SCD implementations, avoid duplicates

**3. Persistent Cache:**
- Saved to disk, reused across sessions
- Best for: Large lookup tables, frequent session runs

**4. Shared Cache:**
- Multiple transformations share same cache
- Best for: Same lookup table used multiple times

---

#### **Persistent Cache Deep Dive** 🔥

**Purpose:** Avoid rebuilding cache every session

**How It Works:**
```
Session 1:
- Builds cache from database (10 min)
- Saves cache to disk
- Session runs (5 min)
Total: 15 min

Session 2:
- Loads cache from disk (30 sec)
- Session runs (5 min)
Total: 5.5 min (3x faster!)
```

**Configuration:**
```
Lookup Transformation Properties:
- Lookup Caching: Enabled
- Persistent Cache: Enabled
- Cache File Name Prefix: LKP_DEPT (unique name)

Session Properties:
- Cache Directory: /informatica/cache/
```

**Cache Files Created:**
```
/informatica/cache/LKP_DEPT.idx (index file)
/informatica/cache/LKP_DEPT.dat (data file)
```

**Cache Refresh:**
```
Option 1: Recache from lookup source
- Delete old cache files before session
- Forces rebuild

Option 2: Scheduled rebuild
- Workflow task to delete cache files periodically
- Balance between performance and data freshness
```

---

#### **Shared Cache Deep Dive**

**Scenario:** Multiple lookups on same table

**Without Shared Cache:**
```
Mapping:
LKP_DEPT_1 (builds cache, 100 MB, 2 min)
LKP_DEPT_2 (builds cache, 100 MB, 2 min)
LKP_DEPT_3 (builds cache, 100 MB, 2 min)

Total: 300 MB memory, 6 min build time
```

**With Shared Cache:**
```
Mapping:
LKP_DEPT_1 (builds cache, 100 MB, 2 min, Shared=Yes)
LKP_DEPT_2 (reuses cache, 0 MB, 0 min, Shared=Yes)
LKP_DEPT_3 (reuses cache, 0 MB, 0 min, Shared=Yes)

Total: 100 MB memory, 2 min build time
```

**Configuration:**
```
All lookup transformations must have:
- Same lookup table/query
- Lookup Caching: Enabled
- Shared Cache: Enabled
- Identical cache file name prefix
```

---

### Aggregator Caching

**How Aggregator Uses Cache:**
```
Aggregator holds grouped data in cache:
- Reads input rows
- Groups by specified columns
- Maintains running aggregates in cache
- Outputs results at end
```

**Cache Size Factors:**
- Number of groups
- Number of aggregate columns
- Data types

**Optimization:**
```
Sorted Input:
- Pre-sort by group-by columns
- Aggregator processes one group at a time
- Releases memory after each group
- Dramatic memory reduction

Without Sorted Input:
All groups in memory simultaneously: 500 MB

With Sorted Input:
One group at a time: 50 MB (10x reduction!)
```

---

### Joiner Caching

**How Joiner Uses Cache:**
```
Joiner loads Detail source into memory:
- Builds index on join columns
- Streams Master source
- Performs lookup for each master row
```

**Cache Size = Detail source size**

**Optimization:**
```
1. Designate smaller source as Detail:
   1M row source = Master
   1K row source = Detail
   Cache: Only 1K rows (minimal)

2. Sorted Input:
   - Sort both sources by join columns
   - Joiner merges streams
   - No cache needed!
```

---

### Index Caching

**Purpose:** Speed up lookup operations in cache

**How It Works:**
```
Without Index:
- Linear search through cache
- Time: O(n)

With Index:
- Hash-based index
- Time: O(1)
```

**Configuration:**
```
Lookup Properties:
- Lookup Caching: Enabled
- Index Cache: Enabled (default)
```

---

### Data Cache vs Index Cache

**Index Cache:**
- Stores lookup keys and pointers
- Small, fast
- Always enabled with lookup cache

**Data Cache:**
- Stores actual lookup data
- Larger
- Contains return columns

**Memory Usage:**
```
Total Cache = Index Cache + Data Cache

Example:
Index Cache: 10 MB (keys)
Data Cache: 90 MB (all columns)
Total: 100 MB
```

---

### Cache Tuning Recommendations

**1. Cache Directory:**
```
Session Properties:
- Cache Directory: Use fast local disk (SSD preferred)
- Avoid network drives
- Ensure sufficient space
```

**2. Cache Size:**
```
Lookup Properties:
- Auto (default): Informatica calculates
- Manual: Specify if you know requirements

For large lookups (>1GB):
Consider persistent cache or Source Qualifier join
```

**3. Cache Partitioning:**
```
For very large lookups:
- Partition lookup transformation
- Each partition has own cache
- Total cache distributed across partitions
```

---

## SUMMARY: MODULE 5 KEY CONCEPTS

### Performance Tuning
✅ Use Workflow Monitor for bottleneck identification
✅ Common bottlenecks: Source, Transformations, Target, Network
✅ Filter early, optimize transformation count
✅ DTM Buffer, Commit Interval tuning critical

### Partitioning
✅ Parallel processing for large datasets
✅ Types: Pass-Through, Round-Robin, Hash, Key Range
✅ Hash partitioning for Aggregator/Joiner (on group-by/join columns)
✅ Number of partitions = Number of CPUs
✅ Check for balanced distribution

### Pushdown Optimization
✅ Execute transformations in database (not Informatica)
✅ Reduces network traffic, leverages database power
✅ Types: Source, Target, Full pushdown
✅ Best for large datasets in same database
✅ Not all transformations supported

### Caching
✅ Static Cache: Read-only (reference data)
✅ Dynamic Cache: Updatable (SCD implementations)
✅ Persistent Cache: Saved to disk, reused across sessions
✅ Shared Cache: Multiple lookups share same cache
✅ Sorted Input dramatically reduces Aggregator/Joiner cache

---

## PRACTICE SCENARIOS FOR MODULE 5 MCQs

**Scenario 1:** Your session processes 10M rows. Aggregator transformation takes 80% of total time. What's the best optimization?
- **Answer:** Add Sorter before Aggregator (sort by group-by columns), enable Sorted Input in Aggregator properties. This reduces memory usage by 70-90%.

**Scenario 2:** You have 4 CPU cores on Informatica server. How many partitions should you create?
- **Answer:** 4 partitions (match number of CPUs for optimal performance)

**Scenario 3:** Your Aggregator groups by DEPT_ID. Which partition type should you use?
- **Answer:** Hash partitioning on DEPT_ID (ensures all rows with same DEPT_ID go to same partition)

**Scenario 4:** Both source and target are in same Oracle database. What optimization can dramatically improve performance?
- **Answer:** Enable Pushdown Optimization (Full or Source) - database performs transformations internally

**Scenario 5:** Lookup table has 5M rows and rarely changes. Session takes 10 min to build cache every run. How to optimize?
- **Answer:** Enable Persistent Cache - cache built once and saved to disk, subsequent sessions load from disk (much faster)

**Scenario 6:** You have 3 lookup transformations using same DEPARTMENT table in one mapping. Memory usage is high. What's the solution?
- **Answer:** Enable Shared Cache on all 3 lookups with same cache file name - cache built once and shared

**Scenario 7:** Session reads 10M rows from source but only needs 100K rows after filtering. Where should you apply the filter?
- **Answer:** Source Qualifier SQL Override (filter at source) - extract only 100K rows instead of 10M

**Scenario 8:** Joiner transformation is slow. Master source has 5M rows, Detail has 50K rows. What's wrong?
- **Answer:** Master/Detail designation reversed - smaller dataset (50K) should be Detail, larger (5M) should be Master

**Scenario 9:** Target load is slow. Loading 5M rows to Oracle database. What can speed it up?
- **Answer:** Use Bulk Load Mode, increase Commit Interval, disable indexes/constraints during load (rebuild after)

**Scenario 10:** Workflow Monitor shows Partition 1 processing 800K rows while Partitions 2-4 process 50K each. What's the issue?
- **Answer:** Unbalanced partitioning - switch from Hash/Key Range to Round-Robin for even distribution

**Scenario 11:** Your session fails with "Out of Memory" error during Aggregator processing. What should you check?
- **Answer:** Enable Sorted Input, reduce number of partitions, increase server memory, or check if data is skewed

**Scenario 12:** You want to speed up lookup on 100K row table that's updated daily. Which cache strategy?
- **Answer:** Static Cache with Persistent Cache - fast lookup, cache rebuilt daily to reflect updates

---

## HANDS-ON EXERCISES

### Exercise 1: Identify Performance Bottleneck
**Objective:** Use Workflow Monitor to find bottleneck
```
Steps:
1. Create mapping with multiple transformations:
   - Source (1M rows)
   - Filter
   - Aggregator (group by DEPT_ID)
   - Lookup (DEPARTMENT table)
   - Expression
   - Target

2. Run session

3. In Workflow Monitor:
   - Right-click session → Get Session Performance Details
   - Identify which transformation takes most time

4. Document findings:
   - Which transformation is bottleneck?
   - What % of total time?
   - Proposed optimization?
```

### Exercise 2: Implement Sorted Input Optimization
**Objective:** Optimize Aggregator with Sorted Input
```
Steps:
1. Create mapping:
   - Source: EMPLOYEES
   - Aggregator: Group by DEPT_ID, SUM(SALARY)
   - Target: DEPT_SALARY_SUMMARY

2. Run and note execution time (baseline)

3. Add optimization:
   - Insert Sorter before Aggregator
   - Sort by DEPT_ID (ascending)
   - Aggregator Properties → Sorted Input = TRUE

4. Run again and compare times

5. Check session log:
   - Memory usage before vs after
   - Time reduction achieved
```

### Exercise 3: Implement Partitioning
**Objective:** Apply partitioning for parallel processing
```
Steps:
1. Create mapping with large dataset:
   - Source: SALES (1M+ rows)
   - Filter: SALE_DATE = $SYSDATE - 1
   - Expression: Calculate profit
   - Target: SALES_SUMMARY

2. Baseline run (no partitioning)

3. Add partitioning:
   - Filter Properties → Partition Tab
   - Partition Type: Round-Robin
   - Number of Partitions: 4

4. Run session, compare performance:
   - Baseline time: ___
   - Partitioned time: ___
   - Speedup: ___x

5. Check session log for partition distribution
```

### Exercise 4: Configure Persistent Cache
**Objective:** Implement persistent cache for faster subsequent runs
```
Steps:
1. Create mapping with lookup:
   - Source: TRANSACTIONS
   - Lookup: CUSTOMER (large table, 500K+ rows)
   - Target: TRANS_WITH_CUSTOMER

2. Configure Lookup:
   - Lookup Caching: Enabled
   - Persistent Cache: Enabled
   - Cache File Name Prefix: LKP_CUSTOMER
   
3. Configure Session:
   - Cache Directory: /informatica/cache/

4. Run session twice:
   - Run 1: Cache built from database (note time)
   - Run 2: Cache loaded from disk (note time)
   
5. Compare cache build times:
   - Run 1 (build): ___ minutes
   - Run 2 (load): ___ seconds
   - Improvement: ___x faster
```

### Exercise 5: Pushdown Optimization
**Objective:** Implement source pushdown
```
Prerequisite: Source and target in same database

Steps:
1. Create mapping:
   - Oracle Source: SALES (10M rows)
   - Filter: REGION = 'WEST'
   - Aggregator: SUM(AMOUNT) by PRODUCT_ID
   - Oracle Target: PRODUCT_SALES

2. Baseline run:
   - Session Properties → Pushdown = None
   - Note execution time

3. Enable pushdown:
   - Session Properties → Config Object
   - Pushdown Optimization = Source
   
4. Run session, compare:
   - Without pushdown: ___ minutes
   - With pushdown: ___ minutes
   - Network data transferred (check logs)

5. Review session log:
   - Find generated SQL statement
   - Verify transformations pushed to database
```

### Exercise 6: Optimize Joiner Performance
**Objective:** Correct Master/Detail designation and apply Sorted Input
```
Steps:
1. Create mapping with Joiner:
   - Source 1: ORDERS (1M rows)
   - Source 2: CUSTOMERS (10K rows)
   - Joiner: Join on CUST_ID
   - Target: ORDER_DETAILS

2. Initial configuration (intentionally wrong):
   - Master: CUSTOMERS (small)
   - Detail: ORDERS (large)
   - Run and note time

3. Correct configuration:
   - Master: ORDERS (large)
   - Detail: CUSTOMERS (small)
   - Run and note improvement

4. Further optimization:
   - Add Sorter before Joiner (both sources, sort by CUST_ID)
   - Joiner Properties → Sorted Input = TRUE
   - Run and note additional improvement

5. Document results:
   - Wrong Master/Detail: ___ min
   - Correct Master/Detail: ___ min
   - With Sorted Input: ___ min
```

---

## ADVANCED PERFORMANCE TUNING TIPS 🔥

### Tip 1: Incremental Aggregation
**Problem:** Aggregating entire historical data every run

**Solution:** Incremental aggregation
```
Instead of:
SELECT DEPT_ID, SUM(SALARY) FROM EMPLOYEES GROUP BY DEPT_ID
(Processes all 10M rows every time)

Do:
1. Store aggregated results
2. Process only new/changed records
3. Update aggregates incrementally

Mapping:
Source (only new records) → Aggregator → 
  Lookup existing aggregates → 
  Expression (add new to existing) → 
  Update Target
```

---

### Tip 2: Pre-Aggregate in Source Qualifier
**Concept:** Perform aggregation in database when possible

```
Instead of:
Source Qualifier (extract 10M rows) → Aggregator → Target

Do:
Source Qualifier SQL Override:
SELECT DEPT_ID, SUM(SALARY) as TOTAL_SALARY
FROM EMPLOYEES
GROUP BY DEPT_ID

Result: Only aggregated rows (50) extracted
```

---

### Tip 3: Use Database Bulk Loaders
**Supported Databases:**
- Oracle: SQL*Loader
- SQL Server: Bulk Insert
- Sybase: Bulk Copy
- DB2: Load Utility

**Configuration:**
```
Session Properties → Target Properties:
- Target Load Type: Bulk
- External Loader: Enabled

Performance: 5-10x faster than normal insert
```

---

### Tip 4: Minimize Data Type Conversions
**Problem:** Conversions slow processing

**Optimization:**
```
Bad:
TO_CHAR(EMP_ID) || '-' || TO_CHAR(DEPT_ID)
(Two conversions)

Good:
EMP_ID || '-' || DEPT_ID
(Implicit conversion, single operation)
```

---

### Tip 5: Use Expression Instead of Multiple Transformations
**Concept:** Single transformation pass is faster

```
Bad:
Source → Expression1 → Expression2 → Expression3 → Filter → Target

Good:
Source → Expression (all logic) → Filter → Target
```

---

### Tip 6: Pipeline Optimization
**Concept:** Optimize data flow order

**Principles:**
1. **Filter Early:** Remove unwanted rows ASAP
2. **Heavy Operations Last:** Expensive transformations after filtering
3. **Minimize Transformations:** Combine where possible

```
Bad Flow:
Source → Lookup → Aggregator → Joiner → Filter → Target
(Heavy operations on full dataset)

Good Flow:
Source → Filter → Lookup → Aggregator → Joiner → Target
(Heavy operations on filtered data)
```

---

### Tip 7: Parallel Workflows
**Concept:** Run independent workflows simultaneously

```
Instead of Sequential:
wf_LoadDimensions (30 min) → wf_LoadFacts (60 min) = 90 min total

Parallel (if independent):
wf_LoadDimensions (30 min) ┐
wf_LoadFacts (60 min)      ├─ Both run together
wf_LoadAggregates (45 min) ┘
Total: 60 min (longest workflow)
```

---

### Tip 8: Reduce Lookup Cache Size
**Techniques:**

**1. SQL Override to filter:**
```
SELECT CUST_ID, CUST_NAME
FROM CUSTOMERS
WHERE STATUS = 'A'  ← Reduce rows
AND REGION = 'USA'
```

**2. Select only needed columns:**
```
SELECT CUST_ID, CUST_NAME  ← Only needed columns
FROM CUSTOMERS
(Don't SELECT *)
```

**3. Use WHERE clause:**
```
Reduces cache from 1M rows to 100K rows
90% memory savings
```

---

### Tip 9: Connection Pooling
**Concept:** Reuse database connections

**Configuration:**
```
Connection Object Properties:
- Maximum Connections: 5
- Connection Retry: 3
- Connection Pooling: Enabled

Benefit: Faster session startup, reduced connection overhead
```

---

### Tip 10: Monitor and Adjust Dynamically
**Continuous Improvement:**

1. **Baseline Metrics:**
   - Record initial performance
   - Document row counts, timing

2. **Monitor Regularly:**
   - Weekly performance reviews
   - Identify degradation trends

3. **Adjust Proactively:**
   - Increase commit intervals as data grows
   - Add partitions for large datasets
   - Archive old data

4. **Test Optimizations:**
   - Always test in non-production
   - Compare before/after metrics
   - Document changes

---

## PERFORMANCE TUNING CHECKLIST

**Source Optimization:**
☐ Use Source Qualifier filter (WHERE clause)
☐ Use Source Qualifier join (if same database)
☐ Index source columns (join, filter, sort columns)
☐ Consider source-based partitioning
☐ Verify network bandwidth

**Transformation Optimization:**
☐ Filter early in pipeline
☐ Enable Sorted Input for Aggregator
☐ Enable Sorted Input for Joiner
☐ Correct Master/Detail in Joiner
☐ Use Persistent Cache for large lookups
☐ Use Shared Cache for repeated lookups
☐ Replace Lookup with SQ join if possible
☐ Minimize transformation count
☐ Avoid unnecessary data type conversions

**Target Optimization:**
☐ Use Bulk Load mode
☐ Increase Commit Interval
☐ Disable indexes during load
☐ Disable constraints during load
☐ Rebuild indexes/constraints after load
☐ Use Pre/Post SQL for DDL

**Session Configuration:**
☐ Increase DTM Buffer Pool Size
☐ Adjust Buffer Block Size
☐ Configure appropriate Commit Interval
☐ Enable partitioning (match CPU count)
☐ Use correct partition type (Hash for Aggregator/Joiner)
☐ Enable Pushdown Optimization (if applicable)

**Monitoring:**
☐ Review Workflow Monitor performance details
☐ Identify bottlenecks
☐ Check partition distribution
☐ Monitor memory usage
☐ Verify commit frequency
☐ Review session logs for errors/warnings

---

## REAL-WORLD PERFORMANCE CASE STUDY

**Scenario:** Daily sales load running for 4 hours, SLA is 2 hours

**Initial State:**
```
Data Volume: 50M transactions/day
Current Time: 4 hours
Target SLA: 2 hours
Required Improvement: 2x faster
```

**Analysis (Workflow Monitor):**
```
Source Read: 30 min (12%)
Filter: 5 min (2%)
Aggregator: 180 min (75%) ← BOTTLENECK!
Lookup: 20 min (8%)
Target Write: 5 min (2%)
Total: 240 min
```

**Optimizations Applied:**

**Step 1: Aggregator Sorted Input**
```
Added Sorter before Aggregator (sort by group-by columns)
Enabled Sorted Input in Aggregator
Result: Aggregator time reduced from 180 min to 45 min
New Total: 105 min (2.3x improvement)
```

**Step 2: Source Qualifier Filter**
```
Moved filter logic to Source Qualifier SQL Override
Reduced extracted rows from 50M to 40M
Result: Source read time reduced from 30 min to 24 min
New Total: 99 min
```

**Step 3: Partitioning**
```
Applied Round-Robin partitioning (4 partitions) on Filter
Parallel processing on 4 CPU cores
Result: Overall processing reduced by 30%
New Total: 69 min
```

**Step 4: Persistent Lookup Cache**
```
Enabled Persistent Cache for lookup transformation
First run: 20 min cache build
Subsequent runs: 2 min cache load
Result: Lookup time reduced from 20 min to 2 min
New Total: 51 min
```

**Step 5: Bulk Load**
```
Enabled Bulk Load mode for target
Increased Commit Interval from 10K to 100K
Result: Target write from 5 min to 2 min
Final Total: 48 min
```

**Final Result:**
```
Initial: 240 min (4 hours)
Final: 48 min (0.8 hours)
Improvement: 5x faster (500% improvement)
SLA: Met with room to spare (2 hour target)
```

---

## TCS WINGS EXAM - MODULE 5 FOCUS AREAS

**Critical Concepts for MCQs:**

1. **Sorted Input Optimization**
   - Most common performance question
   - Aggregator and Joiner benefit
   - Reduces memory usage dramatically

2. **Partitioning Types**
   - Hash for Aggregator/Joiner (on group-by/join columns)
   - Round-Robin for balanced distribution
   - Number of partitions = CPU count

3. **Pushdown Optimization**
   - When to use (same database, large datasets)
   - Types (Source, Target, Full)
   - Limitations (not all transformations supported)

4. **Cache Strategies**
   - Static vs Dynamic (SCD use case)
   - Persistent (reuse across sessions)
   - Shared (multiple lookups)

5. **Master/Detail in Joiner**
   - Smaller source = Detail (loaded in memory)
   - Larger source = Master (drives join)

6. **Bottleneck Identification**
   - Use Workflow Monitor
   - Performance Details view
   - Identify slowest transformation

7. **Common Optimizations**
   - Filter early
   - Source Qualifier filter/join
   - Bulk load mode
   - Increase commit interval
   - Disable indexes during load

**Hands-On Scenarios to Practice:**

1. Configure Sorted Input for Aggregator
2. Apply Hash partitioning on group-by columns
3. Enable Persistent Cache for lookup
4. Configure Pushdown Optimization
5. Correct Master/Detail in Joiner
6. Use Workflow Monitor to identify bottleneck

---

**🎯 MODULE 5 COMPLETE!**

You've now mastered:
- Performance tuning methodologies
- Bottleneck identification and resolution
- Partitioning for parallel processing
- Pushdown optimization techniques
- Advanced caching strategies

**Completion Status:**
✅ Module 1: Fundamentals
✅ Module 2: Core Components
✅ Module 3: Transformations
✅ Module 4: Advanced Techniques
✅ Module 5: Performance & Optimization

**Remaining:**
📍 Module 6: Scenario-Based Concepts (SCD, Incremental Loading) - **HIGH EXAM WEIGHTAGE**
📍 Module 7: Hands-On Preparation

**Ready for Module 6?** This is where we cover Slowly Changing Dimensions (SCD Type 1, 2, 3) and other scenario-based concepts that appear frequently in TCS Wings MCQs!

Type **"Module 6"** to continue! 🚀