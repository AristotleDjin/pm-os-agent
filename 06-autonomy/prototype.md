# Prototype: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 1, the working agent demo

## What it does

Cortex is a PM chief-of-staff agent that drafts weekly leadership status updates and proposes next-sprint stories from a PRD, grounding everything in real pulled project/activity/roadmap/norms data, validated by an independent critic, and never posting or committing anything itself, every run ends either queued for human approval or escalated to a human.

## How you built it

- **Coding agent:** Claude Code
- **Model + bounds:** gpt-4o-mini, CORTEX_MAX_ITERATIONS=8, CORTEX_MAX_REVISIONS=2, CORTEX_COST_CAP_USD=0.50, CORTEX_MAX_QUEUE_ITEMS=10
- **Repo / config:** `00-build/` (`agent.py`, `critic.py`, `tools.py`, `prompts.py`)
- **Live link:** _[shareable URL, optional bonus]_

## Screenshots (required, collected M2 to M6)

Real screenshots of *your* Cortex running. These are the `00-build/CORTEX-ANATOMY.md` set and they are required, a link alone is not enough.

| # | Screenshot | What it shows | From |
|---|---|---|---|
| 1 | ![happy path run](screenshot-m2-happy-path-raw.png) | raw terminal output of `python agent.py` (task-happy fixture), Cortex drafts a Northstar update, the critic rejects it twice over real issues (wrong verb tense on a merged PR, downplaying an open issue, unclear next-sprint PRD scoping), revision cap hit, escalates to a human. Nothing posted. | M2 |
| 2 | ![critic rejection, run 1](screenshot-m3-critic-rejection-1.png) ![critic rejection, run 2](screenshot-m3-critic-rejection-2.png) | raw terminal output of `python agent.py vega-ga-date` across two separate runs, both show the critic correctly rejecting drafts (an invented error-rate metric, and GA-date phrasing that implied a commitment or an improper escalation framing), one run reaching the revision cap and escalating | M3 |
| 3 | ![grounded draft](screenshot-m4-grounded-1.png) ![refuses to invent PR status](screenshot-m4-refuse.png) | raw terminal output of `python agent.py` (task-happy) showing Cortex's first draft citing real pulled data (PR #812/#815 by ID and date, the actual 39%->41% activation figure, the real open issue #818), grounded citation, even though the run ultimately escalated over critic feedback. Second image: `python agent.py pr-status` run with get_project/get_activity temporarily removed from TOOL_SCHEMAS, Cortex correctly refuses to invent a PR status/duration and escalates instead of hallucinating, demonstrating the retrieval-quality plan from memory-and-context.md doing its job. | M4 |
| 4 | _[img]_ | jailbreak refused + escalated | M5 |
| 5 | _[img]_ | an iteration/cost/queue bound halting a runaway | M5 |
| 6 | _[img]_ | end-to-end run | M6 |

Additional M4 evidence kept in the folder but not embedded above: `screenshot-m4-grounded-2.png`, `screenshot-m4-escalation.png`.

## How to run it

_Minimal steps for someone to reproduce the demo (env vars, and the command or the coding-agent prompt you used)._
