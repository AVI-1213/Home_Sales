# Home Sales Analysis with SparkSQL

## Overview

This project leverages SparkSQL to analyze home sales data. The goal is to determine key metrics such as average home prices based on various criteria. The tasks include creating temporary views, partitioning data, caching and uncaching tables, and verifying these operations & optimization using Spark.

## Analysis Using SparkSQL

1. **Average Price for Four-Bedroom Houses:**
   - Calculate the average price for four-bedroom houses sold each year.
   - Round the results to two decimal places.
```Python
Query = """( SELECT YEAR(date) AS Year, ROUND(AVG(price),2) AS AveragePrice
FROM home_sales
WHERE bedrooms = 4
GROUP BY YEAR(date)
ORDER BY Year)"""
spark.sql(Query).show(10)
```

2. **Average Price for Three-Bedroom, Three-Bathroom Houses:**
   - Calculate the average price for houses with three bedrooms and three bathrooms for each year they were built.
   - Round the results to two decimal places.
```Python
Query = """( SELECT date_built AS built_Year, ROUND(AVG(price),2) AS AveragePrice
FROM home_sales
WHERE bedrooms = 3 AND bathrooms = 3
GROUP BY date_built
ORDER BY built_Year)"""
spark.sql(Query).show()
```

3. **Average Price for Specific Houses:**
   - Calculate the average price for houses with three bedrooms, three bathrooms, two floors, and at least 2,000 square feet for each year they were built.
   - Round the results to two decimal places.
```Python
Query = """( SELECT date_built AS built_Year, ROUND(AVG(price),2) AS AveragePrice
FROM home_sales
WHERE bedrooms = 3 AND bathrooms = 3
AND floors = 2 AND sqft_living >= 2000
GROUP BY date_built
ORDER BY built_Year)"""
spark.sql(Query).show()
```

4. **Average Price Based on View Rating:**
   - Calculate the average price of a home per "view" rating for homes with an average price of at least $350,000.
   - Record the runtime for this query and round the results to two decimal places.
```Python
Query = """( SELECT ROUND(AVG(price),2) AS avg_price, view as view_rating
FROM home_sales
GROUP BY view_rating
HAVING avg_price >= 350000
ORDER BY view_rating DESC )"""
spark.sql(Query).show()

--- 3.7974278926849365 seconds ---
```

## Caching and Query Performance

1. **Cache Temporary Table:**
   - Cache the `home_sales` temporary table.
```Python
spark.sql("cache table home_sales")
```

2. **Check Cache Status:**
   - Verify if the `home_sales` temporary table is cached.
```Python
spark.catalog.isCached('home_sales')
```

3. **Run Cached Query:**
   - Using the cached data, rerun the query calculating the average price per "view" rating for homes with an average price of at least $350,000.
   - Record the runtime and compare it to the uncached runtime.
```Python
start_time = time.time()
Query = """( SELECT ROUND(AVG(price),2) AS avg_price, view as view_rating
FROM home_sales
GROUP BY view_rating
HAVING avg_price >= 350000
ORDER BY view_rating DESC)"""
spark.sql(Query).show()
print("--- %s seconds ---" % (time.time() - start_time))
--- 1.650547981262207 seconds ---
```

## Data Partitioning

1. **Partition Data: Create Temporary Table for Parquet Data:Run Partitioned Data Query:**
   - Partition the home sales data by the `date_built` field and save it in Parquet format.
   - Create a temporary table for the partitioned Parquet data.
   - Rerun the query calculating the average price per "view" rating for homes with an average price of at least $350,000 on the partitioned data.
   - Record the runtime and compare it to the uncached runtime.
```Python
df.write.partitionBy("date_built").mode("overwrite").parquet("home_sales")
p_home_sales = spark.read.parquet("home_sales")
p_home_sales.createOrReplaceTempView('p_home_p')

```


## Uncaching and Verification

1. **Uncache Temporary Table:**
   - Uncache the `home_sales` temporary table.
```Python
spark.sql("""UNCACHE TABLE home_sales""")
```
2. **Verify Uncache:**
   - Verify that the `home_sales` temporary table is uncached using PySpark.
```Python
if spark.catalog.isCached("home_sales") :
    print("home_sales is still cached")
else :
    print("home_sales is no longer cached")
```
## Results and Discussion
- **Query Performance:**
```python
cache time:
--- 1.650547981262207 seconds ---
uncache time:
--- 3.7974278926849365 seconds ---
```
  - The performance of the cached and partitioned data queries runs less than half of the time compared to their uncached counterparts which highlight the benefits of these optimization techniques.

## Conclusion
This project demonstrates the power of SparkSQL for data analysis, showcasing various techniques to optimize query performance and derive meaningful insights from large datasets.

## Author
- [Avinash] - [https://github.com/AVI-1213]
