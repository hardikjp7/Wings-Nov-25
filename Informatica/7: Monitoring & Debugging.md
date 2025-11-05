Here are the detailed notes for Module 7: Monitoring & Debugging.
This module covers the tools and techniques used to find errors, check job status, and troubleshoot performance.
Topic 7.1: Using the Workflow Monitor
 * Purpose: The Workflow Monitor is the primary tool for real-time and historical monitoring of job execution. It's a read-only "dashboard."
 * What you can do:
   * View Job Status: See which workflows/sessions are running, succeeded, failed, or aborted.
   * Start/Stop/Abort: You can right-click a running workflow or session to stop (waits for tasks to finish) or abort (kills immediately) it.
   * View Statistics: Check the row-level statistics for a completed session.
   * Get Logs: Access the session and workflow logs.
 * Key Views:
   * Gantt Chart View: A visual timeline of your job. It's useful for seeing how long each task took and which tasks ran in parallel. This is the default view.
   * Task View: A hierarchical list of all jobs that have run. This view is better for sorting and finding specific, older job runs.
Topic 7.2: Understanding Session Logs and Workflow Logs
This is the most important part of troubleshooting. When a job fails, the log is the only place to find out why.
1. Workflow Log
 * What it is: A high-level log of the workflow's execution.
 * What it contains:
   * Which tasks (sessions, commands) started.
   * The final status of each task (Succeeded, Failed).
   * Evaluation of link conditions (e.g., "$s_Load_Data.Status = SUCCEEDED, running next task...").
   * Any workflow-level variables.
 * Use this log to: See which task in the workflow failed. It will not tell you why the task failed.
2. Session Log
 * What it is: A highly detailed log of a single session task's execution. This is where you find the errors.
 * What it contains (Key Sections):
   * Initialization: Details about the session setup, connecting to the source/target, and cache sizes.
   * Transformation Statistics: A summary of how many rows passed through each transformation.
   * Error Messages: The actual error. (e.g., ORA-00904: "SAL_AMT": invalid identifier means a column name is wrong, or Primary key violation means you tried to insert a duplicate key).
   * Source/Target Summary: A high-level count of rows.
   * Performance Summary: A final breakdown of the job, showing Reader, Writer, and Transformation thread busy times. This is used for identifying bottlenecks.
 * Example Error (What to look for):
   Message Code: WRT_8229
   Message: Database errors occurred: ORA-01400: cannot insert NULL into ("SCHEMA"."EMPLOYEES"."EMPLOYEE_ID")
   * Interpretation: The mapping tried to insert a row where the EMPLOYEE_ID was NULL, but the target table defines this column as "Not Null."
Topic 7.3: Debugger
 * Purpose: A tool inside the Designer (not the Monitor) that lets you run your mapping step-by-step and row-by-row to find logic errors.
 * How it works:
   * You set up the debugger with a specific session (so it knows which database to use).
   * You set "Breakpoints" on transformations. A breakpoint is a "stop sign" that tells the debugger to pause execution.
   * You start the debugger, and it processes one row at a time.
   * It will pause at your breakpoint, and you can inspect the data in the "Debugger" window to see the value of every port (column) at that exact point in the flow.
 * Scenario:
   * You have a complex IIF condition in an Expression.
   * IIF(SALARY > 10000 AND COMM_PCT > 0.1, 'HIGH', 'LOW')
   * You think a row should be flagged 'HIGH', but it's coming out 'LOW'.
   * Solution: Set a breakpoint on the Expression. Run the debugger. When that row hits the transformation, you can look at the SALARY and COMM_PCT values to see why your logic is failing.
Would you like to move on to the final module, Module 8: Intermediate/Advanced Topics (Hands-On & Scenarios)?
