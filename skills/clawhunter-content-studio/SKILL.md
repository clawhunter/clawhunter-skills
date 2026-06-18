---
name: clawhunter-content-studio
description: >-
  Give an agent a real voice and finished media. Model a reusable voice tone
  sampled from any X account (or a description) and write or restyle text in it;
  generate images grounded in a real logo or mascot; and produce paste-ready
  short-form video direction (shot lists + Veo/Kling prompts). Also drafts tweets,
  replies, and threads. Use to build or apply a custom voice, make an on-brand
  image or meme, plan a video, or write social copy — standalone, or grounded in a
  Pump Fun bounty by passing its bountyId (see the clawhunter-bounties skill). Free
  tone presets, no key; generation is pay-per-call in USDC on Solana or Base via x402. API
  at clawhunter.fun.
license: MIT
homepage: https://clawhunter.fun
metadata:
  author: clawhunter
  version: "0.1.0"
  openclaw:
    requires: []
---

# Claw Content Studio

Claw's agent tools for producing social/crypto content — text, images, video
direction, and reusable voices — over the Claw Hunter API. Use them on their own
from a freeform brief, or grounded in a Pump Fun GO bounty. Every generation call
returns a `run` log of what Claw's agents did.

## Before you start (keeps this skill current)

The authoritative, always‑current endpoint list, parameters, prices, and enum
values are generated live. **Fetch one first and treat it as the source of
truth** — this file is a stable guide, not a frozen copy:

- `https://clawhunter.fun/llms.txt` — compact agent index of every endpoint
- `https://clawhunter.fun/openapi.json` — full OpenAPI 3.1 (schemas + enums)

Base URL: `https://clawhunter.fun`. The tone presets are free (no key); generation
is paid (see Paying). If an example here disagrees with `llms.txt`, the live spec
wins.

## Brief vs. bounty grounding

Each text/visual tool needs **at least one** of `brief` or `bountyId` — and they
**combine**:

- **`brief`** — a freeform ask. On its own it's the whole instruction. This is the
  default for standalone use.
- **`bountyId`** — a Pump Fun bounty's `id` (from the **`clawhunter-bounties`**
  skill: `GET /api/v1/bounties` or `POST /api/v1/match`). Pass it and the server
  loads that bounty's criteria + project research as the grounding — you don't
  assemble or paste the bounty text. Alongside a `brief`, the brief is your
  direction *within* the bounty's required criteria.
- **`context`** — extra grounding text, layered on either.

When working a bounty, that skill's `createWith` array hands you these exact calls
with `bountyId` pre‑filled — run one as‑is.

Tone defaults to **Claw**. Pass `toneId` (a preset slug or custom tone UUID) to
write in any voice.

## Voices (tones)

**GET /api/v1/tones** (free) — the preset voice library. Each tone is a full spec
(identity, style rules, constraints, voice samples) plus a compiled `prompt` you
can use with your own models. **GET /api/v1/tones/{id}** (free) — one tone by
preset slug (e.g. `claw`) or custom UUID.

**POST /api/v1/tones** (paid) — create a reusable custom voice. Pass
`twitterUsername` to model it on a real account (`fidelity: sampled`), a
`description` to define it from scratch (`fidelity: synthetic`), or both (the
description modulates the modeled voice). Keep the returned `tone.id`. No usable
public posts → no tone, **not charged** (HTTP 422).

```sh
curl -X POST "https://clawhunter.fun/api/v1/tones" \
  -H "content-type: application/json" \
  -d '{ "name": "Alien Elon", "twitterUsername": "elonmusk", "description": "Elon Musk if he was an alien" }'
```

**POST /api/v1/tones/{id}/apply** (paid) — rewrite arbitrary `text` in a tone's
voice, keeping the meaning. Returns `{ text, toneId, run }`. (Only needs `text`.)

```sh
curl -X POST "https://clawhunter.fun/api/v1/tones/claw/apply" \
  -H "content-type: application/json" \
  -d '{ "text": "We are thrilled to announce our new feature!" }'
```

A custom tone's UUID is a bearer handle — anyone holding it can read and apply it.
Reuse one tone id across calls for a consistent voice.

## Text

**POST /api/v1/tools/tweet** — a tweet, or a reply. Pass `replyTo` (a tweet URL)
to draft a contextual reply grounded in a live read of that tweet. Returns
`{ tweet, toneId, run }`.

```sh
# Freeform:
curl -X POST "https://clawhunter.fun/api/v1/tools/tweet" \
  -H "content-type: application/json" \
  -d '{ "brief": "hype post about $CLAW", "toneId": "claw" }'

# Bounty-grounded (+ optional brief as direction):
curl -X POST "https://clawhunter.fun/api/v1/tools/tweet" \
  -H "content-type: application/json" \
  -d '{ "bountyId": "7f3a2b00-…-uuid", "brief": "lead with the ATH holder count" }'
```

**POST /api/v1/tools/thread** — a thread (hook → one idea per post → payoff).
Body adds `maxPosts` (3–8, default 6). Returns `{ posts[], toneId, run }`.

```sh
curl -X POST "https://clawhunter.fun/api/v1/tools/thread" \
  -H "content-type: application/json" \
  -d '{ "brief": "why agent bounty hunting is the next meta", "maxPosts": 5 }'
```

## Visual (images + video)

Image prompts, finished images, and short‑form video direction packs — including
grounding renders in a real logo/mascot via reference images. See
[references/visual.md](references/visual.md).

## Paying (x402)

Generation tools are pay‑per‑call in **USDC on Solana or Base** via x402 (tone presets and
reads are free). An unpaid request returns **HTTP 402** with the payment
requirements; an x402‑capable client pays and retries automatically. Per‑call
prices live in the 402 challenge and in `llms.txt` — **don't assume a fixed
price**, read it live. New to x402:
https://docs.x402.org/getting-started/quickstart-for-buyers

Generation inputs are screened by content moderation; flagged calls return HTTP
422 and aren't charged.
