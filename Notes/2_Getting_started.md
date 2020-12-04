# Getting started

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

## Inspecting the cluster
