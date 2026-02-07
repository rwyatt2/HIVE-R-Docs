---
title: "Phase 4: Reliability & Fallbacks"
sidebar_label: "Phase 4: Reliability & Fallbacks"
---

**

**⏱️ Time**: 8-10 hours

**💰 Budget**: $15-20

**🎯 Goal**: System survives LLM API outages

**🤖 AI Workers**: HIVE-R SRE + Builder agents

**🔄 Step 4.1: Implement Circuit Breaker**

**Your Instruction** (in Cursor):

  

  

Consult the SRE and Builder agents.

  

CONTEXT:

If OpenAI/Anthropic APIs go down, our entire system crashes. We need circuit breaker pattern.

  

TASK:

Implement circuit breaker for all LLM calls with these states:

  1. CLOSED (normal operation)

  2. OPEN (too many failures, stop calling API)

  3. HALF-OPEN (try one request to test if API recovered)

  

CONFIGURATION:

- Max failures: 3

- Timeout: 60 seconds (after 3 failures, wait 60s before retry)

- Track separately per model (OpenAI circuit != Anthropic circuit)

  

IMPLEMENTATION:

- New class: src/lib/circuit-breaker.ts

- Wrap all LLM calls (in middleware or agent wrapper)

- Emit events: circuit_opened, circuit_closed

- Log to monitoring system

  

REQUIREMENTS:

- Thread-safe (if we add workers later)

- Configurable via environment variables

- Graceful errors (user-friendly messages)

  

DELIVERABLES:

- CircuitBreaker class with tests

- Integration in src/middleware/llm-wrapper.ts

- Monitoring dashboard shows circuit state

- Documentation

  

**⏸️ YOUR APPROVAL GATE**:

Review the circuit breaker logic:

☐ States transition correctly (CLOSED → OPEN → HALF_OPEN → CLOSED)

☐ Timeouts are reasonable (60s, not 10 minutes)

☐ Per-model tracking (OpenAI issues don't break Anthropic)

☐ User gets friendly error: "AI service temporarily unavailable"

**Test manually**:

  

  

# Start server

npm run dev

  

# Simulate API failure (block network to api.openai.com)

sudo iptables -A OUTPUT -d api.openai.com -j DROP

  

# Make requests until circuit opens

curl http://localhost:3000/chat [repeat 3 times]

  

# Should get "Circuit breaker open" error

  

# Restore network

sudo iptables -D OUTPUT -d api.openai.com -j DROP

  

# Wait 60 seconds, try again

# Circuit should move to HALF_OPEN, then CLOSED

  

**✅ Validation**:

☐ Circuit opens after 3 failures

☐ Circuit closes after successful recovery

☐ Users get clear error messages

☐ System doesn't crash during outage

**📊 Log Cost**: ~$0.10 (30K tokens)

**🔀 Step 4.2: Add Model Fallbacks**

**Your Instruction** (in Cursor, open src/agents/router.ts):

  

  

Consult the SRE agent.

  

CONTEXT:

Router uses GPT-4o exclusively. If OpenAI is down, routing fails entirely.

  

TASK:

Implement fallback chain:

  1. Primary: GPT-4o with structured output

  2. Fallback 1: GPT-4o without structured output (parse text)

  3. Fallback 2: Claude 3.5 Sonnet

  4. Final fallback: Rule-based router (no LLM)

  

RULE-BASED LOGIC:

If all LLMs fail, use simple keyword matching:

  - "build", "code", "implement" → builder

  - "design", "UI", "UX" → designer

  - "security", "vulnerability" → security

  - "deploy", "production" → sre

  - Default → product_manager (safe choice)

  

IMPLEMENTATION:

- Try each fallback in sequence

- Log which fallback was used (for monitoring)

- Track fallback usage (how often primary fails?)

  

DELIVERABLES:

- Enhanced router with fallback chain

- Rule-based fallback function

- Metrics: fallback_usage_count by level

- Tests for each fallback scenario

  

**⏸️ YOUR APPROVAL GATE**:

Review the fallback logic:

☐ Fallback order makes sense (cost-aware: expensive → cheap → free)

☐ Rule-based router is sensible (covers common cases)

☐ Each fallback is tested

☐ Logs clearly show which fallback was used

**Test**:

  

  

# Mock failures to test fallbacks

# Use environment variable to force fallback mode

FORCE_FALLBACK_LEVEL=2 npm run dev

  

# Make request, check logs

curl http://localhost:3000/chat -d '&#123;"messages":[&#123;"role":"user","content":"build me a login page"&#125;]&#125;'

  

# Log should show: "Router fallback level 2 (Claude) used"

  

**✅ Validation**:

☐ Fallback chain works correctly

☐ Rule-based router handles common cases

☐ Performance acceptable (fallbacks don't add 10s delay)

☐ Metrics track fallback usage

**📊 Log Cost**: ~$0.12 (35K tokens)

**🔁 Step 4.3: Add Retry Logic with Exponential Backoff**

**Your Instruction** (in Cursor):

  

  

Consult the Builder agent.

  

CONTEXT:

Transient errors (rate limits, network hiccups) should be retried, not immediately failed.

  

TASK:

Implement retry logic for all LLM API calls:

  - Max retries: 3

  - Backoff strategy: exponential (1s, 2s, 4s)

  - Retry on: 429 (rate limit), 500 (server error), network timeout

  - Don't retry on: 400 (bad request), 401 (auth error)

  

INTEGRATION:

- Add to LLM wrapper middleware

- Works with circuit breaker (retries before opening circuit)

- Configurable via environment variables

  

REQUIREMENTS:

- Use existing retry library (p-retry or tenacity.js)

- Log each retry attempt

- Add jitter to prevent thundering herd

  

DELIVERABLES:

- Retry wrapper: src/lib/retry.ts

- Integration in LLM middleware

- Tests with mocked failures

- Documentation on retry behavior

  

**⏸️ YOUR APPROVAL GATE**:

Review retry configuration:

☐ Max 3 retries (not infinite loop)

☐ Exponential backoff (not fixed delay)

☐ Jitter added (randomizes delay slightly)

☐ Only retries transient errors (not auth failures)

**Test**:

  

  

# Mock transient failure (rate limit)

# Test that system retries 3 times, then fails

  

npm test tests/lib/retry.test.ts

  

# Check test includes:

# - Succeeds on 2nd attempt

# - Fails after 3 attempts

# - Respects backoff timing

  

**✅ Validation**:

☐ Retries transient errors

☐ Respects max retry limit

☐ Backoff timing correct

☐ Doesn't retry non-transient errors

**📊 Log Cost**: ~$0.08 (25K tokens)

**🏥 Step 4.4: Health Check System**

**Your Instruction** (in Cursor):

  

  

Consult the SRE agent.

  

CONTEXT:

We have /health/live and /health/ready endpoints but they're basic. Enhance them.

  

TASK:

Implement comprehensive health checks:

  - /health/live: Is process alive? (Always 200 unless crashed)

  - /health/ready: Is system ready to serve traffic?

    - Database connection OK?

    - LLM APIs reachable?

    - Circuit breakers not all open?

    - Disk space available?

    - Memory usage < 90%?

  

RESPONSE FORMAT:

&#123;

  "status": "healthy" | "degraded" | "unhealthy",

  "checks": &#123;

    "database": &#123;"status": "ok", "latency_ms": 5&#125;,

    "openai_api": &#123;"status": "ok", "circuit": "closed"&#125;,

    "anthropic_api": &#123;"status": "ok", "circuit": "closed"&#125;,

    "disk_space": &#123;"status": "ok", "free_gb": 42&#125;,

    "memory": &#123;"status": "ok", "usage_pct": 65&#125;

  &#125;,

  "timestamp": "2026-02-07T12:34:56Z"

&#125;

  

DELIVERABLES:

- Enhanced health check: src/routers/health.ts

- Health check functions: src/lib/health-checks.ts

- Tests for each check

- Kubernetes liveness/readiness probes configured

  

**⏸️ YOUR APPROVAL GATE**:

Review health check logic:

☐ Checks are fast (&lt;500ms total)

☐ Doesn't make expensive calls (test with cheap query)

☐ Returns 503 (not 200) when unhealthy

☐ Kubernetes probes use correct endpoints

**Test**:

  

  

# Healthy system

curl http://localhost:3000/health/ready

# Should return 200 with all checks "ok"

  

# Simulate unhealthy (stop database)

docker stop hive-postgres

  

curl http://localhost:3000/health/ready

# Should return 503 with database check "failed"

  

**✅ Validation**:

☐ Health checks detect real issues

☐ Fast response (&lt;500ms)

☐ Kubernetes probes configured

☐ Monitoring system scrapes /health/ready

**📊 Log Cost**: ~$0.08 (25K tokens)

**📝 Step 4.5: Document Failure Scenarios**

**Your Instruction** (in Cursor):

  

  

Consult the Tech Writer and SRE agents.

  

TASK:

Create runbook for common failure scenarios:

  1. OpenAI API down → What happens? (circuit breaker, fallback to Claude)

  2. Database locked → How to recover? (restart, check for long-running queries)

  3. Memory leak → How to detect and mitigate? (health checks, auto-restart)

  4. Budget exceeded → What to do? (alerts fired, manual intervention)

  5. Agent stuck in loop → How to break? (timeout, manual kill)

  

FORMAT:

For each scenario:

  - Symptoms (how you'll know it's happening)

  - Detection (which monitoring alert fires)

  - Impact (what breaks for users)

  - Automatic mitigation (what system does)

  - Manual intervention (what human should do)

  - Prevention (how to avoid in future)

  

OUTPUT:

- docs/operations/runbook.md

- Link from README for operators

  

**⏸️ YOUR APPROVAL GATE**:

Review the runbook:

☐ Covers the 5 most likely failures

☐ Actionable steps (not vague "investigate")

☐ Includes commands to run (copy-pasteable)

☐ Has escalation path (who to call if manual steps fail)

**✅ Validation**:

☐ Runbook exists and is comprehensive

☐ You understand how to respond to each scenario

☐ Commands tested (actually work)

**📊 Log Cost**: ~$0.05 (15K tokens)

**✅ Phase 4 Complete When:**

☐ Circuit breaker implemented and tested

☐ Model fallbacks working (4-level chain)

☐ Retry logic active

☐ Health checks comprehensive

☐ Runbook documented

☐ System survives simulated OpenAI outage

☐ Total cost &lt;$20 for Phase 4

**💡 Key Insight**: Your system can now withstand LLM API outages without crashing.

**🐝 HIVE-R Production Readiness Guide (CONTINUED)**

**