## The Method in RestTemplete?
- getForObject
- postForObject
- put
- delete
- exchange

## Can we change the method's name and argaments?
- No, if its a get request then I should get for the object of post then I should post for the object
- We have to write a lots of boiler plate code when we are using `restTemplete`.
1. First we need to get the service instance / base URL
2. Create the rest templete object and by using the object we need to call the method
3. process the response
It's a boiler plate code / imperative style of codeing. Its mean we have to tell what to do and how to do everything.

### But now we have something called Netflix Feign - declarative web service client
- It makes job easier
- Feign is an abstraction over `rest page` calls by which `microservices` can `communicate with each other`
- It not required no rest implementation only thing we have to write an `interface`
- From `Netflix Feign` we got a **OpenFeign** - open source project evolved from Netflix Feign
- Currently `Spring Cloud OpenFeign` we uses:
  -> is an extension of `OpenFeign`
  -> simplifies the integration of `OpenFeign` into `Spring-based applications`
  -> no REST implementation is required other than the interface definition.

#### Example:
Consumer
------------
```java
@FeignClient(name="cst-employee-producer")		//producer url / name
interface MyFeignClient{

	@RequestMapping(value = "/emp/controller/getDetails", method = RequestMethod.GET)
	ResponseEntity<List<Employee>> findAll();

}
```
- Its called declarative style of codeing
- Feign does the loadbalancing automatically, we have to add the loadbalancer dependency just

#### Without Eureka
Producer				Feign							Consumer
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
	static
	{
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

Consumer
pom.xml
	dependency -> spring-cloud-starter-openfeign
main class
	@EnableFeignClients
	
- create an interface, annotate it with `@FeignClient` and write the methods declarations
```java
package com.accenture.lkm.client;

import java.util.List;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import com.accenture.lkm.model.Employee;

@FeignClient(name="employee",url="http://localhost:7091/")
public interface MyFeignClient {

	@RequestMapping(value = "/emp/controller/getDetails", 
			method = RequestMethod.GET,
			produces = MediaType.APPLICATION_JSON_VALUE)
	public ResponseEntity<List<Employee>> findAll();

	@RequestMapping(value = "/emp/controller/getDetailsById/{id}", 
			method = RequestMethod.GET,
			produces = MediaType.APPLICATION_JSON_VALUE)
	public ResponseEntity<Employee> findByEmployeeId(@PathVariable("id") Integer employeeId);
	
	@RequestMapping(value="/emp/controller/addEmp",
			method=RequestMethod.POST,
			consumes = MediaType.APPLICATION_JSON_VALUE,
			produces = MediaType.TEXT_HTML_VALUE)
	public ResponseEntity<String> addEmployee(@RequestBody Employee employee);
	
	@RequestMapping(value="/emp/controller/updateEmp",
			method=RequestMethod.PUT,
			consumes = MediaType.APPLICATION_JSON_VALUE,
			produces = MediaType.APPLICATION_JSON_VALUE)
	public ResponseEntity<Employee> updateEmployee(@RequestBody Employee employee);
		
	@RequestMapping(value="/emp/controller/deleteEmp/{id}",
			method=RequestMethod.DELETE,
			produces=MediaType.APPLICATION_JSON_VALUE)
	public ResponseEntity<Employee> deleteEmployee(@PathVariable("id") int myId);
}
```
- Here `@FeignClient(name="employee",url="http://localhost:7091/")` if we are register with eureka then the `name` will play a major role if not then the `url` is the important one

#### With Eureka
```java
package com.accenture.lkm.client;

import java.util.List;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import com.accenture.lkm.model.Employee;

@FeignClient(name="feign-employee-producer",decode404=true)
public interface MyFiegnClient {

	@RequestMapping(value = "/emp/controller/getDetails", 
			method = RequestMethod.GET,
			produces = MediaType.APPLICATION_JSON_VALUE)
	public ResponseEntity<List<Employee>> findAll();

	@RequestMapping(value = "/emp/controller/getDetailsById/{id}", 
			method = RequestMethod.GET,
			produces = MediaType.APPLICATION_JSON_VALUE)
	public ResponseEntity<Employee> findByEmployeeId(@PathVariable("id") Integer employeeId);
	
	@RequestMapping(value="/emp/controller/addEmp",
			method=RequestMethod.POST,
			consumes = MediaType.APPLICATION_JSON_VALUE,
			produces = MediaType.TEXT_HTML_VALUE)
	public ResponseEntity<String> addEmployee(@RequestBody Employee employee);
	
	@RequestMapping(value="/emp/controller/updateEmp",
			method=RequestMethod.PUT,
			consumes = MediaType.APPLICATION_JSON_VALUE,
			produces = MediaType.APPLICATION_JSON_VALUE)
	public ResponseEntity<Employee> updateEmployee(@RequestBody Employee employee);
		
	@RequestMapping(value="/emp/controller/deleteEmp/{id}",
			method=RequestMethod.DELETE,
			produces=MediaType.APPLICATION_JSON_VALUE)
	public ResponseEntity<Employee> deleteEmployee(@PathVariable("id") int myId);
}

```



