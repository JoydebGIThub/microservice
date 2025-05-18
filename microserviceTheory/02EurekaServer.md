## Eureka server:
- How the server mantaining the `registry`, how it is doing the `health` monitoring. The eureka also provided the `health monitoring also`.
- In every **30 sec** the `Producer` and the `Consumer` application will be `sending a heart bit signal` to the `Eureka` to renew its signals to indicate the `Eureka` that the service is alive.
- 30 seconds
MSC -> heartbeat signal to the Eureka Server
- Microservices -> Producer, Consumer
1 min -> how many heartbeat signals -> **4 heartbeat signals**
The spring will add **one more** into it for the `background` things to do and that is the `Thresholds`.
**Renews Thresholds -> 5**
**Renews (last min)** -> 8 because the eureka server is connected with itself (Replicas)
- Eureka will give a `red color message` when the `Renews is less than the Thresholds`
- Self preservation mode --> if for some reason the microservice not sending the heartbeat signal to the server for some failure then the server will by default turns on the **Self preservation mode** and it will wait for 90sec before removing the stop any instance. It is a `featurs` where eureka server is stop expring or evecting the instance from the registry when don't receive the expected heartbeat


