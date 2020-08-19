#
# AZ 204 Notes

# Azure VM

VMs can be defined and deployed on Azure in several ways: the Azure portal, a script (using the Azure CLI or Azure PowerShell), or an Azure Resource Manager template.

Resources used in VM:

- A virtual machine that provides CPU and memory resources
- An Azure Storage account to hold the virtual hard disks
- Virtual disks to hold the OS, applications, and data
- A virtual network (VNet) to connect the VM to other Azure services or your on-premises hardware
- A network interface to communicate with the VNet
- An optional public IP address so you can access the VM

Like other Azure services, you&#39;ll need a Resource Group to contain the VM (and optionally group these resources for administration). When you create a new VM, you can either use an existing resource group or create a new one.

## Sizing your VM

| **What are you doing?** | **Consider these sizes** |
| --- | --- |
| **General use computing/web** : Testing and development, small to medium databases, or low to medium traffic web servers. | B, Dsv3, Dv3, DSv2, Dv2 |
| --- | --- |
| **Heavy computational tasks** : Medium traffic web servers, network appliances, batch processes, and application servers. | Fsv2, Fs, F |
| **Large memory usage** : Relational database servers, medium to large caches, and in-memory analytics. | Esv3, Ev3, M, GS, G, DSv2, Dv2 |
| **Data storage and processing** : Big data, SQL, and NoSQL databases that need high disk throughput and I/O. | Ls |
| **Heavy graphics rendering** or video editing, as well as model training and inferencing (ND) with deep learning. | NV, NC, NCv2, NCv3, ND |
| **High-performance computing (HPC)**: Your workloads need the fastest and most powerful CPU virtual machines with optional high-throughput network interfaces. | H |

Storage Options: Options include a traditional platter-based hard disk drive (HDD) or a more modern solid-state drive (SSD). Just like the hardware you purchase, SSD storage costs more but provides better performance.

There are two levels of SSD storage available: standard and premium. Choose Standard SSD disks if you have normal workloads but want better performance. Choose Premium SSD disks if you have I/O intensive workloads or mission-critical systems that need to process data very quickly.

By default, two virtual hard disks (VHDs) will be created for **your Linux VM:**

1. The **operating system disk** : This is your primary drive, and it has a maximum capacity of 2048 GB. It will be labeled as /dev/sda by default in Linux and C: drive for Windows.
2. A **temporary disk** : This provides temporary storage for the OS or any apps. On Linux virtual machines, the disk is /dev/sdb (D: for Windows) and is formatted and mounted to /mnt by the Azure Linux Agent. It is sized based on the VM size and is used to store the swap file.

**Managed disks** are the newer and recommended disk storage model. They elegantly solve this complexity by putting the burden of managing the storage accounts onto Azure. You specify the disk type (Premium or Standard) and the size of the disk, and Azure creates and manages both the disk _and_ the storage it uses. You don&#39;t have to worry about storage account limits, which makes them easier to scale out.

## Understanding the pricing model

There are two separate costs the subscription will be charged for every VM: compute and storage. By separating these costs, you scale them independently and only pay for what you need.

**Compute costs** - Compute expenses are priced on a per-hour basis but billed on a per-minute basis. For example, you are only charged for 55 minutes of usage if the VM is deployed for 55 minutes. You are not charged for compute capacity if you stop and deallocate the VM since this releases the hardware. The hourly price varies based on the VM size and OS you select. The cost for a VM includes the charge for the Windows operating system. Linux-based instances are cheaper because there is no operating system license charge.

**Storage costs** - You are charged separately for the storage the VM uses. The status of the VM has no relation to the storage charges that will be incurred; even if the VM is stopped/deallocated and you aren&#39;t billed for the running VM, you will be charged for the storage used by the disks.

You&#39;re able to choose from two payment options for compute costs.

| **Option** | **Description** |
| --- | --- |
| **Pay as you go** | With the **pay-as-you-go** option, you pay for compute capacity by the second, with no long-term commitment or upfront payments. You&#39;re able to increase or decrease compute capacity on demand as well as start or stop at any time. Prefer this option if you run applications with short-term or unpredictable workloads that cannot be interrupted. For example, if you are doing a quick test, or developing an app in a VM, this would be the appropriate option. |
| --- | --- |
| **Reserved Virtual Machine Instances** | The Reserved Virtual Machine Instances (RI) option is an advance purchase of a virtual machine for one or three years in a specified region. The commitment is made up front, and in return, you get up to 72% price savings compared to pay-as-you-go pricing. **RIs** are flexible and can easily be exchanged or returned for an early termination fee. Prefer this option if the VM has to run continuously, or you need budget predictability, **and** you can commit to using the VM for at least a year. |

## What is a network security group?

Virtual networks (VNets) are the foundation of the Azure networking model and provide isolation and protection. Network security groups (NSGs) are the primary tool you use to enforce and control network traffic rules at the networking level. NSGs are an optional security layer that provides a software firewall by filtering inbound and outbound traffic on the VNet. Security groups can be associated to a network interface (for per host rules), a subnet in the virtual network (to apply to multiple resources), or both levels.

## Opening ports in Azure VMs

By default, new VMs are locked down. Apps can make outgoing requests, but the only inbound traffic allowed is from the virtual network (e.g. other resources on the same local network), and from Azure&#39;s Load Balancer (probe checks). There are two steps to adjusting the configuration to support FTP. When you create a new VM you have an opportunity to open a few common ports (RDP, HTTP, HTTPS, and SSH). However, if you require other changes to the firewall, you will need to do them yourself.

The process for this involves two steps:

1. Create a Network Security Group.
2. Create an inbound rule allowing traffic on port 20 and 21 for active FTP support.

#### What is a fault domain?

A fault domain is a logical group of hardware in Azure that shares a common power source and network switch. You can think of it as a rack within an on-premises datacenter. The first two VMs in an availability set will be provisioned into two different racks so that if the network or the power failed in a rack, only one VM would be affected. Fault domains are also defined for managed disks attached to VMs.

![](RackMultipart20200819-4-xk7hcl_html_ac50eeb8a66d984d.png)

#### What is an update domain?

An update domain is a logical group of hardware that can undergo maintenance or be rebooted at the same time. Azure will automatically place availability sets into update domains to minimize the impact when the Azure platform introduces host operating system changes. Azure then processes each update domain one at a time.

**Azure Site Recovery** replicates workloads from a primary site to a secondary location. If an outage happens at your primary site, you can fail over to a secondary location.

**Azure Backup** is a _backup as a service_ offering that protects physical or virtual machines no matter where they reside: on-premises or in the cloud.

az group create \

--name \&lt;resource-group-name\&gt; \

--location \&lt;resource-group-location\&gt;

az vm create \

--resource-group learn-f92c72eb-d901-47f5-855f-068e27bb16c4 \

--name MeanStack \

--image Canonical:UbuntuServer:16.04-LTS:latest \

--admin-username azureuser \

--generate-ssh-keys

az vm open-port \

--port80 \

--resource-group learn-f92c72eb-d901-47f5-855f-068e27bb16c4 \

--name MeanStack

#

# Azure Event and Event Hubs

## Azure Event Grid

Azure [Event Grid](https://azure.microsoft.com/services/event-grid/) is a fully-managed event routing service running on top of Azure [Service Fabric](https://azure.microsoft.com/services/service-fabric/). Event Grid distributes _events_ from different sources, such as Azure [Blob storage accounts](https://azure.microsoft.com/services/storage/blobs/) or Azure [Media Services](https://azure.microsoft.com/services/media-services/), to different handlers, such as Azure [Functions](https://azure.microsoft.com/services/functions/) or Webhooks. Event Grid was created to make it easier to build event-based and serverless applications on Azure.

There are several concepts in Azure Event Grid that connect a source to a subscriber:

- **Events:** What happened.
- **Event sources:** Where the event took place.
- **Topics:** The endpoint where publishers send events.
- **Event subscriptions:** The endpoint or built-in mechanism to route events, sometimes to multiple handlers. Subscriptions are also used by handlers to filter incoming events intelligently.
- **Event handlers:** The app or service reacting to the event.

Events can be generated by the following Azure resource types:

- **Azure Subscriptions and Resource Groups:** Subscriptions and resource groups generate events related to management operations in Azure. For example, when a user creates a virtual machine, this source generates an event.
- **Container registry:** The Azure Container Registry service generates events when images in the registry are added, removed, or changed.
- **Event Hub:** Event Hub can be used to process and store events from a variety of data sources - typically logging or telemetry related. Event Hub can generate events to Event Grid when files are captured.
- **Service Bus:** Service bus can generate events to Event Grid when there are active messages with no active listeners.
- **Storage accounts:** Storage accounts can generate events when users add blobs, files, table entries, or queue messages. You can use both blob accounts and General-purpose V2 accounts as event sources.
- **Media Services:** Media Services hosts video and audio media and provides advanced management features for media files. Media Services can generate events when an encoding job is started or completed on a video file.
- **Azure IoT Hub:** IoT Hub communicates with and gathers telemetry from IoT devices. It can generate events whenever such communications arrive.
- **Custom events:** Custom events can be generated using the REST API, or with the Azure SDK on Java, GO, .NET, Node, Python, and Ruby. For example, you could create a custom event in the Web Apps feature of Azure App Service. This can happen in the worker role when it picks up a message from a storage queue..

Azure can receive and handle events from Event Grid:

- **Azure Functions:** Custom code that runs in Azure, without the need for explicit configuration of a host virtual server or container. Use an Azure function as an event handler when you want to code a custom response to the event.
- **Webhooks:** A webhook is a web API that implements a push architecture.
- **Azure Logic Apps:** An Azure logic app hosts a business process as a workflow.
- **Microsoft Flow:** Flow also hosts workflows, but it is easier for non-technical staff to use.

## Event Hubs

[Event Hubs](https://azure.microsoft.com/services/event-hubs/) is an intermediary for the publish-subscribe communication pattern. Unlike [Event Grid](https://azure.microsoft.com/services/event-grid/), however, it is optimized for extremely high throughput, a large number of publishers, security, and resiliency.

As Event Hubs receives communications, it divides them into partitions. Partitions are buffers into which the communications are saved. Because of the event buffers, events are not completely ephemeral, and an event isn&#39;t missed just because a subscriber is busy or even offline. The subscriber can always use the buffer to &quot;catch up.&quot; By default, events stay in the buffer for 24 hours before they automatically expire.

All publishers are authenticated and issued a token. This means Event Hubs can accept events from external devices and mobile apps, without worrying that fraudulent data from pranksters could ruin our analysis.

**Choose Event Hubs if:**

- You need to support authenticating a large number of publishers.
- You need to save a stream of events to Data Lake or Blob storage.
- You need aggregation or analytics on your event stream.
- You need reliable messaging or resiliency.

Otherwise, if you need a simple event publish-subscribe infrastructure, with trusted publishers (for instance, your own web server), you should choose Event Grid.

# Service Bus queues and storage queues

There are two Azure features that include message queues: Service Bus and Azure Storage accounts. As a general guide, storage queues are simpler to use but are less sophisticated and flexible than Service Bus queues.

Key advantages of Service Bus queues include:

- Supports larger messages sizes of 256 KB (standard tier) or 1MB (premium tier) per message versus 64 KB
- Supports both at-least-once and at-most-once delivery - choose between a very small chance that a message is lost or a very small chance it is handled twice
- Guarantees **first-in-first-out (FIFO)** order - messages are handled in the same order they are added (although FIFO is the normal operation of a queue, it is not guaranteed for every message)
- Can group multiple messages into a transaction - if one message in the transaction fails to be delivered, all messages in the transaction will not be delivered
- Supports role-based security
- Does not require destination components to continuously poll the queue

Advantages of storage queues:

- Supports unlimited queue size (versus 80-GB limit for Service Bus queues)
- Maintains a log of all messages

**Choose Service Bus queues if:**

- You need an at-most-once delivery guarantee
- You need a FIFO guarantee
- You need to group messages into transactions
- You want to receive messages without polling the queue
- You need to provide role-based access to the queues
- You need to handle messages larger than 64 KB but smaller than 256 KB
- Your queue size will not grow larger than 80 GB
- You would like to be able to publish and consume batches of messages

**Choose queue storage if:**

- You need a simple queue with no particular additional requirements
- You need an audit trail of all messages that pass through the queue
- You expect the queue to exceed 80 GB in size
- You want to track progress for processing a message inside of the queue

To access a queue, you need three pieces of information:

1. Storage account name
2. Queue name
3. Authorization token

# Storage Account

The types of storage accounts are:

- **General-purpose v2 accounts** : Basic storage account type for blobs, files, queues, and tables. Recommended for most scenarios using Azure Storage.
- **General-purpose v1 accounts** : Legacy account type for blobs, files, queues, and tables. Use general-purpose v2 accounts instead when possible.
- **BlockBlobStorage accounts** : Storage accounts with premium performance characteristics for block blobs and append blobs. Recommended for scenarios with high transactions rates, or scenarios that use smaller objects or require consistently low storage latency.
- **FileStorage accounts** : Files-only storage accounts with premium performance characteristics. Recommended for enterprise or high performance scale applications.
- **BlobStorage accounts** : Legacy Blob-only storage accounts. Use general-purpose v2 accounts instead when possible.

Redundancy options for a storage account include:

- Locally redundant storage (LRS): A simple, low-cost redundancy strategy. Data is copied synchronously three times within the primary region.
- Zone-redundant storage (ZRS): Redundancy for scenarios requiring high availability. Data is copied synchronously across three Azure availability zones in the primary region.
- Geo-redundant storage (GRS): Cross-regional redundancy to protect against regional outages. Data is copied synchronously three times in the primary region, then copied asynchronously to the secondary region. For read access to data in the secondary region, enable read-access geo-redundant storage (RA-GRS).
- Geo-zone-redundant storage (GZRS) (preview): Redundancy for scenarios requiring both high availability and maximum durability. Data is copied synchronously across three Azure availability zones in the primary region, then copied asynchronously to the secondary region. For read access to data in the secondary region, enable read-access geo-zone-redundant storage (RA-GZRS).

## Azure Blob storage

It is Microsoft&#39;s object storage solution for the cloud. Blob storage is optimized for storing massive amounts of unstructured data. Unstructured data is data that doesn&#39;t adhere to a particular data model or definition, such as text or binary data.

![](RackMultipart20200819-4-xk7hcl_html_c5ebbac3301020df.png)

A **storage account** provides a unique namespace in Azure for your data. Every object that you store in Azure Storage has an address that includes your unique account name. A **container** organizes a set of blobs, similar to a directory in a file system. Azure Storage supports three types of **blobs** :

- **Block blobs** store text and binary data. Block blobs are made up of blocks of data that can be managed individually. Block blobs store up to about 4.75 TiB of data. Larger block blobs are available in preview, up to about 190.7 TiB
- **Append blobs** are made up of blocks like block blobs, but are optimized for append operations. Append blobs are ideal for scenarios such as logging data from virtual machines.
- **Page blobs** store random access files up to 8 TB in size. Page blobs store virtual hard drive (VHD) files and serve as disks for Azure virtual machines.

## Azure Cosmos DB

Azure Cosmos DB is Microsoft&#39;s globally distributed, multi-model database service. You can elastically scale throughput and storage, and take advantage of fast, single-digit-millisecond data access using your favorite API including: SQL, MongoDB, Cassandra, Tables, or Gremlin.

### **Turnkey global distribution** : Cosmos DB enables you to build highly responsive and highly available applications worldwide. Cosmos DB transparently replicates your data wherever your users are, so your users can interact with a replica of the data that is closest to them.

Distributed databases that rely on replication for high availability, low latency, or both must make tradeoffs. The tradeoffs are between read consistency vs. availability, latency, and throughput.

Azure Cosmos DB approaches data consistency as a spectrum of choices. This approach includes more options than the two extremes of strong and eventual consistency. You can choose from five well-defined levels on the consistency spectrum. From strongest to weakest, the levels are:

- _Strong_
- _Bounded staleness_
- _Session_
- _Consistent prefix_
- _Eventual_

Each level provides availability and performance tradeoffs and is backed by comprehensive SLAs.

In one Azure data center, your data is written out to multiple replicas (four at least). Consistency is all about whether or not you can be sure that the data you are reading at any point in time is – in fact – the latest version of that data, because, you can be reading from any replica at any given time. And if it&#39;s not the latest version, then this is known as a &quot;dirty read.&quot; On the one extreme, strong consistency guarantees you&#39;ll never experience a dirty read, but of course, this guarantee comes with a cost, because Cosmos DB can&#39;t permit reads until all the replicas are updated. This results in higher latency, which degrades performance.

On the other end of the spectrum, there&#39;s eventual consistency, which offers no guarantees whatsoever, and you never know with certainty whether or not you&#39;re reading the latest version of data. This gives Cosmos DB the freedom to serve your reads from any available replica, without having to wait for that replica to receive the latest updates from other writes. In turn, this results in the lowest latency, and delivers the best performance, at the cost of potentially experiencing dirty reads.

## Five Consistency Levels

### Strong

In the case of Strong Consistency, you can be certain that you&#39;re always reading the latest data. But you pay for that certainty in performance. Because when somebody writes to the database, everybody waits for Cosmos DB to acknowledge that the write was successfully saved to all the replicas. Only then can Cosmos DB serve up consistent reads, guaranteed. Cosmos DB actually does support strong constency across regions, assuming that you can tolerate the high latency that will result while replicating globally.

### Bounded Staleness

Next there&#39;s bounded staleness, which is kind of one step away from strong. As the name implies, this means that you can tolerate stale data, but up to a point. So you can actually configure much how out of date you want to allow for stale reads. Think of it as controlling the level of freshness, where you might have dirty reads, but only if the data isn&#39;t too much out of date. Otherwise, for data that&#39;s too stale, Cosmos DB will switch to strong consistency. In terms of defining how stale is stale, you can specify lag in terms of time – like, no stale data older than X, or update operations – as in, no stale data that&#39;s been updated more than Y number of times.

### Session

And then there&#39;s session consistency, which is actually the default consistency level. With session consistency, you maintain a session for your writer – or writers – by defining a unique session key that all writers include in their requests. Cosmos DB then uses the session key to ensure strong consistency for the writer, meaning that a writer is always guaranteed to read the same data that they wrote, and will never read a stale version from an out-of-date replica – while everyone else may experience dirty reads.

When you create a DocumentClient instance with the .NET SDK, it automatically establishes a session key and includes it with every request. This means that within the lifetime of a DocumentClient instance, you are guaranteed strong consistency for reading any data that you&#39;ve written, when using session consistency. This is a perfect balance for many scenarios, which is why session consistency the default. Because very often, you want to sure immediately that the data you&#39;ve written has – in fact – been written, but it&#39;s perfectly OK if it takes a little extra time for everyone else to see what you&#39;ve written, once all the replicas have caught up with your write.

### Consistent Prefix

With consistent prefix, dirty reads are always possible. However, you do get some guarantees. First, when you do get stale data from a dirty read, you can at least be sure that the data you get has in fact been updated to all the replicas, even thought it&#39;s not the most up-to-date version of that data, which has still yet to be replicated. And second, you&#39;ll never experience reads out of order. For example, say the same data gets updated four times, so there are versions A, B, C, and D, where D is the must up to date version, and A, B, and C are stale versions. Now assume A and B have both been replicated, but C and D haven&#39;t yet. You can be sure that, until C and D do get replicated, you&#39;ll never read versions A and B out of order. That is, you might first get version A when only A has been replicated, and then get version B once version B has been replicated, and then you&#39;ll never again get version A. You can only get the stale versions in order, so you&#39;ll only continue getting version B until a later version gets replicated.

### Eventual

Eventual consistency is the weakest consistency level. It offers no guarantees whatsoever, in exchange for the lowest latency and best performance compared to all the other consistency levels. With eventual, you never wait for any acknowledgement that data has been fully replicated before reading. This means that read requests can be serviced by replicas that have already been updated, or by those that haven&#39;t. So you can get inconsistent results in a seemingly random fashion, until eventually, all the replicas get updated and then all the reads become consistent – until the next write operation, of course. So this really is the polar opposite of strong consistency, because now there&#39;s no guarantee that you&#39;re ever reading the latest data, but there&#39;s also never any waiting, which delivers the fastest performance.

## Container

An Azure Cosmos container is the unit of scalability both for provisioned throughput and storage. A container is horizontally partitioned and then replicated across multiple regions. The items that you add to the container and the throughput that you provision on it are automatically distributed across a set of logical partitions based on the partition key. To learn more about partitioning and partition keys, see [Partition data](https://docs.microsoft.com/en-gb/azure/cosmos-db/partition-data).

When you create an Azure Cosmos container, you configure throughput in one of the following modes:

- **Dedicated provisioned throughput mode** : The throughput provisioned on a container is exclusively reserved for that container and it is backed by the SLAs.
- **Shared provisioned throughput mode** : These containers share the provisioned throughput with the other containers in the same database (excluding containers that have been configured with dedicated provisioned throughput). In other words, the provisioned throughput on the database is shared among all the &quot;shared throughput&quot; containers.

## Partition

Azure Cosmos DB uses partitioning to scale individual containers in a database to meet the performance needs of your application. In partitioning, the items in a container are divided into distinct subsets called _logical partitions_. Logical partitions are formed based on the value of a _partition key_ that is associated with each item in a container. All items in a logical partition have the same partition key value. For example, a container holds items. Each item has a unique value for the UserID property. If UserID serves as the partition key for the items in the container and there are 1,000 unique UserID values, 1,000 logical partitions are created for the container. In addition to a partition key that determines the item&#39;s logical partition, each item in a container has an _item ID_ (unique within a logical partition). Combining the partition key and the _item ID_ creates the item&#39;s _index_, which uniquely identifies the item.

#

# Azure Content Delivery Network (CDN)

A content delivery network (CDN) is a distributed network of servers that can efficiently deliver web content to users. CDNs store cached content on edge servers in point-of-presence (POP) locations that are close to end users, to minimize latency. Azure Content Delivery Network (CDN) offers developers a global solution for rapidly delivering high-bandwidth content to users by caching their content at strategically placed physical nodes across the world. Azure CDN can also accelerate dynamic content, which cannot be cached, by leveraging various network optimizations using CDN POPs. For example, route optimization to bypass Border Gateway Protocol (BGP). The benefits of using Azure CDN to deliver web site assets include:

- Better performance and improved user experience for end users, especially when using applications in which multiple round-trips are required to load content.
- Large scaling to better handle instantaneous high loads, such as the start of a product launch event.
- Distribution of user requests and serving of content directly from edge servers so that less traffic is sent to the origin server.

To use Azure CDN, you must own at least one Azure subscription. You also need to create at least one CDN profile, which is a collection of CDN endpoints. Every CDN endpoint represents a specific configuration of content deliver behavior and access. To organize your CDN endpoints by internet domain, web application, or some other criteria, you can use multiple profiles.

Azure Content Delivery Network (CDN) includes four products: **Azure CDN Standard from Microsoft** , **Azure CDN Standard from Akamai** , **Azure CDN Standard from Verizon** , and **Azure CDN Premium from Verizon**.

| **Performance features and optimizations** | **Standard Microsoft** | **Standard Akamai** | **Standard Verizon** | **Premium Verizon** |
| --- | --- | --- | --- | --- |
| [Dynamic site acceleration](https://docs.microsoft.com/en-us/azure/cdn/cdn-dynamic-site-acceleration) | Offered via [Azure Front Door Service](https://docs.microsoft.com/en-us/azure/frontdoor/front-door-overview) | ✓ | ✓ | ✓ |
| [Dynamic site acceleration - adaptive image compression](https://docs.microsoft.com/en-us/azure/cdn/cdn-dynamic-site-acceleration#adaptive-image-compression-azure-cdn-from-akamai-only) |
 | ✓ |
 |
 |
| [Dynamic site acceleration - object prefetch](https://docs.microsoft.com/en-us/azure/cdn/cdn-dynamic-site-acceleration#object-prefetch-azure-cdn-from-akamai-only) |
 | ✓ |
 |
 |
| [General web delivery optimization](https://docs.microsoft.com/en-us/azure/cdn/cdn-optimization-overview#general-web-delivery) | ✓ | ✓, Select this optimization type if your average file size is smaller than 10 MB | ✓ | ✓ |
| [Video streaming optimization](https://docs.microsoft.com/en-us/azure/cdn/cdn-media-streaming-optimization) | via General Web Delivery | ✓ | via General Web Delivery | via General Web Delivery |
| [Large file optimization](https://docs.microsoft.com/en-us/azure/cdn/cdn-large-file-optimization) | via General Web Delivery | ✓, Select this optimization type if your average file size is larger than 10 MB | via General Web Delivery | via General Web Delivery |

# Azure App Configuration

Provides a service to centrally manage application settings and feature flags. Modern programs, especially programs running in a cloud, generally have many components that are distributed in nature. Spreading configuration settings across these components can lead to hard-to-troubleshoot errors during an application deployment. Use App Configuration to store all the settings for your application and secure their accesses in one place. App Configuration complements [Azure Key Vault](https://azure.microsoft.com/services/key-vault/), which is used to store application secrets

While any application can make use of App Configuration, the following examples are the types of application that benefit from the use of it:

- Microservices based on Azure Kubernetes Service, Azure Service Fabric, or other containerized apps deployed in one or more geographies
- Serverless apps, which include Azure Functions or other event-driven stateless compute apps
- Continuous deployment pipeline

# Azure Logic Apps

[Azure Logic Apps](https://azure.microsoft.com/services/logic-apps) is a cloud service that helps you schedule, automate, and orchestrate tasks, business processes, and [workflows](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-overview#logic-app-concepts) when you need to integrate apps, data, systems, and services across enterprises or organizations. Logic Apps simplifies how you design and build scalable solutions for app [integration](https://azure.microsoft.com/product-categories/integration/), data integration, system integration, enterprise application integration (EAI), and business-to-business (B2B) communication, whether in the cloud, on premises, or both. Every logic app workflow starts with a trigger, which fires when a specific event happens, or when new available data meets specific criteria. Many triggers provided by the connectors in Logic Apps include basic scheduling capabilities so that you can set up how regularly your workloads run. For more complex scheduling or advanced recurrences, you can use a Recurrence trigger as the first step in any workflow. Each time that the trigger fires, the Logic Apps engine creates a logic app instance that runs the actions in the workflow. These actions can also include data conversions and workflow controls, such as conditional statements, switch statements, loops, and branching. For example, this logic app starts with a Dynamics 365 trigger with the built-in criteria &quot;When a record is updated&quot;. If the trigger detects an event that matches this criteria, the trigger fires and runs the workflow&#39;s actions. Here, these actions include XML transformation, data updates, decision branching, and email notifications.

| **Category** | **Description** |
| --- | --- |
| **[Managed connectors](https://docs.microsoft.com/en-us/azure/connectors/apis-list#managed-api-connectors)** | Create logic apps that use services such as Azure Blob Storage, Office 365, Dynamics, Power BI, OneDrive, Salesforce, SharePoint Online, and many more. |
| --- | --- |
| **[On-premises connectors](https://docs.microsoft.com/en-us/azure/connectors/apis-list#on-premises-connectors)** | After you install and set up the [on-premises data gateway](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-connection), these connectors help your logic apps access on-premises systems such as SQL Server, SharePoint Server, Oracle DB, file shares, and others. |
| **[Integration account connectors](https://docs.microsoft.com/en-us/azure/connectors/apis-list#integration-account-connectors)** | Available when you create and pay for an integration account, these connectors transform and validate XML, encode and decode flat files, and process business-to-business (B2B) messages with AS2, EDIFACT, and X12 protocols. |

## Azure API Management

Azure API Management (APIM) is a fully managed cloud service that you can use to publish, secure, transform, maintain, and monitor APIs. It helps organizations publish APIs to external, partner, and internal developers to unlock the potential of their data and services. API Management handles all the tasks involved in mediating API calls, including request authentication and authorization, rate limit and quota enforcement, request and response transformation, logging and tracing, and API version management. APIM enables you to create and manage modern API gateways for existing backend services no matter where they are hosted.

Because you can publish Azure Functions through API Management, you can use them to implement a microservices architecture: Each function implements a microservice. By adding many functions to a single API Management product, you can build those microservices into an integrated distributed application. Once the application is built, you can use API Management policies to implement caching or ensure security requirements.

### APIM Consumption Tier

When you choose a usage plan for API Management, you can choose the consumption tier, because it&#39;s especially suited to microservice-based architectures and event-driven systems. For example, it would be a great choice for our online store web API.

The consumption tier uses the same underlying service components as the previous tiers but employs an entirely different architecture based on shared, dynamically allocated resources. It aligns perfectly with serverless computing models. There is no infrastructure to manage, no idle capacity, high-availability, automatic scaling, and usage-based pricing, all of which make it an especially good choice for solutions that involve exposing serverless resources as APIs.

# Azure App Service

Azure App Service is a fully managed web application hosting platform. This platform as a service (PaaS) offered by Azure allows you to focus on designing and building your app while Azure takes care of the infrastructure to run and scale your applications.

An **App Service** plan is a set of virtual server resources that run App Service apps. A plan&#39;s **size** (sometimes referred to as its **sku** or **pricing tier** ) determines the performance characteristics of the virtual servers that run the apps assigned to the plan and the App Service features that those apps have access to. Every App Service web app you create must be assigned to a single App Service plan that runs it. Pricing and reliability levels:

**Shared compute** : **Free** and **Shared** , the two base tiers, run an app on the same Azure VM as other App Service apps, including apps of other customers. These tiers allocate CPU quotas to each app that runs on the shared resources, and the resources cannot scale-out. Free and Shared plans are best for small-scale personal projects with limited traffic demands, with a set limit of 165 MB of outbound data every 24 hours.

**Dedicated compute** : The **Basic, Standard, Premium, and Premium V2** tiers run apps on dedicated Azure VMs. Only apps in the same App Service plan share the same compute resources. The higher the tier, the more VM instances are available to you for scale out. The Standard service plan is best suited for live production workloads, where you are publishing commercial applications to customers. The Premium service plans support high-capacity web apps where you do not want the additional costs of a dedicated (isolated) plan.

**Isolated** : This tier runs dedicated Azure VMs on dedicated Azure virtual networks, which provide network isolation on top of compute isolation to your apps. It provides the maximum scale-out capabilities. You would only select an Isolated service plan when you have a specific requirement for the highest levels of security and performance.

Isolate your app into a new App Service plan when:

- The app is resource-intensive
- You want to scale the app independently from the other apps in the existing plan
- The app needs resources in a different geographical region

Deployment slots: Free, Shared and Basic = 0, Standard = 5, Premium and Isolated = 20

# How a shared access signature works

A shared access signature is a signed URI that points to one or more storage resources and includes a token that contains a special set of query parameters. The token indicates how the resources may be accessed by the client. One of the query parameters, the signature, is constructed from the SAS parameters and signed with the key that was used to create the SAS. This signature is used by Azure Storage to authorize access to the storage resource.

You can sign a SAS in one of two ways:

- With a _user delegation key_ that was created using Azure Active Directory (Azure AD) credentials. A user delegation SAS is signed with the user delegation key.
- With the storage account key. Both a service SAS and an account SAS are signed with the storage account key. To create a SAS that is signed with the account key, an application must have access to the account key.

# Azure RBAC

A _ **security principal** _ is an object that represents a user, group, service principal, or managed identity that is requesting access to Azure resources.

![](RackMultipart20200819-4-xk7hcl_html_4672b7783b0aa111.png)

- User - An individual who has a profile in Azure Active Directory. You can also assign roles to users in other tenants. For information about users in other organizations
- Group - A set of users created in Azure Active Directory. When you assign a role to a group, all users within that group have that role.
- Service principal - A security identity used by applications or services to access specific Azure resources. You can think of it as a _user identity_ (username and password or certificate) for an application.
- Managed identity - An identity in Azure Active Directory that is automatically managed by Azure. You typically use [managed identities](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview) when developing cloud applications to manage the credentials for authenticating to Azure services.

A _ **role definition** _is a collection of permissions. _ **Scope** _ is the set of resources that the access applies to. A _ **role assignment** _ is the process of attaching a role definition to a user, group, service principal, or managed identity at a particular scope for the purpose of granting acces

![](RackMultipart20200819-4-xk7hcl_html_8710f9d2b8971b71.png)

The following are the high-level steps that Azure RBAC uses to determine if you have access to a resource on the management plane. This is helpful to understand if you are trying to troubleshoot an access issue.

1. A user (or service principal) acquires a token for Azure Resource Manager.

The token includes the user&#39;s group memberships (including transitive group memberships).

1. The user makes a REST API call to Azure Resource Manager with the token attached.
2. Azure Resource Manager retrieves all the role assignments and deny assignments that apply to the resource upon which the action is being taken.
3. Azure Resource Manager narrows the role assignments that apply to this user or their group and determines what roles the user has for this resource.
4. Azure Resource Manager determines if the action in the API call is included in the roles the user has for this resource.
5. If the user doesn&#39;t have a role with the action at the requested scope, access is not granted. Otherwise, Azure Resource Manager checks if a deny assignment applies.
6. If a deny assignment applies, access is blocked. Otherwise access is granted.

# Serverless Applications

Azure includes four different technologies that you can use to build and implement workflows that integrate multiple systems:

## Design-first technologies

The principal question here is who will design the workflow: will it be developers or users? In Logic Apps, there is a GUI designer on which you draw out the workflow. It is intuitive and easy to use but you also have the opportunity to delve under the hood and edit the source code for a workflow. This tool is designed for people with development skills. In Microsoft Power Automate, extra help and templates are provided for common types of workflow. There is no way to edit the source code that the tool creates. This tool is designed for users who have a good understanding of the business process but no coding skills.

**Logic Apps**

is a service within Azure that you can use to automate, orchestrate, and integrate disparate components of a distributed application. By using the design-first approach in Logic Apps, you can draw out complex workflows that model complex business processes

**Microsoft Power Automate**

is a service that you can use to create workflows even when you have no development or IT Pro experience. You can create workflows that integrate and orchestrate many different components by using the website or the Microsoft [Power Automate mobile app](https://flow.microsoft.com/mobile/download/).

## Code-first technologies

Because of the extra features that are included with Azure Functions, including wider ranges of trigger events and supported languages, the ability to develop test code in the browser, and the pay-per-use price model, consider Azure Functions to be your default choice. There are two situations in which WebJobs might be a better choice:

- You have an existing Azure App Service application, and you want to model the workflow within the application. This requirement means that the workflow can also be managed as part of the application, for example in an Azure DevOps environment.
- You have specific customizations that you want to make to the JobHost that are not supported by Azure Functions. For example, in a WebJob, you can create a custom retry policy for calls to external systems. This kind of policy can&#39;t be configured in an Azure Function.

**WebJobs**

are a part of the Azure App Service that you can use to run a program or script automatically. There are two kinds of WebJob:

- **Continuous.** These WebJobs run in a continuous loop. For example, you could use a continuous WebJob to check a shared folder for a new photo.
- **Triggered.** These WebJobs run when you manually start them or on a schedule.

To determine what actions your WebJobs takes, you can write code in several different languages. For example, you can script the WebJob by writing code in a Shell Script (Windows, PowerShell, Bash). Alternatively, you can write a program in PHP, Python, Node.js, or Java.

**Azure Functions**

is a simple way for you to run small pieces of code in the cloud, without having to worry about the infrastructure required to host that code. You can write the Function in C#, Java, JavaScript, PowerShell, Python, In addition, with the consumption plan option, you only pay for the time when the code runs. Azure automatically scales your function in response to the demand from users. When you create an Azure Function, you can start by writing the code for it in the portal. Alternatively, if you need source code management, you can use GitHub or Azure DevOps Services.

![](RackMultipart20200819-4-xk7hcl_html_c43c616d8cf1fb93.png)

# Azure Functions

An Azure function app doesn&#39;t do work until something tells it to execute. For example, we could create an Azure function to send out a reminder text message to our customers before an appointment. If we don&#39;t tell the function when it should run, our customers will never receive a message.

## What is a trigger?

A trigger is an object that defines how an Azure function is invoked. For example, if you want a function to execute every 10 minutes, you could use a timer trigger.

Every function must have exactly one trigger associated with it. If you want to execute a piece of logic that runs under multiple conditions, you need to create multiple functions that share the same core function code.

In this module, we&#39;re going to focus on the **timer** , **HTTP** , and **blob** trigger types.

## Types of triggers

Azure Functions support a wide range of trigger types. Here are some of the most common types:

| Types of triggers |
| --- |
| **Type** | **Purpose** |
| **Timer** | Execute a function at a set interval. |
| --- | --- |
| **HTTP** | Execute a function when an HTTP request is received. |
| **Blob** | Execute a function when a file is uploaded or updated in Azure Blob storage. |
| **Queue** | Execute a function when a message is added to an Azure Storage queue. |
| **Azure Cosmos DB** | Execute a function when a document changes in a collection. |
| **Event Hub** | Execute a function when an event hub receives a new event. |

## What is a binding?

A binding is a connection to data within your function. Bindings are optional and come in the form of input and output bindings. An input binding is the data that your function receives. An output binding is the data that your function sends. Unlike a trigger, a function can have multiple input and output bindings.

## What is a binding?

In Azure Functions, bindings provide a declarative way to connect to data from within your code. They make it easier to integrate with data streams consistently in a function. You can have multiple bindings providing access to different data elements. This is powerful because you can connect to your data sources without having to code specific connection logic (like database connections or web API interfaces).

### Types of bindings

There are two kinds of bindings you can use with functions:

1. **Input binding** - An input binding is a connection to a data **source**. Our function can read data from these inputs.
2. **Output binding** - An output binding is a connection to a data **destination**. Our function can write data to these destinations.

There are also _triggers_, which are special types of input bindings that cause a function to execute. For example, an Azure Event Grid notification can be configured as a trigger. When an event occurs, the function will run.

### Types of supported bindings

The _type_ of binding defines where we are reading or sending data. There is a binding to respond to web requests and a large selection of bindings to interact directly with various Azure services as well as third-party services. A binding type can be used as an input, an output or both. For example, a function can write to Azure Blob Storage output binding, but a blob storage update could trigger another function. Some common binding types are listed below:

- Blob Storage
- Azure Service Bus Queues
- Azure Cosmos DB
- Azure Event Hubs
- External Files
- External Tables
- HTTP endpoints

### Binding properties

Three properties are required in all bindings. You may have to supply additional properties based on the type of binding and storage you are using.

1. **Name** - Defines the function parameter through which you access the data. For example, in a queue input binding, this is the name of the function parameter that receives the queue message content.
2. **Type** - Identifies the type of binding, i.e., the type of data or service we want to interact with.
3. **Direction** - Indicates the direction data is flowing, i.e., is it an input or output binding?

Additionally, most binding types also need a fourth property:

1. **Connection** - Provides the name of an app setting key that contains the connection string. Bindings use connection strings stored in app settings to keep secrets out of the function code. This makes your code more configurable and secure.

## Durable functions

Durable functions are an extension of Azure Functions. Whereas Azure Functions operate in a stateless environment, Durable Functions can retain state between function calls. This approach enables you to simplify complex stateful executions in a serverless-environment. Durable Functions scales as needed, and provides a cost effective means of implementing complex workflows in the cloud. Some benefits of using Durable Functions include:

- They enable you to write event driven code. A durable function can wait asynchronously for one or more external events, and then perform a series of tasks in response to these events.
- You can chain functions together. You can implement common patterns such as fan-out/fan-in, which uses one function to invoke others in parallel, and then accumulate the results.
- You can orchestrate and coordinate functions, and specify the order in which functions should execute.
- The state is managed for you. You don&#39;t have to write your own code to save state information for a long-running function.

Durable functions allows you to define stateful workflows using an Orchestration function. An orchestration function provides these extra benefits:

- You can define the workflows in code. You don&#39;t need to write a JSON description or use a workflow design tool.
- Functions can be called both synchronously and asynchronously. Output from the called functions is saved locally in variables and used in subsequent function calls.
- Azure checkpoints the progress of a function automatically when the function awaits. Azure may choose to dehydrate the function and save its state while the function waits, to preserve resources and reduce costs. When the function starts running again, Azure will rehydrate it and restore its state.

## Function types

You can use three durable function types: _Client_, _Orchestrator_, and _Activity_.

**Client** functions are the entry point for creating an instance of a Durable Functions orchestration. They can run in response to an event from many sources, such as a new HTTP request arriving, a message being posted to a message queue, an event arriving in an event stream. You can write them in any of the supported languages.

**Orchestrator** functions describe how actions are executed, and the order in which they are run. You write the orchestration logic in code (C# or JavaScript).

**Activity** functions are the basic units of work in a durable function orchestration. An activity function contains the actual work performed by the tasks being orchestrated.

## Application patterns

You can use Durable Functions to implement many common workflow patterns. These patterns include:

- **Function chaining** - In this pattern, the workflow executes a sequence of functions in a specified order. The output of one function is applied to the input of the next function in the sequence. The output of the final function is used to generate a result.

![](RackMultipart20200819-4-xk7hcl_html_31b57c79fc738cf8.png)

- **Fan out/fan in** - This pattern runs multiple functions in parallel and then waits for all the functions to finish. The results of the parallel executions can be aggregated or used to compute a final result.

![](RackMultipart20200819-4-xk7hcl_html_2d6820de9f107cdb.png)

- **Async HTTP APIs** - This pattern addresses the problem of coordinating state of long-running operations with external clients. An HTTP call can trigger the long-running action. Then, it can redirect the client to a status endpoint. The client can learn when the operation is finished by polling this endpoint.

![](RackMultipart20200819-4-xk7hcl_html_ab7685554b6da4b4.png)

- **Monitor** - This pattern implements a recurring process in a workflow, possibly looking for a change in state. For example, you could use this pattern to poll until specific conditions are met.

![](RackMultipart20200819-4-xk7hcl_html_2279eec6b6eb2f74.png)

- **Human interaction** - This pattern combines automated processes that also involve some human interaction. A manual process within an automated process is tricky because people aren&#39;t as highly available and as responsive as most computers. Human interaction can be incorporated using timeouts and compensation logic that runs if the human fails to interact correctly within a specified response time. An approval process is an example of a process that involves human interaction.

![](RackMultipart20200819-4-xk7hcl_html_b564fea418472046.png)

## Comparison with Logic Apps

Durable Functions and Logic Apps are both Azure services that enable serverless workload. Azure Durable Functions is intended as a powerful serverless compute option to run custom logic. Azure Logic Apps is better suited for integrating Azure services and components. You can use either technology to create complex orchestrations. With Azure Durable Functions, you develop orchestrations by writing code and using the Durable Functions extension. With Logic Apps, you create orchestrations by using the design surface or editing configuration files.

| Comparison with Logic Apps |
| --- |
|
 | **Azure Durable Functions** | **Azure Logic Apps** |
| Development | Code-first (imperative) | Design-first (declarative) |
| --- | --- | --- |
| Connectivity | About a dozen built-in binding types. You can write code for custom bindings. | Large collection of connectors. Enterprise Integration Pack for B2B.You can also build custom connectors. |
| Actions | Each activity is an Azure Function. You write the code for activity functions. | Large collection of ready-made actions. You integrate custom logic through custom connectors. |
| Monitoring | Azure Application Insights | Azure portal, Azure Monitor logs |
| Management | REST API, Visual Studio | Azure portal, REST API, PowerShell, Visual Studio |
| Execution context | Can run locally or in the cloud | Runs only in the cloud |
