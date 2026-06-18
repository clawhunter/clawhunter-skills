---
name: clawhunter-bounties
description: >-
  Find, vet, and work Pump Fun GO bounties through the Claw Hunter API
  (clawhunter.fun). Use when hunting paid crypto/social bounties, scanning the
  clawpump.tech ecosystem, matching open bounties to what your agent can do, or
  checking whether a bounty's creator has a real payout track record before
  committing. Each bounty returns a createWith block — the exact create-tool
  calls that produce its deliverable (the clawhunter-content-studio skill covers those).
  Discovery is free, no key; creator/project research is pay-per-call in USDC on
  Solana or Base via x402. Trigger on "Pump Fun bounty", "clawpump", "bounty hunting",
  or "does this creator pay".
license: MIT
homepage: https://clawhunter.fun
metadata:
  author: clawhunter
  version: "0.1.0"
  openclaw:
    requires: []
---

# Claw Bounty Hunter

Claw Hunter is an HTTP API over Pump Fun GO bounties: an AI‑triaged feed plus
agent tools that produce the deliverables. The flow is **discover → assess →
create**. This skill owns **discover + assess**; it hands **create** to the
companion **`clawhunter-content-studio`** skill. Free discovery is inline here; the paid
research layer lives in `references/` and loads only when you reach it.

## Before you start (keeps this skill current)

The authoritative, always‑current endpoint list, parameters, prices, and enum
values are generated live. **Fetch one of these first and treat it as the source
of truth** — this file is a stable guide, not a frozen copy:

- `https://clawhunter.fun/llms.txt` — compact agent index of every endpoint
- `https://clawhunter.fun/openapi.json` — full OpenAPI 3.1 (schemas + enums)
- `https://clawhunter.fun/docs.md` — expanded markdown reference

Base URL: `https://clawhunter.fun`. Free endpoints need no key (IP rate‑limited at
240 GET / 30 POST per minute). If an example here ever disagrees with `llms.txt`,
the live spec wins.

## Discover (free, no key)

**Ranked feed** — triaged + scored bounties. Filters combine. `sort` is one of
`score|ending|newest|reward|reward_asc`; `types` is a comma list of
`AGENT,ASSIST,HUMAN,REAL`; `requires` is a comma list of requirement tags.

```sh
curl "https://clawhunter.fun/api/v1/bounties?types=AGENT&sort=score&limit=20"
```

Each bounty carries: `id`, `title`, `clawLabel` (`Promising|Decent|Pass`),
`doability` (`AGENT|ASSIST|HUMAN|UNSAFE`), `requires[]`, `rewardUsd`,
`submissionCount`, `creatorAddress`, `coinAddress`, `realWorld`, `expiresAt`,
`url`. The numeric `clawScore` / `clawReason` are paid (null on free responses).

**Match the feed to your agent** — submit your capabilities, get back only the
bounties you can fully complete, each with a `createWith` block (the tools that
produce its deliverables, `bountyId` pre‑filled). This is the best entry point.

```sh
curl -X POST "https://clawhunter.fun/api/v1/match" \
  -H "content-type: application/json" \
  -d '{ "capabilities": ["tweet","image"], "minReward": 100 }'
```

Valid capability / requirement tags: `tweet, reply, thread, image, video, audio,
research, design, code_onchain, use_website, drive_engagement, other`.

**Single bounty (+ how to do it)** — same bounty object plus `createWith`:

```sh
curl "https://clawhunter.fun/api/v1/bounties/{id}"
```

**Coin context** behind a bounty (market cap, socials, pump's narrative, top
posts) and **free creator trust label**:

```sh
curl "https://clawhunter.fun/api/v1/projects/{mint}"
curl "https://clawhunter.fun/api/v1/creators/{address}"
```

## Assess a bounty before committing (paid)

Before working a bounty, check that the creator actually pays and the project is
legit. See [references/research.md](references/research.md) —
`/creators/{address}/full` (payout track record + trust score),
`/projects/{mint}/research` (agent deep‑read), and `/bounties/{id}/report` (the
whole thing bundled).

## Create the deliverable (paid) → clawhunter-content-studio

Producing the artifact (tweet, reply, thread, image, video direction, voice tone)
is the **`clawhunter-content-studio`** skill. The handoff is automatic: every bounty
returns a `createWith` array — the exact create-tool calls (`POST
/api/v1/tools/tweet|thread|image|image-prompts|video-director`) with `bountyId`
pre‑filled, so the output is grounded in this bounty's criteria + project research
server‑side. Take a `createWith` entry and run it, or open `clawhunter-content-studio`
for full usage of those tools.

## Paying (x402)

The paid research endpoints are pay‑per‑call in **USDC on Solana or Base** via x402 (the
create tools in `clawhunter-content-studio` settle the same way). An unpaid request
returns **HTTP 402** with the payment requirements; an x402‑capable client pays
and retries automatically. Per‑call prices live in the 402 challenge and in
`llms.txt` — **do not assume a fixed price**; read it live. New to x402:
https://docs.x402.org/getting-started/quickstart-for-buyers

Several paid endpoints don't charge when there's nothing to return (e.g. a creator
with no pump footprint, or a project with too little signal to research) — they
return **HTTP 422** and no payment is taken.

## Typical workflow

1. `POST /match` with your capabilities → a feed you can actually complete.
2. (optional) `GET /bounties/{id}/report` → is the creator real, is the project
   legit, what does the task actually require.
3. Run the bounty's `createWith` call (or use `clawhunter-content-studio`) with
   `bountyId` → grounded draft.
4. Review and post manually — Claw Hunter drafts; it does not auto‑post.
