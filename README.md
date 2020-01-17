# Spring Cloud Data Flow - Billing App using KAFKA binder

## About App :
This app calculates the cost of phone call bills for users considering the call duration and data usage. 

The app has 3 microservices forming the data pipeline - as explained below: 
* billing-usage-detail-sender : This acts as "SOURCE" from where the Users usage details are pushed to Kafka binder on a topic "usage-detail"
* billing-usage-cost-processor : This acts as "PROCESSOR" which listens to the Kafka topic "usage-detail" from SOURCE app, transforms the data using some logic for calculating the usage cost per user on basis of it data usage and call duration. Finally it pushes back the processed data to the Kafka queue on topic "usage-cost" for the logger application.
* billing-cost-logger: This microservice acts as a "SINK" which listens to the processed data on topic "usage-cost" and spits out the results as logs. 

## SETUP and RUN the APP on Local Spring Data Flow Server: 
###### 1. Chekout all 3 microservices from git and build the jars using "clean install" . Install will place the built jars in .m2 repository - as it the Spring Data Flow Sever will require the maven uristo register these apps. Also note the follwing points: 
  - In applicaion.properties files of all 3 applications we have configured Kafka Topics (input and output data binding) - as explained above.
  - The maven uri for each component/microservice will be of the follwoing pattern:
     -
     ```
     maven://<groupId>:<artifactId>[:<extension>[:<classifier>]]:<version>

###### 2. RUN KAFKA, Zookeeper and Kafka-UI: 
  - Run the following command 
      > docker-compose -f docker-compose-billing-app up -d
  - Open the Kafka UI  @ localhost:9000 and once you deploy the stream (as we'll see below) - you can monitor all the topics, partitions  and messages. 
  - Note: The topic is automatically generated using the format : 
 
 ``` 
   <stream name>.<name of the app registered> 
   (For example in ourcase it will auto-generate two Topics: 
   billing-stream.usage-sender &  billing-stream.usage-cost-processor (from steps 7 and 8 ))
   ```  
###### 3) Download Spring Cloud Data Flow Server jar [Download](https://repo.spring.io/milestone/org/springframework/cloud/spring-cloud-dataflow-server-local/1.7.4.RELEASE/spring-cloud-dataflow-server-local-1.7.4.RELEASE.jar).

###### 4) Strat Spring Cloud Data Flow Server (Check on : localhost:9393 after start)
> java -jar spring-cloud-dataflow-server-local-1.7.4.RELEASE.jar

###### 5) Download Spring Cloud Data Flow Shell jar [Download](http://repo.spring.io/milestone/org/springframework/cloud/spring-cloud-dataflow-shell/1.3.0.M1/spring-cloud-dataflow-shell-1.3.0.M1.jar).

###### 6) Strat Spring Cloud Data Flow shell 
> java -jar spring-cloud-dataflow-shell-1.3.0.M1.jar

###### 7) Register all 3 microservices to Spring Cloud Data Flow Server using below commands in Shell prompt
> app register --name usage-sender --type source --uri maven://io.spring.dataflow.sample:usage-detail-sender:jar:0.0.1-RELEASE

> app register --name usage-cost-processor --type processor --uri maven://io.spring.dataflow.sample:usage-cost-processor:jar:0.0.1-RELEASE

> app register --name cost-logger --type sink --uri maven://io.spring.dataflow.sample:usage-cost-logger:jar:0.0.1-RELEASE

###### 8) Create Cloud Stream to connect between all microservices registered in spring cloud data flow server
> stream create --name billing-stream --definition 'usage-sender | usage-cost-processor | cost-logger'

###### 9) Strat & Deploy Stream 
> stream deploy --name billing-stream

###### 10) Check the Setup:
 - Go to dashboard i.e. localhost:9393 and you'll see application registered and stream created as intented.
 - To check the application logs go to the cindow where the SDF server was started at the end there will be the path to the folder something like this (in windows) :   Logs will be in C:\Users\moonesh\AppData\Local\Temp\spring-cloud-deployer-8031984761677234952\billing-stream-1579184372883\billing-stream.usage-sender

###### 12) References 
 - [View](https://dataflow.spring.io/docs/stream-developer-guides/streams/standalone-stream-kafka/).
 - [Video](https://www.youtube.com/watch?v=THxJJzyVVmg&list=PLVz2XdJiJQxz3L2Onpxbel6r72IDdWrJh&index=18).
