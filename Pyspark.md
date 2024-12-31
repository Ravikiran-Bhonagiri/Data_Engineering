# PySpark Operations ReadMe

## Table of Contents

1. [Introduction](#introduction)
2. [Reading Files](#reading-files)
3. [DBUtils Functions](#dbutils-functions)
4. [Marker Functions in Databricks Notebooks](#marker-functions-in-databricks-notebooks)
5. [Schema and StructType Examples](#schema-and-structtype-examples)
6. [DataFrame Transformations](#dataframe-transformations)
7. [String Functions](#string-functions)
8. [Date Functions](#date-functions)
9. [Handling Nulls](#handling-nulls)
10. [Split and Indexing Functions](#split-and-indexing-functions)
11. [Explode Function](#explode-function)
12. [GroupBy and Aggregations](#groupby-and-aggregations)
13. [Joins](#joins)
14. [Window Functions](#window-functions)
15. [User Defined Functions (UDFs)](#user-defined-functions)
16. [Writing Data](#writing-data)
17. [Spark SQL Transformations](#spark-sql-transformations)

## Introduction
This guide provides an overview of major PySpark operations, including transformations, actions, schema manipulations, and SQL integrations. Each function is explained with concise descriptions and code examples.

## Reading Files
### Reading CSV
```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("ReadFile").getOrCreate()

df_csv = spark.read.csv("path/to/file.csv", header=True, inferSchema=True)
df_csv.show()
```

### Reading JSON
```python
df_json = spark.read.json("path/to/file.json")
df_json.show()
```

### Reading Parquet
```python
df_parquet = spark.read.parquet("path/to/file.parquet")
df_parquet.show()
```

## DBUtils Functions
Databricks `dbutils` provides utility methods for tasks like file system operations.

### Common Functions
#### List Files
```python
dbutils.fs.ls("/mnt/myfolder")
```

#### Mount Storage
```python
dbutils.fs.mount(source="s3://mybucket", mount_point="/mnt/mybucket", extra_configs=configs)
```

#### Display Widgets
```python
dbutils.widgets.text("name", "default", "Enter Name")
```

## Marker Functions in Databricks Notebooks
### Common Marker Functions
#### `%md` - Markdown
```markdown
%md
# This is a Markdown cell
```

#### `%fs` - File System
```shell
%fs ls /mnt/myfolder
```

#### `%sql` - SQL Query
```sql
%sql
SELECT * FROM my_table
```

## Schema and StructType Examples

### DDL_SCHEMA Example
```python
schema = "id INT, name STRING, age INT"
df = spark.read.schema(schema).csv("path/to/file.csv")
df.printSchema()
```

### StructType Example
```python
from pyspark.sql.types import StructType, StructField, IntegerType, StringType

schema = StructType([
    StructField("id", IntegerType(), True),
    StructField("name", StringType(), True),
    StructField("age", IntegerType(), True)
])
df = spark.read.schema(schema).json("path/to/file.json")
df.printSchema()
```

## DataFrame Transformations

### Basic Transformations
#### Select and Alias
```python
df.select("name", "age").alias("user_data").show()
```

#### Filter (Single and Multiple Conditions)
```python
df.filter(df.age > 18).show()
df.filter((df.age > 18) & (df.name == "John")).show()
```

#### withColumnRenamed
```python
df = df.withColumnRenamed("name", "full_name")
df.show()
```

#### withColumn (Type Casting)
```python
df = df.withColumn("age", df["age"].cast("String"))
df.show()
```

#### Sort and Drop
```python
df.sort("age").show()
df.drop("address").show()
```

#### drop_duplicates
```python
df.drop_duplicates(["name"]).show()
```

#### Union and Union by Name
```python
df_union = df1.union(df2)
df_union_by_name = df1.unionByName(df2)
df_union.show()
```

## String Functions
### Initcap, Upper, Lower
```python
from pyspark.sql.functions import initcap, upper, lower

df.select(initcap("name"), upper("name"), lower("name")).show()
```

## Date Functions
### Common Date Functions
```python
from pyspark.sql.functions import current_date, date_add, date_sub, datediff, date_format

df.select(current_date(), date_add(current_date(), 5), date_sub(current_date(), 5),
          datediff(current_date(), "start_date"),
          date_format(current_date(), "yyyy-MM-dd")).show()
```

## Handling Nulls
### Dropping Nulls
```python
df.na.drop("all").show()
df.na.drop("any").show()
df.na.drop(subset=["name", "age"]).show()
```

### Filling Nulls
```python
df.na.fill("unknown", subset=["name"]).show()
df.na.fill(0).show()
```

## Split and Indexing Functions
```python
from pyspark.sql.functions import split

df = df.withColumn("first_name", split(df["full_name"], " ")[0])
df.show()
```

## Explode Function
```python
from pyspark.sql.functions import explode

df_exploded = df.withColumn("elements", explode(df.array_column))
df_exploded.show()
```

## GroupBy and Aggregations
```python
df.groupBy("department").agg({"salary": "avg", "age": "max"}).show()
```

## Joins
### Different Joins
```python
df_inner = df1.join(df2, "id", "inner")
df_left = df1.join(df2, "id", "left")
df_outer = df1.join(df2, "id", "outer")
```

## Window Functions
```python
from pyspark.sql.window import Window
from pyspark.sql.functions import row_number, rank, dense_rank

window_spec = Window.partitionBy("department").orderBy("salary")
df = df.withColumn("row_number", row_number().over(window_spec))
```

## User Defined Functions
```python
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType

def uppercase(name):
    return name.upper()

udf_uppercase = udf(uppercase, StringType())
df = df.withColumn("upper_name", udf_uppercase(df["name"]))
df.show()
```

## Writing Data
### Writing Formats
```python
df.write.csv("output.csv")
df.write.parquet("output.parquet")
df.write.format("delta").save("output_delta")
```

### Write Modes
```python
df.write.mode("append").csv("output.csv")
df.write.mode("overwrite").csv("output.csv")
```

## Spark SQL Transformations
### Create Temp View and Query
```python
df.createOrReplaceTempView("my_table")
sql_df = spark.sql("SELECT * FROM my_table WHERE age > 18")
sql_df.show()
```

