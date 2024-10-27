
# Home Sales Analysis with PySpark

This project uses PySpark to perform data analysis on a dataset of home sales, examining trends such as average home prices based on attributes like the number of bedrooms, bathrooms, year built, and view rating. The dataset is sourced from an AWS S3 bucket in CSV format and saved as a Parquet file partitioned by build year for efficient querying.

## Setup

### Requirements
- PySpark
- Findspark (for local PySpark setup)
- Access to an AWS S3 bucket for data retrieval

### Initialization
1. **Initialize PySpark**:
   ```python
   import findspark
   findspark.init()
   ```
2. **Create a Spark Session**:
   ```python
   from pyspark.sql import SparkSession
   spark = SparkSession.builder.appName("SparkSQL").getOrCreate()
   ```

## Data Loading and Preparation

1. **Load CSV Data**:  
   The dataset is read from an AWS S3 bucket and loaded into a PySpark DataFrame.
   ```python
   from pyspark import SparkFiles
   url = "https://2u-data-curriculum-team.s3.amazonaws.com/dataviz-classroom/v1.2/22-big-data/home_sales_revised.csv"
   spark.sparkContext.addFile(url)
   home_sales = spark.read.csv(SparkFiles.get("home_sales_revised.csv"), sep=",", header=True)
   ```

2. **Create a Temporary View**:  
   A temporary view named `home_sales` is created to allow SQL-based queries.
   ```python
   home_sales.createOrReplaceTempView("home_sales")
   ```

## Analysis Tasks

The following queries are executed to analyze different aspects of the data:

1. **Average Price for Four-Bedroom Homes Per Year**:  
   Calculates the average sale price for homes with four bedrooms, grouped by the sale date, rounded to two decimal places.

2. **Average Price of 3-Bedroom, 3-Bathroom Homes by Build Year**:  
   Retrieves the average price for homes with three bedrooms and three bathrooms, grouped by the `date_built` field.

3. **Average Price of Large, Two-Story 3-Bedroom, 3-Bathroom Homes by Build Year**:  
   Filters homes with three bedrooms, three bathrooms, two floors, and at least 2000 square feet of living space. The average price is calculated per build year.

4. **Average Price by View Rating for High-Value Homes**:  
   Computes the average price of homes by `view` rating, where the average price is at least $350,000, ordered by view rating in descending order.

## Performance Optimization

1. **Caching**:  
   The `home_sales` table is cached to improve query performance for repeated analysis tasks.
   ```python
   spark.catalog.cacheTable("home_sales")
   ```

2. **Runtime Comparison**:  
   The cached and uncached runtimes for the `view` rating query are compared. The cached version is slightly faster.

3. **Partitioned Parquet File**:  
   The DataFrame is saved as a Parquet file partitioned by `date_built`, which can help optimize future queries based on the build year.
   ```python
   home_sales.write.partitionBy("date_built").parquet("home_sales_partitioned")
   ```

## Cleanup

1. **Uncache Table**:  
   After analysis, the `home_sales` table is uncached to free up memory.
   ```python
   spark.catalog.uncacheTable("home_sales")
   ```

2. **Check Cache Status**:  
   Verify if the table is no longer cached.
   ```python
   is_cached = spark.catalog.isCached("home_sales")
   print(f"Is 'home_sales' cached? {is_cached}")
   ```

## Additional Notes

- **Parquet Format**: Parquet data format is more efficient for storage and query optimization.
- **Cached Data**: Cached data significantly reduces query runtime, especially for large datasets.
