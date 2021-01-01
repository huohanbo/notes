# Kafka快速入门
## 引入依赖
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.kafka</groupId>
	<artifactId>spring-kafka</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
	<scope>runtime</scope>
	<optional>true</optional>
</dependency>
<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
	<optional>true</optional>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.springframework.kafka</groupId>
	<artifactId>spring-kafka-test</artifactId>
	<scope>test</scope>
</dependency>
```

## 实现顺序
- 创建消费者和生产者的Map配置
- 根据Map配置创建对应的消费者工厂(consumerFactory)和生产者工厂(producerFactory)
- 根据consumerFactory创建监听器的监听器工厂
- 根据producerFactory创建KafkaTemplate(Kafka操作类)
- 创建监听容器

## 单元测试
- 创建KafkaConfiguration配置类
- 创建DemoListener消费者
- 创建测试类
- 启动项目

# 操作Topic
## 使用@Bean注解创建Topic
```
@Configuration
public class KafkaInitialConfiguration {

    //创建TopicName为topic.quick.initial的Topic并设置分区数为8以及副本数为1
    @Bean
    public NewTopic initialTopic() {
        return new NewTopic("topic.quick.initial",8, (short) 1 );
    }
}
```

## 手动编码创建Topic
同样在KafkaInitialConfiguration类中编码，注册KafkaAdmin和AdminClient两个Bean。
```
@Bean
public KafkaAdmin kafkaAdmin() {
	Map<String, Object> props = new HashMap<>();
	//配置Kafka实例的连接地址
	props.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "127.0.0.1:9092");
	KafkaAdmin admin = new KafkaAdmin(props);
	return admin;
}

@Bean
public AdminClient adminClient() {
	return AdminClient.create(kafkaAdmin().getConfig());
}
```

接下来在DemoTest测试类中编写测试方法，这里需要注意一点Topic的新增删除方法都是异步执行的，为了避免在创建过程中程序关闭导致创建失败，所以在代码最后加了一秒的休眠，执行测试方法我们打开Kafka Tool 2会发现多出了一个"topic.quick.initial2"的Topic
```
@Autowired
private AdminClient adminClient;

@Test
public void testCreateTopic() throws InterruptedException {
	NewTopic topic = new NewTopic("topic.quick.initial2", 1, (short) 1);
	adminClient.createTopics(Arrays.asList(topic));
	Thread.sleep(1000);
}
```

# 发送消息 KafkaTemplate
## 接口参数
```
ListenableFuture<SendResult<K, V>> sendDefault(V data);

ListenableFuture<SendResult<K, V>> sendDefault(K key, V data);

ListenableFuture<SendResult<K, V>> sendDefault(Integer partition, K key, V data);

ListenableFuture<SendResult<K, V>> sendDefault(Integer partition, Long timestamp, K key, V data);

ListenableFuture<SendResult<K, V>> send(String topic, V data);

ListenableFuture<SendResult<K, V>> send(String topic, K key, V data);

ListenableFuture<SendResult<K, V>> send(String topic, Integer partition, K key, V data);

ListenableFuture<SendResult<K, V>> send(String topic, Integer partition, Long timestamp, K key, V data);

ListenableFuture<SendResult<K, V>> send(ProducerRecord<K, V> record);

ListenableFuture<SendResult<K, V>> send(Message<?> message);
```

- topic：这里填写的是Topic的名字
- partition：这里填写的是分区的id，其实也是就第几个分区，id从0开始。表示指定发送到该分区中
- timestamp：时间戳，一般默认当前时间戳
- key：消息的键
- data：消息的数据
- ProducerRecord：消息对应的封装类，包含上述字段
- Message<?>：Spring自带的Message封装类，包含消息及消息头

## 使用sendDefault发送消息
首先在KafkaConfiguration编写一个带有默认Topic参数的KafkaTemplate[**defaultKafkaTemplate**]，同时为另外一个KafkaTemplate加上@Primary注解。
@Primary注解的意思是在拥有多个同类型的Bean时优先使用该Bean，到时候方便我们使用@Autowired注解自动注入。
```
//这个是我们之前编写的KafkaTemplate代码，加入@Primary注解
@Bean
@Primary
public KafkaTemplate<Integer, String> kafkaTemplate() {
	KafkaTemplate template = new KafkaTemplate<Integer, String>(producerFactory());
	return template;
}

@Bean("defaultKafkaTemplate")
public KafkaTemplate<Integer, String> defaultKafkaTemplate() {
	KafkaTemplate template = new KafkaTemplate<Integer, String>(producerFactory());
	template.setDefaultTopic("topic.quick.default");
	return template;
}
```

接着编写测试方法，可以看到我们这里调用的是sendDefault方法，而且并没有在方法参数上添加topicName，
这是因为我们在声明defaultKafkaTemplate这个Bean的时候添加了这行代码  template.setDefaultTopic("topic.quick.default")，
只要调用sendDefault方法，kafkaTemplate会自动把消息发送到名为"topic.quick.default"的Topic中。

## KafkaTemplate同步发送消息
KafkaTemplate异步发送消息大大的提升了生产者的并发能力，但某些场景下我们并不需要异步发送消息，
这个时候我们可以采取同步发送方式，实现也是非常简单的，我们只需要在send方法后面调用get方法即可。
Future模式中，我们采取异步执行事件，等到需要返回值得时候我们再调用get方法获取future的返回值。
```
kafkaTemplate.send("topic.quick.demo", "test sync send message").get();
```

# 消息监听 非注解方式
Spring-Kafka中消息监听大致分为两种类型，一种是单条数据消费，一种是批量消费；两者的区别只是在于监听器一次性获取消息的数量。

GenericMessageListener是我们实现消息监听的一个接口，向上扩展的接口有非常多，
比如：单数据消费的MessageListener、批量消费的BatchMessageListener、
还有具备ACK机制的AcknowledgingMessageListener和BatchAcknowledgingMessageListener等等。

我们在创建监听容器前需要创建一个监听容器工厂，这里只需要配置一下消费者工厂就好了，
之后我们使用它去创建我们的监听容器。consumerFactory()这个参数在之前就已经定义过了，这里就不重复贴代码了。
```
@Bean
public ConcurrentKafkaListenerContainerFactory<Integer, String> kafkaListenerContainerFactory() {
	ConcurrentKafkaListenerContainerFactory<Integer, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
	factory.setConsumerFactory(consumerFactory());
	return factory;
}
```

有了监听容器工厂，我们就可以使用它去创建我们的监听容器
Bean方式创建监听容器
```
@Bean
public KafkaMessageListenerContainer demoListenerContainer() {
	ContainerProperties properties = new ContainerProperties("topic.quick.bean");
	
	properties.setGroupId("bean");
	
	properties.setMessageListener(new MessageListener<Integer,String>() {
		private Logger log = LoggerFactory.getLogger(this.getClass());
		@Override
		public void onMessage(ConsumerRecord<Integer, String> record) {
			log.info("topic.quick.bean receive : " + record.toString());
		}
	});

	return new KafkaMessageListenerContainer(consumerFactory(), properties);
}
```

# 消息监听 KafkaListener
## KafkaListener属性
- id：消费者的id，当GroupId没有被配置的时候，默认id为GroupId
- containerFactory：上面提到了@KafkaListener区分单数据还是多数据消费只需要配置一下注解的containerFactory属性就可以了，这里面配置的是监听容器工厂，也就是ConcurrentKafkaListenerContainerFactory，配置BeanName
- topics：需要监听的Topic，可监听多个
- topicPartitions：可配置更加详细的监听信息，必须监听某个Topic中的指定分区，或者从offset为200的偏移量开始监听
- errorHandler：监听异常处理器，配置BeanName
- groupId：消费组ID
- idIsGroup：id是否为GroupId
- clientIdPrefix：消费者Id前缀
- beanRef：真实监听容器的BeanName，需要在 BeanName前加 "__"

## 监听方法可以写的参数
- data： 对于data值的类型其实并没有限定，根据KafkaTemplate所定义的类型来决定。data为List集合的则是用作批量消费。
- ConsumerRecord：具体消费数据类，包含Headers信息、分区信息、时间戳等
- Acknowledgment：用作Ack机制的接口
- Consumer：消费者类，使用该类我们可以手动提交偏移量、控制消费速率等功能

## 批量消费案例
- 重新创建一份新的消费者配置，配置为一次拉取5条消息
- 创建一个监听容器工厂，设置其为批量消费并设置并发量为5，这个并发量根据分区数决定，必须小于等于分区数，否则会有线程一直处于空闲状态
- 创建一个分区数为8的Topic
- 创建监听方法，设置消费id为batch，clientID前缀为batch，监听topic.quick.batch，使用batchContainerFactory工厂创建该监听容器

```
@Component
public class BatchListener {

    private static final Logger log= LoggerFactory.getLogger(BatchListener.class);

    private Map<String, Object> consumerProps() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, true);
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
        props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "15000");
        //一次拉取消息数量
        props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, "5");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, IntegerDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        return props;
    }

    @Bean("batchContainerFactory")
    public ConcurrentKafkaListenerContainerFactory listenerContainer() {
        ConcurrentKafkaListenerContainerFactory container = new ConcurrentKafkaListenerContainerFactory();
        container.setConsumerFactory(new DefaultKafkaConsumerFactory(consumerProps()));
        //设置并发量，小于或等于Topic的分区数
        container.setConcurrency(5);
        //设置为批量监听
        container.setBatchListener(true);
        return container;
    }

    @Bean
    public NewTopic batchTopic() {
        return new NewTopic("topic.quick.batch", 8, (short) 1);
    }

    @KafkaListener(id = "batch",clientIdPrefix = "batch",topics = {"topic.quick.batch"},containerFactory = "batchContainerFactory")
    public void batchListener(List<String> data) {
        log.info("topic.quick.batch  receive : ");
        for (String s : data) {
            log.info(  s);
        }
    }
}
```

**设置的并发量不能大于partition的数量，如果需要提高吞吐量，可以通过增加partition的数量达到快速提升吞吐量的效果。**

## 监听Topic中指定的分区
使用@KafkaListener注解的topicPartitions属性监听不同的partition分区。
- @TopicPartition：topic--需要监听的Topic的名称，partitions --需要监听Topic的分区id，
- partitionOffsets --可以设置从某个偏移量开始监听
- @PartitionOffset：partition --分区Id，非数组，initialOffset --初始偏移量

```
@Bean
public NewTopic batchWithPartitionTopic() {
	return new NewTopic("topic.quick.batch.partition", 8, (short) 1);
}

@KafkaListener(id = "batchWithPartition",clientIdPrefix = "bwp",containerFactory = "batchContainerFactory",
	topicPartitions = {
		@TopicPartition(topic = "topic.quick.batch.partition",partitions = {"1","3"}),
		@TopicPartition(topic = "topic.quick.batch.partition",partitions = {"0","4"},
				partitionOffsets = @PartitionOffset(partition = "2",initialOffset = "100"))
	}
)
public void batchListenerWithPartition(List<String> data) {
	log.info("topic.quick.batch.partition  receive : ");
	for (String s : data) {
		log.info(s);
	}
}
```

## 使用Ack机制确认消费
我先说说RabbitMQ的Ack机制，RabbitMQ的消费可以说是一次性的，也就是你确认消费后就立刻从硬盘或内存中删除，
而且RabbitMQ粗糙点来说是顺序消费，像排队一样，一个个顺序消费，未被确认的消息则会重新回到队列中，等待监听器再次消费。

但Kafka不同，Kafka是通过最新保存偏移量进行消息消费的，而且确认消费的消息并不会立刻删除，所以我们可以重复的消费未被删除的数据，
**当第一条消息未被确认，而第二条消息被确认的时候，Kafka会保存第二条消息的偏移量，也就是说第一条消息再也不会被监听器所获取**，
除非是根据第一条消息的偏移量手动获取。

使用Kafka的Ack机制比较简单，只需简单的三步即可：
- 设置ENABLE_AUTO_COMMIT_CONFIG=false，禁止自动提交
- 设置AckMode=MANUAL_IMMEDIATE
- 监听方法加入Acknowledgment ack 参数

怎么拒绝消息呢，只要在监听方法中不调用ack.acknowledge()即可。
```
@Component
public class AckListener {

    private static final Logger log= LoggerFactory.getLogger(AckListener.class);

    private Map<String, Object> consumerProps() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
		// 设置ENABLE_AUTO_COMMIT_CONFIG=false，禁止自动提交
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
        props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "15000");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, IntegerDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        return props;
    }

    @Bean("ackContainerFactory")
    public ConcurrentKafkaListenerContainerFactory ackContainerFactory() {
        ConcurrentKafkaListenerContainerFactory factory = new ConcurrentKafkaListenerContainerFactory();
        factory.setConsumerFactory(new DefaultKafkaConsumerFactory(consumerProps()));
		// 设置AckMode=MANUAL_IMMEDIATE
        factory.getContainerProperties().setAckMode(AbstractMessageListenerContainer.AckMode.MANUAL_IMMEDIATE);
        factory.setConsumerFactory(new DefaultKafkaConsumerFactory(consumerProps()));
        return factory;
    }


    @KafkaListener(id = "ack", topics = "topic.quick.ack", containerFactory = "ackContainerFactory")
    public void ackListener(ConsumerRecord record, Acknowledgment ack) {
        log.info("topic.quick.ack receive : " + record.value());
        ack.acknowledge();
    }
}
```

在这段章节开头之初我就讲解了Kafka机制会出现的一些情况，导致没办法重复消费未被Ack的消息，解决办法有如下：

1.重新将消息发送到队列中，这种方式比较简单而且可以使用Headers实现第几次消费的功能，用以下次判断
```
@KafkaListener(id = "ack", topics = "topic.quick.ack", containerFactory = "ackContainerFactory")
public void ackListener(ConsumerRecord record, Acknowledgment ack, Consumer consumer) {
	log.info("topic.quick.ack receive : " + record.value());

	//如果偏移量为偶数则确认消费，否则拒绝消费
	if (record.offset() % 2 == 0) {
		log.info(record.offset()+"--ack");
		ack.acknowledge();
	} else {
		log.info(record.offset()+"--nack");
		kafkaTemplate.send("topic.quick.ack", record.value());
	}
}
```

2.使用Consumer.seek方法，重新回到该未ack消息偏移量的位置重新消费，这种可能会导致死循环，原因出现于业务一直没办法处理这条数据，但还是不停的重新定位到该数据的偏移量上。
```
@KafkaListener(id = "ack", topics = "topic.quick.ack", containerFactory = "ackContainerFactory")
public void ackListener(ConsumerRecord record, Acknowledgment ack, Consumer consumer) {
	log.info("topic.quick.ack receive : " + record.value());

	//如果偏移量为偶数则确认消费，否则拒绝消费
	if (record.offset() % 2 == 0) {
		log.info(record.offset()+"--ack");
		ack.acknowledge();
	} else {
		log.info(record.offset()+"--nack");
		consumer.seek(new TopicPartition("topic.quick.ack",record.partition()),record.offset() );
	}
}
```
