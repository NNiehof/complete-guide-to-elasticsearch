# 1. Introduction

## Uses
* Analyse and structure data (and visualise with Kibana)
* Implement **product recommendation** rules / restrictions as search rules
* **Forecasting** ML (supply chain / customer calls etc) - *note: ML is in X-Pack*
* Event logging through aggregations (why in Es though?)
* **Anomaly detection** on web traffic => send notification
* Monitoring of your API, e.g. number and duration of requests.

## Elastic stack

### Kibana
Analytics and visualisation.  
Configuring authorisation.

### Logstash
Data processing pipeline.  
Data is handled as events and sent to Es (or something else, like Kafka). Input can be e.g. from file, db, Kafka.  
Processing: parse string of data into a json object.
* Input plugins
* Filter plugins (data processing) - e.g. parsing with grok
* Output plugins (stashes)

### X-Pack
Add-on for Es and Kibana.  
* Authentication / authorisation
* User role management
* Monitoring of Es (CPU, memory usage etc)
* Export Kibana visualisations and reports.
* Export data as csv.
* ML for Es.
* Data **graphs**: relationships between related products (popularity-corrected: uses the relevance feature of Es).
* Can integrate with systems via API, e.g. product recommendations in e-mail newsletter
* Es SQL (or, you know, just learn the Es query language).

### Beats
Collection of data shippers. Sends data to Logstash or Es.  
Used for various monitoring and system management stuff. Probably don't need this right now.

## Common architectures

## Search functionality for web app
* Es used for search in parallel with the relational db. Update products in db and Es. Duplication of data, but this is by design.  
* Use Kibana to connect a dashboard to see your website activity.
* Monitor server load to see how it handles increases in traffic.

**Note**: Kibana config is stored in Es. (These were the extra indices I found).