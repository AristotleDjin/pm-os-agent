# Memory & Context: Cortex PM Chief-of-Staff Agent

> Module 4 · Memory & Context

## 1. Context budget

Each loop iteration receives: the current task brief (in full), the target project's state (`get_project`), a narrowed slice of that project's recent activity (`get_activity`), the matched section of team norms for the query at hand (`get_norms`), a narrowed slice of past updates for precedent (`search_past_updates`), and a narrowed slice of the roadmap scoped to the project in question (`get_roadmap`). Nothing from other projects enters the context — no cross-project bleed, per the M1 agent-line and M3 isolation requirement. Priority order: task brief and project state first (define what's being asked and about what), then activity and norms (ground the draft and its tone), then past updates and roadmap (precedent and forward context) last, since they're the most replaceable if the context window gets tight.

## 2. Retrieve vs. long-context: per source

| Source | Size / volatility | Decision | Why |
|---|---|---|---|
| Roadmap (`get_roadmap`) | Medium, changes occasionally, contains CONFIDENTIAL items | Retrieve | Must stay current and must respect confidential flags — narrowing to the relevant project's section supports both. (Honest gap: the current tool returns the whole file regardless of query — see Section 3.) |
| Recent activity (`get_activity`) | Large, grows continuously | Retrieve | Far too big to include whole; narrow to this project's relevant window, not the full engineering history |
| Past updates (`search_past_updates`) | Large and growing | Retrieve | Unbounded archive; pull only updates relevant to this project/query for tone and precedent, not the whole corpus |
| Team norms (`get_norms`) | Medium, changes occasionally, must be current | Retrieve | Must cite the *specific* rule that applied, not the whole playbook — narrowing supports citation and audit |
| This week's task brief | Bounded, this run only | Long-context | Small and live; no benefit to retrieving from something this size |
| Project state (`get_project`) | Small, one record | Long-context | Bounded and structured; nothing to chunk in a single project's status fields |
| The PRD under review | One document, static for this task | Long-context | Bounded; want the model reasoning over the whole thing at once |

**Flagged borderline call (from Part A):** Past updates. The tension: retrieve is the efficient default, but an audit-style question ("has this ever come up before?") sometimes needs the *full* history, not just a recent slice — a naive retrieval could miss an old but relevant precedent. The rubric's citation/audit factor settles it: retrieving a specific, citable precedent is more defensible than dumping the whole archive into context and hoping the model finds the right thing on its own.

## 3. Retrieval quality plan

| Retrieved source | Routing | Grading | Rerank | Self-verify | Cache |
|---|---|---|---|---|---|
| Recent activity (`get_activity`) | · | ✓ | · | ✓ | · |
| Past updates (`search_past_updates`) | ✓ | ✓ | ✓ | · | ✓ |
| Roadmap (`get_roadmap`) | · | ✓ | · | ✓ | ✓ |
| Team norms (`get_norms`) | ✓ | ✓ | · | ✓ | ✓ |

- **Routing**: needed for `search_past_updates` and `get_norms`, since each is queried differently depending on intent (a tone/precedent lookup vs. a specific rule lookup) — one query shape doesn't fit both.
- **Document grading**: needed everywhere retrieval happens. Directly observed in this project: across three live `task-vega-ga-date` runs, Cortex invented error-rate metrics that appeared nowhere in `get_activity`'s actual pulled data — 4.5%, 12%, and 3%. The critic caught two of the three (12% and 3%) but missed the first (4.5%, a false pass) — that's precisely the plausible-but-wrong failure grading exists to catch, and live evidence the catch isn't yet reliable, currently happening only after the fact via the critic, not at the retrieval step itself.
- **Reranking**: needed for `search_past_updates` specifically, since a growing archive means the most relevant precedent could get buried under more recent but less applicable updates.
- **Self-verification**: needed wherever citation/audit matters or a wrong answer is costly — `get_activity` (grounding claims), `get_roadmap` (confidential leak risk), and `get_norms` (citing the wrong rule with false confidence is worse than not citing one).
- **Caching**: safe for `search_past_updates`, `get_roadmap`, and `get_norms`, since none of them change per-run; unsafe for `get_activity`, which changes too frequently to cache safely.

⚠️ **Honest gap, not glossed over:** none of these retrieval-quality moves are actually implemented in the current build. `get_norms` and `get_roadmap` return their entire file every run regardless of query (confirmed directly in Module 3's code review), and "grading" currently happens only implicitly and after the fact, via the critic's post-hoc check on the finished draft — not at retrieval time. This table describes the intended plan; closing the gap between this and the real `tools.py` is genuine future work.

## 4. Memory map (your PM brain)

| Memory type | What Cortex stores | Scope / TTL |
|---|---|---|
| **Working** (in-loop) | The current task brief, the PRD section under review, and this run's pulled activity/roadmap/norms slices | This run only — discarded once the HITL checkpoint or escalation is reached |
| **Episodic** (past runs) | Prior status updates and decisions for this specific project thread, drawn on by `search_past_updates` | Per-project thread; retained 30 days (per the M2 loop-spec's existing state retention figure); ages out after |
| **Semantic** (durable facts/prefs) | Durable project facts: project ID, PRD linkage, standing team norms, approval routing rules | Long-lived; updated deliberately, never overwritten by a single run's draft |
| **Shared** (across agents) | The critic's pass/fail verdict and failed-checks list, handed back to Cortex; the critic's own reasoning never crosses back | Scoped to one drafting attempt; discarded once the revision or escalation resolves |

## 5. Memory risks & mitigations

| Risk | Mitigation |
|---|---|
| Drift | Treat semantic memory as a cache, not a source of truth — always re-fetch `get_norms`/`get_roadmap` live on each run rather than trusting a stored copy that may have quietly diverged |
| Poisoning | Only critic-validated drafts ever get treated as precedent in episodic memory; an unvalidated or rejected draft (e.g. one with an invented metric) never gets written back as "what happened" |
| Staleness | TTL on episodic memory (30 days); volatile facts like GA dates and Sev-1 status are always re-fetched live from `get_project`/`get_activity`, never trusted from a stored summary alone |
| Confidential / retention | Scope memory strictly per-project — no cross-project bleed, per M1's agent-line and M3's isolation requirement; roadmap items flagged CONFIDENTIAL are excluded from anything that could reach an external or company-wide update (currently enforced at the prompt/critic level, a known gap flagged in M3 to close at the infrastructure level) |
