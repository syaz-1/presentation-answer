Selected cloud: **Microsoft Azure**

# Payment platform modernization

**Targets:** 99.99% availability. Recover within 30 minutes. Near-zero loss of committed payment records.

**Flow:** Customer entry → payment services → official records → safe remote copy. Repeat the isolated design per entity.

## 1. Problem, ownership and design

Move payments safely across countries. Separate each legal entity’s subscriptions, networks, records and keys. Keep existing bank and AWS connections during the move.

Platform teams own shared rules. Product teams own applications and service targets. Payment and country-risk owners approve data locations and recovery exposure. Record exceptions with owners and expiry dates.

## 2. Protect records and recover

Use SQL Server on Azure VMs. Require a safe remote copy before completing a payment-record change. Save payment, accounting, duplicate checks and next work together. Retry the same reference without charging twice.

Zero committed loss for one covered location failure is a proposed stricter interpretation of near-zero. It costs more and can pause posting when the remote copy is unreachable. Reassess simpler Azure SQL Database if an acceptable loss is agreed. [2]

Detect failure, reject the old sender at an independent bank gateway, activate protected records, test a payment and redirect callers. Target 30 minutes end to end. Resuming with only local protection needs prior approval; otherwise recovery may take longer.

Keep protected Azure Backup copies. Restore clean records after an attack. Reconcile uncertain payments against external confirmations. Neither both-location loss nor copied corruption carries the zero-loss target.

## 3. Security and operating controls

Stolen privileged access: enforce strong Entra sign-in and time-limited roles; detect unusual elevation in Sentinel; revoke access and investigate.

Harmful releases: review and verify software; detect unapproved versions; block and rebuild. Cross-entity access: isolate permissions; test denied access; block and investigate.

Use ExpressRoute and Firewall for controlled bank connections, Key Vault for secrets, and Azure Monitor for payment journeys. Private connectivity still needs data protection.

## 4. Compare platforms by component

Azure options address EKS, ECS, Lambda and EC2 reference workloads. They are not exact feature copies. [1]

Core payments: choose AKS only if direct Kubernetes placement and operating policies are needed. Container Apps is the simpler alternative. Functions needs a suitable execution design; VMs add OS maintenance.

Reconciliation: choose Container Apps jobs. AKS also fits but adds cluster dependency. Functions suits bounded checks. VMs need more maintenance.

Notifications: choose Functions for short event work. AKS and Container Apps can host handlers. VMs add little value for the proposed task.

Legacy OS-bound adapter: choose VMs initially. AKS and Container Apps need vendor-supported containers. Functions needs redesign. Confirm the vendor constraint.

## 5. Portability and release

Application: separate payment rules from Azure connections. APIs: keep versioned request and event contracts. Data: test export and identify SQL-specific features. Automation: keep repeatable builds; rewrite provider-specific setup when moving.

Promote approved versions with Azure DevOps and Container Registry. Test the same payment behavior with replacement cloud connections. Containers alone do not prove portability.

## 6. Migration, risk and acceptance

Discover > prepare > move reports > pilot one payment group > expand > retire. Each wave needs matching balances, support readiness and agreed service checks. Keep one official writer during coexistence.

Before new writes, return directly. Afterwards, pause and reconcile new payments before returning. Give temporary systems removal dates and owners.

Main risks: remote-link delays, unapproved locations, weak gateway controls and missing support skills. Track both-location capacity, licences and cost per successful payment.

Test peak traffic, zone loss, location loss, duplicate delivery and clean restore. Count missing commits and full recovery time. Locations, volumes and compatibility remain unconfirmed. No operating results are claimed.

## Official sources

[1. Azure hosting comparison](https://learn.microsoft.com/en-us/azure/container-apps/compare-options)
[2. Database-copy controls](https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-availability-group-transact-sql?view=sql-server-ver17)
[3. Azure SQL recovery limits](https://learn.microsoft.com/en-us/azure/reliability/reliability-sql-database)

Detailed evidence: [diagrams](diagrams.md), [decision record](decision-record.pdf), [supporting tables](appendix.md) and [sources](sources.md).
