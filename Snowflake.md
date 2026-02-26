
# Interview-Ready Answers for Snowflake — 40+ Questions

---

## Category 1: Snowflake Basics (Always Asked)

**1. What is Snowflake? Why is it used?**

Snowflake is a cloud-based data warehousing platform built natively for the cloud. Unlike traditional data warehouses that run on dedicated hardware, Snowflake runs entirely on cloud infrastructure — AWS, Azure, or GCP. It separates storage and compute which means you can scale them independently. You only pay for what you use. It supports structured and semi-structured data like JSON, Avro, and Parquet natively. It requires zero infrastructure management — no tuning, no indexes, no partitioning decisions needed by the user. As a data engineer I use Snowflake because it handles massive scale effortlessly, supports concurrent users without performance degradation, and integrates easily with modern data stack tools like dbt, Airflow, and Fivetran.

---

**2. What is the architecture of Snowflake?**

Snowflake has a three-layer architecture. The Storage Layer stores all data in Snowflake's internal optimized columnar format on cloud object storage like S3. Data is automatically compressed, encrypted, and organized. The Compute Layer consists of Virtual Warehouses which are independent clusters of compute resources that execute queries. Multiple warehouses can access the same storage simultaneously without competing with each other. The Cloud Services Layer is the brain of Snowflake — it handles query optimization, metadata management, authentication, access control, and infrastructure management. This separation of storage and compute is what makes Snowflake fundamentally different and more scalable than traditional warehouses like Redshift or Teradata.

---

**3. What is a Virtual Warehouse in Snowflake?**

A Virtual Warehouse is a cluster of compute resources in Snowflake that executes queries, loads data, and performs DML operations. It is essentially the CPU and memory that processes your queries. Warehouses come in different sizes from X-Small to 6X-Large — each size doubles the compute power of the previous. You can start and stop warehouses anytime and you are only charged when a warehouse is running. Multiple warehouses can query the same data simultaneously without any contention because storage is separate from compute. You can also enable auto-suspend to automatically stop a warehouse after a period of inactivity and auto-resume to start it automatically when a query arrives.

---

**4. What is the difference between Snowflake and traditional data warehouses like Redshift or Teradata?**

Traditional data warehouses tightly couple storage and compute on dedicated hardware. Scaling requires buying more hardware. Snowflake separates storage and compute completely allowing independent scaling. In Redshift adding compute means adding nodes which also adds storage even if you don't need it — Snowflake avoids this waste. Snowflake requires almost zero administration — no vacuum, no analyze, no index management. Redshift requires regular maintenance. Snowflake's multi-cluster warehouse handles concurrency automatically while Redshift struggles with many concurrent users. Snowflake natively handles semi-structured data like JSON while Teradata and older Redshift versions require complex workarounds. Snowflake also offers unique features like time travel, data sharing, and zero-copy cloning that traditional warehouses don't have.

---

**5. What are the different editions of Snowflake?**

Snowflake has four editions. Standard edition provides core features including all SQL support, time travel up to 1 day, and basic security. Enterprise edition adds multi-cluster warehouses, time travel up to 90 days, materialized views, and column-level security — this is what most large companies use. Business Critical edition adds enhanced security features like HIPAA compliance, data encryption everywhere, and private connectivity for industries like healthcare and finance. Virtual Private Snowflake is a completely isolated environment for organizations with the highest security requirements. For most data engineering interviews knowing Standard and Enterprise differences is sufficient.

---

**6. What is the difference between databases, schemas, and tables in Snowflake?**

Snowflake follows a three-level hierarchy. A Database is the top level container that holds all objects. Inside a database you have Schemas which are logical groupings of tables, views, and other objects — similar to folders. Inside schemas you have Tables, Views, Stages, Pipes, and other objects. Best practice is to have separate databases for raw, staging, and production data and separate schemas within each database for different business domains. For example you might have a database called PROD with schemas called SALES, FINANCE, and HR each containing their respective tables.

---

**7. What is a Snowflake Stage? What are the types?**

A Stage in Snowflake is a location where data files are stored before being loaded into tables or after being unloaded from tables. It is essentially a pointer to a file storage location. There are two main types. Internal Stages are storage locations managed by Snowflake itself — User Stage is automatically created for each user, Table Stage is automatically created for each table, Named Stage is explicitly created and shared across users. External Stages point to external cloud storage like AWS S3, Azure Blob, or GCP Storage. In data pipelines I use External Stages pointing to S3 to load data from upstream systems into Snowflake using COPY INTO command.

---

**8. What is the COPY INTO command in Snowflake?**

COPY INTO is the primary command for bulk loading data into Snowflake tables from staged files. It is significantly faster than row-by-row INSERT statements because it loads data in bulk using Snowflake's optimized parallel loading. You can load from internal or external stages, specify file formats like CSV, JSON, or Parquet, handle errors with ON_ERROR option, and transform data during load using SELECT within COPY INTO. Once files are loaded Snowflake tracks them and will not reload the same file again unless you use FORCE option. This is very useful in pipelines to prevent duplicate loads.

```sql
COPY INTO employees
FROM @my_s3_stage/employees/
FILE_FORMAT = (TYPE = 'CSV' FIELD_DELIMITER = ',' SKIP_HEADER = 1)
ON_ERROR = 'CONTINUE';
```

---

## Category 2: Snowflake Key Features (Most Important)

**9. What is Time Travel in Snowflake?**

Time Travel allows you to access historical data — data that has been changed or deleted — within a defined retention period. You can query data as it existed at any point in the past, restore tables or schemas to a previous state, and create clones from historical data. Standard edition retains data for 1 day. Enterprise edition allows up to 90 days. You query historical data using AT or BEFORE clause with a timestamp, offset in seconds, or a query ID.

```sql
-- Query data as it was 1 hour ago
SELECT * FROM employees AT (OFFSET => -3600);

-- Query data as it was at specific timestamp
SELECT * FROM employees AT (TIMESTAMP => '2024-01-01 10:00:00'::TIMESTAMP);

-- Restore a dropped table
UNDROP TABLE employees;
```
Time Travel is extremely useful for recovering from accidental deletes or debugging data issues in production.

---

**10. What is Zero Copy Cloning in Snowflake?**

Zero Copy Cloning creates an exact copy of a database, schema, or table instantly without physically copying the underlying data. The clone shares the same storage as the original — only new changes made to either the original or clone result in additional storage. This is incredibly useful for creating development or testing environments from production data without duplicating storage costs or waiting for data to copy. Creating a clone of a 10TB table takes seconds and costs nothing extra until you start modifying data in the clone. I use this in my pipelines to create test environments and for creating backup snapshots before major data transformations.

```sql
-- Clone a table instantly
CREATE TABLE employees_backup CLONE employees;

-- Clone entire database for testing
CREATE DATABASE prod_clone CLONE production_db;
```

---

**11. What is Data Sharing in Snowflake?**

Data Sharing allows you to share live data from your Snowflake account with other Snowflake accounts without copying or moving the data. The data consumer queries your data directly in real time. There is no data duplication, no ETL, and data is always up to date. The provider creates a Share object, grants access to specific databases and tables, and adds consumer accounts. Consumer creates a database from the share and immediately has read access. This is very powerful for sharing data between business units, partners, or customers. Snowflake Marketplace is built on top of this feature where companies publish data products for others to subscribe to.

---

**12. What is Snowpipe? How does it work?**

Snowpipe is Snowflake's continuous data ingestion service that loads data automatically as soon as new files arrive in a stage. Unlike COPY INTO which you run manually or on a schedule, Snowpipe is event-driven — when a new file lands in S3 it triggers a notification which tells Snowpipe to load it immediately. This enables near real-time data loading with latency of just a few minutes. Snowpipe uses serverless compute managed by Snowflake so you don't need a running virtual warehouse for ingestion. You pay per second of compute used. In modern pipelines Snowpipe combined with Kafka or Fivetran is the standard way to ingest streaming or frequent batch data into Snowflake.

---

**13. What are Streams in Snowflake?**

A Stream in Snowflake is a Change Data Capture object that tracks all DML changes — inserts, updates, and deletes — made to a table. It records what changed, what the old value was, and what the new value is. Streams are used to build incremental data pipelines — instead of reprocessing the entire table you only process the rows that changed since the last time you consumed the stream. Each row in a stream has metadata columns: METADATA$ACTION showing INSERT or DELETE, METADATA$ISUPDATE showing if it was an update, and METADATA$ROW_ID. Once you consume a stream by selecting from it inside a DML transaction the stream advances its offset and those changes are marked as consumed.

```sql
-- Create stream on a table
CREATE STREAM employees_stream ON TABLE employees;

-- Query changes
SELECT * FROM employees_stream;

-- Process only new/changed records
INSERT INTO employees_audit
SELECT * FROM employees_stream WHERE METADATA$ACTION = 'INSERT';
```

---

**14. What are Tasks in Snowflake?**

Tasks are Snowflake objects that execute SQL statements or stored procedures on a schedule. They are Snowflake's built-in job scheduler. You can schedule tasks using CRON expressions or simple minute/hour intervals. Tasks can be chained together to create DAG-like pipelines where one task triggers the next. Tasks can also be triggered by streams — when a stream has new data a task automatically wakes up and processes it. This combination of Streams and Tasks is Snowflake's native way to build lightweight ELT pipelines without external orchestration tools.

```sql
-- Create a task that runs every hour
CREATE TASK process_new_orders
WAREHOUSE = compute_wh
SCHEDULE = 'USING CRON 0 * * * * UTC'
AS
INSERT INTO orders_summary
SELECT * FROM orders_stream WHERE METADATA$ACTION = 'INSERT';

-- Start the task
ALTER TASK process_new_orders RESUME;
```

---

**15. What are Materialized Views in Snowflake?**

A Materialized View is a precomputed result set stored as a database object — unlike regular views which run the underlying query every time you query them. Snowflake automatically maintains materialized views in the background and keeps them up to date when underlying data changes. They are useful for expensive aggregation queries that are run frequently. Querying a materialized view is much faster because the heavy computation is already done. However they consume additional storage and maintenance compute. Materialized Views are available in Enterprise edition and above. Best use case is dashboards and reports that query the same complex aggregations repeatedly.

---

## Category 3: Snowflake Performance and Optimization

**16. What is the Query Result Cache in Snowflake?**

Snowflake automatically caches the results of every query for 24 hours. If the exact same query is run again within that period and the underlying data has not changed Snowflake returns the cached result instantly without using any compute — so it is completely free. This is particularly useful for dashboards that refresh periodically with the same queries. The cache is maintained at the Cloud Services layer so it persists even if your virtual warehouse is suspended. You can disable result cache at session level for testing but in production you want it enabled. This is one reason why Snowflake feels so fast for repeated analytical queries.

---

**17. What is the Metadata Cache in Snowflake?**

Snowflake maintains a metadata cache at the Cloud Services layer that stores information about micro-partitions, min/max values per column, row counts, and null counts for every table. This allows Snowflake to answer many queries using only metadata without scanning actual data — for example COUNT(*), MIN, MAX on a table. It also enables partition pruning — Snowflake uses metadata to determine which micro-partitions to skip entirely for a given filter. This metadata cache is always available and doesn't require a running warehouse for metadata-only operations.

---

**18. What is a Micro-Partition in Snowflake?**

Micro-partitions are the fundamental storage unit in Snowflake. All data is automatically divided into micro-partitions which are contiguous units of storage each containing between 50MB to 500MB of uncompressed data. Data within each micro-partition is stored in columnar format and compressed. Snowflake automatically tracks metadata for each micro-partition including min and max values for every column, number of distinct values, and null counts. This metadata enables partition pruning — when you filter on a column Snowflake skips all micro-partitions where the filter value cannot exist based on min/max values. This is automatic and requires no manual partitioning decisions from the user unlike other databases.

---

**19. What is Clustering in Snowflake? How is it different from traditional partitioning?**

Clustering in Snowflake refers to how data is organized across micro-partitions on disk. When you insert data in order of a column like date the data is naturally clustered — all micro-partitions for a given date are together and partition pruning works very well. Over time as data is inserted in random order clustering degrades and more micro-partitions need to be scanned. Automatic Clustering is a Snowflake feature that continuously maintains clustering on specified columns in the background. You define cluster keys on columns frequently used in WHERE and JOIN clauses. Unlike traditional partitioning in other databases Snowflake's clustering is automatic, online, and does not require downtime. It is recommended only for very large tables where query performance has degraded.

```sql
-- Define cluster key on a table
ALTER TABLE orders CLUSTER BY (order_date, region);
```

---

**20. How do you optimize query performance in Snowflake?**

First check if the warehouse size is appropriate for the query — complex queries on large data benefit from larger warehouses. Use result cache by ensuring identical queries reuse cached results. Check clustering — if filtering on a column frequently consider adding a cluster key for large tables. Avoid SELECT * and only select needed columns — Snowflake is columnar so selecting fewer columns reads less data. Use filters early to enable micro-partition pruning. Avoid functions on filter columns as they prevent pruning. Use materialized views for frequently run expensive aggregations. Check for cartesian joins or missing join conditions. Use the Query Profile in Snowflake UI to identify the most expensive steps in your query.

---

**21. What is Multi-Cluster Warehouse in Snowflake?**

Multi-Cluster Warehouse allows a virtual warehouse to automatically scale out by adding more clusters when concurrent query demand increases and scale in by removing clusters when demand decreases. Each cluster is a complete copy of the warehouse's compute resources. When many users query simultaneously instead of queuing behind each other they get routed to different clusters. This solves the concurrency problem — in traditional warehouses too many concurrent users slow everyone down. Multi-Cluster Warehouse is available in Enterprise edition. You set minimum and maximum cluster count and Snowflake handles scaling automatically. This is why Snowflake is popular for BI tools where many dashboard users query simultaneously.

---

**22. What is Query Profile in Snowflake and how do you use it?**

Query Profile is a visual execution plan available in Snowflake's web UI that shows exactly how a query was executed — every operation, how long it took, how many rows were processed, and where the bottlenecks are. You open it from the Query History page after a query runs. Key things to look for are the most expensive nodes by time and bytes processed, whether partition pruning happened and how many partitions were scanned versus total, whether a spillage to disk occurred which means warehouse needs to be larger, and whether result cache was used. I use Query Profile regularly to diagnose slow queries and it is the most important tool for query optimization in Snowflake.

---

## Category 4: Snowflake Data Loading and Transformation

**23. What file formats does Snowflake support for loading?**

Snowflake supports CSV and TSV as delimited text formats, JSON for semi-structured data, Avro which is a row-based binary format, ORC which is a columnar format from Hadoop ecosystem, Parquet which is the most popular columnar format, and XML. For most data engineering pipelines Parquet is the recommended format because it is columnar, compressed, and retains schema information. JSON is useful for loading API data or event data. Snowflake can load all of these using COPY INTO command with appropriate FILE_FORMAT specification.

---

**24. How do you load JSON data into Snowflake?**

Snowflake stores JSON natively in a VARIANT column which can hold any semi-structured data. You load JSON files using COPY INTO into a table with a VARIANT column and then use colon notation to query specific fields.

```sql
-- Create table with VARIANT column
CREATE TABLE raw_events (data VARIANT);

-- Load JSON from stage
COPY INTO raw_events FROM @my_stage/events/
FILE_FORMAT = (TYPE = 'JSON');

-- Query specific fields from JSON
SELECT
  data:user_id::STRING as user_id,
  data:event_type::STRING as event_type,
  data:timestamp::TIMESTAMP as event_time,
  data:properties:page_name::STRING as page_name
FROM raw_events;
```
The double colon is for casting and colon notation navigates nested JSON. FLATTEN function is used to explode arrays within JSON. In my pipelines I load raw JSON into VARIANT columns first and then transform into structured tables for analytics.

---

**25. What is the FLATTEN function in Snowflake?**

FLATTEN is a table function that explodes arrays or nested objects in semi-structured data into multiple rows. When you have a JSON field that contains an array and you want one row per array element you use FLATTEN. It is equivalent to explode in PySpark or UNNEST in other SQL databases.

```sql
-- JSON with array: {"order_id": 1, "items": ["phone", "case", "charger"]}
SELECT
  o.data:order_id::INT as order_id,
  f.value::STRING as item
FROM orders o,
LATERAL FLATTEN(input => o.data:items) f;
```
This gives one row per item per order. LATERAL keyword means the FLATTEN references columns from the same FROM clause. This is very commonly used when working with event data or API data in Snowflake.

---

**26. What is dbt and how does it work with Snowflake?**

dbt stands for data build tool. It is a transformation framework that allows data engineers and analysts to write SQL SELECT statements and dbt handles creating tables and views in the data warehouse. You write models as SELECT queries, define tests to validate data quality, document your data lineage, and dbt compiles everything and runs it against Snowflake. dbt works exceptionally well with Snowflake — it uses Snowflake's warehouses for compute, supports Snowflake-specific features like dynamic tables and clustering, and integrates with Snowflake's information schema for lineage. In modern data teams dbt on Snowflake has become the standard transformation layer replacing stored procedures and complex ETL code.

---

## Category 5: Snowflake Security and Access Control

**27. What is RBAC in Snowflake?**

RBAC stands for Role Based Access Control. Snowflake uses RBAC as its primary security model. Instead of granting privileges directly to users you grant privileges to roles and then assign roles to users. This makes access management scalable and auditable. Snowflake has built-in system roles: ACCOUNTADMIN is the most powerful role for account level management, SYSADMIN manages databases and warehouses, SECURITYADMIN manages users and roles, USERADMIN creates users and roles, and PUBLIC is the default role all users have. Best practice is to create custom roles for each team or function, grant only necessary privileges, and assign those roles to users. Never use ACCOUNTADMIN for day-to-day work.

---

**28. What is the difference between ACCOUNTADMIN, SYSADMIN, and SECURITYADMIN?**

ACCOUNTADMIN is the top level role that can do everything — manage billing, create and delete any object, view all data. It should be used rarely and only by senior administrators. SYSADMIN manages all database objects — creates databases, schemas, tables, warehouses, and grants privileges on these objects. Most data engineering work is done under SYSADMIN or custom roles granted by SYSADMIN. SECURITYADMIN manages users, roles, and grants — it can create roles and assign them to users but cannot access data objects unless SYSADMIN privileges are granted to it. In practice data engineers typically work under a custom role granted by SYSADMIN, never directly as ACCOUNTADMIN.

---

**29. What is Row Level Security and Column Level Security in Snowflake?**

Row Level Security allows you to restrict which rows a user can see in a table based on their role or attributes. Snowflake implements this through Row Access Policies — you create a policy that defines which rows are visible based on the querying user's role and attach it to a table. Column Level Security allows masking sensitive columns based on role. You create a Masking Policy that defines how a column value should be shown — for example showing full SSN to HR role but showing only last 4 digits to other roles. Both features are available in Enterprise edition and are essential for GDPR and data privacy compliance. These are policy-based and apply automatically without changing application queries.

---

**30. What is Network Policy in Snowflake?**

A Network Policy allows you to control which IP addresses can connect to your Snowflake account. You define an allowed list of IP addresses or CIDR ranges and an optional blocked list. When a network policy is applied users can only connect from the specified IP addresses — any connection attempt from an unlisted IP is rejected. This is important for security — for example you might restrict Snowflake access to only your office network or VPN IP ranges. Network policies can be applied at account level affecting all users or at individual user level for more granular control.

---

## Category 6: Snowflake Advanced Features

**31. What are Dynamic Tables in Snowflake?**

Dynamic Tables are a recent Snowflake feature that automatically maintain a materialized query result and keep it up to date with a defined freshness target. Unlike regular materialized views that update immediately on every change, Dynamic Tables let you define how fresh the data needs to be — for example within 1 minute or 1 hour. Snowflake determines the most efficient way to refresh them automatically. They can be chained together to build entire transformation pipelines declaratively — you just define what you want and Snowflake figures out when and how to update each table. This is an alternative to dbt + Airflow for simpler pipeline patterns and is becoming very popular in modern Snowflake architectures.

---

**32. What are External Tables in Snowflake?**

External Tables allow you to query data files stored in external cloud storage like S3 directly as if they were Snowflake tables — without loading the data into Snowflake. The data stays in your cloud storage and Snowflake reads it at query time. External Tables are useful when you have data that you only query occasionally and do not want to pay for Snowflake storage, when you need to keep data in your own cloud storage for compliance, or when multiple systems need to access the same files. However they are significantly slower than native Snowflake tables because data is not in Snowflake's optimized columnar format. They support partitioning on columns derived from file path patterns.

---

**33. What are Stored Procedures in Snowflake?**

Stored Procedures in Snowflake allow you to write procedural logic — loops, conditional statements, error handling — that goes beyond what pure SQL can do. Snowflake supports stored procedures written in JavaScript, Python, Java, Scala, and Snowflake Scripting which is SQL-based procedural language. They are useful for complex multi-step transformations, data validation workflows, and administrative tasks. Stored procedures in Snowflake run with caller's rights or owner's rights — owner's rights means the procedure executes with the privileges of the procedure's owner regardless of who calls it, which is useful for controlled access patterns.

---

**34. What are UDFs in Snowflake?**

UDF stands for User Defined Function. Snowflake allows you to create custom functions in SQL, JavaScript, Python, Java, or Scala that can be called within SQL queries just like built-in functions. Scalar UDFs return one value per row. Table UDFs or UDTFs return a set of rows per input row. UDFs are useful for complex business logic that is reused across many queries, data transformations that are hard to express in SQL, and calling external libraries for specialized processing like ML model inference. Python UDFs can import third-party libraries from Anaconda making them very powerful for data science use cases.

---

**35. What is Snowflake Marketplace?**

Snowflake Marketplace is a platform where data providers can publish and monetize data products and consumers can discover and access third-party datasets directly in their Snowflake account. It is built on Snowflake's Data Sharing technology — consumers get live access to provider's data without any copying or ETL. Marketplace has thousands of datasets across categories like financial data, weather, demographics, geospatial data, and more. Many datasets are free, some are paid. This is very powerful for enriching your internal data with external context — for example joining your sales data with external weather data or economic indicators from Marketplace without building any data pipelines.

---

**36. What is a Snowflake Task Tree or DAG of Tasks?**

Snowflake Tasks can be chained together to form a directed acyclic graph where one task triggers another after it completes. The first task in the chain is the root task which has a schedule. All subsequent tasks are child tasks that have no schedule but instead have a parent task defined. When root task completes it triggers all direct children and so on down the tree. This allows you to build multi-step data pipelines entirely within Snowflake without external orchestration. For example: first task loads raw data, second task transforms it, third task loads aggregated summary, fourth task sends a notification. This is Snowflake's native lightweight alternative to Airflow for simpler pipeline patterns.

---

## Category 7: Snowflake Data Engineering Scenarios

**37. How do you implement SCD Type 2 in Snowflake?**

SCD Type 2 maintains full history of changes by adding new rows for each change with start and end dates. In Snowflake I implement this using MERGE statement with Streams to capture changes.

```sql
MERGE INTO dim_customer AS target
USING customer_changes_stream AS source
ON target.customer_id = source.customer_id
  AND target.is_current = TRUE
WHEN MATCHED AND (target.email != source.email OR target.address != source.address)
  THEN UPDATE SET
    target.end_date = CURRENT_DATE(),
    target.is_current = FALSE
WHEN NOT MATCHED THEN INSERT (
  customer_id, email, address, start_date, end_date, is_current
) VALUES (
  source.customer_id, source.email, source.address,
  CURRENT_DATE(), NULL, TRUE
);
```
The challenge is that closing old record and inserting new record needs to happen atomically — in Snowflake you wrap both in a transaction or use a two-step approach.

---

**38. How do you handle incremental loading in Snowflake?**

For incremental loading I use Streams to capture changes from source tables and Tasks to process them on a schedule. The Stream tracks all inserts, updates, and deletes since last consumption. A Task wakes up on schedule, reads the stream, and applies changes to the target table using MERGE. For external data I use Snowpipe for continuous loading or COPY INTO with a watermark timestamp approach — loading only files or records newer than the last successful load timestamp. I track load metadata in a control table recording last load timestamp, rows loaded, and status for every pipeline run.

---

**39. How do you monitor and debug pipelines in Snowflake?**

Snowflake provides several tools for monitoring. Query History page shows all queries with execution time, bytes scanned, credits used, and status. Query Profile shows visual execution plan for any query. Task History shows execution history of all tasks including successes and failures with error messages. Snowpipe ingestion history tracks all file loads. Information schema and Account Usage schema have views for programmatic monitoring — QUERY_HISTORY, TASK_HISTORY, PIPE_USAGE_HISTORY, WAREHOUSE_METERING_HISTORY. I also set up TASK_FAILURE alerts using Snowflake's notification integrations to send email or Slack alerts when a pipeline task fails. For cost monitoring I check WAREHOUSE_METERING_HISTORY to track credit consumption by warehouse and time.

---

**40. What is the difference between VARIANT, ARRAY, and OBJECT data types in Snowflake?**

All three are semi-structured data types. VARIANT is the most flexible — it can hold any value including strings, numbers, booleans, arrays, objects, or NULL. It is the go-to type for loading raw JSON. ARRAY is specifically for ordered lists of values — equivalent to a JSON array. OBJECT is for key-value pairs — equivalent to a JSON object or dictionary. In practice most people just use VARIANT for everything because it is flexible and Snowflake handles the underlying type automatically. You use specific ARRAY and OBJECT types when you want to enforce that a column always contains an array or always contains an object. All three support the same colon notation for querying nested values.

---

**41. How do you handle errors during data loading in Snowflake?**

COPY INTO command has several ON_ERROR options. ABORT_STATEMENT is the default — it aborts the entire load if any error occurs. CONTINUE skips error rows and loads the rest — good for tolerating bad data. SKIP_FILE skips the entire file if it has errors. SKIP_FILE_num skips a file if more than a specific number of errors occur. After loading you can query VALIDATE function or check LOAD_HISTORY to see which rows failed and why. I always implement error handling in pipelines by loading into a staging table first, validating data quality, and only then moving to production table. Failed records go into an error table for investigation and reprocessing.

---

**42. What is Snowflake Credit and how is billing calculated?**

A Snowflake Credit is the unit of compute consumption. Credits are consumed when virtual warehouses are running — one credit per hour per node in the warehouse. An X-Small warehouse is 1 node consuming 1 credit per hour. Each size up doubles the credit consumption — Small is 2 credits per hour, Medium is 4, Large is 8, and so on. You are billed per second with a minimum of 60 seconds per start. Storage is billed separately based on average compressed data stored per month. Cloud Services layer usage is free if it stays below 10% of daily compute credit consumption — above that it is charged. Snowpipe has its own credit consumption separate from virtual warehouses. Understanding credits is important for cost optimization in production environments.

---

**43. What is the difference between TRANSIENT and TEMPORARY tables in Snowflake?**

Regular tables have full Time Travel retention and Fail-Safe protection — they retain historical data for up to 90 days and are protected by Snowflake's 7-day fail-safe period after that. This means maximum 97 days of data protection but also maximum storage cost. Transient tables have Time Travel up to 1 day and no Fail-Safe protection — they cost significantly less to store and are good for intermediate staging data that does not need long history. Temporary tables exist only for the duration of the current session — they are automatically dropped when the session ends, have no Fail-Safe, and are good for truly temporary work within a session. In ETL pipelines I use Transient tables for staging data to save on storage costs since we do not need long history for intermediate tables.

---

## Quick Revision Cheat Sheet

| Topic | Priority |
|---|---|
| Architecture — 3 layers | 🔴 Must Know |
| Virtual Warehouse | 🔴 Must Know |
| Time Travel | 🔴 Must Know |
| Zero Copy Cloning | 🔴 Must Know |
| Snowpipe | 🔴 Must Know |
| Streams and Tasks | 🔴 Must Know |
| COPY INTO | 🔴 Must Know |
| Micro-partitions | 🟠 Important |
| Result Cache | 🟠 Important |
| RBAC and Security | 🟠 Important |
| Materialized Views | 🟠 Important |
| Data Sharing | 🟠 Important |
| Dynamic Tables | 🟡 Good to Know |
| External Tables | 🟡 Good to Know |
| Multi Cluster Warehouse | 🟡 Good to Know |
| Snowflake Credits | 🟡 Good to Know |

---

## Top 5 Most Likely Asked in Wipro Interview

1. Explain Snowflake architecture and how storage and compute are separated
2. What is Time Travel and Zero Copy Cloning — give real use cases
3. What is the difference between Snowpipe and COPY INTO
4. What are Streams and Tasks and how do you use them together
5. How do you optimize a slow query in Snowflake

---

## One Pro Tip for Snowflake Interviews

Whenever they ask about any Snowflake feature always mention **a real use case** from your experience. For Time Travel say "I used this when someone accidentally deleted records in production." For Zero Copy Cloning say "I clone production to staging environment before testing major pipeline changes." This shows practical knowledge not just theory and separates you from candidates who only memorized definitions.

**You are fully prepared. Go get that offer! 💪**
