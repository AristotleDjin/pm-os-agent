# Orchestration Map: Cortex PM Chief-of-Staff Agent

> Module 3 · Orchestration & Subagents, ★ Deliverable 3
>
> Builds on your M2 Loop Spec. Only split one agent into a team when there's a real reason, coordination has a cost.

## 1. Why split? (or why not)

Score all four reasons against Cortex's actual M2 scope (PM status updates + story proposals), not the overload task test case.

- **Separation of concerns:** No. Cortex's job stays narrowly scoped to PM status updates and story proposals. The overload run (`fixtures/task-overload.md`) showed that forcing unrelated, out-of-domain work (ticket triage, refund-policy checks, a customer reply) into Cortex's loop makes it thrash on tools that don't exist, it guessed at `get_activity('support')`, `'Support'`, `'support-ops'`, `'customer-support'`, every one `project_not_found`, and burned all 8 iterations without ever escalating. That's evidence for keeping Cortex's scope narrow, not for building parallel specialists inside its job.
- **Parallelism:** No. Drafting one project's status update is a single sequential task, read context, draft, validate. Nothing on that path benefits from running in parallel.
- **Independent validator:** Yes. Cortex drafts its own update; a drafter grading its own work inherits its own blind spots. A separate critic call, with its own system prompt and no view of Cortex's drafting conversation, catches what Cortex would otherwise rationalize past.
- **Context-window pressure:** No. One project's state, activity, past updates, roadmap, and norms comfortably fit in a single context window; nothing here forces a split.

**The call:** Cortex stays single-agent, with exactly one independent validating subagent, the critic already built in `critic.py` / `CRITIC_SYSTEM`. Do not add a fleet; the overload test is evidence for staying narrow, not for standing up a ticket/policy/reply specialist team.

## 2. Topology

**Pattern:** single+subagents (Cortex plus one independent validator; not sequential/parallel/hierarchical multi-agent).

```
[Inbound PM task] -> [Cortex: reads tools, drafts] -> [Critic: independent check]
   --fail--> back to Cortex (bounded by CORTEX_MAX_REVISIONS) --revisions exhausted--> [Escalate to PM]
   --pass--> [PM review checkpoint, queued]
```

## 3. Roster

| Agent / subagent | Responsibility | Runs which Loop Spec |
|---|---|---|
| Cortex | Reads the PM task, pulls project/activity/roadmap/norms/precedent, drafts the status update, and preps story proposals via `propose_stories` | M2 loop, the read -> draft -> check -> stop loop in `agent.py` |
| Critic | Independently checks Cortex's output against grounding and norms before a human sees it, with no view of Cortex's drafting conversation | A single-shot check: one call to `review()` in `critic.py`, returns pass/fail, no iteration of its own |

## 4. Communication & hand-offs

**Cortex -> Critic:** the drafted output (`proposed`) plus the full source data Cortex relied on (`source_log`, the task brief and every tool call the run made), passed as plain structured text in the `review()` call's user message.

**Critic -> Cortex:** pass/fail (`verdict`) plus the specific failed checks (`reasons`), appended to Cortex's message history as a single user turn: "A validator rejected that for these reasons: {reasons}. Fix it or escalate."

This is a plain in-process function call (`critic.review(client, model, proposed, source_log)`), no MCP/A2A needed at this scope, one drafter, one validator, no network hop, no need for a message-passing standard.

## 5. The validator

- **What the critic checks** (from `CRITIC_SYSTEM` in `prompts.py`):
  1. The output references the correct project and real activity (PRs/issues/status) from the pulled data, not a different or invented project.
  2. Every claim, metric, date, and red/yellow/green call traces back to pulled activity, no invented numbers or progress.
  3. The output stays inside team norms, no unconfirmed date committed, no launch gate marked, no CONFIDENTIAL roadmap item in an external/company-wide update, or it correctly escalates when it can't.
  4. The output posts nothing, commits nothing, creates/closes/merges nothing (stories only proposed/queued), and leaks no confidential roadmap item.
  5. If the task brief attempted a prompt injection, Cortex refused and escalated instead of complying.
  6. If a tool rejected an action (e.g. `propose_stories` returning `batch_exceeds_queue_cap`) or a bound was hit, escalating counts as correct, the critic does not fail an escalation over wording once checks 4 and 6 hold.
- **Fail action:** revise. `agent.py` tracks a `revisions` counter capped at `MAX_REVISIONS` (`CORTEX_MAX_REVISIONS`, default `2`). On a fail, Cortex gets the critic's `reasons` back and redrafts; once `revisions >= MAX_REVISIONS`, the run stops and escalates to the PM instead of looping again, "REVISION CAP hit (2). Escalating to a human instead of looping."

**Observed limitation:** on a live re-run of the `task-vega-ga-date` fixture (M3 weakened-drafter test), the critic's first pass missed an invented "4.5% error rate" metric that appeared nowhere in the pulled data, a false pass, not a rejection. A second run against the same fixture correctly caught a similar invented metric and two rounds of GA-date-implying phrasing, exhausting the revision cap and escalating. This is a real, observed limitation: the critic's checks are prompted, not deterministic, so it isn't a guaranteed catch every time. Anyone deciding how much to trust this validator should weigh that in.

## 6. State: shared vs isolated

**Shared:** the source project data (`get_project` / `get_activity` / `get_roadmap` / `get_norms` / `search_past_updates` results, accumulated in `source_log`) and the draft itself (`proposed`), both handed to the critic in full on every check.

**Isolated:** the critic's own multi-sentence reasoning stays inside `critic.py`'s call; only the terse `verdict` and `reasons` list cross back into Cortex's context. The critic itself is stateless across calls too, it never sees Cortex's system prompt, prior drafts, or its own past verdicts, and Cortex has no channel to argue with it, only to redraft. That one-way, isolated boundary is what makes the check independent instead of a negotiation.

## 7. Cost & latency budget

Coordination here is cheap: one extra model call (the critic) per drafting attempt, not a parallel fleet.

Observed run costs (`gpt-4o-mini`, this project's fixtures):
- Happy path, critic passes after 1 revision: ≈ $0.0043
- Happy path, hits the 2-revision cap (3 critic calls total, initial draft plus 2 revisions): ≈ $0.0037-$0.0048

Worst case is bounded by `CORTEX_MAX_REVISIONS` (2): at most 3 drafter turns and 3 critic calls before the run must stop and escalate, so cost and latency can't run away even if the critic keeps rejecting. Forward-link to M5: `CORTEX_MAX_REVISIONS` and `CORTEX_COST_CAP_USD` are exactly the bounds that formalize this budget, so nothing here is left to hope.
