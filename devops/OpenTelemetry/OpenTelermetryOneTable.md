# Open Telemetry Semantics


doc : <https://opentelemetry.io/docs/specs/semconv/attributes-registry/>


| Attribute   | Type    | Description | Examples |
|--------------|-----------|------------|------------|
|*client.address* |string |Stable |Client address - domain name if available without reverse DNS lookup; otherwise, IP address or Unix domain socket name. [1] client.example.com; 10.1.2.80; /tmp/my.sock|
|*client.port* |int |Stable |Client port number (ex : 65123 )|
|*cloud.account.id* |string |The cloud account ID the resource is assigned to. | opentelemetry|
|*cloud.availability_zone* |string| Cloud regions often have multiple, isolated locations known as zones to increase availability. Availability zone represents the zone where the resource is running.| us-east-1c|
|*cloud.platform* | string |The cloud platform in use.| alibaba_cloud_ecs|
|*cloud.provider* | string |Name of the cloud provider.| alibaba_cloud|
|*cloud.region* |string |The geographical region the resource is running.| us-central1; us-east-1|
|*cloud.resource_id*| string |Cloud provider-specific native identifier of the monitored cloud resource (e.g. an ARN on AWS, a fully qualified resource ID on Azure, a full resource name on GCP)| arn:aws:lambda:REGION:ACCOUNT_ID:function:my-function;|
|*code.column*| int |The column number in code.filepath best representing the operation. It SHOULD point within the code unit named in code.function.|
|*code.filepath*| string |The source code file name that identifies the code unit as uniquely as possible (preferably an absolute file path).| /usr/local/MyApplication/app/index.php|
|*code.function*| string| The method or function name, or equivalent (usually rightmost part of the code unit’s name).| serveRequest|
|*code.lineno* |int |The line number in code.filepath best representing the operation. It SHOULD point within the code unit named in code.function.| 42|
|*code.namespace*| string |The “namespace” within which code.function is defined. Usually the qualified class or module name, such that code.namespace + some separator + code.function form a unique identifier for the code unit.| com.example.MyHttpService|
|*code.stacktrace*| string |A stacktrace as a string in the natural representation for the language runtime. The representation is to be determined and documented by each language SIG.| at com.example.GenerateTrace.methodB(GenerateTrace.java:13)\n at com.example.GenerateTrace.methodA(GenerateTrace.java:9)\n at com.example.GenerateTrace.main(GenerateTrace.java:5)|
|*container.command*|string|The command used to run the container (i.e. the command name). [1]|otelcontribcol|
|container.command_args|string[]|All the command arguments (including the command/executable itself) run by the container. [2]|[otelcontribcol, --config, config.yaml]|
|*container.command_line*|string|The full command run by the container as a single string representing the full command. [2]|otelcontribcol --config config.yaml|
|*container.id*|string|Container ID. Usually a UUID, as for example used to identify Docker containers. The UUID might be abbreviated.|a3bf90e006b2
|container.image.id|string|Runtime specific image identifier. Usually a hash algorithm followed by a UUID. [2]|sha256:19c92d0a00d1b66d897bceaa7319bee0dd38a10a851c60bcec9474aa3f01e50f|
|*container.image.name*|string|Name of the image the container was built on.|gcr.io/opentelemetry/operator|
|*container.image.repo_digests*|string[]|Repo digests of the container image as provided by the container runtime. [3]|[example@sha256:afcc7f1ac1b49db317a7196c902e61c6c3c4607d63599ee1a82d702d249a0ccb, internal.registry.example.com:5000/example@sha256:b69959407d21e8a062e0416bf13405bb2b71ed7a84dde4158ebafacfa06f5578]|
|*container.image.tags*|string[]|Container image tags. An example can be found in Docker Image Inspect. Should be only the &lt;tag&gt; section of the full name for example from registry.example.com/my-org/my-image:&lt;tag&gt;.|[v1.27.1, 3.5.7-0]|
|*container.labels.&lt;key&gt;*|string|Container labels, &lt;key&gt; being the label name, the value being the label value.|container.labels.app=nginx
|*container.name*|string|Container name used by container runtime.|opentelemetry-autoconf|
|*container.runtime*|string|The container runtime managing this container.|docker; containerd; rkt|
|db.connection_string |string |The connection string used to connect to the database. It is recommended to remove embedded credentials. |Server=(localdb)\v11.0;Integrated Security=true;|
|db.instance.id |string |An identifier (address, unique name, or any other identifier) of the database instance that is executing queries or mutations on the current connection. This is useful in cases where the database is running in a clustered environment and the instrumentation is able to record the node executing the query. The client may obtain this value in databases like MySQL using queries like select @@hostname. |mysql-e26b99z.example.com|
|db.name |string |This attribute is used to report the name of the database being accessed. For commands that switch the database, this should be set to the target database (even if the command fails). [1] |customers; main|
|db.operation |string |The name of the operation being executed, e.g. the MongoDB command name such as findAndModify, or the SQL keyword. [2] |findAndModify; HMSET; SELECT|
|db.statement |string |The database statement being executed. |SELECT * FROM wuser_table; SET mykey "WuValue"|
|db.system |string |An identifier for the database management system (DBMS) product being used. See below for a list of well-known identifiers. |other_sql|
|db.user |string |Username for accessing the database. |readonly_user; reporting_user|
|db.cassandra.consistency_level |string |The consistency level of the query. Based on consistency values from CQL. |all|
|db.cassandra.coordinator.dc |string |The data center of the coordinating node for a query. |us-west-2|
|db.cassandra.coordinator.id |string |The ID of the coordinating node for a query. |be13faa2-8574-4d71-926d-27f16cf8a7af|
|db.cassandra.idempotence |boolean |Whether or not the query is idempotent. ||
|db.cassandra.page_size |int |The fetch size used for paging, i.e. how many rows will be returned at once. |5000|
|db.cassandra.speculative_execution_count |int |The number of times a query was speculatively executed. Not set or 0 if the query was not executed speculatively. |0; 2|
|db.cassandra.table |string |The name of the primary Cassandra table that the operation is acting upon, including the keyspace name (if applicable). [1] |mytable|
|db.cosmosdb.client_id |string |Unique Cosmos client instance id. |3ba4827d-4422-483f-b59f-85b74211c11d|
|db.cosmosdb.connection_mode |string |Cosmos client connection mode. |gateway|
|db.cosmosdb.container |string |Cosmos DB container name. |anystring|
|db.cosmosdb.operation_type |string |CosmosDB Operation Type. |Invalid|
|db.cosmosdb.request_charge |double |RU consumed for that operation |46.18; 1.0|
|db.cosmosdb.request_content_length |int |Request payload size in bytes |
|db.cosmosdb.status_code |int |Cosmos DB status code. |200; 201|
|db.cosmosdb.sub_status_code |int |Cosmos DB sub status code. |1000; 1002|
|db.elasticsearch.cluster.name |string |Represents the identifier of an Elasticsearch cluster. |e9106fc68e3044f0b1475b04bf4ffd5f|
|db.elasticsearch.node.name |string |Represents the human-readable identifier of the node/instance to which a request was routed. |instance-0000000001|
|db.elasticsearch.path_parts.&lt;key&gt; |string |A dynamic value in the url path. [1] |db.elasticsearch.path_parts.index=test-index; db.elasticsearch.path_parts.doc_id=123|
|db.jdbc.driver_classname |string |The fully-qualified class name of the Java Database Connectivity (JDBC) driver used to connect. |org.postgresql.Driver; com.microsoft.sqlserver.jdbc.SQLServerDriver|
|db.mongodb.collection |string |The MongoDB collection being accessed within the database stated in db.name. |customers; products|
|db.mssql.instance_name |string |The Microsoft SQL Server instance name connecting to. This name is used to determine the port of a named instance. |
|db.redis.database_index |int |The index of the database being accessed as used in the SELECT command, provided as an integer. To be used instead of the generic db.name attribute. |0; 1; 15|
|db.sql.table |string |The name of the primary table that the operation is acting upon, including the database name (if applicable). [1] |public.users; |
|destination.address|string|Destination address - domain name if available without reverse DNS lookup; otherwise, IP address or Unix domain socket name. [1] | destination.example.com; 10.1.2.80; /tmp/my.sock|
|destination.port|int|Destination port number|3389; 2888|
|device.id|string|A unique identifier representing the device [1]|2ab2916d-a51f-4ac8-80ee-45ac31a28092|
|device.manufacturer|string|The name of the device manufacturer [2]|Apple; Samsung|
|device.model.identifier|string|The model identifier for the device [3]|iPhone3,4; SM-G920F|
|device.model.name|string|The marketing name for the device model [4]|iPhone 6s Plus; Samsung Galaxy S6|
|disk.io.direction|string|The disk IO operation direction.|read|
|error.type|string|Stable |Describes a class of error the operation ended with. [1]|timeout; java.net.UnknownHostException;|
|exception.escaped|boolean|SHOULD be set to true if the exception event is recorded at a point where it is known that the exception is escaping the scope of the span. [1]|
|exception.message|string|The exception message.|Division by zero; Can't convert 'int' object to str implicitly
exception.stacktrace|string|A stacktrace as a string in the natural representation for the language runtime. The representation is to be determined and documented by each language SIG.|Exception in thread "main" java.lang.RuntimeException: Test exception\n at com.example.GenerateTrace.methodB(GenerateTrace.java:13)\n at com.example.GenerateTrace.methodA(GenerateTrace.java:9)\n at com.example.GenerateTrace.main(GenerateTrace.java:5)|
|exception.type|string|The type of the exception (its fully-qualified class name, if applicable). The dynamic type of the exception should be preferred over the static type in languages that support it.|
|host.arch|string|The CPU architecture the host system is running on.|amd64|
|host.cpu.cache.l2.size|int|The amount of level 2 memory cache available to the processor (in Bytes).|12288000|
|host.cpu.family|string|Family or generation of the CPU.|6; PA-RISC 1.1e|
|host.cpu.model.id|string|Model identifier. It provides more granular information about the CPU, distinguishing it from other CPUs within the same family.|6; 9000/778/B180L|
|host.cpu.model.name|string|Model designation of the processor.|11th Gen Intel(R) Core(TM) i7-1185G7 @ 3.00GHz|
|host.cpu.stepping|int|Stepping or core revisions.|1|
|host.cpu.vendor.id|string|Processor manufacturer identifier. A maximum 12-character string. [1]|GenuineIntel|
|host.id|string|Unique host ID. For Cloud, this must be the instance_id assigned by the cloud provider. For non-containerized systems, this should be the machine-id. See the table below for the sources to use to determine the machine-id based on operating system.|fdbf79e8af94cb7f9e8df36789187052|
|host.image.id|string|VM image ID or host OS image ID. For Cloud, this value is from the provider.|ami-07b06b442921831e5|
|host.image.name|string|Name of the VM image or OS install the host was instantiated from.|infra-ami-eks-worker-node-7d4ec78312; CentOS-8-x86_64-1905|
|host.image.version|string|The version string of the VM image or host OS as defined in Version Attributes.|0.1|
|host.ip|string[]|Available IP addresses of the host, excluding loopback interfaces. [2]|[192.168.1.140, fe80::abc2:4a28:737a:609e]|
|host.mac|string[]|Available MAC addresses of the host, excluding loopback interfaces. [3]|[AC-DE-48-23-45-67, AC-DE-48-23-45-67-01-9F]|
|host.name|string|Name of the host. On Unix systems, it may contain what the hostname command returns, or the fully qualified hostname, or another name specified by the user.|opentelemetry-test|
|host.type|string|Type of host. For Cloud, this must be the machine type.|n1-standard-1|
|http.request.body.size|int|The size of the request payload body in bytes. This is the number of bytes transferred excluding headers and is often, but not always, present as the Content-Length header. For requests using transport encoding, this should be the compressed size.|3495|
|http.request.header.&lt;key&gt;|string[]|Stable|HTTP request headers, &lt;key&gt; being the normalized HTTP Header name (lowercase), the value being the header values. [1]|http.request.header.content-type=["application/json"]; http.request.header.x-forwarded-for=["1.2.3.4", "1.2.3.5"]|
|http.request.method|string|Stable|HTTP request method. [2]|GET; POST; HEAD|
|http.request.method_original|string|Stable|Original HTTP method sent by the client in the request line.|GeT; ACL; foo|
|http.request.resend_count|int|Stable|The ordinal number of request resending attempt (for any reason, including redirects). [3]|3|
|http.response.body.size|int|The size of the response payload body in bytes. This is the number of bytes transferred excluding headers and is often, but not always, present as the Content-Length header. For requests using transport encoding, this should be the compressed size.|3495|
|http.response.header.&lt;key&gt;|string[]|Stable|HTTP response headers, &lt;key&gt; being the normalized HTTP Header name (lowercase), the value being the header values. [4]|http.response.header.content-type=["application/json"]; http.response.header.my-custom-header=["abc", "def"]|
|http.response.status_code|int|Stable|HTTP response status code.|200|
|http.route|string|Stable|The matched route, that is, the path template in the format used by the respective server framework. [5]|/users/:userID?; {controller}/{action}/{id?}|
|http.flavor|string|Deprecated|Deprecated, use network.protocol.name instead.|1.0|
|http.method|string|Deprecated|Deprecated, use http.request.method instead.|GET; POST; HEAD|
|http.request_content_length|int|Deprecated|Deprecated, use http.request.header.content-length instead.|3495|
|http.response_content_length|int|Deprecated|Deprecated, use http.response.header.content-length instead.|3495|
|http.scheme|string|Deprecated|Deprecated, use url.scheme instead.|http; https|
|http.status_code|int|Deprecated|Deprecated, use http.response.status_code instead.|200|
|http.target|string|Deprecated|Deprecated, use url.path and url.query instead.|/search?q=OpenTelemetry#SemConv|
|http.url|string|Deprecated|Deprecated, use url.full instead.|<https://www.foo.bar/search?q=OpenTelemetry#SemConv>|
|http.user_agent|string|Deprecated|Deprecated, use user_agent.original instead.|CERN-LineMode/2.15 libwww/2.17b3; Mozilla/5.0 (iPhone; CPU iPhone OS 14_7_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.2 Mobile/15E148 Safari/604.1|
|k8s.cluster.name|string|The name of the cluster.|opentelemetry-cluster|
|k8s.cluster.uid|string|A pseudo-ID for the cluster, set to the UID of the kube-system namespace. [1]|218fc5a9-a5f1-4b54-aa05-46717d0ab26d|
|k8s.container.name|string|The name of the Container from Pod specification, must be unique within a Pod. Container runtime usually uses different globally unique name (container.name).|redis|
|k8s.container.restart_count|int|Number of times the container was restarted. This attribute can be used to identify a particular container (running or stopped) within a container spec.|0; 2|
|k8s.cronjob.name|string|The name of the CronJob.|opentelemetry|
|k8s.cronjob.uid|string|The UID of the CronJob.|275ecb36-5aa8-4c2a-9c47-d8bb681b9aff|
|k8s.daemonset.name|string|The name of the DaemonSet.|opentelemetry|
|k8s.daemonset.uid|string|The UID of the DaemonSet.|275ecb36-5aa8-4c2a-9c47-d8bb681b9aff|
|k8s.deployment.name|string|The name of the Deployment.|opentelemetry|
|k8s.deployment.uid|string|The UID of the Deployment.|275ecb36-5aa8-4c2a-9c47-d8bb681b9aff|
|k8s.job.name|string|The name of the Job.|opentelemetry|
|k8s.job.uid|string|The UID of the Job.|275ecb36-5aa8-4c2a-9c47-d8bb681b9aff|
|k8s.namespace.name|string|The name of the namespace that the pod is running in.|default|
|k8s.node.name|string|The name of the Node.|node-1|
|k8s.node.uid|string|The UID of the Node.|1eb3a0c6-0477-4080-a9cb-0cb7db65c6a2|
|k8s.pod.annotation.&lt;key&gt;|string|The annotation key-value pairs placed on the Pod, the &lt;key&gt; being the annotation name, the value being the annotation value.|k8s.pod.annotation.kubernetes.io/enforce-mountable-secrets=true; k8s.pod.annotation.mycompany.io/arch=x64; k8s.pod.annotation.data=|
|k8s.pod.labels.&lt;key&gt;|string|The labels placed on the Pod, the &lt;key&gt; being the label name, the value being the label value.|k8s.pod.labels.app=my-app; k8s.pod.labels.mycompany.io/arch=x64; k8s.pod.labels.data=|
|k8s.pod.name|string|The name of the Pod.|opentelemetry-pod-autoconf|
|k8s.pod.uid|string|The UID of the Pod.|275ecb36-5aa8-4c2a-9c47-d8bb681b9aff|
|k8s.replicaset.name|string|The name of the ReplicaSet.|opentelemetry|
|k8s.replicaset.uid|string|The UID of the ReplicaSet.|275ecb36-5aa8-4c2a-9c47-d8bb681b9aff|
|k8s.statefulset.name|string|The name of the StatefulSet.|opentelemetry|
|k8s.statefulset.uid|string|The UID of the StatefulSet.|275ecb36-5aa8-4c2a-9c47-d8bb681b9aff|
|messaging.batch.message_count|int|The number of messages sent, received, or processed in the scope of the batching operation. [1]|0; 1; 2|
|messaging.client_id|string|A unique identifier for the client that consumes or produces a message.|client-5; myhost@8742@s8083jm|
|messaging.destination.anonymous|boolean|A boolean that is true if the message destination is anonymous (could be unnamed or have auto-generated name).|
|messaging.destination.name|string|The message destination name [2]|MyQueue; MyTopic|
|messaging.destination.template|string|Low cardinality representation of the messaging destination name [3]|/customers/{customerId}|
|messaging.destination.temporary|boolean|A boolean that is true if the message destination is temporary and might not exist anymore after messages are processed.|
|messaging.destination_publish.anonymous|boolean|A boolean that is true if the publish message destination is anonymous (could be unnamed or have auto-generated name).|
|messaging.destination_publish.name|string|The name of the original destination the message was published to [4]|MyQueue; MyTopic|
|messaging.gcp_pubsub.message.ordering_key|string|The ordering key for a given message. If the attribute is not present, the message does not have an ordering key.|ordering_key|
|messaging.kafka.consumer.group|string|Name of the Kafka Consumer Group that is handling the message. Only applies to consumers, not producers.|my-group|
|messaging.kafka.destination.partition|int|Partition the message is sent to.|2|
|messaging.kafka.message.key|string|Message keys in Kafka are used for grouping alike messages to ensure they’re processed on the same partition. They differ from messaging.message.id in that they’re not unique. If the key is null, the attribute MUST NOT be set. [5]|myKey|
|messaging.kafka.message.offset|int|The offset of a record in the corresponding Kafka partition.|42|
|messaging.kafka.message.tombstone|boolean|A boolean that is true if the message is a tombstone.|
|messaging.message.body.size|int|The size of the message body in bytes. [6]|1439|
|messaging.message.conversation_id|string|The conversation ID identifying the conversation to which the message belongs, represented as a string. Sometimes called “Correlation ID”.|MyConversationId|
|messaging.message.envelope.size|int|The size of the message body and metadata in bytes. [7]|2738|
|messaging.message.id|string|A value used by the messaging system as an identifier for the message, represented as a string.|452a7c7c7c7048c2f887f61572b18fc2|
|messaging.operation|string|A string identifying the kind of messaging operation. [8]|publish|
|messaging.rabbitmq.destination.routing_key|string|RabbitMQ message routing key.|myKey|
|messaging.rocketmq.client_group|string|Name of the RocketMQ producer/consumer group that is handling the message. The client type is identified by the SpanKind.|myConsumerGroup|
|messaging.rocketmq.consumption_model|string|Model of message consumption. This only applies to consumer spans.|clustering|
|messaging.rocketmq.message.delay_time_level|int|The delay time level for delay message, which determines the message delay time.|3|
|messaging.rocketmq.message.delivery_timestamp|int|The timestamp in milliseconds that the delay message is expected to be delivered to consumer.|1665987217045|
|messaging.rocketmq.message.group|string|It is essential for FIFO message. Messages that belong to the same message group are always processed one by one within the same consumer group.|myMessageGroup|
|messaging.rocketmq.message.keys|string[]|Key(s) of message, another way to mark message besides message id.|[keyA, keyB]|
|messaging.rocketmq.message.tag|string|The secondary classifier of message besides topic.|tagA|
|messaging.rocketmq.message.type|string|Type of message.|normal|
|messaging.rocketmq.namespace|string|Namespace of RocketMQ resources, resources in different namespaces are individual.|myNamespace|
|messaging.system|string|An identifier for the messaging system being used. See below for a list of well-known identifiers.|activemq|
|network.carrier.icc|string|The ISO 3166-1 alpha-2 2-character country code associated with the mobile carrier network.|DE|
|network.carrier.mcc|string|The mobile carrier country code.|310|
|network.carrier.mnc|string|The mobile carrier network code.|001|
|network.carrier.name|string|The name of the mobile carrier.|sprint|
|network.connection.subtype|string|This describes more details regarding the connection.type. It may be the type of cell technology connection, but it could be used for describing details about a wifi connection.|LTE|
|network.connection.type|string|The internet connection type.|wifi|
|network.io.direction|string|The network IO operation direction.|transmit|
|network.local.address|string|Stable|Local address of the network connection - IP address or Unix domain socket name.|10.1.2.80; /tmp/my.sock|
|network.local.port|int|Stable|Local port number of the network connection.|65123
|network.peer.address|string|Stable|Peer address of the network connection - IP address or Unix domain socket name.|10.1.2.80; /tmp/my.sock
|network.peer.port|int|Stable|Peer port number of the network connection.|65123
|network.protocol.name|string|Stable|OSI application layer or non-OSI equivalent. [1]|amqp; http; mqtt|
|network.protocol.version|string|Stable|Version of the protocol specified in network.protocol.name. [2]|3.1.1|
|network.transport|string|Stable|OSI transport layer or inter-process communication method. [3]|tcp; udp|
|network.type|string|Stable|OSI network layer or non-OSI equivalent. [4]|ipv4; ipv6|
|oci.manifest.digest|string|The digest of the OCI image manifest. For container images specifically is the digest by which the container image is known. [1]|sha256:e4ca62c0d62f3e886e684806dfe9d4e0cda60d54986898173c1083856cfda0f4|
|os.build_id|string|Unique identifier for a particular build or compilation of the operating system.|TQ3C.230805.001.B2; 20E247; 22621|
|os.description|string|Human readable (not intended to be parsed) OS version information, like e.g. reported by ver or lsb_release -a commands.|Microsoft Windows [Version 10.0.18363.778]; Ubuntu 18.04.1 LTS|
|os.name|string|Human readable operating system name.|iOS; Android; Ubuntu
|os.type|string|The operating system type.|windows|
|os.version|string|The version string of the operating system as defined in Version Attributes.|14.2.1; 18.04.1|
|process.command|string|The command used to launch the process (i.e. the command name). On Linux based systems, can be set to the zeroth string in proc/[pid]/cmdline. On Windows, can be set to the first parameter extracted from GetCommandLineW.|cmd/otelcol|
|process.command_args|string[]|All the command arguments (including the command/executable itself) as received by the process. On Linux-based systems (and some other Unixoid systems supporting procfs), can be set according to the list of null-delimited strings extracted from proc/[pid]/cmdline. For libc-based executables, this would be the full argv vector passed to main.|[cmd/otecol, --config=config.yaml]|
|process.command_line|string|The full command used to launch the process as a single string representing the full command. On Windows, can be set to the result of GetCommandLineW. Do not set this if you have to assemble it just for monitoring; use process.command_args instead.|C:\cmd\otecol --config="my directory\config.yaml"|
|process.executable.name|string|The name of the process executable. On Linux based systems, can be set to the Name in proc/[pid]/status. On Windows, can be set to the base name of GetProcessImageFileNameW.|otelcol|
|process.executable.path|string|The full path to the process executable. On Linux based systems, can be set to the target of proc/[pid]/exe. On Windows, can be set to the result of GetProcessImageFileNameW.|/usr/bin/cmd/otelcol
process.owner|string|The username of the user that owns the process.|root|
|process.parent_pid|int|Parent Process identifier (PPID).|111
process.pid|int|Process identifier (PID).|1234|
|process.runtime.description|string|An additional description about the runtime of the process, for example a specific vendor customization of the runtime environment.|Eclipse OpenJ9 Eclipse OpenJ9 VM openj9-0.21.0|
|process.runtime.name|string|The name of the runtime of this process. For compiled native binaries, this SHOULD be the name of the compiler.|OpenJDK Runtime Environment|
|process.runtime.version|string|The version of the runtime of this process, as returned by the runtime without modification.|14.0.2|
|rpc.connect_rpc.error_code|string|The error codes of the Connect request. Error codes are always string values.|cancelled|
|rpc.connect_rpc.request.metadata.&lt;key&gt;|string[]|Connect request metadata, &lt;key&gt; being the normalized Connect Metadata key (lowercase), the value being the metadata values. [1]|rpc.request.metadata.my-custom-metadata-attribute=["1.2.3.4", "1.2.3.5"]|
|rpc.connect_rpc.response.metadata.&lt;key&gt;|string[]|Connect response metadata, &lt;key&gt; being the normalized Connect Metadata key (lowercase), the value being the metadata values. [2]|rpc.response.metadata.my-custom-metadata-attribute=["attribute_value"]|
|rpc.grpc.request.metadata.&lt;key&gt;|string[]|gRPC request metadata, &lt;key&gt; being the normalized gRPC Metadata key (lowercase), the value being the metadata values. [3]|rpc.grpc.request.metadata.my-custom-metadata-attribute=["1.2.3.4", "1.2.3.5"]|
|rpc.grpc.response.metadata.&lt;key&gt;|string[]|gRPC response metadata, &lt;key&gt; being the normalized gRPC Metadata key (lowercase), the value being the metadata values. [4]|rpc.grpc.response.metadata.my-custom-metadata-attribute=["attribute_value"]|
|rpc.grpc.status_code|int|The numeric status code of the gRPC request.|0|
|rpc.jsonrpc.error_code|int|error.code property of response if it is an error response.|-32700; 100|
|rpc.jsonrpc.error_message|string|error.|  of response if it is an error response.|Parse error; User already exists|
|rpc.jsonrpc.request_id|string|id property of request or response. Since protocol allows id to be int, string, null or missing (for notifications), value is expected to be cast to string for simplicity. Use empty string in case of null value. Omit entirely if this is a notification.|10; request-7; ``|
|rpc.jsonrpc.version|string|Protocol version as in jsonrpc property of request/response. Since JSON-RPC 1.0 doesn’t specify this, the value can be omitted.|2.0; 1.0|
|rpc.method|string|The name of the (logical) method being called, must be equal to the $method part in the span name. [5]|exampleMethod|
|rpc.service|string|The full (logical) name of the service being called, including its package name, if applicable. [6]|myservice.EchoService|
|rpc.system|string|A string identifying the remoting system. See below for a list of well-known identifiers.|grpc|
|server.address|string|Stable|Server domain name if available without reverse DNS lookup; otherwise, IP address or Unix domain socket name. [1]|example.com; 10.1.2.80; /tmp/my.sock|
|server.port|int|Stable|Server port number. [2]|80; 8080; 443|
|source.address|string|Source address - domain name if available without reverse DNS lookup; otherwise, IP address or Unix domain socket name. [1]|source.example.com; 10.1.2.80; /tmp/my.sock|
|source.port|int|Source port number|3389; 2888|
|thread.id|int|Current “managed” thread ID (as opposed to OS thread ID).|42|
|thread.name|string|Current thread name.|main|
|tls.cipher|string|String indicating the cipher used during the current connection. [1]|TLS_RSA_WITH_3DES_EDE_CBC_SHA; TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256|
|tls.client.certificate|string|PEM-encoded stand-alone certificate offered by the client. This is usually mutually-exclusive of client.certificate_chain since this value also exists in that list.|MII...|
|tls.client.certificate_chain|string[]|Array of PEM-encoded certificates that make up the certificate chain offered by the client. This is usually mutually-exclusive of client.certificate since that value should be the first certificate in the chain.|[MII..., MI...]|
|tls.client.hash.md5|string|Certificate fingerprint using the MD5 digest of DER-encoded version of certificate offered by the client. For consistency with other hash values, this value should be formatted as an uppercase hash.|0F76C7F2C55BFD7D8E8B8F4BFBF0C9EC|
|tls.client.hash.sha1|string|Certificate fingerprint using the SHA1 digest of DER-encoded version of certificate offered by the client. For consistency with other hash values, this value should be formatted as an uppercase hash.|9E393D93138888D288266C2D915214D1D1CCEB2A|
|tls.client.hash.sha256|string|Certificate fingerprint using the SHA256 digest of DER-encoded version of certificate offered by the client. For consistency with other hash values, this value should be formatted as an uppercase hash.|0687F666A054EF17A08E2F2162EAB4CBC0D265E1D7875BE74BF3C712CA92DAF0|
|tls.client.issuer|string|Distinguished name of subject of the issuer of the x.509 certificate presented by the client.|CN=Example Root CA, OU=Infrastructure Team, DC=example, DC=com|
|tls.client.ja3|string|A hash that identifies clients based on how they perform an SSL/TLS handshake.|d4e5b18d6b55c71272893221c96ba240|
|tls.client.not_after|string|Date/Time indicating when client certificate is no longer considered valid.|2021-01-01T00:00:00.000Z|
|tls.client.not_before|string|Date/Time indicating when client certificate is first considered valid.|1970-01-01T00:00:00.000Z|
|tls.client.server_name|string|Also called an SNI, this tells the server which hostname to which the client is attempting to connect to.|opentelemetry.io|
|tls.client.subject|string|Distinguished name of subject of the x.509 certificate presented by the client.|CN=myclient, OU=Documentation Team, DC=example, DC=com|
|tls.client.supported_ciphers|string[]|Array of ciphers offered by the client during the client hello.|["TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384", "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384", "..."]|
|tls.curve|string|String indicating the curve used for the given cipher, when applicable|secp256r1|
|tls.established|boolean|Boolean flag indicating if the TLS negotiation was successful and transitioned to an encrypted tunnel.|True|
|tls.next_protocol|string|String indicating the protocol being tunneled. Per the values in the IANA registry, this string should be lower case.|http/1.1|
|tls.protocol.name|string|Normalized lowercase protocol name parsed from original string of the negotiated SSL/TLS protocol version|ssl|
|tls.protocol.version|string|Numeric part of the version parsed from the original string of the negotiated SSL/TLS protocol version|1.2; 3|
|tls.resumed|boolean|Boolean flag indicating if this TLS connection was resumed from an existing TLS negotiation.|True|
|tls.server.certificate|string|PEM-encoded stand-alone certificate offered by the server This is usually mutually-exclusive of server.certificate_chain since this value also exists in that list.|MII...|
|tls.server.certificate_chain|string[]|Array of PEM-encoded certificates that make up the certificate chain offered by the server. This is usually mutually-exclusive of server.certificate since that value should be the first certificate in the chain.|[MII..., MI...]|
|tls.server.hash.md5|string|Certificate fingerprint using the MD5 digest of DER-encoded version of certificate offered by the server. For consistency with other hash values, this value should be formatted as an uppercase hash.|0F76C7F2C55BFD7D8E8B8F4BFBF0C9EC|
|tls.server.hash.sha1|string|Certificate fingerprint using the SHA1 digest of DER-encoded version of certificate offered by the server. For consistency with other hash values, this value should be formatted as an uppercase hash.|9E393D93138888D288266C2D915214D1D1CCEB2A|
|tls.server.hash.sha256|string|Certificate fingerprint using the SHA256 digest of DER-encoded version of certificate offered by the server. For consistency with other hash values, this value should be formatted as an uppercase hash.|0687F666A054EF17A08E2F2162EAB4CBC0D265E1D7875BE74BF3C712CA92DAF0|
|tls.server.issuer|string|Distinguished name of subject of the issuer of the x.509 certificate presented by the client.|CN=Example Root CA, OU=Infrastructure Team, DC=example, DC=com|
|tls.server.ja3s|string|A hash that identifies servers based on how they perform an SSL/TLS handshake.|d4e5b18d6b55c71272893221c96ba240|
|tls.server.not_after|string|Date/Time indicating when server certificate is no longer considered valid.|2021-01-01T00:00:00.000Z|
|tls.server.not_before|string|Date/Time indicating when server certificate is first considered valid.|1970-01-01T00:00:00.000Z|
|tls.server.subject|string|Distinguished name of subject of the x.509 certificate presented by the server.|CN=myserver, OU=Documentation Team, DC=example, DC=com|
|url.fragment|string|Stable|The URI fragment component|SemConv|
|url.full|string|Stable|Absolute URL describing a network resource according to RFC3986 [1]|<https://www.foo.bar/search?q=OpenTelemetry#SemConv>; //localhost|
|url.path|string|Stable|The URI path component|/search|
|url.query|string|Stable|The URI query component [2]|q=OpenTelemetry|
|url.scheme|string|Stable|The URI scheme component identifying the used protocol.|https; ftp; telnet|
|user_agent.original|string|Stable|Value of the HTTP User-Agent header sent by the client.|CERN-LineMode/2.15 libwww/2.17b3; Mozilla/5.0 (iPhone; CPU iPhone OS 14_7_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.2 Mobile/15E148 Safari/604.1|
