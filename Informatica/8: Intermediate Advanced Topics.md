Here are the detailed notes for Module 8: Intermediate/Advanced Topics (Hands-On & Scenarios).
This module combines many of the previous topics to solve complex, real-world ETL problems. These are the exact types of scenarios you will be tested on.
Topic 8.1: Parameters and Variables
These allow you to make your mappings and workflows flexible and reusable.
 * Parameter (Static):
   * A value that does not change during the session run.
   * It is defined before the session starts.
   * Example: You create a parameter $$Source_File_Name. In a Parameter File, you set $$Source_File_Name = sales_data_20251105.csv. The session reads this value at the start and uses it.
   * Use Case: To avoid hard-coding values like file names, directory paths, or database connections. This allows you to run the same workflow in DEV, TEST, and PROD by simply changing the parameter file.
 * Variable (Dynamic):
   * A value that can change during the session run.
   * The Integration Service stores the final value of the variable at the end of the session.
   * Key Functions: SETVARIABLE($$VarName, value), SETMINVARIABLE(), SETMAXVARIABLE(), SETCOUNTVARIABLE().
   * Scenario (Incremental Load): This is the classic scenario.
     * Create a mapping variable $$Last_Run_Date.
     * In the Source Qualifier, your query is:
       SELECT * FROM ORDERS WHERE CREATE_DATE > '$$Last_Run_Date'
     * At the end of your mapping, in an Expression, you find the maximum CREATE_DATE from the current batch of source rows (e.g., '2025-11-05').
     * You use SETMAXVARIABLE($$Last_Run_Date, CURRENT_MAX_DATE) to update the variable's value.
     * The session runs, loads the data, and saves '2025-11-05' as the new value for $$Last_Run_Date.
     * Tomorrow, when the session runs again, the SELECT query will automatically be ...WHERE CREATE_DATE > '2025-11-05'.
Topic 8.2: Slowly Changing Dimensions (SCDs)
This is the most common and important data warehouse scenario. A "Slowly Changing Dimension" is a dimension (like Customer, Product, Employee) that changes its attributes over time.
Scenario: A customer, John Smith, moves from California to New York. How do you handle this change in your DIM_CUSTOMER table?
 * SCD Type 1 (Overwrite):
   * Goal: Keep no history. Overwrite the old value with the new value.
   * Method:
     * Source row (John Smith, New York) comes in.
     * Lookup the DIM_CUSTOMER table by CUSTOMER_ID.
     * Find a match. Compare the STATE from the source (New York) with the STATE in the target (California).
     * They are different, so flag the row for Update (using an Update Strategy).
     * The session updates the existing row. The STATE column is changed from California to New York.
   * Result: You have lost the information that John ever lived in California.
 * SCD Type 2 (Keep History - Versioning):
   * Goal: Keep a full history of all changes by creating a new version of the row.
   * Method: This is the classic, complex scenario. The target DIM_CUSTOMER table needs extra columns: Start_Date, End_Date, and Is_Current_Flag (Y/N).
   * Source Row: (John Smith, New York)
   * Lookup: Lookup DIM_CUSTOMER on CUSTOMER_ID and Is_Current_Flag = 'Y'.
   * Result: A match is found (the California row).
   * Logic: Since the STATE has changed, the mapping needs to do two things:
     * Action 1 (Update Old Row): Flag the old California row for Update. The update will set its End_Date to yesterday and Is_Current_Flag to N.
     * Action 2 (Insert New Row): Create a new row: (John Smith, New York, Start_Date = today, End_Date = null, Is_Current_Flag = Y). Flag this row for Insert.
   * Result: You now have two rows for John Smith, but only one is "active." You have a complete history of his moves.
 * SCD Type 3 (Keep Limited History):
   * Goal: Keep only the "current" and "previous" value.
   * Method: Add a Previous_State column to the table.
   * Source Row: (John Smith, New York)
   * Lookup: Find the California row.
   * Logic: Flag the row for Update.
     * The STATE (current) column is set to New York.
     * The Previous_State column is set to the value that was in the STATE column (California).
   * Result: A single row: CUSTOMER_ID, NAME, STATE='New York', Previous_State='California'.
Topic 8.3: Mapplets
 * Definition: A Mapplet is a reusable set of transformations.
 * Why use it? You have a complex piece of logic (e.g., a 5-step process to clean and standardize an address) that you need to use in 20 different mappings.
 * How it works:
   * You use the Mapplet Designer to build your logic.
   * You must define a Mapplet Input (to receive data) and a Mapplet Output (to send data).
   * You save the mapplet.
   * In your main Mapping, you can now drag-and-drop the Mapplet from the navigation tree. It will look like a single transformation, but it contains all 5 of your logic steps inside it.
 * Limitation: A mapplet cannot contain certain transformations (e.g., Normalizer, Target Definitions).
Topic 8.4: Performance Tuning
These are scenarios of the "Why is my job slow?" variety.
 * Identifying the Bottleneck:
   * Run the session and open the Workflow Monitor.
   * Go to the Performance Summary in the session log.
   * Check the Busy % for the Reader, Writer, and Transformation threads.
   * If Reader is 99% busy: The bottleneck is the Source. (Fix: Optimize the Source Qualifier query, add a database index).
   * If Writer is 99% busy: The bottleneck is the Target. (Fix: Use Bulk Loading, drop and recreate indexes on the target table, increase "Commit Interval").
   * If Transformation is 99% busy: The bottleneck is in your mapping logic. (This is the most common).
 * Tuning Transformations (The Fix):
   * Aggregator/Joiner/Sorter/Rank/Lookup: These are "blocking" transformations that use cache.
   * Fix: Increase the Data and Index cache sizes for these transformations in the session properties.
   * Fix: Ensure you check the "Sorted Input" checkbox for Aggregator and Joiner if you have pre-sorted the data. This is a massive performance boost.
   * Lookup: Cache the smaller table. Use a SQL override to filter the lookup, reducing its size.
   * Filter/Router: Use the Filter or Source Qualifier as early as possible in the mapping. Don't pull 100 million rows through 5 transformations and then filter out 99 million. Filter them first.
 * Pushdown Optimization:
   * A session-level setting where you tell Informatica to "push" the transformation logic down to the database.
   * Scenario: You have a mapping: Source -> Filter -> Aggregator -> Target.
   * If the Source, Filter, and Aggregator are all in the same Oracle database, Informatica can translate the entire mapping into a single SQL statement:
     INSERT INTO TARGET_TABLE (SELECT ... FROM SOURCE_TABLE WHERE ... GROUP BY ...)
   * This is the fastest possible way to run, as no data is moved to the Integration Service; the database does all the work.
Topic 8.5: Error Handling
 * Scenario: Your source file has 1,000 rows. 5 of them are "bad" (e.g., NULL primary key, or invalid data type). You don't want the whole job to fail, but you want to capture the bad rows to fix them.
 * Method:
   * In the session (Config Object tab), set the "Stop on errors" property to a high number (e.g., 1000). This tells the session not to fail.
   * In the session (Mapping tab), set up the "Error Logging" properties.
   * You can tell Informatica to write any "rejected" rows to a set of Bad Files (also called "Reject Files").
   * The session will complete successfully.
   * You will have your target table with 995 good rows, and a .bad file containing the 5 bad rows, which you can now analyze.
This completes the A-to-Z overview from scratch to intermediate. You now have the notes for all 8 modules.
What is the first scenario-based question or hands-on task you would like to practice?
