---
layout: post
title: Processing Rabbitmq's Stream with "Apache Flink"
tags:
- Flink
- Rabbitmq
- Data Engineering
- Big Data
---

I love <a target="_blank" href="http://spark.apache.org/">Apache Spark</a>. Not just becacuse of it's capability to adapt to so many use-cases, but because it's one of shining star in the <i>Distributing Computing</i> world, has a great design and superb community backing.

However, one of the features I'd want enhancement in is, the way it processes the streams. Spark processes the streams in a <b>micro-batch</b> manner i.e, you set a time interval (could be any value), and Spark will process the events collected in that interval, in batch. This is where Apache Flink comes in!

<a target="_blank" href="http://spark.apache.org/">Apache Flink</a> is often comapred with Spark. I feel Spark is far ahead of Flink, not just in technology; but even community backing of Spark is very big, compared to Flink.

Anyways, this post is not about comparing them, but to provide a detailed example of processing a RabbitMQ's stream using Apache Flink.

<b>Step 1:</b> Install Rabbitmq, Apache Flink in your system. Both installations are very straightforward.

<b>Step 2:</b> Start Rabbitmq server
```bash
rabbitmq-server &
```

<b>Step 3:</b> Create an exchange in it. Go to `http://localhost:15672` (In my example, I'm binding a queue to the exchange. You can directly use a queue, but make sure to make corresponding changes in the code)

<b>Step 4:</b> Clone the repo from <a target="_blank" href="https://github.com/rootcss/flink-rabbitmq.git
">here</a>: (will be explaining the codes inline)
```html
git clone https://github.com/rootcss/flink-rabbitmq.git
```
<b>Step 5:</b> It's built with maven. (Java) So, build it using:
```bash
mvn clean package
```
<b>Step 6:</b> Once built, You're all set to run it now:
```bash
flink run -c com.rootcss.flink.RabbitmqStreamProcessor target/flink-rabbitmq-0.1.jar
```
<b>Step 7:</b> Check the logs at:
```bash
tail -f $FLINK_HOME/log/*
```
and Flink's dashboard at: 
```html
http://localhost:8081/
```
<b>Step 8:</b> Now, you can start publishing events from the RabbitMQ's exchange and see the output in the logs.

Note that, I am not using any <b>Flink's Sink</b> here (writing into the logs). You can use a file system like HDFS or a Database or even Rabbitmq (on a different channel ;))


### Code Explanation
(This version might be a little different from the code in my repo. Just to keep this concise)
```java
// Extend the RMQSource class, since we need to override a method to bind our queue
public class RabbitmqStreamProcessor extends RMQSource{

    // This is mainly because we have to bind our queue to an exchange. If you are using a queue directly, you may skip it
    @Override
    protected void setupQueue() throws IOException {
        AMQP.Queue.DeclareOk result = channel.queueDeclare("simple_dev", true, false, false, null);
        channel.queueBind(result.getQueue(), "simple_exchange", "*");
    }

    public static void main(String[] args) throws Exception {
        // Setting up rabbitmq's configurations; ignore the default values
        RMQConnectionConfig connectionConfig = new RMQConnectionConfig.Builder()
                .setHost("localhost").setPort(5672).setUserName("rootcss")
                .setPassword("password").setVirtualHost("/").build();

        // below ones are pretty intuitive class names, right?
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // Finally adding Rabbitmq as source of the stream for Flink
        DataStream<String> dataStream = env.addSource(new RMQSource<String>(connectionConfig,
                "simple_dev",
                new SimpleStringSchema()));

        // Accepting the events, and doing a flatMap to calculate string length of each event (to keep the things easy)
        DataStream<Tuple2<String, Integer>> pairs = dataStream.flatMap(new TextLengthCalculator());

        // action on the pairs, you can plug your Flink's Sink here as well.
        pairs.print();

        // Start the execution of the worker
        env.execute();
    }
```

<style>
pre code{
  white-space: pre;
}
</style>

And, here is the beautiful web interface of Apache Flink:

<p><img class="img-responsive" src="assets/images/2016-11-12-apache-flink-rabbimq-streams-processor_1.png" alt="Flink Web Dashboard" /></p>
<br>
In the next post, I will be explaining how I bomarded events on both Spark & Flink, to compare their endurance. Just for fun :-D

Stay Tuned!