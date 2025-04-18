# Architect

## Elastic Stack
### Flow
logs > beats > logstash > elasticsearch > kibana

## Components
|APP|Note|
|:---:|:---|
|Beats|Data Collection <br> Data shipper (e.g.Filebeat and Meticbeat)|
|Logstash|Data Processing <br> Server-side data processing|
|ElasticSearch|Storage <br> Store, Searching, Analyzing|
|Kibana|Visualize <br> Data visualization dashboard|
|X-Pack|Extension pack for ElasticSearch <br> Security(RBAC, Encryption, Audit, IP Filtering), Alerting, Monitoring, Reporting, Machine Learning(Anomalies), Graph Analytics, SQL|
