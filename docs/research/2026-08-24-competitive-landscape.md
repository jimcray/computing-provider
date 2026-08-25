# Competitive Landscape: Inference Marketplace Pricing & Provider Payouts

**Internal — not for external distribution.** Competitor claims are dated 2026-08-24 and rot fast; pricing figures should be re-checked before being repeated.

Observation date: **2026-08-24**. Our side is grounded in this repo at commit `a339a45` and the public Swan Inference API (`/api/v1/models`, `/api/v1/providers`, `/api/v1/subscription/plans`, `/api/v1/stats/subscription-pool`). Prepared to inform [governance discussion #24](https://github.com/swanchain/governance/discussions/24) (token plan tiers, stake-for-quota, provider-set vs uniform pricing).

---

## TL;DR

1. **Every peer that launched a flat inference subscription got burned by free-riders and retrofitted a value cap.** Chutes went free → $3/$10/$20 → "5× PAYG-equivalent value, rolling windows, overflow billed at PAYG" → killed the $3 tier, after accounts ran $6,400 of usage on a $20 plan. Swan's pool is on the same trajectory (August projecting ~44% payout to providers). The fix is known and cheap: a per-subscriber cap denominated in **payout value**, not tokens.
2. **Nobody at our layer lets suppliers set their own per-token price except OpenRouter — and OpenRouter is an aggregator of companies, not GPU owners.** Akash has provider-set bids at the compute layer but chose uniform admin pricing for its own per-token product (AkashML). Chutes' platform formula sets prices; miners can't. Venice, Together, io.net Intelligence: uniform. Uniform-price-per-model is the category norm; the decision is about *routing weight*, not consumer-facing price.
3. **Stake-for-quota has one working reference design: Venice DIEM** — stake VVV → mint a tradeable ERC-20 that redeems for a fixed $1/day of inference credit, no rollover, funded by emissions and offset by subscription-driven burns. It answers the community's ask but the cost of the traffic still has to be paid to providers; Venice can absorb it because it has no supply-side payout.
4. **Swan is not price-competitive on its highest-traffic model.** DeepSeek V4 Flash is $0.14/$0.28 at Swan, Together, Chutes, Venice and AkashML — identical. Swan's real differentiators are the open supply side (single Go binary, no stake, no Kubernetes, 90% pass-through) and the $6 plan — the latter of which is currently unfunded.

## Comparison layer

**The Swan Inference marketplace**: consumer per-token/plan pricing on one side, GPU-owner payout on the other. This repo (`computing-provider`) is only the supply-side client: `conf/config.go:86-90` holds endpoint/API key/model list, `models.json` holds endpoint/GPU memory/category (`conf/config.go:230-234`) — there is **no price field and no client-side pricing control**. `inference recommend-models` (`cmd/computing-provider/inference.go:826-855`) only *displays* admin-set prices and an earnings estimate. Everything the governance discussion is about (plan tiers, pool settlement, routing) is implemented server-side and is out of this repo's reach.

Layer mismatches found:
- **Akash core** and **io.net IO Cloud** are GPU *rental* (a layer below). Both have a per-token product at our layer (AkashML, IO Intelligence) — those are what is compared.
- **Together** is a centralized incumbent with no supply side; it is the price/quality benchmark, not a payout peer.
- **Venice** is a demand-side reseller with no supply side; it is the token-utility benchmark.
- **OpenRouter** is an aggregator whose "providers" are companies (Together, DeepInfra, Groq…), not GPU owners.
- **Chutes** is the only true peer: open-ish supply, decentralized, per-token consumer pricing, subscriptions.

## Category map

| | Consumer pricing | Who supplies GPUs | Supplier paid how | Price set by | Subscription |
|---|---|---|---|---|---|
| **Swan** | PAYG + $6 plan | Anyone (Go client, no stake) | 90% of list per token; plan traffic pro-rated from pool | Admin, uniform per model | Yes (1 tier) |
| Chutes (SN64) | PAYG + $10/$20 | Miners (K8s + TDX hw + TAO hotkey) | Alpha emissions by compute-time score; **no revenue share** | Platform formula from GPU $/hr | Yes, 5×-PAYG cap |
| OpenRouter | PAYG, 5.5% top-up fee | Inference companies (waitlist) | Their own price, invoiced monthly | **Each provider** | No |
| Venice | Free/$18/$68/$200 + PAYG | B2B networks (NEAR, Phala…) | Not disclosed | Admin, uniform | Yes (3 tiers) + DIEM |
| Together | PAYG, $5 min prepaid | Own DCs | n/a | Admin | No |
| AkashML | PAYG, $100 credits | Overclock leases from Akash providers | Lease revenue (compute layer) | Admin, uniform | No |
| io.net Intelligence | Free/Pro/Dev credit bundles | Rented from IO Cloud workers | Hourly rental + IO emissions | Admin | Credit bundles |

## Scorecard

Edge per row. "Swan" cites this repo / public API.

| Dimension | Swan | Chutes | OpenRouter | Venice | Together | Akash/AkashML | io.net | Edge |
|---|---|---|---|---|---|---|---|---|
| Supply-side openness | Any GPU, single binary, outbound WS, no stake (`internal/computing/inference_client.go`) | K8s + TDX + hotkey | Companies only | None | None | K8s (Homenode early access, rewards TBD) | Closed binary + 200 IO/card stake | **Swan** |
| Supplier payout legibility | 90% of list, USD; but plan traffic pro-rated (Aug proj. 44%) | Emissions only, TAO-priced | Provider's own price, monthly invoice | n/a | n/a | AKT only, 0% take post-BME | IO tokens, 2% to USDC | OpenRouter / Swan (PAYG) |
| Consumer price on DeepSeek V4 Flash | $0.14/$0.28 | $0.14/$0.28 | pass-through (varies) | $0.14/$0.28 | $0.14/$0.28 | $0.14/$0.28 | $0.199/$0.486 | tie |
| Flat plan value | $6 → 40M tok/wk, 8 conc. | $10/$20, capped at 5× PAYG | none | $18 unlimited text (app), credits for API | none | none | $15/$150 credit bundles | **Swan** (for buyer) — unsustainable |
| Plan abuse controls | Token + req/day caps only | 5× value cap, rolling windows, model exclusions | 50 → 1,000 req/day on free after $10 lifetime spend | Daily prompt caps, credit metering | n/a | n/a | Tier-gated models, daily refresh | **Chutes** |
| Token utility | UBI reward to providers; Pay-with-SWAN planned | Pay in TAO/alpha, autostake+burn | none | Stake → DIEM $1/day credit, sub burns | none | Burn-mint (ACT credit) | Stake to supply; 0% fee if paying in IO | **Venice** |
| Routing among suppliers | Quality/uptime weighted, one price | Least-connections within a chute | Inverse-square price × uptime; `sort`, `max_price` | n/a | n/a | Geo | n/a | **OpenRouter** |
| Live scale | 122M tok/wk, 4 online providers | ~$16–22K/day rev, ~100B+ tok/day | ~10T tok/day | 1.3T tok/mo | ~$1B ARR | 163 active GPUs | ~$35K/day | everyone else |
| Catalog reality | 58 listed, 12 with any provider, 46 empty | 429s at capacity | 400+ | 200+ | churn to "dedicated only" | 6 models | 33 (29 paywalled) | OpenRouter |

## Per-competitor detail

### Chutes (chutes.ai, Bittensor SN64) — the direct peer
- **Business model.** Consumers pay USD PAYG or subscriptions; miners receive SN64 alpha emissions weighted by compute-units (`INSTANCES_QUERY` in `metasync/constants.py`; `docs/specs/scoring-window.md` 2026-04-13), not a revenue share. Revenue in TAO/alpha is auto-staked and burned (`api/autostaker.py`). VERIFIED (github.com/rayonlabs/chutes-api).
- **Features.** OpenAI-compatible endpoints, user-deployable "chutes", TEE deployments with 2.25× score bonus, `:latency`/`:throughput` suffixes, 90% cache discount, Stripe + TAO. VERIFIED.
- **Pricing (2026-08-24).** DeepSeek-V4-Flash $0.14/$0.28; Qwen3-32B $0.104/$0.416; Kimi-K3 $3/$15. Prices are a platform formula from GPU hourly cost and concurrency (`api/constants.py`), **miners cannot set price**. Subscriptions: Plus $10, Pro $20; free 200 req/day retired 2026-03-15; all plans capped at **5× PAYG-equivalent value on rolling windows, overflow billed PAYG** (chutes.ai/news/community-announcement-february, re-verified today). Base $3 tier closed ~May 2026.
- **Growth.** Volume-first via OpenRouter free routing (~40B tok/day of unprofitable traffic), then "From Volume to Value" pivot (2026-03-20): cutting it raised revenue/GPU 45%. VERIFIED.
- **UX.** Consumer side is fine; miner side is heavy (K8s, Ansible, TDX, hotkey, custom Gepetto strategy). VERIFIED (chutes-miner repo).
- **Weaknesses.** Miner income fully TAO-exposed; scoring rewards uptime on high-multiplier chutes rather than served demand; repeated subscription repricing burned the RP community; chronic 429s.

### OpenRouter — the routing benchmark
- **Business model.** Zero markup pass-through; revenue is a 5.5% fee on credit top-ups ($0.80 min; 5% crypto), BYOK 5% after allowance. Series B at $1.3B (May 2026); **acquired by Stripe, announced 2026-08-19**. VERIFIED (openrouter.ai/docs/faq, /blog).
- **Features.** Default routing: drop providers with outages in last 30s, then inverse-square-of-price weighting; `sort: price|throughput|latency`, `max_price`, `:floor`/`:nitro`, Auto Router. Re-verified today (openrouter.ai/docs/features/provider-routing).
- **Pricing (2026-08-24).** Free models: 20 RPM, 50 req/day (<$10 lifetime credits) or 1,000 req/day (≥$10). No subscription. VERIFIED (docs/api-reference/limits).
- **Supply.** Providers set their own USD prices, must expose `/models` with pricing, uptime gates traffic (≥95% normal; <80% fallback-only), waitlisted. VERIFIED (providers/apply).
- **Weaknesses.** Closed to GPU owners; no flat plan; free tier thin and ephemeral; post-acquisition neutrality risk.

### Venice.ai — the token-utility benchmark
- **Business model.** Privacy/uncensored reseller of 200+ open and closed models; compute bought from NEAR AI, Phala and others; **no supply-side program**. 3.5M users, 1.3T tok/mo, $65M Series A at $1B (2026-07-01). VERIFIED.
- **Pricing (2026-08-24).** Free (10 text/day), Pro $18, Pro+ $68, Max $200 with credit banking; API PAYG e.g. DeepSeek V4 Flash $0.14/$0.28. VERIFIED (venice.ai/pricing, docs.venice.ai/overview/pricing).
- **DIEM.** Stake VVV → sVVV → lock at a rising Mint Rate → DIEM; **1 DIEM = $1/day of credit**, no rollover, ERC-20 on Base, transferable. Emissions cut 6M → 5M VVV/yr; each subscription burns $2/$5/$10 of VVV. VERIFIED (docs.venice.ai/overview/vvv-diem, changelog).
- **Weaknesses.** Inflation-funded liability; late stakers get less; no earnings path for GPU owners.

### Together AI — the incumbent benchmark
- $800M Series C at $8.3B (Jul 2026). Serverless: DeepSeek V4 Flash $0.14/$0.28, Llama 3.3 70B $1.04/$1.04, Qwen3.5 9B $0.17/$0.25. No subscription, no free tier, $5 min prepaid, rate limits dynamic and unpublished. Many open models are dedicated-only. 99.9% multi-DC SLA. No third-party GPU supply. VERIFIED (together.ai/pricing, docs.together.ai/docs/billing).

### Akash / AkashML — the provider-set-pricing reference
- **Compute layer.** Reverse auction; providers run a bid script with USD targets converted to uakt (`helm-charts/.../price_script_generic.sh`). Bid spam attack 2025-03-29 forced tx-fee hikes. AEP-76 BME (final 2026-03-23) removed take rates; providers paid in AKT only. Supply today: 163 active / 432 GPUs, 63 providers (console-api dashboard-data, 2026-08-24). VERIFIED.
- **AkashML (our layer).** Overclock-run, admin-priced, uniform per model: Llama 3.3 70B $0.20/$0.52 (raised from $0.13/$0.40 at Nov 2025 launch), DeepSeek V4 Flash $0.14/$0.28. No provider participation in per-token pricing. VERIFIED (akashml.com).
- **Homenode** (consumer GPUs, no K8s) in early access; reward distribution explicitly unresolved (AEP-60). VERIFIED.

### io.net — the emissions-subsidized-supply reference
- **IO Cloud.** Workers stake `200 IO × multiplier` per card (8×H100 = 16,000 IO), 14-day cooldown, slashing; hourly block rewards paid even when idle; 0.25% reservation fee both sides, 2% USDC / 0% IO payment fee. IDE tokenomics (Jun 2026) target USD payouts with buyback-and-burn. VERIFIED (io.net/docs tokenomics, staking).
- **IO Intelligence (our layer).** 33 models live; DeepSeek V4 Flash $0.199/$0.486; Llama 3.3 70B $0.598/$0.738; 29/33 models paywalled behind tiers; free quotas removed from docs; plans are credit bundles ($15/$150 monthly credits) with unpublished subscription prices. VERIFIED (api.intelligence.io.solutions/api/v1/models).
- **Weaknesses.** 2024 Sybil incident (~1.8M spoofed GPUs); marketing GPU counts vs explorer (~1,199 active, secondary); closed worker binary; heavy capital barrier for suppliers.

## Where we lose

- **Scale and reliability.** 4 providers online, 46 of 58 catalog models with no provider, 91% of traffic on three models each served by 1–2 nodes. Every competitor has orders of magnitude more capacity and Together/OpenRouter publish SLAs; we publish none.
- **Plan economics.** The $6/40M-tokens-per-week plan is cheaper than anything on the market *because providers are subsidizing it* — projected 44% payout in August. Chutes proved this exact plan shape attracts commercial traffic laundering.
- **Price.** We match, not beat, the incumbents on the models that actually get traffic. SIP-003's "50–66% below centralized" holds only for the long-tail models nobody is serving.
- **Catalog honesty.** Listing 58 models with 12 live is the same "phantom capacity" problem SIP-003 called out for GPUs.
- **Token utility.** Pay-with-SWAN is not live; the plan is Stripe-only. Venice, Chutes, io.net and Akash all have live token payment loops.
- **Tier tagging.** `gpt-5.4-mini` is `standard` (plan-covered) at $3.24/M output payout — a single subscriber can drain the pool at 10× the rate of open models. Chutes explicitly excluded frontier models from its cheap tier.

## What to copy

| From | Pattern | Portable? |
|---|---|---|
| Chutes | Cap each subscription at N× its PAYG-equivalent **value** (they use 5×), enforced on rolling daily/4-hour windows; overflow billed at PAYG instead of hard-stopping. | Yes — pure server-side billing logic. |
| Chutes | Exclude high-payout models from cheap tiers; publish which models are covered. | Yes. Fix `gpt-5.4-mini` tier immediately. |
| OpenRouter | Uptime as the primary routing gate (≥95% normal, <80% fallback-only), then a *weight* on price. Consumer sees one price; providers can only price themselves *out* (reservation floor), not *up*. | Yes for the uptime gate; price weighting only once a model has ≥3–5 providers. |
| OpenRouter | Free-tier limit that *unlocks with lifetime spend* ($10 → 20× the daily cap). Converts free riders into paying users without a plan. | Yes. |
| Venice | DIEM shape for stake-for-quota: fixed $/day units, no rollover, transferable, funded by a *declared* emission budget and offset by subscription-driven burns. | Shape yes; funding needs an explicit provider-payment source (Growth Fund or capped emissions) or it worsens pro-rating. |
| Venice | Three tiers with credit banking (Pro / Pro+ / Max) segmenting hobbyist vs agent users. | Yes. |
| io.net / Together | `GET /models` exposing price, latency and throughput per model. Swan already exposes price and `online_providers`; add latency/throughput. | Yes, server-side. |
| Akash | Don't do per-provider consumer prices at the token layer — even the reverse-auction network chose uniform pricing for AkashML. | Decision, not code. |

## Recommendations (cheapest, highest leverage first)

1. **Re-tier `gpt-5.4-mini` (and any model with output payout > ~$1/M) out of plan coverage.** Server-side config change. Stops the fastest pool drain today.
2. **Add a per-subscriber value cap: subscription usage ≤ N× plan price in payout value per month, rolling daily window, overflow → PAYG.** Chutes' N=5 on a $20 plan still lost money; for a $6 plan whose pool is at 44%, start at N≈3. This alone likely brings the pool back above 100%.
3. **Publish plan coverage and the pool ratio on the model page** (server + dashboard). Providers already see a "Token plan" badge; consumers should see which models are plan-eligible before subscribing. `internal/dashboard/` is the client-side surface if the provider dashboard should mirror it.
4. **Add a pool floor from the platform's 10% share** so provider downside on plan traffic is bounded (e.g. never below 80%). Costs the platform at most ~$4–8/month at current scale; buys provider trust that no competitor at our layer offers.
5. **Split into three tiers** (Lite ≈ $3 / Pro $6 / Max ≈ $20) with concurrency and model access as the axis, mirroring Venice; keep uniform per-model pricing.
6. **Keep uniform consumer pricing; add a provider reservation floor and an uptime gate to routing** (OpenRouter's model). Revisit price-weighted routing only when a model has ≥5 online providers — today none does.
7. **Stake-for-quota, if pursued, as DIEM-shaped fixed $/day units with an explicit funding line** (capped SWAN emission or Growth Fund), never from the subscription pool.
8. **Prune or mark the 46 empty catalog entries** so the catalog stops advertising capacity that doesn't exist — the same credibility fix SIP-003 applied to GPUs.
9. **Ship Pay-with-SWAN for the plan itself** (SIP-003 §7) — it is the only token loop Swan has promised and every decentralized peer already has one.

## Uncertainties

- **Swan pool mechanics** are taken from the public FAQ and `/stats/subscription-pool`; whether the platform's 10% is taken before or after the pool is funded was not confirmed.
- **Chutes current quota numbers** (req/day per tier) are no longer published; figures cited are from Aug 2025 and may be stale. Miner GPU count (626) is a secondary citation.
- **io.net** plan prices, RPM limits and live explorer stats could not be fetched (JS-only pages); GPU counts are secondary sources.
- **Akash** provider docs 404'd during research; bid-deposit and audited-attribute details rest on the price script and AEP texts.
- **Venice** provider revenue share is undisclosed; current DIEM mint rate/APY not on primary docs.
- **OpenRouter** provider revenue share is undisclosed; Sacra ARR estimate is secondary.
- **Fastest to rot:** every per-token price (all competitors reprice monthly), the Stripe/OpenRouter roadmap, Chutes tier definitions, Swan's own pool ratio (changes daily).
