---
layout: post
title: Processing &amp; Querying data in Cassandra with Apache Spark
tags:
- Spark
- Cassandra
- Data Engineering
- Big Data
---

This post is one of my <a href="new.html">Notes to Self</a> one. I'm simply going to write, how can you connect to Cassandra from Spark, run "SQL" queries and perform analysis on Cassandra's data.

<br>
Let's get started.

<br>
(Platform: Spark v1.6.0, Cassandra v2.7, macOS 10.12.1, Scala 2.11.7)

I'm going to use the package `spark-cassandra-connector` written by awesome <a href="http://www.datastax.com/">Datastax</a> guys.

Assuming you have already configured Cassandra & Spark, it's time to start writing a small Spark job.

<b>Code with explanation</b>:
{% highlight python %}
# imports necessary methods
from pyspark import SparkContext, SparkConf
from pyspark.sql import SQLContext


# setup spark configuration object
# this contains your cassandra connection parameters
conf = SparkConf()\
    .setAppName("PySpark Cassandra") \
    .set("spark.cassandra.connection.host", "192.168.56.101")\
    .set("spark.cassandra.auth.username", "cassandra")\
    .set("spark.cassandra.auth.password", "cassandra")


# creates Spark Context with your cassandra configurations
# local[*] represents that spark is going to use all cores of CPU for this job
sc = SparkContext("local[*]", "PySpark Cassandra", conf=conf)


# creates Spark's sqlContext. This is going to be super useful.
sqlContext = SQLContext(sc)


# creates mapping with tables inside Cassandra for Spark
# I am going to use "system_auth.roles" table here
sqlContext.sql("""CREATE TEMPORARY TABLE roles \
                  USING org.apache.spark.sql.cassandra \
                  OPTIONS ( table "roles", \
                            keyspace "system_auth", \
                            cluster "rootCSSCluster", \
                            pushdown "true") \
              """)

# you can create multiple mappings/temporary tables and write queries on it.
# this in another table: "system.compaction_history"
sqlContext.sql("""CREATE TEMPORARY TABLE compaction_history \
                  USING org.apache.spark.sql.cassandra \
                  OPTIONS ( table "compaction_history", \
                            keyspace "system", \
                            cluster "rootCSSCluster", \
                            pushdown "true") \
              """)


# here's the query we are going to run
query = "SELECT * FROM roles"

print "[Spark] Executing query: %s" % (query)

# The result of the query returns a dataframe
df_payload = sqlContext.sql(query)

# priting the content of dataframe
df_payload.show()
{% endhighlight %}


<br>

<b>Spark job Execution</b>:

To run your spark job, use the command below:
<pre>
spark-submit --packages com.datastax.spark:spark-cassandra-connector_2.10:1.5.0-M2 myfile.py
</pre>

(Note: Check `localhost:4040` in your browser for Spark UI)

-\-packages : This parameter tells Spark to download the external dependencies for the job.

In our case, we are using `spark-cassandra-connector`:


```bash
groupId: com.datastax.spark
artifactId: spark-cassandra-connector_2.10
version: 1.5.0-M2
```

<br>

<b>Output</b>:

```bash
<all your logs will be printed here. including ivy logs>
:
:
[Spark] Executing query: select * from roles
+---------+---------+------------+---------+--------------------+
|     role|can_login|is_superuser|member_of|         salted_hash|
+---------+---------+------------+---------+--------------------+
|cassandra|     true|        true|       []|$2a$10$pQW3iGSC.m...|
+---------+---------+------------+---------+--------------------+
```

<br>
Here, `df_payload` is DataFrame object. You can use all Spark's <i>Transformations</i> &amp; <i>Actions</i> on this. (Check <a href="http://spark.apache.org/docs/latest/sql-programming-guide.html">here</a> for more details)

<br>
<b>Second Part</b>: <u>Starting a Spark shell with Cassandra connection</u>
<br>
Steps for this is part of separate <a href="processing-cassandra-data-with-apache-spark-part-2.html">post</a>.

<hr>
<b>Useful Links </b>:-

1. I really want to thank guys at <a href="http://www.datastax.com/">Datastax</a>. They have written and open sourced, so many packages and drivers for Cassandra.

2. You can contribute to `spark-cassandra-connector` <a href="https://github.com/datastax/spark-cassandra-connector">here</a>.

3. <a href="https://mvnrepository.com/artifact/com.datastax.spark/spark-cassandra-connector_2.10/1.5.0-M2">Link</a> to `spark-cassandra-connector` maven repository.
