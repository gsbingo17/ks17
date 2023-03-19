# Change Stream

## Change Stream

#### What is Spanner Change Stream?

When you want to detect changes to the spanner instances, you can create a change stream object via a DDL. It will then keep track of the CRUD changes and consumed by either directly through API, the Dataflow beam.io or a Debezium-based Kafka connector. It is useful for event driven application, real time data analytics and even database migration.

#### How is this implemented?

It is quite simple: when there is a DML committed to the split, the changes are also populated to the change stream data storage. Then the API or the sdk query against the change stream data storage to retrieve the changes.

A separate metadata database is used to keep track of the change stream states. It also allows the change stream objects to keep track of the underlying schema changes. It can be stored on the same instance as the original data. However, a more recommended way is to store it in a separate instance to isolate the compute resources for easy scaling.

#### How to get started?

It can created by a [DDL](https://cloud.google.com/spanner/docs/change-streams/manage#watch-entire-db) to specify which table and which columns are to be tracked. It is also accessible via the console in the [context menu](https://cloud.google.com/spanner/docs/change-streams/manage#view-change-streams).

Then, there are 3 ways to read the change streams:

* API (not recommended unless you are crystal clear on what you are doing)
* [Dataflow by Spanner IO and write directly to BigQuery, PubSub, or other sinks](https://cloud.google.com/spanner/docs/change-streams/use-dataflow) (recommended)
* Kafka Connector (if your organisation requires)

For Dataflow, it is made extra simple because we have official template to enable one click deployment, making it a good starting point. It also handles gracefully with the metastore integration, Spanner autoscaling, etc. Even if you are not comfortable with Apache Beam Programming, you can just leave the template unchanged and just use the ELT pattern to transform the data at a later stage of the pipeline.
