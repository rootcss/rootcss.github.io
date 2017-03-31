---
layout: post
title: Observations on querying Cassandra on 'multiple' partitions (with/without Spark)
tags:
- Spark
- Cassandra
- Spark
- Big Data
---

Cassandra's brilliancy totally depends on your data models. You should know beforehand about how the data will be accessed/queried; and then design accordingly.

If you're querying a Cassandra table, you are going to start writing your query with the <b>partition key</b>, because as we know, the partition key tells about the data locality in the cluster. Writing a query that includes multiple partition keys is never optimized, because those keys might be on different nodes. Just assume, you have 500 nodes with RF=3 and each node is being scanned for those partition keys.

It's going to be super expensive.

For example, Let's say, I have a `users` table like this:

(partition key - id, clustering key - event_timestamp)
```
id | event_timestamp | city
1  | abc............ | A
1  | def............ | B
2  | abc............ | B
3  | abc............ | C
:
:
```

Now, If I write a query like:

`SELECT * FROM users where id = 1`

This is perfectly optimized, and thanks to our Murmur3 partitioner we will get the result instantly.

<br>
However, if I write a query like:

`SELECT * FROM users where id IN (1, 3, 4, 9, 123, 25, 345, 56, 457, 58, 768, 5435, 2, 547, 456, 345, 2342, 34....)`

On a small cluster this will cause no major issues, but on a 500 nodes cluster, it's going to affect the JVM's Heap badly, as explained above.

<br>
Now, coming to <b>Spark</b>.

On a small scale, you wouldn't even notice the problem. Not just with Spark, but even with CQLSH you wouldn't notice the delay and issues significantly.
However, If your cluster is significantly large, it will be very slow and highly unoptimized, and we don't really like that, right.

<a href="https://github.com/datastax/spark-cassandra-connector" target="_blank">`cassandra-spark-connector`</a> has a method called `joinWithCassandraTable()` to which you can pass a list of partition keys to be looked up.

Internally, this method extracts all the partition keys from the list, and runs a separate parallel query (spark tasks) for each partition key on our "distributed" Spark cluster (it uses Cassandra Java driver to perform this operation). Finally returns an RDD object consisting of results from all tasks.

So, our 2nd query was converted into something like this,
```sql
SELECT * FROM users where id = 1
SELECT * FROM users where id = 3
SELECT * FROM users where id = 4
SELECT * FROM users where id = 123
SELECT * FROM users where id = 25
SELECT * FROM users where id = 345
:
```

Usage of the method:
```scala
val myList = sc.parallelize(partition_keys).map(Tuple1(_))
val myResult = myList.joinWithCassandraTable(keyspace, "users")
```

We cannot say this is an extremely optimized solution, but considering the huge number of advantages that we get from Cassandra, we can compromise a bit here ;-)

And by the way, this method is not yet available for Pyspark, only in Scala. I am attempting to write one for Pyspark, will be sharing the details soon.