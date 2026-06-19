---
tags:
  - flashcards/aws-monitoring
---

# Monitoring — CloudWatch, X-Ray, CloudTrail

Metrics, logs, tracing, and audit for DVA-C02.

---

What is the difference between CloudWatch metrics and logs?
?
**Quick:** Metrics are time-series numeric data; logs are raw text events.<br>
**Deeper:** Metrics live in namespaces, have dimensions, and have built-in stats (Sum, Avg, Min, Max, p99). Logs are stored in log groups → log streams, queryable via Logs Insights. You can extract metrics from logs (Metric Filters). Exam gotcha: standard EC2 metrics are at 5-min intervals; enable detailed monitoring ($) for 1-min. Custom metrics support 1-minute (standard) or 1-second (high-resolution) intervals.

What is a CloudWatch custom metric?
?
**Quick:** Application-defined metric published via PutMetricData.<br>
**Deeper:** Required for memory usage, disk usage inside EC2 (the CloudWatch agent emits these). Resolution: standard (60s) vs high-resolution (1s, more expensive). Use up to 30 dimensions per metric (10 free per metric per name). Exam trick: "monitor EC2 memory" = CloudWatch agent + custom metrics, NOT default metrics.

How long does CloudWatch keep logs and metrics?
?
**Quick:** Logs: configurable retention 1 day to indefinite (default Never Expire); metrics: 15 months.<br>
**Deeper:** Metrics aggregation: 1-second resolution kept 3 hours, 60-second resolution 15 days, 5-minute 63 days, 1-hour 15 months. Logs are billed by ingestion + storage; set retention per log group to control costs. Exam best practice: change default log retention from "never" to 14/30/90 days unless required for compliance.

What are CloudWatch alarms and their states?
?
**Quick:** OK, ALARM, INSUFFICIENT_DATA; trigger on metric thresholds.<br>
**Deeper:** Configure with metric, threshold, evaluation periods, datapoints to alarm. Actions: SNS, Auto Scaling, EC2 actions (stop/terminate/reboot/recover), Lambda. Composite alarms combine alarms with AND/OR logic. Exam trick: alarms only emit on state CHANGE, not on every datapoint — useful and confusing for new users.

What is CloudWatch Logs Insights?
?
**Quick:** Interactive query language for searching and aggregating CloudWatch Logs.<br>
**Deeper:** SQL-like queries: fields, filter, stats, sort, limit, parse. Example: fields @timestamp, @message | filter @message like /ERROR/ | stats count() by bin(5m). Results cached 7 days. Exam trick: Logs Insights is per-region and per-log-group — to query across many groups, use CloudWatch Logs cross-account observability or export to S3 + Athena.

What is the difference between EventBridge and CloudWatch Events?
?
**Quick:** Same service, renamed — EventBridge is the modern name with more sources.<br>
**Deeper:** Old API (events.*) and new API (eventbridge) both work. EventBridge adds custom buses, partner sources, schema registry, archive/replay. Scheduled rules use cron(...) or rate(...). Exam trick: a "CloudWatch Events rule" and an "EventBridge rule on the default bus" are the same thing.

What is AWS X-Ray and what does it trace?
?
**Quick:** Distributed tracing for requests across AWS services and microservices.<br>
**Deeper:** Visualizes request flow through Lambda, API Gateway, ECS, EC2, EBS, etc. Service map shows dependencies and latencies. Detects errors, faults (5xx), throttles. Exam trick: X-Ray sampling defaults to 1 req/sec + 5% — to capture more in low-traffic envs, raise via sampling rules.

What is the difference between X-Ray segments, subsegments, annotations, and metadata?
?
**Quick:** Segments are per-service spans; subsegments are sub-units; annotations are indexed key/values; metadata are unindexed.<br>
**Deeper:** Segments correspond to a single AWS service in the trace. Subsegments are remote calls or internal phases (e.g., DB query). Annotations (up to 50) are indexed and searchable in console filters. Metadata (any size) is for debugging, NOT searchable. Exam trick: searchable = annotation; bulk attached info = metadata.

How does the X-Ray daemon work?
?
**Quick:** A UDP listener on port 2000 that batches segments and ships to the X-Ray service.<br>
**Deeper:** SDK sends to localhost:2000; daemon buffers and forwards. Required on EC2, ECS EC2 mode, on-prem. Lambda runs the daemon for you when active tracing is enabled. App needs xray:PutTraceSegments + PutTelemetryRecords permissions. Exam trick: missing traces from EC2 usually means daemon isn't running or security group blocks egress.

What is CloudTrail and what does it record?
?
**Quick:** Logs of all AWS API calls in your account.<br>
**Deeper:** Captures Management events (control plane like CreateBucket — free, on by default for 90 days in Event history) and Data events (data plane like S3 GetObject, Lambda Invoke — paid, off by default). Insights events detect unusual API activity ($). Multi-region trails capture all regions in one trail. Exam trick: data events are NOT logged unless explicitly enabled — common gotcha for S3 audit questions.

What is the difference between CloudTrail Event history and a trail?
?
**Quick:** Event history is 90-day console view; trail persists to S3 with longer retention.<br>
**Deeper:** Event history is automatic, free, 90 days, console only, management events only. Create a trail to deliver events to S3 (any retention), CloudWatch Logs (for alarms), or EventBridge. Multi-region trails are best practice. Exam trick: "audit access for past year" = trail with S3 (NOT Event history).

What are CloudTrail Insights events?
?
**Quick:** ML-detected anomalies in API call rates and error patterns.<br>
**Deeper:** Auto-flags unusual spikes/drops in write APIs (e.g., suddenly 100x more RunInstances). Paid feature, enabled per trail. Useful for detecting compromised credentials or runaway automation. Exam trick: Insights only analyzes Write management events by default — extend to data events as needed.

How do CloudWatch alarms trigger Auto Scaling?
?
**Quick:** Alarm state ALARM triggers a Scaling Policy action.<br>
**Deeper:** Target tracking is the modern approach — pick a metric (CPU 50%) and ASG manages alarms. Simple scaling adds/removes N instances when alarm fires (with cooldown). Step scaling adjusts based on alarm breach magnitude. Exam trick: target tracking is "set and forget" and is the recommended default.

What CloudWatch agent vs SSM agent?
?
**Quick:** CloudWatch agent collects metrics/logs; SSM agent enables Systems Manager features.<br>
**Deeper:** CloudWatch agent (separate install) collects EC2 memory/disk/swap/processes and custom log files. SSM agent (preinstalled on Amazon Linux 2, Ubuntu, etc.) enables Run Command, Session Manager, Patch Manager. Exam trick: "memory metric on EC2" = CloudWatch agent; "SSH replacement / no open ports" = SSM Session Manager.
