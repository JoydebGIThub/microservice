## Zipkin:
- The ability to debug the issues because the user will click on one button but it might have tigger chain of microservice call, so it will be problem to trace the particular user action because the particular user action can called multiple microservices.
- In addition to this it is difficult to track down why a particular microservices call take so much time
- And it is difficult to try and `Figure out` how request traval through the app. Beacuse in application I might not know every implementation so how the request travel through the application I might not know about it. So, here we are getting a chalange called `Distributed Tracing`.

### Distributed Tracing
- Ability to trace the request spanned across multiple microservice components.
- So, for this we use `Spring CLoud Sleuth`.

### Spring CLoud Sleuth
- This is to address the `distributed tracing` challenge.
- When we add the `Spring Cloud Sleuth` in our application it the `application logs` it provides us `2 ids`

#### Ids:
1. Trace Id (set of span id, it will remain same for the entire request)
2. Span Id (unit of work for the data HTTP request)

#### Example:
- request ------> MSC1 (span id) ------> MSC2  (span id) -------> MSC3  (span id) // for this entire request there will be a single **Trace Id**
- Under the `trace id` we will be having `multiple span id`. `Span id` denotes the task which is execute as the microservice

#### Demo:
Client --->		Consumer 		---->	Mediator		---->		Producer
Trace Id              Span Id
84992a0918e93c3a		84992a0918e93c3a (The 1st MSC hit for the user action, its span id and trace id will be same)
84992a0918e93c3a		2c8a5e49da19cd32
84992a0918e93c3a		26d97e3b766617d5
- How we visualize the follow by seeing the Trace Id and span id in the console
- How To calculate the time taken for the entire request processing
- How to find out which Microservers take a lots of time
- How to calculate how much time a request took for a request trip from one microservice to other
- For all this issue we have **Zipkin**

## Zipkin Server:
- `Sleuth` will send the `traceing` information to the `Zipkin server`.
- If the `Zipkin` depandency is available then the `Sleuth` will send the `traceing` information to the `zipkin server`
- Zipkin server will provide us the `Dashboard` we can visualize the flow
- We can see the `Dependency diagram`
- Zipkin is a `Open source` which is origined by `twitter`
- **Zipkin is for the distributed traceing system**
- It helps to gather timing data which is used to `Troubleshoot` latency problems. So it shows a beautiful diagram which show the `intaraction flow` of the microservice component.
- To working with the zipkin we have to download it first (its a jar file).

```xml
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-sleuth-zipkin</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-sleuth</artifactId>
		</dependency>
```

```java
package com.accenture.lkm;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

import brave.sampler.Sampler;

@SpringBootApplication
public class SleuthZipkinProducer {

    public static void main(String[] args) {
        SpringApplication.run(SleuthZipkinProducer.class, args);
    }

    @Bean
    public Sampler alwaysSampler() {
        return Sampler.ALWAYS_SAMPLE;
    }
}
```
- It is necessay to inform the application on how often the `sleuth log trace` needs to be `exported to the zipkin server`. How ofter we need to export the sleuth log for that create a bean called `alwaysSampler`
- Open the folder where the `zipkin jar` present
- Open the `cmd` in the folder
- run the following command `java -jar zipkin-server-2.21.7-exec.jar`
- Zipkin server will be up now.
- Zipkin server port is `http://localhost:9411`

```java
package com.accenture.lkm.client.controller;

import java.util.Collection;
import java.util.List;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpMethod;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import com.accenture.lkm.model.Employee;

@RestController
public class EmployeeController {
	
	@Autowired
	private RestTemplate restTemplate;
	static String URL ="http://localhost:8095/";
	
	public static Logger logger = LogManager.getLogger(EmployeeController.class);
	
	
	@SuppressWarnings({ "rawtypes", "unchecked" })
	@RequestMapping(value="emp/controller/getDetails",method=RequestMethod.GET,produces=MediaType.APPLICATION_JSON_VALUE)
	public ResponseEntity<Collection<Employee>> getEmployeeDetails(){
		logger.info("Client Controller From method: [getEmployeeDetails] start");
		List list= restTemplate.getForObject(URL+"emp/controller/getDetails",List.class);
		logger.info("Client Controller From method: [getEmployeeDetails] end");
		return new ResponseEntity<Collection<Employee>>(list, HttpStatus.OK);
	}
	
	@RequestMapping(value="emp/controller/getDetailsById/{id}",method=RequestMethod.GET,produces=MediaType.APPLICATION_JSON_VALUE)
	public ResponseEntity<Employee> getEmployeeDetailByEmployeeId(@PathVariable("id") int myId){
		logger.info("Client Controller From method: [getEmployeeDetailByEmployeeId] start");
		Employee employee = restTemplate.getForObject(URL+"emp/controller/getDetailsById/"+myId,Employee.class);
		if(employee!=null){
			logger.info("Client Controller From method: [getEmployeeDetailByEmployeeId] end");
			return new ResponseEntity<Employee>(employee,HttpStatus.OK);
		}
		else
		{
			logger.info("Client Controller From method: [getEmployeeDetailByEmployeeId] end");
			return new ResponseEntity<Employee>(HttpStatus.NOT_FOUND);
		}
		
	}
	
	@RequestMapping(value="/emp/controller/addEmp",
			method=RequestMethod.POST,
			consumes=MediaType.APPLICATION_JSON_VALUE,
			produces=MediaType.TEXT_HTML_VALUE)
	public ResponseEntity<String> addEmployee(@RequestBody Employee employee){
		logger.info("Client Controller From method: [addEmployee] start");
		Integer count=restTemplate.postForObject(URL+"emp/controller/addEmp", new HttpEntity<Employee>(employee),Integer.class );
		employee.setEmployeeId(count);
		logger.info("Client Controller From method: [addEmployee] end");
		return new ResponseEntity<String>("Employee added successfully with id:"+count,HttpStatus.CREATED);
	}
	
	@RequestMapping(value="/emp/controller/updateEmp",
			method=RequestMethod.PUT,
			consumes=MediaType.APPLICATION_JSON_VALUE,
			produces=MediaType.APPLICATION_JSON_VALUE)
	public ResponseEntity<Employee> updateEmployee(@RequestBody Employee employee){
		logger.info("Client Controller From method: [updateEmployee] start");
		Employee employee1 = restTemplate.getForObject(URL+"emp/controller/getDetailsById/"+employee.getEmployeeId(),Employee.class);
		ResponseEntity<Employee> res=null;
		if(employee1!=null)
		{
			logger.info("Client Controller From method: [updateEmployee] if");
			res= restTemplate.exchange(URL+"emp/controller/updateEmp", HttpMethod.PUT, new HttpEntity<Employee>(employee), Employee.class);
		}
		else{
			logger.info("Client Controller From method: [updateEmployee] else");
			return new ResponseEntity<Employee>(HttpStatus.NOT_FOUND);
		}
		logger.info("Client Controller From method: [updateEmployee] end");
		return res;
	}
	@RequestMapping(value="/emp/controller/deleteEmp/{id}",
			method=RequestMethod.DELETE,
			produces=MediaType.APPLICATION_JSON_VALUE)
	public ResponseEntity<Employee> deleteEmployee(@PathVariable("id") int myId){
		logger.info("Client Controller From method: [deleteEmployee] start");
		Employee employee1 = restTemplate.getForObject(URL+"emp/controller/getDetailsById/"+myId,Employee.class);
		ResponseEntity<Employee> res=null;
		if(employee1!=null){
			logger.info("Client Controller From method: [deleteEmployee] if");
			res= restTemplate.exchange(URL+"emp/controller/deleteEmp/"+myId, HttpMethod.DELETE, null, Employee.class);
		}
		else{
			logger.info("Client Controller From method: [deleteEmployee] else");
			res= new ResponseEntity<Employee>(HttpStatus.NOT_FOUND);
		}
		logger.info("Client Controller From method: [deleteEmployee] end");
		return res;
	}
}
```
