# OpenTelemetry metrics

## Baggage

Baggage is a set of application-defined properties contextually associated with a distributed request or workflow execution (see also the W3C Baggage Specification). Baggage can be used, among other things, to annotate telemetry, adding contextual information to metrics, traces, and logs.

In OpenTelemetry Baggage is represented as a set of name/value pairs describing user-defined properties.

### Propagtion
Baggage MAY be propagated across process boundaries or across any arbitrary boundaries (process, $OTHER_BOUNDARY1, $OTHER_BOUNDARY2, etc) for various reasons.

The API layer or an extension package MUST include the following Propagators:

    A TextMapPropagator implementing the W3C Baggage Specification.

See Propagators Distribution for how propagators are to be distributed.

## Context
A Context is a propagation mechanism which carries execution-scoped values across API boundaries and between logically associated execution units. Cross-cutting concerns access their data in-process using the same shared Context object.

A Context MUST be immutable, and its write operations MUST result in the creation of a new Context containing the original values and the specified values updated.

## Metrics type

    Counter
    Gauge
    Histogram
    Summary

## OpenMetrics

https://github.com/OpenObservability/OpenMetrics/blob/main/specification/OpenMetrics.md

https://prometheus.io/docs/instrumenting/exporters/



### Ceph



| metrics |  details |
|----------|------------|
|     ceph_rbd_read_bytes | RBD image bytes read|
|     ceph_rbd_read_latency_count| RBD image reads latency count|
|     ceph_rbd_read_latency_sum| RBD image reads latency total|
|     ceph_rbd_read_ops| RBD image reads count|
|     ceph_rbd_write_bytes| RBD image bytes written|
|     ceph_rbd_write_latency_count| RBD image writes latency count|
|     ceph_rbd_write_latency_sum| RBD image writes latency total|
|     ceph_rbd_write_ops| RBD image writes count|
|     ceph_mds_request| Total number of requests for the MDs daemon|
|     ceph_mds_reply_latency_sum| Reply latency total|
|     ceph_mds_reply_latency_count| Reply latency count|
|     ceph_mds_server_handle_client_request| Number of client requests|
|     ceph_mds_sessions_session_count| Session count|
|     ceph_mds_sessions_total_load| Total load|
|     ceph_mds_sessions_sessions_open| Sessions currently open|
|     ceph_mds_sessions_sessions_stale| Sessions currently stale|
|     ceph_objecter_op_r| Number of read operations|
|     ceph_objecter_op_w| Number of write operations|
|     ceph_mds_root_rbytes| Total number of bytes managed by the daemon|
|     ceph_mds_root_rfiles| Total number of files managed by the daemon|
|     ceph_rgw_put_initial_lat_count| Number of get operations|
|     ceph_rgw_put_initial_lat_sum| Total latency time for the PUT operations|
|     ceph_rgw_put| Total number of PUT operations|
|     ceph_rgw_get_b| Total bytes transferred in PUT operations|
| ceph_rgw_metadata| Provides generic information about the RGW daemon. It can be used together with other metrics to provide more contextual information in queries and graphs. Apart from the three common labels, this metric provides the following extra labels|
|     ceph_daemon| Name of the Ceph daemon. Example| ceph_daemon=”rgw.rgwtest.cephtest-node-00.sxizyq”,|
|     ceph_version| Version of Ceph daemon. Example| ceph_version=”ceph version 17.2.6 (d7ff0d10654d2280e08f1ab989c7cdf3064446a5) quincy (stable)”,|
|     hostname| Name of the host where the daemon runs. Example| hostname|”cephtest-node-00.cephlab.com”,|
| ceph_rgw_req| Number total of requests for the daemon (GET+PUT+DELETE)   Useful to detect bottlenecks and optimize load distribution.|
| ceph_rgw_qlen| RGW operations queue length for the daemon.|
|     Useful to detect bottlenecks and optimize load distribution.|
| ceph_rgw_failed_req| Aborted requests.|
|     ceph_rgw_get_initial_lat_count| Number of get operations|
|     ceph_rgw_get_initial_lat_sum| Total latency time for the GET operations|
|     ceph_rgw_get| Number total of GET requests|
|     ceph_rgw_get_b| Total bytes transferred in GET operations  contextual information in queries and graphs. Apart of the three common labels this metric provide the following extra labels,     compression_mode| compression used in the pool (lz4, snappy, zlib, zstd, none). Example| compression_mode=”none”| description| brief description of the pool type (replica|number of replicas or Erasure code| ec profile). Example| description=”replica|3”|
| ceph_pool_bytes_used| Total raw capacity consumed by user data and associated overheads by pool (metadata + redundancy)||
| ceph_pool_stored| Total of CLIENT data stored in the pool|
| ceph_pool_compress_under_bytes| Data eligible to be compressed in the pool|
| ceph_pool_compress_bytes_used| Data compressed in the pool|
| ceph_pool_rd| CLIENT read operations per pool (reads per second)|
| ceph_pool_rd_bytes| CLIENT read operations in bytes per pool|
| ceph_pool_wr| CLIENT write operations per pool (writes per second)|
| ceph_pool_wr_bytes| CLIENT write operation in bytes per pool|


### gitlab


| metrics |  type | since | descrition | labels |
|--------------|-----------|------------|------------|------------|
|gitlab_cache_misses_total |Counter |10.2 |	Cache read miss |controller, action, store, endpoint_id|
|gitlab_cache_operation_duration_seconds |Histogram |10.2 |Cache access time |operation, store, endpoint_id||
|gitlab_cache_operations_total | Counter | 12.2 | Cache operations by controller or action | controller, action, operation, store, endpoint_id|
|gitlab_cache_read_multikey_count | Histogram | 15.7 | Count of keys in multi-key cache read operations | controller, action, store, endpoint_id|
|gitlab_ci_pipeline_builder_scoped_variables_duration | Histogram | 14.5 | Time in seconds it takes to create the scoped variables for a CI/CD job|
|gitlab_ci_pipeline_creation_duration_seconds | Histogram | 13.0 | Time in seconds it takes to create a CI/CD pipeline | gitlab|
|gitlab_ci_pipeline_size_builds | Histogram | 13.1 | Total number of builds within a pipeline grouped by a pipeline source | source|
|gitlab_ci_runner_authentication_success_total | Counter | 15.2 | Total number of times that runner authentication has succeeded | type|
|gitlab_ci_runner_authentication_failure_total | Counter | 15.2 | Total number of times that runner authentication has failed|
|gitlab_ghost_user_migration_lag_seconds | Gauge | 15.6 | The waiting time in seconds of the oldest scheduled record for ghost user migration|
|gitlab_ghost_user_migration_scheduled_records_total | Gauge | 15.6 | The total number of scheduled ghost user migrations|
|gitlab_ci_active_jobs | Histogram | 14.2 | Count of active jobs when pipeline is created|
|gitlab_database_transaction_seconds | Histogram | 12.1 | Time spent in database transactions, in seconds|
|gitlab_method_call_duration_seconds | Histogram | 10.2 | Method calls real duration | controller, action, module, method|
|gitlab_omniauth_login_total | Counter | 16.1 | Total number of OmniAuth logins attempts | omniauth_provider, status|
|gitlab_page_out_of_bounds | Counter | 12.8 | Counter for the PageLimiter pagination limit being hit | controller, action, bot|
|gitlab_rails_boot_time_seconds | Gauge | 14.8 | Time elapsed for Rails primary process to finish startup|
|gitlab_rails_queue_duration_seconds | Histogram | 9.4 | Measures latency between GitLab Workhorse forwarding a request to Rails|
|gitlab_sql_duration_seconds | Histogram | 10.2 | SQL execution time, excluding SCHEMA operations and BEGIN / COMMIT|
|gitlab_sql_<role>_duration_seconds | Histogram | 13.10 | SQL execution time, excluding SCHEMA operations and BEGIN / COMMIT, grouped by database roles (primary/replica)|
|gitlab_ruby_threads_max_expected_threads | Gauge | 13.3 | Maximum number of threads expected to be running and performing application work|
|gitlab_ruby_threads_running_threads | Gauge | 13.3 | Number of running Ruby threads by name|
|gitlab_transaction_cache_<key>_count_total | Counter | 10.2 | Counter for total Rails cache calls (per key)|
|gitlab_transaction_cache_<key>_duration_total | Counter | 10.2 | Counter for total time (seconds) spent in Rails cache calls (per key)|
|gitlab_transaction_cache_count_total | Counter | 10.2 | Counter for total Rails cache calls (aggregate)|
|gitlab_transaction_cache_duration_total | Counter | 10.2 | Counter for total time (seconds) spent in Rails cache calls (aggregate)|
|gitlab_transaction_cache_read_hit_count_total | Counter | 10.2 | Counter for cache hits for Rails cache calls | controller, action, store, endpoint_id|
|gitlab_transaction_cache_read_miss_count_total | Counter | 10.2 | Counter for cache misses for Rails cache calls | controller, action, store, endpoint_id|
|gitlab_transaction_duration_seconds | Histogram | 10.2 | Duration for successful requests (gitlab_transaction_* metrics) | controller, action, endpoint_id|
|gitlab_transaction_event_build_found_total | Counter | 9.4 | Counter for build found for API /jobs/request|
|gitlab_transaction_event_build_invalid_total | Counter | 9.4 | Counter for build invalid due to concurrency conflict for API /jobs/request|
|gitlab_transaction_event_build_not_found_cached_total | Counter | 9.4 | Counter for cached response of build not found for API /jobs/request|
|gitlab_transaction_event_build_not_found_total | Counter | 9.4 | Counter for build not found for API /jobs/request|
|gitlab_transaction_event_change_default_branch_total | Counter | 9.4 | Counter when default branch is changed for any repository|
|gitlab_transaction_event_create_repository_total | Counter | 9.4 | Counter when any repository is created|
|gitlab_transaction_event_etag_caching_cache_hit_total | Counter | 9.4 | Counter for ETag cache hit. | endpoint|
|gitlab_transaction_event_etag_caching_header_missing_total | Counter | 9.4 | Counter for ETag cache miss - header missing | endpoint|
|gitlab_transaction_event_etag_caching_key_not_found_total | Counter | 9.4 | Counter for ETag cache miss - key not found | endpoint|
|gitlab_transaction_event_etag_caching_middleware_used_total | Counter | 9.4 | Counter for ETag middleware accessed | endpoint|
|gitlab_transaction_event_etag_caching_resource_changed_total | Counter | 9.4 | Counter for ETag cache miss - resource changed | endpoint|
|gitlab_transaction_event_fork_repository_total | Counter | 9.4 | Counter for repository forks (RepositoryForkWorker). Only incremented when source repository exists|
|gitlab_transaction_event_import_repository_total | Counter | 9.4 | Counter for repository imports (RepositoryImportWorker)|
|gitlab_transaction_event_patch_hard_limit_bytes_hit_total | Counter | 13.9 | Counter for diff patch size limit hits|
|gitlab_transaction_event_push_branch_total | Counter | 9.4 | Counter for all branch pushes|
|gitlab_transaction_event_rails_exception_total | Counter | 9.4 | Counter for number of rails exceptions|
|gitlab_transaction_event_receive_email_total | Counter | 9.4 | Counter for received emails | handler|
|gitlab_transaction_event_remote_mirrors_failed_total | Counter | 10.8 | Counter for failed remote mirrors|
|gitlab_transaction_event_remote_mirrors_finished_total | Counter | 10.8 | Counter for finished remote mirrors|
|gitlab_transaction_event_remote_mirrors_running_total | Counter | 10.8 | Counter for running remote mirrors|
|gitlab_transaction_event_remove_branch_total | Counter | 9.4 | Counter when a branch is removed for any repository|
|gitlab_transaction_event_remove_repository_total | Counter | 9.4 | Counter when a repository is removed|
|gitlab_transaction_event_remove_tag_total | Counter | 9.4 | Counter when a tag is remove for any repository|
|gitlab_transaction_event_sidekiq_exception_total | Counter | 9.4 | Counter of Sidekiq exceptions|
|gitlab_transaction_event_stuck_import_jobs_total | Counter | 9.4 | Count of stuck import jobs | projects_without_jid_count, projects_with_jid_count|
|gitlab_transaction_event_update_build_total | Counter | 9.4 | Counter for update build for API /jobs/request/:id|
|gitlab_transaction_new_redis_connections_total | Counter | 9.4 | Counter for new Redis connections|
|gitlab_transaction_rails_queue_duration_total | Counter | 9.4 | Measures latency between GitLab Workhorse forwarding a request to Rails | controller, action, endpoint_id|
|gitlab_transaction_view_duration_total | Counter | 9.4 | Duration for views | controller, action, view, endpoint_id|
|gitlab_view_rendering_duration_seconds | Histogram | 10.2 | Duration for views (histogram) | controller, action, view, endpoint_id|
|http_requests_total | Counter | 9.4 | Rack request count | method, status|
|http_request_duration_seconds | Histogram | 9.4 | HTTP response time from rack middleware for successful requests | method|
|gitlab_transaction_db_count_total | Counter | 13.1 | Counter for total number of SQL calls | controller, action, endpoint_id|
|gitlab_transaction_db_<role>_count_total | Counter | 13.10 | Counter for total number of SQL calls, grouped by database roles (primary/replica) | controller, action, endpoint_id|
|gitlab_transaction_db_write_count_total | Counter | 13.1 | Counter for total number of write SQL calls | controller, action, endpoint_id|
|gitlab_transaction_db_cached_count_total | Counter | 13.1 | Counter for total number of cached SQL calls | controller, action, endpoint_id|
|gitlab_transaction_db_<role>_cached_count_total | Counter | 13.1 | Counter for total number of cached SQL calls, grouped by database roles (primary/replica) | controller, action, endpoint_id|
|gitlab_transaction_db_<role>_wal_count_total | Counter | 14.0 | Counter for total number of WAL (write ahead log location) queries, grouped by database roles (primary/replica) | controller, action, endpoint_id|
|gitlab_transaction_db_<role>_wal_cached_count_total | Counter | 14.1 | Counter for total number of cached WAL (write ahead log location) queries, grouped by database roles (primary/replica) | controller, action, endpoint_id|
|http_elasticsearch_requests_duration_seconds | Histogram | 13.1 | Elasticsearch requests duration during web transactions. Premium and Ultimate only. | controller, action, endpoint_id|
|http_elasticsearch_requests_total | Counter | 13.1 | Elasticsearch requests count during web transactions. Premium and Ultimate only. | controller, action, endpoint_id|
|pipelines_created_total | Counter | 9.4 | Counter of pipelines created|
|rack_uncaught_errors_total | Counter | 9.4 | Rack connections handling uncaught errors count|
|user_session_logins_total | Counter | 9.4 | Counter of how many users have logged in since GitLab was started or restarted|
|upload_file_does_not_exist | Counter | 10.7 | Number of times an upload record could not find its file.|
|failed_login_captcha_total | Gauge | 11.0 | Counter of failed CAPTCHA attempts during login|
|successful_login_captcha_total | Gauge | 11.0 | Counter of successful CAPTCHA attempts during login|
|auto_devops_pipelines_completed_total | Counter | 12.7 | Counter of completed Auto DevOps pipelines, labeled by status|
|artifact_report_<report_type>_builds_completed_total | Counter | 15.3 | Counter of completed CI Builds with report-type artifacts, grouped by report type and labeled by status|
|gitlab_metrics_dashboard_processing_time_ms | Summary | 12.10 | Metrics dashboard processing time in milliseconds | service, stages|
|action_cable_active_connections | Gauge | 13.4 | Number of ActionCable WS clients currently connected | server_mode|
|action_cable_broadcasts_total | Counter | 13.10 | The number of ActionCable broadcasts emitted | server_mode|
|action_cable_pool_min_size | Gauge | 13.4 | Minimum number of worker threads in ActionCable thread pool | server_mode|
|action_cable_pool_max_size | Gauge | 13.4 | Maximum number of worker threads in ActionCable thread pool | server_mode|
|action_cable_pool_current_size | Gauge | 13.4 | Current number of worker threads in ActionCable thread pool | server_mode|
|action_cable_pool_largest_size | Gauge | 13.4 | Largest number of worker threads observed so far in ActionCable thread pool | server_mode|
|action_cable_pool_pending_tasks | Gauge | 13.4 | Number of tasks waiting to be executed in ActionCable thread pool | server_mode|
|action_cable_pool_tasks_total | Gauge | 13.4 | Total number of tasks executed in ActionCable thread pool | server_mode|
|gitlab_ci_trace_operations_total | Counter | 13.4 | Total amount of different operations on a build trace | operation|
|gitlab_ci_trace_bytes_total | Counter | 13.4 | Total amount of build trace bytes transferred|
|action_cable_single_client_transmissions_total | Counter | 13.10 | The number of ActionCable messages transmitted to any client in any channel | server_mode|
|action_cable_subscription_confirmations_total | Counter | 13.10 | The number of ActionCable subscriptions from clients confirmed | server_mode|
|action_cable_subscription_rejections_total | Counter | 13.10 | The number of ActionCable subscriptions from clients rejected | server_mode|
|action_cable_transmitted_bytes_total | Counter | 16.0 | Total number of bytes transmitted over ActionCable | operation, channel|
|gitlab_issuable_fast_count_by_state_total | Counter | 13.5 | Total number of row count operations on issue/merge request list pages|
|gitlab_issuable_fast_count_by_state_failures_total | Counter | 13.5 | Number of soft-failed row count operations on issue/merge request list pages|
|gitlab_ci_trace_finalize_duration_seconds | Histogram | 13.6 | Duration of build trace chunks migration to object storage|
|gitlab_vulnerability_report_branch_comparison_real_duration_seconds | Histogram | 15.11 | Execution duration of vulnerability report present on default branch SQL query|
|gitlab_vulnerability_report_branch_comparison_cpu_duration_seconds | Histogram | 15.11 | Execution duration of vulnerability report present on default branch SQL query|
|gitlab_external_http_total | Counter | 13.8 | Total number of HTTP calls to external systems | controller, action, endpoint_id|
|gitlab_external_http_duration_seconds | Counter | 13.8 | Duration in seconds spent on each HTTP call to external systems|
|gitlab_external_http_exception_total | Counter | 13.8 | Total number of exceptions raised when making external HTTP calls|
|ci_report_parser_duration_seconds | Histogram | 13.9 | Time to parse CI/CD report artifacts | parser|
|pipeline_graph_link_calculation_duration_seconds | Histogram | 13.9 | Total time spent calculating links, in seconds|
|pipeline_graph_links_total | Histogram | 13.9 | Number of links per graph|
|pipeline_graph_links_per_job_ratio | Histogram | 13.9 | Ratio of links to job per graph|
|gitlab_ci_pipeline_security_orchestration_policy_processing_duration_seconds | Histogram | 13.12 | Time in seconds it takes to process Security Policies in CI/CD pipeline|
|gitlab_spamcheck_request_duration_seconds | Histogram | 13.12 | The duration for requests between Rails and the anti-spam engine|
|service_desk_thank_you_email | Counter | 14.0 | Total number of email responses to new Service Desk emails|
|service_desk_new_note_email | Counter | 14.0 | Total number of email notifications on new Service Desk comment|
|email_receiver_error | Counter | 14.1 | Total number of errors when processing incoming emails|
|gitlab_snowplow_events_total | Counter | 14.1 | Total number of GitLab Snowplow Analytics Instrumentation events emitted|
|gitlab_snowplow_failed_events_total | Counter | 14.1 | Total number of GitLab Snowplow Analytics Instrumentation events emission failures|
|gitlab_snowplow_successful_events_total | Counter | 14.1 | Total number of GitLab Snowplow Analytics Instrumentation events emission successes|
|gitlab_ci_build_trace_errors_total | Counter | 14.4 | Total amount of different error types on a build trace | error_reason|
|gitlab_presentable_object_cacheless_render_real_duration_seconds | Histogram | 15.3 | Duration of real time spent caching and representing specific web request objects | controller, action, endpoint_id|
|cached_object_operations_total | Counter | 15.3 | Total number of objects cached for specific web requests | controller, action, endpoint_id|
|redis_hit_miss_operations_total | Counter | 15.6 | Total number of Redis cache hits and misses | cache_hit, cache_identifier, feature_category, backing_resource|
|redis_cache_generation_duration_seconds | Histogram | 15.6 | Time to generate Redis cache | cache_hit, cache_identifier, feature_category, backing_resource|
|gitlab_diffs_reorder_real_duration_seconds | Histogram | 15.8 | Duration in seconds spend on reordering of diff files on diffs batch request | controller, action, endpoint_id|
|gitlab_diffs_collection_real_duration_seconds | Histogram | 15.8 | Duration in seconds spent on querying merge request diff files on diffs batch request | controller, action, endpoint_id|
|gitlab_diffs_comparison_real_duration_seconds | Histogram | 15.8 | Duration in seconds spent on getting comparison data on diffs batch request | controller, action, endpoint_id|
|gitlab_diffs_unfoldable_positions_real_duration_seconds | Histogram | 15.8 | Duration in seconds spent on getting unfoldable note positions on diffs batch request | controller, action|
|gitlab_diffs_unfold_real_duration_seconds | Histogram | 15.8 | Duration in seconds spent on unfolding positions on diffs batch request | controller, action, endpoint_id|
|gitlab_diffs_write_cache_real_duration_seconds | Histogram | 15.8 | Duration in seconds spent on caching highlighted lines and stats on diffs batch request | controller, action, endpoint_id|
|gitlab_diffs_highlight_cache_decorate_real_duration_seconds | Histogram | 15.8 | Duration in seconds spent on setting highlighted lines from cache on diffs batch request | controller, action, endpoint_id|
|gitlab_diffs_render_real_duration_seconds | Histogram | 15.8 | Duration in seconds spent on serializing and rendering diffs on diffs batch request | controller, action, endpoint_id|
|gitlab_memwd_violations_total | Counter | 15.9 | Total number of times a Ruby process violated a memory threshold|
|gitlab_memwd_violations_handled_total | Counter | 15.9 | Total number of times Ruby process memory violations were handled|
|gitlab_sli_rails_request_apdex_total | Counter | 14.4 | Total number of request Apdex measurements. For more information, see Rails request SLIs | endpoint_id, feature_category, request_urgency|
|gitlab_sli_rails_request_apdex_success_total | Counter | 14.4 | Total number of successful requests that met the target duration for their urgency. Divide by gitlab_sli_rails_requests_apdex_total to get a success ratio | endpoint_id, feature_category, request_urgency|
|gitlab_sli_rails_request_error_total | Counter | 15.7 | Total number of request error measurements. For more information, see Rails request SLIs | endpoint_id, feature_category, request_urgency, error|
|job_register_attempts_failed_total | Counter | 9.5 | Counts the times a runner fails to register a job|
|job_register_attempts_total | Counter | 9.5 | Counts the times a runner tries to register a job|
|job_queue_duration_seconds | Histogram | 9.5 | Request handling execution time|
|gitlab_ci_queue_operations_total | Counter | 16.3 | Counts all the operations that are happening inside a queue|
|gitlab_ci_queue_depth_total | Histogram | 16.3 | Size of a CI/CD builds queue in relation to the operation result|
|gitlab_ci_queue_size_total | Histogram | 16.3 | Size of initialized CI/CD builds queue|
|gitlab_ci_current_queue_size | Gauge | 16.3 | Current size of initialized CI/CD builds queue|
|gitlab_ci_queue_iteration_duration_seconds | Histogram | 16.3 | Time it takes to find a build in CI/CD queue|
¨|gitlab_ci_queue_retrieval_duration_seconds | Histogram | 16.3 | Time it takes to execute a SQL query to retrieve builds queue|
|gitlab_connection_pool_size | Gauge | 16.7 | Size of connection pool|
|gitlab_connection_pool_available_count | Gauge | 16.7 | Number of available connections in the pool|
|gitlab_security_policies_scan_result_process_duration_seconds | Histogram | 16.7 | The amount of time to process merge request approval policies|
|gitlab_highlight_usage | Counter | 16.8 | The number of times Gitlab::Highlight is used | used_on|
|dependency_linker_usage | Counter | 16.8 | The number of times dependency linker is used | used_on