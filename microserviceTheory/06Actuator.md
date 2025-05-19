## Actuator:
- Let assum we have 100 and 1000 of microservices, so monitoring each of the microservices is hard.
- So, here `Spring` introduced `Spring Boot Actuator module`.
- This `module` helps of monitor and manage the `spring boot applications` by providing the **production ready features** like `health check-up`, `auditing`, `metrices gathering`, `HTTP tracing` for all this we have `Spring Actuator`.
- All this features we can access through `HTTP end points`. When we using the actuators the actuator provides `lots of end points`, using those end points we can check the `health`.
- This `actuator` can be intrigrated with anything of the external application monitoring tools like `rachel`, `spring cloud admin`. Which provides excellent dashboard to `monitor` and `manage` our applications under one UI.
- This `actuator` uses something called `Micrometer` and `application metrics` to intrigrate our `external application monitoring systems` to make our task super easy to plug in any `application monitoring system` with very little configurations

## Add the Actuator:
- To add the `actuator` in our system we need to add a `spring boot starter` dependency called `spring-boot-starter-actuator`.
```xml
<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
- When we add the dependency many endpoint to check many things in our application is provided by the `actuator`.
- By default only health end point is `exposed` -> `/actuator/health`.
- All the remainder end point are not exposed by default.
- If we want to exposed those end points we need to add something in properties to set up all the end points
```properties
management.endpoints.web.exposure.include=*
```
- `* means "astics"`
- there are many end points present
- All the `actuator end points` are categorized into `3 categories`

### End points Categories:
1. Configuration:
   -
   - `/autoconfig`: It return the Spring automatic configuration, what are the automatic configartion happening in our application if we wanted to see then we need to use `/autoconfig`.
   - `/beans`: It returns the beans that are `configured in the application context`.
   - `/configprops`: It returns what are the configuration we configure in the `application.properties` file and in the `java configuration class`/ configured properties value through application.properties or @Configuration in java code.
   - `/env`: Environment related configuration we can see with the help of `/env`. Environment varibles we are setting, there values, we are `activeing profiles` all those we can see using `/env`. returns the environment configuration values from all the sources like environment variables, commandline arguments and properties files. It also returns the list of the active spring profiles.
   - `/mapping`: returns all the URI templetes and request mapping paths.
   - `/info`: Application informations like who is the author, who are the developers all those information we can see with the help of the end point called `/info`
2. Runtime/Diagostic information
   -
   - `/health`: weather my `database is up` or `my application is up` or how much of the size it is occuping those things are all you can see using `/health` end point. returns the health check information of the application
   - `metrics`: It will give the `detail report of the application transaction`. returns the report of the application transaction volumes and status. it provides an ability to read the application load statics that can be used to create an operational dashboard for the application
   - `/trace`: retruns the detailed trace information for the recent transactions.
3. Operational management
   -
   - `/shutdown`: it gracefully shuts down the application, by default this endpoint is disabled. If we want to enable it then we can give the specification in properties
```properties
endpoints.shutdown.enabled = true // This endpoint is mapped to POST http verb
```

#### URLS:
##### Configuration URLs: 
http://localhost:8090/actuator
http://localhost:8090/actuator/conditions
http://localhost:8090/actuator/beans
http://localhost:8090/actuator/env
http://localhost:8090/actuator/mappings
http://localhost:8090/actuator/info

##### Runtime/Diagnostic information URLs:
http://localhost:8090/actuator/threaddump
http://localhost:8090/actuator/headdump
http://localhost:8090/actuator/health
http://localhost:8090/actuator/metrics

###### Operational Management Functionality URLs:
Below URL is hit by specifying HTTP verb as POST	
http://localhost:8090/actuator/shutdown

```xml
    <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		 <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
    </dependency>
        <!-- for actuator-->
        <dependency>
        	<groupId>org.springframework.boot</groupId>
        	<artifactId>spring-boot-starter-actuator</artifactId>
        </dependency> 
```

```properties
server.port = 8090

# Database
#spring.jpa.hibernate.naming.startegy=org.hibernate.cfg.DefaultNamingStrategy
spring.datasource.url= jdbc:mysql://localhost:3306/springbootdb?useSSL=false
spring.datasource.username=root
spring.datasource.password=root

# Hibernate ddl auto (create, create-drop, update,validate): with "update" the database
# schema will be automatically updated accordingly to java entities found in
# the project
spring.jpa.hibernate.ddl-auto = none

# Show or not log for each sql query
spring.jpa.show-sql = true

# Naming strategy
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl

# Allows Hibernate to generate SQL optimized for a particular DBMS
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.MySQL8Dialect

management.endpoint.health.show-details=ALWAYS

management.endpoints.web.exposure.include=*

management.info.env.enabled=true
info.app.name=SpringBoot Actuator Demo
info.app.description=An Application to demonstrate the usage of Actuator  
info.app.version=1.0
info.app.java.source=1.8
info.app.java.target=1.8

management.endpoint.shutdown.enabled=true
```
```sql
DROP database IF EXISTS springbootdb;
create database springbootdb;
use springbootdb;

DROP TABLE IF EXISTS employee;
CREATE TABLE IF NOT EXISTS employee (
  employeeId int(11) unsigned NOT NULL AUTO_INCREMENT,
  employeeName varchar(20) DEFAULT NULL,
  salary double DEFAULT NULL,
  departmentCode int(11),
  PRIMARY KEY (employeeId)
) ENGINE=InnoDB AUTO_INCREMENT=22 DEFAULT CHARSET=utf8;

INSERT INTO employee (employeeId, employeeName,departmentCode, salary) VALUES
	(1001, 'MSD'   ,101 , 100000),
	(1002, 'James' ,101 , 0),
	(1003, 'Rocky' ,102 , 100000);
commit;

```
### Customize the end point:
- With all the other things in properties we can add some new things to customize tha properties:
```properties
management.endpoints.web.base-path=/appActuator
management.endpoints.web.path-mapping.health=appHealth
management.endpoint.metrics.enabled=false
```
- Write a custom class
```java
package com.accenture.lkm.custom.health.check;

import java.util.Collection;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

import com.accenture.lkm.model.Employee;
import com.accenture.lkm.service.EmployeeServiceImpl;

@Component//Part of component scan
public class CustomHealthCheckIndicator implements HealthIndicator{
	
	@Autowired
	private  EmployeeServiceImpl employeeServiceImpl;
	
	public CustomHealthCheckIndicator() {
		System.out.println("CustomHealthCheckIndicator constructor");
	}
	
	@Override
	public Health health() {
		System.out.println("Calling the Health Method of custom health check indicator");
		Collection<Employee> collectionEmployees = employeeServiceImpl.getEmployeeDetails();
		Health health= null;
		if(collectionEmployees ==null || collectionEmployees.isEmpty()){
			health=Health.down().withDetail("count",0).build(); 
		}
		else{
			health=Health.up().withDetail("count",collectionEmployees.size() ).build(); 
		}
		return health;
	}
}

```
- EmployeeServiceImp
```java
package com.accenture.lkm.service;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.Optional;

import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.accenture.lkm.dao.EmployeeDAO;
import com.accenture.lkm.entity.EmployeeEntity;
import com.accenture.lkm.model.Employee;
@Service
public class EmployeeServiceImpl {

	@Autowired
	private EmployeeDAO employeeDAO;

	public int addEmployee(Employee employee) {

		EmployeeEntity employeeEntity = new EmployeeEntity();
		BeanUtils.copyProperties(employee, employeeEntity);
		EmployeeEntity emp= (EmployeeEntity)employeeDAO.save(employeeEntity);
		System.out.println(emp);
		return emp.getEmployeeId();
	}

	public Collection<Employee> getEmployeeDetails(){
		Collection<EmployeeEntity> collec =employeeDAO.findAll();
		List<Employee> listEmployee = new ArrayList<Employee> ();
		for (EmployeeEntity employeeEntity : collec) {
			Employee employee=new Employee();
			BeanUtils.copyProperties(employeeEntity, employee);
			listEmployee.add(employee);
		}
		return listEmployee;
	}
	
	public Employee getEmployeeDetailByEmployeeId(int employeeId){
		Employee employee =null;
		Optional<EmployeeEntity> employeeEntity= employeeDAO.findById(employeeId);
		if(employeeEntity.isPresent()){
			employee= new Employee();
			BeanUtils.copyProperties(employeeEntity.get(), employee);
		}
		return employee;
	}
	public Employee deleteEmployee(int employeeId){
		Employee employee =null;
		Optional<EmployeeEntity> employeeEntity= employeeDAO.findById(employeeId);
		if(employeeEntity.isPresent()){
			employeeDAO.delete(employeeEntity.get());
			employee= new Employee();
			BeanUtils.copyProperties(employeeEntity, employee);
		}
		return employee;
	}
	
	public Employee updateEmployee(Employee employee){
		Employee employee2 =null;
		Optional<EmployeeEntity> employeeEntity= employeeDAO.findById(employee.getEmployeeId());
		if(employeeEntity.isPresent()){
			BeanUtils.copyProperties(employee, employeeEntity.get());	
			employeeDAO.save(employeeEntity.get());
			employee2= new Employee();
			BeanUtils.copyProperties(employeeEntity, employee2);
		}
		return employee2;
	}
}

```
