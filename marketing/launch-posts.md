# Launch / build-in-public post drafts — v2, ask-tone (2026-07-06)

Direction: humble "I built this, would you evaluate it?" ask, not a product announcement. Tech numbers stay in the body as substance, never as the headline. Evaluators get a concrete offer: try it free for a week and tell us what's broken.

Offer mechanics — LIVE as of 2026-07-06:
- Every new signup automatically gets a 7-day Pro trial (server-side plan override `signup_trial_7d`, no card). The "일주일 무료" promise is fully self-serve — no DM roundtrip needed.
- Console billing page shows "Pro 평가판 · YYYY.MM.DD까지"; after expiry the account falls back to Free automatically.
- DM/comment responses are for trial EXTENSIONS: admin dashboard → users → plan override (pick expiry).

---

## 1. Show HN

**Title:**
Show HN: A shared memory for AI coding agents (Claude Code, Codex, Cursor)

**URL:** https://doriku.io

**First comment (post immediately after submitting):**

Hi HN, solo builder here. I kept losing context every time I switched between Claude Code and Codex — each agent re-discovers the codebase from zero, and two agents happily overwrite each other's edits. So I built Doriku: one workspace (tasks, memory, decisions, file locks) that all MCP-compatible agents read and write — Claude Code, Codex CLI, Cursor, Gemini CLI, Windsurf, anything that speaks MCP.

I honestly don't know yet if this is a real problem for other people or just mine. That's what I'm here to find out.

If you run more than one AI agent on a repo, I'd be grateful if you tried it for a week and told me where it falls apart. Every signup starts with a full 7-day Pro trial automatically — no card, nothing to request. I want criticism more than sign-ups; if a week isn't enough, comment and I'll extend it.

One thing I did learn along the way: our MCP surface had grown to 76 tools (~39 KB of tools/list injected into every session). Consolidating to 12 verb-shaped tools cut that by 60% with zero breaking changes. Happy to share details in the comments if useful.

---

## 2. X/Twitter thread (EN)

**1/** (271/280) AI agents have amnesia — Claude Code figures out the codebase, an hour later Codex starts from zero.

So I built a shared memory + task board for every MCP agent — Claude Code, Codex, Cursor, Gemini CLI: doriku.io

Is this just my problem? Help me find out.

**2/** (257/280) The ask: if you run 2+ AI coding agents, try it for a week and tell me what's broken or pointless.

Every signup starts a full 7-day Pro trial automatically — no card. Need longer? Reply and I'll extend it. I'm optimizing for honest criticism, not sign-ups.

**3/** For the technically curious: building this taught me that MCP tool sprawl is a real context tax — we went from 76 tools to 12 and cut tools/list by 60% (38.8KB → 15.5KB), old names still work. Wrote it up here: https://dev.to/dorikuio/we-collapsed-76-mcp-tools-into-12-and-cut-per-session-context-cost-by-60-do

---

## 3. X/Twitter 스레드 (KO)

**1/** (255/280) Claude Code가 겨우 파악한 코드베이스를, 한 시간 뒤 Codex는 처음부터 다시 배웁니다.

그래서 만들었습니다 — Claude Code·Codex·Cursor·Gemini CLI 등 모든 MCP 호환 툴이 공유하는 메모리 + 태스크 보드, Doriku. doriku.io

저만의 문제인지 궁금합니다.

**2/** 그래서 부탁드립니다 — AI 코딩 에이전트를 2개 이상 쓰시는 분, **일주일만 무료로 써보고 평가해주시겠어요?**

가입하면 Pro 7일 평가판이 자동으로 시작됩니다. 카드 등록도, 요청도 필요 없습니다. 일주일이 부족하면 댓글 주세요, 연장해드릴게요. 가입자 수보다 솔직한 혹평이 필요합니다.

**3/** 어디가 불편한지, 뭐가 쓸모없는지, 온보딩 어디서 막히는지 — 그런 피드백 하나하나가 지금 단계에선 제일 큰 도움입니다. 부탁드립니다 🙏

---

## 4. GeekNews (news.hada.io) — KO

**제목:** AI 코딩 에이전트들(Claude Code·Codex·Cursor 등)이 공유하는 메모리 레이어를 만들어봤습니다 — 일주일 써보고 평가해주실 분 계실까요?

**본문:**

솔로 빌더입니다. Claude Code, Codex, Cursor를 번갈아 쓰면서 두 가지가 계속 답답했습니다: ① 에이전트를 바꿀 때마다 코드베이스 컨텍스트를 처음부터 다시 설명해야 하고, ② 에이전트 두 개가 같은 파일을 서로 덮어씁니다.

그래서 MCP로 붙는 공유 워크스페이스를 만들었습니다. 태스크(의존성 포함), 영속 메모리, 의사결정 기록, 파일 잠금을 한곳에 두고 모든 에이전트가 읽고 쓰는 구조입니다. MCP를 지원하는 툴이면 무엇이든 붙습니다 — Gemini CLI, Windsurf 포함.

**부탁드리고 싶은 것:** 멀티 에이전트로 개발하시는 분들이 일주일 정도 실제로 써보시고 "이건 쓸모없다", "온보딩 여기서 막힌다" 같은 솔직한 평가를 주시면 정말 감사하겠습니다. 가입하면 Pro 7일 평가판이 자동으로 시작됩니다 — 카드 등록도, 별도 요청도 필요 없습니다. 일주일이 부족하면 댓글 주세요, 연장해드리겠습니다.

만들면서 배운 것 하나를 공유하자면 — MCP 도구가 76개까지 늘어나니 tools/list만 39KB, 매 세션 컨텍스트에 그대로 주입되더군요. 동사 중심 12개로 통합해 60% 줄인 과정을 블로그에 정리했습니다(기존 이름은 전부 호환 유지): https://dev.to/dorikuio/we-collapsed-76-mcp-tools-into-12-and-cut-per-session-context-cost-by-60-do

- 사이트: https://doriku.io
- 공식 MCP Registry: io.doriku/task-manager

혹평 환영합니다. 지금 단계에선 그게 제일 필요합니다.

---

## 5. Reddit r/ClaudeAI (+ r/mcp은 기술 각도로)

### r/ClaudeAI — ask 톤

**Title:** Built a shared memory so Claude Code and Cursor stop re-learning my codebase — would anyone evaluate it for a week?

**Body:**

Solo builder. My workflow bounces between Claude Code and Codex, and every switch means the new agent re-discovers everything — plus they occasionally clobber each other's edits.

I built Doriku to fix that for myself: one MCP workspace with shared tasks, persistent memory, decision records, and file locks that every agent reads/writes — works with anything that speaks MCP (Codex CLI, Cursor, Gemini CLI, Windsurf, …).

What I need now is brutally honest evaluation from people who actually run multi-agent workflows. Every signup automatically starts a 7-day full Pro trial — no card. Tell me what's useless, what's confusing, where onboarding breaks. That feedback is worth more than any sign-up right now. (Need longer? Comment and I'll extend your trial.)

(doriku.io — also on the official MCP Registry as io.doriku/task-manager)

### r/mcp — 기술 공유 각도 유지 (커뮤니티 성격상 이쪽이 광고로 안 잘림)

**Title:** Collapsed our MCP server from 76 tools to 12 — tools/list dropped 60%, zero breaking changes

**Body:**

We run an MCP server (shared task/memory sync for AI coding agents) that grew organically to 76 tools. Every session was loading 38,818 bytes of tool definitions before doing any work — and with N agents × M sessions a day, that context tax multiplies fast. Tool routing also degraded: with 76 near-duplicates, models kept picking the wrong variant.

What worked for us:

- Redesigned around 12 verb-shaped tools (`task`, `memory`, `decision`, `agent`, …) that take an `action` parameter and route internally to the same handlers
- Kept all 76 legacy names permanently resolvable — the dispatcher accepts both surfaces, so existing configs never break
- Added a CI regression test that fails if the compact `tools/list` ever exceeds 40% of legacy
- Started serving the compact surface to unauthenticated `initialize`/`tools/list` so directory scanners (Smithery, the official registry) can see capabilities without auth — `tools/call` stays behind auth

Result: 15,514 bytes (40.0% of before), and noticeably better tool selection since models choose from 12 options instead of 76.

If your MCP server has 20+ tools, count your `tools/list` bytes — you might be paying more context tax than you think. Happy to answer questions.

(The server behind this is doriku.io — every signup starts a free 7-day full trial. If you kick the tires for a week and tell me what's broken, that's worth more to me than the sign-up.)

---

## Posting checklist & order

| # | Channel | Content | Timing |
|---|---------|---------|--------|
| 0 | 준비 | ✅ 완료 — 가입 시 7일 Pro 트라이얼 자동 (연장은 어드민 plan override) | 게시 전 |
| 1 | Blog | ✅ **게시 완료 (2026-07-06)**: https://dev.to/dorikuio/we-collapsed-76-mcp-tools-into-12-and-cut-per-session-context-cost-by-60-do — 태그 mcp/ai/devtools/productivity | 완료 |
| 2 | GeekNews | §4 | 당일 저녁 KST |
| 3 | X KO + EN | §3, §2 | 당일 |
| 4 | Reddit r/ClaudeAI, r/mcp | §5 | 익일 |
| 5 | Show HN | §1 | 평일 22:00–24:00 KST, 3시간 댓글 대응 가능한 날 |

Notes:
- 모든 채널 공통: 헤드라인은 "만들어봤는데 평가 부탁", 기술 수치는 본문 신뢰 재료로만.
- "일주일 무료" 문구는 가입 즉시 자동 시작되는 7일 Pro 트라이얼이 뒷받침 (2026-07-06 라이브) — 과장 없음.
- 평가 피드백은 반드시 TASK.md/이슈로 수집해 다음 제품 사이클 입력으로.
