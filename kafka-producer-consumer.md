## SPRINGBOOT & KAFKA INTEGRATION

[Download Kafka] (https://kafka.apache.org/quickstart).

#### Run The Commands Sequentially
##### RUN THE ZOOKEEPER
```
(Windows) bin/windows/zookeeper-server-start.bat config/zookeeper.properties
(Unix) bin/zookeeper-server-start.sh config/zookeeper.properties
```

##### RUN THE KAFKA SERVER
```
(Windows) bin/windows/kafka-server-start.bat config/server.properties
(Unix) bin/kafka-server-start.sh config/server.properties
```

##### CREATE KAFKA TOPIC
```
(Windows) bin/windows/kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic  Kafka_Example
(Unix) bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic  Kafka_Example
```

##### CONSUME TOPIC
```
(Windows) bin/windows/kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic Kafka_Example --from-beginning
(Unix) bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic Kafka_Example --from-beginning
```

#### POM File
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.4.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example.kafka</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>springboot-demo-1</name>
	<description>Junit-Stub</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
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
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.springframework.kafka</groupId>
			<artifactId>spring-kafka-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```

#### application.properties
```
#Kafka Topic
message.topic.name=Kafka_Example
 
spring.kafka.bootstrap-servers=localhost:9092
 
#Unique String which identifies which consumer group this consumer belongs to
spring.kafka.consumer.group-id=jcg-group

```

#### KafkaApplication.java
```java
package com.example.kafka;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.KafkaHeaders;
import org.springframework.messaging.Message;
import org.springframework.messaging.support.MessageBuilder;

import com.example.kafka.domain.User;

@SpringBootApplication
public class KafkaApplication implements CommandLineRunner {
 
    private static final Logger LOG = LoggerFactory.getLogger("KafkaApp");
 
    @Value("${message.topic.name}")
    private String topicName;
 
    private final KafkaTemplate<String, User> kafkaTemplate;
 
    @Autowired
    public KafkaApplication(KafkaTemplate<String, User> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }
 
    public static void main(String[] args) {
        SpringApplication.run(KafkaApplication.class, args);
    }
 
    @Override
    public void run(String... strings) {
        Message<User> message = MessageBuilder.withPayload(new User("Ege-Deniz","Merhabaaaa"))
								              .setHeader(KafkaHeaders.TOPIC, topicName)
								              .build();
    	
        kafkaTemplate.send(message);
        LOG.info("Published message to topic: {}.", topicName);
    }
 
    @KafkaListener(topics = "${message.topic.name}", groupId = "jcg-group")
    public void listen(User message) {
        LOG.info("Received message in JCG group: {}", message);
    }
 
}
```
#### KafkaProducerConfig.java

```java
package com.example.kafka.config;

import java.util.HashMap;
import java.util.Map;

import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;
import org.springframework.kafka.support.serializer.JsonSerializer;

import com.example.kafka.domain.User;

@Configuration
public class KafkaProducerConfig {

	@Value("${spring.kafka.bootstrap-servers}")
	private String bootstrapServers;
	
	@Bean
	public ProducerFactory<String, User> producerFactory(){
		Map<String,Object> configProps = new HashMap<>();
		configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
		configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
		configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
		return new DefaultKafkaProducerFactory<>(configProps);
	}
	
	@Bean
	public KafkaTemplate<String, User> kafkaTemplate(){
		return new KafkaTemplate<>(producerFactory());
	}
	
}

```


#### KafkaConsumerConfig.java

```java
package com.example.kafka.config;

import java.util.HashMap;
import java.util.Map;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.support.serializer.JsonDeserializer;

import com.example.kafka.domain.User;

@EnableKafka
@Configuration
public class KafkaConsumerConfig {

	@Value("${spring.kafka.bootstrap-servers}")
    private String bootstrapAddress;
 
    @Value("${spring.kafka.consumer.group-id}")
    private String groupId;
    
    @Bean
    public ConsumerFactory<String, User> consumerFactory(){
    	Map<String,Object> props = new HashMap<>();
    	props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapAddress);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
    	props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
    	props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
    	props.put(JsonDeserializer.TRUSTED_PACKAGES, "*");
    	return new DefaultKafkaConsumerFactory<>(props);
    }
    
    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, User> kafkaListenerContainerFactory(){
    	ConcurrentKafkaListenerContainerFactory<String, User> factory = new ConcurrentKafkaListenerContainerFactory<>();
    	factory.setConsumerFactory(consumerFactory());
    	return factory;
    }
    	
	
}
```

#### User.java

```java
package com.example.kafka.domain;

import java.io.Serializable;

public class User implements Serializable{

	private static final long serialVersionUID = 1L;
	private String name;
	private String message;
	
	public User() {
	}

	public User(String name, String message) {
		this.name = name;
		this.message = message;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getMessage() {
		return message;
	}

	public void setMessage(String message) {
		this.message = message;
	}

	@Override
	public String toString() {
		return "User [name=" + name + ", message=" + message + "]";
	}
	
}

```

#### UserController.java

```java
package com.example.kafka.resource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.example.kafka.domain.User;

@RestController
@RequestMapping("kafka")
public class UserController {
	
	@Value("${message.topic.name}")
	private String topic;
	
	@Autowired
	private KafkaTemplate<String, User> kafkaTemplate;
	
	@GetMapping("/publish/{message}")
	public String publish(@PathVariable String message) {
		kafkaTemplate.send(topic, new User("Yahya",message));
		return "Published successfully!";
	}
	
}
```