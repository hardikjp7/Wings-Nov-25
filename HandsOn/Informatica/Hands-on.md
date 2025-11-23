# Informatica PowerCenter - Exam Preparation Guide

Based on the practice challenges you've shared, I'll help you prepare for the TCS Wings Informatica hands-on exam. Let me break down the key concepts and patterns you need to master.

## Core Skills You Must Know

### 1. **Oracle SQL Setup & Table Creation**
You'll need to:
- Connect to Oracle SQL Developer (system/Admin)
- Create tables with proper data types
- Load CSV data into tables
- Understand data type conversions (VARCHAR to DATE)

**Key Pattern:**
```sql
CREATE TABLE table_name (
    column1 datatype,
    column2 datatype,
    ...
);
```

### 2. **Informatica Repository & Connection Setup**

**Standard Steps You'll Repeat:**
- Connect to Repository Manager (Administrator/Administrator)
- Create project folder
- Create ODBC connection (Oracle, localhost, 1521, xe)
- Create Relational Connection in Workflow Manager (oracle/system/Admin/xe)

### 3. **Common Informatica Transformations**

Based on the challenges, master these transformations:

**A. Aggregator Transformation**
- Calculate SUM, AVG, COUNT, MIN, MAX
- Group by specific columns
- Used in almost every task

**B. Expression Transformation**
- Extract numeric parts: `SUBSTR`, `REGEXP_REPLACE`
- Concatenate strings: `||` operator
- Date calculations: `SHIP_DATE - ORDER_DATE`
- Conditional logic: `IIF()` function
- Rounding: `ROUND(value, 2)` for decimal precision

**C. Filter Transformation**
- Filter rows based on conditions
- Multiple conditions with AND/OR

**D. Sorter Transformation**
- Sort ascending/descending
- Multiple sort keys

**E. Rank Transformation**
- Get TOP N records
- Rank based on specific columns

**F. Router Transformation**
- Split data into multiple groups based on conditions

**G. Joiner Transformation**
- Join multiple sources (though not in these examples)

## Common Task Patterns

### Pattern 1: Data Cleaning
```
Source → Filter (remove nulls/duplicates) 
       → Expression (transform columns)
       → Filter (country/region specific)
       → Target
```

**Typical Operations:**
- Remove duplicates
- Filter by country/state/city
- Extract numeric values from IDs
- Concatenate columns
- Drop unnecessary columns

### Pattern 2: Aggregation & Summary
```
Source → Filter (specific criteria)
       → Aggregator (GROUP BY, SUM, AVG, COUNT)
       → Expression (calculations)
       → Sorter (order results)
       → Filter (threshold values)
       → Target
```

**Typical Operations:**
- Group by customer/city/region
- Calculate totals and averages
- Sort by calculated values
- Filter top performers

### Pattern 3: Ranking & Top N
```
Source → Filter (category/location)
       → Aggregator (count orders)
       → Sorter (descending)
       → Rank (top N)
       → Target
```

### Pattern 4: Categorization
```
Source → Expression (calculate values)
       → Expression (categorize using IIF)
       → Aggregator (count by category)
       → Target
```

## Key Formulas & Expressions You'll Use

### String Operations:
```
Extract numeric: REGEXP_REPLACE(CUSTOMER_ID, '[^0-9]', '')
Concatenate: EXTRACTED_ID || '-' || CUSTOMER_NAME
```

### Date Operations:
```
Processing days: SHIP_DATE - ORDER_DATE
Convert to date: TO_DATE(column, 'DD/MM/YYYY')
```

### Conditional Logic:
```
IIF(condition, true_value, false_value)
IIF(PROCESSING_DAYS < 1, 'Immediate delivery',
    IIF(PROCESSING_DAYS <= 3, 'Moderate Delivery', 
        'Long Term delivery'))
```

### Mathematical Operations:
```
Average: TOTAL_SALES / COUNT_ORDERS
Percentage: (DELIVERED / TOTAL) * 100
Round: ROUND(value, 2)
```

## Workflow Creation Steps (Memorize This!)

**Every task follows this pattern:**

1. **Create Mapping** in Designer
   - Import source
   - Add transformations
   - Create target
   - Link all objects

2. **Create Session** in Workflow Manager
   - Task → Create → Session
   - Assign mapping
   - Configure connections (source & target)
   - Set target load type: Normal + Insert

3. **Create Workflow**
   - Workflows → Create
   - Name: Workflow_[TaskName]
   - Drag session into workflow
   - Link Start → Session

4. **Execute & Monitor**
   - Start workflow
   - Check Workflow Monitor
   - Verify target table data

## Common Naming Conventions

You MUST follow these naming patterns:

```
Mapping:  Map_[TaskName] or m_[taskname]
Workflow: Workflow_[TaskName] or w_[taskname]
Session:  Session_[TaskName] or s_[taskname]
Target:   [TaskName] table
```

## Critical Tips for Exam Success

### Before Starting:
1. ✅ Read the entire problem statement twice
2. ✅ Note the sample output columns - this is your target structure
3. ✅ Identify which transformations you'll need
4. ✅ Plan the transformation flow on paper

### During Execution:
1. ✅ Always remove nulls/duplicates in data cleaning tasks
2. ✅ Pay attention to decimal precision (use ROUND where specified)
3. ✅ Double-check filter conditions (Country, State, City, Category)
4. ✅ Verify column names match the sample output exactly
5. ✅ Test expressions before connecting to target

### Common Mistakes to Avoid:
1. ❌ Wrong data types in target table
2. ❌ Forgetting to drop columns mentioned in requirements
3. ❌ Incorrect naming conventions
4. ❌ Not setting "Insert" in session target properties
5. ❌ Missing decimal rounding where required
6. ❌ Wrong sort order (ascending vs descending)

## Practice Exercise Checklist

For each practice problem, ensure you can:

- [ ] Create Oracle tables with correct structure
- [ ] Load CSV data successfully
- [ ] Set up ODBC and relational connections
- [ ] Use Aggregator for grouping and calculations
- [ ] Write complex IIF expressions
- [ ] Extract and transform string data
- [ ] Calculate date differences
- [ ] Sort and rank data
- [ ] Filter with multiple conditions
- [ ] Create and execute workflows successfully
- [ ] Verify output matches sample exactly

## Time Management Strategy

In a typical 2-3 hour exam:

- **15 mins**: Setup (Oracle tables, connections)
- **20 mins**: Data Cleaning task
- **25 mins**: Each analysis task (4 tasks = 100 mins)
- **10 mins**: Testing and validation
- **10 mins**: Buffer for issues

## Quick Reference Card

**Most Used Transformations Order:**
1. Source Qualifier (always first)
2. Filter (remove unwanted rows)
3. Expression (calculations, transformations)
4. Aggregator (grouping, sum, avg, count)
5. Expression (post-aggregation calculations)
6. Sorter (ordering)
7. Rank (top N records)
8. Filter (threshold filtering)
9. Target (always last)

Would you like me to create specific practice scenarios or dive deeper into any particular transformation or concept?
