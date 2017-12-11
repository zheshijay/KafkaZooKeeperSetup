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
Start Kafka/Brokers:
```
 Single Broker:
 bin/kafka-server-start.sh config/server.properties
 
 
 Multiple Brokers:
nohup sudo bin/kafka-server-start.sh config/server-01.properties  </dev/null >/dev/null 2>&1 &
nohup sudo bin/kafka-server-start.sh config/server-02.properties  </dev/null >/dev/null 2>&1 &
nohup sudo bin/kafka-server-start.sh config/server-03.properties  </dev/null >/dev/null 2>&1 &
nohup sudo bin/kafka-server-start.sh config/server-04.properties  </dev/null >/dev/null 2>&1 &
nohup sudo bin/kafka-server-start.sh config/server-05.properties  </dev/null >/dev/null 2>&1 &
```

Stop Kafka
```
bin/kafka-server-stop.sh config/server.properties
```


## 2. Create Topic

Create Topic:
```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic kafkatest
```

## 3. Send & Receive Messages

Producer:
```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic kafkatest
```
Comsumer:
```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic kafkatest
```
---------------------- 

## Single Node-Multiple Brokers Configuration

```
Step01: Create/Copy N server.properties files.


```
Broker01  9092
Broker02  9093
Broker03  9094
Broker04  9095
Broker05  9096

```

jay@kafka-dev-01:/usr/local/kafka/kafka_2.11-0.11.0.2/config$ ls -ltr
-rw-r--r-- 1 root root 1023 Nov 10 23:47 zookeeper.properties
-rw-r--r-- 1 root root 1032 Nov 10 23:47 tools-log4j.properties
-rw-r--r-- 1 root root 1900 Nov 10 23:47 producer.properties
-rw-r--r-- 1 root root 4696 Nov 10 23:47 log4j.properties
-rw-r--r-- 1 root root 1199 Nov 10 23:47 consumer.properties
-rw-r--r-- 1 root root 2730 Nov 10 23:47 connect-standalone.properties
-rw-r--r-- 1 root root 1111 Nov 10 23:47 connect-log4j.properties
-rw-r--r-- 1 root root  881 Nov 10 23:47 connect-file-source.properties
-rw-r--r-- 1 root root  883 Nov 10 23:47 connect-file-sink.properties
-rw-r--r-- 1 root root 5807 Nov 10 23:47 connect-distributed.properties
-rw-r--r-- 1 root root  909 Nov 10 23:47 connect-console-source.properties
-rw-r--r-- 1 root root  906 Nov 10 23:47 connect-console-sink.properties
-rw-r--r-- 1 root root 7014 Dec  9 00:46 server-01.properties
-rw-r--r-- 1 root root 7014 Dec  9 00:46 server-02.properties
-rw-r--r-- 1 root root 7014 Dec  9 00:47 server-03.properties
-rw-r--r-- 1 root root 6963 Dec  9 00:47 server-04.properties
-rw-r--r-- 1 root root 7013 Dec  9 00:48 server-05.properties

Step02: Mainly change 3 values for each Kafka broker:
  broker.id=0
  port=9092
  log.dirs=/var/log/kafka/kafka-01

For example, in server-1.properties used for broker1, we define the following:
broker.id=1
port=9093
log.dirs=/var/log/kafka/kafka-05

Start all brokers:
Broker1
bin/kafka-server-start.sh config/server-01.properties
Broker2
bin/kafka-server-start.sh config/server-02.properties
Broker3
bin/kafka-server-start.sh config/server-03.properties

```

### Multiple Broker Testing

```
Create Topic: replication-factor is 3, because we have 3 brokers now.
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 5 -partitions 1 --topic jay-multiple-brokers-topic

Create Producer ( use all 3 brokers ):
bin/kafka-console-producer.sh --broker-list localhost:9092,localhost:9093,localhost:9094,localhost:9095,localhost:9096 --topic jay-multiple-brokers-topic

Create Consumer:
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic jay-multiple-brokers-topic
```


### Server Tuning

Use RollingFileAppender:
log4j.appender.kafkaAppender=org.apache.log4j.RollingFileAppender
log4j.appender.kafkaAppender.MaxFileSize=500MB
log4j.appender.kafkaAppender.MaxBackupIndex=5


Find all running brokers:
```
jay@kafka-dev-01:/usr/local/kafka/zookeeper-3.4.10$ ./bin/zkCli.sh -server localhost:2181 ls /brokers/ids | tail -1
[1, 2, 3, 4, 5]
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


