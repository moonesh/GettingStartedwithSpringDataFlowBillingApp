#Spring Cloud Data Flow - Billing App using KAFKA binder

##About App :
This app calculates the cost of phone call bills for users considering the call duration and data usage. 

The app has 3 microservices forming the data pipeline - as explained below: 
* billing-usage-detail-sender : This acts as "SOURCE" from where the Users usage details are pushed to Kafka binder on a topic "usage-detail"
* billing-usage-cost-processor : This acts as "PROCESSOR" which listens to the Kafka topic "usage-detail" from SOURCE app, transforms the data using some logic for calculating the usage cost per user on basis of it data usage and call duration. Finally it pushes back the processed data to the Kafka queue on topic "usage-cost" for the logger application.
* billing-cost-logger: This microservice acts as a "SINK" which listens to the processed data on topic "usage-cost" and spits out the results as logs. 

##SETUP and RUN the APP on Local Spring Data Flow Server: 
###### 1. Chekout all 3 microservices from git and build the jars using "clean install" . Install will place the built jars in .m2 repository - as it the Spring Data Flow Sever will require the maven uristo register these apps. Also note the follwing points: 
  - In applicaion.properties files of all 3 applications we have configured Kafka Topics (input and output data binding) - as explained above.
  - The maven uri for each component/microservice will be of the follwoing pattern:
     - maven://<groupId>:<artifactId>[:<extension>[:<classifier>]]:<version>

###### 2. RUN KAFKA and Zookeeper : 
 - Take the following steps to run both and create the topics as mentioned above:
     -  git clone https://github.com/wurstmeister/kafka-docker
     - Replace the code in kafka-docker > docker-compose.yml with:
	 
	   ```
		    version: '2'
			services:
			  zookeeper:
				image: wurstmeister/zookeeper
				ports:
				  - "2181:2181"
			  kafka:
				build: .
				ports:
				  - "9092:9092"
				environment:
				  KAFKA_ADVERTISED_HOST_NAME: 127.0.0.1
				  KAFKA_CREATE_TOPICS: "usage-detail,usage-cost"
				  KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
				volumes:
				  - /var/run/docker.sock:/var/run/docker.sock
 
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

###### 10) Strat & Deploy Stream 
> stream deploy --name billing-stream

###### 11) References [Download](https://dataflow.spring.io/docs/stream-developer-guides/streams/standalone-stream-kafka/).
