---
title: "Phase 7: CI/CD & Automation"
sidebar_label: "Phase 7: CI/CD & Automation"
---

**

**⏱️ Time**: 8-10 hours

**💰 Budget**: $15-20

**🎯 Goal**: Automated testing, deployment, and changelog generation

**🤖 AI Workers**: HIVE-R SRE + Builder + Tech Writer agents

**🔄 Step 7.1: Set Up GitHub Actions CI Pipeline**

**Your Instruction** (in Cursor):

  

  

Consult the SRE agent.

  

CONTEXT:

Currently, we run tests manually. We need automated CI on every push/PR.

  

TASK:

Create GitHub Actions workflows for:

  1. CI (test): Run on every push and PR

     - Install dependencies

     - Run linter (ESLint)

     - Run type check (TypeScript)

     - Run tests (Vitest)

     - Run security audit (npm audit)

     - Generate coverage report

     - Comment coverage on PR

  2. CI (build): Verify build succeeds

     - Build backend (npm run build)

     - Build frontend (cd client && npm run build)

     - Build Docker image

  3. CI (docker): Test Docker image

     - Build image

     - Run container

     - Run smoke tests against container

  

DELIVERABLES:

- .github/workflows/ci-test.yml

- .github/workflows/ci-build.yml

- .github/workflows/ci-docker.yml

- Badge in README.md (build status)

- Documentation: docs/development/ci-cd.md

  

**⏸️ YOUR APPROVAL GATE**:

Review the CI workflows:

☐ Runs on pull_request and push to main

☐ Uses caching (node_modules, npm cache)

☐ Fails if tests fail (blocks merge)

☐ Posts results to PR (coverage, lint errors)

**Create workflows**:

  

  

# .github/workflows/ci-test.yml

name: CI - Test

  

on:

  push:

    branches: [main]

  pull_request:

    branches: [main]

  

jobs:

  test:

    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v3

      - uses: actions/setup-node@v3

        with:

          node-version: '18'

          cache: 'npm'

      - name: Install dependencies

        run: npm ci

      - name: Run linter

        run: npm run lint

      - name: Run type check

        run: npm run type-check

      - name: Run tests

        run: npm test

      - name: Generate coverage

        run: npm run test:coverage

      - name: Upload coverage

        uses: codecov/codecov-action@v3

        with:

          files: ./coverage/coverage-final.json

  

**Test CI**:

  

  

# Create a test PR with intentional failure

git checkout -b test-ci

echo "const x: number = 'string';" >> src/test-fail.ts

git add . && git commit -m "test: trigger CI"

git push origin test-ci

  

# Open PR on GitHub

# CI should fail with type error

  

# Fix and verify CI passes

rm src/test-fail.ts

git commit -am "fix: remove test failure"

git push

  

# CI should pass, PR should show green checkmark

  

**✅ Validation**:

☐ CI runs on every PR

☐ Fails on test failures

☐ Fails on lint errors

☐ Fails on type errors

☐ Coverage report posted to PR

**📊 Log Cost**: ~$0.10 (30K tokens)

**🚀 Step 7.2: Set Up Automated Deployment (CD)**

**Your Instruction** (in Cursor):

  

  

Consult the SRE agent.

  

CONTEXT:

Manual deployments are error-prone and slow. Automate production deployments.

  

TASK:

Create deployment workflows:

  1. Deploy to staging (on push to main)

     - Build Docker image

     - Push to registry (Docker Hub, ECR, GCR)

     - Deploy to staging server (SSH or K8s)

     - Run smoke tests

     - Notify Slack

  2. Deploy to production (on git tag)

     - Build Docker image (with version tag)

     - Push to registry

     - Deploy with blue-green strategy

     - Run health checks

     - Rollback if health checks fail

     - Notify Slack with deployment link

  

DEPLOYMENT TARGET:

[Choose based on your infrastructure]

- Option A: Docker Swarm on VPS

- Option B: Kubernetes cluster

- Option C: Cloud Run / ECS Fargate

  

DELIVERABLES:

- .github/workflows/deploy-staging.yml

- .github/workflows/deploy-production.yml

- Deployment scripts: scripts/deploy.sh

- Rollback scripts: scripts/rollback.sh

- Documentation: docs/operations/deployment.md

  

**⏸️ YOUR APPROVAL GATE**:

Review deployment strategy:

☐ Staging deploys automatically (for testing)

☐ Production requires manual tag (protection)

☐ Has rollback mechanism

☐ Health checks verify deployment success

☐ Secrets managed securely (GitHub Secrets)

**Setup deployment** (Docker Swarm example):

  

  

# .github/workflows/deploy-production.yml

name: Deploy to Production

  

on:

  push:

    tags:

      - 'v*'  # Trigger on version tags (v1.0.0)

  

jobs:

  deploy:

    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v3

      - name: Build Docker image

        run: |

          docker build -t hive-r:$&#123;&#123; github.ref_name &#125;&#125; .

          docker tag hive-r:$&#123;&#123; github.ref_name &#125;&#125; hive-r:latest

      - name: Push to registry

        run: |

          echo $&#123;&#123; secrets.DOCKER_PASSWORD &#125;&#125; | docker login -u $&#123;&#123; secrets.DOCKER_USERNAME &#125;&#125; --password-stdin

          docker push hive-r:$&#123;&#123; github.ref_name &#125;&#125;

          docker push hive-r:latest

      - name: Deploy to production

        uses: appleboy/ssh-action@master

        with:

          host: $&#123;&#123; secrets.PROD_HOST &#125;&#125;

          username: $&#123;&#123; secrets.PROD_USER &#125;&#125;

          key: $&#123;&#123; secrets.PROD_SSH_KEY &#125;&#125;

          script: |

            docker pull hive-r:$&#123;&#123; github.ref_name &#125;&#125;

            docker service update --image hive-r:$&#123;&#123; github.ref_name &#125;&#125; hive_app

      - name: Wait for health check

        run: |

          sleep 30

          curl -f https://api.hive-r.com/health/ready || exit 1

      - name: Notify Slack

        uses: 8398a7/action-slack@v3

        with:

          status: $&#123;&#123; job.status &#125;&#125;

          text: 'Deployed $&#123;&#123; github.ref_name &#125;&#125; to production'

          webhook_url: $&#123;&#123; secrets.SLACK_WEBHOOK &#125;&#125;

  

**Test deployment**:

  

  

# Create a version tag

git tag v1.0.0

git push origin v1.0.0

  

# Watch GitHub Actions

# Should build, push, deploy, and verify

  

# Check production

curl https://api.hive-r.com/health/ready

# Should return healthy status

  

**✅ Validation**:

☐ Staging deploys on every push to main

☐ Production deploys on version tags

☐ Deployment succeeds without manual intervention

☐ Rollback tested and working

☐ Slack notifications work

**📊 Log Cost**: ~$0.12 (35K tokens)

**📝 Step 7.3: Automated Changelog Generation**

**Your Instruction** (in Cursor):

  

  

Consult the Tech Writer agent.

  

CONTEXT:

Tracking changes manually is tedious. Automate changelog from git commits.

  

TASK:

Implement automated changelog generation:

  - Use Conventional Commits format (feat:, fix:, docs:, etc)

  - Generate CHANGELOG.md on every release

  - Group by type (Features, Bug Fixes, Breaking Changes)

  - Link to GitHub issues/PRs

  - Include contributor credits

  

TOOLS:

- standard-version or semantic-release

- Commit message linting (commitlint)

- Pre-commit hook to enforce format

  

COMMIT FORMAT:

type(scope): subject

  

feat(auth): add JWT refresh token support

fix(cache): resolve Redis connection leak

docs(readme): update installation instructions

BREAKING CHANGE: API endpoint /v1/chat renamed to /v2/chat

  

DELIVERABLES:

- Commitlint configuration (.commitlintrc.json)

- Husky pre-commit hooks

- GitHub Action to generate changelog

- Updated CHANGELOG.md

- Documentation: docs/development/commit-conventions.md

  

**⏸️ YOUR APPROVAL GATE**:

Review commit conventions:

☐ Format is clear and easy to follow

☐ Covers common types (feat, fix, docs, chore)

☐ Breaking changes clearly marked

☐ Examples provided for team

**Setup changelog automation**:

  

  

# Install tools

npm install -D @commitlint/cli @commitlint/config-conventional husky standard-version

  

# Configure commitlint

cat > .commitlintrc.json &lt;< 'EOF'

&#123;

  "extends": ["@commitlint/config-conventional"]

&#125;

EOF

  

# Setup husky hooks

npx husky install

npx husky add .husky/commit-msg 'npx --no -- commitlint --edit "$1"'

  

# Add version script to package.json

npm pkg set scripts.release="standard-version"

  

**Test changelog**:

  

  

# Make commits with conventional format

git commit -m "feat(cache): implement semantic caching"

git commit -m "fix(auth): resolve token expiration bug"

  

# Generate changelog

npm run release

  

# Check CHANGELOG.md

cat CHANGELOG.md

# Should show grouped commits with links

  

**✅ Validation**:

☐ Commit messages validated (can't commit without format)

☐ Changelog auto-generated

☐ Changelog is well-formatted and readable

☐ Version bumped automatically (semantic versioning)

**📊 Log Cost**: ~$0.06 (20K tokens)

**🔐 Step 7.4: Automated Security Scanning**

**Your Instruction** (in Cursor):

  

  

Consult the Security agent.

  

CONTEXT:

New vulnerabilities are discovered daily. Automate security scanning.

  

TASK:

Add security scanning to CI/CD:

  1. Dependency scanning (npm audit, Snyk)

     - Scan on every PR

     - Fail if critical vulnerabilities

     - Auto-create PRs to update vulnerable deps

  2. Code scanning (CodeQL, Semgrep)

     - Scan for common vulnerabilities (SQL injection, XSS, etc)

     - Scan for secrets accidentally committed

     - Fail if high-severity issues

  3. Container scanning (Trivy, Grype)

     - Scan Docker images for OS vulnerabilities

     - Fail if critical CVEs

  4. License compliance

     - Ensure all dependencies have compatible licenses

     - No GPL in commercial project (if applicable)

  

DELIVERABLES:

- .github/workflows/security.yml

- Snyk integration (or GitHub Dependabot)

- CodeQL configuration

- Container scanning in CI

- Security policy: SECURITY.md

  

**⏸️ YOUR APPROVAL GATE**:

Review security scanning config:

☐ Scans run on every PR (not just main)

☐ Doesn't block on low-severity issues

☐ Has process for triaging vulnerabilities

☐ Auto-updates non-breaking security fixes

**Setup security scanning**:

  

  

# .github/workflows/security.yml

name: Security Scanning

  

on:

  pull_request:

  schedule:

    - cron: '0 0 * * 0'  # Weekly

  

jobs:

  dependencies:

    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v3

      - name: Run npm audit

        run: npm audit --audit-level=high

  code-scanning:

    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v3

      - uses: github/codeql-action/init@v2

        with:

          languages: javascript, typescript

      - uses: github/codeql-action/analyze@v2

  container-scanning:

    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v3

      - name: Build image

        run: docker build -t hive-r:test .

      - name: Scan with Trivy

        uses: aquasecurity/trivy-action@master

        with:

          image-ref: hive-r:test

          severity: 'CRITICAL,HIGH'

  

**Test security scanning**:

  

  

# Trigger manually

git push origin your-branch

  

# Check GitHub Actions -> Security tab

# Should see scanning results

  

# Intentionally add a vulnerable dependency

npm install express@3.0.0  # Old version with CVEs

  

# Push and verify CI fails with vulnerability report

  

**✅ Validation**:

☐ Security scans run automatically

☐ Vulnerabilities detected correctly

☐ CI blocks critical vulnerabilities

☐ Dependabot/Snyk creates auto-update PRs

**📊 Log Cost**: ~$0.08 (25K tokens)

**📊 Step 7.5: Automated Performance Testing**

**Your Instruction** (in Cursor):

  

  

Consult the SRE and Data Analyst agents.

  

CONTEXT:

Performance regressions slip in silently. Catch them early with automated tests.

  

TASK:

Add performance testing to CI:

  1. Load testing (Artillery, K6)

     - Run on every release candidate

     - Test: 100 concurrent users, 5 min duration

     - Assert: p95 latency &lt;1s, error rate &lt;1%

     - Compare against baseline (detect regressions)

  2. Frontend performance (Lighthouse CI)

     - Run on every PR that touches frontend

     - Assert: Performance score >90

     - Track metrics over time

  3. Database performance

     - Benchmark critical queries

     - Assert: &lt;50ms for reads, &lt;100ms for writes

     - Detect N+1 queries

  

DELIVERABLES:

- .github/workflows/performance.yml

- Load test scripts: tests/performance/load-test.yml

- Performance budgets in CI

- Grafana dashboard for performance trends

- Alert if regression detected

  

**⏸️ YOUR APPROVAL GATE**:

Review performance thresholds:

☐ Realistic (based on current performance)

☐ Not too strict (allows for normal variance)

☐ Catches real regressions (10% slower = fail)

**Setup performance testing**:

  

  

# tests/performance/load-test.yml (Artillery)

config:

  target: "http://localhost:3000"

  phases:

    - duration: 60

      arrivalRate: 10

      name: "Warm up"

    - duration: 300

      arrivalRate: 20

      name: "Sustained load"

scenarios:

  - name: "Chat flow"

    flow:

      - post:

          url: "/chat"

          json:

            messages: [&#123;"role": "user", "content": "test"&#125;]

          expect:

            - statusCode: 200

            - contentType: json

            - hasProperty: response

  

# Add to CI

# .github/workflows/performance.yml

- name: Run load test

  run: |

    npm start &

    sleep 10

    npx artillery run tests/performance/load-test.yml --output report.json

    npx artillery report report.json

    # Check assertions

    p95=$(jq '.aggregate.latency.p95' report.json)

    if [ $p95 -gt 1000 ]; then

      echo "Performance regression: p95 latency $&#123;p95&#125;ms > 1000ms"

      exit 1

    fi

  

**Test performance CI**:

  

  

# Run load test locally first

npm start &

npx artillery run tests/performance/load-test.yml

  

# Check report

# Should pass with p95 &lt;1s

  

# Introduce performance regression (sleep in code)

# Push and verify CI fails

  

**✅ Validation**:

☐ Performance tests run in CI

☐ Regressions detected

☐ Frontend performance tracked

☐ Database benchmarks pass

**📊 Log Cost**: ~$0.10 (30K tokens)

**✅ Phase 7 Complete When:**

☐ CI runs tests on every PR

☐ CD deploys to staging/production automatically

☐ Changelog auto-generated

☐ Security scanning active

☐ Performance testing in CI

☐ All pipelines passing

☐ Total cost &lt;$20 for Phase 7

**💡 Key Insight**: Automation = Confidence. Ship faster with fewer bugs.

**🎉 MAJOR MILESTONE**: Your development workflow is now professional-grade.

**