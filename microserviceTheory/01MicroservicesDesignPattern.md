## Microservice Design Pattern
- This design pattern is used to over come the chalanges in the distributed environment. There are lots of microservice design pattern.
  1. Service Registry and Discovery
  2. Load balancing
  3. circute Breaker
  4. Distibuted messageing
 
- This design pattern is given by the `spring cloud` itself. They have already implemented this design patterns and they are providing to us in the from of `Netflex components`.
- Spring cloud version is not bundle with the spring boot. Spring boot has different versions and spring cloud has different versions
- The version of spring boot we uses then the compitable spring cloud version should be used
- In the spring cloud `Netflex` it provides `Netflex open source` software which can be intrigrated with spring boot
- Some of the **netflex features** are `Service discovery`, `declarated rest client` in our spring cloud netflex
- We are going to see different microservices design pattern and the implementation of it how to use it in our applications so that we can address the challenges that we are fetching in the microservice environment

|Spring boot|Spring Cloud|
|-----------|------------|
|1.5x       |Camden, Dalston|
|2.0x       |Edgeware|
|2.1x       |Finchley|
|2.2x       |Greenwich|
|2.7x       |2021.0.3|

### 1st Design pattern:
#### Service Registry and Client side service discovery design pattern
- Let assume we have some Microservice instances and different Ip and post

|Instance		| IP		|
|-----------------------|---------------|
|Producer A - instance1|10.4.2.10:8080|
|Producer A - instance2|10.4.2.11:8081|
|Producer A - instance3|10.4.2.11:8082|
|Producer B - instance1|20.4.2.10:9080|
|Producer B - instance2|20.4.2.11:9081|

- If a client want to send any request to Producer A then it needs to know about `IP` and the `Port`
- Let assume this microservices are running on `vertualized` and `containerized` enviroments where the no of instances and locations are changes dynamically. In Docker and Kubarnative container it may run the IP and port will be dynamically changes then how the client will know about the IP and the Port that will be a chalanges.
- So, microservices are mainly run on `vertualized` and `containerized` environment and the no of instances and services and the locations are changes dynamically.
- How does the client `discover` the location of the microservice which are dynamically changes here we need to use **Service Registry** and **Client site service discovery** design pattern.

**Eureka Server** is a `Service Registry` and `Client site service discovery` design pattern **implementations**. 
- Eureka server is an `applications` that holds informations about the `Client and Service` applications. Every microservice will be `register` to the `Eureka server`, so that `Eureka server` knowns all the microservices running on which port no and IP address all that information will be present in the server.
- Eureka Server is also called as **Discovery Server**

#### How Eureka Server works:
- In the Eureka server has `Service Registry`. So, all the microservices `Register` with the `Eureka server`. So, all the information will be stored under `Service Registry`.
- Now the `Client Application` will also register with `Eureka`, we called it as `Registry aware client`, it means it knows about the `Registry`.
- If the client now want to connect with the `Producer A - instance1` it goes to the `Service Registry` and from there it knows about the port and IP and will be able to connect with it.
- When makeing a request to the service the client `optaines` the location of the service instance by querying the `Service registry` which knows the location of all the `service instances`
```
			|--------		  Producer A - instance1   		10.4.2.10:8080
			| |--------		Producer A - instance2			10.4.2.11:8081
			| | |--------	Producer A - instance3			10.4.2.11:8082
	|---Service Registry			
	|		| |--------		Producer B - instance1			20.4.2.10:9080
	|		|--------		  Producer B - instance2			20.4.2.11:9081
	|		
	|		
Client
Registry aware client
```
##### Demo:
```
		|-----------------Eureka Server-------------------------|
	register|s		producer, ip, port			|
		|							|
Employee Producer			calls			Employee Consumer
producer, ip, port		<---------------------		consumer, ip, port
```
**Eureka server** is working in differnt modes:
- 
1. Independent: We will having `1 Eureka server` and all the `microservice` will `register` with that `Eureka server`, we called is as `Independent mode`.
2. Cluster: Instead of only one eureka server we can keep `multiple eureka servers` and from a `Cluster`.
  Example:
   - Eureka server 1 connect with Eureka server 2 and 3
   - Eureka server 2 connect with Eureka server 3 and 1
   - Eureka server 3 connect with Eureka server 1 and 2
  
Eureka Server1 has 2 MSC `register`. The register information of `Eureka server1` will be syncd with `Eureka server2`. What MSC register with `Eureka server1` information will be present in `Eureka server2`. If 1 is down the information will be present in the 2 also

#### Independent Mode:
Eureka Server
	pom.xml
		-> parent -> spring boot -> 2.7.3
		-> dependency -> spring-cloud-starter-netflix-eureka-server
		-> dependencyManagement -> spring-cloud-dependencies (2021.0.3)

	main class
		-> @SpringBootApplication
		-> @EnableEurekaServer (It is used to enable the `Registry` with which other applications can communicate)
		
	application.properties
		-> eureka.client.serviceUrl.defaultZone:http://localhost:7090/eureka (for independent mode we give the `eureka server's` own url, in case of `cluster mode` here we give the other server's url: http://localhost:7091/eureka, http://localhost:7092/eureka)
		-> eureka.client.register-with-eureka=false (we set this `true` if the eureka server is running in cluster mode)
		-> eureka.client.fetch-registry=false (we set this `true` if the eureka server is running in cluster mode)


Employee Producer
	pom.xml
		-> parent -> spring boot -> 2.7.3
		-> dependency -> spring-cloud-starter-netflix-eureka-client (if eureka is server, then what are the microservice register with eureka called is as `client`)
		-> dependencyManagement -> spring-cloud-dependencies (2021.0.3)
	main class
		-> @SpringBootApplication
		-> @EnableDiscoveryClient (this producer application is used to register the producer as a `Discovery client` with eureka. This will also help the producer application to discover the other consumers)
	application.properties
		-> eureka.client.serviceUrl.defaultZone:http://localhost:7090/eureka (eureka server's URL)
	bootstrap.properties (The things we configer here we can't change it, this values here will not be overridden in any properties file. It has the high precedence then other properties files)
		-> spring.application.name=cst-employee-producer (name of the application to avoid confusions)


Employee Consumer
	pom.xml
		-> parent -> spring boot -> 2.7.3
		-> dependency -> spring-cloud-starter-netflix-eureka-client
		-> dependencyManagement -> spring-cloud-dependencies (2021.0.3)
		
	main class
		-> @SpringBootApplication
		-> @EnableDiscoveryClient
		
	application.properties
		-> eureka.client.serviceUrl.defaultZone: http://localhost:7090/eureka
		
	bootstrap.properties
		-> spring.application.name = cst-employee-consumer

- So, how to consumer can know about the producer with help of `eureka server`. So, in consumer site we needs to write the `Discovery client` it should be `autowired`. using the `Discovery client` we can get the `instances of the Producer`. There can be more then one instances so it gives the `list of server instances`. From the list we can get the 1st instance. And from the service instance we can get the `url`. Then using the `RestTemplete` we can call the producer.
```java
  @Autowired
	DiscoveryClient discoveryClient;
	
	List<ServiceInstance> instances = discoveryClient.getInstances("cst-employee-producer");
	ServiceInstance instance = instances.get(0);
	
	String baseUrl = instance.getUri().toString();
	baseUrl = baseUrl + "/emp/controller/getDetails";
	
	RestTemplate --> rest call
```

#### Producer we have controller:
```java
package com.accenture.lkm.controller;

import java.util.Collection;
import java.util.LinkedHashMap;
import java.util.Map;

import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import com.accenture.lkm.model.Employee;

@RestController
public class EmployeeController {
	
	private static Map<Integer, Employee> mapOfEmloyeess = new LinkedHashMap<Integer, Employee>();
	static int count=10003;
	static{
		mapOfEmloyeess.put(10001, new Employee("Jack",10001,12345.6,1001));
		mapOfEmloyeess.put(10002, new Employee("Justin",10002,12355.6,1002));
		mapOfEmloyeess.put(10003, new Employee("Eric",10003,12445.6,1003));
	}
	
	@RequestMapping(value="emp/controller/getDetails",method=RequestMethod.GET,produces=MediaType.APPLICATION_JSON_VALUE)
	public ResponseEntity<Collection<Employee>> getEmployeeDetails(){
		Collection <Employee> listEmployee = mapOfEmloyeess.values();
		return new ResponseEntity<Collection<Employee>>(listEmployee, HttpStatus.OK);
	}
	
	@RequestMapping(value="emp/controller/getDetailsById/{id}",method=RequestMethod.GET,produces=MediaType.APPLICATION_JSON_VALUE)
	public ResponseEntity<Employee> getEmployeeDetailByEmployeeId(@PathVariable("id") int myId){
		Employee employee = mapOfEmloyeess.get(myId);
		if(employee!=null){
			return new ResponseEntity<Employee>(employee,HttpStatus.OK);
		}
		else{
			return new ResponseEntity<Employee>(HttpStatus.NOT_FOUND);
		}
	}
	
	@RequestMapping(value="/emp/controller/addEmp",method=RequestMethod.POST,consumes=MediaType.APPLICATION_JSON_VALUE,produces=MediaType.TEXT_HTML_VALUE)
	public ResponseEntity<String> addEmployee(@RequestBody Employee employee){
		count++;
		mapOfEmloyeess.put(count,employee);
		employee.setEmployeeId(count);
		return new ResponseEntity<String>("Employee added successfully with id:"+count,HttpStatus.CREATED);
	}
	
	@RequestMapping(value="/emp/controller/updateEmp",method=RequestMethod.PUT,consumes=MediaType.APPLICATION_JSON_VALUE,produces=MediaType.APPLICATION_JSON_VALUE)
	public ResponseEntity<Employee> updateEmployee(@RequestBody Employee employee){
		if(mapOfEmloyeess.get(employee.getEmployeeId())==null){
			Employee employee2=null;
			return new ResponseEntity<Employee>(employee2,HttpStatus.INTERNAL_SERVER_ERROR);
		}
		mapOfEmloyeess.put(employee.getEmployeeId(),employee);
		return new ResponseEntity<Employee>(employee,HttpStatus.OK);
	}
	
	@RequestMapping(value="/emp/controller/deleteEmp/{id}",method=RequestMethod.DELETE,produces=MediaType.APPLICATION_JSON_VALUE)
	public ResponseEntity<Employee> deleteEmployee(@PathVariable("id") int myId){
		if(mapOfEmloyeess.get(myId)==null){
			Employee employee2=null;
			return new ResponseEntity<Employee>(employee2,HttpStatus.INTERNAL_SERVER_ERROR);
		}
		Employee employee = mapOfEmloyeess.remove(myId);
		return new ResponseEntity<Employee>(employee,HttpStatus.OK);
	}
}

```
#### Producer we have Employee:
```java
package com.accenture.lkm.model;

public class Employee {
	private String employeeName;
	private Integer employeeId;
	private double salary;
	private Integer departmentCode;

	
	
	public Employee() {
		super();
	}

	public Employee(String employeeName, Integer employeeId, double salary,
			Integer departmentCode) {
		super();
		this.employeeName = employeeName;
		this.employeeId = employeeId;
		this.salary = salary;
		this.departmentCode = departmentCode;
	}

	

	public String getEmployeeName() {
		return employeeName;
	}

	public void setEmployeeName(String employeeName) {
		this.employeeName = employeeName;
	}

	public Integer getEmployeeId() {
		return employeeId;
	}

	public void setEmployeeId(Integer employeeId) {
		this.employeeId = employeeId;
	}

	public double getSalary() {
		return salary;
	}

	public void setSalary(double salary) {
		this.salary = salary;
	}

	public Integer getDepartmentCode() {
		return departmentCode;
	}

	public void setDepartmentCode(Integer departmentCode) {
		this.departmentCode = departmentCode;
	}
}

```
#### Producer we have also the same Employee:
#### Producer we have Controller:
```java
package com.accenture.lkm.client;

import java.util.ArrayList;
import java.util.Collection;
import java.util.LinkedHashMap;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.http.HttpMethod;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import com.accenture.lkm.model.Employee;
import com.fasterxml.jackson.databind.ObjectMapper;


@RestController
public class ConsumerControllerClient {

	@Autowired
	private DiscoveryClient discoveryClient;
	
	//public String REST_SERVICE_URI = "http://localhost:7091/emp/controller/";

	@SuppressWarnings({ "rawtypes", "unchecked" })
	@RequestMapping(value = "emp/controller/getDetailsClient", method = RequestMethod.GET, produces = MediaType.APPLICATION_JSON_VALUE)
	public ResponseEntity<Collection<Employee>> getEmployee(){
		
		List<ServiceInstance> instances = discoveryClient.getInstances("cst-employee-producer");
		ServiceInstance serviceInstance = instances.get(0); // IPs, Port  
		String baseUrl = serviceInstance.getUri().toString();// http://ip:port/
		baseUrl = baseUrl + "/emp/controller/getDetails";
		System.out.println(baseUrl);
		RestTemplate restTemplate = new RestTemplate();
		ResponseEntity<List> employeeMapRespEntity = restTemplate.exchange(baseUrl, HttpMethod.GET, null,List.class); 
		
		List<LinkedHashMap<String, Object>>  employeeMap = employeeMapRespEntity.getBody();
		ObjectMapper mapper =  new ObjectMapper();
		Collection<Employee> list  = new ArrayList<Employee>();
		if (employeeMap != null) {
			for (LinkedHashMap<String, Object> map : employeeMap) {
				//Map object should be converted to Employee type 
				Employee emp=mapper.convertValue(map, Employee.class);
				list.add(emp);
			}
		}
		return new ResponseEntity<Collection<Employee>>(list,employeeMapRespEntity.getStatusCode());
	}
}
```
