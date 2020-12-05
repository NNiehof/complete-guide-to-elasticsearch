# 2. Getting started

## Elasticsearch directory
### config/
* **elasticsearch.yml**: Es settings
* **JVM.options**: Java Virtual Machine options
* **log4j2.properties**: logging config

### jdk/
Java Development Kit: the Java runtime

### lib/
Dependencies such as log4j logging framework, and Apache Lucene.

### modules/
Modules with functionality (X-Pack features are here).

### plugins/
No standard plugins. Modules come shipped with Es, but plugins can be added from third party.

Note: `$ES_HOME` points to root of Es dir.

## Basic architecture
### Node
Each node stores part of data. Can spread over machines. Each node is an Es instance. Can have as many nodes as you want, without containers of VMs.

=> How is data distributed, and how does it know what is stored where?

### Cluster
Cluster is a collection of related nodes (often need just one). Might have **several clusters for different purposes**, e.g. one for performance monitoring, another one for search engine.  
Node is always part of a cluster. If you make a node without a cluster, it will create a cluster.

### Document
Json object with data. Metadata is attached when stored in Es.

### Index
Define collection of documents.

## Inspecting the cluster with Kibana
> GET /_cluster/health
`_cluster` is the API. All Es APIs are prepended with an underscore. After that, you specify the command ('health').  

List nodes (`_cat` API).
> GET /_cat/nodes?v
`?v` for verbose.

Look at your indices  
> GET /_cat/indices?v

## cURL queries
In Kibana console, you also have the option "copy as curl".

Kibana search:
> GET /csat/_search

curl search:
> curl -XGET "http://localhost:9200/csat/_search" -H 'Content-Type: application/json' -d'{  "query": {    "match_all": {}  }}'  

A header has been added by Kibana.

## Sharding and scalability
Sharding divides an index into smaller pieces. Sharding is at the index level, not node or cluster level.
Horizontally scales storage level.

Each node has a limited storage capacity. **Splitting a larger index into shards** makes it possible to store it even though it doesn't fit on an individual node.

A shard can be more or less seen as an independent index. **Each shard is a Lucene index**: an Es index consists of one or more Lucene indices.

Shard has **no predefined size**. May store up to ~ 2 billion docs. Now you can store billions of docs in a single index.

**Search query** can be performed on multiple shards **in parallel**.

If you're going to store millions of docs in your index, maybe split it into a shard or five, but generally don't go nuts with it. A single shard is often enough.

## Understanding replication
What is a node's hard drive fails? The data would be lost. These are things that keep us up at night.

### Replication
Es supports **replication** for **fault tolerance**.
* Configure at the index level.
* Primary shards and replica shards
    - visible as `pri` and `rep` when inspecting indices
* One replica set up by default at index creation.
* Replication only works with multiple nodes (replica of noda A is put in node B, and vice versa).
* Replica nodes don't need to be powerful, they just need to run on **separate hardware**.
* If data is also stored elsewhere, and it's not crucial to be up at all times, 1 or 2 replicas will generally be enough.
* For critical systems, replicate at least twice.

### Snapshots
Can make snapshots of index state at current point in time.
* Use for backup, and high performance / high availability replication.
* Use file to restore index.
* Can use this for reverting after you fuck up your index.
* Replication is just for node failure, not for rollback.

### Increasing query throughput with replication
* Runs queries in parallel.
* Es automatically routes requests to the best shard.
* CPU paralellisation (multiple cores) for multiple replica shards on the same node.
    - Note that this helps for performance, but not for fail safety.

## Adding more nodes to the cluster (for dev)
Don't do this on production, you uncultured swine.
### The right way
1. Download & extract Es into another dir
2. Configure cluster and node names
3. Start Es

Adding a node gives the message `Cluster health status changed from [YELLOW] to [GREEN] (reason: [shards started [[pages][0]]]).`. This is because with the second node, replication has been automatically added.

### The lazier way: overwrite settings
1. Reuse an existing Es dir
2. Overwrite config values with `-E [setting name]=[new value]`
    - `bin\elasticsearch.bat -E node.name=node-3 -E path.data=.\node-3\data -E path.logs=.\node-3\logs`

Note: should probably start Es from the root dir, not from `bin`. It seems to not find its logs dir properly.

## Overview of node roles
* **Master-eligible role**
    - Config: `node.master: true | false`
    - The best suitable master-eligible node is elected as master node automatically.
* **Data role**
    - Stores data
    - Perform queries
    - Config: `node.data: true | false`
    - Used as part of an explicit master-data node configuration.
* **Ingest role**
    - Runs ingest pipelines
    - Process data when indexing docs
    - Simplified version of Logstash
    - Config: `node.ingest: true | false`
    - If you're running a lot of data transformation, it may be useful to have a dedicated ingest node.
* **Machine learning role**
    - Config: `node.ml: true | false`
    - `xpack.ml.enabled: true | false` to enable ML API on this node
    - Run ML without affecting other tasks
* **Coordination role**
    - Query distribution and result aggregation
    - Config: all other roles (specified above) set to false.
    - Use as load balancer for large clusters
* **Voting-only role**
    - You will probably never use this
    - Only participates in electing a master node
    - For massive clusters

### When to change node roles?
* Useful for large clusters
* Optimising for scaling number of requests
* Only do this when you know what you're doing, otherwise it makes little sense.
* Generally change other things first (nodes, shards, replicas) before you try this.