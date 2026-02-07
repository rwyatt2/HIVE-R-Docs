---
title: "Phase 0: Pre-Flight Checklist"
sidebar_label: "Phase 0: Pre-Flight Checklist"
---

**

**⏱️ Time**: 30 minutes

**💰 Cost**: $0

**🎯 Goal**: Prepare environment for AI-driven development

### Step 0.1: Verify HIVE-R is Running

  

  

# In your HIVE-R directory

cd ~/Desktop/HIVE-R  # or wherever your HIVE-R is located

  

# Start the server

npm run dev

  

Open browser to http://localhost:3000 - confirm you see the HIVE-R interface.

### Step 0.2: Test MCP Connection in Cursor

**In Cursor**:

1. Open Settings (Cmd+,)

2. Navigate to "Features" → "Model Context Protocol"

3. Click "Edit Config" → Should open ~/.cursor/mcp_config.json

**Verify your config looks like this**:

  

  

&#123;

  "mcpServers": &#123;

    "hive": &#123;

      "command": "node",

      "args": ["/ABSOLUTE/PATH/TO/HIVE-R/dist/mcp-server.js"],

      "env": &#123;

        "OPENAI_API_KEY": "your-key-here",

        "ANTHROPIC_API_KEY": "your-key-here"

      &#125;

    &#125;

  &#125;

&#125;

  

**Test the connection**:

• Open Cursor AI chat (Cmd+L)

• Type: "Use the consult_hive_swarm tool and ask the system status"

• **✅ Pass**: You get a response from HIVE-R agents

• **❌ Fail**: See [Troubleshooting: MCP Connection](10-appendix.md#troubleshooting-mcp-connection)

### Step 0.3: Create Implementation Workspace

  

  

# Create a working directory for this project

mkdir -p ~/HIVE-R-Production-Implementation

cd ~/HIVE-R-Production-Implementation

  

# Create tracking file

cat > implementation-log.md &lt;< 'EOF'

# HIVE-R Production Implementation Log

  

## Week 1

- [ ] Phase 0: Pre-flight complete

- [ ] Phase 1: Security fixes

- [ ] Phase 2: Cost monitoring

- [ ] Phase 3: Test suite

  

## Week 2

- [ ] Phase 4: Reliability

  

## Week 3

- [ ] Phase 5: Observability

  

## Week 4

- [ ] Phase 6: Performance

  

## Week 5

- [ ] Phase 7: CI/CD

  

## Week 6

- [ ] Phase 8: Deployment

  

---

  

## Session Notes

[Add your notes after each session]

EOF

  

### Step 0.4: Set Up Budget Tracking

Create a simple budget tracker:

  

  

cat > budget-tracker.md &lt;< 'EOF'

# AI API Cost Tracker

  

| Date | Phase | Task | Tokens | Cost | Running Total |

|:-----|:------|:-----|:-------|:-----|:--------------|

| | | | | | $0.00 |

  

**Budget**: $200 total

**Alerts**:

- 🟢 &lt;$50 spent

- 🟡 $50-150 spent  

- 🔴 >$150 spent

  

**Cost-Saving Tips**:

- Use Claude Sonnet for complex tasks ($3/MTok)

- Use GPT-4o-mini for simple tasks ($0.15/MTok)

- Use DeepSeek R1 for planning ($0.55/MTok)

EOF

  

**✅ Phase 0 Complete When**:

☐ HIVE-R server running

☐ MCP connection working in Cursor

☐ Workspace created

☐ Budget tracker ready

**