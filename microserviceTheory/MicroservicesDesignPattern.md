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
|Producer A - instance1|10.4.2.10:8080|
|Producer A - instance2|10.4.2.11:8081|
|Producer A - instance3|10.4.2.11:8082|
|Producer B - instance1|20.4.2.10:9080|
|Producer B - instance2|20.4.2.11:9081|
- If a client want to send any request to Producer A then it needs to know about `IP` and the `Port`
- Let assume this microservices are running on `vertualized` and `containerized` enviroments where the no of instances and locations are changes dynamically. In Docker and Kubarnative container it may run the IP and port will be dynamically changes then how the client will know about the IP and the Port that will be a chalanges.
- So, microservices are mainly run on `vertualized` and `containerized` environment and the no of instances and services and the locations are changes dynamically.
- How does the client `discover` the location of the microservice which are dynamically changes here we need to use `Service Registry` and `Client site service discovery` design pattern.

**Eureka Server** is a `Service Registry` and `Client site service discovery` design pattern **implementations**. 
- Eureka server is an `applications` that holds informations about the `Client and Service` applications. Every microservice will be `register` to the `Eureka server`, so that `Eureka server` knowns all the microservices running on which port no and IP address all that information will be present in the server.
- Eureka Server is also called as **Discovery Server**

#### How Eureka Server works:
- In the Eureka server has `Service Registry`. So, all the microservices `Register` with the `Eureka server`. So, all the information will be stored under `Service Registry`.
- Now the `Client Application` will also register with `Eureka`, we called it as `Registry aware client`, it means it knows about the `Registry`.
- If the client now want to connect with the `Producer A - instance1` it goes to the `Service Registry` and from there it knows about the port and IP and will be able to connect with it.
- When makeing a request to the service the client `optaines` the location of the service instance by querying the `Service registry` which knows the location of all the `service instances`
