Selected cloud: **Microsoft Azure**

# Official sources

Checked on 5 November 2026. These are official Microsoft references for the Azure target and its SQL Server component. No third-party framework sources are used.

The documents confirm service capabilities and limits. Workload placement, the proposed zero-loss interpretation, entity boundaries and approval rules are our design recommendations. The sources do not prove that the bank meets 99.99% availability or 30-minute recovery.

| Source | What it supports |
|---|---|
| [Azure compute selection](https://learn.microsoft.com/en-us/azure/architecture/guide/technology-choices/compute-decision-tree) | Choosing a hosting model based on workload and operating responsibility. |
| [AKS versus managed containers](https://learn.microsoft.com/en-us/azure/container-apps/compare-options) | Direct Kubernetes access versus managed applications and jobs. |
| [Microservice compute comparison](https://learn.microsoft.com/en-us/azure/architecture/microservices/design/compute-options) | Differences in scaling, releases and application hosting. |
| [Functions hosting plans](https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale) | Execution limits, ready capacity and network support depend on the plan. |
| [AKS baseline](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/containers/aks/baseline-aks) | Private cluster, network, zone and operating considerations. |
| [Azure shared foundations](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/) | Shared platform setup and application-team responsibilities. |
| [Azure SQL Database reliability](https://learn.microsoft.com/en-us/azure/reliability/reliability-sql-database) | Zone protection differs from asynchronous regional copying. |
| [SQL Server required-copy setting](https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-availability-group-transact-sql?view=sql-server-ver17) | Explicit commit requirements; distributed-group support starts with SQL Server 2022. |
| [Distributed database groups](https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/distributed-availability-groups?view=sql-server-ver16) | Two local groups and the absence of one shared application listener. |
| [Distributed-group recovery](https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/configure-distributed-availability-groups?view=sql-server-ver17) | Supported setup and manual recovery steps. |
| [SQL Server recovery on Azure VMs](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/hadr-cluster-best-practices?view=azuresql) | Local voting, zones and supported database connections. |
| [Service Bus reliability](https://learn.microsoft.com/en-us/azure/reliability/reliability-service-bus) | Repeated delivery, data replication and metadata-only recovery are different concerns. |
| [ExpressRoute protection](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-about-encryption) | Private connectivity is not encrypted by default. |
| [API Management reliability](https://learn.microsoft.com/en-us/azure/reliability/reliability-api-management) | Zone and region support varies by tier. |
| [SQL Server backups on Azure VMs](https://learn.microsoft.com/en-us/azure/backup/backup-azure-sql-database) | Saved database copies and restore behavior. |
| [SQL backup support matrix](https://learn.microsoft.com/en-us/azure/backup/sql-support-matrix) | Check operating system, database version and deployment support. |
| [Protected backup vaults](https://learn.microsoft.com/en-us/azure/backup/backup-azure-immutable-vault-concept) | Locked protection and its scope. |

Before building, verify regional service availability, approved data locations, versions and commercial terms again. Neither a feature description nor a service-level promise replaces a whole-payment-journey test.
