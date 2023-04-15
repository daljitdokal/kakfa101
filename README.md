# Kafka 101

--------------------------------------------------------------
## NOTE: Begin by setting up an account with Confluent Cloud.

### Signup

Please use following details to signup with Confluent Cloud and learn kafka 101 with labs.

- Url: https://www.confluent.io/
- Course: KAFKA101 - 30 days with $400 free create for cloud cluster with option to select AWS/google cloud
- Coupon code to add extra $25 credit: KAKFA101

--------------------------------------------------------------

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


--------------------------

### KAFKA 101

https://developer.confluent.io/learn-kafka/apache-kafka/events/

 - Create Cluster
 - Create topic
   - create message
- Install CLI
  - CLI: Create API-key
  - CLI: Use Producer and Consumer for messages

```
curl -L --http1.1 https://cnfl.io/cli | sh -s -- -b /usr/local/bin
confluent update

confluent login --save

confluent environment list
confluent environment use {ID}

confluent kafka cluster list
confluent kafka cluster use {ID}

confluent api-key create --resource {ID}
confluent api-key use {API Key} --resource {ID}

# Produce and Consume Using the Confluent CLI
confluent kafka topic list
```

### Terminal 1

```
confluent kafka topic consume --from-beginning poems
```

### Terminal 2

```
confluent kafka topic produce poems --parse-key
```

When prompted, enter the following strings as written:

```
4:"Message from DJ"
5:"From the ashes a fire shall awaken"
6:"A light from the shadows shall spring"
7:"Renewed shall be blad that was broken"
8:"The crownless again shall be king"
```

Observe the messages as they’re being output in the consumer terminal window.

