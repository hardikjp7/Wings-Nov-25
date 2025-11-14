# Informatica and Tableau Quiz - Questions and Answers

## Question No. 1

You are trying to create multiple lookups in which one of them is add name in Informatica session. You do not want the add_name to return multiple rows for a single search:
a) Which of the following will you use properties can you override after the declaration in the above session?
b) which of the following

**Options:**

A. a) use a cached map item value for the lookup
b) Bulk loading map

B. a) Use a cached dynamic lookup
b) Target table name

C. a) use a workflow of the single lookup field
b) Target properties

D. a) update an empty record in the lookup to block multiple searches
b) Target db connection

**Correct Answer: B** - Use a cached dynamic lookup to prevent multiple rows from being returned, and Target table name can be overridden.

---

## Question No. 2

Which of the following view uses the text tables to display the numbers associated with dimension members?

**Options:**

A. Crosstab

B. Dashboard

C. Calculated field

D. Data Pane

**Correct Answer: A** - Crosstab view uses text tables to display numbers associated with dimension members.

---

## Question No. 3

A developer working on filters wants to limit the result set from a filter. Which of the following filters can he use for this?

**Options:**

A. Top filter

B. Multiple values filter

C. Single value filter

D. Wildcard match filter

**Correct Answer: A** - Top filter is used to limit the result set to top N values.

---

## Question No. 4

In which of the following types of filters, a filter condition is applied first to the data source, and then some other filters are used only for the resulting records?

**Options:**

A. Quick filter

B. Action filter

C. Context filter

D. Dimension filter

**Correct Answer: C** - Context filter is applied first to the data source, creating a temporary table, and then other filters are applied.

---

## Question No. 5

You are creating a Tableau report and need to use either an extract or a live connection. What is the best option available to choose an extract over a live connection?

**Options:**

A. You need the latest possible data at all times.

B. You need the data source to be only supported by a live connection via ODBC

C. You need to join the tables that are in the data source.

D. You need to apply an aggregation that takes too long when using a live connection.

**Correct Answer: D** - You need to apply an aggregation that takes too long when using a live connection. Extracts improve performance for complex aggregations.

---

## Question No. 6

Carla has been working on data aggregation on a Tableau project for which she needs to implement the table calculations. Which of the following options correctly defines the calculations?

**Options:**

A. Calculations are functions that can be used to send SQL expressions directly to the database

B. Calculations are functions that allow you to perform computations on values in a table

C. Calculations are functions that allow you to pass data directly to a running external service instance

D. Calculations are functions that can specify, include, or omit dimensions from the current view.

**Correct Answer: B** - Calculations are functions that allow you to perform computations on values in a table.

---

## Question No. 7

The table contains the data of people who prefer tea over coffee and availability of tea and coffee. Which of the following options is the best visualization to find the relation between these two parameters?

**Options:**

A. Histogram

B. Scatter Plot

C. Bar Chart

D. Density Map

**Correct Answer: B** - Scatter Plot is best for finding relationships/correlations between two parameters.

---

## Question No. 8

A developer wants to sort a graph with more than one dimension without using the combined field. Which of the following tools can she use in this case?

**Options:**

A. Dynamic sort option

B. Nested sort option

C. Multiple sort option

D. Negative sort option

**Correct Answer: B** - Nested sort option allows sorting on multiple dimensions hierarchically.

---

## Question No. 9

You have data on expired medicine from the years 2001 to 2020 of all products in pharmaceutical companies. You have to calculate the average amount of expired medicine between the years 2010 and 2015. Which of the following options can be used for this?

**Options:**

A. RUNNING_COUNT(SUM([Expired_Amount]))

B. RUNNING_AVG(SUM([Expired_Amount]))

C. AVG(SUM([Expired_Amount]))

D. RUNNING_SUM(SUM([Expired_Amount]))

**Correct Answer: C** - AVG(SUM([Expired_Amount])) calculates the average of aggregated values.

---

## Question No. 10

What is the role of "Tableau Server" in a Tableau deployment?

**Options:**

A. To develop and design dashboards and visualizations

B. To automate data refreshes at regular intervals

C. To share and collaborate on Tableau workbooks and dashboards within an organization

D. To connect to external data sources for data visualization

**Correct Answer: C** - To share and collaborate on Tableau workbooks and dashboards within an organization.

---

## Question No. 11

Sarah, a developer, requires a chart used to visualize the matrix value graphically using colors. What of the following can she use for this?

**Options:**

A. Tree Map

B. Histogram

C. Heat Map

D. Pie Chart

**Correct Answer: C** - Heat Map is used to visualize matrix values graphically using colors.

---

## Question No. 12

You need to compute values using specified dimensions in addition to whatever dimensions are present in the view. Which of the following options can you use to accomplish the task?

**Options:**

A. FIXED LOD

B. CUSTOM LOD

C. INCLUDE LOD

D. EXCLUDE LOD

**Correct Answer: C** - INCLUDE LOD computes values using specified dimensions in addition to those in the view.

---

## Question No. 13

If you are using a live connection, Tableau automatically selects the option "Auto Update Worksheet". Choose the correct option that will disable this feature.

**Options:**

A. Analysis -> Deselect the "Auto Update Worksheet" option

B. Dashboard --> Auto Updates -> Deselect the "Auto Update Worksheet" option

C. Worksheet -> Auto Updates -> Deselect the "Auto Update Worksheet" option

D. Format -> Auto Updates -> Deselect the "Auto Update Worksheet" option

**Correct Answer: C** - Worksheet -> Auto Updates -> Deselect the "Auto Update Worksheet" option.

---

## Question No. 14

A company has no sales, so they implement the following function. What will be the output of this code? ZN(SUM([SALES]))

**Options:**

A. Null

B. 0

C. It will throw an error

D. -1

**Correct Answer: B** - 0. The ZN function returns 0 if the expression is NULL.

---

## Question No. 15

A developer working on Tableau wants to use a file containing the details of a workbook. Which of the following options would help him use the required file?

**Options:**

A. Tableau workbook

B. Tableau packaged workbook

C. Tableau data source

D. Tableau packaged data source

**Correct Answer: B** - Tableau packaged workbook (.twbx) contains the workbook along with all supporting files.

---

## Question No. 16

While going through the code for a Tableau project, you found the following code snippet. What will be the output of this code snippet? FLOOR(CEILING(3.456))

**Options:**

A. 4

B. 3

C. 3.5

D. Syntax error

**Correct Answer: A** - 4. CEILING(3.456) = 4, then FLOOR(4) = 4.

---

## Question No. 17

You've been assigned the responsibility of optimizing the performance of a data warehouse system, specifically one that deals with complex analytical queries. This system often encounters lengthy processing times when handling these intricate queries. For this, you decide to use a technique that involves breaking down a query into smaller tasks that can be executed simultaneously across multiple CPU cores. Which technique should you employ to improve processing speed and efficiency in this scenario?

**Options:**

A. Data Sharding

B. Indexing

C. Parallel Processing

D. Materialized Views

**Correct Answer: C** - Parallel Processing allows breaking down queries into smaller tasks executed simultaneously across multiple CPU cores.

---

## Question No. 18

Penny, a developer, wants to display the progress of the value of a task or resource over a period. Which of the following charts should she use for this?

**Options:**

A. Bubble Chart

B. Bullet Chart

C. Crosstab Chart

D. Gantt Chart

**Correct Answer: D** - Gantt Chart displays progress of tasks/resources over a period of time.

---

## Question No. 19

A developer wants to display the cumulative effect of sequential positive and negative values. Which of the following charts should he use for this?

**Options:**

A. Bullet Chart

B. Crosstab Chart

C. Gantt Chart

D. Waterfall Charts

**Correct Answer: D** - Waterfall Charts display the cumulative effect of sequential positive and negative values.

---

## Question No. 20

As a cloud developer, Richie is implementing different types of date sets for a project. Which of the following options does not demonstrate a discrete date set?

**Options:**

A. Days in a week

B. Different days in a week

C. Different months

D. Same months in different years

**Correct Answer: D** - Same months in different years represents a continuous date set, not discrete.

---

## Question No. 21

As a site developer, Caroline is working on a project report for which she needs to use the bullet graph. For which of the following options can you use a bullet graph?

**Options:**

A. Analyzing the trend for a time period

B. Comparing the actual against the target sales

C. Adding data to bins and calculating count measures

D. Displaying the sales growth for a particular year.

**Correct Answer: B** - Comparing the actual against the target sales. Bullet graphs are designed for performance comparison against targets.

---

## Question No. 22

You are working on a report, and it requires performing the blending process. Which type of join should you use to achieve the desired results?

**Options:**

A. Right Join

B. Left Join

C. Full Join

D. Inner Join

**Correct Answer: B** - Left Join. Data blending in Tableau uses left join by default.

---

## Question No. 23

What is the purpose of the "Show Me Pane" in Tableau?

**Options:**

A. To display a list of available data sources

B. To suggest appropriate visualization types based on the selected data fields

C. To create calculated fields using predefined formulas

D. To export visualizations to other formats such as PDF or image files

**Correct Answer: B** - To suggest appropriate visualization types based on the selected data fields.

---

## Question No. 24

In Tableau, what is the purpose of a "calculated field"?

**Options:**

A. To filter data based on specific criteria

B. To perform mathematical operations on existing data fields

C. To create hierarchies within the data structure

D. To visualize geographical data on maps

**Correct Answer: B** - To perform mathematical operations on existing data fields.

---

## Question No. 25

What is a frequently used technique in data visualization that involves adding a little purposeful noise to the image to avoid overlapping and to preserve the integrity of what is communicated known as?

**Options:**

A. Blending

B. Joining

C. Jittering

D. Filtering

**Correct Answer: C** - Jittering adds small random noise to data points to avoid overlapping in visualizations.

---

## Question No. 26

What will be the best visualization to compare the profit of different products manufactured by your company?

**Options:**

A. Histogram

B. Scatter Plot

C. Bar Chart

D. Density Map

**Correct Answer: C** - Bar Chart is best for comparing categorical data like profit across different products.

---

## Question No. 27

What will be the output of the following code snippet? FINDNTH("multiplication", "i", 2)

**Options:**

A. 2

B. 8

C. 4

D. 0

**Correct Answer: B** - 8. FINDNTH finds the position of the nth occurrence of a substring. The second "i" in "multiplication" is at position 8.

---

## Question No. 28

You have been working on a project and need to specify dimensions to exclude from the view level of detail. Which of the following options will be helpful in this scenario?

**Options:**

A. FIXED LOD

B. CUSTOM LOD

C. INCLUDE LOD

D. EXCLUDE LOD

**Correct Answer: D** - EXCLUDE LOD is used to specify dimensions to exclude from the view level of detail.

---

## Question No. 29

A data analyst has to complete a project in 4 months. He divided his project into multiple tasks and wanted to schedule his tasks. Which of the following is the best visualization suited to solve the problem?

**Options:**

A. Bullet Chart

B. Heat Map

C. Gantt Chart

D. Bar Graph

**Correct Answer: C** - Gantt Chart is ideal for task scheduling and project timeline visualization.

---

## Question No. 30

Jennifer wants to open a URL within a dashboard rather than opening the system's web browser. Is it possible to deploy a URL action on a dashboard?

**Options:**

A. Yes, using the Tableau server

B. Yes, using a Tableau object.

C. Yes, using a Web Page object

D. Yes, it requires a plugin

**Correct Answer: C** - Yes, using a Web Page object. This allows embedding web content directly in the dashboard.

---

## Question No. 31

You are working on a project and are required to display measures over a period of time. Which of the following options would you use for this task?

**Options:**

A. Line Chart

B. Bar Chart

C. Histogram

D. Scatter Plot

**Correct Answer: A** - Line Chart is ideal for displaying measures over a period of time (trends).

---

## Question No. 32

You're tasked with designing a data warehouse for a multinational retail corporation. The corporation manages a diverse range of products across different categories, and it requires a data model that allows for flexible reporting and analysis. Which data modeling approach would you recommend, and why?

**Options:**

A. Implement a Dimensional Modeling approach with a Star Schema. Use dimension tables for product attributes like category, brand, and location, and a central fact table for sales transactions. This approach provides easy querying and flexibility for reporting.

B. Adopt a Snowflake Schema for better normalization. Break down dimension tables into sub-dimensions for more granular reporting. Utilize multiple fact tables to capture different aspects of sales transactions. This approach offers more detailed reporting capabilities.

C. Utilize a NoSQL data model for improved scalability and flexibility. Design document-based collections to represent products, categories, and sales transactions. Leverage document databases like MongoDB for storing and querying the data. This approach provides flexibility for evolving product structures.

D. Implement a Graph Data Model to represent the relationships between products, categories, and sales transactions. Utilize a graph database like Neo4j to store and traverse these relationships. This approach is beneficial for analyzing complex interdependencies.

**Correct Answer: A** - Implement a Dimensional Modeling approach with a Star Schema. This is the standard approach for data warehousing, providing optimal query performance and flexibility for reporting.

---

## Question No. 33

Which of the following visualizations is the least appropriate for categorical data?

**Options:**

A. Line chart

B. Pie chart

C. Bar chart

D. Combo chart

**Correct Answer: A** - Line chart is least appropriate for categorical data as it's designed for continuous data trends over time.

---

## Question No. 34

You are given the data of students that contains many tables under one schema. Which feature will help you extract the information from the tables?

**Options:**

A. Union

B. Key-column pair

C. Groups

D. Tasks

**Correct Answer: A** - Union helps combine data from multiple tables with similar structure.

---

## Question No. 35

You created a column chart with the product name and price and got a chart with the same color for all the columns. Which of the following steps should you follow to assign one color for each column?

**Options:**

A. Drag and drop the price in the color marks card and change the measure to continuous

B. Drag and drop the price in the color marks card and change the measure to discrete.

C. Drag and drop the price into colors.

D. Drag and drop the product name into colors.

**Correct Answer: D** - Drag and drop the product name into colors. This assigns a unique color to each product category.

---

## Question No. 36

Identify the file present in the bookmarks folder in the Tableau repository that contains a single worksheet.

**Options:**

A. .tbm

B. .tm

C. .tb

D. .tk

**Correct Answer: A** - .tbm (Tableau Bookmark) file contains a single worksheet.

---

## Question No. 37

Paulie wants to use a file format containing the data used in a twb file in a highly compressed columnar data format. Identify the file format that can be used for this.

**Options:**

A. Tableau data extract

B. Tableau data source

C. Tableau packaged workbook

D. Tableau workbook

**Correct Answer: A** - Tableau data extract (.tde or .hyper) uses highly compressed columnar data format.

---

## Question No. 38

Harry has implemented the filtering of data in a Tableau project. Which of the following options is not a type of filter in Tableau?

**Options:**

A. Custom

B. Context

C. Normal

D. Highlight

**Correct Answer: C** - Normal is not a type of filter in Tableau. The main filter types are Extract, Data Source, Context, Dimension, Measure, and Table Calculation filters.

---

## Question No. 39

You want to create a flexible dashboard that includes three bar charts given below. You want to select a particular department in your dashboard and have the other two sheets adjust to only view values for that department.
1. Amount by department
2. Amount by category
3. Quantity by category

Which of the following features will you use for such a requirement?

**Options:**

A. Filters

B. Parameters

C. Sets

D. Groups

**Correct Answer: A** - Filters, specifically using filter actions in dashboards, allow interactive filtering across multiple sheets.

---

## Question No. 40

Four functionalities of RANK_DENSE in Tableau are provided in the options. Which of them is the correct option?

**Options:**

A. It returns the rank for the current row in the partition. With this function, the set of values (6, 9, 9, 14) would be ranked (3, 2, 2, 1).

B. It returns the rank for the current row in the partition. With this function, the set of values (6, 9, 9, 14) would be ranked (0.25, 0.75, 0.75, 1.00).

C. It returns the rank for the current row in the partition. With this function, the set of values (6, 9, 9, 14) would be ranked (4, 2, 3, 1).

D. It returns the rank for the current row in the partition. With this function, the set of values (6, 9, 9, 14) would be ranked (4, 3, 3, 1).

**Correct Answer: A** - RANK_DENSE ranks values (6, 9, 9, 14) as (3, 2, 2, 1) in descending order, with no gaps for tied values.githu