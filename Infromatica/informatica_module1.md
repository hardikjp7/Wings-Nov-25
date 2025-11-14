# MODULE 1: INFORMATICA POWERCENTER FUNDAMENTALS

## Topic 1: Introduction to ETL and Data Warehousing Concepts

### What is ETL?

**ETL = Extract, Transform, Load**

A process that:
- **Extract**: Pulls data from various source systems (databases, files, APIs, legacy systems)
- **Transform**: Cleanses, validates, applies business rules, aggregates, and formats data
- **Load**: Inserts transformed data into target systems (data warehouse, data marts, databases)

### Why ETL?

**Business Need:**
- Organizations have data scattered across multiple systems (CRM, ERP, HR, Sales)
- Need consolidated, clean data for reporting and analytics
- Source systems can't be directly queried (performance, complexity, data quality issues)

**ETL Benefits:**
- Data consolidation from heterogeneous sources
- Data quality improvement (validation, cleansing)
- Historical data preservation
- Performance optimization for reporting
- Business rule implementation

### Data Warehousing Basics

**Data Warehouse:** A centralized repository that stores integrated data from multiple sources, optimized for query and analysis.

**Key Characteristics:**
- **Subject-Oriented**: Organized around business subjects (Sales, Inventory, Customer)
- **Integrated**: Data from different sources is standardized
- **Time-Variant**: Maintains historical data
- **Non-Volatile**: Data is stable, not frequently updated

**Architecture Components:**
```
Source Systems → ETL Process → Data Warehouse → BI/Reporting Tools
     ↓              ↓                ↓                ↓
  OLTP DBs      Informatica    Star/Snowflake    Tableau/PowerBI
  Flat Files                    Schema
  APIs
```

### ETL vs ELT

| ETL | ELT |
|-----|-----|
| Transform before loading | Load first, then transform |
| Processing on ETL server | Processing on target database |
| Traditional approach | Modern cloud-based approach |
| Informatica PowerCenter | Cloud platforms (Snowflake, BigQuery) |

---

## Topic 2: Informatica PowerCenter Architecture & Components

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   INFORMATICA DOMAIN                     │
│  (Management framework for all services and nodes)       │
└─────────────────────────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                  ↓
   ┌─────────┐      ┌──────────┐      ┌──────────┐
   │Repository│      │Integration│      │  Other   │
   │ Service  │      │  Service   │      │ Services │
   └─────────┘      └──────────┘      └──────────┘
        │                 │
        ↓                 ↓
   ┌─────────┐      ┌──────────┐
   │Repository│      │PowerCenter│
   │ Database │      │  Server   │
   └─────────┘      └──────────┘
```

### Core Components Explained

#### 1. **Domain**
- Administrative framework that manages all Informatica services
- Single domain can have multiple nodes (servers)
- Domain configuration stored in domain configuration database

#### 2. **Repository Service**
- Manages the repository (metadata database)
- Handles versioning, locking, and access control
- Stores all metadata: mappings, workflows, sessions, transformations

#### 3. **Repository Database**
- Central metadata storage (Oracle, SQL Server, DB2)
- Contains:
  - Source/target definitions
  - Transformation logic
  - Mapping designs
  - Session/workflow configurations
  - Connection information

#### 4. **Integration Service** (PowerCenter Server)
- The execution engine - runs workflows and sessions
- Extracts data from sources
- Applies transformations
- Loads data to targets
- Manages memory and threads for processing

#### 5. **Client Tools** (Covered in Topic 3)
- Designer, Workflow Manager, Workflow Monitor, Repository Manager

---

### Processing Flow: How It All Works Together

**Development Phase:**
```
Developer → PowerCenter Client → Repository Service → Repository DB
         (designs mappings)      (saves metadata)    (stores metadata)
```

**Execution Phase:**
```
Scheduler/Manual Trigger → Integration Service → Reads metadata from Repository
                                ↓
                    Extracts from Source Systems
                                ↓
                    Applies Transformations (in memory)
                                ↓
                    Loads to Target Systems
                                ↓
                    Logs execution details
```

---

## Topic 3: PowerCenter Client Tools Overview

### 1. **Repository Manager**

**Purpose:** Administrative tool for repository management

**Key Functions:**
- Create and manage repositories
- Create folders (organize objects)
- Manage users, groups, and permissions
- Import/export objects
- Version control operations
- Repository backup/restore

**When to Use:**
- Setting up new projects
- Managing access control
- Organizing development work
- Migrating objects between environments

---

### 2. **Designer** ⭐ (Most Used Tool)

**Purpose:** Design ETL mappings and transformations

**Four Sub-Managers (Windows):**

#### a) **Source Analyzer**
- Import source definitions (tables, flat files, XML)
- View source metadata (columns, data types)
- Edit source properties

#### b) **Target Designer**
- Define target structures
- Create new target tables
- Import existing target definitions

#### c) **Transformation Developer**
- Create reusable transformations
- Build mapplets (reusable mapping components)
- Define expression logic

#### d) **Mapplet Designer**
- Create mapplets (groups of transformations)
- Reusable across multiple mappings

**Workspace/Mapping Designer:**
- Canvas where you design data flow
- Drag transformations
- Connect ports (columns)
- Define business logic

---

### 3. **Workflow Manager**

**Purpose:** Create and manage workflows (execution plans)

**Key Objects:**

**a) Session:**
- Execution instance of a mapping
- Contains runtime parameters:
  - Source/target connections
  - Session properties (commit intervals, error handling)
  - Performance settings

**b) Workflow:**
- Container for sessions and tasks
- Defines execution order
- Includes decision logic, email tasks, command tasks

**c) Worklet:**
- Reusable workflow component
- Group of tasks that can be reused

**d) Scheduler:**
- Schedule workflow execution
- Define frequency (daily, weekly, monthly)
- Set dependencies

---

### 4. **Workflow Monitor**

**Purpose:** Monitor and troubleshoot workflow execution

**Key Features:**
- Real-time execution status
- Session logs (detailed execution information)
- Error messages and troubleshooting
- Performance metrics:
  - Rows read/written
  - Processing speed
  - Transformation bottlenecks
- Ability to stop/abort running workflows
- Historical execution data

**Log Types:**
- **Session Log**: Overall session execution details
- **Workflow Log**: Workflow-level execution
- **Error Log**: All errors encountered
- **Performance Details**: Partition and thread-level performance

---

### 4. **Repository Manager**
(Already covered above - administrative tool)

---

## Topic 4: Understanding Repositories

### What is a Repository?

A **relational database** that stores all metadata created in Informatica PowerCenter.

**Metadata = "Data about data"**
- Not the actual business data
- Definitions, configurations, logic, connections

---

### Types of Repositories

#### 1. **Domain Configuration Repository**
- Stores domain-level information
- Service configurations
- Node details
- License information
- One per domain

#### 2. **PowerCenter Repository** (Main Focus)
- Stores all development metadata
- Can have multiple repositories in a domain
- Types:
  - **Standalone Repository**: Independent, single repository
  - **Global Repository**: Shared objects across projects
  - **Local Repository**: Project-specific, can link to global

---

### Repository Contents

**What's Stored:**

1. **Source Definitions**
   - Table structures from source databases
   - Flat file layouts
   - XML schemas

2. **Target Definitions**
   - Target table structures
   - Load specifications

3. **Transformations**
   - All transformation logic
   - Expression formulas
   - Lookup conditions

4. **Mappings**
   - Complete data flow designs
   - Connections between objects

5. **Mapplets**
   - Reusable transformation groups

6. **Sessions & Workflows**
   - Execution configurations
   - Scheduling information

7. **Connection Objects**
   - Database connections (ODBC, native)
   - FTP connections
   - Application connections

8. **Metadata Extensions**
   - Custom properties

---

### Repository Database

**Supported Platforms:**
- Oracle
- Microsoft SQL Server
- IBM DB2

**Key Tables (for reference):**
- `REP_SESS_LOG`: Session logs
- `REP_LOAD_SUMMARY`: Load statistics
- `OPB_MAPPING`: Mapping metadata
- `OPB_WIDGET`: Transformation metadata

---

### Folder Structure

Repositories are organized into **folders**:

```
Repository
  ├── Folder: HR_Project
  │     ├── Sources
  │     ├── Targets
  │     ├── Mappings
  │     └── Workflows
  │
  ├── Folder: Sales_Project
  │     ├── Sources
  │     ├── Targets
  │     ├── Mappings
  │     └── Workflows
```

**Best Practices:**
- Organize by project or business unit
- Separate development, testing, production repositories
- Use meaningful folder names
- Implement version control

---

### Repository Service

**Responsibilities:**
- Manages repository database connections
- Handles metadata locking (prevents concurrent edits)
- Manages version control
- Provides metadata to Integration Service during execution
- Handles user authentication and authorization

**High Availability:**
- Repository Service can run on multiple nodes
- Failover support for production environments

---

## KEY CONCEPTS SUMMARY FOR MCQs

### ETL Fundamentals
✅ ETL extracts data from sources, transforms it, loads to targets  
✅ Data warehouse is subject-oriented, integrated, time-variant, non-volatile  
✅ ETL processing happens on ETL server; ELT on target database  

### Architecture
✅ Domain → manages all services  
✅ Repository Service → manages metadata  
✅ Integration Service → executes workflows  
✅ Repository Database → stores metadata  

### Client Tools
✅ **Designer** → create mappings (4 sub-managers: Source Analyzer, Target Designer, Transformation Developer, Mapplet Designer)  
✅ **Workflow Manager** → create sessions and workflows  
✅ **Workflow Monitor** → monitor execution, view logs  
✅ **Repository Manager** → manage repository, users, folders  

### Repository
✅ Stores metadata, not actual data  
✅ Contains sources, targets, transformations, mappings, sessions, workflows  
✅ Organized in folders  
✅ Repository Service manages locking and versioning  

---

## PRACTICE SCENARIOS FOR MCQs

**Scenario 1:** You need to create a mapping to load customer data from Oracle to SQL Server. Which tool will you use?
- **Answer:** Designer (specifically, mapping workspace)

**Scenario 2:** Your workflow failed last night. Which tool will you check to see the error details?
- **Answer:** Workflow Monitor (session logs)

**Scenario 3:** You want to schedule a workflow to run daily at 2 AM. Which tool and object will you use?
- **Answer:** Workflow Manager → Scheduler object

**Scenario 4:** Where is the transformation logic physically stored?
- **Answer:** Repository Database (as metadata)

**Scenario 5:** Which service actually extracts data from source systems during execution?
- **Answer:** Integration Service (PowerCenter Server)

---

## NEXT STEPS

Now that you understand:
- What ETL does and why it's needed
- PowerCenter architecture and how components interact
- Client tools and their purposes
- Repository structure and contents

**You're ready to move to Module 2** where we'll start working with actual source and target definitions, and begin building transformations!

**Quick Check Questions Before We Move On:**
1. Can you explain in your own words what happens when you "run a workflow"?
2. Which tool would you open first to start developing an ETL process?
3. What's the difference between metadata and data?