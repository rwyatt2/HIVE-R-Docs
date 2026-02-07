---
title: "Phase 2: Cost Monitoring Infrastructure"
sidebar_label: "Phase 2: Cost Monitoring Infrastructure"
---

**

**⏱️ Time**: 6-8 hours

**💰 Budget**: $10-15

**🎯 Goal**: Real-time visibility into AI API costs

**🤖 AI Workers**: HIVE-R Builder + Data Analyst agents

**💰 Step 2.1: Create Cost Tracking Schema**

**Your Instruction** (in Cursor):

  

  

Consult the HIVE-R swarm.

  

CONTEXT:

We need to track every LLM API call with: agent name, model, tokens in/out, cost, timestamp, latency.

  

TASK:

Data Analyst agent: Design optimal schema for cost tracking

  - What fields do we need?

  - What indexes for fast queries?

  - Daily/weekly/monthly aggregation strategy?

  

Builder agent: Implement the schema

  - Update src/lib/db.ts (or create new migration)

  - Add TypeScript types

  - Create helper functions: logUsage(), getDailyCost(), getAgentCosts()

  

REQUIREMENTS:

- Compatible with existing SQLite database

- Efficient queries (add indexes)

- No breaking changes to existing tables

  

DELIVERABLES:

- Schema in docs/architecture/cost-tracking-schema.md

- Migration script: scripts/migrations/001-add-cost-tracking.sql

- TypeScript types: src/types/cost-tracking.ts

- Helper functions: src/lib/cost-tracker.ts

- Tests: tests/lib/cost-tracker.test.ts

  

**⏸️ YOUR APPROVAL GATE**:

Review the schema design:

☐ Includes all necessary fields (see analysis section 5.1)

☐ Has indexes on timestamp and agent name (for fast queries)

☐ Migration is reversible (has DOWN script)

**Example of what you're looking for**:

  

  

CREATE TABLE IF NOT EXISTS token_usage (

  id INTEGER PRIMARY KEY AUTOINCREMENT,

  timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,

  agent_name TEXT NOT NULL,

  model TEXT NOT NULL,

  tokens_input INTEGER,

  tokens_output INTEGER,

  cost_usd REAL,

  latency_ms INTEGER,

  user_id TEXT,

  session_id TEXT

);

  

CREATE INDEX idx_token_usage_timestamp ON token_usage(timestamp);

CREATE INDEX idx_token_usage_agent ON token_usage(agent_name);

  

**Approve**: "Schema looks good, proceed with implementation"

**Test**:

  

  

# Run migration

npm run db:migrate

  

# Verify table exists

sqlite3 data/hive.db "SELECT sql FROM sqlite_master WHERE name='token_usage';"

  

# Run tests

npm test tests/lib/cost-tracker.test.ts

  

**✅ Validation**:

☐ Migration runs successfully

☐ Table has correct schema

☐ Indexes created

☐ Tests pass

**📊 Log Cost**: ~$0.08 (25K tokens)

**📊 Step 2.2: Integrate Cost Tracking Middleware**

**Your Instruction** (in Cursor, open src/index.ts):

  

  

Consult the Builder agent.

  

CONTEXT:

We have cost tracking functions (src/lib/cost-tracker.ts) but they're not hooked into actual LLM calls.

  

TASK:

1. Create middleware that wraps every LLM call

2. Extract token usage from LangChain response metadata

3. Calculate cost based on model pricing

4. Log to token_usage table

5. Check against daily budget limit ($50 default)

6. Throw error if budget exceeded

  

PRICING (hardcode these):

- gpt-4o: $0.005 input, $0.015 output (per 1K tokens)

- gpt-4o-mini: $0.00015 input, $0.0006 output

- claude-3-5-sonnet: $0.003 input, $0.015 output

- claude-3-haiku: $0.00025 input, $0.00125 output

  

INTEGRATION POINTS:

- src/agents/router.ts (wraps GPT-4o call)

- src/agents/builder.ts (wraps Claude call)

- All other agent files

  

REQUIREMENTS:

- Use LangChain callbacks to get token counts

- Minimal performance overhead

- Graceful failure (log error but don't break request)

  

DELIVERABLES:

- Middleware: src/middleware/cost-tracking.ts

- Integration in all agent files

- Budget config: DAILY_BUDGET in .env

- Tests with mocked LLM calls

  

**⏸️ YOUR APPROVAL GATE**:

Check the integration approach:

☐ Uses LangChain callbacks (not custom parsing)

☐ Pricing constants are correct

☐ Budget check happens AFTER logging (so you see what caused overage)

☐ Doesn't slow down requests significantly

**Test manually**:

  

  

# Set low budget for testing

echo "DAILY_BUDGET=0.10" >> .env

  

# Make several requests until budget exceeded

curl -X POST http://localhost:3000/chat \

  -H "Authorization: Bearer $TOKEN" \

  -d '&#123;"messages":[&#123;"role":"user","content":"Hello"&#125;]&#125;'

  

# Should eventually return budget exceeded error

  

# Check logs

sqlite3 data/hive.db "SELECT SUM(cost_usd) FROM token_usage WHERE date(timestamp) = date('now');"

  

**✅ Validation**:

☐ Each LLM call logs to token_usage table

☐ Costs calculated accurately

☐ Budget enforcement works

☐ Performance &lt;10ms overhead per call

**📊 Log Cost**: ~$0.10 (30K tokens)

**📈 Step 2.3: Build Cost Dashboard API**

**Your Instruction** (in Cursor):

  

  

Consult the Data Analyst and Builder agents.

  

CONTEXT:

We're tracking costs in the database. Now we need API endpoints to visualize them.

  

TASK:

Data Analyst: Design queries for these views:

  - Today's total cost

  - Cost per agent (today/week/month)

  - Cost trend over time (daily aggregates)

  - Top 5 most expensive queries

  - Projected monthly cost (based on current burn rate)

  

Builder: Implement REST API endpoints:

  - GET /api/admin/costs/today

  - GET /api/admin/costs/by-agent?period=today|week|month

  - GET /api/admin/costs/trend?days=30

  - GET /api/admin/costs/top-queries?limit=10

  - GET /api/admin/costs/projection

  

REQUIREMENTS:

- Require admin authentication (not all users)

- Efficient queries (use indexes)

- Cache results for 5 minutes (Redis or in-memory)

- Return JSON with clear structure

  

DELIVERABLES:

- New router: src/routers/admin/costs.ts

- Integration in src/routers/admin.ts

- API documentation: docs/api/costs.md

- Tests: tests/routers/admin/costs.test.ts

  

**⏸️ YOUR APPROVAL GATE**:

Review the query designs:

☐ Use proper SQL aggregations (not loading all rows into memory)

☐ Have LIMIT clauses to prevent huge responses

☐ Cache expensive queries

**Test**:

  

  

# Generate some usage data first

npm run scripts/generate-test-data.ts

  

# Test endpoints

curl http://localhost:3000/api/admin/costs/today \

  -H "Authorization: Bearer $ADMIN_TOKEN" | jq

  

# Should return:

# &#123;

#   "date": "2026-02-07",

#   "total_cost": 1.25,

#   "total_requests": 47,

#   "by_agent": &#123;...&#125;

# &#125;

  

**✅ Validation**:

☐ All endpoints return valid JSON

☐ Queries are fast (&lt;100ms)

☐ Auth required (401 without token)

☐ Numbers match database totals

**📊 Log Cost**: ~$0.12 (35K tokens)

**🎨 Step 2.4: Build Cost Dashboard Frontend**

**Your Instruction** (in Cursor, open client/src/pages/Dashboard.tsx):

  

  

Consult the Designer and Builder agents.

  

CONTEXT:

We have cost API endpoints. Now we need a dashboard to visualize them.

  

TASK:

Designer: Propose dashboard layout with:

  - Big number cards (today's cost, this month, projection)

  - Bar chart (cost per agent)

  - Line chart (cost trend over 30 days)

  - Table (top 10 expensive queries)

  - Budget gauge (e.g., $45 / $50 daily limit)

  

Builder: Implement React components:

  - Use existing UI components from client/src/components/

  - Fetch data from /api/admin/costs/* endpoints

  - Auto-refresh every 30 seconds

  - Loading states and error handling

  - Responsive (mobile + desktop)

  

REQUIREMENTS:

- Match existing design system (check client/src/styles/)

- Use Chart.js or Recharts for visualizations

- No external design libraries (use your existing CSS)

  

DELIVERABLES:

- Updated Dashboard.tsx with cost metrics

- New component: CostDashboard.tsx

- Chart components (if needed)

- CSS in Dashboard.module.css

  

**⏸️ YOUR APPROVAL GATE**:

Review the design mockup (Designer will provide):

☐ Layout is clean and readable

☐ Most important metric (today's cost) is prominent

☐ Color coding (green = under budget, yellow = approaching, red = over)

**Approve**: "Design approved, implement it"

**Test**:

  

  

# Start frontend dev server

cd client && npm run dev

  

# Open http://localhost:5173

# Navigate to Dashboard

# Should see cost metrics updating

  

**✅ Validation**:

☐ Dashboard loads without errors

☐ Numbers match API responses

☐ Charts render correctly

☐ Auto-refresh works

☐ Mobile responsive

**📊 Log Cost**: ~$0.15 (45K tokens - includes frontend code)

**🚨 Step 2.5: Implement Budget Alerts**

**Your Instruction** (in Cursor):

  

  

Consult the SRE and Builder agents.

  

CONTEXT:

We can track costs, but we need proactive alerts BEFORE we blow the budget.

  

TASK:

Design alert system with:

  - Check every 10 minutes (cron job or setInterval)

  - Alert thresholds: 50%, 80%, 100%, 120% of daily budget

  - Alert channels: Log to console, Send to Slack (if SLACK_WEBHOOK configured), Email (optional)

  

Implementation:

  - New service: src/services/budget-alerts.ts

  - Tracks which alerts have fired (don't spam same alert)

  - Graceful degradation (if Slack fails, still log)

  

Integration:

  - Start service on server boot

  - Expose /api/admin/alerts endpoint to see history

  

DELIVERABLES:

- Budget alert service

- Slack integration (optional)

- Alert history API

- Documentation in docs/operations/cost-alerts.md

  

**⏸️ YOUR APPROVAL GATE**:

Review alert logic:

☐ Won't spam (tracks already-sent alerts)

☐ Has kill switch (can disable via env var)

☐ Logs to file as backup (in case Slack is down)

**Test**:

  

  

# Set very low budget to trigger alert

echo "DAILY_BUDGET=0.05" >> .env

npm run dev

  

# Make requests until you hit 50% of budget

# Check console logs for alert

  

# If Slack configured:

# Check your Slack channel for alert message

  

**✅ Validation**:

☐ Alerts fire at correct thresholds

☐ No duplicate alerts in same period

☐ Slack integration works (if configured)

☐ System doesn't crash if Slack fails

**📊 Log Cost**: ~$0.08 (25K tokens)

**✅ Phase 2 Complete When:**

☐ Cost tracking logs every LLM call

☐ Dashboard shows real-time costs

☐ Budget alerts fire correctly

☐ APIs documented

☐ Tests passing

☐ Total cost &lt;$15 for Phase 2

**💡 Key Insight**: You can now see exactly where money is going and prevent runaway costs.

**📸 Take Screenshot**: Of your cost dashboard showing real data. This is your proof of progress.

**