# Monitoring Interview Questions

## Prometheus & Grafana

Q: What is Prometheus?\
Ans: An open-source monitoring and time-series database system that scrapes (pulls) metrics from configured targets over HTTP at regular intervals, stores them, and lets you query them via PromQL — commonly paired with Grafana for visualization and Alertmanager for alerting.

Q: What is Grafana?\
Ans: An open-source visualization and dashboarding tool that queries data sources (Prometheus, CloudWatch, Elasticsearch, Loki, etc.) and renders them as customizable graphs, tables, and alerts on shareable dashboards.

Q: What is Node Exporter?\
Ans: A Prometheus exporter that runs on a host and exposes hardware/OS-level metrics (CPU, memory, disk, network) in Prometheus's scrape format, so Prometheus can collect standard machine-level metrics from any Linux/Windows host.

Q: How would you set up Prometheus and Grafana?\
Ans: Deploy Prometheus (via Helm chart on Kubernetes, or as a binary/container elsewhere) with a `prometheus.yml` config defining scrape targets (via static configs or service discovery); deploy exporters (Node Exporter, kube-state-metrics, application `/metrics` endpoints) on targets; deploy Grafana and add Prometheus as a data source (its HTTP endpoint); build/import dashboards querying the collected metrics; configure Alertmanager for notification routing on top.

Q: How do you configure Prometheus to scrape multiple microservices?\
Ans: Add a `scrape_config` job per service (or one job using relabeling to distinguish services) specifying targets — static IP:port entries, or dynamic service discovery (Kubernetes SD, Consul SD, EC2 SD) so Prometheus automatically discovers new instances as services scale.

Q: How do you configure Prometheus to scrape all namespaces?\
Ans: Use `kubernetes_sd_configs` with role `pod` or `endpoints` without namespace restriction (omit `namespaces.names`, or leave it empty so it watches all namespaces), then use `relabel_configs` to filter which pods to actually scrape based on annotations like `prometheus.io/scrape: "true"`.

Q: How do you set up alerts in Prometheus?\
Ans: Define alerting rules in Prometheus (`groups: - rules: - alert: HighCPU expr: ... for: 5m`) that evaluate a PromQL expression continuously; when the condition holds for the configured duration, Prometheus fires the alert to **Alertmanager**, which handles deduplication, grouping, silencing, and routing to receivers (Slack, PagerDuty, email, etc.).

Q: How do you monitor microservices using Prometheus and Grafana?\
Ans: Instrument each service to expose a `/metrics` endpoint (via a client library — e.g., `prometheus-client`), have Prometheus scrape all instances via service discovery, build Grafana dashboards per service (and an overview dashboard) tracking the RED metrics (Rate, Errors, Duration) plus resource usage, and set alerting rules on SLO-relevant thresholds.

Q: Useful PromQL queries for CPU, memory, errors, restarts?\
Ans:
- CPU: `rate(container_cpu_usage_seconds_total[5m])`
- Memory: `container_memory_working_set_bytes`
- Error rate: `sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))`
- Pod restarts: `increase(kube_pod_container_status_restarts_total[1h])`

Q: Explain a Grafana dashboard you created.\
Ans: A representative example: a service-overview dashboard with panels for request rate, p50/p95/p99 latency, error rate (all from `http_requests_total`/`http_request_duration_seconds` via PromQL), pod CPU/memory usage (`kube-state-metrics` + `cAdvisor`), and restart counts — with template variables for namespace/service so one dashboard covers all environments, plus alert thresholds annotated directly on the latency and error-rate panels.

Q: What types of monitoring have you implemented?\
Ans: Infrastructure monitoring (CPU/memory/disk/network via Node Exporter/CloudWatch), application/service monitoring (request rate, latency, error rate — RED metrics), Kubernetes monitoring (pod/node health, restarts, resource usage via kube-state-metrics), log-based monitoring/alerting (ELK/Loki), synthetic/uptime checks, and distributed tracing for request-flow visibility.

Q: What kind of metrics do you monitor in production?\
Ans: Golden signals: **latency** (response time percentiles), **traffic** (request rate), **errors** (error rate/count by type), and **saturation** (CPU, memory, disk, queue depth, connection pool usage) — plus business-level metrics (orders processed, signups) where relevant.

Q: How do you collect logs in Grafana?\
Ans: Via **Grafana Loki** — a log aggregation system designed to work like Prometheus but for logs (indexes only labels, not full text), with Promtail (or another agent) shipping logs from hosts/containers into Loki, queried in Grafana using LogQL alongside metric dashboards.

Q: How do you debug issues using Prometheus and Grafana?\
Ans: Start from an alert or a dashboard anomaly, drill into the relevant panel's time range to correlate with a deploy/traffic spike, cross-reference related metrics (e.g., high latency panel next to CPU/memory/error-rate panels) to isolate whether it's resource saturation, an error spike, or a dependency issue, then pivot to logs/traces for the affected time window to find the root cause.

## ELK Stack

Q: What is ELK Stack?\
Ans: Elasticsearch, Logstash, and Kibana — a log aggregation and analysis stack: Logstash (or Filebeat/other shippers) collects and processes logs, Elasticsearch indexes and stores them for fast search, and Kibana visualizes/queries them.

Q: What is Elasticsearch?\
Ans: A distributed, JSON-document search and analytics engine (built on Apache Lucene) that indexes data for fast full-text search and aggregation — the storage/query layer of the ELK stack.

Q: What is Logstash?\
Ans: A server-side data processing pipeline that ingests logs from multiple sources, transforms/parses/enriches them (via filters like `grok`), and ships them to a destination (typically Elasticsearch).

Q: What is Kibana?\
Ans: The visualization and exploration UI for Elasticsearch — used to build dashboards, run ad-hoc searches, and set up alerts over indexed log/metric data.

Q: How do logs flow through ELK?\
Ans: Application/host logs → collected by a lightweight shipper (Filebeat) or Logstash directly → optionally parsed/enriched/filtered by Logstash → indexed into Elasticsearch → queried and visualized in Kibana.

Q: Logs are not appearing in Kibana — what do you check?\
Ans: Whether the shipper (Filebeat/Logstash) is actually running and reading the source log files; Logstash pipeline errors (bad grok patterns silently dropping events); whether the Elasticsearch index exists and matches the Kibana index pattern; the Kibana time-range filter (often the culprit — logs exist but are outside the selected window); network connectivity/auth between the shipper and Elasticsearch; and Elasticsearch cluster health/disk watermarks (a red/yellow cluster can reject writes).

Q: How do you design an ELK stack?\
Ans: Lightweight shippers (Filebeat) on each host/container send logs to a Logstash tier (or directly to Elasticsearch, using Elasticsearch ingest pipelines for simpler cases) for parsing/enrichment; a properly-sized Elasticsearch cluster with appropriate shard/replica counts and index lifecycle management (hot-warm-cold tiers, retention policies) to control storage growth; Kibana on top for dashboards and alerting; and a queue (Kafka/Redis) in front of Logstash for buffering during ingestion spikes in high-volume setups.

Q: How do you set alerts in Kibana?\
Ans: Use Kibana Alerting (rules based on Elasticsearch queries — threshold, anomaly, or custom queries) which evaluate on a schedule and trigger connectors (email, Slack, webhook, PagerDuty) when conditions are met.

Q: What log collectors have you used?\
Ans: Common options: Filebeat/Metricbeat (lightweight, Elastic ecosystem), Fluentd/Fluent Bit (CNCF, popular in Kubernetes), Logstash (heavier, more processing power), Promtail (for Loki), and cloud-native agents like the CloudWatch Agent.

## CloudWatch

Q: What is CloudWatch?\
Ans: AWS's native monitoring and observability service — collects metrics, logs, and events from AWS resources and applications, supports dashboards, alarms, and automated actions.

Q: What are CloudWatch Metrics?\
Ans: Time-ordered data points representing a resource's performance/behavior over time (e.g., CPUUtilization, RequestCount) — either published automatically by AWS services or as custom metrics from your own application code.

Q: What are CloudWatch Alarms?\
Ans: Watchers on a metric that transition between OK/ALARM/INSUFFICIENT_DATA states based on a threshold over an evaluation period, capable of triggering actions like SNS notifications, Auto Scaling actions, or EC2 recovery.

Q: How do you send alerts from CloudWatch?\
Ans: Create a CloudWatch Alarm on the relevant metric, and set its alarm action to publish to an **SNS topic**, which fans out to subscribers (email, SMS, Lambda, Slack via a Lambda subscriber, PagerDuty, etc.).

Q: Difference between CloudWatch and CloudTrail?\
Ans: CloudWatch monitors operational **performance/health** (metrics, logs, alarms — "how is it running"). CloudTrail records **API activity/audit trail** (who called what API, when, from where — "who did what") for governance, compliance, and security investigation.

Q: What is AWS X-Ray?\
Ans: A distributed tracing service that tracks requests as they travel through multiple services/components, producing a service map and per-segment latency breakdown — used to pinpoint bottlenecks/errors in distributed/microservice architectures.

Q: How do you reduce CloudWatch costs?\
Ans: Reduce log retention periods, filter/exclude noisy or unnecessary log groups before ingestion, use metric filters instead of shipping full logs where only a count/pattern is needed, lower custom metric resolution/cardinality (avoid high-cardinality dimensions), consolidate alarms, and use CloudWatch Logs Insights sparingly (billed per GB scanned) rather than broad ad-hoc queries.

## General Monitoring Concepts

Q: Difference between Monitoring and Logging?\
Ans: Monitoring tracks the ongoing **state/health** of a system in near-real-time via metrics and alerts ("is it healthy right now"). Logging records **discrete events** that occurred, useful for after-the-fact investigation and detailed context ("what exactly happened, and why").

Q: Difference between Metrics and Logs?\
Ans: **Metrics** are numeric, aggregatable time-series data (efficient to store/query, good for trends/alerting, low detail). **Logs** are discrete, often unstructured/text records of individual events (rich detail, more expensive to store/search at volume, better for root-cause investigation than trend detection).

Q: What is the difference between Logs, Metrics, and Traces?\
Ans: **Logs** — discrete event records with full context. **Metrics** — aggregated numeric measurements over time. **Traces** — the path and timing of a single request as it flows across multiple services, showing where time was spent end-to-end. Together they form the three pillars of observability, each answering a different kind of question.

Q: Benefits of having a single monitoring solution?\
Ans: One pane of glass instead of context-switching across tools; consistent alerting/on-call workflow; easier correlation across metrics/logs/traces for the same incident; simpler tooling/training/maintenance overhead; and centralized access control/auditing.

Q: CloudWatch vs New Relic?\
Ans: CloudWatch is deeply integrated with AWS services natively and cost-effective if you're AWS-only, but has a steeper/clunkier UX for cross-service correlation and APM. New Relic (and similar APM tools like Datadog) offers richer application performance monitoring, cross-cloud/hybrid support, and generally better developer UX for tracing/dashboards, at higher cost and requiring agent instrumentation.

## Advanced Scenarios

Q: How do you monitor Kubernetes nodes and pods?\
Ans: Deploy `kube-state-metrics` (cluster object state — pod status, deployment replicas) and Node Exporter/cAdvisor (resource usage) alongside Prometheus (often via the kube-prometheus-stack Helm chart), visualized in Grafana with pre-built Kubernetes dashboards, and alert on pod restarts, OOMKills, node pressure conditions, and resource saturation.

Q: How do you monitor an application if developers forgot to add a health endpoint?\
Ans: Fall back to external signals: TCP/port-level checks (is it listening), process-level checks (is the process running, via Node Exporter's process metrics), infrastructure metrics (CPU/memory), log-based health inference (error rate in logs), or synthetic transaction monitoring (simulate a real user request from outside) until a proper `/health` endpoint can be added.

Q: How would you create a health check endpoint outside the application?\
Ans: Deploy a lightweight sidecar/wrapper (e.g., an NGINX or small script) that checks the application's actual port/process/dependency reachability and exposes a simple `/health` HTTP endpoint reflecting that — or use infrastructure-level checks (an ALB/NLB TCP health check on the app's port) as a stand-in until a real in-app endpoint exists.

Q: How do you expose health status based on CPU usage?\
Ans: Have the health-check wrapper/sidecar read current CPU usage (e.g., from `/proc/stat` or a metrics endpoint) and return a degraded/unhealthy status if it exceeds a defined threshold, so load balancers or orchestrators can route around an overloaded instance even without app-level awareness.

Q: How do you identify whether an issue is infrastructure-level or application-level?\
Ans: Check infrastructure metrics first (CPU/memory/disk/network on the host, and whether the underlying node/instance is healthy) — if those are normal but the app is failing/slow, it's likely application-level (bug, bad deploy, dependency failure, connection pool exhaustion); correlate with recent deploys, and check whether the issue is isolated to one instance/pod (application/instance-specific) or affects all instances uniformly (more likely infrastructure or a shared dependency).

Q: How do you analyze 500 errors without a log aggregator?\
Ans: SSH/exec into the affected instance(s)/pod(s) directly and `tail -f`/`grep` the application and web server logs (e.g., `journalctl`, `/var/log/nginx/error.log`, `kubectl logs`) around the time of the errors, check exit codes/stack traces, correlate with `dmesg`/OOM killer logs and resource usage (`top`, `free`) at that timestamp, and check upstream/downstream dependency logs (DB, cache) if the app logs point to a timeout.
