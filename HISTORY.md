# HISTORY — doriku_tools

> EN-only change log. Archive index below.

---

## [#1] 2026-02-15 01:50:00

### 1. User Request
- Initialize the `doriku_tools` project root directory for developing and managing standalone tools provided by the doriku service.

### 2. File Changes
- **Created**: `TASK.md` — Empty task registry (JSON format)
- **Created**: `HISTORY.md` — EN change log (this file)
- **Created**: `HISTORY_KR.md` — KR change log

### 3. Technical Summary
Project bootstrap. The `doriku_tools` repository serves as the mono-root for all independent tools that the doriku service exposes. Management files (`TASK.md`, `HISTORY.md`, `HISTORY_KR.md`) initialized per operational guidelines.

---

## [#2] 2026-07-06

### 1. User Request
- Document the pricing unit-economics calculation structure; investigate why the console billing card renders in English under the Korean locale; audit i18n coverage across every console/landing page for all supported locales (en/ko/zh) and fix the gaps.

### 2. File Changes
- **Created**: `../PRICING.md` — pricing & unit-economics SSOT (per-user cost, Paddle fee model, $30/mo server, breakeven 4 early-bird / 2 full-price users, step-function scaling plan)
- **console (doriku.service.api.admin)**: ~390 hardcoded strings wired to next-intl across 21 pages + 7 components; new `messages/{en,ko,zh}/pages.json` (387 keys each); billing card fully localized (features, Subscribe/Current plan, Beta access, early-bird note); settings MCP-surface labels no longer leak Korean into en/zh
- **landing (doriku.service.landing)**: locale-aware `generateMetadata` for root layout + pricing; `common.a11y` strings for Header/ThemeToggle/ComingSoon; stale "76+ MCP tools" copy replaced with "12 focused MCP tools" (hero stat + 3 locales)

### 3. Technical Summary
Root cause of the English billing card: strings hardcoded as literals (PRO_FEATURES array, TierCard labels) bypassing next-intl entirely — message-key parity across locales was already perfect. Fix pattern: per-page namespaces spread from `pages.json` in `i18n/request.ts`. Verified live: 21 console pages fetched under ko/zh cookies show zero raw-key leaks; en shows zero Hangul; landing titles localized per locale. Remaining intentional English: docs/privacy/terms/refund pages (per decision), template/email placeholder examples.

---

## [#3] 2026-07-06 (evening)

### 1. User Request
- Polish sync-feed event copy; prepare launch content in ask-tone; make the promised "free 7-day evaluation" real.

### 2. File Changes
- **api**: `internal/auth/auth.go` — new signups default to `free` plan + automatic 7-day `solo_pro` override (`signup_trial_7d`); `internal/plans/limits.go` — unknown-plan fallback `solo_basic` → `free`; `internal/billing/billing.go` + `billing_handlers.go` — subscription response exposes active `plan_override`
- **console**: dashboard sync feed renders per-event localized sentences (28 event types × en/ko/zh) with task title from delta; billing page shows "Pro trial · until YYYY.MM.DD" badge
- **landing**: pricing page trial note (3 locales)
- **marketing**: `doriku-tools/marketing/blog-mcp-consolidation.md` + `launch-posts.md` v2 (ask-tone, self-serve trial copy)

### 3. Technical Summary
Trial rides the existing `admin_plan_overrides` machinery — expiry enforced by `GetActivePlanOverride`'s `expires_at` filter, so no cron needed; accounts fall back to Free automatically. Fixed latent inconsistency: new users previously got legacy `solo_basic` limits (unlimited agents/tasks) while the site advertised Free limits. Live-verified: trial note on 3 locales, billing response field present. New-signup E2E pending a fresh Google account test.

## [#4] 2026-07-07

### 1. User Request
- Fix agents-table status badge line-wrapping and date formats (console-wide, all locales); launch the product across dev.to / X / Threads / Reddit / HN / GeekNews with an ask-tone ("please evaluate for a week") pitch instead of a feature announcement; make the promised 7-day trial self-serve; widen positioning from a fixed tool trio to "every MCP-compatible agent"; run 4-hourly reaction monitoring.

### 2. File Changes
- **console**: `components/badge.tsx` — `whitespace-nowrap`; `lib/utils.ts` — `formatDate`→`YYYY.MM.DD`, new `formatDateTime` (+ time), locale-aware `formatRelative` (`Intl.RelativeTimeFormat`) and `formatWeekdayShort`; call sites updated across tasks/api-keys/invitations/summaries/webhooks/billing/dashboard/kanban-board
- **console dashboard**: sync feed renders natural per-event sentences (28 event types × en/ko/zh) instead of concatenated `{actor} {event_type} {entity}`; entity badge fallback localized
- **landing**: hero/steps/features/CTA copy + SEO meta + keywords widened to Codex/Gemini CLI/"every MCP-compatible agent" (3 locales); `src/app/og/route.tsx` — share-card copy and "70+ MCP Tools" → "12 Focused MCP Tools"
- **marketing (new)**: `doriku-tools/marketing/blog-mcp-consolidation.md` (published to dev.to), `launch-posts.md` v2 (ask-tone, widened targeting, weighted-length-checked X threads), `launch-log.md` (per-platform posting log + live status board)
- **accounts created**: dev.to `dorikuio` (GitHub OAuth), X `@dorikuio`, HN `dorikuio`, Reddit `u/dorikuio`, GeekNews `doriku` — all pointed at contact@doriku.io except X (personal gmail alias, not yet switched); Threads switched to existing `@doriku.io` (Instagram-linked)

### 3. Technical Summary
Badge/date bugs: Badge lacked `whitespace-nowrap`; date formatting was scattered `toLocaleDateString()`/hardcoded-English-relative calls — centralized into `lib/utils.ts` and made locale-aware via the `DORIKU_LOCALE` cookie. Launch execution: dev.to blog published (measured numbers: 76→12 tools, 38,818B→15,514B tools/list, 40.0%), X thread live, Threads post live with community topic tag "AI Agents" (109 views, 18× the X reach — attributed to topic-tag distribution vs. a 0-follower new account). Two external commenters (dev.to: alexshev, eduzsh; Threads: arumwu) engaged with technical questions about the 76→12 redesign methodology — all replied to using only verified facts (no fabricated usage-audit claims), and the reply publicly commits to per-tool telemetry as a beta follow-up (candidate engineering task, not yet scheduled). GeekNews blocked by a 1-week account-age platform rule (auto-resumes 2026-07-13); Reddit posting deferred to same-day 21:00 KST after karma warm-up; Show HN date not yet chosen. Session-only cron (4h) monitors dev.to/X/Threads reactions and will expire in 7 days — not durable across session restarts.

### 4. Known follow-ups
- Per-tool MCP call telemetry (frequency/failure/retry/wrong-selection) — publicly promised in dev.to + Threads replies, not yet implemented (`mcp_calls` currently aggregates daily totals only).
- Switch X account recovery email from gmail alias to contact@doriku.io.
- New-signup 7-day-trial E2E still unverified with a fresh Google account.
- Exposed live API key from a prior session still needs rotation (carried over from 2026-05-27).

---

## [#5] 2026-07-13

### 1. User Request
- Decide the project's fate after the launch produced no signups; decommission everything; verify local↔GitHub parity, write an as-built architecture doc into the backend repo, archive all records, then delete the local code.

### 2. File Changes
- **doriku.service.api**: `docs/ARCHITECTURE.md` (new) — as-built topology, blue/green deploy mechanics, Postgres/Redis config, env var inventory, ops crons, decommission record, 9-step revival procedure. Pushed as c22a1f7; the triggered Deploy CI run was cancelled and the deploy workflows of all four service repos were disabled (`gh workflow enable` to revive).
- **doriku-tools**: `marketing/` (launch-log v. final incl. decommission record, launch-posts, blog source) committed for the first time; `plan_mcp_consolidation.md` committed; `PRICING.md` moved in from the repo-less top level; `TASK.md` — all 8 tasks cancelled with reasons; this HISTORY entry.
- **Server (27.102.137.74)**: all doriku containers/volumes/images/crons and `/opt/doriku`, `/home/hosting/io/doriku` removed after a verified final backup (pg_dump + env files + scripts + nginx/ssl → founder-held offline archive; nightly R2 dumps also retained). ko-gap volumes untouched. doriku.io / api.doriku.io confirmed down.
- **Local Claude Code**: doriku MCP entry, Stop/PreToolUse hooks, and doriku-session agent removed/archived.

### 3. Technical Summary
Decision basis measured on 2026-07-13: 0 external signups ever (all 3 accounts the founder's own), ~6 total clicks from all launch channels (Threads 4, Reddit 2, dev.to/X 0) — the funnel died at reach (zero-reputation brand accounts; Show HN and r/ClaudeAI never fired), so product validation never started. The founder chose dormancy, then upgraded to full decommission the same day. Post-mortem: the original architecture (N humans × 1 agent each, team-collaboration control plane) failed on GTM (team adoption barrier × zero distribution × non-acute pain), not on architecture; recorded revival direction is a repositioning to a single-user cross-tool agent-fleet control plane ("one person's fleet control tower"), for which the existing primitives (workspace/agent registry/task claim/locks/heartbeat) transfer unchanged.

### 4. Known follow-ups
- User billing decisions only: DAOU VPS cancellation (ko-gap volumes die with it), doriku.io domain renewal vs lapse, Paddle product/coupon archival.
- Local non-code artifacts kept outside git: final backup archive, server SSH key (needed until VPS cancellation), extracted local Claude config remnants.

---

## Archives
<!-- None yet -->
