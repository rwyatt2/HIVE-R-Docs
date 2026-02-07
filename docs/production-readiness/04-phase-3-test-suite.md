---
title: "Phase 3: Test Suite Generation"
sidebar_label: "Phase 3: Test Suite Generation"
---

**

**⏱️ Time**: 8-12 hours

**💰 Budget**: $20-30

**🎯 Goal**: 70% test coverage using HIVE-R to test itself

**🤖 AI Workers**: HIVE-R Tester + Builder agents

**🧪 Step 3.1: Generate Tests for Core Graph Logic**

**Your Instruction** (in Cursor, open src/graph.ts):

  

  

🎯 META-TASK: Use HIVE-R to test HIVE-R

  

Consult the HIVE-R swarm.

  

CONTEXT:

This is the core LangGraph orchestration logic. It's critical but has zero tests.

  

TASK FOR TESTER AGENT:

Analyze this file and generate comprehensive Vitest test suite covering:

  1. Graph construction (nodes added correctly)

  2. State updates (messages append correctly)

  3. Agent routing (router chooses correct agent)

  4. Error handling (what if agent throws error?)

  5. Edge cases (empty messages, invalid agent names)

  

TEST REQUIREMENTS:

- Mock all LLM calls (use vitest.mock())

- Test both success and failure paths

- Use fixtures for common test data

- Fast (no real API calls, &lt;1s per test)

- Clear test names (describe what's being tested)

  

TASK FOR BUILDER AGENT:

- Review Tester's proposed tests

- Implement any helper functions needed

- Ensure tests actually compile and run

  

OUTPUT:

- New file: tests/graph.test.ts

- Test fixtures: tests/fixtures/agent-responses.ts

- At least 10 test cases

- 100% coverage of graph.ts critical paths

  

**⏸️ YOUR APPROVAL GATE**:

Review the generated test file:

☐ Tests are meaningful (not just checking trivial things)

☐ Mocks look realistic (resembles actual LLM responses)

☐ Coverage includes error cases (not just happy path)

☐ Test names are descriptive: "should route to builder agent when user asks for code"

**Run tests**:

  

  

npm test tests/graph.test.ts

  

# Check coverage

npm run test:coverage -- tests/graph.test.ts

# Should show high coverage (80%+) for graph.ts

  

**If tests fail**: Don't debug manually! Ask AI:

  

  

Tests are failing with this error: [paste error]

  

Analyze the failure and fix the tests (not the source code unless there's a real bug).

  

**✅ Validation**:

☐ All tests pass

☐ Coverage >80% for graph.ts

☐ Tests run fast (&lt;5s total)

☐ No real API calls made

**📊 Log Cost**: ~$0.15 (45K tokens for test generation)

**🔨 Step 3.2: Generate Tests for Agent Implementations**

**Your Instruction** (in Cursor):

  

  

Consult the Tester agent.

  

CONTEXT:

We have 13 agent files in src/agents/. Each needs unit tests.

  

TASK:

For EACH agent file (router, builder, security, planner, etc.):

  1. Analyze the agent's prompt and tools

  2. Generate test cases for expected behaviors

  3. Mock the underlying LLM calls

  4. Test tool invocations (if agent uses tools)

  

PRIORITIZE THESE AGENTS FIRST:

  1. router.ts (most critical)

  2. builder.ts (most used)

  3. security.ts (high risk)

  4. tester.ts (meta: test the tester!)

  

BATCH APPROACH:

- Generate tests for all 4 priority agents in one go

- Use consistent test structure across all

- Share fixtures where possible

  

OUTPUT:

- tests/agents/router.test.ts

- tests/agents/builder.test.ts

- tests/agents/security.test.ts

- tests/agents/tester.test.ts

- Shared fixtures: tests/fixtures/llm-responses.ts

  

**⏸️ YOUR APPROVAL GATE**:

This is a big batch. Review one test file first:

☐ tests/agents/router.test.ts looks comprehensive

☐ Mocking strategy makes sense

☐ Tests cover the agent's key responsibilities

**Approve remaining agents**: "Router tests look good, generate the other 3 with same approach"

**Run tests**:

  

  

npm test tests/agents/

  

# Check which agents have good coverage

npm run test:coverage -- tests/agents/

# Target: 70%+ for each priority agent

  

**✅ Validation**:

☐ All 4 priority agents have test files

☐ Tests pass

☐ Coverage >70% for router, builder, security

☐ No real LLM calls (check test run time &lt;10s)

**📊 Log Cost**: ~$0.40 (120K tokens - this is the biggest batch)

**🛠️ Step 3.3: Generate Integration Tests**

**Your Instruction** (in Cursor):

  

  

Consult the Tester agent.

  

CONTEXT:

Unit tests are good, but we need integration tests for multi-agent workflows.

  

TASK:

Create integration tests for these user journeys:

  1. "Build a simple React app" (Founder → PM → Planner → Builder)

  2. "Review this code for security issues" (Security → Reviewer)

  3. "Deploy this to production" (SRE → Security → Deployment)

  

TEST REQUIREMENTS:

- Start from user message

- Trace through multiple agent handoffs

- Mock LLM responses for each agent in sequence

- Verify final output matches expectations

- Test that context flows between agents

  

TECHNICAL APPROACH:

- Use real graph.ts (not mocked)

- Mock only the LLM calls

- Fixtures for multi-turn conversations

- Longer tests (30s timeout acceptable)

  

OUTPUT:

- tests/integration/multi-agent-workflows.test.ts

- Detailed fixtures: tests/fixtures/workflows/

- At least 3 complete workflows tested

  

**⏸️ YOUR APPROVAL GATE**:

Integration tests are complex. Review the approach:

☐ Tests actually invoke multiple agents (not shortcuts)

☐ Fixtures represent realistic agent responses

☐ Tests verify handoff logic (agent A → agent B)

☐ Clear failure messages (which agent failed, why)

**Run tests**:

  

  

npm test tests/integration/

  

# These will be slower (that's OK)

# Should complete in &lt;30s total

  

**If tests are too slow** (>1min): Ask AI to add more aggressive mocking or parallelize tests.

**✅ Validation**:

☐ Integration tests pass

☐ Cover 3 key workflows

☐ Tests complete in &lt;30s

☐ Verify agent handoffs work correctly

**📊 Log Cost**: ~$0.20 (60K tokens)

**🔍 Step 3.4: Add Test Coverage Reporting**

**Your Instruction** (in Cursor, open package.json):

  

  

Consult the Builder agent.

  

TASK:

Add test coverage reporting to our npm scripts.

  

REQUIREMENTS:

- Use Vitest's built-in coverage (c8 or istanbul)

- Generate HTML report (for visual review)

- Generate JSON report (for CI integration)

- Add npm script: "test:coverage"

- Coverage thresholds:

  - Statements: 70%

  - Branches: 60%

  - Functions: 70%

  - Lines: 70%

  

OUTPUT:

- Updated package.json with coverage config

- Updated vitest.config.ts with thresholds

- Documentation: docs/development/testing.md

- CI integration (runs in GitHub Actions)

  

**⏸️ YOUR APPROVAL GATE**:

Review the thresholds:

☐ 70% is achievable (not too low or too high)

☐ Critical files (graph.ts, agents/*) excluded from low coverage

☐ HTML report will be gitignored (not committed)

**Run coverage**:

  

  

npm run test:coverage

  

# Opens coverage/index.html in browser

# Review which files have low coverage

  

**If coverage is &lt;70%**: Ask AI to generate more tests for specific files:

  

  

Coverage for src/lib/memory.ts is only 45%. Generate tests to bring it to 70%+.

  

**✅ Validation**:

☐ Overall coverage >70%

☐ HTML report generated

☐ Coverage badge shows in README (optional)

☐ CI configured to run coverage

**📊 Log Cost**: ~$0.05 (15K tokens)

**🤖 Step 3.5: Document Testing Strategy**

**Your Instruction** (in Cursor):

  

  

Consult the Tech Writer agent.

  

CONTEXT:

We now have a comprehensive test suite. Document how to use it.

  

TASK:

Create developer documentation covering:

  1. How to run tests locally

  2. How to run specific test files

  3. How to write new tests (with examples)

  4. Testing philosophy (unit vs integration)

  5. Mocking strategy (when to mock, when to use real code)

  6. Coverage requirements (70% for new code)

  7. CI integration (tests run on every PR)

  8. Troubleshooting common test issues

  

OUTPUT:

- docs/development/testing.md

- Code examples for common test patterns

- Link from main README.md

  

**⏸️ YOUR APPROVAL GATE**:

Read the documentation:

☐ Clear instructions for beginners

☐ Code examples are copy-pasteable

☐ Explains WHY we test (not just HOW)

☐ Covers troubleshooting

**✅ Validation**:

☐ Documentation exists and is thorough

☐ README links to it

☐ You understand how to add tests yourself (if needed)

**📊 Log Cost**: ~$0.03 (10K tokens)

**✅ Phase 3 Complete When:**

☐ Test coverage >70% overall

☐ All critical files (graph, agents) >80% coverage

☐ Integration tests for 3 key workflows

☐ Coverage reporting configured

☐ Testing documentation complete

☐ All tests passing on local machine

☐ Total cost &lt;$30 for Phase 3

**💡 Key Insight**: You can now refactor and add features confidently. Tests catch regressions.

**🎉 MAJOR MILESTONE**: Your codebase is now tested better than 90% of AI projects.

**