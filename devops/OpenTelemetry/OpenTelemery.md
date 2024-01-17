Open Telemetry Semantics

# Client

| Attribute   | Type    | Description | Examples |
|--------------|-----------|------------|------------|
|*client.address* |string |Stable |Client address - domain name if available without reverse DNS lookup; otherwise, IP address or Unix domain socket name. [1] client.example.com; 10.1.2.80; /tmp/my.sock|
|*client.port* |int |Stable |Client port number (ex : 65123 )|

# Cloud Attributes

|Attribute |Type |Description |Examples|
|--------------|-----------|------------|------------|
|*cloud.account.id* |string |The cloud account ID the resource is assigned to. | opentelemetry|
|*cloud.availability_zone* |string| Cloud regions often have multiple, isolated locations known as zones to increase availability. Availability zone represents the zone where the resource is running.| us-east-1c|
|*cloud.platform* | string |The cloud platform in use.| alibaba_cloud_ecs|
|*cloud.provider* | string |Name of the cloud provider.| alibaba_cloud|
|*cloud.region* |string |The geographical region the resource is running.| us-central1; us-east-1|
|*cloud.resource_id*| string |Cloud provider-specific native identifier of the monitored cloud resource (e.g. an ARN on AWS, a fully qualified resource ID on Azure, a full resource name on GCP)| arn:aws:lambda:REGION:ACCOUNT_ID:function:my-function;|

# Code Attributes

|Attribute |Type |Description |Examples|
|--------------|-----------|------------|------------|
|*code.column*| int |The column number in code.filepath best representing the operation. It SHOULD point within the code unit named in code.function.|
|*code.filepath*| string |The source code file name that identifies the code unit as uniquely as possible (preferably an absolute file path).| /usr/local/MyApplication/app/index.php|
|*code.function*| string| The method or function name, or equivalent (usually rightmost part of the code unit’s name).| serveRequest|
|*code.lineno* |int |The line number in code.filepath best representing the operation. It SHOULD point within the code unit named in code.function.| 42|
|*code.namespace*| string |The “namespace” within which code.function is defined. Usually the qualified class or module name, such that code.namespace + some separator + code.function form a unique identifier for the code unit.| com.example.MyHttpService|
|*code.stacktrace*| string |A stacktrace as a string in the natural representation for the language runtime. The representation is to be determined and documented by each language SIG.| at com.example.GenerateTrace.methodB(GenerateTrace.java:13)\n at com.example.GenerateTrace.methodA(GenerateTrace.java:9)\n at com.example.GenerateTrace.main(GenerateTrace.java:5)|

# Container Attributes

|Attribute |Type |Description |Examples|
|--------------|-----------|------------|------------|
|*container.command*|string|The command used to run the container (i.e. the command name). [1]|otelcontribcol|
|container.command_args|string[]|All the command arguments (including the command/executable itself) run by the container. [2]|[otelcontribcol, --config, config.yaml]|
|*container.command_line*|string|The full command run by the container as a single string representing the full command. [2]|otelcontribcol --config config.yaml|
|*container.id*|string|Container ID. Usually a UUID, as for example used to identify Docker containers. The UUID might be abbreviated.|a3bf90e006b2
|container.image.id|string|Runtime specific image identifier. Usually a hash algorithm followed by a UUID. [2]|sha256:19c92d0a00d1b66d897bceaa7319bee0dd38a10a851c60bcec9474aa3f01e50f|
|*container.image.name*|string|Name of the image the container was built on.|gcr.io/opentelemetry/operator|
|*container.image.repo_digests*|string[]|Repo digests of the container image as provided by the container runtime. [3]|[example@sha256:afcc7f1ac1b49db317a7196c902e61c6c3c4607d63599ee1a82d702d249a0ccb, internal.registry.example.com:5000/example@sha256:b69959407d21e8a062e0416bf13405bb2b71ed7a84dde4158ebafacfa06f5578]|
|*container.image.tags*|string[]|Container image tags. An example can be found in Docker Image Inspect. Should be only the <tag> section of the full name for example from registry.example.com/my-org/my-image:<tag>.|[v1.27.1, 3.5.7-0]|
|*container.labels.<key>*|string|Container labels, <key> being the label name, the value being the label value.|container.labels.app=nginx
|*container.name*|string|Container name used by container runtime.|opentelemetry-autoconf|
|*container.runtime*|string|The container runtime managing this container.|docker; containerd; rkt|


# Generic Database Attributes

|Attribute |Type |Description |Examples|
|--------------|-----------|------------|------------|
|db.connection_string |string |The connection string used to connect to the database. It is recommended to remove embedded credentials. |Server=(localdb)\v11.0;Integrated Security=true;|
|db.instance.id |string |An identifier (address, unique name, or any other identifier) of the database instance that is executing queries or mutations on the current connection. This is useful in cases where the database is running in a clustered environment and the instrumentation is able to record the node executing the query. The client may obtain this value in databases like MySQL using queries like select @@hostname. |mysql-e26b99z.example.com|
|db.name |string |This attribute is used to report the name of the database being accessed. For commands that switch the database, this should be set to the target database (even if the command fails). [1] |customers; main|
|db.operation |string |The name of the operation being executed, e.g. the MongoDB command name such as findAndModify, or the SQL keyword. [2] |findAndModify; HMSET; SELECT|
|db.statement |string |The database statement being executed. |SELECT * FROM wuser_table; SET mykey "WuValue"|
|db.system |string |An identifier for the database management system (DBMS) product being used. See below for a list of well-known identifiers. |other_sql|
|db.user |string |Username for accessing the database. |readonly_user; reporting_user|

## Cassandra Attributes

|Attribute |Type |Description |Examples|
|--------------|-----------|------------|------------|
|db.cassandra.consistency_level |string |The consistency level of the query. Based on consistency values from CQL. |all|
|db.cassandra.coordinator.dc |string |The data center of the coordinating node for a query. |us-west-2|
|db.cassandra.coordinator.id |string |The ID of the coordinating node for a query. |be13faa2-8574-4d71-926d-27f16cf8a7af|
|db.cassandra.idempotence |boolean |Whether or not the query is idempotent. ||
|db.cassandra.page_size |int |The fetch size used for paging, i.e. how many rows will be returned at once. |5000|
|db.cassandra.speculative_execution_count |int |The number of times a query was speculatively executed. Not set or 0 if the query was not executed speculatively. |0; 2|
|db.cassandra.table |string |The name of the primary Cassandra table that the operation is acting upon, including the keyspace name (if applicable). [1] |mytable|

## CosmosDB Attributes

Attribute |Type |Description |Examples
|--------------|-----------|------------|------------|
db.cosmosdb.client_id |string |Unique Cosmos client instance id. |3ba4827d-4422-483f-b59f-85b74211c11d
db.cosmosdb.connection_mode |string |Cosmos client connection mode. |gateway
db.cosmosdb.container |string |Cosmos DB container name. |anystring
db.cosmosdb.operation_type |string |CosmosDB Operation Type. |Invalid
db.cosmosdb.request_charge |double |RU consumed for that operation |46.18; 1.0
db.cosmosdb.request_content_length |int |Request payload size in bytes |
db.cosmosdb.status_code |int |Cosmos DB status code. |200; 201
db.cosmosdb.sub_status_code |int |Cosmos DB sub status code. |1000; 1002

## Elasticsearch Attributes

Attribute |Type |Description |Examples
|--------------|-----------|------------|------------|
|db.elasticsearch.cluster.name |string |Represents the identifier of an Elasticsearch cluster. |e9106fc68e3044f0b1475b04bf4ffd5f|
|db.elasticsearch.node.name |string |Represents the human-readable identifier of the node/instance to which a request was routed. |instance-0000000001|
|db.elasticsearch.path_parts.<key> |string |A dynamic value in the url path. [1] |db.elasticsearch.path_parts.index=test-index; db.elasticsearch.path_parts.doc_id=123|

## JDBC Attributes

Attribute |Type |Description |Examples
|--------------|-----------|------------|------------|
|db.jdbc.driver_classname |string |The fully-qualified class name of the Java Database Connectivity (JDBC) driver used to connect. |org.postgresql.Driver; com.microsoft.sqlserver.jdbc.SQLServerDriver|

## MongoDB Attributes

Attribute |Type |Description |Examples
|--------------|-----------|------------|------------|
|db.mongodb.collection |string |The MongoDB collection being accessed within the database stated in db.name. |customers; products|

## MSSQL Attributes
Attribute |Type |Description |Examples
|--------------|-----------|------------|------------|
|db.mssql.instance_name |string |The Microsoft SQL Server instance name connecting to. This name is used to determine the port of a named instance. |

## Redis Attributes

Attribute |Type |Description |Examples
|--------------|-----------|------------|------------|
|db.redis.database_index |int |The index of the database being accessed as used in the SELECT command, provided as an integer. To be used instead of the generic db.name attribute. |0; 1; 15|

## SQL Attributes

|Attribute |Type |Description |Examples|
|--------------|-----------|------------|------------|
|db.sql.table |string |The name of the primary table that the operation is acting upon, including the database name (if applicable). [1] |public.users; |

# Destination Attributes

These attributes may be used to describe the receiver of a network exchange/packet. These should be used when there is no client/server relationship between the two sides, or when that relationship is unknown. This covers low-level network interactions (e.g. packet tracing) where you don’t know if there was a connection or which side initiated it. This also covers unidirectional UDP flows and peer-to-peer communication where the “user-facing” surface of the protocol / API does not expose a clear notion of client and server.

|Attribute|Type|Description|Examples|
|--------------|-----------|------------|------------|
|destination.address|string|Destination address - domain name if available without reverse DNS lookup; otherwise, IP address or Unix domain socket name. [1] | destination.example.com; 10.1.2.80; /tmp/my.sock|
|destination.port|int|Destination port number|3389; 2888|

[1]: When observed from the source side, and when communicating through an intermediary, destination.address SHOULD represent the destination address behind any intermediaries, for example proxies, if it’s available.

# Device Attributes

|Attribute|Type|Description|Examples|
|--------------|-----------|------------|------------|
|device.id|string|A unique identifier representing the device [1]|2ab2916d-a51f-4ac8-80ee-45ac31a28092|
|device.manufacturer|string|The name of the device manufacturer [2]|Apple; Samsung|
|device.model.identifier|string|The model identifier for the device [3]|iPhone3,4; SM-G920F|
|device.model.name|string|The marketing name for the device model [4]|iPhone 6s Plus; Samsung Galaxy S6|

# Disk Attributes

|Attribute|Type|Description|Examples|
|--------------|-----------|------------|------------|
|disk.io.direction|string|The disk IO operation direction.|read|

# Error Attributes

|Attribute|Type|Description|Examples|
|--------------|-----------|------------|------------|
|error.type|string|Stable |Describes a class of error the operation ended with. [1]|timeout; java.net.UnknownHostException;|


# Exception Attributes

|Attribute|Type|Description|Examples|
|--------------|-----------|------------|------------|
|exception.escaped|boolean|SHOULD be set to true if the exception event is recorded at a point where it is known that the exception is escaping the scope of the span. [1]|
|exception.message|string|The exception message.|Division by zero; Can't convert 'int' object to str implicitly
exception.stacktrace|string|A stacktrace as a string in the natural representation for the language runtime. The representation is to be determined and documented by each language SIG.|Exception in thread "main" java.lang.RuntimeException: Test exception\n at com.example.GenerateTrace.methodB(GenerateTrace.java:13)\n at com.example.GenerateTrace.methodA(GenerateTrace.java:9)\n at com.example.GenerateTrace.main(GenerateTrace.java:5)|
|exception.type|string|The type of the exception (its fully-qualified class name, if applicable). The dynamic type of the exception should be preferred over the static type in languages that support it.|

# Host Attributes

|Attribute|Type|Description|Examples|
|--------------|-----------|------------|------------|
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

# HTTP Attributes

|Attribute|Type|Description|Examples|
|--------------|-----------|------------|------------|
|http.request.body.size|int|The size of the request payload body in bytes. This is the number of bytes transferred excluding headers and is often, but not always, present as the Content-Length header. For requests using transport encoding, this should be the compressed size.|3495|
|http.request.header.<key>|string[]|Stable|HTTP request headers, <key> being the normalized HTTP Header name (lowercase), the value being the header values. [1]|http.request.header.content-type=["application/json"]; http.request.header.x-forwarded-for=["1.2.3.4", "1.2.3.5"]|
|http.request.method|string|Stable|HTTP request method. [2]|GET; POST; HEAD|
|http.request.method_original|string|Stable|Original HTTP method sent by the client in the request line.|GeT; ACL; foo|
|http.request.resend_count|int|Stable
The ordinal number of request resending attempt (for any reason, including redirects). [3]|3|
|http.response.body.size|int|The size of the response payload body in bytes. This is the number of bytes transferred excluding headers and is often, but not always, present as the Content-Length header. For requests using transport encoding, this should be the compressed size.|3495|
|http.response.header.<key>|string[]|Stable|HTTP response headers, <key> being the normalized HTTP Header name (lowercase), the value being the header values. [4]|http.response.header.content-type=["application/json"]; http.response.header.my-custom-header=["abc", "def"]|
|http.response.status_code|int|Stable|HTTP response status code.|200|
|http.route|string|Stable|The matched route, that is, the path template in the format used by the respective server framework. [5]|/users/:userID?; {controller}/{action}/{id?}|
