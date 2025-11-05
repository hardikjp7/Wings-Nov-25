Here are the detailed notes for Module 4: Core Transformations.
This is the most important module for scenario-based questions. These transformations are the building blocks for all ETL logic.
Topic 4.1: Active vs. Passive Transformations
This is a fundamental concept. Every transformation is either Active or Passive.
 * Passive Transformation:
   * A transformation that does NOT change the number of rows passing through it.
   * It maintains a one-to-one row count. If 100 rows go in, 100 rows come out.
   * It also does not change the row order.
   * Examples: Expression, Sequence Generator.
 * Active Transformation:
   * A transformation that CAN change the number of rows.
   * The row count going in might not equal the row count coming out.
   * Example: A Filter transformation might receive 100 rows but only output 50 (changing the row count).
   * It may also change the row order (e.g., Sorter).
   * Examples: Filter, Router, Sorter, Aggregator, Joiner, Rank, Update Strategy.
Topic 4.2: Expression Transformation (Passive)
 * Type: Passive
 * Purpose: To perform row-level calculations or modifications. It's the most common transformation.
 * What it does: It allows you to create new columns or change existing ones for each single row that passes through.
 * Ports (Columns):
   * Input (I): A port that only receives data.
   * Output (O): A port that only sends data (often the result of a calculation).
   * Input/Output (I/O): A port that both receives data and sends it.
   * Variable (V): A temporary, local variable used only within the expression. It is not passed out of the transformation. This is used for complex calculations or to hold a value from a previous row.
 * Common Scenarios:
   * Concatenation: FIRST_NAME || ' ' || LAST_NAME
   * Math: PRICE * QUANTITY (to create a TOTAL_SALES port)
   * Data Cleansing: TRIM(PRODUCT_NAME)
   * Conditional Logic: IIF(SALARY > 50000, 'High', 'Low') (IIF is Informatica's "Immediate If")
   * Date Functions: ADD_TO_DATE(HIRE_DATE, 'DD', 30)
Topic 4.3: Filter Transformation (Active)
 * Type: Active
 * Purpose: To filter out (remove) rows based on a condition.
 * How it works: You define a Filter Condition.
   * If the condition for a row evaluates to TRUE, the row passes through.
   * If the condition evaluates to FALSE or NULL, the row is dropped.
 * Key Property:
   * Filter Condition: STATE = 'CA' or SALARY > 10000 AND DEPT_ID != 20
 * Scenario (SQ vs. Filter):
   * Use a Source Qualifier (SQ) filter if the source is a database and you want to filter before data leaves the database (more efficient).
   * Use a Filter transformation if the source is a Flat File (which has no WHERE clause) or if you need to filter based on a calculation you made in a previous Expression transformation.
Topic 4.4: Router Transformation (Active)
 * Type: Active
 * Purpose: To test a row against multiple conditions and send it to different "groups" or "paths" based on those conditions.
 * How it works: It's like a CASE statement or a series of IF...ELSE IF... blocks.
 * Key Components:
   * Input Group: All rows enter here.
   * Output Groups (User-Defined): You create named groups, each with a condition.
     * Example Group 1: DEPT_ID = 10
     * Example Group 2: DEPT_ID = 20
     * Example Group 3: SALARY > 100000
   * Default Group: All rows that do NOT meet any of the user-defined group conditions will go to this group.
 * Scenario (Filter vs. Router):
   * Use Filter if you have one condition and you only want to KEEP or DROP rows (e.g., "Keep only department 10").
   * Use Router if you have multiple conditions and want to split the data into multiple streams (e.g., "Stream 1 is Dept 10", "Stream 2 is Dept 20", "Stream 3 is everyone else"). A Router can do the job of multiple Filter transformations.
Topic 4.5: Sorter Transformation (Active)
 * Type: Active
 * Purpose: To sort data in ascending or descending order based on one or more ports (columns).
 * Key Properties:
   * Sort Key: The port(s) you want to sort by (e.g., DEPT_ID).
   * Direction: Ascending (ASC) or Descending (DESC) for each key.
   * Distinct: A checkbox that, if checked, will remove duplicate rows based on the sort keys.
 * Important Scenario: Some transformations require data to be sorted before they can work.
   * Aggregator: Can be used for incremental aggregation.
   * Joiner: Can be used if you enable the "Sorted Input" option for performance.
   * Update Strategy: If you're updating a table, sorting by primary key can be faster.
Topic 4.6: Aggregator Transformation (Active)
 * Type: Active
 * Purpose: To perform aggregate calculations, like SUM, COUNT, AVG, MIN, MAX.
 * How it works: It groups rows and then performs a calculation on each group.
 * Key Properties:
   * Group By: The port(s) that define the group (e.g., DEPT_ID). This is similar to the SQL GROUP BY clause.
   * Aggregate Expression: The calculation to perform on each group (e.g., SUM(SALARY)).
 * Sorted Input:
   * A checkbox. If your data is already sorted by the Group By keys (e.g., you used a Sorter or an ORDER BY in the SQ), check this box.
   * Benefit: This dramatically improves performance. Instead of storing all rows in memory (cache) to perform the grouping, the Aggregator only compares the current row to the previous one.
Topic 4.7: Joiner Transformation (Active)
 * Type: Active
 * Purpose: To join two different data sources (pipelines) based on a join condition.
 * Key Components:
   * Master Pipeline: The data stream that is "held back" or cached. The Integration Service stores this in memory or on disk.
   * Detail Pipeline: The data stream that is read row-by-row and compared against the Master.
   * Best Practice: The source with fewer rows should be the Master pipeline to save cache/memory.
 * Key Properties:
   * Join Condition: The columns to join on (e.g., MASTER.DEPT_ID = DETAIL.DEPT_ID).
   * Join Type:
     * Normal Join: (Like SQL INNER JOIN) Only matching rows are output.
     * Master Outer Join: (Like SQL RIGHT OUTER JOIN) All rows from the Detail pipeline + matching rows from the Master.
     * Detail Outer Join: (Like SQL LEFT OUTER JOIN) All rows from the Master pipeline + matching rows from the Detail.
     * Full Outer Join: All rows from both pipelines, matched where possible.
 * Sorted Input: Like the Aggregator, if both data streams are sorted on the join keys, you can check this box to improve performance.
Topic 4.8: Rank Transformation (Active)
 * Type: Active
 * Purpose: To select the Top N or Bottom N rows from a group.
 * How it works:
   * It sorts the data based on a "Rank Port."
   * It groups the data using a "Group By Port."
   * It selects the top or bottom N rows from each group.
 * Key Properties:
   * Rank Port: The column to rank by (e.g., SALARY).
   * Top/Bottom: Choose one.
   * Number of Ranks (N): The number of rows to select (e.g., 5 for Top 5).
   * Group By Port(s): The column to define the groups (e.g., DEPT_ID).
 * Scenario: "Find the top 3 highest-paid employees in each department."
   * Rank Port: SALARY (Direction: Top)
   * Number of Ranks: 3
   * Group By Port: DEPT_ID
Topic 4.9: Sequence Generator (Passive)
 * Type: Passive
 * Purpose: To generate a unique sequence of numbers (e.g., 1, 2, 3, 4...).
 * Key Properties:
   * Start Value: The number to start with.
   * Increment By: The value to add for each new number (e.g., 1).
   * End Value: The maximum value to reach.
   * Current Value: The next value that will be generated.
   * Reset: A checkbox. If checked, the sequence will reset to the Start Value every time the session runs. If unchecked, it will continue from where it left off (e.g., session 1 ends at 1000, session 2 starts at 1001).
 * Scenario: Creating unique Surrogate Keys for a data warehouse dimension table.
Topic 4.10: Update Strategy (Active)
 * Type: Active
 * Purpose: To "flag" rows for different database operations: Insert, Update, Delete, or Reject.
 * How it works:
   * You add this transformation and write a condition (e.g., IIF(ISNULL(TGT_CUSTOMER_ID), DD_INSERT, DD_UPDATE)).
   * This "flags" the row with a special code.
   * The Integration Service does not perform the operation.
   * The row is passed to the Target Definition. In the session's target properties, you must set the "Treat Source Rows As" property to "Data Driven".
   * When the service sees the "Data Driven" setting, it looks for the flag on each row and performs the correct DML (Insert, Update, Delete) operation.
 * Constants:
   * DD_INSERT (Value: 0)
   * DD_UPDATE (Value: 1)
   * DD_DELETE (Value: 2)
   * DD_REJECT (Value: 3)
Topic 4.11: Normalizer (Active)
 * Type: Active
 * Purpose: To convert columns into rows. This is used to "normalize" data, typically from a COBOL source (mainframe file) where data is denormalized.
 * Scenario: You have a source file where sales are in columns:
   PRODUCT | SALES_JAN | SALES_FEB | SALES_MAR
   'Apple' | 100 | 150 | 200
 * The Normalizer can pivot this into:
   PRODUCT | MONTH | SALES
   'Apple' | 'JAN' | 100
   'Apple' | 'FEB' | 150
   'Apple' | 'MAR' | 200
 * It's the opposite of a PIVOT function.
Topic 4.12: Union Transformation (Active)
 * Type: Active
 * Purpose: To merge two or more data streams into a single stream. It is similar to a SQL UNION ALL.
 * How it works:
   * You create multiple input groups.
   * You connect different data pipelines to these groups.
   * All input groups are merged into a single output group.
   * Rule: The "shape" of all input groups must match. They must have the same number of ports, and the data types of the ports must be compatible in order.
 * Scenario: You have two source tables, EMPLOYEES_USA and EMPLOYEES_UK, with the same columns. You want to load them into a single target table GLOBAL_EMPLOYEES. You would read each source, then use a Union to merge them before loading the target.
Would you like to move on to Module 5: Advanced Transformation Concepts?
