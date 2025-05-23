## DistributedLogManagement:
- Our Microservice components are Distributed, means one microservices will be running in one container, so different microservices are running on different microservices.
- So, the log generated in that microservice will be present in that particuler container or a vertual machine.
- So, the log generated by the microservice container to may be there in a system war, or server or in the Docker container where ever it is running.
- Logs are also distributed now, because our applications are running on different place
- The application which have contains of multiple microservices and services are running on different machines, so when a client request comes, the request will go to multiple services or service instances, and each service instances are generated logs, this log files will have errors, warings, informations, debug messages in different formats.
- Microservices are asolated and they don't share the database or the log file, so this is introducing a new chalange called `distributed log management / log aggregation`.
- The ability to view the logs of the request which are distributed accross multiple services, so I want to see the logs of entire service request, so I need to `aggregate` all the logs then I can see for a particular request.
- So we introduced `ELK`

## ELK:
- It is a set of 3 `open source` tool.
  1. E -> Elasticsearch
  2. L -> Logstash
  3. K -> Kibana
- The intrigation of this 3 open source tools are used to address the log aggregation chalange in the microservices architecture.

### Elasticsearch:
- It is a distributed, JSON-based search and analytics engine
- It also has a NoSQL database engine to store the logs, to search, to analyze big volumes of data
- It is mostly used by the application that requires the complex search requirement when we have a large volume of data, we wanted to do implement the complex search requirement so I go for this elasticsearch.
- Appart from the search the tool is also offering annalytic and many advance features.
- https://www.elastic.co/downloads/elasticsearch
- elasticsearch-6.8.1

### Logstash:
- This is used for aggregating and process the data then it will send the data to `Elasticsearch` to store it in the database.
- The logs that are getting generated, consider it is in `unstructured` manner, the `logstash` convert the unstructured log data to the `JSON-sturctured` formate.
- So, for this the logstash has a `filter plug in`, that will do the `convertion`.
- `Logstash` is a opensource data processing pipline that enables us to injust the data from multiple sources symaltaniously.
- It it tranfrom the data before the data index to the `Elasticsearch`
- https://www.elastic.co/downloads/logstash

### Kibana:
- Kibana is a `data visualisation tool`.
- This tool is used to `visualize` the `elasticsearch` documents and it helps us have a quick inside into it.
- The `Kibana dashboard` has a lots of `diagram` like `intaractive diagram`, `visualize the complex querys`.
- It like a `dashboard` where we can visualize the data that is stored in the `Elasticsearch`.
- So, we can do the advance data analize, visualizeing the data in the from of charts, tables, maps all that is possible in `kibana`
- https://www.elastic.co/downloads/kibana

#### ELK architechture has 2 different types:
1. All the MSC are running on the local machine (all the log files are in the same machine, so we aggergate the log files then stored in the elasticsearch and use kibana to visualize the data)
2. All the MSC are running on different machines (all the log files will be an the different machines, so before aggeregate the log file, 1st we need to take the log file using **FileBeats**. Its act as a `data shiper`, that is install as agent on the server running the microservice components, it will take the log file the ship it to the machine where we need it).
- ELK1:
<img width="766" alt="ELK1" src="https://github.com/user-attachments/assets/44ca7cad-54be-428e-8a96-14f94d8cb720" />

- ELK2:
<img width="766" alt="ELK2" src="https://github.com/user-attachments/assets/4a6d7d22-403f-4216-a926-b313c3eca2bf" />


