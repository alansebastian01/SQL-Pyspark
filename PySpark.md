
## What PySpark actually is

**Apache Spark** is a system for processing large datasets.

**PySpark** is the Python interface to Spark.

So when you write:

```python
df = spark.read.csv("sales.csv")
```

Python is not directly doing all the heavy processing. Your Python code is telling **Spark** what work you want done.

Think of it like this:

```text
You write Python
      ↓
   PySpark
      ↓
 Apache Spark
      ↓
CPU cores / multiple computers
      ↓
Large dataset gets processed
```

Spark was designed for situations where the data or computation is too large for a single machine to handle efficiently.

---

# 1. Why do we even need PySpark?

Imagine you have a file with:

```text
1,000 rows
```

Pandas can easily handle it.

Now imagine:

```text
10 million rows
```

Pandas may still handle it depending on your computer.

But now imagine:

```text
5 billion rows
3 TB of data
```

A laptop with 16 GB RAM cannot load 3 TB into memory.

Spark solves this by dividing the data into smaller pieces and processing those pieces separately.

For example:

```text
3 TB Dataset
     |
     ├── 500 GB → Computer A
     ├── 500 GB → Computer B
     ├── 500 GB → Computer C
     ├── 500 GB → Computer D
     ├── 500 GB → Computer E
     └── 500 GB → Computer F
```

Each computer processes part of the data.

Spark combines the results afterward.

This is called **distributed computing**.

---

# 2. PySpark vs Pandas

At first, PySpark DataFrames look similar to Pandas DataFrames.

Pandas:

```python
import pandas as pd

df = pd.read_csv("employees.csv")

df[df["salary"] > 100000]
```

PySpark:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()

df = spark.read.csv(
    "employees.csv",
    header=True,
    inferSchema=True
)

df.filter(df.salary > 100000).show()
```

Both are doing something similar.

But internally they work very differently.

| Pandas                              | PySpark                            |
| ----------------------------------- | ---------------------------------- |
| Usually one computer                | Can use many computers             |
| Data usually loaded into memory     | Data distributed across partitions |
| Best for small/medium datasets      | Best for very large datasets       |
| Operations often happen immediately | Operations are usually lazy        |
| Python executes most operations     | Spark engine executes operations   |

That last difference is extremely important.

---

# 3. The SparkSession

Almost every PySpark program starts with something like:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("MyFirstSparkApp") \
    .getOrCreate()
```

The variable:

```python
spark
```

is your main connection to Spark.

You use it to:

* read files
* create DataFrames
* run SQL
* configure Spark
* interact with the Spark engine

For example:

```python
df = spark.read.csv("customers.csv")
```

or:

```python
df = spark.read.parquet("customers.parquet")
```

or:

```python
spark.sql("SELECT * FROM customers")
```

A beginner-friendly way to think about `SparkSession` is:

```text
SparkSession = your control panel for Spark
```

---

# 4. What is a Spark DataFrame?

A DataFrame is basically a table.

For example:

```text
+----+-------+------+----------+
| id | name  | age  | salary   |
+----+-------+------+----------+
| 1  | Alice | 25   | 70000    |
| 2  | Bob   | 30   | 90000    |
| 3  | Carol | 35   | 120000   |
+----+-------+------+----------+
```

Each row is a record.

Each column represents an attribute.

You could create this in PySpark:

```python
data = [
    (1, "Alice", 25, 70000),
    (2, "Bob", 30, 90000),
    (3, "Carol", 35, 120000)
]

columns = ["id", "name", "age", "salary"]

df = spark.createDataFrame(data, columns)
```

Then:

```python
df.show()
```

prints something like:

```text
+---+-----+---+------+
| id| name|age|salary|
+---+-----+---+------+
|  1|Alice| 25| 70000|
|  2|  Bob| 30| 90000|
|  3|Carol| 35|120000|
+---+-----+---+------+
```

---

# 5. Understanding schemas

A Spark DataFrame has a **schema**.

The schema tells Spark what type each column contains.

You can see it with:

```python
df.printSchema()
```

You might get:

```text
root
 |-- id: long
 |-- name: string
 |-- age: long
 |-- salary: long
```

This matters because Spark treats:

```text
"100"
```

and:

```text
100
```

differently.

The first is a string.

The second is a number.

For example:

```python
df.filter(df.salary > 100000)
```

only works correctly if `salary` has a numeric type.

Common Spark types include:

```text
StringType
IntegerType
LongType
DoubleType
BooleanType
DateType
TimestampType
ArrayType
StructType
```

You don't need to memorize them yet.

Just understand:

> Every column has a data type.

---

# 6. Selecting columns

Suppose your DataFrame has:

```text
name
age
salary
department
```

You can select columns:

```python
df.select("name", "salary").show()
```

Result:

```text
+-----+------+
| name|salary|
+-----+------+
|Alice| 70000|
|  Bob| 90000|
|Carol|120000|
+-----+------+
```

You can also use:

```python
df.select(df.name, df.salary).show()
```

Another common syntax is:

```python
from pyspark.sql.functions import col

df.select(
    col("name"),
    col("salary")
).show()
```

You will see `col()` **a lot** in PySpark.

---

# 7. Filtering rows

Filtering means keeping rows that match a condition.

Example:

```python
df.filter(df.salary > 80000).show()
```

You could also write:

```python
df.where(df.salary > 80000).show()
```

`filter()` and `where()` are basically equivalent.

Using `col()`:

```python
from pyspark.sql.functions import col

df.filter(col("salary") > 80000).show()
```

Multiple conditions:

```python
df.filter(
    (col("salary") > 80000) &
    (col("age") < 40)
).show()
```

Notice:

```python
&
```

means AND.

And:

```python
|
```

means OR.

Example:

```python
df.filter(
    (col("department") == "Engineering") |
    (col("department") == "Finance")
)
```

---

# 8. Creating new columns

Suppose you have:

```text
salary
```

and want a bonus column.

You can use:

```python
from pyspark.sql.functions import col

df = df.withColumn(
    "bonus",
    col("salary") * 0.10
)
```

Now:

```text
salary    bonus
70000     7000
90000     9000
120000    12000
```

You could also create total compensation:

```python
df = df.withColumn(
    "total_compensation",
    col("salary") + col("bonus")
)
```

`withColumn()` is one of the most common PySpark methods.

---

# 9. Renaming columns

You can rename a column:

```python
df = df.withColumnRenamed(
    "salary",
    "annual_salary"
)
```

So:

```text
salary
```

becomes:

```text
annual_salary
```

---

# 10. Removing columns

Use:

```python
df = df.drop("age")
```

You can remove multiple columns:

```python
df = df.drop("age", "bonus")
```

---

# 11. Sorting

To sort ascending:

```python
df.orderBy("salary").show()
```

or:

```python
df.sort("salary").show()
```

Descending:

```python
from pyspark.sql.functions import col

df.orderBy(
    col("salary").desc()
).show()
```

---

# 12. GroupBy and aggregation

This is extremely important in data engineering.

Imagine:

```text
name     department     salary
Alice    Engineering    100000
Bob      Engineering    120000
Carol    Finance         90000
David    Finance        110000
```

You want average salary per department.

```python
df.groupBy("department") \
  .avg("salary") \
  .show()
```

Result:

```text
Engineering    110000
Finance        100000
```

You can calculate several things:

```python
from pyspark.sql.functions import avg, sum, count, max, min

df.groupBy("department").agg(
    avg("salary").alias("avg_salary"),
    max("salary").alias("max_salary"),
    count("*").alias("employees")
).show()
```

This is conceptually very similar to SQL:

```sql
SELECT
    department,
    AVG(salary),
    MAX(salary),
    COUNT(*)
FROM employees
GROUP BY department;
```

---

# 13. PySpark and SQL work together

This is one reason Spark is popular.

You can use DataFrame syntax:

```python
df.groupBy("department").avg("salary")
```

or SQL.

First register the DataFrame:

```python
df.createOrReplaceTempView("employees")
```

Then:

```python
spark.sql("""
    SELECT
        department,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
""").show()
```

These are simply two ways of expressing similar operations.

If you already know SQL, that will make learning PySpark much easier.

---

# 14. Joins

Joins are one of the most important concepts in real data engineering.

Suppose you have:

**employees**

```text
employee_id    name    department_id

1              Alice   10
2              Bob     20
3              Carol   10
```

And another table:

**departments**

```text
department_id    department_name

10               Engineering
20               Finance
```

You can combine them:

```python
employees.join(
    departments,
    on="department_id",
    how="inner"
).show()
```

Result:

```text
employee_id    name     department_id    department_name

1              Alice    10               Engineering
2              Bob      20               Finance
3              Carol    10               Engineering
```

Common join types:

```text
inner
left
right
full
left_semi
left_anti
```

The first four correspond closely to SQL joins.

---

# 15. Null values

Real datasets contain missing data all the time.

Example:

```text
name      age
Alice     25
Bob       NULL
Carol     34
```

You can remove rows containing nulls:

```python
df.dropna()
```

Or only check certain columns:

```python
df.dropna(subset=["age"])
```

You can replace null values:

```python
df.fillna({
    "age": 0
})
```

You can find null values:

```python
from pyspark.sql.functions import col

df.filter(
    col("age").isNull()
).show()
```

Or non-null:

```python
df.filter(
    col("age").isNotNull()
).show()
```

---

# 16. The most important Spark concept: lazy evaluation

This is where Spark starts becoming very different from normal Python.

Suppose you write:

```python
df2 = df.filter(col("salary") > 50000)
```

Spark may **not actually process the data yet**.

Then you write:

```python
df3 = df2.select("name", "salary")
```

Again, Spark may do nothing.

Then:

```python
df4 = df3.orderBy("salary")
```

Still nothing.

Spark is building a plan:

```text
Read data
   ↓
Filter salary > 50000
   ↓
Select name, salary
   ↓
Sort salary
```

Then you call:

```python
df4.show()
```

Now Spark says:

> Okay, you actually want the result. I'll execute the plan.

This is called **lazy evaluation**.

---

# 17. Transformations vs actions

This leads to a very important distinction.

## Transformations

Transformations describe changes to data.

Examples:

```python
df.select(...)
df.filter(...)
df.withColumn(...)
df.groupBy(...)
df.join(...)
df.orderBy(...)
```

They usually do not immediately execute the full computation.

Think:

```text
Transformation = tell Spark WHAT you want
```

## Actions

Actions ask Spark to actually produce a result.

Examples:

```python
df.show()
df.count()
df.collect()
df.first()
df.take(10)
```

Think:

```text
Action = tell Spark to DO the work
```

For example:

```python
df2 = df.filter(col("salary") > 100000)
```

Transformation.

Then:

```python
df2.count()
```

Action.

Spark actually starts processing.

---

# 18. Why lazy evaluation exists

Because Spark can optimize your work.

Suppose you write:

```python
df = spark.read.parquet("customers")

df2 = df.select(
    "name",
    "country",
    "salary"
)

df3 = df2.filter(
    col("country") == "USA"
)

df3.show()
```

Spark can examine the entire plan before executing it.

It might realize:

> The dataset contains 50 columns, but this person only needs 3.

So Spark avoids unnecessarily reading the other columns.

This can dramatically improve performance.

---

# 19. Partitions

This is arguably the most important concept for understanding distributed Spark.

Imagine a dataset containing 1 billion rows.

Spark may split it into:

```text
Partition 1
Partition 2
Partition 3
Partition 4
...
Partition 500
```

Each partition contains part of the data.

Different Spark workers can process different partitions simultaneously.

Example:

```text
                 Dataset

         ┌──────────┼──────────┐
         ↓          ↓          ↓

    Partition 1  Partition 2  Partition 3

         ↓          ↓          ↓

      Core 1      Core 2      Core 3
```

This is how Spark achieves parallelism.

Instead of:

```text
one CPU processes 1 billion rows
```

you might have:

```text
100 CPUs processing different parts
```

at the same time.

---

# 20. Driver and executors

Spark applications generally have two important roles.

### Driver

The **driver** is the brain.

It:

* runs your main program
* understands your Spark commands
* creates execution plans
* coordinates the work

### Executors

Executors are the workers.

They:

* process partitions
* execute calculations
* return results

Conceptually:

```text
              Driver
            "Do this job"
                 |
       ┌─────────┼─────────┐
       ↓         ↓         ↓

   Executor 1 Executor 2 Executor 3

   Partitions Partitions Partitions
```

If you use a Spark cluster with 20 machines, there could be many executors performing work simultaneously.

---

# 21. What exactly is a Spark cluster?

A cluster is simply multiple computers working together.

For example:

```text
Spark Cluster

Machine A
Machine B
Machine C
Machine D
Machine E
```

Instead of buying one gigantic machine with:

```text
1 TB RAM
200 CPUs
```

an organization might use many smaller machines together.

Spark coordinates them.

Cloud platforms frequently run Spark clusters on:

* AWS
* Microsoft Azure
* Google Cloud
* Databricks

---

# 22. Jobs, stages, and tasks

You'll eventually see these terms in Spark's UI.

Suppose you run:

```python
df.groupBy("country").count().show()
```

Spark breaks this into smaller pieces.

Conceptually:

```text
Action
 ↓
Job
 ↓
Stages
 ↓
Tasks
```

A **job** is usually triggered by an action.

A job can contain multiple **stages**.

Each stage contains many **tasks**.

A task typically operates on one partition.

Example:

```text
Job
 │
 ├── Stage 1
 │      ├── Task 1 → Partition 1
 │      ├── Task 2 → Partition 2
 │      └── Task 3 → Partition 3
 │
 └── Stage 2
        ├── Task 1
        └── Task 2
```

You don't need to master this immediately.

Just remember:

```text
Job
   → stages
       → tasks
           → partitions
```

---

# 23. Shuffle

This is one of the biggest performance concepts in Spark.

Suppose your data is distributed like:

```text
Partition 1:
USA
Canada
USA

Partition 2:
India
USA
India

Partition 3:
Canada
India
USA
```

Now you run:

```python
df.groupBy("country").count()
```

Spark needs all `USA` records together, all `India` records together, etc.

So data gets moved between machines:

```text
Original partitions
        ↓
   Data moves
        ↓
New partitions grouped by country
```

That movement is called a **shuffle**.

Shuffles can be expensive because they involve:

* network traffic
* disk operations
* serialization
* memory

Operations that commonly cause shuffles include:

```text
groupBy
join
distinct
orderBy
repartition
```

You'll hear Spark engineers talk about reducing shuffles constantly.

---

# 24. `collect()` can be dangerous

Imagine your distributed Spark DataFrame contains:

```text
500 GB
```

Then you run:

```python
df.collect()
```

Spark tries to send all that data back to your **driver machine**.

If your driver has:

```text
16 GB RAM
```

you have a problem.

It may crash.

Instead, use things like:

```python
df.show()
```

or:

```python
df.limit(100).collect()
```

or aggregate the data first.

A very important beginner rule:

> Don't use `collect()` on large datasets unless you know the result is small.

---

# 25. Reading files

Spark can read many data formats.

CSV:

```python
df = spark.read \
    .option("header", True) \
    .option("inferSchema", True) \
    .csv("sales.csv")
```

JSON:

```python
df = spark.read.json("data.json")
```

Parquet:

```python
df = spark.read.parquet("sales.parquet")
```

Spark also integrates with databases and systems such as:

```text
PostgreSQL
MySQL
SQL Server
Snowflake
BigQuery
S3
Azure Data Lake
HDFS
Kafka
Delta Lake
```

---

# 26. Why Parquet is important

You'll frequently see Parquet in Spark projects.

CSV stores data somewhat like:

```text
Alice,25,USA,70000
Bob,30,UK,90000
```

Parquet is a specialized analytical data format.

Its big advantage is that it is **columnar**.

Suppose your table has 100 columns:

```text
column1
column2
column3
...
column100
```

but your query only needs:

```text
name
salary
```

Parquet allows Spark to read mainly those required columns instead of processing everything.

That makes queries much faster.

In data engineering, you'll commonly encounter:

```text
Parquet
Delta
Iceberg
```

---

# 27. Writing data

After transforming data, you usually save it somewhere.

CSV:

```python
df.write \
  .mode("overwrite") \
  .csv("output")
```

Parquet:

```python
df.write \
  .mode("overwrite") \
  .parquet("output")
```

Common write modes include:

```text
overwrite
append
ignore
error
```

For example:

```python
df.write.mode("append").parquet("sales")
```

adds new data.

---

# 28. A realistic PySpark pipeline

Suppose you're a data engineer working for an online store.

You receive daily transactions:

```text
transactions.csv
```

You might do this:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, sum

spark = SparkSession.builder \
    .appName("SalesPipeline") \
    .getOrCreate()

df = spark.read \
    .option("header", True) \
    .option("inferSchema", True) \
    .csv("transactions.csv")
```

Remove invalid records:

```python
clean_df = df.filter(
    col("amount").isNotNull()
)
```

Keep successful transactions:

```python
clean_df = clean_df.filter(
    col("status") == "SUCCESS"
)
```

Calculate revenue by country:

```python
revenue_df = clean_df \
    .groupBy("country") \
    .agg(
        sum("amount").alias("revenue")
    )
```

Save the result:

```python
revenue_df.write \
    .mode("overwrite") \
    .parquet("country_revenue")
```

Your entire pipeline is:

```text
Raw transactions
      ↓
Read with Spark
      ↓
Remove bad records
      ↓
Filter successful orders
      ↓
Aggregate revenue
      ↓
Write Parquet
```

That is a simplified example of a real **ETL pipeline**.

---

# 29. What ETL means

You'll hear this constantly in data engineering.

ETL means:

```text
Extract
Transform
Load
```

For example:

```text
Extract

Read sales data from S3
        ↓

Transform

Remove bad rows
Join customer information
Calculate metrics
        ↓

Load

Write results into warehouse
```

PySpark is heavily used for the **Transform** part.

---

# 30. Example: a more complete transformation

Imagine:

```python
sales_df
```

contains:

```text
order_id
customer_id
product
quantity
price
country
```

You can calculate revenue:

```python
from pyspark.sql.functions import col

sales_df = sales_df.withColumn(
    "revenue",
    col("quantity") * col("price")
)
```

Then filter:

```python
sales_df = sales_df.filter(
    col("revenue") > 100
)
```

Then aggregate:

```python
from pyspark.sql.functions import sum

summary = sales_df.groupBy(
    "country"
).agg(
    sum("revenue").alias("total_revenue")
)
```

Then sort:

```python
summary = summary.orderBy(
    col("total_revenue").desc()
)
```

Finally:

```python
summary.show()
```

Conceptually:

```text
sales
  ↓
calculate revenue
  ↓
remove small orders
  ↓
group by country
  ↓
calculate total revenue
  ↓
sort
  ↓
show result
```

---

# 31. One important PySpark habit

You'll often see beginners write:

```python
df.salary
```

This works in many cases.

But I'd recommend becoming comfortable with:

```python
col("salary")
```

Example:

```python
from pyspark.sql.functions import col

df.filter(
    col("salary") > 100000
)
```

because `col()` makes complex Spark transformations easier to read and maintain.

---

# 32. Common PySpark functions

You'll use these constantly:

```python
from pyspark.sql.functions import (
    col,
    lit,
    when,
    sum,
    avg,
    count,
    min,
    max,
    concat,
    lower,
    upper,
    trim
)
```

For example:

```python
df.withColumn(
    "salary_after_raise",
    col("salary") * 1.10
)
```

Or conditional logic:

```python
from pyspark.sql.functions import when

df = df.withColumn(
    "salary_level",
    when(col("salary") >= 100000, "High")
    .otherwise("Normal")
)
```

Equivalent SQL idea:

```sql
CASE
    WHEN salary >= 100000 THEN 'High'
    ELSE 'Normal'
END
```

---

# 33. PySpark doesn't behave like normal Python loops

This is an important mindset shift.

A beginner might try:

```python
for row in df.collect():
    print(row)
```

That defeats much of Spark's purpose.

You're pulling distributed data into Python and processing it locally.

Spark-style thinking is:

```python
df.filter(...)
  .withColumn(...)
  .groupBy(...)
  .agg(...)
```

You're describing operations over the entire dataset.

Think:

```text
BAD mental model:

Take row 1
process it
take row 2
process it
take row 3
process it
```

Instead:

```text
Spark mental model:

Tell Spark:
"Apply this operation to all rows."
```

---

# 34. Avoid Python UDFs when possible

You'll eventually encounter **UDFs**, or User Defined Functions.

Example:

```python
def double_salary(x):
    return x * 2
```

You can turn this into a Spark UDF.

But beginners often use UDFs unnecessarily.

Built-in Spark functions are usually faster:

```python
df.withColumn(
    "double_salary",
    col("salary") * 2
)
```

is preferable to writing a Python UDF.

Rule of thumb:

> Use Spark's built-in functions whenever possible.

---

# 35. Caching

Suppose an expensive DataFrame is used repeatedly:

```python
processed_df = huge_df \
    .filter(...) \
    .join(...) \
    .groupBy(...)
```

If you run:

```python
processed_df.count()
```

and later:

```python
processed_df.show()
```

Spark might recompute earlier transformations.

You can tell Spark:

```python
processed_df.cache()
```

Then Spark may retain the data for faster reuse.

But caching isn't automatically good.

It uses memory.

So don't start caching everything.

---

# 36. Repartition vs coalesce

You'll encounter:

```python
df.repartition(100)
```

This changes the number of partitions.

For example:

```text
20 partitions
      ↓
repartition(100)
      ↓
100 partitions
```

You can also use:

```python
df.coalesce(5)
```

typically when reducing partitions.

These become important for performance and output-file management.

But you don't need to optimize partitions on day one.

---

# 37. PySpark architecture in one picture

Here's the mental model I want you to remember:

```text
YOU

Python / PySpark code

        ↓

DRIVER

Creates execution plan

        ↓

Spark distributes work

        ↓

┌────────────┬────────────┬────────────┐
│ Executor 1 │ Executor 2 │ Executor 3 │
│            │            │            │
│ Partition  │ Partition  │ Partition  │
│ 1, 2, 3    │ 4, 5, 6    │ 7, 8, 9    │
└────────────┴────────────┴────────────┘

        ↓

Results combined

        ↓

Output
```

---

# 38. Where PySpark is typically used

You might see an architecture like:

```text
Applications
     ↓
Kafka
     ↓
Data Lake / S3
     ↓
PySpark
     ↓
Cleaned data
     ↓
Data Warehouse
     ↓
Tableau / Power BI / Analysts
```

Or:

```text
Database
   ↓
PySpark ETL
   ↓
Delta Lake
   ↓
Databricks SQL
   ↓
Dashboards
```

---

# 39. Spark and Databricks

These two names often appear together.

**Spark** is the distributed data processing engine.

**Databricks** is a cloud platform built heavily around Spark.

A simplified analogy:

```text
Spark = engine

Databricks = a platform built around that engine
```

Databricks gives you things like:

* notebooks
* Spark clusters
* scheduling
* Delta Lake
* SQL warehouses
* job pipelines
* governance
* machine learning tools

So when a job posting says:

```text
Python
SQL
PySpark
Databricks
AWS
```

these technologies often work together.

---

# 40. What you should learn first

Don't try to learn every Spark optimization immediately.

I would learn PySpark in this order:

```text
Python basics
      ↓
SQL basics
      ↓
Spark DataFrames
      ↓
select / filter / withColumn
      ↓
groupBy / aggregations
      ↓
joins
      ↓
null handling
      ↓
Spark SQL
      ↓
lazy evaluation
      ↓
partitions
      ↓
shuffles
      ↓
cache
      ↓
performance optimization
      ↓
real ETL pipelines
```

The first **six or seven items** matter much more for a beginner than advanced Spark internals.

---

# 41. The 80/20 PySpark commands

If you're just starting, concentrate on these:

```python
df.show()

df.printSchema()

df.select()

df.filter()

df.withColumn()

df.withColumnRenamed()

df.drop()

df.groupBy()

df.agg()

df.join()

df.orderBy()

df.dropna()

df.fillna()

df.write()
```

And these functions:

```python
col()
when()
lit()
sum()
avg()
count()
min()
max()
```

If you're comfortable with those, you can already do a surprising amount of practical PySpark work.

---

# 42. One final mental model

When you write normal Python, you often think:

```text
Python is processing my data.
```

When you write PySpark, think:

```text
I am using Python to describe a data-processing plan.

Spark takes that plan,
optimizes it,
splits the data,
sends work to executors,
and processes the partitions in parallel.
```

That's the fundamental idea behind PySpark.

For a beginner, I'd summarize the whole subject like this:

```text
PySpark
   =
Python syntax
   +
SQL-style transformations
   +
distributed computing
   +
big-data processing
```

And your first goal shouldn't be to understand every Spark internals concept. Get comfortable manipulating a DataFrame first. Once `select`, `filter`, `withColumn`, `groupBy`, and `join` feel natural, concepts like partitions, shuffles, executors, and optimization become much easier to understand.
