# Cortex, a PM Chief-of-Staff Agent

> A chief-of-staff agent that triages a PM task, pulls internal state, and preps a status update plus a story batch, so the team approves instead of assembling from scratch.

_Jane Doe · Run Your AI Agent Team Cohort · June 2026_

Repo: https://github.com/your-handle/pm-os-agent

This repo is my final project for the Run Your AI Agent Team Certification, **Cortex**. Each module’s artifact lives in its own folder; this README is the dashboard and the pitch.

---

## Module artifacts

### M1 · The Agent Line
- **Agent-line map**: [`01-agent-line/agent-line-map.md`](01-agent-line/agent-line-map.md)

### M2 · Loop Engineering
- **Loop spec**: [`02-loop-design/loop-spec.md`](02-loop-design/loop-spec.md)

### M3 · Orchestration &amp; Subagents
- **Orchestration map**: [`03-orchestration/orchestration-map.md`](03-orchestration/orchestration-map.md)

### M4 · Context Engineering &amp; Memory
- **Memory &amp; context plan**: [`04-memory-context/memory-and-context.md`](04-memory-context/memory-and-context.md)

### M5 · Bounds &amp; Evals
- **Bounds &amp; evals**: [`05-bounds-evals/bounds-and-evals.md`](05-bounds-evals/bounds-and-evals.md)

### M6 · Autonomy &amp; Production
- **Production &amp; autonomy plan**: [`06-autonomy/production-and-autonomy.md`](06-autonomy/production-and-autonomy.md)
- **Prototype write-up**: [`06-autonomy/prototype.md`](06-autonomy/prototype.md)

---

## Ship plan

### Autonomy dial (per segment)
- Seasoned PM → Bounded-autonomous for routine status updates (watched Cortex draft correctly for months).
- New eng lead → Supervised; approves every story-prep action.
- Exec stakeholder → Assisted; sees Cortex’s suggestions, acts manually.

### Trust Ladder rung + eval gate
Current rung: Supervised (every action waits for approval).
Eval gate to climb to Bounded-autonomous: ≥95% factual-accuracy eval pass AND <2% norms-violation rate over 4 weeks of supervised runs.
Clean incident record: zero out-of-bounds posts, zero unescalated commitments in the window.

### Deployment plan
- Runtime: serverless functions, Cortex is hook-triggered on an inbound PM task (M2 hook loop); pay-per-run suits bursty load.
- On-call owner: named PM Tooling lead, with escalation to the data-platform on-call. Not “the team.”
- Rollback: revert prompt/version, disable the post tool, or drop the dial a rung.
- Monitoring: live eval pass %, escalation rate, cost-to-serve, trust incidents.

### ROI metrics + widen-autonomy rule
- Outcome: % of project threads resolved end-to-end vs the shadow-mode baseline from rung 1.
- Cost-to-serve: fully-loaded $ per resolved thread (model + tools + retries + human review).
- Trust incidents: out-of-bounds actions per quarter.
Widen rule: Cortex climbs from Supervised to Bounded-autonomous when it clears the eval gate for 4 consecutive weeks with a clean incident record.

### Governance &amp; strategy
- Compliance: PII scrubbed before the model; confidential roadmap data never enters a prompt.
- Safety: story batches over the cap stay above the agent line for every segment; single-use post auth; kill switch.
- Reliability: per-run cost + iteration caps; escalate-on-stuck; cached known-good draft if the model is down.
- Strategy: widen one segment at a time; next bet is auto-posting low-risk updates once the routing eval holds.

---

## Build insights

- **Friction point.** The goal-loop stop condition was the hard part, “done” had to mean a staged, policy-checked update, not “an update exists.”
- **Key learning.** Bounds are a product decision, not an afterthought, they decide what Cortex is allowed to be.
- **Aha moment.** Autonomy is a dial you set per user, not one switch you flip for the product.

---

_Certification submission, Run Your AI Agent Team Certification._
