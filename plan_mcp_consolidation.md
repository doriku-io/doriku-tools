# MCP Tool Surface Consolidation Plan

> Redesign of the doriku MCP tool surface: 76 tools → 12 verb-based tools
> Date: 2026-07-06
> Status: PHASE 1 IMPLEMENTED (see §10) — rollout phases 2-3 pending

---

## 1. Problem Statement

Measured against the production endpoint (`https://api.doriku.io/mcp`):

| Metric | Current | Target |
|---|---|---|
| Tools advertised in `tools/list` | 76 | 12 |
| `tools/list` response size | 38.7 KB | ≤ 10 KB |
| Estimated context cost per session | ~10k tokens | ~2.5k tokens |
| Server instructions | ~350 words, mandatory-tracking directive | ~120 words, pull-oriented |

Why this matters:

1. **Context cost.** Clients without deferred tool loading (Cursor, Windsurf, most MCP hosts) inject all 76 schemas into every session. ~10k tokens of standing overhead directly contradicts the product pitch of making multi-tool AI work cheaper.
2. **Tool-choice degradation.** 18 task-family tools force the model to choose between near-duplicates (`task_create` vs `task_capture`, `task_log` vs `task_progress_report`). `doriku_capabilities` currently exists largely to tell the model which duplicate to prefer — evidence the surface itself is the problem.
3. **Dead control-plane tools.** `agent_interrupt` / `*_inject_task` publish to the WebSocket hub, but MCP agents (Claude Code etc.) never connect to WS, and `agent_heartbeat` returns only `{"status":"alive"}` with no command channel. These tools cannot reach their target from the MCP side.
4. **Unreliable capture path.** The "AUTOMATIC TASK TRACKING (MANDATORY)" instruction depends on probabilistic model compliance and duplicates what the deterministic Claude Code hook (`sync-doriku.sh`) already does, with no mapping between the two — double-tracking risk.

## 2. Design Principles

1. **One tool per domain, `action` verb parameter.** The pattern already proven in-house by `doriku_workflow` / `doriku_step`.
2. **Flat schemas.** Single `action` enum + a flat optional-property set. Per-action requirements are stated in one-line property descriptions and enforced server-side, reusing the existing error infrastructure (`classifyMCPToolError`, hints, `required_plan` inference). No `oneOf` — many MCP hosts render it poorly.
3. **Quality variants become the default, not siblings.** `task_capture` folds into `task.create` (structured fields optional, quality feedback returned when present). `progress_report` folds into `task.log`. `context_capture` into `context.set`. `decision_record` absorbs `decision_create`.
4. **Hooks own capture; MCP owns retrieval and explicit records.** The server instructions stop mandating tracking. MCP tools are for what the model actively needs: search, resume, handoff, decisions, memory, coordination.
5. **No tool advertised without a working delivery path.** Control-plane commands leave the MCP surface until heartbeat-based delivery exists (§6).

## 3. New Tool Surface (12 tools)

| # | Tool | Actions | Absorbs (count) |
|---|------|---------|-----------------|
| 1 | `doriku_task` | `create`, `get`, `list`, `search`, `update`, `delete`, `log`, `assign`, `claim`, `release`, `depend`, `decompose`, `tree`, `ready`, `resume`, `github_link`, `github_links` | 19 |
| 2 | `doriku_agent` | `register`, `heartbeat`, `list`, `status`, `activity`, `message`, `work_set`, `work_status` | 8 |
| 3 | `doriku_memory` | `set`, `get`, `list`, `search`, `delete`, `glossary`, `preference` | 7 |
| 4 | `doriku_context` | `set`, `get`, `list`, `delete` | 5 |
| 5 | `doriku_decision` | `record`, `list`, `update` | 4 |
| 6 | `doriku_project` | `create`, `list`, `summarize`, `summary` | 4 |
| 7 | `doriku_workspace` | `list`, `create`, `snapshot`, `changes` | 4 |
| 8 | `doriku_workflow` | existing verbs + `step_claim`, `step_get`, `step_complete`, `step_fail` | 2 |
| 9 | `doriku_automation` | `create`, `list`, `update`, `trigger`, `dispatcher_config`, `dispatcher_status` | 6 |
| 10 | `doriku_governance` | `lock`, `unlock`, `locks`, `lock_policy`, `cost_cap`, `audit_list`, `audit_export` | 7 |
| 11 | `doriku_search` | (single-purpose, unchanged) | 1 |
| 12 | `doriku_session` | `handoff`, `next_action`, `token_usage`, `capabilities` | 4 |
| — | **Removed from MCP** (console/REST only, see §6) | `agent_interrupt`, `agent_inject_task`, `session_interrupt`, `session_inject_task`, `ws_session_list` | 5 |

Coverage: 71 absorbed + 5 removed = **76 / 76**.

## 4. Legacy → New Mapping

| Legacy tool | New call |
|---|---|
| `doriku_task_create` | `doriku_task` `action:"create"` |
| `doriku_task_capture` | `doriku_task` `action:"create"` (+ `objective`, `done_criteria`) |
| `doriku_task_get` | `doriku_task` `action:"get"` |
| `doriku_task_list` | `doriku_task` `action:"list"` |
| `doriku_task_search` | `doriku_task` `action:"search"` |
| `doriku_task_update` | `doriku_task` `action:"update"` |
| `doriku_task_delete` | `doriku_task` `action:"delete"` |
| `doriku_task_log` | `doriku_task` `action:"log"` |
| `doriku_task_progress_report` | `doriku_task` `action:"log"` (+ `summary`, `next_step`, `blockers`) |
| `doriku_task_assign` | `doriku_task` `action:"assign"` |
| `doriku_task_claim` | `doriku_task` `action:"claim"` |
| `doriku_task_release` | `doriku_task` `action:"release"` |
| `doriku_task_depend` | `doriku_task` `action:"depend"` |
| `doriku_task_decompose` | `doriku_task` `action:"decompose"` |
| `doriku_task_tree` | `doriku_task` `action:"tree"` |
| `doriku_task_ready` | `doriku_task` `action:"ready"` |
| `doriku_task_resume_pack` | `doriku_task` `action:"resume"` |
| `doriku_github_link` | `doriku_task` `action:"github_link"` |
| `doriku_github_links` | `doriku_task` `action:"github_links"` |
| `doriku_agent_register` | `doriku_agent` `action:"register"` |
| `doriku_agent_heartbeat` | `doriku_agent` `action:"heartbeat"` (returns `pending_commands`, §6) |
| `doriku_agent_list` | `doriku_agent` `action:"list"` |
| `doriku_agent_status` | `doriku_agent` `action:"status"` |
| `doriku_agent_activity` | `doriku_agent` `action:"activity"` |
| `doriku_agent_message` | `doriku_agent` `action:"message"` |
| `doriku_work_broadcast` | `doriku_agent` `action:"work_set"` |
| `doriku_work_status` | `doriku_agent` `action:"work_status"` |
| `doriku_memory_set/get/list/search/delete` | `doriku_memory` same-name actions |
| `doriku_glossary` | `doriku_memory` `action:"glossary"` |
| `doriku_preference` | `doriku_memory` `action:"preference"` |
| `doriku_context_set` | `doriku_context` `action:"set"` |
| `doriku_context_capture` | `doriku_context` `action:"set"` (+ `source`, `scope`) |
| `doriku_context_get/list/delete` | `doriku_context` same-name actions |
| `doriku_decision_create` | `doriku_decision` `action:"record"` |
| `doriku_decision_record` | `doriku_decision` `action:"record"` |
| `doriku_decision_list/update` | `doriku_decision` same-name actions |
| `doriku_project_create/list` | `doriku_project` same-name actions |
| `doriku_project_summary` | `doriku_project` `action:"summarize"` |
| `doriku_project_summary_get` | `doriku_project` `action:"summary"` |
| `doriku_workspace_list/create/snapshot` | `doriku_workspace` same-name actions |
| `doriku_task_changes` | `doriku_workspace` `action:"changes"` (it is a workspace-level sync primitive) |
| `doriku_workflow` | unchanged |
| `doriku_step` | `doriku_workflow` `verb:"step_claim" \| "step_get" \| "step_complete" \| "step_fail"` |
| `doriku_automation_create/list/update/trigger` | `doriku_automation` same-name actions |
| `doriku_dispatcher_config/status` | `doriku_automation` `action:"dispatcher_config"` / `"dispatcher_status"` |
| `doriku_file_lock/unlock/locks` | `doriku_governance` `action:"lock"` / `"unlock"` / `"locks"` |
| `doriku_lock_policy` | `doriku_governance` `action:"lock_policy"` |
| `doriku_cost_cap` | `doriku_governance` `action:"cost_cap"` |
| `doriku_audit_list/export` | `doriku_governance` `action:"audit_list"` / `"audit_export"` |
| `doriku_search` | unchanged |
| `doriku_handoff_summary` | `doriku_session` `action:"handoff"` |
| `doriku_next_action` | `doriku_session` `action:"next_action"` |
| `doriku_token_usage` | `doriku_session` `action:"token_usage"` |
| `doriku_capabilities` | `doriku_session` `action:"capabilities"` |
| `doriku_agent_interrupt` | **removed from MCP** — console/REST |
| `doriku_agent_inject_task` | **removed from MCP** — console/REST |
| `doriku_session_interrupt` | **removed from MCP** — console/REST |
| `doriku_session_inject_task` | **removed from MCP** — console/REST |
| `doriku_ws_session_list` | **removed from MCP** — admin-scoped, console/REST |

## 5. Schema Drafts

### 5.1 `doriku_task` (full draft)

```json
{
  "name": "doriku_task",
  "description": "Tasks: lifecycle, coordination, and recovery. Select operation with 'action'. Server validates per-action requirements and returns hints on missing fields.",
  "inputSchema": {
    "type": "object",
    "required": ["action"],
    "properties": {
      "action": {
        "type": "string",
        "enum": ["create", "get", "list", "search", "update", "delete", "log",
                 "assign", "claim", "release", "depend", "decompose", "tree",
                 "ready", "resume", "github_link", "github_links"],
        "description": "create: new task (add objective/done_criteria for quality feedback). get/list/search: read. update/delete: modify. log: progress note (add summary/next_step for structured report). assign/claim/release: agent ownership. depend: dependencies. decompose: split into subtasks. tree: subtask hierarchy. ready: unblocked tasks. resume: full resume pack (state, deps, logs, contexts, decisions). github_link(s): attach/list GitHub URLs."
      },
      "task_id":       {"type": "string", "format": "uuid", "description": "Required for get/update/delete/log/assign/claim/release/depend/decompose/tree/resume/github_link/github_links"},
      "title":         {"type": "string", "description": "create: required. update: optional"},
      "description":   {"type": "string"},
      "objective":     {"type": "string", "description": "create: what outcome this task must produce"},
      "done_criteria": {"type": "array", "items": {"type": "string"}, "description": "create: verifiable completion criteria"},
      "status":        {"type": "string", "enum": ["pending", "queued", "running", "completed", "failed", "cancelled", "timeout"]},
      "priority":      {"type": "integer", "enum": [0, 1, 2, 3]},
      "tags":          {"type": "array", "items": {"type": "string"}},
      "project_id":    {"type": "string", "format": "uuid"},
      "visibility":    {"type": "string", "enum": ["workspace", "agent"]},
      "result":        {"type": "object", "description": "update: task result payload"},
      "message":       {"type": "string", "description": "log: progress note"},
      "summary":       {"type": "string", "description": "log: structured progress summary"},
      "next_step":     {"type": "string", "description": "log: what happens next"},
      "blockers":      {"type": "array", "items": {"type": "string"}, "description": "log: current blockers"},
      "agent_id":      {"type": "string", "format": "uuid", "description": "assign: target agent"},
      "depends_on":    {"type": "array", "items": {"type": "string"}, "description": "depend: prerequisite task UUIDs"},
      "subtasks":      {"type": "array", "items": {"type": "object"}, "description": "decompose: subtask definitions"},
      "url":           {"type": "string", "description": "github_link: GitHub issue/PR/commit URL"},
      "query":         {"type": "string", "description": "search: keyword query"},
      "filters":       {"type": "object", "description": "list: status/tag/project/agent filters"},
      "cursor":        {"type": "string", "description": "list: pagination cursor"}
    }
  }
}
```

### 5.2 `doriku_agent` (full draft)

```json
{
  "name": "doriku_agent",
  "description": "Agent presence and coordination. heartbeat also delivers pending commands (interrupt / inject_task) queued by the console.",
  "inputSchema": {
    "type": "object",
    "required": ["action"],
    "properties": {
      "action": {
        "type": "string",
        "enum": ["register", "heartbeat", "list", "status", "activity", "message", "work_set", "work_status"],
        "description": "register: bind this session as an agent. heartbeat: liveness ping; response includes pending_commands[] to act on. list/status/activity: read agents. message: send to another agent. work_set: publish current file/branch/activity. work_status: all agents' current work."
      },
      "name":         {"type": "string", "description": "register: agent display name"},
      "type":         {"type": "string", "description": "register: e.g. claude-code, cursor"},
      "capabilities": {"type": "array", "items": {"type": "string"}},
      "agent_id":     {"type": "string", "format": "uuid", "description": "status/message: target agent"},
      "message_type": {"type": "string", "description": "message: type tag"},
      "payload":      {"type": "object", "description": "message: content"},
      "file":         {"type": "string", "description": "work_set: file currently being edited"},
      "branch":       {"type": "string", "description": "work_set: git branch"},
      "activity":     {"type": "string", "description": "work_set: one-line description"}
    }
  }
}
```

### 5.3 `doriku_memory` (full draft)

```json
{
  "name": "doriku_memory",
  "description": "Workspace-scoped persistent memory shared across sessions, projects, and AI tools. glossary/preference tune memory search scoring.",
  "inputSchema": {
    "type": "object",
    "required": ["action"],
    "properties": {
      "action": {
        "type": "string",
        "enum": ["set", "get", "list", "search", "delete", "glossary", "preference"],
        "description": "set: upsert by key. get: read by key. list: browse with filters. search: lexical (default) or semantic (Pro). delete: remove by key. glossary: manage synonym expansion. preference: manage category boost axes."
      },
      "key":      {"type": "string", "description": "set/get/delete: memory key"},
      "content":  {"type": "string", "description": "set: memory body"},
      "category": {"type": "string"},
      "tags":     {"type": "array", "items": {"type": "string"}},
      "pinned":   {"type": "boolean"},
      "query":    {"type": "string", "description": "search: query text"},
      "mode":     {"type": "string", "enum": ["lexical", "semantic"], "description": "search: semantic requires solo_pro or workspace OpenAI key"},
      "op":       {"type": "string", "enum": ["get", "set", "delete"], "description": "glossary/preference: sub-operation"},
      "entries":  {"type": "object", "description": "glossary/preference: payload for op=set"}
    }
  }
}
```

### 5.4 Remaining tools (abbreviated)

Same construction — `action` enum + flat properties reusing the current per-tool fields verbatim:

- `doriku_context(action, key, value, source, scope, category, project_id)`
- `doriku_decision(action, decision_id, title, problem, alternatives, chosen, rationale, status, project_id)`
- `doriku_project(action, project_id, name, description)`
- `doriku_workspace(action, workspace_id, name, since_seq)` — `changes` takes `since_seq`, pairs with `snapshot`
- `doriku_workflow(verb, ...)` — existing schema + four `step_*` verbs with `step_run_id`, `result`, `error`
- `doriku_automation(action, rule_id, name, trigger_type, trigger_config, conditions, actions, enabled, dispatcher config fields)`
- `doriku_governance(action, path, lock_id, policy, cap, filters, time_range)`
- `doriku_search(query)` — unchanged
- `doriku_session(action, handoff fields | usage fields)`

## 6. Control Plane: Fix or Retire

Current state: `agent_interrupt` / `inject_task` write to the WS hub (`service/realtime_commands.go`), but no MCP-connected agent consumes WS, and heartbeat carries no commands. The tools are advertised but cannot work end-to-end.

**Option A — deliver via heartbeat (recommended, implemented).**
1. Commands are already persisted in the `command_logs` table (migration 000013); WS-missed ones carry status `undelivered`.
2. `doriku_agent action:"heartbeat"` response gains `pending_commands: [{command_id, command_type, task_id, reason, payload, created_at}]`; delivered ones are marked `sent`. Commands older than 15 minutes are skipped — a late interrupt is worse than a dropped one.
3. The Claude Code side polls via a `Stop`/`PostToolUse` hook or a lightweight `doriku poll` CLI daemon — deterministic, no model cooperation needed. (CLI-side polling not yet built.)
4. Interrupt/inject stay console-triggered (REST), never MCP-triggered — removing them from the MCP surface is still correct.

**Option B — retire from the agent-facing product.** Reposition as dashboard visibility only. Zero engineering, honest surface.

Either way the five control-plane tools leave `tools/list` now.

## 7. Server Instructions Rewrite

Replace the MANDATORY-tracking block with (~120 words):

```
You are connected to Doriku, a shared sync layer for AI tools and teams.

Task capture is automatic via editor hooks — do not create tracking tasks proactively.

Use these tools when they help the current work:
- doriku_search / doriku_task(action:"search"): find prior related work before starting anything non-trivial.
- doriku_task(action:"resume"): restore full context when resuming or taking over a task.
- doriku_session(action:"handoff"): record a structured handoff before ending a long session.
- doriku_decision(action:"record"): record significant technical decisions with alternatives and rationale.
- doriku_memory: store/recall knowledge that must survive across sessions and projects.

In multi-agent workspaces: claim work with doriku_task(action:"claim") and guard shared files with doriku_governance(action:"lock").
```

Also remove the hook-vs-model double-tracking: since hooks own capture, the model-side "create a task for every request" path disappears entirely.

## 8. Backward Compatibility & Rollout

1. **Alias map, permanent.** `tools/call` accepts all 76 legacy names forever via a static `map[legacyName]{tool, action}` translated before dispatch. Cached tool lists and old configs keep working; `tools/list` just stops advertising them.
2. **Surface flag.** Workspace (or API-key) setting `mcp_surface: "compact" | "legacy"`, default `legacy` at launch. `tools/list` branches on it. New `@doriku/cli setup` writes `compact`.
3. **Phases.**
   - Phase 1: implement 12 tools + alias map + flag (server-only change, no client break).
   - Phase 2: dogfood `compact` on own workspace; watch tool-call error rates via audit log.
   - Phase 3: default `compact` for new signups; announce deprecation window for `legacy` listing (calls never break).
   - Phase 4 (optional): heartbeat `pending_commands` + CLI poll hook (§6 Option A).
4. **Testing.** Extend `session_test.go` with alias-dispatch cases; per-action required-field validation tests reuse the `mcpErrorToolResult` fixtures.

## 9. Measured Impact (implemented, verified against a live server)

| Dimension | Before | After |
|---|---|---|
| `tools/list` payload | 38.7 KB / 76 schemas | **15.4 KB / 12 schemas (measured)** |
| Standing context per session (non-deferred hosts) | ~10k tokens | ~3.9k tokens |
| Task-family choice ambiguity | 18 sibling tools, 4 documented "preferred over" pairs | 1 tool, quality fields inline |
| Dead tools advertised | 5 | 0 |
| Capture reliability | model-dependent (instruction) | deterministic (hook) |
| Legacy client breakage | — | none (permanent alias map) |

The original ~9 KB estimate was structurally unreachable: ~180 properties across 12
tools cost ~12.5 KB in JSON structure alone (names, types, action enums) before any
description text. 15.4 KB (40% of legacy, 2.5× reduction) is the practical floor
without cutting per-action field documentation, which is what makes a 12-tool
surface usable. A regression test pins the compact surface under 40% of legacy.

## 10. Phase 1 Implementation Notes (2026-07-06)

Implemented in `doriku.service.api`:

- `internal/mcp/surface_compact.go` — 12 compact tool schemas, `compactRoutes`
  action table, `resolveCompactCall` router (quality-upgrade rules inline), and
  the rewritten compact server instructions.
- `internal/mcp/server.go` — `mcpSurface()` reads the workspace setting
  `mcp_surface` (default `legacy`); `initialize` and `tools/list` branch on it;
  `tools/call` translates compact names before the legacy dispatch switch, so
  both surfaces are always callable regardless of the flag.
- `internal/mcp/tools_agent.go` + `internal/repository/command_repo.go` —
  heartbeat delivers pending `command_logs` (§6 Option A) and marks them `sent`.
- `internal/mcp/surface_compact_test.go` — router mapping table, quality-upgrade
  routing, invalid-action errors, legacy passthrough, surface listing sizes,
  end-to-end compact dispatch, heartbeat delivery/redelivery/staleness, and the
  workspace-flag test. Full `go test ./...` green (Postgres-backed).

Runtime-verified over real HTTP against a booted server (auto-migrated, isolated
Docker network): legacy default 76 tools; workspace flip → 12 tools + compact
instructions; `doriku_task action:"create"` creates a real task; the structured
variant routes to the capture handler and returns quality meta (score 85);
invalid action returns the action list as a hint; heartbeat delivered a seeded
interrupt exactly once.

Enabling for a workspace (until console UI exists):
`UPDATE workspaces SET settings = settings || '{"mcp_surface":"compact"}' WHERE id = ...;`

Deployed to production 2026-07-06 via the canonical `doriku-io/doriku.service.api`
repo (main at 23772cd, GitHub Actions blue-green, image `ghcr.io/doriku-io/*`).
Verified live: legacy surface unchanged (76 tools), compact aliases callable,
new error hints active, /health ok.

Deploy post-mortem: the first push went to `Deiamor/doriku.service.api`, a stale
pre-migration duplicate — Deiamor credentials get 404 on the private doriku-io
repos, which was misread as "repo moved back". That detour deployed once from
`ghcr.io/deiamor/*` and temporarily rewrote the server's deploy script (commits
663b5ac + a4116fc on the Deiamor copy). Fixed by adding Deiamor as collaborator
on the doriku-io repo, reverting the namespace change (23772cd), and redeploying
from the canonical line. Recommendation: archive the four stale `Deiamor/*`
duplicates to prevent recurrence.

Not yet done: CLI-side heartbeat polling hook (§6 step 3), console toggle for
`mcp_surface`, default flip for new signups (Phase 3).
