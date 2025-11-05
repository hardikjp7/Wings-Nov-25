Here are the detailed notes for Module 6: Workflow Manager (Running the Jobs).
The Designer is where you create the blueprint (the Mapping). The Workflow Manager is where you build the house (execute the job). A mapping cannot be run by itself; it must be placed inside a Session Task, which is then placed inside a Workflow.
Topic 6.1: Tasks
A Task is any single object you place in the Workflow Manager. It represents one step in your process.
 * Reusable Task: A task you create once and can re-use in many different workflows (e.g., a "Send Error Email" task).
 * Non-Reusable Task: A task that is tied to one specific workflow.
Common Task Types:
 * Session Task: The most important task. Its job is to run a single mapping.
 * Command Task: Used to run any shell command or script (.bat, .sh) on the server.
   * Scenario:
     * To delete a source file after it has been loaded successfully (e.g., rm /source_files/sales_data.csv).
     * To "touch" a file (create an empty "trigger file") to signal another process to start.
 * Timer Task: Used to pause the workflow.
   * Scenario: Wait for 1 hour before checking for a file, or simply Start the workflow at a specific time (e.g., 2:00 AM).
 * Event-Wait Task: Pauses the workflow until a specific event happens.
   * Scenario: The workflow will wait at this task until a file (e.g., sales_trigger.dat) appears in a specific folder. This is used to build "file-watcher" workflows that run automatically when a source file arrives.
 * Event-Raise Task: Used to create an event that another (Event-Wait) task might be listening for.
 * Email Task: Used to send automated emails.
   * Scenario: If a session task fails, send an email to the support team with the error details.
Topic 6.2: Session Task
This is the task that connects a Mapping to the Integration Service. When you create a session, you are configuring the "run-time" properties for that mapping.
 * Purpose: To define how a specific mapping should run.
 * Key Configuration Tabs:
   * Properties Tab:
     * Logging: Set the log level (Normal, Terse, Verbose).
     * Error Handling: Stop on errors (How many errors should it hit before failing? 1 is default).
     * Parameter File: Specify a file that holds variables (e.t., $$Source_File_Name).
   * Config Object Tab:
     * Performance: Set the cache sizes (e.g., Aggregator Cache Size, Joiner Cache Size).
     * Error Handling: Define how to handle "rejected" rows (rows that fail transformation logic).
   * Mapping Tab: This is the most critical tab. This is where you connect the logical mapping to the physical world.
     * Connections: For each source and target in your mapping, you must assign a Connection Object.
       * Example: Your mapping's source is SRC_EMP. Here, you tell the session, "For SRC_EMP, use the Oracle_DEV_HR connection."
     * Target Properties:
       * Load Type: Normal (default) or Bulk (a faster, less-logged load method).
       * Insert/Update/Delete: Check the boxes for which DML operations are allowed.
       * Treat Source Rows As: This is crucial for Update Strategy. The default is Insert. To make an Update Strategy transformation work, you must set this to Data Driven.
       * Truncate Table: Check this box to TRUNCATE TABLE (delete all rows) before loading new data.
Topic 6.3: Workflows and Worklets
 * Workflow
   * Definition: A Workflow is the largest runnable object. It is a collection of one or more tasks linked together to define the order of execution.
   * Analogy: A workflow is a "project plan."
   * Example:
     * Start (Workflow start icon)
     * s_Load_Dim_Customers (Session task)
     * s_Load_Dim_Products (Session task)
     * s_Load_Fact_Sales (Session task, can only run after the two dimension loads are successful)
   * Scheduling: The workflow is the object that you schedule to run (e.g., "Run this workflow every night at 3:00 AM").
 * Worklet
   * Definition: A Worklet is a reusable group of tasks.
   * Analogy: A worklet is a "template" or "sub-routine."
   * Scenario: Imagine you have a set of 5 tasks you need to run for every country:
     * Check for file.
     * Run session.
     * Archive file.
     * On success, send email.
     * On failure, send error email.
   * Instead of building this 5-task logic 20 times (for 20 countries), you build it once inside a worklet (wklt_Standard_Load_Process). Then, in your main workflow, you just use 20 instances of that worklet.
Topic 6.4: Link Conditions (Controlling the Flow)
You connect tasks in a workflow using Links (arrows). By default, the next task runs only if the previous task succeeds. You can change this behavior.
 * How it works: Double-click the link between two tasks.
 * The Condition: The link's condition is based on the status of the previous task. The previous task has a built-in variable: $TaskName.Status
 * Common Statuses:
   * SUCCEEDED (The task ran successfully)
   * FAILED (The task failed)
 * Default Link Condition: $PreviousTask.Status = SUCCEEDED
 * Scenario-Based Logic:
   * Run-on-Failure: To run a task only if the session fails (e.g., an error email task).
     * $s_Load_Sales.Status = FAILED
   * Run-Always: To run a "cleanup" task regardless of whether the session succeeded or failed.
     * $s_Load_Sales.Status = SUCCEEDED OR $s_Load_Sales.Status = FAILED
   * Advanced Logic: You can use other variables, like $s_Load_Sales.TgtSuccessRows > 0, to run a task only if the session succeeded and actually loaded at least one row.
Would you like to move on to Module 7: Monitoring & Debugging?
