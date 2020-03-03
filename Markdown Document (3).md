<center><h1>Introduction and Setup for Yantriks Availability 3.0</h1></center>


[Introduction](#introduction)<br>
[ System Requirements ](#sys)  <br>
&emsp; [Hardware Requirements](#Hard)   <br>
&emsp; [Software Requirements](#Soft)   <br>
&emsp; [Installation Components](#installation)  <br>
[Installation and Setup](#installationandsetup)  <br>
&emsp;[Overview](#overview)  <br>
&emsp;[Pre-Requisites](#pre-requisites)  <br>
&emsp;&nbsp; &nbsp; &nbsp; [Creation of Kafka topics](#creation)  <br>
&emsp;&emsp; &nbsp; &nbsp; &nbsp; [Kafka topics](#kafkatopics)  <br>
&emsp;&emsp;&emsp;&emsp;[Topics for master and eligibility data key updates](#master)  <br>
&emsp;&emsp;&emsp;&emsp;[Entity Topics for consuming updates of master and eligibility data](#consuming)  <br>
&emsp;&emsp;&emsp;&emsp;[Topics for consuming Supply/ Supply Configuration from Inventory Lite/ 360](#supply)  <br>
&emsp;&emsp;&emsp;&emsp;[Internal Cache Update Topics for Yantriks Availability Services](#cache)  <br>
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp;[Topic for publishing product ATP](#product)   <br>
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp;[Cassandra Setup](#cassandra)  <br>
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp;[Application Initialization & Startup on VM](#app)  <br>
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp;[Application Initialization on Cloud (GCP K8S Cluster)](#cloud)  <br>
[Application Properties](#app-properties)  <br>
&emsp;[Overview](#over-view)     <br>
&emsp;&emsp;&ensp;[Logging level properties](#level)  <br>
&emsp;&emsp;&ensp;[Server and Spring Boot Profile properties](#server)  <br>
&emsp;&emsp;&ensp;[Management & Metrics Publish properties](#mgmt)  <br>
&emsp;&emsp;&ensp;[Cassandra properties](#cassandrasetup)  <br>
&emsp;&emsp;&ensp;[Kafka Properties](#kafka)  <br>
&emsp;&emsp;&emsp;&emsp;[kafka.consumer.yas.supply.transaction-id-separator :](#consumer)  <br>
&emsp;&emsp;&ensp; [Availability Service Properties](#service)  <br>
&emsp;&emsp;&ensp; [Kubernetes discovery and Hazelcast Properties](#kubernetes)  <br>
&emsp; [Kafka Topic Name Overrides](#kafkatopic) <br>
[Samples](#samples)  <br>
&emsp;[Sample Message Published to aggregated-inventory topic](#samplemsg)  <br>
&emsp;[Sample application.yaml](#sampleapp)  <br>


<a name="introduction"></a>
<h1>Introduction</h1>



This guide gives a brief overview of deployment of Yantriks Availability Services and the related components and dependencies.<br>

<a name="sys"></a>
# System Requirements<br>
<a name="hard"></a>
## Hardware requirements<br>
The application can be deployed on bare metal VMs or on AWS/ GCP and Azure.<br>
The applications can be run in a containerized deployment mode or can be deployed on independent VMs.<br>
<a name="soft"></a>
## Software Requirements<br>
- Apache Cassandra : 3.11 & above (http://cassandra.apache.org/)<br>
- Hazelcast  <br>
- Apache Kafka 2.0 (https://kafka.apache.org/quickstart)<br>
<a name="installation"></a>
## Installation Components<br>
- Yantriks Availability : microservice-availability-app.jar  <br>
- Yantriks Eligibility and Master Data : microservices-common-app.jar :   <br>
- Yantriks Streamer : microservices-streamer-app.jar  <br>
- Yantriks Inventory Lite : microservices-inventory-lite-app.jar <br>

**Note** : The app jars will have a version number in the jar name as well. Eg : Yantriks Availability Service App Jar will be named as  microservice-availability-3.0-app.jar.<br>
<a name="installationandsetup"></a>
# Installation and Setup
<a name="overview"></a>
## Overview
Yantriks Availability Service as a whole has the following microservices : <br>
- *Yantriks Availability Service* (API Documentation : https://swagger.cicd.yantriks.in/Availability-Service/dev/): The core service which is responsible for availability, capacity booking and reservation APIs.<br>

- *Yantriks Eligibility & Master Data* (API Documentation : https://swagger.cicd.yantriks.in/common/1.3/): We also refer to this as Common Services. The common services APIs are responsible for setup of master and eligibility data across Selling Channels/ FulfillmentTypes/ Locations & Products.

- *Yantriks Common Service Streamer* : This is a consumer module which is responsible for propagation of updates of master data to Yantriks Availability Service.

- *Yantriks Inventory Lite* (API Documentation : https://swagger.cicd.yantriks.in/inventory-lite/1.1): The Inventory Services APIs are responsible for consumption and reconciliation of supply feeds and propagation of reconciled feeds to Yantriks Availability Service and any other external systems.
<a name="pre-requisites"></a>
# Pre-Requisites
<a name="creation"></a>
## Creation of Kafka Topics
The following kafka topics need to be created for updates between different microservices and are also required for internal Cache propagation of messages in a clustered environment.
<a name="kafkatopics"></a>
## Kafka Topics
<a name="master"></a>
### *Topics for master and eligibility data key updates*
On every change in master-data configuration the Common Services layer publishes the respective keys of the given entity record which would have undergone change. Below are the list of topics required for the same.

- *yso-fulfillment-type-updates-key* : On any change to configuration in fulfillment types, we publish the respective key to this topic.

- *yso-organization-updates-key* : On any change in configuration of Organization/ Selling channel the respective keys are published.

- *yso-location-fulfillment-type-updates-key* : On any change in configurations at Location or Location-FulfillmentType level, the respective keys are published.

- *yso-product-fulfillment-type-updates-key* : On any change in configurations at Product or Product-FulfillmentType level, the respective keys are published.

- *yso-product-location-fulfillment-type-updates-key* : On any change in configurations at Product-Location-Fulfillment Type Level, the respective keys are published.

On consumption of the messages published in the above keys, the streamer app pulls the changes for the keys and publishes it to different entity topics as mentioned in the following section [Entity Topics for consuming updates in master and eligibility data](#consuming). The messages in these topics will then be consumed by Yantriks Availability Services in order to update it’s master data caches.
<a name="consuming"></a>
*Entity Topics for consuming updates of master and eligibility data*
- yso-organization : On modification of organization or selling channel configuration, the latest snapshot of the same is published to this topic.

- yso-fulfillment-type : On modification of fulfillment type configuration, the latest snapshot of the same is published to this topic.

- yso-location-fulfillment-type : On modification of location or location level fulfillment type  configurations, the latest snapshot of the same is published to this topic.

- yso-product-fulfillment-type :  On modification of product or product level fulfillment type  configurations, the latest snapshot of the same is published to this topic.

- yso-product-location-fulfillment-type : On modification of product-location level fulfillment type configurations, the latest snapshot is published to this topic.

The application server has an embedded layer which keeps listening to these topics and propagates the caches within the cluster (in case of in memory caches through the [Internal Cache Update Topics](#cache)) or updates the Hazelcast cache layer.
<a name="supply"></a>
#### *Topics for consuming Supply/ Supply Configuration from Inventory Lite/ 360*
- *supply-type-definition-change* : Inventory Lite application defines configuration for SupplyTypes and whether a given supply type is sellable or not. These updates are published to the given topic, which the availability service reads and populates it in it’s in-memory cache.

- *aggregated-inventory* : For any change in supply, the Inventory Lite/ 360 system publishes continuous updates to the given topic. The Availability service application will be consuming these feeds to populate it’s Inventory Cache.
<a name="cache"></a>
#### *Internal Cache Update Topics for Yantriks Availability Services*
The Availability Service application has some set of configurations which it stores in internal in-memory caches. The cache data  . To view the list of Internal Caches, please refer to Yantriks Availability Service Application Configuration document.
To propagate this data across the cluster, we need the below set of topics.
- *yas-fulfillment-type-internal-cache-updates* :  This topic is required for propagation of fulfillment type configuration across the application cluster.

- *yas-horizon-config-internal-cache-updates* : This topic is required for propagation of horizon configuration across the application cluster. Horizon Configuration contains information related to ATP, Supply and capacity Horizons for a given Org/ Selling Channel / Fulfillment type combination.

- *yas-location-internal-cache-updates* : This topic is required for propagation of horizon configuration across the application cluster.

- *yas-selling-channel-internal-cache-updates* : This topic is required for propagation of Selling Channel configuration across the application cluster.

- *yas-supply-config-internal-cache-updates* : This topic is required for propagation of Supply type configuration across the application cluster. The Supply Type configuration contains the list of supply types and defines whether they are sellable or not.

- *yas-publish-atp-config-internal-cache-updates* : This topic is required for propagation of Publish ATP configuration details across the application cluster. The Publish ATP Configuration entity contains information related to thresholds for an ATP at Selling Channel & Fulfillment type level. Based on these thresholds, we determine the alert Status for an ATP and publish ATP to the topic mentioned in section below.

<a name="product"></a>
#### Topic for publishing product ATP

- yas-product-atp-updates : On enabling ATP publish (Refer to swagger https://swagger.cicd.yantriks.in/Availability-Service/dev/ Section 4.1), and setting up ATP publish Configurations, the ATP for a SellingChannel Product to this topic.
For more information on the same, please refer to *Yantriks Availability Service Application Configuration* document.
<a name="cassandra"></a>
#### Cassandra Setup
As part of a release of Yantriks Availability Services and for it to work in conjunction with Common Services. We have three sets of schema files which we will be providing.
- common-schema.cql
- availability-schema.cql
- inventory-schema.cql

In order to setup Cassandra please refer to  (http://cassandra.apache.org/).

Assuming Cassandra is setup in your environment.
Execute the command :
CREATE KEYSPACE yantriks WITH replication = {'class':'SimpleStrategy', 'replication_factor' : 1};

After creating keyspaces, execute the above cql files.

**Note :** The replication strategy and the replication mode mentioned in the document is for a development/ QA environment setup.<br>
For a production/ performance environment, the replication factor/ replication strategy / read-write consistency will change.

<a name="app"></a>
#### Application Initialization & Startup on VM

**Steps to start the Commons Application:**<br>
1) Create the tables by running the given cql script using the command below.
SOURCE ‘{File path}/common-schema.cql’;<br>
2) Start the jar by  running below command Java -jar microservice-common-app.jar<br>
3) Your application will be started locally on port 8094, use below swagger url to access the apis.
http://localhost:8094/swagger-ui.html

**Steps to start the Availability Application:**<br>
1) Create the tables by running the given cql script using the command below.<br>
SOURCE ‘{File path}/availability-schema.cql’;<br>
2) Start the jar by running below command<br>
Java -jar microservice-availability-app.jar<br>
3) Your application will be started locally on port 8080, use below swagger url to access the apis.<br>
4) http://localhost:8080/swagger-ui.html<br>

**Note :**
- The availability application by default starts in an embedded mode. The default profile for the same is kafka-updates.
- In order to start the availability application in client-server mode the same jar needs to be started with the below two profiles :
    - hazelcast-client : This is the client profile which will not have hazelcast server level changes.<br>
    - hazelcast-cluster,kafka-updates : This is the server profile. The server module is responsible for all computation logic in addition for consuming inventory supply and supply configuration data.

**Steps to start the Inventory lite Application:**<br>

1) Create the tables by running the given cql script using the command below.<br>
    SOURCE ‘{File path}/inventory-schema.cql’;<br>
2) Start the jar by running below command<br>
    Java -jar microservice-inventory-app.jar<br>
3) Your application will be started locally on port 8098, use below swagger url to access the apis.<br>
4) http://localhost:8098/swagger-ui.html

**Steps to start the Common Services Streamer Application:**

1) Start the kafka streamer which streams the data from common to Availability Service.<br>
Java -jar microservice-streamer-app.jar<br>
2) Your application will be started locally on port 8078, use below swagger url to access the apis.
<a name="cloud"></a>
#### Application Initialization on Cloud (GCP K8S Cluster)

Below steps talks about running our application in GCP Kubernetes, it would be helpful if the reader has an idea about GKE(Google Kubernetes Engine). For more info, please go through the link https://cloud.google.com/kubernetes-engine/docs/tutorials/

1) Creating a Kubernetes cluster in the GCP with appropriate configuration required, like if we need to create a private cluster for production purpose etc.<br>
2) Creating a nodepool with appropriate sizing in order to deploy our application as a PODs(Containerized apps).<br>
3) As a part of application build, we are going to create images and push the same into our private container registry which holds all versions of our application docker images.<br>
4) We are maintaining templatized YAML files for deploying the application. For example, to deploy an application we may need to configure a set of templatized YAML files(ConfigMap, Service, Statefulset) to the appropriate numbers needed.<br>
   a) **ConfigMap**: This file holds the application properties needed to be configured as a part of the application. And other configuration files which are needed for the application deployment(hazelcast.xml, telegraf.conf) if there are any.<br>
   b) **Service**: This file will be used to expose our application as a SERVICE to outside the container.<br>
   c) **Statefulset**:<br>
        i) Docker image version which needs to be used for the deployment.<br>
       ii) Pod sizing configuration(CPU, Memory)<br>
      iii) Updating configuration like JMX, JAVA_OPTS, Heap size.<br>
      iv) Nodepool name changes to deploy into the specific nodepool.<br>
5) Once the nodepool is created, then connect to the cluster with command provided at the GCP console and apply these files with kubectl command to create the PODs.


Note: To integrate influx, we need to start the influxDB in the kubernetes. For that, we need to come up with YAML files with appropriate configuration and deploy the influxDB files. Enable the metrics property to publish and update the URL in both the **ConfigMap**(application and telegraf) YAML files.
<a name="app-properties"></a>
### Application Properties
<a name="over-view"></a>
#### Overview

The section below walks through all the configuration properties required for startup of Yantriks Availability Services.
<a name="level"></a>
#### Logging level properties

In order to change log levels we use the logging level properties of default Spring Boot application.

In order to change log levels we use the logging level properties of default Spring Boot application.

```
logging:
       level.com.yantriks.ypfp: WARN
       level.com.yantriks.yas: WARN
       level.io.micrometer.influx: WARN

```
This changes the log level across various packages. In case we have enabled metrics publish using micrometer and influx, we need to setup this property as WARN, in order to not fill our logs with updates with each data which is published to influx.
<a name="server"></a>
#### Server and Spring Boot Profile properties
```
server:
       port: 8080

```
The property defines the port which the application needs to start at.

```
spring:
      profiles:
        active: "kafka-updates,hazelcast-cluster,hazelcast-client"
      output:
        ansi:
          enabled: ALWAYS
      main:
        banner-mode: log
      webflux:
        dateFormat: "yyyy-MM-dd"

```
- spring.profiles.active : This defines which mode the application needs to be started in.<br>
   - kafka-updates : the application starts in embedded mode with api and hz server part of the same application.<br>
   - hazelcast-client : The application starts in API server mode and connects to the hazelcast server (hazelcast-cluster) profile. All APIs will be exposed on this application.<br>
   - kafka-updates,hazelcast-cluster : The application starts with hazelcast server mode and also acts as a consumer of supplies. No, APIs will be exposed on this port.<br>
- spring.output.ansi.enabled : Colour coding of console logging. (Optional)<br>
- spring.output.main.banner-mode: Enables logging of the application banner. (Optional)<br>
- spring.webflux.dateFormat : Set it up as yyyy-MM-dd

<a name="mgmt"></a>
#### Management & Metrics Publish properties
```
management:
       metrics:
         distribution:
           percentiles-histogram:
             "[http.server.requests]": true
           percentiles:
             "[http.server.requests]": 0.95, 0.98, 0.99
         web:
           server:
             auto-time-requests: true
         export:
           influx:
             uri: <Influx DB URL>
             db: <Influx DB Schema>
             auto-create-db: true
             username: admin
             password: admin
             enabled: true
             step: 30s
       endpoints:
         web:
           exposure:
             include: "*"
       endpoint:
         health:
           show-details: always


```

Internally, we publish our metrics to influx so as to monitor our application health. The above properties are defined for the same. In case we need to integrate metrics through influx the following two properties need to be changed <br>
+ management.export.influx.db : Provide the influx DB schema.
+ Management.export.influx.uri : Provide the influx DB URI. Eg : http://localhost:8086

<a name="cassandrasetup"></a>
  ### Cassandra properties
  ```
  cassandra:
         contactPoints: 10.0.0.11
         port: 9042
         keyspace: yantriks
         consistency: LOCAL_QUORUM
         username: cassandra
         password: cassandra
         cluster-name:
         local:
           host:
             connection:
               max:
                 requests: 32768
             connections:
               core: 4
               max: 10
         remote:
           host:
             connection:
               max:
                 requests: 2000
             connections:
               core: 2
               max: 4

```
  <br>
  As part of our product offerings, we use cassandra as our underlying database.
Some of the key properties which will definitely require a change in case of application deployment<br>

  + cassandra.contact.points : Needs to be updated with internal IP of one or multiple cassandra seed IPs. Need to be comma-separated if we are providing multiple IPs.<br>
  + cassandra.port : Default port of cassandra is 9042. Provide a port at which your Cassandra Instance is setup.<br>
  + cassandra.username: Username for connecting to cassandra.<br>
  + cassandra.password :  Password for connecting to cassandra.<br>
<a name="kafka"></a>
### Kafka Properties<br>
```
kafka:
      consumer:
        commit.interval: 5s
        spring.json.trusted.packages: "*"
        spring.json.use.type.headers: false
        retry.limit: 3
        retry.backoff.initial: 10s
        retry.backoff.max: 1m
        bootstrap.servers: localhost:9092
        group.id: availability-service-application
        auto.offset.reset: latest
        yas:
          supply:
            enabled: true
            transaction-id-seperator: ","
      producer:
        bootstrap.servers: localhost:9092
        max.in.flight.requests.per.connection: 100
        acks: all

```
Kafka is extensively used by our application for internal and external communications.
Some of the key properties which will require a change :
+ kafka.consumer.group.id : The group id defined here is for the inventory supply consumers soa as to load balance between multiple consumers of supply. <br>
+ kafka.consumer.yas.supply.enabled : Set this property as true to enable consumption of supplies <br>
<a name="consumer"></a>
+ kafka.consumer.yas.supply.transaction-id-separator : <br>
The default value of this property: “,”. Inventory Lite/ 360 application also sends us updates related to consumption of demand. The Feed published to the topic contains an element transactionId. The transaction Id field has to be a String in a particular format of OrderId, LineId, StoreId, LocationType in sequence separated by a delimiter. This property defines the separator.<br>
+ kafka.consumer.bootstrap.servers & kafka.producer.bootstrap.servers : Define comma separated list of kafka servers which need to be connected to. Consumer servers are for consuming incoming messages, eg: Common Services Configuration Data, Inventory Supplies. Producer servers are for publishing messages, eg : ATP Publish.<br>
<a name="service"></a>
### Availability Service Properties<br>
```
yas:
      cache:
        reservation:
          demand-type: "RELEASE"
        kafka:
          serializer:
            buffer-size: 1024
          updates:
            fulfillment-type-cache-updates-enabled: true
            horizon-config-cache-updates-enabled: true
            location-cache-updates-enabled: true
            selling-channel-cache-updates-enabled: true
            supply-config-cache-updates-enabled: true
            atp-publish-config-cache-updates-enabled: true
        location:
          services:
            cache:
              location-services-server-uri: http://localhost:8094
        product:
          services:
            cache:
              product-services-server-uri: http://localhost:8094
              inventory-services-server-uri: http://localhost:8096
        kryo:
          pool-size: 20
        integration:
          selling-channel-consumer-enabled: true
          fulfillment-type-consumer-enabled: true
          location-fulfillment-type-consumer-enabled: true
          product-fulfillment-type-consumer-enabled: true
          product-location-fulfillment-type-consumer-enabled: true
          supply-config-consumer-enabled: true
          default-update-user: integration
      availability:
        config:
          services:
            cache:
              supply-config-services-server-uri: http://localhost:8096
              zipCode-services-server-uri: http://localhost:8094
              fulfillment-type-services-server-uri: http://localhost:8094
              selling-channel-services-server-uri: http://localhost:8094
      publish:
        atp-publish-enabled: false


```
Please find below the description of topics defined :<br>
+ yas.cache.reservation.demand-type : Based on the demand matrix, configured in the application, (Refer to *Yantriks Availability Service Application Configuration* document), provide the demand-type you want to trigger consumption of demand. If the demand type property is not defined, demands in the application will not be consumed. <br>
+ yas.cache.updates : For a client-server model or an embedded clustered environment setup the properties below need to be set to true. All these are maintained as in memory caches in our system and enabling these properties allow cache propagation of the entities. Also refer to [Internal Cache Update Topics for Yantriks Availability Services]() section.<br>
  - fulfillment-type-cache-updates-enabled: true<br>
  - horizon-config-cache-updates-enabled: true<br>
  - location-cache-updates-enabled: true<br>
  - selling-channel-cache-updates-enabled: true<br>
  - supply-config-cache-updates-enabled: true<br>
  - atp-publish-config-cache-updates-enabled: true<br>

+ yas.cache.location.services.cache<br>
  - location-services-server-uri : Need to specify the URL of Common Services for fetching location information. By default Common Services starts at port 8094.<br>
+ yas.cache.product.services.cache <br>
 - product-services-server-uri : Need to specify the URL of Common Services for fetching product information. By default Common Services starts at port 8094.<br>
 - inventory-services-server-uri : Need to specify the URL of Inventory Lite/360 Services for fetching supply information. By default Inventory Service starts at port 8096.<br>
+yas.availability.config.services.cache<br>
 - supply-config-services-server-uri: Need to specify the URL of Inventory Lite/360 Services for fetching supply configuration information. By default Inventory Service starts at port 8096.<br>
 - fulfillment-type-services-server-uri: Need to specify the URL of Common Services for fetching fulfillment type information. By default Common Services starts at port 8094.<br>
 - Selling-channel-services-server-uri: Need to specify the URL of Common Services for fetching org/ selling channel information. By default Common Services starts at port 8094.<br>
+ yas.cache.kafka.serializer.buffer-size : 1024<br>
+ yas.cache.kryo.pool-size : 20<br>
+ yas.cache.integration : The below properties are required to be enabled for consumption of configuration data from common services and Inventory Lite/ 360. If set as false the configuration data will not be consumed. Please refer to the section Kafka Topics for more info.<br>
 - selling-channel-consumer-enabled: true<br>
 -   fulfillment-type-consumer-enabled: true<br>
  - location-fulfillment-type-consumer-enabled: true<br>
  - product-fulfillment-type-consumer-enabled: true<br>
  -  product-location-fulfillment-type-consumer-enabled: true<br>
  - supply-config-consumer-enabled: true<br>
  -  default-update-user: integration<br>
+ yas.cache.publish.atp-publish-enabled : If set as true, the application publishes ATP at Selling Channel/ Product level for every supply/ demand update based on ATP Publish configurations. (Refer to *Yantriks Availability Service Application Configuration* document)
<a name="kubernetes"></a>
## Kubernetes discovery and Hazelcast Properties

```
microservice:
      availability:
        hazelcast:
          kubernetes:
            discovery: true
            namespace: internal-dev
            serviceName: availability-hazelcast
            serviceDnsTimeout: 5
            apiToken: <Please provide token here>

```
In the case of Kubernetes clustered deployment, we use the kubernetes discovery mechanism  for discovering Hazelcast nodes.

microservice.availability.hazelcast.kubernetes.discovery: It is a boolean property which we need to specify in order to turn on or off kubernetes discovery.

microservice.availability.hazelcast.kubernetes.namespace: Kubernetes namespace where hazelcast pods are running.

Microservice.availability.hazelcast.kubernetes.servicename: Service name is used to scan all the pods which are associated with that and to form the cluster.
microservice.availability.hazelcast.kubernetes.apiToken: API token to the Kubernetes API.
For more info: https://github.com/hazelcast/hazelcast-kubernetes#kubernetes-api

In case of a VM based deployment or when each Hazelcast Server instance is assigned an internal static IP, the following property can be used :
microservice.availability.hazelcast.members : List of comma separated internal IPs on which the Hazelcast Server (In case of client-server model) or the Embedded server nodes are running.

## Foundation Properties

- ypfp.consume.region : Specify the region name in which the deployment is done. Eg : US-Central, East etc.
- ypfp.customer-name : Specify the customer name for which the deployment is done.
- ypfp.topic-name-delimiter : As mentioned in the section [Kafka Topics](), we have pre-defined topic names, with a delimiter of hyphen. There are cases where customers may choose to override the delimiter. The following property is used to override the same. The default value is **"-"  (Hyphen)**
<a name="kafkatopic"></a>
## Kafka Topic Name Overrides

There are cases where customers may want to override the topic names as our internal topic names do not follow their standard of defining topics.

While creating topics all our topics except the one for internal [cache propagation](#)  are created by a prefix Customer Name and a suffix Region with a delimiter. The Customer name, region and delimiter properties are defined in the section [Foundation Properties]().

In order to override these topics the format is :
override.topic-name.${customer_name}-yso-organization-${region}: test_organization

In case of internal topics for cache propagation the property format is :
override.topic-name.yas-location-internal-cache-updates : test_location_internal
<a name="samples"></a>
# Samples
<a name="samplemsg"></a>
### Sample Message Published to aggregated-inventory topic

Below is a sample message published to aggregated-inventory topic for supply and demand consumption.
For more details refer to the output of inventory search API ([search/{orgId}/{productId}/{uom}]()) under https://swagger.cicd.yantriks.in/inventory-lite/1.1/
```
{
  "futureInventory": [
    {
      "locationType": "STORE",
      "locations": [
        {
          "locationId": "S001",
          "supplyTypes": [
            {
              "quantity": 1,
              "segment": "segment1",
              "supplyDate": "2018-06-21",
              "supplyType": "WIP"
            }
          ]
        }
      ]
    }
  ],
  "orgId": "ORG001",
  "physicalInventory": [
    {
      "locationType": "STORE",
      "locations": [
        {
          "locationId": "S001",
          "supplyTypes": [
            {
              "quantity": 1,
              "segment": "segment1",
              "supplyType": "Onhand"
            }
          ]
        }
      ]
    }
  ],
  "shipment":
  {
    "shippedQty":1.0,
    "transactionId":"ORDER_1,LINE_1,S001,STORE"
  },
  "productId": "1266009",
  "uom": "EACH"
}

```
In the above sample we have the shipment element with a transaction Id. The format of the transaction Id is defined in this [section]().
<a name="sampleapp"></a>
## Sample application.yaml

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: availability-hazelcast-config
  namespace: internal-dev
data:
  application.yml: |-
     logging:
       level.com.yantriks.ypfp: WARN
       level.com.yantriks.yas: WARN
       level.io.micrometer.influx: WARN
       level.yas-reservation: WARN
       level.yas-availability: WARN
       group:
         yas-reservation: com.yantriks.ypfp.availability.services.calculation.service.impl.ProductAvailabilityCalculatorImpl, com.yantriks.ypfp.availability.services.calculation.service.impl.CachingProductAggregatorServiceImpl
         yas-availability: com.yantriks.ypfp.availability.services.reservation.services.cache.hazelcast.service.ProductReservationCachedServiceImpl, com.yantriks.ypfp.availability.services.reservation.services.api.controller.ReservationController
     server:
       port: 8080
     spring:
       profiles:
         active: "kafka-updates,hazelcast-cluster"
       output:
         ansi:
           enabled: ALWAYS
       main:
         banner-mode: log
       webflux:
         dateFormat: "yyyy-MM-dd"
     management:
       metrics:
         distribution:
           percentiles-histogram:
             "[http.server.requests]": true
           percentiles:
             "[http.server.requests]": 0.95, 0.98, 0.99
         web:
           server:
             auto-time-requests: true
         export:
           influx:
             uri: localhost:8086
             db: yas_hazelcast_metrics
             auto-create-db: true
             username: admin
             password: admin
             enabled: true
             step: 30s
       endpoints:
         web:
           exposure:
             include: "*"
       endpoint:
         health:
           show-details: always
     yas:
       cache:
         reservation:
           demand-type: "RELEASE"
         kafka:
           serializer:
             buffer-size: 1024
           updates:
             fulfillment-type-cache-updates-enabled: true
             horizon-config-cache-updates-enabled: true
             location-cache-updates-enabled: true
             selling-channel-cache-updates-enabled: true
             supply-config-cache-updates-enabled: true
             atp-publish-config-cache-updates-enabled: true
         location:
           services:
             cache:
               location-services-server-uri: http://localhost:8094
         product:
           services:
             cache:
               product-services-server-uri: http://localhost:8094
               inventory-services-server-uri: http://localhost:8096
         kryo:
           pool-size: 20
         integration:
           selling-channel-consumer-enabled: true
           fulfillment-type-consumer-enabled: true
           location-fulfillment-type-consumer-enabled: true
           product-fulfillment-type-consumer-enabled: true
           product-location-fulfillment-type-consumer-enabled: true
           supply-config-consumer-enabled: true
           default-update-user: integration
       availability:
         config:
           services:
             cache:
               supply-config-services-server-uri: http://localhost:8096
               zipCode-services-server-uri: http://localhost:8094
               fulfillment-type-services-server-uri: http://localhost:8094
               selling-channel-services-server-uri: http://localhost:8094
       publish:
         atp-publish-enabled: false
     cassandra:
       contactPoints: 10.0.0.11
       port: 9042
       keyspace: yantriks_dev
       consistency: LOCAL_QUORUM
       username: cassandra
       password: cassandra
       cluster-name:
       local:
         host:
           connection:
             max:
               requests: 32768
           connections:
             core: 4
             max: 10
       remote:
         host:
           connection:
             max:
               requests: 2000
           connections:
             core: 2
             max: 4
     microservice:
       availability:
         hazelcast:
           kubernetes:
             discovery: true
             namespace: internal-dev
             serviceName: availability-hazelcast
             serviceDnsTimeout: 5
             apiToken: <Please provide token here>
           executorConfigs:
             - name: "hz:yas:executor:product-atp-calculation"
               statisticsEnabled: false
     kafka:
       consumer:
         commit.interval: 5s
         spring.json.trusted.packages: "*"
         spring.json.use.type.headers: false
         retry.limit: 3
         retry.backoff.initial: 10s
         retry.backoff.max: 1m
         bootstrap.servers: locahost:9092

```
