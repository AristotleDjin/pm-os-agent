# Agent Line Map: Cortex PM Chief-of-Staff Agent

> Module 1 · The Agent Line

## The workflow, decision by decision

List every discrete decision or action in your agent's workflow, then score each one and place it **above** the line (a human owns it) or **below** (the agent owns it). Borderline calls get an HITL checkpoint.

| Decision / action | Reversibility (H/M/L) | Blast radius (H/M/L) | Measurability (H/M/L) | Above / Below | HITL? |
|---|---|---|---|---|---|
| Pull project state / recent activity from internal systems | H | L | H | Below | · |
| Decide which past updates / context are relevant to a PM task | H | L | M | Below | spot-check |
| Draft the weekly leadership status update | H | L | H | Below | spot-check |
| Decide the tone / commitment level of the update (e.g. promising a date) | M | M | L | Above | required |
| Flag a project as at-risk or an issue as a likely escalation | M | M | M | Below | required |
| Choose which risk call to escalate to a human | M | M | M | Below | required |
| Propose next sprint's stories from the PRD (within cap) | H | L | H | Below | spot-check |
| Post an update, or approve a company-wide one | L | H | L | Above | required |

## Agent anatomy (sketch)

- **Model:** A cheap, fast default model for routine drafting/retrieval steps (pulling activity, drafting updates, proposing story batches); escalate to a stronger frontier model when the critic rejects an output twice, or when a decision lands in the HITL band (at-risk flags, escalation calls) and needs sharper judgment before it reaches a human.
- **Tools:** project + activity lookup (read-only) · past-update search · roadmap access · team norms reference · story proposal tool (hard-capped by `CORTEX_MAX_QUEUE_ITEMS`)
- **Memory:** roadmap, prior decisions, and team norms persist across runs; raw pulled activity and intermediate drafts are purged after each run to avoid stale context leaking into future updates.
- **Loop:** placeholder, defined in M2 loop-spec.md
- **Bounds:** placeholder, defined in M5 bounds-and-evals.md
- **Evals:** placeholder, defined in M5 bounds-and-evals.md

## The golden rule, applied

- **Decide the tone / commitment level of the update** stays above the line because a wrong tone (e.g. promising a date the team can't hit) is hard to walk back and fuzzy to measure after the fact — reversibility and measurability both fail.
- **Post an update, or approve a company-wide one** stays above the line because it's nearly impossible to undo and has a high blast radius if wrong — reversibility is the deciding factor, and no demo quality changes that.

## Hardest call

**Decision:** "Choose which risk call to escalate to a human."

I went back and forth between below-the-line and requiring HITL — Cortex could plausibly judge escalation-worthiness well in a demo. **Blast radius** finally settled it: a wrong escalation call (staying silent when it should have flagged, or crying wolf) is costly to undo either way, so a human stays in the loop before anything is finalized.
