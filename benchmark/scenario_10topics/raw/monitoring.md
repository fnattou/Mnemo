# Monitoring, Logging & Observability

## Decision Summary
TaskBoard implements **CloudWatch centralized logging, Prometheus metrics, and PagerDuty alerting** creating unified observability without external dependencies while maintaining cost-effectiveness for startup scale.

## Logging Architecture
Structured logging in JSON format enables efficient querying and prevents brittle regex parsing:
```json
{ "timestamp": "2025-04-12T15:30:00Z", "level": "error", "service": "task-api", "user_id": "123", "duration_ms": 245, "error": "TaskNotFound" }
```
CloudWatch Logs aggregates streams from all ECS tasks. Log retention: 7 days for debug logs, 30 days for errors/warnings. Costs controlled via log group policies excluding verbose debug output in production.

## Metrics & Alerting
- **Request metrics**: p50/p95/p99 latency, throughput (req/sec), error rates by status code
- **Business metrics**: Tasks created/completed per hour, user login count, project creation rate
- **Infrastructure**: CPU/memory utilization, disk I/O, database connection pool saturation
- **Alerts**: Triggered on sustained (>5min) threshold breach, not single spikes

## Alert Thresholds & Response
- **P99 latency > 2s**: Page on-call engineer, check for database lock contention
- **Error rate > 1%**: Investigate, typically misconfigured client or upstream service failure
- **Memory utilization > 85%**: Horizontal scaling triggered automatically via ECS auto-scaling policies
- **Database CPU > 75%**: Alert engineer; scaling decisions require manual review (read replica, schema optimization)

## Dashboard Organization
- **Service health dashboard**: Green/amber/red status indicators for each service, deployment history
- **User journey dashboard**: Signup funnel, feature adoption metrics, churned users
- **SLA dashboard**: Real-time compliance with 99.5% uptime SLA, projected monthly status
- **Cost dashboard**: Per-service AWS costs, trends, anomalies

## Tracing & Distributed Context
- X-Request-ID propagated through microservices enabling request journey visualization
- Sampled tracing (1% of requests) in CloudWatch Insights for latency troubleshooting
- Correlation IDs link related async jobs (email notifications, exports)

## Alternatives Considered
1. **Datadog/New Relic**: Superior UX; licensing costs scale with volume, prohibitive for bootstrap phase
2. **ELK Stack (self-hosted)**: Operational overhead; Elasticsearch maintenance and cluster sizing complex
3. **Splunk**: Powerful but expensive; overkill for current scale

## Critical Practices
- Avoid cardinality explosion: user_id as metric label fine (finite set); timestamp as label (infinite cardinality) forbidden
- Alert fatigue prevention: tuning thresholds quarterly based on actual noise rate
- On-call rotation: 1-week shifts, single engineer on call, escalation to full team for > 1hr incidents
- Blameless post-mortems: focus on system failure modes, not individual errors
