---
title: "Phase 8: Production Deployment"
sidebar_label: "Phase 8: Production Deployment"
---

**

**⏱️ Time**: 8-10 hours

**💰 Budget**: $10-15

**🎯 Goal**: HIVE-R running in production, serving real traffic

**🤖 AI Workers**: HIVE-R SRE + Builder + Data Analyst agents

**🌍 Step 8.1: Production Environment Setup**

**Your Instruction** (in Cursor):

  

  

Consult the SRE agent.

  

CONTEXT:

We need production infrastructure that's reliable, scalable, and cost-effective.

  

TASK:

Recommend and document production architecture:

OPTION A: Single Server (MVP, &lt;100 users)

  - VPS (DigitalOcean, Linode, Vultr) - $20-40/month

  - Docker Compose for orchestration

  - Caddy for HTTPS + reverse proxy

  - Automated backups to S3

  - Cost: ~$50/month

OPTION B: Multi-Server (100-1K users)

  - Load balancer + 2-3 app servers

  - Managed PostgreSQL

  - Managed Redis

  - Docker Swarm or Kubernetes

  - Cost: ~$200-300/month

OPTION C: Cloud Managed (1K+ users)

  - Cloud Run / ECS Fargate (auto-scaling)

  - RDS PostgreSQL

  - ElastiCache Redis

  - CloudFront CDN

  - Cost: $500-1500/month (usage-based)

  

Based on your needs, recommend one and create:

  - Infrastructure diagram

  - Setup scripts (terraform or bash)

  - Environment configuration

  - DNS setup instructions

  - SSL certificate setup (Let's Encrypt)

  

DELIVERABLES:

- docs/deployment/production-architecture.md

- Infrastructure as Code: deploy/terraform/ or deploy/docker-swarm/

- Setup checklist: docs/deployment/production-checklist.md

  

**⏸️ YOUR APPROVAL GATE**:

Review the architecture recommendation:

☐ Matches your user scale and budget

☐ Has backup and recovery plan

☐ Monitoring integrated

☐ SSL/HTTPS configured

☐ Secrets managed securely (not in git)

**For Option A (Single Server - Recommended for MVP)**:

  

  

# On your production server (Ubuntu 22.04)

# Copy this setup script to server

  

#!/bin/bash

# deploy/scripts/production-setup.sh

  

# Install Docker

curl -fsSL https://get.docker.com | sh

sudo usermod -aG docker $USER

  

# Install Caddy (for HTTPS)

sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https

curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/caddy-stable-archive-keyring.gpg] https://dl.cloudsmith.io/public/caddy/stable/deb/debian any-version main" | sudo tee /etc/apt/sources.list.d/caddy-stable.list

sudo apt update && sudo apt install -y caddy

  

# Create app directory

mkdir -p /opt/hive-r

cd /opt/hive-r

  

# Clone repo (or copy files)

git clone https://github.com/yourusername/HIVE-R.git .

  

# Setup environment

cp .env.example .env

nano .env  # Fill in production values

  

# Start services

docker-compose -f docker-compose.prod.yml up -d

  

# Configure Caddy for HTTPS

cat > /etc/caddy/Caddyfile &lt;< 'EOF'

api.yourdomain.com &#123;

    reverse_proxy localhost:3000

    encode gzip

&#125;

EOF

  

sudo systemctl restart caddy

  

echo "✅ Production setup complete!"

echo "Visit https://api.yourdomain.com/health/ready"

  

**Test production deployment**:

  

  

# On your local machine, deploy to production

./deploy/scripts/deploy-production.sh

  

# Wait for deployment

# Check health

curl https://api.yourdomain.com/health/ready

  

# Should return: &#123;"status":"healthy"&#125;

  

**✅ Validation**:

☐ Server accessible via HTTPS

☐ SSL certificate valid (Let's Encrypt)

☐ Health checks passing

☐ All services running (docker ps)

☐ Logs visible (docker logs)

**📊 Log Cost**: ~$0.10 (30K tokens)

**📊 Step 8.2: Production Monitoring Setup**

**Your Instruction** (in Cursor):

  

  

Consult the SRE agent.

  

CONTEXT:

Production is running. Now we need visibility into its health.

  

TASK:

Set up production monitoring:

  1. Application monitoring

     - Sentry (error tracking) - already configured

     - Prometheus + Grafana (metrics) - deploy to prod

     - Health check monitoring (UptimeRobot, Pingdom)

  2. Infrastructure monitoring

     - Server resources (CPU, RAM, disk)

     - Docker container health

     - Network traffic

  3. Business metrics

     - Active users

     - Request volume

     - Cost per day

     - Revenue (if applicable)

  4. Alerting

     - Slack for warnings

     - PagerDuty/Opsgenie for critical (optional)

     - Email for daily digests

  

DELIVERABLES:

- Production monitoring stack (deployed)

- Dashboards for all key metrics

- Alert rules configured

- On-call runbook

- Monitoring access credentials (secure)

  

**⏸️ YOUR APPROVAL GATE**:

Review monitoring coverage:

☐ Covers all critical services

☐ Alerts are actionable (not noise)

☐ Has escalation path

☐ Dashboard accessible to team

**Deploy monitoring**:

  

  

# On production server

cd /opt/hive-r

  

# Update docker-compose.prod.yml to include monitoring

cat >> docker-compose.prod.yml &lt;< 'EOF'

  prometheus:

    image: prom/prometheus:latest

    volumes:

      - ./prometheus.yml:/etc/prometheus/prometheus.yml

      - prometheus-data:/prometheus

    restart: unless-stopped

  grafana:

    image: grafana/grafana:latest

    volumes:

      - grafana-data:/var/lib/grafana

    environment:

      - GF_SECURITY_ADMIN_PASSWORD=$&#123;GRAFANA_PASSWORD&#125;

      - GF_SERVER_DOMAIN=grafana.yourdomain.com

    restart: unless-stopped

  

volumes:

  prometheus-data:

  grafana-data:

EOF

  

# Add to Caddy for public access

cat >> /etc/caddy/Caddyfile &lt;< 'EOF'

grafana.yourdomain.com &#123;

    reverse_proxy grafana:3000

&#125;

EOF

  

# Restart

docker-compose -f docker-compose.prod.yml up -d

sudo systemctl restart caddy

  

**Setup external monitoring** (UptimeRobot):

1. Go to [uptimerobot.com](http://uptimerobot.com) (free tier)

2. Add HTTP monitor for [https://api.yourdomain.com/health/ready](https://api.yourdomain.com/health/ready)

3. Set check interval to 5 minutes

4. Add alert contacts (email, Slack)

**✅ Validation**:

☐ Grafana accessible at [grafana.yourdomain.com](http://grafana.yourdomain.com)

☐ Dashboards showing live data

☐ Alerts configured in UptimeRobot

☐ Received test alert successfully

**📊 Log Cost**: ~$0.08 (25K tokens)

**🔐 Step 8.3: Production Security Hardening**

**Your Instruction** (in Cursor):

  

  

Consult the Security agent.

  

CONTEXT:

Production must be secure against attacks.

  

TASK:

Harden production environment:

  1. Server hardening

     - Firewall (UFW): Allow only 80, 443, 22 (SSH)

     - SSH: Key-only auth, disable password, non-standard port

     - Fail2ban: Block brute-force attempts

     - Automatic security updates

  2. Application hardening

     - Rate limiting (already implemented, verify active)

     - WAF (Cloudflare free tier or ModSecurity)

     - DDoS protection (Cloudflare)

     - Security headers (CSP, HSTS, X-Frame-Options)

  3. Data protection

     - Database encryption at rest

     - SSL/TLS for all connections

     - Secrets in environment variables (not in git)

     - Regular backups (automated, tested)

  4. Access control

     - Least privilege principle

     - Separate staging/production credentials

     - Audit logging (who accessed what)

     - 2FA for admin accounts

  

DELIVERABLES:

- Server hardening script: deploy/scripts/harden-server.sh

- Security audit report

- Backup and restore procedures

- Access control documentation

  

**⏸️ YOUR APPROVAL GATE**:

Review security measures:

☐ All unnecessary ports closed

☐ SSH hardened (keys only, non-default port)

☐ WAF active (even free Cloudflare helps)

☐ Backups automated and tested

**Run hardening script**:

  

  

#!/bin/bash

# deploy/scripts/harden-server.sh

  

# Setup firewall

sudo ufw default deny incoming

sudo ufw default allow outgoing

sudo ufw allow 22/tcp    # SSH (change port later)

sudo ufw allow 80/tcp    # HTTP

sudo ufw allow 443/tcp   # HTTPS

sudo ufw enable

  

# Harden SSH

sudo sed -i 's/#Port 22/Port 2222/' /etc/ssh/sshd_config

sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config

sudo systemctl restart sshd

  

# Install fail2ban

sudo apt install -y fail2ban

sudo systemctl enable fail2ban

  

# Automatic security updates

sudo apt install -y unattended-upgrades

sudo dpkg-reconfigure -plow unattended-upgrades

  

# Setup automated backups

cat > /etc/cron.daily/backup-hive &lt;< 'EOF'

#!/bin/bash

TIMESTAMP=$(date +%Y%m%d-%H%M%S)

tar -czf /backups/hive-$&#123;TIMESTAMP&#125;.tar.gz /opt/hive-r/data

aws s3 cp /backups/hive-$&#123;TIMESTAMP&#125;.tar.gz s3://your-backup-bucket/

find /backups -mtime +7 -delete

EOF

chmod +x /etc/cron.daily/backup-hive

  

echo "✅ Server hardened"

  

**Test backups**:

  

  

# Trigger backup manually

sudo /etc/cron.daily/backup-hive

  

# Verify backup exists

ls -lh /backups/

  

# Test restore (on staging server)

# scp backup, extract, verify data intact

  

**✅ Validation**:

☐ Firewall active (only necessary ports open)

☐ SSH hardened (test login with key)

☐ Fail2ban active (check sudo fail2ban-client status)

☐ Backups running (check cron logs)

☐ Restore tested successfully

**📊 Log Cost**: ~$0.08 (25K tokens)

**📋 Step 8.4: Create Production Runbook**

**Your Instruction** (in Cursor):

  

  

Consult the SRE and Tech Writer agents.

  

CONTEXT:

When production breaks at 3am, you need clear instructions.

  

TASK:

Create comprehensive production runbook:

SECTIONS:

  1. Architecture Overview

     - Diagram of production setup

     - What each component does

     - How to access each service

  2. Common Operations

     - Deploy new version

     - Rollback to previous version

     - Restart service

     - Scale up/down

     - View logs

     - Access database

  3. Troubleshooting

     - Service is down → Check health endpoint, restart if needed

     - High latency → Check Grafana, identify bottleneck

     - High costs → Check cost dashboard, identify expensive queries

     - Database is locked → Kill long-running queries

     - Disk is full → Clean old logs, expand volume

  4. Incident Response

     - Severity levels (Critical, High, Medium, Low)

     - Response procedures

     - Communication templates

     - Post-mortem template

  5. Regular Maintenance

     - Daily: Check health dashboard

     - Weekly: Review costs, update dependencies

     - Monthly: Review logs, backup test, security audit

     - Quarterly: Disaster recovery drill

  

DELIVERABLES:

- docs/operations/production-runbook.md

- Quick reference card (one-page PDF)

- Incident response template

  

**⏸️ YOUR APPROVAL GATE**:

Review the runbook:

☐ Covers all likely scenarios

☐ Commands are copy-pasteable

☐ Has contact information (who to escalate to)

☐ Clear and actionable

**Example runbook sections**:

  

  

# Production Runbook

  

## Emergency Contacts

- On-call: [Your phone]

- Backup: [Team member]

- Hosting: [Provider support]

  

## Quick Commands

  

### View Logs

```bash

# Application logs

docker logs -f hive-app --tail 100

  

# System logs

sudo journalctl -u docker -f

  

**Restart Service**

  

  

cd /opt/hive-r

docker-compose -f docker-compose.prod.yml restart app

  

**Rollback**

  

  

# Rollback to previous version

docker-compose -f docker-compose.prod.yml pull app:previous

docker-compose -f docker-compose.prod.yml up -d app

  

**Incident Response**

**Severity: Critical (Service Down)**

1. Check health endpoint: curl https://api.yourdomain.com/health/ready

2. If timeout, check server: ssh production-server

3. Check Docker: docker ps (all containers running?)

4. Restart if needed: docker-compose restart

5. Check logs for errors: docker logs hive-app

6. Post in #incidents Slack channel

7. If not resolved in 15 min, call backup engineer

**Severity: High (Performance Degraded)**

1. Open Grafana dashboard

2. Check latency: Is p95 >2s?

3. Check cost: Is burn rate >$50/hour?

4. If cost spike: Enable rate limiting stricter

5. If latency spike: Check for slow queries in database

6. Document findings in incident log

  

  

  

**✅ Validation**:

- [ ] Runbook exists and is comprehensive

- [ ] Team has reviewed and understands it

- [ ] Commands tested (actually work)

- [ ] Contact information up to date

  

**📊 Log Cost**: ~$0.10 (30K tokens)

  

---

  

### 🎯 Step 8.5: Go-Live Checklist & Launch

  

**Your Instruction** (in Cursor):

  

  

Consult the SRE and Data Analyst agents.

CONTEXT:

We're ready to launch. Create final pre-launch checklist.

TASK:

Create comprehensive go-live checklist:

PRE-LAUNCH (1 week before):

☐ All tests passing in CI

☐ Security audit complete (no critical issues)

☐ Performance tested (can handle 10x current traffic)

☐ Backups automated and tested

☐ Monitoring dashboards ready

☐ Runbook documented

☐ DNS configured (propagation takes 24-48h)

☐ SSL certificates valid (Let's Encrypt or purchased)

☐ Team trained on runbook

☐ Incident response plan tested (tabletop exercise)

LAUNCH DAY:

☐ Final staging test (full user journey)

☐ Database migrated to production

☐ Environment variables set correctly

☐ Deploy production version

☐ Health checks passing

☐ Smoke tests passing

☐ Monitoring active and alerting

☐ Announce internally (team Slack)

☐ Monitor closely for first 4 hours

POST-LAUNCH (first week):

☐ Daily: Review metrics, logs, costs

☐ Day 1: User feedback survey

☐ Day 3: Performance review

☐ Day 7: Cost review, optimize if needed

☐ Week 1 retrospective

DELIVERABLES:

• Go-live checklist: docs/operations/go-live-checklist.md

• Launch announcement template

• Day 1/Week 1 review templates

  

  

  

**⏸️ YOUR APPROVAL GATE**:

  

Review the checklist:

- [ ] Nothing critical missing

- [ ] Realistic timeline

- [ ] Clear ownership (who does each item)

- [ ] Rollback plan ready

  

**Execute go-live**:

```bash

# On launch day

cd /opt/hive-r

  

# 1. Final backup before launch

./deploy/scripts/backup-now.sh

  

# 2. Deploy production version

git pull origin main

docker-compose -f docker-compose.prod.yml pull

docker-compose -f docker-compose.prod.yml up -d

  

# 3. Run smoke tests

./scripts/smoke-test.sh

  

# 4. Monitor

watch -n 5 'curl -s https://api.yourdomain.com/health/ready | jq'

  

# 5. Check Grafana dashboard

# Open in browser: https://grafana.yourdomain.com

  

# 6. Announce

./scripts/announce-launch.sh  # Posts to Slack

  

**First 24 Hours Monitoring**:

☐ Hour 1: Health checks green

☐ Hour 4: No incidents

☐ Hour 12: Costs tracking as expected

☐ Hour 24: User feedback collected

**✅ Validation**:

☐ All checklist items complete

☐ System running smoothly

☐ No major incidents

☐ Team confident in operations

**📊 Log Cost**: ~$0.05 (15K tokens)

**🎉 Step 8.6: Post-Launch Optimization & Retrospective**

**Your Instruction** (in Cursor):

  

  

Consult the Data Analyst and Tech Writer agents.

  

CONTEXT:

You've been in production for 1 week. Time to review and optimize.

  

TASK:

Conduct Week 1 retrospective:

DATA ANALYSIS:

  - Actual vs predicted costs

  - Actual vs predicted traffic

  - Error rate trends

  - Performance trends (latency, throughput)

  - User behavior patterns

  - Cache hit rates

  - Model routing effectiveness

  

FINDINGS:

  - What worked well?

  - What didn't work as expected?

  - What surprised us?

  - Where are the bottlenecks?

  - Where can we optimize costs?

  

OPTIMIZATION PLAN:

  - Quick wins (can do in next week)

  - Medium-term improvements (next month)

  - Long-term strategic changes (next quarter)

  

DELIVERABLES:

- Week 1 report: docs/retrospectives/week-1-production.md

- Optimization backlog (GitHub issues)

- Updated metrics dashboards

- Team learnings document

  

**Gather Data**:

  

  

# Export week 1 metrics

curl https://grafana.yourdomain.com/api/datasources/proxy/1/api/v1/query_range \

  -G --data-urlencode 'query=rate(hive_http_requests_total[1h])' \

  --data-urlencode 'start=2026-02-01T00:00:00Z' \

  --data-urlencode 'end=2026-02-08T00:00:00Z' \

  > week1-metrics.json

  

# Analyze costs

sqlite3 data/hive.db &lt;< 'EOF'

SELECT 

  date(timestamp) as day,

  SUM(cost_usd) as daily_cost,

  COUNT(*) as requests,

  AVG(cost_usd) as avg_cost_per_request

FROM token_usage

WHERE timestamp >= datetime('now', '-7 days')

GROUP BY date(timestamp);

EOF

  

**Review with Team**:

  

  

# Week 1 Production Retrospective

  

## Metrics Summary

- **Total Requests**: 12,450

- **Total Cost**: $87.32 (under budget of $100/week)

- **Average Latency**: 234ms (p95: 890ms)

- **Error Rate**: 0.3% (target: &lt;1%)

- **Uptime**: 99.8% (downtime: 14 minutes on Day 3)

  

## What Went Well ✅

- Launch was smooth, no major incidents

- Cost tracking helped identify expensive queries

- Cache hit rate of 42% saved $50+ in API costs

- Team responded quickly to Day 3 incident

  

## What Didn't Go Well ❌

- Day 3 outage (database lock, 14 min downtime)

- Higher than expected token usage from Router

- Frontend bundle size larger than planned (350KB)

- One customer complaint about slow response

  

## Surprises 🔍

- Model routing saved more than expected (45% vs predicted 30%)

- Most traffic comes from 3 power users

- Semantic caching works better than keyword caching

  

## Optimization Plan

**Quick Wins** (this week):

- Optimize Router prompt (reduce tokens by 30%)

- Implement lazy loading for frontend charts

- Add database connection pooling

  

**Medium-Term** (this month):

- Migrate to PostgreSQL (resolve locking issues)

- Fine-tune cache TTLs based on actual data

- Add more aggressive rate limiting for outlier users

  

**Long-Term** (this quarter):

- Explore fine-tuned models (cheaper Router)

- Add read replicas for scaling

- Implement multi-region deployment

  

**✅ Validation**:

☐ Retrospective completed

☐ Optimization backlog created

☐ Team aligned on priorities

☐ Monitoring refined based on learnings

**📊 Log Cost**: ~$0.08 (25K tokens)

**✅ Phase 8 Complete When:**

☐ Production environment running

☐ HTTPS working with valid certificate

☐ Monitoring and alerting active

☐ Security hardened

☐ Runbook documented

☐ Go-live checklist 100% complete

☐ Week 1 retrospective done

☐ System stable with &lt;1% error rate

☐ Total cost &lt;$15 for Phase 8

**💡 Key Insight**: Production is not a destination, it's the beginning of continuous improvement.

**🎊 CONGRATULATIONS! YOU'VE COMPLETED ALL 8 PHASES!**

**🏆 What You've Achieved**

**You now have**:

• ✅ Production-ready HIVE-R system with 70%+ test coverage

• ✅ Comprehensive cost monitoring and budget controls

• ✅ Full observability (logs, metrics, traces)

• ✅ Automated CI/CD pipeline

• ✅ Security hardened against common attacks

• ✅ Performance optimized (60-70% cost reduction)

• ✅ Professional deployment infrastructure

• ✅ Complete operational documentation

**Total Investment**:

• **Time**: 6 weeks (48-72 hours actual work)

• **AI Cost**: ~$200 in API calls

• **Infrastructure**: ~$50-100/month

• **Result**: Enterprise-grade multi-agent system

**📊 FINAL VALIDATION CHECKLIST**

Use this to verify everything is working:

**Core Functionality**

☐ Can invoke HIVE-R agents via MCP in Cursor

☐ Can make requests via REST API

☐ Agents coordinate correctly (Router → Specialist)

☐ Responses are streamed in real-time (SSE)

**Reliability**

☐ System survives OpenAI API outage (circuit breaker + fallbacks)

☐ Health checks detect real issues

☐ Automatic recovery from transient failures

☐ Backup and restore procedures tested

**Security**

☐ No secrets in git repository

☐ Input validation prevents injection attacks

☐ Authentication enforced on protected routes

☐ Rate limiting prevents abuse

☐ Security scan shows 0 critical issues

**Performance**

☐ Cache hit rate >40%

☐ p95 latency &lt;1 second

☐ Cost per request &lt;$0.05

☐ Model routing working (cheap models for simple tasks)

**Observability**

☐ Structured logs in JSON format

☐ Prometheus metrics exposed

☐ Grafana dashboards showing live data

☐ Distributed tracing in Jaeger

☐ Alerts firing correctly

**Operations**

☐ CI runs tests on every PR

☐ CD deploys to production on git tag

☐ Monitoring active (UptimeRobot + Grafana)

☐ Runbook documented and team trained

☐ Incident response plan tested

**📚