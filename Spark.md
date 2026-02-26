
# Interview-Ready Answers for Spark & PySpark

---

## Category 1: Spark Basics (Always Asked)

**1. What is Apache Spark? Why is it used?**

Apache Spark is an open-source distributed computing framework designed for fast processing of large scale data. It processes data in memory rather than writing intermediate results to disk like Hadoop MapReduce did — this makes it 10 to 100 times faster. It supports batch processing, real time streaming, machine learning, and graph processing all in one unified framework. As a data engineer I use Spark when data is too large to process on a single machine and needs to be distributed across a cluster.

---

**2. What is the difference between Spark and Hadoop MapReduce?**

Hadoop MapReduce writes intermediate data to disk after every step which makes it very slow for multi-step jobs. Spark keeps intermediate data in memory making it significantly faster. Spark also has a much simpler and expressive API compared to MapReduce. Spark supports streaming, SQL, machine learning all natively while MapReduce only does batch processing. However Hadoop HDFS is still commonly used as storage layer underneath Spark. Think of it as Spark replaced MapReduce as the processing engine but HDFS is still used for storage.

---

**3. What are the main components of Spark ecosystem?**

Spark Core is the base engine that handles distributed task scheduling, memory management, and fault recovery. Spark SQL allows querying structured data using SQL or DataFrame API. Spark Streaming handles real time data processing. MLlib is Spark's machine learning library. GraphX handles graph computation. In modern Spark, Structured Streaming has largely replaced old Spark Streaming and DataFrames and Datasets have become the primary API replacing old RDDs for most use cases.

---

**4. What is SparkContext and SparkSession?**

SparkContext was the original entry point to Spark in older versions — it was used to create RDDs and connect to a cluster. SparkSession was introduced in Spark 2.0 as a unified entry point that combines SparkContext, SQLContext, and HiveContext into one object. In modern PySpark you always start by creating a SparkSession and it internally manages SparkContext for you. You should always use SparkSession now — SparkContext directly is only needed for very low level operations.

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("MyApp") \
    .getOrCreate()
```

---

**5. What is the difference between RDD, DataFrame, and Dataset?**

RDD stands for Resilient Distributed Dataset — it is the lowest level abstraction in Spark. It is a distributed collection of objects with no schema. It gives full control but has no optimization. DataFrame is a distributed collection of data organized into named columns like a table — it has a schema and benefits from Spark's Catalyst optimizer making it much faster than RDDs. Dataset combines benefits of RDD and DataFrame — it has schema like DataFrame but is strongly typed like RDD. In PySpark Dataset is not available because Python is dynamically typed, so you work with DataFrames. In practice always use DataFrames unless you need very low level control.

---

**6. What is a DAG in Spark?**

DAG stands for Directed Acyclic Graph. When you write Spark transformations Spark does not execute them immediately — instead it builds a DAG which is a logical execution plan showing all the steps needed to compute the final result. When an action is triggered Spark submits the DAG to the DAG Scheduler which breaks it into stages and tasks. DAG allows Spark to optimize the execution plan before running, recover from failures by recomputing only lost partitions, and avoid unnecessary computation. This is fundamental to how Spark achieves fault tolerance and performance.

---

**7. What is lazy evaluation in Spark?**

Lazy evaluation means Spark does not execute transformations immediately when you call them. It only records what transformations need to be done. Actual execution happens only when you call an action like collect, count, show, or write. This is beneficial because Spark can look at all transformations together and optimize the entire execution plan at once — for example combining multiple filter operations or reordering steps for efficiency. In practical terms if you write 10 transformations in a row Spark waits until you call an action and then executes everything in the most optimal way.

---

**8. What is the difference between transformations and actions in Spark?**

Transformations are operations that create a new RDD or DataFrame from an existing one — they are lazy and not executed immediately. Examples are filter, map, select, groupBy, join, withColumn. Actions are operations that trigger actual execution and return a result to the driver or write data to storage. Examples are collect, count, show, take, write, save. The key rule is: transformations are lazy, actions trigger execution. Every time you call an action Spark executes all pending transformations in the DAG to compute the result.

---

## Category 2: PySpark DataFrame Operations (Most Important)

**9. How do you create a DataFrame in PySpark?**

```python
# From a list
data = [("Alice", 30), ("Bob", 25)]
columns = ["name", "age"]
df = spark.createDataFrame(data, columns)

# From CSV file
df = spark.read.csv("path/to/file.csv", header=True, inferSchema=True)

# From Parquet
df = spark.read.parquet("path/to/file.parquet")

# From existing Pandas DataFrame
import pandas as pd
pandas_df = pd.DataFrame({"name": ["Alice"], "age": [30]})
df = spark.createDataFrame(pandas_df)
```
In production I mostly read from Parquet or Delta format as they are columnar and much faster than CSV.

---

**10. What are the most commonly used DataFrame transformations?**

```python
# Select specific columns
df.select("name", "age")

# Filter rows
df.filter(df.age > 25)
df.where(df.salary > 50000)

# Add new column
df.withColumn("salary_double", df.salary * 2)

# Rename column
df.withColumnRenamed("old_name", "new_name")

# Drop column
df.drop("column_name")

# Group and aggregate
df.groupBy("department").agg({"salary": "avg", "id": "count"})

# Sort
df.orderBy(df.salary.desc())

# Remove duplicates
df.dropDuplicates(["name", "department"])
```

---

**11. How do you handle NULL values in PySpark?**

```python
# Drop rows with any null
df.dropna()

# Drop rows where specific columns are null
df.dropna(subset=["name", "salary"])

# Fill nulls with a value
df.fillna(0)  # fills all numeric nulls with 0
df.fillna({"salary": 0, "name": "Unknown"})

# Replace nulls using when/otherwise
from pyspark.sql.functions import when, col
df.withColumn("salary", when(col("salary").isNull(), 0).otherwise(col("salary")))

# Filter out nulls
df.filter(col("salary").isNotNull())
```
In my pipelines I always handle nulls explicitly before writing to target tables to avoid downstream issues.

---

**12. How do you perform joins in PySpark?**

```python
# Inner join
df1.join(df2, on="id", how="inner")

# Left join
df1.join(df2, on="id", how="left")

# Join on multiple columns
df1.join(df2, on=["id", "department"], how="inner")

# Join on different column names
df1.join(df2, df1.emp_id == df2.id, how="inner")

# Full outer join
df1.join(df2, on="id", how="full")
```
Important thing to mention in interview: always broadcast smaller DataFrames in joins to avoid shuffle — this is a key performance optimization.

---

**13. What are Window functions in PySpark?**

```python
from pyspark.sql.functions import row_number, rank, lag, sum
from pyspark.sql.window import Window

# Define window
window = Window.partitionBy("department").orderBy(col("salary").desc())

# Row number
df.withColumn("row_num", row_number().over(window))

# Rank
df.withColumn("rank", rank().over(window))

# Running total
window2 = Window.partitionBy("department").orderBy("date").rowsBetween(Window.unboundedPreceding, Window.currentRow)
df.withColumn("running_total", sum("salary").over(window2))

# LAG - get previous row value
window3 = Window.partitionBy("dept").orderBy("month")
df.withColumn("prev_month_sales", lag("sales", 1).over(window3))
```
Window functions in PySpark work exactly like SQL window functions — PARTITION BY maps to partitionBy and ORDER BY maps to orderBy.

---

**14. How do you run SQL queries on a PySpark DataFrame?**

```python
# Register DataFrame as temporary view
df.createOrReplaceTempView("employees")

# Run SQL query
result = spark.sql("""
    SELECT department, AVG(salary) as avg_salary
    FROM employees
    GROUP BY department
    HAVING AVG(salary) > 50000
""")

result.show()
```
This is very useful when migrating existing SQL logic to Spark — you can run your SQL queries directly without rewriting everything in DataFrame API. I use this frequently in my current role when working with analysts who are more comfortable writing SQL.

---

## Category 3: Spark Architecture (Important for Senior Roles)

**15. Explain Spark architecture — Driver, Executor, Cluster Manager.**

Spark follows a master-slave architecture. Driver is the master process that runs your main program, creates SparkSession, builds the DAG, and coordinates execution. It breaks the job into tasks and distributes them to executors. Executors are worker processes running on cluster nodes — they execute the tasks assigned by driver, store data in memory or disk for caching, and send results back to driver. Cluster Manager manages and allocates resources across the cluster — it can be YARN, Kubernetes, Mesos, or Spark's own standalone manager. Driver talks to cluster manager to request executor resources, then directly communicates with executors for task distribution.

---

**16. What is a Spark job, stage, and task?**

When an action is called it creates a Job. A Job is the highest level unit of work. Each Job is broken into Stages — a new stage is created at every shuffle boundary, meaning wherever data needs to be redistributed across partitions. Each Stage is broken into Tasks — one task per partition. So if you have 100 partitions in a stage, 100 tasks run in parallel. Understanding this hierarchy helps when reading Spark UI to debug slow jobs — you look at which stage is slow and how many tasks are running in parallel.

---

**17. What is a shuffle in Spark? Why is it expensive?**

Shuffle happens when Spark needs to redistribute data across partitions — for example during groupBy, join, or orderBy operations where data with same keys needs to come to the same partition. Shuffle is expensive because it involves writing data to disk, sending it over network, and reading it back — all of which are slow operations compared to in-memory processing. Too many shuffles slow down your job significantly. As a data engineer you should minimize shuffles by using broadcast joins for small tables, partitioning data on join keys, and using partition-aware operations.

---

**18. What is partitioning in Spark? How does it affect performance?**

Partitioning is how Spark divides data across the cluster. Each partition is processed by one task on one executor. Too few partitions means less parallelism and some executors sit idle. Too many partitions means too much overhead managing tiny tasks. General rule is to have 2 to 4 partitions per CPU core available in your cluster. You can control partitions using repartition which shuffles data and creates balanced partitions, or coalesce which reduces partitions without full shuffle. Partitioning data on frequently joined or filtered columns — called partition pruning — can dramatically improve query performance by letting Spark skip irrelevant partitions entirely.

---

**19. What is the difference between repartition and coalesce?**

Both change the number of partitions but work differently. Repartition performs a full shuffle and can both increase and decrease partition count — it creates evenly balanced partitions. Use it when you want to increase partitions or need balanced data distribution. Coalesce only decreases partition count and avoids full shuffle by merging partitions on the same node — it is faster and more efficient but can create unbalanced partitions. Use coalesce when you just want to reduce partitions before writing to reduce output file count. In practice I use coalesce before writing final output to reduce small files problem.

---

**20. What is broadcast join? When do you use it?**

Broadcast join is an optimization where a small DataFrame is sent to all executor nodes so the join can happen locally on each node without any shuffle. When one table is small enough to fit in memory of each executor you should always broadcast it. This eliminates shuffle completely and dramatically improves join performance.

```python
from pyspark.sql.functions import broadcast

result = large_df.join(broadcast(small_df), on="id", how="inner")
```
Spark also does this automatically for tables below spark.sql.autoBroadcastJoinThreshold which defaults to 10MB. In my current pipelines I explicitly broadcast dimension tables when joining with large fact tables.

---

## Category 4: Performance Optimization (Asked for Senior/Experienced Roles)

**21. What are the common performance optimization techniques in Spark?**

Use DataFrames instead of RDDs as they benefit from Catalyst optimizer. Use broadcast joins for small tables to eliminate shuffle. Cache or persist DataFrames that are reused multiple times. Minimize shuffles by avoiding unnecessary groupBy and orderBy. Use columnar file formats like Parquet or ORC instead of CSV. Filter data as early as possible in your pipeline — push filters upstream. Partition data on columns you frequently filter or join on. Tune partition count based on data size and cluster resources. Avoid using collect on large DataFrames as it brings all data to driver and can cause out of memory errors. Use explain to check execution plan before running expensive jobs.

---

**22. What is caching and persistence in Spark? What is the difference?**

Cache stores a DataFrame in memory for reuse. If you use the same DataFrame multiple times Spark recomputes it from scratch each time — caching avoids this. Persist gives more control over storage level — you can choose to store in memory only, memory and disk, disk only, or with replication. Cache is just shorthand for persist with default memory and disk storage level. Use caching when a DataFrame is used multiple times in your pipeline.

```python
df.cache()  # stores in memory
df.persist()  # default storage level

from pyspark import StorageLevel
df.persist(StorageLevel.MEMORY_AND_DISK)

# Always unpersist when done to free memory
df.unpersist()
```

---

**23. What is the small files problem in Spark and how do you fix it?**

Small files problem happens when Spark writes too many tiny files to storage — for example when you have too many partitions each writing a small file. This causes performance issues when reading because each file open operation has overhead and metadata servers get overloaded. Fix it by using coalesce or repartition before writing to reduce number of output files. You can also use Delta Lake auto-optimize or OPTIMIZE command to compact small files. In HDFS you can use file compaction jobs. In my pipelines I always coalesce to a reasonable file count based on total data size — generally aiming for files between 128MB to 512MB each.

---

**24. How do you handle skewed data in Spark?**

Data skew happens when some partitions have significantly more data than others — causing some tasks to take much longer and becoming a bottleneck. You can identify skew by looking at task duration in Spark UI — if most tasks finish quickly but a few take very long you have skew. Solutions include salting the skewed key — adding a random prefix to distribute skewed keys across more partitions. Use broadcast join if the skewed side is a dimension table. Use skew hint in Spark 3.0+ which handles skew automatically with adaptive query execution. Repartition on a different column to redistribute data more evenly.

---

**25. What is Adaptive Query Execution in Spark 3.0?**

Adaptive Query Execution or AQE is a feature introduced in Spark 3.0 that optimizes query execution plans at runtime based on actual data statistics rather than estimates. It has three main features: dynamically coalescing shuffle partitions so Spark adjusts partition count based on actual data size after shuffle, dynamically switching join strategies for example converting sort merge join to broadcast join if one side turns out to be small, and dynamically optimizing skew joins by splitting skewed partitions into smaller ones. AQE is enabled by default in Spark 3.2 and significantly improves performance without manual tuning.

---

## Category 5: Real World Data Engineering with Spark

**26. How do you read and write different file formats in PySpark?**

```python
# CSV
df = spark.read.csv("path", header=True, inferSchema=True)
df.write.csv("output_path", header=True, mode="overwrite")

# Parquet (preferred format)
df = spark.read.parquet("path")
df.write.parquet("output_path", mode="overwrite")

# JSON
df = spark.read.json("path")
df.write.json("output_path")

# Delta Lake
df = spark.read.format("delta").load("path")
df.write.format("delta").mode("overwrite").save("path")

# Partitioned write
df.write.partitionBy("year", "month") \
  .parquet("output_path", mode="overwrite")
```
Always prefer Parquet over CSV in production — it is columnar, compressed, and much faster for analytics.

---

**27. How do you implement incremental processing in Spark?**

```python
# Read only new data based on date partition
df = spark.read.parquet("source_path") \
    .filter(col("date") == "2024-01-01")

# Get max timestamp from target and load only new records
max_ts = spark.read.parquet("target_path") \
    .agg({"updated_at": "max"}).collect()[0][0]

new_data = spark.read.parquet("source_path") \
    .filter(col("updated_at") > max_ts)

new_data.write.mode("append").parquet("target_path")
```
In production I prefer using Delta Lake for incremental processing as it handles ACID transactions and makes merge operations much simpler and more reliable.

---

**28. What is Delta Lake and why is it used?**

Delta Lake is an open source storage layer that brings ACID transactions to data lakes. Regular data lakes have no transaction support — if a job fails midway you get corrupted or partial data. Delta Lake solves this with full ACID compliance. It also supports time travel — you can query data as it was at any point in time. It allows schema enforcement and evolution. It supports MERGE operation for upserts which is essential for SCD Type 2 and incremental loads. It has auto-optimize and auto-compaction features. In modern data engineering Delta Lake has become the standard for building reliable data lakehouses on cloud platforms.

---

**29. How do you do a MERGE or upsert operation in PySpark with Delta Lake?**

```python
from delta.tables import DeltaTable

delta_table = DeltaTable.forPath(spark, "path/to/delta/table")

delta_table.alias("target").merge(
    source_df.alias("source"),
    "target.id = source.id"
).whenMatchedUpdateAll() \
 .whenNotMatchedInsertAll() \
 .execute()
```
This updates existing records when id matches and inserts new records when there is no match — classic upsert behavior. This is equivalent to SQL MERGE statement and is heavily used in incremental data pipelines. In my current role I use this pattern for all dimension table loads.

---

**30. How do you handle schema evolution in Spark?**

Schema evolution means handling changes in source data schema — new columns added, columns removed, or data types changed. In Spark you can use mergeSchema option when reading Parquet or Delta to automatically handle new columns being added. With Delta Lake you can set schema evolution mode to allow automatic schema changes. For type changes you need to explicitly cast columns. In my pipelines I always define an explicit schema instead of using inferSchema in production because inferSchema is slow on large files and can guess wrong types. When source schema changes I version control the schema definition and handle migration explicitly.

---

## Quick Revision Cheat Sheet

| Topic | Priority |
|---|---|
| RDD vs DataFrame vs Dataset | 🔴 Must Know |
| Lazy Evaluation | 🔴 Must Know |
| Transformations vs Actions | 🔴 Must Know |
| Joins and Broadcast Join | 🔴 Must Know |
| Window Functions | 🔴 Must Know |
| Partitioning and Shuffle | 🔴 Must Know |
| Cache and Persist | 🟠 Important |
| Performance Optimization | 🟠 Important |
| Delta Lake and MERGE | 🟠 Important |
| AQE and Skew Handling | 🟡 Good to Know |
| Spark Architecture | 🟡 Good to Know |

---

## Top 5 Most Likely to Be Asked Tomorrow

1. Difference between RDD, DataFrame, Dataset
2. What is lazy evaluation — explain with example
3. Broadcast join and when to use it
4. How do you optimize a slow Spark job
5. How do you handle incremental loads in Spark

---

## One Pro Tip for Spark Interviews

Whenever they ask about optimization always mention **three things:** broadcast joins, caching reused DataFrames, and minimizing shuffles. These three cover 80% of real world Spark performance problems and show you have practical experience.

**You are ready. Go get that offer! 💪**
