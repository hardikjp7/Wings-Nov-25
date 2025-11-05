Here are the detailed notes for Module 2: PowerCenter Architecture.
This module is crucial for understanding how the different parts of Informatica work together.
Topic 2.1: PowerCenter Components & Architecture
Informatica PowerCenter's architecture is based on a Service-Oriented Architecture (SOA). This means it's a collection of different services that work together to perform ETL tasks.
The architecture has two main parts:
 * Server Components (The "engine" that does the work)
 * Client Components (The "dashboard" you use to tell the engine what to do)
Server Architecture
The server components run on one or more machines (servers) and are collectively called the Informatica Domain.
 * Domain (Informatica Domain)
   * This is the main administrative unit. Think of it as the "master" a_dministrator_ that controls all other services and components.
   * It manages security, users, permissions, and connections between different parts.
   * A Domain is run by a Node. A node is a physical or virtual machine. You can have multiple nodes for scalability and high availability (failover).
 * Repository Service
   * This service manages the Informatica Repository.
   * The Repository is a database (like Oracle, SQL Server, etc.) that stores all the metadata created in Informatica.
   * Metadata is "data about data." It's the blueprint of your ETL, not the actual business data (like sales figures).
   * What it stores:
     * Mappings (the logic you build)
     * Workflows (the jobs you run)
     * Source and Target definitions (like table structures)
     * Database connection information
     * Session run logs
   * Scenario: When you build a mapping in the Designer and click "Save," the Repository Service is responsible for writing that mapping into the repository database.
 * Integration Service
   * This is the most important service. It is the actual ETL engine.
   * When you run a workflow, the Integration Service takes over.
   * Its job:
     * It reads the mapping logic from the repository (via the Repository Service).
     * It connects to the source systems to Extract the data.
     * It performs all the Transformations in memory (e.g., filtering, joining, calculating).
     * It connects to the target systems to Load the transformed data.
   * It creates and writes the session log files.
How they work together (Simple Flow):
 * You (the developer) use a Client Tool (like Designer) to create a Mapping.
 * You save the mapping. The Repository Service stores it in the Repository Database.
 * You tell the Integration Service to run the mapping (using a Workflow).
 * The Integration Service asks the Repository Service for the mapping metadata.
 * The Integration Service then executes the job: E, T, and L.
Topic 2.2: PowerCenter Client Tools
These are the applications you install on your local computer to interact with the Informatica Domain (Server).
 * Repository Manager
   * This is the main administrative tool for the repository.
   * Key Functions:
     * Folder Management: Create, edit, and delete folders. (Folders are used to organize your projects, e.g., "HR_Project", "Sales_Project").
     * Security: Manage users, groups, and permissions (e.g., giving a user read-only access to a folder).
     * Migration (Deployment): This is a critical function. You use it to copy objects (like mappings and workflows) from one environment to another (e.g., from Development -> Test -> Production).
 * Designer
   * This is where you spend 80% of your time as a developer.
   * It's the main canvas for building the ETL logic.
   * It has several sub-tools (called Analyzers) within it:
     * Source Analyzer: To import definitions of your source tables/files.
     * Target Designer: To import or create definitions of your target tables/files.
     * Mapping Designer: The main workspace where you drag-and-drop your sources, targets, and transformations to build the data flow logic (the mapping).
     * Mapplet Designer: To create reusable pieces of transformation logic.
 * Workflow Manager
   * A mapping (from the Designer) only defines the logic. It doesn't run it.
   * The Workflow Manager is used to execute the logic.
   * Key Objects:
     * Task: The smallest unit of work. The most important task is the Session Task.
     * Session Task: This task links a specific mapping (from the Designer) to the physical world. In the session, you define:
       * The actual database connections to use (e.g., Dev_Source_DB, Dev_Target_DB).
       * Error handling, performance settings (caching), and logging.
     * Workflow: A workflow is a collection of tasks linked together. You can link them in order (e.g., "Run Session 1, and if it succeeds, run Session 2"). This is where you build the complete job schedule.
 * Workflow Monitor
   * This is the "control room" or "dashboard" for monitoring your jobs.
   * Key Functions:
     * View running, completed, and failed jobs (workflows and sessions).
     * Check statistics: How many rows were read? How many were written? How long did it take?
     * Access and read the session logs and workflow logs to troubleshoot errors.
     * Start, stop, and abort running jobs.
Would you like to move on to Module 3: PowerCenter Designer (Building the ETL Logic)?
