# elk 

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->


## Monitoring 

 * From https://logz.io/blog/aws-elasticsearch-log-management/
 * https://code972.com/blog/2017/12/111-why-you-shouldnt-use-aws-elasticsearch-service
 * From personnal experience :)
 * https://read.acloud.guru/things-you-should-know-before-using-awss-elasticsearch-service-7cd70c9afb4f
 
 
 
### Common trap 

 - Take care to clasical ES limits 
    - memory 
	- ulimit for the user running ES
	- extend the default refreshInterval
	- if we want several instance, insure they run on different disk
	- check log files permissions
    - control index rotation (on index per ay, and we kept only last X index open))
 - design 
   - on large installation we need to have a queue mechanism in front of elk (kafka / redis)
   - take into account : logs criticality / volume / retention (size, time) / HA
   - take care of the mapping , we do not need to do full text analysis on logs (at least on all the field)
   - use beats/ingest instead of logstash 
   - try to use the 'dissect filter' in logstash as a pipeline processor.  10 times Faster than groks.


## Best practices 

 1. Queuing. Install a queuing system such as Kafka, Redis or RabbitMQ. This is imperative to include in any ELK reference architecture because Logstash might overutilize Elasticsearch, which will then slow down Logstash until the small internal queue bursts and data will be lost. In addition, without a queuing system it becomes almost impossible to upgrade the Elasticsearch cluster because there is no way to store data during critical cluster upgrades.
 
 2. The ELK Stack includes Logstash which reads data from the queuing system, parses the logs, and then creates JSON messages for Elasticsearch to index. Running Logstash in a consistent and scalable manner is not a small challenge. You’ll need to hook up at least one Logstash server to read data and push logs to from the queuing system to AWS Elasticsearch. Also, you need to make sure that you correctly configure memory consumption for Logstash and verify GROK patterns to correctly parse the data. (See more in our Logstash tutorial and more on the pitfalls to avoid.) Logstash also plays a role as a log shipper.

 3. Elasticsearch Scalability. This is easier said than done. The AWS offering is static and does not scale-out automatically. Naive scalability (with scaling groups) would not work here since when the load on Elasticsearch increases, so the last thing you want to do is add another node — it will increase the load significantly as Elasticsearch moves shards to the new nodes, often resulting in cluster failure. Unfortunately, there is no easy answer here. At Logz.io, we’ve invested years of developers’ time to solve this problem in our environment. What you’ll probably have to do is allocate additional resources, monitor clusters carefully, and manually increase capacity whenever you think “winter is coming.”

 4. High Availability. If you’re running on top of AWS, you probably know that EC2 instances sometimes just stop working. It happens to us almost every day. If you want a system on which you can rely in production, you need to make sure the ELK implementation is highly available by setting up:
   - A highly-available queuing system, one that runs on two AZs with full replication
   - Logstash servers that read from the queuing system
   - An Elasticsearch cluster that will run across AZs and have three master nodes

 5. Data Curation. You’ll need to implement a cron job that uses the Curator application to delete old indices (otherwise, you’re running with a sand-clock until Elasticsearch will crash). It is also recommended to optimize older indices to improve the performance of Elasticsearch — just be careful not to run these processes during load times because they are resource heavy!

 6. Conflict Mapping. This is probably one of the more challenging items in this list. Mapping is like a ‘database schema’ in Elasticsearch lingo. When using Elasticsearch for log management, the software usually uses dynamic mapping that can build the schema “on the fly” as new log types appear. However, this mechanism is highly susceptible to any deviation from the initially-created mapping. When you have a deviation — and trust me, you’ll have plenty of them! — you’ll need to go back to your log data, grok parsing, or application code itself and modify it to avoid future conflicts. If you don’t do that, these events will load Elasticsearch and be dropped.

 7. Security With Multi-User & Role-Based Access. As with any cloud system, you’ll want to collaborate with your team by assigning various users different access roles. That’s not possible with the open-source ELK stack. You can put an Nginx reverse proxy in front of a Kibana server, but that will give you only very simple access. Elastic offers Shield, a proprietary module that supports role-based access for Elasticsearch, but it is not supported in AWS-hosted Elasticsearch.

 8. Dashboards and Visualizations. If you’re already a pro with ELK and have all of your data parsed and correctly visualized, you can skip this section. If you’re in the process of visualizing your data to understand it, you can use our ELK Apps library. You can download relevant dashboards and import them to your Kibana.

 9. Log Shipping. This part actually happens within your datacenter or VPC. You’ll need to securely and effectively ship data to the queuing system. Logstash and Fluentd are good options (see our comparison of the two), and using rsyslog is also very common. For Windows, NxLog has a reasonably good reputation. Fine tuning each of these agents and correctly configuring them to ship data is imperative. You can have a look here for more information on the available shippers. If you’ll want to read logs from AWS Cloudtrail, ELB, S3, or other AWS repositories, you’ll need to implement a pull module (Logstash offers some) that can periodically go to S3 and pull data. AWS-hosted Elasticsearch does not offer out-of-the-box integration with these agents but you can read online and set them up independently.

 10. Log Parsing. Parsing and correctly structuring logs is fundamental to the ability to search and visualize the data effectively. Even a small environment can include 20 or 30 different log types. Parsing logs can be done with Logstash, which transforms text lines to structured JSON objects. Manually building grok expressions to parse through log data is a tedious and error-prone process. This is also not offered by the AWS Elasticsearch service out-of-the-box. However, many grok expressions can be found online but careful compatibility testing with the specific version is required. If you search the Web, you will probably find some GitHub repo that will include grok data that other people have already developed.

 11. Alerts. The open-source ELK Stack does not provide an alerting capability. Instead, you can develop a cron job that will automatically run queries and generate e-mails based on search results. A lot of bits and pieces need to be included to make it effective such as sliding window, statefulness, and more, but hard-refreshing dashboards and searches manually is more difficult. Elastic offers a premium service called Watcher, but it is not available on AWS-hosted Elasticsearch.

 12. S3 Archiving. The ability to retain full access to logs for long periods of time such as one year — or even seven years — requires you to build a system that can archive all log events and then ingest them back when needed. You can either extend the cluster to support one year of log data or archive the events from Elasticsearch into a static repository such as S3 or Glacier. Data retention has a critical effect on the memory, CPU, and disk that are all used by Elasticsearch, so it is highly recommended that such mechanisms be developed to allow for longer periods of retention times.

 13. ELK Monitoring. Monitoring each component of a stack is important in any critical system. Nagios has some plugins to monitor Elasticsearch, and you’ll need to make sure that you correctly monitor the queue size of your queuing system and the health of your Logstash and Kibana components.

 14. Cost. Since auto-scaling is not supported in AWS-hosted Elasticsearch, what many people need to do is over-allocate resources — sometimes by as much as ten times the day-to-day normal usage — in order to create a sustainable system. Elasticsearch can get very expensive very quickly, especially as a cluster grows.


