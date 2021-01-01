# 前言
## 参考资料
- [Kafka官网](https://kafka.apache.org/23/javadoc/index.html?org/apache/kafka/clients/consumer/KafkaConsumer.html)

# KafkaConsumer
使用Kafka群集记录的客户端。

此客户端透明地处理Kafka代理的故障，并透明地适应它在集群内迁移的主题分区。 此客户端还与代理交互，以允许消费者组使用消费者组来平衡消费。

使用者维护与必要代理的TCP连接以获取数据。 使用后未能关闭消费者将泄漏这些连接。 消费者不是线程安全的。 有关详细信息，请参阅多线程处理。

##  跨版本兼容性
##  抵消和消费者位置
##  消费者群体和主题订阅
##  检测消费者故障

# 构造方法
KafkaConsumer(Map<String,Object> configs)
通过提供一组键值对作为配置来实例化消费者。

KafkaConsumer(Map<String,Object> configs, Deserializer<K> keyDeserializer, Deserializer<V> valueDeserializer)
通过提供一组键值对作为配置，以及键和值Deserializer来实例化消费者。

KafkaConsumer(Properties properties)
通过提供Properties对象作为配置来实例化消费者。

KafkaConsumer(Properties properties, Deserializer<K> keyDeserializer, Deserializer<V> valueDeserializer)
通过提供Properties对象作为配置，以及键和值Deserializer来实例化消费者。

# 相关参数
## TopicPartition 主题名称和分区
TopicPartition(String topic, int partition) 

## PartitionInfo 分区信息
这用于描述MetadataResponse中的每分区状态

## Duration 基于时间的时间量

## OffsetAndMetadata 偏移量和元数据
Kafka偏移提交API允许用户在提交偏移时提供额外的元数据（以字符串的形式）。 
这可能很有用（例如）存储有关哪个节点进行提交的信息，提交的时间等等。

## MetricName 度量名称
MetricName类封装了度量标准的名称，逻辑组及其相关属性。它应该使用metrics.MetricName（...）构建。

##
# 方法摘要
## assign 手动分配分区列表
手动分配分区列表
```
void assign(Collection<TopicPartition> partitions)
```

## assignment 获取当前分配给此消费者的分区集
```
Set<TopicPartition> assignment()
```	

## beginningOffsets 获取给定分区的第一个偏移量
```
Map<TopicPartition,Long> beginningOffsets (Collection<TopicPartition> partitions)

Map<TopicPartition,Long> beginningOffsets(Collection<TopicPartition> partitions, Duration timeout)
```

## close 关闭消费者
```
void close()
关闭消费者，默认超时时间30s

void close(Duration timeout)
关闭消费者，在指定的超时时间内
```

## commitAsync 异步提交偏移
```
void commitAsync()
在所有订阅的主题和分区列表的最后一次轮询（持续时间）上返回的提交偏移量

void commitAsync(Map<TopicPartition,OffsetAndMetadata> offsets, OffsetCommitCallback callback)
将指定的主题和分区列表的指定偏移量提交给Kafka

void commitAsync(OffsetCommitCallback callback)
在订阅的主题和分区列表的最后一个poll（）上返回的提交偏移量
```

## commitSync 提交偏移
将指定的主题和分区列表的指定偏移量提交给Kafka
```
void commitSync()
Commit offsets returned on the last poll() for all the subscribed list of topics and partitions

void commitSync(Duration timeout)
Commit offsets returned on the last poll() for all the subscribed list of topics and partitions

void commitSync(Map<TopicPartition,OffsetAndMetadata> offsets)
Commit the specified offsets for the specified list of topics and partitions

void commitSync(Map<TopicPartition,OffsetAndMetadata> offsets, Duration timeout)
Commit the specified offsets for the specified list of topics and partitions
```

## committed 获取给定分区的最后一个提交的偏移量
```
OffsetAndMetadata committed(TopicPartition partition)
获取给定分区的最后一个提交的偏移量（无论此进程是否发生了提交）。

OffsetAndMetadata committed(TopicPartition partition, Duration timeout)
获取给定分区的最后一个提交的偏移量（无论此进程是否发生了提交）。
```

## endOffsets 获取给定分区的最后一个偏移量
```
Map<TopicPartition,Long> endOffsets(Collection<TopicPartition> partitions)
获取给定分区的最后一个偏移量

Map<TopicPartition,Long> endOffsets(Collection<TopicPartition> partitions, Duration timeout)
获取给定分区的最后一个偏移量
```

## metrics 获取消费者保留的指标
```
Map<MetricName,? extends Metric> metrics()
获取消费者保留的指标
```

## offsetsForTimes 按时间戳查找给定分区的偏移量
```
Map<TopicPartition,OffsetAndTimestamp> offsetsForTimes(Map<TopicPartition,Long> timestampsToSearch)
按时间戳查找给定分区的偏移量
Map<TopicPartition,OffsetAndTimestamp> offsetsForTimes(Map<TopicPartition,Long> timestampsToSearch, Duration timeout)
按时间戳查找给定分区的偏移量
```

## partitionsFor 获取给定主题的分区信息
```
List<PartitionInfo>	partitionsFor(String topic)
获取给定主题的分区信息
List<PartitionInfo>	partitionsFor(String topic, Duration timeout)
获取给定主题的分区信息
```

## pause 暂停从请求的分区中获取数据
```
void pause(Collection<TopicPartition> partitions)
暂停从请求的分区中获取数据
```

## paused 获取先前暂停的分区集
```
Set<TopicPartition> paused()
获取先前暂停的分区集
```

## poll 获取数据
```
ConsumerRecords<K,V> poll(Duration timeout)
获取使用 (subscribe / assign API) 指定的主题或分区的数据
ConsumerRecords<K,V> poll(long timeoutMs)
已过时
```

## position 获取将要获取的下一条记录的偏移量
```
long position(TopicPartition partition)
获取将要获取的下一条记录的偏移量（如果存在具有该偏移量的记录）
long position(TopicPartition partition, Duration timeout)
获取将要获取的下一条记录的偏移量（如果存在具有该偏移量的记录）
```

## resume 恢复已暂停的指定分区
```
void resume(Collection<TopicPartition> partitions)
恢复已暂停的指定分区
```

## seek
void seek(TopicPartition partition, long offset)
Overrides the fetch offsets that the consumer will use on the next poll(timeout).
void seek(TopicPartition partition, OffsetAndMetadata offsetAndMetadata)
Overrides the fetch offsets that the consumer will use on the next poll(timeout).
void seekToBeginning(Collection<TopicPartition> partitions)
Seek to the first offset for each of the given partitions.
void seekToEnd(Collection<TopicPartition> partitions)
Seek to the last offset for each of the given partitions.

void	subscribe(Collection<String> topics)
Subscribe to the given list of topics to get dynamically assigned partitions.
void	subscribe(Collection<String> topics, ConsumerRebalanceListener listener)
Subscribe to the given list of topics to get dynamically assigned partitions.
void	subscribe(Pattern pattern)
Subscribe to all topics matching specified pattern to get dynamically assigned partitions.
void	subscribe(Pattern pattern, ConsumerRebalanceListener listener)
Subscribe to all topics matching specified pattern to get dynamically assigned partitions.
Set<String>	subscription()
Get the current subscription.
void	unsubscribe()
Unsubscribe from topics currently subscribed with subscribe(Collection) or subscribe(Pattern).
void	wakeup()
Wakeup the consumer.



# 用法示例
## 自动控制提交
```
Properties props = new Properties();
props.setProperty("bootstrap.servers", "localhost:9092");
props.setProperty("group.id", "test");
props.setProperty("enable.auto.commit", "true");
props.setProperty("auto.commit.interval.ms", "1000");
props.setProperty("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.setProperty("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("foo", "bar"));
while (true) {
	ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
	for (ConsumerRecord<String, String> record : records)
		System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
}
```

设置**enable.auto.commit**意味着自动提交偏移量，其频率由**auto.commit.interval.ms**控制。

## 手动控制提交
用户还可以控制何时应将记录视为已消耗并因此提交其偏移量，而不是依赖于消费者定期提交消耗的偏移量。 
当消息的消耗与某些处理逻辑耦合时，这是有用的，因此消息在完成处理之前不应被视为已消耗。
```
Properties props = new Properties();
props.setProperty("bootstrap.servers", "localhost:9092");
props.setProperty("group.id", "test");
props.setProperty("enable.auto.commit", "false");
props.setProperty("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.setProperty("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("foo", "bar"));
final int minBatchSize = 200;
List<ConsumerRecord<String, String>> buffer = new ArrayList<>();
while (true) {
	ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
	for (ConsumerRecord<String, String> record : records) {
		buffer.add(record);
	}
	if (buffer.size() >= minBatchSize) {
		insertIntoDb(buffer);
		consumer.commitSync();
		buffer.clear();
	}
}
```
注意：使用自动偏移提交也可以为您提供“至少一次”交付，但要求是您必须在任何后续调用之前或关闭使用者之前使用从每次调用poll（Duration）返回的所有数据。
如果您未能执行上述任何一项操作，则承诺的偏移量可能会超过消耗的位置，从而导致丢失记录。
使用手动偏移控制的优点是您可以直接控制记录被视为“消耗”的时间。

上面的示例使用commitSync将所有已接收的记录标记为已提交。 
在某些情况下，您可能希望通过明确指定偏移量来更好地控制已提交的记录。 
在下面的示例中，我们在完成处理每个分区中的记录后提交偏移量。
```
try {
	while(running) {
		ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(Long.MAX_VALUE));
		for (TopicPartition partition : records.partitions()) {
			List<ConsumerRecord<String, String>> partitionRecords = records.records(partition);
			for (ConsumerRecord<String, String> record : partitionRecords) {
				System.out.println(record.offset() + ": " + record.value());
			}
			long lastOffset = partitionRecords.get(partitionRecords.size() - 1).offset();
			consumer.commitSync(Collections.singletonMap(partition, new OffsetAndMetadata(lastOffset + 1)));
		}
	}
} finally {
	consumer.close();
}
```

注意：提交的偏移量应始终是应用程序将读取的下一条消息的偏移量。 
因此，在调用commitSync（偏移量）时，您应该在最后处理的消息的偏移量中添加一个。

## 手动分配分区
在前面的示例中，我们订阅了我们感兴趣的主题，并让Kafka根据组中的活跃消费者为这些主题动态分配公平的分区。
但是，在某些情况下，您可能需要更好地控制分配的特定分区。例如：
- 如果进程正在维护与该分区关联的某种本地状态（如本地磁盘上的键值存储），那么它应该只获取它在磁盘上维护的分区的记录。
- 如果进程本身具有高可用性，并且如果失败则将重新启动（可能使用集群管理框架，如YARN，Mesos或AWS工具，或作为流处理框架的一部分）。在这种情况下，Kafka不需要检测故障并重新分配分区，因为消耗过程将在另一台机器上重新启动。

要使用此模式，您只需使用要使用的分区的完整列表调用assign（Collection），而不是使用subscribe订阅主题。
```
consumer.subscribe(Arrays.asList("foo"));
->
String topic = "foo";
TopicPartition partition0 = new TopicPartition(topic, 0);
TopicPartition partition1 = new TopicPartition(topic, 1);
consumer.assign(Arrays.asList(partition0, partition1));
```

## 使用外部存储偏移
消费者应用程序不需要使用Kafka的内置偏移存储，它可以在自己选择的商店中存储偏移量。
这样做的主要用例是允许应用程序在同一系统中存储偏移量和消耗结果，其结果和偏移量都是以原子方式存储的。
这并不总是可行，但是当它存在时，它将使消耗完全原子化并且给出“完全一次”语义，这比使用Kafka的偏移提交功能的默认“至少一次”语义更强。

每条记录都有自己的偏移量，因此要管理自己的偏移量，您只需执行以下操作：
- 配置enable.auto.commit = false
- 使用每个ConsumerRecord提供的偏移量来保存您的位置。
- 在重新启动时使用seek（TopicPartition，long）恢复使用者的位置。

当手动完成分区分配时，这种类型的使用最简单（这可能在上述搜索索引用例中）。
如果分区分配是自动完成的，则需要特别注意处理分区分配更改的情况。
这可以通过在对subscribe（Collection，ConsumerRebalanceListener）和subscribe（Pattern，ConsumerRebalanceListener）的调用中提供ConsumerRebalanceListener实例来完成。
例如，当从消费者处获取分区时，消费者将希望通过实现ConsumerRebalanceListener.onPartitionsRevoked（Collection）来为这些分区提交其偏移量。
将分区分配给使用者时，使用者将希望查找这些新分区的偏移量，并通过实现ConsumerRebalanceListener.onPartitionsAssigned（Collection）将消费者正确初始化为该位置。

## 控制消费偏移位置
在大多数用例中，消费者将简单地从头到尾消费记录，定期提交其位置（自动或手动）。
然而，Kafka允许消费者手动控制其位置，随意在分区中向前或向后移动。
这意味着消费者可以重新使用旧记录，或跳过最新记录而不实际使用中间记录。

Kafka允许使用seek（TopicPartition，long）指定位置以指定新位置。
寻求服务器维护的最早和最新偏移的特殊方法也可用（seekToBeginning（Collection）和seekToEnd（Collection））。

## 控制消费流量
如果为消费者分配了多个分区来从中获取数据，它将尝试同时使用所有这些分区，从而有效地为这些分区提供相同的优先级以供消费。
但是，在某些情况下，消费者可能希望首先关注从全速分配的分区的某个子集中获取，并且仅在这些分区很少或没有数据要消耗时才开始获取其他分区。

Kafka支持动态控制消耗流，方法是
使用pause（Collection）和resume（Collection）暂停指定分配的分区上的消耗，
并在将来的poll（Duration）调用中分别恢复指定的暂停分区上的消耗。

## 读取事物消息

## 多线程处理
Kafka消费者不是线程安全的。 所有网络I / O都发生在进行调用的应用程序的线程中。 
用户有责任确保正确同步多线程访问。 未同步的访问将导致ConcurrentModificationException。

此规则的唯一例外是wakeup（），它可以安全地从外部线程用于中断活动操作。 
在这种情况下，将从操作的线程阻塞中抛出WakeupException。 这可以用于从另一个线程关闭使用者。 
以下代码段显示了典型模式：
```
public class KafkaConsumerRunner implements Runnable {
     private final AtomicBoolean closed = new AtomicBoolean(false);
     private final KafkaConsumer consumer;

     public KafkaConsumerRunner(KafkaConsumer consumer) {
       this.consumer = consumer;
     }

     public void run() {
         try {
             consumer.subscribe(Arrays.asList("topic"));
             while (!closed.get()) {
                 ConsumerRecords records = consumer.poll(Duration.ofMillis(10000));
                 // Handle new records
             }
         } catch (WakeupException e) {
             // Ignore exception if closing
             if (!closed.get()) throw e;
         } finally {
             consumer.close();
         }
     }

     // Shutdown hook which can be called from a separate thread
     public void shutdown() {
         closed.set(true);
         consumer.wakeup();
     }
}
```

然后在单独的线程中，可以通过设置关闭标志并唤醒消费者来关闭消费者。
```
closed.set(true);
consumer.wakeup();
```

我们故意避免实现特定的线程模型进行处理。这留下了几个用于实现记录的多线程处理的选项。

1.每个线程一个消费者
一个简单的选择是为每个线程提供自己的消费者实例。以下是此方法的优缺点：
- PRO：这是最容易实现的
- PRO：它通常是最快的，因为不需要线程间的协调
- PRO：它使每个分区的有序处理非常容易实现（每个线程只按接收顺序处理消息）。
- CON：更多的消费者意味着与群集的TCP连接更多（每个线程一个）。一般来说，Kafka非常有效地处理连接，因此这通常是一个很小的成本。
- CON：多个使用者意味着更多的请求被发送到服务器，而数据的批量略少，这可能导致I / O吞吐量的一些下降。
- CON：所有进程中的总线程数将受到分区总数的限制。

2.解耦消息消费和消息处理
另一种方法是让一个或多个消费者线程执行所有数据消耗，并将ConsumerRecords实例移交给实际处理记录处理的处理器线程池所消耗的阻塞队列。这个选项同样有利有弊：
- PRO：此选项允许独立扩展消费者和处理器的数量。这使得可以让单个消费者提供许多处理器线程，从而避免对分区的任何限制。
- CON：保证处理器之间的顺序需要特别小心，因为线程将独立执行，因为线程执行时间的好运，实际上可以在稍后的数据块之后处理较早的数据块。对于没有订购要求的处理，这不是问题。
- CON：手动提交位置变得更加困难，因为它要求所有线程协调以确保该分区的处理完成。

这种方法有许多可能的变化。
例如，每个处理器线程可以拥有自己的队列，并且使用者线程可以使用TopicPartition散列到这些队列中，以确保按顺序使用并简化提交。