Selected cloud: **Microsoft Azure**

# Supporting decisions and delivery checks

## 1. Compare application platforms — task 3

EKS, ECS, Lambda and EC2 are the brief's reference options. Here the decision compares Azure Kubernetes Service, Azure Container Apps, Azure Functions and Azure Virtual Machines for the same components.

### Core payment API and processing

**Needs:** Continuous service, agreed peak capacity, controlled releases, private connections and a staffed support model. I also assume a need for direct Kubernetes placement and maintenance policies. Confirm that extra assumption.

| Option | Why it could fit | Why I select or reject it here |
|---|---|---|
| AKS | Direct access to Kubernetes controls. Separate node capacity and placement rules for core services. | **Select under the stated operating assumption.** The bank must support clusters, upgrades and capacity. |
| Container Apps | Managed container hosting with scaling and version-based traffic changes. | **Credible alternative.** Prefer it if no direct Kubernetes control is needed. It is not rejected because payments are important. |
| Functions | Supports request- and event-driven code. Suitable plans can keep instances ready. | **Not the first choice here.** The proposed core service is a continuously running container application. Function-specific execution and scaling need separate validation. |
| Virtual Machines | Full OS control and support for existing software. | **Not preferred for new core code.** It gives the bank more patching and scaling work without a demonstrated OS requirement. |

Microsoft distinguishes direct Kubernetes access from managed container hosting. Both options can support releases and scaling. Those service facts support the comparison, but the bank's actual control requirements decide the choice. [AKS and Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/compare-options)

### Other representative components

| Component and need | AKS | Container Apps | Functions | Virtual Machines | Decision |
|---|---|---|---|---|---|
| Reconciliation: container job comparing records on a schedule | Can run the job on the shared cluster | Has scheduled and event-triggered jobs | Fits bounded checks; long jobs need further design | Can schedule the job, but needs OS support | **Container Apps**, because this job needs no custom cluster controls. |
| Notifications: short work after a payment event | Can run a consumer | Can run a container handler | Fits the trigger-based workload | Can run a process, with extra maintenance | **Functions**, with a plan chosen for private access, waiting time and cost. |
| Legacy adapter: vendor-supported OS installation | Requires supported container packaging | Requires supported container packaging | Requires a compatible redesign | Keeps the supported installation | **Virtual Machines** initially. The vendor constraint must be confirmed. |

These are proposed workload shapes, not discovered bank facts. Reassess the choices after inventory. Start with only the services needed by the first migration wave.

### Operating consequences and proof

AKS has a continuing cluster-support cost. Container Apps removes more cluster work, while the bank still owns application correctness and capacity settings. Functions cost and behavior depend on the plan and workload. Virtual Machines leave more maintenance with the bank. No option is universally cheapest.

For core services, test peak load during zone loss, an upgrade and a bad release. For workers, test restart and replay. For Functions, check the selected plan's execution, scaling and private-connection behavior. For legacy VMs, prove vendor support and recovery. [Azure compute selection](https://learn.microsoft.com/en-us/azure/architecture/guide/technology-choices/compute-decision-tree), [Functions hosting plans](https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale)

### Portability: what stays and what changes

| Area | Keep reusable | Azure-specific work when moving | Proof |
|---|---|---|---|
| Application patterns | Payment rules, stable references, duplicate protection and retry rules | Replace Service Bus and identity connection code behind small interfaces | Run the same payment tests with replacement connections. |
| APIs | Versioned request and event formats; clear errors and payment status meanings | Recreate API Management policies and access checks | A test client works without changing the payment contract. |
| Data | Documented record meaning, export format and reconciliation rules | Migrate SQL Server-specific queries, procedures and copy settings | Export a sample, restore it on the proposed target and compare balances. |
| Automation | Repeatable build steps, approved software packages and release checks | Rewrite cloud resource definitions, permissions and network setup | Build and deploy a representative service in the target environment. |

Containers help package code. They do not automatically move databases, permissions or cloud automation. The selected SQL Server recovery design is a known dependency. Portability here means controlled replacement work, not an unchanged whole-platform move.

## 2. Protect payments and recover — task 4

**Record integrity:** Save the payment, accounting changes, duplicate-request check and next work item together. Either all complete or none is treated as complete. Reject the same reference with a different amount. Delivery can repeat; receivers must recognize prior processing.

**Replication:** Use SQL Server 2022 Enterprise on Azure Virtual Machines. Each location has a protected local database group across zones. Link the groups using a distributed availability group. Require a durable remote copy before completing a change. The cross-location setting `REQUIRED_SYNCHRONIZED_SECONDARIES_TO_COMMIT=1` enforces that requirement. Selecting synchronous copying alone is insufficient. The bank owns database and operating-system maintenance. [Microsoft commit controls](https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-availability-group-transact-sql?view=sql-server-ver17)

**Failure limits:** Zero loss is the proposed stricter interpretation of near-zero, pending payment-owner approval. Target zero committed loss after one covered location fails while required protection was active. New posting blocks when the required remote copy is unavailable. Simultaneous loss of both copies, copied corruption and new payments made under reduced protection do not carry that same claim. Latency and outage tests must also support the 99.99% journey target.

| Recovery stage | Required action | Owner |
|---|---|---|
| Detect and decide | Start the clock at customer impact. Confirm the failure and approved recovery mode. | Product on-call and incident leader |
| Prevent conflicting work | Check that isolated old commits block. Revoke the old location's separate gateway credentials and close sessions. | Data team and gateway owner |
| Activate recovery | Follow the supported SQL role-change procedure. Point applications at the recovery group's database connection address. | Data and platform teams |
| Resume safely | Recover messages, test a payment and then redirect callers. | Product and platform teams |
| Check and restore protection | Compare records and external results. Rebuild the remote copy before returning to normal operation. | Payment operations and data team |

Budget the full sequence within 30 minutes. The bank gateway must be independent of the failed location and reject old senders immediately. Its capability is an assumption to prove. New posting with only local protection needs prior approval. If prohibited, wait for another remote copy; this can exceed 30 minutes.

**Reconciliation:** If payment P-104 for RM100 succeeds externally but its reply is lost, check P-104's status. Do not create a new payment. Compare bank records, external confirmations and settlement reports. Resolve differences through approved correction entries.

**Messages:** Use Service Bus Premium data Geo-Replication. Message copies may lag. Retain work records in the protected ledger so missing messages can be rebuilt. Replayed messages must keep their original references. Copying only message-service settings is insufficient. [Microsoft recovery behavior](https://learn.microsoft.com/en-us/azure/reliability/reliability-service-bus)

**Backups and cyber-recovery:** Use Azure Backup with approved retention and locked Recovery Services vault protection. Validate support and restore behavior for the final database-group layout. Use the supported database-backup path, rather than assuming a preview snapshot option fits. Keep key recovery and recovery administrators separate. Restore clean records into an isolated subscription after an attack. Reconcile later legitimate payments. Live replication can copy damage, so it does not replace clean backups. Clean recovery needs separate targets. [Backup guidance](https://learn.microsoft.com/en-us/azure/backup/backup-azure-sql-database), [vault protection](https://learn.microsoft.com/en-us/azure/backup/backup-azure-immutable-vault-concept)

**Proof:** Practice location recovery quarterly and sample restores monthly. Include zone loss, broken links with both locations alive, repeated messages and lost external replies. Record missing commits, balances, full recovery time and corrective actions. These are proposed checks, not completed results.

## 3. Threat-control matrix — task 5

| Threat | Prevent | Detect | Respond | Owner |
|---|---|---|---|---|
| Stolen powerful account | Entra stronger sign-in checks; approved, time-limited privileged roles | Sentinel alerts on elevation and unusual changes | Revoke access; review changes | Access team and security |
| Access to another entity | Separate subscriptions, application identities, records and keys | Denied-access tests and access reviews | Block the path; investigate disclosure | Platform and product |
| False or repeated payment | Validate caller and payment; enforce lasting references | Fraud alerts and record comparison | Hold suspicious payments; resolve the original reference | Payments and fraud |
| Compromised software release | Reviewed changes, checked dependencies and verified build signatures | Defender findings and unapproved-release alerts | Block the version; rebuild from trusted software | Engineering and security |
| Both locations send payments | Required remote commits; separate sender identities | Copy-rule alerts and gateway sender checks | Reject old identity; pause unsafe sending | Data and gateway owners |
| Data leaves approved locations | Azure Policy, private access and restricted outgoing connections | Firewall records and location checks | Stop transfer; follow country response rules | Country risk and security |
| Deleted or corrupted records | Protected Azure Backup retention and recoverable keys | Deletion alerts, backup checks and balance checks | Isolate damage; restore a clean copy | Data and security |
| Harmful requests or exhausted capacity | Application Gateway WAF, API Management limits and spare capacity | Azure Monitor journey and workload alerts | Block abuse; restore safe capacity | Product and platform |

Keep sensitive payment contents out of activity records. Store required evidence in approved locations with protected retention. Test controls in both locations. A service name alone does not prove a control works.

## 4. Migration roadmap — task 6

| Wave | Work and dependencies | Exit check | Return point |
|---|---|---|---|
| Discover | Confirm locations, current engine, gateway controls, volumes and owners. | Risk and payment owners agree failure scope and recovery policy. | Current system stays unchanged. |
| Prepare | Build isolated subscriptions, protected connections, monitoring and both-location capacity. Needs network providers and platform staff. | Access, route and recovery-copy tests pass. | Fix the foundation before moving work. |
| Lower-risk move | Move reports and reconciliation. Test existing bank and AWS contracts. | Results match; support and cost are understood. | Route these functions back. |
| Payment pilot | Select one entity and a safely separable payment group. Needs vendor approval, rail test windows and payment staff. | Balances match; peak-load, duplicate, old-sender and timed recovery tests pass. | Before new writes, return directly. After new writes, pause and reconcile before returning. |
| Expand and retire | Increase approved groups after observation. Remove unused access and connections. | Each group meets service targets. Required history remains accessible. | Stop expansion. Keep one official writer for each group. |

Old and new systems coexist through documented requests and messages. Comparison testing must suppress real external sending. Avoid splitting the same account balance between independent ledgers.

Give every temporary connection, exception and old application an owner, removal date and cost. Review them at each wave. A return to older records must not erase valid new payments.

## 5. Risk register

| Risk | Impact | Action and review trigger | Owner |
|---|---|---|---|
| Required remote copying is unavailable or misconfigured | Critical | Block unsafe commits. Reopen design after a protection failure. | Data owner |
| Cross-location waiting breaks response or availability targets | High | Test actual approved locations. Review any missed target. | Product and platform |
| A second location fails during reduced-protection operation | Critical | Pre-approve or prohibit the mode. Restore remote protection urgently. | Payments and country risk |
| Recovery gateway cannot reject the old sender | Critical | Prove immediate rejection before the pilot. Otherwise keep dispatch paused. | Gateway owner |
| Required services or copies are not approved in a country | Critical | Complete the service and location review before deployment. | Country risk |
| Staffing, vendor limits or duplicated capacity delay the move or raise cost | High | Confirm support and compatibility. Price licences, both locations, networks and retained copies. | Product, vendor and finance owners |

Cost reports should show spending by entity and cost per successful payment. Include recovery capacity and support. No savings estimate is claimed without volumes and current prices.
