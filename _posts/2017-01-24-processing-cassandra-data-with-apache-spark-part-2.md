---
layout: post
title: Spark Shell for Processing &amp; Querying data in Cassandra
tags:
- Spark
- Cassandra
- Data Engineering
- Big Data
---

My previous <a href="processing-cassandra-data-with-apache-spark.html">post</a>  explains, how can you write a Spark job and execute it. In this post, I am writing down steps to initiate a Spark shell (pyspark or spark-shell), with a pre-established connection to Cassandra. In addition to this, I'll write down some sample codes and their outputs, in order to show the usage of Spark Transformations/Actions.


To start the shell, just run this command on your shell.
<br>
<pre>
pyspark \
      --packages com.datastax.spark:spark-cassandra-connector_2.10:1.5.0-M2 \
      --conf spark.cassandra.connection.host=192.168.56.101 \
      --conf spark.cassandra.auth.username=cassandra \
      --conf spark.cassandra.auth.password=cassandra
</pre>

I think, most of the parameters are pretty intuitive. In short, we are just providing the dependency packages and cassandra connection configurations. Make sure you provide path to `pyspark` or add it in your `$PATH`.

```bash
Python 2.7.11 (default, Nov 10 2016, 03:37:47)
[GCC 4.2.1 Compatible Apple LLVM 8.0.0 (clang-800.0.42.1)] on darwin

Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 1.6.0
      /_/

Using Python version 2.7.11 (default, Nov 10 2016 03:37:47)
SparkContext available as sc, HiveContext available as sqlContext.


>>> sqlContext.sql("""CREATE TEMPORARY TABLE roles \
...                   USING org.apache.spark.sql.cassandra \
...                   OPTIONS ( table "roles", \
...                             keyspace "system_auth", \
...                             cluster "rootCSSCluster", \
...                             pushdown "true") \
...               """)
DataFrame[]


>>> sqlContext.sql('SELECT * from roles').show()
+---------+---------+------------+---------+--------------------+
|     role|can_login|is_superuser|member_of|         salted_hash|
+---------+---------+------------+---------+--------------------+
|cassandra|     true|        true|       []|$2a$10$pQW3iGSC.m...|
+---------+---------+------------+---------+--------------------+


>>> sqlContext.sql("""CREATE TEMPORARY TABLE compaction_history \
...                   USING org.apache.spark.sql.cassandra \
...                   OPTIONS ( table "compaction_history", \
...                             keyspace "system", \
...                             cluster "rootCSSCluster", \
...                             pushdown "true") \
...               """)
DataFrame[]


>>> sqlContext.sql('SELECT * FROM compaction_history LIMIT 3').show()
+--------------------+--------+---------+-----------------+--------------------+-------------+
|                  id|bytes_in|bytes_out|columnfamily_name|        compacted_at|keyspace_name|
+--------------------+--------+---------+-----------------+--------------------+-------------+
|b54ccf00-e236-11e...|   20729|     4301|   size_estimates|2017-01-24 18:42:...|       system|
|170ddba0-e23f-11e...|   12481|     3085| sstable_activity|2017-01-24 19:42:...|       system|
|914d2e70-e195-11e...|      41|        0|  schema_triggers|2017-01-23 23:28:...|       system|
+--------------------+--------+---------+-----------------+--------------------+-------------+


>>> sqlContext.sql('SELECT columnfamily_name, COUNT(*) AS count FROM compaction_history GROUP BY columnfamily_name').show()
+-----------------+-----+
|columnfamily_name|count|
+-----------------+-----+
| schema_functions|    2|
|schema_aggregates|    2|
|  schema_triggers|    2|
| schema_usertypes|    2|
|   size_estimates|    2|
| sstable_activity|    4|
+-----------------+-----+


>>> sqlContext.sql('SELECT SUM(bytes_in) AS sum FROM compaction_history').show()
+-----+
|  sum|
+-----+
|73610|
+-----+


>>> df_payload = sqlContext.sql('SELECT bytes_in, bytes_out, columnfamily_name FROM compaction_history')
>>> df_payload.count()
14


# Simple Map Reduce example
>>> df_payload\
...      .map(lambda x: (x['columnfamily_name'], 1))\
...      .reduceByKey(lambda x, y: x+y)\
...      .toDF()\
...      .show()
+-----------------+---+
|               _1| _2|
+-----------------+---+
|   size_estimates|  2|
| sstable_activity|  4|
| schema_usertypes|  2|
|schema_aggregates|  2|
| schema_functions|  2|
|  schema_triggers|  2|
+-----------------+---+

>>>

```

<br>
I'll be adding more examples with time. :)


<style>
pre code{
  white-space: pre;
}
</style>
