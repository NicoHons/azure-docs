---
title: Pricing tiers - Azure Database for PostgreSQL - Single Server
description: This article describes the compute and storage options in Azure Database for PostgreSQL - Single Server.
ms.service: postgresql
ms.subservice: single-server
ms.topic: conceptual
ms.author: sunila
author: sunilagarwal
ms.date: 10/14/2020
ms.custom: references_regions
---

# Pricing tiers in Azure Database for PostgreSQL - Single Server

[!INCLUDE [applies-to-postgresql-single-server](../includes/applies-to-postgresql-single-server.md)]

You can create an Azure Database for PostgreSQL server in one of three different pricing tiers: Basic, General Purpose, and Memory Optimized. The pricing tiers are differentiated by the amount of compute in vCores that can be provisioned, memory per vCore, and the storage technology used to store the data. All resources are provisioned at the PostgreSQL server level. A server can have one or many databases.

| Resource / Tier | **Basic** | **General Purpose** | **Memory Optimized** |
|:---|:----------|:--------------------|:---------------------|
| Compute generation | Gen 4, Gen 5 | Gen 4, Gen 5 | Gen 5 |
| vCores | 1, 2 | 2, 4, 8, 16, 32, 64 |2, 4, 8, 16, 32 |
| Memory per vCore | 2 GB | 5 GB | 10 GB |
| Storage size | 5 GB to 1 TB | 5 GB to 16 TB | 5 GB to 16 TB |
| Database backup retention period | 7 to 35 days | 7 to 35 days | 7 to 35 days |

To choose a pricing tier, use the following table as a starting point.

| Pricing tier | Target workloads |
|:-------------|:-----------------|
| Basic | Workloads that require light compute and I/O performance. Examples include servers used for development or testing or small-scale infrequently used applications. |
| General Purpose | Most business workloads that require balanced compute and memory with scalable I/O throughput. Examples include servers for hosting web and mobile apps and other enterprise applications.|
| Memory Optimized | High-performance database workloads that require in-memory performance for faster transaction processing and higher concurrency. Examples include servers for processing real-time data and high-performance transactional or analytical apps.|

After you create a server, the number of vCores, hardware generation, and pricing tier (except to and from Basic) can be changed up or down within seconds. You also can independently adjust the amount of storage up and the backup retention period up or down with no application downtime. You can't change the backup storage type after a server is created. For more information, see the [Scale resources](#scale-resources) section.

## Compute generations and vCores

Compute resources are provided as vCores, which represent the logical CPU of the underlying hardware. China East 1, China North 1, US DoD Central, and US DoD East utilize Gen 4 logical CPUs that are based on Intel E5-2673 v3 (Haswell) 2.4-GHz processors. All other regions utilize Gen 5 logical CPUs that are based on Intel E5-2673 v4 (Broadwell) 2.3-GHz processors.

## Storage

The storage you provision is the amount of storage capacity available to your Azure Database for PostgreSQL server. The storage is used for the database files, temporary files, transaction logs, and the PostgreSQL server logs. The total amount of storage you provision also defines the I/O capacity available to your server.

| Storage attributes | **Basic** | **General Purpose** | **Memory Optimized** |
|:---|:----------|:--------------------|:---------------------|
| Storage type | Basic Storage | General Purpose Storage | General Purpose Storage |
| Storage size | 5 GB to 1 TB | 5 GB to 16 TB | 5 GB to 16 TB |
| Storage increment size | 1 GB | 1 GB | 1 GB |
| IOPS | Variable |3 IOPS/GB<br/>Min 100 IOPS<br/>Max 20,000 IOPS | 3 IOPS/GB<br/>Min 100 IOPS<br/>Max 20,000 IOPS |

> [!NOTE]
> Storage up to 16TB and 20,000 IOPS is supported in the following regions: Australia East, Australia South East, Brazil South, Canada Central, Canada East,  Central US, China East 2, China North 2, East Asia, East US, East US 1, East US 2, France Central, India Central, India South, Japan East, Japan West, Korea Central, Korea South, North Central US, North Europe, South Central US, Southeast Asia, Switzerland North, Switzerland West, US Gov East, US Gov SouthCentral, US Gov SouthWest,  UK South, UK West, West Europe, West Central US, West US, and West US 2.
>
> All other regions support up to 4TB of storage and 6000 IOPS.
>

You can add additional storage capacity during and after the creation of the server, and allow the system to grow storage automatically based on the storage consumption of your workload.

>[!NOTE]
> Storage can only be scaled up, not down.

The Basic tier does not provide an IOPS guarantee. In the General Purpose and Memory Optimized pricing tiers, the IOPS scale with the provisioned storage size in a 3:1 ratio.

You can monitor your I/O consumption in the Azure portal or by using Azure CLI commands. The relevant metrics to monitor are [storage limit, storage percentage, storage used, and IO percent](concepts-monitoring.md).

### Reaching the storage limit

Servers with less than equal to 100 GB provisioned storage are marked read-only if the free storage is less than 512MB or 5% of the provisioned storage size. Servers with more than 100 GB provisioned storage are marked read only when the free storage is less than 5 GB.

For example, if you have provisioned 110 GB of storage, and the actual utilization goes over 105 GB, the server is marked read-only. Alternatively, if you have provisioned 5 GB of storage, the server is marked read-only when the free storage reaches less than 512 MB.

When the server is set to read-only, all existing sessions are disconnected and uncommitted transactions are rolled back. Any subsequent write operations and transaction commits fail. All subsequent read queries will work uninterrupted.  

You can either increase the amount of provisioned storage to your server or start a new session in read-write mode and drop data to reclaim free storage. Running `SET SESSION CHARACTERISTICS AS TRANSACTION READ WRITE;` sets the current session to read write mode. In order to avoid data corruption, do not perform any write operations when the server is still in read-only status.

We recommend that you turn on storage auto-grow or to set up an alert to notify you when your server storage is approaching the threshold so you can avoid getting into the read-only state. For more information, see the documentation on [how to set up an alert](how-to-alert-on-metric.md).

### Storage auto-grow

Storage auto-grow prevents your server from running out of storage and becoming read-only. If storage auto grow is enabled, the storage automatically grows without impacting the workload. For servers with less than equal to 100 GB provisioned storage, the provisioned storage size is increased by 5 GB as soon as the free storage is below the greater of 1 GB or 10% of the provisioned storage. For servers with more than 100 GB of provisioned storage, the provisioned storage size is increased by 5% when the free storage space is below the greater of 10 GB or 5% of the provisioned storage size. Maximum storage limits as specified above apply.

For example, if you have provisioned 1000 GB of storage, and the actual utilization goes over 950 GB, the server storage size is increased to 1050 GB. Alternatively, if you have provisioned 10 GB of storage, the storage size is increase to 15 GB when less than 1 GB of storage is free.

Remember that storage can only be scaled up, not down.

## Backup storage

Azure Database for PostgreSQL provides up to 100% of your provisioned server storage as backup storage at no additional cost. Any backup storage you use in excess of this amount is charged in GB per month. For example, if you provision a server with 250 GB of storage, you’ll have 250 GB of additional storage available for server backups at no charge. Storage for backups in excess of the 250 GB is charged as per the [pricing model](https://azure.microsoft.com/pricing/details/postgresql/). To understand factors influencing backup storage usage, monitoring and controlling backup storage cost, you can refer to the [backup documentation](concepts-backup.md).

## Scale resources

After you create your server, you can independently change the vCores, the hardware generation, the pricing tier (except to and from Basic), the amount of storage, and the backup retention period. You can't change the backup storage type after a server is created. The number of vCores can be scaled up or down. The backup retention period can be scaled up or down from 7 to 35 days. The storage size can only be increased. Scaling of the resources can be done either through the portal or Azure CLI. For an example of scaling by using Azure CLI, see [Monitor and scale an Azure Database for PostgreSQL server by using Azure CLI](../scripts/sample-scale-server-up-or-down.md).

> [!NOTE]
> The storage size can only be increased. You cannot go back to a smaller storage size after the increase.

When you change the number of vCores, the hardware generation, or the pricing tier, a copy of the original server is created with the new compute allocation. After the new server is up and running, connections are switched over to the new server. During the moment when the system switches over to the new server, no new connections can be established, and all uncommitted transactions are rolled back. This window varies, but in most cases, is less than a minute.

Scaling storage and changing the backup retention period are true online operations. There is no downtime, and your application isn't affected. As IOPS scale with the size of the provisioned storage, you can increase the IOPS available to your server by scaling up storage.

## Pricing

For the most up-to-date pricing information, see the service [pricing page](https://azure.microsoft.com/pricing/details/PostgreSQL/). To see the cost for the configuration you want, the [Azure portal](https://portal.azure.com/#create/Microsoft.PostgreSQLServer) shows the monthly cost on the **Pricing tier** tab based on the options you select. If you don't have an Azure subscription, you can use the Azure pricing calculator to get an estimated price. On the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/) website, select **Add items**, expand the **Databases** category, and choose **Azure Database for PostgreSQL** to customize the options.

## Next steps

- Learn how to [create a PostgreSQL server in the portal](tutorial-design-database-using-azure-portal.md).
- Learn about [service limits](concepts-limits.md).
- Learn how to [scale out with read replicas](how-to-read-replicas-portal.md).
