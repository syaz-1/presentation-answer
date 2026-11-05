Selected cloud: **Microsoft Azure**

# Decision 1 — Require a remote copy before committing payments

## Page 1 of 2 — Decision and alternatives

**Status:** Recommended design. **Revised:** 5 November 2026. **Owners:** Payment owner, data team and platform lead.

### Decision

Use SQL Server 2022 Enterprise on Azure Virtual Machines. Keep three database computers in each approved location. Require a durable remote copy before completing every payment-record change. Target zero committed loss for one covered location failure.

I choose stronger record protection over continued posting during a broken connection. Pause posting if the remote copy cannot be reached. A waiting request must not receive a successful result.

### Why this choice

The brief says near-zero, without an allowed loss count or value. Zero loss is my proposed stricter interpretation for one location failure. It is not a stated requirement. The payment owner must approve the interpretation before the pilot.

This avoids inventing an allowance for missing payments. It adds maintenance, licences and network delay. It can reduce posting availability. Test the trade-off against both response-time and 99.99% journey targets.

### How it works

Each location has a local database recovery group. Link them with a distributed availability group. Require synchronous remote copying and set `REQUIRED_SYNCHRONIZED_SECONDARIES_TO_COMMIT=1` on that cross-location group. Local copies cannot replace the required remote copy.

A commit is a completed database change. Protect it even if the customer never received its reply. Microsoft documents this commit control for SQL Server 2022 and later. [Commit controls](https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-availability-group-transact-sql?view=sql-server-ver17)

### Rejected alternatives

Azure SQL Database has less maintenance. Its normal regional copying happens after commit and forced recovery can lose records. Prefer this simpler option if the owner accepts a measurable loss tolerance and tests support it. I reject it for the ledger only under the proposed zero-loss assumption. [Microsoft limits](https://learn.microsoft.com/en-us/azure/reliability/reliability-sql-database)

Waiting for copying only after a local commit protects some customer replies. It does not protect every completed local change. I also reject two independent writers for the same ledger. They add conflict and duplicate-payment risk.

## Page 2 of 2 — Failure behavior and proof

### Recover without two active senders

An isolated old database must block new commits because remote protection is required. Also remove its separate credentials at the bank's external-payment gateway. Close old sessions. The gateway must reject each new instruction from that location.

Stop old computers where management access works. Do not rely on that action alone. The gateway is an explicit dependency outside the failed location. Prove its independent availability and sender controls.

Then follow the documented SQL role-change procedure. Update application connections to the recovery group's listener. Check a payment before redirecting callers. The distributed group has no single application listener. Use supported local listeners; check the chosen Azure VM listener configuration. [Database layout](https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/distributed-availability-groups?view=sql-server-ver16)

### Protection after switching

After one location is lost, new posting has only local protection. Resuming requires a pre-approved, time-limited reduced-protection mode. A second location failure can then lose new payments. Restore remote protection urgently.

If this exposure is prohibited, wait for another safe remote copy. A two-location design then cannot promise full posting within 30 minutes. Approve that policy before the pilot.

### Required evidence

Run peak traffic while losing a zone, a location and the link between running locations. Count all test commits using independent records. Check that the isolated old system cannot commit or dispatch. Recover waiting messages and uncertain external payments without charging twice.

Measure detection, approval, database recovery, connection changes and full customer recovery. Pass only with zero missing protected commits and recovery within 30 minutes in the covered exercise. Test response time and a clean restore too. Results remain unmeasured.

### Limits and review triggers

This design does not promise zero loss after both copies are destroyed or corrupted. The normal protection target does not apply to new payments made with only local protection.

Review if the loss tolerance changes, or latency, cost, staffing, residency or outages make the design unsuitable. Do not silently disable protection to meet availability. The payments and risk owners approve the failure policy. Product and platform owners prove operating readiness.
