Here are the detailed notes for Module 3: PowerCenter Designer (Building the ETL Logic).
This module covers the main tool you will use to build your ETL logic. The Mapping is the core blueprint for data movement and transformation.
Topic 3.1: Source Analyzer
 * Purpose: To connect to and import the structure (metadata) of your source data. You are not importing the data itself, only the definition of "what it looks like."
 * How it works:
   * You open the Source Analyzer in the Designer.
   * You choose your source type.
 * Common Source Types:
   * Relational Database: You connect to a database (e.g., Oracle, SQL Server) and import the table definition (column names, data types, primary/foreign key relationships).
   * Flat File: You import a file (e.g., .csv, .txt, .dat). The Designer starts a wizard where you must define:
     * Delimited: The file uses a character to separate columns (e.g., a comma ,, a pipe |, or a tab).
     * Fixed-Width: Each column occupies a specific, fixed number of characters in each row.
     * You also define if the first row contains column headers.
Topic 3.2: Target Designer
 * Purpose: To define the structure of your final destination table or file.
 * How it works:
   * Import: Just like the Source Analyzer, you can connect to a target database and import an existing table's structure.
   * Create: You can also use the Target Designer to create a new target definition from scratch. This is common when your target table doesn't exist yet, and you want Informatica to create it for you the first time the workflow runs.
   * From Source: You can also drag a Source definition into the Target Designer to create a matching Target definition.
Topic 3.3: Mapping Designer
 * Purpose: This is the most important workspace. It's the visual "canvas" where you build the end-to-end flow of data from source to target.
 * Definition: A Mapping is the visual representation of the ETL logic, showing the flow of data from Source Definitions, through Transformations, to Target Definitions.
 * Key Components of a Mapping:
   * Source Definition: An instance of a source (from the Source Analyzer). This is where the data comes from.
   * Target Definition: An instance of a target (from the Target Designer). This is where the data goes.
   * Transformations: These are the objects that change the data. Each transformation has a specific job (e.g., Filter, Aggregate, Join).
   * Links: These are the "pipes" that connect the ports (columns) from one object to another, defining the path the data will follow.
Topic 3.4: Source Qualifier (SQ) Transformation
 * What is it? When you drag any Relational (database) or Flat File source into the Mapping Designer, Informatica automatically creates a Source Qualifier (SQ) transformation and connects it.
 * This is a critical transformation. It has two main jobs:
   * It is the object that physically connects to the source database or file and extracts the data.
   * It converts the source's native data types (e.g., an Oracle "VARCHAR2") into the standard "Informatica" data types that all other transformations in the mapping will use.
Key Properties (For Scenario-Based Questions)
The Source Qualifier is an Active Transformation. You can configure its properties to optimize and filter data before it even enters the mapping pipeline. This is highly efficient.
 * Source Filter:
   * What it does: Lets you write a WHERE clause to filter data at the source database level.
   * Example: SALARY > 50000 AND DEPT_ID = 10
   * Scenario: Why use this instead of a Filter transformation? Because the database filters the data before sending it to Informatica. If you have 100 million rows and only need 1,000, it's better to let the database filter them instead of pulling all 100 million rows into Informatica and then filtering them.
 * User-Defined Join:
   * What it does: Allows you to join two or more tables from the same source database using SQL.
   * Scenario: If you have EMPLOYEES and DEPARTMENTS tables in the same Oracle database, you can join them here.
   * Advantage: This join is performed by the source database, which is highly optimized for joins, rather than pulling both tables into Informatica and using a Joiner transformation.
 * SQL Query (Query Override):
   * What it does: This allows you to completely replace Informatica's default SELECT statement with your own custom SQL query.
   * Example: You could write a query with a GROUP BY, HAVING, or ORDER BY clause.
   * Scenario: This is a very common technique. If you need to sort the data before it enters the pipeline (e.g., for an Aggregator), you can add an ORDER BY clause here.
   * Rule: When you write a custom query, you must ensure the columns in your SELECT statement are in the exact same order as the ports in the Source Qualifier transformation.
 * Number of Sorted Ports:
   * What it does: If your SQL Query override includes an ORDER BY clause, you must tell the Source Qualifier how many of the first columns are sorted.
   * Example: If your query is SELECT EMP_ID, EMP_NAME, DEPT_ID FROM EMP ORDER BY DEPT_ID, EMP_ID, you would set "Number of Sorted Ports" to 3 (or 2, depending on the requirement for downstream transformations). This informs other transformations that the data is arriving pre-sorted.
Would you like to move on to Module 4: Core Transformations?
