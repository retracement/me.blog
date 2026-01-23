+++
date = '2022-07-29T07:15:27+01:00'
draft = false
title = 'A Quick Introduction to Azure Purview'
+++

![Data Catalog](/images/2022/catalogue-of-data-ai-600.png)

Azure Purview is Microsoft's next-generation data catalog service in Azure which is a direct replacement for the first-generation service known as Azure Data Catalog. While some people refer to Purview as Azure Data Catalog v2, It is important to note that Purview is more of a re-write of Azure Data Catalog rather than an upgrade, and as such, no migration path between both services currently exists.

Azure Purview can be used to inventory, classify, map, and govern your on-premises, multi-cloud, and SaaS data estate, providing a holistic up-to-date map of your data with automatic data discovery, data classification, and end-to-end data lineage.

Purview's Unified Platform implements the Apache Atlas API allowing both read and write capabilities against it. Having this capability provides the ability for external applications to easily query and update the data catalog or it could be used for integration with other services and solutions such as custom-built metadata stores <sub>***1**</sub> – although we will detail some restrictions that may cause difficulties in doing so.

In this quick post, we will detail the three key Purview capabilities at the time of writing.

# Data Catalogue
The key component of Azure Purview is its data cataloging capabilities, but in order to scan data sources, they must first be defined as scan definitions. Purview can then automatically connect to these data sources and build up a list of objects and their schemas as data assets. The unified platform looks at these data assets, document their data lineages, and applies data classification.

It is also possible to define a business glossary providing a hierarchical view of your data, and these glossary terms may then be applied to your data assets. In this way, you are able to filter the business glossary terms against all of your discovered assets.

Interestingly you can add Power BI as a data source to Purview, and a scan will list the contents of a workspace including Power BI reports, datasets, and dataflows. The lineage of Power BI assets is (at the time of writing) restricted, and Purview does not give the same level of entity and column-level lineage that would be available for Data Factory pipeline lineage scans. The produced reporting lineage only provides basic reference information and does not even show the lineage of the report visualisations.

While fairly rich querying of the data catalog exists, Purview provides the Asset insights functionality to provide catalog reporting capability. This is sadly currently very limited, and there is community feedback to suggest more value would be gained from integrating Power BI reporting capability against Purview to write your own reports.

# Data lineage
Data lineage is important since it helps you understand your data supply chain. Specifically, we want to know where does data come from and where does it go to? How is the data transformed, and how is it consumed?

Purview implements basic data lineage capabilities and will automatically pick up the lineage of assets by scanning certain services such as Azure Data Factory. Lineage can be established by scanning these service pipelines and determining which assets are related through copy activities. The lineage view shows which columns exist in each of these assets. Unfortunately, at present only the Data Activity copy activity is supported, meaning that all of the transformation and other activities are not.

We have already touched upon some of the lineage shortfalls with Power BI assets and won't repeat them again in this section, but just be aware they exist. There are other missing data lineage capabilities, and this includes a lack of current support for database views. This is obviously a big hole in the Data lineage capability and will hopefully be addressed in the future.

# Data classification
Data classification is equally important and helps you provide business context to your data, identify PII data (that may or may not be a risk), and empowers consumers to find valuable trustworthy data through the use of those tags.

Defining data sources does not automatically define or start a collection or classification, it simply just defines the source. Therefore, in order to get these insights from the data source, you need to configure a new scan – which is essentially just a name, credentials, and scope. For example, a scope applied to a scan of a data lake source could select specific folders for scanning, or to scan everything from its root.

A scan also requires a scan rule set for each data source that needs to be scanned. The scan rule set is simply a collection of scan rules, and each of these defines one or more file types to be included for each rule, as well as what classification rules to apply. Classification rules are either system or custom defined, the latter of which obviously allows user-defined patterns to meet specific domain-specific requirements. The classification rules are therefore simply attempting to classify data that meet certain criteria such as finding email addresses, telephone numbers, and certain regex patterns against the underlying metadata.

Custom file types may be in use on the data lake, and whilst Purview does allow for the creation of these definitions (of files using different file extensions), this functionality is lacking in certain areas. For example, when a custom file type is defined, a customer delimiter must be specified, otherwise, an existing system-defined file type must be provided for the basis of this file format. In other words, the choices are either a CSV format type using a customer delimiter (and file extensions) OR an existing system file type but using different file extensions.

It is also worth pointing out that while it is possible to manually edit your data assets (something you may need to do to apply a new classification), this currently prevents Purview from automatically updating the asset for the entire schema of the asset for future scans. Clearly, this shortfall would be an unacceptable problem when using Purview towards an integrated cataloging solution.

# Summary
With the release of Azure Purview, Microsoft provides us with a data governance solution that is aimed toward the Enterprise. Microsoft has not made any secret of its commitment to building upon Purview capabilities and intention towards integrating this service with many of its services and tooling. For example, it is possible to connect an Azure Synapse Workspace to a Purview account to provide Purview search functionality to Synapse <sub>***2**</sub>. In this post, we have detailed a variety of functionality that is currently missing in this important service, and have also neglected to mention others <sub>***3**</sub>.

Azure Purview should certainly be on your organisations radar for providing a cloud-native data dictionary, data lineage, and data classification solution, and is certainly an improvement upon Azure Data Factory, but before making the leap of faith, it is important to understand if it meets all your metadata requirements today.

---

# Footnotes
***1** A custom-built meta-store might be useful in an environment in which organisations have multiple toolsets to provide data governance capabilities, and reasons for doing so might include avoiding vendor lock-in, complementing missing functionality, or building upon pre-existing technical investments.

***2** See the Microsoft Learn article Discover, connect, and explore data in Synapse using Microsoft Purview

***3** While many of the current Purview shortfalls that exist at the time of writing are expected to be resolved over time, there are capabilities that Purview alone is not expected to address and will require additional services. A good example here is the lack of data quality and master data services functionality within Purview itself, although the use of third-party services such as Informatica or Cluedin are expected to integrate well with Purview.