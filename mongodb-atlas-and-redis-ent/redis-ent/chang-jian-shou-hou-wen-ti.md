# 常见售后问题

### Out of Memory <a href="#out-of-memory" id="out-of-memory"></a>

**Summary of the issue:**

Customer has reached the DB memory limit, and

1. Client receives OOM (Out-Of-Memory) error when writing data to the DB if keys cannot be evicted
2. Client keys are being evicted (if eviction policy is enabled) which could be causing increased latency on the DB

**Gather information:**

1. Find out the max memory limit on the DB from the DB configuration page in UI
2. Find out the eviction policy set for the DB.
3. Find out the current memory usage and usage trend
4. Find out the rate of eviction from the metrics/graphs, if any and compare it with latency graph
5. Find out if any keys are expiring

**Troubleshooting steps:**

Ask the customer if the application intended to have this amount of usage or it is an application bug

1. If bug then ask customer to fix it and if needed RedisLabs support can assist the customer
2. If not then explain the issue to the customer and redirect them to document explaining about eviction policies so they can select one that suits their use case. They can also simply increase the max memory limit of the DB (until 50GB with replication and 25GB without replication beyond which they will need to enable clustering on the DB) to a desired level. The customer can also set a lower expiration value for the keys so those expire fast enough and do not fill up the DB.

**Potential root cause:**

1. Customer application issue
2. Under estimation of max memory by the customer when setting up the DB initially

**Recommended solution:**

1. Ask the customer to set an appropriate eviction policy or increase the max memory limit if they do not intend to evict data.
2. Ask the customer to fix the bug in their application, if any or escalate to RedisLabs support if the customer requires assistance in fixing the bug.

### Latency <a href="#latency" id="latency"></a>

**Summary of the issue:**

The customer application experiences high response time when reading and writing data to Redis DB. At the same time, the graphs on RedisLabs dashboard or Stackdriver show high latency on the DB i.e. generally above 1 millisecond.

**Gather information:**

1. Grab the the graphs from the UI (RL dashboard or Stackdriver) to confirm the DB has latency.
2. Collect SLOWLOG from the UI or ask the customer to provide it by using the redis command **SLOWLOG GET 128**
3. Find out the keys with the largest number of elements. This can be done using the [--bigkeys option in redis-cli](https://redis.io/topics/rediscli#scanning-for-big-keys).
4. Find out keys that consume more memory. This can be done using the “--memkeys” option in redis-cli.
5. Find out the ops/sec (throughput) of the database from the UI

**Troubleshooting steps:**

After confirming the latency in UI, the first thing that should be checked is SLOWLOG entries to see if any commands are present there.

1. If there are commands in the SLOWLOG which coincide with the time of increased latency, then we need to bifurcate those commands as:
   * CPU intensive commands with higher time complexity like O(N). For e.g. KEYS, SSCAN, SMEMBERS, ZSETS, ZRANGE, ZREVRANGE, etc.
   * Non-CPU intensive commands that have lower time complexity like O(1) or similar. For e.g. GET, SET, DEL, SCAN, INCR, INFO, etc. The most obvious reason for these simple commands to show up in the SLOWLOG is High CPU usage by the redis process due to which there is a delay in processing the requests.
2. If the SLOWLOG is empty but there is latency on the DB then most likely it is an issue with the infrastructure. The issue needs to be escalated to RedisLabs support so that we can investigate it.

If there is no latency in the UI but the customer complains about timeouts or delay in being able to read/write to the DB then this is Application/Network latency which generally is out of RedisLabs’ control. It is the latency between the application and the endpoint. The latency graph on our UI shows only the latency between the shard and the endpoint of the DB( refer data plane diagram above). In such a case, to confirm the network latency between application and DB one can use the [--latency switch of redis-cli](https://redis.io/topics/rediscli#monitoring-the-latency-of-redis-instances) on the application server.

**Potential root cause:**

1. If commands are found in SLOWLOG then it could be bad redis usage or DB ops/sec limits reached.
2. If not, but there is latency on the UI then most likely it is infra issue or a bug in redis.
3. If no latency in UI then it is an issue outside of Redis, mostly network or redis client config.

**Recommended solution:**

1. If CPU intensive commands are found in SLOWLOG then we need to guide the customer on proper redis usage for which the ticket should be forwarded to RedisLabs support along with all the data gathered during the initial investigation.
2. If non-CPU intensive commands are found in SLOWLOG then the DB might have reached the maximum ops/sec achievable by the current config of the DB. For further consultation the case needs to be forwarded to RedisLabs support.
3. If none of the above 2 reasons are found, then there is a possibility of having infra issues or a bug in redis. In that case, escalate the ticket right away to RedisLabs Support.
4. If there is an application latency, then the customer needs to resolve it at their end but both Google and RedisLabs support should be able to assist the customer if requested. The customer also can verify if the application server is as close as possible (same availability zone) to the database.

### Connectivity <a href="#connectivity" id="connectivity"></a>

**Summary of the issue:**

Customer application is unable to connect to the Redis DB.

**Gather information:**

1. DB configuration from the UI.
2. Endpoint used by the customer in their application.
3. IP from where the customer is trying to connect.

**Troubleshooting steps:**

1. Check if the database has Source IP ACL enabled and if so check with the customer if they are connecting from those IP/s.
2. Check if the database has SSL enabled and if so the customer is using the correct client certificate, CA certificate and client certificate key.
3. Check if the customer is connecting to the correct public or private endpoint .

**Potential root cause:**

1. Incorrect endpoint or port number used by the customer.
2. Incorrect SSL certificate.
3. Firewall or any network security application/device.
4. Connecting from an IP which is not listed in the SourceIP ACL.
5. Underlying infra issues.

**Recommended solution:**

1. Ensure use of correct endpoint and port.
2. Ensure use of correct SSL certificates and key.
3. Ensure endpoint port or IP is not blocked on the application server by any network security software or appliance.
4. IP of the application server in question is listed in SourceIP ACL, if any.
