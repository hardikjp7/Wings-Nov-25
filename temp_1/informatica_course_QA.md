# Informatica PowerCenter Quiz - Questions and Answers

## Question No. 1

You need to extract data from many source systems, load it onto the destination system, and then convert the data

Will the ETL tool be useful here?

**Options:**

A. No, ETL will follow the below steps
1. Extract data from a different source
2. Transforming the data is not possible
3. Load data to the target system
This is followed in ETL, not ELT

B. No, ETL will follow the below steps
1. Extract data from a different source
2. Transform the data
3. Load data to the target system
This is followed by ETL, not ELT

C. Yes, ETL will follow the below steps
1. Extract data from a different source
2. Transform the data
3. Load data to the target system
This is followed in ETL, not ELT

D. No, ETL will follow the below steps:
1. Extract data from a different source
2. Transform the data
3. Load data to the source system
This is followed in ETL, not ELT

**Correct Answer: C** - Yes, ETL will follow the Extract → Transform → Load steps, which matches the requirement to extract, load, and convert data.

---

## Question No. 2

You are given the below table:

Source: TableA
- ID: 1, Name: A
- ID: null, Name: B

Lookup: TableB
- ID: 4, Loc: Bang
- ID: null, Loc: Hyd

You are using the lookup policy on multiple matches "allow the first row", what will be the output rows? Will the null record pass through the lookup or not?

**Options:**

A. Two nulls are not the same when the data type is decimal

B. Two nulls are not the same when datatype is an integer

C. Two nulls are not the same when datatype is a string

D. No, two nulls are not same

**Correct Answer: D** - No, two nulls are not same. In Informatica, NULL values are not considered equal to each other in lookups.

---

## Question No. 3

Suppose we configure Mapping with a non-reusable sequence generator populating key-value SEQ ID for the target table

The following properties set in Sequence Generator and Mapping are executed for two runs:
- Number of Cached Values = 1000
- Current Value = 101
- Increment By = 1

The first run of Mapping loaded 500 records and the second run loaded 600 records

Which of the below values can we find in the SEQ ID column in the target table?

**Options:**

A. 599 & 1699

B. 1028702

C. 356 8756

D. 61081156

**Correct Answer: A** - 599 & 1699. First run: 101-600 (500 records), but 1000 values cached. Second run: starts at 1101, loads 600 records (1101-1700), so last value is 1700.

---

## Question No. 4

Assume you are using incremental aggregation, in which of the below cases the integration service creates a new aggregate cache:

1. New version of mapping saved
2. The Previous run was not succeeded
3. Aggregate files were deleted
4. Number of Partitions decreased
5. Session Configured to reinitialize the cache
6. Number of Partitions increased

**Options:**

A. 1,2,3,4,5 & 6

B. 1,3,4 & 5

C. 1,2,3,4 & 5

D. 3,4,5,6

**Correct Answer: A** - 1,2,3,4,5 & 6. All these conditions cause the Integration Service to create a new aggregate cache.

---

## Question No. 5

Which one of the below options is correct regarding the limitations of joiner transformation?

**Options:**

A. You cannot use joiner transformation when the input pipeline contains stored procedure transformation.

B. You cannot use joiner transformation when the input pipeline contains Update Strategy transformation.

C. You cannot use joiner transformation when the input pipeline contains SQL transformation

D. You cannot use joiner transformation when the input pipeline contains RANK transformation

**Correct Answer: B** - You cannot use joiner transformation when the input pipeline contains Update Strategy transformation.

---

## Question No. 6

Which transformation in Informatica PowerCenter is used for performing aggregate functions like sum, average, and count?

**Options:**

A. Joiner Transformation

B. Filter Transformation

C. Aggregator Transformation

D. Rank Transformation

**Correct Answer: C** - Aggregator Transformation is used for performing aggregate functions.

---

## Question No. 7

While configuring Lookup Transformation, which one of the below "field options" would be displayed ONLY when the Lookup transformation uses a dynamic cache?

**Options:**

A. Edit native field metadata.

B. Synchronize icon

C. Ignore in Comparison

D. Retain existing fields at runtime

**Correct Answer: C** - Ignore in Comparison is displayed only when using a dynamic cache.

---

## Question No. 8

An organisation uses Dataddo and Informatica together to do ETL processes. Which of the following options would be available for each source that would be registered?

**Options:**

A. You can join both tables in source qualifier (SQ) as they belong to the same database

B. You need two source qualifiers for fetching data from two tables

C. You need three source qualifiers for fetching data from two tables.

**Correct Answer: B** - You need two source qualifiers for fetching data from two tables (one for each source system).

---

## Question No. 9

You have a table with field name "DAY," then the session will fail because SQL standards don't allow using "DAY" as the field name. Which of the below options is the optimal way to resolve this issue?

**Options:**

A. Change the field name in DB

B. Change the field Name in informatica Transformation

C. Create a reserved word file in the Integration Service installation directory

D. Use Target Override query

**Correct Answer: C** - Create a reserved word file in the Integration Service installation directory to handle reserved keywords.

---

## Question No. 10

Consider the scenario in which you must load data to a table accessed for Reporting the whole day. Which one of the below options could not be implemented to improve the Performance of Load?

**Options:**

A. Creation of new Partition at Source DB

B. Creation of New Partitions in Informatica Session

C. Creation of New Connection String with New User Id for Loading Data

D. Change the load type to Bulk mode

**Correct Answer: C** - Creation of New Connection String with New User Id for Loading Data does not improve load performance.

---

## Question No. 11

After placing a source qualifier in mapping, you need to filter the data based on the age column. Therefore, you set the filter condition as "where Age >= 18". What will be returned here?

**Options:**

A. You cannot add a condition like this in SQ

B. It will throw an error

C. The session will fail at run time

**Correct Answer: B** - It will throw an error. The correct syntax should not include "where" keyword in SQ filter condition.

---

## Question No. 12

When you are loading data from the source, you use the extract to fetch the data, and then load it to the target. What types of extracts are there?

**Options:**

A. There are two types of extracts:
1. Full extract
2. Logical extract

B. There are two types of extracts:
1. Full extract
2. Physical extract

C. There are two types of extracts:
1. Full extract
2. Offline extract

D. There are two types of extracts:
1. Full extract
2. Incremental extract

**Correct Answer: D** - There are two types: Full extract and Incremental extract.

---

## Question No. 13

You need to gather information about the file transmission and reception running or completed in a system. Which option can help you configure this?

**Options:**

A. Data logs

B. Source control logs

C. File transfer logs

**Correct Answer: C** - File transfer logs help track file transmission and reception.

---

## Question No. 14

You have to join the sales table with the salesman table and find out the list of salesman whose sales cross the 100000 limits. Can you use the condition "sale > 100000" in joiner transformation?

**Options:**

A. Yes, the greater than symbol can be used inside the Joiner transformation

B. No, the greater than symbol cannot be used inside the Joiner transformation.

C. Yes, a greater than equal symbol can be used inside the Joiner transformation if any of the sources is a flat file.

D. No, greater than symbol can be used inside the Joiner transformation.

**Correct Answer: B** - No, the greater than symbol cannot be used inside the Joiner transformation. Joiner only supports equality conditions (=). Use Filter transformation after joining.

---

## Question No. 15

While Configuring Dynamic cache, which one of the below options do you need to set for multiple match policy?

**Options:**

A. Return Error

B. Return Any Row

C. Return First Row

D. Return Last Row

**Correct Answer: D** - Return Last Row is the policy set for multiple match in dynamic cache.

---

## Question No. 16

Consider a scenario where you must populate Target Table data from 5 Flat files, 3 Oracle Tables, and 2 DB2 Tables. Assume all Flat Files have different structures and are placed in the same source folder & Tables are placed in the Same Schema. What would be the minimum number of Joiner Transformation required in your mapping to fulfill the requirement?

**Options:**

A. 07

B. 05

C. 06

D. 09

**Correct Answer: D** - 09 joiners. You need 9 joiner transformations to combine all 10 sources (n-1 joiners for n sources).

---

## Question No. 17

Assume you copy a session from Folder A to Folder B in the Workflow Manager Tool. Which one of the below options will hold TRUE?

1. Session and associated mapping will be Copied to Destination Folder B
2. Session will get copied only if associated mapping exists in Destination Folder B
3. Session will get copied if you assign a new mapping in Folder B
4. Session will get copied along with connection objects if an associated mapping exists in Destination Folder B

**Options:**

A. 2,3 & 4

B. 1,2 & 3

C. 2 & 4

D. 2 & 3

**Correct Answer: D** - 2 & 3. Session will be copied only if the associated mapping exists in the destination folder, or if you assign a new mapping.

---

## Question No. 18

Which of the following transformations in Informatica PowerCenter is used to combine data from two or more sources based on a specified condition?

**Options:**

A. Source Qualifier Transformation

B. Joiner Transformation

C. Expression Transformation

D. Rank Transformation

**Correct Answer: B** - Joiner Transformation is used to combine data from multiple sources.

---

## Question No. 19

Consider the scenario where your client asked to display their sales value in the specified currencies, and you configured the below Parameter at Informatica:

- Workflow level parameter $CUR=USD
- First Session level parameter $CUR=EUR
- Second Session level parameter $CUR=
- Third Session level parameter $CUR=
- Fourth Session level parameter $CUR=INR

Now you received an ad-hoc request from the client to populate output for the first & Second session in EUR only for today's Data. What Changes do you need to make to achieve this?

**Options:**

A. Update the Second Session Parameter Currency to EUR. Run the workflow revert the changes after run

B. Update the Workflow Parameter CUR to EUR, Run the workflow and revert the change after run

C. Unschedule daily workflow, Create new parameter file and define Parameter as per client requirement and run the job through pmcmd command Reschedule the workflow

D. Update Both Workflow and second Session level Parameter CUR to EUR, Run the workflow and revert the change after run

**Correct Answer: C** - Unschedule daily workflow, Create new parameter file and define Parameter as per client requirement and run the job through pmcmd command, then Reschedule the workflow. This is the best approach for ad-hoc requests without affecting the regular scheduled workflow.

---

## Question No. 20

You need to do Vertical join 2 Datasets. Dataset A contains 5 Fields and B Contain 3 fields. How will you achieve this in Informatica?

**Options:**

A. Connect First 3 Ports of A & 3 Ports of B to Union Transformation

B. Connect 3 Ports of matching Data Type of A & 3 Ports of B to Union Transformation

C. Add Dummy Ports to Both A & B and make it Structurally Similar, Then Do union

D. Add Dummy Ports only to B and do Union

**Correct Answer: D** - Add Dummy Ports only to B and do Union. Union transformation requires both inputs to have the same structure.

---

## Question No. 21

Consider a scenario where you must delete all duplicate records from table "A" using Informatica and keep unique records intact. Assume you are using table A as source and target. Which of the below option help you to accomplish the requirement?

**Options:**

A. 1. Configure "Treat Source rows" as Delete in Session Properties
2. Filter out the "ID" of Duplicate record Using Aggregator and Filter
3. Link the "ID" to Target

B. 1. Configure "Treat Source rows" as Data driven in Session Properties
2. Filter out the "ID" of Duplicate record Using Aggregator and Filter
3. Mark the ID with Flag "DD_DELETE" Using Update Strategy Transformation Link the "ID" to Target

C. 1. Configure "Treat Source rows" as Data driven in Session Properties
2. Filter out the "ID" of Duplicate record Using Aggregator and Filter
3. Mark the ID with Flag "DD_DELETE" Using Update Strategy Transformation Link the "ID" to Target
4. Use Target update Override query

D. 1. Configure session with Delete query using pseudocolumn in Pre_SQL or Post SQL
2. Design a Mapping which not passes any record to Target

**Correct Answer: B** - Configure as Data driven, filter duplicates, mark with DD_DELETE flag using Update Strategy, and link to target.

---

## Question No. 22

MD5 (Message Digest Function) is a hash function in Informatica used to evaluate data integrity. Assume you have to check the integrity of 10 columns. Which one of the below options will help you to check the integrity?

**Options:**

A. MD5(COL1,COL2,COL3,...COL10)

B. MD5(COL1), MD5(COL2)...MD5(COL10)

C. MD5(COL1+COL2+...+COL10)

D. MD5(COL1||COL2||COL3...||COL10)

**Correct Answer: D** - MD5(COL1||COL2||COL3...||COL10). Concatenate all columns and then apply MD5 function.

---

## Question No. 23

In a mapping, you have placed source qualifier, you need to filter the data based on age column, so you give filter condition as "where Age >= 18". What will be returned here?

**Options:**

A. You can not add condition like this in SQ

B. Yes, it will throw an error

C. Session will fail at run time

D. You can add this where source is Salesforce

**Correct Answer: B** - Yes, it will throw an error. The "where" keyword should not be included in SQ filter condition; only the condition itself.

---

## Question No. 24

John wants to pass output ports First Pipeline Expression Transformation "A" and Second Pipeline Expression Transformation "B" to New Expression Transformation "C". Which one of the below options is true?

**Options:**

A. He Can Directly Drag and drop ports from A & B to C and Link the Ports

B. He Needs to insert Joiner Before C

C. He needs to Pass all the ports of B to A First, Then only he can able to Get all Ports in C

D. He needs to put aggregator Before C

**Correct Answer: B** - He Needs to insert Joiner Before C. You cannot connect two separate pipelines directly to one transformation without a Joiner.

---

## Question No. 25

Which one of the given options is correct regarding Joiner Transformation and Lookup Transformation in a mapping?

**Options:**

A. Joiner Transformation can able to replace Lookup Transformation in a mapping by performing all the functions of Lookup Transformation.

B. Lookup Transformation can able to replace Joiner Transformation in a mapping by performing all the functions of Joiner Transformation

C. Joiner Transformation could not able to do self Join while Lookup Transformation could not able to do Full outer Join

D. Joiner Transformation could not able to do Unequal Join while Lookup Transformation could not able to do Right outer Join

**Correct Answer: D** - Joiner Transformation cannot do Unequal Join (only equality joins) while Lookup Transformation cannot do Right outer Join.

---

## Question No. 26

While configuring Lookup Transformation, if the lookup condition matches null values and the input field is also NULL, which one of the below options holds for mapping task evaluation of Null value?

**Options:**

A. Null Value would be equal to Null Value in the Lookup

B. Ignore the Null Value

C. Mapping Task will fail with NULL Error

D. Null Value would be considered Not equal to the Null Value of Lookup

**Correct Answer: D** - Null Value would be considered Not equal to the Null Value of Lookup. NULL values are not considered equal in Informatica.

---

## Question No. 27

In Informatica PowerCenter, which transformation is used to identify the top N records based on a specified criteria within each group?

**Options:**

A. Sorter Transformation

B. Aggregator Transformation

C. Filter Transformation

D. Rank Transformation

**Correct Answer: D** - Rank Transformation is used to identify top N records.

---

## Question No. 28

Constraint-based load ordering is used to load the data into a parent table and child tables. Which of the below options is incorrect regarding Constraint-based load ordering?

**Options:**

A. Constraint based load ordering option applies for only insert operations using the constraint based load ordering option in the Config Object tab of the session

B. You cannot update or delete the rows using the constraint based load ordering.

C. We have to define the primary key and foreign key relationships for the targets in the warehouse or target designer to use Constraint based load ordering

D. The target tables must be in the same Target connection group to Use Constraint based load ordering

**Correct Answer: A** - This is incorrect. Constraint-based load ordering can be applied for insert, update, and delete operations, not just inserts.

---

## Question No. 29

You can schedule workflow using Informatica scheduler. Which one of the given options is not correct about the Informatica scheduler?

**Options:**

A. You can schedule the workflow to run continuously and run forever

B. You can manually execute the workflow which is in scheduled state

C. You can create multiple instances of workflow and schedule each workflow instances to run at different times

D. You can unschedule the workflow while current instance of workflow is in running state

**Correct Answer: D** - You cannot unschedule the workflow while the current instance is in running state.

---

## Question No. 30

You need to perform a mapping that is configured for pushdown optimization to preview the SQL query. The result then gets configured in the database. What is the correct step to be followed to perform the mapping?

**Options:**

A. Give the new mapping a different name, and then apply your updates to the new mapping

B. Perform a test run of the valid mapping to verify the results of the mapping before you create a mapping task.

C. Make the runtime environment active, which runs the preview job

**Correct Answer: B** - Perform a test run of the valid mapping to verify the results before creating a mapping task.

---

## Question No. 31

You can join, filter, and sort multiple tables through SQL Override in Source Qualifier Transformation. What would be the disadvantage of Doing SQL Override in Source Qualifier Transformation?

**Options:**

A. Slow Performance Compared to Native SQ query

B. Impact Analysis for Changes in associated Table requires manual effort

C. SQL Override query will run as a single thread and not get partitioned

D. SQL Override query could not optimized using Pushdown Optimization

**Correct Answer: B** - Impact Analysis for Changes in associated Table requires manual effort, as SQL overrides are not automatically tracked by Informatica.

---

## Question No. 32

Which among the given options will hold true for optimal performance of Joiner Transformation?

1. For an Unsorted Joiner transformation, designate the source with fewer rows as the Master source
2. For an Unsorted Joiner transformation, designate the source with fewer columns as the Master source
3. For a sorted Joiner transformation, designate the source with fewer duplicate key values as the Detail source
4. For a sorted Joiner transformation, designate the source with fewer duplicate key values as the Master source

**Options:**

A. 1 & 4

B. 1,2 & 4

C. 1,3 & 4

D. 2 & 4

**Correct Answer: C** - 1,3 & 4. For unsorted joiner, use fewer rows as master. For sorted joiner, use fewer duplicate keys as master and more duplicates as detail.

---

## Question No. 33

Emma wants to use different tools in her window in order to create and edit repository objects, such as sources, targets, mapplets, transformations, and mappings. Which option should be used by her in this case?

**Options:**

A. You can restore the session using the session attributes called session recovery, which is a complete load record rather than a full record since your session failed in the middle of the workflow run

B. Because your session crashed in the middle of running the workflow, you can recover it by loading the partial record rather than the entire record using the session recovery option in the session properties

C. Create an email task and use the workflow variable to send mail

**Correct Answer:** None of the options provided are relevant to the question. The correct answer should be "Designer" tool, which is used to create and edit repository objects.

---

## Question No. 34

Which of the options below is ideal for change data capture and load incremental data?

**Options:**

A. Storing Maximum load Time Value as a Variable and Process the records with timestamp greater than Variable Time

B. Using Control Table to store Time Value and Process the records based on Time

C. Update the records in source with Processed flag

D. Compare the records of Source and Target using HASH function

**Correct Answer: B** - Using Control Table to store Time Value and Process the records based on Time is the most robust and standard approach for CDC.

---

## Question No. 35

Assume you must migrate 10-year data archive table data with a Volume of 100 TB from one DB to another, How will you achieve this through Informatica without impacting your typical day-to-day process?

**Options:**

A. Create a One to One Mapping and load the data in Single Full load

B. Create One to One mapping with Source Filter Having date/Key Range and Load the data in automated batches & Load Balancing

C. Migrate through Third Party tools

D. Migrate only the Recent data as Full data migration impact daily load process

**Correct Answer: B** - Create One to One mapping with Source Filter Having date/Key Range and Load the data in automated batches & Load Balancing. This ensures minimal impact on daily processes.

---

## Question No. 36

Assume your source flat file has 5 Fields, and you must segregate the duplicate records from unique records without aggregator transformation, How will you achieve this in Informatica?

**Options:**

A. Using Sorter Transformation & Joiner Transformation

B. Using Sorter & Rank Transformation

C. Using Sorter & Expression Transformation

D. Using Sorter & Lookup Transformation

**Correct Answer: C** - Using Sorter & Expression Transformation. Sort the data and use Expression with variable ports to identify duplicates.

---

## Question No. 37

Assume you have a requirement to merge 10 files of data into a single master file using the minimum number of objects in mapping? What would be the minimum number of Informatica objects required to achieve this?

**Options:**

A. 03

B. 02

C. 04

D. 05

**Correct Answer: A** - 03 objects: Source Definition, Union Transformation (to combine all sources), and Target Definition.

---

## Question No. 38

Consider a scenario where you need to fetch missing customer data details by doing a Look-up on your OLD Customer archive files. You have 10 customer archive Flat files based on year. How will you achieve this efficiently through Lookup Transformation in Informatica?

**Options:**

A. Create 10 Connected Lookup Transformation and execute mapping and execute once

B. Create 10 Unconnected Lookup Transformation and call the lookup till fetch the customer details

C. Create 1 Connected Lookup Transformation with Indirect Load Type

D. Create 1 Connected Lookup Transformation and parameterize the file Name

**Correct Answer: C** - Create 1 Connected Lookup Transformation with Indirect Load Type, which allows you to specify multiple source files dynamically.

---

## Question No. 39

John wants to load a few records to a Table. Using one of the below Applications. Which application will help him to load data?

**Options:**

A. Designer

B. Repository Manager

C. Workflow Manager

D. Workflow Monitor

**Correct Answer: C** - Workflow Manager is used to create and run workflows that load data. (Designer creates mappings, Workflow Manager executes them).

---

## Question No. 40

You need to change the associated mapping of a session from Mapping "A" to Mapping "B". How will you achieve this efficiently with minimal impact?

**Options:**

A. Delete the Mapping "A" in designer and associate mapping "B" to the session in workflow manager

B. Create new session from the mapping B

C. Make a copy of session and change the mapping component from "A" to "B"

D. Make the mapping "A" invalid, by adding new target. Refresh the session and associate session with New mapping "B", then Revert back the mapping "A" Changes

**Correct Answer: C** - Make a copy of session and change the mapping component from "A" to "B". This is the most efficient approach with minimal impact.