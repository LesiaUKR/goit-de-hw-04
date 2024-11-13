
# HW | Apache Spark | Optimization and SparkUI

Hello 😉

In this homework, the code is already written for you! But that might only make the assignment harder! 
Your task is to run several variations of one/similar code and consider the appearance of SparkUI.

You need to:

- Run three programs
- Take screenshots of the three sets of Jobs
- Analyze and justify the presence of a specific number of Jobs in each set
- Understand what the cache function does and why it is used.


## Step-by-Step Execution Guide

### Part 1
We'll take code that is already familiar to you and add an intermediate action:

```python
from pyspark.sql import SparkSession

# Create Spark session
spark = SparkSession.builder \
    .master("local[*]") \
    .config("spark.sql.shuffle.partitions", "2") \
    .appName("MyGoitSparkSandbox") \
    .getOrCreate()

# Load dataset
nuek_df = spark.read \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .csv('./nuek-vuh3.csv')

nuek_repart = nuek_df.repartition(2)

nuek_processed = nuek_repart \
    .where("final_priority < 3") \
    .select("unit_id", "final_priority") \
    .groupBy("unit_id") \
    .count()

# Intermediate action added here
nuek_processed = nuek_processed.where("count>2")

nuek_processed.collect()

input("Press Enter to continue...5")

# Close Spark session
spark.stop()
```
Run the code. Take a screenshot of all Jobs (there should be 5).

### Part 2
Add an intermediate action collect:

```python

from pyspark.sql import SparkSession

# Create Spark session
spark = SparkSession.builder \
    .master("local[*]") \
    .config("spark.sql.shuffle.partitions", "2") \
    .appName("MyGoitSparkSandbox") \
    .getOrCreate()

# Load dataset
nuek_df = spark.read \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .csv('./nuek-vuh3.csv')

nuek_repart = nuek_df.repartition(2)

nuek_processed = nuek_repart \
    .where("final_priority < 3") \
    .select("unit_id", "final_priority") \
    .groupBy("unit_id") \
    .count()

# Intermediate action: collect
nuek_processed.collect()

# Additional line added here
nuek_processed = nuek_processed.where("count>2")

nuek_processed.collect()

input("Press Enter to continue...5")

# Close Spark session
spark.stop()
```
Run the code. Take a screenshot of all Jobs (there should be 8).

🧠 Think: Why does adding one intermediate action nuek_processed.collect() result in 3 more Jobs?

### Part 3
Use the cache function on the intermediate result.

☝🏻The cache() function in PySpark is used to cache (store in memory) data from an RDD (Resilient Distributed Dataset) or DataFrame. This can speed up the execution of subsequent actions or transformations on the same data, as PySpark won’t have to recompute the same data repeatedly.

#### How cache() works:

1. **In-Memory Caching:** When you call cache() on an RDD or DataFrame, the data is stored in memory (RAM) distributed across all nodes in the cluster, allowing for faster calculations.
2. **Lazy Execution:** Calling cache() doesn’t immediately execute the calculation. Only when an action like count(), collect(), or show() is performed will the data be computed and cached.
3. **Storage Mechanism:** By default, cache() uses memory. If data doesn’t fit into memory, Spark will store it on disk.
4. **Cache Control:** Using cache() saves data with the storage level MEMORY_ONLY. Other levels, like MEMORY_AND_DISK, can be used with persist().

5. No need to dive too deep into the details. Just know that memory storage is common, and disk storage is rare 😉.

```python

from pyspark.sql import SparkSession

# Create Spark session
spark = SparkSession.builder \
    .master("local[*]") \
    .config("spark.sql.shuffle.partitions", "2") \
    .appName("MyGoitSparkSandbox") \
    .getOrCreate()

# Load dataset
nuek_df = spark.read \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .csv('./nuek-vuh3.csv')

nuek_repart = nuek_df.repartition(2)

nuek_processed_cached = nuek_repart \
    .where("final_priority < 3") \
    .select("unit_id", "final_priority") \
    .groupBy("unit_id") \
    .count() \
    .cache()  # Added cache function

# Intermediate action: collect
nuek_processed_cached.collect()

# Additional line added here
nuek_processed = nuek_processed_cached.where("count>2")

nuek_processed.collect()

input("Press Enter to continue...5")

# Release memory from DataFrame
nuek_processed_cached.unpersist()

# Close Spark session
spark.stop()
```
Run the code. Take a screenshot of all Jobs (there should be 7).

🧠 Think: Why does using cache() reduce the number of Jobs?