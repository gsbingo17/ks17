---
description: 内容仅供参考，请以官方文档为准
---

# 常见售后问题

### Issue: Cluster is operating sluggishly <a href="#issue-cluster-is-operating-sluggishly" id="issue-cluster-is-operating-sluggishly"></a>

Have the customer provide the following metrics for the past eight hours; these will help show spikes in workload:

1. Opcounters - These show commands, queries, updates, deletes, getmores and inserts. If any of these show a large increase, it can help explain the sluggishness, such as through an increase of workload.
2. Document Metrics - These show how many documents were returned, inserted and updated. Usually if we see a large amount of returns, or inserts, this also shows a large increase of workload.
3. Memory - If memory is low, it can indicate resource contention, also in line with workload spikes.
4. Page Faults - If page faults are occuring, it is also a strong indication of memory/resource contention.
5. Disk IOPS - IOPS spiking indicates a workload change as well.
6. Have them verify if their workload has changed, or if they are getting an abnormal increase.

**Resolution**

Either they will need to limit their workload, or to increase their cluster tier to the next level.

### Issue: How to Kill Operations in Atlas <a href="#issue-how-to-kill-operations-in-atlas" id="issue-how-to-kill-operations-in-atlas"></a>

**On M0-M5 Clusters**

For M0 (free tier) and M2, M5 (shared tier) clusters, all MongoDB users associated with an account can execute the [`db.killOp()`](https://docs.mongodb.com/manual/reference/method/db.killOp/index.html) command via the [`mongo`](https://docs.mongodb.com/manual/mongo/index.html) shell to terminate operations by all other MongoDB users associated with the same cluster. See the [Commands with Special Behavior](https://docs.atlas.mongodb.com/reference/unsupported-commands/index.html#commands-with-special-behavior) section of the MongoDB documenation for more information.

**On M10+ Clusters, Replica Set**

For M10+ clusters, an Atlas user with the role [Organization Owner](https://docs.atlas.mongodb.com/reference/user-roles/#Organization-Owner), [Project Owner](https://docs.atlas.mongodb.com/reference/user-roles/#Project-Owner) or [Project Data Access Admin](https://docs.atlas.mongodb.com/reference/user-roles/#Project-Data-Access-Admin) can kill long-running operations via the [Real Time Performance Panel (RTPP) tab in Atlas](https://docs.atlas.mongodb.com/real-time-performance-panel/index.html) OR perform the [`db.killOp()`](https://docs.mongodb.com/manual/reference/method/db.killOp/index.html) command via the [`mongo` shell](https://docs.mongodb.com/manual/mongo/index.html) to kill operations _that were started by that user_.

**On M10+ Clusters, Sharded Cluster**

Please review the [Sharded Cluster portion of the `db.killOp()` command](https://docs.mongodb.com/manual/reference/method/db.killOp/index.html#sharded-cluster) for detailed information on the specific steps to take to kill in-flight operations on sharded clusters.

### Issue: My Live Migrate attempt returned an error "Could not reach specified source" during the validation process <a href="#issue-my-live-migrate-attempt-returned-an-error-could-not-reach-specified-source-during-the-validati" id="issue-my-live-migrate-attempt-returned-an-error-could-not-reach-specified-source-during-the-validati"></a>

**Resolution**

Based on the error provided, please note that this is relevant to the source cluster. If you have SSL enabled on your source please provide the appropriate cert and toggle the "SSL" option within the Live Migrate UI. For Atlas to Atlas migrations you need only set the SSL toggle to enabled in order to proceed.

Additional information can be found in the following documentation:

* [Replica Set Migration Validation](https://docs.atlas.mongodb.com/import/live-import/#pre-migration-validation)
* [Sharded Migration Validation](https://docs.atlas.mongodb.com/import/live-import-sharded/#pre-migration-validation)
* [Atlas's Live Migrate documentation](https://docs.atlas.mongodb.com/import/live-import/index.html)
* [Troubleshooting Live Migrate Validation documentation](https://docs.atlas.mongodb.com/import/live-import-troubleshooting/#common-live-migration-validation-errors)

### Issue: My Live Migrate attempt returned an error "Username or Password provided is not correct" during the validation process <a href="#issue-my-live-migrate-attempt-returned-an-error-username-or-password-provided-is-not-correct-during" id="issue-my-live-migrate-attempt-returned-an-error-username-or-password-provided-is-not-correct-during"></a>

**Resolution**

Based on the error provided, it is important to note that Live Migrate leverages the `admin` database for authentication and authorization. As such, please ensure the user is defined on the `admin` database for the source with the appropriate permissions and a known password.

Additional information can be found in the following documentation:

* [Replica Set Migration Validation](https://docs.atlas.mongodb.com/import/live-import/#pre-migration-validation)
* [Sharded Migration Validation](https://docs.atlas.mongodb.com/import/live-import-sharded/#pre-migration-validation)
* [Atlas's Live Migrate documentation](https://docs.atlas.mongodb.com/import/live-import/index.html)
* [Troubleshooting Live Migrate Validation documentation](https://docs.atlas.mongodb.com/import/live-import-troubleshooting/#common-live-migration-validation-errors)

### Issue: My Live Migrate attempt returned an error "Other" during the validation process <a href="#issue-my-live-migrate-attempt-returned-an-error-other-during-the-validation-process" id="issue-my-live-migrate-attempt-returned-an-error-other-during-the-validation-process"></a>

**Resolution**

Please provide the customer with the following documentation and instruct them to reach out to MongoDB Support should they encounter further difficulties in migrating.

* [Replica Set Migration Validation](https://docs.atlas.mongodb.com/import/live-import/#pre-migration-validation)
* [Sharded Migration Validation](https://docs.atlas.mongodb.com/import/live-import-sharded/#pre-migration-validation)
* [Atlas's Live Migrate documentation](https://docs.atlas.mongodb.com/import/live-import/index.html)
* [Troubleshooting Live Migrate Validation documentation](https://docs.atlas.mongodb.com/import/live-import-troubleshooting/#common-live-migration-validation-errors)

### Issue: Cannot connect over Peering network <a href="#peering-troubleshooting" id="peering-troubleshooting"></a>

**Resolution**

1. Confirm the customer can connect outside of the VPC network.
   1. If they cannot, then a local connectivity issue may be happening.
   2. If they can, continue troubleshooting.
2. Have customer run the following commands (from a host within the VPC):
   1. telnet hostname:27017
      1. If the telnet is successful, and returns the private IP, they are able to successfully connect over peering.
      2. if the telnet times out, but does return the private IP, ask for the peering screenshots in step 3.
      3. If the telnet returns a public IP, successfully or not, then check the peering configuration in step 3.
3. Have customer share screenshots of the following:
   1. VPC Network
      1. Ensure the customer has linked the networks correctly (GCP -> Atlas).
      2. If they are not linked, please have customer recreate the configuration.
      3. If they are linked, continue troubleshooting.
   2. Security Section -> Network Access
      1. Firewall rules for inbound/outbound traffic (only if they are not using the default settings)
         1. Verify they are not blocking mongod 27017 inbound.
   3. VPC Network Peering
      1. Details about the connection, including the CIDR.
      2. Ensure the peering is connected to the right VPC network.
      3. Ensure the peered VPC network and peered project ID is correct from the Atlas UI.
   4. Project ID
      1. Google Project ID (my-gcp-12345)
   5. Atlas UI -> Networking -> Security
      1. Atlas whitelist for cluster access.
         1. Verify they have the proper GCP CIDR whitelisted.
         2. If whitelisted, check peering configuration.
         3. If not, have them whitelist and test via step 2.

### Issue: Questions regarding Data Transfer costs <a href="#issue-questions-regarding-data-transfer-costs" id="issue-questions-regarding-data-transfer-costs"></a>

Refer customer to the [Data Transfer section of the Billing documentation](https://docs.atlas.mongodb.com/billing/#data-transfer).

For additional information, please see the MongoDB Atlas documentation on [Billing](https://docs.atlas.mongodb.com/billing/#billing). They can also use the [Atlas Pricing Tool](https://www.mongodb.com/cloud/atlas/pricing) to estimate the costs of the particular Atlas deployment they wish to utilize for their use case.
