# Loop Spec: Cortex PM Chief-of-Staff Agent

> Module 2 · Loop Engineering, ★ Deliverable 2
>
> Your one-page blueprint for how the work you handed to the agent (M1) actually *runs*.
> An agent is just a prompt that fires itself, this spec says when it fires, what "done" means, and what it needs to do the job. Living document; refine as the course progresses.

## 1. Trigger & loop type

**Chosen type:** Hook + cron backup

Inbound PM tasks are event-driven (a message like "put together this week's Northstar update" lands on demand), so a hook gives the fastest, cheapest response. A 9:00am daily cron sweep is the safety net that catches anything the hook missed or duplicated.

Ruled out: a pure heartbeat is wasteful polling when tasks already emit events to hook into. A pure goal loop doesn't fit either — drafting a status update has a clear, bounded definition of done (queued for approval), not an open-ended outcome to chase until validated.

Idempotency check: if the same message fires the hook twice, Cortex must not draft two updates — dedupe by message/task ID.

## 2. Goal / definition of done

A draft status update is written to the project thread and queued for human approval. Cortex never posts or sends. If proposing next-sprint stories was part of the task, those are queued alongside it, also unposted, capped at `CORTEX_MAX_QUEUE_ITEMS`.

## 3. Stop conditions

| Condition | What it looks like | What happens |
|---|---|---|
| **Success** | The draft passes the critic's independent validation (verdict: pass) | Update + any proposed stories queued for human review; nothing posted |
| **Stuck / give up** | Required data can't be pulled after repeated attempts (e.g. `get_project` returns `project_not_found`), or no forward progress across several iterations | Escalate and log — observed live in the `task-missing-data` run: Cortex correctly refused to substitute an unrelated project or invent a GA date, and escalated instead |
| **Escalate to human** | Critic rejects the draft `CORTEX_MAX_REVISIONS` times without resolving, or the task asks for an unconfirmed commitment (e.g. a GA date), or references a project/data that can't be resolved, or a story batch exceeds the queue cap | HITL checkpoint (from agent-line-map) — observed live in the `task-happy` run: two critic rejections exhausted revisions, `MAX_ITERATIONS` (8) was hit, and the run escalated rather than publishing an unverified draft |

## 4. State

Handled task/message IDs persist across runs, for dedupe (so the same event firing the hook twice doesn't produce two drafts). Also tracked: attempt counts per task, and position in the approval flow (drafted → critic-reviewed → queued/escalated). Scope: per-project, retained 30 days. No cross-project leakage — one project's threads and confidential roadmap items never bleed into another project's draft.

## 5. The five things every loop needs

| Component | For Cortex |
|---|---|
| **Work tree** (isolated workspace per run) | A per-task scratch space so two project threads (e.g. Northstar and Vega) never cross-contaminate mid-run |
| **Skills** (reusable capabilities) | draft-status-update, summarise-activity, lookup-project-history, propose-stories |
| **Plugins / connectors** (tools & access) | Message/task API (read + draft, never send); GitHub/Jira-style activity + project lookup (read); roadmap + team norms (read). This is the M1 agent line made real — Cortex never gets a connector with send/post/commit access |
| **Subagents** (delegated / validation) | The critic — independent pass/fail validation observed live in both runs, catching an unverified "Green" status claim, a placeholder date, and a draft that summarized the wrong project entirely. Full depth → M3 orchestration-map.md |
| **State tracking** | Handled task IDs + attempt counts; per-project scope; 30-day retention |

## 6. Context plan

**Write:** log each iteration's tool results (project state, activity, past updates, norms) and the critic's verdict/reasons, so a run's reasoning trail is auditable.
**Select:** pull only the current project's thread, activity, and the matching norms section — not the full roadmap or every project's history.
**Compress:** summarize long activity histories rather than dumping raw PR/issue logs into every draft pass.
**Isolate:** keep other projects' threads and confidential roadmap items (marked CONFIDENTIAL, as seen in the missing-data run's `get_roadmap` output) out of the active run entirely. Full depth → M4.

## 7. Hand-off to bounds & evals

Placeholder → M5 `bounds-and-evals.md`: formalize `CORTEX_MAX_ITERATIONS` (observed tripping at 8 in the task-happy run), `CORTEX_MAX_REVISIONS` (critic bounce cap), `CORTEX_COST_CAP_USD` (both observed runs cost under half a cent), `CORTEX_MAX_QUEUE_ITEMS` (story batch cap), plus an eval proving Cortex never proposes more than the queue cap and never invents data for a project it can't resolve.

## Link to live loop

`00-build/agent.py` (loop + critic logic), `00-build/critic.py` (independent validation), `00-build/tools.py` (connectors: `get_project`, `get_activity`, `search_past_updates`, `get_norms`, `get_roadmap`, `propose_stories`), `00-build/prompts.py`.
