Here are the detailed notes for Module 5: Advanced Transformation Concepts (Intermediate Level).
This module focuses on the Lookup Transformation, which is one of the most powerful, flexible, and frequently-tested components in Informatica.
Topic 5.1: Lookup Transformation (Active/Passive)
 * Type: Can be Passive (default) or Active.
 * Purpose: To "look up" data in a related table or file. The most common use is to find a corresponding value or to check if a record already exists.
 * How it works:
   * Data from your main pipeline (e.g., a Sales flat file) flows into the Lookup transformation.
   * This data has a "key" port (e.g., PRODUCT_ID).
   * The Lookup transformation takes this key and searches for a match in a "lookup source" (e.g., the DIM_PRODUCT table).
   * It finds a match based on a lookup condition (e.g., IN_PRODUCT_ID = DIM_PRODUCT.PRODUCT_ID).
   * It can then return values from the lookup source (e.g., PRODUCT_KEY, PRODUCT_NAME) back into the main pipeline.
 * Typical Scenarios:
   * Finding Surrogate Keys: The sales file has PRODUCT_ID (101), but the data warehouse fact table needs the PRODUCT_KEY (a surrogate key, e.g., 5001). You look up the DIM_PRODUCT table to find the key.
   * Checking for Existence (SCDs): You look up the DIM_CUSTOMER table with the CUSTOMER_ID from the source to see if that customer already exists.
   * Enriching Data: You have an EMP_ID and want to get the DEPT_NAME from the DEPARTMENTS table.
 * Active vs. Passive:
   * Passive (Default): Returns only one value for any given input, even if it finds multiple matches. If it finds a match, it returns the value. If not, it returns NULL. The number of rows leaving the transformation is exactly the number of rows that entered.
   * Active: You can configure the Lookup to return all matching rows if multiple matches are found. This will change the number of rows (e.g., 1 row goes in, 5 rows come out), making it an Active transformation. This is set in the properties by changing the "Lookup Policy on Multiple Match" to "Return All Values."
Topic 5.2: Connected vs. Unconnected Lookups
This is a critical intermediate concept. A Lookup can be used in two different ways.
1. Connected Lookup
 * What it is: It is "connected" directly in the pipeline, just like any other transformation. It's part of the data flow.
 * Data Flow: Rows flow from a source (or another transformation) into the Lookup, and out of the Lookup to the next transformation (or target).
 * Returns:
   * Can return multiple ports (columns) from the lookup source.
   * Example: You send in EMP_ID and can get back DEPT_ID, DEPT_NAME, and LOCATION all at once.
 * Caching: Can use all cache types (static, dynamic, persistent).
 * When to use: This is the default and most common type. Use it when you need to return multiple columns or when you need to access the lookup's output ports in the very next transformation.
2. Unconnected Lookup
 * What it is: It is "unconnected" to the main pipeline. It sits off to the side in the mapping.
 * Data Flow: Data does not flow through it.
 * How it works:
   * You must call the Unconnected Lookup from another transformation (usually an Expression) using a special function: :LKP.lookup_name(arg1, arg2, ...).
   * The arguments you pass to the function are the "input" ports for the lookup.
   * The lookup performs its search and returns a single value back to the Expression.
 * Returns:
   * Can only return one port (column) back to the calling expression.
   * Rule: The port you want to return must be marked with the "R" (Return) property in the Lookup's "Ports" tab.
 * Caching: Can only use static or persistent cache. Cannot use a dynamic cache.
 * When to use:
   * When you only need to return one value.
   * When you want to call the lookup conditionally (e.g., inside an IIF statement in an Expression).
   * Scenario: IIF(ISNULL(TGT_KEY), :LKP.LKP_GET_KEY(SRC_ID), TGT_KEY)
     This expression says, "If the target key is null, then call the lookup; otherwise, just use the existing target key." This improves performance by not running the lookup for every row.
| Feature | Connected Lookup | Unconnected Lookup |
|---|---|---|
| Pipeline | Sits directly in the data flow. | Sits outside the data flow. |
| How to call | Receives all rows automatically. | Called from an Expression using :LKP function. |
| Input Ports | Receives input from pipeline links. | Receives input as arguments in the :LKP call. |
| Return Values | Can return multiple ports. | Can return only one port (the 'R' port). |
| Dynamic Cache | Yes, can use dynamic cache. | No, cannot use dynamic cache. |
| Use Case | Get multiple columns; check every row. | Get one column; call conditionally for performance. |
Topic 5.3: Lookup Caching (Static, Dynamic, Persistent)
This is a key performance tuning concept. To avoid running a SQL query against the lookup table for every single source row, the Integration Service builds a "cache" (a copy) of the lookup data in memory.
 * Lookup Cache: A "snapshot" of the lookup table's data.
 * Static Cache (Default)
   * How it works:
     * When the session starts, the Integration Service runs a query (SELECT ... FROM LOOKUP_TABLE) and builds the entire cache in memory before the first source row is processed.
     * For every source row, it looks for a match in the memory cache.
     * The cache does not change during the session.
   * Problem: What if the lookup table is a dimension table (e.g., DIM_CUSTOMER) that you are also loading in the same mapping? The cache won't know about the new customers you just inserted.
   * Use Case: Best for "read-only" tables (e.g., DIM_TIME, DEPARTMENTS) that are not changing during the ETL run.
 * Dynamic Cache
   * How it works:
     * Builds the cache just like a static cache.
     * Key Difference: You can update the cache in memory as the mapping runs.
     * When a row comes in, the lookup checks the cache.
     * If a match is found, it processes.
     * If a match is not found, it can insert that new row into the cache (and flag it as DD_INSERT). This new row is now available for the next source row to find.
   * Use Case: This is the core of SCD Type 1 loading. You use a dynamic cache on the target dimension table.
     * Scenario (New Customer):
       * Source row (Customer 101) comes in.
       * Lookup checks dynamic cache. No match.
       * Lookup inserts Customer 101 into the cache and passes the row to an Update Strategy.
       * The Update Strategy sees the DD_INSERT flag and inserts the new customer into the target table.
 * Persistent Cache
   * How it works: This is a static cache that is saved to disk when the session finishes.
   * The next time the session (or another session) runs, it can be configured to reuse the cache files from disk instead of re-building the cache from the database.
   * Use Case: Extremely large, static tables (e.g., a 50 GB DIM_PRODUCT table) that don't change often. Building the cache takes 30 minutes. If you can re-use the cache, you save 30 minutes on every job run.
Topic 5.4: SQL Transformation (Active/Passive)
 * Type: Can be Active or Passive.
 * Purpose: To run any SQL query (including SELECT, INSERT, UPDATE, DELETE, or a Stored Procedure) in the middle of a mapping.
 * Modes:
   * Query Mode: You pass in values to the SQL query (e.g., in a WHERE clause) and it returns rows. This is like a lookup but you have full SQL control.
   * Script Mode: You run a pre-written SQL script from a file.
 * Scenario: You need to call a complex Stored Procedure in the database for each row to get a calculated value, or you need to perform an UPDATE on a summary table in the middle of your flow.
Topic 5.5: Transaction Control Transformation (Active)
 * Type: Active
 * Purpose: To control database transaction boundaries (COMMIT and ROLLBACK) at a row-level, rather than at the end of the session.
 * How it works: You use an IIF statement to define when to commit.
   * Condition: IIF(DEPT_ID != PREV_DEPT_ID, TC_COMMIT_BEFORE, TC_CONTINUE_TRANSACTION)
 * Constants:
   * TC_COMMIT_BEFORE: Commit the previous transaction, then process this row in a new one.
   * TC_COMMIT_AFTER: Process this row, then commit the transaction.
   * TC_ROLLBACK_BEFORE: Roll back the previous transaction.
   * TC_CONTINUE_TRANSACTION: (Default) Do nothing, just continue.
 * Scenario: You are loading a large file containing data for many different departments. You want to COMMIT the data to the database after each department is fully loaded. You would use a Sorter (by DEPT_ID) and then a Transaction Control to issue a commit every time the DEPT_ID changes. This way, if the job fails while loading department 50, the data for departments 1-49 is already safely committed.
Would you like to move on to Module 6: Workflow Manager (Running the Jobs)?
