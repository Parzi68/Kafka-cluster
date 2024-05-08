# Basic Kafka Cluster
This repository contains the configuration files required for zookeeper and Kafka brokers for launching a basic 3-node kafka cluster locally without any security.

## To check all the brokers connected to the Zookeeper
`
zookeeper-shell.sh localhost:2181
ls /brokers/ids
`

## To check all the topics available on Kafka
`
kafka-topics.sh --zookeeper localhost:2181 --list
`
*For latest versions of Apache Kafka,*<br/>
`
kafka-topics.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --list
`

## To create a topic inside Kafka
`
kafka-topics.sh --zookeeper localhost:2181 --create --topic mytopic --partitions 2 --replication-factor 3
`

*For latest versions of Apache Kafka,*<br/>
`
kafka-topics.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --create --topic mytopic --partitions 2 --replication-factor 3
`
### To create a topic with additional configurations
`
kafka-topics.sh --zookeeper localhost:2181 --create --topic my-topic --partitions 2 --replication-factor 3 --config min.insync.replicas=2
`

*For latest versions of Apache Kafka,*<br/>
`
kafka-topics.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --create --topic my-topic --partitions 2 --replication-factor 3 --config min.insync.replicas=2
`
## To describe a topic
`
kafka-topics.sh --zookeeper localhost:2181 --describe --topic my-topic
`

*For latest versions of Apache Kafka,*<br/>
`
kafka-topics.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --describe --topic my-topic
`
## To Monitor the consumer groups
`
kafka-run-class.sh kafka.admin.ConsumerGroupCommand --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --group first-consumer --describe
`

## To check latest Offset on the partition level for a topic
`
kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092,localhost:9093 --topic mytopic
`

## To delete a topic
### Method 1
`
kafka-topics.sh --zookeeper localhost:2181 --delete --topic topic-name
`
<br /><br /> If the topic has data, then it could take some time and this topic would still be displayed when the command *kafka-topics.sh --list* is executed.

### Method 2
* Stop all the brokers.
* Delete the topic directory inside the path mentioned in *log.dirs* of server.properties.
* Execute *zookeeper-shell.sh* and connect to the zookeeper instance.
* Delete the topic by executing *rmr /brokers/topics/topic-name*. FYI - *rmr* is used to perform recursive delete operations on a node.
* Restart all the brokers.
* Now, the deleted topic should not be displayed when the command *kafka-topics.sh --list* is executed.


## Commands to produce and consume to/from a topic

### To produce message value to a topic
`
kafka-console-producer.sh --broker-list localhost:9092,localhost:9093 --topic first-topic
`

### To consume a topic from beginning
`
kafka-console-consumer.sh --bootstrap-server localhost:9092,localhost:9093 --topic first-topic --from-beginning --group first-consumer
`

### To consume N (for example N=10) messages from a topic
`
kafka-console-consumer.sh --bootstrap-server localhost:9092,localhost:9093 --topic first-topic --max-messages 10 --group first-consumer
`


# SASL_SSL
This repository contains the configuration files for zookeeper, Kafka brokers, producers, and consumers for launching a basic 3-node kafka cluster with SASL_SSL security. Please check the step-by-step [video tutorial](https://www.youtube.com/watch?v=U0XennY3_Ac) on my Youtube channel.

## Note
1. Please remember just for showing a demo, I have shown the password directly inside the property files. Please never do that on a production environment; oor make sure that the property files are placed inside admin-only directories with highly restricted access.
2. For creating kafka topics in secured way (no more using *--zookeeper* and port 2181), please follow the steps mentioned [here](https://github.com/vinclv/data-engineering-minds-kafka/blob/main/config/sasl_ssl/SecureTopicCreation.md).

## kafka-configs.sh commands to create, delete and list SASL/SCRAM credentials
### To create a SASL/SCRAM user
`kafka-configs.sh --zookeeper localhost:2182 --zk-tls-config-file zookeeper-client.properties --entity-type users --entity-name my-user --alter --add-config 'SCRAM-SHA-512=[password=DEM123]'`

### To describe a SASL/SCRAM user
`kafka-configs.sh --zookeeper localhost:2182 --zk-tls-config-file zookeeper-client.properties --entity-type users --entity-name my-user --describe`

### To describe/list all available SASL/SCRAM users
`kafka-configs.sh --zookeeper localhost:2182 --zk-tls-config-file zookeeper-client.properties --entity-type users --describe`

### To delete a SASL/SCRAM user
`kafka-configs.sh --zookeeper localhost:2182 --zk-tls-config-file zookeeper-client.properties --entity-type users --entity-name my-user --alter --delete-config 'SCRAM-SHA-512'`

## Example ACL commands 
### To grant PRODUCER access for the user to the topic *mytopic*
`kafka-acls.sh --authorizer-properties zookeeper.connect=localhost:2182 --zk-tls-config-file zookeeper-client.properties --add --allow-principal User:my-user --operation WRITE --operation DESCRIBE --operation DESCRIBECONFIGS --topic mytopic`

### To grant CONSUMER access for the user to the topic *mytopic* and with consumer group *mycgroup*

`kafka-acls.sh --authorizer-properties zookeeper.connect=localhost:2182 --zk-tls-config-file zookeeper-client.properties --add --allow-principal User:my-user --operation READ --operation DESCRIBE --topic mytopic`

`kafka-acls.sh --authorizer-properties zookeeper.connect=localhost:2182 --zk-tls-config-file zookeeper-client.properties --add --allow-principal User:my-user --operation READ --group mycgroup`

### To list all the ACLs associated with the user
`kafka-acls.sh --authorizer-properties zookeeper.connect=localhost:2182 --zk-tls-config-file zookeeper-client.properties --list --principal User:my-user`


## Commands to produce and consume to/from a topic

### To produce message value to a topic
`
kafka-console-producer.sh --broker-list localhost:9092,localhost:9093 --topic ssl-topic --producer.config producer.properties
`

### To consume a topic from beginning
`
kafka-console-consumer.sh --bootstrap-server localhost:9092,localhost:9093 --topic ssl-topic --consumer.config consumer.properties --from-beginning
`

### To consume N (for example N=10) messages from a topic
`
kafka-console-consumer.sh --bootstrap-server localhost:9092,localhost:9093 --topic ssl-topic --consumer.config consumer.properties --max-messages 10
`

### To consume last N messages - For example, from a specific offset till latest (on partition level). Remember that you need to comment the group.id in consumer.properties
`
kafka-console-consumer.sh --bootstrap-server localhost:9092,localhost:9093 --topic ssl-topic --consumer.config consumer.properties --partition 1 --offset 1
`

### To print additional information along with message such as headers, keys, timestamp, etc.. Remember this works only from Kafka 2.7
`
kafka-console-consumer.sh --bootstrap-server localhost:9092,localhost:9093 --topic ssl-topic --consumer.config consumer.properties --max-messages 10 --property print.headers=true --property print.timestamp=true
`

## Commands for kafka-consumer-groups.sh

### To list all the consumer groups
`
kafka-consumer-groups.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --command-config consumer-group-client.properties --list
`

### To describe the consumer group
`
kafka-consumer-groups.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --command-config consumer-group-client.properties --describe --group sasl-consumer
`

### To describe all the members of the consumer group
`
kafka-consumer-groups.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --command-config consumer-group-client.properties --describe --group sasl-consumer --members
`

`
kafka-consumer-groups.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --command-config consumer-group-client.properties --describe --group sasl-consumer --members --verbose
`

### To describe the state of the consumer group
`
kafka-consumer-groups.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --command-config consumer-group-client.properties --describe --group sasl-consumer --state
`

### To reset the offsets of the consumer group on a topic level

#### To latest offset
`
kafka-consumer-groups.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094  --command-config consumer-group-client.properties --group sasl-consumer --reset-offsets --topic  ssl-topic --to-latest --execute
`

#### To least offset 
`
kafka-consumer-groups.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094  --command-config consumer-group-client.properties --group sasl-consumer --reset-offsets --topic  ssl-topic --to-earliest --execute
`

#### To shift by number of offsets
`
kafka-consumer-groups.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094  --command-config consumer-group-client.properties --group sasl-consumer --dry-run --reset-offsets --topic  ssl-topic --shift-by 1 --execute
`

#### To perform dry run without executing
`
kafka-consumer-groups.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094  --command-config consumer-group-client.properties --group sasl-consumer --dry-run --reset-offsets --topic  ssl-topic --to-latest
`

#### To reset offsets on partition level 
`
kafka-consumer-groups.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094  --command-config consumer-group-client.properties --group sasl-consumer --reset-offsets --topic  ssl-topic:0 --to-latest --execute
`

`
kafka-consumer-groups.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094  --command-config consumer-group-client.properties --group sasl-consumer --reset-offsets --topic  ssl-topic:1 --to-latest --execute
`

