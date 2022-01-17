# Aiven Kakfa Quickstart with Python

[Apache Kafka](https://kafka.apache.org/) is a popular open-source distributed event streaming platform. It was originally developed at LinkedIn in 2010 to simplify data integration across the organization storage and processing systems. It was later open sourced in early 2011 and since then it has been one of the most successful open-source projects in history. It is adopted by more than 80% of fortune 100 companies, with some, using it to stream more than 1 trillion messages/events a day.

Apache Kafka's excellent performance is the reason why it is so popular. It is designed to deliver the same consistent performance regardless of the number of events it ingests and stores per second. It is also capable of handling these events at a very low latency of the range of milliseconds, demanded by most real-time use cases. Organizations use Apache Kafka to power many use-cases, the following are some examples:

1. **Microservices** - Apache Kafka is widely used as the backbone of communication between different microservices.  This is because it standardises how different microservices can communicate with each other and allows you to build loosely coupled microservices. 

2. **Realtime Analytics and ML** - Some use-cases require data to be processed as soon as it becomes available. For example, in online fraud detection, companies use Apache Kakfa to ingest and process online payment transactions in real-time to prevent and block any fraudulent transactions.

3. **Streaming ETL** - Using Apache Kafka, you can process and transform your data on the fly before writing it into your destination data store. Allowing you to gain insights from the data faster. BI and ML use-cases benefit from this, as data could now be made available in minutes instead of days or hours.

4. **Event Sourcing** -  More organizations have been adopting this pattern, where each object state is stored as a set of changes instead of storing the final state. This allows you to fail safely, as any object can be reconstructed from events stored in Apache Kafka. This also offers greater flexibility as any event can be stored in an event store like Apache Kafka.

Although Apache Kafka offers many benefits, it can become challenging to manage, configure and monitor in production setups. [Aiven for Apache Kafka](https://aiven.io/kafka) is a fully managed service that makes it easy to get started with Apache Kafka in the cloud of your choice. 

With a few clicks in the console, you can create an Apache Kafka cluster. From there, Aiven replaces unhealthy brokers, automatically replicates data for high availability while ensuring that it is optimally rebalanced, manages Apache ZooKeeper nodes, automatically deploys hardware patches as needed, makes important metrics visible through the console, and supports Apache Kafka version upgrades so you can take advantage of improvements to the open-source version of Apache Kafka. Most importantly, Aiven simplifies the process of monitoring your Kafka environments by enabling out of the box [Service Integrations](https://help.aiven.io/en/articles/1456441-getting-started-with-service-integrations) with [Aiven for Grafana](https://aiven.io/grafana) - a fully managed analytics and monitoring solution.

## Introduction

This guide shows you an example on how to get started with Aiven for Apache Kafka. We will build a Python producer that will run on your local computer. The producer will securely publish payment transactions to a Kafka topic that we will create.

Finally, to gain better visibility on your Aiven Kafka setup, we will publish Kafka advanced telemetry data to [Aiven for InfluxDB](https://aiven.io/influxdb)(fully managed time series database) to be visualised in Aiven for Grafana .

```bash
├──Artifacts                          <-- Directory that will hold tutorial Artifacts
│   ├── producer.py                   <-- Producer python code to generate payment transactions
└── README.md
```

## Solution Architecture

<img width="1117" alt="arch1" src="https://user-images.githubusercontent.com/31252819/149686010-048cf2c1-cd34-422f-8f02-1b2b3d24b71d.png">

Below are the high-level steps:

1. Publish payment transactions to Aiven for Apache Kafka 
   * Create Aiven for Apache Kafka Service
   * Create an Apache Kafka topic
   * Create and run a Python Producer
   * View published messages in Aiven console
2. Observability
   * Launch InfluxDB and Grafana services
   * Enable Service Integrations
   * Access the default dashboards in Grafana
   * Create a custom dashboard


## Prerequisites

1. Sign up to [Aiven](https://console.aiven.io/signup.html) and get $300 credit. If you are already using Aiven, then [Operator or Admin role](https://developer.aiven.io/docs/platform/concepts/projects_accounts_access#project-members-and-roles) access to Aiven console is required to complete this tutorial.
2. [Install Python](https://realpython.com/installing-python/) 3.8.2 or later on your local machine.

## Estimated Costs

The costs of the tutorial will depend on how long it will take to complete the tutorial. The tutorial should take 1-2 hours to complete.

Below are the expected charges based 2 hours completion.

|    Service   | Hourly price |  Total      |
| -------------| -------------|---------    |
|    Kafka     |    $0.274    | $0.548      |
|   InfluxDB   |    $0.082    | $0.164      |
|   Grafana    |    $0.048    | $0.096      |
|  **Total**   |    $0.404    | **$0.808**  |

## Publish payment transactions to Aiven for Apache Kafka

<img width="574" alt="arch2" src="https://user-images.githubusercontent.com/31252819/149685595-82ba779f-06dc-4646-8b70-cb1265794a13.png">

### Step 1: Create Aiven for Apache Kafka Service

1. Log in to the [Aiven web console](https://console.aiven.io/).
2. On the Services page, click **Create a new service**.
<br />This opens a new page with the available service options.
3. For the service type, choose **Kafka version 3.0**

4. For the cloud provider and region, choose a cloud provider and region of your choice. 
> **NOTE**: pricing for the same service may vary between different providers and regions. The service summary on the right side of the console shows you the pricing for your selected options. Since this is a tutorial, I choose **Google Cloud** and **google-europe-north1 (Finland)** as they provide the lowest cost.

<img width="798" alt="kafka2" src="https://user-images.githubusercontent.com/31252819/149666522-aae84338-5a8b-4b31-acb5-0755cd82fb01.png">

5. For service plan, choose **Startup**, then **Startup-2**.


This plan provides 3 brokers spread across 3 geographically distinct Availability zones. Each broker has 1 CPU, 2 GB RAM and 30 GB SSD Storage.

> NOTE: In production, choosing the right plan depends on the use-case. Therefore, we recommend sizing your Kafka deployment based on your retention, ingestion and consumption needs. 
> 
> For example, choosing a Startup plan will limit the data retention to 90 GB (30 GB per Node). Moreover, it does not support [Kafka connect](https://aiven.io/kafka-connect) integration. If your use-case requires more data storage or integration with Kafka connect, then choose a **Business** or **Premium** plan. For business-critical production use-cases we recommend choosing Business or Premium. 
> 
> However, regardless of the plan you choose, all plans allow you to seamlessly switch between different plans. Therefore, you can always start small and scale based on your capacity and integration needs. For more information, see [Scaling options in Apache Kafka](https://developer.aiven.io/docs/products/kafka/concepts/horizontal-vertical-scaling)

<img width="789" alt="Screenshot 2022-01-16 at 15 40 19" src="https://user-images.githubusercontent.com/31252819/149668169-7ea4fba4-267f-4869-95d3-9b23c06b0adc.png">

6. Enter **kafka-tutorial** as a name for your service.

7. On the right pane, you will find a summary of your deployment and charges. Click **Create Service**
 <img width="272" alt="kafka4" src="https://user-images.githubusercontent.com/31252819/149668492-1187e76a-856c-4d75-8227-5d40a0cdea82.png">

It will take a few minutes before our new Kafka 3-node cluster shows up in RUNNING state.


### Step 2: Create an Apache Kafka topic

In Kafka, a *topic* is a category/feed name to which records are stored and published. Producers write data to topics and consumers read from topics.

There are 2 ways to create topics in Kafka:

1. By admins - This is recommended in production. By doing this, Kafka producers can only push data to pre-created topics.

2. Producers - This allows producers to dynamically create topics on the fly while pushing the first record. Although, this provides flexibility in development phase, it is not recommended in production as you could eventually end-up 1000s of topics that are hardly used.

Therefore, since the recommended approach is to only create topics when needed, we will keep the default setting of ```auto.create.topics.enable``` to ```false``` and create a topic manually instead.

 1. On the service page in [Aiven web console](https://console.aiven.io/), click on the newly created Kafka service **kafka-tutorial**.
 
 2. Click on the **Topics** tab
 
 3. Enter **transactions** as topic name and click **Add topic**.
 > Note: We used the default configuration which creates **1 partition** for the topic, sets the replication factor to **2 copies** and retention to **7 days**.

<img width="619" alt="kafka6" src="https://user-images.githubusercontent.com/31252819/149670465-de256f00-b454-4922-a0c3-29554dd1cb07.png">

The newly created topic will be found under **Topic list**.


### Step 3: Create and run a Python Producer

We will use the [kafka-python](https://pypi.org/project/kafka-python/) client to build our producer. The producer will use [Faker](https://faker.readthedocs.io/en/master/) (python library) and kafka-python to generate and publish fake payment transactions to our newly created Kafka cluster. The following is a sample transaction record:
```
{
"id": "e8b47ac9-aa1a-4314-9f89-0a17f2497b0f"
"event_time": "2022-01-16T20:05:37.362114"
"transaction_amt":  1234
"ip_address": "120.150.116.29"
"email_address": "kevinadams@example.net"
"transaction_currency": "UAH"
"card_number": "180049174468438"
}
```
#### Download dependencies and code

Run the following steps on your local computer.

1. Install kafka-python client and Faker by running the following:

```pip3 install kafka-python && pip3 install Faker```

2. Using command line clone the repo onto your local development machine using ```git clone <repo url>```

3. Change to the producer code directory.

```
cd getting-started-with-aiven-kafka/Artifacts
```
  
#### Getting cluster connection details
 
 To connect the producer to our cluster, we need to get the cluster connection details.
 
 1. On the service page in [Aiven web console](https://console.aiven.io/), click on the newly created Kafka service **kafka-tutorial**.
 
 2. Under Connection information:
    * Copy Service URI - this is the endpoint that will be used to connect the producer to our cluster
    * Download the **Access Key**, **Access Certificate** and **CA Certificate** to ```getting-started-with-aiven-kafka/Artifacts``` directory on your computer.

<img width="658" alt="kafka5" src="https://user-images.githubusercontent.com/31252819/149669530-657985a8-c6a0-4c18-9b48-e3484a863318.png">

#### Local development

In this sub-section we will edit, walk through and run the pre-created producer code.

1. Open producer.py file in an editor of your choice.

2. First we import the dependencies.

```python
from kafka import KafkaProducer
import json
from time import sleep
from datetime import datetime
import random
```

2. Next we import and initialize Faker.

```python

# Import and initialise faker
from faker import Faker

fake = Faker()
```
 
3. Define a function that generates fake transaction data.


```python
# creating function to generate payment transactions
def genrateTransaction():

    uuid = fake.uuid4()
    
    key = {
        'id': uuid
        }
    
    value = {
        'id': uuid,
        'event_time': datetime.now().isoformat(sep='T'),
        'transaction_amt': random.randint(5,2000),
        'ip_address': fake.ipv4(),
        'email_address': fake.email(),
        'transaction_currency': fake.currency_code(),
        'card_number': fake.credit_card_number()
        }
    return key, value
    
```

4. Then initialize a Producer.
<br /> Replace **<SERVICE_URI>** with the Aiven cluster endpoint you copied previously.

```python

# creating a Kafka producer client that will publish transactions to kafka
path = "../Artifacts/"
producer = KafkaProducer(
    bootstrap_servers="<SERVICE_URI>",
    security_protocol="SSL",
    ssl_cafile=path+"ca.pem",
    ssl_certfile=path+"service.cert",
    ssl_keyfile=path+"service.key",
    value_serializer=lambda v: json.dumps(v).encode('utf-8'),
    key_serializer=lambda v: json.dumps(v).encode('utf-8')
)

```

5. Publish 300 transactions to Kafka by generating 5 transactions/second.
<br /> Replace **<kafka_topic>** with **"transactions"** or the topic name you created earlier.

```python
# Publish 300 (5/sec) payment transactions to a Kafka topic named transactions

topic_name = "<kafka_topic>" # Topic name created in Aiven web console. eg transactions
for number in range(300):

    sleep(0.2)  # Sleep of 0.2 seconds
    
    key, value = genrateTransaction()

    try:
    
        FutureRecord = producer.send(topic_name,key=key,value=value)
        producer.flush()
        record_metadata = FutureRecord.get(timeout=10)
        print("sent event with timestamp {} to Kafka! topic {} partition {} offset {}".format(value['event_time'],record_metadata.topic, record_metadata.partition, record_metadata.offset))

    except Exception as e:
        print(e.with_traceback())
```

5. Using command line, run ```producer.py``` with:

```
python3 ./producer.py
```


### Step 4: View published messages in Aiven console

When working with Kafka, developers normally need a quick and easy way to view the state of their topics and see if recent messages are being ingested correctly. To do this, you would need to install and manage third-party tools like [Conduktor](https://www.conduktor.io/) or [Kafdrop](https://github.com/obsidiandynamics/kafdrop). Aiven simplifies this process by enabling Aiven for Apache Kafka users to view messages from Aiven console. Here we will use this feature to verify that our producer is working and producing messages successfully to Kafka.

 1. On the service page in [Aiven web console](https://console.aiven.io/), click the new kafka service **kafka-tutorial**.
 
 2. On the **Overview** tab, find **Kafka REST API (Karapace)** section and **enable** it. This enables the HTTP REST based interface to the Kafka cluster and it is required to be able to view the messages we published earlier.
 
 2. Click on the **Topics** tab at the top of the page.
 
 3. Under **Topic Lists**, click on **transactions** topic.
<img width="945" alt="kafka7" src="https://user-images.githubusercontent.com/31252819/149677014-cad4ca2d-1384-4859-aff5-bf072fa4ecf3.png">
 This will open the **Topic info** page


 4. Click on **Messages**.

<img width="879" alt="kafka8" src="https://user-images.githubusercontent.com/31252819/149677179-56566e11-beac-406e-8c59-8af73dedf54d.png">

 5. Click on **Fetch Message** then enable **Decode from base64**.

<img width="1235" alt="kafka9" src="https://user-images.githubusercontent.com/31252819/149677293-24f89295-3b94-4bc7-a917-af5ef9560106.png">


Now you can view the actual topic content and verify that the messages have been successfully ingested into our Kafka topic.

<img width="1092" alt="kafka10" src="https://user-images.githubusercontent.com/31252819/149677366-0ba44811-4897-4241-88dd-898d07c5236f.png">


## Observability

In production, it is imperative to understand and monitor how Kafka environments are performing under load. Using InfluxDB and Grafana, Aiven customers can seamlessly collect and visualize Aiven Kafka and system resource metrics. Aiven simplifies this process even further by providing granular dashboards for Aiven Kafka instances by default.

In this section, we will publish Kafka advanced telemetry data to InfluxDB and visualise it using Grafana.
<img width="818" alt="arch3" src="https://user-images.githubusercontent.com/31252819/149685798-97747f92-c45f-477d-8e85-de56920d8e71.png">


### Launch InfluxDB and Grafana services


1. On the Services page, launch a new InfluxDB service.
  * **Service:** InfluxDB
  * **Cloud provider:** Google Cloud
  * **Service plan:** startup-4
  
  <img width="264" alt="kafka11" src="https://user-images.githubusercontent.com/31252819/149678305-73e182e3-d94d-42fc-9486-a53b3335026c.png">

2. On the Services page, launch a new Grafana service.
  * **Service:** Grafana
  * **Cloud provider:** Google Cloud
  * **Service plan:** startup-1
  
  <img width="269" alt="kafka12" src="https://user-images.githubusercontent.com/31252819/149678349-567dddb0-5441-4708-974d-6eb5bb6ebedb.png">

### Enable Service Integrations

1. Once the services are launched, open the InfluxDB service **Overview** page and find the **Service Integrations** section. Click on **Manage integrations**.

<img width="670" alt="kafka13" src="https://user-images.githubusercontent.com/31252819/149678605-3fafe730-9864-4804-8b88-771404d35210.png">

2. Click on **Use integration** for both **Dashboards** and **Metrics** and use your **new Grafana service** and **kafka-tutorial** respectively. The integration overview should look similar to this:

<img width="1327" alt="kafka14" src="https://user-images.githubusercontent.com/31252819/149678857-f2be2202-74e3-4904-ba13-fd53a272f932.png">

That is all that is needed to get the advanced Kafka telemetry data flowing to the InfluxDB service! Click on **Close** to close the integrations dialog.

### Access the default dashboards in Grafana

To access Grafana dashboards, we need to get Grafana connection details.

1.  Open the Grafana service **Overview** page.

2.  Under Connection information, take a note of **User** and **Password**.

<img width="649" alt="kafka15" src="https://user-images.githubusercontent.com/31252819/149679041-0c165ff1-7e3b-4a58-ad03-e0db2c10323b.png">

3. Click on the **Service URI** to open Grafana and enter the copied username and password.

<img width="951" alt="kafka16" src="https://user-images.githubusercontent.com/31252819/149679196-e2baba7d-0b15-46ab-a119-38c83b638cfd.png">

4. On the left pane, hover your mouse over Dashboards icon and then click on **Browse**.

<img width="729" alt="kafka17" src="https://user-images.githubusercontent.com/31252819/149679347-1e9bf117-afbb-42bf-9cf1-ec6f372deb23.png">

5. Click the dashboard name **Aiven Kafka-kafka-tutorial-Resources** to open up the Kafka metrics dashboard.

<img width="1380" alt="kafka18" src="https://user-images.githubusercontent.com/31252819/149679466-52d717a5-6615-4147-8da0-f46e9ac3af3f.png">

### Create a custom dashboard

There are a few key Kafka metrics that should be monitored in any setup. The following are 5 examples:

* **Disk usage**: Disk space currently consumed vs. available. Consider scaling your storage if disk utilization exceeds 70%.
* **UnderReplicatedPartitions**: To ensure data durabilty this metric should always show as 0. A value greater than 0 indicates that there unreplicated partitions.
* **RequestsPerSecond**: Number of (producer|consumer|follower) requests per second.
* **TotalTimeMs**: Total time (in ms) to serve Produce/Fetch request.
* **UncleanLeaderElectionsPerSec**: Number of “unclean” elections per second. Unclean leader election occurs when a broker becomes the new leader replicating the latest data updates from the previous leader. This could lead to data loss, and thus, ideally this metric should always be 0. 
<br /> If you favour data durabilty over availabilty keep the default setting of [unclean.leader.election.enable](https://kafka.apache.org/documentation/#brokerconfigs_unclean.leader.election.enable) to **false**. If availabilty is more important, set this configuration parameter to **true**.

Follow the steps below to create a new custom dashboard:

1. On the left pane, hover your mouse over Dashboards icon and then click on **Browse**

2. Click **New Dashboard**.
<img width="1147" alt="kafka19" src="https://user-images.githubusercontent.com/31252819/149680596-65cb6338-131c-492e-8fe6-a62c1a9b6bf0.png">

3. Choose **Add a new panel**.

4. Choose your InfluxDB Datasource, it should look similar to this:
<img width="1022" alt="kafka20" src="https://user-images.githubusercontent.com/31252819/149680690-a0bb4f50-1415-482a-8a33-920f0519c585.png">

5. Under Query A:
  * For FROM clause, select default for database and **disk** for measurement,
  * For WHERE clause, pick project = __yourproject__; service = __yourservice__,
  * For SELECT, pick field(used_percent), followed by mean(),
  * For Panel name, enter **Disk Usage**
  * Click on Save 
  
<img width="1436" alt="kafka21" src="https://user-images.githubusercontent.com/31252819/149682871-5c2f25d0-1ef1-443c-abd1-cef9001515d6.png">


6. For Dashboard name, enter **Key Kafka Metrics** and click **Save**.

7. For each of the below metrics, click on **Add panel** icon and repeat steps 3 and 4.
<img width="504" alt="kafka22" src="https://user-images.githubusercontent.com/31252819/149682718-b6deafaf-1302-4ac7-a3d0-9a1f9bf07ebe.png">

  
  * **UnderReplicatedPartitions**:
      * For FROM clause, select default for database and **kafka.server:ReplicaManager.UnderReplicatedPartitions** for measurement,
      * For WHERE clause, pick project = __yourproject__; service = __yourservice__,
      * For SELECT, pick field(Value), followed by max() (under Selectors)
      * For Panel name, enter **Under Replicated Partitions**
      * Click on **Apply**
  * **RequestsPerSecond**:
      * For FROM clause, select default for database and **kafka.network:RequestMetrics,name=RequestsPerSec** for measurement,
      * For WHERE clause, pick project = __yourproject__; service = __yourservice__, request= **Produce**
      * For SELECT, pick field(Count), followed by count()
      * For Panel name, enter **Produce Requests Per Sec**
      * Click on **Apply**
  * **TotalTimeMs**:
      * For FROM clause, select default for database and **kafka.network:RequestMetrics,name=TotalTimeMs** for measurement,
      * For WHERE clause, pick project = __yourproject__; service = __yourservice__,
      * For SELECT, pick field(95thPercentile), followed by mean()
      * For Panel name, enter **95thPercentile time to serve requests**
      * Click on **Apply**


8. Now the Dashboards should look similar to this:

<img width="1381" alt="kafka23" src="https://user-images.githubusercontent.com/31252819/149682127-64de29ce-255d-48a0-a2e4-f6a7b3536de5.png">

9. **HOMEWORK:** add a new panel to visualise **UncleanLeaderElectionsPerSec**.


## Cleanup

To avoid additional charges, turn-off your Aiven Services after completing the tutorial. After powering a service off, you still have the ability to turn it back on if needed. 

For Services that have been powered-off for 90 days, Aiven will send an email notification(s) reminding you that the service has been powered-off for a long period. After 180 consecutive days being powered-off the service will be marked for automatic deletion. 

To power-off the services created in this tutorial, repeat the following steps for each service (Kafka, InfluxDB and Grafana):

1. Click on the service in [Aiven web console](https://console.aiven.io/)

2. Click **Power off service**

3. Check **I understand the effects of this action** and click on **Power off**
> **WARNING**: For InfluxDB and Grafana any data added after the latest backup will be lost when powering off. For Kafka all data on the cluster and all topic data and configuration will be lost on power off.

