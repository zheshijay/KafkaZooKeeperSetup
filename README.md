# KafkaZooKeeperSetup
Kafka ZooKeeper Setup

## 1. Start Service

### 1.1 ZooKeeper
Start ZooKeeper:

```
bin/zkServer.sh start
```

Stop ZooKeeper:
```
bin/zkServer.sh stop
```

### 1.2 Kafka
Start Kafka:
```
 bin/kafka-server-start.sh config/server.properties
```
Start Kafka Broker
```
bin/kafka-server-start.sh config/server.properties
```
Stop Kafka
```
bin/kafka-server-stop.sh config/server.properties
```


## 2. Create Topic

Create Topic:
```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic jay-kafkatest
```

## 3. Send & Receive Messages

Producer:
```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic Hello-Kafka
```
Comsumer:
```
bin/kafka-console-consumer.sh --zookeeper localhost:2181 —topic jay-kafkatest --from-beginning
```
---------------------- 

## Single Node-Multiple Brokers Configuration

```
Step01: Create/Copy N server.properties files.

Step02: Mainly change 3 values for each Kafka broker:
  brokerId
  port
  log.dir

For example, in server-1.properties used for broker1, we define the following:
brokerid=1
port=9092
log.dir=/var/log/kafka01-logs/broker1

Start all brokers:
Broker1
bin/kafka-server-start.sh config/server.properties
Broker2
bin/kafka-server-start.sh config/server-01.properties
Broker3
bin/kafka-server-start.sh config/server-02.properties

```

### Multiple Broker Testing

```
Create Topic: replication-factor is 3, because we have 3 brokers now.
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 -partitions 1 --topic jay-multiple-brokers-topic

Create Producer ( use all 3 brokers ):
bin/kafka-console-producer.sh --broker-list localhost:9092,localhost:9093,localhost:9094 --topic jay-multiple-brokers-topic

Create Consumer:
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic jay-multiple-brokers-topic
```

###
```
package kafka.demo;
import java.util.Properties;
import java.util.Scanner;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;

public class JayKafkaProducer {
	 private static Scanner in;
     public static void main(String[] argv)throws Exception {
    	 
         String topicName = "jay-multiple-brokers-topic";
         in = new Scanner(System.in);
         System.out.println("Enter message(type exit to quit)");

         //Configure the Producer
         Properties configProperties = new Properties();
         configProperties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,"localhost:9092");
         configProperties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,"org.apache.kafka.common.serialization.ByteArraySerializer");
         configProperties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,"org.apache.kafka.common.serialization.StringSerializer");

         org.apache.kafka.clients.producer.Producer producer = new KafkaProducer<String, String>(configProperties);
         String line = in.nextLine();
         while(!line.equals("exit")) {
             ProducerRecord<String, String> rec = new ProducerRecord<String, String>(topicName, line);
             producer.send(rec);
             line = in.nextLine();
         }
         in.close();
         producer.close();
     }
}

```


