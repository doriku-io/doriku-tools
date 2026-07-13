# Launch posting log

## 🧊 PROJECT DECOMMISSIONED (cold storage) — user decision 2026-07-13

Superseding the same-day dormancy decision below: the user chose to **fully clear the server** ("어차피 서비스는 모두 깃헙에 최신화되어 있잖아? 서버에서는 깔끔하게 치워버리자").

**Executed 2026-07-13:**
- Final backup taken and verified BEFORE the wipe: fresh `pg_dump` (1.2M) + `/opt/doriku` (all .env secrets) + `/home/hosting/io/doriku` (deploy scripts, nginx, ssl) + crontab → `~/dev/doriku/server-final-backup-20260713/doriku-final-backup-20260713.tar.gz` (56M, **contains secrets — never commit**). Daily R2 backups also exist (cron was uploading pg_dump to Cloudflare R2 nightly).
- Server wiped: 7 doriku containers, 4 doriku volumes (postgres/redis data), all images, doriku crontab (backup/health-monitor/daily-report), `/opt/doriku`, `/home/hosting/io/doriku` — all removed. **ko-gap remnants untouched** (`comko-gap_*` volumes ×3, `/home/hosting/com`).
- Verified down: https://doriku.io and https://api.doriku.io/mcp both unreachable (curl 000).
- Local Claude Code integration removed: `doriku` entry deleted from `~/.claude/mcp.json`, doriku Stop/PreToolUse hooks removed from `~/.claude/settings.json`, `doriku-stop-hook.sh` / `doriku-file-guard.sh` / `agents/doriku-session.md` archived into the backup dir.
- The exposed `drk_live_d223c38...` key is now moot — the API it authenticated against no longer exists.

**Cold-storage inventory (full revival possible):** code = GitHub `doriku-io/*` (4 repos, private) + GHCR images; data = local tarball + R2 dumps; decision history = this file + the pg_dump (includes decision_record aa1d0b63, the dormancy record).

**User decisions still open (billing, not urgent):** (a) DAOU VPS ~$30/mo — now nearly empty except ko-gap volumes; cancel to actually save the money (cancelling kills ko-gap volumes too). (b) doriku.io domain — keep renewing cheaply to hold the name, or let lapse. (c) Paddle products/EARLYBIRD — archive at leisure; checkout is already dead, zero customers affected.

---

## ⏸️ (superseded same day) PROJECT DORMANT — user decision 2026-07-13

The launch campaign is **terminated**. Doriku enters dormancy (maintenance-free, self-host mode):

- **No further posting**: GeekNews (#6) cancelled (account was eligible from 7/13 but post was never made), Show HN (#5) abandoned, Glama manual registration cancelled.
- **No monitoring**: reaction-check crons are dead and will not be re-registered. Existing threads (dev.to / X / Threads / Reddit) are left as-is; no obligation to answer new comments.
- **r/ClaudeAI modmail**: left pending — if mods approve later, the post simply goes live unattended.
- **Server stays up** (~$30/mo DAOU VPS) as the founder's personal Claude Code infra (hooks/memory/session tracking). Paddle checkout + EARLYBIRD coupon left live — harmless at current traffic.
- **Publicly committed per-tool telemetry feature (dev.to/Threads replies): will not be built** while dormant.
- Decision basis (measured 2026-07-13): 0 external signups ever (3 accounts, all founder's own); launch week delivered ~6 clicks total from all channels (Threads 4, Reddit 2, dev.to/X 0); funnel died at reach (all zero-reputation brand accounts), so product validation never actually started.
- **Revival triggers**: (a) founder personally feels multi-agent/cross-tool coordination pain in real use (Claude Code + Codex), or (b) clear external demand signal for cross-tool agent fleet control. Repositioning direction if revived: "one person's agent fleet control plane" (single-user, multi-agent, cross-tool) instead of team collaboration SaaS.

## Status board (updated 2026-07-09 — re-verified live via browser-bridge, checked in order 5→3→2→1, #6 skipped) — FROZEN as of dormancy, kept for the record

| # | Platform | Account | Status | Engagement | Next action |
|---|----------|---------|--------|------------|-------------|
| 1 | dev.to (blog) | dorikuio | ✅ live | 1 reaction · **8 comments** — Alex Shev thread (3 msgs, all answered incl. 7/9 follow-up on audit-log-as-ops-feature, tied to the per-tool telemetry commitment) + Edu Peralta (answered). All caught up. | Monitor (4h cron) |
| 2 | X | @dorikuio | ✅ live (EN thread ×3) | 7/5/4 views per tweet, **0 ext. replies** (unchanged) | Monitor (4h cron) |
| 3 | Threads | @doriku.io | ✅ live (EN + topic "AI Agents" + tech reply) · profile/bio done | **124 views** (up from 109) · arumwu's follow-up question answered 2026-07-07 (dogfooding wrong-tool-selection anecdote + beta instrumentation plan) | Monitor (4h cron) |
| 4 | Reddit | u/dorikuio | ✅ **posted 2026-07-08** — r/mcp live; r/ClaudeAI submitted but held by Reddit's site-wide spam filter (new/low-karma account), pending manual mod review | r/mcp: 1 upvote, 0 comments. r/ClaudeAI: 1 upvote, 2 comments (both AutoModerator queue notices) | r/ClaudeAI: wait for mod approval or message r/ClaudeAI mods to request review — needs user call before contacting mods |
| 5 | Show HN | dorikuio | 🛑 **blocked** — HN is site-wide restricting Show HN submissions from newer accounts | — | Not a timing/account-age issue we can just wait out on a fixed date — HN's own message says to build real participation (comments/karma) first, then "it will be fine to post an occasional Show HN." Needs user decision. |
| 6 | GeekNews | doriku | ⏳ aging until **7/13** (on track) | — | Auto-alert via doriku task 9e49428e |

Infra done: OG share card updated (Codex + 12 tools), 7-day trial live, all brand accounts on contact@doriku.io (X pending — still gmail alias), 4h reaction-check cron v2 active (job 664061ab, this session only, expires in 7 days) — **note: this cron was session-scoped and is likely dead now since it survived past the original session**. Directory listings confirmed live: Official MCP Registry, Smithery (smithery.ai/server/doriku/doriku), mcp.so. Not yet ingested: PulseMCP. Not yet confirmed: Glama (JS-rendered search). GitHub doriku.cli: 0 stars. Signup/trial conversion numbers not yet confirmed (server SSH unavailable from this environment; admin dashboard check pending an idle browser tab).

---


One entry per platform. Records what was posted, where, with which account, and how — so replies, edits, and follow-ups can be handled without guessing. Update this file immediately after every posting action.

Reference copy: `launch-posts.md` (final v2, ask-tone). Blog source: `blog-mcp-consolidation.md`.

---

## 1. dev.to — ✅ published 2026-07-06 (KST evening)

- **URL**: https://dev.to/dorikuio/we-collapsed-76-mcp-tools-into-12-and-cut-per-session-context-cost-by-60-do
- **Account**: `dorikuio` — created during this posting via GitHub OAuth (doriku-io). Profile name "Doriku", bio set.
- **Title**: "We collapsed 76 MCP tools into 12 — and cut per-session context cost by 60%"
- **Tags**: #mcp #ai #devtools #productivity
- **Content**: full technical write-up from `blog-mcp-consolidation.md`, with two adjustments made at publish time (synced back to the source file):
  - Confessional framing line added as 2nd paragraph: "Full disclosure up front: every mistake in this story is our own making — nobody handed us 76 tools, we grew them." (user decision: option C)
  - Table header fixed: empty first cell → "Metric"
- **How it was posted**: editor filled programmatically (title + markdown body); tags failed in the UI (autocomplete component ignores programmatic/typed commits), so they were applied via the dev.to REST API (`PUT /api/articles/4080583`). The API key generated for this was **revoked immediately after use** (it had appeared in the session transcript).
- **Article ID**: 4080583 (needed for any future API edits)
- **Role in the sequence**: anchor content — GeekNews body and X EN thread link to this URL.
- **Comment thread (Alex Shev, 8 comments total)**: 7/7 exchange on "context surface area" framing → 7/8 exchange on byte-savings vs. call-log-legibility as the more durable win → 7/9 follow-up ("legible call log becomes an operations feature") answered same day, connecting the audit log's evolution from internal debug artifact to first customer-facing debugging surface, and tying it to the per-tool telemetry beta commitment made in the adjacent Edu Peralta sub-thread. All replies substantive, not acknowledgments.

## 2. GeekNews (news.hada.io) — ⏳ account created, posting blocked until 2026-07-13

- **Account**: `doriku` — created 2026-07-06 via signup form (Cloudflare Turnstile auto-passed). Recovery email: byunghee1994@gmail.com (verification link clicked from Gmail). Password: generated in-browser, NOT recorded anywhere — recover via 비밀번호 찾기 with the email if needed.
- **Blocker**: GeekNews requires accounts to be ≥1 week old before posting ("가입 후 일주일이 지나야 작성할 수 있습니다"). Earliest posting date: **2026-07-13**.
- **Alternative**: if the user has a personal aged GeekNews account, it could post §4 sooner.
- Copy ready: `launch-posts.md` §4 (blog URL embedded). Post type: Show.

## 3. X/Twitter — ✅ EN thread posted 2026-07-06 (KST night)

- **Account**: `@dorikuio` — created 2026-07-06 by the user manually (@doriku was taken by an unrelated person, "dorina kurzenberger"). Signup email: byunghee1994+doriku@gmail.com.
- **Profile**: display name "Doriku", avatar = logo.svg rendered to 400px PNG, bio "Shared memory & task sync for AI coding agents over MCP — Claude Code, Codex, Cursor & friends. Every signup starts a 7-day Pro trial. Solo-built.", website doriku.io.
- **Thread (EN, §2 final)**: https://x.com/dorikuio/status/2074136403580207290
  - 1/ amnesia hook + doriku.io (id 2074136403580207290)
  - 2/ 7-day-trial ask (id 2074136647369933302)
  - 3/ 76→12 numbers + dev.to link
- **Language decision**: EN only — user call ("영어로 가야지"); the KO thread (§3) was not posted. Reusable later if a KR-focused account/channel is wanted.
- **How**: X's composer strips newlines from programmatic/CDP typing (drafts also got corrupted mid-edit). Working method: **web intents** — `x.com/intent/post?text=` (URL-encoded \n preserved) for tweet 1, then `intent/post?in_reply_to=<id>&text=` for each reply. Remember this for future X posting.

## 4. Reddit — ✅ posted 2026-07-08

- **Account**: `u/dorikuio` — created 2026-07-06 with contact@doriku.io (verification code read from the Ouranos mailbox; final signup steps completed by the user). The user's personal account (PrestigiousStock3889) was logged out per user decision — brand account only in this browser.
- **r/mcp** — ✅ live: https://www.reddit.com/r/mcp/comments/1uqu8sv/collapsed_our_mcp_server_from_76_tools_to_12/ — flair "showcase" (tried "server" first, r/mcp's AI mod flagged a Rule 4 mismatch, switched to showcase and it published clean). Got organic engagement overnight: u/bsampera asked a real technical question (did tool-routing accuracy improve or just shift ambiguity from tool-name to action-name?) — answered honestly (no clean before/after numbers, reasoning why action-selection should still be structurally easier, tied back to the publicly-committed per-tool telemetry plan). u/sje397 linked a similar tool (MCPico, a proxy-based hierarchical tool bundler) — replied comparing approaches and admitting they have real benchmarked success-rate numbers that we don't.
- **Unplanned organic engagement found**: a prior-session comment by `dorikuio` on an unrelated r/ClaudeAI post ("Claude code set up a launchagent at login mid-session...", u/albzzz) got a substantive reply about session-attribution logging and git worktrees. Replied in depth: worktrees solve live-clobbering but push conflicts to merge time and don't cover non-git state (DB migrations, ports, queues); explained why Doriku went the file-lock route instead (interdependent work needs the second writer to notice the first, not just isolation) and that worktree-per-agent is right when work is genuinely independent. Posted successfully at https://www.reddit.com/r/ClaudeAI/comments/1uqs6ip/claude_code_set_up_a_launchagent_at_login/
- **r/ClaudeAI** — ⚠️ submitted but held: https://www.reddit.com/r/ClaudeAI/comments/1uqungn/built_a_shared_memory_so_claude_code_and_cursor/ — flair "Built with Claude". Confirmed via the post's own JSON (`removed_by_category: "reddit"`, `banned_by: null`, `is_robot_indexable: false`) that this is Reddit's own site-wide anti-spam removal (new/low-karma account), not a subreddit mod action — invisible to everyone but the author until a mod manually approves it. User decision (2026-07-08): message the mods. Sent via modmail (reddit.com/message/compose?to=/r/ClaudeAI), title "Post caught by Reddit's spam filter - requesting manual approval", explaining the auto-removal and asking for a look. Sent successfully ("메시지 전송됨"). Awaiting mod response.
- No warm-up comments were done before either post (skipped this round — both accounts were already 2 days old with some karma from prior session activity).
- Copy used verbatim: §5a (r/ClaudeAI, ask-tone), §5b (r/mcp, technical) from `launch-posts.md`.
- Tracked as doriku task 7e985856 (deadline 2026-07-07 21:00 KST → Slack alert, fired late but action now taken).

## 5. Show HN — 🛑 blocked 2026-07-08 (attempted, site-wide restriction)

- **Account**: `dorikuio` — created 2026-07-06 (no captcha), recovery email set to contact@doriku.io, about set. Password generated in-browser and not stored — use HN "forgot password" with contact@doriku.io if ever needed.
- **Attempt result**: filled title ("Show HN: A shared memory for AI coding agents (Claude Code, Codex, Cursor)") + url (https://doriku.io) at news.ycombinator.com/submit and clicked submit. HN returned an interstitial page instead of creating the post: **"Update re Show HNs — We're temporarily restricting Show HNs because of a massive influx, mostly by users who aren't yet familiar with the site or its culture. You're welcome on HN! Take some time to get to know the community, become a good contributor, and then it will be fine to post an occasional Show HN."** with links to newsguidelines.html / newswelcome.html / showhn.html.
- This reads as a site-wide policy gate tied to account history (2-day-old account, no prior comments/karma), not a retryable timing issue — no evening will fix it on its own.
- **Not attempted as a workaround**: posting the same content as a plain Ask HN / regular submission instead of "Show HN:" — untested whether that would also be blocked, and changes the framing from what was approved. Left for user decision.
- Title + first comment copy still valid and ready whenever this unblocks (§1).
- Tracked as doriku task b3032417 (provisional deadline 2026-07-08 22:00 KST → Slack alert; submission was attempted on-time but rejected by HN itself).

## 6. Threads — ✅ EN post live 2026-07-07 (KST early AM)

- **Account**: `@doriku.io` — existing Doriku Instagram-linked Threads account (user decision: brand IG account, not personal @mireebang). Threads web switched to it via Instagram SSO.
- **Post**: https://www.threads.com/@doriku.io/post/DadVVq9Eric — EN ask-tone (amnesia hook + 7-day trial ask), **topic tag "AI Agents"** for community feed discovery. Tech self-reply with the dev.to link attached.
- **Iterations**: KO version posted first → deleted (user call: EN + the 🙏 emoji rendered as � via intent URL). Final EN version uses ASCII-only punctuation — no broken chars.
- **Profile**: Instagram bio updated to "One shared memory & task board for every AI agent that speaks MCP - Claude Code, Codex, Cursor & beyond. Free 7-day Pro trial on signup." (emphasis per user: EVERY AI agent, not a fixed tool trio). Threads-side bio also updated to the same copy (2026-07-07). Quirk: Threads web bio save is **two-step** — the 소개 수정 sheet's 완료 only closes the sheet; the outer 프로필 편집 dialog's 완료 does the real save. Clicking just one silently discards.
- **How**: threads.com/intent/post?text= prefill works (newlines preserved); topic added via the "커뮤니티 또는 주제" input with real keystrokes + suggestion click. Emoji in intent URLs breaks — ASCII only.

### Product follow-up (from eduzsh exchange)
- **Per-tool MCP telemetry**: publicly committed in the reply — collect per-tool call frequency, failure rate, retries, missing args, wrong-tool selections during beta; merge/demote dead weight, promote frequent flows into presets. Candidate next engineering task on doriku.service.api (mcp_calls currently aggregates daily totals only).

### Found during posting (follow-up fixes)
- **Stale OG image**: the link-preview card (landing /og route) still says "Claude figures it out. Cursor picks it up. Windsurf..." and **"70+ MCP Tools"** — predates both the 12-tool consolidation and the all-MCP-tools positioning. Fix `doriku.service.landing/src/app/og/route.tsx` and redeploy; every social share benefits.
- Threads 관심사 tags: currently "AI, mcp" — could add "AI Agents" (manual, optional).

---

## Account & email policy

- **Brand email everywhere: contact@doriku.io** (Ouranos/Google Workspace mailbox — verification mail readable there, NOT forwarded to the personal gmail). Applied: Reddit (signup), HN (recovery), GeekNews (changed from personal gmail on 2026-07-06). X @dorikuio still uses byunghee1994+doriku@gmail.com — switch in X settings when convenient. dev.to auth rides GitHub OAuth (doriku-io).
- **Handle**: `dorikuio` on dev.to / X / Reddit / HN; `doriku` on GeekNews (@doriku was free there; taken on X by an unrelated person).
- Passwords for GeekNews/HN were generated in-browser and intentionally not recorded; recovery path is the email on file. Reddit/X passwords were set by the user.

## Automation for aged/scheduled steps

Session crons don't survive Claude restarts, so scheduling rides Doriku itself (tasks with deadlines → deadline alerts through the live Slack integration):

| doriku task | fires | action |
|---|---|---|
| 9e49428e | 2026-07-13 09:00 KST | GeekNews Show post (account aging complete) — say "GeekNews 게시 진행해" in any Claude session; this log + §4 has full context |
| 7e985856 | 2026-07-07 21:00 KST | Reddit posts (r/ClaudeAI + r/mcp) after comment warm-up |
| b3032417 | 2026-07-08 22:00 KST (provisional) | Show HN — user picks the actual evening |

---

## Standing notes

- Every public promise is backed: 7-day Pro trial auto-starts on signup (live), numbers measured from production 2026-07-06 (76→12 tools, 38,818 → 15,514 bytes = 40.0%).
- Feedback that comes in from any channel goes into TASK.md as product input.
- Per-channel etiquette: HN/Reddit — reply availability matters more than posting time; never post-and-ghost.
