---
title: "Phase 6: Performance Optimization"
sidebar_label: "Phase 6: Performance Optimization"
---

**

**⏱️ Time**: 10-12 hours

**💰 Budget**: $25-30

**🎯 Goal**: 50% cost reduction, 80% latency reduction (for cached requests)

**🤖 AI Workers**: HIVE-R Data Analyst + Builder + Planner agents

**⚡ Step 6.1: Implement Response Caching**

**Your Instruction** (in Cursor):

  

  

Consult the Planner and Builder agents.

  

CONTEXT:

Many user requests are similar or identical. We're calling expensive LLMs unnecessarily.

  

TASK:

Planner: Design caching strategy

  - What to cache? (completed agent responses)

  - Cache key? (hash of messages + agent + model)

  - TTL? (1 hour for code generation, 24h for planning)

  - Invalidation? (user can force refresh, or version changes)

  - Storage? (Redis for shared cache, or in-memory for dev)

  

Builder: Implement semantic caching

  - Use embeddings to find similar queries (cosine similarity >0.95)

  - Store: embedding vector, original query, response, metadata

  - Check cache before every agent invocation

  - Metrics: cache_hit_rate, cache_latency

  

SEMANTIC SEARCH:

- Generate embedding for new query (OpenAI text-embedding-ada-002)

- Vector search in Redis (with RedisSearch) or Qdrant

- If similarity >0.95, return cached response

- If miss, invoke agent and cache result

  

DELIVERABLES:

- Cache strategy doc: docs/architecture/caching.md

- Redis setup in docker-compose

- Semantic cache implementation: src/lib/semantic-cache.ts

- Integration in agent middleware

- Cache management endpoints: /api/admin/cache/clear

  

**⏸️ YOUR APPROVAL GATE**:

Review the caching strategy:

☐ Similarity threshold (0.95) is reasonable (not too loose)

☐ TTLs make sense (short for dynamic, long for static)

☐ Cache doesn't store sensitive data (user IDs anonymized)

☐ Has manual override (users can bypass cache with flag)

**Setup Redis**:

  

  

# Add to docker-compose.yml

cat >> docker-compose.yml &lt;< 'EOF'

  redis:

    image: redis/redis-stack:latest  # Includes RedisSearch

    ports:

      - "6379:6379"

      - "8001:8001"  # RedisInsight UI

    volumes:

      - redis-data:/data

EOF

  

docker-compose up -d redis

  

**Test caching**:

  

  

# First request (cache miss)

time curl http://localhost:3000/chat -d '&#123;"messages":[&#123;"role":"user","content":"build a login form"&#125;]&#125;'

# Should take 3-5 seconds

  

# Second identical request (cache hit)

time curl http://localhost:3000/chat -d '&#123;"messages":[&#123;"role":"user","content":"build a login form"&#125;]&#125;'

# Should take &lt;100ms

  

# Similar request (semantic hit)

time curl http://localhost:3000/chat -d '&#123;"messages":[&#123;"role":"user","content":"create a login page"&#125;]&#125;'

# Should also be fast (similar query)

  

# Check metrics

curl http://localhost:3000/metrics | grep cache_hit_rate

# Should show >0.5 (50% hit rate after a few requests)

  

**✅ Validation**:

☐ Cache reduces latency by 80%+ for hits

☐ Semantic search works (similar queries match)

☐ Cache doesn't serve stale data (TTL works)

☐ Cache hit rate >30% after 100 requests

**💰 Cost Savings**: 50% reduction if cache hit rate = 50%

**📊 Log Cost**: ~$0.20 (60K tokens)

**🎯 Step 6.2: Implement Intelligent Model Routing**

**Your Instruction** (in Cursor):

  

  

Consult the Data Analyst and Planner agents.

  

CONTEXT:

We're using expensive models (Claude, GPT-4o) for all tasks. Many tasks could use cheaper models.

  

TASK:

Data Analyst: Analyze task complexity

  - Simple tasks (&lt;100 tokens output): GPT-4o-mini ($0.15/MTok)

  - Medium tasks (100-500 tokens): GPT-4o ($5/MTok)

  - Complex tasks (>500 tokens, code generation): Claude ($3/MTok)

  

Planner: Design routing logic

  - Estimate task complexity from prompt (use heuristics or small model)

  - Select cheapest model that can handle it

  - Track model performance (did cheaper model succeed?)

  - Auto-upgrade if cheap model fails repeatedly

  

HEURISTICS:

- "simple question", "what is", "explain" → cheap model

- "build", "implement", "refactor" → expensive model

- "review", "analyze", "compare" → medium model

  

IMPLEMENTATION:

- Model router: src/lib/model-router.ts

- Complexity estimator (rule-based or ML)

- Fallback if cheap model fails

- Metrics: cost_saved_by_routing

  

DELIVERABLES:

- Model routing logic

- Integration in agent invocations

- Performance tracking dashboard

- A/B test results (routed vs always-expensive)

  

**⏸️ YOUR APPROVAL GATE**:

Review the routing heuristics:

☐ Rules make sense (not too aggressive on downgrading)

☐ Has safety net (upgrades if cheap model fails)

☐ Tracks performance (can revert if quality drops)

☐ Doesn't route security tasks to cheap models

**Test routing**:

  

  

# Simple query (should use cheap model)

curl http://localhost:3000/chat -d '&#123;"messages":[&#123;"role":"user","content":"what is react?"&#125;]&#125;'

# Check logs for: "Model selected: gpt-4o-mini"

  

# Complex query (should use expensive model)

curl http://localhost:3000/chat -d '&#123;"messages":[&#123;"role":"user","content":"build a full authentication system with JWT"&#125;]&#125;'

# Check logs for: "Model selected: claude-3-5-sonnet"

  

# After 100 requests, check savings

curl http://localhost:3000/metrics | grep cost_saved

# Should show significant savings (30-50%)

  

**✅ Validation**:

☐ Simple tasks use cheap models

☐ Complex tasks use expensive models

☐ Quality doesn't drop (test manually or with eval set)

☐ Cost savings 30-50%

**💰 Cost Savings**: Additional 30-40% on top of caching

**📊 Log Cost**: ~$0.15 (45K tokens)

**🗄️ Step 6.3: Optimize Database Queries**

**Your Instruction** (in Cursor):

  

  

Consult the Data Analyst and Builder agents.

  

CONTEXT:

Database queries can be slow, especially as data grows. Optimize now before it becomes a problem.

  

TASK:

Data Analyst: Analyze current queries

  - Run EXPLAIN on all queries in src/lib/*.ts

  - Identify slow queries (>100ms)

  - Recommend indexes

  - Check for N+1 query problems

  

Builder: Implement optimizations

  - Add missing indexes

  - Rewrite slow queries (use JOINs instead of multiple SELECTs)

  - Add query result caching (for analytics)

  - Use connection pooling

  - Consider read replicas if high read load

  

FOCUS AREAS:

1. token_usage table (heavy writes, frequent aggregations)

2. chat_history table (joins with users)

3. agent_state table (if exists)

  

DELIVERABLES:

- Database analysis: docs/architecture/db-optimization.md

- Index creation migration

- Query rewrites (before/after benchmarks)

- Connection pool configuration

- Performance comparison report

  

**⏸️ YOUR APPROVAL GATE**:

Review the proposed indexes:

☐ Indexes on foreign keys (user_id, session_id)

☐ Indexes on frequently queried columns (timestamp, agent_name)

☐ Not over-indexed (too many indexes slow writes)

☐ Composite indexes for common query patterns

**Run optimization**:

  

  

# Run database migrations

npm run db:migrate

  

# Run benchmark before/after

npm run benchmark:db

  

# Check query performance

sqlite3 data/hive.db &lt;< 'EOF'

EXPLAIN QUERY PLAN 

SELECT SUM(cost_usd) FROM token_usage 

WHERE timestamp > datetime('now', '-1 day') 

GROUP BY agent_name;

EOF

# Should show "USING INDEX idx_token_usage_timestamp"

  

**✅ Validation**:

☐ All slow queries now &lt;50ms

☐ Indexes used (check EXPLAIN output)

☐ Write performance not degraded

☐ Connection pool active (check logs)

**📊 Log Cost**: ~$0.10 (30K tokens)

**🔥 Step 6.4: Implement Request Rate Limiting**

**Your Instruction** (in Cursor):

  

  

Consult the Security and Builder agents.

  

CONTEXT:

Users (or attackers) could spam our API, causing cost explosions.

  

TASK:

Implement multi-tier rate limiting:

  - Anonymous users: 10 requests/hour

  - Authenticated free users: 100 requests/hour

  - Authenticated paid users: 1000 requests/hour

  - Admin users: No limit

  

IMPLEMENTATION:

- Use @hono/rate-limit or custom Redis-based limiter

- Track by: IP address (anonymous) or user ID (authenticated)

- Headers: X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset

- Response: 429 Too Many Requests with retry-after header

  

SPECIAL CASES:

- Burst allowance (allow 20 requests in 1 minute, but 100/hour average)

- Whitelist certain IPs (monitoring systems)

- Different limits for different endpoints (GET /health unlimited)

  

DELIVERABLES:

- Rate limiting middleware: src/middleware/rate-limit.ts

- Configuration in .env (limits per tier)

- User tier detection (from auth token)

- Tests for rate limit enforcement

- Documentation for API consumers

  

**⏸️ YOUR APPROVAL GATE**:

Review rate limits:

☐ Limits are reasonable (not too strict for legitimate use)

☐ Different tiers make sense

☐ Headers inform users of limits

☐ Has bypass for emergencies (admin override)

**Test**:

  

  

# Test anonymous limit (10/hour)

for i in &#123;1..15&#125;; do

  curl http://localhost:3000/chat -d '&#123;"messages":[&#123;"role":"user","content":"test"&#125;]&#125;'

done

# After 10 requests, should get 429

  

# Check headers

curl -I http://localhost:3000/chat

# Should see: X-RateLimit-Remaining: 9

  

# Test with auth token (higher limit)

for i in &#123;1..20&#125;; do

  curl http://localhost:3000/chat \

    -H "Authorization: Bearer $TOKEN" \

    -d '&#123;"messages":[&#123;"role":"user","content":"test"&#125;]&#125;'

done

# Should succeed (authenticated = 100/hour)

  

**✅ Validation**:

☐ Rate limiting enforces limits correctly

☐ Different tiers have different limits

☐ Headers inform users

☐ Burst allowance works

**📊 Log Cost**: ~$0.08 (25K tokens)

**📦 Step 6.5: Optimize Bundle Size (Frontend)**

**Your Instruction** (in Cursor, open client/package.json):

  

  

Consult the Builder agent.

  

CONTEXT:

Large JavaScript bundles slow page load and hurt user experience.

  

TASK:

Analyze and optimize frontend bundle:

  - Run bundle analyzer (webpack-bundle-analyzer or similar)

  - Identify large dependencies

  - Implement code splitting (lazy load routes)

  - Tree-shake unused code

  - Use lighter alternatives for heavy libs

  

TARGETS:

- Initial load: &lt;200KB gzipped

- Time to Interactive: &lt;3s on 4G

  

COMMON OPTIMIZATIONS:

- Lazy load dashboard page (only load when navigating)

- Use date-fns instead of moment.js (if used)

- Dynamic imports for chart libraries

- Remove unused CSS (PurgeCSS)

- Compress images

  

DELIVERABLES:

- Bundle analysis report: docs/architecture/bundle-optimization.md

- Optimized webpack/vite config

- Lazy loading for routes

- Before/after Lighthouse scores

  

**⏸️ YOUR APPROVAL GATE**:

Review bundle analysis:

☐ Identifies biggest contributors (chart lib, etc)

☐ Proposes reasonable alternatives (not "rewrite everything")

☐ Code splitting doesn't break functionality

☐ Images optimized (WebP, compressed)

**Run optimization**:

  

  

cd client

  

# Analyze bundle

npm run build

npm run analyze  # If configured, or:

npx vite-bundle-visualizer

  

# Check bundle size

ls -lh dist/assets/*.js

# Should be &lt;200KB for main bundle

  

# Run Lighthouse

npx lighthouse http://localhost:5173 --output=json

  

# Compare before/after Performance score

  

**✅ Validation**:

☐ Bundle size reduced by 30%+

☐ Lighthouse Performance score >90

☐ Time to Interactive &lt;3s

☐ All features still work

**📊 Log Cost**: ~$0.10 (30K tokens)

**✅ Phase 6 Complete When:**

☐ Response caching implemented (50%+ hit rate)

☐ Model routing saving 30-40% costs

☐ Database queries optimized (&lt;50ms)

☐ Rate limiting active

☐ Frontend bundle optimized (&lt;200KB)

☐ Overall cost reduced by 60-70%

☐ Latency reduced by 80% for cached requests

☐ Total cost &lt;$30 for Phase 6

**💡 Key Insight**: Performance = Cost Savings. Fast systems are cheap systems.

**🎉 MAJOR MILESTONE**: Your system now runs 60-70% cheaper with better performance.

**