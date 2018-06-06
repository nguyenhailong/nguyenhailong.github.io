---
layout: post
title: "Hive Performance Checklist"
author: long_nguyen
modified:
excerpt: "Hive Performance Checklist"
tags: []
---
Apache Hive is a data warehouse software project built on top of Hadoop for providing data summarization, query and analysis. Hive is a SQL-like language so that it is easy to use and becomes popular to programmers. However, one of the biggest challenges Hive users face is the slow response time. 

Below is my collection checklist for optimizing Hive performance:

## USE TEZ (Hive 0.13+)
Apache Tez provides more efficient processing than the MapReduce execution engine, by reducing operations and limiting the amount of intermediate data that is written to disk.

As show in below figure, the traditional MapReduce execution engine has several steps in which the intermediate data from the reducers are written back to HDFS, which incurs the performance penalty for disk I/O. Contrast this with the data flow of the Tez execution engine shown on the right side, where the reducer’s intermediate data is passed directly to the next reducer in the execution plan and bypasses the expense of writing the data to disk.

![Tez example](https://goo.gl/dKwvvB)

```SQL
set hive.execution.engine=tez;
set hive.prewarm.enabled=true;
set hive.prewarm.numcontainers=10;
```
## USE Optimized Row Columnar (ORC) Format
The ORC format is a column-based storage format, meaning that rather than storing all of the data for an individual row of data consecutively on disk, the data for each column of storage contiguously instead. This helps to avoid unnecessary disk access for queries that do not contain certain columns, by “skipping over” large sections of data not needed in the results.

```SQL
CREATE TABLE A_ORC (
customerID int, name string, age int, address string
) STORED AS ORC tblproperties (“orc.compress" = “SNAPPY”);
```

## USE VECTORIZATION
Vectorized query execution improves performance of operations like scans, aggregations, filters and joins, by performing them in batches of 1024 rows at once instead of single row each time. Introduced in Hive 0.13, this feature significantly improves query execution time, and is easily enabled with two parameters
settings:

```SQL
set hive.vectorized.execution.enabled = true;
set hive.vectorized.execution.reduce.enabled = true;
```

## COST BASED QUERY OPTIMIZATION (Hive0.14+)
The cost-based optimization (CBO) engine uses statistics in the Hive Metastore to produce optimal query plans. There are two types of statistics that are used for optimization: table stats, which include the uncompressed size of the table, number of rows, and number of files used to store the data, and column stats, which include NDV (number of distinct values) and min/max/count values.

The CBO does join reordering, improves plans for star and bushy join schemas, and provides opportunistic improvements based on sample queries. The downside of the CBO is the fact that you must gather and maintain accurate statistics about your tables in order for the cost-based optimization engine to be effective. Unfortunately, the collection of table statistics is an expensive operation, but the benefits can be reaped on all subsequent queries involving the table for which statistics were collected.

```SQL
set hive.cbo.enable=true;
set hive.compute.query.using.stats=true;
set hive.stats.fetch.column.stats=true;
set hive.stats.fetch.partition.stats=true;
ANALYZE TABLE weather_ORC COMPUTE STATISTICS;
```
## Use STREAMTABLE for JOIN queries

Joins play an important role and the order of joining table becomes very important. When doing table joins, Hadoop starts from the last table listed and joins to the left. Most of us put the largest table as the first table in the list, meaning it is last in the joining process. This then requires more resources to process.

- Put the large table as the last table in the list so Hadoop starts here
- Or, use SELECT /* +STREAMTABLE(<table name>) */ .... Streamtable tells Hadoop which table to start with.

## Use MAPJOIN to speedup join queries

Mapjoin is a little-known feature of Hive. It allows a table to be loaded into memory so that a (very fast) join could be performed entirely within a mapper without having to use a Map/Reduce step. If your queries frequently rely on small table joins (e.g. cities or countries, etc.) you might see a very substantial speed-up from using mapjoins. For example: 

```SQL
SELECT /*+ MAPJOIN(c) */ * FROM orders o JOIN cities c ON (o.city_id = c.id);
```
Alternative way is to turn on mapjoins is to let Hive do it automatically. Simply set hive.auto.convert.join to true in your config, and Hive will automatically use mapjoins for any tables smaller than hive.mapjoin.smalltable.filesize (default is 25MB).

## Use OLAP functionality (OVER and RANK) instead of Join wherever possible
For example, we would like to find the latest URL for each sessionID. One might consider the following approach:
```SQL
SELECT click_event.* FROM click_event inner join 
(select session_ID, max(time_stamps) as max_ts from click_event group by session_ID) latest ON click_event.session_ID = latest.session_ID
AND click_event.time_stamps = latest.max_ts;
```
In the above query, we build a sub-query to collect the timestamp of the latest event in each session, and then use an inner join to filter out the rest. While the query is a reasonable solution—from a functional point of view—it turns out there is a better way to re-write this query as follows:

```SQL
SELECT * FROM
(SELECT *, RANK() over (partition by session_ID, order by time_stamps DESC) as rank FROM click_event) ranked_clicks
WHERE ranked_clicks.rank=1;

```

## Use SORT BY and DISTRIBUTE BY instead of ORDER BY wherever possible

*ORDER BY takes only single reducer* to process the data which may take an unacceptably long time to execute for longer data sets.
Hive provides an alternative, *SORT BY, that orders the data only within each reducer* and performs a local ordering where each reducer’s output will be sorted. Better performance is traded for total ordering.
In both cases, the syntax differs only by the use of the ORDER or SORT keyword. We can specify any columns you wish and specify whether or not the columns are ascending using the ASC keyword (the default) or descending using the DESC keyword.
Here is an example using ORDER BY:

```SQL
SELECT s.year_month_date, s.symbol, s.price_close
FROM stocks s
ORDER BY s.year_month_date ASC, s.symbol DESC;
```
Here is the same example using SORT BY instead:
```SQL
SELECT s.year_month_date, s.symbol, s.price_close
FROM stocks s
SORT BY s.year_month_date ASC, s.symbol DESC;
```
The two queries look almost identical, but in the second case, if more than one reducer is invoked, the output will be sorted differently. While each reducer’s output files will be sorted, the data will probably overlap with the output of other reducers.
Here Hive provides *DISTRIBUTE BY* with *SORT BY* which controls how map output is divided among reduces.
The idea is like all data that flows through a MapReduce job is organized into key-value pairs and Hive must use this feature internally when it converts your queries to MapReduce jobs. By default, MapReduce computes a hash on the keys output by mappers and tries to evenly distribute the key-value pairs among the available reducers using the hash values.
```SQL
SELECT s.year_month_date, s.symbol, s.price_close
FROM stocks s
DISTRIBUTE BY s.symbol
SORT BY s.symbol ASC, s.year_month_date ASC;
```
DISTRIBUTE BY works similar to GROUP BY in the sense that it controls how reducers receive rows for processing, while SORT BY controls the sorting of data inside the reducer.

## HAVING clause for filtering the rows not for other purpose

HAVING clause is used to filter the rows after all the rows are selected. It is just like a filter. We should avoid HAVING clause for any other purposes. Considering below example:

Use:
```SQL
SELECT cellName, count(cellName)
FROM internal_channel_switching
WHERE cellName != ‘1059’ AND cellName != ‘5730’
GROUP BY cellName;
```
Instead of:
```SQL
SELECT cellName, count(cellName)
FROM internal_channel_switching
GROUP BY cellName
HAVING cellName != ‘1059’ AND cellName != ‘5730’;
```

## Minimize number of subquery blocks in the query 
Sometimes you may have more than one subqueries in your main query. Try to minimize the number of subquery block in your query. Considering below example:

Use:
```SQL
SELECT name
FROM employee
WHERE (salary, age) = (SELECT MAX (salary), MAX (age)
FROM employee_details) AND emp_dept = ‘Computer Science;
```
Instead of:
```SQL
SELECT name
FROM employee
WHERE salary = (SELECT MAX(salary) FROM employee_details)
AND age = (SELECT MAX(age) FROM employee_details) AND emp_dept = ‘Computer Science’;
```

## Use operator EXISTS, IN and table joins appropriately in your query
- Usually IN has the slowest performance.
- IN is efficient when most of the filter criteria is in the sub-query.
- EXISTS is efficient when most of the filter criteria is in the main query.

Considering below example

Use:
```SQL
Select * from product p
where EXISTS (select * from order_items o
where o.product_id = p.product_id);
```
Instead of:
```SQL
Select * from product p
where product_id IN (select product_id from order_items);
```

## Give importance to the conditions in WHERE clause 

Recommended
```SQL
SELECT emp_id, first_name, salary FROM employee WHERE salary > 50000;
```
Not recommended
```SQL
SELECT emp_id, first_name, salary FROM employee WHERE salary != 50000;
```

Recommended
```SQL
SELECT emp_id, first_name, salary
FROM employee
WHERE first_name LIKE 'Pravat%';
```
Not recommended
```SQL
SELECT emp_id, first_name, salary
FROM employee
WHERE SUBSTR(first_name,1,3) = 'Pra';
```

Recommended
```SQL
SELECT emp_id, first_name, salary
FROM employee
WHERE first_name LIKE NVL ( :name, '%');
```
Not Recommended
```SQL
SELECT emp_id, first_name, salary
FROM employee
WHERE first_name = NVL ( :name, first_name);
```

Recommended
```SQL
SELECT product_id, product_name
FROM product
WHERE unit_price BETWEEN MAX(unit_price) and MIN(unit_price);
```
Not Recommended
```SQL
SELECT product_id, product_name
FROM product
WHERE unit_price >= MAX(unit_price)
and unit_price <= MIN(unit_price);
```

Recommended
```SQL
SELECT emp_id, first_name, salary
FROM employee WHERE dept = 'ComputerScience'
AND location = 'Singapore';
```
Not Recommended
```SQL
SELECT emp_id, first_name, salary
FROM employee
WHERE dept || location= 'ComputerScienceSingapore';
```

Use non-column expression on one side of the query because it will be processed earlier.