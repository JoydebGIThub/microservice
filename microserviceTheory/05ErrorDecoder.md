## Decode the Error code
- If we want the `Feign` decode the error status code also then we can address this by using `decode404=true`
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

@FeignClient(name="employee",url="http://localhost:7091/",decode404=true)
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
- If the producer return other status code then we can handle it by implemention a ErrorDecoder class to handle the error by the producer
```java
package com.accenture.lkm.client;

import org.springframework.stereotype.Component;

import feign.Response;
import feign.codec.ErrorDecoder;
@Component
public class FeignErrorDecoder implements ErrorDecoder {

	@Override
	public Exception decode(String methodKey, Response response) {
		if (response.status() == 500) {
			return new RuntimeException("Employee with given id doesnt exists");
		}
		return null;
	}
}
```
