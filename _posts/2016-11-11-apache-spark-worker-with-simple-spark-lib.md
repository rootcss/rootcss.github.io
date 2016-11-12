---
layout: post
title: Writing Apache Spark workers with "Simple Spark Lib"
tags:
- Spark
- Cassandra
- Data Engineering
---

<a target="_blank" href="http://spark.apache.org/">Apache Spark</a> is a great project, could be plugged with most of the data sources/databases eg, HDFS, Cassandra, MongoDB, Kafka, Postgres, Redshift etc. I have been using Spark for ad-hoc querying, bunch of Aggregations &amp; Segregations over Cassandra from a long time and noticed that, every time I use to write (or paste) same code for configuration &amp; connection. Also, I knew when someone else wants to do the similar work from my team, he/she will have to do the same thing, including learning what that means and understanding it. Think of someone doing that, if he is using Spark for the first time?

<br>TLDR;

I decided to write a wrapper over `PySpark` which obviously supports Cassandra, Redshift etc. It primarily provided following two advantages:
1. I never repeated myself while writing the workers again
2. My Team members do not need to figure out those Spark specific code in order to do some simple ad-hoc tasks

I named it "`Simple Spark Lib`" and, here's how to use it:

Step 1: Clone the repo from <a href="https://github.com/rootcss/simple_spark_lib">here</a>:
{% highlight bash %}
git clone https://github.com/rootcss/simple_spark_lib.git
{% endhighlight %}
Step 2: Install the library:
{% highlight bash %}
python setup.py install
{% endhighlight %}
Step 3: Write the worker:
{% highlight python %}
# First, import the library
from simple_spark_lib import SimpleSparkCassandraWorkflow

# Define connection configuration for cassandra
cassandra_connection_config = {
  'host':     '192.168.56.101',
  'username': 'cassandra',
  'password': 'cassandra'
}

# Define Cassandra Schema information
cassandra_config = {
  'cluster': 'rootCSSCluster',
  'tables': {
    'api_events': 'events_production.api_events',
  }
}
# Initiate your workflow
workflow = SimpleSparkCassandraWorkflow(appName="Simple Example Worker")

# Setup the workflow with configurations
workflow.setup(cassandra_connection_config, cassandra_config)

# Run your favourite query
df = workflow.process(query="SELECT * FROM api_events LIMIT 10")

print df.show()
{% endhighlight %}
Step 4: Save it &amp; Execute the worker:
{% highlight bash %}
simple-runner my_spark_woker.py -d cassandra
{% endhighlight %}

`simple_spark_lib` enables you to use the capability of spark without writing the actual Spark codes. I made it public, hoping it might be useful to someone else too.

If you are interested, go through other examples in the repo and feel free to contribute. :-)
