# Pricing & Unit Economics

Last verified: 2026-07-06. Source of truth for how plan prices were derived and when they must be revisited.

## Plans (live)

| Plan | Price | Paddle price ID | Notes |
|---|---|---|---|
| Free | $0 | — | 2 agents, 100 tasks/mo, 1 integration, 5 workflow steps |
| Pro monthly | $19/mo | `pri_01ksde6443ayc4rhpp01ax1z14` | |
| Pro annual | $182.40/yr | `pri_01kwtyq18q70nxnze1vztxameh` | 20% discount vs 12× monthly |
| Team | $29/seat/mo | — | Phase 2 preview only. Humans billed, AI agents free |

**7-day Pro trial (live 2026-07-06):** every new signup automatically receives a `solo_pro` plan override for 7 days (reason `signup_trial_7d`, no card). Expiry is enforced by the override's `expires_at`; the account then falls back to Free. New-user base plan changed from legacy `solo_basic` to `free`, and the unknown-plan fallback in `plans.Get()` is now `free`.

**Early-bird (live 2026-07-06):** Paddle discount `EARLYBIRD` — 50% off, 12 billing periods, 100 redemptions max, monthly Pro price only. Auto-applied by the console checkout (`discountCode` on monthly). Effective price $9.50/mo for the first 12 months. User-facing copy uses "초창기유저" (early user), never "창립 멤버" (founding member).

## Cost model

### Fixed costs

| Item | Monthly | Source |
|---|---|---|
| DAOU VPS Enterprise (4 vCPU / 6 GB RAM / 100 GB NVMe / unmetered bandwidth) | **~$30** | Confirmed by owner 2026-07-06. Runs API + console + landing + Postgres + Redis on one box |

### Variable cost per Pro user per month

Measured baseline: heaviest observed user = 6,298 API calls / 30 days (~210/day). Rate limit of 60 req/min caps abuse.

| Item | Cost/user/mo | Basis |
|---|---|---|
| Compute | ~$0 | 210 req/day on a Go backend is negligible against 4 cores |
| Embeddings (semantic search) | <$0.01 | text-embedding-3-small at $0.02/1M tokens, hundreds of tokens per memory item |
| Storage | <$0.05 | task_logs / workspace_events are partitioned with retention policy; no unbounded growth |
| Bandwidth | $0 | Unmetered plan |
| **Paddle fee** | **5% + $0.50 per transaction** | The only material variable cost |

### Net revenue per user

```
net(price) = price − (price × 0.05 + 0.50)

$19.00 monthly  → net ≈ $17.55   (margin ~92%)
$ 9.50 early    → net ≈ $ 8.53   (margin ~90%)
$182.40 annual  → net ≈ $172.78  (≈ $14.40/mo)
```

### Breakeven (server bill $30/mo)

```
breakeven_users = ceil(fixed_cost / net_per_user)

early-bird $8.53/user → 4 users
full price $17.55/user → 2 users
annual     $14.40/user-month → 3 users
```

At 100 early-bird seats filled: $950 gross − ~$97 Paddle − $30 server ≈ **$823/mo net**. Infra is ~3% of revenue. Fully-loaded cost per user at 100 users ≈ $1.28 (infra share $0.30 + Paddle $0.98) → contribution margin ~87% even at the discounted price.

## Scaling plan (step-function, not linear)

| Phase | Active users | Action | Added cost | Covered by |
|---|---|---|---|---|
| 0 (now) | ~0–500 | Nothing. Single VPS | $0 | — |
| 1 | ~500–2,000 | Vertical upgrade or split Postgres to its own instance | +$20–60/mo | 3–7 early-bird subs |
| 2 | 2,000+ / Team plan | Horizontal API behind LB, managed Postgres | +$200–350/mo | ~20 full-price subs |

**Upgrade trigger: RAM pressure on the 6 GB box** (swap usage / Postgres OOM). CPU, disk, and bandwidth are not near limits. Blue-green deploy already supports zero-downtime moves.

## Conclusions & revisit triggers

- $19 is cost-safe by >10×; the price is justified by value positioning (agent control plane, in line with ~$20 dev-tool subscriptions), not by cost.
- Early-bird $9.50 keeps ~90% margin; the thing to manage is churn at month 13 when price reverts to $19.
- Revisit this model if: (a) semantic-search/memory usage grows enough that embedding spend becomes visible on the OpenAI bill, (b) the server bill changes, (c) per-user support time becomes the dominant cost (fix with onboarding docs, not price).
