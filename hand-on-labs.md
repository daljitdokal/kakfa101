# Kafka 101

--------------------------------------------------------------
### NOTE: Begin by setting up an account with Confluent Cloud.

#### Signup

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

## Hands On: Partitioning

```
confluent kafka topic list

# In particular, make note of the num.partitions value, which is 6.
confluent kafka topic describe poems

# num.partitions value = 1
confluent kafka topic create --partitions 1 poems_1

# Produce data to the topics using the produce command and --parse-key flag and then enter the strings:
confluent kafka topic produce poems_1 --parse-key

	1:”All that is gold does not glitter”
	2:"Not all who wander are lost"
	3:"The old that is strong does not wither"
	4:"Deep roots are not harmed by the frost"
	5:"From the ashes a fire shall awaken"
	6:"A light from the shadows shall spring"
	7:"Renewed shall be blad that was broken"
	8:"The crownless again shall be king"

# num.partitions value = 4
confluent kafka topic create --partitions 4 poems_4

# Produce data to the topics using the produce command and --parse-key flag and then enter the strings:
confluent kafka topic produce poems_4 --parse-key

	1:”All that is gold does not glitter”
	2:"Not all who wander are lost"
	3:"The old that is strong does not wither"
	4:"Deep roots are not harmed by the frost"
	5:"From the ashes a fire shall awaken"
	6:"A light from the shadows shall spring"
	7:"Renewed shall be blad that was broken"
	8:"The crownless again shall be king"
```
Using the Jump to offset field, explore the partitions of all three topics—poems, poems_1, and poems_4—to observe how the messages are distributed differently across these topics.
