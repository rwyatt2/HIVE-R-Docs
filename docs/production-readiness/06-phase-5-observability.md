---
title: "Phase 5: Observability Stack"
sidebar_label: "Phase 5: Observability Stack"
---

**

**⏱️ Time**: 10-12 hours

**💰 Budget**: $20-25

**🎯 Goal**: Deep visibility into system behavior

**🤖 AI Workers**: HIVE-R SRE + Data Analyst + Builder agents

**📊 Step 5.1: Implement Structured Logging**

**Your Instruction** (in Cursor):

  

  

Consult the SRE and Builder agents.

  

CONTEXT:

We're using console.log() scattered everywhere. This is unmaintainable in production.

  

TASK:

Migrate to structured JSON logging with Pino:

  1. Replace all console.log/error/warn with logger

  2. Add context to every log (requestId, userId, agentName)

  3. Configure log levels by environment (debug in dev, info in prod)

  4. Add log rotation (don't fill disk)

  5. Format for log aggregation tools (ELK, Datadog)

  

LOG STRUCTURE:

&#123;

  "level": "info",

  "time": "2026-02-07T12:34:56.789Z",

  "requestId": "req-123",

  "userId": "user-456",

  "agentName": "builder",

  "msg": "Agent invocation started",

  "duration": 1234,

  "tokens": 500,

  "cost": 0.015

&#125;

  

IMPLEMENTATION:

- Install: pino, pino-pretty (dev), pino-rotating-file-stream

- Create logger singleton: src/lib/logger.ts

- Add middleware to inject requestId

- Update ALL files that use console.log

  

DELIVERABLES:

- Logger configuration

- Migration of all console.log calls

- Log rotation setup (daily, keep 7 days)

- Environment-based log levels

- Documentation: docs/operations/logging.md

  

**⏸️ YOUR APPROVAL GATE**:

Review the logger configuration:

☐ Uses JSON in production (for parsing)

☐ Uses pretty-print in development (for readability)

☐ RequestId propagates through all logs in a request

☐ Sensitive data not logged (API keys, passwords)

**Test**:

  

  

# Start server, make request

npm run dev

curl http://localhost:3000/chat -d '&#123;"messages":[&#123;"role":"user","content":"test"&#125;]&#125;'

  

# Check logs (should be JSON in one line)

tail -f logs/hive.log | jq

  

# Should see structured logs with all context fields

  

**✅ Validation**:

☐ All console.logs replaced

☐ Logs are structured JSON

☐ RequestId tracks through multi-agent flow

☐ Log rotation working (check after 24h)

☐ No sensitive data in logs

**📊 Log Cost**: ~$0.20 (60K tokens - lots of file changes)

**📈 Step 5.2: Add Prometheus Metrics**

**Your Instruction** (in Cursor):

  

  

Consult the SRE agent.

  

CONTEXT:

Logs tell us what happened. Metrics tell us trends over time.

  

TASK:

Implement Prometheus metrics for:

  - Request rate (requests/sec by endpoint)

  - Response latency (histogram: p50, p95, p99)

  - Error rate (% of requests that failed)

  - Agent invocations (counter by agent name)

  - Token usage (counter by model)

  - Cost (gauge: current spend rate)

  - Circuit breaker state (gauge: 0=closed, 1=open)

  

IMPLEMENTATION:

- Install: prom-client

- Create metrics registry: src/lib/metrics.ts

- Add middleware to track HTTP metrics

- Expose /metrics endpoint (for Prometheus scraper)

- Custom metrics for agent-specific tracking

  

METRIC NAMING:

Follow Prometheus conventions:

- hive_http_requests_total (counter)

- hive_http_request_duration_seconds (histogram)

- hive_agent_invocations_total (counter)

- hive_token_usage_total (counter)

- hive_cost_dollars (gauge)

  

DELIVERABLES:

- Metrics registry and instrumentation

- /metrics endpoint

- Grafana dashboard JSON (for import)

- Documentation on adding custom metrics

  

**⏸️ YOUR APPROVAL GATE**:

Review the metrics design:

☐ Covers the key system behaviors

☐ Has labels for filtering (agent, model, status_code)

☐ Histogram buckets appropriate (0.1s, 0.5s, 1s, 5s, 10s)

☐ /metrics endpoint doesn't require auth (Prometheus needs it)

**Test**:

  

  

# Start server

npm run dev

  

# Make some requests

for i in &#123;1..10&#125;; do

  curl http://localhost:3000/chat -d '&#123;"messages":[&#123;"role":"user","content":"test"&#125;]&#125;'

done

  

# Check metrics endpoint

curl http://localhost:3000/metrics

  

# Should see Prometheus format:

# hive_http_requests_total&#123;method="POST",path="/chat",status="200"&#125; 10

# hive_agent_invocations_total&#123;agent="router"&#125; 10

  

**✅ Validation**:

☐ /metrics endpoint returns valid Prometheus format

☐ Metrics update in real-time

☐ Includes both system and custom metrics

☐ No performance impact (&lt;5ms per request)

**📊 Log Cost**: ~$0.12 (35K tokens)

**📉 Step 5.3: Set Up Grafana Dashboard**

**Your Instruction** (in Cursor):

  

  

Consult the Data Analyst and SRE agents.

  

CONTEXT:

We have metrics. Now we need visualization.

  

TASK:

Data Analyst: Design dashboard layout with panels for:

  - Request rate (line chart, last 1h)

  - Response latency (heatmap, p50/p95/p99)

  - Error rate (gauge, % of requests)

  - Agent usage (bar chart, invocations per agent)

  - Token usage (stacked area, by model over time)

  - Cost burn rate (line chart, $/hour)

  - Circuit breaker status (status indicator)

  

SRE: Create Grafana dashboard JSON:

  - Use Prometheus as data source

  - Auto-refresh every 30s

  - Time range selector (1h, 6h, 24h, 7d)

  - Variables for filtering (agent, model)

  - Alert rules (error rate >5%, cost >$10/hour)

  

DELIVERABLES:

- Grafana dashboard JSON: grafana/dashboards/hive-overview.json

- Docker compose with Grafana + Prometheus

- Setup instructions: docs/operations/monitoring-setup.md

- Screenshot of dashboard for README

  

**⏸️ YOUR APPROVAL GATE**:

Review the dashboard design (Data Analyst will provide mockup):

☐ Most important metrics above the fold

☐ Color coding (green=good, yellow=warning, red=critical)

☐ Time range covers operational needs (1h for incidents, 7d for trends)

**Setup Grafana**:

  

  

# Add to docker-compose.yml

cat >> docker-compose.yml &lt;< 'EOF'

  prometheus:

    image: prom/prometheus:latest

    volumes:

      - ./prometheus.yml:/etc/prometheus/prometheus.yml

      - prometheus-data:/prometheus

    ports:

      - "9090:9090"

  grafana:

    image: grafana/grafana:latest

    volumes:

      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards

      - ./grafana/datasources:/etc/grafana/provisioning/datasources

      - grafana-data:/var/lib/grafana

    ports:

      - "3001:3000"

    environment:

      - GF_SECURITY_ADMIN_PASSWORD=admin

EOF

  

# Create Prometheus config

cat > prometheus.yml &lt;< 'EOF'

global:

  scrape_interval: 15s

  

scrape_configs:

  - job_name: 'hive'

    static_configs:

      - targets: ['host.docker.internal:3000']

EOF

  

# Start monitoring stack

docker-compose up -d prometheus grafana

  

**Access Grafana**:

1. Open [http://localhost:3001](http://localhost:3001)

2. Login (admin/admin)

3. Add Prometheus data source ([http://prometheus:9090](http://prometheus:9090))

4. Import dashboard JSON

**✅ Validation**:

☐ Grafana loads dashboard

☐ All panels show data

☐ Metrics update in real-time

☐ Alerts configured

**📊 Log Cost**: ~$0.10 (30K tokens)

**🔍 Step 5.4: Implement Distributed Tracing**

**Your Instruction** (in Cursor):

  

  

Consult the SRE agent.

  

CONTEXT:

When a request flows through 5+ agents, we need to trace it end-to-end.

  

TASK:

Implement OpenTelemetry distributed tracing:

  - Trace a request from entry (/chat) through all agent invocations

  - Each agent span shows: name, duration, model used, tokens, cost

  - Parent-child relationships visible (Router → Builder → Reviewer)

  - Export traces to Jaeger (visualization)

  

IMPLEMENTATION:

- Install: @opentelemetry/api, @opentelemetry/sdk-node, @opentelemetry/exporter-jaeger

- Initialize tracer: src/lib/tracer.ts

- Instrument HTTP server (auto)

- Manually instrument agent invocations

- Propagate trace context through LangGraph state

  

SPAN ATTRIBUTES:

- agent.name

- agent.model

- agent.tokens_in

- agent.tokens_out

- agent.cost

- agent.duration_ms

  

DELIVERABLES:

- Tracer initialization

- Agent instrumentation

- Jaeger setup in docker-compose

- Documentation: docs/operations/tracing.md

  

**⏸️ YOUR APPROVAL GATE**:

Review the tracing approach:

☐ Spans have meaningful names (not "function1", "function2")

☐ Trace context propagates through async agent calls

☐ Doesn't add >10ms latency per request

☐ Sampling configured (don't trace 100% in prod, maybe 10%)

**Setup Jaeger**:

  

  

# Add to docker-compose.yml

cat >> docker-compose.yml &lt;< 'EOF'

  jaeger:

    image: jaegertracing/all-in-one:latest

    ports:

      - "16686:16686"  # UI

      - "14268:14268"  # Collector

    environment:

      - COLLECTOR_ZIPKIN_HTTP_PORT=9411

EOF

  

docker-compose up -d jaeger

  

**Test**:

  

  

# Make a request

curl http://localhost:3000/chat -d '&#123;"messages":[&#123;"role":"user","content":"build a button"&#125;]&#125;'

  

# Open Jaeger UI

open http://localhost:16686

  

# Search for traces, select one

# Should see waterfall: Router → Builder → (other agents)

  

**✅ Validation**:

☐ Traces appear in Jaeger

☐ Spans show parent-child relationships

☐ Agent names and durations visible

☐ Can trace a request through 5+ agents

**📊 Log Cost**: ~$0.15 (45K tokens)

**🚨 Step 5.5: Configure Alerting**

**Your Instruction** (in Cursor):

  

  

Consult the SRE agent.

  

CONTEXT:

Metrics and dashboards are reactive. We need proactive alerts.

  

TASK:

Configure Prometheus alerting rules for:

  - High error rate: >5% of requests failing (5min average)

  - High latency: p95 >5 seconds (5min average)

  - Agent failures: Any agent failing >50% of time

  - Cost spike: $/hour >$20 (when monthly budget is $1500)

  - Circuit breaker open: Any circuit open >5 minutes

  - Low disk space: &lt;10GB remaining

  

ALERT ROUTING:

- Critical: Page on-call (PagerDuty/Opsgenie)

- Warning: Slack notification

- Info: Log only

  

IMPLEMENTATION:

- Prometheus alert rules: prometheus/alerts.yml

- Alertmanager config: prometheus/alertmanager.yml

- Slack webhook integration

- Test alerts with mock failures

  

DELIVERABLES:

- Alert rules file

- Alertmanager configuration

- Docker compose with Alertmanager

- Runbook links in alerts (so responder knows what to do)

  

**⏸️ YOUR APPROVAL GATE**:

Review alert thresholds:

☐ Not too sensitive (won't page for every hiccup)

☐ Not too lenient (catches real issues)

☐ Has runbook links (alerts are actionable)

☐ Grouped to prevent alert storm (max 1 alert per 5min)

**Setup Alertmanager**:

  

  

# Create alert rules

cat > prometheus/alerts.yml &lt;< 'EOF'

groups:

  - name: hive_alerts

    interval: 30s

    rules:

      - alert: HighErrorRate

        expr: rate(hive_http_requests_total&#123;status=~"5.."&#125;[5m]) > 0.05

        for: 5m

        labels:

          severity: critical

        annotations:

          summary: "High error rate: &#123;&#123; $value &#125;&#125;%"

          runbook: "https://github.com/you/hive-r/docs/runbook.md#high-error-rate"

      - alert: HighCostBurn

        expr: rate(hive_cost_dollars[1h]) * 3600 > 20

        for: 10m

        labels:

          severity: warning

        annotations:

          summary: "Cost burn rate: $&#123;&#123; $value &#125;&#125;/hour"

EOF

  

# Add Alertmanager to docker-compose.yml

cat >> docker-compose.yml &lt;< 'EOF'

  alertmanager:

    image: prom/alertmanager:latest

    volumes:

      - ./prometheus/alertmanager.yml:/etc/alertmanager/alertmanager.yml

    ports:

      - "9093:9093"

EOF

  

**Test alerts**:

  

  

# Trigger high error rate (stop database to cause errors)

docker stop hive-postgres

  

# Make requests (should fail)

for i in &#123;1..20&#125;; do curl http://localhost:3000/chat; done

  

# Wait 5 minutes

# Check Alertmanager: http://localhost:9093

# Should see "HighErrorRate" alert firing

  

**✅ Validation**:

☐ Alerts fire correctly

☐ Slack notifications work

☐ Alerts resolve when issue fixed

☐ No alert spam (grouped properly)

**📊 Log Cost**: ~$0.08 (25K tokens)

**✅ Phase 5 Complete When:**

☐ Structured logging active (all console.logs replaced)

☐ Prometheus metrics exposed

☐ Grafana dashboard showing real data

☐ Distributed tracing working (Jaeger shows spans)

☐ Alerting configured and tested

☐ Monitoring stack runs in Docker

☐ Total cost &lt;$25 for Phase 5

**💡 Key Insight**: You can now debug production issues 10x faster with full observability.

**📸 Take Screenshot**: Of Grafana dashboard + Jaeger trace for your portfolio/docs.

**