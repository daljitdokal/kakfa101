## Introduction

### What Is Apache Kafka?

Apache Kafka is an event streaming platform used to collect, process, store, and integrate data at scale. It has numerous use cases including distributed logging, stream processing, data integration, and pub/sub messaging.

In order to make complete sense of what Kafka does, we'll delve into what an "event streaming platform" is and how it works. So before delving into Kafka architecture or its core components, let's discuss what an event is. This will help explain how Kafka stores events, how to get events in and out of the system, and how to analyze event streams.


### What Are Events?

An event is any type of action, incident, or change that's identified or recorded by software or applications. For example, a payment, a website click, or a temperature reading, along with a description of what happened.

In other words, an event is a combination of notification—the element of when-ness that can be used to trigger some other activity—and state. That state is usually fairly small, say less than a megabyte or so, and is normally represented in some structured format, say in JSON or an object serialized with Apache Avro™ or Protocol Buffers.


### Kafka and Events – Key/Value Pairs

Kafka is based on the abstraction of a distributed commit log. By splitting a log into partitions, Kafka is able to scale-out systems. As such, Kafka models events as key/value pairs. Internally, keys and values are just sequences of bytes, but externally in your programming language of choice, they are often structured objects represented in your language’s type system. Kafka famously calls the translation between language types and internal bytes serialization and deserialization. The serialized format is usually JSON, JSON Schema, Avro, or Protobuf.

Values are typically the serialized representation of an application domain object or some form of raw message input, like the output of a sensor.

Keys can also be complex domain objects but are often primitive types like strings or integers. The key part of a Kafka event is not necessarily a unique identifier for the event, like the primary key of a row in a relational database would be. It is more likely the identifier of some entity in the system, like a user, order, or a particular connected device.

This may not sound so significant now, but we’ll see later on that keys are crucial for how Kafka deals with things like parallelization and data locality.

### Kafka Topics

Events have a tendency to proliferate—just think of the events that happened to you this morning—so we’ll need a system for organizing them. Apache Kafka's most fundamental unit of organization is the topic, which is something like a table in a relational database. As a developer using Kafka, the topic is the abstraction you probably think the most about. You create different topics to hold different kinds of events and different topics to hold filtered and transformed versions of the same kind of event.

A topic is a log of events. Logs are easy to understand, because they are simple data structures with well-known semantics. First, they are append only: When you write a new message into a log, it always goes on the end. Second, they can only be read by seeking an arbitrary offset in the log, then by scanning sequential log entries. Third, events in the log are immutable—once something has happened, it is exceedingly difficult to make it un-happen. The simple semantics of a log make it feasible for Kafka to deliver high levels of sustained throughput in and out of topics, and also make it easier to reason about the replication of topics, which we’ll cover more later.

Logs are also fundamentally durable things. Traditional enterprise messaging systems have topics and queues, which store messages temporarily to buffer them between source and destination.

Since Kafka topics are logs, there is nothing inherently temporary about the data in them. Every topic can be configured to expire data after it has reached a certain age (or the topic overall has reached a certain size), from as short as seconds to as long as years or even to retain messages indefinitely. The logs that underlie Kafka topics are files stored on disk. When you write an event to a topic, it is as durable as it would be if you had written it to any database you ever trusted.

The simplicity of the log and the immutability of the contents in it are key to Kafka’s success as a critical component in modern data infrastructure—but they are only the beginning.

### Kafka Partitioning

If a topic were constrained to live entirely on one machine, that would place a pretty radical limit on the ability of Apache Kafka to scale. It could manage many topics across many machines—Kafka is a distributed system, after all—but no one topic could ever get too big or aspire to accommodate too many reads and writes. Fortunately, Kafka does not leave us without options here: It gives us the ability to partition topics.

Partitioning takes the single topic log and breaks it into multiple logs, each of which can live on a separate node in the Kafka cluster. This way, the work of storing messages, writing new messages, and processing existing messages can be split among many nodes in the cluster.

#### How Partitioning Works

Having broken a topic up into partitions, we need a way of deciding which messages to write to which partitions. Typically, if a message has no key, subsequent messages will be distributed round-robin among all the topic’s partitions. In this case, all partitions get an even share of the data, but we don’t preserve any kind of ordering of the input messages. If the message does have a key, then the destination partition will be computed from a hash of the key. This allows Kafka to guarantee that messages having the same key always land in the same partition, and therefore are always in order.

For example, if you are producing events that are all associated with the same customer, using the customer ID as the key guarantees that all of the events from a given customer will always arrive in order. This creates the possibility that a very active key will create a larger and more active partition, but this risk is small in practice and is manageable when it presents itself. It is often worth it in order to preserve the ordering of keys.


### Kafka Brokers

So far we have talked about events, topics, and partitions, but as of yet, we have not been too explicit about the actual computers in the picture. From a physical infrastructure standpoint, Apache Kafka is composed of a network of machines called brokers. In a contemporary deployment, these may not be separate physical servers but containers running on pods running on virtualized servers running on actual processors in a physical datacenter somewhere. However they are deployed, they are independent machines each running the Kafka broker process. Each broker hosts some set of partitions and handles incoming requests to write new events to those partitions or read events from them. Brokers also handle replication of partitions between each other.

### Replication

It would not do if we stored each partition on only one broker. Whether brokers are bare metal servers or managed containers, they and their underlying storage are susceptible to failure, so we need to copy partition data to several other brokers to keep it safe. Those copies are called follower replicas, whereas the main partition is called the leader replica. When you produce data to the leader—in general, reading and writing are done to the leader—the leader and the followers work together to replicate those new writes to the followers.

This happens automatically, and while you can tune some settings in the producer to produce varying levels of durability guarantees, this is not usually a process you have to think about as a developer building systems on Kafka. All you really need to know as a developer is that your data is safe, and that if one node in the cluster dies, another will take over its role.

### Kafka Producers

The API surface of the producer library is fairly lightweight: In Java, there is a class called KafkaProducer that you use to connect to the cluster. You give this class a map of configuration parameters, including the address of some brokers in the cluster, any appropriate security configuration, and other settings that determine the network behavior of the producer. There is another class called ProducerRecord that you use to hold the key-value pair you want to send to the cluster.

To a first-order approximation, this is all the API surface area there is to producing messages. Under the covers, the library is managing connection pools, network buffering, waiting for brokers to acknowledge messages, retransmitting messages when necessary, and a host of other details no application programmer need concern herself with.

### Kafka Consumers

Using the consumer API is similar in principle to the producer. You use a class called KafkaConsumer to connect to the cluster (passing a configuration map to specify the address of the cluster, security, and other parameters). Then you use that connection to subscribe to one or more topics. When messages are available on those topics, they come back in a collection called ConsumerRecords, which contains individual instances of messages in the form of ConsumerRecord objects. A ConsumerRecord object represents the key/value pair of a single Apache Kafka message.

KafkaConsumer manages connection pooling and the network protocol just like KafkaProducer does, but there is a much bigger story on the read side than just the network plumbing. First of all, Kafka is different from legacy message queues in that reading a message does not destroy it; it is still there to be read by any other consumer that might be interested in it. In fact, it’s perfectly normal in Kafka for many consumers to read from one topic. This one small fact has a positively disproportionate impact on the kinds of software architectures that emerge around Kafka, which is a topic covered very well elsewhere.

Also, consumers need to be able to handle the scenario in which the rate of message consumption from a topic combined with the computational cost of processing a single message are together too high for a single instance of the application to keep up. That is, consumers need to scale. In Kafka, scaling consumer groups is more or less automatic.

## Kafka Connect

In the world of information storage and retrieval, some systems are not Apache Kafka. Sometimes you would like the data in those other systems to get into Kafka topics, and sometimes you would like data in Kafka topics to get into those systems. As Apache Kafka's integration API, this is exactly what Kafka Connect does.

#### What Does Kafka Connect Do?

On the one hand, Kafka Connect is an ecosystem of pluggable connectors, and on the other, a client application. As a client application, Connect is a server process that runs on hardware independent of the Kafka brokers themselves. It is scalable and fault-tolerant, meaning you can run not just one single Connect worker but a cluster of Connect workers that share the load of moving data in and out of Kafka from and to external systems. Kafka Connect also abstracts the business of code away from the user and instead requires only JSON configuration to run. For example, here’s how you’d stream data from Kafka to Elasticsearch:

```
{
    "connector.class": "io.confluent.connect.elasticsearch.ElasticsearchSinkConnector",
    "topics"         : "my_topic",
    "connection.url" : "http://elasticsearch:9200",
    "type.name"      : "_doc",
    "key.ignore"     : "true",
    "schema.ignore"  : "true"
}
```

### How Kafka Connect Works

A Connect worker runs one or more connectors. A connector is a pluggable component that is responsible for interfacing with the external system. A source connector reads data from an external system and produces it to a Kafka topic. A sink connector subscribes to one or more Kafka topics and writes the messages it reads to an external system. Each connector is either a source or a sink connector, but it is worthwhile to remember that the Kafka cluster only sees a producer or a consumer in either case. Everything that is not a broker is a producer or a consumer.

### Benefits of Kafka Connect

One of the primary advantages of Kafka Connect is its large ecosystem of connectors. Writing the code that moves data to a cloud blob store, or writes to Elasticsearch, or inserts records into a relational database is code that is unlikely to vary from one business to the next. Likewise, reading from a relational database, Salesforce, or a legacy HDFS filesystem is the same operation no matter what sort of application does it. You can definitely write this code, but spending your time doing that doesn’t add any kind of unique value to your customers or make your business more uniquely competitive.

All of these are examples of Kafka connectors available in the Confluent Hub, a curated collection of connectors of all sorts and most importantly, all licenses and levels of support. Some are commercially licensed and some can be used for free. Confluent Hub lets you search for source and sink connectors of all kinds and clearly shows the license of each connector. Of course, connectors need not come from the Hub and can be found on GitHub or elsewhere in the marketplace. And if after all that you still can’t find a connector that does what you need, you can write your own using a fairly simple API.

Now, it might seem straightforward to build this kind of functionality on your own: If an external source system is easy to read from, it would be easy enough to read from it and produce to a destination topic. If an external sink system is easy to write to, it would again be easy enough to consume from a topic and write to that system. But any number of complexities arise, including how to handle failover, horizontally scale, manage commonplace transformation operations on inbound or outbound data, distribute common connector code, configure and operate this through a standard interface, and more.

Connect seems deceptively simple on its surface, but it is in fact a complex distributed system and plugin ecosystem in its own right. And if that plugin ecosystem happens not to have what you need, the open-source Connect framework makes it simple to build your own connector and inherit all the scalability and fault tolerance properties Connect offers.

