Selected cloud: **Microsoft Azure**

# Architecture diagrams

Each diagram answers a different question: who connects, how a payment moves, where systems run, and how recovery works.

## Context — who needs the platform?

```mermaid
flowchart LR
  channels[Customers and bank channels]
  bank[Existing bank systems]
  aws[Active AWS applications]
  rails[External payment systems]
  platform[Azure payment platform - separate legal entities]
  channels -->|Payment requests and status| platform
  bank <-->|Approved protected connection| platform
  aws <-->|Required existing connection| platform
  platform <-->|Instructions and confirmations| rails
```

## Enterprise layers — which decisions support the outcome?

![Enterprise layers and decisions](enterprise-view.svg)

Business sets the outcome. Data sets the protection rules. Applications deliver the payment journey. Technology provides the required services. Governance checks that these decisions stay connected.

## Logical payment flow — how is one payment handled?

```mermaid
flowchart LR
  channel[Customer - stable payment reference]
  check[Check caller, payment and prior reference]
  save[Save payment, accounting and next work together]
  remote[Required safe remote copy]
  deliver[Read retained work and deliver through Service Bus]
  rail[Bank gateway and external payment system]
  compare[Compare ledger, confirmations and settlement]
  channel --> check --> save
  save <-->|Required before commit| remote
  save -->|Completed result - same reference on retry| channel
  save --> deliver
  deliver <-->|Send once logically - query unknown outcomes| rail
  rail --> compare
  save --> compare
```

The stored work item is part of the payment transaction. Publishing a message is a later step. A repeated delivery must not create another charge. Use the original reference to resolve an uncertain external result.

## Deployment — where are the boundaries?

![Azure deployment, entity boundaries and failure areas](deployment-view.svg)

Solid arrows show request and access paths. Dotted arrows show copying, location selection or supporting controls. The two locations are separate failure areas. Each needs approved services and enough capacity for a zone failure.

```mermaid
flowchart TB
  caller[Customers and bank channels]
  entry[Azure Traffic Manager - select active location]
  bank[Existing bank systems and active AWS dependency]
  hub[ExpressRoute - independent backup path - Azure Firewall]
  identity[Entra ID and Azure Policy - separate rights and shared rules]
  release[Azure DevOps and Container Registry - checked releases]
  subgraph entityA[Entity A - separate live and recovery subscriptions]
    subgraph main[Approved main location - failure area 1]
      front[Application Gateway WAF v2 and private API Management Premium classic]
      subgraph mainapps[Private Azure Kubernetes Service]
        z1[Zone 1 - application copies]
        z2[Zone 2 - application copies]
        z3[Zone 3 - application copies]
      end
      ledger[SQL Server on Azure VMs - protected local group across zones]
      work[Service Bus Premium - retained work delivery]
      key[Key Vault - limited service access]
    end
    subgraph recovery[Approved recovery location - failure area 2]
      backupfront[Prepared WAF and private API Management]
      backupapps[Prepared private AKS - application copies across three zones]
      copy[SQL Server on Azure VMs - protected recovery group across zones]
      messages[Service Bus Premium - data copy]
      backupkey[Recovery Key Vault and private access]
    end
  end
  entityB[Entity B - separate subscriptions, networks, applications, records and keys]
  gateway[Independent bank gateway - only current sender allowed]
  monitor[Azure Monitor and Sentinel - approved regional records]
  saved[Azure Backup - protected saved copies]
  caller -->|Location lookup| entry
  caller -->|Protected payment request| front
  entry -.->|Return approved active entry| front
  entry -.->|Only after recovery checks| backupfront
  bank <-->|Approved protected connection| hub
  hub <-->|Controlled private path| front
  front --> mainapps
  mainapps -->|Private database address| ledger
  mainapps --> work
  mainapps --> key
  ledger -.->|Commit requires remote durability| copy
  work -.->|Data copying - replay gaps| messages
  backupfront --> backupapps
  backupapps --> copy
  backupapps --> messages
  backupapps --> backupkey
  mainapps -->|Separate regional credentials| gateway
  backupapps -->|After old sender rejected| gateway
  identity -.-> entityA
  identity -.-> entityB
  release -.-> mainapps
  release -.-> backupapps
  entityA -.->|Redacted activity and alerts| monitor
  ledger -.-> saved
  copy -.-> saved
```

Request checks and stored records have private network access. Restrict network paths and application permissions separately. Verify where Azure Firewall inspection applies; private access alone does not prove inspection. ExpressRoute is not encrypted by default. Protect sensitive traffic and test an independent backup route. [Microsoft connection guidance](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-about-encryption)

Use API Management Premium (classic) with explicit zone configuration and capacity. Keep both locations ready. AKS application copies need enough surviving-zone capacity. [API Management behavior](https://learn.microsoft.com/en-us/azure/reliability/reliability-api-management), [AKS baseline](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/containers/aks/baseline-aks)

The gateway is outside the failed location. Its sender controls are an assumption to prove. Activity records, keys and backups need approved locations too. The standalone drawing summarizes the same boundaries; the text supplies operational detail.

## Recovery — what decision does the incident leader make?

```mermaid
flowchart LR
  detect[Detect customer impact]
  isolate[Prove old commits block and reject old sender]
  approve{Safe to switch and recovery mode approved?}
  wait[Pause posting and escalate]
  activate[Activate recovery records and applications]
  verify[Test a payment and redirect callers]
  restore[Rebuild remote protection and reconcile]
  detect --> isolate --> approve
  approve -->|No| wait
  approve -->|Yes| activate --> verify --> restore
```

Target full recovery within 30 minutes. Include detection and approval time. New posting without a healthy remote copy requires an agreed temporary policy. Otherwise, wait for another safe copy. Recovery from corruption or attack follows a separate clean-restore process.
