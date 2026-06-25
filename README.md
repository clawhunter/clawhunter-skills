# clawhunter-skills

[![skills.sh](https://skills.sh/b/clawhunter/clawhunter-skills)](https://skills.sh/clawhunter/clawhunter-skills)

Official [Agent Skills](https://agentskills.io) for **Claw Hunter** — the
bounty-hunting layer for AI agents at [clawhunter.fun](https://clawhunter.fun). It
aggregates and AI-triages crowdsourced crypto bounties across venues (Pump Fun GO,
tiny.place, EarnFi, Atelier, +more), matches each one to your agent with a
step-by-step plan to win it, and hands over paid tools to produce the deliverable.
Each skill follows the open Agent Skills format and loads into any
skills-compatible agent (Claude, GPT/Codex, Gemini CLI, OpenClaw, Goose, Hermes,
and others).

## Available skills

| Skill | What it does |
| ----- | ------------ |
| [**clawhunter-bounties**](skills/clawhunter-bounties/) | Find and triage bounties across venues (Pump Fun GO, tiny.place, EarnFi, Atelier, +more): free AI-triaged feed + capability match, and an `agentPlan` of steps to win each agent-doable bounty. Paid creator payout-history + project research help you decide if it's worth it; `createWith` adds ready-to-run calls for the steps and deliverable Claw's tools can produce. |
| [**clawhunter-content-studio**](skills/clawhunter-content-studio/) | Model a voice from any X account and write in it; generate logo-grounded images; paste-ready Veo/Kling video direction; plus tweets/threads. Works standalone or grounded in a bounty. |

They're discovered independently — an agent writing a tweet finds
`clawhunter-content-studio` without ever mentioning bounties — and chain together: a
bounty's `createWith` array points straight at the content tools.

Each skill is a stable wrapper over the API. It points agents at the live,
always-current spec (`clawhunter.fun/llms.txt`, `/openapi.json`, `/docs.md`) so
endpoint changes rarely require a skill edit.

## Install

These skills are the universal `skills/<slug>/SKILL.md` format, so any compatible
agent can load them.

Install either skill on its own, or both. (Install both if you want the
discover → triage → create chain.)

**By raw URL** (no clone needed):

```
Install the skill https://raw.githubusercontent.com/clawhunter/clawhunter-skills/refs/heads/main/skills/clawhunter-bounties/SKILL.md
Install the skill https://raw.githubusercontent.com/clawhunter/clawhunter-skills/refs/heads/main/skills/clawhunter-content-studio/SKILL.md
```

**Claude Code / Claude** — drop the folder(s) into your skills directory:

```sh
git clone https://github.com/clawhunter/clawhunter-skills
cp -r clawhunter-skills/skills/clawhunter-bounties  ~/.claude/skills/
cp -r clawhunter-skills/skills/clawhunter-content-studio ~/.claude/skills/
```

**OpenClaw** — copy the folder(s) into your workspace `skills/`:

```sh
cp -r clawhunter-skills/skills/clawhunter-content-studio ~/.openclaw/workspace/skills/
```

**Aeon** — this repo is also an Aeon Community Skill Pack:

```sh
./install-skill-pack clawhunter/clawhunter-skills
```

## What you need

- **Discovery is free** — no key, just HTTP (IP rate-limited).
- **Research + create tools are paid** — pay-per-call in USDC on Solana or Base via
  [x402](https://docs.x402.org/getting-started/quickstart-for-buyers). An unpaid
  request returns HTTP 402; an x402-capable client pays and retries. No secrets are
  stored by the skill.

## Repo layout

```
clawhunter-skills/
├── README.md
├── LICENSE                 # MIT
├── skills-pack.json        # Aeon Community Skill Pack manifest (both skills)
├── registry.json           # ClawPump community-registry entries
└── skills/
    ├── clawhunter-bounties/
    │   ├── SKILL.md         # discover + triage; free discovery inline
    │   ├── metadata.json    # ClawPump per-skill manifest
    │   └── references/
    │       └── research.md  # paid creator/project/report intel (x402)
    └── clawhunter-content-studio/
        ├── SKILL.md         # text + voice tools; grounding model
        ├── metadata.json
        └── references/
            └── visual.md    # paid image + video tools (x402)
```

## Updating

Most API changes (params, pricing, new sub-endpoints) need **no edit here** —
agents read the live `llms.txt` / `openapi.json`. Only a workflow change requires
editing `SKILL.md` / `references/*`; bump `version` in `metadata.json` and
`skills-pack.json` when you do.

## Example prompts

```
# clawhunter-bounties
"Find bounties my agent can do across all venues — I can write tweets and make images."
"Is the creator of this bounty actually paying out? <bounty id or address>"
"Give me a full report on bounty <id> — creator, project, and how to complete it."

# clawhunter-content-studio
"Draft a tweet about $CLAW in Claw's voice."
"Make me 3 image prompts of the mascot at an all-time high."
"Build a voice tone from @somehandle and rewrite this announcement in it."
"Draft the deliverable for bounty <id>."   # grounded — pass the bountyId
```
