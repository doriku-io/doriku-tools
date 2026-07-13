# We collapsed 76 MCP tools into 12 — and cut per-session context cost by 60%

*Doriku engineering, 2026-07-06. Draft for dev.to / blog / HN submission.*

Every MCP server pays a hidden tax: the moment an AI agent connects, your entire `tools/list` gets injected into its context window. Every tool name, every description, every JSON schema — before the user has typed a single word.

Full disclosure up front: every mistake in this story is our own making — nobody handed us 76 tools, we grew them.

Doriku is a shared control plane for AI coding agents — Claude Code, Codex, Cursor, Gemini CLI, Windsurf, or any MCP-compatible tool talks to one task/memory/decision store. We grew fast and organically: task CRUD, dependency tracking, file locks, decisions, semantic memory, workflows, cost caps, approvals… Each feature politely added its own tools. One day we counted: **76 tools, 38,818 bytes of `tools/list` payload**, loaded into every session of every agent, every day — before any work happened.

## The problem isn't the number, it's the multiplication

38 KB doesn't sound like much until you multiply it: N agents × M sessions per day × every reconnect. For a heavy user we measured 6,298 API calls over 30 days across multiple daily sessions. That's megabytes of context per user per month spent on *tool definitions* — pure overhead that competes with actual working context and dilutes the model's attention across 76 choices.

There's a quality cost too. Agents pick tools by reading descriptions. With 76 near-neighbors (`doriku_task_update` vs `doriku_task_progress_report` vs `doriku_work_status`…), models mis-route. Fewer, sharper tools = better routing.

## Design: verbs, not endpoints

The mistake we'd made was mirroring our REST API — one tool per endpoint. Agents don't think in endpoints; they think in intents. So we redesigned the surface around 12 verb-shaped tools:

```
task        — create / get / update / list / claim / complete / decompose …
memory      — set / get / search / delete
context     — capture / restore session context
decision    — record / list / update
agent       — register / heartbeat / message
project, workspace, search, lock, workflow, admin, capabilities
```

Each compact tool takes an `action` parameter and routes internally to the same handlers the legacy tools used. One schema per domain instead of eight.

**Result: 12 tools, 15,514 bytes — exactly 40.0% of the legacy payload.** We locked that in with a regression test that fails the build if the compact surface ever creeps past 40% of legacy.

## Breaking no one

76 tools were already in the wild — in MCP configs, in agent prompts, in muscle memory. So:

- **Every legacy tool name still works, permanently.** The dispatcher resolves both surfaces regardless of which one is advertised.
- Workspaces choose their surface (`legacy` / `compact`) in settings; new workspaces default to compact.
- The regression suite asserts every routed legacy name is still dispatchable.

Migration cost for users: zero. They restart their MCP session and the agent simply sees 12 tools instead of 76.

## A side quest: discovery without auth

MCP directories (Smithery, the official registry, mcp.so) scan servers by calling `initialize` and `tools/list` anonymously. Our server used to 401 those — directories listed us as "no capabilities." We now serve the compact surface to unauthenticated discovery while keeping `tools/call` behind auth. Listings fixed, security unchanged.

## Numbers that mattered

| Metric | Legacy | Compact |
|---|---|---|
| Tools | 76 | 12 |
| `tools/list` payload | 38,818 B | 15,514 B (40.0%) |
| Context saved per session | — | ~23 KB |
| Breaking changes | — | 0 |

If you run an MCP server with more than ~20 tools, you're probably paying this tax too. Count your `tools/list` bytes. Then think in verbs.

---

*Doriku is a shared memory and coordination layer for AI coding agents — [doriku.io](https://doriku.io). On the [MCP Registry](https://registry.modelcontextprotocol.io) as `io.doriku/task-manager`. I'm a solo builder and honestly still figuring out whether this solves a problem beyond my own — if you run multiple agents on one repo, I'd be grateful if you tried it for a week (free, no card) and told me what's broken.*
