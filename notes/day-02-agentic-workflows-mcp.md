# Day 2: Agentic Workflows & Model Context Protocol (MCP)

## 1. Overview
Building reliable autonomous agents requires moving past naive prompt-response loops toward structured architectural patterns. Today's notes focus on:
1. **Agent Architecture Archetypes** (ReAct, Plan-and-Execute, Evaluator-Optimizer).
2. **Model Context Protocol (MCP)** specification and client-server architecture.
3. **State Management & Context Compaction** for long-running workflows.

---

## 2. Core Agent Archetypes

| Pattern | Description | Best For | Trade-offs |
| :--- | :--- | :--- | :--- |
| **ReAct (Reason + Act)** | Interleaves reasoning traces with tool execution step-by-step. | Dynamic problem solving, search. | Higher token usage, prone to infinite loops without depth limits. |
| **Plan-and-Execute** | Planner creates a multi-step task list; executor agents resolve tasks sequentially or in parallel. | Complex multi-stage engineering tasks. | Rigid if intermediate steps reveal unexpected state changes. |
| **Evaluator-Optimizer** | One model generates solutions; an independent critic model evaluates and requests iterative refinements. | Code synthesis, quality-critical docs. | Multiplied latency and LLM inference cost. |

---

## 3. Model Context Protocol (MCP) Architecture
Anthropic's open Model Context Protocol standardizes how language models access external contexts, tools, and prompts across heterogeneous tools.

```mermaid
graph LR
    subgraph Host / Client
        Agent[AI Agent / LLM]
        MCPClient[MCP Client]
    end

    subgraph MCP Server Ecosystem
        MCPServer1[GitHub MCP Server]
        MCPServer2[Filesystem / DB Server]
        MCPServer3[Cloud Infrastructure MCP]
    end

    Agent <--> MCPClient
    MCPClient <-->|JSON-RPC 2.0 (stdio / SSE)| MCPServer1
    MCPClient <-->|JSON-RPC 2.0 (stdio / SSE)| MCPServer2
    MCPClient <-->|JSON-RPC 2.0 (stdio / SSE)| MCPServer3
```

### Core Primitives
1. **Tools**: Model-controlled executable functions with strict JSON schema definitions.
2. **Resources**: Application-controlled read-only data streams (e.g., file contents, API docs, telemetry).
3. **Prompts**: Pre-configured prompt templates with parameter substitution exposed to users/agents.

---

## 4. Engineering Guardrails for Autonomous Agents
- **Idempotency**: Ensure tool actions (e.g., branch creation, updates) can be retried safely.
- **Fail-safe Human-in-the-Loop**: High-consequence mutations (merges, deletions, deployments) must mandate explicit user confirmation.
- **Context Summarization / Memory Palaces**: Truncate raw tool outputs and inject structured semantic diffs to prevent context explosion.
