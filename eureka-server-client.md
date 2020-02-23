## EUREKA SERVER
#### Dependencies
```html
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
	</dependency>
```

#### application.properties (it is set not to register itself)
```html
server.port=8761
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
```

#### Add @EnableEurekaServer in the main class
```java
@SpringBootApplication
@EnableEurekaServer
public class DiscoveryServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(DiscoveryServerApplication.class, args);
	}

}
```

## EUREKA CLIENT (edge-service)
#### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.2.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.eureka</groupId>
	<artifactId>edgeservice</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>EdgeService</name>
	<description>EdgeService</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Hoxton.SR1</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-zuul</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-openfeign</artifactId>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.springframework.hateoas/spring-hateoas -->
		<dependency>
			<groupId>org.springframework.hateoas</groupId>
			<artifactId>spring-hateoas</artifactId>
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
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

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

```xml
server.port=8089
spring.application.name=edge-service
```

#### All Classes for Edge Service

```java
package com.eureka.edgeservice;

import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.hateoas.config.EnableHypermediaSupport;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import lombok.Data;

@EnableHypermediaSupport(type = EnableHypermediaSupport.HypermediaType.HAL)
@EnableFeignClients
@EnableCircuitBreaker
@EnableDiscoveryClient
@EnableZuulProxy
@SpringBootApplication
public class EdgeServiceApplication {

	public static void main(String[] args) {
		SpringApplication.run(EdgeServiceApplication.class, args);
	}

}

@Data
class Item {
	private String name;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
	
}

@FeignClient("item-catalog-service")
interface ItemClient {	
	@GetMapping("/items")
	List<Item> readItems(); 
		
	@GetMapping("/test")
	List<String> getTest(); 
}

@RestController
class GoodItemAdapterRestController {
	private final ItemClient itemClient;

	public GoodItemAdapterRestController(ItemClient itemClient) {
		this.itemClient = itemClient;
	}
	
    public List<Item>  fallback() {
    	return new ArrayList<>();
    }
    
    public List<String> fallback2() {
    	return new ArrayList<>();
    }
    
    @HystrixCommand(fallbackMethod = "fallback")
    @GetMapping("/top-brands")
    @CrossOrigin(origins = "*")
    public List<Item> goodItem(){    					 
    	List<Item> items = itemClient.readItems(); 
    	return items.stream().filter(this::isGreat).collect(Collectors.toList());    	
    }

    @HystrixCommand(fallbackMethod = "fallback2")
    @GetMapping("/test")
    @CrossOrigin(origins = "*")
    public List<String> getTest(){
    	return itemClient.getTest();    					 
    }

    private boolean isGreat(Item item) {
    	return !item.getName().equals("Nike") &&
    			!item.getName().equals("Adidas") &&
    			 !item.getName().equals("Reebok");
    }
}
```

## EUREKA CLIENT (edge-service)

#### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.2.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.eureka</groupId>
	<artifactId>itemcatalog</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>ItemCatalog</name>
	<description>ItemCatalog</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Hoxton.SR1</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-rest</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<scope>runtime</scope>
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
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

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

```java
server.port=8088
spring.application.name=item-catalog-service

# H2
spring.h2.console.enabled=true
spring.h2.console.path=/h2

# Datasource
#spring.datasource.url=jdbc:h2:file:~/test
#spring.datasource.username=sa
#spring.datasource.password=
#spring.datasource.driver-class-name=org.h2.Driver
```

#### All Classes

```java
package com.eureka.itemcatalog;

import java.io.Serializable;
import java.util.stream.Stream;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.rest.core.annotation.RepositoryRestResource;
import org.springframework.stereotype.Component;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.ToString;

@EnableEurekaClient
@SpringBootApplication
public class ItemCatalogApplication {

	public static void main(String[] args) {
		SpringApplication.run(ItemCatalogApplication.class, args);
	}

}

//@Data
//@AllArgsConstructor
//@NoArgsConstructor
//@ToString
@Entity
class Item implements Serializable{

	private static final long serialVersionUID = 1L;

	@Id
	@GeneratedValue
	private Long id;

	private String name;

	public Item() {
	}

	public Item(String name) { // Parameterized Constructor
		this.name = name;
	}

	public Long getId() {
		return id;
	}

	public void setId(Long id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	@Override
	public String toString() {
		return "Item [id=" + id + ", name=" + name + "]";
	}
		
}

@RepositoryRestResource
interface ItemRepository extends JpaRepository<Item, Long> {
}

@Component
class ItemInitializer implements CommandLineRunner {

	private final ItemRepository itemRepository;

	public ItemInitializer(ItemRepository itemRepository) {
		this.itemRepository = itemRepository;
	}

	@Override
	public void run(String... args) throws Exception {
		Stream.of("Lining", "Puma", "Bad Boy", "Air Jordan", "Nike", "Adidas", "Rebook")
				.forEach(item -> itemRepository.save(new Item(item)));

		itemRepository.findAll().forEach(System.out::println);
	}

}

@RestController
public class TestController {
	
	@Autowired
	private ItemRepository itemRepository;
	
	@GetMapping(value = "/test")
	public List<String> test() {
		return 		Arrays.asList("yahya","berna","ege","deniz");
	}

	@GetMapping(value = "/items", produces = MediaType.APPLICATION_JSON_VALUE)
	public @ResponseBody List<Item> items() {
		List<Item> items = itemRepository.findAll();
		return items;
	}
	
}
```
