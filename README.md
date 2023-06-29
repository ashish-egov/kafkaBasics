# kafkaBasics

# Topics, Partitions, and Offsets

Apache Kafka is a  distributed streaming platform  that enables the publishing and subscription of streams of records. A record is a key-value pair that contains a payload of data. In  Kafka, records are organized into topics.

A  **topic**  is a category or  feed name  to which records are published. Topics are partitioned for scalability and efficiency. A partition is an ordered, immutable sequence of records that are continually appended to. Each partition is assigned a unique identifier called a partition id.

Kafka maintains a numerical offset for each record within a partition. The offset is a  sequential id number  that uniquely identifies each record within the partition. In other words, the offset represents the position of a record within a partition.

## Partitioning

Partitioning allows for the  horizontal scaling  of Kafka by distributing the load across multiple brokers. Each partition is assigned to a specific broker in the Kafka cluster. Producers write records to a specific partition, and consumers read records from a specific partition.

Partitioning also provides fault tolerance. If a broker fails, its partitions can be assigned to other brokers in the cluster.

When creating a topic, you can specify the number of partitions to use. A good  rule of thumb  is to have at least as many partitions as the number of consumers in the consumer group that will be consuming from the topic.

## Offsets

Offsets are used to keep track of the position of a consumer within a partition. A consumer can choose to start reading from any offset within a partition. Once a consumer has read a record, its offset is considered "consumed". The consumer can commit its offset to Kafka, which allows it to resume reading from where it left off in case it is restarted.

Kafka also provides the ability to store offsets externally, such as in a database, which provides more flexibility and control over  offset management.

## Example

Let's say we have a Kafka topic called "orders". We want to partition the topic into 3 partitions to allow for horizontal scaling and  fault tolerance. We have 2 producers writing records to the topic, and 4 consumers reading records from the topic.

Each producer will write records to a specific partition. For example, producer 1 may write records to  partition 1  and producer 2 may write records to partition 2. This allows for  load balancing  across the partitions.

Each consumer will read records from a specific partition. For example, consumer 1 may read records from partition 1 and consumer 2 may read records from partition 2. This allows for parallel processing of records by the consumers.

Each record within a partition is assigned a unique offset. For example, partition 1 may contain records with offsets 1, 2, 3, and 4, while partition 2 may contain records with offsets 1, 2, 3, 4, 5, and 6.

If a consumer reads a record with offset 3 from partition 1, its offset is considered "consumed". If the consumer restarts, it can resume reading from offset 4 in partition 1.

# Producers and Message Keys

In Kafka, a  **producer**  is a client that publishes records to a Kafka topic. When a producer sends a record to Kafka, it can specify a  message key  along with the record's value. The message key is an optional field that can be used for partitioning and  message ordering.

## Partitioning with  Message Keys

If a message key is specified, Kafka uses a  hash function  to determine the partition to which the record will be written. This ensures that all records with the same key are written to the same partition, which allows for message ordering and grouping.

For example, let's say we have a Kafka topic called "orders". We want to partition the topic into 3 partitions to allow for horizontal scaling and fault tolerance. We have a producer writing records to the topic, and we want to ensure that all records with the same order id are written to the same partition.

To achieve this, the producer can specify the  order id  as the message key when sending a record to Kafka. Kafka will use a hash function to determine the partition based on the message key. All records with the same order id will be written to the same partition, which allows for easy retrieval and processing of all records with the same order id.

## Ordering with Message Keys

If message ordering is important, a producer can also use the message key to ensure that records with the same key are written to the same partition in the order they were sent. This allows for easy retrieval and processing of records in the order they were produced.

For example, let's say we have a Kafka topic called "logins". We want to ensure that all records with the same  user id  are written to the same partition in the order they were produced. To achieve this, the producer can specify the user id as the message key when sending a record to Kafka. Kafka will use the message key to determine the partition, and all records with the same user id will be written to the same partition in theorder they were produced.

## Example

Let's say we have a Kafka topic called "sales". We want to partition the topic into 3 partitions to allow for horizontal scaling and fault tolerance. We have a producer writing records to the topic, and we want to ensure that all records for the same product are written to the same partition.

To achieve this, the producer can specify the  product id  as the message key when sending a record to Kafka. Kafka will use a hash function to determine the partition based on the message key. All records with the same product id will be written to the same partition, which allows for easy retrieval and processing of all records for the same product.

If the producer wants to ensure that records for the same product are written to the partition in the order they were produced, it can use the product id as the message key and ensure that the records are produced in the order they should be written to Kafka. This allows for easy retrieval and processing of records in the order they were produced.

In summary, producers can use message keys to ensure that records are written to specific partitions based on the key, which allows for easy retrieval and processing of records with the same key. Message keys can also be used to ensure that records are written to the partition in the order they were produced, which allows for easy retrieval and processing of records in the order they were produced.

# Consumers and Deserialization

In  Kafka, a  **consumer**  is a client that subscribes to one or more Kafka topics and reads records from them. When a consumer reads a record from Kafka, it receives a key-value pair that contains a payload of data. The payload can be in any format, such as  JSON,  Avro, or binary.

## Deserialization

Before a consumer can process a record's payload, it must first deserialize it into a usable form. Deserialization is the process of converting the binary data in a record's payload into a  structured format  that can be understood by the consumer.

Kafka supports multiple  serialization formats, including JSON, Avro, and binary. The choice of  serialization format  depends on the use case and the data being stored in Kafka.

## Example

Let's say we have a Kafka topic called "orders". We have a  producer writing records  to the topic, and each record contains an  order id, a  customer id, and a list of items in the order. The payload is in JSON format.

To read records from the "orders" topic, we create a consumer that subscribes to the topic. When the consumer reads a record from Kafka, it receives a key-value pair that contains the order id as the key and the payload as the value.

Before the consumer can process the payload, it must first deserialize it from JSON format into a usable form. To do this, the consumer can use a  JSON deserializer  that converts the JSON data into a structured format, such as a  Java object  or a Map.

For example, the  Java code  below shows how a consumer can deserialize a record's payload from  JSON format  into a  Java  object:

```
import com.fasterxml.jackson.databind.ObjectMapper;

public class OrderDeserializer implements Deserializer<Order> {
    private final ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public Order deserialize(String topic, byte[] data) {
        try {
            return objectMapper.readValue(data, Order.class);
        } catch (IOException e) {
            throw new RuntimeException("Error deserializing order", e);
        }
    }
}

```

In this example, we use the  Jackson JSON library  to deserialize the record's payload into a Java object of type  `Order`. The  `deserialize`  method takes the topic name and the binary data of the record's value as arguments, and returns a deserialized  `Order`  object.

Once the consumer has deserialized the record's payload, it can process the data in any way it chooses. For example, it could write the data to a database, perform some analysis on the data, or send the data to another system for further processing.

In summary, consumers read records from Kafka topics and receive key-value pairs that contain payloads in various serialization formats. Before processing the payload, the consumer must first deserialize it into a usable form. Kafka supports multiple serialization formats, and the choice of serialization format depends on the use case and the data being stored in Kafka.

# Consumer Groups and Consumer Offsets

In  Kafka, a  **consumer group**  is a set of consumers that collectively consume records from one or more Kafka topics. Consumer groups allow for  parallel processing  of records and provide  fault tolerance.

Each consumer in a consumer group reads records independently from the other consumers in the group. The records are distributed across the consumers in a group based on the number of partitions in the topics being consumed.

## Consumer Offsets

Kafka keeps track of the position of a consumer within a partition using a numerical offset. When a consumer reads records from a partition, it can choose to commit its offset to Kafka. This allows the consumer to resume reading from where it left off in case it is restarted.

The  committed offset  is stored in a Kafka internal topic called the "__consumer_offsets" topic. Each record in this topic contains the consumer group, the topic, the partition, and the committed offset for a particular consumer.

## Example

Let's say we have a Kafka topic called "orders" that is partitioned into 3 partitions. We have a consumer group consisting of 2 consumers that are reading records from the "orders" topic.

Each consumer in the group reads records independently from the other consumer. For example, consumer 1 may read records from  partition 1, while consumer 2 may read records from partition 2. This allows for parallel processing of records.

When a consumer reads a record, it also receives the record's offset within the partition. For example, consumer 1 may read a record from partition 1 with an offset of 100. If consumer 1 commits its offset to Kafka, the committed offset for consumer 1 in the "__consumer_offsets" topic will be updated to 100.

If consumer 1 is restarted, it can resume reading from offset 101 in partition 1. This allows for fault tolerance and ensures that each consumer in the group is reading from a unique position within the partition.

## Rebalancing

Consumer groups also support automatic rebalancing. If a new consumer is added to a group or an existing consumer is removed, Kafka will automatically rebalance the partitions across the consumers in the group.

For example, let's say we have a consumer group consisting of 3 consumers that are reading records from the "orders" topic. Each consumer is assigned a partition to read from, and the partitions are distributed evenly across the consumers.

If a new consumer is added to the group, Kafka will automatically rebalance the partitions across the 4 consumers in the group. Each consumer will be assigned a new set of partitions to read from, and the partitions will be distributed evenly across the consumers.

## Offset Management

Kafka provides several APIs for managing consumer offsets. Consumers can commit their offsets manually, or they can use  automatic offset management, which periodically commits offsets based on a specified time interval or number of records.

Kafka also provides the ability to store offsets externally, such as in a database, which provides more flexibility and control over  offset management.

In summary, consumer groups allow for parallel processing of records and provide fault tolerance. Kafka keeps track of the position of a consumer within a partition using a  numerical offset, which allows consumers to resume reading from where they left off in case they are restarted. Kafka also supports  automatic rebalancing  of partitions across consumers in a group and provides several  APIs  for managing consumer offsets.
