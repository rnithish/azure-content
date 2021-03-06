<properties
   pageTitle="Resiliency checklist | Microsoft Azure"
   description="Checklist that provides guidance for resiliency concerns during design."
   services=""
   documentationCenter="na"
   authors="petertaylor9999"
   manager=""
   editor=""
   tags=""/>

<tags
   ms.service="best-practice"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="09/27/2016"
   ms.author="petertay"/>

# Azure resiliency guidance: Resiliency checklist

Designing your application for resiliency requires planning for and mitigating a variety of failure modes that could occur. Review the items in this checklist against your application design to make it more resilient.

## Requirements

- **Define your customer's availability requirements.** Your customer will have availability requirements for the components in your application and this will affect your application's design. Get agreement from your customer for the availability targets of each piece of your application, otherwise your design may not meet the customer's expectations. For more information, see the [Defining your resiliency requirements](guidance-resiliency-overview.md#defining-your-resiliency-requirements) section of the [Designing resilient applications for Azure](guidance-resiliency-overview.md) document.

## Failure Mode Analysis

- **Perform a failure mode analysis (FMA) for your application.** FMA is a process for building resiliency into an application early in the design stage. The goals of an FMA include:  

    - Identify what types of failures an application might experience. 
    - Capture the potential effects and impact of each type of failure on the application.
    - Identify recovery strategies. 

    For more information, see [Designing resilient applications for Azure: Failure mode analysis][fma].  

## Application

- **Deploy multiple instances of services.** Services will inevitably fail, and if your application depends on a single instance of a service it will inevitably fail also. To provision multiple instances for [Azure App Service](../app-service/app-service-value-prop-what-is.md), select an [App Service Plan](../app-service/azure-web-sites-web-hosting-plans-in-depth-overview.md) that offers multiple instances. For Azure Cloud Services, configure each of your roles to use [multiple instances](../cloud-services/cloud-services-choose-me.md#scaling-and-management). For [Azure Virtual Machines (VMs)](../virtual-machines/virtual-machines-windows-about.md), ensure that your VM architecture includes more than one VM and that each VM is included in an [availability set][availability-sets].   

- **Use a load balancer to distribute requests.** A load balancer distributes your application's requests to healthy service instances by removing unhealthy instances from rotation. If your service uses Azure App Service or Azure Cloud Services, it is already load balanced for you. However, if your application uses Azure VMs, you will need to provision a load balancer. See the [Azure Load Balancer](../load-balancer/load-balancer-overview.md) overview for more details. 

- **Configure Azure Application Gateways to use multiple instances.** Depending on your application's requirements, an [Azure Application Gateway](../application-gateway/application-gateway-introduction.md) may be better suited to distributing requests to your application's services. However, single instances of the Application Gateway service are not guaranteed by an SLA so it's possible that your application could fail if the Application Gateway instance fails. Provision more than one medium or larger Application Gateway instance to guarantee availability of the service under the terms of the [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/).

- **Use Availability Sets for each application tier**. Placing your instances in an [availability set][availability-sets] guarantees connectivity to at least one VM instance within the terms of the [SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_2/). If your VMs aren't in an availability set you don't have guarantees protecting them and it's possible that they could all fail or be updated simultaneously. 

- **Consider deploying your application across multiple regions.** If your application is deployed to a single region, in the rare event the entire region becomes unavailable, your application will also be unavailable. This may be unacceptable under the terms of your application's SLA. If so, consider deploying your application and its services across multiple regions. A multi-region deployment can use an active-active pattern (distributing requests across multiple active instances) or an active-passive pattern (keeping a "warm" instance in reserve, in case the primary instance fails). We recommend that you deploy multiple instances of your application's services across regional pairs. For more information, see [Business continuity and disaster recovery (BCDR): Azure Paired Regions](../best-practices-availability-paired-regions.md).

- **Implement resiliency patterns for remote operations where appropriate.** If your application depends on communication between remote services, the communication path will inevitably fail. If there are multiple failures, the remaining healthy instances of your application's services could be overwhelmed with requests. There are several patterns useful for dealing with common failures including the timeout pattern, the [retry pattern][retry-pattern], the [circuit breaker][circuit-breaker] pattern, and others. For more information, see [Designing resilient applications for Azure](guidance-resiliency-overview.md#implementing-resiliency-strategies). 

- **Use autoscaling to respond to increases in load.** If your application is not configured to scale out automatically as load increases, it's possible that your application's services will fail if they become saturated with user requests. For more details, see the following:

    - General: [Scalability checklist](../best-practices-scalability-checklist.md) 
    - Azure App Service: [Scale instance count manually or automatically][app-service-autoscale]
    - Cloud Services: [How to auto scale a cloud service][cloud-service-autoscale]
    - Virtual Machines: [Automatic scaling and virtual machine scale sets][vmss-autoscale]


- **Implement asynchronous operations whenever possible.** Synchronous operations can monopolize resources and block other operations while the caller waits for the process to complete. Design each part of your application to allow for asynchronous operations whenever possible. For more information on how to implement asynchronous programming in C#, see [Asynchronous Programming with async and await][asynchronous-c-sharp].

- **Configure and test health probes for your load balancers and traffic managers.** Ensure that your health logic checks all of the critical parts of the system and responds appropriately to Load Balancer and Traffic Manager probes. If your application doesn't check a critical function such as storage or a database service but responds with a 200 (OK) to the Load Balancer or Traffic Manager, an unhealthy instance will remain in rotation and will still be sent requests even though it's not capable of serving them. See [Health Endpoint Monitoring Pattern](https://msdn.microsoft.com/library/dn589789.aspx) for guidance on implementing health monitoring in your application.

- **Monitor third-party services.** If your application has dependencies on third-party services, identify where and how these third-party services can fail and what effect those failures will have on your application. A third-party service may not include monitoring and diagnostics, so it's important to log your invocations of them and correlate them with your application's health and diagnostic logging using a unique identifier. For more information on best practices for monitoring and diagnostics, see the [Monitoring and Diagnostics guidance][monitoring-and-diagnostics-guidance] document.

- **Ensure that any third-party service you consume provides an SLA.** If your application depends on a third-party service, but the third party provides no guarantee of availability in the form of an SLA, your application's availability also cannot be guaranteed. Your SLA is only as good as the least available component of your application.

- **Use Azure Traffic Manager to route your application's traffic to different regions.**  Azure Traffic Manager performs load balancing functions at the DNS level and can route traffic to different regions based on the traffic routing methods you specify and the health of your application's endpoints. See the [How Traffic Manager works](../traffic-manager/traffic-manager-how-traffic-manager-works.md) document for more information.   

## Data management

- **Understand the replication methods for your application's data sources.** Your application data will be stored in different data sources and have different availability requirements. Evaluate the replication methods for each type of data storage in Azure, including [Azure Storage Replication](../storage/storage-redundancy.md) and [SQL Database Active Geo-Replication](../sql-database/sql-database-geo-replication-overview.md) to ensure that your application's data requirements are satisfied.

- **Ensure that no single user account has access to both production and backup data.** Your data backups are compromised if one single user account has permission to write to both production and backup sources. A malicious user could purposely delete all your data, while a regular user could accidentally delete it. Design your application to limit the permissions of each user account so that only the users that require write access have write access and it's only to either production or backup, but not both.

- **Document your data source fail over and fail back process and test it.** In the case where your data source fails catastrophically, a human operator will have to follow a set of documented instructions to fail over to a new data source. If the documented steps have errors, an operator will not be able to successfully follow them and fail over the resource. Regularly test the instruction steps to verify that an operator following them is able to successfully fail over and fail back the data source.

- **Validate your data backups.** Regularly verify that your backup data is what you expect by running a script to validate data integrity, schema, and queries. There's no point having a backup if it's not useful to restore your data sources. Log and report any inconsistencies so the backup service can be repaired.

- **Consider using a storage account type that is geo-redundant.** Data stored in an Azure Storage account is always replicated locally. However, there are multiple replication strategies to choose from when a Storage Account is provisioned. Select [Azure Read-Access Geo Redundant Storage (RA-GRS)](../storage/storage-redundancy.md#read-access-geo-redundant-storage) to protect your application data against the rare case when an entire region becomes unavailable.

    > [AZURE.NOTE] For VMs, do not rely on RA-GRS replication to restore the VM disks (VHD files). Instead, use [Azure Backup][azure-backup].   

## Operations

- **Implement monitoring and alerting best practices in your application.** Without proper monitoring, diagnostics, and alerting, there is no way to detect failures in your application and alert an operator to fix them. For more information on best practices, see the [Monitoring and Diagnostics guidance][monitoring-and-diagnostics-guidance] document.

- **Measure remote call statistics and make the information available to the application team.**  If you don't track and report remote call statistics in real time and provide an easy way to review this information, the operations team will not have an instantaneous view into the health of your application. And if you only measure average remote call time, you will not have enough information to reveal issues in the services. Summarize remote call metrics such as latency, throughput, and errors in the 99 and 95 percentiles. Perform statistical analysis on the metrics to uncover errors that occur within each percentile.

- **Track the number of transient exceptions and retries over an appropriate timeframe.** If you don't track and monitor transient exceptions and retry attempts over time, it's possible that an issue or failure could be hidden by your application's retry logic. That is, if your monitoring and logging only shows success or failure of an operation, the fact that the operation had to be retried multiple times due to exceptions will be hidden. A trend of increasing exceptions over time indicates that the service is having an issue and may fail. For more information, see [Retry service specific guidance][retry-service-guidance].

- **Implement an early warning system that alerts an operator.** Identify the key performance indicators of your application's health, such as transient exceptions and remote call latency, and set appropriate threshold values for each of them. Send an alert to operations when the threshold value is reached. Set these thresholds at levels that identify issues before they become critical and require a recovery response.

- **Document the release process for your application.** Without detailed release process documentation, an operator might deploy a bad update or improperly configure settings for your application. Clearly define and document your release process, and ensure that it's available to the entire operations team. Best practices for resilient deployment of your application are detailed in the [resilient deployment][guidance-resilient-deployment] section of the Resiliency Guidance document.

- **Ensure that more than one person on the team is trained to monitor the application and perform any manual recovery steps.** If you only have a single operator on the team who can monitor the application and kick off recovery steps, that person becomes a single point of failure. Train multiple individuals on detection and recovery and make sure there is always at least one active at any time.

- **Automate your application's deployment process.** If your operations staff is required to manually deploy your application, human error can cause the deployment to fail. For more information on best practices for automating application deployment, see the [resilient deployment][guidance-resilient-deployment] section of the Resiliency Guidance document.

- **Design your release process to maximize application availability.** If your release process requires services to go offline during deployment, your application will be unavailable until they come back online. Use the [blue/green](http://martinfowler.com/bliki/BlueGreenDeployment.html) or [canary release](http://martinfowler.com/bliki/CanaryRelease.html) deployment technique to deploy your application to production. Both of these techniques involve deploying your release code alongside production code so users of release code can be redirected to production code in the event of a failure. For more information, see the [resilient deployment][guidance-resilient-deployment] section of the Resiliency Guidance document.

- **Log and audit your application's deployments.** If you use staged deployment techniques such as blue/green or canary releases there will be more than one version of your application running in production. If a problem should occur, it's critical to determine which version of your application is causing a problem. Implement a robust logging strategy to capture as much version-specific information as possible. 

- **Ensure that your application does not run up against [Azure subscription limits](../azure-subscription-service-limits.md).** Azure subscriptions have limits on certain resource types, such as number of resource groups, number of cores, and number of storage accounts.  If your application requirements exceed Azure subscription limits, create another Azure subscription and provision sufficient resources there.

- **Ensure that your application does not run up against [per-service limits](../azure-subscription-service-limits.md).** Individual Azure services have consumption limits &mdash; for example, limits on storage, throughput, number of connections, requests per second, and other metrics. Your application will fail if it attempts to use resources beyond these limits. This will result in service throttling and possible downtime for affected users. 

    Depending on the specific service and your application requirements, you can often avoid these limits by scaling up (for example, choosing another pricing tier) or scaling out (adding new instances).  

- **Design your application's storage requirements to fall within Azure storage scalability and performance targets.** Azure storage is designed to function within predefined scalability and performance targets, so design your application to utilize storage within those targets. If you exceed these targets your application will experience storage throttling. To fix this, provision additional Storage Accounts. If you run up against the Storage Account limit, provision additional Azure Subscriptions and then provision additional Storage Accounts there. For more information, see [Azure Storage Scalability and Performance Targets](../storage/storage-scalability-targets.md).

- **Select the right VM size for your application.** Measure the actual CPU, memory, disk, and I/O of your VMs in production and verify that the VM size you've selected is sufficient. If not, your application may experience capacity issues as the VMs approach their limits. VM sizes are described in detail in the [Sizes for virtual machines in Azure](../virtual-machines/virtual-machines-windows-sizes.md) document.

- **Determine if your application's workload is stable or fluctuating over time.** If your workload fluctuates over time, use Azure VM scale sets to automatically scale the number of VM instances. Otherwise, you will have to manually increase or decrease the number of VMs. For more information, see the [Virtual Machine Scale Sets Overview](../virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md).

- **Select the right service tier for Azure SQL Database.** If your application uses Azure SQL Database, ensure that you have selected the appropriate service tier. If you select a tier that is not able to handle your application's database transaction unit (DTU) requirements, your data use will be throttled. For more information on selecting the correct service plan, see the [SQL Database options and performance: Understand what's available in each service tier](../sql-database/sql-database-service-tiers.md) document. 

- **Have a rollback plan for deployment.** It's possible that your application deployment could fail and cause your application to become unavailable. Design a rollback process to go back to a last known good version and minimize downtime. See the [resilient deployment][guidance-resilient-deployment] section of the Resiliency Guidance document for more information. 

- **Create a process for interacting with Azure support.** If the process for contacting [Azure support](https://azure.microsoft.com/support/plans/) is not set before the need to contact support arises, downtime will be prolonged as the support process is navigated for the first time. Include the process for contacting support and escalating issues as part of your application's resiliency from the outset.

- **Ensure that your application doesn't use more than the maximum number of storage accounts per subscription.** Azure allows a maximum of 200 storage accounts per subscription. If your application requires more storage accounts than are currently available in your subscription, you will have to create a new subscription and create additional storage accounts there. For more information, see [Azure subscription and service limits, quotas, and constraints](../azure-subscription-service-limits.md#storage-limits).

- **Ensure that your application doesn't exceed the scalability targets for virtual machine disks.** An Azure IaaS VM supports attaching a number of data disks depending on several factors, including the VM size and type of storage account. If your application exceeds the scalability targets for virtual machine disks, provision additional storage accounts and create the virtual machine disks there. For more information, see [Azure Storage Scalability and Performance Targets] (../storage/storage-scalability-targets.md#scalability-targets-for-virtual-machine-disks)

## Test

- **Perform failover and failback testing for your application.** If you haven't fully tested failover and failback, you can't be certain that the dependent services in your application come back up in a synchronized manner during disaster recovery. Ensure that your application's dependent services failover and fail back in the correct order.

- **Perform fault-injection testing for your application.** Your application can fail for many different reasons, such as certificate expirations, exhaustion of system resources in a VM, or storage failures. Test your application in an environment as close as possible to production by either simulating or triggering real failures. Delete certificates, artificially consume system resources, delete a storage source. Verify your application's ability to recover from all types of faults, alone and in combination.

- **Run tests in production using both synthetic and real user data.** Test and production are rarely identical, so it's important to use blue/green or a canary deployment and test your application in production. This allows you to test your application in production under real load and ensure it will function as expected when fully deployed.

## Security

- **Implement application-level protection against distributed denial of service (DDoS) attacks.** Azure services are protected against DDos attacks at the network layer. However, Azure cannot protect against application-layer attacks, because it is difficult to distinguish between true user requests and malicious user requests. For more information on how to protect against application-layer DDoS attacks, see the "Protecting against DDoS" section of the  [Microsoft Azure Network Security](http://download.microsoft.com/download/C/A/3/CA3FC5C0-ECE0-4F87-BF4B-D74064A00846/AzureNetworkSecurity_v3_Feb2015.pdf) document.

- **Implement the principle of least privilege for access to the application's resources.** The default for access to the application's resources should be as restrictive as possible. Grant higher level permissions on an approval basis. Granting overly permissive access to your application's resources by default can result in someone purposely or accidentally deleting resources. Azure provides [role-based access control](../active-directory/role-based-access-built-in-roles.md) to manage user privileges, but it's important to verify least privilege permissions for other resources that have their own permissions systems such as SQL Server. 

## Telemetry

- **Log telemetry data while the application is running in the production environment.** Capture robust telemetry information while the application is running in the production environment or you will not have sufficient information to diagnose the cause of issues while it's actively serving users. More information is available in the logging best practices is available in the [Monitoring and Diagnostics guidance][monitoring-and-diagnostics-guidance] document.

<!--- **Allow logging levels to be configured at runtime in production.** If you have configured your application to log minimal information in production for performance reasons, you might not have enough information to diagnose the root cause of issues that only occur in production. Add functionality to allow operators to adjust logging verbosity at runtime so you can request that they increase it. [PT - needs to be verified per Masashi]-->

- **Implement logging using an asynchronous pattern.** If logging operations are synchronous, they might block your application code. Ensure that your logging operations are implemented as asynchronous operations. 

- **Correlate log data across service boundaries.** In a typical n-tier application, a user request may traverse several service boundaries. For example, a user request typically originates in the web tier and is passed to the business tier and finally persisted in the data tier. In more complex scenarios, a user request may be distributed to many different services and data stores. Ensure that your logging system correlates calls across service boundaries so you can track the request throughout your application.

##  Azure Resources 

- **Use Azure Resource Manager templates to provision resources.** Resource Manager templates make it easier to automate deployments via PowerShell or the Azure CLI, which leads to a more reliable deployment process. For more information, see [Azure Resource Manager overview][resource-manager].

- **Give resources meaningful names.** Giving resources meaningful names makes it easier to locate a specific resource and understand its role. For more information, see [Recommended naming conventions for Azure resources](guidance-naming-conventions.md) 

- **Use role-based access control (RBAC)**. Use RBAC to control access to the Azure resources that you deploy. RBAC lets you assign authorization roles to members of your DevOps team, to prevent accidental deletion or changes to deployed resources. For more information, see [Get started with access management in the Azure portal](../active-directory/role-based-access-control-what-is.md) 

- **Use resource locks for critical resources, such as VMs.** Resource locks prevent an operator from accidentally deleting a resource. For more information, see [Lock resources with Azure Resource Manager](../resource-group-lock-resources.md) 

- **Regional pairs.** When deploying to two regions, choose regions from the same regional pair. In the event of a broad outage, recovery of one region is prioritized out of every pair. Some services such as Geo-Redundant Storage provide automatic replication to the paired region. For more information, see [Business continuity and disaster recovery (BCDR): Azure Paired Regions](../best-practices-availability-paired-regions.md) 

- **Organize resource groups by function and lifecycle**.  In general, a resource group should contain resources that share the same lifecycle. This makes it easier to manage deployments, delete test deployments, and assign access rights, reducing the chance that a production deployment is accidentally deleted or modified. Create separate resource groups for production, development, and test environments. In a multi-region deployment, put resources for each region into separate resource groups. This makes it easier to redeploy one region without affecting the other region(s). 

## Azure Services 

The following checklist items apply to specific services in Azure.

###  App Service 

- **Use Standard or Premium tier.** These tiers support staging slots and automated backups. For more information, see [Azure App Service plans in-depth overview](../app-service/azure-web-sites-web-hosting-plans-in-depth-overview.md) 

- **Avoid scaling up or down.** Instead, select a tier and instance size that meet your performance requirements under typical load, and then [scale out](../app-service-web/web-sites-scale.md) the instances to handle changes in traffic volume. Scaling up and down may trigger an application restart.  

- **Store configuration as app settings.** Use app settings to hold configuration settings as app settings. Define the settings in your Resource Manager templates, or using PowerShell, so that you can apply them as part of an automated deployment / update process, which is more reliable. For more information, see [Configure web apps in Azure App Service](../app-service-web/web-sites-configure.md). 

- **Create separate App Service plans for production and test.** Don't use slots on your production deployment for testing.  All apps within the same App Service plan share the same VM instances. If you put production and test deployments in the same plan, it can negatively affect the production deployment. For example, load tests might degrade the live production site. By putting test deployments into a separate plan, you isolate them from the production version.  

- **Separate web apps from web APIs**. If your solution has both a web front-end and a web API, consider decomposing them into separate App Service apps. This design makes it easier to decompose the solution by workload. You can run the web app and the API in separate App Service plans, so they can be scaled independently. If you don't need that level of scalability at first, you can deploy the apps into the same plan, and move them into separate plans later, if needed.

- **Avoid using the App Service backup feature to back up Azure SQL databases.** Instead, use [SQL Database automated backups](SQL Database backups). App Service backup exports the database to a SQL .bacpac file, which costs DTUs.  

- **Deploy to a staging slot.** Create a deployment slot for staging. Deploy application updates to the staging slot, and verify the deployment before swapping it into production. This reduces the chance of a bad update in production. It also ensures that all instances are warmed up before being swapped into production. Many applications have a significant warmup and cold-start time. For more information, see [Set up staging environments for web apps in Azure App Service](../app-service-web/web-sites-staged-publishing.md). 

-  **Create a deployment slot to hold the last-known-good (LKG) deployment.** When you deploy an update to production, move the previous production deployment into the LKG slot. This makes it easier to roll back a bad deployment. If you discover a problem later, you can quickly revert to the LKG version. For more information, see [Azure reference architecture: Basic web application](guidance-web-apps-basic.md). 

- **Enable diagnostics logging**, including application logging and web server logging. Logging is important for monitoring and diagnostics. See [Enable diagnostics logging for web apps in Azure App Service](../app-service-web/web-sites-enable-diagnostic-log.md)

- **Log to blob storage**. This makes it easier to collect and analyze the data. 

- **Create a separate storage account for logs.** Don't use the same storage account for logs and application data. This helps to prevent logging from reducing application performance. 

- **Monitor performance.** Use a performance monitoring service such as [New Relic](http://newrelic.com/) or [Application Insights](../application-insights/app-insights-overview.md) to monitor application performance and behavior under load.  Performance monitoring gives you real-time insight into the application. It enables you to diagnose issues and perform root-cause analysis of failures. 


###  Application Gateway 

- **Provision at least two instances.** Deploy Application Gateway with at least two instances. A single instance is a single point of failure. Use two or more instances for redundancy and scalability. In order to qualify for the [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/), you must provision two or more medium or larger instances. 

### Azure Search

- **Provision more than one replica.** Use at least two replicas for read high-availability, or three for read-write high-availability.

- **Configure indexers for multi-region deployments.** If you have a multi-region deployment, consider your options for continuity in indexing.

    - If the data source is geo-replicated, point each indexer of each regional Azure Search service to its local data source replica.  

    - If the data source is not geo-replicated, point multiple indexers at the same data source, so that Azure Search services in multiple regions continuously and independently index from the data source. For more information, see [Azure Search performance and optimization considerations][search-optimization].

###  Azure Storage 

- **For application data, use read-access geo-redundant storage (RA-GRS).** RA-GRS storage replicates the data to a secondary region, and provides read-only access from the secondary region. If there is a storage outage in the primary region, the application can read the data from the secondary region. For more information, see [Azure Storage replication](../storage/storage-redundancy.md).

- **For VM disks, use Premium Storage** For more information, see [Premium Storage: High-Performance Storage for Azure Virtual Machine Workloads](../storage/storage-premium-storage.md).

- **For Queue storage, create a backup queue in another region.** For Queue storage, a read-only replica has limited use, because you can't queue or dequeue items. Instead, create a backup queue in a storage account in another region. If there is a storage outage, the application can use the backup queue, until the primary region becomes available again. That way, the application can still process new requests.  

###  DocumentDB 

- **Replicate the database across regions.** With a multi-region account, your DocumentDB database has one write region and multiple read regions. If there is a failure in the write region, you can read from another replica. The Client SDK handles this automatically. You can also fail over the write region to another region. For more information, see [Distribute data globally with DocumentDB](../documentdb/documentdb-distribute-data-globally.md).


###  SQL Database 

- **Use Standard or Premium tier.** These tiers provide a longer point-in-time restore period (35 days). For more information, see [SQL Database options and performance](../sql-database/sql-database-service-tiers.md).

- **Enable SQL Database auditing.** Auditing can be used to diagnose malicious attacks or human error. For more information, see [Get started with SQL database auditing](../sql-database/sql-database-auditing-get-started.md). 

- **Use Active Geo-Replication** Use Active Geo-Replication to create a readable secondary in a different region.  If your primary database fails, or simply needs to be taken offline, perform a manual failover to the secondary database.  Until you fail over, the secondary database remains read-only.  For more information, see [SQL Database Active Geo-Replication](../sql-database/sql-database-geo-replication-overview.md). 

- **Use sharding**. Consider using sharding to partition the database horizontally. Sharding can provide fault isolation. For more information, see [Scaling out with Azure SQL Database](../sql-database/sql-database-elastic-scale-introduction.md). 

- **Use point-in-time restore to recover from human error.**  Point-in-time restore returns your database to an earlier point in time. For more information, see [SQL Database backups](../sql-database/sql-database-automated-backups.md).

- **Use geo-restore to recover from a service outage.** Geo-restore restores a database from a geo-redundant backup.  For more information, see [SQL Database backups](../sql-database/sql-database-business-continuity.md).


###  SQL Server (running in a VM)

- **Replicate the database.** Use SQL Server Always On Availability Groups to replicate the database. Provides high availability if one SQL Server instance fails. For more information, see  [More information...](guidance-compute-n-tier-vm.md) 

- **Back up the database**. If you are already using [Azure Backup](https://azure.microsoft.com/documentation/services/backup/) to back up your VMs, consider using [Azure Backup for SQL Server workloads using DPM](../backup/backup-azure-backup-sql.md). With this approach, there is one backup administrator role for the organization and a unified recovery procedure for VMs and SQL Server. Otherwise, use [SQL Server Managed Backup to Microsoft Azure](https://msdn.microsoft.com/library/dn449496.aspx). 


###  Traffic Manager 

- **Perform manual failback.** After a Traffic Manager failover, perform manual failback, rather than automatically failing back. Before failing back, verify that all application subsystems are healthy.  Otherwise, you can create a situation where the application flips back and forth between data centers. For more information, see [Running VMs in multiple regions](guidance-compute-multiple-datacenters.md).

- **Create a health probe endpoint**. Create a custom endpoint that reports on the overall health of the application. This enables Traffic Manager to fail over if any critical path fails, not just the front end. The endpoint should return an HTTP error code if any critical dependency is unhealthy or unreachable. Don't report errors for non-critical services, however. Otherwise, the health probe might trigger failover when it's not needed, creating false positives. For more information, see [Traffic Manager endpoint monitoring and failover]../traffic-manager/traffic-manager-monitoring.md).


###  Virtual Machines 

- **Avoid running a production workload on a single VM.** A single VM deployment is not resilient to planned or unplanned maintenance. . Instead, put multiple VMs in an availability set or VM scale set, with a load balancer in front. In order to qualify for the SLA, at least two Virtual Machines must be deployed within the same availability set. For more information, see [Virtual Machine Scale Sets Overview](../virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md). 

- **Specify the availability set when you provision the VM.** Currently, there is no way to add a Resource Manager VM to an availability set after the VM is provisioned. When you add a new VM to an existing availability set, make sure to create a NIC for the VM, and add the NIC to the back-end address pool on the load balancer. Otherwise, the load balancer won't route network traffic to that VM. 

- **Put each application tier into a separate Availability Set.** In an N-tier application, don't put VMs from different tiers into the same availability set. VMs in an availability set are placed across fault domains (FDs) and update domains (UD). However, to get the redundancy benefit of FDs and UDs, every VM in the availability set must be able to handle the same client requests. 

- **Choose the right VM size based on performance requirements.** When moving an existing workload to Azure, start with the VM size that's the closest match to your on-premise servers. Then measure the performance of your actual workload with respect to CPU, memory, and disk IOPS, and adjust the size if needed. This helps to ensure the application behaves as expected in a cloud environment. Also, if you need multiple NICs, be aware of the NIC limit for each size. 

- **Use premium storage for VHDs.** Azure Premium Storage provides high-performance, low-latency disk support. For more information, see [Premium Storage: High-Performance Storage for Azure Virtual Machine Workloads](../storage/storage-premium-storage.md) Choose a VM size that supports premium storage. 

- **Create a separate storage account for each VM.** Place the VHDs for one VM into a separate storage account. This helps to avoid hitting the IOPS limits for storage accounts. For more information, see [Azure Storage Scalability and Performance Targets](../storage/storage-scalability-targets.md). However, if you are deploying a large number of VMs, be aware of the per-subscription limit for storage accounts. See [Storage limits](../azure-subscription-service-limits.md#storage-limits).

- **Create a separate storage account for diagnostic logs**. Don't write diagnostic logs to the same storage account as the VHDs, to avoid having the diagnostic logging affect the IOPS for the VM disks. A standard locally redundant storage (LRS) account is sufficient for diagnostic logs. 

- **Install applications on a data disk, not the OS disk.** Otherwise, you may reach the disk size limit. 

- **Use Azure Backup to back up VMs.** Backups protect against accidental data loss. For more information, see [Protect Azure VMs with a recovery services vault](../backup/backup-azure-vms-first-look-arm.md).

- **Enable diagnostic logs**, including basic health metrics, infrastructure logs, and [boot diagnostics][boot-diagnostics]. Boot diagnostics can help you diagnose a boot failure if your VM gets into a non-bootable state. For more information, see [Overview of Azure Diagnostic Logs][diagnostics-logs].

- **Use the AzureLogCollector extension**. (Windows VMs only.) This extension aggregates Azure platform logs and uploads them to Azure storage, without the operator remotely logging into the VM. For more information, see [AzureLogCollector Extension](../virtual-machines/virtual-machines-windows-log-collector-extension.md).


###  Virtual Network 

- **To whitelist or block public IP addresses, add an NSG to the subnet.** Block access from malicious users, or allow access only from users who have privilege to access the application.  

- **Create a custom health probe.** Load Balancer Health Probes can test either HTTP or TCP. If a VM runs an HTTP server, the HTTP probe is a better indicator of health status than a TCP probe. For an HTTP probe, use a custom endpoint that reports the overall health of the application, including all critical dependencies. For more information, see [Azure Load Balancer overview](../load-balancer/load-balancer-overview.md).

- **Don't block the health probe.** The Load Balancer Health probe is sent from a known IP address, 168.63.129.16. Don't block traffic to or from this IP in any firewall policies or network security group (NSG) rules. Blocking the health probe would cause the load balancer to remove the VM from rotation. 

- **Enable Load Balancer logging.** The logs show how many VMs on the back-end are not receiving network traffic due to failed probe responses. For more information, see [Log analytics for Azure Load Balancer](../load-balancer/load-balancer-monitor-log.md).


<!-- links -->
[app-service-autoscale]: ../monitoring-and-diagnostics/insights-how-to-scale.md
[asynchronous-c-sharp]:https://msdn.microsoft.com/library/mt674882.aspx
[availability-sets]:../virtual-machines/virtual-machines-windows-manage-availability.md
[azure-backup]: https://azure.microsoft.com/documentation/services/backup/
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[circuit-breaker]: https://msdn.microsoft.com/library/dn589784.aspx
[cloud-service-autoscale]: ../cloud-services/cloud-services-how-to-scale.md
[diagnostics-logs]: ../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md
[fma]: guidance-resiliency-failure-mode-analysis.md
[guidance-resilient-deployment]: guidance-resiliency-overview.md#resilient-deployment
[monitoring-and-diagnostics-guidance]: ../best-practices-monitoring.md
[resource-manager]: ../azure-resource-manager/resource-group-overview.md
[retry-pattern]: https://msdn.microsoft.com/library/dn589788.aspx
[retry-service-guidance]: ../best-practices-retry-service-specific.md
[search-optimization]: ../search/search-performance-optimization.md
[vmss-autoscale]: ../virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview.md
