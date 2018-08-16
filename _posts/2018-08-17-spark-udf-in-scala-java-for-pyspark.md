---
layout: post
title: Pyspark&#58; Why you should write UDFs in Scala/Java
tags:
- Spark
- Big Data
- Python
- Scala
---

While writing Pyspark programs, we would generally write UDFs in Python, which is a very obvious thing to do. However, the performance of Python UDFs are not as good as those written in Scala/Java.

Spark is written in Scala and Data/objects of a Spark program are stored in JVM. Pyspark API is just a wrapper over SparkSession, RDDs/DataFrame and other JVM objects (a few parts are in native python as well). This means that a Pyspark program goes through serialization and deserialization of JVM objects and data. This back and forth conversion affects the performance of Pyspark program drastically. Pyspark UDFs are a good example where this conversion happens a lot.

<br>
<b>Example Implementation</b>:

Here is a very simple Java UDF to count length of a string:

```java
package com.rootcss;

import org.apache.spark.sql.api.java.UDF1;

public class SparkJavaUdfExample implements UDF1<String, Integer> {
    public Integer call(String input) {
        return input.length();
    }
}
```
(full Java code is available here: https://github.com/rootcss/PysparkJavaUdfExample)
<br><br>

Once compiled, you can start pyspark by including the jar in the path.
```bash
bin/pyspark --jars /path/to/target/SparkJavaUdfExample-1.0-SNAPSHOT.jar
```
<br>

Here's the usage in Pyspark:

```bash
>>> from pyspark.sql.types import IntegerType
>>> # Now, register the UDF in Pyspark
>>> sqlContext.registerJavaFunction("myCustomUdf", "com.rootcss.SparkJavaUdfExample", IntegerType())
>>> spark.sql("SELECT myCustomUdf('shekhar')").collect()
+------------+
|UDF(shekhar)|
+------------+
|           7|
+------------+
```

<br>
<b>References</b>:
- Slide #59 in https://0x0fff.com/wp-content/uploads/2015/11/Spark-Architecture-JD-Kiev-v04.pdf
- This post covers this whole topic in more details - https://medium.com/wbaa/using-scala-udfs-in-pyspark-b70033dd69b9









