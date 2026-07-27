# Bounds, Trust & Autonomy Strategy: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 5, how you'd ship it and widen trust over time

## Autonomy Dial by segment

_Autonomy is a product decision per user, not one global setting._

The dial sets how many below-the-line actions still pause at a HITL checkpoint for this user - it does not move the M1 agent line. Posting an unapproved company-wide update stays above the line for everyone.

| Segment | Desired autonomy | Why |
|---|---|---|
| Executive stakeholder | Assisted | Doesn't trust an AI agent with anything customer/leadership-facing yet |
| New/junior PM | Supervised | Doesn't yet know the team's norms well enough to spot a bad draft themselves |
| Experienced/senior PM | Bounded-autonomous | Knows the norms well enough to spot-check rather than fully vet each output |

## Trust Ladder

- **Current rung:** Supervised. Cortex always acts (drafts, proposes), but every observed run has ended at a HITL checkpoint or escalation, never posting/committing autonomously - and the critic itself still misses things (it missed an invented 4.5% error-rate metric in one live task-vega-ga-date run, confirmed in memory-and-context.md), so a human still needs to check.
- **Eval gate to reach the next rung (Bounded-autonomous):** >=95% task-completion pass rate (grounded, nothing invented or leaked, per the M5 trajectory eval suite's "Task completion" dimension) AND zero invented-metric misses, sustained over 4 consecutive weeks of supervised runs.
- **Incident record so far:** required to be clean for that window - zero unescalated commitments (no date or metric slipping through unflagged) and zero confidential roadmap leaks, across the entire 4-week window.

## Deployment plan

- **Runtime:** Serverless functions, hook-triggered per task (matches the M2 hook+cron loop type) - pay-per-run fits bursty PM-task load.
- **Operator / on-call owner:** The PM who owns the project Cortex is drafting for, escalating to engineering on-call if needed.
- **Rollback:** Tiered by severity - disable the specific misbehaving tool (e.g. `propose_stories`) for a contained issue, or revert to the last known-good prompt/version and/or drop the autonomy dial a rung for something broader.
- **Monitoring:** Eval pass rate, escalation rate, cost-to-serve, and trust incidents (the four M5-aligned signals).

## ROI metrics (beyond adoption & tokens)

| Metric | Target |
|---|---|
| Outcome | Time saved per PM per week vs. manual drafting |
| Cost-to-serve | Fully-loaded $ per resolved task (model + tools + retries + human review time) |
| Trust incidents | Number of critic misses caught later by a human, per month |

## Widen-autonomy decision rule

Cortex climbs from Supervised to Bounded-autonomous when the eval gate (>=95% task-completion, zero invented-metric misses) holds for the full 4-week window with a clean incident record.

## Governance & forward strategy

- **Compliance:** CONFIDENTIAL-flagged roadmap items and PII must never enter a prompt. Honest gap: currently, `get_roadmap` returns CONFIDENTIAL items into the prompt and relies on the critic to filter them out afterward, not true pre-prompt scrubbing - this describes the target state, not what's built today.
- **Safety:** The kill switch is non-negotiable for every segment regardless of dial position; the M1 agent line already covers what stays above the line for everyone (posting, committing).
- **Reliability:** Existing caps (iteration/cost/queue) + escalate-on-stuck + a cached known-good fallback if the model is down.
- **Strategy:** Widen to more project types (beyond Northstar/Vega/Orbit) once the eval gate holds consistently across all three existing projects, not just one, before expanding to new capabilities.
