# PySpark Beginner Guide

This guide collects the PySpark concepts and examples covered so far. It is designed for beginners and can be run locally or in Google Colab.

## Contents

1. [What is PySpark?](#1-what-is-pyspark)
2. [Install and start PySpark](#2-install-and-start-pyspark)
3. [Understanding `master("local[*]")`](#3-understanding-masterlocal)
4. [Create your first DataFrame](#4-create-your-first-dataframe)
5. [Inspect a DataFrame](#5-inspect-a-dataframe)
6. [Select, filter, and add columns](#6-select-filter-and-add-columns)
7. [Sort and group data](#7-sort-and-group-data)
8. [Transformations, actions, and lazy evaluation](#8-transformations-actions-and-lazy-evaluation)
9. [Caching reused results](#9-caching-reused-results)
10. [Spark query-plan stages](#10-spark-query-plan-stages)
11. [Reading an extended query plan](#11-reading-an-extended-query-plan)
12. [Does optimization depend on RDD size?](#12-does-optimization-depend-on-rdd-size)
13. [Practice exercises](#13-practice-exercises)
14. [Quick reference](#14-quick-reference)

## 1. What is PySpark?

PySpark is the Python API for Apache Spark. It is used to process and analyze data, especially when the data is too large for one machine.

A PySpark DataFrame is similar to:

- A table in a database
- An Excel worksheet
- A pandas DataFrame

Spark can divide work into partitions and process those partitions in parallel.

## 2. Install and start PySpark

### Local installation

```bash
pip install pyspark
```

Check the installed version:

```bash
python -c "import pyspark; print(pyspark.__version__)"
```

### Google Colab

Run this in the first cell:

```python
!pip install -q pyspark
```

### Create a Spark session

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("PySparkBeginnerGuide")
    .master("local[*]")
    .getOrCreate()
)

print("Spark version:", spark.version)
print("Spark master:", spark.sparkContext.master)
```

`SparkSession` is the main entry point for working with DataFrames and Spark SQL.

Stop Spark when the application is finished:

```python
spark.stop()
```

## 3. Understanding `master("local[*]")`

```python
.master("local[*]")
```

This tells Spark where and how to run the application:

- `local` means run on the current computer.
- `[*]` means Spark may use all available CPU cores.

Other examples:

```python
.master("local[1]")  # One CPU core
.master("local[2]")  # Two CPU cores
.master("local[4]")  # Four CPU cores
.master("local[*]")  # All available CPU cores
```

The line is not always required. If a script works without it, the master is probably being supplied by `spark-submit`, a notebook, an IDE, an environment variable, or an existing Spark session.

Check the active master:

```python
print(spark.sparkContext.master)
```

For example, the master can be passed outside the code:

```bash
spark-submit --master local[*] app.py
```

Leaving the master out of production application code often makes the same program easier to run on different clusters.

## 4. Create your first DataFrame

```python
data = [
    (1, "Alice", 25, "Engineering", 70000),
    (2, "Bob", 30, "Sales", 60000),
    (3, "Charlie", 35, "Engineering", 90000),
    (4, "Diana", 28, "Marketing", 65000),
    (5, "Evan", 32, "Sales", 75000),
]

columns = ["id", "name", "age", "department", "salary"]

df = spark.createDataFrame(data, columns)
df.show()
```

The comma after the last record is an optional **trailing comma**:

```python
data = [
    (1, "Alice"),
    (2, "Bob"),
]
```

This is also valid:

```python
data = [
    (1, "Alice"),
    (2, "Bob")
]
```

Trailing commas are common because they make it easy to add or reorder lines. A comma is required when creating a one-item tuple:

```python
number = (1)    # Integer
number = (1,)   # Tuple containing one integer
```

## 5. Inspect a DataFrame

```python
df.show()          # Display rows
df.printSchema()   # Display column names and types

print(df.columns)  # Python list of column names
print(df.first())  # First Row object
print(df.count())  # Number of rows
```

The schema will look similar to:

```text
root
 |-- id: long (nullable = true)
 |-- name: string (nullable = true)
 |-- age: long (nullable = true)
 |-- department: string (nullable = true)
 |-- salary: long (nullable = true)
```

`nullable = true` means the column is allowed to contain a missing value (`null`).

## 6. Select, filter, and add columns

Import commonly used column functions:

```python
from pyspark.sql.functions import col
```

### Select columns

```python
df.select("name", "age").show()
```

Equivalent form:

```python
df.select(col("name"), col("age")).show()
```

### Filter rows

```python
df.filter(col("age") >= 30).show()

df.filter(col("department") == "Engineering").show()
```

Use `&` for AND and `|` for OR. Put each condition in parentheses:

```python
df.filter(
    (col("age") >= 30) &
    (col("department") == "Engineering")
).show()
```

```python
df.filter(
    (col("department") == "Sales") |
    (col("department") == "Marketing")
).show()
```

### Add a column

```python
df_with_future_age = df.withColumn(
    "age_after_five_years",
    col("age") + 5
)

df_with_future_age.show()
```

DataFrames are immutable. `withColumn()` returns a new DataFrame; it does not change `df` in place.

## 7. Sort and group data

Import aggregation functions:

```python
from pyspark.sql.functions import avg, count, max, min
```

### Sort rows

```python
df.orderBy("age").show()

df.orderBy(col("age").desc()).show()

df.orderBy(
    col("department").asc(),
    col("salary").desc()
).show()
```

`sort()` is an alias for `orderBy()`:

```python
df.sort("salary").show()
```

### Group rows

```python
df.groupBy("department").count().show()
```

Calculate several statistics:

```python
department_summary = (
    df
    .groupBy("department")
    .agg(
        count("*").alias("employee_count"),
        avg("salary").alias("average_salary"),
        max("salary").alias("highest_salary"),
        min("salary").alias("lowest_salary"),
    )
)

department_summary.show()
```

`alias()` gives the calculated column a readable name.

## 8. Transformations, actions, and lazy evaluation

Spark divides DataFrame operations into two important categories.

### Transformations

Transformations describe work and return a new DataFrame. They normally do not calculate the final result immediately.

```python
filtered_df = df.filter(col("age") >= 30)
selected_df = filtered_df.select("name", "department", "salary")
result_df = selected_df.orderBy(col("salary").desc())
```

Common transformations include:

```text
select()
filter() / where()
withColumn()
drop()
orderBy() / sort()
groupBy().agg()
join()
distinct()
```

### Actions

Actions ask Spark to produce or save a result. They trigger execution of the transformation plan.

```python
result_df.show()
result_df.count()
result_df.first()
result_df.take(2)
result_df.collect()
```

Writing output is also an action:

```python
result_df.write.mode("overwrite").parquet("output/employees")
```

Be careful with `collect()`. It transfers every result row to the Python driver and can cause an out-of-memory error for large data.

### Lazy evaluation

When transformations are written, Spark creates a plan instead of immediately processing all the rows:

```python
result_df = (
    df
    .filter(col("age") >= 30)
    .select("name", "department", "salary")
    .orderBy(col("salary").desc())
)
```

Conceptually:

```text
Read DataFrame
    → Filter age >= 30
    → Select three columns
    → Sort salary descending
```

The following action asks Spark to execute that plan:

```python
result_df.show()
```

The rule to remember is:

> Transformations describe the work; actions start the work.

Lazy evaluation lets Spark examine all requested operations together and optimize them before execution.

## 9. Caching reused results

Each action may execute the transformation plan again:

```python
filtered_df = df.filter(col("age") >= 30)

filtered_df.show()   # First action
filtered_df.count()  # Another action; plan may run again
```

Cache a DataFrame when an expensive result will be reused:

```python
filtered_df.cache()

# The first action computes the data and populates the cache.
filtered_df.show()

# This action can reuse cached data.
print(filtered_df.count())

filtered_df.unpersist()
```

`cache()` is lazy. It marks the DataFrame for caching, but an action is needed to populate the cache.

Do not cache every DataFrame. Cached data uses memory and is most useful when the same expensive result is used multiple times.

## 10. Spark query-plan stages

Transformations produce a query plan that passes through these stages:

```text
DataFrame transformations
          ↓
Parsed/unresolved logical plan
          ↓
Analyzed logical plan
          ↓
Optimized logical plan
          ↓
Physical plan
          ↓
Action executes the physical plan
```

Use this to inspect all stages:

```python
result_df.explain("extended")
```

It prints:

```text
== Parsed Logical Plan ==
== Analyzed Logical Plan ==
== Optimized Logical Plan ==
== Physical Plan ==
```

### Parsed logical plan

This is Spark's initial interpretation of the requested operations. Some references may not yet be fully resolved.

It answers:

> What operations did the programmer request?

### Analyzed logical plan

Spark's analyzer resolves column names and verifies data types.

It checks that:

- Referenced columns exist.
- Expressions use compatible types.
- The final output schema can be determined.

It answers:

> Are all columns and types valid?

### Optimized logical plan

Spark applies safe optimization rules without changing the result. Examples include:

- Removing unnecessary columns
- Simplifying casts and expressions
- Adding useful null checks
- Moving filters closer to the data source
- Simplifying constant expressions

It answers:

> Can Spark express the same request more efficiently?

### Physical plan

Spark selects executable operations, such as:

- Scanning data
- Filtering records
- Shuffling partitions
- Sorting records
- Selecting a join algorithm

It answers:

> How will Spark execute the request?

### Inspect individual stages

For learning, PySpark's internal Java interface can display individual plans:

```python
query = result_df._jdf.queryExecution()

print("PARSED LOGICAL PLAN")
print(query.logical().toString())

print("ANALYZED LOGICAL PLAN")
print(query.analyzed().toString())

print("OPTIMIZED LOGICAL PLAN")
print(query.optimizedPlan().toString())

print("PHYSICAL PLAN")
print(query.executedPlan().toString())
```

APIs beginning with `_`, such as `_jdf`, are internal APIs. Use `explain("extended")` in normal application code.

## 11. Reading an extended query plan

For this query:

```python
result_df = (
    df
    .filter(col("age") >= 30)
    .select("name", "department", "salary")
    .orderBy(col("salary").desc())
)

result_df.explain("extended")
```

You may see an optimized plan similar to:

```text
Sort [salary#21L DESC NULLS LAST], true
+- Project [name#18, department#20, salary#21L]
   +- Filter (isnotnull(age#19L) AND (age#19L >= 30))
      +- LogicalRDD [id#17L, name#18, age#19L,
                     department#20, salary#21L], false
```

Read a plan tree from the bottom upward:

```text
LogicalRDD → Filter → Project → Sort
```

### Common symbols

| Item | Meaning |
|---|---|
| `+-` | The child operation below another operation |
| `#18` | Spark's internal unique ID for a column |
| `L` | A `long`/`bigint` column |
| `'salary` | A reference that has not yet been fully resolved |
| `DESC` | Descending order |
| `NULLS LAST` | Place missing values after non-null values |
| `true` on `Sort` | Global sort rather than only sorting each partition |

Internal column IDs such as `#18` normally do not matter to application code.

### `LogicalRDD`

```text
LogicalRDD [id, name, age, department, salary], false
```

This is the logical source created from the Python collection. The final `false` means it is not a streaming DataFrame.

### `Filter`

```text
Filter (age#19L >= cast(30 as bigint))
```

The `age` column is a `bigint`, so Spark represents the literal `30` with a compatible type.

The optimized version may become:

```text
Filter (isnotnull(age#19L) AND (age#19L >= 30))
```

Spark simplified the cast and explicitly accounted for null values.

### `Project`

```text
Project [name#18, department#20, salary#21L]
```

In a Spark plan, `Project` means select the listed columns. It corresponds to:

```python
.select("name", "department", "salary")
```

### `Sort`

```text
Sort [salary#21L DESC NULLS LAST], true
```

This sorts all records by salary from highest to lowest, placing null salaries last.

### `Scan ExistingRDD`

In the physical plan:

```text
Scan ExistingRDD[id, name, age, department, salary]
```

Spark reads the DataFrame created from the existing Python data. A DataFrame read from Parquet or CSV would show a file scan instead.

### `Exchange`

```text
Exchange rangepartitioning(
    salary DESC NULLS LAST,
    200
), ENSURE_REQUIREMENTS
```

`Exchange` normally means a **shuffle**: Spark redistributes records between partitions.

A global `orderBy()` requires Spark to arrange records into appropriate salary ranges before sorting them. The `200` is the configured shuffle-partition count:

```python
print(spark.conf.get("spark.sql.shuffle.partitions"))
```

For a tiny dataset, Adaptive Query Execution may combine these into fewer runtime partitions.

### Physical `Sort`

```text
Sort [salary DESC NULLS LAST], true, 0
```

After the shuffle, Spark sorts records inside the correctly arranged partitions. The last `0` is an internal setting that beginners can ignore.

### `AdaptiveSparkPlan`

```text
AdaptiveSparkPlan isFinalPlan=false
```

Adaptive Query Execution (AQE) allows Spark to adjust the physical plan using information collected during execution. It can:

- Combine small shuffle partitions
- Change certain join strategies
- Handle skewed partitions
- Improve partition sizing

`isFinalPlan=false` means the displayed plan is not yet the final runtime-adjusted plan.

Try executing and inspecting again:

```python
result_df.collect()
result_df.explain("formatted")
```

Depending on the Spark version and environment, the plan may then show its final adaptive form.

## 12. Does optimization depend on RDD size?

The optimized logical plan is usually created mainly through **rule-based optimization**, not by examining every row or measuring the actual RDD size.

For example, these logical changes do not require the data size:

- Simplifying `cast(30 as bigint)` to `30`
- Adding `isnotnull(age)`
- Removing unused columns
- Simplifying expressions
- Pushing filters toward a data source

Spark can also use estimated statistics through **cost-based optimization (CBO)**. Statistics can include:

- Estimated row count
- Estimated size in bytes
- Number of distinct values
- Minimum and maximum values
- Null counts

These statistics are particularly helpful for choosing join order and join strategies. A DataFrame created from a Python list or an existing RDD may have little or no reliable size information.

Inspect estimated statistics:

```python
stats = result_df._jdf.queryExecution().optimizedPlan().stats()
print(stats.toString())
```

A very large placeholder such as `8.0 EiB` can mean Spark does not know the actual size; it does not mean the small DataFrame truly occupies that much space.

File-backed formats such as Parquet can provide better metadata than a Python-created RDD:

```python
df.write.mode("overwrite").parquet("employees_parquet")

parquet_df = spark.read.parquet("employees_parquet")
print(parquet_df._jdf.queryExecution().optimizedPlan().stats().toString())
```

The division of responsibility is approximately:

| Stage | Main responsibility |
|---|---|
| Optimized logical plan | Apply logical rules and possibly use estimated statistics |
| Physical planning | Choose executable algorithms and data movement |
| Adaptive execution | Use actual runtime statistics to adjust parts of the physical plan |

For example, physical planning may choose between `BroadcastHashJoin` and `SortMergeJoin`. AQE may change some decisions after seeing actual shuffle sizes.

## 13. Practice exercises

Use the `df` created earlier.

### Exercise 1

Display only `name` and `department`.

```python
df.select("name", "department").show()
```

### Exercise 2

Find employees younger than 30.

```python
df.filter(col("age") < 30).show()
```

### Exercise 3

Find Engineering employees who are at least 30.

```python
df.filter(
    (col("department") == "Engineering") &
    (col("age") >= 30)
).show()
```

### Exercise 4

Sort salaries from highest to lowest.

```python
df.orderBy(col("salary").desc()).show()
```

### Exercise 5

Calculate average age for each department.

```python
df.groupBy("department").agg(
    avg("age").alias("average_age")
).show()
```

### Exercise 6

Count employees by department and sort by count.

```python
df.groupBy("department").count().orderBy(
    col("count").desc()
).show()
```

### Exercise 7

Build transformations, inspect the plans, and then trigger an action.

```python
practice_result = (
    df
    .filter(col("salary") >= 65000)
    .select("name", "salary")
    .orderBy(col("salary").desc())
)

# Inspect without requesting result rows.
practice_result.explain("extended")

# Trigger execution.
practice_result.show()
```

## 14. Quick reference

### DataFrame operations

| Goal | Code |
|---|---|
| Show rows | `df.show()` |
| Show schema | `df.printSchema()` |
| Select columns | `df.select("name", "age")` |
| Filter rows | `df.filter(col("age") >= 30)` |
| Add a column | `df.withColumn("new_age", col("age") + 1)` |
| Sort ascending | `df.orderBy(col("age").asc())` |
| Sort descending | `df.orderBy(col("age").desc())` |
| Group rows | `df.groupBy("department")` |
| Count rows | `df.count()` |
| Inspect plans | `df.explain("extended")` |
| Cache result | `df.cache()` |
| Remove cache | `df.unpersist()` |

### Concepts to remember

1. A transformation returns a new DataFrame and describes work.
2. An action triggers Spark execution.
3. Lazy evaluation allows Spark to optimize the entire query.
4. Read plan trees from the bottom upward.
5. The logical plan describes what to calculate.
6. The physical plan describes how to calculate it.
7. A physical `Exchange` usually indicates a shuffle.
8. AQE can adjust parts of the physical plan using runtime information.
9. Cache only when an expensive result will be reused.
10. Avoid `collect()` on data that may be too large for the driver.

## Suggested next topics

Continue learning in this order:

1. Reading CSV, JSON, and Parquet files
2. Handling missing values
3. Joins
4. Spark SQL
5. Window functions
6. Partitions and shuffles
7. Broadcast joins
8. Data skew
9. Performance tuning

