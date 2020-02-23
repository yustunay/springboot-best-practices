### SPRING CLOUD CIRCUIT BREAKER - NETFLIX HYSTRIX
#### Dependencies (with dashboard)
```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>	
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>	
```


#### Add @EnableCircuitBreaker Annotation In Main Class
```java
@EnableCircuitBreaker
@EnableEurekaClient
@SpringBootApplication
public class EdgeServiceApplication {

	public static void main(String[] args) {
		SpringApplication.run(EdgeServiceApplication.class, args);
	}

}
```

#### @HystrixCommand should be used on method. If two different service is called in this method, services have to be created in different classes!
```java
    public List<Item> getFallbackTopBrands() {
    	System.out.println("Fallback method executed!");
    	return new ArrayList<>();
    }
    
    @HystrixCommand(fallbackMethod = "getFallbackTopBrands",
    		commandProperties = {
    				@HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "2000"),
    				@HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "5"),
    				@HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "50"),
    				@HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "5000")    				
    		})
    @GetMapping("/top-brands")
    @CrossOrigin(origins = "*")
    public List<Item> getTopBrands(){    					 
    	List<Item> items = itemClient.readItems(); 
    	return items.stream().filter(this::isGreat).collect(Collectors.toList());    	
    }
```

### Hystrix Dashboard
#### Dependencies
```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

#### Add @EnableHystrixDashboard Annotation On the Main Class
```java
@EnableHypermediaSupport(type = EnableHypermediaSupport.HypermediaType.HAL)
@EnableFeignClients
@EnableCircuitBreaker
@EnableHystrixDashboard
@EnableDiscoveryClient
@EnableZuulProxy
@SpringBootApplication
public class EdgeServiceApplication {

	public static void main(String[] args) {
		SpringApplication.run(EdgeServiceApplication.class, args);
	}

}
```

#### Add following property into applcation.properties file
```
management.endpoints.web.exposure.include=hystrix.stream
```

#### Dashboard Link
http://localhost:8089/hystrix

#### Enter Hystrix App Info
http://<HYSTRIX-APP>:<PORT>/actuator/hystrix.stream


#### Using BulkHead Pattern (Using Different Pools)
```
@HystrixCommand(
   fallbackMethod = "getFallbackCatalogItem",
   threadPoolKey = "movieInfoPool",
   threadPoolProperties = {
		@HystrixProperty(name = "coreSize", value = "20"),
		@HystrixProperty(name = "maxQueueSize", value = "10")
		}
   )
public CatalogItem getCatalogItem(Rating raing){
...  
```


Check out the youtube videos [Java Brains](https://www.youtube.com/watch?v=WfsomLHaSzQ&list=PLqq-6Pq4lTTbXZY_elyGv7IkKrfkSrX5e&index=21)