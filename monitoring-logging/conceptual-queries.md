Q1. What is Observability, and how is it different from monitoring?
Monitoring tells us when something is wrong. Observability tells us why it is wrong.
Observability is based on three pillars:
• Metrics (Prometheus)
• Logs
• Traces
Prometheus focuses mainly on metrics-based observability

Q2. What is Prometheus and why is it widely used?
Prometheus is an open-source monitoring and alerting system designed for:
• Time-series metrics
• Pull-based metric collection
• Powerful querying using PromQL
It is popular because it:
• Works well with Kubernetes
• Is highly scalable
• Has strong alerting and visualization support

Q3. Explain Prometheus architecture.
Prometheus consists of:
• Prometheus Server – scrapes and stores metrics
• Exporters – expose metrics (Node Exporter, JVM Exporter)
• Service Discovery – finds targets dynamically
• Alertmanager – manages alerts
• Grafana – visualization layer

Q4. What is a metric in Prometheus?
A metric is a time-series data point identified by:
• Metric name
• Labels (key-value pairs)
• Timestamp
• Value
Example:
http_requests_total{method="GET",status="200"}

Q5. What are the different metric types in Prometheus?
Type Description
Counter Always increases (e.g., request count)
Gauge Can go up and down (e.g., memory usage)
Histogram Measures distributions with buckets
Summary Similar to histogram but client-side

Q6. What is PromQL?
PromQL (Prometheus Query Language) is used to:
• Query metrics
• Aggregate data
• Perform calculations
Example:
rate(http_requests_total[5m])

Q7. Difference between rate() and irate()?
• rate() → average over time (stable,preferred)
• irate() → instant rate (more volatile)

Q8. What is an exporter?
An exporter converts system/application metrics into Prometheus format.
Examples:
• Node Exporter(OS metrics)
• Blackbox Exporter(endpoint checks)
• JVM Exporter

Q9. How does Prometheus store data?
Prometheus stores data in a local TSDB(Time-Series Database) with:
• Efficient compression
• Retention-based deletion

Q10. What are labels and why are they important?
Labels provide dimensions for metrics.
They allow:
• Filtering
• Aggregation
• Multi-dimensional analysis
Example:
{pod="payment-service", namespace="prod"}

Scenario-Based Questions(Prometheus)
Q11. Your Prometheus server is running out of disk space. What would you do?
• Reduce retention period
• Enable remote storage(Thanos/Cortex)
• Clean unused metrics
• Increase disk size
• Reduce label cardinality

Q12. Metrics are missing from a Kubernetes pod. How do you debug?
• Check pod annotations
• Verify ServiceMonitor/PodMonitor
• Check target status in Prometheus UI
• Validate exporter endpoint
• Check network policies

Q13. Prometheus performance is degrading. What could be the reasons?
• High cardinality labels
• Too many scrape targets
• Long retention
• Expensive PromQL queries

Q14. How do you monitor application latency using Prometheus?
• Use Histogram or Summary
• Track request duration
• Use histogram_quantile() for percentiles

Q15. How do you handle multi-cluster monitoring?
• Deploy Prometheus per cluster
• Use Thanos/Cortex for aggregation
• Centralized Grafana dashboards

Conceptual Questions(Grafana)
Q16. What is Grafana?
Grafana is an open-source visualization and analytics tool used to:
• Create dashboards
• Visualize metrics
• Set alerts

Q17. How does Grafana integrate with Prometheus?
Grafana uses Prometheus as a data source and executes PromQL queries to visualize metrics.

Q18. What are dashboards and panels in Grafana?
• Dashboard - Collection of panels
• Panel - Single visualization(graph,table,gauge)

Q19. What types of visualizations does Grafana support?
• Time-series graphs
• Heatmaps
• Gauges
• Tables
• Bar charts

Q20. What are Grafana variables?
Variables make dashboards dynamic and reusable.
Example:
• $namespace
• $pod

Scenario-Based Questions(Grafana)
Q21. A Grafana dashboard is slow. How do you optimize it?
• Reduce query complexity
• Increase time range step
• Limit panels per dashboard
• Use recording rules

Q22. How do you create alerts in Grafana?
• Define alert conditions on panels
• Configure thresholds
• Set notification channels(Slack,Email)

Q23. Multiple teams use the same Grafana instance. How do you manage access?
• Use Organizations
• Use Folders
• Apply RBAC
• Integrate with LDAP/OAuth

Q24. How do you correlate metrics with logs and traces?
• Use consistent labels(traceId,service)
• Integrate with Loki(logs)
• Integrate with Tempo(traces)

Q25. How do you visualize error rates in Grafana?
• Use PromQL to calculate error percentage
• Plot time-series graph
• Add threshold-based alerts

Q26. Production latency increased suddenly. How do you troubleshoot using Prometheus & Grafana?
1. Check latency dashboards
2. Identify affected service
3. Drill down by pod/instance
4. Correlate with CPU/memory
5. Check error rates
6. Validate recent deployments
