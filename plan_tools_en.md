# doriku-tools Plan

> Analysis and prioritization of standalone tool candidates for the doriku service
> Date: 2026-02-15

---

## 1. Background & Purpose

### 1.1 doriku Platform Overview

doriku is a **project synchronization platform across diverse AI agents, servers, people, and organizations**.
It connects multiple AI coding tools (Claude Code, Cursor, Windsurf, etc.) and human team members into a single real-time workspace to streamline collaborative development.

### 1.2 Supported Protocols

| Layer | Protocol | Endpoint |
|-------|----------|----------|
| REST API | HTTPS | `https://api.doriku.io/api/v1` |
| WebSocket | WSS | `wss://api.doriku.io/ws` |
| MCP | Streamable HTTP | `https://api.doriku.io/mcp` |
| Webhooks | HTTPS (HMAC-SHA256) | User-configured |

### 1.3 Current Capabilities

- Task CRUD, decomposition, and assignment
- Agent registration, heartbeat, online/offline tracking
- Channel messaging (send/receive)
- Claude Code Hook auto-sync (`sync-doriku.sh`)
- WebSocket real-time event propagation (~200ms)

### 1.4 Purpose of This Document

Based on doriku's core differentiator (cross-agent, cross-organization synchronization), this document derives **tool candidates to be developed and managed in the doriku-tools repository**, informed by market gap analysis and technology trends, and proposes a prioritized roadmap.

---

## 2. Market Analysis

### 2.1 Key Pain Points in Multi-AI Agent Collaboration

| # | Pain Point | Details | Source |
|---|-----------|---------|--------|
| P1 | **Git Conflicts & File Contention** | Parallel agents editing the same files guarantee integration problems. Git Worktrees are the de facto workaround but remain a manual, infrastructure-level pattern | Clash, CCManager |
| P2 | **Context Fragmentation & Memory Loss** | Zero awareness across sessions of project history, other agents' decisions, or past architectural choices | RedMonk 2025 |
| P3 | **Productivity Paradox** | METR study: experienced developers actually took **19% longer** with AI tools (while perceiving a 20% speedup). Management, review, and correction overhead negates raw speed gains | METR 2025 |
| P4 | **Declining Trust & Review Burden** | 52% of developers report agents affect their work, 69% report productivity gains — but **trust in AI output is simultaneously declining**. Multi-agent scenarios multiply review burden | Stack Overflow 2025 |
| P5 | **Tool Fragmentation** | Claude Code, Cursor, Windsurf, and Copilot use different context models, file handling, and prompting architectures. No cross-tool awareness mechanism exists | Industry-wide |

### 2.2 Existing Platform Comparison

| Platform | Approach | Strengths | Limitations |
|----------|----------|-----------|-------------|
| **Devin** | Fully autonomous single agent | Sandboxed environment, Slack communication, 3x SWE-bench performance | Single agent only. No parallel collaboration |
| **Factory AI** | Agent-Native Development | LLM/interface agnostic, organizational context ingestion, CI/CD integration. 200% QoQ growth | Closed system. Limited cross-tool coordination |
| **OpenHands** | Open-source coding agent SDK | 67K GitHub stars, 72% SWE-Bench resolution rate, self-hosting | Focused on platform building over orchestration |
| **Agent-MCP** | MCP-based multi-agent coordination | Knowledge graph, task management, conflict resolution | Early stage, limited production validation |
| **AgentBase** | Multi-tool visual orchestrator | Visual canvas, edit isolation toggle, progress tracking | **Local only**. No cross-server/organization support |
| **Claude Code Agent Teams** | Native multi-agent | Lead agent → dependency graph → background agent spawning | Limited to Claude Code |
| **doriku** | Cross-tool/organization sync platform | MCP+API+WS 3-channel, cross-server, cross-organization | Tool ecosystem expansion needed (subject of this document) |

### 2.3 Market Gap Analysis

| Gap | Description | Current Solutions | doriku Opportunity |
|-----|-------------|-------------------|-------------------|
| **Cross-Agent Awareness Layer** | Agent A has no knowledge of Agent B's work | None | **Solvable via WS-based real-time broadcast** |
| **Proactive Conflict Prevention** | Only post-hoc conflict resolution exists | Clash (Git-level only) | **Task → file mapping + AST analysis for prediction** |
| **Unified Review Pipeline** | No unified view of multi-agent PRs | None | **PR aggregation + dependency + quality metric integration** |
| **Persistent Project Memory** | CLAUDE.md, .cursorrules, etc. scattered per tool | Per-tool individual files | **Central context store with unified sync** |
| **Cross-Organization Governance** | Agent action auditing, human approval tracking | None | **Immutable audit log + policy guard** |
| **Cost/Efficiency Analytics** | Per-agent token consumption and API cost tracking | None | **Central metric collection + dashboard** |

### 2.4 MCP Ecosystem Status & Gaps

**Ecosystem Scale**: 17,647+ servers registered on MCP.so

**Key Security Issues** (analysis of 2,614 MCP implementations):
- 82% — Use file system operations vulnerable to Path Traversal
- 67% — Use sensitive APIs related to Code Injection
- 53% — Rely on long-lived static secrets (API keys, PATs)
- Only 8.5% use OAuth authentication

**Protocol-Level Gaps**:
- No multi-tenancy support
- No admin controls (MCP server access governance)
- No context-aware dynamic discovery
- Limited real-time push notification patterns
- No standardized non-human client identity
- No compliance test suites

### 2.5 Key Trends (Anthropic 2026 Agentic Coding Report)

- **57%** of organizations deploying multi-step agent workflows
- **16%** have progressed to cross-functional or end-to-end processes
- Gartner: **1,445% surge** in multi-agent system inquiries
- Four strategic priorities:
  1. Master multi-agent coordination
  2. Scale human-agent oversight
  3. Extend agentic coding beyond engineering teams
  4. Embed security architecture as a core design principle

---

## 3. Tool Candidates — Detailed Specifications

### Category A: Coordination (Real-Time Conflict Prevention)

Target Pain Points: P1 (Git Conflicts), P5 (Tool Fragmentation)

#### A-1. file-lock

| Field | Description |
|-------|-------------|
| **Purpose** | Preemptive lock acquisition/release when agents edit files. Warns on concurrent access |
| **Protocols** | MCP (lock request/release) + WS (real-time lock state broadcast) |
| **MCP Tools** | `acquire_file_lock(file_path, agent_id, ttl)` → Acquire lock |
| | `release_file_lock(file_path, agent_id)` → Release lock |
| | `list_file_locks(workspace_id)` → List current locks |
| **WS Events** | `file.locked` / `file.unlocked` / `file.lock_conflict` |
| **Core Logic** | TTL-based auto-release (for abnormal agent termination), priority queue (human > agent), directory-level lock support |
| **Differentiator** | Existing solutions are local-only. doriku provides cross-server/organization file locking |

#### A-2. work-broadcast

| Field | Description |
|-------|-------------|
| **Purpose** | Real-time broadcast of agent's current task, files, and branch |
| **Protocols** | WS (primary) + API (history queries) |
| **WS Events** | `agent.work_started` / `agent.work_updated` / `agent.work_stopped` |
| **API Endpoints** | `GET /api/v1/agents/{id}/current-work` → Specific agent's current work |
| | `GET /api/v1/workspaces/{id}/active-work` → All active work in workspace |
| **Data Model** | `{ agent_id, task_id, files: [path], branch, started_at, description }` |
| **Core Logic** | Integrated with heartbeat — auto work_stopped on heartbeat failure, automatic conflict warning based on file list overlap |
| **Differentiator** | Cross-tool work visibility across Claude Code, Cursor, Windsurf, and other heterogeneous agents |

#### A-3. conflict-predict

| Field | Description |
|-------|-------------|
| **Purpose** | Proactive analysis of potential file/module overlap between two agents' tasks |
| **Protocols** | API (analysis requests) + WS (real-time alerts) |
| **API Endpoints** | `POST /api/v1/predict-conflicts` → Task-list-based conflict prediction |
| | `GET /api/v1/conflict-map/{workspace_id}` → Current conflict risk map |
| **Analysis Levels** | Level 1: File path matching (fast), Level 2: Git diff-based change region analysis (moderate), Level 3: AST-level dependency graph analysis (precise) |
| **WS Events** | `conflict.warning` — Pushed to relevant agents when conflict risk detected |
| **Differentiator** | Clash analyzes Git-level only. doriku combines task metadata + file dependencies + AST analysis |

---

### Category B: Shared Memory (Project Context Synchronization)

Target Pain Points: P2 (Context Fragmentation), P5 (Tool Fragmentation)

#### B-1. context-sync

| Field | Description |
|-------|-------------|
| **Purpose** | Unified management and synchronization of per-agent config files (CLAUDE.md, .cursorrules, AGENTS.md, etc.) |
| **Protocols** | MCP (read/write) + API (management) |
| **MCP Tools** | `get_project_context(workspace_id)` → Return unified project context |
| | `update_project_context(workspace_id, section, content)` → Update context |
| | `get_context_for_agent(workspace_id, agent_type)` → Agent-type-specific tailored context |
| **Data Model** | Section-based management: `project_rules`, `code_conventions`, `architecture`, `stack`, `env_setup` |
| **Core Logic** | Per-agent-type transformation — auto-convert the same rules into CLAUDE.md format / .cursorrules format / AGENTS.md format |
| **Differentiator** | No cross-tool context unification solution exists in the market. doriku can be the first to provide this |

#### B-2. decision-log

| Field | Description |
|-------|-------------|
| **Purpose** | Central storage for ADRs (Architecture Decision Records). All agents can query decision history |
| **Protocols** | MCP (CRUD) |
| **MCP Tools** | `log_decision(workspace_id, title, context, decision, consequences)` |
| | `search_decisions(workspace_id, query)` → Keyword/tag-based search |
| | `get_decision(decision_id)` → Specific decision details |
| **Data Model** | `{ id, title, status(proposed/accepted/deprecated), context, decision, consequences, tags, created_by, created_at }` |
| **Core Logic** | Immutable log (new version on modification), tag-based automatic linking of related decisions, auto-suggest relevant ADRs when agent works on related code |
| **Differentiator** | Cross-session, cross-agent, cross-organization access to "why was this decided" |

#### B-3. project-summary

| Field | Description |
|-------|-------------|
| **Purpose** | Auto-index codebase structure, stack, and key patterns to provide during new agent onboarding |
| **Protocols** | API (create/update) + MCP (query) |
| **API Endpoints** | `POST /api/v1/projects/{id}/index` → Trigger indexing |
| | `GET /api/v1/projects/{id}/summary` → Query project summary |
| **MCP Tools** | `get_project_summary(workspace_id)` → Return structured project summary |
| | `get_module_summary(workspace_id, module_path)` → Specific module details |
| **Generated Items** | Directory tree, tech stack detection, key entry points, API endpoint listing, DB schema overview, test coverage, recent change history |
| **Differentiator** | Dramatically reduces the time for agents to "understand the project from scratch" at new session start |

---

### Category C: Review & Quality (Multi-Agent Quality Management)

Target Pain Points: P3 (Productivity Paradox), P4 (Review Burden)

#### C-1. pr-aggregator

| Field | Description |
|-------|-------------|
| **Purpose** | Unified view of PRs created by multiple agents. Display interdependencies and conflict potential |
| **Protocols** | API (aggregation) + WS (real-time updates) |
| **API Endpoints** | `GET /api/v1/workspaces/{id}/prs` → All PRs in workspace |
| | `GET /api/v1/workspaces/{id}/pr-dependencies` → Inter-PR dependency graph |
| | `GET /api/v1/workspaces/{id}/pr-conflicts` → Inter-PR conflict analysis |
| **WS Events** | `pr.created` / `pr.updated` / `pr.conflict_detected` / `pr.merged` |
| **Core Logic** | Auto-collect GitHub/GitLab PRs, cross-analyze changed files, recommend merge order, calculate review priority |
| **Differentiator** | No multi-agent PR unified review tool exists today |

#### C-2. code-review-request

| Field | Description |
|-------|-------------|
| **Purpose** | Agent A requests code review from Agent B or a human |
| **Protocols** | MCP (request creation) + WS (notifications) |
| **MCP Tools** | `request_review(workspace_id, pr_url, reviewer_type, description)` |
| | `list_review_requests(workspace_id, status)` |
| | `submit_review(review_id, result, comments)` |
| **Routing Logic** | `reviewer_type`: `auto` (system selects optimal reviewer), `agent` (specific agent), `human` (specific person), `team` (team channel) |
| **Differentiator** | Structured review workflow between agent-agent and agent-human |

#### C-3. test-coverage-sync

| Field | Description |
|-------|-------------|
| **Purpose** | Central collection of test results executed by each agent. Provide overall coverage map |
| **Protocols** | API (result upload/query) |
| **API Endpoints** | `POST /api/v1/workspaces/{id}/test-results` → Upload test results |
| | `GET /api/v1/workspaces/{id}/coverage-map` → Unified coverage map |
| | `GET /api/v1/workspaces/{id}/untested-changes` → Untested changes |
| **Core Logic** | Per-agent test execution history tracking, duplicate test detection, untested area highlighting, coverage trends |
| **Differentiator** | Test coverage unification in distributed agent environments is an untapped area |

---

### Category D: DevOps Bridge (External Service Integration)

Target: Accelerating enterprise adoption

#### D-1. ci-status

| Field | Description |
|-------|-------------|
| **Purpose** | Real-time propagation of pipeline status from GitHub Actions, GitLab CI, etc. to workspace |
| **Protocols** | WS (real-time) + API (history) |
| **Supported CI** | GitHub Actions, GitLab CI, CircleCI, Jenkins (webhook ingestion) |
| **WS Events** | `ci.run_started` / `ci.run_completed` / `ci.run_failed` |
| **Core Logic** | Auto-notify related tasks/agents on CI failure, provide failure log summary, enable agent auto-fix trigger |

#### D-2. issue-bridge

| Field | Description |
|-------|-------------|
| **Purpose** | Bidirectional synchronization with Jira, Linear, and GitHub Issues |
| **Protocols** | API (CRUD) + Webhook (external → doriku) |
| **Supported Services** | Jira, Linear, GitHub Issues, GitLab Issues |
| **Sync Direction** | Bidirectional: doriku task ↔ external issue. Auto-propagate status changes |
| **Mapping** | doriku `pending` → Jira `To Do`, doriku `running` → Jira `In Progress`, doriku `completed` → Jira `Done` |
| **Core Logic** | Auto-create issue on task creation, auto-link PRs, custom field mapping support |

#### D-3. deploy-coordinator

| Field | Description |
|-------|-------------|
| **Purpose** | Pre-deployment check of all agents' incomplete work, deploy lock, and rollback coordination |
| **Protocols** | MCP (lock/unlock) + WS (status broadcast) |
| **MCP Tools** | `request_deploy_lock(workspace_id, environment, reason)` |
| | `check_deploy_readiness(workspace_id, environment)` → Check incomplete work, unmerged PRs, failed CI |
| | `release_deploy_lock(workspace_id, environment)` |
| **Core Logic** | Pause notification to agents during deploy lock, automated checklist verification, notify related agents on rollback |

---

### Category E: Analytics & Governance (Audit & Cost Tracking)

Target: Enterprise governance requirements

#### E-1. audit-trail

| Field | Description |
|-------|-------------|
| **Purpose** | Immutable log of agent actions. Record which agent modified which file, when, and who approved it |
| **Protocols** | API (write/query) |
| **API Endpoints** | `POST /api/v1/audit-log` → Record audit event |
| | `GET /api/v1/audit-log?workspace_id=&agent_id=&from=&to=` → Filtered query |
| | `GET /api/v1/audit-log/export` → CSV/JSON export |
| **Event Types** | `file.modified`, `task.status_changed`, `deploy.requested`, `review.approved`, `policy.violated` |
| **Core Logic** | Append-only storage, tamper-proof hash chain, retention period policies, per-organization access control |

#### E-2. cost-tracker

| Field | Description |
|-------|-------------|
| **Purpose** | Collect per-agent token consumption, API call costs, and work efficiency metrics |
| **Protocols** | API (collection/query) + WS (real-time dashboard) |
| **Collected Metrics** | Per-agent token input/output volume, API call count, cost per task, tasks completed per hour, efficiency comparison by agent type |
| **API Endpoints** | `POST /api/v1/usage` → Report usage |
| | `GET /api/v1/usage/summary?workspace_id=&period=` → Period summary |
| | `GET /api/v1/usage/by-agent?workspace_id=` → Per-agent details |
| **Core Logic** | Budget limit setting → alert on threshold, anomaly detection, team/project allocation |

#### E-3. policy-guard

| Field | Description |
|-------|-------------|
| **Purpose** | Block/warn agent operations on policy violations (forbidden patterns, mandatory reviews, security rules) per organization |
| **Protocols** | MCP (policy query/validation) |
| **MCP Tools** | `check_policy(workspace_id, action, context)` → Validate policy compliance |
| | `list_policies(workspace_id)` → List active policies |
| | `get_policy_violations(workspace_id, period)` → Violation history |
| **Policy Types** | `file_restriction` (prohibit modification of specific files/directories), `review_required` (mandatory human review under certain conditions), `secret_detection` (block secret key commits), `branch_protection` (block direct push to protected branches), `cost_limit` (block on cost limit exceeded) |
| **Core Logic** | Hook integration — auto-validate policy before agent tool invocation, block on violation + record to audit-trail |

---

## 4. Prioritization & Roadmap

### 4.1 Prioritization Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| **Immediate Value** | 30% | Value users can feel immediately after installation |
| **Feasibility** | 25% | Ease of building on current doriku infrastructure (MCP/API/WS) |
| **Differentiation** | 25% | Unique value versus competing solutions |
| **Enterprise Pull** | 20% | Contribution to paid customer conversion |

### 4.2 Priority Matrix

| Rank | Tool | Immediate Value | Feasibility | Differentiation | Enterprise | **Score** |
|------|------|----------------|-------------|-----------------|------------|-----------|
| **1** | context-sync | 9 | 9 | 9 | 7 | **8.6** |
| **2** | file-lock | 9 | 8 | 8 | 7 | **8.1** |
| **3** | work-broadcast | 8 | 9 | 7 | 6 | **7.6** |
| **4** | audit-trail | 6 | 8 | 7 | 10 | **7.6** |
| **5** | issue-bridge | 7 | 7 | 6 | 9 | **7.2** |
| **6** | decision-log | 7 | 8 | 8 | 5 | **7.1** |
| **7** | policy-guard | 5 | 7 | 8 | 9 | **7.0** |
| **8** | conflict-predict | 8 | 5 | 9 | 6 | **7.0** |
| **9** | cost-tracker | 5 | 8 | 6 | 8 | **6.6** |
| **10** | pr-aggregator | 7 | 6 | 7 | 6 | **6.6** |
| **11** | project-summary | 7 | 6 | 6 | 5 | **6.2** |
| **12** | ci-status | 6 | 7 | 4 | 7 | **6.0** |
| **13** | deploy-coordinator | 5 | 6 | 6 | 7 | **5.9** |
| **14** | code-review-request | 6 | 6 | 6 | 5 | **5.8** |
| **15** | test-coverage-sync | 5 | 5 | 6 | 6 | **5.5** |

### 4.3 Proposed Roadmap

```
Phase 1 (MVP Core) ─────────────────────────────────────────
  context-sync     ██████████  ← Top priority. Build directly on existing MCP
  file-lock        ██████████  ← Leverage WS infra. Addresses biggest pain point
  work-broadcast   ████████    ← Natural bundle with file-lock

Phase 2 (Enterprise Foundation) ─────────────────────────────
  audit-trail      ████████    ← Enterprise governance foundation
  issue-bridge     ████████    ← Jira/Linear integration = adoption gateway
  decision-log     ███████     ← Complete project memory

Phase 3 (Intelligence) ──────────────────────────────────────
  policy-guard     ███████     ← Complete security + audit system
  conflict-predict ███████     ← AST analysis = technical moat
  cost-tracker     ██████      ← Cost visibility

Phase 4 (Ecosystem) ─────────────────────────────────────────
  pr-aggregator       ██████
  project-summary     ██████
  ci-status           ██████
  deploy-coordinator  █████
  code-review-request █████
  test-coverage-sync  █████
```

---

## 5. Technical Principles

### 5.1 Repository Structure

```
doriku-tools/
├── context-sync/        ← Each tool in an independent folder
│   ├── README.md
│   ├── src/
│   ├── tests/
│   └── package.json (or go.mod)
├── file-lock/
├── work-broadcast/
├── ...
├── TASK.md
├── HISTORY.md
├── HISTORY_KR.md
└── .gitignore
```

### 5.2 Common Principles

1. **Independent Deployment**: Each tool can be built and deployed independently
2. **Explicit Protocols**: Each tool clearly defines which protocols it uses (MCP/API/WS)
3. **Fail-Silent**: Never block agent operations on network failures (adhering to doriku's existing philosophy)
4. **Tests Required**: Each tool includes unit tests + integration tests
5. **Security First**: OAuth authentication, input validation, Path Traversal prevention (lessons from MCP security gaps)

---

## 6. References

- [Anthropic 2026 Agentic Coding Trends Report](https://resources.anthropic.com/hubfs/2026%20Agentic%20Coding%20Trends%20Report.pdf)
- [MCP Official Roadmap](https://modelcontextprotocol.io/development/roadmap)
- [MCP: What's Working, What's Broken](https://www.stackone.com/blog/mcp-where-its-been-where-its-going)
- [State of MCP Server Security 2025](https://astrix.security/learn/blog/state-of-mcp-server-security-2025/)
- [Agent-MCP Framework](https://github.com/rinadelph/Agent-MCP)
- [AgentBase Orchestrator](https://github.com/AgentOrchestrator/AgentBase)
- [Factory AI](https://factory.ai/)
- [Claude Code Agent Teams](https://code.claude.com/docs/en/agent-teams)
- [Clash - Merge Conflict Manager](https://github.com/clash-sh/clash)
- [RedMonk: 10 Things Developers Want from Agentic IDEs](https://redmonk.com/kholterhoff/2025/12/22/10-things-developers-want-from-their-agentic-ides-in-2025/)
- [METR: AI Impact on Developer Productivity](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/)
- [Stack Overflow 2025 Developer Survey - AI](https://survey.stackoverflow.co/2025/ai)
