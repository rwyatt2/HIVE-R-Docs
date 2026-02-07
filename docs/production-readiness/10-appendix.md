---
title: "Appendix: Troubleshooting"
sidebar_label: "Appendix: Troubleshooting"
---

**

### Troubleshooting: MCP Connection

**Symptom**: Cursor says "Tool not found: consult_hive_swarm"

**Solutions**:

  

  

# 1. Verify HIVE-R server is running

curl http://localhost:3000/health/live

# Should return &#123;"status":"ok"&#125;

  

# 2. Check MCP server starts correctly

node /path/to/HIVE-R/dist/mcp-server.js

# Should not show errors

  

# 3. Verify mcp_config.json path is absolute (not relative)

cat ~/.cursor/mcp_config.json

# "args": ["/Users/you/Desktop/HIVE-R/dist/mcp-server.js"]  ✅

# "args": ["~/Desktop/HIVE-R/dist/mcp-server.js"]            ❌

  

# 4. Restart Cursor completely (not just reload)

  

### Troubleshooting: Tests Failing

**Symptom**: npm test shows failures

**Solutions**:

  

  

# 1. Run single test file to isolate

npm test tests/graph.test.ts

  

# 2. Check if mocks are correct

# Look for: "Cannot read property 'invoke' of undefined"

# → LLM not mocked correctly

  

# 3. Update snapshots if intentional changes

npm test -- --update-snapshots

  

# 4. Ask AI to fix

# In Cursor: "Tests are failing with this error: [paste]. Fix the tests."

  

### Troubleshooting: High Costs

**Symptom**: Token usage exceeds budget

**Solutions**:

  

  

# 1. Identify expensive queries

sqlite3 data/hive.db &lt;< 'EOF'

SELECT agent_name, COUNT(*) as calls, SUM(cost_usd) as total_cost

FROM token_usage

WHERE date(timestamp) = date('now')

GROUP BY agent_name

ORDER BY total_cost DESC

LIMIT 10;

EOF

  

# 2. Check cache hit rate

curl http://localhost:3000/metrics | grep cache_hit_rate

# If &lt;30%, investigate why caching isn't working

  

# 3. Reduce daily budget temporarily

echo "DAILY_BUDGET=10.00" >> .env

npm restart

  

# 4. Enable more aggressive rate limiting

# Edit src/middleware/rate-limit.ts

# Change: arrivalRate: 10 (was 20)

  

### Troubleshooting: Deployment Failures

**Symptom**: docker-compose up fails

**Solutions**:

  

  

# 1. Check Docker is running

docker ps

# If error, restart Docker daemon

  

# 2. Check for port conflicts

lsof -i :3000

# If something else is using port 3000, stop it or change port

  

# 3. Verify environment variables

docker-compose config

# Should show resolved env vars (not $&#123;VAR&#125; placeholders)

  

# 4. Check logs

docker-compose logs app

# Look for startup errors

  

# 5. Rebuild from scratch

docker-compose down -v  # Remove volumes

docker-compose build --no-cache

docker-compose up -d

  

### Troubleshooting: Monitoring Not Working

**Symptom**: Grafana shows no data

**Solutions**:

  

  

# 1. Check Prometheus is scraping

curl http://localhost:9090/targets

# Should show "UP" for hive target

  

# 2. Verify metrics endpoint

curl http://localhost:3000/metrics

# Should return Prometheus format

  

# 3. Check Prometheus config

cat prometheus.yml

# Target should be: host.docker.internal:3000 (not localhost)

  

# 4. Restart monitoring stack

docker-compose restart prometheus grafana

  

**🚀 NEXT STEPS: CONTINUOUS IMPROVEMENT**

Your system is production-ready, but there's always room for improvement:

**Month 2 Goals:**

☐ Achieve 80%+ test coverage

☐ Reduce costs by additional 20% (fine-tuning)

☐ Add 2-3 more specialized agents

☐ Implement user feedback loop

☐ Start building agent marketplace

**Month 3 Goals:**

☐ Migrate to PostgreSQL (for scaling)

☐ Add multi-region deployment

☐ Implement self-improving agents

☐ Launch public beta

☐ Achieve $10K MRR

**Month 6 Goals:**

☐ 1000+ active users

☐ 99.95% uptime SLA

☐ Self-sustaining (revenue > costs)

☐ Community-built agents in marketplace

☐ Featured in AI engineering publications

**📖 YOUR ONLINE GUIDE IS READY**

This document IS your online guide. To use it:

1. **Bookmark this page** or save as PDF

2. **Work through one step at a time** (checkboxes help track progress)

3. **Copy-paste prompts exactly** into Cursor or Claude

4. **Review AI's work** at each approval gate

5. **Validate** before moving to next step

6. **Log costs** in budget-tracker.md after each step

**Remember**:

• ✅ You are the **Thought Leader** (strategic decisions, approval gates)

• 🤖 AI is the **Worker** (coding, testing, documentation)

• 📋 This guide is your **Playbook** (step-by-step instructions)

• ✓ Checkboxes are your **Progress Tracker**

**🎉 You're ready to build! Start with Phase 0 and work through systematically.**

**Good luck, and remember**: The best AI systems are built iteratively, with human oversight at every critical decision point. You've got this! 🚀

_Last Updated: February 7, 2026_

_Guide Version: 1.0_

_Maintained by: Your AI Development Team_