Here are the detailed notes for Module 1: Informatica & Data Warehousing Foundations.
Topic 1.1: What is Data Warehousing (DWH)?
A Data Warehouse (DWH) is a central repository of integrated data from one or more different sources. It stores current and historical data in one place. The primary goal is not to run daily operations but to support Business Intelligence (BI) activities, such as reporting, analytics, and data mining, to help users make decisions.
Key Concepts
 * OLTP (Online Transaction Processing)
   * This is the system that captures your day-to-day transactions.
   * Example: A bank's ATM system, a retail store's checkout (POS) system, a website's order entry.
   * Goal: Fast data writing (INSERT, UPDATE, DELETE). Optimized for recording transactions quickly.
   * Data: Highly normalized, current, and detailed.
 * OLAP (Online Analytical Processing)
   * This is the Data Warehouse system.
   * Example: A system that answers, "What was the total sales for our top 5 products in the North region for each of the last 4 quarters?"
   * Goal: Fast data reading (SELECT). Optimized for complex queries and analysis.
   * Data: Denormalized, historical, and summarized.
| Feature | OLTP (Operational System) | OLAP (Data Warehouse) |
|---|---|---|
| Primary Goal | Run the business | Analyze the business |
| Query Type | Simple, fast transactions (e.g., get one customer) | Complex, analytical queries (e.g., sum of sales) |
| Data | Current, real-time | Historical, aggregated |
| Database Design | Normalized (to reduce data redundancy) | Denormalized (to speed up queries) |
| Users | Front-line workers (e.g., cashier, bank teller) | Analysts, Managers, Executives |
Data Modeling (Schema)
In a data warehouse, data is modeled to be easy to understand and fast to query.
 * Dimension (Dimension Table)
   * It describes the "who, what, where, when" of your data.
   * It contains descriptive, textual attributes.
   * Example:
     * Product Dimension: Product_ID, Product_Name, Category, Brand.
     * Time Dimension: Date_Key, Day, Month, Quarter, Year.
     * Customer Dimension: Customer_ID, Customer_Name, City, State, Country.
 * Fact (Fact Table)
   * It describes the "what happened" in measurable terms.
   * It contains numeric, additive values (called measures or facts).
   * It is surrounded by keys from the Dimension tables (called Foreign Keys).
   * Example:
     * Sales Fact: Product_ID (FK), Time_ID (FK), Customer_ID (FK), Sales_Amount (Measure), Quantity_Sold (Measure).
 * Star Schema
   * The most common and simplest DWH schema.
   * It consists of one central Fact Table connected to multiple Dimension Tables.
   * It looks like a star, with the Fact table at the center.
   * Advantage: Simple queries, fast performance.
 * Snowflake Schema
   * An extension of the Star Schema.
   * The Dimension Tables are normalized into multiple, smaller related tables.
   * Example: The Product dimension might be split into a Product table and a Category table (Product -> Category).
   * Advantage: Saves storage space (less redundancy).
   * Disadvantage: More complex queries (requires more joins), which can be slower.
Topic 1.2: What is ETL?
ETL stands for Extract, Transform, Load. It is the process that all data warehouses use to get data from source systems and put it into the warehouse.
 * Extract:
   * The process of reading and pulling data from various source systems.
   * Sources: Databases (Oracle, SQL Server), Flat Files (.csv, .txt), Mainframes, Web Services (APIs), etc.
   * Challenge: Sources can be in different formats, locations, and technologies.
 * Transform:
   * The most critical and complex part. This is where the data is cleaned, standardized, and business logic is applied.
   * Common Transformations:
     * Cleaning: Removing duplicates, handling nulls (e.g., NULL -> N/A), fixing spelling.
     * Standardizing: Making data consistent (e.g., USA, U.S.A., United States all become USA).
     * Enriching: Combining data from multiple sources (e.g., JOINing customer and sales data).
     * Calculating: Creating new data (e.g., Revenue = Quantity * Price).
     * Aggregating: Summarizing data (e.g., SUM(Sales) grouped by Region).
 * Load:
   * The process of writing the transformed data into the final target system, which is usually the Data Warehouse (Fact and Dimension tables).
   * Load Types:
     * Initial Load (Full Load): Loading all the data for the first time.
     * Incremental Load (Delta Load): Loading only the new or changed data since the last load. This is much more common.
Topic 1.3: Introduction to Informatica PowerCenter
 * What is it? Informatica PowerCenter is one of the most popular and powerful ETL tools on the market.
 * What does it do? It provides a graphical interface (GUI) to visually design and run ETL processes (called mappings and workflows).
 * Why use a tool like this?
   * Instead of writing thousands of lines of complex SQL or code (like Python) to perform ETL, you can use a drag-and-drop interface.
   * It can connect to almost any data source.
   * It's highly scalable and designed for high-performance processing of huge data volumes.
   * It provides powerful features for error handling, scheduling, and monitoring.
For your TCS exam, the scenario-based questions will test your understanding of how to use PowerCenter's features (especially transformations) to solve a given Transform (T in ETL) problem.
Would you like to move on to Module 2: PowerCenter Architecture?
