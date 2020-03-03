# YANTRIKS AVAILABILITY 3.0
## PRODUCT CONCEPT
<br>
<br>

![baskett](images/2020/03/ecommerce.png)


# Introduction
This document provides a conceptual understanding of the Yantriks Availability (YAS). The document is organized in a way that it first throws light on the market challenges and drivers for YAS and how the solution has addressed it.<br>
The product concept also provides an overview of the architecture of YAS and deep dives on the features. The document however does not provide details on design, configuration and implementation of the listed features.

# Business Environment
Customer demands are ever increasing along with rapid changes to business drivers. Factors like introductions of new products or product lines, new customer segments, addition of new business channels not only influence but requires businesses to change direction and innovate to keep customers happy as well as keep operations costs low.<br>
With rapid increase in E-commerce adoption, customer expectations for service excellence at low cost is at the peak. Businesses are continuously striving to keep themselves relevant and ahead of the curve.  This has put extreme pressure on the Business processes and IT systems. IT systems have to adapt to fast changing requirements without having to spend mega dollars. IT Managers are finding it challenging to have the system flexible, nibble as well as scalable to ensure the smooth running of businesses.

# Present Challenges
Over the last decades, Retail businesses have been working to provide a seamless experience to customers across all the touch points. This means that  CIOs have to ensure information is readily available across the channels and IT system are geared up to not only process it but provide actional inputs.  CEOs / CIOs are struggling to answer similar questions: Will my IT system support the peak day business? Can I guarantee my customer that his/her order will be processed on a Sale day?  Can I guarantee my customer that what he sees in inventory is indeed correct and he can confidently place the order?<br>
These questions are arising not because of the poor and non-functional IT system and infrastructure but because the systems and infrastructure were designed during a different era and made to work in the internet era.<br>
Additionally new business models like marketplaces have emerged, which requires business processes and systems to be more open and ready for a network of information exchange. <br>
These challenges are significant enough to impact the bottom line for businesses hence getting due attention from businesses.

# Technology Obsolescence
As discussed earlier, the rapid adoption of multi channels has led to significant increase in inventory and order related transactions that current system has to deal with. Also the system needs to become open to exchange information from marketplaces. <br>
This requires that the system needs to handle “internet scale” of transaction as well as become flexible to integrate with external systems.  As enterprise systems scramble to optimize application design solutions to the scale of operations, they are inhibited by the fundamental architecture that was built for a different era.  Let's take an example of an existing Retail echo system:

![](images/2020/03/flowchart.jpg)<br>

Enterprise systems usually get an inventory feed from POS / WMS. The inventory feed is assimilated into the enterprise inventory system, which may be stand-alone or part of the enterprise Order Management system. Such inventory systems would then typically provide a periodic feed of availability of inventory data to order capture system across channels.The feed may be continuous or threshold based or periodic. The feed is typically assimilated by the channels.The replica / cache of inventory feed thus maintained by the website is thus used by the actual website during browsing and shopping for the purposes of inventory check. Often, if inventory levels are low, the website may have special intelligence to make real-time calls to backend inventory systems to avoid oversells. <br>

This typical flow is with the prominent shortcomings of
-	Stale inventory resulting in oversells or undersells,
-	Vague promise dates
-	Increased safety stock that is often taken as a countermeasure
-	Scalability of systems that are designed on underlying “legacy platforms” such as RDBMS.

These shortcomings directly impact the Order profitability and results in lack of customer confidence on the Retailer.<br>
The existing IT ecosystem also struggles to provide reliable service to multiple business consumers like Marketplaces, POS, Mobile etc and carries the risk of becoming a single point of failure. This makes the business vulnerable to downtime especially during peak business season causing decisive customer dissatisfaction and loss of business.
It is evident that current IT ecosystem is struggling to cater to scalability demand and causing Retailers to look for alternatives. Retailers are also taking a cautious approach to change critical components in the ecosystem instead of big-bang approach. This approach is enabling them to choose best of breed components designed and developed to solve specific problems of the ecosystem.

# Yantriks Availability Solution
Yantriks Availability is  a comprehensive high ROI state of the art solution. The solution is visualized and designed as a Central Hub for all Inventory operations. All the channels will either query or update the Central Hub for inventory. The solution is designed utilizing latest technological components providing Retailers advantages in terms of scalability, reliability and speed of business.<br>

YIA REST APIs are served out of a Spring Boot application, hooked up to an in-memory grid computing technology, backed by Cassandra, acting as the big data persistence layer.

&emsp;&emsp;&ensp; ![](images/2020/03/diagram.png)

YA delivers following Technological benefits to businesses:<br>
- Distributed architecture provides higher degree of availability
- Scalability as per the load, application can get additional nodes or shutdown under-utilized ones
- Unmatched Response Time, utilizing the hyper quick in-memory grid along with persistence of big data

The business benefits that will be achieved with YA are :
-	Richer Product list page, providing options like Ships within, Get it by, Pick up options, urgency messages etc due to superior technology making the deeper queries quick and less costly.
-	Avoids oversell, since it is the central hub for all channels there is no possibility of stale inventory or out of sync inventory
-	Reduction in safety stock, as more accurate inventory picture makes the utilization better.

YA has hooks to integrate with any enterprise OMS system. The Yantriks API can directly integrated into order channel UIs due to the single-digit millisecond response times for “available to promise” as well as “reservation” APIs even at high scale during peak periods.  Yantriks Inventory can also stream out threshold based network level as well as location level available to promise information to online search index, allowing Retailers to have a more near-real time “in-stock” or “out-of-stock” information.<br>
Once fully implemented, Yantriks would become the centralized inventory system, taking away the heavy workload from legacy OMS / inventory platforms, and allowing the existing enterprise systems valuable breathing room.<br>
It is envisaged that Yantriks Availability  will be utilized for inventory instead of the OMS and all inventory operations will be done directory on YA . There can be phased approaches for implementation where only for a specific types of order, Yantriks Availability is utilized and gradually it can be implemented for all types of orders. <br>
To illustrate the above statement, let us study the flow for two different scenarios. First scenario is Buy Online Pickup At Store. In this scenario, customers will pick up the Order from the chosen Store, hence inventory check and reservation has to be done at Store level. Customer will be shown nearby stores where inventory is available, once he/she chooses the store and adds item to Cart, a temporary reservation has to be done to ensure inventory is reserved till the shopping is completed. Once Customer confirms the order, the reservation is confirmed.

&emsp;&emsp;&ensp; ![](images/2020/03/flowchart1.png)

In the second scenario, the customer places the order and wants to get it shipped. Here  the temporary reservation is done without the Store information as it is unknown at this point from where the order will be shipped. Once the order is confirmed, order will get scheduled by Order Management system and Store / DC location will be determined. OMS will then modify the temporary reservation to confirm.<br>
As you can see from the above scenarios, OM and Yantriks Availability will serve different purposes. While OM will be the Order repository, Yantriks Availability will be the main system for Inventory records and  soft as well as hard allocation of it against orders. The two scenarios depicts how Yantriks availability will interact with Web Channel and OM system based on when Inventory information is required and when inventory allocation is to be made.<br>
It is important to note that Yantriks Availability will need some master data like Locations, Items, Business rules etc. Typically this is provided as interfaces from OMS system. There will also be ongoing feed for Supply from external system as and when new inventory is received across the Stores / Warehouses.
![](/images/2020/03/flowchart2.png)

## Architecture
Yantriks  Availability architecture is enabled using following components<br>

**Hazelcast** : It is a java based open source in-memory data grid. It evenly distributes data among the nodes of a cluster, enabling horizontal scaling of processing and available storage. Hazelcast provide central, predictable scaling of applications through in-memory access to frequently used data and across an elastically scalable data grid.<br>

**Apache Cassandra**:  Cassandra is an open source distributed NoSql database management system designed to handle large amounts of data. It offers robust support for clusters spanning multiple data centers, with asynchronous masterless replication allowing low latency operations.  Cassandra is used as data backup for Yantriks Inventory Cache.<br>

 A Pictorial depiction of the entire setup will be as below:

&emsp;&emsp;&ensp; ![ ](images/2020/03/diagram1.png)

Conceptually Data is first written to in-memory database i.e. Hazelcast. Later the data from moved from Hazelcast to Cassandra for persistence. Similarly Data is first retrieved from in-memory database and if not found then it is looked up in persistence layer i.e. Cassandra.

Typically in-memory database optimizes what data is kept in the cache. It removes data (called eviction) from time to time based on multiple inbuilt rules.
The architecture philosophy of YA is to achieve:
- High Availability and Fault tolerance
- Elastic Scalability
- High Performance

## Feature Capability
Yantriks  Availability features capabilities can be classified in three major areas : Available to Promise, Alternate Store Pickup and Reservation. <br>
### Available to Promise.
Once the Inventory picture is available, Yantriks Inventory is ready to provide ATP information. Yantriks ATP is based on Supply minus Demand minus Safety Factor. Only those supplies that are necessary for ATP calculation must be fed to Yantriks Inventory. Demands are internally managed within Yantriks. Enterprise OMS notifies Yantriks about Order life cycle changes, and Yantriks moves demand within itself. With any supply and demand change, Yantriks recomputes enterprise level availability as well as store level availability.Yantriks compute ATP after applying various rules like Exclusions, Node Control, Store capacity etc.<br>
### Alternate Store Pickup
When the website shows the user the nearest pickup option, the shopper may prefer a different location. Yantriks can provide a list of alternative pickup locations having availability for the item within a pre-configured radius. The choices are ranked based on distance from the shopper’s postal code. Available to Promise information incorporates safety factor by channel. <br>
### Reservations
Once the shopper adds the product to stock, website would want to reserve the item to ensure that nobody else gets that specific item. Often the reservation is done based on a threshold. If the product has ample stock, then website takes a chance, allowing the order to flow through without reservation. This leaves a small room for oversell if there was a sudden surge of demand. But by and large this mechanism works for normal scenario. However, during peak loads, promotions, new product launches, this mechanism breaks down. Yantriks provides rapid response Reservation API that can be invoked for every add to cart regardless of threshold, thus plugging the oversell leak point. Yantriks reservation can be invoked at the enterprise level (for shipping from a warehouse) or at a specific store for in-store pickup and ship from store.<br>
### Master and Business Rules data requirements
To support various uses cases for Availability, YA needs to understand various master and business rules data. These data requirements can be further classified as Master data, Eligibility and ATP Matrix. <br>
### ATP Matrix
The ATP Matrix is used to calculate ATP for any product. ATP Matrix is configuration of various supply types and demand types that needs to be considered to compute correct ATP number. In addition to Supply Types and Demand Types, ATP Matrix will need to define Selling channel and Transaction type. This will ensure that implementations can have different ATP number for different Selling Channels. Even within the same selling channel,ATP can be different based on Transaction Type. For e.g. ATP can be more relaxed when calling transaction is PDP / PLP but it will need to be more specific when the transaction is Schedule Order.<br>
### Eligibility
Eligibility is defined primarily for Product and Location. Eligibility helps decide whether a product or location or combination is eligible for Promise or not. Here is the list of eligibility that can be defined:
- Selling to Location and Product as well as combination of both
- Fulfilment Type to Location and Product as well as combination of both
- Fulfillment Service to Location and Product as well as combination of both
- Fulfilment Type to Location Type
- Fulfilment Service to Location Type
- Product Category to Fulfilment Type
- Product Category to Fulfilment Service

When a request for ATP is made for a product, YA first evaluates eligibility to ascertain whether the product is eligible for the Selling Channel, Fulfilment Type, Location etc. If the product is not eligible for any of the one then ATP will be returned as 0.<br>

Implementations have flexibility to decide the order in which eligibility should be evaluated. For e.g system can be configured to evaluate Location Type + Fulfilment Type before Location + Fulfilment Type. This is can be configured using Configuration hierarchy. However it is not recommended to change what is supported out of the box.<br>

### Master Data
Master data that is needed for YA is:<br>
Product, Locations, Selling Channel, Fulfilment Type and Fulfilment Service.
- **Node Control**<br>
Apart from these static data, Node Control is something that is setup as per business operations. Node Control allows you to disable a Product across network or for selected Locations. This is typically done when businesses experience inventory mismatches, they want to put the Product on hold either across network or for a location.

- **Safety Stock**<br>
Another important data element that is required is Safety Stock. Safety Stock is the qty that needs to be protected in Store for walk-in customers. Availability excluded this quantity in computation of ATP. Yantriks Availability provides capability to define safety stock at multiple levels. You can define safety stock at following levels:
   - Selling Channel / Fulfilment Type
   - Location / Fulfilment Type
   - Location Type / Fulfilment Type
   - Product / Fulfilment Type
   - Product/Location/Fulfilment Type
   - Product Category / Fulfilment Type
   -Product Class /Fulfilment Type

### Supply Feeds
   Supply or Inventory feed is important transaction data that will be cached in Yantriks Availability. YA will maintain Supply as Physical as well as by date for future incoming. YA will cache only sellable supply.<br>
   In Yantriks ecosystem, Supply feed at the aggregate level will be received from Yantriks Inventory360. The actual feed of supply - absolute and delta will be integrated with Yantriks Inventory360 and as and when there are supply changes, it will be pushed to Availability, which will then refresh its cache.<br>
### Reservations / Demand
   Reservations will be created in Yantriks Availability directly using the exposed endpoints. Reservations request will be either all lines of an order or subset of lines. Whenever there is a change in the line, OM system will send an absolute update for the specific line. Typical example of line changes are cancel, quantity modification etc. When an order line is shipped out of the location that particular update should be handled separately. It should not be managed as part of reservation update. In other words, consume demand should NOT be a direct update to Availability.
  - Confirm Shipment / Consume Reservation Update
Whenever a shipment is shipped, system needs to reduce the supply as well as remove the corresponding demand. This needs to be done a way to ensure the integrity of the ATP is maintained.
To achieve this, Yantriks Availability takes the confirm shipment update from Yantriks Inventory 360. It does not support consume demand update operation. Implementations should integrate the confirm shipment update with Inventory360 with Order No and Line details as reference. Based on this update, Inventory360 will reduce the supply as well as update the Availability to remove the Reservation for the Order No / Line details.

### Application Interplay
There are multiple applications that will be deployed in a typical implementation as Yantriks being a distributed architecture Product comprising of various microservices.<br>
Here is a block diagram depicting various microservices and how an external system like OMS will interact with each for various data elements.<br>

![](images/2020/03/flowchart3.png)

### Publish ATP
ATP will be published whenever there is a change. ATP published data can be used by external systems like WEB / Marketplaces etc.<br>
There is an ATP publish configuration where publish rules are provided. Publish rules are :<br>
Selling Channel, Transaction Type and ATP level indicator. ATP will be published only for Selling channel and Transaction type specified here. How to classify the ATP Number - Red, Yellow, Green or High, Medium, LOW along with the threshold values can also be configured. <br>
Following system property needs to be set to ensure ATP publish is enabled:

**atp.publish.enabled: true**<br>
Kafka topic name needs to be configured where ATP will be published.

**atp.publish.topic.name:**<br> YAS-PRODUCT-ATP-UPDATES

Following are the parameters that are defaulted while publishing ATP

**atp.publish.considerGtin: true**<br>
**atp.publish.considerCapacity: false**<br>
**atp.publish.quantity: 1.0**<br>
**atp.publish.segment: DEFAULT**<br>
**atp.publish.showSplits: false**<br>
