+++
date = '2023-01-31T15:36:14Z'
draft = false
title = 'Why Is My Azure Solution Running Slow'
+++
As an Azure data professional, one of the most common questions you are most likely to hear from users is "why do cloud solutions often appear to have performance issues post-deployment". This perception is particularly common for Azure migration projects moving on-premises into the cloud, but do these observations really match reality or are they misguided?

In this introductory article, we will take a look into some of the reasons why Azure cloud-based solutions are often seen to be problematic, or in some cases, are problematic, and provide some ideas to avoid these negative perceptions in your cloud environment.

# CAPEX vs OPEX mindset
The way in which a solution is financed can have a fundamental effect on the way in which initial deployment and ongoing support are decided.

On-premises solutions are usually CAPEX-driven meaning:

These project types buy the biggest and most performant solution that the budget allows.
There is a desire to accommodate future workloads for years to come. For example, once a project has procured a solution, they understand that they are generally stuck with it!
There is a strong focus on careful requirements analysis (particularly with performance in mind).
Cloud solutions are OPEX driven since you pay for service selection and usage meaning:

There is an ongoing balance between cost and capability.
The ability to easily scale up (or out) means there is less focus on "getting it right first time" and result in a sub-optimal initial deployment.
A less thorough analysis of performance requirements is usually a due to the perception of ease of change, as well as a lack of understanding of how these requirements translate (see the Shape of Workloads section).

**Shape of Workloads**

Every single solution and its constituent components have specific workload types to process. These workloads can be a mixture of compute, memory, storage, or network requirements. The workloads will have various properties (such as latency, processing, or bandwidth requirements) which can substantially impact the performance of a solution. Understanding the shape of a workload is important to understanding the service capability needed to accommodate it.

Considerations are as follows:

- A mismatch in one performance requirement can usually have a knock on effect to others. For example, a lack of memory can result in higher storage requests. This problem is often seen in database systems (such as SQL Server) or operating systems (such as MS Windows) and is known as paging or spooling. Mismatching in Cloud deployments probably occurs more frequently for many of the reasons described elsewhere in this text (lack of sizing equivalence, sizing cost concerns, architectural and paradigm differences, etc.)
- Seasonality of workloads will often mean that seasonal variations and periodic peaks and troughs in performance can often be a result of inadequate performance monitoring of a source system.
- There are usually service disparities between a cloud service performance capability and on-premises hardware (we discuss this in the Cloud Architecture section below).

Often in projects, not enough work is done to establish the true shape of data, however, with CAPEX solutions problems do not usually materialise early on post-delivery due to the excessive sizes of hardware procured upfront. Cloud services are usually undersized from day one of delivery for the reasons discussed earlier.

**Cloud Architecture**

Under the covers of Azure Cloud Architecture is a complex multi-tenant virtualised environment that is completely abstracted from the underlying specialised virtual machines. Many services have an even higher level of abstraction (particularly with PaaS services) in which the exact performance capacity can be obfuscated. This can result in many of the concerns below:

- There is often no direct equivalence between on-premises architecture and cloud services (for example CPU, Memory, Storage, etc). Obviously, this makes it harder to match existing architectural patterns to the right cloud service size (a practice known as "right-sizing"). It is all too easy to get wrong and to under-size. There are various examples of abstracted service capacities, some range from complete abstraction (as is the case with Cosmos DB), to partial abstract in which only pre-defined (and fixed) t-shirt style sizing can be selected (for example in Azure Databricks). Some services do however provide slightly less abstraction with the capability to select compute, memory, and storage requirements independently.
![Cosmos througput](/images/cosmos-performance-request-units.webp)
**Figure 1 - Cosmos account throughput**

   An example of Cosmos DB performance abstraction can be seen above. The concept of RU or Request Unit is Microsoft's abstraction for consumption performance. Right-sizing RU's does not remove the requirement for adequate partitioning and indexing (and querying!). Failure to implement performant structures and code will raise the RU requirements (and cost!). An explanation of request units can be found here https://docs.microsoft.com/en-us/azure/cosmos-db/request-units
- Microsoft regularly rolls out service updates across its cloud platform. They will of course do this using the canary deployment methodology, but ultimately a production performance regression (or bug) could result in performance blips or even service unavailability (though these are rare). It is important to highlight that rolling updates mean higher security due to a lack of known exploits being present in their service code base, but these do come with this increased risk (however minimal).
- Different architectural paradigms (for example Data Lakehouse) or Globally distributed databases can result in performance concerns or considerations.
- In-built Cloud service protections can provide added performance latencies – for example Cosmos DB partition replicas which automatically create 3 additional copies of data. Another example can be seen with Azure Storage in which the default Locally Redundant Storage option automatically maintains 3 replicas of data within a single data center. Azure Storage LRS provides 99.999999999% (11 9's) availability by default, but obviously, there will be a minimal performance impact in doing so.
- Cloud services require different specialist (and experienced) developer and administrator skillsets to ensure and maintain optimal solution performance. On-premises services are well established and have a huge pool of available subject matter experts in the marketplace. This of course reduces (but does not remove) the probability that on-premises deployments will be well-managed, well-supported, and performant.
- A move towards the Cloud will often result in an initial push towards Hybrid deployments in which an on-premises solution will be integrated with a new Cloud solution. This can:
- Increase network latencies across networks
- Obfuscate problem areas (obviously the problem is the cloud people will say!)
- Provide an unnecessary dependency on on-premises performance

As we can see, there are lots of reasons why performance concerns might be encountered in Cloud architectures, but it is also worth calling out one further reason. Recent architectural trends and user consumption patterns have resulted in an emerging shift towards "self-service everything" (reporting, data models, analytics). This can be great for productivity and collaboration but has resulted in many inexperienced users causing performance degradations due to poor queries being written or excessive reporting. For example, inefficient querying of a data set with inefficient (or missing) filters could result in an entire data set being "scanned". A scan of a 20TB table could have huge performance and cost ramifications.

**Mitigations**

So far we have detailed reasons why performance issues can occur in Cloud deployments but this does not mean that things have to be this way. Microsoft Azure provides many different ways in which we can avoid many of the problems we describe above.

- Many services provide auto-scale and auto-pause functionality (or it can often be scripted and automated for services that don't), and this can mean that larger service sizes can be chosen as a scale target based upon real-world consumption without incurring excessive costs. A cluster or service can run using small to medium capacity for small to medium workloads, or can automatically scale out (or up) for larger ones.
- Burstable service offerings (such as Azure VM B-series) can be selected to provide temporary bursts in performance without paying for unnecessary service capacity. For example, the Standard_B8ms VM can "burst" up to 400% max CPU performance for 30-minute windows. This type of technology can be perfect for seasonal or unexpected workloads
- There are many available Azure services that can help establish through monitoring and analysis to understand if a cloud service meets actual requirements, and examples of these services include Azure Advisor, Azure Log Analytics, Azure Data Explorer, and Kusto Query Language. Azure Data Explorer and Kusto Query Language could provide ongoing shape analysis against TBs of log data.
- It is possible (and important) to implement automated performance testing to catch problems before solutions are even progressed into production. These would commonly be implemented through the DevOps Continuous Integration and Continuous Delivery pipelines.
- A bigger focus on solution documentation and guidance should be provided to consumers (and required training given to specialist staff).
- Many services implement cost controls – these can avoid unexpected billing expenses but could also result in service unavailability. One of the biggest use cases for cost controls lies in self-service consumption in which service unavailability for these personas would not impact mission-critical systems.

**Summary**

Cloud Driven solutions do not inherently result in unavoidable performance issues when moving to the cloud. Cloud services can provide mission-critical, "infinite scale" solutions but these are (of course) at a cost. It is important however to understand whether the cost of a solution is being spent correctly and that all mitigations are considered to arrive at a correct balance between performance and cost.

Clearly, OPEX can result in too much focus being put on cost-effective solutions rather than performant solutions. The result is usually a right sizing mismatch between the shape of workloads and the capabilities of the selected services to execute them.

Much more effort must be taken in projects to understand the performance requirements of an existing system (and shape of the workloads), and ongoing monitoring of service performance post-deployment -as well as adequate performance testing before solutions even enter production.

Failure to properly understand and implement recommended practices and ways of working with Cloud infrastructure can cause negative perceptions and difficulties for future adoption by businesses that could have been otherwise avoided and in this post, we have hopefully helped you understand how the Azure cloud can help you avoid some of these problems.