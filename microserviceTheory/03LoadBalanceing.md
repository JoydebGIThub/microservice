## Load Balancing:
- If there is more instances the how we balance the load that comming up, that is handle by the **Load Balanceing**.
- There are 2 types of **load balanceing**:
  1. Server-side load balanceing
  2. Client-side load balanceing
 
### Server-side load balancer:
- where the server is responsible for `distribution of the load`.
- In the load balanceing the application is deployed as a `war file` in the `Server A`, `Server B` or `Server C`. A load balancer(AWS application load balancer) will be placed infront of the `Servers` when the client makes the call the request will go to the load balancer only the load balancer decides to which application server it should `forward the request` by **round robin algorithm**. By using that the load balancer will rout the request to the `Server A`, `Server B` or `Server C`.
- The problem here is if one of the `server is stop responding` we have to remove the server manually from the `load balancer IP table`.
- Load balancer keeps `one IP table`. It has to update the `IP table` manually. So, it required a manually intervension.
```
					Client
					   |
					Load Balancer
					    |
				|------	IP table-----------------|      
				|		|         	 |
			   Service A	   Service B		Service C
```
### Client-side load balancer:
- if the caller/ consumer is calling the producer then the `consumer is responsible for the distribution of the load`.
- here let assume we have 2 instance, instance 1 of Producer A and instance 2 of Producer A, will register with the `eureka server`.
- If we want to send the 1st request to instance 1 and 2nd to instance 2 then we add the `load balancer` infront of the `consumer`.
- This load balancer will help to pick the `producer` so `service resistery always update itself` if `one instance` goes down it removes that from the `registry`. So, when the load balancer talk to the server it always get the updated things. So, there is no need for manual intervension and the load balancer is in the client side.
- By default it follows `round robin`.
- So microservice load balanceing mainly use `Client side load balanceing`.
```

		Producer A - Instance 1					Producer A - Instance 2
				|			                               |
				|-------------------Eureka Server----------------------|
							|
						    Load Balancer
							|
						     Consumer
```
- there are 2 ways
  1. LoadBalancer RestTemplete
  2. LoadBalancerClient
 
#### DEMO:
1. Execute the Eureka Server
2. Execute the 1004_Employee_Producer_LoadBalancer_Demo 8085
3. Change the port number in the application.properties 8086
4. Execute the 1004_Employee_Producer_LoadBalancer_Demo
5. Change the port number in the application.properties 8087
6. Execute the 1004_Employee_Producer_LoadBalancer_Demo
7. Execute 1005_Employee_Consumer_LoadBalancer_Demo
8. POSTMAN
	http://localhost:7096/consumer1/getDetails	(multiple times and observe the console)
  - Using RestTemplete
1. 1st add @LoadBalanced and create a RestTemplete bean
```java
package com.accenture.lkm;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@EnableDiscoveryClient
public class LoadBalancerConsumer {

    public static void main(String[] args){
       SpringApplication.run(LoadBalancerConsumer.class, args);
    }
    
	@LoadBalanced
	@Bean
	public RestTemplate restTemplate() {
		return new RestTemplate();
	}
}
```
2. 2nd in the consumer application we are injecting the `RestTemplete` and it is a `loadbalanced restTemplete`
```java
package com.accenture.lkm.client;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpMethod;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class ConsumerControllerClient1 {

	@Autowired
	private RestTemplate restTemplate;

	@RequestMapping(path = "/consumer1/getDetails")
	public ResponseEntity<String> getEmployee(){

		String baseUrl = "http://cst-employee-producer";
		baseUrl = baseUrl + "/emp/controller/getDetails";

		ResponseEntity<String> response = null;
		response = restTemplate.exchange(baseUrl, HttpMethod.GET,null, String.class);
		
		return response;
	}
}
```
	http://localhost:7096/consumer2/getDetails	(multiple times and observe the console)
  - Using loadBalancer:
1. 1st step:
```properties:
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```
2. 2nd step:
```java
package com.accenture.lkm.client;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.loadbalancer.LoadBalancerClient;
import org.springframework.http.HttpMethod;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class ConsumerControllerClient2 {
	
	@Autowired
	private LoadBalancerClient loadBalancer;
	
	@RequestMapping(path="/consumer2/getDetails")
	public ResponseEntity<String> getEmployee(){
		//Load balancer round robin algorithm is applied and one instance of the producer is choosen
		ServiceInstance serviceInstance=loadBalancer.choose("cst-employee-producer");//Load balancer round robin algorithim is applied and one instance of the producer is choosen
		String baseUrl=serviceInstance.getUri().toString();
		baseUrl=baseUrl+"/emp/controller/getDetails";
	
		System.out.println(">>>From Client -> Server Instance Id port number: >>"+serviceInstance.getPort());
		
		//RestTemplate is not a Spring managed bean, it is user defined object
		RestTemplate restTemplate = new RestTemplate();
		ResponseEntity<String> response=restTemplate.exchange(baseUrl,HttpMethod.GET,null,String.class);
		
		return response;
	}
}
```
- here 3 instances of producer is running and the consumer is only knows the name of the producer so now the picking up one by one instance is given to the load balancer and which is on the client/consumer side so we called it as a `client side load balancer`
