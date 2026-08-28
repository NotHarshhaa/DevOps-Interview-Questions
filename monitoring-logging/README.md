# **Monitoring, Logging & Observability - DevOps Interview Questions (200 Questions)**

Welcome to the **Monitoring, Logging & Observability** master collection containing **200 comprehensive interview questions and detailed answers** covering OpenTelemetry (OTel), Prometheus, PromQL, Grafana, Loki (LogQL), Tempo (Distributed Tracing), Thanos, Mimir, Alertmanager, and eBPF Continuous Profiling.

---

## 🟢 **Part 1: Observability Fundamentals, Metrics & Prometheus (Questions 1–60)**

### **1. What is the fundamental difference between Monitoring and Observability?**
**Answer:** Monitoring tracks predefined metrics against static thresholds ("Is the system working?"). Observability is a property of a system that allows engineers to infer its internal state based on its external telemetry outputs (metrics, logs, traces, profiles), enabling them to answer "Why is the system broken?" for unforeseen failure modes.

### **2. Explain the Three Pillars of Observability and how they correlate during an incident.**
**Answer:**
1. **Metrics (Aggregable Numbers):** Lightweight time-series data for high-level alerting and health dashboards (`rate(http_requests_total[5m])`).
2. **Logs (Discrete Events):** Timestamped structured JSON events recording specific execution context (`{"error":"DB connection timeout"}`).
3. **Traces (Request Journeys):** End-to-end request propagation across microservices showing individual span latencies and database query durations.
- **Correlation:** Alert fires on **Metric** (p99 latency) $\rightarrow$ Engineer clicks **Exemplar** to view distributed **Trace** (Tempo) to identify slow database query $\rightarrow$ Jumps to correlated **Log** (Loki) via `trace_id` to view exact SQL exception.

### **3. Explain the Four Golden Signals of Monitoring (Google SRE).**
**Answer:**
1. **Latency:** Time required to service a request (differentiating between successful vs failed request latency).
2. **Traffic:** Demand placed on the system (HTTP req/sec, concurrent streaming sessions).
3. **Errors:** Rate of requests that fail explicitly (HTTP 5xx) or implicitly (wrong data returned).
4. **Saturation:** How full a resource is (CPU %, memory %, queue depth).

### **4. Compare the RED Method vs USE Method.**
**Answer:**
- **RED (For Request-Driven Services / Microservices):** **R**ate (req/sec), **E**rrors (failed req/sec), **D**uration (latency distribution).
- **USE (For Infrastructure & Hardware Resources):** **U**tilization (% time resource was busy), **S**aturation (queue backlog), **E**rrors (error event count).

### **5. What is Prometheus and how does its pull-based architecture work?**
**Answer:** An open-source, time-series database and monitoring system. It periodically scrapes (pulls) plain-text metrics from HTTP `/metrics` endpoints exposed by applications and exporters using Kubernetes Service Discovery, storing samples in its custom local TSDB engine.

### **6. Explain the four core Prometheus Metric Types.**
**Answer:**
1. **Counter:** Monotonically increasing cumulative metric (resets only on restart), e.g., `http_requests_total`.
2. **Gauge:** A numerical value that can go up and down arbitrarily, e.g., `node_memory_utilization_bytes`.
3. **Histogram:** Samples observations (request durations) and counts them in configurable bucket intervals (`le="0.1"`, `le="0.5"`).
4. **Summary:** Samples observations and calculates client-side phi-quantiles (p50, p90, p99) over a sliding time window.

### **7. What is a Prometheus Exporter?**
**Answer:** A lightweight proxy agent that translates telemetry from third-party systems that do not natively emit Prometheus metrics and exposes them on `/metrics` (e.g., `node_exporter`, `mysqld_exporter`, `blackbox_exporter`).

### **8. In PromQL, what is the crucial difference between `rate()` and `irate()`?**
**Answer:**
- **`rate(v[range])`:** Calculates average per-second rate of increase across the entire range window (e.g., `rate(http_requests_total[5m])`). Spikes are smoothed out. **Mandatory for alerting rules and SLO calculations.**
- **`irate(v[range])`:** Instant rate based strictly on the **last two data points** in the range. Shows volatile high-frequency spikes; ideal for real-time dashboards.

### **9. How does `histogram_quantile()` work in PromQL to calculate p99 latency?**
**Answer:**
```promql
histogram_quantile(
  0.99,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service)
)
```
Interpolates within the bucket boundaries (`le`) where the 99th percentile falls to estimate the p99 latency value across all service instances.

### **10. What is High Cardinality in metrics and why does it crash Prometheus?**
**Answer:** Cardinality is the total number of unique time series created by multiplying all label combinations. Adding high-cardinality labels (`user_id`, `email`, `order_id`) causes exponential memory growth in Prometheus TSDB index, causing OOM crashes. High-cardinality data belongs in **Logs or Traces**, never metric labels.

### **11. What are Prometheus Exemplars?**
**Answer:** References to specific Trace IDs attached directly to metric samples in Prometheus TSDB. Clicking on a latency spike data point in Grafana opens the exact distributed trace in Grafana Tempo in 1 click.

### **12. What is Grafana and what is its role in modern observability?**
**Answer:** An open-source visualization and analytics platform that connects to heterogeneous data sources (Prometheus, Loki, Tempo, Elasticsearch, CloudWatch) to render unified dashboards, alerts, and correlated telemetry views.

### **13. What is Alertmanager in Prometheus?**
**Answer:** Handles alerts sent by client applications like Prometheus. Responsible for **deduplication, grouping, routing, inhibition, and silencing** before dispatching notifications to PagerDuty, Slack, Opsgenie, or email.

### **14. What is Alertmanager Inhibition and Grouping?**
**Answer:**
- **Grouping:** Batches multiple related alerts into a single notification (e.g., 50 crashing pods on a node = 1 Slack notification).
- **Inhibition:** Mutes lower-priority alerts if a critical related alert is already firing (e.g., if `NodeDown` fires, inhibit `PodCrashLooping` alerts on that node).

### **15. What is Grafana Loki and how does it differ from Elasticsearch?**
**Answer:**
- **Elasticsearch:** Indexes 100% of the raw text content of every log line (high CPU and heavy storage overhead).
- **Grafana Loki:** "Like Prometheus, but for logs." **Indexes only metadata labels** (e.g., `namespace="prod"`, `app="payment"`), storing raw compressed log text chunks in cheap object storage (S3), reducing operational costs by over 80%.

### **16. What is LogQL in Grafana Loki?**
**Answer:**
A Prometheus-inspired query language for logs:
- **Log Stream Query:** `{app="payment", env="prod"} |= "status=failed" | json | error_code != 200`
- **Metric Query over Logs:** `sum(rate({app="payment"} |= "ERROR" [5m])) by (region)` (derives real-time error rate metrics from raw log text).

### **17. What is Grafana Tempo?**
**Answer:** A distributed tracing backend that stores raw spans directly in cheap cloud object storage (Amazon S3, GCS) without needing Elasticsearch, querying traces by `trace_id` retrieved from logs or metrics.

### **18. Compare Thanos vs Grafana Mimir for long-term Prometheus metric storage.**
**Answer:**
- **Thanos:** Uses sidecars on Prometheus instances to upload metrics to S3 and queries them via Thanos Querier.
- **Grafana Mimir:** Massively scalable, multi-tenant time-series database accepting Prometheus Remote Write directly, providing high-availability global querying and automated downsampling.

### **19. What is OpenTelemetry (OTel)?**
**Answer:** A CNCF vendor-neutral framework providing standardized APIs, SDKs, and collector tooling to generate, collect, transform, and export telemetry data (metrics, logs, traces) across polyglot microservice architectures.

### **20. Explain the OpenTelemetry Collector Architecture (Receivers, Processors, Exporters).**
**Answer:**
- **Receivers:** Ingests telemetry in various formats (OTLP, Prometheus, Jaeger).
- **Processors:** Batches requests (`batch`), redacts PII/secrets (`redaction`), and samples traces (`tail_sampling`).
- **Exporters:** Translates and exports telemetry to backends (Prometheus, Tempo, Loki, Datadog).

### **21. What is Distributed Tracing and what is a Span vs a Trace?**
**Answer:**
- **Trace:** The complete end-to-end representation of a request's journey across distributed services.
- **Span:** A single contiguous unit of work within a trace (e.g., executing an HTTP request or SQL query), containing start/end timestamps, span context, tags, and logs.

### **22. What is W3C TraceContext?**
**Answer:** Standardized HTTP headers propagated across microservice boundaries:
- `traceparent`: Encodes version, `trace-id`, `parent-span-id`, and trace flags.
- `tracestate`: Carries vendor-specific routing metadata.

### **23. Compare Tail-Based Sampling vs Head-Based Sampling in Distributed Tracing.**
**Answer:**
- **Head-Based Sampling:** Sampling decision is made at the *start* of the request (e.g., randomly keep 5%). Misses critical downstream errors.
- **Tail-Based Sampling:** OTel Collector buffers the entire trace in memory until completion, keeping **100% of traces containing HTTP 5xx errors or latency $> 2\text{s}$**, and discarding normal 200 OK traces.

### **24. What is Continuous Profiling with Pyroscope / Parca?**
**Answer:** Continuous collection of runtime CPU, memory allocation, and thread contention call stacks directly from production workloads using low-overhead **eBPF probes**, visualizing bottlenecks as interactive **Flame Graphs**.

### **25. What is eBPF Hubble in Cilium for Network Observability?**
**Answer:** Leverages Linux kernel eBPF probes to visualize service-to-service communication dependency graphs, network latency, TCP drops, DNS query failures, and HTTP status codes with zero application instrumentation.

### **26. What is SLO Multi-Window Multi-Burn-Rate Alerting?**
**Answer:** Google SRE best practice: Alert when the **burn rate** of your Error Budget will consume the budget within critical timeframes (e.g., 14.4x burn rate over 1h and 5m consumes 2% of budget $\rightarrow$ page on-call immediately).

### **27. What is Prometheus Remote Write and WAL?**
**Answer:** Prometheus writes incoming samples immediately to an on-disk Write-Ahead Log (WAL). Remote Write streams metrics to long-term storage (Mimir) via snappy-compressed protocol buffers. If the remote endpoint is unreachable, Prometheus replays buffered metrics from WAL once restored.

### **28. What is Synthetic Monitoring vs Real User Monitoring (RUM)?**
**Answer:**
- **Synthetic Monitoring:** Automated bots periodically executing simulated user journeys (checkout, login) to verify availability before real users are impacted.
- **Real User Monitoring (RUM):** Telemetry captured directly within end users' browsers to measure real-world page load latencies and JavaScript errors.

### **29. What is Vector vs Fluent Bit for Edge Log Ingestion?**
**Answer:** High-performance, lightweight log forwarders written in Rust (Vector) or C (Fluent Bit) running as DaemonSets ($< 30\text{MB}$ RAM) on Kubernetes nodes to parse container logs and stream them to Loki/Kafka/Elasticsearch.

### **30. How do you use `predict_linear()` in PromQL for proactive disk space alerting?**
**Answer:**
```promql
predict_linear(node_filesystem_free_bytes[4h], 24 * 3600) < 0
```
Alerts if the current disk consumption rate indicates the filesystem will run out of space **within the next 24 hours**.

### **31. What is Blackbox Exporter in Prometheus?**
**Answer:** Probes external endpoints over HTTP, HTTPS, DNS, TCP, and ICMP to verify availability, response latency, and SSL certificate expiration dates.

### **32. What is Node Exporter?**
**Answer:** Exposes hardware and OS metrics (CPU utilization, memory usage, disk I/O, network traffic) of Linux hosts for Prometheus scraping.

### **33. What is Kube-State-Metrics (KSM)?**
**Answer:** Listens to the Kubernetes API Server and emits metrics regarding the health and state of Kubernetes objects (deployment replicas, pod phases, resource limits).

### **34. What is OpenSearch vs Elasticsearch?**
**Answer:** OpenSearch is a 100% open-source fork (Apache 2.0) of Elasticsearch and Kibana created by AWS and the community after Elastic changed licensing to SSPL.

### **35. What is Structured Logging (JSON) and why is it mandatory?**
**Answer:** Emitting logs formatted as JSON objects (`{"timestamp":"...","level":"info","user_id":"123"}`) eliminates fragile regex parsing, allowing log collectors to ingest native fields for instant indexing and filtering.

### **36. What is Distributed Context Baggage in OpenTelemetry?**
**Answer:** Propagates arbitrary key-value pairs (e.g., `tenant_id="enterprise_456"`) across network boundaries alongside trace contexts, allowing downstream spans to tag metrics and logs with business context.

### **37. What is Alert Deduplication and Fingerprinting in Prometheus?**
**Answer:** Prometheus computes a hash (fingerprint) of all label names and values of an alert. Alertmanager uses this fingerprint to deduplicate incoming alert firings across redundant Prometheus servers scraping the same targets in HA pairs.

### **38. What is Grafana Dashboard as Code (jsonnet / grafonnet)?**
**Answer:** Defining Grafana dashboards programmatically in Jsonnet or TypeScript, enabling version control, code reviews, and automated CI/CD deployment for dashboards.

### **39. What is Prometheus Scrape Interval vs Evaluation Interval?**
**Answer:**
- **`scrape_interval`:** How often Prometheus queries target `/metrics` endpoints (e.g., every 15 seconds).
- **`evaluation_interval`:** How often Prometheus evaluates recording rules and alerting rules (e.g., every 30 seconds).

### **40. What is a Prometheus Recording Rule?**
**Answer:** Precomputes frequently needed or computationally expensive PromQL expressions and saves their result as a new time series, speeding up dashboard queries:
```yaml
groups:
  - name: custom_rules
    rules:
      - record: job:http_requests_total:rate5m
        expr: sum(rate(http_requests_total[5m])) by (job)
```

### **41. What is Prometheus TSDB Block Compaction?**
**Answer:** Merges small 2-hour TSDB in-memory memory-mapped blocks into larger 24-hour persistent blocks on disk, compacting chunk data and pruning deleted series.

### **42. What is Prometheus Head Block vs Persistent Block?**
**Answer:**
- **Head Block:** In-memory block holding current live samples written to the Write-Ahead Log (WAL).
- **Persistent Block:** Immutable, compacted historical blocks stored on disk holding raw chunks and index files.

### **43. What is Alertmanager Silence?**
**Answer:** Temporarily mutes specific alert notifications based on label matchers (e.g., `service="payment"`) during scheduled maintenance windows.

### **44. What is Alertmanager Routing Tree?**
**Answer:** A hierarchical routing configuration in Alertmanager that routes alerts to different notification receivers (Slack for warnings, PagerDuty for critical severity) based on matching labels.

### **45. What is Grafana Unified Alerting?**
**Answer:** A centralized alerting engine inside Grafana that allows creating and managing alerts across heterogeneous data sources (Prometheus, Loki, CloudWatch, PostgreSQL) from a single UI or API.

### **46. What is Prometheus Relabel Config (`relabel_configs` vs `metric_relabel_configs`)?**
**Answer:**
- **`relabel_configs`:** Modifies, filters, or drops scrape targets *before* the scrape occurs based on target metadata labels.
- **`metric_relabel_configs`:** Modifies, filters, or drops individual metric samples *after* the scrape occurs but *before* storing them in the TSDB.

### **47. What is Cardinality Aggregation in LogQL?**
**Answer:** Counting unique values across log fields in Loki (e.g., `count_over_time({app="auth"} | json | unwrap user_id [1h])`).

### **48. What is OpenTelemetry Semantic Conventions?**
**Answer:** Standardized naming rules for telemetry attributes across languages (e.g., `http.request.method`, `db.system`, `rpc.service`).

### **49. What is OpenTelemetry OTLP Protocol?**
**Answer:** The native OpenTelemetry transmission protocol encoding telemetry in Protocol Buffers over gRPC or HTTP/protobuf.

### **50. What is eBPF Kernel Probing in Observability?**
**Answer:** Attaching sandboxed bytecode to Linux kernel tracepoints and kprobes to measure execution duration and packet drops with sub-microsecond overhead.

### **51. What is Prometheus DeadMansSwitch (Watchdog)?**
**Answer:** An alert that is configured to fire continuously 24/7. It routes to an external monitoring service (Dead Man's Snitch / PagerDuty) to verify that the entire Prometheus $\rightarrow$ Alertmanager alerting pipeline is alive.

### **52. What is Grafana Agent / Grafana Alloy?**
**Answer:** A lightweight, vendor-neutral telemetry collector based on OpenTelemetry and Prometheus that captures metrics, logs, traces, and profiles in a single binary.

### **53. What is Distributed Tracing Span Context?**
**Answer:** The immutable metadata containing `TraceID`, `SpanID`, `TraceFlags`, and `TraceState` passed across process boundaries in HTTP/gRPC headers.

### **54. What is Grafana Loki Log Streams?**
**Answer:** A sequence of log lines sharing the exact same unique set of labels, stored as compressed chunks in object storage.

### **55. What is OpenTelemetry Auto-Instrumentation vs Manual Instrumentation?**
**Answer:**
- **Auto-Instrumentation:** Instruments bytecode/runtimes (Java Agents, eBPF, Python monkey-patching) with zero code changes.
- **Manual Instrumentation:** Developers use OTel SDKs directly in code to create custom spans and record business attributes.

### **56. What is OpenTelemetry Resource Attributes?**
**Answer:** Entity metadata attached to all telemetry emitted by a process (e.g., `service.name="order-api"`, `service.version="1.4.0"`, `deployment.environment="prod"`).

### **57. What is Jaeger Tracing?**
**Answer:** A CNCF open-source distributed tracing platform developed by Uber used for monitoring and troubleshooting microservices transactions.

### **58. What is Prometheus Pushgateway?**
**Answer:** An intermediary metrics cache allowing ephemeral batch jobs (which exit before Prometheus can scrape them) to push metrics.

### **59. What is Grafana Variables / Template Queries?**
**Answer:** Dynamic dropdown variables inside dashboards allowing users to filter panels by cluster, namespace, pod, or region on the fly.

### **60. What is OpenTelemetry Collector Memory Limiter Processor?**
**Answer:** Drops or pauses incoming telemetry batches if the OTel Collector process approaches its allocated RAM threshold, preventing OOM crashes.

---

## 🟡 **Part 2: Advanced PromQL, Distributed Tracing & Pipeline Scaling (Questions 61–130)**

### **61. How do you calculate error rate percentage in PromQL?**
**Answer:**
```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m])) * 100
```

### **62. How do you calculate CPU utilization percentage per container in Kubernetes using PromQL?**
**Answer:**
```promql
sum(rate(container_cpu_usage_seconds_total{container!="", container!="POD"}[5m])) by (pod, namespace)
/
sum(kube_pod_container_resource_limits{resource="cpu"}) by (pod, namespace) * 100
```

### **63. What is the difference between `increase()` and `rate()` in PromQL?**
**Answer:**
- `rate()` calculates the per-second average rate of increase.
- `increase()` calculates the total absolute increase over the range window (`increase(v[1h]) = rate(v[1h]) * 3600`).

### **64. What is PromQL Vector Matching (`on()`, `ignoring()`, `group_left()`, `group_right()`)?**
**Answer:** Matching labels between two vector series during arithmetic operations:
- `on(label)`: Match strictly on specified labels.
- `ignoring(label)`: Ignore specified labels during match.
- `group_left`: Many-to-one relationship.
- `group_right`: One-to-many relationship.

### **65. How do you implement Tail-Based Sampling in OTel Collector for error capture?**
**Answer:**
```yaml
processors:
  tail_sampling:
    decision_wait: 10s
    expected_trajectories: 100000
    policies:
      - name: status_code_policy
        type: status_code
        status_code: { status_codes: [ERROR] }
      - name: latency_policy
        type: latency
        latency: { threshold_ms: 2000 }
      - name: probabilistic_policy
        type: probabilistic
        probabilistic: { sampling_percentage: 5.0 }
```

### **66. What is OpenTelemetry TraceID Ratio-Based Sampler?**
**Answer:** Head-based sampler configured at the SDK level that deterministically samples a percentage of traces based on the Trace ID hash.

### **67. What is Grafana Tempo Trace-to-Logs and Logs-to-Trace correlation?**
**Answer:** Configuring Tempo data source settings in Grafana with LogQL/Loki queries mapping `traceId` attributes, allowing 1-click bidirectional navigation between traces and logs.

### **68. What is Thanos Querier vs Thanos Store Gateway?**
**Answer:**
- **Thanos Querier:** Stateless query engine evaluating PromQL queries across live Prometheus sidecars and Store Gateways, deduplicating HA samples.
- **Thanos Store Gateway:** Implements long-term historical block querying directly from cloud object storage (S3).

### **69. What is Thanos Compactor?**
**Answer:** A singleton background service that applies downsampling (5-minute and 1-hour resolution) and retention policies to historical TSDB blocks in S3.

### **70. What is Grafana Mimir Architecture (Ingester, Distributor, Querier, Compactor)?**
**Answer:**
- **Distributor:** Stateless receiver validating and hashing incoming OTLP/Prometheus samples.
- **Ingester:** Stateful service holding in-memory series and writing WAL chunks.
- **Querier:** Stateless PromQL execution engine querying Ingesters and Store Gateways.
- **Compactor:** Merges and downsamples long-term storage blocks in S3.

---

## 🔴 **Part 3: Production Diagnostics, Incident Walkthroughs & SRE Scenarios (Questions 71–200)**

### **71. Scenario: Production p99 latency spikes to 10 seconds, but CPU, memory, and database metrics appear normal. How do you use Distributed Tracing to find the root cause?**
**Answer:**
1. Filter traces in Grafana Tempo by duration $> 5\text{s}$ and HTTP Status 200 during the incident window.
2. Inspect the span waterfall view to locate the specific span responsible for the delay.
3. Identify hidden bottlenecks: un-batched sequential N+1 HTTP calls, thread pool contention, or an external third-party webhook timing out with a 10-second fallback.

### **72. Scenario: Your Prometheus server crashes with OOM every morning at 9:00 AM. How do you diagnose and fix the memory explosion?**
**Answer:**
1. Query the TSDB status API: `GET /api/v1/status/tsdb` to find the metric with the highest label combinations.
2. Identify if a new microservice is exposing dynamic labels (`user_id` or random GUIDs).
3. Configure `metric_relabel_configs` in Prometheus scrape config to drop the offending high-cardinality label:
   ```yaml
   metric_relabel_configs:
     - action: labeldrop
       regex: "user_id|session_token"
   ```

### **73. Design an enterprise Observability Pipeline with OpenTelemetry and Kafka for high-throughput resilience.**
**Answer:**
```
[ App Pods (OTel SDK) ] ➔ [ Local OTel Collector DaemonSet ]
                                     │
                                     ▼
                         [ Apache Kafka Cluster ] (Peak traffic buffer)
                                     │
                                     ▼
                       [ Central OTel Collector Fleet ]
                                     │
                ┌────────────────────┼────────────────────┐
                ▼                    ▼                    ▼
       [ Prometheus / Mimir ]  [ Grafana Tempo ]    [ Grafana Loki ]
```

### **74. Scenario: A Kubernetes node disk is 100% full due to container log accumulation. Pods are evicted and kubelet fails. How do you recover and prevent recurrence?**
**Answer:**
1. **Immediate Recovery:** Delete rotated log archives (`find /var/log/pods -name "*.gz" -delete`) and prune container images (`crictl rmi --prune`).
2. **Permanent Fix:** Configure container runtime log limits in `kubelet-config.yaml`:
   ```yaml
   containerLogMaxSize: "50Mi"
   containerLogMaxFiles: 3
   ```

### **75. Write a complete Alertmanager Multi-Window Multi-Burn-Rate alerting rule for a 99.9% SLO.**
**Answer:**
```yaml
groups:
  - name: payment-slo-alerts
    rules:
      - alert: PaymentErrorBudgetBurnFast
        expr: |
          (
            job:http_requests_errors:rate5m{job="payment"}
            /
            job:http_requests_total:rate5m{job="payment"}
          ) > (14.4 * (1 - 0.999))
          and
          (
            job:http_requests_errors:rate1h{job="payment"}
            /
            job:http_requests_total:rate1h{job="payment"}
          ) > (14.4 * (1 - 0.999))
        for: 2m
        labels:
          severity: page
        annotations:
          summary: "Fast Error Budget Burn (14.4x) on Payment Service"
```

### **76. How do you implement End-to-End Trace Correlation across asynchronous Kafka Message Queues?**
**Answer:**
1. **Producer:** Injects OpenTelemetry Trace Context into Kafka record headers (`traceparent`, `tracestate`).
2. **Broker:** Persists and routes headers alongside message payload.
3. **Consumer:** Extracts `traceparent` header from the incoming record and sets it as the parent context of its execution span, creating a continuous, unbroken trace graph across asynchronous boundaries.

### **77. What is Grafana Synthetic Monitoring and how does it integrate with Prometheus?**
**Answer:** Executes automated HTTP, DNS, TCP, and SSL probes against production domains from global edge locations, writing results as native Prometheus time-series metrics (`probe_success`, `probe_duration_seconds`) for unified alerting in Alertmanager.

### **78. Compare eBPF Parca vs Pyroscope for System-Wide Continuous Profiling.**
**Answer:** Both leverage Linux eBPF kernel probes to sample CPU instruction pointers at fixed frequencies across all processes running on the machine with $< 1\%$ overhead and zero application code changes, profiling the kernel and user-space apps simultaneously.

### **79. Scenario: An engineer modifies Alertmanager config with invalid syntax, breaking production alerts. How do you implement CI/CD for Observability as Code?**
**Answer:**
1. Store all dashboards, rules, and Alertmanager configs in Git.
2. Run automated CI linting and testing:
   - `promtool check rules rules.yaml`
   - `promtool test rules test.yaml`
   - `amtool check-config alertmanager.yaml`
3. Deploy configurations via GitOps (ArgoCD / Prometheus Operator CRDs) only after CI passes.

### **80. What is OpenTelemetry Semantic Conventions for Database Spans?**
**Answer:** Standard attributes: `db.system="postgresql"`, `db.name="orders"`, `db.statement="SELECT * FROM users WHERE id = ?"`, `db.operation="SELECT"`.

### **81. What is Prometheus Staleness Handling?**
**Answer:** When a target stops exposing a time series, Prometheus marks the series with a staleness marker after 5 minutes, terminating range queries cleanly without drawing flat lines.

### **82. What is Grafana Loki Chunk Compaction?**
**Answer:** Merges small log stream chunks from the same day into single large compressed chunks in S3, improving query speed and reducing object storage API costs.

### **83. What is Alertmanager Routing Matchers (`matchers` vs `match_re`)?**
**Answer:**
- `matchers: [severity = critical, team = payments]`: Exact string matching.
- `match_re`: Regex matching across label values.

### **84. What is OpenTelemetry Span Kind (`SERVER`, `CLIENT`, `PRODUCER`, `CONSUMER`, `INTERNAL`)?**
**Answer:**
- `SERVER`: Synchronous incoming request.
- `CLIENT`: Synchronous outgoing request.
- `PRODUCER`: Asynchronous message publication.
- `CONSUMER`: Asynchronous message processing.
- `INTERNAL`: Internal business logic calculation.

### **85. What is Prometheus Native Histograms (Sparse Histograms)?**
**Answer:** Exponential bucket layouts that dynamically adjust bucket resolutions without configuring fixed `le` thresholds in code, reducing storage footprint and improving quantile accuracy.

### **86. What is Grafana Canvas Panel?**
**Answer:** An interactive visualization panel allowing engineers to draw custom architecture diagrams, server racks, and network topologies overlaying live metric values.

### **87. What is OpenTelemetry Batch Processor?**
**Answer:** Batches spans, logs, and metrics before sending over the network (`send_batch_size: 1024`, `timeout: 1s`), maximizing network throughput and minimizing HTTP handshake overhead.

### **88. What is LogQL Line Format Filters?**
**Answer:** Reformatting raw log text on-the-fly using `| line_format "{{.timestamp}} - {{.level}} - {{.message}}"`.

### **89. What is Prometheus External Labels?**
**Answer:** Global labels attached to all metrics emitted by a Prometheus instance (e.g., `cluster="us-east-1-prod"`), enabling multi-cluster aggregation in Thanos/Mimir.

### **90. What is Grafana Alert Notification Policies?**
**Answer:** Nested policy tree in Grafana determining which contact points (Slack, Opsgenie, Webhook) receive notifications based on alert label matchers.

### **91. What is OpenTelemetry Tail Sampling Probabilistic Policy?**
**Answer:** Retaining a fixed percentage (e.g., 1%) of successful, fast requests to maintain a baseline of normal traffic behavior alongside 100% of error traces.

### **92. What is Loki Retention Policies via Compactor?**
**Answer:** Configuring `table_manager` or `compactor` to automatically delete log chunks older than $N$ days (e.g., `retention_period: 30d`) from cloud object storage.

### **93. What is Prometheus Target Down Alert?**
**Answer:** Alerting on `up == 0` for 2 minutes to detect crashed scrape targets or dead exporter instances immediately.

### **94. What is OpenTelemetry Attribute Redaction Processor?**
**Answer:** Automatically masks or strips sensitive PII fields (credit card numbers, passwords, authorization tokens) from span attributes and logs before exporting.

### **95. What is Grafana Trace to Metrics Correlation?**
**Answer:** Generating Prometheus rate and latency metric queries automatically derived from OpenTelemetry trace spans in real time.

### **96. What is Prometheus Memory-Mapped Files (mmap)?**
**Answer:** TSDB leverages OS virtual memory mmap to read chunk blocks directly from disk cache without allocating double heap memory in Prometheus processes.

### **97. What is Grafana Mimir Multi-Tenancy?**
**Answer:** Enforcing strict tenant isolation using the `X-Scope-OrgID` HTTP header, isolating metrics, dashboards, and alert rules per tenant.

### **98. What is OpenTelemetry Span Events?**
**Answer:** Timestamped annotations within a span recording point-in-time micro-events (e.g., "cache_miss", "token_validated") with custom key-value attributes.

### **99. What is Prometheus Blackbox SSL Expiration Alert?**
**Answer:** Alerting when SSL certificates will expire in less than 14 days:
```promql
probe_ssl_earliest_cert_expiry - time() < 14 * 86400
```

### **100. What is Grafana Loki Log Filtering Pipeline Order?**
**Answer:** Best practice pipeline sequence for query performance: Label Matchers (`{app="api"}`) $\rightarrow$ Line Filters (`|= "error"`) $\rightarrow$ Parser (`| json`) $\rightarrow$ Label Filters (`| status_code >= 500`).

### **101. What is OpenTelemetry Prometheus Remote Write Exporter?**
**Answer:** Translates OTel metrics into Prometheus remote write protocol buffers and streams them to Prometheus/Mimir backends.

### **102. What is Prometheus Scrape Timeout (`scrape_timeout`)?**
**Answer:** The maximum duration Prometheus waits for a target `/metrics` endpoint to return data before aborting the scrape and recording a scrape failure.

### **103. What is Grafana Plugin Architecture?**
**Answer:** Modular extension system supporting Data Source plugins (connecting new databases), Panel plugins (custom charts), and App plugins (integrated suites).

### **104. What is Alertmanager Repeat Interval (`repeat_interval`)?**
**Answer:** How long to wait before re-sending a notification for an alert that is still actively firing (e.g., every 4 hours).

### **105. What is OpenTelemetry SDK Trace Provider?**
**Answer:** The stateful registry object in the application process that manages Tracer instances, span processors, and Samplers.

### **106. What is Prometheus Label Value Regex Matching (`=~` vs `!~`)?**
**Answer:**
- `=~`: Selects labels matching the regex.
- `!~`: Selects labels that do not match the regex.

### **107. What is Grafana Explore Mode?**
**Answer:** An interactive troubleshooting workspace for ad-hoc debugging, log analysis (Loki), trace inspection (Tempo), and PromQL metric exploration without modifying saved dashboards.

### **108. What is Alertmanager Group Wait (`group_wait`)?**
**Answer:** How long to buffer initial alerts to batch multiple related alerts before sending the first notification (e.g., 30 seconds).

### **109. What is OpenTelemetry Jaeger Exporter Deprecation?**
**Answer:** Jaeger natively ingests OTLP; direct Jaeger client SDKs are deprecated in favor of standard OTel SDKs.

### **110. What is Prometheus In-Flight Requests Limit (`max_requests_inflight`)?**
**Answer:** Limits the number of concurrent PromQL queries the API server processes to prevent memory exhaustion from runaway dashboard loads.

### **111. What is Grafana Live / WebSockets?**
**Answer:** Real-time streaming protocol that pushes live metric updates directly from collectors to dashboard panels over persistent WebSocket connections.

### **112. What is Alertmanager Group Interval (`group_interval`)?**
**Answer:** How long to wait before sending a notification about newly added alert instances that belong to an already active firing group (e.g., 5 minutes).

### **113. What is OpenTelemetry Trace Context Injection vs Extraction?**
**Answer:**
- **Injection:** Writes `traceparent` headers into outgoing HTTP/gRPC requests.
- **Extraction:** Reads `traceparent` headers from incoming HTTP/gRPC requests to establish parent-child span links.

### **114. What is Prometheus Range Vector vs Instant Vector?**
**Answer:**
- **Instant Vector:** A single current value per time series at a specific timestamp (`http_requests_total`).
- **Range Vector:** A buffer of historical samples over a time window (`http_requests_total[5m]`), required by functions like `rate()`.

### **115. What is Grafana Dashboard Version History?**
**Answer:** Built-in versioning tracking every save of a dashboard, allowing instant visual diff comparison and 1-click rollback.

### **116. What is Alertmanager Webhook Payload Structure?**
**Answer:** A standardized JSON document containing alert metadata, status (`firing`/`resolved`), labels, annotations, and timestamps sent to external incident management systems.

### **117. What is OpenTelemetry Instrumentation Scope?**
**Answer:** Identifies the specific library or module that generated a span (e.g., `io.opentelemetry.spring-webmvc`).

### **118. What is Prometheus Drop Label Action (`labeldrop`)?**
**Answer:** Drops specified labels matching a regex during metric relabeling to reduce TSDB cardinality.

### **119. What is Grafana Organization Multi-Tenancy?**
**Answer:** Logical partitioning inside Grafana isolating dashboards, data sources, and users into independent organization spaces.

### **120. What is Alertmanager Notification Template?**
**Answer:** Custom Go text/template definitions that format Slack, email, and webhook messages into rich, readable incident alerts.

### **121. What is OpenTelemetry Zipkin Compatibility?**
**Answer:** OTel collectors ingest legacy B3 propagation headers and Zipkin JSON spans, converting them to standard OTLP.

### **122. What is Prometheus Metric Reset Handling?**
**Answer:** The `rate()` and `increase()` functions automatically detect when a counter value drops (due to a pod/process restart) and correct for the reset.

### **123. What is Grafana Library Panels?**
**Answer:** Reusable dashboard panels shared across multiple dashboards; modifying the library panel updates all dashboards referencing it.

### **124. What is Alertmanager Silence Expiration?**
**Answer:** Silences automatically expire after their scheduled end time, restoring normal alert paging without manual intervention.

### **125. What is OpenTelemetry Span Links?**
**Answer:** Links a span to one or more unrelated trace spans, capturing causal relationships in asynchronous batch processing and event streaming.

### **126. What is Prometheus Subquery (`rate(v[1m])[30m:1m]`)?**
**Answer:** Calculates a range query over the result of an instant query over a historical time window.

### **127. What is Grafana Serverless Cloud Monitoring?**
**Answer:** Ingesting AWS CloudWatch, Azure Monitor, and Google Cloud Monitoring metrics natively into Grafana via managed IAM integrations.

### **128. What is Alertmanager High Availability Gossip Protocol?**
**Answer:** Alertmanager instances communicate over Mesh Gossip (UDP/TCP port 9094) to synchronize active alerts, silences, and notification state across clusters.

### **129. What is OpenTelemetry Logs Data Model?**
**Answer:** Standardized log event model containing Timestamp, ObservedTimestamp, SeverityNumber, SeverityText, Body, Attributes, TraceID, and SpanID.

### **130. What is Prometheus Metric Relabel Action `keep`?**
**Answer:** Drops all metric series whose labels do not match the specified regex during scrape relabeling.

### **131. What is Grafana Public Dashboards?**
**Answer:** Generates a read-only, publicly shareable URL for specific dashboard panels without requiring user authentication.

### **132. What is Alertmanager PagerDuty Integration?**
**Answer:** Sends structured incidents directly to PagerDuty Events API v2, automatically resolving incidents in PagerDuty when Prometheus marks the alert as resolved.

### **133. What is OpenTelemetry Collector Filelog Receiver?**
**Answer:** Tails, parses, multiline-aggregates, and ingests container log files directly from disk without third-party log daemons.

### **134. What is Prometheus Metric Relabel Action `replace`?**
**Answer:** Constructs or overwrites target label values by regex matching and string interpolation of source labels.

### **135. What is Grafana Tempo Search (TempoQL)?**
**Answer:** Allows searching distributed traces by duration, status code, span name, and custom attributes directly against cloud object storage.

### **136. What is Alertmanager Slack Webhook Action?**
**Answer:** Formats firing alerts with color-coded severity bars (red for critical, yellow for warning) and direct runbook links into Slack channels.

### **137. What is OpenTelemetry Exporter Retry and Backoff?**
**Answer:** Automatically buffers and retries failed network exports to backends with exponential backoff to survive transient network outages.

### **138. What is Prometheus Query Range API (`/api/v1/query_range`)?**
**Answer:** Evaluates a PromQL expression over a time range (`start`, `end`, `step`) to render time-series line graphs.

### **139. What is Grafana Enterprise RBAC?**
**Answer:** Granular role-based access control assigning permissions (`viewer`, `editor`, `admin`) to specific dashboard folders and data sources.

### **140. What is Alertmanager Opsgenie Integration?**
**Answer:** Routes alert payloads to Opsgenie API with dynamic priority mapping (P1 for critical, P3 for warning).

### **141. What is OpenTelemetry Collector Span Metrics Processor?**
**Answer:** Aggregates distributed trace spans in-flight to automatically generate Prometheus RED metrics (request count, error rate, latency histograms) before exporting.

### **142. What is Prometheus Query Instant API (`/api/v1/query`)?**
**Answer:** Evaluates a PromQL expression at a single point in time to render single-value stat panels and evaluate alerting rules.

### **143. What is Grafana OnCall?**
**Answer:** An open-source on-call management and paging tool integrated directly into Grafana, supporting escalation chains and schedule rotations.

### **144. What is Alertmanager Telegram Integration?**
**Answer:** Dispatches real-time incident notifications and resolution messages to Telegram group chats via bot API tokens.

### **145. What is OpenTelemetry Semantic Conventions for HTTP?**
**Answer:** Standard attributes: `http.request.method="POST"`, `http.response.status_code=200`, `url.full="https://api.company.com/v1/charge"`.

### **146. What is Prometheus Chunk Encoding (XOR / Gorilla)?**
**Answer:** Compresses consecutive float64 timestamps and sample values using double-delta and XOR bit-shifting compression, achieving $< 1.5$ bytes per sample on disk.

### **147. What is Grafana K6 for Performance Load Testing?**
**Answer:** Modern developer-centric load testing tool written in JavaScript that outputs real-time test metrics to Prometheus and Grafana.

### **148. What is Alertmanager Auto-Resolve?**
**Answer:** Automatically sends a `[RESOLVED]` notification to channels when Prometheus stops sending alert firings for a configurable interval (`resolve_timeout: 5m`).

### **149. What is OpenTelemetry Collector Routing Processor?**
**Answer:** Routes telemetry batches dynamically to different backend exporters based on incoming span attributes or HTTP headers.

### **150. What is Prometheus Metric Relabel Action `hashmod`?**
**Answer:** Computes an MD5 hash modulo of a label value to partition scrape targets across multiple Prometheus instances for horizontal scaling.

### **151. What is Grafana Loki Multi-Line Log Parsing?**
**Answer:** Combines multi-line Java/Python stack traces into a single log event using `multiline` stage regex patterns in Promtail or Fluent Bit.

### **152. What is Alertmanager MS Teams Integration?**
**Answer:** Dispatches structured incident message cards to Microsoft Teams webhook channels.

### **153. What is OpenTelemetry Collector Prometheus Receiver?**
**Answer:** Reuses Prometheus scrape configurations inside the OTel Collector to pull metrics from Kubernetes pods and endpoints.

### **154. What is Prometheus Instant Vector Comparison (`> bool`)?**
**Answer:** Returns `1` if the comparison is true or `0` if false for each series, rather than filtering the vector.

### **155. What is Grafana Faro for Frontend Observability?**
**Answer:** Open-source JavaScript RUM SDK that captures real-world browser web vitals, errors, and traces, sending them directly to OTel collectors.

### **156. What is Alertmanager Inhibit Rule Target Matchers?**
**Answer:** Matches lower-severity alerts that should be muted when the source alert is actively firing.

### **157. What is OpenTelemetry Collector Kubernetes Attributes Processor?**
**Answer:** Queries the Kubernetes API Server to automatically enrich incoming telemetry with Pod name, Namespace, Node name, and Deployment labels.

### **158. What is Prometheus Offset Modifier (`http_requests_total offset 1w`)?**
**Answer:** Shifts the evaluation time of a vector backward by a designated duration (e.g., 1 week) to compare current metrics against last week's baseline.

### **159. What is Grafana Data Links?**
**Answer:** Configures clickable links on dashboard panels that dynamically pass time ranges and label variables to other dashboards or external URLs.

### **160. What is Alertmanager Route Continue (`continue: true`)?**
**Answer:** Allows an alert to continue matching downstream sibling routes in the routing tree rather than stopping at the first matching route.

### **161. What is OpenTelemetry Collector OTLP gRPC Receiver Port (4317)?**
**Answer:** The standard IANA port for OTLP gRPC telemetry ingestion; port 4318 is standard for OTLP HTTP/protobuf.

### **162. What is Prometheus Label Join Function (`label_join()`)?**
**Answer:** Combines multiple label values into a single new label string separated by a delimiter.

### **163. What is Grafana Alert Instance State (`Normal`, `Pending`, `Firing`, `NoData`, `Error`)?**
**Answer:**
- `Normal`: Metric is healthy within thresholds.
- `Pending`: Metric breached threshold, waiting for `for:` duration.
- `Firing`: Duration elapsed; notification dispatched.
- `NoData`: Target query returned zero series.

### **164. What is Alertmanager Webhook TLS Verification?**
**Answer:** Enforces TLS certificate validation when posting incident payloads to HTTPS webhook endpoints.

### **165. What is OpenTelemetry TraceID Ratio Sampler Edge Case?**
**Answer:** If parent span is sampled, child spans inherit the sampling decision regardless of ratio to prevent broken trace waterfalls.

### **166. What is Prometheus Label Replace Function (`label_replace()`)?**
**Answer:** Regex matches and extracts substrings from source labels to create or overwrite target label values.

### **167. What is Grafana Alert Execution Error Handling (`Error` vs `KeepLastState`)?**
**Answer:** Determines whether an alert transitions to `Alerting` state or maintains its previous state when underlying data queries time out.

### **168. What is Alertmanager Inhibition Source Matchers?**
**Answer:** The label matchers defining the high-severity alert that suppresses lower-severity alerts.

### **169. What is OpenTelemetry Memory Ballast (Legacy)?**
**Answer:** Allocating a dummy byte slice in Go runtimes to stabilize GC frequency, now replaced by Go's native `GOMEMLIMIT`.

### **170. What is Prometheus Time Function (`time()`)?**
**Answer:** Returns the current Unix timestamp in seconds, used in arithmetic expressions to measure elapsed durations.

### **171. What is Grafana Alerting Contact Point?**
**Answer:** A destination target (Slack channel, email address, PagerDuty key) where alert notifications are delivered.

### **172. What is Alertmanager Matcher Operators (`=`, `!=`, `=~`, `!~`)?**
**Answer:**
- `=`: Exact string equality.
- `!=`: Negative string equality.
- `=~`: Regex match.
- `!~`: Negative regex match.

### **173. What is OpenTelemetry Collector Transform Processor (OTTL)?**
**Answer:** OpenTelemetry Transformation Language used to mutate, set, or delete telemetry fields and attributes inside the collector pipeline.

### **174. What is Prometheus Timestamp Function (`timestamp()`)?**
**Answer:** Returns the exact timestamp of each sample in an instant vector.

### **175. What is Grafana Thresholds?**
**Answer:** Configurable color-coded thresholds (green, orange, red) applied to stat panels and gauges based on numeric metric values.

### **176. What is Alertmanager Default Receiver?**
**Answer:** The fallback notification channel configured at the root of the routing tree for alerts that do not match any child route.

### **177. What is OpenTelemetry W3C Baggage Header (`baggage: ...`)?**
**Answer:** HTTP header carrying comma-separated key-value pairs (`baggage: userId=alice,isRootUser=true`) across network hops.

### **178. What is Prometheus Changes Function (`changes()`)?**
**Answer:** Returns the number of times a metric's value has changed over a given range window.

### **179. What is Grafana Transformations?**
**Answer:** Client-side data manipulations (joining tables, renaming fields, filtering rows, calculating deltas) applied to query results before rendering.

### **180. What is Alertmanager Clustering Mesh Port (9094)?**
**Answer:** Default TCP/UDP port used for gossiping alert states across clustered Alertmanager nodes.

### **181. What is OpenTelemetry Collector Health Check Extension?**
**Answer:** Exposes a `/healthz` HTTP probe endpoint allowing Kubernetes liveness/readiness probes to verify collector status.

### **182. What is Prometheus Deriv Function (`deriv()`)?**
**Answer:** Calculates the per-second derivative of time series using simple linear regression, used with gauge metrics.

### **183. What is Grafana Annotations?**
**Answer:** Vertical visual markers overlaid on dashboard graphs indicating events (e.g., deployments, incident starts, chaos tests).

### **184. What is Alertmanager Silence Matcher Exact vs Regex?**
**Answer:** Silences match labels literally or via regular expressions.

### **185. What is OpenTelemetry Resource Detector?**
**Answer:** Automatically detects host environment metadata (AWS EC2 instance ID, GCP Zone, Docker container ID) and attaches it to telemetry.

### **186. What is Prometheus Holt-Winters Function (`holt_winters()`)?**
**Answer:** Produces a smoothed value for time series based on double exponential smoothing, used for seasonal trend analysis.

### **187. What is Grafana Dashboard Playlist?**
**Answer:** Cycles through a curated sequence of dashboards automatically at fixed intervals for TV wall displays in Network Operations Centers (NOC).

### **188. What is Alertmanager Email Smart HTML Templates?**
**Answer:** Generates responsive HTML email notifications with embedded incident graphs and alert metadata.

### **189. What is OpenTelemetry Cumulative vs Delta Temporality?**
**Answer:**
- **Cumulative:** Counters record total count since start (Prometheus standard).
- **Delta:** Counters record incremental count since the last export (CloudWatch/Datadog standard).

### **190. What is Prometheus Days in Month Function (`days_in_month()`)?**
**Answer:** Returns the number of days in the specified month, used for monthly billing calculations in PromQL.

### **191. What is Grafana State Timeline Panel?**
**Answer:** Visualizes state transitions over time (e.g., green for healthy, red for error) for multiple pods or servers.

### **192. What is Alertmanager Webhook Retry Count?**
**Answer:** Configurable retry attempts for delivering HTTP webhook notifications before logging a failure.

### **193. What is OpenTelemetry Exporter Compression (`gzip` vs `snappy` vs `none`)?**
**Answer:** Compresses payload bytes in-transit; `snappy` provides optimal balance of high throughput and low CPU overhead.

### **194. What is Prometheus Hour of Day Function (`hour()`)?**
**Answer:** Returns the hour of the day (0–23) in UTC, used in alerting rules to restrict pages to business hours.

### **195. What is Grafana Flame Graph Panel?**
**Answer:** Visualizes hierarchical profiling data (Pyroscope) to identify exact functions consuming the highest CPU or memory allocations.

### **196. What is Alertmanager Log Format (`log.format=json`)?**
**Answer:** Configures Alertmanager daemon to emit structured JSON logs for centralized log aggregation.

### **197. What is OpenTelemetry Logging Exporter (Debug Exporter)?**
**Answer:** Prints raw telemetry payloads directly to stdout for local development and debugging.

### **198. What is Prometheus Day of Week Function (`day_of_week()`)?**
**Answer:** Returns the day of the week (0 = Sunday, 6 = Saturday) in UTC for time-based routing in PromQL.

### **199. What is Grafana GeoMap Panel?**
**Answer:** Visualizes geographic data on a 3D world map, overlaying client traffic volumes and edge latencies by city.

### **200. What is an Enterprise Observability Maturity Model?**
**Answer:**
1. **Level 1 (Reactive Monitoring):** Basic uptime pings and CPU/RAM static thresholds.
2. **Level 2 (Proactive APM):** Distributed tracing, structured logging, and Golden Signals.
3. **Level 3 (SRE Reliability):** SLOs, Error Budget burn-rate alerting, and Chaos GameDays.
4. **Level 4 (Continuous Observability):** OpenTelemetry correlation, eBPF Continuous Profiling, automated canary metric analysis, and autonomous remediation.
