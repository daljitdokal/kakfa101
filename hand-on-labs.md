# Kafka 101

--------------------------------------------------------------
## NOTE: Begin by setting up an account with Confluent Cloud.

### Signup

Please use following details to signup with Confluent Cloud and learn kafka 101 with labs.

- Url: https://www.confluent.io/
- Course: KAFKA101 - 30 days with $400 free create for cloud cluster with option to select AWS/google cloud
- Coupon code to add extra $25 credit: KAKFA101

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

```
## Produce and Consumer Using the Confluent CLI

### Terminal 1 - Consumer

```
confluent kafka topic list
confluent kafka topic consume --from-beginning poems
```

### Terminal 2 - Producer

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

