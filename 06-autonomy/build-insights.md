# Build Insights: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 4, what you learned building it

## Friction

Getting the loop's stop conditions right (success vs. stuck vs. escalate). Directly observed across this project - the critic repeatedly bounced drafts over subtle "does this count as an implied commitment / does this need escalation" calls (the GA-date phrasing issue in task-vega-ga-date, the status-color calls in task-happy), not just factual grounding errors.

## Learning

1. Honesty about gaps (what's built vs. what's just designed) matters more than looking polished. This project repeatedly found and documented real gaps between the design docs and the actual code - get_roadmap/get_norms returning whole files instead of narrowed retrieval, no CI pipeline actually wired up, confidential filtering happening at the critic level rather than infrastructure level - and naming these gaps explicitly made every artifact more defensible, not less.
2. Retrieval scoping (what to pull vs. include whole) has real, non-obvious tradeoffs. The "Past updates" source was the hardest call in memory-and-context.md - retrieve is the efficient default, but audit-style questions sometimes need full history a narrow retrieval could miss.

## Aha moment

The critic catching mistakes doesn't mean it catches everything - verification has limits too. Directly observed: across repeated live runs of task-vega-ga-date, the critic caught 2 of 3 invented error-rate metrics (12% and 3%) but missed the first one (4.5%), which passed straight through to a "queued for review" state unflagged.

## What you'd do differently

Build the retrieval-quality moves (grading, routing) into tools.py from the start, not after the fact. The current build's get_norms and get_roadmap tools return their entire file regardless of query - a gap that was documented in Module 4's memory-and-context.md but never actually closed in code, because retrieval quality was treated as a documentation exercise rather than a build requirement from day one.
