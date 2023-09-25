##### 添加依赖

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.8.0</version>
</dependency>
```



#### 创建生产者

```java
import org.apache.kafka.clients.producer.*;

import java.util.Properties;

public class KafkaProducerExample {
	private final static String STRING_SERIALIZER = "org.apache.kafka.common.serialization.StringSerializer";
    
    public static void main(String[] args) {
        String bootstrapServers = "localhost:9092";
        String topic = "my-topic";

        // Kafka 生产者配置
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, STRING_SERIALIZER);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, STRING_SERIALIZER);

        // 创建 Kafka 生产者
        KafkaProducer<String, String> producer = new KafkaProducer<>(props);

        // 发送消息
        for (int i = 0; i < 10; i++) {
            String message = "Message " + i;
            ProducerRecord<String, String> record = new ProducerRecord<>(topic, message);

            producer.send(record, new Callback() {
                @Override
                public void onCompletion(RecordMetadata metadata, Exception exception) {
                    if (exception != null) {
                        System.err.println("Error sending message: " + exception.getMessage());
                    } else {
                        System.out.println("Message sent successfully! Topic: " + metadata.topic()
                                + ", Partition: " + metadata.partition()
                                + ", Offset: " + metadata.offset());
                    }
                }
            });
        }

        // 关闭 Kafka 生产者
        producer.close();
    }
}
```

##### 大致过程

1. 创建 `ProducerRecord`：生产者在发送消息之前需要创建一个 `ProducerRecord` 对象。该对象包含消息的主题、键、值等信息。键和值通常会被序列化为字节数组，以便在网络上传输。
2. 分区分配器（Partitioner）：如果在创建 `ProducerRecord` 对象时未指定分区（partition），则分区分配器将根据键值来选择一个分区。分区分配器根据特定的策略，例如哈希、循环或随机算法，将消息分配给一个分区。这确保具有相同键的消息将被发送到同一个分区，从而保持消息的顺序性。
3. 批量发送：生产者在发送消息时通常会进行批量发送，将多条消息聚合成一个批次。这样可以减少网络开销和提高吞吐量。生产者会将**相同主题和相同分区**的多条消息放入同一个批次中，另一个线程负责将这些批次消息发送到对应的 Kafka broker。
4. 发送结果和重试：当 Kafka broker 接收到消息后，如果成功写入到分区中，则会返回一个包含消息的主题、分区和位移等信息的 `RecordMetadata` 对象。如果写入失败，会抛出异常。生产者在接收到结果后，可以根据需要进行重试操作，以确保消息被成功发送。

##### 生产者指定分区

```java
// 指定键
ProducerRecord<String, String> record = new ProducerRecord<>("topicName", "key", "value");
// 指定分区
int partition = 2; 
ProducerRecord<String, String> record = new ProducerRecord<>("topicName", partition, "key", "value");
```

> 注意，手动指定分区可能会导致负载不均衡，因为某些分区可能会接收到更多的消息。默认情况下，Kafka 使用一个分区分配器（Partitioner）来决定消息的分区，你可以自定义分区分配器来根据具体需求进行分区策略的定制



##### 指定偏移量

自动：

- Kafka 提供了自动管理偏移量的功能。在创建 KafkaConsumer 对象时，可以设置 `enable.auto.commit` 属性为 `true`，并指定一个自动提交偏移量的频率。KafkaConsumer 会自动定期提交偏移量。Kafka 会跟踪每个消费者组的偏移量，并在后台自动提交。

- 这种方式简单且方便，但可能会存在一些消息丢失的风险，因为偏移量的提交是异步的。

```java
props.put("enable.auto.commit", "true");
props.put("auto.commit.interval.ms", "5000");
```

手动：

- 在消费消息之前，可以使用 `consumer.subscribe()` 方法来订阅一个或多个主题。
- 在消费消息时，可以使用 `consumer.poll()` 方法来拉取消息记录。在处理消息之后，可以通过记录消息的偏移量来跟踪当前消费的进度。
- 使用 `consumer.commitSync()` 方法来手动提交偏移量。



#### 创建消费者

```java
import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.TopicPartition;

import java.util.Collections;
import java.util.Properties;

public class KafkaConsumerExample {
    private final static String STRING_SERIALIZER = "org.apache.kafka.common.serialization.StringSerializer";

    public static void main(String[] args) {
        String bootstrapServers = "localhost:9092";
        String topic = "my-topic";
        String groupId = "my-group";

        // Kafka 消费者配置
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, STRING_SERIALIZER);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, STRING_SERIALIZER);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);

        // 创建 Kafka 消费者
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

        // 订阅主题
        consumer.subscribe(Collections.singletonList(topic));

        // 拉取消息
        while (true) {
            // 阻塞;等待指定的时间(100ms)，尝试从 Kafka 集群获取消息记录（ConsumerRecords）
            ConsumerRecords<String, String> records = consumer.poll(100); 

            for (ConsumerRecord<String, String> record : records) {
                System.out.println("Received message: Topic = " + record.topic()
                        + ", Partition = " + record.partition()
                        + ", Offset = " + record.offset()
                        + ", Key = " + record.key()
                        + ", Value = " + record.value());
            }

            // 手动提交偏移量
            consumer.commitSync();
        }
    }
}
```







