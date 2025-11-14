# TCS Wings Exam - Data Warehousing Study Notes

## 1. Introduction to Data Warehouses

### Data Warehouse Key Concepts
- **Definition**: A centralized repository that stores integrated data from multiple sources for analysis and reporting
- **Characteristics**:
  - Subject-oriented: Organized around major subjects (customers, products, sales)
  - Integrated: Data from various sources is consolidated
  - Time-variant: Historical data is maintained
  - Non-volatile: Data is stable and not frequently updated
- **Purpose**: Supports business intelligence activities, decision-making, and analytical reporting

### Importance of Data Warehouses
- Enables data-driven decision making
- Provides single source of truth for enterprise data
- Improves data quality and consistency
- Enhances query performance for analytical workloads
- Separates analytical processing from operational systems
- Supports historical analysis and trend identification
- Facilitates complex reporting and data mining

### Data Lake vs. Data Warehouse

| Aspect | Data Lake | Data Warehouse |
|--------|-----------|----------------|
| **Data Structure** | Raw, unstructured/semi-structured | Structured, processed |
| **Schema** | Schema-on-read | Schema-on-write |
| **Users** | Data scientists, advanced analysts | Business analysts, executives |
| **Processing** | ELT (Extract, Load, Transform) | ETL (Extract, Transform, Load) |
| **Cost** | Lower storage cost | Higher storage cost |
| **Flexibility** | High flexibility | Lower flexibility |
| **Purpose** | Exploration, machine learning | Reporting, BI |

### Data Mart vs. Data Warehouse

| Aspect | Data Mart | Data Warehouse |
|--------|-----------|----------------|
| **Scope** | Department/function specific | Enterprise-wide |
| **Size** | Smaller (GBs to TBs) | Larger (TBs to PBs) |
| **Complexity** | Less complex | More complex |
| **Implementation Time** | Faster (weeks to months) | Longer (months to years) |
| **Users** | Specific department | Entire organization |
| **Data Sources** | Limited sources or DW subset | Multiple enterprise sources |
| **Example** | Sales data mart, HR data mart | Complete organizational data |

### ETL Process

**ETL = Extract, Transform, Load**

1. **Extract**
   - Pull data from source systems (databases, APIs, files)
   - Handle various data formats
   - Capture incremental changes

2. **Transform**
   - Cleanse data (remove duplicates, fix errors)
   - Standardize formats and values
   - Apply business rules
   - Aggregate and calculate metrics
   - Join data from multiple sources
   - Handle missing values

3. **Load**
   - Insert transformed data into data warehouse
   - Can be full load or incremental load
   - Maintain data integrity and relationships

**Modern Alternative: ELT**
- Extract → Load → Transform
- Data loaded first, transformed in the warehouse
- Leverages cloud computing power

### Traditional Data Warehouse Options
- **Oracle Database**: Enterprise-grade, robust features
- **Microsoft SQL Server**: Windows-based, integrated with Microsoft ecosystem
- **IBM Db2 Warehouse**: Mainframe integration, enterprise scalability
- **Teradata**: High-performance analytics, parallel processing
- **SAP BW/4HANA**: In-memory computing, SAP integration
- **PostgreSQL**: Open-source option with analytical extensions

---

## 2. Dimension Modeling

### Dimensional Modeling
- **Definition**: Design technique optimized for querying and analyzing data in a data warehouse
- **Purpose**: Simplify complex database structures for better query performance
- **Key Benefits**:
  - Intuitive for business users
  - Fast query performance
  - Flexible for changing business requirements
  - Supports drill-down and roll-up operations

**Components**:
- **Fact Tables**: Contain measurable business metrics
- **Dimension Tables**: Provide context to facts

### Dimensional Modeling Alternatives

1. **Third Normal Form (3NF)**
   - Traditional relational modeling
   - Minimizes data redundancy
   - Complex queries required
   - Better for transactional systems

2. **Data Vault**
   - Auditable, scalable approach
   - Separates business keys, relationships, and descriptive data
   - Good for regulatory compliance

3. **Anchor Modeling**
   - Handles temporal data changes
   - Highly normalized
   - Complex but flexible

### Dimensional Modeling vs. Traditional Approach

| Aspect | Dimensional (Star/Snowflake) | Traditional (3NF) |
|--------|------------------------------|-------------------|
| **Purpose** | Analytical queries (OLAP) | Transactional processing (OLTP) |
| **Normalization** | Denormalized | Highly normalized |
| **Query Complexity** | Simpler joins | Complex multi-table joins |
| **Performance** | Faster for analytics | Faster for transactions |
| **Redundancy** | Some redundancy accepted | Minimal redundancy |
| **User-Friendliness** | Intuitive for business users | Technical, complex |
| **Maintenance** | Easier to modify | More complex changes |

### Dimensions

**Definition**: Descriptive attributes that provide context to facts

**Types of Dimensions**:

1. **Conformed Dimensions**
   - Shared across multiple fact tables
   - Ensures consistency (e.g., Date, Customer)

2. **Degenerate Dimensions**
   - Dimension key stored in fact table without separate dimension table
   - Example: Invoice number, Order ID

3. **Junk Dimensions**
   - Combines multiple low-cardinality flags/indicators
   - Reduces fact table columns

4. **Role-Playing Dimensions**
   - Same dimension used multiple times with different meanings
   - Example: Date dimension as Order Date, Ship Date, Delivery Date

5. **Slowly Changing Dimensions (SCD)**
   - **Type 0**: No changes allowed
   - **Type 1**: Overwrite old values (no history)
   - **Type 2**: Add new row with version (full history)
   - **Type 3**: Add new column (limited history)

**Common Dimensions**:
- Time/Date dimension
- Customer dimension
- Product dimension
- Location/Geography dimension
- Employee dimension

### Star Schema

**Structure**:
- Central fact table surrounded by dimension tables
- Direct relationship between fact and dimensions
- Denormalized dimension tables

**Characteristics**:
- Simple structure (star-shaped when visualized)
- Fewer joins required
- Faster query performance
- More storage space due to denormalization
- Easier for business users to understand

**Example**:
```
        Product Dim
              |
Customer -- FACT TABLE -- Date Dim
              |
        Location Dim
```

**Advantages**:
- Optimal query performance
- Simple to understand and navigate
- Easy to maintain

**Disadvantages**:
- Data redundancy in dimensions
- Higher storage requirements

### Snowflake Schema

**Structure**:
- Normalized version of star schema
- Dimension tables are broken into sub-dimensions
- Multiple levels of relationships

**Characteristics**:
- Dimension tables are normalized
- Reduces data redundancy
- More complex structure
- More joins required for queries

**Example**:
```
Product Category
      |
Product Subcategory
      |
Product Dim -- FACT TABLE -- Date Dim
                    |
              Store Dim
                    |
              City Dim
                    |
              State Dim
```

**Advantages**:
- Less storage space (normalized)
- Better data integrity
- Easier to maintain dimension hierarchies

**Disadvantages**:
- More complex queries (more joins)
- Slower query performance
- Less intuitive for users

**Star vs. Snowflake**:
- Star: Performance over storage
- Snowflake: Storage efficiency over performance

---

## 3. Create Your First Data Warehouse

### How to Install SQL Server

**Prerequisites**:
- Windows OS (or Linux for SQL Server on Linux)
- Minimum 6 GB hard disk space
- Minimum 1 GB RAM (4 GB recommended)

**Installation Steps**:
1. Download SQL Server Developer/Express Edition from Microsoft website
2. Run the installation executable
3. Choose installation type:
   - Basic (quick setup)
   - Custom (advanced configuration)
   - Download media only
4. Accept license terms
5. Select installation location
6. Choose instance configuration:
   - Default instance
   - Named instance
7. Configure server settings:
   - Authentication mode (Windows/Mixed)
   - Add administrators
8. Complete installation
9. Note down instance name and connection details

**Components to Install**:
- Database Engine Services
- SQL Server Replication (optional)
- Full-Text Search (optional)

### How to Connect SSMS to SQL Server

**SSMS = SQL Server Management Studio**

**Steps**:
1. Download and install SSMS from Microsoft website (separate from SQL Server)
2. Launch SSMS
3. Connection dialog appears:
   - **Server type**: Database Engine
   - **Server name**: 
     - Local: `localhost` or `.` or `(local)`
     - Named instance: `localhost\INSTANCENAME`
     - Remote: `ServerName` or `IPAddress`
   - **Authentication**:
     - Windows Authentication (default)
     - SQL Server Authentication (requires username/password)
4. Click "Connect"

**Troubleshooting**:
- Ensure SQL Server service is running
- Check firewall settings
- Verify TCP/IP protocol is enabled
- Confirm SQL Server Browser service is running (for named instances)

### How to Create a Data Warehouse Using SQL Server

**Step-by-Step Process**:

1. **Create Database**
```sql
CREATE DATABASE SalesDataWarehouse;
GO

USE SalesDataWarehouse;
GO
```

2. **Create Dimension Tables**
```sql
-- Date Dimension
CREATE TABLE DimDate (
    DateKey INT PRIMARY KEY,
    FullDate DATE,
    Year INT,
    Quarter INT,
    Month INT,
    MonthName VARCHAR(20),
    Day INT,
    DayOfWeek INT,
    DayName VARCHAR(20)
);

-- Product Dimension
CREATE TABLE DimProduct (
    ProductKey INT PRIMARY KEY IDENTITY(1,1),
    ProductID VARCHAR(50),
    ProductName VARCHAR(100),
    Category VARCHAR(50),
    SubCategory VARCHAR(50),
    Price DECIMAL(10,2)
);

-- Customer Dimension
CREATE TABLE DimCustomer (
    CustomerKey INT PRIMARY KEY IDENTITY(1,1),
    CustomerID VARCHAR(50),
    CustomerName VARCHAR(100),
    City VARCHAR(50),
    State VARCHAR(50),
    Country VARCHAR(50)
);
```

3. **Create Fact Table**
```sql
CREATE TABLE FactSales (
    SalesKey INT PRIMARY KEY IDENTITY(1,1),
    DateKey INT FOREIGN KEY REFERENCES DimDate(DateKey),
    ProductKey INT FOREIGN KEY REFERENCES DimProduct(ProductKey),
    CustomerKey INT FOREIGN KEY REFERENCES DimCustomer(CustomerKey),
    Quantity INT,
    UnitPrice DECIMAL(10,2),
    TotalAmount DECIMAL(15,2),
    Discount DECIMAL(10,2)
);
```

4. **Create ETL Procedures**
```sql
-- Example: Load Date Dimension
CREATE PROCEDURE LoadDimDate
AS
BEGIN
    DECLARE @StartDate DATE = '2020-01-01';
    DECLARE @EndDate DATE = '2025-12-31';
    
    WHILE @StartDate <= @EndDate
    BEGIN
        INSERT INTO DimDate VALUES (
            CAST(CONVERT(VARCHAR, @StartDate, 112) AS INT),
            @StartDate,
            YEAR(@StartDate),
            DATEPART(QUARTER, @StartDate),
            MONTH(@StartDate),
            DATENAME(MONTH, @StartDate),
            DAY(@StartDate),
            DATEPART(WEEKDAY, @StartDate),
            DATENAME(WEEKDAY, @StartDate)
        );
        
        SET @StartDate = DATEADD(DAY, 1, @StartDate);
    END
END;
```

5. **Create Indexes for Performance**
```sql
-- Create indexes on foreign keys
CREATE INDEX IX_FactSales_DateKey ON FactSales(DateKey);
CREATE INDEX IX_FactSales_ProductKey ON FactSales(ProductKey);
CREATE INDEX IX_FactSales_CustomerKey ON FactSales(CustomerKey);
```

6. **Set Up Backup and Maintenance**
- Configure regular backups
- Create maintenance plans
- Monitor disk space and performance

---

## 4. Modern Data Warehouses

### Cloud Data Warehouse

**Definition**: Data warehouse deployed and managed in the cloud, offering scalability and flexibility

**Key Features**:
- **Elastic Scalability**: Scale compute and storage independently
- **Pay-as-you-go**: Cost based on actual usage
- **Managed Service**: Vendor handles maintenance, updates, backups
- **Global Availability**: Access from anywhere
- **Automatic Updates**: Latest features without manual intervention
- **Built-in Security**: Encryption, access controls, compliance certifications

**Advantages**:
- No hardware investment
- Rapid deployment
- Automatic scaling
- High availability and disaster recovery
- Integration with cloud services
- Performance optimization handled by vendor

**Challenges**:
- Data transfer costs
- Network latency concerns
- Vendor lock-in
- Ongoing subscription costs
- Data sovereignty and compliance issues

### Cloud vs. On-Premises Data Warehouse

| Aspect | Cloud Data Warehouse | On-Premises Data Warehouse |
|--------|----------------------|----------------------------|
| **Initial Cost** | Low (no hardware) | High (servers, infrastructure) |
| **Operational Cost** | Subscription-based | Maintenance, staff, utilities |
| **Scalability** | Instant, elastic | Limited, requires hardware |
| **Deployment Time** | Hours to days | Weeks to months |
| **Maintenance** | Vendor-managed | In-house IT team |
| **Upgrades** | Automatic | Manual, time-consuming |
| **Performance** | Consistent, optimized | Depends on hardware investment |
| **Security** | Vendor-managed, shared responsibility | Full control, organization-managed |
| **Customization** | Limited | Extensive |
| **Disaster Recovery** | Built-in, multi-region | Requires separate investment |
| **Data Control** | Shared with vendor | Complete control |
| **Compliance** | Vendor certifications | Direct control for audits |

**When to Choose Cloud**:
- Rapid growth expected
- Limited IT infrastructure
- Need for global access
- Variable workloads
- Quick time-to-market

**When to Choose On-Premises**:
- Strict regulatory requirements
- Sensitive data concerns
- Predictable, stable workloads
- Existing infrastructure investment
- Full control requirements

### Cloud Data Warehouse Options

#### 1. **Amazon Redshift**
- **Provider**: AWS
- **Architecture**: Columnar storage, MPP (Massively Parallel Processing)
- **Features**:
  - Auto-scaling capabilities
  - Redshift Spectrum (query S3 data)
  - Integration with AWS ecosystem
  - Concurrency scaling
- **Best For**: AWS-centric organizations, large-scale analytics
- **Pricing**: Pay per node-hour

#### 2. **Google BigQuery**
- **Provider**: Google Cloud Platform
- **Architecture**: Serverless, fully managed
- **Features**:
  - Separation of compute and storage
  - Built-in machine learning (BigQuery ML)
  - Real-time analytics
  - Automatic scaling
  - Standard SQL support
- **Best For**: Ad-hoc queries, serverless architecture
- **Pricing**: Pay per query (data scanned)

#### 3. **Microsoft Azure Synapse Analytics**
- **Provider**: Microsoft Azure
- **Architecture**: Unified analytics platform
- **Features**:
  - Combines data warehousing and big data
  - Serverless and dedicated options
  - Integration with Power BI
  - Built-in security and compliance
  - Support for both SQL and Spark
- **Best For**: Microsoft ecosystem, hybrid analytics
- **Pricing**: Provisioned or serverless options

#### 4. **Snowflake**
- **Provider**: Snowflake Inc. (runs on AWS, Azure, GCP)
- **Architecture**: Multi-cluster shared data
- **Features**:
  - Complete separation of compute and storage
  - Zero-copy cloning
  - Time travel and data recovery
  - Cross-cloud data sharing
  - Automatic optimization
- **Best For**: Multi-cloud strategy, data sharing
- **Pricing**: Storage + compute credits

#### 5. **Oracle Autonomous Data Warehouse**
- **Provider**: Oracle Cloud
- **Architecture**: Self-driving, self-securing
- **Features**:
  - Automated management
  - Built-in security
  - Machine learning optimization
  - Integration with Oracle ecosystem
- **Best For**: Oracle database users, enterprise applications
- **Pricing**: OCPU and storage-based

#### 6. **IBM Db2 Warehouse on Cloud**
- **Provider**: IBM Cloud
- **Architecture**: SMP and MPP options
- **Features**:
  - In-memory processing
  - Oracle compatibility
  - Built-in analytics
  - Elastic scaling
- **Best For**: IBM ecosystem, enterprise migration
- **Pricing**: Subscription-based

**Selection Criteria**:
1. **Performance Requirements**: Query speed, data volume
2. **Cost Structure**: Storage vs. compute pricing
3. **Existing Infrastructure**: Cloud provider alignment
4. **Scalability Needs**: Growth expectations
5. **Integration Requirements**: BI tools, data sources
6. **Security and Compliance**: Industry regulations
7. **Skill Set**: Team expertise with platforms
8. **Features**: Specific capabilities needed (ML, real-time, etc.)

---

## Quick Reference Summary

### Key Exam Topics to Remember:

**Data Warehouse Fundamentals**:
- Subject-oriented, integrated, time-variant, non-volatile
- ETL vs. ELT processes
- Data lake (raw data) vs. Data warehouse (structured)
- Data mart (department) vs. Data warehouse (enterprise)

**Dimensional Modeling**:
- Fact tables (metrics) + Dimension tables (context)
- Star schema: Denormalized, faster queries
- Snowflake schema: Normalized, less storage
- SCD Types: 1 (overwrite), 2 (new row), 3 (new column)

**SQL Server Implementation**:
- Create database and tables with proper keys
- Use IDENTITY for surrogate keys
- Create foreign key relationships
- Build indexes on foreign keys for performance

**Cloud Solutions**:
- Elastic scalability and pay-as-you-go pricing
- Major platforms: Redshift, BigQuery, Synapse, Snowflake
- Cloud: Quick deployment, managed service
- On-premises: Full control, higher initial cost

**Good Luck with Your TCS Wings Exam!**