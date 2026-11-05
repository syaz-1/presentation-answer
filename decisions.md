Selected cloud: **Microsoft Azure**

# Enterprise decision sheet

The purpose is to resolve business problems before debating products. Each decision has a clear owner and a reason to revisit it.

| ID / area | Problem and decision | Why this fits | Alternative and cost | Proof / review trigger |
|---|---|---|---|---|
| **B1 — Business** | Country entities need control. Give each legal entity separate live systems and an accountable payment owner. | Makes service, risk and cost responsibility clear. | A shared regional system is cheaper but gives mistakes a wider reach. | Test cross-entity access. Review after a country-rule change. |
| **D1 — Data** | I propose zero committed loss for one covered failure as a stricter interpretation of near-zero. Require a safe remote copy before completing record changes. | Protects completed changes, including those with a lost customer reply. | Copying later is faster but may lose committed records. The selected design pauses posting during some connection failures. | Disconnect the remote copy. New protected commits must stop. Review if delay or downtime breaks agreed targets. |
| **A1 — Applications** | Teams need change without conflicting payments. Separate payment rules, external connections and record comparison. Keep one official writer. | Clear responsibilities make releases and fault diagnosis easier. | Two independent ledger writers create more conflict handling. A single large application is simpler initially but harder to change in parts. | Retry one payment reference. It must cause one financial effect. Review when payment flows or ownership change. |
| **T1 — Technology** | Assume core services need direct Kubernetes placement and maintenance controls. Select AKS if that need is confirmed. | Provides direct access to the proposed cluster-level controls. | Compare all four options for each workload. Prefer Container Apps without custom cluster needs, Functions for event handlers, and VMs for OS-bound software. | Lose one zone and reverse a bad release. Review if cluster operations cost more than the controls justify. |
| **G1 — Governance** | Fast delivery can weaken controls. Central teams own shared rules; product teams own applications and service targets. | Keeps responsibility with teams able to act. | Approving every change centrally slows delivery. Giving every team unrestricted access increases risk. | Check release permissions and named alert owners. Review repeated exceptions. |
| **R1 — Recovery policy** | After losing a location, full remote protection is unavailable. Pre-agree whether new posting may run with local protection only. | Makes the recovery decision explicit before an incident. | Waiting for a new remote copy protects more strongly but may exceed 30 minutes. | Timed recovery exercise and visible protection status. Review any missed target or unapproved mode change. |

B1 and R1 need business and country-risk approval. D1 belongs to the data owner. A1 belongs to the product lead. T1 belongs to the platform lead. G1 is shared by platform, product and security leadership..

